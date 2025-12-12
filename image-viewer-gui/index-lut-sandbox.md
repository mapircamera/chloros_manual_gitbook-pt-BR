# Sandbox de Índice/LUT

A Sandbox de Índice/LUT é um espaço de trabalho interativo dentro do Visualizador de Imagens Chloros que permite experimentar cálculos de índices multiespectrais e visualizações de cores em tempo real. Esta poderosa ferramenta ajuda a testar diferentes índices, refinar intervalos de valores e criar visualizações prontas para publicação sem reprocessar todo o conjunto de dados.

## O que é a Sandbox de Índice/LUT?

### Objetivo

A Sandbox oferece:

* **Cálculo de índice em tempo real** - Aplique qualquer índice de vegetação instantaneamente
* **Ajuste LUT interativo** - Ajuste gradientes e intervalos de cores
* **Otimização do fluxo de trabalho** - Determine as melhores configurações antes do processamento em lote

### Sandbox vs. Processamento de projeto

**Sandbox de índice/LUT (interativo):**

* Uma imagem por vez
* Feedback instantâneo
* Experimental e iterativo
* Sem alterações permanentes nos arquivos
* Perfeito para explorar e testar

**Processamento de projeto (em lote):**

* Conjunto de dados inteiro de uma vez
* Configurações pré-definidas
* Arquivos de saída permanentes
* Demorado
* Ideal quando as configurações estão finalizadas

{% hint style=&quot;success&quot; %}
**Melhor fluxo de trabalho**: use a Sandbox para experimentar e encontrar as configurações ideais de índice e LUT e, em seguida, aplique essas configurações durante o processamento do projeto para todo o conjunto de dados.
{% endhint %}

***

## Trabalhando com a Sandbox de índice/LUT

### Entendendo os índices pré-calculados

No Chloros, os índices podem ser aplicados durante o processamento do projeto. Para determinar quais configurações de índice e LUT você deseja aplicar às exportações, é mais fácil usar a sandbox do visualizador de imagens.

A sandbox permite que você:

* **Aplicar novos índices e gradientes de cor (LUTs)** para visualizar os dados
* **Ajustar as configurações de visualização** de forma interativa
* **Visualizar** imagens de índice já calculadas
* **Inspecionar** valores de pixel em todos os níveis de zoom

### Abrindo a área restrita

A área restrita do Índice/LUT é acessada na guia **Visualizador de imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. Clique em uma imagem na grade de imagens do navegador de arquivos para abri-la na guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> .
2. Clique na guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir a barra lateral pop-out à esquerda, caso ainda não esteja aberta

### Selecionando uma imagem para aplicar um índice/LUT

Para trabalhar com um índice na barra lateral <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> :

1. **Abra uma imagem** da grade de imagens principal clicando nela
2. A guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> será aberta
3. Clique no **menu suspenso Camada** (canto superior direito do visualizador)
4. Selecione a camada no menu suspenso:
   * RAW (Refletância)

### Aplicando um índice a uma imagem

Quando a imagem estiver em tela cheia e a barra lateral da guia **Visualizador de imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> estiver aberta:

1. Marque a caixa Índice na parte superior da barra lateral
2. Escolha o filtro da sua câmera no menu suspenso à esquerda
3. Escolha a fórmula de índice desejada no menu suspenso à direita
4. Arraste os círculos de cor do canal do filtro para os locais na fórmula de índice abaixo
5. Quando a fórmula for válida, a imagem será atualizada e mostrará os valores do índice
6. Mova o cursor do mouse para ver os valores no local do cursor
7. Amplie para ver os pixels individuais e seus valores associados

Cada índice tem um intervalo de valores e um significado específicos:

#### NDVI Exemplo

```
Formula: (NIR - Red) / (NIR + Red)

For Survey3W RGN camera:
NIR = 850nm band
Red = 661nm band

Result range: -1.0 to +1.0
Typical vegetation: 0.4 to 0.9
Stressed vegetation: 0.2 to 0.4
Bare soil: 0.0 to 0.2
Water: -0.1 to 0.1
```

Para obter a documentação completa da fórmula do índice, consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

***

## Trabalhando com LUTs (Tabelas de Consulta)

### O que é uma LUT?

Uma **Tabela de Consulta (LUT)** mapeia valores numéricos de índice para cores para visualização:

* **Entrada**: Valor do pixel do índice (por exemplo, NDVI 0,65)
* **Saída**: cor RGB (por exemplo, verde brilhante)
* **Objetivo**: tornar os padrões mais fáceis de ver e interpretar

**Escala de cinza vs. LUT colorida:**

* Escala de cinza: científica e neutra, mostra dados brutos
* LUT colorida: intuitiva e impactante, destaca padrões e diferenças

{% hint style=&quot;success&quot; %}
**Poder de visualização**: Aplicar uma LUT colorida a uma imagem de índice em escala de cinza torna muito mais fácil identificar padrões, anomalias e áreas de interesse rapidamente.
{% endhint %}

### Aplicando uma LUT a uma imagem de índice

Depois de ter uma imagem de índice mostrando

1. Clique no <img src="../.gitbook/assets/image.png" alt="" data-size="line"> botão “+Adicionar LUT”
2. Selecione o gradiente de cores
3. Ajuste os pontos finais mínimo/máximo do recorte
4. Ajuste o modo de recorte
5. Marque a caixa Índice na barra lateral do **Visualizador de imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para aplicar a LUT

### Escolhendo um gradiente de cor

**Selecionando um gradiente:**

1. No painel LUT, localize a **barra de gradiente colorida**
2. Passe o mouse sobre ela para ver as predefinições de gradiente disponíveis
3. Selecione o gradiente desejado
4. A imagem **é atualizada imediatamente** com novas cores quando a caixa Índice é marcada

{% hint style=&quot;success&quot; %}
**Melhor prática**: para índices de vegetação como NDVI, o gradiente Red-Amarelo-Green é mais intuitivo, pois se alinha às associações naturais de cores (verde = saudável, amarelo = moderado, vermelho = estressado).
{% endhint %}

### Ajustando classes de cores

O **controle Classes** determina quantos degraus de cores distintos aparecem no seu gradiente:

**Opções de contagem de classes:**

* **2-5 classes**: categorias muito amplas, zonas distintas
* **6-10 classes**: equilibradas, boas para classificação
* **11-20 classes**: gradientes suaves, aparência contínua
* **20+ classes**: quase contínuas, suavidade máxima

**Como ajustar:**

1. No painel LUT, localize os **quadrados de amostra de cor abaixo da barra de gradiente**
2. Ajuste o número de classes adicionando com o botão +
3. Remova o número de classes clicando duas vezes em uma amostra de cor
4. O gradiente é atualizado **em tempo real** na imagem

**Efeito na visualização:**

* **Menos classes** (3-5): Cria zonas distintas, classificação simplificada, categorias mais fáceis de distinguir
* **Classes médias** (6-10): Abordagem equilibrada, boa para a maioria das aplicações
* **Mais classes** (15-20): Transições suaves, variação detalhada, aparência fotográfica

**Quando usar:**

* **Poucas classes (3-5)**: Slides de apresentação, mapas de classificação, relatórios simples
* **Classes médias (6-10)**: Análise geral, detalhes equilibrados, relatórios padrão
* **Muitas classes (15-20)**: Análise científica, inspeção detalhada, resultados com qualidade de publicação

### Ajustando intervalos de valores

Os **controles de intervalo de valores** determinam quais valores de índice são mapeados para quais cores em seu gradiente:

**Controles de intervalo no painel LUT:**

* **Valor mínimo**: Limite inferior da escala de cores
* **Valor máximo**: limite superior da escala de cores
* **Valores intermediários**: distribuídos automaticamente entre o mínimo e o máximo (com base na contagem de classes)

#### Ajustando valores mínimos/máximos

**Para ajustar intervalos de valores:**

1. No painel LUT, localize os campos de entrada **Valor mínimo** e **Valor máximo**
2. Clique no campo **Valor mínimo**
3. Digite o valor mínimo desejado (por exemplo, `0.2`)
4. Pressione **Enter** ou clique fora do campo
5. Repita para o campo **Valor máximo** (por exemplo, `0.9`)
6. A visualização **é atualizada imediatamente**

{% hint style=&quot;info&quot; %}
**Escalonamento automático**: quando você aplica uma LUT pela primeira vez, o Chloros define automaticamente o mínimo/máximo para o intervalo de dados real na imagem. Você pode então restringir esse intervalo para se concentrar em intervalos de valores específicos de interesse.
{% endhint %}

**Exemplo de ajustes de intervalo NDVI:**

* **Intervalo completo**: `-1.0` a `1.0` (mostrar todos os valores possíveis)
* **Focado na vegetação**: `0.2` a `0.9` (excluir solo nu e água)
* **Apenas vegetação saudável**: `0.5` a `0.9` (destacar apenas plantas vigorosas)
* **Detecção de estresse**: `0.2` a `0.5` (enfatize áreas problemáticas)
* **Intervalo personalizado**: ajuste com base nos valores de pixel observados

**Por que ajustar os intervalos?**

* **Aumente o contraste** na sua área de interesse
* **Exclua valores irrelevantes** (por exemplo, corpos d&#x27;água, solo descoberto)
* **Padronize a visualização** em várias imagens ou datas
* **Enfatize diferenças sutis** dentro de um intervalo de valores estreito

### Recorte de valores fora do intervalo

Quando os valores dos pixels estão fora do intervalo mínimo/máximo definido, você pode controlar como eles são exibidos usando **modos de recorte**.

#### **Opções de modo de recorte disponíveis:**

#### 1. Mínimo e máximo

* Pixels **abaixo do mínimo** → exibir usando a **primeira cor** no gradiente (por exemplo, vermelho)
* Pixels **acima do máximo** → exibição usando a **última cor** no gradiente (por exemplo, verde)
* **Caso de uso**: enfatizar extremos, mostrar a faixa completa de dados com cores saturadas nos limites
* **Exemplo**: os valores NDVI abaixo de 0,2 aparecem todos em vermelho, os valores acima de 0,9 aparecem todos em verde

#### 2. Fundo transparente

* Os pixels **fora do intervalo** tornam-se **totalmente transparentes**
* Apenas os pixels **dentro do intervalo** mostram o gradiente de cores
* **Caso de uso**: sobreposição GIS, isolando intervalos de valores específicos, destacando apenas áreas de interesse
* **Exemplo**: Mostrar apenas NDVI 0,4-0,7 em cores, todo o resto transparente

{% hint style=&quot;warning&quot; %}
**Limitação de transparência**: Os pixels transparentes aparecerão como a cor de fundo no visualizador. Quando exportados durante o processamento, a transparência é preservada no formato PNG, mas não no JPG.
{% endhint %}

#### 3. Fundo do índice

* Os pixels **fora do intervalo** são exibidos em **escala de cinza** (mostrando os valores brutos do índice)
* Os pixels **dentro do intervalo** mostram **gradiente de cor**
* **Caso de uso**: destaque sutil, mantém o contexto enquanto enfatiza áreas de interesse
* **Exemplo**: destaque em cores a vegetação estressada (NDVI 0,3-0,5) enquanto mostra áreas saudáveis em cinza

#### 4. Fundo original

* Os pixels **fora do intervalo** exibem a **imagem multiespectral original**
* Os pixels **dentro do intervalo** mostram **gradiente de cor**
* **Caso de uso**: mais intuitivo - combina o contexto da imagem natural com a sobreposição analítica de cores
* **Exemplo**: veja a aparência real do campo/colheita com áreas de estresse codificadas por cores sobrepostas

### Escolhendo o modo de recorte certo

| Modo de recorte              | Ideal para                                   | Estilo de visualização          |
| -------------------------- | ------------------------------------------ | ---------------------------- |
| **Mínimo e máximo**    | Exibição completa dos dados, análise científica     | Todos os pixels coloridos           |
| **Fundo transparente** | Sobreposições GIS, isolando intervalos específicos    | Cor no intervalo, em branco fora dele |
| **Fundo de índice**       | Ênfase sutil, mantendo o contexto dos dados  | Cor no intervalo, cinza fora dele  |
| **Fundo original**    | Relatórios, apresentações, análise intuitiva | Cor no intervalo, foto fora dele |

### Criando cores LUT personalizadas

Para ter controle total sobre sua visualização, você pode criar **gradientes de cores personalizados** editando paradas de cores individuais.

**Para criar um gradiente personalizado:**

1. No painel LUT, localize a **barra de visualização do gradiente**
2. Procure os **quadrados de amostra de cores** abaixo do gradiente
3. **Clique em uma parada de cor** para selecioná-la
4. Um **seletor de cores** será aberto
5. Escolha uma nova cor usando:
   * **Roda de cores**: seleção visual de cores
   * **Controles deslizantes RGB/HSV**: controle preciso das cores
   * **Entrada de código hexadecimal**: especificação exata da cor (por exemplo, `#FF0000` para vermelho)
6. Clique fora do seletor de cores **para aplicar a nova cor**
7. O gradiente **é atualizado imediatamente** na imagem

**Adicionando ou removendo paradas de cor:**

* **Adicionar uma parada**: Clique no ícone + para adicionar uma nova amostra no final
* **Remover uma parada**: Clique duas vezes no quadrado de cor para remover a amostra

**Estratégias de personalização:**

* **Inverter gradiente**: inverta a ordem das cores para reverter o significado (por exemplo, verde = baixo, vermelho = alto)
* **Cores da marca**: combine a paleta de cores da sua organização para relatórios
* **Adequado para daltônicos**: use combinações de laranja-azul ou roxo-amarelo
* **Otimização de impressão**: escolha cores que funcionem tanto na impressão colorida quanto na impressão em escala de cinza
* **Limite múltiplo**: use cores distintas em limites de valores específicos para classificação

{% hint style=&quot;info&quot; %}
**Salvar gradientes personalizados**: gradientes personalizados podem ser salvos e reutilizados. Clique no ícone salvar no painel LUT para preservar seus esquemas de cores personalizados para uso futuro.
{% endhint %}

***

## Fluxo de trabalho interativo

### Atualizações em tempo real

Todos os ajustes LUT na área restrita atualizam a imagem **instantaneamente e de forma interativa**:

* **Trocar camada** → A imagem muda imediatamente
* **Selecionar gradiente** → As cores são atualizadas instantaneamente
* **Ajustar intervalo de valores** → O contraste muda em tempo real
* **Alterar classes** → A suavidade do gradiente é atualizada imediatamente
* **Modificar recorte** → A exibição do fundo muda instantaneamente
* **Editar cores** → O gradiente personalizado é aplicado imediatamente

**Não é necessário o botão “Aplicar”** — todas as alterações são ao vivo e interativas!

{% hint style=&quot;success&quot; %}
**Feedback ao vivo**: o feedback visual instantâneo permite que você experimente rapidamente diferentes configurações até encontrar a visualização ideal para suas necessidades de análise.
{% endhint %}

### Fluxo de trabalho de refinamento iterativo

**Fluxo de trabalho típico de otimização de LUT:**

1. **Selecione a camada de índice** (por exemplo, RAW (Refletância))
2. **Aplique o índice** - Escolha o filtro da câmera e a fórmula do índice, arraste os círculos coloridos para o local apropriado na fórmula do índice
3. **Aplique o gradiente LUT** - Comece com a predefinição Red-Yellow-Green
4. **Inspecione os valores dos pixels** - Mova o cursor e observe os intervalos de valores
5. **Ajuste o mínimo/máximo** - Restrinja para focar na vegetação (por exemplo, 0,2 a 0,9)
6. **Escolha o recorte** - Experimente “Original Background” (Fundo original) para o contexto
7. **Refine as cores** - Personalize o gradiente, se necessário, para dar ênfase específica
8. **Finalize as configurações** - Documente as configurações e copie para Configurações do projeto para processamento de exportação

### Inspeção do valor dos pixels

Compreender os valores reais dos pixels é crucial para definir intervalos LUT eficazes:

**Como inspecionar os valores:**

1. Os valores dos pixels são exibidos quando a imagem tem a caixa Índice ou ambas as caixas Índice e LUT **marcadas**.
2. **Mova o cursor** sobre diferentes áreas da imagem
3. **Observe os valores dos pixels** exibidos na legenda ao passar o cursor
4. Amplie para ver os pixels individuais destacados com um valor flutuante
5. **Anote** os intervalos de valores para diferentes características:
   * **Vegetação saudável**: por exemplo, NDVI 0,55-0,85
   * **Vegetação estressada**: por exemplo, NDVI 0,30-0,50
   * **Solo nu**: por exemplo, NDVI 0,05-0,25
   * **Água** (se presente): por exemplo, NDVI -0,05 a 0,10

**Usando valores de pixel para definir intervalos de LUT:**

Após inspecionar os valores de pixel, ajuste o mínimo/máximo da LUT de acordo:

**Exemplo de cenário:**

* **Observação**: Valores do solo = 0,05-0,25, Estressado = 0,25-0,50, Saudável = 0,50-0,85
* **Objetivo**: Visualizar apenas a saúde das plantas (excluir o solo)
* **Configurações LUT**: Mínimo = `0.25`, Máximo = `0.85`
* **Recorte**: “Fundo original” para ver o solo em sua cor natural
* **Resultado**: O gradiente de cores se aplica apenas à vegetação, o solo é exibido como na imagem original

{% hint style=&quot;info&quot; %}
**Faixa dinâmica**: diferentes culturas, estações e estágios de crescimento terão diferentes faixas de valores. Sempre verifique os valores dos pixels em seu conjunto de dados específico antes de definir as faixas LUT.
{% endhint %}

***

## Índices personalizados (Chloros+)

### Criação de fórmulas de índice personalizadas

{% hint style=&quot;info&quot; %}
**Onde criar**: os índices personalizados podem ser configurados em **Configurações do projeto** antes do processamento, bem como na barra lateral da área restrita do Visualizador de imagens.
{% endhint %}

**Para criar um índice personalizado:**

1. **Abra as Configurações do projeto** (antes do processamento) ou a barra lateral da área restrita do Visualizador de imagens
2. Navegue até o **menu suspenso Fórmula do índice**
3. Procure a opção **“Personalizado”** (é necessário estar conectado com a licença Chloros+)
4. **Defina sua fórmula** usando variáveis de banda:
   * Nomes das bandas: `NIR`, `Red`, `Green`, `Blue`, `RedEdge`, etc.
   * Operadores: `+`, `-`, `*`, `/`, `^` (expoente)
   * Funções: `sqrt()`, `abs()`, etc. (se suportado)
   * Parênteses: `()` para ordem das operações
5. **Nomeie seu índice** (por exemplo, “MeuÍndice” ou “NDVIPersonalizado”)
6. **Salve a configuração**

**Exemplos de fórmulas personalizadas:**

```
Modified NDVI with offset:
(NIR - Red) / (NIR + Red + 0.5)

Simple ratio:
NIR / Red

Complex multi-band:
(NIR - Red) / (NIR + Red - Blue)

Exponential index:
(NIR / Red) ^ 2
```

{% hint style=&quot;warning&quot; %}
**Validação da fórmula**: certifique-se de que sua fórmula usa bandas disponíveis em sua câmera. Por exemplo, RedEdge só está disponível em câmeras com um filtro RedEdge.
{% endhint %}

***

## Próximos passos

Agora que você entende o Index/LUT Sandbox:

* **Aplicar ao processamento**: use as configurações descobertas em [Configurações do projeto](../project-settings/project-settings.md)
* **Processamento em lote**: aplique índices otimizados a conjuntos de dados completos
* **Saiba mais**: leia [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md)

Documentação relacionada:

* [**Camadas de imagem**](image-layers.md) - Gerenciamento e visualização de camadas
* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) - Noções básicas do visualizador de imagens
* [**Processamento de imagens (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Fluxo de trabalho completo de processamento
