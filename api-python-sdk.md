# API : Python SDK

{% hint style="info" %}
**Procurando o API completo?** Esta página é um tutorial prático. Todas as classes públicas, métodos, assinaturas exatas e exemplos prontos para copiar e colar estão na [Referência do SDK](reference/sdk-reference.md), que é otimizada para assistentes de IA.**Está trabalhando com um assistente de IA?** Cole este URL no chat para que ele tenha o Chloros 1.2.0 completo e atualizado:

`https://mapir.gitbook.io/chloros/reference/sdk-reference.md`

Cada página deste manual está disponível como markdown bruto em seu slug em letras minúsculas + `.md`, e o manual completo está indexado em `https://mapir.gitbook.io/chloros/llms.txt`.
{% endhint %}

O **Chloros Python SDK** (`chloros-sdk` no PyPI) controla tudo o que o aplicativo de desktop pode fazer a partir do Python: processamento em lote de imagens, controle em tempo real da câmera LATTICE e do conjunto de sensores, sessões de sensores de luz DAQ e automação de projetos salvos. Trata-se de uma camada fina sobre o mesmo backend local que a GUI e o CLI utilizam (HTTP no `127.0.0.1:5000`), portanto, o comportamento é idêntico nas três interfaces.

## Instalação

A instalação ocorre em duas etapas: primeiro, o pacote para desktop Chloros (que fornece o backend de processamento e os runtimes de hardware); depois, o pacote Python.

**Etapa 1 — Instale o Chloros.** Windows: execute o instalador para desktop (caminho padrão: `C:\Program Files\MAPIR\Chloros\`) a partir da página [Download](download.md). Linux: instale o pacote `.deb` ([Instalação do Linux](linux/linux-installation.md)).**Etapa 2 — Instale o SDK** (Python 3.7+):

```bash
pip install chloros-sdk
```

Talvez você nem precise do pip: cada instalador vem com um wheel SDK correspondente. O instalador Windows o instala automaticamente no seu sistema Python; o Linux `.deb` o coloca em `/usr/lib/chloros/sdk/` e exibe o comando exato `pip install --user`. O PyPI é atualizado nas compilações de lançamento, portanto, o `pip install chloros-sdk` corresponde à versão estável mais recente.

**Etapa 3 — Faça login uma vez por máquina:**

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache em `~/.chloros/` (em ambas as plataformas). No Windows, você pode fazer login de forma equivalente pela guia “<img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">” do aplicativo para desktop. O SDK requer um plano Chloros+ pago — consulte [Requisitos de licença](#license-requirement) abaixo.

| Requisito | Detalhes |
| --- | --- |
| **Chloros instalado** | Windows: instalador para desktop; Linux: pacote `.deb` (fornece o binário de back-end) |
| **Python** | 3.7 ou superior (desenvolvido/testado na versão 3.10) |
| **Sistema operacional** | Windows 10/11 de 64 bits, Ubuntu 22.04 LTS ou mais recente, ou NVIDIA Jetson (JetPack 6) |
| **Licença** | Login ativo no Chloros+, qualquer plano pago (Copper ou superior) |

## A vitória em 60 segundos

Uma única chamada cria um projeto, importa uma pasta, configura o processamento e executa o pipeline — iniciando automaticamente o backend caso ainda não esteja em execução:

```python
import chloros_sdk

results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)
```

(No Linux, use os caminhos do Linux: `/home/user/drone_images/flight001`. O SDK funciona da mesma forma em ambas as plataformas.)

Está processando uma pasta de captura do LATTICE? Use o wrapper compatível com o LATTICE — ele aplica os padrões corretos (sem detecção de painel-alvo, debayer padrão):

```python
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)
```

## `ChlorosLocal` — controle total do pipeline

Para qualquer coisa além de um comando de uma linha, use o `ChlorosLocal`. Ele inicia o backend na primeira utilização (`auto_start_backend=True`), cria e configura projetos, monitora o andamento e retorna um resumo após a execução.

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
Mantenha o padrão `http://127.0.0.1:5000` em vez de substituí-lo por `localhost` — no Windows, `localhost` é resolvido primeiro como `::1` e leva cerca de 2 segundos por solicitação no backend apenas para IPv4.
{% endhint %}

Use-o como um gerenciador de contexto para garantir a limpeza:

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

`configure()` aceita estas palavras-chave: `debayer`, `vignette_correction`, `reflectance_calibration`, `indices`, `export_format`, `ppk`, `daq_log_path`, `input_level`, `radiometric_output`, `array_alignment`, `array_alignment_crop`, `array_alignment_interpolation` e `custom_settings`. Os principais valores:

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"                  # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
```

Os controles específicos da LATTICE (`input_level`, `radiometric_output`, a família `array_alignment*`) estão documentados com suas tabelas completas de valores na [Referência do SDK](reference/sdk-reference.md#supported-values).

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

Após a conclusão, o `process()` anexa o resumo de processamento do backend como `result["summary"]`. Cada entrada no `summary["hints"]` é uma frase completa explicando qualquer aspecto digno de nota — por exemplo, por que uma execução produziu saída zero — e cada dica também é reemitida como um Python `UserWarning`, de modo que as execuções vazias se autodiagnosticam mesmo que você nunca inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

{% hint style="warning" %}
**`process()` não é gerado quando uma execução não produz imagens.** Esse é o único ponto em que o SDK e o CLI diferem deliberadamente: O `chloros-cli process` trata “produtos foram solicitados, nenhum foi gravado” como uma falha e sai com um código diferente de zero, enquanto o SDK retorna normalmente e relata a condição por meio do `summary` / dicas. Se o seu pipeline precisar ser interrompido em uma execução vazia, verifique isso você mesmo — inspecione o `summary` (ou conte os arquivos na pasta do projeto) em vez de confiar em uma exceção.
{% endhint %}

## Smart Connect — hardware ativo

Três auxiliares abrem sessões persistentes no pool de hardware do backend — o mesmo pool que a GUI usa; assim, os scripts SDK coexistem com o aplicativo de desktop sem disputar portas seriais ou largura de banda da rede. Todos os três iniciam automaticamente um backend local caso nenhum esteja em execução.

### Câmera LATTICE única — `connect_camera`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)   # microseconds, dB
    cam.capture("output/")
```

### Matriz sincronizada — `connect_array`

O `connect_array` é o ponto de entrada recomendado para configurações com várias câmeras. Ele executa o mesmo fluxo de preparação inteligente que a interface gráfica do usuário (GUI): análise de rede, seleção automática da camada de sincronização, sincronização de tempo PTP, seleção do formato de pixel por câmera, inicialização do AE e ativação do gatilho GPIO. A **primeira porta serial é a mestre** (ela dispara o pulso de acionamento do hardware); as demais são escravas.

```python
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")
```

Adicione o `smart=True` a qualquer captura em matriz para aguardar que a exposição automática se estabilize em todas as câmeras antes do acionamento. Para modos de captura (Única / Contínua / Interval / Mais rápida), gravadores, burst-to-video e alinhamento de matriz, consulte a [Referência do SDK](reference/sdk-reference.md#synchronized-array--arraysession-smart-prep).

### Sensor de luz DAQ — `connect_daq_sensor`

Sem argumentos, o `connect_daq_sensor()` detecta automaticamente o meio de transporte (ordem de prioridade: Ethernet → BLE → USB):

```python
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

Cada quadro transporta o valor de 135 pontos do `spectrum` (W/m²/nm quando calibrado), um sinalizador `is_saturated` e o CIE `x`, `y`, `z`. Para especificar um sensor ou protocolo de transporte específico — a escolha confiável em hosts com múltiplas interfaces de rede, onde a descoberta automática por Ethernet pode não detectar um DAQ-E em bom estado na primeira tentativa —, passe uma dica explícita:

```python
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")        # implies BLE
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")     # implies Ethernet
```

Observe que os perfis de correção de maiúsculas (`cap_id`) **não** são um controle do tipo SDK — selecione-os por meio de `chloros-cli daq pool-connect --cap-id …` / `pool-set-cap`.

### Projetos salvos — `open_project`

Um projeto Chloros salvo mantém o hardware conectado (`cameras.json` + `sensors.json` juntamente com `project.json`), e o `chloros_sdk.open_project(path)` pode reconectar tudo de uma só vez e realizar capturas por nome de dispositivo. Consulte [Automação de Projetos](reference/sdk-reference.md#project-automation--chlorosproject) na referência.

## O que uma instalação apenas via pip oferece

Verifique os sinalizadores de disponibilidade no nível do módulo antes de usar superfícies de hardware:

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)    # True iff lattice_sdk imported cleanly
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)       # True iff daq_sdk imported cleanly
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)  # True iff ChlorosProject deps available
```

Em um host com **apenas** o `pip install chloros-sdk` e sem o pacote de desktop Chloros:

* `ChlorosLocal`, `process_folder` e `process_lattice_capture` **não** funcionam — eles precisam do binário de back-end que vem no instalador do desktop.
* Os auxiliares de conexão inteligente (`connect_camera`, `connect_array`, `connect_daq_sensor`) são clientes puramente HTTP, portanto funcionam com um back-end em outra máquina — mas os back-ends fornecidos se ligam apenas ao loopback; portanto, você mesmo deve redirecionar a porta (por exemplo, `ssh -N -L 5000:127.0.0.1:5000 user@chloros-host`) e passar o `backend_url="http://127.0.0.1:5000"` junto com o `auto_start_backend=False`. Consulte [Modo de back-end remoto](reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel).
* As classes LATTICE de hardware direto (`LatticeCamera`, `CameraPool`, …) podem ser importadas, mas precisam do runtime Arena SDK do pacote para desktop — sem ele, `CAMERA_AVAILABLE` é `False`.
* `daq_sdk` (as classes DAQ diretas) vem com a instalação para desktop, não com o pacote PyPI, portanto, o `DAQ_AVAILABLE` é o `False` em um host que usa apenas o pip — acione os sensores DAQ por meio do `connect_daq_sensor()` contra um backend (tunelado) em vez disso.

## Requisitos de licença

O acesso ao SDK requer um login ativo no Chloros+ em qualquer plano pago — **Copper ou superior**(Copper / Bronze / Silver / Gold); o plano gratuito Iron não oferece acesso ao SDK/CLI. A aplicação é**do lado do servidor**: toda solicitação SDK deve conter tanto uma sessão ativa quanto um plano pago; caso contrário, o back-end retorna `403` / `PLAN_UPGRADE_REQUIRED` (gerado como `ChlorosLicenseError` pelo `ChlorosLocal` e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um chamador que está desconectado recebe `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — a reexecução de `chloros-cli login` corrige o primeiro caso, mas não o segundo.

O uso offline funciona dentro do período de carência do plano: o nível é lido do cache de validação do servidor (5 minutos) ou do cache de licença assinada e vinculada à máquina (30 dias para planos mensais; até o vencimento da assinatura para planos anuais). Quando o período de carência expira, o plano passa para a versão gratuita e o acesso ao SDK é interrompido até que a máquina se conecte ao servidor uma vez. O `chloros-cli status` permanece acessível no nível gratuito, de modo que o motivo fica sempre visível. Consulte [Chloros+ Login](chloros+-login.md).

## Exceções

Intercepte a classe base para lidar com “qualquer coisa que tenha dado errado”:

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

* [Referência do SDK](reference/sdk-reference.md) — a superfície completa do API, otimizada para assistentes de IA.
* [Referência do CLI](reference/cli-reference.md) — cada subcomando do CLI corresponde a uma chamada do SDK.
* [Download](download.md) — instaladores para Windows e Linux.
