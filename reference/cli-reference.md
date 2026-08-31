# Chloros CLI Referência

**Versão:**

1.2.0**Gerado em:**29/07/2026 19:19 ·**Revisado:** 30/08/2026**Público-alvo:** Otimizado para uso por LLM; legível por humanos.**Escopo:** Todos os subcomandos do `chloros-cli` voltados para o usuário, com opções e exemplos que podem ser copiados e colados.

Este documento é a referência completa para o comando `chloros-cli`de linha de comando que vem com o MAPIR Chloros. Ele é intencionalmente exaustivo para que um LLM (ou humano) possa compor qualquer fluxo de trabalho suportado a partir das listas abaixo sem precisar inspecionar o código-fonte.

Se você precisar apenas dos destaques, vá para:
- [Guia rápido de cinco minutos](#five-minute-quickstart)
- [Fluxo de trabalho de primeira conexão da câmera LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxo de trabalho de primeira conexão do sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess)

---

## Convenções

- Todos os comandos têm o prefixo `chloros-cli`. No Windows, o binário é `chloros-cli.exe`; no Linux/Jetson , é `chloros-cli`.
- Argumentos opcionais são apresentados como `--flag`. Argumentos posicionais obrigatórios são apresentados sem colchetes.
- Quando um valor padrão é fornecido, omitir o sinalizador faz com que esse valor seja usado.
- O CLI é um cliente leve HTTP sobre o backend Chloros (servidor Flask no `127.0.0.1:5000`). O backend é iniciado automaticamente pela maioria dos comandos. O `CHLOROS_BACKEND_URL=<url>` direciona as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para um backend remoto — os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixam deliberadamente o `http://127.0.0.1:<port>` e o ignoram (o literal IPv4 evita a penalidade de ~2 s por solicitação do `localhost`→`::1`, com penalidade de cerca de 2 s por solicitação). Consulte [Variáveis de ambiente](#environment-variáveis).
- É necessário um login com a conta Chloros+ para todas as chamadas SDK/CLI (execute o `chloros-cli login` uma vez por máquina; armazenado em cache no `~/.chloros/`).
- Os exemplos utilizam caminhos do Linux; no Windows, substitua o `/home/user/...` pelo `C:/Users/.../...`.

---

## Sinopse de nível superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opções globais

| Sinalizador | Descrição |
| --- | --- |
| `--backend-exe PATH` | Substitui o executável do backend detectado automaticamente. |
| `--port N` | Porta do backend HTTP (padrão: `5000`). |
| `-v, --verbose` | Ativar saída detalhada. |
| `--restart` | Reiniciar à força o backend (encerra qualquer instância do `backend_server.py` em execução). |
| `--version` | Exibir versão (`Chloros CLI 1.2.0`). |
| `--help` | Exibir ajuda de nível superior. |

### Índice de comandos

| Comando | Finalidade |
| --- | --- |
| [`process`](#chloros-cli-process) | Processar uma pasta de capturas Survey3 ou LATTICE de ponta a ponta. |
| [`login`](#chloros-cli-login) | Autenticar esta máquina com uma conta Chloros+. |
| [`logout`](#chloros-cli-logout) | Limpar credenciais armazenadas em cache. |
| [`status`](#chloros-cli-status) | Mostrar o status atual da licença/autenticação. |
| [`export-status`](#chloros-cli-export-status) | Exibir o andamento da exportação do Thread-4 em tempo real durante uma execução do `process`. |
| [`language`](#chloros-cli-language) | Definir ou listar o idioma de exibição do CLI (38 idiomas suportados). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Pasta padrão do projeto (compartilhada com a GUI). |
| [`update`](#chloros-cli-update) | Verificar e instalar atualizações do CLI (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnósticos do sistema + testes de funcionamento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Status/controle do grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Controle e captura da câmera LATTICE (mais de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Controle do sensor espectral DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir e executar um projeto Chloros salvo (câmeras + DAQs). |

---

## Instalação

O `chloros-cli` vem incluído no instalador para desktop do Chloros em todas as plataformas compatíveis — não há um download separado do CLI. A instalação do pacote da plataforma adiciona o `chloros-cli` ao seu `PATH`, juntamente com o aplicativo para desktop e o binário de back-end que ele aciona.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> O instalador também inclui scripts de inicialização práticos (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abrem um shell CLI pronto para uso; eles são abordados no [Guia do Usuário do CLI](../CLI.md) e não serão repetidos aqui.

### Windows (.exe)

1. Baixe o instalador Windows na página de downloads.
2. Execute o `Chloros-Setup-x.y.z.exe` e siga as instruções do assistente. O caminho padrão de instalação é `C:\Program Files\Chloros\` (o CLI fica em `C:\Program Files\Chloros\cli\`, que o instalador adiciona ao PATH).
3. Abra um novo terminal (`cmd.exe`, PowerShell ou Windows Terminal) para que o `PATH` atualizado seja reconhecido.

```powershell
chloros-cli --version
```

O instalador adiciona automaticamente o `chloros-cli.exe` ao seu sistema `PATH` e inclui o runtime do Arena SDK necessário para câmeras LATTICE.

### Linux amd64 (.deb)

Para estações de trabalho x86_64 com Ubuntu 22.04 LTS ou mais recente / baseadas em Debian.

> **O Ubuntu 20.04 não é compatível.** A lista de dependências do pacote é derivada do
> que o backend realmente vincula, e isso inclui o `libc6 (>= 2.34)`;
> o Focal vem com a glibc 2.31. O `apt` recusa a instalação em vez de permitir que ela falhe em
> tempo de execução.

```bash
sudo dpkg -i chloros-amd64.deb
sudo apt-get install -f         # only if dpkg reports missing dependencies
chloros-cli --version
```

O pacote .deb instala:
- `chloros-cli` para `/usr/bin/chloros-cli`
- O backend compilado para `/usr/lib/chloros/chloros-backend`
- O runtime do Arena SDK (para câmeras LATTICE)
- Modelos de redução de ruído, pacotes de calibração e configuração do canal de atualização

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Mesmo layout que o .deb para amd64, com uma compilação CUDA otimizada para Jetson Orin / Orin NX / Orin Nano.

### Autenticação única por máquina

Todas as plataformas exigem um login único no Chloros+ antes que as chamadas SDK/CLI funcionem:

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache no `~/.chloros/user_session.json`.

### Verificar a instalação

```bash
chloros-cli --version           # prints "Chloros CLI 1.2.0"
chloros-cli selftest            # full 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

> **É necessária uma assinatura do Chloros+.**O CLI requer um plano ativo do Chloros+.**Copper**é o nível básico do Chloros+ — todos os níveis pagos do Chloros+ têm acesso ao CLI/SDK; apenas o nível gratuito**Iron** não tem. (Mapa de IDs de planos: `0`=Iron/grátis, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Faça o upgrade em [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Esse limite mínimo é aplicado pelo backend, não apenas pelo CLI: uma solicitação marcada como SDK/CLIsem um plano pago é rejeitada com o código `403 PLAN_UPGRADE_REQUIRED`, independentemente de vir do `chloros-cli`, do Python SDKou de um cliente personalizado HTTP. Um usuário desconectado recebe o código `401 AUTH_REQUIRED`. O acesso funciona offline durante o período de carência do plano (30 dias para planos mensais e até o vencimento para planos anuais) e é interrompido quando esse período expira; o `chloros-cli status` continua funcionando, de modo que o motivo fica visível (é a única rota SDK/CLI isenta do filtro de níveis — `GET /api/license-status`).

---

## Guia de Início Rápido em Cinco Minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI NDRE GNDVI

# 3. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 4. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 5. Read a spectrum from a connected DAQ-U
chloros-cli daq pool-connect --port COM3
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

---

## `chloros-cli process`

Processe uma pasta de imagens por meio do pipeline completo Chloros (detecção de alvo → calibração → vinheta → refletância → exportação de índice).

### Sinopse

```
chloros-cli process INPUT [OPTIONS]
```

### Argumentos posicionais

| Argumento | Descrição |
| --- | --- |
| `INPUT` | Caminho para a pasta de entrada contendo os arquivos `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou os arquivos `.dng`. |

### Opções comuns

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `-o, --output PATH` | uma nova pasta com data e hora no caminho padrão do projeto (`~/Chloros Projects`, a menos que configurado) | Pasta do projeto a ser criada ou reutilizada. Se a pasta já contiver um `project.json`, um pasta irmã com o nome `_1`/`_2` será criada em vez de sobrescrever a existente. |
| `-n, --project-name NAME` | automático (carimbo de data/hora) | Nome do projeto. |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utiliza um Chloros+ modelo de rede neural; mais lento, mas com maior qualidade. |
| `--vignette / --no-vignette` | `--vignette` | Correção de vinheta. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibração de refletância (usa o alvo do painel, se encontrado; calibração NIST por série para LATTICE). Para o LATTICE multiespectral, isso também funciona como o botão de alternância do **produto** de refletância — consulte [Botões de alternância de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desativado | Aplicar correções PPK GNSS a partir de arquivos sidecar. |
| `--exposure-pin-1 MODEL` | desativado | Fixar o modelo “pin-1” do equipamento de câmera dupla Survey3 do equipamento de câmera dupla. |
| `--exposure-pin-2 MODEL` | desativado | Fixar o modelo “pin-2”. |
| `--recal-interval SECONDS` | 0 | Forçar a reexecução dos cálculos de calibração a cada N segundos do tempo de captura. |
| `--timezone-offset HOURS` | local | Substituir o deslocamento de fuso horário incorporado nos metadados de saída. |
| `--format FORMAT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação (`NDVI`, `NDRE`, `GNDVI`, `EVI`, `SAVI`, `OSAVI`, `CIG`, …). |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para arquivos TIFF do LATTICE (Survey3 .raw não é afetado). Além disso, a porta de escape que permite que uma captura **sem raw** seja processada — veja [Como é uma pasta de capturas](#what-a-captures-folder-look-like). |
| `--debayered / --no-debayered` | ativado | Emitir o produto linear sem debayering (`Debayered_Images`). Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ativado | Emite a pré-visualização na tela (`Preview_Images`): RGB = equilíbrio de branco (iluminante DAQ, quando disponível; caso contrário, mundo cinza) + gama; multiespectral = alongamento de cores falsas. |
| `--radiance / --no-radiance` | ativado | Emite radianância em float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referência para o produto de refletância LATTICE: `auto` = o alvo dentro do quadro aprovado pela QA é a referência absoluta, reserva para radiação descendente do DAQ (ρ = π·L/E); `target` = estrito (sem substituição do DAQ); `daq` = autoridade do DAQ. Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nenhum | Diretório de varreduras de refletância do alvo **medida** por unidade (`<serial>.csv`); recorre aos espectros nominais T3/T4P em caso de falha. |
| `--array-alignment / --no-array-alignment` | ativado | Matrizes LATTICE: aplica o alinhamento módulo a módulo registrado no XMP de cada captura (`Chloros:Alignment*`) a todos os produtos processados (desbayering / visualização / radiação / refletância / índice). Operação nula para imagens sem as tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | recorte | Recorta as exportações alinhadas à região de sobreposição comum da matriz para que todos os módulos compartilhem uma única área de cobertura; o `--no-…` mantém a tela completa do sensor (preenchimento em preto fora da fonte). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reamostragem para a distorção de alinhamento. `nearest` preserva os DNs exatos da fonte (sempixel de valores radiométricos). |

### Opções de detecção de alvo

| Sinalizador | Descrição |
| --- | --- |
| `--min-target-size PIXELS` | Tamanho mínimo do alvo no painel (px) para o detector. |
| `--target-clustering 0-100` | Sensibilidade de agrupamento. |
| `--target / --targets` | Tratar a pasta de entrada apenas como(ignorar a detecção de levantamento). |

### Exemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI NDRE GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a custom output location with a project name
chloros-cli process "C:/input" -o "C:/output" -n "Field_A_2026-05-26"
```

### Opções de exportação por produto (LATTICE multiespectral)

O processamento do LATTICE se ramifica para **todos os produtos aplicáveis em uma única passagem**. As quatro opções por tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — estão todos**ativados por padrão**; use o formulário `--no-<type>` para desativar um. As câmeras principais RGB sempre emitem apenas imagens sem debayering + pré-visualização (sem radianância/refletância por banda); portanto, `--radiance`/`--reflectance` não têm efeito para elas. Os sinalizadores são ignorados para Survey3 `.raw` (que segue o caminho padrão de refletância/alvo). *(O antigo sinalizador `--radiometric-output {reflectance,radiance,sensor-response}` foi **removido** e substituído por esses sinalizadores; não há mais o nível `sensor-response`.)*

| Produto | Saída | É necessária aquisição de dados de radiação descendente? |
| --- | --- | --- |
| `--debayered` | Desmosaico linear (`Debayered_Images`). | Não. |
| `--preview` | Visualização prévia (`Preview_Images`): RGB = WB + gama; multiespectral = alongamento em cores falsas. | Não. |
| `--radiance` | float32 W/m²/sr/nm a partir da cadeia radiométrica completa (`Radiance_Images`). | Nº |
| `--reflectance` | uint16 refletância ρ (`32768` = 1,0), pronto para o Pix4D. | **Sim**, a menos que um alvo dentro do quadro aprovado no controle de qualidade (QA) o ancore (veja abaixo). |

`--reflectance-source` seleciona a referência de refletância:**`auto`**(padrão) define um alvo dentro do quadro aprovado no controle de qualidade como a**referência absoluta**— as cadeias de linha empíricas ancoradas no alvo sãoem painéis de teste e o vencedor medido é aplicado — recorrendo à divisória descendente do DAQ (ρ = π·L/E) quando não há alvo presente ou o QA falha;**`target`**é rigoroso (sem substituição do DAQ);**`daq`**opta pelo comportamento autoritário do DAQ. A geometria do alvo (ArUco / ROI fixa / faixa) provém da configuração de alvo do projeto; `--target-reflectance-dir DIR` mantém varreduras**medidas** por unidade (`<serial>.csv`) consultadas pelo número de série/QR da unidade-alvo, com os espectros nominais T3/T4P como alternativa.

O caminho de refletância do DAQ resolve automaticamente a **radiação descendente com carimbo de data/hora correspondente**a partir de um**`.daq`**(DAQ-U/M/E) registrado**ou de um `.csv` nativo do DAQ-M**encontrado junto com as imagens. Se um pacote de calibração por câmera ou DAQ não estiver armazenado em cache localmente, o pipeline**o busca automaticamente na AWS** na primeira utilização (requer conexão com a internet uma vez; armazenado em cache sob o nome `~/.chloros/`).

#### Leitura de pixels de refletância (Pix4D / Metashape / seus próprios scripts)

A refletância é armazenada como um valor inteiro DN, e **o valor de DN que corresponde a ρ = 1,0 depende da câmera de origem**:

| Origem | ρ = 1,0 corresponde a | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem até ρ 2,0) | O XMP `Chloros:PixelScale=32768` está gravado no arquivo. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há tags XMP `Chloros:*` — essa ausência *é* o sinal. |

**Leia `Chloros:PixelScale` e divida por esse valor**, em vez de supor uma constante. A tag é definida no domínio uint16, portanto permanece como `32768` em todos os formatos de saída que redimensionam — `TIFF (16-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` são todos autodescritivos (normalizam o tipo de dados armazenado de volta para uint16 primeiro: ×257 a partir de 8 bits, ×65535 a partir de float).

> **Um caso não carrega escala, por padrão.** Quando uma captura de fonte de 8-bits (BayerRG8) é gravada como TIFF de 8 bits, o pipeline *limita* o valor a 0..255 em vez de reescalar; assim, todo valor acima de ρ≈0,008 é nivelado para 255 e nenhuma escala descreve o arquivo. Chloros omite deliberadamente tanto o `Chloros:PixelScale` quanto a tupla `MicaSense:RadiometricCalibration` ali, e registra o motivo. **Se a tag estiver ausente em um arquivo de refletância LATTICE, não presuma uma escala — reexporte em 16 bits ou 32 bits**, em vez de dividir pixels que nunca foram divisíveis.

#### EXIF transferido para a exportação

O `process` copia o **bloco GPS e seu ExifIFD** da captura de origem para cada produto; portanto, uma
exportação contém `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` junto com a georreferência.

**`FocalLength` não é opcional para a fotogrametria.** O Pix4D calcula a distância entre amostras no solo a partir da
distância focal mais a altitude; sem essa tag, ele recorre a uma escala totalmente errada. Em um
voo sobre um pomar de laranjeiras com 49 capturas, a falta da tag transformou um local de 411 m × 160 m em um reconstruído
de 47,8 km × 13 km — uma ortofoto de 455 MP composta principalmente por “nodata”, que então foi interpretado como um problema de mosaico e
um problema de BigTIFF antes que alguém verificasse a GSD. Se sua ortofoto apresentar uma escala implausível,
execute primeiro o `exiftool -FocalLength` no produto exportado.

A cópia é deliberadamente **não** é o `-all:all`: as tags estruturais do IFD0 corrompem a saída do LATTICE quando
copiadas, e o `ExifImageWidth` / `ExifImageHeight` são excluídos porque descrevem a
*captura ***fonte*** — uma exportação que já tenha sido redimensionada, de outra forma, conteria dimensões
contraditórias em relação à sua própria malha. O XMP é gravado diretamente, em vez de copiado, porque o ExifTool
descarta as tags XMP da mesma invocação quando o bloco XMP é copiado (o que eliminaria as tags de calibração MAPIR
).

### Onde os arquivos de saída são salvos

Os produtos são gravados **na pasta do projeto, agrupados por câmera e, em seguida, por formato de arquivo**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para o LATTICE (correspondendo à captura&#x27;EXIF
`Model`) e `<model>_<filter>` para Survey3 — duas câmeras que compartilham um sensor e um filtro, mas diferem
em distorção de lente, campo de visão e distorção diferem. O formato
da pasta segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` para
`TIFF (32-bit, Percent)`.

> **Todo produto exportado mantém o nome do arquivo ORIGINAL.** Uma exportação do Radiance de
> `capture_…_raw.tif` continua se chamando `capture_…_raw.tif` — ele apenas está localizado em
> `tiff32/Radiance_Images/`. **A pasta identifica o produto, não o nome do arquivo**; portanto, uma busca por
> `*radiance*.tif` não encontra nada; em vez disso, faça a correspondência com o diretório.

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

O `process` também lida com as gravações `.daq` na sua pasta de entrada, e **não**
precisa de nenhuma imagem para fazer isso: um DAQ-U / DAQ-M / DAQ-E operado sozinho já constitui uma
captura completa, e uma pasta contendo apenas arquivos `.daq` é uma entrada válida.

Um DAQ pode ser gravado **sem** sua calibração — é isso que os
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) (`record_daq.py`) fazem por padrão: eles gravam as contagens brutas do sensor e marcam o arquivo para que
Chloros busque a calibração de fábrica desse sensor **pela porta serial** (primeiro no cache local,
depois na nuvem MAPIR Cloud) e a aplica. O `process` grava o resultado de volta:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

O `.csv` contém uma linha por leitura: carimbo de data/hora UTC, tempo de integração, potência total,
lux fotópico/escotópico, PPFD (e sua divisão em azul/verde/vermelho), comprimento de onda de pico e, em seguida, o
espectro completo na própria grade de comprimentos de onda do sensor. O `.daq` reimporta sem ser
calibrado uma segunda vez.

Se for bem-sucedido, a execução gera o relatório `Light-sensor products written: N (calibrated .daq + .csv)`.
O texto entre parênteses descreve o que foi realmente gravado, portanto, a leitura é
`(RAW COUNTS — this sensor has no calibration bundle)` para um sensor sem pacote e
`(N calibrated, M raw counts)` para uma pasta contendo ambos. Os próprios títulos do backend,
`[DAQ-EXPORT]` e `[RUN-SUMMARY]`, derivam sua redação da mesma forma — nenhum dos
três pode classificar uma exportação bruta como calibrada.

Uma gravação DAQ-U / DAQ-M / DAQ-E cujo pacote de calibração não possa ser obtido — você está
offline ou aquele sensor não possui calibração registrada — é **ignorada com um motivo** em uma
linha `[DAQ-EXPORT]`, nunca gravada como um arquivo “calibrado” contendo contagens brutas.
Conecte-se à internet e execute novamente. O motivo é aquele que o leitor realmente
estabeleceu para aquele arquivo (esquema ilegível, ausência de pacote, erro de gravação), e o resumo da execução
lista motivos **distintos** — vinte arquivos ignorados por um único motivo são interpretados como um
único motivo, e não como vinte repetições dele.

#### Gravações DAQ-A são exportadas como contagens brutas

A família **DAQ-A** é anterior ao sistema de pacotes por número de série e não possui pacote de calibração
para ser buscado — em vez disso, é calibrada em campo em relação a um alvo de refletância, e é
por isso que nunca precisou de um. Recusar essas gravações deixou os usuários sem nenhuma maneira de extrair seus
números, então eles as exportam com um **nome diferente**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Um nome de arquivo diferente, em vez de um sinalizador dentro do arquivo, porque a informação precisa permanecer intacta
ao ser enviada por e-mail apenas como um nome simples. O cabeçalho `.csv` indica
`raw spectral sensor counts (NOT irradiance)` e alerta que os valores são comparáveis
**dentro** do arquivo — que é exatamente para isso que a calibração baseada em alvo os utiliza — e
não entre sensores. As colunas fotométricas dependentes de potência (potência total, lux fotópico e
escotópico, PPFD) são gravadas como **NULL** em vez de serem integradas a partir de contagens, e o resumo da execução
indica `RAW COUNTS`; portanto, os dados “exportados” em um log não pode ser interpretado como irradiância.

Gravações antigas **v1.01 / v1.02** (um DAQ-A-SD as grava) não contêm época por leitura,
apenas a hora de gravação do arquivo. O comparador de imagem↔luz descendente ainda as rejeita — comparar um
quadro com um horário de gravação seria um erro imperceptível — mas o exportador as lê, e
o CSV exibe `clock=daq_created_on`, de modo que o produto indica em qual relógio ele está.

### Observações

- `process` detecta automaticamente se sua pasta é do tipo Survey3, LATTICE ou mista.
- O progresso é transmitido por meio de Server-Sent Events; o CLI mostra o progresso em tempo real por thread (Detecção, Análise, Processamento, Exportação).
- Para o Linux/Jetson, o CLI verifica a memória swap e pode emitir um aviso antes de processar pastas grandes. O debayer sensível à textura também aplica automaticamente um limite de frequência da GPU em Jetsons de baixo consumo de energia (Nano, Orin Nano).
- Em caso de sucesso, a execução informa quantos produtos de imagem foram gravados (`Image products written: N`).

#### Uma execução que não grava imagens falha

Se você solicitou produtos e a execução não gravou **nenhum** — apenas `project.json` e
`calibration_data.json` — o `process` trata isso como uma falha: ele exibe
`Processing finished but wrote no image products.` e **sai com valor diferente de zero**, para que um script possa
detectá-la. A mensagem indica o nome da pasta do projeto e as causas mais comuns:

- a pasta de entrada não foi reconhecida como uma captura (verifique o layout e `--input-level`), ou
- todos os produtos solicitados foram ignorados por serem inaplicáveis para essas câmeras (por exemplo, solicitar
  radianância/refletância de câmeras que suportam apenas RGB).

Execute novamente com `--verbose` e verifique o log do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explicam, porcâmera que, de outra forma, não chegariam à saída do CLI.

Uma execução deliberada apenas com metadados — todos os produtos desativados e sem `--indices` — ainda é um
**sucesso**, pois uma saída de imagem vazia é o resultado correto nesse caso.

O mesmo vale para uma **execução apenas com o sensor de luz**: uma pasta de gravações `.daq` não possui imagens para exportar
por definição, e a execução é avaliada com base no `.daq` / `.csv` que ele gravou em seu lugar.

---

## `chloros-cli login`

Autentique esta máquina com uma conta na nuvem Chloros+. As credenciais são armazenadas em cache de forma segura no `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (removendo partes da senha ou duplicando partes dela). Em caso de erro 401, o CLI tenta automaticamente novamente com `$$` anexado novamente e, em seguida, com metade da senha sem repetições; se a nova tentativa for bem-sucedida, ele faz o login e exibe a sintaxe correta com aspas simples para ser usada na próxima vez.

> **Uso sem interface gráfica/por script: a ausência de sessão em cache significa um prompt interativo, e não uma falha imediata.** Qualquer comando que inicie um backend (`process`, `status`, `export-status`, `time-sync`, …) executado sem uma licença em cache/sessão em cache entra em um prompt interativo `Email:` / `Password:` na stdin antes de prosseguir. Um trabalho autônomo sem sessão em cache, portanto, ficará travado aguardando entrada — execute `chloros-cli login EMAIL PASSWORD` uma vez por máquina antes de agendar trabalhos sem interface gráfica.

---

## `chloros-cli logout`

Limpa a sessão armazenada em cache e força um novo login na próxima chamada.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Exibe o nível de licença atual (Iron/Copper/Bronze/Silver/Gold), o usuário autenticado e a contagem de vinculações de dispositivos.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifica o andamento da exportação do Thread-4 em tempo real. É seguro chamar **durante** a execução de um `process` a partir de outro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Define o idioma de exibição do CLI (38 idiomas suportados, incluindo CJK, RTL e indianos). Recorre ao inglês de forma elegante em consoles legados que não conseguem renderizar o script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemplos

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandos da pasta do projeto

Esses comandos gerenciam o local padrão da pasta do projeto (compartilhada com a GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

---

## `chloros-cli update`

Linux / Apenas para Jetson. Verifica se o `version_url` a partir de `/etc/chloros/update.conf` e oferece o download e a instalação do `.deb` correspondente.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

No Linux/Jetson, o CLI também executa uma **verificação automática de atualizações a cada inicialização** (sem bloqueio, nunca atrasa o comando): ele lê o `/etc/chloros/update.conf`, armazena o resultado em cache por 1 hora no `~/.chloros/update_cache.json` e exibe o `Update available: vX.Y.Z / Run: chloros-cli update` quando existe uma versão mais recente. Ignora silenciosamente qualquer erro e no Windows.

---

## `chloros-cli selftest`

Executa um teste de funcionamento de 7 etapas: versão, disponibilidade de porta, inicialização do backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presença do modelo de redução de ruído, prontidão do CUDA + redutor de ruído.

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status e controle do grandmaster PTP. O host Chloros executa o grandmaster PTP; as câmeras LATTICE e as unidades DAQ-E funcionam como escravas dele para gerar carimbos de data/hora entre dispositivos.

| Subcomando | Descrição |
| --- | --- |
| `status` | Mostrar o estado do grandmaster, as prioridades BMCA e a identidade do relógio. |
| `peers` | Listar os escravos detectados via Delay_Req (câmeras + sensores DAQ-E). |
| `cameras` | Integridade do PTP por câmera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar o processo do grandmaster. |
| `set-priority --priority1 N --priority2 N` | Substituir as prioridades do BMCA. |

### Exemplos

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controle da câmera LATTICE. Todos os subcomandos passam pelo backend Chloros; o backend é responsável pelo conjunto de câmeras, de modo que as chamadas subsequentes de CLI reutilizam o mesmo identificador aberto.

### Opções comuns (compartilhadas pela maioria dos subcomandos)

| Sinalizador | Descrição |
| --- | --- |
| `-d, --device N` | Índice da câmera (padrão: 0). |
| `-s, --serial SN` | Número de série específico; substitui `--device`. |
| `--serials SN1,SN2,…` | Números de série separados por vírgulas para operação com múltiplas câmeras. |
| `--all` | Operar em todas as câmeras detectadas. |
| `--exposure US` | Tempo de exposição em microssegundos. |
| `--gain DB` | Ganho em dB. |
| `--pixel-format FMT` | p. ex., `BayerRG8`, `BayerRG12`. |
| `--width N` / `--height N` | Dimensões da imagem. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplica uma predefinição de configurações. Todos operam em modo livre, exceto `triggered`, que arma a câmera para uma detecção de borda por hardware na Linha 2 — se nada acionar essa linha, ela aguardará indefinidamente em vez de capturar. |
| `-o, --output DIR` | Diretório de saída (padrão: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamanho do pacote GVSP. `auto` executa sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxo de trabalho de primeira conexão da câmera LATTICE

```bash
# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Auto-shrink frame size to fit the wire
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referência de subcomandos

#### Descoberta e informações

| Subcomando | Finalidade |
| --- | --- |
| `lattice info` | Listar câmeras conectadas (fabricante, modelo, número de série, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analisar o sistema host para obter a configuração ideal da câmera. O comando `--no-discover` ignora a descoberta de câmeras (mais rápido, análise apenas pela placa de rede). |
| `lattice network [--fix] [--estimate] [--cameras N]` | Verificar/corrigir configurações da placa de rede; estimar largura de banda/FPS. |
| `lattice network-analysis --master SN --slaves SN1,SN2,… [--width N] [--height N] [--pixel-format FMT] [--binning N] [--force-tier TIER] [--backend-url URL] [--json]` | Capacidade de rede do backend com esquema estável + recomendação de matriz (retorna `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `auto_capped_fps` mantém a resolução solicitada, mas limita o fps alvo — leia `recommended.recommended_target_fps` e passe-o como destino de conexão; trate-o como sucesso, não como erro. |
| `lattice analyze-array [--models M1,M2,…] [--binning N] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--json]` | Análise de cenários hipotéticos sem abrir as câmeras. **`--n-active` é o número total de câmeras na rede, não apenas o desta matriz**— aumente-o quando câmeras autônomas transmitirem simultaneamente, ou quando o orçamento da rede for calculado com base em uma demanda que as subestime (padrão: `len(--models)`). Sempre exibe o `Wire budget:` agregado (MB/s solicitados versus limite máximo seguro contra colisões) e as linhas `Max cameras:`, e sinaliza `** OVER-SUBSCRIBED**` quando o array sobrecarrega a largura de banda — consulte [Modelo de fps e burst do array](#array-fps--burst-model). |
| `lattice gpu` | Mostrar o status da GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verificar ou atualizar o firmware da câmera. A seleção local `.fwa` é fixada: o arquivo em `firmware/<MODEL_PREFIX>/` que corresponde ao `MIN_FIRMWARE_VERSION` da compilação é gravado quando presente (apenas a versão mais alta como alternativa), de modo que uma imagem do fornecedor mais recente armazenada no disco permanece inativa até que esse pino seja alterado — versões deliberadamente mais recentes chegam às unidades por meio do manifesto assinado da AWS, que é preferido quando mais recente. |
| `lattice presets [--apply NAME]` | Listar ou aplicar predefinições da câmera. |
| `lattice status` | Status da câmera ao vivo. |

#### Captura

| Subcomando | Finalidade |
| --- | --- |
| `lattice capture [--format tiff\|png\|jpg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--force-daq]` | Quadro único. **Salva todos os tipos de exportação por padrão** (`--processing all`); consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). `--levels` salva um subconjunto explícito (substitui `--processing`); o `--force-daq` grava a leitura do DAQ atribuída como um sidecar `.daq`, mesmo em uma captura apenas em formato bruto. `--jpeg-quality` = JPEG qualidade 1–100 (padrão 95). |
| `lattice continuous [--format tiff\|png\|jpg] [--jpeg-quality N] [--queue-depth N]` | Transmite para o disco até Ctrl+C. |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Visualização MJPEG ao vivo no navegador. `--ae-damping` define o amortecimento da exposição automática (0,4–100). |

#### Ajuste do sensor

| Subcomando | Finalidade |
| --- | --- |
| `lattice configure [--get N1 N2…] [--set N=V N=V…] [--dump] [--json]` | Ler/gravar qualquer nó GenICam. |
| `lattice exposure [--auto] [--auto-once] [--off] [--set US] [--brightness N] [--damping F] [--upper-limit US]` | Exposição e AE. |
| `lattice gain [--auto] [--off] [--set DB]` | Ganho e ganho automático. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N] [--binning-mode Sum\|Average]` | ROI do sensor e binning. |
| `lattice format [--set FMT] [--list]` | Formato de pixel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Gatilho de hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sem sinalizadores = WB de disparo único) | Operações de WB. Apenas câmeras RGB/Bayer; uma operação nula (ignorada) em M3M monocromático. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct K] [--get]` | RGB pipeline de cores de exibição. `natural` (padrão) é o acabamento ao vivo mais econômico; `enhanced` adiciona remoção de franjas + vibração + contraste local CLAHE para o visual completo com paridade de hub a cerca de 2× o custo de acabamento por quadro, portanto, uma taxa de quadros **ao vivo** mais baixa — as capturas salvas sempre recebem o acabamento completo de qualquer maneira. Apenas para câmeras RGB/Bayer; ignorado em M3M monocromáticas. |
| `lattice color [--saturation N] [--contrast N] [--reset] [--get]` | Saturação/contraste da exibição (câmeras com filtro RGB). Ignorado em M3M. |
| `lattice filter [--set NAME] [--list]` | Define o modelo de filtro da câmera (`RGN-IMX265`, `OCN`, `NGB`, …). |
| `lattice power [--sleep]` | Verificar nós de potência/térmicos; alternar para modo inativo de baixo consumo. |

#### Calibração e sensores

| Subcomando | Finalidade |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de um alvo de refletância. |
| `lattice dls [--connect] [--spectrum] [--irradiance] [--mac MAC] [--filter NAME] [--json]` | Comandos integrados para o sensor de luz descendente. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplicar correção de vinheta a imagens existentes. |

#### Multicâmera (Sessões Transitórias)

| Subcomando | Finalidade |
| --- | --- |
| `lattice multi-info` | Listar todas as câmeras com funções de sincronização. |
| `lattice multi-capture [--format FMT] [--jpeg-quality N] [--processing LEVEL]` | Um quadro sincronizado de cada câmera. Salva **todos os tipos de exportação por padrão**quando uma matriz persistente está conectada; o fallback transitório sem matriz é**apenas despixorado** (execute `array-connect` primeiro para o restante). |
| `lattice multi-stream [--fps F] [--count N] [--format FMT] [--jpeg-quality N]` | Transmite quadros sincronizados (transitório). |
| `lattice multi-test [--count N]` | Teste de temporização de sincronização GPIO. |
| `lattice multi-detect [--line LINE] [--json]` | Detecção automática da fiação mestre/escravo do GPIO. |

#### Alinhamento

| Subcomando | Finalidade |
| --- | --- |
| `lattice align-calibrate [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N]` — além dos controles do detector/comparador `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, controles do RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinação de-quadros `[--averaging mean\|median\|inlier_weighted]`, restrições geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrição espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` e substituições por escravo `[--per-cam-override SN:KEY=VALUE]` (repetível) | Calcula o perfil de alinhamento a partir de câmeras em tempo real. `--prefilter` usa como padrão o `gradient` (mapa de bordas; corresponde ao alinhador de GUI/matriz — as bordas permanecem entre as bandas espectrais). O `--matcher flann` compensa a partir de ~5.000 características; o `--averaging median` é robusto contra uma captura defeituosa; o `inlier_weighted` pondera pela contagem de correspondências; o `--lock-scale` projeta para a rotação mais próxima (sem escala), o `--lock-axis` zera um componente de translação; o `--mask` se aplica a todas as câmeras (use o `--per-cam-override` para configurações por câmera, por exemplo, o `--per-cam-override 214701292:method=phase`). O `--rms-threshold-px` recusa salvar uma calibração cujo RMS de reprojeção exceda o limite. |
| `lattice align-apply --profile PATH [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N]` | Captura um quadro multibanda alinhado. `--bit-depth` usa como padrão a correspondência com a câmera; `--no-crop` mantém o quadro completo (preenchendo com preto); `--interpolation` (padrão `linear`) e `--border-mode`/`--border-value` (padrão: `constant`/0) controlam a distorção pela CPU — o caminho pela GPU é bilinear independentemente disso. |
| `lattice align-stream --profile PATH [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N]` | Quadros multibanda alinhados ao fluxo (mesmos controles de warp que `align-apply`). |
| `lattice align-info --profile PATH [--json]` | Exibir detalhes do perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Alterar a ordem das camadas. |

#### Índice / Cálculos de vegetação

```bash
# Offline: compute NDVI from an aligned multi-band TIFF
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: discover array, calibrate alignment, capture, compute index, in one go
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --save-multiband -o output/
```

Conjunto completo de sinalizadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (repetível), `--capture-level raw|debayered|radiance|reflectance|unknown` (substitui o nível de captura registrado na fonte TIFF; padrão: lido dos metadados de TIFF), `--output PATH`, `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Com o `--live`, os controles de distorção de alinhamento também se aplicam: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`.

> **Os símbolos do `--channel` diferenciam maiúsculas de minúsculas.** O lado do símbolo deve corresponder exatamente aos nomes dos canais da predefinição (as predefinições usam letras minúsculas, por exemplo: NDVI = `red`, `nir` — verifique `--list-presets`), e o lado da banda deve corresponder a um nome de banda na pilha alinhada (ou ser um índice de banda com base em 0 no modo offline). `--channel red=Red_660 --channel nir=NIR_850` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.

#### Conexões persistentes (Smart-Prep, fluxo equivalente à GUI)

Esses comandos mantêm as câmeras abertas no pool de back-end entre as invocações do CLI.

| Subcomando | Finalidade |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adiciona uma câmera ao pool (câmera única, sem matriz). |
| `lattice cam-disconnect [--serial SN] [--all]` | Libera. |
| `lattice cam-list` | Lista as câmeras no pool. |
| **`lattice array-connect`**|**Conectar um array sincronizado persistente (O ponto de entrada recomendado).** Executa todo o fluxo de preparação inteligente da GUI. |
| `lattice array-disconnect [--array-id ID] [--all]` | Liberar um array. |
| `lattice array-list` | Listar matrizes conectadas. |
| `lattice array-status [--array-id ID]` | FPS em tempo real, PTP, último erro. |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--index\|--no-index] [--force-daq] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Uma captura sincronizada da matriz ao vivo — Única / Contínua / Intervalada / Mais rápida. **Padrão: `all`** (um arquivo por tipo de exportação aplicável por câmera). Câmeras ignoradas (por exemplo, RGB excluída da radiação/refletância) são relatadas com `Skipped: SN:<serial> (<reason>)`; a leitura do DAQ usada para refletância é salva junto e relatada com `DAQ: <path>`. Consulte [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only]` | Grava a visualização do índice combinado em tempo real como vídeo/GIF (nível de monitoramento; requer que o fluxo combinado esteja aberto). |
| `lattice array-burst [--duration S] [--max-frames N] [--build] [--products …]` | Série de imagens Bayer brutas com alta taxa de quadros por segundo (nível de análise; reprocessamento offline). |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocessar uma sequência raw salva em vídeo(s) calibrado(s). |

##### Opções do `array-connect`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | Detectar automaticamente todas as câmeras LATTICE (requer ≥2) | A primeira série é a MASTER. Quando omitida, a detecção filtra para modelos LATTICE (`TRI032*`) e conecta todos eles. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | automático | Taxa de disparo do gatilho mestre. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituir o seletor de camada. |
| `--wire-ceiling-mbps MB_PER_S` | detectado automaticamente | **O orçamento de banda sustentado do host, em MB/s — o número do qual depende toda a alocação do array.** Reduza-o quando o array relatar quadros com corrupção GVSP: o valor automático é derivado da, que superestima adaptadores USB, pistas PCIe estreitas e estruturas compartilhadas ocupadas. Persiste no bloco de captura do array do projeto; portanto, uma reconexão via “reopen / CLI / SDK” o restaura. Consulte [Integridade do array](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | auto | Agrupamento de hardware. |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativar PTP (os carimbos de data/hora entre câmeras **não** são comparáveis). |

### Smart-AE / Smart-Capture

Os arranjos LATTICE executam o AE contínuo em segundo plano assim que são conectados, mas uma cena recém-focalizada leva um momento para convergir. `array-capture --smart` é a **opção prática integrada**: ela aguarda até que a AE se estabilize em todas as câmeras do array e, em seguida, aciona a captura. Use-a ao trocar de cena no meio de uma sessão.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

A política de estabilização é conservadora por padrão: tempo limite de 5 s, janela de estabilidade de 1,5 s, tolerância de variação de exposição de ±5 %. Ajuste por meio do SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se precisar de um comportamento diferente da automação.

### Níveis de exportação de captura (padrão do `all`)

A partir desta versão, `lattice capture`, `lattice multi-capture` e `lattice array-capture` **têm como padrão o `--processing all`** — um arquivo salvo por tipo de exportação que se aplica a cada câmera, correspondendo ao comportamento da opção “Capturar tudo” da interface gráfica. Os níveis são:

| Nível | Saída | Aplica-se a |
| --- | --- | --- |
| `raw` | Bayer de canal único (câmeras monocromáticas: a banda única) diretamente do sensor. | Todas as câmeras. |
| `debayered` | Desmosaico BGR de 3 canais (câmeras monocromáticas: escala de cinza de 1 canal). | Todas as câmeras. |
| `radiance` | float32 W/m²/sr/nm por meio da cadeia radiométrica completa. | Apenas multiespectral (M3C/M3M) — **omitido para câmeras com filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pronto para Pix4D. | Apenas multiespectral, e **somente quando um DAQ estiver vinculado + a câmera estiver calibrada**; caso contrário, ignorado. |
| `preview` / `display` | Cadeia completa de visualização na GUI (CCM + WB + gama de acordo com o perfil da câmera). `lattice capture` nomeia este como `preview`; `array-capture`/`multi-capture` usam `display`. | Todas as câmeras. |

Passe um único nível para salvar apenas esse (`--processing debayered`). Ao solicitar o `all`, os níveis que não se aplicam a uma determinada câmera são ignorados (e relatados), sem gerar erro — uma câmera desconectada ou não calibrada ainda recebe `raw` / `debayered` / `preview`.

Para qualquer quadro de refletância, a leitura de radiação descendente do DAQ efetivamente utilizada é gravada em um **`.daq`** ao lado da imagem (para que a captura possa ser reprocessada posteriormente) e relatada em uma linha `DAQ:`.

### Como é uma pasta de capturas

Cada tipo de exportação é armazenado em sua **própria subpasta** dentro de `-o`, de modo que uma captura em vários níveis nunca mistura tipos:

```
output/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
└── *.daq          the downwelling reading matched to the capture
```

`<ts>` é o carimbo de data/hora da captura e `<serial>` o número de série da câmera, de modo que um grupo sincronizado compartilha um
carimbo de data/hora entre as câmeras. **Observe a única assimetria:** o nível `display` é armazenado em uma pasta
chamada `preview/`, enquanto os próprios arquivos mantêm `_display` no nome — a pasta e o sufixo diferem
apenas para esse nível. Níveis desconhecidos são direcionados para uma pasta com seu próprio nome e, se a subpasta
não puder ser criada, o arquivo é gravado na raiz de saída, em vez de ser perdido.

**Reprocessamento de uma pasta de capturas:**aponte `chloros-cli process` para a**raiz das capturas**
(`output/`). O `process` normalmente importa apenas a pasta que você nomear, mas quando essa pasta não contém
imagens e possui subpastas, ele percorre automaticamente a árvore — assim, as subpastas do nível raiz e a
raiz `.daq` são todos selecionados de uma só vez. Cada nível de uma captura é importado como uma única imagem, com
os outros níveis disponíveis como modos, em vez de uma imagem por nível.

Nomeando uma **subpasta de nível** diretamente (por exemplo, `output/raw/`) também funciona. Ao fazer isso, a raiz
`.daq` fica para trás; portanto, copie ou indique a leitura do DAQ ao lado quando você derivar novamente um produto radiométrico
a partir de `raw/` — caso contrário, a correspondência de carimbo de data/hora não terá nada com que se relacionar.

**O processamento sempre começa a partir de `raw`.** Dentro de cada captura, o quadro bruto é a fonte do pipeline;
`debayered`, `radiance`, `reflectance` e `preview` surgem como modos visualizáveis, mas nunca são realimentados
de volta ao pipeline. O reprocessamento de um produto derivado reaplicaria vinheta, CCM e
cálculos de radiação que já estão incorporados em seus pixels; portanto, o Chloros é descartado em vez de
ser processado duas vezes. Duas consequências que vale a pena conhecer:

- As renderizações `index/` e `composite/` **nunca** são processadas. Elas são saídas, não capturas —
  uma renderização com LUT NDVI não tem interpretação significativa de radiance.
- Uma pasta de capturas exportada **sem** o `raw` (por exemplo, `array-capture --processing reflectance`) não tem
  nenhuma fonte válida no pipeline. Essas capturas são importadas e exibidas normalmente, mas o `process` as ignora
  e informa isso:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se você realmente precisar passar um produto derivado — uma sessão de hub capturada com
  o `demosaic` ativado, ou uma pasta legada — o `--input-level {raw,debayered,processed}` força o ponto de entrada
  e anula a omissão. Esse sinalizador é a saída de emergência deliberada; o `auto` (padrão)
  nunca processa uma captura que não tenha dados brutos.

### Capturas ignoradas em matrizes com filtros mistos

Quando você mistura câmeras RGB e multiespectrais em uma matriz, `array-capture --processing radiance` (ou `reflectance`) salva os quadros multiespectrais e **ignora** as câmeras RGB — a radiação por pixel Bayer não é significativa para um sensor de banda larga. O CLI imprime cada arquivo salvo (com seu nível de exportação), cada `.daq` gravado e cada omissão explicitamente, de modo que a contagem de arquivos não é surpreendente:

```
  Saved: output/sync_…_SN213800234.tif [reflectance] (SN:213800234, fid:1)
  Saved: output/sync_…_SN214000533.tif [reflectance] (SN:214000533, fid:1)
  Saved: output/sync_…_SN214701288.tif [reflectance] (SN:214701288, fid:1)
  DAQ:   output/sync_…_daq-e-54b5e0.daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Os tokens de motivo de omissão seguem o padrão `<level>-not-applicable-to-rgb-cam`. A refletância também pode ser omitida com `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, e com `dls-uncalibrated-band-<nm>` quando a banda se encontra principalmente fora da faixa radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm) — entre os SKUs disponíveis para envio, apenas o F988, cujo caminho suportado é o fluxo de trabalho com painel de refletância.

Use `--processing debayered` (ou `display`) para incluir todas as câmeras, independentemente do tipo de filtro, ou o padrão `all` para obter todos os níveis aplicáveis por câmera de uma só vez.

---

## Modos de captura, gravadores e reprocessamento offline

Todos eles operam em uma **matriz persistente** (execute `array-connect` primeiro). Eles espelham o painel de captura da interface gráfica (GUI).

### Modos do `array-capture`

O `array-capture` é um único comando com quatro modos de obturador, além de um conjunto de opções de exportação:

| Modo | Sinalizador | Comportamento |
| --- | --- | --- |
| **Único** *(padrão)* | (nenhum) | Um grupo de captura sincronizado, em seguida, sai. |
| **Contínuo** | `--continuous` | Passagens consecutivas até `Ctrl+C`, `--count N` ou `--duration S`. |
| **Intervalo** | `--interval S` | Uma passagem a cada `S` segundos (medidos a partir do início de cada passagem), com os mesmos limites. |
| **Mais rápido** | `--fastest` | Apenas dados brutos + a leitura DAQ atribuída + o composto de índice combinado; ignora os cálculos de radianância/refletância/exibição para que o quadro seja gerado rapidamente. Implica `--processing raw --force-daq`. Reproduzao `.daq` salvo em produtos calibrados posteriormente. |

Opções de exportação (combináveis com qualquer modo; todas compartilham a interface gráfica/ponto final SDK):

| Sinalizador | Efeito |
| --- | --- |
| `--processing LEVEL` | Nível único de exportação, ou `all` (padrão). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportação (por exemplo, `raw,radiance,reflectance`); **substitui `--processing`**. |
| `--aligned` / `--no-aligned` | Realiza o alinhamento da exportação não bruta de cada membro ao [perfil de alinhamento](#alignment) da matriz (co-registrado). Os dados brutos permanecem sem warping, mas carregam a transformação nos metadados. Recorre ao modo não alinhado (com um aviso) se a matriz não tiver perfil. |
| `--index` / `--no-index` | Salvar / ignorar a sobreposição do índice de vegetação por câmera onde estiver configurada. Padrão: renderizá-la. |
| `--force-daq` | Salvar a leitura DAQ/DLS atribuída como um arquivo sidecar `.daq`, mesmo quando nenhum nível selecionado precisar dela (por exemplo, uma captura apenas de dados brutos), para que os quadros possam ser reprocessados em refletância/índice offline. |
| `--smart` | Aguardar que o AE se estabilize em todas as câmeras antes de acionar (consulte [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | TIFF compressão de pixels. `deflate` (padrão) = zlib L1 sem perdas + preditor horizontal, ~4,1 MB por quadro em resolução total; `none` = não compactado, gravação ~5× mais rápida a ~6,3 MB por quadro — use para obter a taxa sustentada máxima quando o disco permitir. Ambos são sem perdas e são lidos de forma idêntica na importação. |

> **Gravação única TIFF + o modelo de taxa sustentada.**As capturas são gravadas em**uma**única passagem pelo arquivo TIFF, transportando pixels + XMP + IFD0 Marca/Modelo (medido em Mono12 de resolução total: 36 ms compactado / 6,5 ms não compactado, contra ~148 ms para o método antigo de “gravar e depois reescrever com o ExifTool”); o único trabalho restante do ExifTool (o refinamento do sub-IFD) é executado em um processador de fundo assíncrono, e um quadro fica completo e pronto para importação mesmo que esse processador nunca seja executado. Observe que a compactação DEFLATE mantém o GIL do Python, portanto as gravações compactadas**não**são paralelizadas entre as threads de gravação de cada câmera — a captura sustentada em resolução total com 8 câmeras na taxa do sensor (~10,4 fps) requer o `--compression none`**e** um disco da classe NVMe (~500 MB/s de gravações sustentadas). O mesmo controle é exposto como `compression` no `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --aligned --no-index -o out/
```

### `array-record` — vídeo/GIF com índice combinado (nível de monitoramento)

Grava tudo o que a **visualização ao vivo com índice combinado** esteja exibindo em um `.avi` (e, opcionalmente, em um `.gif`). Como ele capta o composto ao vivo, o fluxo combinado deve estar aberto (por exemplo, a matriz está sendo visualizada na GUI) para que os quadros sejam capturados. Ele verifica o progresso a cada 2 s e interrompe no `--duration`, `Ctrl+C` ou quando o gravador encerra automaticamente.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino (omita se houver apenas uma conectada). |
| `-o, --output DIR` | `output` | Diretório de saída (local no backend). |
| `--fps F` | `10` | Taxa de quadros de gravação. |
| `--duration S` | até Ctrl+C | Parada automática após `S` segundos. |
| `--gif` | desativado | Também gravar um GIF animado. |
| `--gif-only` | desativado | Gravar apenas um GIF (sem `.avi`). |

### `array-burst` — sequência de alta taxa de quadros em raw-Bayer (nível de análise)

Lê diretamente o buffer do grupo sincronizado do loop de captura — **sem necessidade de cadeia de calibração, exiftool ou visualização ao vivo** — portanto, opera na taxa máxima de captura da câmera. Grava quadros raw + um manifesto por quadro + um `.daq` por leitura DLS distinta sob `<output>/bursts/<base>/`. Reprocesse off-line (próximo comando) ou passe `--build` para fazer isso imediatamente ao parar.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Diretório de saída (a sequência de dados é armazenada em `<DIR>/bursts/<base>/`). |
| `--duration S` | até Ctrl+C | Parada automática após `S` segundos. |
| `--max-frames N` | sem limite | Parada automática após `N` quadros brutos. |
| `--build` | desativado | Após a parada, reprocessar imediatamente a sequência (igual a `array-build-video`). |
| `--products …` | `combined:index` | Com `--build`: quais vídeos devem ser gerados (veja abaixo). |
| `--fps F` | `10` | Com `--build`: fps do vídeo de saída. |
| `--save-tiffs` | desativado | Com `--build`: também salvar arquivos TIFF calibrados por quadro. |
| `--gif` | desativado | Com `--build`: também gravar GIFs animados. |

### `array-build-video` — reprocessar offline uma sequência salva

Sincroniza temporalmente cada quadro bruto com a leitura `.daq` salva mais próxima e o envia pela **mesma cadeia de índice de radiação / refletância / que o pipeline de importação**, gerando um ou mais vídeos.

`--products` é uma lista separada por vírgulas de itens `kind:level`, onde `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Um `level` isolado (sem `kind:`) tem como padrão o `per_cam`. O padrão é o `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--burst-dir DIR` | (obrigatório) | Caminho para a pasta de burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista de `kind:level`, conforme acima. |
| `--fps F` | `10` | fps do vídeo de saída. |
| `--save-tiffs` | desativado | Também salvar arquivos TIFF calibrados por quadro junto com o(s) vídeo(s). |
| `--gif` | desativado | Também gravar GIFs animados. |

> **Escolha o gravador correto.** `array-record` é *de nível de monitoramento* — ele captura o composto ao vivo exatamente como é exibido e precisa que o fluxo esteja aberto. `array-burst` → `array-build-video` é *de nível de análise* — ele salva os dados brutos do sensor na taxa total e reconstrói vídeos calibrados de radiação/refletância/índice posteriormente, sem a necessidade de visualização ao vivo.

### Câmeras Mono (M3M) de banda única

A linha **M3M**é a versão monocromática da**M3C**da Bayer: um filtro de interferência de banda estreita por câmera (`M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685`), de modo que o sensor fornece uma**única faixa em escala de cinza**, sem mosaico Bayer. Não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de branco para definir — todo o fluxo de processamento de cores da tela do RGB simplesmente não se aplica.

O que isso significa no CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectam uma câmera monocromática e**ignoram a operação com uma mensagem de uma linha**, em vez de aplicar configurações sem sentido. Eles ainda funcionam normalmente com uma câmera RGB/Bayer M3C na mesma sessão.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** ainda funcionam — a radianância e a refletância são mapas radiométricos *por banda* e estão perfeitamente bem definidos para uma banda. Quadros monocromáticos carregam uma matriz de resposta do sensor **identitária** (sem desmistura 3×3), de modo que o plano passa pela matemática de calibração inalterado.
- **Uma única câmera monocromática não pode produzir um índice de vegetação.**NDVI/NDRE/etc. precisam de pelo menos duas bandas (por exemplo, Red + NIR). Para obter um índice a partir de hardware monocromático, aponte**várias** câmeras M3M em diferentes comprimentos de onda, alinhe-as em uma pilha multibanda e calcule *esse*:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

Os símbolos de `--channel` devem corresponder **exatamente** aos nomes dos canais da predefinição (diferencia maiúsculas de minúsculas; NDVI corresponde a `red` e `nir` — veja `--list-presets`), e os nomes laterais de banda devem corresponder a uma banda na pilha alinhada (o modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`).

O discriminador em toda a pilha é o token `M3M` na string do modelo (ele nunca aparece em uma string `M3C`), exibido na GUI/SDK como `is_mono`.

---

## Configuração e ajuste da placa de rede do host (matrizes LATTICE)

As câmeras LATTICE transmitem GVSP pela placa de rede Ethernet do host; portanto, para matrizes com várias câmeras, o **driver**da placa e o**tamanho do anel de recepção** são tão importantes quanto a taxa de transmissão. Configurações incorretas aparecem como um bloqueio `FRAMES WILL DROP` / `Reduce ROI to enable` no painel de Configurações da Matriz (e no `lattice network-analysis` / no `analyze_array_network()` do SDK), mesmo quando as próprias câmeras estão funcionando corretamente.

### Adaptadores USB 10GbE — Realtek RTL8157 (“Controlador da Família Realtek USB 10GbE”)

| Item | Valor necessário | Por que é importante |
| --- | --- | --- |
| **Versão do driver**|**≥ v10.67 (janeiro de 2026)**, INF `rtump64x64sta.inf` | O driver legado de**2016**(v10.65, `rtump64x64.inf`) lida incorretamente com o desligamento e verificações de erros com**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ao desligar, reiniciar ou entrar no modo de suspensão. A transição fica travada (tempo limite de ~5 minutos), o usuário desliga o computador à força, e desligamentos incorretos repetidos**corrompem o repositório WMI**(o PowerShell e outras ferramentas começam a apresentar falhas com o código `Invalid class`) e**bloqueiam a pilha USB** na próxima inicialização (a placa de rede não é ativada; as unidades USB param de ser reconhecidas). Faça a atualização no site realtek.com (ou junto ao fabricante do dongle) antes de contar com reinicializações corretas. |
| **Buffers de recepção**— palavra-chave `ReceiveBufferLen` |**256**(máximo do driver) | O anel RX da placa de rede. O padrão do driver de**32**deixa apenas ~0,26 MB de anel utilizável — muito pequeno para uma rajada multicâmera —, de modo que o painel do array reporta `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e bloqueia as conexões. Em**256**, o anel é grande (**~13,5 MB, medido no host de 10 GbE do laboratório**), proporcionando ao pipeline de recepção margem real para rajadas GVSP com múltiplas câmeras. (Se uma determinada configuração realmente *estabelece conexão* é decidido por duas verificações — a verificação de admissão **sensível ao esvaziamento**e a verificação de**sobrescrição agregada** — não se trata de uma comparação bruta entre rajada e anel; consulte [Modelo de fps e rajadas do array](#array-fps--burst-model).) |
| **URBs de recepção**— palavra-chave `PendingReceives` |**64** (máx.) | Blocos de solicitações USB em trânsito; aumentam juntamente com os buffers de recepção para absorção de bursts. |
| **Quadro Jumbo** — palavra-chave `*JumboPacket` | **9014** | Necessário para pacotes GVSP de 9.000 bytes (6 vezes menos pacotes por quadro do que em 1500). |

> ⚠️ **Uma atualização do driver da placa de rede REINICIA essas propriedades avançadas para os valores padrão.**Após atualizar ou substituir o driver da placa,**reaplique** `ReceiveBufferLen=256` e `PendingReceives=64`, ou o painel da matriz bloqueará novamente, mesmo que “nada tenha mudado no hardware”. Essa é a principal causa de um equipamento que antes funcionava se recusar repentinamente a se conectar.

Aplique a partir de uma **conteúdo elevado** (substitua pelo nome do seu adaptador, por exemplo, `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` abrange adaptadores USB 10 GbE.** Agora ele detecta o tipo de adaptador e ajusta a palavra-chave correta para o anel de recepção: `*ReceiveBuffers`→2048 para placas de rede PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 para o controlador Realtek **controlador USB** 10GbE (que não expõe `*ReceiveBuffers`). Os valores-alvo são limitados ao máximo relatado por cada driver (`NumericParameterMaxValue`), de modo que nunca seja gravado um valor fora do intervalo. Execute a partir de um terminal com **privilégios elevados**; como qualquer ajuste baseado no Registro, a alteração entra em vigor após a reinicialização do adaptador ou a reinicialização do sistema. Os comandos manuais `Set-NetAdapterAdvancedProperty` acima continuam sendo uma boa alternativa — eles são aplicados em tempo real (reassociam o adaptador) sem a necessidade de reinicialização.

### Noções básicas de rede (todos os links LATTICE)

- **Endereçamento:** link-local `169.254.0.0/16` (GigE Vision LLA). O host recebe um endereço estático `169.254.x.x/16`; as câmeras e o DAQ-E se autoatribuem no mesmo intervalo. Não é necessário DHCP/gateway.
- **Tamanho do pacote:**prefira jumbo (9000), mas deixe que o auto-probe determine o tamanho — ele remedeia a cada conexão e já ignora o limite de 1500-bytes por meio de um teste GVSP, de modo que ele opta pelo jumbo sempre que a linha realmente suportar. Defina com o `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` apenas quando tiver certeza de que sabe mais do que o teste, e prefira a configuração por comando em vez de permanente: uma definição ignora o teste; portanto, se o caminho não puder realmente suportar 9.000**cada**, a captura expira com o `SC_ERR_TIMEOUT -1011` (consulte [Variáveis de Ambiente](#environment-variables)).
- **O anel RX se ajusta com o `ReceiveBufferLen`:**no padrão `32`, o anel utilizável é de ~0,26 MB (muito pequeno para qualquer rajada com múltiplas câmeras); no valor máximo `256`, ele é grande (cerca de 13,5 MB medidos no host de 10 GbE do laboratório), proporcionando margem real. A decisão sobre se uma configuração será conectada é então determinada pela verificação de admissão com consideração de drenagem**e** pela verificação de sobrescrita agregada abaixo — e não por uma comparação bruta entre burst e anel.

### Modelo de fps e burst do array

Como interpretar o painel de configurações da matriz (e `lattice analyze-array` / o SDK do `analyze_array_network`):

- **O burst é somado por câmera no formato de pixel real de cada uma.**As câmeras mono**M3M**transmitem**Mono12 (2 B/px)**; as câmeras Bayer**M3C**transmitem 8 ou 12 bits (o TRI032S emite silenciosamente BayerRG12 mesmo quando é solicitado BayerRG8). Portanto, um quadro em resolução total com 4 câmeras tem**cerca de 12,6 MB se todas forem de 8 bits, mas cerca de 25 MB com três câmeras mono de 12 bits**. A projeção determina o formato de cada câmera a partir de seu modelo (cache de identidade), de modo que o burst corresponde ao que o cabo realmente transmite — e não a uma suposição genérica de BayerRG8.
- **Um adaptador Ethernet USB tem limite máximo de 200 MB/s, independentemente das especificações declaradas.** A tabela de eficiência que converte uma taxa de link em um valor sustentado é derivada do PCIe; uma placa de rede USB anuncia sua taxa de link *Ethernet*, mas é limitada pelo barramento USB e seu driver. Um dongle USB 10GbE costumava atingir ~1.063 MB/s “sustentada” — um número que nunca foi verificado — e o ritmo resultante corrompeu de 6 a 18 % dos quadros, embora ainda relatasse uma taxa de quadros por segundo (fps) alvo satisfatória. As placas de rede conectadas via USB agora têm um limite máximo de **200 MB/s** como valor absoluto (o limite é o barramento, portanto não varia de acordo com a especificação nominal; um adaptador USB de 1 GbE atinge cerca de 80 MB/s e não é afetado). O registro de capacidade `wire_ceiling_source` indica isso explicitamente, e o `nic_is_usb` indica isso. É possível substituir qualquer um dos valores com `--wire-ceiling-mbps`.
- **A admissão leva em conta o consumo de recursos, não a comparação entre burst completo e ring.** Um burst simultâneo precisa apenas caber no *backlog transitório* = `max(0, Σ per-cam arrival − host drain) × emit_window`, não no burst inteiro. Em uma malha de host rápido / cams lentas (um host **PCIe**10G + 4 cams de 1 GbE: chegada ≈ 320 MB/s, drenagem ≈ 1.063 MB/s), o host esvazia mais rápido do que as câmeras enchem, o backlog é ≈ 0; portanto, a emissão simulada em resolução total**admite**, mesmo que a rajada de 25 MB exceda o anel de 13,5 MB. Coloque as mesmas quatro câmeras atrás de um adaptador**USB**de 10 GbE e a drenagem será de 200 MB/s, e não 1063 — a chegada ultrapassa essa taxa, e a perda se manifesta como quadros corrompidos, em vez de uma taxa de quadros mais baixa. Em um host de 1 GbE, o limite mínimo de DLThr das câmeras de 31,25 MB/s faz com que a chegada ultrapasse a taxa de drenagem → ele**bloqueia** corretamente (para *essa* classe de bloqueio, reduza a ROI ou use binning ≥ 2). A admissão é uma das **duas** portas de conexão — a outra é a verificação de sobrescrita agregada abaixo.
- **Os fps projetados representam um limite máximo conservador para a recuperação serial.**Atualmente, o loop de captura do host puxa o buffer de cada câmera**serialmente**(~uma janela de emissão por câmera cada), de modo que o ciclo é limitado por `max(readout+emit, N × emit)`, com a emissão por câmera restrita ao**link de acesso**da câmera (1 GbE ≈ 80 MB/s), e não ao uplink do host. Para um conjunto de 4 câmeras em resolução total de 12 bits, isso equivale a**~2,8 fps**, correspondendo aos ~2,7–3,0 medidos. O fps é deliberadamente**independente da exposição**; portanto, em cenas com pouca luz, o valor real pode cair ligeiramente abaixo do limite máximo à medida que a exposição se prolonga. A recuperação serial é o verdadeiro limitador de fps; a paralelização aumentaria o limite máximo em direção à taxa de emissão única.
- **A supersubscrição agregada é um bloqueador de conexão rígido.**A alocação de largura de banda por câmera tem um limite mínimo de**8 MB/s**(`ARRAY_PER_CAM_FLOOR_BPS`); portanto, uma vez que esse limite mínimo seja atingido, a demanda agregada (`per_cam × N`) pode exceder o**limite máximo de segurança contra colisões**limite de segurança contra colisões na linha**(`sustained × sim_emit_factor`). Limites práticos de resolução total em 1 GbE:**6 câmeras com MTU de 1500, 9 com jumbo**. Esse limite é uma propriedade exclusiva da linha e do limite mínimo — é**independente do tamanho do quadro**; portanto,**o agrupamento e uma ROI menor NÃO ajudam** (eles reduzem os bytes por *quadro*, não os bytes por *segundo* regulados pelo GevSCPD); as únicas soluções são menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede (NIC) mais rápida. O sintoma seria perda de pacotes no GVSP, e não uma redução gradual dos fps; portanto, o `analyze-array` zera os valores de fps alcançáveis e exibe o `**OVER-SUBSCRIBED**`, e o `array-connect` com uma resolução fixa **se recusa a se conectar** (caso contrário, o walk-down agruparia os quadros, o que também não resolve esse tipo de bloqueio). O código `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` reclassifica a recusa como um aviso sonoro para testes de bancada — consulte [Variáveis de ambiente](#environment-variables).

### Integridade do array — qual subsistema está perdendo quadros

O `GET /api/camera/array/<array_id>/capability` de um array conectado contém um
`health`, reavaliado em uma janela contínua de **10 segundos**. Ele divide a perda de quadros
em duas causas que exigem soluções opostas, em vez de relatar uma taxa “incompleta”
que não especifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serial) | O quadro **chegou e estava estruturalmente corrompido**— perda de pacotes GVSP. |**Rede**: capacidade da linha, ritmo, anel de recepção da NIC, MTU |
| `never_arrived_rate_pct` (por serial) | O quadro **nem chegou**— a câmera não disparou ou nada saiu dela. |**Acionador / sincronização**: cabo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A pior taxa de cada câmera. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmera (ambas as causas juntas). | — |
| `stable_for_seconds` | Por quanto tempo cada câmera permaneceu abaixo de 0,01 %. | — |

Acima de 5%, o backend registra uma linha `[array-health <id>] WARN` indicando a divisão — na
primeira violação, em uma mudança de faixa de gravidade, uma vez por minuto enquanto persistir e uma vez quando
for resolvida. A metade corrompida imprime `[gvsp-corrupt <SN>]` na primeira ocorrência por câmera e
motivo, seguido de um resumo a cada 60 s. Todas as avaliações ainda são registradas no arquivo de log do backend;
os contadores são atualizados a cada buffer, independentemente do que for impresso.

O mesmo registro informa o número ao qual toda a alocação está vinculada:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de banda sustentado do host em vigor, em MB/s. |
| `wire_ceiling_source` | De onde veio esse número, em palavras — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `--wire-ceiling-mbps` (ou o campo **Orçamento de banda*) da GUI o definir. |
| `nic_is_usb` | `true` para um adaptador USB Ethernet — veja o limite de 200 MB/s acima. |

**Interpretação:** um valor diferente dezero `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` em 0
significa que o acionamento e a sincronização do cabo estão perfeitos e que 100% da perda está no caminho da rede
— reduza o `--wire-ceiling-mbps` e reconecte. O padrão inverso aponta para o
cabo de sincronização ou para a linha de acionamento.

> **O `--target-fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido
> uma vez na conexão; portanto, diminuir a taxa de acionamento altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na demanda não produziu melhora;
> diminuir o limite máximo do cabo de 240 para 200 MB/s reduziu a taxa de quadros corrompidos do mesmo equipamento de 10,4 %
> de quadros corrompidos para 0,00%.

> **A redução automática no meio do fluxo não está disponível no firmware TRI032S.** Um array em execução
> não pode corrigir isso sozinho; desconecte e reconecte para que o seletor de tempo de conexão possa
> replanejar com o novo limite máximo.

### Sintoma → correção

| Sintoma (Configurações do array / conexão / `analyze_array_network`) | Causa | Solução |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` reiniciado para 32 (normalmente após uma atualização de driver) | Defina `ReceiveBufferLen`→256, `PendingReceives`→64; reabra o painel (reinicie o backend se ele tiver armazenado em cache o tamanho antigo do anel) |
| A reinicialização/desligamento trava; posteriormente, erros WMI `Invalid class`, a placa de rede não é ativada, dispositivos USB ausentes | Driver antigo Realtek USB 10 GbE de 2016 → BSOD `0x9F` → desligamentos forçados | Atualize o driver do adaptador para a versão ≥ v10.67 (2026) e, em seguida, reapliqueaplique as configurações do anel de recepção acima |
| A conexão é estabelecida, mas retorna resolução inferior à nativa | O Smart-prep reduziu automaticamente o quadro para se ajustar ao cabo | Atualize o link / aceite a redução / `--force-tier slip-emit-and-capture` |
| O array relata uma taxa de quadros por segundo (fps) alvo válida, mas entrega apenas uma fração dela; `health.gvsp_corrupt_rate_pct` diferente de zero, `never_arrived_rate_pct` 0 | O orçamento de banda do cabo inferido pelo host superestima o que ele realmente suporta (típico em um adaptador USB Ethernet, uma pista PCIe estreita ou uma malha compartilhada) | Reconecte-se com um `--wire-ceiling-mbps` mais baixo e verifique novamente o bloco de integridade. **Não** `--target-fps` — O ritmo do GevSCPD é fixado na conexão |
| Câmeras ausentes dos grupos publicados; `health.never_arrived_rate_pct` diferentezero, `gvsp_corrupt_rate_pct` 0 | Caminho de acionamento/sincronização — as câmeras não estão disparando, não é um problema de rede | Verifique o cabo de sincronização M8 e o `--line`; confirme se todos os membros estão armados (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` excedidos em `analyze-array`, ou recusa de conexão com resolução fixa (`array over-subscribes the wire`) | A demanda agregada por câmera (8 MB/× N câmeras) excede o limite máximo da conexão à prova de colisões — 6 câmeras em resolução total em 1 GbE com MTU de 1.500, 9 com quadros jumbo | Menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. **ROI/binning NÃO ajudará** (o limite máximo é). O `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` supera o limite em testes de bancada (aceita a perda de pacotes) |

---

## `chloros-cli daq`

Comandos do sensor espectral. Duas classes:
- **`pool-*`**— clientes HTTP “leves” que acionam o sensor por meio do pool persistente .**Este é o caminho suportado, e o único presente no CLI fornecido.** O backend é responsável pelo transporte, de modo que a GUI, os scripts CLI e SDK compartilham um único identificador ativo, em vez de disputarem pela porta serial.
- **Todo o restante**(`test`, `record`, `live`, `stream`, `connect`, `info`, `net`, `ota`, `sample-rate`, `calibrate`, `serve`, `ws`, `udp`, `mqtt`, `reflectance`, `login`, `logout`, `status`) — acesso direto ao hardware, documentado abaixo para fins de completude. Esses requerem o pacote `daq` Python, que**não está incluído em nenhum artefato fornecido**: o CLI compilado o exclui (o `scripts/Build-CLI.ps1` define o `--nofollow-import-to=daq`, e os transportes `pyserial` / `bleak` / `zeroconf` com ele), e o pacote SDK do PyPI também não o contém. Eles só funcionam a partir de um checkout do código-fonte, portanto, trate-os como um caminho de desenvolvimento interno do MAPIR, em vez do que como algo a ser utilizado.
- **`discover` / `list`** ficam no meio dos dois: são comandos diretos de hardware a partir de um checkout do código-fonte, mas em uma compilação pronta para uso eles recorrem ao `pool-discover` e o backend realiza a varredura. Assim, a varredura funciona em qualquer lugar — o que é importante, pois é a única maneira de descobrir o MAC BLE de um DAQ-M.

> **`chloros-cli daq --help`** (e `-h` / `help`) listam os subcomandos do `pool-*` — a ajuda é deliberadamente encaminhada para o cliente do pool, de modo a refletir os comandos que realmente são executados. Se você invocar um subcomando direto de hardware em uma versão distribuída, o programa será encerrado com um erro explícito indicando o pacote ausente e redirecionando você de volta para o `pool-*`; nada falha silenciosamente. (`discover` / `list` são a exceção — eles redirecionam para o `pool-discover` e simplesmente funcionam.)
>
> **Tudo o que um cliente precisa está acessível por meio do `pool-*`** — conectar, transmitir, gravar arquivos `.daq` calibrados e trocar perfis de capacitores. O DAQ também pode ser acionado a partir do Python com o `chloros_sdk.connect_daq_sensor()`, que usa o mesmo caminho compartilhado.

### Fluxo de trabalho da primeira conexão do sensor DAQ

```bash
# 1. Smart-detect any DAQ on this machine (Ethernet → BLE → USB precedence)
chloros-cli daq connect

# 2. Detailed scan: every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U COM port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq discover                      # or: daq pool-discover
chloros-cli daq discover --only ble           # BLE only
chloros-cli daq discover --json               # machine-readable

# 3. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect           # smart-detect
chloros-cli daq pool-connect --port COM3                       # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 4. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 5. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

# 6. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 7. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referência do `pool-*`

| Subcomando | Finalidade |
| --- | --- |
| `daq pool-connect` (smart-detect) | Abre um sensor no pool de back-end. |
| `daq pool-connect --port PORT` | DAQ-U em uma porta serial específica. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC-scanned. |
| `daq pool-connect --mac MAC` | DAQ-M via BLE em um MAC conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet em um host conhecido. |
| `daq pool-connect --eth` | DAQ-E via Ethernet, host descoberto automaticamente (mDNS + fallback ARP; funciona a partir de um cache ARP vazio em Windows e Linux). |
| `daq pool-connect --integration-time MS --frame-avg N --no-ae` | Ajustar janela de integração / estado AE. |
| `daq pool-connect --no-stream` | Conectar, mas não iniciar a transmissão ainda (retomar com `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de correção de Cap. O padrão no backend é `sunshine_cosine`. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Verifique cada transporte em busca de sensores aos quais você possa se conectar, sem estabelecer conexão. **É assim que você encontra o MAC BLE de um DAQ-M.** `daq discover` / `daq list` é direcionado automaticamente para cá nas compilações fornecidas. Os sensores já abertos no pool não são listados — um DAQ-M conectado deixa de anunciar —, portanto, use `pool-list` para esses casos. |
| `daq pool-list` | Mostrar todos os sensores no pool de back-end. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Quadros mais recentes do espectro N. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [--duration SEC] [--output DIR] [--device-name NAME] [--stop]` | Iniciar/parar uma gravação .daq. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de capacitância em tempo de execução. |

### Subcomandos de hardware direto (apenas na versão de código-fonte — não incluídos nas compilações distribuídas)

> Listados para fins de completude. Eles requerem o pacote `daq` Python, além de `pyserial` / `bleak` / `zeroconf`, nenhum dos quais está incluído na versão compilada CLI ou na versão do PyPI SDK — eles só são executados a partir de uma cópia do código-fonte MAPIR. **Se você estiver usando uma versão lançada do Chloros, utilize os comandos `pool-*` acima**; eles abrangem as opções connect, stream, record e seleção de cap.

```bash
chloros-cli daq test --port COM3                           # Verify connection
chloros-cli daq connect --eth                              # Smart-detect over ETH
chloros-cli daq info --eth-host daq-e-xxx.local            # Device summary as JSON
chloros-cli daq discover --only usb,ble --timeout 5        # Scan local interfaces
chloros-cli daq list                                       # Alias of discover
# ^ discover/list are the exception in this section: in a shipped build they
#   fall back to `pool-discover` (the backend does the scan), so they work
#   without a source checkout. The only difference is that the fallback needs
#   the Chloros backend running, as all pool-* commands do.

# Streaming JSON Lines to stdout (pipeable)
chloros-cli daq stream --port COM3 --format jsonl --photometrics

# Record to .daq for 60 seconds
chloros-cli daq record --port COM3 --duration 60 -o ~/Documents/spectra/

# Live spectrum visualization in a window
chloros-cli daq live --port COM3 --record

# Dual-sensor reflectance (ambient + object) → JSON Lines
chloros-cli daq reflectance \
  --ambient-eth-host daq-e-field.local \
  --object-eth-host daq-e-canopy.local \
  --record -o ~/Documents/reflectance/

# Convenience: pick integration_time + frame_avg for a target rate
chloros-cli daq sample-rate --port COM3 --target-hz 5

# Calibration profile management
chloros-cli daq calibrate --port COM3 --list
chloros-cli daq calibrate --port COM3 --set field_calibration_2026_05

# DAQ-E network config (mDNS auto-discovers the host)
chloros-cli daq net --eth-host daq-e-xxx.local set-ip --mode static --ip 192.168.2.20
chloros-cli daq net --eth-host daq-e-xxx.local set-name "sky-sensor"
chloros-cli daq net --eth-host daq-e-xxx.local set-ptp --enabled true --domain 0
chloros-cli daq net --eth-host daq-e-xxx.local set-auto-stream true          # auto-stream on boot
chloros-cli daq net --eth-host daq-e-xxx.local set-require-signature         # require factory-signed cal (fw v1.6.0+; refused while the held cal is unsigned)
chloros-cli daq net --eth-host daq-e-xxx.local set-time                      # push host clock (refused when PTP SLAVE)
chloros-cli daq net --eth-host daq-e-xxx.local set-auth-token --current "" --new "s3cret"   # control-channel auth ("" new = disable)
chloros-cli daq net --eth-host daq-e-xxx.local set-ota-password "newpass"    # change OTA password (min 4 chars)
chloros-cli daq net --eth-host daq-e-xxx.local factory-reset                 # clear all NVS settings and reboot
chloros-cli daq net --eth-host daq-e-xxx.local reboot

# OTA firmware update
chloros-cli daq ota --eth-host daq-e-xxx.local \
  --firmware daq_e_1.21.bin --password mapir-daq-e

# Bridge spectra to other protocols
chloros-cli daq serve --port COM3 --tcp-port 9000           # TCP JSON-lines
chloros-cli daq ws    --port COM3 --ws-port 9001            # WebSocket
chloros-cli daq udp   --port COM3 --udp-port 9002           # UDP broadcast
chloros-cli daq mqtt  --port COM3 --broker mqtt.example.com --topic daq/spectrum
```

---

## `chloros-cli project`

Abra, conecte-se e execute um projeto Chloros salvo (uma pasta com `cameras.json` + `sensors.json` + `project.json`). Tudo passa pelo backend, de modo que a GUI e o CLI produzem um estado de hardware idêntico.

### Referência de subcomandos

| Subcomando | Finalidade |
| --- | --- |
| `project open PATH` | Imprime o manifesto de dispositivos do projeto (câmeras, matrizes, sensores). |
| `project devices PATH [--reconnect]` | Listar ou reexecutar a descoberta. |
| `project connect PATH [--cameras-only] [--sensors-only]` | Conectar todas as câmeras/matrizes/sensores salvos. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Captura única de uma câmera ou matriz. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P]` | Série de N quadros de uma câmera ou matriz especificada (padrão de `-n/--count`: 5; `-i/--interval`: intervalo entre quadros em segundos, padrão: 0). As sequências de matrizes eliminam a duplicação de grupos sincronizados repetidos (vigilância de obsolescência), de modo que uma matriz com ciclo parcial não possa devolver N cópias de um quadro; exibe os resultados por iteração. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--poll-interval S]` | Transmissão para disco por meio de uma tarefa de back-end. `--poll-interval` = intervalo em segundos entre as sondagens `/stats` (padrão 2,0). |
| `project sensor read PATH NAME [--json]` | Quadro de espectro mais recente. |
| `project sensor log PATH NAME --seconds SEC [-o DIR] [--device-name NAME]` | Gravar .daq. |
| `project run PATH RECIPE.yaml` | Executar uma receita de captura YAML/JSON. `--dry-run` valida sem executar. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcula o alinhamento para uma matriz — consulte [a tabela de sinalizadores abaixo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Imprimir o perfil de alinhamento atual. |
| `project align clear PATH NAME` | Limpar o perfil armazenado em cache. |
| `project align tweak PATH NAME --serial SN --dx N --dy N --rotation-deg N --scale N` | Ajustar a transformação de um escravo. |
| `project align export PATH NAME --to FILE` | Salvar o perfil em JSON. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carregar um perfil salvo. |

#### Opções do `project align calibrate`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alinhamento. **Essas grafias diferem de `lattice align-calibrate`**, que aceita as formas abreviadas `orb` / `akaze` / `phase`; os dois comandos não são intercambiáveis neste sinalizador. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar o modelo para ajuste. |
| `--frames N` | `1` | Instantâneos de quadros sincronizados para calcular a média. |
| `--reference SN` | o mestre | Número de série da câmera de referência; todos os demais membros são distorcidos para se alinharem a ela. |
| `--max-features N` | `5000` | Limite de contagem de características ORB. |
| `--ratio-threshold F` | `0.75` | Teste da razão de Lowe. |
| `--ransac-threshold-px F` | `3.0` | Limite do . |
| `--min-matches N` | `15` | **Controle de qualidade** — rejeitar a resolução se o número de correspondências com pontos válidos for inferior a este valor. |
| `--max-reproj-err-px F` | `4.0` | **Limite de qualidade** — rejeitar a resolução acima desse erro de reprojeção RMS. |
| `--checkerboard RxC` | — | Geometria da placa para `--method checkerboard`, por exemplo, `9x6`. |
| `--name PROFILE` | vazio | Nome do perfil incorporado no JSON salvo. **Não é o nome da matriz** — esse é o `NAME`. |

Os dois controles de qualidade são a razão pela qual uma calibração pode ser bem-sucedida na resolução e ainda assim
ser recusada para salvamento: um perfil que falhe em qualquer um deles causaria, silenciosamente, um desalinhamento em todas as
capturas posteriores; por isso, ele é recusado em vez de ser persistido.

### Exemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL de receita

`project run RECIPE.yaml` aceita um arquivo YAML ou JSON que descreva uma sequência de ações:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        exposure_auto: "Off"
        gain: 6.0
        gain_auto: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ações suportadas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. A ação `burst` aceita `name` (obrigatório), `count` (padrão 5), `interval` (segundos, padrão 0), `output`, `format` e `settings` (igual àsconfigurações da câmera, como `apply`); as sequências de fotos em série usam o mesmo watchdog de grupo recém-sincronizado que o `project burst`.

Execute:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

---

## Variáveis de ambiente

| Variável | Efeito |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Substitui o backend URL (padrão: `http://127.0.0.1:5000`) — **respeitada apenas pelas famílias de comandos `lattice`, `project` e `daq pool-*`.** Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixam o `http://127.0.0.1:<port>` e ignoram essa variável (o literal IPv4 contorna a penalidade de Windows `localhost`→`::1` — penalidade de ~2 s por solicitação), de modo que sempre têm como alvo a máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | `1` rebaixa a matriz devidorecusa de conexão por subscrição (demanda agregada por câmera &gt; limite de segurança contra colisões na linha com o `pin_resolution`) para um aviso sonoro e continuação, aceitando perda de pacotes GVSP. Apenas para uso em bancos de testes — consulte [Modelo de fps e burst da matriz](#array-fps--burst-model). |
| `CHLOROS_CLI_MODE` | Definido pelo próprio CLI; instrui o backend a habilitar o processamento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignora a sonda de fallback do GVSP (apenas resultados de ICMP). **Isso desativa o jumbo, não apenas silencia o log** — a câmera responde a pings DF apenas até 1.500 em cada caminho, portanto, essa sonda é a única capaz de detectar pacotes jumbo. Economiza cerca de 1 s por câmera por conexão; custa cerca de 1,45× o limite da largura de banda se a rede *pudesse* ter transportado pacotes jumbo. O SDK avisa quando você define essa configuração. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixa o tamanho do pacote GVSP em N bytes; ignora completamente a sondagem. É preferível usar a configuração por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) em vez de defini-la permanentemente: um tamanho fixo impede a adaptação à rede à sua frente; além disso, fixar 9.000 em um caminho que não suporta pacotes jumbo faz com que **todos** capturas a expirarem com o `SC_ERR_TIMEOUT -1011`. |
| `TMPDIR` (Linux) | Substituir o diretório de extração do Nuitka onefile. O CLI usa automaticamente o `/mnt/ssd/tmp`, se estiver presente. |

---

## Códigos de saída

| Código | Significado |
| --- | --- |
| `0` | Sucesso. |
| `1` | Falha genérica (a maioria dos erros de subcomandos). |
| `2` | Erro de argumento. |
| `130` | Interrompido por Ctrl+C. |

---

## Dicas para resolução de problemas

- **“É necessário fazer login”** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina.
- **“backend inacessível&quot;** → Inicie o aplicativo de desktop Chloros, execute o binário do backend diretamente (`chloros-backend`) ou verifique o `CHLOROS_BACKEND_URL` se estiver em ambiente remoto.
- **Os comandos do `lattice` falham com a mensagem “Drivers da câmera LATTICE não encontrados”** → O runtime do Arena SDK não está instalado; o CLI vem com o `win32api` integrado no Windows, mas o runtime C faz parte do instalador da GUI.
- **“Array connect” / “Array Settings” exibe “FRAMES WILL DROP” ou “Reduza a ROI para habilitar”** → O anel de recepção da placa de rede do host é muito pequeno (geralmente reajustado para 32 após uma atualização do driver da placa de rede). Consulte [Configuração e ajuste da placa de rede do host](#host-nic-setup--tuning-lattice-arrays) — defina `ReceiveBufferLen=256`, `PendingReceives=64`.
- **A máquina trava ao reiniciar/desligar; em seguida, o WMI apresenta o código de erro `Invalid class` / a placa de rede não é ativada / unidades USB não são reconhecidas** → Driver desatualizado do adaptador USB 10 GbE causando o código de erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o driver do adaptador — consulte [Configuração e ajuste da NIC do host](#host-nic-setup--tuning-lattice-arrays).
- **Aviso de swap do Jetson** → Adicione um swap baseado em arquivo; o CLI exibe os comandos exatos `fallocate` / `swapon`.
- **Comandos diretos do DAQ ausentes** → Esperado: o `chloros-cli` fornecido exclui deliberadamente o pacote `daq`, portanto, apenas o `pool-*` está presente (o SDK do PyPI também não o inclui). Use o `pool-*`, que controla o mesmo sensor por meio do backend, ou o `chloros_sdk.connect_daq_sensor()` do Python.

---

## Veja também

- [Referência do Python e do SDK](sdk-reference.md) — equivalente programático de todos os comandos CLI.
- [Guia de sensores DAQ](../daq/README.md) — fiação e calibração específicas para cada sensor.
- Documentação online: `https://mapir.gitbook.io/chloros/cli`
