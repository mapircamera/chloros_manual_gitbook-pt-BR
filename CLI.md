# CLI : Linha de comando

> **Referência completa:**[CLI Referência](reference/cli-reference.md) documenta**todos os parâmetros de todos os subcomandos** e está otimizada para assistentes de IA — cole o URL no seu assistente e peça um comando que funcione: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Dica para ferramentas de IA:** qualquer página deste manual está disponível no formato Markdown bruto ao acrescentar `.md` ao seu URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), e `https://mapir.gitbook.io/chloros/llms.txt` indexa o manual inteiro para uso por LLM.

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.2.0 build so the version line reads "Chloros CLI 1.2.0" -->


## O que é oCLI


O `chloros-cli` é a interface de linha de comando do mesmo mecanismo de processamento usado pelo aplicativo de desktopChloros
. Trata-se de um cliente “HTTP
” leve sobre o back-endChloros
(um servidor local no `127.0.0.1:5000`) — a maioria dos comandos inicia o back-end automaticamente, de modo que uma única chamada ao `chloros-cli process …` é tudo o que um script precisa.

Ele roda no **Windows
10/11 (x64)**e no**Linux
(x86_64 e NVIDIA Jetson arm64 no JetPack 6)**, em qualquer terminal, sem necessidade de interface gráfica. Verifique sua instalação com:

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```

Visão geral das famílias de comandos:

* **Processamento e conta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — consulte [Idiomas suportados](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (somenteLinux
/Jetson)
* **Hardware ativo** — `lattice` (controle de câmera LATTICE, mais de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automação** — `project` (executar um projeto salvo doChloros
sem interface gráfica, incluindo receitas de captura em YAML)

Opções globais que vale a pena conhecer: `--port N` (porta do backend, padrão `5000`), `-v/--verbose`, `--restart` (reiniciar forçado o backend), `--backend-exe PATH`. Consulte a [Referência doCLI
](reference/cli-reference.md) para obter a lista completa.

***

## Instalação

OCLI
**vem incluído no instalador doChloros** em todas as plataformas — não há um download separado doCLI
. Baixe o instalador na página [Download](download.md).

###Windows


O instalador coloca oCLI
em:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e adiciona essa pasta ao seu sistema `PATH` — **abra um novo terminal**após a instalação para que o `PATH` atualizado seja reconhecido. O instalador também coloca scripts de inicialização (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) na raiz da instalação, além de um**Chloros
CLI
** no menu Iniciar, cada um dos quais abre um terminal com o `chloros-cli` pronto para uso.

###Linux


Instale o `.deb` para sua arquitetura:

```bash
# Linux x86_64
sudo dpkg -i chloros-amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Isso instala o `chloros-cli` até o `/usr/bin/chloros-cli` (já na versão `PATH`) e o backend para a versão `/usr/lib/chloros/chloros-backend`, juntamente com o ambiente de execução do ArenaSDK
necessário para as câmeras LATTICE. Consulte [Linux
Instalação](linux/linux-installation.md) para obter detalhes.

### Verificação

```bash
chloros-cli --version    # "Chloros CLI 1.2.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Login e Licenciamento

CLI
(ePython
SDK
) o acesso requer um **plano pagoChloros
+**— qualquer plano pago oferece esse recurso; o plano gratuito não. A restrição é aplicada**no lado do servidor** pelo backend, não pelo binárioCLI
: uma chamada sem login é rejeitada com o código de erro `401 AUTH_REQUIRED`, e uma chamada com usuário conectado no plano gratuito com o código de erro `403 PLAN_UPGRADE_REQUIRED`, independentemente de ela vir do `chloros-cli`, doSDK
ou de um clienteHTTP
desenvolvido manualmente. Faça a atualização em [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Faça login **uma vez por máquina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.2.0 build showing the license tier line -->


{% hint style="warning" %}
**Senhas com caracteres especiais**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` é corrompida pelo shell; oCLI
detecta isso com um erro 401 e tenta novamente automaticamente, mas aspas simples evitam o problema por completo).
{% endhint %}

A sessão é armazenada em cache em `~/.chloros/user_session.json` e continua funcionando offline durante o período de carência do plano (30 dias para planos mensais; até o vencimento para planos anuais). O `chloros-cli status` funciona mesmo sem um plano pago, portanto, o motivo da recusa está sempre visível.

{% hint style="danger" %}
**Está agendando tarefas sem interface gráfica? Faça login primeiro.**Um comando de geração de back-end (`process`, `status`, `export-status`, …) executado**sem sessão em cache**não falha rapidamente — ele entra em um prompt interativo `Email:` / `Password:` na stdin. Uma tarefa cron ou etapa de CI não supervisionada, portanto,**ficará travada aguardando entrada**. Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez na máquina antes de agendar qualquer coisa.
{% endhint %}

***

## Sua primeira execução de processamento

Aponte o `process` para uma pasta de capturas — ele detecta automaticamenteSurvey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ou uma combinação:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

O progresso é transmitido ao vivo por cada thread do pipeline (Detecção, Análise, Processamento, Exportação), e uma execução bem-sucedida termina relatando quantos produtos de imagem foram gravados (`Image products written: N`).



<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N" summary line -->
### Onde os resultados são salvos

`process` grava em uma **pasta de projeto**, não na sua pasta de entrada:

* Sem o `-o`: o projeto é criado na sua pasta de projetos padrão (compartilhada com a interface gráfica; gerencie-a com o `get-project-folder` / `set-project-folder`, alternativa `~/Chloros Projects`), nomeada por `-n/--project-name` ou por um carimbo de data/hora (`YYYYMMDD_HHMMSS`) quando omitida.
* Com `-o PATH`: essa pasta **é** a pasta do projeto. Se ela já contiver um `project.json`, um arquivo irmão com o sufixo `_1`/`_2`… será criado em vez de sobrescrever o existente.

Dentro do projeto, os produtos são agrupados **por câmera e, em seguida, por formato de arquivo**:

```
<project>/
├── project.json
├── calibration_data.json
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para LATTICE (correspondendo ao EXIF da captura `Model`) e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) paraSurvey3
. A pasta de formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para `TIFF (32-bit, Percent)`.

{% hint style="info" %}
**Todo produto exportado mantém o nome do arquivo de ORIGEM.**Uma exportação do Radiance do `capture_..._raw.tif` ainda é chamada de `capture_..._raw.tif` — ela apenas fica na pasta `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do arquivo**; portanto, use um padrão de busca para o diretório, não para o sufixo `*radiance*`.
{% endhint %}

### As opções que você realmente usará

| Sinalizador | Padrão | O que faz |
| --- | --- | --- |
| `-o, --output PATH` | pasta padrão do projeto | Localização da pasta do projeto (veja acima). |
| `-n, --project-name NAME` | carimbo de data/hora | Nome do projeto. |
| `--format FMT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)` ou `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação a serem exportados (consulte [Índices de vegetação](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neural, mais lento, qualidade máxima (Chloros
+, GPU NVIDIA). |
| `--vignette / --no-vignette` | ativado | Correção de vinheta. |
| `--reflectance / --no-reflectance` | ativado | Calibração de refletância; para LATTICE, isso também ativa ou desativa o produto de refletância. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para arquivos TIFF do LATTICE. |

Para todo o restante — ajuste de detecção de alvo, PPK, pinos de exposição, sinalizadores de alinhamento de matriz — consulte a [seção `process` da Referência doCLI
](reference/cli-reference.md).

***

## Escolhendo o que exportar (produtos LATTICE)

O processamento do LATTICE se ramifica para **todos os produtos aplicáveis em uma única passagem**. Quatro opções por produto estão todas**ativadas por padrão**; use o formulário `--no-` para desativar uma delas:

| Botão | Produto |
| --- | --- |
| `--debayered` | Demosaico linear → `Debayered_Images/` |
| `--preview` | Visualização na tela (equilíbrio de branco + gama; expansão de cores falsas para multiespectral) → `Preview_Images/` |
| `--radiance` | radiancia float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 refletância, compatível com Pix4D → `Reflectance_Calibrated_Images/` |

RGB
as câmeras master sempre emitem apenas dados pós-debayering + pré-visualização — a radianância/refletância por banda não tem significado para um sensor de banda larga, portanto, essas opções não têm efeito para elas.Survey3
`.raw` ignora as opções e segue o caminho padrão de refletância/alvo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (padrão `auto`) seleciona a referência de refletância: `auto` cria um [alvo de calibração](calibration-targets.md) dentro do quadro que passa no controle de qualidade como referência absoluta e recorre à divisão de incidência descendente do sensor de luz do DAQ (ρ = π·L/E) quando não há alvo presente; o `target` é estrito (sem substituição pelo DAQ); `daq` tem autoridade do DAQ. Varreduras de alvos medidas por unidade podem ser fornecidas com `--target-reflectance-dir`.

{% hint style="info" %}
**Leitura de pixels de refletância:**o DN que significa ρ = 1,0 é**por fonte** — Os arquivos LATTICE inserem a marcação `Chloros:PixelScale=32768` no XMP; os arquivos doSurvey3
usam 65535 (e não contêm tags `Chloros:*`). Leia a tag e divida por ela, em vez de assumir um valor constante. Detalhes e o único caso extremo deliberado sem escala estão na [Referência doCLI
](reference/cli-reference.md).
{% endhint %}

**O processamento sempre começa a partir de `raw`.** Produtos derivados (exportações de debayering/radiação/refletância) nunca são realimentados pelo pipeline — reimportá-los e processá-los aplicaria duas vezes os cálculos de calibração; portanto,Chloros
os ignora e informa isso. `--input-level` é a saída de emergência deliberada para quando você realmente precisa forçar um ponto de entrada.

***

## Quando uma execução falha

A partir da versão 1.2.0, o `process` apresenta uma falha evidente, em vez de “ser bem-sucedido” sem exibir nenhum resultado:

* Uma execução que **solicitou produtos, mas não gravou nenhum**— apenas `project.json` e `calibration_data.json` — exibe `Processing finished but wrote no image products.` e**encerra com valor diferente de zero**, de modo que os scripts possam detectá-lo. As causas mais comuns: a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou todos os produtos solicitados eram inaplicáveis para essas câmeras (por exemplo, solicitar radianância/refletância de câmeras que registram apenas oRGB
).
* Uma **execução deliberada apenas com metadados** (todos os produtos desativados, sem `--indices`) ainda é considerada bem-sucedida — uma saída de imagem vazia é o resultado correto nesse caso.
* Execute novamente com o `--verbose` e verifique o log do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explicam as omissões por câmera.

Códigos de saída: `0` sucesso · `1` falha genérica · `2` erro de argumento · `130` interrompido por Ctrl+C.

***

## Índices de vegetação

Execute o `--indices` com um ou mais nomes de predefinições; cada índice é salvo em sua própria pasta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI NDRE GNDVI
```

Os 22 nomes predefinidos que o `process --indices` aceita:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

{% hint style="warning" %}
**Existem três listas de índices — não as confunda.**O menu suspenso “Configurações do projeto” da GUI possui 27 fórmulas (adiciona `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — essas cinco são exclusivas da GUI e**não** são válidas para `--indices`). O comando `lattice index --preset` (ao vivo/offline) utiliza sua própria lista separada de 22 predefinições. As fórmulas e os cálculos de banda estão documentados em [Fórmulas de Índices Multiespectrais](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: uma introdução rápida

A família `daq pool-*` controla os sensores espectrais DAQ dMAPIR
(DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) por meio do pool persistente do backend — a GUI,CLI
eSDK
compartilham um único identificador ativo. **`pool-*` é o caminho DAQ compatível noCLI
fornecido**; outros subcomandos `daq` que você possa ver referenciados são uma superfície exclusiva para uso interno doMAPIR
e saem com um erro explícito que o direciona para `pool-*`.

```bash
# 1. Open a pooled session (pick the line matching your sensor)
chloros-cli daq pool-connect                              # smart-detect
chloros-cli daq pool-connect --port COM3                  # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF      # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local   # DAQ-E by hostname (reliable)

# 2. List pooled sensors and their ids
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 3. Read the latest calibrated spectrum (W/m²/nm)
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 4. Record a calibrated .daq file for 60 s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 5. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

`pool-record` sem `--duration` é executado até `pool-record --stop`; o diretório de saída padrão é `~/Documents/DAQ Live View/` **na máquina do backend**. O perfil de correção de capacitância é escolhido no momento da conexão (`--cap-id`, padrão do backend `sunshine_cosine`) e pode ser trocado em tempo real com `pool-set-cap` — os perfis de limite e a faixa calibrada do sensor são abordados nos capítulos sobre DAQ deste manual.

{% hint style="warning" %}
**DAQ-E em um host com várias placas de rede:** a primeira detecção automática do `pool-connect --eth` após a inicialização pode falhar, mesmo com um sensor em bom estado. O `--eth-host <ip-or-hostname>` é a forma confiável — utilize-o sempre que a detecção não for bem-sucedida.
{% endhint %}

***

## Câmeras LATTICE, PTP e automação de projetos

A família `lattice` (mais de 45 subcomandos) abrange todo o processo de trabalho com câmeras LATTICE: detecção, capturas únicas, matrizes sincronizadas persistentes com o fluxo de conexão “smart-prep” da GUI, visualização ao vivo no navegador, alinhamento, cálculos de índice e diagnósticos da placa de rede do host. Uma amostra:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Juntamente com isso: o `chloros-cli time-sync` gera relatórios sobre o grandmaster PTP executado pelo hostChloros
(as câmeras LATTICE e os sensores DAQ-E funcionam como escravos dele para carimbos de data/hora entre dispositivos), e o `chloros-cli project` abre um projeto salvo doChloros
e controla suas câmeras, matrizes e sensores de forma autônoma — incluindo receitas de captura em YAML programadas por script.

Essas três famílias (`lattice`, `project`, `daq pool-*`) também são as únicas que reconhecem o comando `CHLOROS_BACKEND_URL` para controlar um backend **remoto**; os comandos principais sempre têm como alvo a máquina local.

Os tutoriais completos estão nos capítulos sobre o LATTICE deste manual; todos os sinalizadores estão na [Referência doCLI
](reference/cli-reference.md).

***

## Solução de problemas: os 5 principais

| Sintoma | Solução |
| --- | --- |
| O `Login required` ou uma tarefa agendada trava no prompt do `Email:` | Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez nesta máquina — comandos sem uma sessão em cache são executados de forma interativa, em vez de falharem imediatamente. |
| `backend unreachable` | Inicie o aplicativo de desktopChloros
ou execute o binário de back-end diretamente (`chloros-backend`). Se você direcionar o `lattice`/`project`/`daq pool-*` para um backend remoto, verifique o `CHLOROS_BACKEND_URL`. |
| Conexão da matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | O anel de recepção da placa de rede do host foi redefinido para os padrões — a principal causa de um equipamento que antes funcionava se recusar a se conectar, geralmente após uma atualização do driver da placa de rede. Execute o `chloros-cli lattice network --fix` em um terminal com **privilégios elevados** (ou defina o `ReceiveBufferLen=256`, `PendingReceives=64`); consulte a seção *Configuração e ajuste da placa de rede do host* da documentação de referência. |
| O subcomando `daq` é encerrado: “requer o pacote DAQ completo…” | Esperado em compilações fornecidas — oCLI
compilado fornece apenas a família `daq pool-*`, que abrange conexão, transmissão, gravação e seleção de cap. Use o `pool-*` (ou o `chloros_sdk.connect_daq_sensor()` disponível emPython
). |
| O Jetson exibe um aviso sobre swap antes de pastas grandes | Adicione swap baseado em arquivo — oCLI
exibe os comandos exatos `fallocate`/`swapon` a serem executados. |

***

## Obtendo ajuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todos os sinalizadores, todos os subcomandos:** [CLI
Referência](reference/cli-reference.md)
* **Equivalente noPython
:** [Python
SDK
](api-python-sdk.md) e a [SDK
Referência](reference/sdk-reference.md)
* **Suporte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
