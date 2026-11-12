# Área de teste de Índice/LUT

A Área de teste de Índice/LUT é o espaço de trabalho interativo na barra lateral do Visualizador de Imagens Chloros. Você escolhe uma fórmula, vincula os canais da sua câmera a ela, aplica um gradiente de cores e ajusta o intervalo de valores — e a imagem é atualizada em tempo real enquanto você faz isso. Desde a versão 1.2.0, você também pode **salvar o que criou**, seja para uma imagem ou para todo o projeto, sem precisar reprocessar.

## Para que serve a Área de Testes

| Área de Testes de Índice/LUT (interativa)        | Processamento do Projeto (em lote)       |
| -------------------------------------- | -------------------------------- |
| Uma imagem por vez, feedback instantâneo  | Todo o conjunto de dados em uma única execução     |
| Experimental e iterativo             | Configurações pré-definidas          |
| Renderiza em tempo real; salva somente quando você solicitar  | Sempre grava os arquivos finais      |
| Perfeito para encontrar as configurações certas | Ideal quando as configurações estiverem finalizadas |

{% hint style="success" %}
**O fluxo de trabalho habitual**: ajuste na Sandbox até que a visualização mostre o que você deseja; em seguida, exporte diretamente da Sandbox ou copie as mesmas configurações de índice e LUT para [Configurações do Projeto](../project-settings/project-settings.md) para que a próxima execução de processamento as aplique a todas as imagens.
{% endhint %}

***

## Abrindo o Sandbox

1. Clique em uma imagem na grade — ela será aberta em tela cheia na aba **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
2. Clique no ícone do **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> para abrir a barra lateral esquerda, caso ela ainda não esteja aberta
3. Escolha uma camada multibanda no menu suspenso de camadas no canto superior direito — **RAW (Refletância)** é a escolha usual, pois os valores de índice calculados com base na refletância calibrada são comparáveis entre as imagens

A barra lateral exibe, de cima para baixo:

* o nome da imagem e o modelo da câmera
* o botão **Exportar/Salvar imagem(ns)** — aparece assim que a opção Índice ou LUT for marcada
* as caixas de seleção **Índice**e**LUT**
* o painel de configuração do índice
* o painel **Valores do Cursor** com a leitura, o histograma e o controle de GSD

{% hint style="warning" %}
**Não disponível para câmeras monocromáticas.** Em uma imagem LATTICE M3M de banda única, ambas as caixas de seleção estão desativadas, com a dica de ferramenta _“Não disponível para sensores monocromáticos (M3M)”_ — um índice multibanda não é definido em uma única banda. Para calcular índices a partir de câmeras M3M, combine duas ou mais em uma pilha multibanda alinhada e use o mecanismo de índice do LATTICE.
{% endhint %}

***

## Aplicando um índice

1. Marque a caixa **Índice** na parte superior da barra lateral
2. Escolha o filtro da sua câmera no menu suspenso à esquerda (`RGN`, `OCN`, `NGB`, `RGB`, `RE`, `NIR`)
3. Escolha uma fórmula de índice no menu suspenso à direita — 27 fórmulas integradas, além de quaisquer fórmulas personalizadas que você tenha salvo
4. A fórmula é exibida como expressão matemática abaixo, com um círculo vazio em cada slot de banda. **Arraste um círculo colorido de canal para um espaço** para vinculá-lo
5. Assim que todos os espaços utilizados pela fórmula forem vinculados, a imagem é atualizada e exibe os valores de índice
6. Passe o cursor sobre a imagem para ler os valores; o painel **Valores do Cursor** adiciona uma linha de índice com o valor sob o cursor

Clique duas vezes em um espaço vinculado para limpá-lo. Uma fórmula incompleta é um estado normal durante o arrastar, não um erro — a imagem simplesmente não é atualizada até que a fórmula esteja completa.

Os círculos de canal são codificados por cores: vermelho = Red, verde = Green, azul = Blue, laranja = Orange, ciano = Cyan, roxo = NIR, magenta = RE. As mesmas cores são usadas para os pontos dos canais e as curvas do histograma no painel “Valores do cursor”.

### Exemplo do NDVI

```

Formula: (NIR - Red) / (NIR + Red)

For a Survey3W RGN camera:
  NIR = 850 nm band
  Red = 661 nm band

Result range:          -1.0 to +1.0
Typical vegetation:     0.4 to 0.9
Stressed vegetation:    0.2 to 0.4
Bare soil:              0.0 to 0.2
Water:                 -0.1 to 0.1
```

Para consultar a referência completa das fórmulas — todas as três listas de predefinições e quais nomes funcionam em quais contextos —, veja [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

### Com a opção Índice marcada, mas sem LUT

A imagem é exibida em **escala de cinza**, estendida entre os dois valores de limiar. Isso é intencional: a imagem de índice é composta por dados escalares, e a escala de cinza é a representação fiel desses dados. Adicione uma LUT quando desejar cores.

***

## Trabalhando com LUTs (Tabelas de Consulta)

Uma **tabela de consulta** mapeia valores de índice para cores: insira NDVI 0,65 e obtenha um verde específico. Ela não altera os dados — apenas muda a forma como você os interpreta.

### Adicionando uma LUT

1. Clique no botão **“+ Adicionar LUT”** <img src="../.gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> abaixo da fórmula
2. Escolha um gradiente de cores
3. Defina o mínimo e o máximo de recorte
4. Escolha um modo de recorte
5. Marque a caixa **LUT** na barra lateral para renderizá-la

A caixa de seleção **LUT** permanece desativada até que uma LUT tenha sido efetivamente configurada no índice.

### Escolhendo um gradiente de cor

Passe o mouse sobre a **barra de gradiente**para abrir a lista de predefinições — o Chloros vem com**sete** predefinições de gradiente:

| # | Gradiente                            | Forma                                                               |
| - | ----------------------------------- | ------------------------------------------------------------------- |
| 1 | Red → Amarelo → Green (**padrão**)  | Divergente — corresponde à percepção comum da vegetação: verde = saudável |
| 2 | Roxo → Amarelo → Green             | Divergente, com uma extremidade inferior distinta                                  |
| 3 | Marrom → Branco → Blue                | Divergente em torno de um ponto médio claro                                   |
| 4 | Preto → Roxo → Rosa → Amarelo claro | Sequencial, do escuro para o claro                                           |
| 5 | Red → Amarelo → Blue                 | Divergente em torno de um ponto médio claro                                   |
| 6 | Roxo → Blue → Green → Amarelo      | Sequencial, do escuro para o claro                                           |
| 7 | Orange → Branco → Roxo             | Divergente em torno de um ponto médio claro                                   |

Um gradiente **divergente**coloca uma cor neutra no meio da janela, o que funciona bem quando o ponto médio tem um significado específico (um limite, uma data de referência). Um gradiente**sequencial** varia monotonamente do escuro para o claro, o que funciona bem para uma quantidade que apenas tem “mais” e “menos”.

Cada predefinição possui sete pontos de cor. Clique em uma predefinição e a imagem será atualizada imediatamente (quando a caixa LUT estiver marcada).

### Editando os pontos de cor

Abaixo da barra de gradiente há uma fileira de amostras de cor, uma para cada ponto:

* **Alterar uma cor**: clique em uma amostra para abrir o seletor de cores (roda de cores, controles deslizantes RGB/HSV ou um código hexadecimal, como `#FF0000`)
* **Adicionar um ponto**: clique no botão**+** no final da linha — um ponto branco é adicionado
* **Remover um ponto**:**clique duas vezes** na amostra
* **Manter um gradiente editado**: clique no ícone de salvar ao lado da barra de gradiente para adicionar seu gradiente editado à lista de predefinições, para que você possa selecioná-lo novamente

O gradiente que você configurou em um índice é armazenado junto com esse índice nas configurações do projeto, de modo que ele permanece mesmo após o fechamento e a reabertura do projeto.

**Menos pontos**produzem zonas distintas que se apresentam como uma classificação;**mais pontos** produzem transições suaves, quase fotográficas. Três a cinco pontos são adequados para slides de apresentação e mapas de classificação; seis a dez são adequados para análises gerais; quinze ou mais são adequados para inspeções detalhadas e figuras de publicação.

### Definindo o intervalo de valores

O controle de limite é um **controle deslizante com duas alças**que varia de −1 a +1, com uma caixa de texto editável em cada extremidade para valores exatos e um botão**AUTO**.

* Arraste qualquer uma das alças ou digite um número na caixa correspondente e pressione Enter
* **AUTO**define o intervalo para o**2º e o 98º percentis** dos valores de índice válidos da imagem — um bom ponto de partida que ignora valores atípicos. O Chloros arredonda o resultado de forma adaptativa: para 4 casas decimais em um intervalo muito estreito, 3 em um intervalo estreito e 2 nos demais casos
* Qualquer ajuste manual tem precedência sobre a opção AUTO até que você pressione AUTO novamente

Exemplo de janelas do NDVI:

| Objetivo                                    | Mín.  | Máx. |
| --------------------------------------- | ---- | --- |
| Mostrar tudo                         | −1,0 | 1,0 |
| Apenas vegetação, excluir solo e água | 0,2  | 0,9 |
| Apenas vegetação saudável                | 0,5  | 0,9 |
| Enfatizar estresse                        | 0,2  | 0,5 |

Restringir a janela aumenta o contraste dentro da sua área de interesse e coloca todo o restante fora do intervalo — onde o **Modo de Recorte** decide o que acontece com esses dados.

***

## Modos de recorte

Quando o valor do índice de um pixel fica fora da janela de mínimo/máximo, o Modo de Recorte decide como ele será desenhado.

| Rótulo do menu suspenso                  | Valor armazenado      | Os pixels fora do intervalo são desenhados como                                                                                                |
| ------------------------------- | ----------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Mínimo e Máximo** (padrão) | `clip`            | A cor mais próxima do gradiente — valores abaixo do mínimo assumem a primeira cor, valores acima do máximo assumem a última |
| **Fundo transparente**      | `transparent`     | Totalmente transparente (alfa real)                                                                                                  |
| **Fundo indexado**| `indexColor`      | Escala de cinzas, estendida por todo o**faixa completa** do índice da imagem, de modo que a estrutura fora da faixa ainda fique visível em cinza                |
| **Fundo original**         | `backgroundColor` | A própria imagem subjacente, de modo que a sobreposição de cor fica sobre a cena real                                                |

| Modo                       | Ideal para                               | Aparência                                      |
| -------------------------- | -------------------------------------- | ----------------------------------------- |
| **Mínimo e Máximo**      | Exibição completa dos dados, análise científica | Cada pixel colorido                      |
| **Fundo Transparente** | Sobreposições de SIG, isolando uma faixa de valores   | Cor dentro da janela, nada fora |
| **Fundo indexado**       | Ênfase mantendo o contexto dos dados    | Cor por dentro, cinza por fora               |
| **Fundo original**    | Relatórios e apresentações              | Cor por dentro, fotografia por fora         |

{% hint style="info" %}
**Pixels sem dados são sempre transparentes, em todos os modos.** Um pixel cujo índice não seja finito (uma divisão por 0) ou seja exatamente −1,0 ou +1,0 (sentinelas de saturação, decorrentes de uma faixa apresentar leitura zero enquanto a outra não) é tratado como ausência de dados, em vez de como um valor extremo. Isso mantém realces queimados e sombras perdidas fora da sua escala de cores, em vez de representá-los como a leitura mais extrema no quadro. A mesma regra define quais pixels alimentam os limiares AUTO e o histograma de índices, de modo que os três estejam em concordância.
{% endhint %}

A transparência é preservada quando a exportação é gravada como PNG. Ela não pode ser representada em JPG.

***

## Lendo valores enquanto você ajusta

O painel **Valores do Cursor**, abaixo do painel de configuração, é o instrumento de medição da Sandbox:

* Mova o cursor sobre a imagem e leia os valores de origem por canal, além do valor do índice em sua própria linha
* Ative o botão **ÍNDICE** acima do histograma para ver a distribuição dos valores de índice no quadro, com seus dois limiares de recorte representados por linhas tracejadas laranja e o valor do cursor por uma linha branca — essa é a maneira mais rápida de escolher uma janela que realmente contenha seus dados
* Ative **CURSOR** para ver linhas de marcação nos valores sob o ponteiro
* Amplie além de 60× (menos se um tamanho de bloco GSD estiver definido) para destacar pixels individuais exibidos com um valor flutuante

Uma rotina prática:

1. Anote os valores sobre vegetação saudável, vegetação estressada, solo nu e água
2. Observe onde esses agrupamentos se situam no histograma do índice
3. Defina os valores mínimo e máximo para delimitar o agrupamento de seu interesse
4. Escolha um modo de recorte — _Original Background_ mantém a cena visível ao redor

***

## Exportando da Sandbox

Tudo o que foi mostrado acima é uma pré-visualização ao vivo até que você salve. O botão **Exportar/Salvar imagem(ns)** na parte superior da barra lateral abre um painel que desliza sobre a barra lateral (em vez de cobrir a imagem, para que você ainda possa ver o que está avaliando).

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>### Opções

| Opção                          | Efeito                                                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aplicar à imagem atual**      | Salva exatamente a imagem exibida, com essas configurações                                                                                                |
| **Aplicar a todas as imagens do projeto** | Reexecuta a mesma configuração em todas as imagens do projeto. Imagens sem as faixas necessárias para este índice são ignoradas, não sendo consideradas erros |
| **Barra de gradiente de índice/LUT**      | Também grava uma imagem de legenda separada por exportação, com o intervalo de valores identificado                                                                     |
| **Histograma de índice**             | Também grava uma imagem de histograma separada por exportação, mostrando os valores mínimo e máximo dos dados e os limites de recorte                                               |

Se o **tamanho do bloco GSD** na guia da imagem estiver acima de 1, o painel avisará antes de você confirmar: a exportação salvará o que você está visualizando, incluindo a média por bloco. Defina o controle GSD de volta para 1 primeiro se desejar a resolução total.

### Para onde vão os arquivos

Cada clique em **Exportar**aloca uma**nova pasta, que nunca é reutilizada**:

```
<project folder>/Sandbox_Exports/<IndexName>_<Index|LUT>_<NNN>/
```

Exemplos: `Sandbox_Exports/NDVI_LUT_001/`, depois `Sandbox_Exports/NDVI_LUT_002/` para a próxima execução. A numeração é derivada da varredura do que já está no disco, portanto, ela permanece mesmo após reinicializações e mesmo se você excluir pastas manualmente. Nada é sobrescrito — o objetivo da Sandbox é comparar uma tentativa com a anterior.

Dentro da pasta, por imagem:

| Arquivo                                                   | Conteúdo                                                   |
| ------------------------------------------------------ | ---------------------------------------------------------- |
| `<source name>_<IndexName>_<Index\|LUT>.png`           | A imagem renderizada, pixel a pixel, exatamente como o visualizador exibiu |
| `<source name>_<IndexName>_<Index\|LUT>_legend.png`    | O arquivo “sidecar” da barra de gradiente, se solicitado                     |
| `<source name>_<IndexName>_<Index\|LUT>_histogram.png` | O arquivo “sidecar” do histograma de índice, se solicitado                  |

Os dois arquivos complementares são sempre gravados em **resolução total**, mesmo quando a imagem principal é calculada por média de blocos: o tamanho do bloco corresponde à resolução da tela, e ambos os arquivos complementares leem os valores reais do índice por pixel. Elas também exibem mais informações do que as versões na tela — ambas indicam a janela de alongamento _e_ os valores mínimo e máximo reais dos dados, de modo que uma legenda salva ainda é legível meses depois, mesmo sem o projeto aberto.

### Progresso e resultados

A exportação de todo o projeto leva alguns minutos; portanto, a execução reporta o andamento por meio de um canal de progresso em tempo real, em vez de bloquear o sistema:

* Uma barra de progresso exibe `current / total` e o arquivo que está sendo gravado
* Ao terminar, o painel informa quantas imagens foram exportadas, quantas foram ignoradas e o caminho da pasta de saída
* As imagens ignoradas são listadas com o motivo (são exibidas até cinco; depois, uma linha “+N mais”). O motivo mais comum é uma camada que não possui os canais necessários para esse índice
* Se **nenhuma** imagem do projeto puder usar o índice, a execução reporta falha, em vez de deixar uma pasta vazia

Apenas uma exportação em sandbox é executada por vez. Iniciar uma segunda enquanto outra está em andamento é recusado com uma mensagem clara, em vez de permitir que duas execuções disputem o mesmo arquivo de projeto.

### A grade exibe a execução

Cada execução concluída aparece como um botão próprio na [grade de imagens](image-grid.md) da barra de ferramentas, rotulado como `<IndexName> <Index|LUT> <NNN>`. É assim que você compara execuções: faça duas exportações com diferentes gradientes ou limites e, em seguida, alterne entre os dois botões na grade.

***

## Fórmulas de índice personalizadas (Chloros+)

{% hint style="info" %}
**Onde criá-las**: na barra lateral do Sandbox ou nas**Configurações do projeto** antes do processamento. Ambas são gravadas na mesma lista no nível do projeto.
{% endhint %}

1. Abra a calculadora de fórmulas personalizadas no menu suspenso de fórmulas de índice (requer login com uma assinatura qualificada do Chloros+)
2. Escreva a fórmula usando os **símbolos de faixa e slot** `x`, `y`, `z`, `a`, `b`, `c` — não são nomes de faixas
3. Operadores disponíveis: `+`, `-`, `*`, `/`, `^` e `()` para agrupamento
4. Funções disponíveis: `sqrt()`, `log()`, `ln()`, `abs()`, `sign()`, `log1p()`, `log2()`
5. Nomeie e salve-a — ela aparecerá na parte inferior da lista suspensa de fórmulas e você poderá vincular seus slots arrastando os círculos dos canais, exatamente como em uma predefinição integrada

```

Modified NDVI with an offset:   (y-x)/(y+x+0.5)
Simple ratio:                   y/x
Three-band difference:          (y-x)/(y+x-z)
Squared ratio:                  (y/x)^2
```

{% hint style="warning" %}
**As fórmulas personalizadas estão disponíveis apenas na interface gráfica (GUI).** A opção CLI/SDK `--indices` expande os 22 nomes de predefinições integradas e ignora silenciosamente todo o restante, incluindo suas fórmulas personalizadas. Para processar em lote uma fórmula personalizada, configure-a nas Configurações do Projeto e execute o processamento, ou use a exportação “Aplicar a todas as imagens do projeto” da Sandbox.
{% endhint %}

***

## Solução de problemas

### “Esta camada não possui os canais necessários para este índice”

A fórmula lê uma posição de canal que a camada atual não possui — por exemplo, um índice de três slots em um arquivo de um ou dois canais. Mude para uma camada multibanda (refletância ou sem Bayer) ou escolha um índice compatível com o filtro da sua câmera.

### “Não foi possível acessar o backend de processamento de imagens”

O backend não está respondendo. Verifique a guia “Log”; se o backend estiver reiniciando, o Sandbox se recupera sozinho assim que ele voltar a funcionar.

### A imagem não mudou quando arrastei um círculo

A fórmula ainda não está completa. Uma fórmula incompleta é tratada como um estado normal de arrastamento — nada é renderizado e nada é relatado como erro. Preencha todos os campos utilizados pela fórmula.

### A imagem inteira está de uma única cor

Sua janela de recorte provavelmente está muito fora dos dados. Pressione **AUTO**para alinhá-la aos percentis 2º e 98º, ou ative o histograma**ÍNDICE** para ver onde os dados realmente se encontram.

### As cores exportadas não correspondem ao que eu vi

Elas deveriam — o caminho de exportação é um espelhamento deliberado da visualização ao vivo, incluindo o alfa do modo de recorte, e a média por bloco é aplicada _após_ a colorização, exatamente como o visualizador faz. Se houver diferenças, verifique se o tamanho do bloco GSD não mudou entre a visualização e a exportação.

***

## Próximos passos

* [**Camadas de imagem**](image-layers.md) — em qual camada executar um índice e o que seus valores significam
* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) — a leitura do cursor, o histograma e o controle de GSD em detalhes
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — todas as predefinições, em todas as superfícies
* [**Configurações do projeto**](../project-settings/project-settings.md) — como aplicar as configurações definidas em uma execução de processamento
