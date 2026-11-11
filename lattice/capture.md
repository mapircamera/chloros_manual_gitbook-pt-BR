# Configurações e modos de captura

A captura na aba “Câmeras” é controlada por um botão vermelho **Capturar tudo**e um painel**Configurações de captura**, que determina o que esse botão gera: quais câmeras participam, quais tipos de exportação cada câmera salva e se o obturador dispara uma vez, continuamente ou em intervalos. Esta página documenta todo o fluxo — a configuração, a captura em si, onde os arquivos são salvos no disco e como reprocessá-los posteriormente para obter produtos calibrados. Os controles da câmera e da matriz em si estão em [Configurações da câmera](camera-settings.md).

{% hint style="info" %}
**As capturas exigem um projeto aberto.** As opções “Capturar tudo” e a engrenagem de “Configurações de captura” ficam desativadas até que um projeto esteja aberto (“Crie ou abra um projeto para salvar as capturas”). Todas as capturas são salvas na pasta do projeto em `captures/`.
{% endhint %}

## O painel “Configurações de captura”

Abra-o com a **engrenagem ao lado de “Capturar tudo”**na lista de câmeras da barra lateral ou com o botão**“Abrir configurações de captura…”** na parte inferior de qualquer painel de configurações por câmera. O cabeçalho exibe “Configurações de captura” com um botão ← para voltar.

<!-- SCREENSHOT-NEEDED: the full Capture Settings pane — Single/Continuous/Interval mode buttons at top, the bulk export-type toggle rows (All Raw … All Index), the orange Fastest Capture toggle, an array group card with the Aligned checkbox and Record buttons, and an expanded per-camera row showing per-type checkboxes. -->



Suas seleções aqui — câmeras incluídas, caixas de seleção por tipo e o modo de captura — são salvas **por projeto** e restauradas quando você o reabre.

### Modos de captura

Três botões de modo na parte superior do painel:

| Modo | O que faz | Subconfigurações (padrões) |
| --- | --- | --- |
| **Única** *(padrão)* | Uma captura em todas as câmeras selecionadas. | — |
| **Contínua**| Capturas consecutivas até uma condição de parada. | Parar por**Número de capturas** (padrão 1) *ou* **Duração da captura** (padrão 10 s; unidades: segundos / minutos / horas / dias). |
| **Intervalo**(timelapse) | Série de capturas programadas por temporizador. |**Capturas / intervalo**(padrão 1) ·**A cada**N unidades (padrão 5 s) ·**Por** N unidades (padrão 1 m). |

Nos modos Contínuo ou Intervalo, o botão Capturar tudo se transforma em um botão **Parar (N)** durante a execução, contando as capturas à medida que são realizadas. Um drive cheio interrompe a execução (consulte [O fluxo “Capturar tudo”](#the-capture-all-flow)).**O modo Contínuo em matrizes é executado no backend (novidade na versão 1.3.0).** Quando todas as câmeras da lista pertencem a uma matriz (câmeras autônomas ocultas ou excluídas), o modo Contínuo não envia mais uma solicitação por captura: o backend do Chloros executa ele mesmo o ciclo de captura, gravando um grupo sincronizado enquanto seleciona o próximo, e o botão verifica sua contagem. Com câmeras autônomas na lista, o Chloros captura passo a passo, como antes.

{% hint style="warning" %}
**Captura rápida no Windows: mantenha o backend do Chloros excluído do Microsoft Defender.** A verificação em tempo real de cada arquivo gravado por uma captura de alta taxa consome cerca de 30% da taxa de captura nos testes do MAPIR (centenas de arquivos por segundo em uma pequena área de interesse). O instalador do Windows oferece a opção **“Excluir o backend do Chloros da verificação do Microsoft Defender (captura de câmera muito mais rápida; recomendado)”**, marcada por padrão, e remove a exclusão ao desinstalar. Se você desmarcou essa opção, execute o instalador novamente com ela marcada ou adicione uma exclusão de processo do Defender para `chloros-backend.exe` (na pasta `backend` da pasta de instalação do Chloros). Quando o Defender utiliza um núcleo da CPU ou mais durante a captura contínua de um conjunto de câmeras, o backend do Chloros registra isso uma vez em seu log, com essas mesmas duas soluções.
{% endhint %}

<!-- SCREENSHOT-NEEDED: the capture-mode area of Capture Settings with Interval selected — showing the "Captures / interval", "Every N (unit)" and "For N (unit)" rows with their defaults (1, 5 s, 1 m). -->




### Escolha de câmeras e tipos de exportação

O texto de ajuda do painel resume bem: escolha quais câmeras e tipos de exportação o “Capturar tudo” irá gerar — tudo está ativado por padrão, e as escolhas são salvas com este projeto.

* Os botões **Selecionar tudo / Desmarcar tudo** ativam ou desativam as caixas de seleção de inclusão de todas as câmeras de uma só vez.
* **Botões para alternar tipos de exportação em massa**(duas fileiras de botões):**Todos em Raw / Todos com debayering / Todos em pré-visualização / Todos em radiação / Todos em refletância / Todos em índice**. Cada um apresenta três estados de cor: verde ✓ = ativado para todas as câmeras que suportam esse tipo, âmbar – = ativado para algumas, cinza = nenhuma. Um botão fica desativado quando nenhuma câmera conectada suporta esse tipo. Todos ficam desativados (em cinza) enquanto a opção “Captura mais rápida” estiver ativada.
* **Linhas por câmera**: uma caixa de seleção de inclusão, além de uma lista expansível (▸/▾) dos tipos de exportação aplicáveis àquela câmera, com caixas de seleção individuais. A linha exibe uma contagem de ativação como “4/6”.

### Tipos de exportação e quais câmeras os suportam

Existem seis tipos de exportação: **Raw, Debayered, Radiance, Reflectance, Preview, Index**. Apenas os tipos aplicáveis aparecem na linha de cada câmera:

| Tipo de exportação | Conteúdo | RGB (FRGB) | Multiespectral Bayer (FRGN/FOCN/FNGB) | Mono (M3M) |
| --- | --- | --- | --- | --- |
| **Raw** | Mosaico Bayer (mono: a única banda) diretamente do sensor | ✓ | ✓ | ✓ |
| **Sem Bayer** | Desmosaico linear: o desvio de escuro (DSNU) do pacote de calibração removido do raw, seguido de um desmosaico sensível a bordas — sem correção de linearidade, campo plano ou separação de cores. Salvo como um RGB de 3 canais em formato TIFF (mono: escala de cinza de 1 canal) | ✓ | ✓ | ✓ |
| **Pré-visualização** | Cadeia de exibição completa (equilíbrio de branco + gama de acordo com o perfil da câmera, além da configuração de Destaques da câmera RGB; multiespectral: alongamento em cores falsas) | ✓ | ✓ | ✓ |
| **Radiação** | float32 W/m²/sr/nm por meio da cadeia radiométrica completa | — (não disponível) | ✓ | ✓ |
| **Refletância** | uint16 ρ (32768 = 1,0) | — (não disponível) | ✓ — exibido apenas quando a câmera possui um sensor de luz DAQ (próprio ou herdado de sua matriz) | igual ao multiespectral |
| **Índice** | Renderização do índice de vegetação (LUT) | — | ✓ — requer uma expressão de índice habilitada e não vazia na câmera, e não é oferecido a membros de matriz combinada (a matriz possui um índice compartilhado) | — (um índice requer ≥2 bandas; consulte [Câmeras Mono e Índices de Vegetação](mono-indices.md)) |

Radiação e refletância nunca são oferecidas para câmeras RGB — a radiação por Bayer não é significativa para um sensor fotométrico de banda larga.

### Captura mais rápida

O botão de alternância **⚡ Captura mais rápida — somente raw**(laranja quando ativado) substitui todas as seleções de exportação por**somente raw** — além de uma composição de índice combinado gratuita para matrizes — para que o quadro seja salvo o mais rápido possível: os cálculos de radianância/refletância/exibição são totalmente ignorados no momento da captura.

{% hint style="info" %}
**Um `.daq` ainda é salvo.** Quando um sensor de luz é atribuído, a Captura Mais Rápida ainda grava a leitura de radiação descendente do DAQ (na pasta `daq/` das capturas) — para que os produtos de radiação, refletância e índice possam ser gerados posteriormente por meio de um novo processamento (consulte [Reprocessamento de capturas](#re-processing-captures-into-calibrated-products)). O Fastest Capture também não altera suas seleções nas caixas de seleção: desative-o e elas serão restauradas.
{% endhint %}

### Controles por matriz

Cada matriz conectada possui seu próprio cartão de grupo no painel:

* **Caixa de seleção “Incluir”** (três opções para todos os membros) e o nome da matriz com seu modo de exibição: “(combinado | separado)”.
* **Caixa de seleção “Alinhado”**(padrão**ativada**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, de modo que as exportações fiquem registradas em pixels entre as câmeras. Os arquivos brutos permanecem sem distorção, mas carregam a transformação em seus metadados. (O próprio perfil é calculado no [painel de configurações da matriz](camera-settings.md#alignment-co-registration).) Os arquivos dos membros da matriz são salvos na orientação nativa de cada câmera, com qualquer configuração de Espelhamento/Rotação registrada no arquivo, em vez de aplicada.
* As linhas das câmeras membros estão aninhadas dentro do cartão.

O cartão da matriz também hospeda dois gravadores. Pense neles como **monitoramento x análise**:

| Gravador | Nível | O que grava |
| --- | --- | --- |
| **● Gravar vídeo de índice combinado / ■ Interromper gravação** *(somente matrizes combinadas)* | **Monitoramento** | A composição do índice combinado ao vivo em vídeo a 10 fps — 8 bits, resolução de pré-visualização, LUT incorporada. Requer um projeto aberto e uma visualização ao vivo em streaming. Exibe quadros + tempo decorrido durante a gravação. |
| **⦿ Gravar rajada bruta / ■ Interromper sequência bruta** *(qualquer matriz)* | **Análise**| Quadros Bayer brutos na taxa de captura ao vivo (sem processamento), além de um manifesto por quadro e leituras `.daq`, convertidos para `captures/bursts/`. Após uma sequência, um botão**Criar vídeo** aparece: ele reprocessa a sequência offline em vídeo calibrado — índice combinado e/ou radiação/refletância/índice por câmera — além de arquivos TIFF opcionais. A criação do índice combinado é iniciada automaticamente quando você interrompe a sequência. |

<!-- SCREENSHOT-NEEDED: an array group card in Capture Settings while a raw burst is recording — the ⦿/■ burst button in its recording state with frame count, and (in a second capture) the Build video button that appears after stopping. -->

## O fluxo

<!-- SCREENSHOT-NEEDED: the sidebar during a capture — Capture All showing live "Capturing… 3/6" progress text, and (second capture) the result flash "Saved N files". -->

“Capturar tudo”

Pressione **Capturar tudo** na lista de câmeras da barra lateral:

1. Todas as câmeras incluídas, visíveis e não pausadas capturam com os tipos de exportação selecionados. **Os conjuntos de câmeras disparam como um único gatilho sincronizado** (um único grupo sincronizado entre todos os membros — consulte [Conjuntos de múltiplas câmeras](arrays.md)); câmeras independentes capturam individualmente.
2. Câmeras ocultas (olho) ou em pausa são ignoradas. Um conjunto só é totalmente bloqueado quando *todos* os seus membros estão ocultos ou em pausa.
3. Sempre que um sensor de luz é atribuído, a leitura correspondente do DAQ de radiação descendente é salva como um arquivo `.daq` na pasta `daq/` das capturas — mesmo para capturas apenas em formato bruto — para que produtos radiométricos possam sempre ser derivados posteriormente.
4. O botão mostra o andamento em tempo real — “Capturando… concluído/total” — e, no modo Contínuo/Intervalo, passa a ser **Parar (N)**. Cada item de captura tem um tempo limite de 300 s.
5. Quando a passagem termina, um aviso de resultado exibe **“N arquivos salvos”**ou**“N salvos, F falhou”**, além de “(S ocultos/pausados/ignorados)” quando câmeras foram ignoradas. Se o disco ficar cheio, a execução é interrompida e exibe a mensagem**“Interrompido: disco cheio. Salvos N, F falharam”** — cada arquivo que não pôde ser gravado é contado como falha.

## Onde as capturas são salvas

As capturas são salvas no projeto aberto em `<project>/captures/`. Cada tipo de exportação é salvo em sua **própria subpasta**, de modo que uma captura em vários níveis nunca mistura tipos:

```
<project>/captures/
├── raw/           capture_<ts>_SN<serial>_raw.tif
├── debayered/     capture_<ts>_SN<serial>_debayered.tif
├── radiance/      capture_<ts>_SN<serial>_radiance.tif
├── reflectance/   capture_<ts>_SN<serial>_reflectance.tif
├── preview/       capture_<ts>_SN<serial>_display.tif
├── index/         per-camera vegetation-index (LUT) render, when Index is selected
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_clampmask.tif, reflectance/…, index/… — unmix clamp masks,
│                  one subfolder per product folder, only when a band was clamped
├── bursts/        raw-burst recordings (frames + manifest + .daq per burst)
└── daq/           the downwelling reading(s) matched to each capture (.daq), each with a
                   .daq.cameras.json naming the cameras it describes
```

* `<ts>` é o carimbo de data/hora da captura e `<serial>` é o número de série da câmera. Capturas independentes são nomeadas como `capture_<ts>_SN<serial>_<level>`; capturas em série a partir de um disparador sincronizado são nomeadas como `sync_<ts>_SN<serial>_<level>` e **compartilham um único carimbo de data e hora entre todas as câmeras do grupo** (o sufixo de nível é omitido quando uma câmera salva apenas um único nível).
* **Uma assimetria a ser observada:** o nível de exibição é armazenado em uma pasta chamada `preview/`, enquanto os arquivos mantêm `_display` no nome — a pasta e o sufixo diferem apenas para esse nível.
* Níveis desconhecidos são direcionados para uma pasta com seu próprio nome; se não for possível criar uma subpasta, o arquivo é gravado na raiz das capturas, em vez de ser perdido.
* Os arquivos TIFF de captura são compactados sem perda (DEFLATE) por padrão e contêm todos os metadados de calibração e processamento **dentro do XMP do arquivo** — as capturas são autodescritivas, sem arquivos complementares além da leitura `.daq` (e seu `.daq.cameras.json`).
* **Máscaras de limitação (novidade na versão 1.3.0).**Nos casos em que a separação espectral 3×3 precisava limitar uma banda a 0 em pelo menos um pixel, os produtos de radianância, refletância e índice recebem uma máscara por pixel, `clamp_masks/<product folder>/<product name>_clampmask.tif` (bit 0 = a banda no canal R do produto, bit 1 = G, bit 2 = B); a tag XMP do produto, `Chloros:UnmixClampMask`, aponta para ela. As máscaras ficam**ao lado** das pastas dos produtos, nunca dentro delas; assim, uma pasta de produto como `reflectance/` pode ser enviada ao software de fotogrametria tal como está. O próprio importador do Chloros ignora as máscaras.
* **Orientação (1.3.0).** Os produtos Preview, Radiance, Reflectance e Index de uma câmera autônoma são girados pela configuração Mirror/Rotation; os arquivos Raw e Debayered permanecem como foram lidos pelo sensor. Todos os arquivos membros da matriz permanecem em seu formato nativo. Cada arquivo registra a configuração em seu XMP, de modo que o reprocessamento a aplica da mesma forma.
* **A qual câmera um `.daq` pertence (1.3.0).** O `.daq` de uma captura é associado, no momento do processamento, apenas às imagens das câmeras cujos nomes constam em seu `.daq.cameras.json`. Uma câmera autônoma sem sensor de luz próprio, portanto, não utiliza mais a leitura de outra câmera — vincule um sensor a ela ou registre um na aba “Sensores de Luz”. Um registro de sensor de luz (e qualquer `.daq` sem esse arquivo, incluindo capturas mais antigas) ainda atende a todas as câmeras.

Esse é o mesmo layout que os arquivos `chloros-cli lattice capture` / `array-capture` gravam em seu diretório `-o` — documentado na [Referência da CLI § Como é uma pasta de capturas](../reference/cli-reference.md#what-a-captures-folder-looks-like).

<!-- SCREENSHOT-NEEDED: OS file explorer showing a real <project>/captures/ folder after a multi-level array capture — the raw/debayered/radiance/reflectance/preview subfolders, the daq/ and clamp_masks/ folders, and sync_<ts>_SN<serial>_<level>.tif filenames visible inside one subfolder. -->

## Reprocessamento de capturas em produtos calibrados

Os quadros brutos capturados, juntamente com o arquivo `.daq` salvo, são tudo o que o pipeline de processamento precisa — é por isso que o Fastest Capture é seguro para trabalhos reais.

* **GUI**: adicione a pasta de capturas a um projeto ([Adicionando arquivos a um projeto](../processing-images-gui/adding-files-to-a-project.md)) e processe normalmente.
* **CLI**: indique o `process` como **raiz das capturas**:

```bash
chloros-cli process "C:/ChlorosProjects/MyField/captures"
```

O `process` normalmente importa apenas a pasta que você especificar, mas quando essa pasta não contém imagens e possui subpastas, ele percorre automaticamente a hierarquia — assim, as subpastas de nível e a pasta `daq/` são capturadas de uma só vez. Cada captura é importada como uma **única imagem**, com seus outros níveis anexados como modos de visualização, e não como uma imagem por nível.

Nomear uma subpasta de nível diretamente (por exemplo, `…/captures/raw/`) também funciona, mas deixa a pasta `daq/` de fora — copie-a junto quando vocêderivar um produto radiométrico a partir de `raw/`; caso contrário, a correspondência de carimbo de data/hora não terá nada com que se alinhar.

{% hint style="warning" %}
**O processamento sempre começa a partir de `raw`.** Em cada captura, o quadro bruto é a fonte do pipeline; `debayered`, `radiance`, `reflectance`, e `preview` surgem como modos visualizáveis, mas nunca são realimentados pelo pipeline — reprocessar um produto derivado reaplicaria a vinheta, a cor e os cálculos de radiação já incorporados em seus pixels; por isso, o Chloros, em vez de realizar um processamento duplo, opta por recusar. As renderizações `index/` e `composite/` nunca são processadas (são saídas, não capturas). Uma pasta de capturas salva **sem** importações de dados brutos é exibida normalmente, mas o `process` a ignora e informa isso; o `--input-level {raw,debayered,processed}` é a saída de emergência deliberada que força um ponto de entrada. Consulte a [Referência da CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like) para ver as mensagens exatas de ignoração.
{% endhint %}

Mais dois comportamentos que vale a pena conhecer ao criar scripts de reprocessamento:

* Uma execução do `chloros-cli process` que solicitou produtos, mas **não gerou nenhum produto de imagem**, falha de forma evidente e sai com um valor diferente de zero** — você nunca terá uma execução vazia silenciosa. Execuções bem-sucedidas relatam suas contagens de produtos. (Uma execução deliberada apenas com metadados ainda conta como sucesso.)
* Exportações processadas e reimportadas nunca ocupam o slot de dados brutos de uma captura — os dados brutos originais sempre permanecem como fonte do pipeline.

## Equivalentes na CLI



Tudo nesta página pode ser executado sem interface gráfica. Os modos de captura da GUI correspondem diretamente ao `chloros-cli lattice array-capture`:

| GUI | CLI |
| --- | --- |
| Única | `chloros-cli lattice array-capture` |
| Contínua | `array-capture --continuous [--count N] [--duration S]` |
| Intervalo | `array-capture --interval S [--duration S]` |
| Captura mais rápida | `array-capture --fastest` |
| Caixa de seleção “Alinhado” | `--aligned / --no-aligned` |
| Caixas de seleção do tipo de exportação | `--processing LEVEL` ou `--levels L1,L2,…` (padrão: `all`) |
| Gravar vídeo de índice | `chloros-cli lattice array-record` |
| Gravar sequência bruta / Criar vídeo | `chloros-cli lattice array-burst` / `array-build-video` |

As tabelas completas de sinalizadores, a opção de captura estabilizada com AE inteligente (`--smart`) e o modelo de taxa sustentada estão descritos na [Referência da CLI: § Modos de captura, gravadores e reprocessamento offline](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).
