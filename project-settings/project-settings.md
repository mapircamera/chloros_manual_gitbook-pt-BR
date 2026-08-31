# Configurações do projeto

A barra lateral “Configurações do projeto” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

) no Chloros permite que você configure todos os aspectos do processamento de imagens, detecção de alvos de calibração, cálculos de índices multiespectrais e opções de exportação para o seu projeto. Essas configurações são salvas junto com o seu projeto e podem ser guardadas como modelos para reutilização em vários projetos.

## Acessando as configurações do projeto

Para acessar as configurações do projeto:

1. Abra um projeto no Chloros
2. Clique na guia **Configurações do projeto**<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">

na barra lateral esquerda
3. O painel de configurações exibirá todas as opções de configuração disponíveis, organizadas por categoria

<!-- SCREENSHOT-NEEDED: Full Project Settings sidebar of a LATTICE project, scrolled so the Processing category is visible showing the per-product export checkboxes (Export sensor response, Export vignette corrected, Export debayered, Export preview, Export radiance, Export reflectance) and the Debayer method row. -->

{% hint style="info" %}
**As configurações que dependem de outras configurações aparecem desativadas (em cinza).** Quando uma opção principal impossibilita uma configuração (por exemplo, desmarcar *Calibração de refletância / equilíbrio de branco* impossibilita *Exportar refletância*), o controle dependente é desativado e sua dica de ferramenta indica a opção que precisa ser alterada.
{% endhint %}

***

## Exibição

### Resolução da miniatura da imagem

* **Tipo**: Seleção em menu suspenso
* **Opções**: `Default (512 px)`, `1024 px`, `2048 px`, `Full resolution`
* **Padrão**: Padrão (512 px)
* **Descrição**: Resolução (aresta mais longa, em pixels) na qual as miniaturas da grade de imagens são renderizadas. Valores mais altos parecem mais nítidos quando ampliados, mas demoram mais para carregar e consomem mais memória. A resolução total corresponde ao tamanho original da imagem.
* **Observação**: Apenas para exibição — isso nunca afeta o processamento nem os arquivos exportados.***

## Detecção de alvos

Essas configurações controlam como o Chloros detecta e processa alvos de calibração em suas imagens. Ambas ficam ativas apenas enquanto a **calibração de refletância / balanço de branco** estiver ativada (caso contrário, ficam desativadas, pois a detecção de alvos é totalmente ignorada).

### Área mínima de amostragem de calibração (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10.000 pixels
* **Padrão**: 25 pixels
* **Descrição**: Define a área mínima (em pixels) necessária para que uma região detectada seja considerada uma amostra válida de alvo de calibração. Valores menores detectarão alvos menores, mas podem aumentar os falsos positivos. Valores maiores exigem regiões-alvo maiores e mais nítidas para a detecção.
* **Quando ajustar**:
  * Aumente se estiver obtendo detecções falsas em pequenos artefatos da imagem
  * Diminua se seus alvos de calibração parecerem pequenos nas imagens e não estiverem sendo detectados

### Agrupamento mínimo de alvos (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Padrão**: 60
* **Descrição**: Controla o limite de agrupamento para regiões de cores semelhantes durante a detecção de alvos de calibração. Valores mais altos exigem que cores mais semelhantes sejam agrupadas, resultando em uma detecção de alvos mais conservadora. Valores mais baixos permitem maior variação de cores dentro de um grupo de alvos.
* **Quando ajustar**:
  * Aumente se os alvos de calibração estiverem sendo divididos em várias detecções
  * Diminua se os alvos de calibração com variação de cor não estiverem sendo totalmente detectados

***

## Processamento

Essas configurações controlam como o Chloros processa e calibra suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Aplica correção de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é um fenômeno óptico comum em que os cantos e as bordas de uma imagem parecem mais escuros do que o centro devido às características da lente.
* **Efeito colateral**: Essa opção também seleciona qual *produto alternativo não calibrado* uma execução gera (veja abaixo).

### Calibração de refletância / equilíbrio de branco

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Ativa a calibração de refletância — a partir de alvos de calibração detectados no quadro e/ou dados de luz incidente do sensor de luz do DAQ, dependendo da câmera e do que estiver disponível. Isso normaliza os valores de refletância em todo o seu conjunto de dados e garante medições consistentes, independentemente das condições de iluminação.
* **Quando desativado**: A detecção de alvos é totalmente ignorada, e**nenhum produto de refletância pode ser gerado por nenhuma câmera** — tanto no Survey3 orientado por alvos quanto no LATTICE orientado por DAQ. As configurações dependentes (*Exportar refletância*, *Intervalo mínimo de recalibração* e os limiares de detecção de alvo) ficam desativadas.

### Produtos alternativos não calibrados: Exportar resposta do sensor / Exportar com correção de vinheta

* **Tipo**: Duas caixas de seleção
* **Padrões**: Ambas ativadas (marcadas)
* **Descrição**: Quando um quadro não pode ser calibrado quanto à refletância (nenhum alvo de calibração foi encontrado ou a calibração de refletância está desativada), ele é gravado como um *produto alternativo não calibrado*. **Existe exatamente um dos dois produtos de fallback por execução, para cada modelo de câmera**, escolhido pelo botão *Correção de vinheta*:
  * Correção de vinheta **ativada**→ `Vignette_Corrected_Images/` (regida por**Exportar com correção de vinheta**)
  * Correção de vinheta **desativada**→ `Sensor_Response_Images/` (regida pela opção**Exportar resposta do sensor**)
* O produto alternativo que não está em uso aparece desativado (em cinza). Desmarcar o que está em uso impede que esse arquivo seja gravado.

### Produtos de exportação do LATTICE

Para projetos que contenham capturas do LATTICE, cada quadro importado do LATTICE é distribuído para todos os produtos habilitados **e aplicáveis**em uma única etapa de processamento. Quatro caixas de seleção controlam essa distribuição (todas**ativadas** por padrão):

| Configuração | Pasta de saída | O que é exportado |
| --- | --- | --- |
| **Exportar sem debayering** | `Debayered_Images/` | A imagem linear sem debayering. Aplica-se ao RGB e a câmeras multiespectrais. |
| **Exportar pré-visualização** | `Preview_Images/` | A pré-visualização exibida. RGB = equilíbrio de branco (iluminante DAQ, quando disponível; caso contrário, escala de cinza) + gama; multiespectral = alongamento de cores falsas. |
| **Radiação de exportação** | `Radiance_Images/` | Radiação espectral em Float32, em W/m²/sr/nm. Apenas multiespectral (M3C/M3M) — não se aplica aos masters RGB. Sempre gravado como TIFF de 32 bits, independentemente da configuração *Formato de imagem calibrado*. |
| **Refletância de exportação**| `Reflectance_Calibrated_Images/` | Refletância Uint16, dimensionada de modo que**32768 = refletância 1,0** (marcada como XMP `Chloros:PixelScale`). Apenas multiespectral, gravada quando um registro descendente correspondente `.daq` (ou um alvo no quadro aprovado no controle de qualidade) cobre o quadro. |

* As câmeras principais RGB emitem dados pós-desbayering + pré-visualização; a radianância/refletância são ignoradas para elas, por não serem aplicáveis.
* A profundidade de bits dos dados pós-desbayering/pré-visualização segue a configuração *Formato de imagem calibrada*; a radianância é sempre float32.
* O processamento do Survey3 não é afetado por essas quatro opções.

Essas mesmas quatro opções de ativação/desativação existem sem nome como `chloros-cli process --debayered / --preview / --radiance / --reflectance` e como os parâmetros correspondentes do SDK. Elas substituíram o antigo sinalizador `--radiometric-output`, que não existe mais.

{% hint style="warning" %}
**Desativar todos os produtos aplicáveis faz com que a execução falhe.** A partir da versão 1.2.0, uma execução de processamento que solicitou produtos, mas não gravou nenhum produto de imagem, reporta falha e o CLI sai com um valor diferente de zero, em vez de reportar um sucesso silencioso. O log indica o produto que não foi possível gravar e o motivo. Uma execução deliberadamente restrita a metadados (sem nada solicitado) ainda é considerada bem-sucedida.
{% endhint %}

### Fonte de refletância (configuração do projeto, definida via CLI/SDK)

O projeto também armazena qual **referência de refletância** o produto de refletância do LATTICE utiliza. Não há um controle específico no painel de configurações; o valor é armazenado na configuração do projeto como `Processing → "Target reflectance source"` e é definido com o parâmetro `chloros-cli process --reflectance-source {auto,target,daq}` ou o parâmetro SDK&#x27;parâmetro `reflectance_source`:

* **`auto`** (padrão): um alvo de calibração dentro do quadro que seja aprovado no controle de qualidade (QA) torna-se a referência absoluta, recorrendo à divisão de radiação descendente do DAQ (ρ = πL/E) quando não há alvo presente ou o controle de qualidade falha.
* **`target`**: refletância estritamente orientada pelo alvo — sem substituição pelo DAQ.
* **`daq`**: refletância determinada pelo DAQ; os alvos no quadro não são usados como referência.

O valor armazenado é comparado sem distinção entre maiúsculas e minúsculas, e algumas variações ortográficas são aceitas como sinônimos: `target`, `target_image`, `empirical` e `empirical_line` significam, todos, **alvo**; `daq`, `dls`, `light_sensor` e `sensor` significam, todos,**daq**. Qualquer outra coisa — incluindo uma chave ausente — resulta em**auto**.

As varreduras de alvo **medidas** por unidade são consultadas pelo número de série/QR da unidade-alvo, como `<serial>.csv`, em três locais: o diretório indicado por `--target-reflectance-dir` (armazenado como `Processing → "Target reflectance dir"`), na própria pasta `target_reflectance/` do projeto e no caminho da variável de ambiente `CHLOROS_TARGET_REFLECTANCE_DIR`. Quando não há nenhuma digitalização medida para essa unidade, utiliza-se, em vez disso, a curva nominal publicada para o modelo-alvo.

### Método de debayer

* **Tipo**: Seleção em menu suspenso
* **Opções**:
  * Padrão (Rápido, Qualidade Média)
  * Sensível à textura (Lento, Qualidade Máxima) \[Chloros+]
* **Padrão**: Padrão (rápido, qualidade média)
* **Descrição**: Seleciona o algoritmo de demosaicing usado para converter dados brutos do sensor com padrão Bayer em imagens coloridas. O método “Padrão (rápido, qualidade média)” oferece um equilíbrio ideal entre velocidade de processamento e qualidade de imagem. O método “Sensível à textura (lento, qualidade máxima)” \[Chloros+] utiliza um demosaicing de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML, que remove praticamente todo o ruído do processo de demosaicing. O modelo “Sensível à Textura” requer memória da GPU (VRAM) para ser executado. Recomendamos usá-lo quando você tiver mais de 4 GB de VRAM disponível para um processamento mais rápido.
* **Sempre que a linha for um menu suspenso**: o menu suspenso com duas opções aparece somente quando**ambas**as condições forem verdadeiras — você estiver conectado com uma assinatura qualificada do Chloros+,**e** o projeto não contiver capturas LATTICE. Caso contrário, a linha é exibida como texto simples com o texto “`Standard (Fast, Medium Quality)`”, sem nenhuma opção para escolha.
* **Observação sobre o LATTICE**: Não existe um modelo “Texture Aware” treinado para o LATTICE, e o pipeline força o demosaico padrão para quadros do LATTICE, independentemente do valor armazenado. Se você adicionar uma pasta LATTICE a um projeto que já tivesse a opção “Texture Aware” selecionada, Chloros reescreve a configuração de volta para “Padrão”, em vez de deixar um valor desatualizado em `project.json`.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3.600 segundos
* **Padrão**: 0 segundos
* **Descrição**: Define o intervalo de tempo mínimo (em segundos) entre o uso de alvos de calibração. Quando definido como 0, o Chloros utilizará todos os alvos de calibração detectados. Quando definido com um valor maior, o Chloros utilizará apenas alvos de calibração separados por pelo menos esse número de segundos, reduzindo o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de calibração.
* **Quando ajustar**:
  * Defina como 0 para obter a máxima precisão de calibração quando as condições de iluminação variam
  * Aumente (por exemplo, para 60 a 300 segundos) para um processamento mais rápido quando a iluminação for consistente e houver imagens frequentes de alvos de calibração

### Desvio de fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: Especifica o desvio de fuso horário (em horas em relação ao UTC) para os carimbos de data e hora dos dados do sensor de luz, usado ao correlacionar registros do sensor de luz com os horários de captura das imagens. As gravações mais recentes do `.daq` contêm sua própria proveniência de fuso horário; portanto, isso é necessário principalmente para registros mais antigos gravados no horário local.

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Padrão**: Desativado (desmarcado)
* **Descrição**: Ativa o uso de correções cinemáticas pós-processadas (PPK) de gravadores DAQ MAPIR que contêm um GPS (GNSS). Quando ativado, o Chloros utilizará quaisquer arquivos de log .daq contendo dados de pinos de exposição no diretório do seu projeto e aplicará correções precisas de geolocalização às suas imagens.
* **Requisito**: Deve haver um arquivo de log .daq com entradas de pino de exposição no diretório do seu projeto
* **Quando habilitar**: Recomenda-se habilitar sempre a correção PPK se houver entradas de feedback de exposição no seu arquivo de log .daq.

### Pino de exposição 1

* **Tipo**: Seleção em menu suspenso
* **Visibilidade**: Visível apenas quando “Aplicar correções PPK” estiver ativado E houver dados de exposição disponíveis para o Pino 1
* **Opções**:
  * Nomes de modelos de câmera detectados no projeto
  * “Não usar” – Ignora este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao Pino de exposição 1 para sincronização de tempo PPK. O pino de exposição registra o momento exato em que o obturador da câmera é acionado, o que é fundamental para uma geolocalização PPK precisa.
* **Comportamento da seleção automática**:
  * Uma câmera + um pino: Seleciona automaticamente a câmera
  * Uma câmera + dois pinos: O pino 1 é automaticamente atribuído à câmera
  * Várias câmeras: É necessária a seleção manual

### Pino de exposição 2

* **Tipo**: Seleção por menu suspenso
* **Visibilidade**: Visível apenas quando “Aplicar correções PPK” está ativado E há dados de exposição disponíveis para o Pino 2
* **Opções**:
  * Nomes de modelos de câmera detectados no projeto
  * “Não usar” — Ignora este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao Pino de exposição 2 para sincronização de tempo PPK ao usar uma configuração com duas câmeras.
* **Comportamento da seleção automática**:
  * Uma câmera + um pino: o Pino 2 é automaticamente definido como “Não usar”
  * Uma câmera + dois pinos: o Pino 2 é automaticamente definido como “Não usar”
  * Várias câmeras: é necessária a seleção manual
* **Observação**: a mesma câmera não pode ser atribuída simultaneamente ao Pino 1 e ao Pino 2.***

## Sensor de Luz DAQ

Esta seção aparece em “Configurações do Projeto” e lista todos os arquivos de luz descendente DAQ do projeto — gravações `.daq` e registros de luz descendente DAQ-M `.csv`. As gravações feitas na aba Sensores de Luz são adicionadas automaticamente ao projeto aberto.

<!-- SCREENSHOT-NEEDED: Project Settings "DAQ Light Sensor" section of a project containing at least one .daq file, showing the "Cap override (all files)" dropdown and a per-file row with its resolved cap. -->

Cada linha mostra o arquivo, o modelo do sensor e a correção da tampa difusora efetivamente em vigor para aquele arquivo. Acima das linhas há um único controle válido para todo o projeto:

### Substituição da tampa (todos os arquivos)

* **Tipo**: Seleção por menu suspenso
* **Opções**: `Auto` mais os perfis de correção da tampa válidos para os tipos de sensores presentes no projeto
* **Padrão**: Automático
* **Salvo como**: `Processing → "DAQ cap id"` (padrão `auto`)
* **Descrição**: `Auto` utiliza o cap registrado em cada arquivo (presume-se o cap de luz solar quando nada foi registrado — todos os DAQs MAPIR vêm com o corretor de luz solar). A escolha de uma tampa específica substitui**todos** os arquivos de radiação descendente no projeto: os registros brutos são corrigidos com ela, e os registros que já possuem uma tampa são re-referenciados (a correção registrada é desfeita e a selecionada é aplicada).
* **Importante**: A tampa selecionada deve corresponder à tampa que foi fisicamente instalada durante a gravação. Nem o sensor nem o software conseguem detectar a tampa física — um ID de tampa incompatível corrige os espectros de maneira incorreta.

Existe deliberadamente **um único** controle para todo o projeto, em vez de menus suspensos por arquivo: a configuração se aplica a todas as fontes de luz descendente no projeto.***

## Alinhamento de matriz

Esta seção aparece **apenas** quando pelo menos uma imagem no projeto contém a transformação de alinhamento módulo a módulo que as matrizes LATTICE aplicam no momento da captura (tags XMP `Chloros:Alignment*`). Ela mostra quantas imagens contêm tags de alinhamento, qual câmera é a de referência (identificador `REF`) e uma tabela com a contagem de imagens por câmera.

<!-- SCREENSHOT-NEEDED: Project Settings "Array Alignment" section for an imported LATTICE array capture set, showing the tagged-image count, the per-camera rows with the REF badge, and the three controls (Apply array alignment, Crop to common overlap, Resampling). -->

### Aplicar alinhamento de matriz

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Armazenado como**: `Processing → "Array alignment"`
* **Descrição**: Distorce cada produto processado (debayering / pré-visualização / radiação / refletância / índice) para a geometria de referência compartilhada da matriz, utilizando a transformação registrada no momento da captura. Desativado = exporta na geometria nativa de cada sensor.

### Recortar para sobreposição comum

* **Tipo**: Caixa de seleção (ativa apenas quando *Aplicar alinhamento de matriz* estiver ativado)
* **Padrão**: Ativado (marcado)
* **Armazenado como**: `Processing → "Array alignment crop"`
* **Descrição**: Recorta as exportações alinhadas para a região compartilhada por todos os módulos da câmera, de modo que cada banda tenha a mesma área de cobertura. Desativado mantém a tela completa do sensor (preenchimento preto fora da fonte).

### Reamostragem

* **Tipo**: Menu suspenso (ativo apenas quando *Aplicar alinhamento da matriz* estiver ativado)
* **Opções**: `Bilinear (smooth, default)`, `Nearest (preserve exact values)`, `Cubic (sharpest)`
* **Padrão**: Bilinear
* **Armazenado como**: `Processing → "Array alignment interpolation"`
* **Descrição**: Interpolação utilizada pela transformação de alinhamento. “Mais próximo” preserva os valores exatos da fonte (sem mistura entre pixels) para análise radiométrica rigorosa; “Bilinear” é ideal para mapeamento e uso visual.

As mesmas três opções existem sem nome como `chloros-cli process --array-alignment`, `--array-alignment-crop` e `--array-alignment-interp {bilinear,nearest,cubic}`.

***

## Índice

Essas configurações permitem definir índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel especial de configuração de índices
* **Descrição**: Abre um painel interativo onde é possível selecionar e configurar índices multiespectrais de vegetação (NDVI, NDRE, EVI, etc.) a serem calculados durante o processamento da imagem. É possível adicionar vários índices, cada um com suas próprias configurações de visualização.
* **Índices disponíveis**: O menu suspenso da interface gráfica inclui**27** fórmulas de índices multiespectrais predefinidas (consulte [Fórmulas de Índices Multiespectrais](multispectral-index-formulas.md) para obter a lista completa, incluindo os nomes que também são aceitos pela opção CLI/SDK `--indices`).
* **Recursos**:
  * Selecione entre fórmulas de índices predefinidas
  * Arraste os canais de filtro da sua câmera para os espaços de banda da fórmula
  * Configure gradientes de cor para visualização (LUT — Tabelas de Referência)
  * Defina valores de limiar e modos de recorte
  * Crie fórmulas de índices personalizadas
* **Observação**: Os índices não são calculados para câmeras monocromáticas LATTICE M3M de banda única — os índices multibanda são indefinidos em uma banda. O Survey3 e o LATTICE M3C não são afetados.

<!-- SCREENSHOT-NEEDED: Project Settings > Index section with one index added and expanded: the filter dropdown, the formula dropdown open showing preset names, the coloured channel circles above the rendered formula, and the "+ Add LUT" button below it. -->

Cada índice adicionado exibe sua fórmula como expressão matemática, com um círculo colorido por slot de banda: vermelho = Red, verde = Green, azul = Blue, laranja = Orange, ciano = Cyan, roxo = NIR, magenta = RE. Arraste um círculo da linha acima da fórmula para um slot para vinculá-la; clique duas vezes em um slot vinculado para desmarcá-lo. O índice só é calculado quando todos os slots usados pela fórmula tiverem um canal.

### Fórmulas personalizadas (recurso do Chloros+)

* **Tipo**: Matriz de definições de fórmulas personalizadas
* **Disponibilidade**: Requer login com uma assinatura qualificada do Chloros+.
* **Descrição**: Permite criar e salvar fórmulas personalizadas de índices multiespectrais usando operações matemáticas entre bandas. As fórmulas personalizadas são salvas com as configurações do seu projeto e podem ser usadas da mesma forma que os índices integrados.
* **Como criar**:
  1. No painel de configuração do Índice, abra a calculadora de fórmulas personalizadas
  2. Escreva a fórmula usando os **símbolos de slots de banda**, não os nomes das bandas
  3. Salve a fórmula com um nome descritivo — ela aparecerá na parte inferior do menu suspenso de fórmulas, e você arrastará os círculos de canal da sua câmera para os respectivos slots, exatamente como em uma predefinição integrada
* **Sintaxe da fórmula**:
  * Slots de banda: `x`, `y`, `z`, `a`, `b`, `c` — seis posições que você mapeia para canais reais arrastando
  * Operadores: `+`, `-`, `*`, `/`, `^` e `()` para agrupamento
  * Funções: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
* **Por que símbolos, e não nomes de bandas**: uma fórmula escrita como `(y-x)/(y+x)` funciona em qualquer câmera, pois o mapeamento de arrastar edecide se `y` é o NIR de 850 nm de um filtro RGN ou o NIR de um filtro OCN. As predefinições integradas são armazenadas da mesma forma — consulte [Fórmulas de Índice Multiespectral](multispectral-index-formulas.md) para ver a forma exata dos símbolos de todas as 27.
* **Onde funcionam**: as fórmulas personalizadas são salvas com as configurações do projeto e podem ser usadas na [Área de Testes de Índice/LUT](../image-viewer-gui/index-lut-sandbox.md), bem como no processamento. Elas**não** são aceitas pela lista de nomes CLI/SDK `--indices`, que apenas expande os 22 nomes de predefinições integradas.***

## Exportação

Essas configurações controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrada

* **Tipo**: Seleção no menu suspenso
* **Opções**:
  * **TIFF (16 bits)** - Formato TIFF de 16 bits não compactado
  * **TIFF (32 bits, porcentagem)** - TIFF de 32 bits com ponto flutuantecom valores de refletância expressos em porcentagem
  * **PNG (8 bits)** - Formato PNG de 8 bits compactado
  * **JPG (8 bits)** - Formato JPEG comprimido de 8 bits
* **Padrão**: TIFF (16 bits)
* **Descrição**: Seleciona o formato de arquivo para salvar imagens processadas e calibradas. As exportações são salvas em uma subpasta específica para cada formato dentro da pasta de cada câmera (`tiff16`, `tiff32`, `png8`, `jpg8`), com uma pasta `<Product>_Images/` por produto. Os arquivos exportados mantêm o nome do arquivo de origem — a pasta, e não um sufixo no nome do arquivo, identifica o produto.
* **Recomendações de formato**:
  * **TIFF (16 bits)**: Recomendado para análises científicas e fluxos de trabalho profissionais. Preserva a máxima qualidade dos dados, sem artefatos de compressão. Ideal para análise multiespectral e processamento posterior em softwares de SIG.
  * **TIFF (32 bits, porcentagem)**: Ideal para fluxos de trabalho que exigem valores de refletância expressos em porcentagem (0-100%). Oferece precisão máxima para medições radiométricas.
  * **PNG (8 bits)**: Adequado para visualização na web e visualização geral. Arquivos menores com compressão sem perdas, mas com faixa dinâmica reduzida.
  * **JPG (8 bits)**: Tamanhos de arquivo menores, ideal apenas para visualizações prévias e exibição na web. Utiliza compressão com perda, o que não é adequado para análises científicas.
* **Observação**: A radianância do LATTICE é sempre exportada como TIFF de 32 bits com precisão flutuante, independentemente desta configuração.***

## Salvar modelo de projeto

Este recurso permite que você salve as configurações atuais do seu projeto como um modelo reutilizável.

* **Tipo**: Campo de texto + botão “Salvar”
* **Descrição**: Digite um nome descritivo para o seu modelo de configurações e clique no ícone de salvar. O modelo armazenará todas as configurações atuais do seu projeto (detecção de alvos, opções de processamento, índices e formato de exportação) para fácil reutilização em projetos futuros. Os modelos são armazenados na pasta `Project Templates/` dentro da pasta de salvamento do seu projeto e também podem ser selecionados ou exportados a partir do menu principal (*Selecionar modelo* / *Salvar modelo* / *Exportar modelo*).
* **Casos de uso**:
  * Crie modelos para diferentes sistemas de câmera (RGB, multiespectral, NIR)
  * Salve configurações padrão para tipos específicos de culturas ou fluxos de trabalho de análise
  * Compartilhe configurações consistentes entre toda a equipe
* **Como usar**:
  1. Configure todas as definições desejadas para o projeto
  2. Digite um nome para o modelo (por exemplo, “RedEdge Survey3 NDVI Padrão”)
  3. Clique no ícone de salvar
  4. O modelo agora pode ser carregado ao criar novos projetos

***

## Pasta de salvamento do projeto

Esta configuração especifica onde os novos projetos são salvos por padrão.

* **Tipo**: Exibição do caminho do diretório + botão “Editar”
* **Padrão (Windows)**: `C:\Users\[Username]\Chloros Projects`
* **Padrão (Linux)**: `~/Chloros Projects`
* **Descrição**: Mostra o diretório padrão atual onde novos projetos Chloros são criados. Clique no ícone de edição para selecionar um diretório diferente. A substituição é armazenada como uma única linha de texto no `~/.chloros/working_directory.txt` — no Windows, que é o `C:\Users\<Username>\.chloros\working_directory.txt`. Se esse arquivo estiver ausente ou indicar um caminho que não existe mais, o Chloros recorre ao padrão acima. O CLI lê e grava o mesmo arquivo, portanto, o `chloros-cli` e a interface gráfica (GUI) sempre concordam sobre onde os projetos estão localizados.
* **Os modelos de projeto** ficam em uma subpasta `Project Templates/` deste diretório.
* **Quando alterar**:
  * Defina como uma unidade de rede para colaboração em equipe
  * Altere para uma unidade com mais espaço de armazenamento para conjuntos de dados grandes
  * Organize os projetos por ano, cliente ou tipo de projeto em pastas diferentes
* **Observação**: Alterar essa configuração afeta apenas os NOVOS projetos. Os projetos existentes permanecem em seus locais originais.***

## Persistência das configurações

Um projeto Chloros é uma **pasta**. Todas as configurações do projeto são salvas em `project.json` dentro dela; o hardware conectado é memorizado junto com elas em `cameras.json` e `sensors.json`; assim, ao reabrir um projeto, suas câmeras e sensores de luz também são reconectados. Ao reabrir um projeto, todas as configurações são restauradas exatamente como você as deixou. Os projetos salvos também podem ser executados sem interface gráfica usando o `chloros-cli project` ou o `open_project` do SDK.

### Hierarquia de configurações

As configurações são aplicadas na seguinte ordem:

1. **Padrões do sistema** — Padrões integrados definidos pelo Chloros
2. **Configurações do modelo** — Se você carregar um modelo ao criar um projeto
3. **Configurações do projeto salvo** - Configurações salvas com o arquivo do projeto
4. **Ajustes manuais** - Quaisquer alterações feitas durante a sessão atual

### Configurações e processamento de imagens

As configurações de processamento são lidas quando uma execução de processamento é iniciada. Alterar uma configuração não altera retroativamente os produtos que já estão no disco — execute o processamento novamente para aplicar as novas configurações. Algumas configurações nunca afetam o processamento:

* Resolução da miniatura da imagem (apenas para exibição)
* Salvar modelo do projeto
* Salvar pasta do projeto

***

## Referência de chaves de configuração

Para automação (CLI `--config`, SDK `configure` ou leitura direta de `project.json` diretamente), estas são as chaves exatas em `Project Settings`:

| Caminho da chave | Tipo | Padrão |
| --- | --- | --- |
| `Display → Image Thumbnail Resolution` | `"512" \| "1024" \| "2048" \| "full"` | `"512"` |
| `Target Detection → Minimum calibration sample area (px)` | número de 0 a 10.000 | `25` |
| `Target Detection → Minimum Target Clustering (0-100)` | número de 0-100 | `60` |
| `Processing → Vignette correction` | bool | `true` |
| `Processing → Reflectance calibration / white balance` | bool | `true` |
| `Processing → Export sensor response` | bool | `true` |
| `Processing → Export vignette corrected` | bool | `true` |
| `Processing → Export debayered` | bool | `true` |
| `Processing → Export preview` | bool | `true` |
| `Processing → Export radiance` | bool | `true` |
| `Processing → Export reflectance` | bool | `true` |
| `Processing → Array alignment` | bool | `true` |
| `Processing → Array alignment crop` | bool | `true` |
| `Processing → Array alignment interpolation` | `"Bilinear" \| "Nearest" \| "Cubic"` | `"Bilinear"` |
| `Processing → Debayer method` | `"Standard (Fast, Medium Quality)" \| "Texture Aware (Slow, Highest Quality)"` | Padrão |
| `Processing → Minimum recalibration interval` | número 0-3600 | `0` |
| `Processing → Light sensor timezone offset` | número -12..12 | `0` |
| `Processing → Apply PPK corrections` | booleano | `false` |
| `Processing → DAQ cap id` | ID do perfil de limite ou `"auto"` | `"auto"` |
| `Processing → Target reflectance source` | `"auto" \| "target" \| "daq"` | `"auto"` |
| `Index → Add index` | lista de configurações de índice | `[]` |
| `Export → Calibrated image format` | `"TIFF (16-bit)" \| "TIFF (32-bit, Percent)" \| "PNG (8-bit)" \| "JPG (8-bit)"` | `"TIFF (16-bit)"` |

As chaves `Array alignment` são gravadas na primeira vez que a seção “Alinhamento de matriz” é renderizada ou quando uma chamada de automação as define. Enquanto estiverem ausentes, o pipeline usa os mesmos valores mostrados acima (`true`, `true`, bilinear), de modo que um projeto.json sem elas se comporta exatamente da mesma forma que um com elas.

### Chaves armazenadas em `project.json` sem controle no painel de configurações

Elas estão localizadas na mesma árvore `Project Settings` e são lidas durante o processamento, mas você não encontrará um widget para elas na barra lateral:

| Caminho da chave | Tipo | Padrão | Definido por |
| --- | --- | --- | --- |
| `Processing → LATTICE input level` | `"auto" \| "raw" \| "debayered" \| "processed"` | `"auto"` | `chloros-cli process --input-level`, SDK `input_level=`. Substitui a forma como os arquivos TIFF de entrada do LATTICE são interpretados; `auto` infere a partir da tag XMP de cada arquivo `Chloros:ProcessingLevel` e do número de canais. Ignorado para capturas Survey3 `.raw`. Deliberadamente não é uma configuração da interface gráfica — a opção “auto” é correta em todos os casos normais. |
| `Processing → Target reflectance dir` | string de caminho | `""` | `chloros-cli process --target-reflectance-dir`, ou o destino do projeto API |
| `Processing → Target reflectance config` | dicionário indexado pelo número de série da câmera | `{}` | Registrando um alvo dentro do quadro (modo `fixed_block` / `fixed_strip` / `aruco`) |
| `Processing → DAQ-U log path` | string de caminho | `""` | SDK `process_folder(daq_log_path=…)`. Aponta para uma gravação `.daq` ou para uma pasta contendo essas gravações |
| `Target Detection → Minimum calibration target squares` | número | `4` | Padrão antigo; sem controle e sem sinalizador CLI |
| `UI → Grid thumbnail size` | número | `160` | O próprio controle deslizante de zoom das miniaturas da grade de imagens |

Duas preferências do visualizador são armazenadas **no nível superior em `project.json`**, totalmente fora do `Project Settings`, pois se tratam de estado de exibição e não de configurações de processamento:

| Caminho-chave | Tipo | Padrão | Definido por |
| --- | --- | --- | --- |
| `viewer_display → gsd_bin` | inteiro de 1 a 256 | `1` | Controle de GSD (px) da aba da imagem — consulte [Abrindo uma imagem em tela cheia](../image-viewer-gui/opening-an-image-full-screen.md) |

***

## Práticas recomendadas

1. **Comece com as configurações padrão**: As configurações padrão funcionam bem para a maioria dos sistemas de câmera MAPIR e fluxos de trabalho típicos.
2. **Crie modelos**: Depois de otimizar as configurações para um fluxo de trabalho ou câmera específicos, salve-as como um modelo para garantir a consistência entre os projetos.
3. **Teste antes do processamento completo**: Ao experimentar novas configurações, teste em um pequeno subconjunto de imagens antes de processar todo o seu conjunto de dados.
4. **Documente suas configurações**: Use nomes descritivos para os modelos, indicando o sistema de câmera, o tipo de processamento e a finalidade (por exemplo, “Survey3\_RGB\_NDVI\_Agricultura”).
5. **Seleção do formato de exportação**: Escolha seu formato de exportação com base no uso final:
   * Análise científica → TIFF (16 bits ou 32 bits)
   * Processamento GIS → TIFF (16 bits)
   * Visualização rápida → PNG (8 bits)
   * Compartilhamento na web → JPG (8 bits)

***

Para obter mais informações sobre índices multiespectrais no Chloros, consulte a página [Fórmulas de índices multiespectrais](multispectral-index-formulas.md).
