# Abrindo uma imagem em tela cheia

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>Uma imagem aberta em tela cheia, com o seletor de camadas no canto superior direito</p></figcaption></figure>

O Visualizador de Imagens Chloros é a interface em tela cheia para visualizar, inspecionar e medir suas imagens. É nele que você lê **valores reais de pixels** — DN por canal, porcentagem de refletância ou radiação em W/m²/sr/nm — em vez da pré-visualização distorcida exibida na tela.

## Acessando o Visualizador de Imagens

### A partir do Navegador de Arquivos

1. Abra a guia **Navegador de Arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Clique em qualquer **miniatura** na [grade de imagens](image-grid.md)
3. A imagem é aberta em tela cheia na guia **Visualizador de Imagens**

A imagem é aberta no produto que estava sendo exibido na grade. Se a grade estiver configurada para `RAW (Reflectance)`, essa será a camada em que você será direcionado.

### Abrindo a barra lateral do Visualizador de Imagens

Clique no ícone **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda para abrir o painel de análise. Ele contém, de cima para baixo:

* o nome da imagem e o modelo da câmera
* o botão **Exportar/Salvar Imagem(ns)** (somente quando um Índice ou LUT estiver ativo)
* as caixas de seleção **Índice**e**LUT** e o painel de configuração do índice — consulte [Área de Testes de Índice/LUT](index-lut-sandbox.md)
* o painel **Valores do Cursor**: leitura por canal, histograma da camada e o controle GSD***

## Navegação e zoom

### Navegar pelas imagens

* **Próxima imagem**: o botão → ou a tecla**→** (seta para a direita)
* **Imagem anterior**: o botão ← ou a tecla**←** (seta para a esquerda)
* **Ir para uma imagem específica**: volte à grade e clique na miniatura correspondente

O zoom e o deslocamento permanecem ativos enquanto você navega entre as imagens, permitindo percorrer um conjunto mantendo o foco na mesma parte do quadro.

### Zoom

O zoom é controlado pela **roda do mouse**, em incrementos de 15%, ancorado no cursor — o ponto sob o ponteiro permanece sob o ponteiro. O intervalo é limitado pelo tamanho da imagem e da janela: não é possível diminuir o zoom além do ajuste à janela, e o limite superior é definido pela resolução nativa da imagem.

Não há teclas dedicadas para zoom no visualizador em tela cheia. (Na grade, **Ctrl + `+` / `−`** redimensiona as miniaturas — um controle diferente.)

### Deslocamento com zoom

Clique e mantenha pressionado o botão esquerdo do mouse sobre a imagem e arraste. A panorâmica é limitada, de modo que a imagem não pode ser arrastada para fora da tela.

### Inspeção por pixel em zoom elevado

Quando a ampliação efetiva ultrapassa **60×**, o Chloros desenha uma caixa de destaque ao redor do pixel individual exibido sob o cursor e um valor flutuante ao lado dele.

A ampliação “efetiva” leva em conta o tamanho do bloco GSD: com um tamanho de bloco de 8, o destaque aparece no zoom de 7,5× em vez de 60×, pois um pixel exibido já corresponde a 8 × 8 pixels de origem. Ao reduzir o zoom abaixo do limite, o destaque desaparece.

### Atalhos de teclado

| Tecla                             | Onde       | Ação                              |
| ------------------------------- | ----------- | ----------------------------------- |
| **→**                           | Tela cheia | Próxima imagem                          |
| **←**                           | Tela cheia | Imagem anterior                      |
| **Ctrl + R**                    | Tela cheia | Redefinir o índice/área de teste LUT         |
| **Ctrl + `+`**/**Ctrl + `=`** | Grade        | Miniaturas maiores (4 px por pressionamento)  |
| **Ctrl + `−`**                  | Grade        | Miniaturas menores (4 px por pressionamento) |

***

## Valores do cursor

Passe o cursor sobre a imagem e o painel **Valores do cursor** exibirá o valor de cada canal abaixo dela.

{% hint style="success" %}
**Esses são os números reais do arquivo.** A tela na tela é uma pré-visualização esticada de 8 bits e não pode fornecê-los; portanto, o Chloros amostra o arquivo do produto real para a leitura. É por isso que um quadro bruto de 12 bits exibe valores acima de 255 e que uma camada de radiancia float32 exibe unidades físicas.
{% endhint %}

### O que significam as colunas

O painel se adapta à camada que você está visualizando:

| Camada que você está visualizando              | Colunas exibidas    | Observações                                                                                           |
| ---------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------- |
| Refletância                        | **DN**e**%** | A porcentagem é calculada com a escala própria do arquivo — veja abaixo                                      |
| Radiação                           | **W/m²/sr/nm**   | Valores físicos em float; não há coluna DN, pois um DN não tem significado aqui                           |
| Bruto / Desbayizado / pré-visualização / JPG    | **DN**           | Números digitais inteiros                                                                         |
| Exportações de refletância em porcentagem de 32 bits | Apenas **%**       | O valor de ponto flutuante armazenado não é um DN; portanto, arredondá-lo para um número inteiro resultaria na exibição de valores sem sentido como `0` ou `1` |

Cada linha é identificada com o nome do canal do filtro da sua câmera — `Red / Green / NIR` para RGN, `Orange / Cyan / NIR` para OCN, `NIR / Green / Blue` para NGB, `Red / Green / Blue` para RGB e o nome da banda única para as câmeras RE, NIR e câmeras mono M3M. Cada etiqueta possui um ponto colorido que corresponde aos círculos de canal utilizados no editor de fórmulas de índice.

As imagens **de índice e LUT** salvas são um caso especial: elas contêm componentes de mapa de cores em vez de bandas espectrais, portanto, suas linhas são rotuladas como `Red / Green / Blue` (ou `Index` para um arquivo de índice de canal único) em vez de com os nomes dos filtros da câmera.

Quando um índice está ativo na área de teste, uma linha extra aparece abaixo dos canais, mostrando o **valor do índice** na posição do cursor, com o nome do índice e um ponto branco que corresponde ao seu marcador no histograma.

### A porcentagem de refletância usa a escala própria de cada arquivo

{% hint style="warning" %}
**Não presuma que 65535 = 100%.** O Chloros armazena a refletância em diferentes escalas, dependendo da câmera que a produziu, e o visualizador determina a escala correta para cada arquivo.
{% endhint %}

| Origem                  | DN que equivale à refletância 1,0 | Como é identificado                                                                                                                               |
| ----------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| **LATTICE**(M3C / M3M) |**32768**                      | Tag XMP `Chloros:PixelScale=32768` gravada em toda exportação de refletância do LATTICE. A margem de 2× permite que o arquivo contenha ρ acima de 1,0 sem distorção |
| **Survey3**|**65535**                      | Sem tag de escala XMP Chloros — a calibração Survey3 grava ρ × dtype-max e clipa em 1,0                                                               |

O visualizador, a área de teste de índice/LUT e a exportação de índice resolvem a escala por meio da mesma implementação única; portanto, um valor lido no cursor é o mesmo valor utilizado nos cálculos do índice.

Duas consequências que vale a pena conhecer:

* Um TIFF de **32 bits em porcentagem**armazena DN/65535 como um float, e uma exportação**de 8 bits** do tipo PNG/JPG armazena DN × 255/65535 — o visualizador reconverte ambos antes de exibir uma porcentagem.
* Um caso não pode ser recuperado: uma **exportação TIFF de 8 bits de uma captura com fonte de 8 bits** é limitada a 0–255 em vez de ser redimensionada e, deliberadamente, não contém nenhuma tag de escala. Para esses arquivos, o painel imprime apenas o DN, sem a coluna de porcentagem. Essa é a resposta correta, não um bug.

***

## O histograma da camada

Abaixo das linhas do cursor, há um histograma em tempo real da camada que você está visualizando, em **256 intervalos**. Por padrão, ele traça uma curva combinada, ponderada `(R + 2G + B) / 4` — o mesmo espaço de medição usado pelos histogramas da câmera LATTICE. Ao ativar**RGB**, ele é substituído por curvas por canal nas cores dos canais, misturadas de forma aditiva para que as sobreposições permaneçam legíveis. Camadas monocromáticas sempre traçam a curva única.

O eixo horizontal está na unidade própria da camada:

| Camada       | Unidade do eixo  | Máximo do eixo                                               |
| ----------- | ---------- | ---------------------------------------------------------- |
| Refletância | porcentagem    | 125% — a margem do produto permite ρ acima de 1,0           |
| Radiação    | W/m²/sr/nm | O pico do próprio quadro, arredondado para dois algarismos significativos |
| Dados de 8 bits  | DN         | 255                                                        |
| Dados de 12 bits | DN         | 4095                                                       |
| Dados de 16 bits | DN         | 65535                                                      |

Quando o eixo está em DN e atinge um desses três limites máximos, o Chloros também reconhece a profundidade de bits do que você está visualizando.

Três botões estão localizados acima do histograma:

| Botão     | Padrão | Efeito                                                                                                                                                                                                                                                                                   |
| ---------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CURSOR** | Ativado | Traça linhas de referência no histograma nos valores exatos mostrados nas linhas acima, para que você possa ver onde o pixel sob o cursor se situa na distribuição do quadro. No modo RGB, há um marcador por canal em sua própria cor; caso contrário, um único marcador branco no valor combinado |
| **ÍNDICE**| Ativado      | Aparece apenas enquanto um índice estiver ativo. Alterna o histograma das bandas de origem para a**distribuição de valores de índice**, com os dois limites de clipping representados por linhas tracejadas laranja e o valor de índice do cursor como uma linha branca                                                          |
| **RGB**| Desativado     | Alterna da curva combinada para curvas por canal. Em um sensor monocromático, este botão exibe**MONO** e fica desativado — há apenas um canal para exibir                                                                                                                                  |

O histograma é calculado com base nos **blocos visíveis**, e não nos pixels de origem por trás deles: altere o tamanho do bloco GSD e a distribuição será recalculada, de modo que o histograma, o marcador do cursor e a imagem exibida estejam sempre em concordância.

***

## Tamanho do bloco GSD

Na parte inferior do painel está o controle **GSD (px)**: um campo numérico, um controle deslizante de**1 a 256**e um botão**RESET**.

Ele torna a imagem _exibida_ menos detalhada ao calcular a média de um bloco N × N de pixels de origem em um único pixel exibido. `1` é a resolução nativa.

* Isso afeta **a visualização em tela cheia, as miniaturas da grade, a exibição do cursor e ambos os histogramas** — tudo o que mostra a imagem utiliza a mesma resolução de base.
* Trata-se de **apenas exibição**. O processamento e a exportação não são afetados. A única exceção é deliberada: uma exportação via [Index/LUT Sandbox](index-lut-sandbox.md) salva o que você está visualizando, portanto mantém o tamanho de bloco atual, e o painel de exportação avisa quando o tamanho do bloco estiver acima de 1.
* O valor é armazenado **por projeto** como `viewer_display.gsd_bin` no `project.json`, portanto, ele é mantido mesmo após o fechamento e a reabertura do programa.
* A indicação do cursor mostra o bloco, e não o pixel de origem, sempre que o tamanho do bloco for maior que 1 — o valor exibido é a média do bloco sob o cursor.

{% hint style="info" %}
**Por que “tamanho do bloco” e não centímetros por pixel?** Um valor em cm/px requer uma altura acima do solo. Os dados EXIF de um único quadro contêm a altitude GPS acima do nível médio do mar, não acima do terreno para o qual a câmera estava apontada; portanto, o Chloros não imprimirá uma distância ao solo que não possa ser calculada com precisão. O tamanho do bloco em pixels de origem é o mesmo recurso alternativo que as ferramentas de nuvem do MAPIR utilizam quando a distância de amostragem ao solo é desconhecida.
{% endhint %}

***

## Tipos de imagem que você pode visualizar

O menu suspenso de camadas no canto superior direito do visualizador lista todas as versões da imagem atual. As entradas exibidas dependem da câmera e do que foi processado — consulte [Camadas de imagem](image-layers.md) para ver a lista completa e saber como o menu suspenso funciona.

### Survey3

* **JPG** — o arquivo de visualização da própria câmera
* **RAW (Original)** — o `.RAW` de origem, submetido a débayering para exibição, sem correções
* **RAW (Alvo)** — um quadro identificado como contendo um alvo de calibração
* **RAW (Refletância)** — o produto de refletância calibrado (65535 = ρ 1,0)
* **Correção de vinheta**/**Resposta do sensor** — o produto alternativo não calibrado
* **Equilíbrio de branco** — o produto com equilíbrio de branco
* **RAW (Índice `<INDEX>`)**e**LUT `<INDEX>`** — imagens de índice calculadas

### LATTICE

As capturas do LATTICE utilizam o mesmo menu suspenso, com os nomes dos níveis do pipeline:

| Camada                 | O que ela contém                                                        |
| --------------------- | -------------------------------------------------------------------- |
| **RAW (Original)**    | O quadro RAW de origem, conforme capturado                                     |
| **RAW (Debayered)**   | A imagem linear sem efeito de bayes                                           |
| **RAW (Pré-visualização)** | A pré-visualização na tela — alongamento em cores falsas para câmeras multiespectrais |
| **Equilíbrio de branco**    | A pré-visualização na tela para câmeras principais RGB (equilíbrio de branco + gama)   |
| **RAW (Radiação)**    | Radiação espectral em Float32 em W/m²/sr/nm                              |
| **RAW (Refletância)** | Refletância em uint16, 32768 = ρ 1,0                                    |

A radiancia e a refletância são exclusivas do modo multiespectral: uma câmera mestre RGB não possui radiometria por banda, portanto essas camadas não são geradas para ela.

***

## Índice e aplicação de LUT

Aplique índices multiespectrais e tabelas de consulta (LUTs) de cores a partir da barra lateral:

1. Abra o **Visualizador de Imagens** na barra lateral d<img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Marque **Índice**

3. Escolha o filtro da sua câmera e uma fórmula de índice; em seguida, arraste os círculos dos canais para os espaços da fórmula
4. Adicione uma LUT e escolha um gradiente, limiares e um modo de recorte
5. Verifique os valores no cursor e salve o resultado com **Exportar/Salvar imagem(ns)**Consulte [Área de testes de Índice/LUT](index-lut-sandbox.md) para o passo a passo completo.

***

## Solução de problemas

### A imagem não abre

**Possíveis causas**: o arquivo foi movido ou excluído após a importação; o produto nunca foi gravado; memória insuficiente para uma imagem muito grande.**O que fazer**:

1. Verifique se o arquivo da camada ainda existe na árvore de saída do projeto
2. Abra o arquivo em um visualizador externo para confirmar se está intacto
3. Feche outros aplicativos para liberar memória

### A imagem está preta, branca ou com cores distorcidas

**Possíveis causas**: o alongamento da tela não tem com o que trabalhar (um quadro quase constante); uma camada float32 com valores incomuns; um índice que não produziu dados válidos.**O que fazer**:

1. Verifique os valores do cursor — se todos os canais estiverem em zero ou próximos de zero, o problema está nos dados, não na exibição
2. Verifique o histograma: um único pico em uma das extremidades indica que o quadro está cortado ou vazio
3. Verifique o log de processamento da execução que gerou a camada

### Os valores parecem errados

**Possíveis causas**: você está em uma camada diferente da que imagina; está comparando uma porcentagem com um DN bruto; está comparando um arquivo LATTICE com um arquivo Survey3 usando o mesmo divisor.**O que fazer**:

1. Confirme a camada selecionada no menu suspenso — as unidades do painel seguem a camada
2. Para refletância, use a coluna **%** em vez de dividir o DN manualmente; se for necessário dividir, use o `Chloros:PixelScale` desse arquivo (32768 para o LATTICE; se estiver ausente, significa 65535 para o Survey3)
3. Defina o tamanho do bloco GSD de volta para 1 — acima de 1, você estará lendo uma média do bloco, não um pixel
4. Verifique se a calibração de refletância foi realmente executada para aquele quadro; um produto alternativo não calibrado (Resposta do Sensor / Corrigido para Vignette) não é refletância

***

## Próximos passos

* [**Camadas de imagem**](image-layers.md) — o nome de cada camada, quando existir, e o que seus valores significam
* [**Área de Testes de Índices/LUT**](index-lut-sandbox.md) — crie, ajuste e exporte visualizações de índices
* [**Marcadores de Mapa**](map-markers.md) — o mesmo conjunto de imagens em um mapa
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — a referência do índice

Para o fluxo de trabalho de processamento, consulte [Processamento de imagens (GUI)](../processing-images-gui/adding-files-to-a-project.md).
