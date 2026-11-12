# Grade de imagens

Após importar imagens para um projeto, você as verá organizadas em uma grade na área principal. É nessa grade que você escolhe **qual versão de cada imagem está visualizando** — os botões acima dela alternam todas as miniaturas de uma só vez entre os arquivos de origem e cada produto processado.

## Tamanho das miniaturas

Use o controle deslizante de zoom no canto superior direito para ajustar o tamanho das miniaturas das imagens. O controle deslizante varia de **64 px a 1.200 px**.

* **Ctrl + roda do mouse** também redimensiona as miniaturas.
* **Ctrl + `+`**/**Ctrl + `=`**e**Ctrl + `−`** aumentam o tamanho em 4 px a cada pressionamento. O intervalo de tamanho definido pelo teclado vai de 64 px, no menor, até o tamanho que couber exatamente duas miniaturas por linha na janela atual, no maior.
* O tamanho escolhido é salvo com o projeto (`UI → Grid thumbnail size` em `project.json`, padrão `160`), portanto, ao reabrir o projeto, ele é restaurado.

<figure><img src="../.gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>A *resolução* da miniatura é uma configuração separada do *tamanho* da miniatura: consulte **Exibição → Resolução da miniatura da imagem** em [Configurações do projeto](../project-settings/project-settings.md) (padrão: 512 px na borda mais longa). O tamanho refere-se ao tamanho com que o bloco é desenhado; a resolução, ao nível de detalhe utilizado para preenchê-lo.

***

## A barra de ferramentas da grade

A fileira de botões acima da grade possui até três grupos, da esquerda para a direita:

1. **Por Gatilho / Por Câmera** — modo de agrupamento. Aparece apenas em projetos que contenham capturas do LATTICE.
2. **Botões de filtro de câmera** — um por câmera do LATTICE. Aparecem apenas no modo Por câmera.
3. **Botões de modo de exportação/visualização** — qual produto cada miniatura exibe.

Quando a janela é estreita demais para exibir todos eles, os grupos são recolhidos da direita para a esquerda, transformando-se em menus suspensos ao passar o cursor: os botões de exportação/visualização são recolhidos primeiro, seguidos pelos botões de câmera. O grupo recolhido deixa um único botão exibido, identificado com a opção ativa no momento, e ao passar o cursor sobre ele, o conjunto completo é exibido. **Os botões “Por Gatilho” e “Por Câmera” nunca se recolhem.

<!-- SCREENSHOT-NEEDED: Image grid toolbar of a LATTICE array project at full width, showing all three button groups inline: Per Trigger / Per Camera, three camera filter buttons labelled "LATT-M3M (serial)", and the export/view buttons including TIFF, RAW (Original), RAW (Radiance), RAW (Reflectance). -->

***

## Botões de exportação e visualização

Esses botões alternam as miniaturas da grade entre os tipos de imagem. **Um botão aparece assim que o produto ao qual se refere existe** — o que, para os arquivos de origem, significa imediatamente após a importação, e não após o processamento. O Chloros refaz a digitalização dos produtos do projeto enquanto uma execução está em andamento; portanto, os botões aparecem durante o processamento, à medida que cada produto começa a ser gravado no disco.

### O botão básico

O botão de exportação mais à esquerda é identificado com **o que você realmente importou**:

| O que você importou | Rótulo do botão |
| --- | --- |
| Survey3 RAW+JPG | `JPG` |
| Capturas do LATTICE com uma pré-visualização na tela ao lado do quadro RAW | `PNG` ou `TIFF`, dependendo de quais forem as pré-visualizações |
| Capturas do LATTICE em que o arquivo base **é** o quadro RAW | *sem botão* — o `RAW (Original)` já exibe esse arquivo |

Em um projeto misto, o rótulo segue a extensão mais utilizada pela maioria das imagens.

### Botões do produto

| Botão | Exibe | Quando aparece |
| --- | --- | --- |
| **Alvos** | Imagens com um alvo de calibração detectado | Após uma execução que detectou alvos |
| **Refletância** | Imagens de refletância calibradas | Apenas em projetos Survey3 — os projetos LATTICE usam `RAW (Reflectance)`, portanto a grade nunca exibe dois botões de refletância |
| **Equilíbrio de branco** | O produto com equilíbrio de branco (câmeras RGB) | Após o processamento |
| **Correção de vinheta** | A opção alternativa não calibrada com correção de vinheta | Após uma execução em que a calibração de refletância não pôde ser aplicada e a *correção de vinheta* estava ativada |
| **Resposta do sensor** | A opção alternativa de resposta do sensor não calibrada | O mesmo, mas com a *correção de vinheta* desativada |
| **`RAW (<INDEX> Index)`** | Um botão por índice calculado | Após uma execução com índices configurados |
| **`<INDEX> LUT`** | Um botão por índice mapeado por cor | Após uma execução com uma LUT configurada |
| **`<Index> <Index\|LUT> <NNN>`** | Um botão por execução de exportação da [Área de teste de índices/LUT](index-lut-sandbox.md) | No momento em que a exportação da área de teste é concluída |

### Botões de nível do LATTICE

Projetos que contêm capturas do LATTICE adicionam estes botões, rotulados com o nome do nível em vez do nome do produto:

| Botão | Nível |
| --- | --- |
| **RAW (Original)** | O quadro bruto de origem, conforme importado |
| **RAW (Radiação)** | Radiação espectral em Float32, W/m²/sr/nm |
| **RAW (Refletância)** | Refletância em uint16, 32768 = ρ 1,0 |

O `RAW (Original)` fica disponível assim que a importação é concluída — não requer nenhum processamento. Quando uma importação do LATTICE não possui nenhum botão de base (o arquivo de base de cada captura é seu quadro bruto), a grade se move automaticamente para o primeiro botão de nível disponível, de modo que o destaque na barra de ferramentas corresponda ao que você está vendo.

As exportações de dois níveis Chloros **não possuem botão de grade próprio**:

* **Sem debayering** — a visualização do `RAW (Original)` já é renderizada sem debayering; portanto, um segundo botão em uma imagem visualmente idêntica seria um ruído. O produto `RAW (Debayered)` ainda é gravado no disco e continua selecionável no menu suspenso de camadas em tela cheia.
* **Pré-visualização** — nas câmeras RGB, a pré-visualização é registrada como a camada `White Balanced`, que possui um botão. Em câmeras multiespectrais, ela é registrada como `RAW (Preview)` e pode ser acessada a partir do menu suspenso de camadas em tela cheia.

{% hint style="info" %}
Esses botões de nível só são exibidos para projetos que realmente contenham quadros LATTICE. Os projetos Survey3 registram alguns dos mesmos nomes de camadas internas, e os botões são filtrados para eles, de modo que uma grade Survey3 mantém seu conjunto familiar `JPG / Targets / Reflectance`.
{% endhint %}

Clicar na miniatura de uma grade abre o [Visualizador de Imagens](opening-an-image-full-screen.md) em tela cheia no **mesmo produto que a grade está exibindo** — se a grade estiver configurada como `Targets`, a miniatura abre a imagem de destino exportada.

<figure><img src="../.gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: This GIF predates the LATTICE level buttons and the toolbar group separators. Reshoot on a LATTICE project cycling base -> RAW (Original) -> RAW (Radiance) -> RAW (Reflectance) -> an index button, so the new button set and the level names are visible. -->

***

## Agrupamento de um projeto LATTICE: Por Gatilho vs. Por Câmera

As capturas em matriz produzem várias imagens do mesmo instante a partir de diferentes módulos de câmera. O agrupamento determina como a grade as organiza. Ambos os modos exibem barras de cabeçalho recolhíveis em largura total; **cada grupo começa expandido**, e o Chloros lembra aqueles que você fecha. O estado de recolhimento é rastreado separadamente por modo; portanto, fechar um grupo no modo “Por câmera” não fecha nada no modo “Por disparador”.

### Por câmera (padrão)

Um grupo por módulo de câmera. O cabeçalho mostra o modelo da câmera e o número de série (`LATT-M3M — <serial>`), além da contagem de fotos. Os blocos dentro de um grupo são ordenados cronologicamente por evento de captura.

Nesse modo, a barra de ferramentas também exibe um **botão de filtro de câmera por câmera**, identificado como `MODEL (SERIAL)`. Todas as câmeras aparecem selecionadas inicialmente; clicar em um botão desmarca essa câmera e remove seu grupo da grade. Essa é a maneira mais rápida de revisar uma faixa ao longo de todo um voo.

### Por Acionamento

Um grupo por evento de captura — o conjunto de quadros que todos os módulos capturaram no mesmo acionamento. O cabeçalho mostra a hora da captura, o número de câmeras que contribuíram e um ícone para cada modelo de câmera no grupo. Os blocos dentro de um grupo são ordenados pelo número de série da câmera, de modo que a mesma banda fica na mesma coluna para cada acionamento.

<!-- SCREENSHOT-NEEDED: Image grid in Per Trigger mode for a 3-camera LATTICE array, showing two consecutive trigger groups with their header bars (chevron, capture timestamp, "3 cameras", and the three model badges) and one group collapsed to show the closed state. -->
Imagens que não são do LATTICE em um projeto misto não são agrupadas — elas são exibidas como blocos simples após os grupos.

***

## As miniaturas da grade seguem o tamanho do bloco GSD

Se você definiu um tamanho de bloco **GSD (px)** na barra lateral da aba “Imagem”, as miniaturas em grade são exibidas com essa mesma resolução no solo — não apenas na visualização em tela cheia. Um tamanho de bloco de 8 significa que cada pixel exibido é a média de um bloco de 8 × 8 pixels de origem, em qualquer lugar do aplicativo que mostre a imagem.

Como um bloco tem apenas algumas centenas de pixels de largura, tamanhos de bloco grosseiros deixam de fazer diferença visível na grade bem antes de deixarem de fazer diferença na visualização em tela cheia: um quadro de 4.000 px desenhado em um bloco de 160 px já tem aproximadamente 25 pixels de origem por pixel exibido. Consulte [Abrindo uma imagem em tela cheia](opening-an-image-full-screen.md#gsd-block-size) para conhecer o controle em si.

***

## Páginas relacionadas

* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) — o visualizador em tela cheia, valores do cursor e histograma
* [**Camadas de imagem**](image-layers.md) — o menu suspenso de camadas dentro do visualizador em tela cheia
* [**Área de testes de índice/LUT**](index-lut-sandbox.md) — criação e exportação de visualizações de índice
* [**Configurações do projeto**](../project-settings/project-settings.md) — as opções de exportação que determinam quais produtos estão disponíveis
