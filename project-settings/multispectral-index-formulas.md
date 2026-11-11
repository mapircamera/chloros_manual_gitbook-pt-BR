---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Fórmulas de índices multiespectrais

As fórmulas de índices abaixo utilizam uma combinação das faixas de transmissão média dos filtros do Survey3:

<table><thead><tr><th align="center">Cor do filtro Survey3</th><th width="196.199951171875" align="center">Nome do filtro Survey3</th><th width="159.800048828125" align="center">Faixa de transmissão (FWHM)</th><th align="center">Transmissão Média</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN- Cyan</td><td align="center">476–512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543–558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598–640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653–668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re-RedEdge</td><td align="center">712–735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798–848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835–865 nm</td><td align="center">850 nm</td></tr></tbody></table>

Quando essas fórmulas são utilizadas, o nome pode terminar em “\_1” ou “\_2”, o que corresponde ao filtro NIR utilizado, seja o NIR1 ou o NIR2.

Para as câmeras LATTICE M3C (triplamente passante de banda Bayer), o mesmo mecanismo de índice utiliza as faixas de filtro do M3C:

| Filtro M3C | Faixa 1 (centro/FWHM) | Faixa 2 (centro/FWHM) | Faixa 3 (centro/FWHM) |
| --- | --- | --- | --- |
| FRGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | Red 625 nm / 30 nm |
| FRGN | Red 660 nm / 21 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |
| FOCN | Orange 615 nm / 21 nm | Cyan 490 nm / 38 nm | NIR 808 nm / 14 nm |
| FNGB | Blue 475 nm / 30 nm | Green 550 nm / 30 nm | NIR 850 nm / 30 nm |

As câmeras LATTICE M3M são de banda única (um filtro de banda estreita por câmera); portanto, os índices multibanda não são calculados para uma imagem M3M isolada. Para calcular índices com a M3M, combine duas ou mais câmeras em uma pilha multibanda alinhada e utilize o mecanismo de índices do LATTICE (`chloros-cli lattice index` ou a Calculadora de Índices em tempo real da interface gráfica).

***

## Onde cada nome de índice funciona

O Chloros tem **três** superfícies de índice, e suas listas predefinidas não são idênticas. Use esta seção para verificar se um nome funcionará onde você planeja usá-lo.

| Onde você está | Qual lista se aplica | Contagem |
| --- | --- | --- |
| Configurações do projeto → Índice → Adicionar índice (GUI) | Superfície 1 | 27 |
| Visualizador de Imagens [Área de Testes de Índice/LUT](../image-viewer-gui/index-lut-sandbox.md) (GUI) | Superfície 1 | 27 |
| `chloros-cli process --indices NDVI,NDRE` | Superfície 2 | 22 |
| SDK `process_folder(indices=[...])` | Superfície 2 | 22 |
| `chloros-cli lattice index --preset` | Superfície 3 | 22 (um 22 diferente) |
| Calculadora de Índice em tempo real na aba Câmeras | Superfície 3 | 22 (um 22 diferente) |

O Surface 1 e o Surface 2 funcionam com **uma imagem por vez de uma câmera**, usando os slots de símbolos `x`/`y`/`z`(/`a`) vinculados aos canais de filtro dessa câmera. A superfície 3 trabalha com uma**pilha multibanda alinhada** — várias câmeras LATTICE co-registradas em um único cubo — e se refere aos canais pelo nome em letras minúsculas.

### 1. Configurações do projeto na GUI / Menu suspenso da área de teste do Visualizador de Imagens — 27 fórmulas

O menu suspenso as lista nesta ordem (é a ordem de inserção, não alfabética):

`NDVI, GNDVI, CVI, ENDVI, EVI, MSR, OSAVI, TDVI, LAI, FCI1, FCI2, GARI, GCI, GEMI, GLI, GOSAVI, GRVI, GSAVI, LCI, MNLI, MSAVI2, NDRE, NLI, RDVI, SAVI, VARI, WDRVI`

Na GUI, você arrasta os canais de filtro da sua câmera para os slots de banda da fórmula; assim, qualquer fórmula pode ser usada com qualquer atribuição de banda que sua câmera suporte. As fórmulas personalizadas que você salvou são anexadas abaixo dessa lista.

As **cinco fórmulas exclusivas da GUI** — aquelas que a lista da CLI/SDK `--indices` não aceita — são implementadas da seguinte forma:

| Predefinição exclusiva da GUI | Fórmula (conforme implementada) | Slots |
| --- | --- | --- |
| FCI1 | `x*y` | x, y |
| FCI2 | `x*y` | x, y |
| GARI | `(y-(x-1.7*(z-a)))/(y+(x-1.7*(z-a)))` | x, y, z, a (quatro slots) |
| GEMI | `((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5))*(1-0.25*((2*(y*y-x*x)+1.5*y+0.5*x)/(y+x+0.5)))-((x-0.125)/(1-x))` | x, y |
| LCI | `(y-x)/(y+z)` | x, y, z |

O mapeamento pretendido para cada um é apresentado em sua própria seção mais adiante nesta página (por exemplo, GARI espera x=Green, y=NIR, z=Blue, a=Red). GARI é a única fórmula no Chloros que utiliza um quarto slot.

### 2. Expansão de nomes do CLI/SDK `--indices` — 22 predefinições

A opção `chloros-cli process --indices` (e o parâmetro SDK `indices`) aceita estes nomes de predefinições:

`NDVI, GNDVI, NDRE, OSAVI, SAVI, MSAVI2, EVI, MSR, TDVI, LAI, GCI, GRVI, GSAVI, GOSAVI, NLI, MNLI, RDVI, WDRVI, CVI, ENDVI, GLI, VARI`

{% hint style="warning" %}
**Nomes de índices desconhecidos são ignorados silenciosamente.** Um nome fora desta lista (incluindo as cinco fórmulas exclusivas da GUI: `FCI1`, `FCI2`, `GARI`, `GEMI`, `LCI` e qualquer fórmula personalizada que você tenha salvo na GUI) é descartado com apenas um aviso no log — a execução continua sem esse índice, e a própria execução ainda é relatada como bem-sucedida. O aviso é exibido da seguinte forma:

```
[INDEX_EXPAND] skipping unknown preset 'LCI'; known: ['CVI', 'ENDVI', 'EVI', ...]
```

Os nomes são comparados sem distinção entre maiúsculas e minúsculas após a remoção de espaços em branco; portanto, `ndvi`, `NDVI` e ` NDVI ` são a mesma predefinição. Uma predefinição também é ignorada se precisar de uma faixa que o filtro da sua câmera não ofereça.
{% endhint %}

As fórmulas exatas conforme implementadas (os símbolos `x`/`y`/`z` representam slots de banda; o mapeamento padrão é mostrado por predefinição):

| Predefinição | Fórmula (conforme implementada) | Filtro padrão | Slots (x, y, z) |
| --- | --- | --- | --- |
| NDVI | `(y-x)/(y+x)` | RGN | Redro, NIR |
| GNDVI | `(y-x)/(y+x)` | RGN | Green, NIR |
| NDRE | `(y-x)/(y+x)` | RE | RE, NIR |
| OSAVI | `(y-x)/(y+x+0.16)` | RGN | Red, NIR |
| SAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Red, NIR |
| MSAVI2 | `(2*y+1-sqrt((2*y+1)*(2*y+1)-8*(y-x)))/2` | RGN | Red, NIR |
| EVI | `2.5*(y-x)/(y+6*x-7.5*z+1)` | RGN | Red, NIR, Blue |
| MSR | `((y/x)-1)/(sqrt(y/x)+1)` | RGN | Red, NIR |
| TDVI | `1.5*(y-x)/sqrt(y*y+x+0.5)` | RGN | Red, NIR |
| LAI | `3.618*(2.5*(y-x)/(y+6*x-7.5*z+1))-0.118` | RGN | Red, NIR, Blue |
| GCI | `(y/x)-1` | RGN | Green, NIR |
| GRVI | `y/x` | RGN | Green, NIR |
| GSAVI | `1.5*(y-x)/(y+x+0.5)` | RGN | Green, NIR |
| GOSAVI | `(y-x)/(y+x+0.16)` | RGN | Green, NIR |
| NLI | `((y*y)-x)/((y*y)+x)` | RGN | Red, NIR |
| MNLI | `((y*y-x)*(1+0.5))/((y*y)+x+0.5)` | RGN | Red, NIR |
| RDVI | `(y-x)/sqrt(y+x)` | RGN | Red, NIR |
| WDRVI | `(0.2*y-x)/(0.2*y+x)` | RGN | Red, NIR |
| CVI | `(z/y)*(x/y)` | RGN | Red, Green, NIR |
| ENDVI | `((x+y)-(2*z))/((x+y)+(2*z))` | NGB | NIR, Green, Blue |
| GLI | `((y-x)+(y-z))/((2*y)+x+z)` | RGB | Red, Green, Blue |
| VARI | `(y-x)/(y+x-z)` | RGB | Red, Green, Blue |

#### Como um nome de predefinição se transforma em posições de banda

Quando você passa um nome simples, como `NDVI`, o Chloros precisa decidir qual canal de qual arquivo cada símbolo lê. Ele usa esta tabela, que mapeia um código de filtro para a posição na matriz de cada canal:

| Código do filtro | Canal → índice da matriz |
| --- | --- |
| OCN | Orange 0, Cyan 1, NIR 2 (`Red` é aceito como um alias para Orange, também 0) |
| RGN | Red 0, Green 1, NIR 2 |
| NGB | NIR 0, Green 1, Blue 2 |
| RGB | Red 0, Green 1, Blue 2 |
| RE | RE 0 |
| NIR | NIR 0 |

O **filtro padrão** da predefinição (a coluna “Filtro padrão” acima) é usado quando o projeto contém imagens com esse filtro. Caso contrário, o Chloros analisa os filtros efetivamente presentes no projeto na ordem `RGN, OCN, NGB, RGB, RE, NIR` e seleciona o primeiro que possa fornecer todos os canais necessários à predefinição. Se nenhum for capaz disso, a predefinição é descartada para aquela execução. É por isso que o `NDVI` solicitado em um conjunto de dados apenas com OCN ainda produz um resultado razoável — ele se vincula às posições Orange e NIR do OCN.

As cadeias de caracteres do modelo LATTICE M3C contêm o filtro com o prefixo `F` (`LATT-M3C-L41-FRGN`), mas o prefixo é descartado quando o código do filtro é lido da imagem; assim, uma câmera FRGN resolve por meio da linha `RGN` acima e não requer tratamento especial.

### 3. Mecanismo de indexação LATTICE (`lattice index --preset`, Calculadora de Índice em tempo real) — 22 predefinições

O mecanismo LATTICE opera em pilhas multibanda alinhadas (matrizes em tempo real ou arquivos TIFF multibanda exportados) e utiliza nomes de canais em letras minúsculas (`red`, `green`, `blue`, `red_edge`, `nir`). Sua lista de predefinições difere das duas acima:

| Predefinição | Fórmula | Canais |
| --- | --- | --- |
| NDVI | `(nir - red) / (nir + red)` | vermelho, NIR |
| GNDVI | `(nir - green) / (nir + green)` | verde, infravermelho próximo |
| BNDVI | `(nir - blue) / (nir + blue)` | azul, infravermelho próximo |
| NDRE | `(nir - red_edge) / (nir + red_edge)` | borda do vermelho, infravermelho próximo |
| ENDVI | `((nir + green) - 2*blue) / ((nir + green) + 2*blue)` | azul, verde, infravermelho próximo |
| SAVI | `1.5 * (nir - red) / (nir + red + 0.5)` | vermelho, infravermelho próximo |
| OSAVI | `(nir - red) / (nir + red + 0.16)` | vermelho, NIR |
| MSAVI | `(2*nir + 1 - sqrt((2*nir + 1)**2 - 8*(nir - red))) / 2` | vermelho, NIR |
| EVI | `2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)` | azul, vermelho, NIR |
| EVI2 | `2.5 * (nir - red) / (nir + 2.4*red + 1)` | vermelho, infravermelho |
| CVI | `nir * red / green**2` | vermelho, verde, infravermelho |
| MSR | `((nir/red) - 1) / (sqrt(nir/red) + 1)` | vermelho, infravermelho |
| TDVI | `1.5 * (nir - red) / sqrt(nir**2 + red + 0.5)` | vermelho, infravermelho |
| LAI | `3.618 * (2.5 * (nir - red) / (nir + 6*red - 7.5*blue + 1)) - 0.118` | azul, vermelho, infravermelho |
| GLI | `(2*green - red - blue) / (2*green + red + blue)` | vermelho, verde, azul |
| NGRDI | `(green - red) / (green + red)` | vermelho, verde |
| VARI | `(green - red) / (green + red - blue)` | vermelho, verde, azul |
| TGI | `green - 0.39*red - 0.61*blue` | vermelho, verde, azul |
| EXG | `2*green - red - blue` | vermelho, verde, azul |
| CIRE | `(nir / red_edge) - 1` | vermelho\_borda, nir |
| CIGREEN | `(nir / green) - 1` | verde, nir |
| NDWI | `(green - nir) / (green + nir)` | verde, nir |

Execute o comando `chloros-cli lattice index --list-presets` para imprimir esta tabela a partir da sua versão instalada, e o `--list-gradients` para os gradientes de cor disponíveis. Os símbolos dos canais diferenciam maiúsculas de minúsculas e devem corresponder aos nomes em minúsculas da predefinição (por exemplo, `--channel red=Red_660 --channel nir=NIR_850`).

***

## CVI - Índice de Vegetação por Clorofila

Este índice é sensível ao teor de clorofila das folhas em uma ampla faixa de densidades de copa. Ele utiliza as bandas NIR, vermelha e verde.

$$
CVI = NIR * {Red \over Green^{2}}
$$

Na GUI e na lista de predefinições da CLI/SDK, a fórmula do símbolo é `(z/y)*(x/y)`, com x=Red, y=Green, z=NIR (bandas de uma câmera RGN); a predefinição do mecanismo de índices do LATTICE é `nir * red / green**2`. Todas as superfícies calculam o mesmo índice.

_Referência: Vincini, M., E. Frazzi e P. D&#x27;Alessio. “A Broad-Band Leaf Chlorophyll Vegetation Index at the Canopy Scale.” Precision Agriculture 9 (2008): 303-319._

***

## ENDVI - Índice de Vegetação por Diferença Normalizada Aprimorado

Este índice utiliza o canal azul, além do NIR e do verde, e é popular entre câmeras com filtro NGB, nas quais a banda azul substitui a vermelha.

$$
ENDVI = {(NIR + Green) - (2 * Blue) \over (NIR + Green) + (2 * Blue)}
$$

A implementação é a fórmula de símbolo `((x+y)-(2*z))/((x+y)+(2*z))` — atribua os canais NIR e Green da sua câmera aos slots x/y e o Blue ao z (para uma câmera NGB: x=NIR, y=Green, z=Blue).

***

## EVI - Índice de Vegetação Aprimorado

Este índice foi originalmente desenvolvido para uso com dados do MODIS como uma melhoria em relação ao NDVI, otimizando o sinal de vegetação em áreas de alto índice de área foliar (LAI). É mais útil em regiões com alto LAI, onde o NDVI pode saturar. Ele utiliza a região de refletância azul para corrigir sinais de fundo do solo e reduzir influências atmosféricas, incluindo a dispersão de aerossóis.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Os valores do EVI devem variar de 0 a 1 para pixels de vegetação. Elementos claros, como nuvens e edifícios brancos, juntamente com elementos escuros, como a água, podem resultar em valores anômalos de pixels em uma imagem EVI. Antes de criar uma imagem EVI, deve-se mascarar as nuvens e os elementos brilhantes da imagem de refletância e, opcionalmente, definir um limiar para os valores dos pixels entre 0 e 1.

_Referência: Huete, A., et al. “Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.” Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 — Índice de Cobertura Florestal 1

_Apenas na GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice distingue as copas florestais de outros tipos de vegetação utilizando imagens de refletância multiespectral que incluem uma faixa de borda vermelha.

$$
FCI1 = Red * RedEdge
$$

Áreas florestadas apresentarão valores mais baixos de FCI1 devido à menor refletância das árvores e à presença de sombras dentro da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. “Índices robustos de cobertura florestal para imagens multiespectrais.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 — Índice de Cobertura Florestal 2

_Apenas GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice distingue as copas florestais de outros tipos de vegetação utilizando imagens de refletância multiespectral que não incluem uma faixa de borda vermelha.

$$
FCI2 = Red * NIR
$$

Áreas florestadas apresentarão valores mais baixos de FCI2 devido à menor refletância das árvores e à presença de sombras dentro da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. “Índices robustos de cobertura florestal para imagens multiespectrais.” Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI — Índice de Monitoramento Ambiental Global

_Apenas GUI — não disponível como CLI/SDK na predefinição `--indices`._

Este índice de vegetação não linear é utilizado para o monitoramento ambiental global a partir de imagens de satélite e busca corrigir os efeitos atmosféricos. É semelhante ao NDVI, mas é menos sensível aos efeitos atmosféricos. É afetado pelo solo nu; portanto, não é recomendado para uso em áreas de vegetação esparsa ou moderadamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Onde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referência: Pinty, B., e M. Verstraete. GEMI: um índice não linear para monitorar a vegetação global a partir de satélites. Vegetation 101 (1992): 15-20._

***

## GARI — Índice Verde Resistente à Atmosfera (Green)

_Apenas GUI — não disponível como predefinição CLI/SDK `--indices`._

Este índice é mais sensível a uma ampla faixa de concentrações de clorofila e menos sensível aos efeitos atmosféricos do que o NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A constante gama é uma função de ponderação que depende das condições dos aerossóis na atmosfera. O ENVI utiliza um valor de 1,7, que é o valor recomendado por Gitelson, Kaufman e Merzylak (1996, página 296).

_Referência: Gitelson, A., Y. Kaufman e M. Merzylak. “Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.” Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI — Índice de Clorofila Green

Este índice é utilizado para estimar o teor de clorofila nas folhas em uma ampla variedade de espécies vegetais.

$$
GCI = {NIR \over Green} - 1
$$

A disponibilidade de comprimentos de onda amplos no NIR e no verde proporciona uma melhor previsão do teor de clorofila, ao mesmo tempo em que permite maior sensibilidade e uma relação sinal-ruído mais elevada.

_Referência: Gitelson, A., Y. Gritz e M. Merzlyak. “Relações entre o teor de clorofila foliar e a refletância espectral e algoritmos para avaliação não destrutiva da clorofila em folhas de plantas superiores.” Journal of Plant Physiology 160 (2003): 271-282._

***

## GLI — Índice de Folha Green

Esse índice foi originalmente projetado para uso com uma câmera digital RGB para medir a cobertura de trigo, onde os números digitais (DNs) de vermelho, verde e azul variam de 0 a 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Os valores do GLI variam de -1 a +1. Valores negativos representam o solo e características não vivas, enquanto valores positivos representam folhas e caules verdes.

_Referência: Louhaichi, M., M. Borman e D. Johnson. “Plataforma com localização espacial e fotografia aérea para documentação dos impactos do pastoreio no trigo.” Geocarto International 16, n.º 1 (2001): 65-70._

***

## GNDVI — Índice de Vegetação de Diferença Normalizada Green

Este índice é semelhante ao NDVI, exceto pelo fato de medir o espectro verde de 540 a 570 nm, em vez do espectro vermelho. Este índice é mais sensível à concentração de clorofila do que o NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referência: Gitelson, A., e M. Merzlyak. “Sensoriamento remoto da concentração de clorofila nas folhas de plantas superiores.” Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Índice de Vegetação Otimizado para o Verde (Green) Ajustado ao Solo

Este índice foi originalmente desenvolvido com fotografia infravermelha colorida para prever as necessidades de nitrogênio do milho. É semelhante ao OSAVI, mas substitui a banda vermelha pela banda verde.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referência: Sripada, R., et al. “Determinação das necessidades de nitrogênio durante a safra do milho por meio de fotografia aérea em infravermelho colorido”. Tese de doutorado, Universidade Estadual da Carolina do Norte, 2005._

***

## GRVI — Índice de Vegetação da Razão do Verde (Green Ratio Vegetation Index)

Esse índice é sensível às taxas de fotossíntese nas copas das florestas, pois as refletâncias do verde e do vermelho são fortemente influenciadas pelas mudanças nos pigmentos foliares.

$$
GRVI = {NIR \over Green }
$$

_Referência: Sripada, R., et al. “Fotografia aérea em infravermelho colorido para determinar as necessidades de nitrogênio no início da estação para o milho.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Índice de Vegetação Ajustado ao Solo Green

Este índice foi originalmente desenvolvido com fotografia em infravermelho colorido para prever as necessidades de nitrogênio do milho. É semelhante ao SAVI, mas substitui a banda vermelha pela verde.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referência: Sripada, R., et al. “Determinação das necessidades de nitrogênio do milho durante a safra por meio de fotografia aérea em infravermelho colorido.” Tese de doutorado, Universidade Estadual da Carolina do Norte, 2005._

***

## LAI – Índice de Área Folhosa

Este índice é utilizado para estimar a cobertura foliar e prever o crescimento e o rendimento da cultura. O ENVI calcula o LAI verde utilizando a seguinte fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Onde EVI é:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valores elevados de LAI variam normalmente entre aproximadamente 0 e 3,5. No entanto, quando a cena contém nuvens e outros elementos brilhantes que produzem pixels saturados, os valores de LAI podem exceder 3,5. O ideal é mascarar as nuvens e os elementos brilhantes da cena antes de criar uma imagem de LAI.

_Referência: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. “Dados multiespectrais aéreos para quantificação do Índice de Área Folhar, concentração de nitrogênio e eficiência fotossintética na agricultura.” Remote Sensing of Environment 81, n.º 2-3 (2002): 179-193._

***

## LCI — Índice de Clorofila Folhar

_Apenas GUI — não disponível como CLI/SDK na predefinição `--indices`._

Este índice é utilizado para estimar o teor de clorofila em plantas superiores, sendo sensível à variação na refletância causada pela absorção da clorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referência: Datt, B. “Sensoriamento remoto do teor de água em folhas de eucalipto.” Journal of Plant Physiology 154, n.º 1 (1999): 30-36._

***

## MNLI - Índice Não Linear Modificado

Este índice é um aprimoramento do Índice Não Linear (NLI) que incorpora o Índice de Vegetação Ajustado ao Solo (SAVI) para levar em conta o fundo do solo. O ENVI utiliza um valor de 0,5 para o fator de ajuste de fundo da copa (_L_).

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referência: Yang, Z., P. Willis e R. Mueller. “Impacto da imagem AWIFS aprimorada por relação de bandas na precisão da classificação de culturas.” Anais do Simpósio de Sensoriamento Remoto Pecora 17 (2008), Denver, CO._

***

## MSAVI2 – Índice de Vegetação Ajustado ao Solo Modificado 2

Este índice é uma versão mais simples do índice MSAVI proposto por Qi et al. (1994), que aprimora o Índice de Vegetação Ajustado ao Solo (SAVI). Ele reduz o ruído do solo e aumenta a faixa dinâmica do sinal de vegetação. O MSAVI2 baseia-se em um método indutivo que não utiliza um valor _L_ constante (como no SAVI) para destacar a vegetação saudável.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referência: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. “A Modified Soil Adjusted Vegetation Index.” Remote Sensing of Environment 48 (1994): 119-126._

***

## MSR – Razão Simples Modificada

Este índice é uma modificação da razão simples NIR/Red, projetada para linearizar sua relação com parâmetros biofísicos, e é mais sensível do que o NDVI em densidades de vegetação mais elevadas.

$$
MSR = {(NIR / Red) - 1 \over \sqrt{NIR / Red} + 1}
$$

_Referência: Chen, J. “Avaliação de índices de vegetação e de uma razão simples modificada para aplicações boreais.” Canadian Journal of Remote Sensing 22 (1996): 229-242._

***

## NDRE – Diferença Normalizada RedEdge

Este índice é semelhante ao NDVI, mas compara o contraste entre o NIR e o RedEdge, em vez do Red, o que frequentemente detecta o estresse da vegetação mais cedo.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI – Índice de Vegetação por Diferença Normalizada

Este índice é uma medida de vegetação verde e saudável. A combinação de sua formulação de diferença normalizada com o uso das regiões de maior absorção e refletância da clorofila o torna robusto em uma ampla gama de condições. Ele pode, no entanto, saturar em condições de vegetação densa, quando o LAI se torna elevado.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

O valor desse índice varia de -1 a 1. A faixa comum para vegetação verde é de 0,2 a 0,8.

_Referência: Rouse, J., R. Haas, J. Schell e D. Deering. Monitoramento de sistemas de vegetação nas Grandes Planícies com o ERTS. Terceiro Simpósio do ERTS, NASA (1973): 309-317._

***

## NLI - Índice Não Linear

Este índice pressupõe que a relação entre muitos índices de vegetação e parâmetros biofísicos da superfície é não linear. Ele lineariza as relações com parâmetros da superfície que tendem a ser não lineares.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referência: Goel, N., e W. Qin. “Influências da arquitetura do dossel nas relações entre vários índices de vegetação e o LAI e o Fpar: uma simulação computacional.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI — Índice de Vegetação Ajustado ao Solo Otimizado

Este índice baseia-se no Índice de Vegetação Ajustado ao Solo (SAVI). Ele utiliza um valor padrão de 0,16 para o fator de ajuste de fundo da copa. Rondeaux (1996) determinou que esse valor proporciona maior variação do solo do que o SAVI para baixa cobertura vegetal, ao mesmo tempo em que demonstra maior sensibilidade para coberturas vegetais superiores a 50%. Esse índice é mais adequado para áreas com vegetação relativamente esparsa, onde o solo é visível através da copa.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referência: Rondeaux, G., M. Steven e F. Baret. “Optimization of Soil-Adjusted Vegetation Indices.” Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI – Índice de Vegetação por Diferença Renormalizada

Este índice utiliza a diferença entre os comprimentos de onda do infravermelho próximo e do vermelho, juntamente com o NDVI, para destacar a vegetação saudável. Ele é insensível aos efeitos do solo e da geometria de incidência solar.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referência: Roujean, J., e F. Breon. “Estimativa da PAR absorvida pela vegetação a partir de medições de refletância bidirecional.” Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI — Índice de Vegetação Ajustado ao Solo

Este índice é semelhante ao NDVI, mas suprime os efeitos dos pixels do solo. Ele utiliza um fator de ajuste de fundo da copa, _L_, que é uma função da densidade da vegetação e frequentemente requer conhecimento prévio das quantidades de vegetação. Huete (1988) sugere um valor ótimo de _L_=0,5 para levar em conta as variações de fundo do solo de primeira ordem. Esse índice é mais adequado para áreas com vegetação relativamente esparsa, onde o solo é visível através da copa das árvores.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referência: Huete, A. “A Soil-Adjusted Vegetation Index (SAVI)”. Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Índice de Vegetação por Diferença Transformada

Este índice é útil para monitorar a cobertura vegetal em ambientes urbanos. Ele não sofre saturação como o NDVI e o SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referência: Bannari, A., H. Asalhi e P. Teillet. “Índice de Vegetação por Diferença Transformada (TDVI) para Mapeamento da Cobertura Vegetal”. Em Anais do Simpósio de Geociências e Sensoriamento Remoto, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI — Índice Visível Resistente à Atmosfera

Este índice é baseado no ARVI e é utilizado para estimar a fração de vegetação em uma cena com baixa sensibilidade aos efeitos atmosféricos.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referência: Gitelson, A., et al. “Linhas de vegetação e solo no espaço espectral visível: um conceito e uma técnica para estimativa remota da fração de vegetação”. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI — Índice de Vegetação de Ampla Faixa Dinâmica

Este índice é semelhante ao NDVI, mas utiliza um coeficiente de ponderação (_a_) para reduzir a disparidade entre as contribuições dos sinais do infravermelho próximo e do vermelho para o NDVI. O WDRVI é particularmente eficaz em cenas com densidade de vegetação moderada a alta, quando o NDVI excede 0,6. O NDVI tende a se estabilizar à medida que a fração de vegetação e o índice de área foliar (LAI) aumentam, enquanto o WDRVI é mais sensível a uma faixa mais ampla de frações de vegetação e a variações no LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

O coeficiente de ponderação (_a_) pode variar de 0,1 a 0,2. Um valor de 0,2 é recomendado por Henebry, Viña e Gitelson (2004).

_Referências_

_Gitelson, A. “Índice de Vegetação de Ampla Faixa Dinâmica para Quantificação Remota de Características Biofísicas da Vegetação.” Journal of Plant Physiology 161, n.º 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. “O Índice de Vegetação de Ampla Faixa Dinâmica e sua Utilidade Potencial para Análise de Lacunas.” Gap Analysis Bulletin 12: 50-56._
