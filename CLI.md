# CLI: Linha de comando

> **Referência completa:**[Referência da CLI](reference/cli-reference.md) documenta**todos os parâmetros de cada subcomando** e é otimizada para assistentes de IA — cole sua URL no seu assistente e peça um comando que funcione: `https://mapir.gitbook.io/chloros/reference/cli-reference`
>
> **Dica para ferramentas de IA:** qualquer página deste manual está disponível no formato Markdown bruto ao acrescentar `.md` à sua URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`), e `https://mapir.gitbook.io/chloros/llms.txt` indexa o manual inteiro para uso por modelos de linguagem grande (LLM).

> **Quer que seu assistente execute o próprio Chloros?** `chloros-mcp`, um servidor MCP instalado com o Python SDK, funciona com o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP: consulte [Usando o Chloros com assistentes de IA](ai-assistants.md).

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: banner shows CLI 1.1.0; reshoot the CLI welcome/banner output on the 1.3.0 build so the version line reads "Chloros CLI 1.3.0" -->


## O que é a CLI


O `chloros-cli` é a interface de linha de comando do mesmo mecanismo de processamento que o aplicativo de desktop Chloros
utiliza. Trata-se de um cliente HTTP
leve sobre o backend do Chloros
(um servidor local no `127.0.0.1:5000`) — a maioria dos comandos inicia o backend automaticamente, portanto, uma única chamada ao `chloros-cli process …` é tudo o que um script precisa. O backend permanece em execução para o próximo comando: ele é interrompido no comando `chloros-cli logout` (a menos que o aplicativo esteja aberto ou que uma execução ou compilação esteja em andamento), no comando `chloros-cli backend stop`, ou automaticamente após 30 minutos sem nenhum cliente, execução ou dispositivo conectado. O comando `chloros-cli backend status` mostra isso.

Ele roda no **Windows
10/11 (x64)**e no**Linux
(x86_64 e NVIDIA Jetson arm64 no JetPack 6)**, em qualquer terminal, sem necessidade de GUI. Verifique sua instalação com:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

As famílias de comandos, em resumo:

* **Processamento e conta** — `process`, `login`, `logout`, `status`, `export-status`, `language` (38 idiomas — consulte [Idiomas suportados](supported-languages.md)), `set-project-folder` / `get-project-folder` / `reset-project-folder`, `selftest`, `update` (Somente Linux
/Jetson)
* **Hardware em funcionamento** — `lattice` (controle de câmera LATTICE, mais de 45 subcomandos), `daq pool-*` (sensores de luz DAQ), `time-sync` (PTP)
* **Automação** — `project` (executar um projeto Chloros
salvo no modo headless, incluindo receitas de captura em YAML)

Opções globais que vale a pena conhecer: `--port N` (porta do backend, padrão `5000`), `-v/--verbose`, `--restart` (reiniciar forçado o backend), `--backend-exe PATH`. Consulte a [Referência da CLI
](reference/cli-reference.md) para obter a lista completa.

***

## Instalação

A CLI
**vem incluída no instalador do Chloros** em todas as plataformas — não há download separado da CLI
. Baixe o instalador na página [Download](download.md).

### Windows


O instalador coloca a CLI
em:

```

C:\Program Files\Chloros\cli\chloros-cli.exe
```

e adiciona essa pasta ao seu sistema `PATH` uma vez (uma atualização também remove entradas duplicadas deixadas por instaladores anteriores) — **abra um novo terminal**após a instalação para que o `PATH` atualizado seja reconhecido. O instalador também coloca scripts de inicialização (`Chloros_CLI.bat` / `Chloros_CLI.ps1`) na raiz da instalação, além de um**Chloros
CLI
** no menu Iniciar, cada um dos quais abre um terminal com o `chloros-cli` pronto para uso.

### Linux


Instale o `.deb` para sua arquitetura com o `apt` (não o `dpkg -i`, que ignora os requisitos recomendados do SDK
do Python
):

```bash
# Linux x86_64
sudo apt install ./chloros_1.3.0_amd64.deb

# NVIDIA Jetson (arm64, JetPack 6)
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
```

Isso instala o `chloros-cli` até o `/usr/bin/chloros-cli` (já presente no `PATH`), o backend e os programas de CLI
descompactados em `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/` (o `/usr/lib/chloros/chloros-backend` é um iniciador que inicia o backend), o runtime do Arena SDK
necessário ao backend para câmeras LATTICE e o Python
SDK
para o sistema `python3`. Consulte [Instalação no Linux
](linux/linux-installation.md) para obter detalhes.

### Verificar

```bash
chloros-cli --version    # "Chloros CLI 1.3.0"
chloros-cli selftest     # 7-step diagnostic: backend, API, GPU/CUDA, denoiser models
chloros-cli status       # license tier + logged-in user
```

***

## Login e licenciamento

O acesso à CLI
(e ao Python
SDK
) requer um **plano pago do Chloros
+**— qualquer plano pago oferece esse acesso; o plano gratuito não. A restrição mínima é aplicada**no lado do servidor** pelo backend, não pelo binário da CLI
: uma chamada sem login é rejeitada com o código `401 AUTH_REQUIRED`, e uma chamada com login no plano gratuito com o código `403 PLAN_UPGRADE_REQUIRED`, independentemente de ela vir do `chloros-cli`, do SDK
ou de um cliente HTTP
personalizado. Faça a atualização em [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing).

Faça login **uma vez por máquina**:

```bash
chloros-cli login user@example.com 'YourPassword'
chloros-cli status
```

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: login success output predates 1.2.0; reshoot `chloros-cli login` followed by `chloros-cli status` on the 1.3.0 build showing the license tier line -->


{% hint style="warning" %}
**Senhas com caracteres especiais**(`$`, `!`, spaces): wrap the password in**single quotes**, as shown above. In PowerShell double quotes, `$$` é corrompida pelo shell — o CLI
detecta isso em um erro 401 e tenta novamente automaticamente, mas aspas simples evitam o problema por completo).
{% endhint %}

A sessão é armazenada em cache em `~/.chloros/user_session.json` e continua funcionando offline durante o período de carência do plano (30 dias para planos mensais; até o vencimento para planos anuais). O `chloros-cli status` funciona mesmo sem um plano pago, portanto, o motivo da recusa está sempre visível.

{% hint style="danger" %}
**Agendando tarefas sem interface gráfica? Faça login primeiro.**Um comando de geração de backend (`process`, `status`, `export-status`, …) executado**sem sessão em cache**não apresenta falha imediata — ele entra em um prompt interativo `Email:` / `Password:` no stdin. Uma tarefa cron automatizada ou etapa de CI, portanto,**ficará travada aguardando entrada**. Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez na máquina antes de agendar qualquer coisa.
{% endhint %}

***

## Sua primeira execução do processamento

Aponte o `process` para uma pasta de capturas — ele detecta automaticamente o Survey3
(`.raw` + `.jpg`), LATTICE (`.tif`/`.tiff`), `.dng` ou uma combinação:

```bash
chloros-cli process "C:\Images\flight_001"          # Windows
chloros-cli process ~/images/flight_001              # Linux
```

Fluxos de progresso em tempo real por thread do pipeline (Detecção, Análise, Processamento, Exportação), e uma execução bem-sucedida termina relatando quantos produtos de imagem foram gerados: `Image products written: N`, seguido por `(plus M clamp masks)` quando a execução também gravou [máscaras de limitação](output-image-formats.md#clamp-masks) (elas não são contadas como produtos).

<!-- SCREENSHOT-NEEDED: terminal capture of a `chloros-cli process` run on a LATTICE captures folder completing successfully — per-thread progress lines visible and the final "Image products written: N (plus M clamp masks)" summary line -->


Se um backend do Chloros
já estiver em execução (por exemplo, o do aplicativo para desktop), o `process` o utiliza e abre seu projeto nele. Ele recusa a operação enquanto o aplicativo estiver aberto em um projeto que ele mesmo abriu: execute a operação a partir do aplicativo ou feche esse projeto. Se uma execução anterior ainda estiver sendo concluída, ele aguarda até 15 segundos antes de recusar. O `--restart` força a inicialização de um backend novo. No Linux
, um backend executado como serviço do systemd nunca é reiniciado: o `process` o utiliza como está e, nos casos em que, de outra forma, o reiniciaria (`--restart` ou uma importação que não encontrou nada), ele exibe o código `sudo systemctl restart <unit>` e encerra a execução com o código `1`.

### Onde as saídas são armazenadas

O `process` grava em uma **pasta de projeto**, não na sua pasta de entrada:

* Sem `-o`: o projeto é criado na sua pasta de projetos padrão (compartilhada com a interface gráfica; gerencie-a com `get-project-folder` / `set-project-folder`; quando nenhuma estiver definida, o `Documents/MAPIR/Chloros Projects` na sua pasta de usuário), nomeado por `-n/--project-name` ou por um carimbo de data/hora (`YYYYMMDD_HHMMSS`) quando omitido.
* Com o `-o PATH`: essa pasta **é** a pasta do projeto. Se ela já contiver um `project.json`, um `_1`/`_2`… com sufixo é criado em vez de sobrescrever. Um `PATH` relativo é considerado em relação à pasta a partir da qual você executa o `chloros-cli`.

{% hint style="info" %}
**Caminhos relativos (alterados na versão 1.3.0).** Cada caminho que o backend abre — `-o`/`--output` em qualquer comando (incluindo o `output` padrão dos comandos de captura), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask` — é resolvido com base na pasta a partir da qual você executa o `chloros-cli`; portanto, o `-o captures/` grava no `<current folder>/captures/`. Na versão 1.2.0, o backend os resolvia em sua própria pasta de instalação, o que resultava na falha “Acesso negado”. Em relação a um backend remoto (`CHLOROS_BACKEND_URL`), os caminhos são enviados exatamente como digitados e resolvidos naquela máquina.
{% endhint %}

{% hint style="warning" %}
**Windows
: mantenha a pasta do projeto curta, a menos que os caminhos longos estejam ativados.** Com os caminhos longos do Windows
desativados (configuração padrão do Windows
), nenhum caminho de arquivo pode ter mais de 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. `process` verifica isso antes de iniciar: uma pasta de projeto muito profunda é rejeitada com a mensagem **“A pasta de projeto é muito profunda para este computador”**, a pasta de projeto mais longa que cabe e como ativar os caminhos longos — nada é processado. Use um `-o` (ou `set-project-folder`) mais curto ou ative os caminhos longos do Windows
(a opção **Ativar caminhos longos do Windows** do instalador, marcada por padrão) e reinicie o Chloros
.
{% endhint %}

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

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE (correspondendo ao EXIF da captura `Model`) e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para o Survey3
. A pasta de formato segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para `TIFF (32-bit, Percent)`.

Para uma câmera multiespectral M3C, uma pasta `clamp_masks/` também pode aparecer em uma pasta de formato, ao lado das pastas de produtos (por exemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/` e `tiff32/clamp_masks/Radiance_Images/`). Ela contém as máscaras por pixel dos produtos de radianância e refletância nos quais a separação de bandas limitou um valor negativo a zero — consulte [Máscaras de limitação](output-image-formats.md#clamp-masks). Nenhuma pasta de produto contém uma máscara.

{% hint style="info" %}
**Todo produto exportado mantém o nome do arquivo ORIGINAL.**Uma exportação de radiação de `capture_..._raw.tif` ainda se chama `capture_..._raw.tif` — ela apenas fica na pasta `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do arquivo**; portanto, use uma expressão glob para o diretório, não para o sufixo `*radiance*`.
{% endhint %}

### As opções que você realmente usará

| Opção | Padrão | O que faz |
| --- | --- | --- |
| `-o, --output PATH` | pasta padrão do projeto | Localização da pasta do projeto (veja acima). |
| `-n, --project-name NAME` | carimbo de data/hora | Nome do projeto. |
| `--format FMT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. O Chloros
não aumenta a resolução: em todas as imagens de 8 bits, um formato com maior profundidade é gravado como `TIFF (8-bit)`, e isso é indicado uma vez. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação a serem exportados (consulte [Índices de vegetação](#vegetation-indices)). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` = debayer neural, mais lento, qualidade máxima (Chloros
+, requer uma GPU CUDA: sem ela, a execução usa `standard`; rejeitado para capturas LATTICE, que sempre recebem `standard`). |
| `--vignette / --no-vignette` | ativado | Correção de vinheta. |
| `--reflectance / --no-reflectance` | ativado | Calibração de refletância; para o LATTICE, esta opção também ativa ou desativa o produto de refletância. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Força o ponto de entrada do pipeline para arquivos TIFF do LATTICE. |

Para todo o restante — ajuste de detecção de alvo, PPK, pinos de exposição, sinalizadores de alinhamento de matriz — consulte a [seção `process` da Referência da CLI
](reference/cli-reference.md).

***

## Escolhendo o que exportar (produtos LATTICE)

O processamento LATTICE se ramifica para **todos os produtos aplicáveis em uma única passagem**. Quatro opções por produto estão todas**ativadas por padrão**; use o formulário `--no-` para desativar uma delas:

| Botão | Produto |
| --- | --- |
| `--debayered` | Demosaico linear → `Debayered_Images/`: o deslocamento de escuro do pacote de calibração removido, seguido de um demosaico sensível a bordas, sem correção de linearidade, campo plano ou separação de canais — o mesmo arquivo que uma captura salva em seu nível `debayered`. Mantido na orientação nativa do sensor (registrada no XMP). |
| `--preview` | Visualização na tela (equilíbrio de branco + gama; expansão em cores falsas para multiespectral) → `Preview_Images/` |
| `--radiance` | radiação em float32, W/m²/sr/nm → `Radiance_Images/` (sempre `tiff32/`) |
| `--reflectance` | uint16 refletância, pronta para Pix4D → `Reflectance_Calibrated_Images/` |

As câmeras principais RGB
sempre emitem apenas dados pós-debayering + pré-visualização — a radianância/refletância por banda não tem significado para um sensor de banda larga, portanto, essas opções não têm efeito para elas. O Survey3
`.raw` ignora as opções e segue o caminho padrão de refletância/alvo.

```bash
# Radiance only — no DAQ downwelling needed
chloros-cli process ~/captures/lattice_flight --no-debayered --no-preview --no-reflectance
```

**`--reflectance-source {auto,target,daq}`** (padrão `auto`) seleciona a referência de refletância: `auto` define um [alvo de calibração](calibration-targets.md) dentro do quadro que passou no controle de qualidade (QA) como referência absoluta e recorre à divisão de luz descendente do sensor de luz do DAQ (ρ = π·L/E) quando não há alvo presente (um alvo que forneça um resultado que passe no controle de qualidade com um sinalizador compete com o DAQ na pontuação medida); `target` é estrito (sem substituição pelo DAQ); `daq` tem autoridade sobre o DAQ. Varreduras de alvo medidas por unidade podem ser fornecidas com `--target-reflectance-dir`.

{% hint style="info" %}
**Leitura de pixels de refletância:**o DN que significa ρ = 1,0 é**por fonte** — os arquivos LATTICE inserem a tag `Chloros:PixelScale=32768` no XMP; os arquivos Survey3
usam 65535 e não contêm a tag `Chloros:PixelScale` (eles contêm outras tags de processamento `Chloros:*`). Leia a tag e divida por ela, em vez de assumir um valor constante. Detalhes e o único caso extremo deliberado sem escala estão na [Referência da CLI
](reference/cli-reference.md).
{% endhint %}

**O processamento sempre começa a partir de `raw`.** Produtos derivados (exportações de debayering/radiação/refletância) nunca são realimentados pelo pipeline — reimportá-los e processá-los aplicaria duas vezes os cálculos de calibração; por isso, o Chloros
os ignora e avisa sobre isso. `--input-level` é a saída de emergência deliberada para quando você realmente precisa forçar um ponto de entrada.

***

## Quando uma execução falha

A partir da versão 1.2.0, o `process` apresenta uma falha evidente, em vez de “ser bem-sucedido” sem exibir nenhum resultado:

* Uma execução que **solicitou produtos, mas não gravou nenhum**— apenas `project.json` e `calibration_data.json` — exibe `Processing finished but wrote no image products.` e**encerra com valor diferente de zero**, de modo que os scripts possam detectá-lo. As causas habituais: a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou todos os produtos solicitados eram inaplicáveis para essas câmeras (por exemplo, solicitar radianância/refletância de câmeras apenas RGB
).
* Uma **execução deliberada apenas com metadados** (todos os produtos desativados, sem `--indices`) ainda é considerada bem-sucedida — uma saída de imagem vazia é o resultado correto nesse caso.
* Execute novamente com o `--verbose` e verifique o log do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`, que explicam as omissões por câmera.
* No Windows
, com caminhos longos desativados, uma pasta de projeto muito profunda para o limite de 260 caracteres no caminho é rejeitada **antes** do início do processamento, sendo utilizada a pasta de projeto mais longa que couber (consulte [Onde os arquivos de saída são salvos](#where-the-outputs-land)).
* Os comandos de captura (`lattice capture`, `lattice array-capture` e as outras famílias de captura) listam todos os arquivos que não conseguiram gravar e saem com o código `1`; as capturas repetidas são interrompidas quando a unidade fica cheia.

Códigos de saída: `0` sucesso · `1` falha genérica · `2` erro de argumento · `130` interrompido por Ctrl+C.

***

## Índices de vegetação

Execute o `--indices` com um ou mais nomes predefinidos; cada índice é salvo em sua própria pasta `<INDEX>_Index_Images/`:

```bash
chloros-cli process ~/images/flight_001 --indices NDVI GNDVI OSAVI
```

Os 22 nomes predefinidos que o `process --indices` aceita:

`NDVI` `GNDVI` `NDRE` `OSAVI` `SAVI` `MSAVI2` `EVI` `MSR` `TDVI` `LAI` `GCI` `GRVI` `GSAVI` `GOSAVI` `NLI` `MNLI` `RDVI` `WDRVI` `CVI` `ENDVI` `GLI` `VARI`

Um nome que ainda não consta na lista do projeto é adicionado a ele. Um nome que não seja uma predefinição, ou que as faixas das câmeras não possam formar, é ignorado com um aviso indicando o nome. Se a CLI
não conseguir ler a lista de índices atual do projeto, ela mantém a lista inalterada e exibe um aviso, em vez de substituí-la.

{% hint style="warning" %}
**Existem três listas de índices — não as confunda.**O menu suspenso “Configurações do projeto” da GUI possui 27 fórmulas (adiciona `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` — essas cinco são exclusivas da GUI e**não** são válidas para `--indices`). O comando `lattice index --preset` (ao vivo/offline) utiliza sua própria lista separada de 22 predefinições. As fórmulas e os cálculos de bandas estão documentados em [Fórmulas de Índices Multiespectrais](project-settings/multispectral-index-formulas.md).
{% endhint %}

***

## Sensores de luz DAQ: uma introdução rápida

A família `daq pool-*` controla os sensores espectrais DAQ da MAPIR
(DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet) por meio do pool persistente do backend — a GUI, a CLI
e o SDK
compartilham um único identificador ativo. **O `pool-*` é o caminho DAQ suportado na CLI
fornecida**; outros subcomandos `daq` que você possa ver mencionados são uma superfície exclusiva para uso interno da MAPIR
e são encerrados com um erro explícito que o direciona para o `pool-*`.

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

`pool-record` sem `--duration` é executado até `pool-record --stop`; o diretório de saída padrão é `~/Documents/DAQ Live View/` **na máquina do backend**. O perfil de correção de cap é escolhido no momento da conexão (`--cap-id`, padrão do backend `sunshine_cosine`) e pode ser trocado em tempo real com `pool-set-cap` — os perfis de cap e a faixa calibrada do sensor são abordados nos capítulos sobre DAQ deste manual.

{% hint style="warning" %}
**DAQ-E em um host com várias placas de rede:** a primeira descoberta automática do `pool-connect --eth` após a inicialização pode falhar, mesmo com um sensor em bom estado. O `--eth-host <ip-or-hostname>` é a opção confiável — utilize-o sempre que a descoberta não for bem-sucedida.
{% endhint %}

***

## Câmeras LATTICE, PTP e automação de projetos

A família `lattice` (mais de 45 subcomandos) abrange todo o processo de trabalho com câmeras LATTICE: descoberta, capturas únicas, matrizes sincronizadas persistentes com o fluxo de conexão “smart-prep” da GUI, visualização ao vivo no navegador, alinhamento, cálculos de índice e diagnósticos da placa de rede do host. Uma amostra:

```bash
chloros-cli lattice info                                          # discover cameras
chloros-cli lattice capture -o output/                            # one frame, all export types
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4       # persistent synced array
chloros-cli lattice array-capture --processing reflectance -o out/
```

Juntamente com ela: o `chloros-cli time-sync` gera relatórios sobre o grandmaster PTP executado pelo host Chloros
(as câmeras LATTICE e os sensores DAQ-E funcionam como escravos dele para carimbos de data/hora entre dispositivos), e o `chloros-cli project` abre um projeto Chloros
salvo e controla suas câmeras, matrizes e sensores sem interface gráfica — incluindo receitas de captura YAML programadas.

Essas três famílias (`lattice`, `project`, `daq pool-*`) também são as únicas que reconhecem o `CHLOROS_BACKEND_URL` para controlar um backend **remoto**; os comandos principais sempre têm como alvo a máquina local.

Os tutoriais completos estão nos capítulos sobre o LATTICE deste manual; todos os parâmetros estão na [Referência da CLI
](reference/cli-reference.md).

***

## Solução de problemas: os 5 principais

| Sintoma | Solução |
| --- | --- |
| `Login required` ou uma tarefa agendada trava no prompt `Email:` | Execute o `chloros-cli login EMAIL 'PASSWORD'` uma vez nesta máquina — comandos sem uma sessão em cache são executados de forma interativa, em vez de falharem imediatamente. |
| `backend unreachable` | Inicie o aplicativo de desktop Chloros
ou inicie o backend manualmente: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` no Windows
(na pasta `backend` da sua pasta de instalação; ela não está no `PATH`), `chloros-backend` no Linux
. O `chloros-cli backend status` mostra se algum está em execução; `lattice`/`project`/`daq pool-*` nunca iniciam um. Se você os direcionar para um backend remoto, verifique o `CHLOROS_BACKEND_URL`. |
| Conexão da matriz bloqueada: `FRAMES WILL DROP` / `Reduce ROI to enable` | O anel de recepção da placa de rede do host foi redefinido para os valores padrão — a principal causa de um equipamento que antes funcionava se recusar a se conectar, geralmente após uma atualização do driver da placa de rede. Execute o comando `chloros-cli lattice network --fix` em um terminal com **privilégios elevados** (ou defina `ReceiveBufferLen=256`, `PendingReceives=64`); consulte a seção *Configuração e ajuste da placa de rede do host* da referência. |
| O subcomando `daq` é encerrado: “requer o pacote DAQ completo…” | Use os comandos `daq pool-*`, que controlam o mesmo sensor por meio do backend: conexão, transmissão, gravação, seleção de cap e, para um DAQ-E, suas configurações de rede, reinicialização e atualizações de firmware (ou `chloros_sdk.connect_daq_sensor()` em Python
). |
| O Jetson exibe um aviso de swap antes de pastas grandes | Adicione swap baseado em arquivo — a CLI
exibe os comandos exatos `fallocate`/`swapon` a serem executados. |

***

## Obtendo ajuda

```bash
chloros-cli --help              # top-level help
chloros-cli process --help      # per-command help
chloros-cli lattice --help
chloros-cli daq --help          # lists the pool-* subcommands
```

* **Todos os sinalizadores, todos os subcomandos:** [Referência da CLI
](reference/cli-reference.md)
* **Equivalente em Python
:** [Python
SDK
](api-python-sdk.md) e a [Referência do SDK
](reference/sdk-reference.md)
* **Suporte:** info@mapir.camera · [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
