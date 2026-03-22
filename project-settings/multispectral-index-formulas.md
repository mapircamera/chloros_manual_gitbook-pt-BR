---
description: This page lists some multispectral indices that Chloros uses
metaLinks:
  alternates:
    - >-
      https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/multispectral-index-formulas
---

# Fórmulas de índices multiespectrais

As fórmulas de índices abaixo utilizam uma combinação das faixas de transmissão média do filtro Survey3:

<table><thead><tr><th align="center">Cor do filtro Survey3</th><th width="196.199951171875" align="center">Survey3 Nome do filtro</th><th width="159.800048828125" align="center">Faixa de transmissão (FWHM)</th><th align="center">Transmissão média</th></tr></thead><tbody><tr><td align="center">Blue</td><td align="center">NGB - Blue</td><td align="center">468-483 nm</td><td align="center">475 nm</td></tr><tr><td align="center">Cyan</td><td align="center">OCN - Cyan</td><td align="center">476-512 nm</td><td align="center">494 nm</td></tr><tr><td align="center">Green</td><td align="center">RGN | NGB - Green</td><td align="center">543-558 nm</td><td align="center">547 nm</td></tr><tr><td align="center">Orange</td><td align="center">OCN - Orange</td><td align="center">598-640 nm</td><td align="center">619 nm</td></tr><tr><td align="center">Red</td><td align="center">RGN - Red</td><td align="center">653-668 nm</td><td align="center">661 nm</td></tr><tr><td align="center">RedEdge</td><td align="center">Re - RedEdge</td><td align="center">712-735 nm</td><td align="center">724 nm</td></tr><tr><td align="center">NIR1</td><td align="center">OCN - NIR1</td><td align="center">798-848 nm</td><td align="center">823 nm</td></tr><tr><td align="center">NIR2</td><td align="center">RGN | NGB | NIR - NIR2</td><td align="center">835-865 nm</td><td align="center">850 nm</td></tr></tbody></table>Quando essas fórmulas são usadas, o nome pode terminar em &quot;\_1&quot; ou &quot;\_2&quot;, o que corresponde ao filtro NIR, seja NIR1 ou NIR2, que foi utilizado.

***

## EVI - Índice de Vegetação Aprimorado

Este índice foi originalmente desenvolvido para uso com dados MODIS como uma melhoria em relação ao NDVI, otimizando o sinal de vegetação em áreas de alto índice de área foliar (LAI). É mais útil em regiões com alto LAI, onde o NDVI pode saturar. Ele utiliza a região de refletância azul para corrigir sinais de fundo do solo e reduzir influências atmosféricas, incluindo a dispersão de aerossóis.

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Os valores de EVI devem variar de 0 a 1 para pixels de vegetação. Elementos brilhantes, como nuvens e edifícios brancos, juntamente com elementos escuros, como água, podem resultar em valores de pixel anômalos em uma imagem EVI. Antes de criar uma imagem EVI, deve-se mascarar as nuvens e os elementos brilhantes da imagem de refletância e, opcionalmente, aplicar um limiar aos valores dos pixels de 0 a 1.

_Referência: Huete, A., et al. &quot;Overview of the Radiometric and Biophysical Performance of the MODIS Vegetation Indices.&quot; Remote Sensing of Environment 83 (2002):195–213._

***

## FCI1 - Índice de Cobertura Florestal 1

Este índice distingue as copas das árvores de outros tipos de vegetação utilizando imagens de refletância multiespectral que incluem uma banda de borda vermelha.

$$
FCI1 = Red * RedEdge
$$

Áreas florestadas apresentarão valores mais baixos de FCI1 devido à menor refletância das árvores e à presença de sombras dentro da copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Índices robustos de cobertura florestal para imagens multiespectrais.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## FCI2 - Índice de Cobertura Florestal 2

Este índice distingue as copas das árvores de outros tipos de vegetação utilizando imagens de refletância multiespectral que não incluem uma banda de borda vermelha.

$$
FCI2 = Red * NIR
$$

As áreas florestadas terão valores FCI2 mais baixos devido à menor refletância das árvores e à presença de sombras na copa.

_Referência: Becker, Sarah J., Craig S.T. Daughtry e Andrew L. Russ. &quot;Índices robustos de cobertura florestal para imagens multiespectrais.&quot; Photogrammetric Engineering &amp; Remote Sensing 84.8 (2018): 505-512._

***

## GEMI - Índice de Monitoramento Ambiental Global

Este índice de vegetação não linear é utilizado para monitoramento ambiental global a partir de imagens de satélite e busca corrigir os efeitos atmosféricos. É semelhante ao NDVI, mas é menos sensível aos efeitos atmosféricos. É afetado pelo solo descoberto; portanto, não é recomendado para uso em áreas de vegetação esparsa ou moderadamente densa.

$$
GEMI = eta (1 - 0.25 * eta) - {Red - 0.125 \over 1 - Red}
$$

Onde:

$$
eta = {2(NIR^{2}-Red^{2}) + 1.5 * NIR + 0.5 *  Red \over NIR + Red + 0.5}
$$

_Referência: Pinty, B., e M. Verstraete. GEMI: um Índice Não-Linear para Monitorar a Vegetação Global a partir de Satélites. Vegetation 101 (1992): 15-20._

***

## GARI - Green Índice Resistente à Atmosfera

Este índice é mais sensível a uma ampla gama de concentrações de clorofila e menos sensível aos efeitos atmosféricos do que o NDVI.

$$
GARI = {NIR - [Green - \gamma(Blue - Red)] \over NIR + [Green - \gamma(Blue - Red)]   }
$$

A constante gama é uma função de ponderação que depende das condições dos aerossóis na atmosfera. O ENVI utiliza um valor de 1,7, que é o valor recomendado por Gitelson, Kaufman e Merzylak (1996, página 296).

_Referência: Gitelson, A., Y. Kaufman e M. Merzylak. &quot;Use of a Green Channel in Remote Sensing of Global Vegetation from EOS-MODIS.&quot; Remote Sensing of Environment 58 (1996): 289-298._

***

## GCI - Green Índice de clorofila

Este índice é utilizado para estimar o teor de clorofila nas folhas em uma ampla variedade de espécies vegetais.

$$
GCI = {NIR \over Green} - 1
$$

A disponibilidade de comprimentos de onda amplos e verdes permite uma melhor previsão do teor de clorofila, ao mesmo tempo em que proporciona maior sensibilidade e uma relação sinal-ruído mais elevada.

_Referência: Gitelson, A., Y. Gritz e M. Merzlyak. “Relações entre o teor de clorofila foliar e a refletância espectral e algoritmos para avaliação não destrutiva da clorofila em folhas de plantas superiores.” Journal of Plant Physiology 160 (2003): 271-282._

***

## Índice de Folhas GLI - Green

Este índice foi originalmente projetado para uso com uma câmera digital RGB para medir a cobertura de trigo, onde os números digitais (DNs) de vermelho, verde e azul variam de 0 a 255.

$$
GLI = {(Green - Red) + (Green - Blue)  \over (2 * Green) + Red + Blue }
$$

Os valores do GLI variam de -1 a +1. Valores negativos representam o solo e características não vivas, enquanto valores positivos representam folhas verdes e caules.

_Referência: Louhaichi, M., M. Borman e D. Johnson. &quot;Plataforma Espacialmente Localizada e Fotografia Aérea para Documentação dos Impactos do Pastoreio no Trigo.&quot; Geocarto International 16, n.º 1 (2001): 65-70._

***

## GNDVI - Green Índice de Vegetação por Diferença Normalizada

Este índice é semelhante ao NDVI, exceto que mede o espectro verde de 540 a 570 nm em vez do espectro vermelho. Este índice é mais sensível à concentração de clorofila do que o NDVI.

$$
GNDVI = {(NIR - Green) \over (NIR + Green)  }
$$

_Referência: Gitelson, A., e M. Merzlyak. “Remote Sensing of Chlorophyll Concentration in Higher Plant Leaves.” Advances in Space Research 22 (1998): 689-692._

***

## GOSAVI - Green Índice de Vegetação Ajustado ao Solo Otimizado

Este índice foi originalmente concebido com fotografia infravermelha colorida para prever as necessidades de nitrogênio do milho. É semelhante ao OSAVI, mas substitui a banda verde pela vermelha.

$$
GOSAVI = {NIR - Green \over NIR + Green + 0.16)  }
$$

_Referência: Sripada, R., et al. &quot;Determinação das necessidades de nitrogênio durante a estação para o milho utilizando fotografia aérea infravermelha colorida.&quot; Tese de doutorado, Universidade Estadual da Carolina do Norte, 2005._

***

## Índice de vegetação da razão GRVI - Green

Este índice é sensível às taxas de fotossíntese nas copas das florestas, uma vez que as refletâncias do verde e do vermelho são fortemente influenciadas pelas mudanças nos pigmentos foliares.

$$
GRVI = {NIR \over Green }
$$

_Referência: Sripada, R., et al. “Fotografia aérea em infravermelho colorido para determinar as necessidades de nitrogênio no início da estação no milho.” Agronomy Journal 98 (2006): 968-977._

***

## GSAVI - Green Índice de Vegetação Ajustado ao Solo

Este índice foi originalmente concebido com fotografia infravermelha colorida para prever as necessidades de nitrogênio do milho. É semelhante ao SAVI, mas substitui a banda verde pela vermelha.

$$
GSAVI = 1.5 * {(NIR - Green) \over (NIR + Green + 0.5)  }
$$

_Referência: Sripada, R., et al. &quot;Determinação das Necessidades de Nitrogênio Durante a Temporada para o Milho Usando Fotografia Aérea em Infravermelho Colorido.&quot; Tese de doutorado, Universidade Estadual da Carolina do Norte, 2005._

***

## LAI - Índice de Área Folhosa

Este índice é usado para estimar a cobertura foliar e prever o crescimento e o rendimento da cultura. O ENVI calcula o LAI verde utilizando a seguinte fórmula empírica de Boegh et al (2002):

$$
LAI = 3.618 * EVI - 0.118
$$

Onde EVI é:

$$
EVI = 2.5 *  {(NIR - Red) \over (NIR + 6 * Red - 7.5 * Blue + 1)}
$$

Valores altos de LAI variam tipicamente de aproximadamente 0 a 3,5. No entanto, quando a cena contém nuvens e outros elementos brilhantes que produzem pixels saturados, os valores de LAI podem exceder 3,5. O ideal é mascarar nuvens e elementos brilhantes da sua cena antes de criar uma imagem LAI.

_Referência: Boegh, E., H. Soegaard, N. Broge, C. Hasager, N. Jensen, K. Schelde e A. Thomsen. “Dados multiespectrais aéreos para quantificar o índice de área foliar, a concentração de nitrogênio e a eficiência fotossintética na agricultura.” Remote Sensing of Environment 81, n.º 2-3 (2002): 179-193._

***

## LCI - Índice de clorofila foliar

Este índice é utilizado para estimar o teor de clorofila em plantas superiores, sendo sensível à variação na refletância causada pela absorção da clorofila.

$$
LCI = {NIR2 - RedEdge \over NIR2 + Red}
$$

_Referência: Datt, B. “Sensoriamento remoto do teor de água em folhas de eucalipto.” Journal of Plant Physiology 154, n.º 1 (1999): 30-36._

***

## MNLI - Índice Não Linear Modificado

Este índice é um aprimoramento do Índice Não Linear (NLI) que incorpora o Índice de Vegetação Ajustado ao Solo (SAVI) para levar em conta o fundo do solo. O ENVI utiliza um valor de fator de ajuste de fundo da copa (_L_) de 0,5.

$$
MNLI = {(NIR^{2} - Red) * (1 + L) \over (NIR^{2} + Red + L)  }
$$

_Referência: Yang, Z., P. Willis e R. Mueller. &quot;Impact of Band-Ratio Enhanced AWIFS Image to Crop Classification Accuracy.&quot; Anais do Simpósio de Sensoriamento Remoto Pecora 17 (2008), Denver, CO._

***

## MSAVI2 - Índice de Vegetação Ajustado ao Solo Modificado 2

Este índice é uma versão mais simples do índice MSAVI proposto por Qi et al. (1994), que aprimora o Índice de Vegetação Ajustado ao Solo (SAVI). Ele reduz o ruído do solo e aumenta a faixa dinâmica do sinal de vegetação. O MSAVI2 baseia-se em um método indutivo que não utiliza um valor _L_ constante (como no SAVI) para destacar a vegetação saudável.

$$
MSAVI2 = {2 * NIR + 1 - \sqrt{(2 * NIR + 1)^{2} - 8(NIR - Red)} \over 2}
$$

_Referência: Qi, J., A. Chehbouni, A. Huete, Y. Kerr e S. Sorooshian. &quot;A Modified Soil Adjusted Vegetation Index.&quot; Remote Sensing of Environment 48 (1994): 119-126._

***

## NDRE - Diferença Normalizada RedEdge

Este índice é semelhante ao NDVI, mas compara o contraste entre o NIR e o RedEdge, em vez do Red, o que frequentemente detecta o estresse da vegetação mais cedo.

$$
NDRE = {NIR - RedEdge \over NIR + RedEdge  }
$$

***

## NDVI - Índice de Vegetação por Diferença Normalizada

Este índice é uma medida de vegetação verde saudável. A combinação de sua formulação de diferença normalizada com o uso das regiões de maior absorção e refletância da clorofila torna-o robusto em uma ampla gama de condições. Ele pode, no entanto, saturar em condições de vegetação densa quando o LAI se torna alto.

$$
NDVI = {NIR - Red \over NIR + Red  }
$$

O valor deste índice varia de -1 a 1. A faixa comum para vegetação verde é de 0,2 a 0,8.

_Referência: Rouse, J., R. Haas, J. Schell e D. Deering. Monitoramento de Sistemas Vegetais nas Grandes Planícies com o ERTS. Terceiro Simpósio ERTS, NASA (1973): 309-317._

***

## NLI - Índice Não Linear

Este índice pressupõe que a relação entre muitos índices de vegetação e parâmetros biofísicos da superfície é não linear. Ele lineariza relações com parâmetros da superfície que tendem a ser não lineares.

$$
NLI = {NIR^{2} - Red \over NIR^{2} + Red  }
$$

_Referência: Goel, N. e W. Qin. “Influências da arquitetura do dossel nas relações entre vários índices de vegetação e o LAI e o Fpar: uma simulação computacional.” Remote Sensing Reviews 10 (1994): 309-347._

***

## OSAVI - Índice de Vegetação Ajustado ao Solo Otimizado

Este índice baseia-se no Índice de Vegetação Ajustado ao Solo (SAVI). Ele utiliza um valor padrão de 0,16 para o fator de ajuste de fundo da copa. Rondeaux (1996) determinou que esse valor proporciona maior variação do solo do que o SAVI para baixa cobertura vegetal, ao mesmo tempo em que demonstra maior sensibilidade à cobertura vegetal superior a 50%. Este índice é mais adequado para áreas com vegetação relativamente esparsa, onde o solo é visível através da copa.

$$
OSAVI = {(NIR - Red) \over (NIR + Red + 0.16)  }
$$

_Referência: Rondeaux, G., M. Steven e F. Baret. &quot;Optimization of Soil-Adjusted Vegetation Indices.&quot; Remote Sensing of Environment 55 (1996): 95-107._

***

## RDVI - Índice de Vegetação por Diferença Renormalizada

Este índice utiliza a diferença entre os comprimentos de onda do infravermelho próximo e do vermelho, juntamente com o NDVI, para destacar a vegetação saudável. É insensível aos efeitos do solo e da geometria de observação solar.

$$
RDVI = {(NIR- Red) \over \sqrt{(NIR + Red)}  }
$$

_Referência: Roujean, J., e F. Breon. &quot;Estimativa da PAR Absorvida pela Vegetação a partir de Medições de Refletância Bidirecional.&quot; Remote Sensing of Environment 51 (1995): 375-384._

***

## SAVI - Índice de Vegetação Ajustado pelo Solo

Este índice é semelhante ao NDVI, mas suprime os efeitos dos pixels de solo. Ele utiliza um fator de ajuste de fundo da copa, _L_, que é uma função da densidade da vegetação e frequentemente requer conhecimento prévio das quantidades de vegetação. Huete (1988) sugere um valor ótimo de _L_=0,5 para levar em conta variações de fundo do solo de primeira ordem. Este índice é mais adequado para áreas com vegetação relativamente esparsa, onde o solo é visível através da copa das árvores.

$$
SAVI = {1.5 * (NIR- Red) \over (NIR + Red + 0.5)  }
$$

_Referência: Huete, A. &quot;Um Índice de Vegetação Ajustado ao Solo (SAVI).&quot; Remote Sensing of Environment 25 (1988): 295-309._

***

## TDVI - Índice de Vegetação por Diferença Transformada

Este índice é útil para monitorar a cobertura vegetal em ambientes urbanos. Ele não satura como o NDVI e o SAVI.

$$
TDVI = 1.5 * {(NIR- Red) \over \sqrt{NIR^{2} + Red + 0.5}  }
$$

_Referência: Bannari, A., H. Asalhi e P. Teillet. “Índice de Vegetação por Diferença Transformada (TDVI) para Mapeamento da Cobertura Vegetal” Em Anais do Simpósio de Geociências e Sensoriamento Remoto, IGARSS &#x27;02, IEEE International, Volume 5 (2002)._

***

## VARI - Índice Visível Resistente à Atmosfera

Este índice é baseado no ARVI e é usado para estimar a fração de vegetação em uma cena com baixa sensibilidade aos efeitos atmosféricos.

$$
VARI = {Green - Red \over Green + Red - Blue  }
$$

_Referência: Gitelson, A., et al. &quot;Linhas de vegetação e solo no espaço espectral visível: um conceito e técnica para estimativa remota da fração de vegetação. International Journal of Remote Sensing 23 (2002): 2537−2562._

***

## WDRVI - Índice de Vegetação de Ampla Faixa Dinâmica

Este índice é semelhante ao NDVI, mas utiliza um coeficiente de ponderação (_a_) para reduzir a disparidade entre as contribuições dos sinais do infravermelho próximo e do vermelho para o NDVI. O WDRVI é particularmente eficaz em cenas com densidade de vegetação moderada a alta, quando o NDVI excede 0,6. O NDVI tende a se estabilizar quando a fração de vegetação e o índice de área foliar (LAI) aumentam, enquanto o WDRVI é mais sensível a uma faixa mais ampla de frações de vegetação e a variações no LAI.

$$
WDRVI = {(\alpha * NIR- Red) \over (\alpha * NIR + Red)}
$$

O coeficiente de ponderação (_a_) pode variar de 0,1 a 0,2. Um valor de 0,2 é recomendado por Henebry, Viña e Gitelson (2004).

_Referências_

_Gitelson, A. &quot;Índice de Vegetação de Ampla Faixa Dinâmica para Quantificação Remota de Características Biofísicas da Vegetação.&quot; Journal of Plant Physiology 161, n.º 2 (2004): 165-173._

_Henebry, G., A. Viña e A. Gitelson. &quot;O Índice de Vegetação de Ampla Faixa Dinâmica e sua Utilidade Potencial para Análise de Lacunas.&quot; Gap Analysis Bulletin 12: 50-56._
