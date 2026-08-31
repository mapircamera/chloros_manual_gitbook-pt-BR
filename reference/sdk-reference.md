# Chloros Python SDK Referência

**Versão:**

1.2.0**Gerado em:**29/07/2026 19:19 ·**Revisado em:** 30/08/2026**Pacote:** `chloros-sdk` (PyPI)**Público-alvo:** Otimizado para uso por LLM; legível por humanos.**Escopo:** Todas as classes, funções e auxiliares públicas expostas pelo `import chloros_sdk`, com exemplos que podem ser copiados e colados, abrangendo processamento de imagens, controle de câmera única, matrizes sincronizadas, sensores DAQ e automação de projetos.

Se você precisar apenas dos destaques, vá para:
- [Instalação e Introdução Rápida](#instalação)
- [Smart-Connect para Matriz LATTICE](#smart-connect-for-lattice-cameras)
- [Sessões de sensores DAQ](#daq-sensor-sessions)
- [Automação de projetos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)

---

## Arquitetura em 60 segundos

O SDK é uma camada fina de Python sobre o backend Chloros (o mesmo servidor Flask usado pela GUI de desktop e pelo CLI). Para automação, você importa `chloros_sdk` e chama métodos de alto; nos bastidores, cada chamada se transforma em uma solicitação HTTP para o backend local na porta 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente não `localhost`, que remete para `::1` em Windows e custa cerca de 2 s por solicitação em um backend que opera apenas com IPv4). O backend é responsável pelo conjunto de equipamentos — câmeras, sensores DAQ, perfis de alinhamento, buffers de quadros — de modo que scripts dSDKm possam coexistir com a GUI sem disputar portas seriais ou largura de banda da placa de rede.

Existem três interfaces que você utilizará:

1. **`ChlorosLocal` + funções livres** (`process_folder`, `process_lattice_capture`) — Pipeline de processamento de imagens. Execute uma pasta inteira passando pelas etapas de calibração / debayer / exportação de índice a partir de uma única chamada `Python`.
2. **Manipuladores Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abra uma sessão de backend persistente para hardware ativo. O mesmo fluxo “smart-prep” da GUI: sonda de rede, seleção automática de camada, PTP, inicialização de AE, configuração de acionamento por GPIO.
3. **`ChlorosProject` / `open_project`** — Carregue um projeto salvo (pasta com `cameras.json` + `sensors.json` + `project.json`), conectar tudo de uma vez e realizar capturas de drive com identificadores nomeados.

As superfícies 1 e 2 **inicializam automaticamente um backend local** caso ainda não haja um em escuta (o mesmo binário incluído que a GUI/CLI iniciam) — portanto, um script simples funciona a partir de um shell novo sem que você precise iniciar um backend primeiro. Passe `auto_start_backend=False` para desativar essa função (por exemplo, ao apontar para um backend remoto, que nunca é iniciado). Consulte [Inicialização automática do backend](#backend-auto-start). O Surface 3 se comporta de maneira diferente: o `open_project()` não aceita o parâmetro `auto_start_backend`, e o `connect_all()` nunca inicia um backend — ele tenta se conectar ao `http://127.0.0.1:5000` uma vez e, se nada responder, recorre silenciosamente ao controle direto (sem backend) do dispositivo `lattice_sdk`. Apenas o `proj.process()` e o `stream(..., overlays=True)` criam de forma diferida um `ChlorosLocal()` (que inicia automaticamente).

Todos os três são restritos por autenticação: execute o `chloros-cli login` uma vez na máquina ou faça login pela interface gráfica da área de trabalho. Chamadas para `SDK` sem uma sessão válida geram o erro `ChlorosAuthenticationError`.

Requisitos:
- Python 3.7+ (conforme declarado pelo pacote; desenvolvido/testado na versão 3.10)
- Chloros Desktop instalado localmente (o binário de backend vem incluído no instalador)
- Login ativo no Chloros+. O nível mínimo para SDK / CLI é o plano **Copper**ou superior (Copper / Bronze / Silver / Gold); o plano gratuito**Iron**não tem acesso a SDK / CLI. Isso é aplicado**no lado do servidor**: toda solicitação com a flag SDK / CLI deve conter tanto uma sessão ativa quanto um plano pago, ou o backend retorna `403` com `error_code: PLAN_UPGRADE_REQUIRED` (exibido como `ChlorosLicenseError` pelos auxiliares `ChlorosLocal`, e como `ChlorosConnectError` pelos auxiliares `connect_*`). Um solicitante desconectado recebe `401` / `AUTH_REQUIRED` (`ChlorosAuthenticationError`) — os dois são distintos porque a reexecução do `chloros-cli login` corrige o primeiro, mas não o segundo.
- O uso offline é permitido dentro do período de carência do plano: o nível é lido do cache de validação do servidor (5 min) ou do cache de licenças assinado e vinculado à máquina(30 dias para planos mensais, até o vencimento da assinatura para planos anuais). Quando esse período de carência expira, o plano passa a ser gratuito e o acesso a SDK / CLI é interrompido até que a máquina consiga se conectar ao servidor uma vez. `chloros-cli status` (`GET /api/license-status`) permanece acessível no nível gratuito, de modo que o motivo fica visível — é a única rota SDK / CLI isenta da restrição de nível.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS ou mais recente**, ou Jetson (JetPack 6). O Ubuntu 20.04**não** é compatível: as dependências do `.deb` derivam do que o backend vincula, incluindo o `libc6 (>= 2.34)`, e a versão Focal vem com a glibc 2.31.

---

## Instalação

O Python SDK é uma camada fina Python sobre o backend Chloros. Para tudo além de alguns fluxos de trabalho exclusivamente de DAQ, é necessário ter o **pacote para desktop Chloros instalado localmente** (instalador Windows ou Linux `.deb`) — é ele que fornece o binário do backend, o runtime Arena SDK para câmeras LATTICE e os pacotes de calibração.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Etapa 1 — Instale o pacote da plataforma Chloros

#### Windows (.exe)

1. Baixe o `Chloros-Setup-x.y.z.exe` na página de downloads.
2. Execute o instalador e siga as instruções do assistente. O caminho padrão de instalação é `C:\Program Files\MAPIR\Chloros\`.
3. Abra o Chloros pelo menos uma vez e faça login com sua conta Chloros+.

#### Linux amd64 (.deb)

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

### Etapa 2 — Instale o Python SDK

**O instalador Chloros vem com um pacote SDK correspondente.** Todo instalador Windows e pacote .deb Linux instala um `chloros_sdk-X.Y.Z-py3-none-any.whl` no disco que corresponde exatamente à versão da GUI / CLI / backend. Você não precisa ficar acompanhando o PyPI para se manter em sincronia.

#### Windows

O instalador executa automaticamente o `pip install` com o wheel incluído, utilizando o Python do seu sistema (o executável `py.exe` é preferencial; caso contrário, recorre ao `python -m pip`). Não é necessária nenhuma ação — o `import chloros_sdk` funciona em seu ambiente Python após uma instalação bem-sucedida. Se não houver nenhum Python instalado no computador, o instalador ignora silenciosamenteesta etapa e a GUI + CLI continuam funcionando.

#### Linux (.deb)

O pacote .deb instala o wheel como `/usr/lib/chloros/sdk/`. O `postinst` exibe o comando exato — as distribuições PEP 668 recusam gravações globais do pip por padrão, portanto, não fazemos a instalação automática:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implantações do Jetson em ambiente isolado (air-gapped), isso ocorre totalmente offline — o wheel já está no disco.

#### PyPI Público

Para hosts que utilizam apenas o pip (sem pacote de desktop do Chloros instalado; fluxos de trabalho com backend remoto ou apenas DAQ):

```bash
pip install chloros-sdk
```

O PyPI é atualizado nas compilações do instalador da versão de lançamento, de modo que o wheel publicado corresponde à versão estável mais recente. As compilações de desenvolvimento (por exemplo, `1.1.4.dev1`) são fornecidas apenas por meio do wheel do instalador incluído.

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("DAQ_AVAILABLE    =", chloros_sdk.DAQ_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **É necessária uma assinatura do Chloros+.** Todas as chamadas ao SDK exigem um login ativo no Chloros+. Execute o `chloros-cli login user@example.com 'YourPassword'` uma vez por máquina; as credenciais são armazenadas em cache no `~/.chloros/`.

### Preciso do Pacote Desktop?

O pacote pip por si só **não** é suficiente para a maioria dos fluxos de trabalho. Veja a seguir o que cada superfície SDK precisa:

| Superfície SDK | Precisa do Pacote Desktop? | Por quê |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sim** | Inicia automaticamente o binário de backend em `/usr/lib/chloros/chloros-backend` (Linux) ou `C:\Program Files\MAPIR\Chloros\…` (Windows). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sim**(local)**/ Não**(remoto) | Clientes Pure HTTP via backend. Backend local → pacote para desktop necessário. Backend remoto → `backend_url=`**por meio de um túnel** (consulte Modo de Backend Remoto — backends fornecidos se ligam apenas ao loopback). |
| `ChlorosProject` / `open_project` | **Sim** | Executa projetos salvos por meio do backend. |
| Classes LATTICE diretas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sim** | Requer o runtime nativo do Arena SDK, que vem incluído no pacote para desktop. Caso contrário, o `CAMERA_AVAILABLE` é equivalente ao `False` na importação. |
| Classes DAQ diretas (`DAQUSensor`, `DAQMSensor`, `DAQESensor`, `SensorFleet`, `discover_all`) | **Não** | DAQ puro (Python) via pyserial/bleak/zeroconf. Um ambiente exclusivamente pip pode controlar os DAQs de ponta a ponta. |

### Modo de backend remoto (host exclusivamente pip, via túnel)

> **O backend fornecido não está acessível pela LAN.** As compilações
> de produção se ligam apenas ao loopback (ambas as famílias de loopback) e recusam categoricamente o
> único modo que não é de loopback (`CHLOROS_CLOUD_MODE`); portanto,
> o `backend_url="http://<lan-ip>:5000"` **não funciona com um
> Chloros instalado** — esse padrão só funcionava com um backend source/dev
> . Para controlar um backend em outra máquina, redirecione sua porta de loopback
> e aponte o SDK para o túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"   # the tunnel endpoint

chloros_sdk.connect_camera("213800234", backend_url=BACKEND)
chloros_sdk.connect_array(serials, backend_url=BACKEND)
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local", backend_url=BACKEND)
```

Hosts sem interface gráfica / CI / robótica podem manter uma máquina com a instalação completa da área de trabalho como o “servidor Chloros” e o `pip install chloros-sdk` em todos os outros locais — mas o transporte entre eles é o túnel configurado pelo usuário acima, e não uma conexão direta via LAN URL.

> **Limitação conhecida — o `ChlorosLocal` não é compatível exclusivamente com o pip.** Atualmente, o `ChlorosLocal(backend_url=BACKEND)` resolve um binário de backend local em seu construtor *antes* de consultar o URL, e gera o erro `ChlorosBackendError` (“Backend Chloros não encontrado…”) quando nenhum pacote de desktop está instalado — mesmo com um backend remoto acessível. Apenas a interface de conexão inteligente acima (`connect_camera` / `connect_array` / `connect_daq_sensor`, além de `analyze_array_network` e os auxiliares `list_*` / `discover_*`) funciona a partir de um host que utiliza apenas o pip.

### Fluxo de trabalho apenas com DAQ (host apenas com o pacote pip)

Se você precisar apenas de sensores DAQ e não utilizar câmeras LATTICE nem processamento de imagens, o pacote pip é autônomo:

```bash
pip install chloros-sdk
```

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

sensor = DAQUSensor(port="/dev/ttyUSB0")
sensor.connect()
sensor.start_streaming()
```

Sem backend, sem .deb, sem necessidade de login em Chloros+ para trabalho de DAQ direto com o hardware.

---

## Início rápido

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "NDRE", "GNDVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Índice de nível superior do API

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()

# === Project lifecycle ===
chloros_sdk.open_project(path)                    # → ChlorosProject
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk / daq_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics
chloros_sdk.DAQUSensor, DAQMSensor, DAQESensor, SensorFleet, discover_all

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.DAQ_AVAILABLE        # True iff daq_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True iff ChlorosProject deps available
```

---

## Processamento de imagens — `ChlorosLocal`

A classe principal do pipeline. Inicia o backend na primeira utilização, cria e configura projetos, monitora o progresso e retorna resumos após a execução.

### Construtor

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

### Métodos

| Método | Descrição |
| --- | --- |
| `create_project(project_name, camera=None)` | Cria um novo projeto (opcionalmente com um modelo de câmera como `"Survey3N_RGN"`). |
| `import_images(folder_path, recursive=False)` | Importa imagens RAW/TIF/JPG/DNG **e gravações do sensor de luz `.daq`**. Retorna `count` (imagens) e `scan_count` (gravações). Exibe um aviso somente se a pasta não contiver nenhum dos dois. |
| `export_light_sensor(daq=True, csv=True)` | Grava os arquivos calibrados `.daq` + `.csv` para cada gravação do sensor de luz do projeto no arquivo `<project>/Light Sensor/`. Consulte [Gravações do sensor de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Defina os parâmetros de processamento. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Execute o pipeline. Retorna `{"status": "complete", "async": False}`, além de uma chave `summary` quando o backend a fornecer — consulte [Resumo pós-execução e dicas](#post-run-summary--hints). |
| `get_config()` / `get_status()` / `status()` | Verifica o estado do backend. |
| `logout()` | Limpa as credenciais armazenadas em cache. |
| `shutdown_backend()` | Encerrar o backend (se iniciado com `SDK`). |
| `discover_cameras()` | Detectar câmeras LATTICE **por meio do backend desta instância** (`/api/camera/discover`). Retorna uma lista de dicionários (`serial`, `model`, `ip`, …) — com a mesma estrutura que a GUI/ CLI. Lista vazia se nenhuma for encontrada ou se o backend estiver inacessível. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura um único quadro**pelo backend**(iniciado automaticamente por este identificador) para que receba a mesma preparação que a GUI/ CLI (padrão de 12 bits, reutilização do pool, metadados de calibração incorporados). Resolva o alvo com `serial=` ou `device_index=`; passe `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Retorna o dicionário de metadados legados (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Retorna quadros de pré-visualização com sobreposição de composição a partir de um cliente MJPEG leve agrupado — cliente MJPEG leve pela rota `/api/camera/<serial>/stream-annotated` do back-end (zebra / grade / mira / histograma / pico / ponto desenhados no lado do servidor). `decode=True` gera matrizes BGR; `False` gera bytes brutos JPEG. Também acessível por projeto como `ChlorosProject.stream(overlays=True)`. |

Use como gerenciador de contexto para limpeza garantida:

```python
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

### Gravações de sensores de luz — calibradas `.daq` + `.csv`

Um DAQ-U / DAQ-M / DAQ-E pode ser gravado **sem** seu pacote de calibração. É isso
que os gravadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`): eles gravam as contagens brutas dos sensores e marcam o
arquivo de forma que o Chloros busque a calibração de fábrica desse sensor **pelo número de série** — primeiro no cache local,
depois na nuvem MAPIR — e a aplique na importação.

O Chloros grava o resultado de volta como dois produtos por gravação, em
`<project>/Light Sensor/`:

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | O arquivo reprocessável — mesmo esquema de uma gravação ao vivo, agora declarando o pacote que o produziu. Reimportá-lo **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectralem W/m²/nm na grade de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas (potência total, lux fotópico/escotópico, PPFD e sua divisão em azul/verde/vermelho, comprimento de onda de pico). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Apenas sensores sem pacote (DAQ-A).** Contagens espectrais brutas do sensor — *não* irradiância. Veja abaixo. |

O `process()` realiza essa exportação como uma de suas etapas. Ele **não** requer imagens:
um sensor de luz operado de forma independente é um fluxo de trabalho de primeira classe, e tal projeto tem zero
imagens por definição.

**As gravações do DAQ-A são exportadas como contagens brutas.** A família DAQ-A é anterior ao sistema de pacotes por série
e não possui nenhum pacote a ser buscado — ela é calibrada em campo em relação a um
alvo de refletância, e é por isso que nunca precisou de um. Essas gravações são exportadas
sob o prefixo `_raw` em vez de `_calibrated`: um nome de arquivo diferente em vez de um sinalizador
dentro do arquivo, pois a informação precisa permanecer intacta ao ser enviada por e-mail apenas como um nome simples. O
cabeçalho `.csv` indica `raw spectral sensor counts (NOT irradiance)` e alerta que os
valores são comparáveis **dentro** do arquivo — exatamente para isso a calibração baseada em alvo
os utiliza — e não entre sensores. As colunas fotométricas dependentes de potência (potência total,
lux fotópico/escotópico, PPFD) retornam **NULL** em vez de serem integradas a partir das contagens.

Um DAQ-U / DAQ-M / DAQ-E cujo pacote simplesmente não pôde ser obtido ainda é **ignorada**,
não gravada em formato bruto: nesse caso, o pacote existe e “reconectar e reprocessar” é um conselho válido.

Gravações legadas **v1.01 / v1.02** (gravadas por um DAQ-A-SD) não contêm época por leitura,
apenas a hora de gravação do arquivo. O comparador de imagens↔luz descendente ainda as rejeita — comparar um
quadro com uma hora de gravação seria um erro imperceptível —, mas o exportador as lê, e o
CSV imprime `clock=daq_created_on` para que o produto indique em qual relógio ele está.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Uma gravação cujo pacote de calibração não pode ser obtido (offline, ou um sensor sem
calibração registrada) é relatada sob o código `skipped` **com o motivo**. Ela nunca é
gravada como um arquivo “calibrado” contendo contagens brutas — conecte-se à internet e
execute novamente, e a exportação será concluída.

### Retornos de chamada de progresso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumo pós-execução e Dicas

Após a conclusão, o `process()` busca o `GET /api/processing-summary` e anexa o corpo como `result["summary"]`. A busca é feita com o melhor esforço possível e nunca bloqueia um retorno bem-sucedido — se o resumo não estiver disponível, o `process()` recorre ao formato simples do `{"status": "complete", "async": False}`. Cada entrada em `summary["hints"]` — frases completas com a correção sugerida, por exemplo, por que uma execução produziu saída zero — também éemitida como um `UserWarning` do tipo “Python”, de modo que execuções com saída zero são autodiagnosticáveis, mesmo que você nunca inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` é a parte legível por máquina:

| Chave | O que ela conta |
| --- | --- |
| `models` | Grupos de câmeras na execução. |
| `images_in_groups` | Imagens de origem nesses grupos. |
| `targets_found` | Alvos de refletância detectados. |
| `images_calibrated` | Imagens calibradas pela execução. |
| `exported_files` | **Arquivos de produtos de imagem gerados pela execução.** |
| `daq_recordings_exported` / `daq_recordings_skipped` | Registros do sensor de luz, contados separadamente propositalmente — eles provêm de uma etapa diferente e existem para execuções sem nenhuma imagem; portanto, incluí-los faria com que uma execução apenas de DAQ parecesse ter exportado imagens. |

Juntamente com eles: `summary["output_dirs"]` (todos os diretórios nos quais foi gravado),
`summary["light_sensor_export"]`, `summary["stopped"]` (verdadeiro quando o usuário interrompeu a
execução, para que contagens parciais não sejam interpretadas como uma execução concluída com produção insuficiente) e
`summary["groups"]` (agrupo).

`exported_files` é registrado pelo pipeline **à medida que grava**, e não extraído posteriormente dos
objetos de imagem do projeto. As estratégias paralela e de GPU criam seus próprios objetos de imagem
(em subprocessos de trabalho para os caminhos de GPU), de modo que a varredura antiga relatava
`0 file(s) written` para cada uma dessas execuções e, em seguida, emitia a dica de exportações zero — em execuções
nas quais tudo havia funcionado. Se você criar um script com base nesse número, uma execução paralela válida agora
relata uma contagem diferente de zero.

Os saltos do sensor de luz relatam o motivo que o leitor realmente identificou para cada arquivo — um
esquema ilegível, um pacote ausente, um erro de gravação — **deduplicado**; assim, vinte arquivos
ignorados por um único motivo são interpretados como um único motivo, em vez de vinte repetições dele.

> **`process()` não é acionado quando uma execução não produz imagens.** Este é o único ponto em que o SDK e
> o CLI diferem deliberadamente: o `chloros-cli process` trata “produtos foram solicitados, nenhum foi
> gravado” como uma falha e sai com um valor diferente de zero, enquanto o SDK retorna normalmente e relata a
> condição por meio de `summary` / hints. Se o seu pipeline parar em uma execução vazia, verifique você
> mesmo — inspecione o `summary` (ou conte os arquivos na pasta do projeto) em vez de confiar na
> ausência de uma exceção. As causas mais comuns são uma pasta de entrada que não foi reconhecida como uma
> captura e produtos ignorados por serem inaplicáveis às câmeras presentes (por exemplo, radiação de câmeras que utilizam apenas
> RGB).

### Funções de conveniência

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

### Valores suportados

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer, Chloros+ only
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as already-debayered BGR
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Saída radiométrica (pipeline multiespectral LATTICE)

O nível de exportação multiespectral LATTICE (M3C/M3M) do pipeline `process` — `reflectance` (padrão), `radiance`, `sensor-response` ou `all` (todos os modos aplicáveis por imagem) — corresponde à configuração de processamento **“Saída radiométrica”** do projeto. O `configure()` possui uma palavra-chave dedicada para isso:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A saída de emergência avançada — escrever a chave `"Radiometric output"` do projeto por meio de `custom_settings` — ainda funciona, mas lembre-se de que ela substitui todo o bloco de configurações (veja o aviso abaixo):

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Radiometric output": "radiance"},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (o padrão) divide a radiação da câmera pelo **fluxo descendente do DAQ com data e hora correspondentes**, resolvido automaticamente a partir de um `.daq` (DAQ-U/M/E)**ou um `.csv` nativo do DAQ-M**encontrado junto com as imagens; qualquer pacote de calibração por câmera ou DAQ ausente localmente é**buscado automaticamente da AWS** na primeira utilização. O CLI expõe isso como opções de ativação/desativação por tipo de produto no `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.

> `custom_settings` **substitui** todo o bloco de configurações calculadas (por padrão, ele ignora as outras palavras-chave e a validação do `configure()`). Ao usá-lo, inclua todas as chaves `Project Settings` que forem relevantes para você, conforme o exemplo acima.

---

## Smart-Connect para câmeras LATTICE

Sessões persistentes de backend para hardware ao vivo. Os mesmos endpoints que a GUI utiliza, portanto o comportamento é idêntico em SDK / CLI / GUI.

### Câmera única — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

#### Assinatura do `connect_camera()`

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

#### Métodos do `CameraSession`

| Método | Descrição |
| --- | --- |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lê nós GenICam; retorna `{nodes, errors, enums, device}`. |
| `set_settings(**kwargs)` | Grava nós por nome amigável (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, settings=None, timeout=None)` | Captura um **único** quadro. Retorna uma lista com um elemento contendo dicionários de metadados do quadro. (A captura em sequência/a captura de múltiplos quadros foi removida — chame `capture()` em um loop se precisar de uma série.) |
| `disconnect()` | Libera do pool. Não realiza nenhuma operação se estivermos conectados a uma sessão já aberta. |

Controles de exportação do `capture()` (mesmo modelo da matriz + GUI):

- `processing` / `levels` — `processing="all"` salva todos os tipos de exportação aplicáveis; `levels=["raw","radiance"]` salva apenas esses (substitui `processing`). Omita ambos para usar o padrão do backend.
- `force_daq=True` — salva a leitura DAQ/DLS atribuída como um sidecar `.daq` mesmo em uma captura apenas em formato bruto, para que o quadro possa ser reprocessado em refletância/índice posteriormente. Não tem efeito se nenhum DAQ estiver vinculado.

### Matriz Sincronizada — `ArraySession` (Smart-Prep)

`connect_array` é **o ponto de entrada recomendado** para configurações com várias câmeras. Ele executa internamente todo o fluxo de preparação inteligente da GUI:

1. **Análise de rede** (`/api/camera/array/recommend`) — identifica o maior tamanho de quadro que se encaixa na camada sim-emit sem perda de quadros.
2. **Seleção automática de camada** — `sim-capture-sim-emit` se a conexão suportar; caso contrário, `sim-capture-ftd-stagger` ou `slip-emit-and-capture`.
3. **Redimensionamento automático**— reduz silenciosamente o tamanho do quadro ou aumenta o agrupamento quando a linha não consegue sustentar a resolução solicitada.**Essa rede de segurança não cobre a superscrição agregada**: um número excessivo de câmeras para a largura de banda não pode ser corrigido pela redução de quadros — consulte [Superscrição](#over-subscription-the-per-cam-floor).
4. **PTP habilitado**por padrão — os carimbos de data/hora entre câmeras são sincronizados em um relógio compartilhado com precisão de**~1 ms**. A exposição simultânea é proveniente do disparador de hardware do M8 (**&lt; 100 µs** entre módulos), não do PTP: o PTP alinha *carimbos de data/hora*, não exposições.
5. **Seleção automática do formato de pixel por câmera**** — câmeras RGB → `BayerRG8`, multiespectral → `BayerRG12`.
6. **Inicialização do AE** — captura o estado atual do AE de cada câmera para que a conexão nãonão seja reiniciada a exposição durante a operação.
7. **Configuração do disparador GPIO** — `connect_array` arma todas as câmeras (`TriggerMode=On`, `TriggerSource=Line2`) para que o pulso do mestre acione os escravos através pelo cabo M8. Esta etapa é exclusiva para matrizes: uma única câmera aberta com `LatticeCamera` opera de forma autônoma.

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

#### Assinatura do `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # master trigger fire rate (auto if None)
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=180.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

Valores do `force_tier`:
- `"sim-capture-sim-emit"` — verdadeiramente simultâneo (todas as câmeras disparam na mesma flanco do clock).
- `"sim-capture-ftd-stagger"` — escalonamento flexível no domínio do tempo (as câmeras emitem em momentos ligeiramente deslocados, de modo que os pacotes sejam serializados na linha).
- `"slip-emit-and-capture"` — captura sequencial por cam (sem sincronização temporal; única opção quando nenhum tamanho de quadro se encaixa na simulação).

`wire_ceiling_mbps` substitui o **orçamento de banda sustentável do host** em MB/s — o único
número do qual depende toda a alocação da matriz. Deixe como `None` para usar o valor detectado automaticamente
. Reduza-o quando a matriz relatar quadros corrompidos pelo GVSP: o valor automático é derivado
da taxa de link anunciada pela NIC, que superestima adaptadores USB, pistas PCIe estreitas e
estruturas compartilhadas ocupadas — e essa superestimativa se manifesta como quadros corrompidos, em vez de como um
link visivelmente lento. O valor é mantido no bloco de captura da matriz do projeto; portanto, uma
reabertura ou um comando `connect_array` posterior o restaura como qualquer outra configuração da matriz.
Consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames).

#### Sobrescrição (o limite mínimo por câmera)

O controle de ritmo de emissão simulada aloca a cada câmera uma parcela do orçamento de banda seguro contra colisões, com um limite mínimo de **8 MB/s por câmera**(`per_cam_floor_bps`). Quando o `N × floor` excede o limite máximo seguro contra colisões, o array**sobresubscreve a banda**— o modo de falha é a perda de pacotes GVSP, não uma taxa de quadros mais baixa — e não há solução relacionada ao tamanho do quadro:**o agrupamento e a ROI reduzem os bytes por quadro, não os bytes regulados por segundo**que a verificação agregada compara. Limites práticos de resolução total em um host de 1 GbE:**6 câmeras com MTU de 1.500, 9 com quadros jumbo** (o `max_cams_collision_safe` na resposta da análise informa o limite máximo para sua conexão). Soluções: menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.

- As respostas `analyze_array_network()` e `/api/camera/array/connect` contêm `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` é verdadeiro, a projeção **zerou os campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) em vez de relatar uma taxa enganosa de “lenta, mas funcionando”.
- `POST /api/camera/array/connect` aceita um parâmetro de corpo `pin_resolution` (**somente HTTP — não é um kwarg do SDK**; `connect_array` não o expõe). O pinning remove a rede de segurança do walk-down de binning; assim, uma conexão com excesso de solicitações e o parâmetro `pin_resolution` definido é**rejeitada categoricamente** com um erro que indica todas as soluções possíveis. Sem o pinning, a conexão prossegue com o walk-down, mas avisa que a redução não pode zerar o agregado.
- Saída de emergência para testes: defina `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` no ambiente do backend para rebaixar a recusa a um aviso sonoro — você se conecta de qualquer maneira e aceita a perda de pacotes.

#### Integridade da matriz — qual subsistema está perdendo quadros

`GET /api/camera/array/<array_id>/capability` carrega um bloco `health` ativo em um
array conectado, reavaliado em uma janela contínua de **10 segundos**. Ele divide a perda de quadros
em duas causas que exigem correções opostas, em vez de uma taxa “incompleta” que
não especifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serial) | O quadro **chegou e estava estruturalmente corrompido**— perda de pacotes GVSP. |**Rede**: largura de banda, ritmo, anel de recepção da placa de rede, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nem chegou**— a câmera não disparou ou nada saiu dela. |**Disparador/sincronização**: Cabo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Pior taxa de cada câmera. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmera (ambas as causas juntas). | — |
| `stable_for_seconds` | Por quanto tempo cada câmera permaneceu abaixo de 0,01 %. | — |

Juntamente com o `health`, o mesmo registro informa o número em que toda a alocação fica pendente:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de banda sustentado do host em vigor, em MB/s. |
| `wire_ceiling_source` | De onde veio esse número, por extenso — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando o `wire_ceiling_mbps=` o definir. |
| `nic_is_usb` | `true` para um adaptador USB Ethernet. |

Não há wrapper de SDK para este endpoint — leia-o diretamente:

```python
import requests, chloros_sdk

arr = chloros_sdk.attach_array(["213800234", "214000533"])
h = requests.get(
    f"http://127.0.0.1:5000/api/camera/array/{arr.array_id}/capability",
    timeout=10).json()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    arr.disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretação:** um valor diferente de zero em `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` igual a 0 significa que
o acionamento e a sincronização do cabo estão perfeitos e 100% da perda está no caminho da rede — reduza
`wire_ceiling_mbps` e reconecte. O padrão inverso aponta para o cabo de sincronização ou para a
linha de acionamento.

> **`target_fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido uma vez na
> conexão; portanto, diminuir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na demanda não produziu nenhuma melhora, enquanto
> diminuir o limite máximo da linha de 240 para 200 MB/s levou o mesmo equipamento de 10,4% de quadros corrompidos para
> 0,00%.

> **A redução automática no meio do fluxo não está disponível no firmware do TRI032S.** Um array em operação não pode
> corrigir isso sozinho; desconecte e reconecte para que o seletor de tempo de conexão replaneje com base
> no novo limite máximo.

Um **adaptador Ethernet USB é limitado a 200 MB/s** pelo probe, independentemente de sua
placa de identificação: a tabela de eficiência que converte uma taxa de link em um valor sustentado é
derivada do PCIe, e uma placa de rede USB anuncia sua taxa de link Ethernet enquanto está limitada pelo
barramento USB e seu driver. O limite é absoluto, não uma fração — um adaptador USB de 1 GbE
atinge cerca de 80 MB/s e não é afetado.

#### Métodos `ArraySession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | `{fps, ptp, frame_count, last_error, …}` em tempo real. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, smart=False, timeout=300.0)` | Um grupo de captura sincronizado. Retorna uma `CaptureResult` (lista de dicionários de quadros + `.skipped`). Controles de exportação abaixo. |
| `capture(..., smart=True)` | **Captura inteligente** — aguarda a estabilização do AE em todas as câmeras e, em seguida, aciona a captura. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, timeout=120.0)` | Captura mais rápida: apenas dados brutos + a leitura DAQ atribuída (+ o índice combinado livre). Espelha o botão “Captura mais rápida” da GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Contínua / Intervalo em um loop delimitado. Retorna `list[CaptureResult]`.**Requer `count` e/ou `duration_s`** para que seja encerrado (o SDK não possui Ctrl+C). |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, timeout=30.0)` | Inicia a gravação da visualização ao vivo do índice combinado em vídeo/GIF → `RecorderHandle`. Um gravador composto por matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, timeout=30.0)` | Iniciar uma sequência de imagens Bayer brutas com alta taxa de quadros → `RecorderHandle`. Reprocessar offline com `build_video()`. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocessar offline uma sequência raw salva em vídeo calibrado(s). Bloqueia até a conclusão (`wait=True`) e retorna `{outputs, errors, combined}`. |
| `build_video_status(job_id, timeout=15.0)` | Verifica um trabalho de compilação offline: `{running, result, error, burst_dir}`. |
| `disconnect()` | Libera toda a matriz. |

Controles de exportação do `capture()` (mesmo endpoint usado pela GUI/ CLI):

- `processing` / `levels` — `processing="all"` (ou `levels=["raw","radiance",…]`) salva todos os tipos de exportação aplicáveis por câmera; um único valor `processing` salva apenas esse nível.
- `aligned=True` — distorce todas as exportações não brutas de cada membro para o [perfil de alinhamento](#array-alignment) da matriz (co-registrado); os dados brutos permanecem sem warping, mas carregam a transformação nos metadados. Recorre ao modo não alinhado (com um aviso exibido no `alignment` do resultado) se a matriz não tiver perfil.
- `render_index=False` — ignora a sobreposição decâmera; por padrão, ela é renderizada onde estiver configurada.
- `force_daq=True` — salva a leitura DAQ/DLS atribuída como um arquivo sidecar `.daq`, mesmo quando nenhum nível selecionado precisar dela.

**CompressãoTIFF (botão “HTTP” apenas):** O `ArraySession.capture()` não envia a chave `compression`, portanto, aplica-se a configuração padrão do backend — o `POST /api/camera/array/capture` lê um parâmetro de corpo `compression`, `"deflate"` por padrão (zlib L1 sem perdas + preditor horizontal, ~4,1 MB por quadro em resolução total). O `"none"` grava sem compressão (~6,3 MB/quadro) com uma**gravação ~5 vezes mais rápida** — ambas são sem perdas e são lidas de forma idêntica na importação. O `SDK` não expõe nenhum kwarg para isso; a saída alternativa é ``chloros-cli lattice array-capture --compression none`` ou `HTTP` bruto. O DEFLATE também mantém o GIL de `Python`, portanto, as gravações compactadas nãoseu paralelo entre as threads de gravação por câmera — a captura sustentada em resolução total com 8 câmeras na taxa do sensor requer o `compression: "none"`. Detalhes: [CLI Referência → array-capture](cli-reference.md).**Substituições de exportação por membro (HTTP -somente):**o mesmo endpoint também aceita `exclude_serials` (lista — exclui membros do conjunto salvo; a matriz ainda é acionada como um grupo sincronizado e os membros excluídos são retornados em `excluded`), `serial_levels` (substituições no nível por câmera de `{serial: [level tokens]}`) e `serial_index` (substituições de sobreposição de índice por câmera em `{serial: bool}`). Esses são parâmetros de corpo com paridade com a GUI e**ainda não são argumentos-chave dSDK**; os membros ausentes dos mapas recorrem aos valores globais da matriz em `levels` / `render_index`, válidos para toda a matriz.

##### Inspecionando Cams Ignoradas — `CaptureResult.skipped`

`ArraySession.capture()` retorna um `CaptureResult`, que é uma subclasse de `list`: iterar, indexar, `len()` — todos os padrões existentes continuam funcionando. O novo código pode inspecionar o atributo `.skipped` para verificar quais câmeras foram excluídas e por quê. O caso mais comum é o de câmeras RGB em uma matriz de filtros mistos quando se solicita por `processing="radiance"` ou `"reflectance"` — a radiação por pixel Bayer não faz sentido para um sensor de banda larga, então o backend ignora essas câmeras em vez de produzir resultados sem sentido.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Os tokens de motivo seguem o padrão `<level>-not-applicable-to-rgb-cam` (uma entrada por nível ignorado, cada uma contendo `level`). As omissões específicas de refletânciaespecíficos de refletância são `reflectance-skipped-no-fresh-dls` (nenhuma leitura nova de radiação descendente disponível), `reflectance-skipped-bound-daq-unavailable (…)` (não foi possível acessar o DAQ vinculado) e `dls-uncalibrated-band-<nm>` — a banda fica em sua maior parte fora da faixa radiometricamente calibrada do sensor de luz DAQ (~374–974 nm), de modo que a divisão de refletância baseada no DAQé rejeitada e o quadro reverte abertamente para a resposta do sensor. Entre os SKUs disponíveis, apenas o F988 aciona esse erro; o fluxo de trabalho suportado por essa câmera é o do painel de refletância.

Níveis do `processing`:

| Nível | Saída |
| --- | --- |
| `"raw"` | Bayer de canal único (câmeras monocromáticas: a banda única) diretamente do sensor. |
| `"debayered"` *(padrão do SDK)* | BGR de 3 canais via demosaico bilinear (câmeras monocromáticas: escala de cinza de 1 canal). |
| `"radiance"` | float32 W/m²/sr/nm por meio da cadeia radiométrica completa. Apenas multiespectral — câmeras RGB são ignoradas. |
| `"reflectance"` | uint16 0..32768 (compatível com Pix4D); requer emparelhamento com DAQ ativo para referência absoluta. Apenas multiespectral. |
| `"display"` | Cadeia completa correspondente à visualização da GUI (CCM + WB + gama de acordo com o perfil da câmera). |
| `"all"` | **Um arquivo por nível aplicável** para cada câmera (correspondendo à configuração padrão da GUI “Capturar tudo”/CLI). O `CaptureResult` retornado contém, então, um dicionário de quadro por `(cam, level)`, com o nível em cada dicionário; os níveis inaplicáveis aparecem no `.skipped`. A leitura do DAQ usada para qualquer quadro de refletância é salva como um sidecar `.daq`. |

> **Observação — o padrão difere do CLI.** O `ArraySession.capture()` tem como padrão o `processing="debayered"`; o comando `chloros-cli lattice array-capture` tem como padrão o `processing="all"`. Passe o `processing="all"` explicitamente a partir do SDK para espelhar o salvamento em vários níveis do CLI /GUI.

### Modos de captura e gravadores

A superfície da matriz espelha o painel de captura da GUI: Único / Contínuo / Intervalo / Mais rápido, além de dois gravadores (vídeo composto ao vivo e rajada em formato bruto → reprocessamento offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Combined-index video/GIF recorder (needs the combined live view streaming)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**é o loop Contínuo/Intervalo do SDK. Como não há `Ctrl+C` para interrompê-lo a partir de um script, você**deve** passar `count` e/ou `duration_s` (ele para quando qualquer um deles for atingido). O `interval_s` é medido a partir do início de cada passagem (em consonância com a GUI). Os kwargs restantes são passados diretamente para o `capture()`.
- **`record`** é *nível de monitoramento*: ele captura o composto de índice combinado em tempo real conforme exibido; portanto, o fluxo combinado deve estar aberto para que os quadros sejam recebidos. Um gravador de composto por matriz (gera exceção se já houver um em execução).
- **`burst` → `build_video`** é de *nível de análise*: o `burst` grava quadros brutos + um manifesto por quadro + um `.daq` por leitura DLS distinta sob o `<output>/bursts/<base>/` na taxa máxima do loop de captura (sem cadeia, sem exiftool, sem visualização ao vivo). O `build_video` sincroniza cada quadro com o `.daq` mais próximo e reexecuta a cadeia de radiação/refletância/índice do pipeline de importação. `products` é uma lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (padrão: o índice combinado). `burst().stop()` também inicia automaticamenteinicia automaticamente uma compilação do índice combinado com o melhor esforço possível, retornada como `build_job` no resultado da parada.

#### `RecorderHandle`

Retornado por `ArraySession.record()` e `ArraySession.burst()`. Use-o como um gerenciador de contexto para interromper automaticamente ao sair do escopo ou execute-o manualmente.

| Membro | Descrição |
| --- | --- |
| `job_id` | ID da tarefa de backend (str). |
| `kind` | `"composite"` (de `record`) ou `"raw"` (de `burst`). |
| `start_stats` | O dicionário retornado pela chamada `start`. |
| `result` | `None` durante a execução; o dicionário final do resultado da parada após a interrupção. |
| `stats(timeout=10.0)` | Estatísticas do trabalho em tempo real (quadros gravados, fps alcançados, tempo decorrido). |
| `stop(timeout=60.0)` | Interrompe o gravador; retorna e armazena em cache o resultado final. Idempotente (uma segunda chamada retorna o resultado armazenado em cache). |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

### Conectando-se a um array já conectado — `attach_array`

Se a matriz já estiver ativa (a GUI a abriu ou uma sessão anterior do SDK chamou `connect_array`), use `attach_array` para obter um identificador dela, em vez de se reconectar. `connect_array` sempre gera o erro “A câmera <sn>já </sn>está <sn>no conjunto <id>” nessa situação, porque enviar um POST com `/array/connect` para um membro do pool não é idempotente; o `attach_array` lê o `/api/camera/array/list` e faz a correspondência por array_id ou números de série.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Padrão: SDK Os scripts que compartilham o ambiente com a GUI da área de trabalho devem tentar primeiro o `attach_array` e, caso ainda não haja nenhum array no pool, recorrer ao `connect_array`.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Importante — a saída do context-manager DESCONECTA.**O `ArraySession.disconnect()` sempre envia um POST para o `/array/disconnect`; não há uma proteção “attached-not-owned” como existe para `CameraSession` / `DAQSensorSession`. Se você estiver compartilhando o espaço com a GUI e não quiser desmontar a matriz ao sair do escopo,**não use o bloco `with`** — mantenha o identificador em uma variável normal e ignore o `disconnect()` explícito:
>
> ```python
> arr = chloros_sdk.attach_array(serials)
> arr.capture("output/", processing="reflectance")
> # … script ends; array stays up for the GUI
> ```

### Auxiliar de Análise de Rede

Útil antes de abrir a matriz — permite prever se as configurações propostas serão adequadas:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    print(f"Shrink to {r['out_width']}x{r['out_height']} binning={r['binning']}")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

`status` é um dos seguintes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (caso contrário, `error`). `auto_capped_fps` significa que a resolução solicitada se encaixa no anel RX apenas com uma taxa de disparo limitada — mantenha a resolução e passe de `target_fps=result["recommended"]["recommended_target_fps"]` para `connect_array` (veja [Exemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)).

**Como interpretar a projeção** (mesmo modelo do painel “Configurações da Matriz” da GUI):

- **Burst (`frame_bytes_total`) é somado por câmera no formato de pixel real de cada câmera.**As câmeras mono**M3M**transmitem Mono12 (2 B/px) independentemente do valor `pixel_format` que você passar, portanto, um quadro em resolução total de 4 câmeras tem**~25 MB** com três câmeras mono, e não os ~12,6 MB que a suposição de 8 bits em todos os canais indicaria. O backend determina o formato de cada câmera com base em seu modelo.
- **A admissão (`burst_fits_nic_ring`) leva em conta o escoamento**, e não a comparação entre burst completo e anel: o sim-emit funciona quando o host esvazia o anel de recepção mais rápido do que as câmeras o preenchem. Um host de 10 G + câmeras de 1 GbE**admite** resolução total mesmo quando a sequência excede o anel; um host de 1 GbE bloqueia (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` é um limite máximo conservador para recuperação serial** — `max(readout+emit, N×emit)` com emissão poremissão por câmera limitada ao link de 1 GbE, independentemente da exposição. Por exemplo, ~2,8 fps para um conjunto de 4 câmeras em resolução total de 12 bits (corresponde aos ~2,7–3,0 medidos no tempo de execução). Modelo completo: [CLI Referência → Modelo de fps e rajada do conjunto](cli-reference.md#array-fps--burst-model).
- **A supersubscrição (`oversubscribed: true`) significa que o limite mínimo de N × por câmera excede o limite máximo seguro contra colisões** — os campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) apresentam valor 0, e a redução automática/agrupamento não conseguem corrigir isso (eles reduzem os bytes por quadro, não os bytes por segundo). As soluções são: reduzir o número de câmeras, usar quadros jumbo ou uma placa de rede mais rápida; `max_cams_collision_safe` informa o limite máximo (6 câmeras em resolução total em 1 GbE com MTU de 1.500, 9 com quadros jumbo). A resposta também contém os códigos `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps` (8 MB/s). Consulte [Sobrescrição](#over-subscription-the-per-cam-floor).

### Descoberta e listagem

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

---

## Smart-AE / Smart-Capture

Os arrays LATTICE executam o AE contínuo em segundo plano assim que são conectados, mas uma cena recém-focalizada leva um momento para convergir. **A Smart-Capture** é a solução prática integrada: ela verifica a exposição de cada câmera, aguarda até que o conjunto esteja estável em toda a janela e, então, aciona a captura. É equivalente à interface gráfica: o botão de captura “inteligente” do aplicativo para desktop aciona o mesmo endpoint de back-end.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Ao utilizar o `ChlorosProject` (próxima seção), você tem mais opções de ajuste:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

A política de AE inteligente é conservadora por padrão. Aperte o `exposure_tolerance_pct` para trabalhos radiométricos exigentes; alargue-o para cenas que mudam rapidamente, nas quais você só quer algo “suficientemente próximo”.

---

## Sessões de sensores DAQ

Pool de back-end persistente para sensores espectrais (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Espelha a superfície da câmera: detecção inteligente, reutilização do pool, conexão idempotente.

### Detecção inteligente (configuração zero)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioridade: Ethernet → BLE → USB. Passe qualquer indicação explícita para fixar o transporte.

### Transporte fixado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

### Métodos `DAQSensorSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Resumo da entrada do pool (estado de streaming/gravação, faixa de comprimento de onda, SHA de calibração, tempo de integração, frame_avg, estado AE). |
| `latest(n=1, timeout=10.0)` | Retorna até Nmais recentes. |
| `stream_start()` / `stream_stop()` | Retomar/pausar a transmissão (o identificador permanece aberto). |
| `record_start(output_dir=None, device_name=None)` | Iniciar a gravação de um arquivo .daq. Retorna o caminho do arquivo. É recusado para DAQ-U/M sem um pacote de calibração AWS (DAQ-E está isento). |
| `record_stop()` | Interrompe a gravação. Retorna `{path, rows}`. |
| `disconnect()` | Libera do pool. Não tem efeito para identificadores conectados, mas não pertencentes ao usuário. |

> **Os perfis de correção de cap (`cap_id`) não são um controle do tipo “SDK”.** `connect_daq_sensor()` / `DAQSensorSession` não expõem nenhum parâmetro `cap_id` nem método `set_cap`. Selecione um perfil de correção de limite da frota por meio do CLI (`chloros-cli daq pool-connect --cap-id …` / `chloros-cli daq pool-set-cap …`) ou pelas rotas `/api/daq` HTTP do backend (`/api/daq/connect` e `/api/daq/<id>/cap-id` aceitam `cap_id`).

### Descoberta — localizando um endereço para se conectar

O `discover_daq_sensors()` varre as interfaces USB / BLE / ETH em busca de sensores que você *poderia* abrir. É a contraparte do DAQ para o `discover_lattice_cameras()` e a única maneira de obter um **MAC BLE do DAQ-M** — um DAQ-E possui um nome de host e um DAQ-U, uma porta COM, mas o MAC não está impresso no dispositivo nem listado pelo sistema operacional.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               None   {'manufacturer': 'Intel'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrição |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome do host — passar para `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Rótulo legível por humanos. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` ou `None` para uma porta que a varredura não consegue identificar (adaptadores seriais USB são indistinguíveis sem um medidor, portanto, os desconhecidos são exibidos em vez de ocultados). |
| `extra` | Detalhes por transporte (nome anunciado do BLE, fabricante do USB, IP/FW do DAQ-E/…). Valores vazios são omitidos. |

| Parâmetro | Padrão | Descrição |
| --- | --- | --- |
| `transports` | todos os três | Sequência (ou string CSV) que limita a varredura. Vale a pena passar esse parâmetro quando você sabe o que deseja — o BLE é a parte mais lenta. |
| `scan_timeout` | 5 | Janela de varredura por transporte em segundos; o backend limita o valor a 1–20. |
| `timeout` | 60,0 | Limite máximo de “HTTP” para toda a chamada (assim como em outras partes do SDK). |
| `auto_start_backend` | `True` | Inicia um backend local se nenhum estiver em execução. Nunca é iniciado para um `backend_url` remoto. |

> **Sensores já abertos no pool não aparecem.** Um periférico BLE conectado interrompe a transmissão de anúncios e uma porta COM aberta não pode ser detectada; portanto, a descoberta lista o que está *disponível para conexão*. É esperado um resultado vazio logo após você conectar algo — use o `list_daq_sensors()` para o que você já possui. Transportes cuja varredura não pode ser executada (sem bleak/zeroconf instalados) são ignorados em vez de gerarem erros, de modo que uma máquina sem Bluetooth ainda recebe suas respostas de USB e ETH.

### Listagem

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Compartilhamento com GUI/CLI

Se a GUI já tiver um sensor aberto, chamar `connect_daq_sensor(port="COM3")` a partir de Python retorna um identificador marcado como `already_connected=True`. O `disconnect()` da sessão passa a ser uma operação nula, de modo que seu script SDK nãoconsegue desligar o sensor da GUI ao sair do scope.

### Classes de hardware direto (sem backend)

`daq_sdk` é reexportado por `chloros_sdk`, de modo que você também pode controlar sensores de ponta a ponta durante o processo sem o backend:

> **Disponibilidade:**o `daq_sdk` vem incluído na instalação do desktop do Chloros,**não** no pacote do PyPI — O `pip install chloros-sdk` fornece o `lattice_sdk`, mas deixa de fora o `chloros_sdk.DAQ_AVAILABLE == False`. Verifique esse sinalizador antes de usar essas classes; em um host que utilize apenas o pip, acione o sensor por meio do [`connect_daq_sensor()`](#daq-sensor-sessions), que não requer bibliotecas de transporte locais.

```python
from chloros_sdk import DAQUSensor, DAQMSensor, DAQESensor, discover_all

# Discovery
for d in discover_all(timeout=3.0):
    print(d.model, d.display, d.address)   # USB serials: d.extra.get("serial_number")

# Direct DAQ-U
sensor = DAQUSensor(port="COM3")
sensor.connect()
sensor.start_streaming()
# ... use sensor.add_spectrum_callback(...) ...
sensor.stop()
```

Prefira o caminho de conexão inteligente (`connect_daq_sensor`) quando desejar propriedade compartilhada com a GUI; use as classes diretas para scripts sem interface gráfica que possuam o sensor exclusivamente.

---

## Automação de Projetos — `ChlorosProject`

Um projeto “Chloros” salvo é uma pasta contendo `cameras.json` + `sensors.json` + `project.json`. O `open_project` carrega o manifesto, e o `connect_all` coloca todos os dispositivos salvos online com suas configurações salvas — no mesmo estado de hardware que a interface gráfica (GUI) produziria.

### Exemplo mínimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Read a DAQ
spectrum = proj.sensors["Sky"].read()

# Trigger every device simultaneously
proj.capture_all("./out")

proj.disconnect_all()
```

Ou como um gerenciador de contexto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Métodos do `ChlorosProject`

| Método | Descrição |
| --- | --- |
| `connect_all(cameras=True, arrays=True, sensors=True, verbose=False, align=None)` | Descobre e conecta todos os dispositivos salvos. Retorna um relatório de conexão por classe. Utiliza um backend em execução quando há um em escuta no `127.0.0.1:5000`; caso contrário, recorre silenciosamente ao modo direto (sem backendsem backend) — nunca inicia um backend. |
| `disconnect_all()` | Desconecta tudo. |
| `capture_all(output_dir=".")` | Um quadro de cada câmera + matriz + espectro de cada sensor. |
| `stream(camera, overlays=False, fps=10.0)` | Gerador que produz quadros BGR `numpy` de uma câmera (ou matriz) nomeada. `overlays=False` é um loop de captura direto `lattice_sdk` (matrizes geram dicionários `{serial: frame}`). O `overlays=True` é roteado por meio do `ChlorosLocal.camera_stream()` → o feed MJPEG `/api/camera/<serial>/stream-annotated` do backend, com o bloco `ui.overlay` salvo pela câmera sendo passado como parâmetros de consulta. Requer o modo backend e uma **câmera autônoma**: uma câmera no modo direto gera o erro `RuntimeError` (o backend não consegue acessar uma câmera pertencente a esse processo) e um array gera `NotImplementedError` (sobrepõe o composto por câmera — transmite um membro pelo nome). Equivalente de execução única: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Executa o alinhamento em todas as matrizes conectadas no momento. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executa o pipeline de calibração / pipeline de indexação nas imagens do projeto (envolve `ChlorosLocal.process`; esses quatro são os **únicos** argumentos-chave aceitos — `indices=` etc. geram exceção `TypeError`; define índices via `ChlorosLocal.configure()`). Constrói de forma preguiçosa um `ChlorosLocal()`, que inicia automaticamente um backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` indexado por nome E número de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexado por nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexado por nome E slot_id.
- `proj.config` — `project.json["config"]` (dicionário).

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Níveis de processamento.** `capture()`, `grab()` e `frame_stream()` utilizam todos o mesmo token `processing`
, e a cadeia é cumulativa — cada nível executa tudo o que está acima dele:

| Nível | Saída | Observações |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo do sensor | Sem demosaico. Sobreposições não estão disponíveis neste nível. |
| `debayered` | BGR de 3 canais (**padrão**) | Desmosaico bilinear. O único nível que funciona sem o modo backend. |
| `radiance` | float32, W/m²/sr/nm | Cadeia radiométrica completa: desmosaico + separação 3×3 (multiespectral) + DSNU + campo plano + escala NIST, com exposição × ganho eliminados para que os valores sejam absolutos. |
| `reflectance` | uint16, 32768 = 1,0 | Radiação dividida pela irradiância descendente (ρ = π·L/E). Requer uma leitura DLS/DAQ — veja a nota abaixo. |
| `display` | 8 bits, semelhante a sRGB | Renderização equivalente à GUI: CCM + balanço de branco + gama por meio do perfil de cor ativo da câmera. |

Qualquer coisa diferente de `debayered` requer o modo backend; umagera
`NotImplementedError`. `reflectance` requer uma leitura de luz descendente utilizável — o ponto final do quadro puxa
o DAQ agrupado para o slot DLS da câmera automaticamente, mas, sem um DAQ vinculado, a cadeia recusa a
saída de refletância e registra honestamente o rebaixamento nos metadados retornados, em vez de silenciosamente
devolver um produto de qualidade inferior.

> **Escala DN de refletância — não a codifique fixamente.** A refletância LATTICE usa `32768` = ρ 1,0 e registra
> XMP `Chloros:PixelScale=32768`; a refletância Survey3 usa `65535` = ρ 1,0 e carrega nenhuma
> tag `Chloros:*`. Leia a tag e divida por ela. Ela está definida no domínio uint16, portanto permanece
> `32768` para todos os formatos que redimensionam (TIFF de 16 bits, PNG /JPG de 8 bits, percent de 32 bits) — normalize
> primeiro o tipo de dados armazenado de volta para uint16 (×257 de 8-bit, ×65535 a partir de float). A única exceção:
> uma captura com fonte de 8 bits gravada como TIFF de 8 bits é *cortada*, não redimensionada; portanto, nenhuma escala descreve
> ela — Chloros omite totalmente o `PixelScale` e a tupla MicaSense nesse caso. Trate uma
> tag ausente em um arquivo de refletância LATTICE como “nenhuma escala válida”, e não como um padrão.

> **EXIF transferido para a exportação.** `process()` copia o bloco GPS da captura de origem
> **e seu ExifIFD** para cada produto, portanto, as exportações contêm `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, bem como a
> georreferência. `FocalLength` é o valor a partir do qual o Pix4D calcula a distância de amostragem do solo — sem ela,
> a reconstrução recai em uma escala totalmente errada (um caso medido transformou um local de 411 m
> em um de 47,8 km). A cópia não é deliberadamente `-all:all`: as tags estruturais do IFD0 quebram
> a saída do LATTICE, e `ExifImageWidth`/`Height` são excluídos porque descrevem a captura da fonte
> em vez da imagem raster exportada.

Sub-sinalizadores da fase de captura (aplicáveis aos níveis radiométricos — `radiance`, `reflectance`, `display`):

| Sinalizador | Padrão | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + separação 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de balanço de branco. Compatível com DLS quando um DAQ está vinculado à câmera. |
| `apply_index` | `False` | Avaliação do índice de vegetação. |
| `index_expression` | `None` | Fórmula de substituição. Se não estiver vazio → habilita automaticamente o índice. |
| `annotated` | `False` | Sobreposição de decorações da GUI (zebra/grade/pico). Não disponível para `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **O tipo de retorno é `CapturePathMap`, e não `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` é `Dict[str, Union[str, List[str]]]`: um de nível único
> `processing` atribui um caminho a cada serial, enquanto um de nível múltiplo (`"all"`, ou uma
> lista explícita `levels`) fornece a **lista ordenada** de todos os produtos salvos para aquela
> câmera. Uma composição combinada ao vivo, caso esteja sendo transmitida, chega sob a chave extra
> `"combined"`, em vez de sob um número de série. O código que pressupõe `str` apresenta falha na
> formato de lista sem que nenhum verificador de tipos levante objeções — a anotação indicava `Dict[str, str]`
> por algum tempo após o lançamento do formato de lista, e é por isso que o alias existe. Normalize
> quando desejar o formato plano:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

### Alinhamento de matrizes

`ArrayHandle` expõe toda a superfície de alinhamento. Os perfis são válidos apenas para a sessão por padrão — chame `export_alignment()` explicitamente para persistir.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alinhamento no momento da conexão

`connect_all(align=...)` pode alinhar automaticamente todas as matrizes no momento da conexão:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Recorre ao `project.json["config"]["auto_align_on_connect"]` quando não especificado.

### `SensorHandle`

```python
spectrum = proj.sensors["Sky"].read()
# (spectrum_list, is_saturated, integration_time, x, y, z) — matches the
# daq_sdk add_spectrum_callback signature.
```

---

## Hardware direto (sem backend)

Quando você deseja zerar a dependência do backend (CI, robôs headless, embarcados), importe `lattice_sdk` e `daq_sdk` diretamente — ambos são reexportados por `chloros_sdk`. Verifique se `CAMERA_AVAILABLE` / `DAQ_AVAILABLE`: o `lattice_sdk` está no pacote PyPI (mas requer a presença do runtime Arena SDK), enquanto o `daq_sdk` vem apenas com a instalação para desktop.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Predefinições e o gatilho

Três das quatro predefinições **free-run**: a câmera expõe continuamente e um
`capture()` retorna o próximo quadro. O `triggered` é a exceção — ele arma a
câmera para uma mudança de estado de hardware na Linha 2, portanto não captura nada até que ocorra uma.

| Predefinição | Acionador | Use quando |
| --- | --- | --- |
| `default` | free-run | uso geral |
| `high_speed` | free-run | 8 bits, limite de 60 fps, exposição curta |
| `high_quality` | free-run | 12 bits, sem limite de fps — a escolha usual para fotos |
| `triggered` | **preparada, Linha 2** | a câmera está conectada a um cabo de sincronização M8 e outro dispositivo a aciona |

Se você escolher `triggered` (ou configurar `trigger_mode="On"` manualmente) sem nada
acionando a Linha 2, todos os `capture()` entrarão em timeout — corretamente, já que você solicitou
que a câmera aguardasse. O SDK explica isso quando ocorre; consulte
[SC_ERR_TIMEOUT durante a captura](#direct-hardware-backend-free).

> **Observação — As mensagens “GVSP probe” / `SC_ERR_TIMEOUT -1011` na conexão não são erros.**&gt; Ao se conectar, o SDK tenta negociar**quadros jumbo** (pacotes GVSP de 9.000 bytes) para obter maior taxa de transferência. Em um link direto ponto a ponto de placa de rede (por exemplo, um endereço `169.254.x.x` local do link), a rede geralmente não consegue transportar quadros jumbo, portanto, essa tentativa expira e gera mensagens como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Esse é o **plano de contingência previsto**: o SDK reverte automaticamente para pacotes padrão de 1.500 bytes e a câmera continua se conectando normalmente (as linhas `[chunk-enable …]` que se seguem fazem parte da sequência normal de conexão). A captura continua funcionando.
>
> Você pode pular essa sonda, mas **não se trata apenas de um silenciador de logs — ela desativa os quadros jumbo.** A câmera responde a pings “Don&#x27;t-Fragment” apenas até 1.500 bytes, independentemente da qualidade da sua rede; portanto, o teste de ping por si só nunca consegue detectar quadros jumbo; esse teste é a única forma de fazê-lo. Desative-o e a câmera continuará pacotes padrão de 1.500 bytes para sempre, em qualquer rede:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Só vale a pena em uma rede que você *sabe* que não suporta jumbo, onde economiza cerca de um segundo no tempo de conexão por câmera. Como se trata de uma troca real, e não apenas cosmética, o SDK agora informa isso quando você usa a função:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Não altere essa configuração, a menos que tenha um motivo.** Se permanecer ativada, cada conexão reavalia a rede disponível: conecte-se a um switch compatível com pacotes jumbo e a próxima conexão detectará os pacotes jumbo automaticamente, sem necessidade de configuração e sem reinicialização.
>
> Se você *quiser* a taxa de transferência jumbo, habilite o jumbo de ponta a ponta (MTU da placa de rede de 9.000 + um switch que os transmita), ou defina-o com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando souber que o link suporta isso — embora seja preferível usar o comando `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando específico em vez de configurá-lo permanentemente, já que um tamanho fixo ignora a sondagem e impede a adaptação à rede à sua frente. **Todos os** dispositivo no caminho precisa transmitir pacotes jumbo — incluindo qualquer divisor ou injetor de PoE, que é o motivo comum pelo qual uma configuração que, de outra forma, seria compatível com jumbonão consiga transportá-los.

> **O código `SC_ERR_TIMEOUT -1011` durante o `capture()` / `grab*()` é um problema diferente — esse é um erro real.**&gt; A nota acima refere-se apenas ao `-1011` registrado pela**sonda de tempo de conexão**. O mesmo erro gerado a partir de uma**captura** significa que a câmera se conectou corretamente, mas não está enviando imagens:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> O indício revelador é uma câmera cujo canal de *controle* está funcionando corretamente — a detecção funciona, as configurações e as gravações do `[chunk-enable …]` são bem-sucedidas — enquanto *todos* os quadros atingem o tempo limite.
>
> **A causa mais comum é que a câmera está configurada para um acionador de hardware.** Com `trigger_mode="On"` e `trigger_source="Line2"`, a câmera não emite absolutamente nada até que uma borda elétrica chegue ao cabo de sincronização M8. Se não houver nenhum cabo acionando essa linha, cada captura fica esperando indefinidamente. A câmera não está com defeito e a rede está funcionando normalmente — ela está fazendo exatamente o que foi programada para fazer.
>
> `CameraSettings()` e os códigos `default` / `high_speed` / `high_quality` predefinem o modo livre, e uma captura que atinge o tempo limite enquanto está armada exibe uma mensagem explicativa em vez de apenas imprimir um `-1011`. O `PRESETS["triggered"]` arma a Linha 2, conforme projetado.
>
> Para forçar qualquer câmera a operar em modo livre:
>
> ```python
> settings = PRESETS["high_quality"]
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se ainda assim ocorrer tempo limite com `trigger_mode="Off"`, a câmera realmente não está enviando dados — envie-nos o log e `ip link show`.

#### Perfis de cor (visualização ao vivo do RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleciona o perfil de cor de exibição para a **visualização ao vivo** em câmeras RGB (câmeras multiespectrais ignoram essa configuração):

| Perfil | Significado |
| --- | --- |
| `raw` | Ignora completamente a cadeia radiométrica. |
| `linear` | DSNU + flat + WB, sem CCM, sem gama. |
| `natural` | Linear + CCM medido + gama sRGB, apenas com o acabamento básico (suavização de croma + dessaturação de realces) — a configuração padrão realista. |
| `enhanced` | `natural` mais o acabamento completo com paridade de hub (remoção de franjas, vibração, contraste local CLAHE). Aparência mais rica com aproximadamente **o dobro do custo de acabamento por quadro**, portanto, uma taxa de quadros AO VIVO mais baixa. |
| `custom_temp` | `natural`, mas o equilíbrio de branco fixado em `custom_cct_k` Kelvin (DLS ignorado; limitado a 2.000–10.000 K no backend). |

O perfil é um controle de velocidade/aparência **exclusivo para visualização ao vivo**: as capturas salvas sempre recebem o acabamento completo e rico, independentemente do perfil selecionado; portanto, escolher o `natural` para ganhar tempo de quadro não reduz a qualidade do que é gravado no disco. Um perfil desconhecido aumenta o `ValueError`; quando um backend chloros está acessível, a alteração também é enviada por POST para ele, de modo que o próximo quadro de pré-visualização reflita essa mudança (usuários do direct-SDK sem um backend ainda recebem a alteração nas configurações).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Câmeras Mono (M3M) e `Calibration`

Uma câmera monocromática **M3M** (`M3M-<lens>-F<wavelength>`) é de banda única: um plano em escala de cinza, sem mosaico de Bayer, sem matriz de interferência espectral 3×3. O `Calibration` reconhece isso e expõe um sinalizador `is_mono`. A refletância ainda se aplica como ummapa radiométrico de banda única (a separação de cores é a matriz identidade), mas cálculos multibanda em uma única câmera geram resultados válidos em vez de retornarem valores sem sentido:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Para construir um índice de vegetação a partir de hardware monocromático, combine várias câmeras M3M em diferentes comprimentos de onda em uma pilha multibanda alinhada (consulte [Alinhamento de Matriz](#array-alignment)) e calcule o índice nessa pilha, em vez de em uma única câmera.

Modo direto do DAQ:

```python
from chloros_sdk import (
    DAQUSensor, DAQMSensor, DAQESensor,
    SensorFleet, discover_all, DiscoveredSensor,
    apply_sensor_settings, SensorSettings,
)

for d in discover_all(timeout=3.0):
    print(d)

sensor = DAQUSensor(port="COM3")
sensor.connect()
apply_sensor_settings(sensor, settings={"integration_time_ms": 64, "frame_avg": 20})
sensor.start_streaming()
# ... sensor.add_spectrum_callback(your_callback) ...
sensor.stop()
```

> **Chaves aceitas pelo `apply_sensor_settings`**— exatamente `integration_time_ms`, `frame_avg`, `ae_enabled`, `sunshine_diffuser_installed` (DAQ-E; obsoleto em favor de `cap_id`), `filter_model` (DAQ-M), e `cap_id` (todos os tipos de DAQ; `None`/`""`/`"none"` = sensor simples, sem correção de tampa). Chaves desconhecidas são**silenciosamente ignoradas** — por exemplo, `{"integration_time": 64}` não faz nada (deve ser `integration_time_ms`). Retorna `{"applied": [...], "errors": {...}}` e nunca gera exceção.

`chloros_sdk`exporta apenas a superfície principal usada acima. O API público completo `daq_sdk` (22 nomes) adiciona o seguinte — importe-os diretamente de `daq_sdk`:

```python
from daq_sdk import (
    DAQULogger, DAQMLogger, DAQELogger,     # rotating-file recorders (the ones the GUI uses)
    ConnectResult, FleetRecordResult,       # SensorFleet result types
    discover_all_detailed, build_sensor,    # detailed discovery + build-by-descriptor
    scan_eth_devices, DaqEControl,          # DAQ-E Ethernet scan + control channel
    scan_ble_devices, detect_ble_device, list_ble_devices,   # DAQ-M BLE discovery
    detect_port, list_serial_ports,         # DAQ-U serial-port discovery
    TcpSerial,                              # serial-over-TCP transport shim
)
```

---

## Exceções

Intercepte a classe base para lidar com “qualquer coisa que deu errado Chloros”:

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

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` são exportados no nível superior junto com os demais; eles também podem ser importados a partir de `chloros_sdk.exceptions`, conforme mostrado.

Hierarquia:

```

ChlorosError
├── ChlorosBackendError           (backend failed to start / unreachable)
├── ChlorosConnectionError        (HTTP transport failure)
├── ChlorosLicenseError           (subscription / tier gate)
├── ChlorosAuthenticationError    (login required)
├── ChlorosConfigurationError     (bad configure() / open_project() inputs)
└── ChlorosProcessingError        (pipeline failed)

ChlorosConnectError                (raised by connect_camera / connect_array /
                                    connect_daq_sensor only — derives from
                                    plain Exception, NOT from ChlorosError,
                                    so `except ChlorosError` will not catch it)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemplos de ponta a ponta

### 1. Processar uma pasta com uma barra de progresso personalizada

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "NDRE", "GNDVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matriz LATTICE em tempo real → Refletância + Referência DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanha de captura orientada a projeto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Fluxo de quadros de múltiplas câmeras → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captura direta no hardware sem interface gráfica (sem backend)

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verificação de recursos antes de conectar um conjunto de 4 câmeras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["out_width"], height=r["out_height"],
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente à receita de captura (Python puro)

A DSL de receitas do CLI possui um equivalente direto no Python:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply
    # (CameraHandle has no direct apply method; use the underlying lattice_sdk
    #  helper or the backend's /api/camera/<sn>/apply-settings via requests)
    # For most cases just use cam.cam.set_exposure(...) in direct mode or
    # the GUI's saved settings via project.connect_all().

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Início automático do backend

Os pontos de entrada do smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor` e `discover_lattice_cameras` — são clientes “HTTP” leves que presumem que um backend esteja à escuta no `127.0.0.1:5000` (o endereço URL padrão da superfície do smart-connect). Quando a GUI ou o CLI já estiver em execução, um deles já estará. A partir de um script básico, pode não haver — portanto, essas funções **inicializam automaticamente o binário do backend incluído** (sem janela, da mesma forma que o `ChlorosLocal` faz) antes de sua primeira chamada e, em seguida, aguardam até `backend_startup_timeout` para que ele seja iniciado.

Regras:

- **Apenas um URL local é iniciado.** Um `backend_url` apontando para `localhost` / `127.0.0.1` / `[::1]` é elegível; qualquer outro host é considerado como sendo a máquina de outra pessoa e nunca é iniciado.
- **O backend permanece em execução para reutilização** (da mesma forma que o CLI) — não há desligamento implícito quando seu script é encerrado. Ao executar o script novamente, o backend ativo é reutilizado.
- **Desative essa opção com `auto_start_backend=False`** em qualquer uma dessas chamadas (por exemplo, quando você apontou para um backend remoto ou gerencia o ciclo de vida do backend por conta própria).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se o binário incluído não puder ser localizado ou iniciado, a chamada subsequente HTTP gera um rastreamento acionável, **específico para a plataforma** `ChlorosConnectError`, em vez de um simples rastreamento de conexão recusada — no Windows, ele direciona você para o aplicativo de desktop ou para um comando `chloros-cli`; no Linux (sem GUI), ele direciona você para um comando `chloros-cli` ou para o `.deb`.

---

## Ambiente e cabeçalhos

O SDK marca todas as chamadas de backend HTTP com `X-Chloros-Client: sdk`. O backend aplica as regras de licenciamento SDK / CLI (é necessário fazer login **e** ter um plano pago Chloros+), em vez do caminho de nível gratuito da GUI. Isso é definido automaticamente no momento da importação — você não precisa fazer nada.

`http://localhost` e `http://127.0.0.1` são detectados como backend local. Chamadas para outros hosts (por exemplo, seu próprio serviço de análise) não são alteradas.

Substitua o backend URL passando `backend_url=` (ou `api_url=` no `ChlorosLocal`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Um `backend_url` que não seja de loopback alcança apenas um backend de origem/backend de dispositivo — os backends fornecidos se limitam ao loopback; consulte o Modo de Backend Remoto para o padrão de túnel.)

---

## Versões e Compatibilidade

- A versão SDK é exposta como `chloros_sdk.__version__`.
- O SDK vincula o comportamento à versão do backend incluído. Combinar um backend mais antigo SDK com um backend mais recente geralmente funciona (pontos finais compatíveis com versões futuras), mas combinar um backend mais recente SDK com um backend mais antigo pode gerar erros `404` em novos pontos finais — atualize o aplicativo para desktop para que fique compatível.
- A interface do Smart Connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) e o endpoint de análise de rede retornam esquemas estáveis de JSON; novos campos são adicionais.

---

## Dicas para solução de problemas

- **`ChlorosAuthenticationError: Login required`** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina ou faça login por meio do aplicativo de desktop Chloros.
- **`ChlorosConnectError: No Chloros backend is running …`** → As chamadas de conexão inteligente iniciam automaticamente um backend local; portanto, isso só aparece quando o binário incluído não pode ser encontrado/iniciado (por exemplo, um host que usa apenas o pip, sem o pacote para desktop). A mensagem é específica para cada plataforma: em Windows, abra o aplicativo para desktop ou execute qualquer comando `chloros-cli`; em Linux, execute um comando `chloros-cli` (não há GUI) ou instale o `.deb`. Para um backend remoto, passe o `backend_url=` (e o `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** na importação → falha na carregar (normalmente, as DLLs de tempo de execução do Arena SDK não estão instaladas). A superfície sem câmera ainda funciona.
- **A conexão de matriz retorna resolução inferior à nativa**→ O smart-prep do backend reduz automaticamente o tamanho do quadro para se ajustar ao wireframe. Use o `analyze_array_network()` para verificar o motivo; em seguida, atualize o link, aceite a redução ou passe `force_tier="slip-emit-and-capture"` para captura sequencial. A rede de segurança contra redução**não** cobre a agregação por-inscrição (`oversubscribed: true`, campos de fps 0): o excesso de câmeras para a largura de banda não pode ser corrigido por binning/ROI — reduza o número de câmeras, habilite quadros jumbo ou mude para uma placa de rede mais rápida (consulte [Superscrição](#over-subscription-the-per-cam-floor)).
- **`analyze_array_network()` relata que o anel de recepção da placa de rede está muito pequeno (~0,26 MB) / portas de conexão exibem “FRAMES WILL DROP”** → O anel de recepção da placa de rede do host está em seu valor padrão (geralmente redefinido para 32 após uma atualização do driver da placa de rede). Em um adaptador Realtek USB 10 GbE, defina `ReceiveBufferLen=256` e `PendingReceives=64` (com privilégios elevados) e, em seguida, reinicie o backend para que ele releia o anel. Procedimento completo: [CLI Referência → Configuração e ajuste da placa de rede do host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **O host trava ao reiniciar/desligar; posteriormente, ocorrem erros WMI `Invalid class` / a placa de rede não é ativada** → Driver USB 10GbE desatualizado causando `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o driver do adaptador para uma versão atual (≥ 2026) e reaplique as configurações do anel de recepção. Consulte [Referência do CLI → Configuração e ajuste da placa de rede do host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Refletância recusada** → Um DAQ ativo deve estar vinculado à câmera (ou matriz) para obter a refletância em escala absoluta. Faça a vinculação por meio da GUI ou use o `processing="radiance"` (W/m²/sr/nm), que não requer um sensor emparelhado.
- **A captura do `smart=True` demora mais do que o esperado** → A convergência do AE depende da dinâmica da cena; aumente o valor do `exposure_tolerance_pct` ou diminua o do `stability_window_s` se desejar um disparador mais rápido (menos estável).

---

## Veja também

- [Referência do CLI](cli-reference.md) — cada subcomando CLI corresponde a uma chamada SDK.
- [Guia de sensores DAQ](../daq/README.md) — regras específicas de fiação, calibração e gravação para cada sensor.
- Documentação online: `https://mapir.gitbook.io/chloros/api-python-sdk`</id></sn>
