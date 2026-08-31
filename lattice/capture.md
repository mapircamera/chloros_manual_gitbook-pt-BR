# Configurações e modos de captura

A captura na aba “Câmeras” é controlada por um botão vermelho **Capturar tudo**e um painel**Configurações de captura**, que determina o que esse botão gera: quais câmeras participam, quais tipos de exportação cada câmera salva e se o obturador dispara uma vez, continuamente ou em intervalos. Esta página documenta todo o fluxo — a configuração, a captura propriamente dita, onde os arquivos são salvos no disco e como reprocessá-los posteriormente para obter produtos calibrados. Os controles das câmeras e do conjunto de câmeras em si estão em [Configurações da Câmera](camera-settings.md).

{% hint style="info" %}
**As capturas exigem que um projeto esteja aberto.** As opções “Capturar tudo” e a engrenagem de “Configurações de captura” ficam desativadas até que um projeto seja aberto (“Crie ou abra um projeto para salvar as capturas”). Cada captura é salva na pasta do projeto em `captures/`.
{% endhint %}

## O painel “Configurações de captura”

Abra-o com a **engrenagem ao lado de “Capturar tudo”**na lista de câmeras da barra lateral ou com o botão**“Abrir configurações de captura…”** na parte inferior de qualquer painel de configurações específico de cada câmera. O cabeçalho exibe “Configurações de captura” com um botão ← para voltar.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->

Suas seleções aqui — câmeras incluídas, caixas de seleção por tipo e o modo de captura — são salvas **por projeto** e restauradas quando você o reabre.

### Modos de captura

Três botões de modo na parte superior do painel:

| Modo | O que faz | Subconfigurações (padrões) |
| --- | --- | --- |
| **Único** *(padrão)* | Uma captura em todas as câmeras selecionadas. | — |
| **Contínuo**| Capturas consecutivas até uma condição de parada. | Parar por**Número de capturas** (padrão 1) *ou* **Duração da captura** (padrão 10 s; unidades: segundos / minutos / horas / dias). |
| **Intervalo**(timelapse) | Série de capturas programadas por temporizador. |**Capturas / intervalo**(padrão 1) ·**A cada**N unidades (padrão 5 s) ·**Por** N unidades (padrão 1 m). |

Nos modos Contínuo ou Intervalo, o botão “Capturar tudo” se transforma em um botão **Parar (N)** durante a execução, contando as capturas à medida que são realizadas.

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->

### Escolha de câmeras e tipos de exportação

O texto de ajuda do painel resume tudo: escolha quais câmeras e tipos de exportação o “Capturar tudo” irá gerar — tudo está ativado por padrão, e as opções são salvas com este projeto.

* Os botões **Selecionar tudo / Desmarcar tudo** alternam as caixas de seleção de inclusão de todas as câmeras de uma só vez.
* **Botões para alternar tipos de exportação em massa**(duas fileiras de botões):**Todos em Raw / Todos com Debayering / Todos em Visualização / Todos em Radiação / Todos em Refletância / Todos em Índice**. Cada um apresenta três estados de cor: verde ✓ = ativado para todas as câmeras que suportam esse tipo, âmbar – = ativado para algumas, cinza = nenhum. Um botão fica desativado quando nenhuma câmera conectada suporta esse tipo. Todos ficam desativados enquanto a “Captura mais rápida” estiver ativada.
* **Linhas por câmera**: uma caixa de seleção “Incluir”, além de uma lista expansível (▸/▾) dos tipos de exportação aplicáveis àquela câmera, com caixas de seleção individuais. A linha exibe uma contagem de itens ativos, como “4/6”.

### Tipos de exportação e quais câmeras os suportam

Existem seis tipos de exportação: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Apenas os tipos aplicáveis aparecem na linha de cada câmera:

| Tipo de exportação | Conteúdo | RGB (FRGB) | Multiespectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico de Bayer (mono: banda única) diretamente do sensor | ✓ | ✓ | ✓ |
| **Sem Bayer** | Desmosaico linear (mono: escala de cinza de 1 canal) | ✓ | ✓ | ✓ |
| **Pré-visualização** | Cadeia de exibição completa (equilíbrio de branco + gama de acordo com o perfil da câmera; multiespectral: expansão em cores falsas) | ✓ | ✓ | ✓ |
| **Radiação** | float32 W/m²/sr/nm via cadeia radiométrica completa | — (não disponível) | ✓ | ✓ |
| **Refletância** | uint16 ρ (32768 = 1,0) | — (não disponível) | ✓ — exibido apenas quando a câmera possui um sensor de luz DAQ (próprio ou herdado de sua matriz) | igual ao multiespectral |
| **Índice** | Renderização do índice de vegetação (LUT) | — | ✓ — requer uma expressão de índice habilitada e não vazia na câmera, e não é oferecido a membros de matrizes combinadas (a matriz possui um índice compartilhado) | — (um índice requer ≥2 bandas; consulte [Câmeras Mono e Índices de Vegetação](mono-indices.md)) |

Radiação e refletância nunca são oferecidas para câmeras RGB — a radiação por Bayer não é significativa para um sensor fotométrico de banda larga.

### Captura mais rápida

O botão de alternância **⚡ Captura mais rápida — somente raw**(laranja quando ativado) substitui todas as seleções de exportação por**somente raw** — além de uma composição gratuita com índice combinado para matrizes — para que o quadro seja salvo o mais rápido possível: os cálculos de radiação/refletância/exibição são totalmente ignorados no momento da captura.

{% hint style="info" %}
**Um `.daq` ainda é salvo.** Quando um sensor de luz é atribuído, a Captura Mais Rápida ainda grava a leitura de radiação descendente do DAQ ao lado dos quadros em formato raw — assim, os produtos de radiação, refletância e índice podem ser gerados posteriormente por meio de um novo processamento (consulte [Reprocessamento de capturas](#re-processing-captures-into-calibrated-products)). O Fastest Capture também não altera suas seleções de caixas de seleção: desative-o e elas serão restauradas.
{% endhint %}

### Controles por matriz

Cada matriz conectada recebe seu próprio cartão de grupo no painel:

* **Caixa de seleção “Incluir”** (três estados para os membros) e o nome da matriz com seu modo de exibição: “(combinado | separado)”.
* **Caixa de seleção “Alinhado”**(padrão**ativado**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, de modo que as exportações sejam registradas em pixels entre as câmeras. Os arquivos raw permanecem sem warping, mas carregam a transformação em seus metadados. (O próprio perfil é calculado no [painel de configurações da matriz](camera-settings.md#alignment-co-registration-combined-only).)
* As linhas das câmeras que compõem a matriz estão aninhadas dentro do cartão.

O cartão da matriz também hospeda dois gravadores. Pense neles como **monitoramento x análise**:

| Gravador | Nível | O que grava |
| --- | --- | --- |
| **● Gravar vídeo de índice combinado / ■ Parar a gravação** *(somente matrizes combinadas)* | **Monitoramento** | A composição ao vivo do índice combinado em vídeo a 10 fps — 8 bits, resolução de pré-visualização, LUT incorporada. Requer um projeto aberto e uma visualização ao vivo por streaming. Mostra quadros + tempo decorrido durante a gravação. |
| **⦿ Gravar rajada bruta / ■ Interromper sequência bruta** *(qualquer matriz)* | **Análise**| Quadros Bayer brutos na taxa de captura ao vivo (sem processamento), além de um manifesto por quadro e leituras `.daq`, convertidos para `captures/bursts/`. Após uma sequência, um botão**Criar vídeo** aparece: ele reprocessa a sequência offline em vídeo calibrado — índice combinado e/ou radiação/refletância/índice por câmera — além de arquivos TIFF opcionais. A criação do índice combinado é iniciada automaticamente quando você interrompe a sequência. |##

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

O fluxo

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

“Capturar tudo” Clique em **Capturar tudo** na lista de câmeras da barra lateral:

1. Todas as câmeras incluídas, visíveis e que não estejam em pausa capturam com os tipos de exportação selecionados. **Os conjuntos de câmeras disparam como um único gatilho sincronizado** (um único grupo sincronizado entre todos os membros — consulte [Conjuntos de múltiplas câmeras](arrays.md)); câmeras independentes capturam individualmente.
2. Câmeras ocultas (olho) ou pausadas são ignoradas. Um conjunto só é totalmente bloqueado quando *todos* os seus membros estão ocultos ou pausados.
3. Sempre que um sensor de luz é atribuído, a leitura correspondente do DAQ (downwelling) é salva como um arquivo `.daq` junto com as imagens — mesmo para capturas apenas em formato bruto — para que produtos radiométricos possam sempre ser derivados posteriormente.
4. O botão mostra o andamento em tempo real — “Capturando… concluído/total” — e, no modo Contínuo/Intervalo, passa a ser **Parar (N)**. Cada item de captura tem um tempo limite de 300 s.
5. Quando a passagem termina, um aviso de resultado exibe **“N arquivos salvos”**ou**“N salvos, F falhou”**, além de “(S ocultos/pausados/ignorados)” quando câmeras foram ignoradas.

## Onde as capturas são salvas

As capturas são salvas no projeto aberto em `<project>/captures/`. Cada tipo de exportação é salvo em sua **própria subpasta**, de modo que uma captura em vários níveis nunca mistura os tipos:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── *.daq          the downwelling reading matched to the capture
```

* `<ts>` é o carimbo de data/hora da captura e `<serial>` é o número de série da câmera. Capturas independentes são nomeadas como `capture_<ts>_SN<serial>_<level>`; capturas em matriz de um único disparador sincronizado são nomeadas como `sync_<ts>_SN<serial>_<level>` e **compartilham um único carimbo de data e hora entre todas as câmeras do grupo** (o sufixo do nível é omitido quando uma câmera salva apenas um único nível).
* **Uma assimetria a ser observada:** o nível de exibição é armazenado em uma pasta chamada `preview/`, enquanto os arquivos mantêm `_display` no nome — a pasta e o sufixo diferem apenas para esse nível.
* Níveis desconhecidos são direcionados para uma pasta com seu próprio nome; se não for possível criar uma subpasta, o arquivo é gravado na raiz das capturas, em vez de ser perdido.
* Os arquivos TIFF de captura são compactados sem perda (DEFLATE) por padrão e contêm todos os metadados de calibração e processamento **dentro do XMP do arquivo** — as capturas são autodescritivas, sem arquivos complementares além da leitura `.daq`.

Esse é o mesmo layout que o `chloros-cli lattice capture` / `array-capture` grava em seu diretório `-o` — documentado na [Referência do CLI § Como é uma pasta de capturas](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, a .daq file at the root, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocessamento de capturas em produtos calibrados

Os quadros brutos capturados, juntamente com o `.daq` salvo, são tudo o que o pipeline de processamento precisa — é por isso que o Fastest Capture é seguro para trabalho real.

* **GUI**: adicione a pasta de capturas a um projeto ([Adicionando arquivos a um projeto](../processing-images-gui/adding-files-to-a-project.md)) e processe como de costume.
* **CLI**: aponte o `process` para a**raiz das capturas**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

O `process` normalmente importa apenas a pasta que você especificar, mas quando essa pasta não contém imagens e possui subpastas, ele percorre automaticamente a hierarquia — assim, as subpastas de cada nível e os arquivos da raiz `.daq` são selecionados de uma só vez. Cada captura é importada como uma **única imagem** com seus outros níveis anexados como modos visualizáveis, e não como uma imagem por nível.

Nomear diretamente uma subpasta de nível (por exemplo, `…/captures/raw/`) também funciona, mas deixa para trás os arquivos raiz `.daq` — copie-os junto quando você derivar novamente um produto radiométrico a partir de `raw/`; caso contrário, a correspondência de data e hora não terá nada com que se alinhar.

{% hint style="warning" %}
**O processamento sempre começa a partir de `raw`.**Dentro de cada captura, o quadro bruto é a fonte do pipeline; `debayered`, `radiance`, `reflectance`, e `preview` surgem como modos visualizáveis, mas nunca são realimentados pelo pipeline — o reprocessamento de um produto derivado reaplicaria a vinheta, a cor e os cálculos de radiação já incorporados em seus pixels, portanto, o Chloros é rejeitado em vez de passar por um processamento duplo. As renderizações `index/` e `composite/` nunca são processadas (são saídas, não capturas). Uma pasta de capturas salva**sem** importações de arquivos RAW é exibida normalmente, mas o `process` a ignora e informa isso; o `--input-level {raw,debayered,processed}` é a saída de emergência deliberada que força um ponto de entrada. Consulte a [Referência do CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) para ver as mensagens exatas de ignoramento.
{% endhint %}

Mais dois comportamentos que vale a pena conhecer ao criar scripts de reprocessamento:

* Uma execução do `chloros-cli process` que solicitou produtos, mas **não gerou nenhum produto de imagem**, falha de forma evidente e sai com um valor diferente de zero** — você nunca terá uma execução vazia silenciosa. Execuções bem-sucedidas relatam suas contagens de produtos. (Uma execução deliberada apenas com metadados ainda conta como sucesso.)
* Exportações processadas e reimportadas nunca ocupam o slot de dados brutos de uma captura — os dados brutos originais sempre permanecem como fonte do pipeline.

## Equivalentes do CLI

Tudo nesta página pode ser executado sem interface gráfica. Os modos de captura da GUI correspondem diretamente ao `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Único | `chloros-cli lattice array-capture` |
| Contínuo | `array-capture --continuous [--count N] [--duration S]` |
| Intervalo | `array-capture --interval S [--duration S]` |
| Captura mais rápida | `array-capture --fastest` |
| Caixa de seleção alinhada | `--aligned / --no-aligned` |
| Caixas de seleção do tipo de exportação | `--processing LEVEL` ou `--levels L1,L2,…` (padrão: `all`) |
| Gravar vídeo de índice | `chloros-cli lattice array-record` |
| Gravar sequência bruta / Criar vídeo | `chloros-cli lattice array-burst` / `array-build-video` |

As tabelas completas de sinalizadores, a opção de captura estabilizada com AE inteligente (`--smart`) e o modelo de taxa sustentada estão descritos na [Referência § Modos de Captura, Gravadores e Reprocessamento Offline de CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
