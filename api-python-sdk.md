# API: Python SDK

{% hint style="info" %}
**Procurando a API completa?** Esta página é um tutorial prático. Todas as classes públicas, métodos, assinaturas exatas e exemplos prontos para copiar e colar estão na [Referência do SDK](reference/sdk-reference.md), que é otimizada para assistentes de IA.**Trabalhando com um assistente de IA?** Cole este URL no chat para que ele tenha a API completa e atualizada do Chloros 1.3.0:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Cada página deste manual está disponível como markdown bruto em seu slug em letras minúsculas + `.md`, e o manual completo está indexado em `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

O **Chloros Python SDK** (`chloros-sdk` no PyPI) controla tudo o que o aplicativo de desktop pode fazer a partir do Python: processamento de imagens em lote, controle em tempo real da câmera LATTICE e da matriz, sessões de aquisição de dados (DAQ) com sensores de luz e automação de projetos salvos. Trata-se de uma camada fina sobre o mesmo backend local que a GUI e a CLI utilizam (HTTP em `127.0.0.1:5000`), de modo que o comportamento é idêntico nas três interfaces.

## Instalação

O SDK vem junto com o pacote para desktop do Chloros, que também fornece o backend de processamento e os runtimes de hardware, e sua versão sempre corresponde à do pacote.

**Passo 1 — Instale o Chloros.** Windows: execute o instalador para desktop (caminho padrão `C:\Program Files\Chloros\`) a partir da página [Download](download.md). Linux: instale o pacote `.deb` com o `sudo apt install ./chloros_1.3.0_<arch>.deb` ([Instalação no Linux](linux/linux-installation.md)).**Etapa 2 — Verifique o SDK** (Python 3.7+). Normalmente, não há nada para instalar:

* **Windows:** o instalador instala o wheel do SDK incluído no Python do seu sistema (se não houver Python instalado, ele pula esta etapa).
* **Linux:** o `.deb` instala o SDK para o sistema `python3` no `/usr/lib/python3/dist-packages`. Seus pré-requisitos, `python3-numpy` e `python3-requests`, são recomendados pelo pacote: o `apt install` os inclui, enquanto o `dpkg -i` não.
* **Um virtualenv** não reconhece os pacotes do sistema; instale o wheel incluído nele: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` no Linux, ou o `chloros_sdk-*.whl` na pasta `sdk` da pasta de instalação do Windows (`C:\Program Files\Chloros\sdk\` por padrão).

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"   # Windows: py -c "…"
```

Em uma máquina sem o pacote Chloros, cujo SDK aciona um backend em outra máquina (consulte [O que uma instalação apenas com pip oferece](#what-a-pip-only-install-gets)), instale-o a partir do PyPI: `pip install chloros-sdk`. O PyPI é atualizado nas compilações de lançamento, de modo que corresponde à versão estável mais recente.

**Etapa 3 — Faça login uma vez por máquina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache no `~/.chloros/` (em ambas as plataformas). No Windows, você pode fazer login de forma equivalente na guia “Usuário” <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> do aplicativo para desktop. O SDK requer um plano pago do Chloros+ — consulte [Requisitos de licença](#license-requirement) abaixo.

| Requisito | Detalhes |
| --- | --- |
| **Chloros instalado** | Windows: instalador para desktop; Linux: pacote `.deb` (fornece o binário de back-end e o SDK) |
| **Python** | 3.7 ou superior (desenvolvido/testado na versão 3.10) |
| **Sistema operacional** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS ou mais recente, ou NVIDIA Jetson (JetPack 6) |
| **Licença** | Login ativo no Chloros+, qualquer plano pago (Copper ou superior) |

## A vitória em 60 segundos

Uma única chamada cria um projeto, importa uma pasta, configura o processamento e executa o pipeline — iniciando automaticamente o backend caso ele ainda não esteja em execução:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)
```

(No Linux, use caminhos do Linux: `/home/user/drone_images/flight001`. O SDK funciona da mesma forma nas duas plataformas.)

Processando uma pasta de captura do LATTICE? Use o wrapper compatível com o LATTICE — ele aplica os padrões corretos (sem detecção de painel-alvo, debayer padrão):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controle total do pipeline

Para qualquer coisa além de um comando de uma linha, use o `ChlorosLocal`. Ele utiliza um backend que já esteja em execução (o do aplicativo para desktop) ou inicia o instalado na primeira utilização (`auto_start_backend=True`; no Windows, a partir da pasta `backend` da pasta de instalação do Chloros; `C:\Program Files\Chloros\backend\` por padrão), cria e configura projetos, monitora o andamento e retorna um resumo após a execução.

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

{% hint style="info" %}
Mantenha o `http://127.0.0.1:5000` padrão em vez de substituí-lo por `localhost` — no Windows, o `localhost` é resolvido primeiro como `::1` e leva cerca de 2 segundos por solicitação no backend apenas para IPv4.
{% endhint %}

Use-o como um gerenciador de contexto. Sair do bloco encerra o uso do backend por esta instância, não o backend em si: ele permanece em execução para o próximo script e se desliga automaticamente após 30 minutos sem uso. `shutdown_backend()` encerra uma instância já iniciada.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` aceita estas palavras-chave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. Os principais valores:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (8-bit)"            # 8-bit TIFF; what all-8-bit images get for a deeper format
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Os controles específicos do LATTICE (`input_level`, `radiometric_output` e a família `array_alignment*`) estão documentados com suas tabelas completas de valores na [Referência do SDK](reference/sdk-reference.md#supported-values).

### Acompanhando o andamento

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Lendo o resumo pós-execução — e identificando execuções vazias

Após a conclusão, o `process()` anexa o resumo de processamento do backend como `result["summary"]`. Cada entrada no `summary["hints"]` é uma frase completa explicando qualquer aspecto digno de nota — por exemplo, por que uma execução produziu zero saída — e cada dica também é reemitida como um `UserWarning` em Python, de modo que as execuções vazias são autodiagnosticáveis, mesmo que você nunca inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` não é acionado quando uma execução não produz imagens.** Este é o único ponto em que o SDK e a CLI diferem deliberadamente: `chloros-cli process` trata “produtos foram solicitados, nenhum foi gravado” como uma falha e sai com um valor diferente de zero, enquanto o SDK retorna normalmente e relata a condição por meio de `summary` / dicas. Se seu pipeline precisar ser interrompido em uma execução vazia, verifique isso você mesmo — inspecione o `summary` (ou conte os arquivos na pasta do projeto) em vez de confiar em uma exceção.
{% endhint %}

## Smart Connect — hardware ativo

Três auxiliares abrem sessões persistentes no pool de hardware do backend — o mesmo pool que a CLI, outros scripts e as abas de câmera da GUI utilizam, de modo que os scripts do SDK coexistam com eles sem disputar portas seriais ou largura de banda da rede. Todos os três iniciam automaticamente um backend local caso nenhum esteja em execução. O painel DAQ da GUI não faz parte do pool: feche um sensor de luz na GUI antes de abri-lo a partir de um script.

### Câmera LATTICE única — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

{% hint style="info" %}
**Os caminhos relativos apontam para o local onde seu script é executado (alterado na versão 1.3.0).** Com um backend local, um caminho relativo `output_dir` (incluindo o padrão `"output"`) — e os outros caminhos que o backend abre (`burst_dir`, `profile_path`, `save_path`, `mask_path`) — são resolvidos em relação ao diretório de trabalho do seu script, de modo que o `cam.capture("output/")` grava no `<current folder>/output/`. Na versão 1.2.0, um backend instalado os resolvia em sua própria pasta de instalação e apresentava a falha “Acesso negado”. Em um backend remoto, os caminhos são enviados exatamente como digitados.
{% endhint %}

### Matriz sincronizada — `connect_array`

`connect_array` é o ponto de entrada recomendado para setups com várias câmeras. Ele executa o mesmo fluxo de preparação inteligente que a GUI: análise de rede, seleção automática da camada de sincronização, sincronização de tempo PTP, seleção do formato de pixel por câmera, inicialização do AE e ativação do gatilho GPIO. A **primeira porta serial é a mestre** (ela dispara o pulso de acionamento do hardware); as demais são escravas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adicione `smart=True` a qualquer captura em matriz para aguardar que a exposição automática se estabilize em todas as câmeras antes do disparo.

Novidade na versão 1.3.0: um `ArraySession` também fornece quadros sincronizados diretamente, sem gravar arquivos:

* `arr.grab()` retorna um quadro por câmera, todos a partir de um único disparo, como matrizes numpy (`{serial: array}`); `.group_id` é o acionador compartilhado e `.errors` identifica qualquer câmera que tenha ficado de fora.
* `arr.frame_stream(fps=5.0)` gera um conjunto desse tipo por tick, para processamento em tempo real em seu próprio pipeline.
* `arr.continuous("output/")` executa a captura contínua do próprio backend (o modo Contínuo da GUI) e retorna um `ContinuousHandle` com `status()`, `wait()` e `stop()` — use-o em vez de um loop de captura quando precisar que cada acionamento seja salvo em alta taxa.

Para modos de captura (Único / Contínuo / Intervalo / Mais rápido), gravadores, burst-to-video e alinhamento de matriz, consulte a [Referência do SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sem argumentos, o `connect_daq_sensor()` detecta automaticamente o protocolo de transporte (ordem de prioridade: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada quadro contém o `spectrum` de 135 pontos (W/m²/nm quando calibrado), um sinalizador `is_saturated` e CIE `x`, `y`, `z`. Para especificar um sensor ou meio de transporte específico, passe uma dica explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Observe que os perfis de correção de cap (`cap_id`) **não** são um controle do SDK — selecione-os por meio de `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projetos salvos — `open_project`

Um projeto Chloros salvo mantém o hardware conectado (`cameras.json` + `sensors.json` juntamente com `project.json`), e o `chloros_sdk.open_project(path)` pode reconectar suas câmeras, matrizes e sensores de luz de uma só vez e acionar capturas pelo nome do dispositivo. Os sensores de luz se reconectam por meio de um backend do Chloros em execução, com suas configurações e capacidade salvas. Consulte [Automação de Projetos](reference/sdk-reference.md#project-automation--chlorosproject) na referência.

## O que uma instalação apenas com o pip oferece

Verifique os sinalizadores de disponibilidade no nível do módulo antes de usar superfícies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Em um host com **apenas** o `pip install chloros-sdk` e sem o pacote de desktop do Chloros:

* `ChlorosLocal`, `process_folder` e `process_lattice_capture` **não** funcionam — eles precisam do binário de backend que vem no instalador do desktop.
* Os auxiliares de conexão inteligente (`connect_camera`, `connect_array`, `connect_daq_sensor`) são clientes HTTP puros, portanto funcionam com um backend em outra máquina — mas os backends fornecidos se ligam apenas ao loopback; portanto, você deve redirecionar a porta manualmente (por exemplo, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passar o `backend_url="http://127.0.0.1:5000"` junto com o `auto_start_backend=False`. Consulte [Modo de Backend Remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* As classes LATTICE de hardware direto (`LatticeCamera`, `CameraPool`, …) são importadas, mas requerem o tempo de execução do SDK da Arena, proveniente do pacote para desktop — sem ele, `CAMERA_AVAILABLE` é `False`.
* Os sensores DAQ são acionados apenas por meio de um backend (tunelado) — `connect_daq_sensor()`, ou o `proj.sensors` de um projeto; o backend abre o sensor em sua própria máquina.

## Assistentes de IA — `chloros-mcp`

O SDK inclui o `chloros-mcp`, um servidor MCP que permite que o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP controlem o Chloros: processem imagens e operem câmeras LATTICE, matrizes e sensores de luz DAQ. Suas ferramentas abrangem o que este SDK faz. Ele é instalado junto com o SDK; portanto, para o Claude Code, a configuração é feita com uma única linha:

```bash
claude mcp add chloros -- chloros-mcp
```

Outros clientes: [Usando o Chloros com assistentes de IA](ai-assistants.md). Opções e todas as ferramentas: [Servidor MCP (assistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants).

## Requisitos de licença

O acesso ao SDK requer um login ativo no Chloros+ em qualquer plano pago — **Copper ou superior**(Copper / Bronze / Silver / Gold); o plano gratuito Iron não tem acesso ao SDK/CLI. A verificação é feita**no lado do servidor**: toda solicitação de SDK deve conter tanto uma sessão ativa quanto um plano pago; caso contrário, o backend retorna `403` / `PLAN_UPGRADE_REQUIRED` (gerado como `ChlorosLicenseError` pelo `ChlorosLocal` e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um usuário desconectado recebe `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — a reexecução do `chloros-cli login` corrige o primeiro caso, mas não o segundo.

O uso offline funciona dentro do período de carência do plano: o nível de serviço é lido do cache de validação do servidor (5 minutos) ou do cache de licença assinada e vinculada à máquina (30 dias para planos mensais; até o vencimento da assinatura para planos anuais). Quando o período de carência expira, o plano passa para a versão gratuita e o acesso ao SDK é interrompido até que a máquina se conecte ao servidor uma vez. O `chloros-cli status` permanece acessível no nível gratuito, de modo que o motivo fica sempre visível. Consulte [Login no Chloros+](chloros+-login.md).

## Exceções

Intercepte a classe base para lidar com “qualquer problema no Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

Todas as exceções do pipeline (`ChlorosBackendError`, `ChlorosConnectionError`, `ChlorosLicenseError`, `ChlorosAuthenticationError`, `ChlorosConfigurationError`, `ChlorosProcessingError`) derivam de `ChlorosError`. Uma observação importante: `ChlorosConnectError` — gerado apenas por `connect_camera` / `connect_array` / `connect_daq_sensor` — deriva diretamente de `Exception`, **não** de `ChlorosError`; portanto, `except ChlorosError` não irá detectá-lo. A hierarquia completa está na [Referência do SDK](reference/sdk-reference.md#exceptions).

## Veja também

* [Referência do SDK](reference/sdk-reference.md) — a superfície completa da API, otimizada para assistentes de IA.
* [Referência da CLI](reference/cli-reference.md) — cada subcomando da CLI corresponde a uma chamada do SDK.
* [Usando o Chloros com assistentes de IA](ai-assistants.md) — receitas de configuração e prompts do `chloros-mcp`.
* [Download](download.md) — instaladores para Windows e Linux.
