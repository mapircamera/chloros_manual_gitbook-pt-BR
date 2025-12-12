Camadas de imagem

O menu suspenso Camadas de imagem no Visualizador de imagens Chloros permite alternar rapidamente entre diferentes versões da mesma imagem - desde as capturas originais até saídas de refletância processadas e imagens de índice calculadas.

## O que são camadas de imagem?

No Chloros, **camadas** referem-se às diferentes saídas de imagem disponíveis para uma única imagem de origem. Quando você processa imagens, o Chloros cria várias versões:

* **Imagens originais** (arquivos JPG e RAW da sua câmera)
* Saídas **calibradas por refletância** (se a calibração por refletância estiver ativada)
* **Imagens-alvo** (se a imagem contiver alvos de calibração)
* **Imagens de índice** (NDVI, NDRE, GNDVI, etc., se os índices estiverem configurados)

O **menu suspenso Seletor de camadas** no canto superior direito do Visualizador de imagens permite alternar instantaneamente entre essas versões sem sair do visualizador.

***

## Tipos de camadas disponíveis

### JPG

* A imagem de pré-visualização JPG original da sua câmera
* Sempre disponível para todas as imagens
* Não processada, conforme capturada pela câmera
* Mais rápida para carregar e exibir

**Quando visualizar:**

* Visualização rápida da captura original
* Verificação da composição e do enquadramento da imagem
* Verificação da qualidade da captura antes do processamento

### RAW (Original)

* Os dados originais do sensor RAW da sua câmera
* Desembayered sem pós-processamento aplicado
* Profundidade de bits maior do que JPG (normalmente dados do sensor de 12 ou 14 bits)

**Quando visualizar:**

* Inspecionando a qualidade dos dados originais do sensor
* Verificando se há problemas ou artefatos no sensor
* Comparando os resultados antes e depois do processamento

### RAW (alvo)

* Aparece apenas para imagens identificadas como contendo alvos de calibração
* Mostra a imagem RAW original com o alvo detectado
* Usado para verificar se a detecção do alvo foi bem-sucedida

**Quando visualizar:**

* Confirmar se os alvos de calibração foram detectados corretamente
* Verificar a qualidade da imagem do alvo
* Solucionar problemas de calibração

{% hint style=&quot;info&quot; %}
**Camada de alvo**: esta camada só aparece no menu suspenso para imagens que contêm alvos de calibração. Imagens de captura regulares não terão essa opção.
{% endhint %}

### RAW (Refletância)

* A imagem de saída de refletância calibrada
* Vinheta corrigida (se ativada no processamento)
* Refletância calibrada usando dados de alvo (se ativada)
* Multibanda TIFF com todos os canais da câmera
* Os valores de pixel representam a porcentagem de refletância (ao usar o modo de porcentagem)
* Pronto para manipulação com o [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizar:**

* Inspecionando resultados calibrados
* Verificando a qualidade da calibração
* Verificando os valores dos pixels para precisão científica
* Comparando com o original para ver os efeitos da calibração

{% hint style=&quot;success&quot; %}
**Recomendado**: Use a camada RAW (Refletância) ao verificar os valores dos pixels para medições e análises científicas.
{% endhint %}

### RAW (NDVI Índice)... e similares

* Imagem do índice de vegetação calculado (NDVI neste exemplo)
* O nome do índice muda com base no índice que foi configurado durante o processamento
* Exemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagem em escala de cinza de banda única mostrando os resultados do cálculo do índice
* Uma camada aparece para cada índice configurado nas Configurações do projeto

**Nomes de índices possíveis:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* E muitos mais... (consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md))

**Quando visualizar:**

* Examinando os resultados do cálculo do índice
* Verificando os intervalos de valores do índice
* Identificando áreas de interesse
* Verificando as imagens do índice antes de usá-las no GIS ou na análise

***

## Usando o seletor de camadas

### Abrindo o menu suspenso

1. Abra uma imagem no modo de tela cheia (clique em qualquer miniatura no Visualizador de imagens)
2. Localize o **menu suspenso de camadas** no canto superior direito do visualizador
3. O menu suspenso mostra a camada selecionada no momento (por exemplo, “JPG”)
4. Clique no menu suspenso para ver todas as camadas disponíveis

### Alternando camadas

1. Clique no menu suspenso de camadas para abrir a lista
2. Todas as camadas disponíveis para a imagem atual são exibidas
3. Clique em qualquer nome de camada para alternar para essa versão
4. A imagem é atualizada imediatamente para mostrar a camada selecionada

**Troca rápida:**

* O menu suspenso lembra sua última seleção
* Ao navegar para a próxima imagem, o Chloros tenta mostrar o mesmo tipo de camada
* Se essa camada não existir na próxima imagem, o padrão será JPG

### Disponibilidade de camadas

Nem todas as camadas estão disponíveis para todas as imagens:

**Sempre disponível:**

* ✅ JPG (todas as imagens têm uma pré-visualização JPG)

**Disponível condicionalmente:**

* ⚠️ RAW (Original) - Apenas se a imagem foi capturada no modo RAW ou RAW+JPG
* ⚠️ RAW (Alvo) - Apenas se a imagem contiver alvos de calibração detectados
* ⚠️ RAW (Refletância) - Somente após o processamento com a calibração de refletância ativada
* ⚠️ RAW (\[Índice] Índice) - Somente após o processamento com índices configurados

***

## Persistência da camada

### Navegando entre imagens

Quando você navega para uma imagem diferente (usando as setas do teclado ou clicando nas miniaturas):

**A preferência da camada é preservada:**

* Se estiver visualizando “RAW (Refletância)”, a próxima imagem mostrará “RAW (Refletância)” (se disponível)
* Se estiver visualizando “RAW (Índice NDVI)”, a próxima imagem mostrará “RAW (Índice NDVI)” (se disponível)
* Se a mesma camada não existir, o padrão será JPG

**Exemplo de fluxo de trabalho:**

1. Abra a Imagem 1, mude para RAW (NDVI Index)
2. Pressione → para visualizar a Imagem 2
3. A Imagem 2 exibe automaticamente a camada RAW (NDVI Index)
4. Continue navegando - todas as imagens mostram a camada NDVI
5. Muito eficiente para revisar os resultados do índice em várias imagens

***

## Fluxos de trabalho comuns

### Fluxo de trabalho 1: Comparação antes/depois

**Objetivo**: Comparar a imagem original com a imagem calibrada

1. Abra a imagem processada no Visualizador de imagens
2. Selecione **RAW (Original)** no menu suspenso
3. Observe os valores de vinheta e não calibrados
4. Alterne para **RAW (Refletância)** no menu suspenso
5. Compare - vinheta removida, valores calibrados

### Fluxo de trabalho 2: Revisão do índice

**Objetivo**: Revisar rapidamente os resultados NDVI em todo o conjunto de dados

1. Abra a primeira imagem processada
2. Selecione **RAW (Índice NDVI)** no menu suspenso
3. Use a tecla de seta → para navegar até a próxima imagem
4. A camada NDVI persiste automaticamente
5. Continue por todas as imagens, verificando os padrões NDVI
6. Alterne para **RAW (Índice NDRE)** para comparar

### Fluxo de trabalho 3: Verificação do alvo

**Objetivo**: Verificar se todas as imagens-alvo foram detectadas corretamente

1. Navegue até uma imagem-alvo
2. Selecione **RAW (Alvo)** no menu suspenso
3. Verifique se os alvos de calibração estão claramente visíveis e foram detectados
4. Navegue até a próxima imagem alvo
5. Repita a verificação para todos os alvos

### Fluxo de trabalho 4: Inspeção do valor do pixel

**Objetivo**: Verificar os valores de refletância para precisão científica

1. Abra a imagem processada
2. Selecione a camada **RAW (Refletância)**
3. Ative o modo **Porcentagem de pixel** (botão na barra de ferramentas superior direita)
4. Mova o cursor sobre as áreas de vegetação
5. Verifique se os valores dos pixels estão dentro dos intervalos esperados (30-70% para NIR, 5-15% para Red)
6. Verifique se as áreas de solo e água apresentam valores adequados

***

## Compreensão dos valores dos pixels por camada

Camadas diferentes mostram intervalos de valores de pixels diferentes:

### Camada JPG

* **Intervalo**: 0-255 (8 bits)
* **Significado**: Valores exibidos, com correção gama
* **Uso**: Apenas para inspeção visual, não para medições científicas

### RAW (Original)

* **Intervalo**: 0-65535 (16 bits)
* **Significado**: Números digitais brutos do sensor
* **Uso**: Verificação do desempenho do sensor, não calibrado

### RAW (Refletância)

* **Intervalo**: 0-65.535 (16 bits TIFF) ou 0,0-1,0 (32 bits Porcentagem)
* **Significado**: Porcentagem de refletância calibrada
* **Uso**: medições e análises científicas

**Para 16 bits TIFF:** divida por 65.535 para obter a porcentagem de refletância **Para 32 bits Porcentagem:** os valores representam diretamente a porcentagem (0,5 = 50% de refletância)

### RAW (Imagens de índice)

* **Intervalo**: Varia de acordo com o índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: Resultado do cálculo do índice
* **Exemplos**:
  * NDVI: -1 a +1 (vegetação normalmente 0,4 a 0,9)
  * NDRE: -1 a +1 (detecção de estresse)
  * EVI: 0 a 1 (vegetação aprimorada)

***

## Dicas e melhores práticas

### Troca eficiente de camadas

* **Conhecimento dos atalhos do teclado**: embora não haja atalhos do teclado para camadas, as setas de navegação (←/→) funcionam em todas as camadas
* **Fluxos de trabalho consistentes**: escolha uma camada (por exemplo, NDVI) e revise todo o conjunto de dados antes de mudar para outra
* **Comparações rápidas**: alterne entre Original e Refletância para verificar a qualidade do processamento

### Considerações sobre desempenho

* **JPG carrega mais rápido**: use para navegação rápida por muitas imagens
* **Camadas RAW carregam mais lentamente**: resolução e profundidade de bits mais altas
* **Camadas de índice**: velocidade semelhante às camadas de refletância
* **O primeiro carregamento é o mais lento**: As visualizações subsequentes da mesma camada são armazenadas em cache e são mais rápidas

### Verificação de qualidade

* **Sempre verifique o RAW (Original)**: verifique a qualidade dos dados de origem antes de confiar nos resultados processados
* **Compare camadas**: use a alternância de camadas para validar se o processamento funcionou corretamente
* **Verifique os intervalos de índice**: use o modo Porcentagem de pixels com camadas de índice para verificar se os valores são razoáveis

***

## Solução de problemas

### Camada indisponível

**Problema**: A camada esperada não aparece no menu suspenso

**Possíveis causas:**

* A imagem não foi processada (apenas JPG e RAW (Original) disponíveis)
* A calibração da refletância foi desativada durante o processamento
* O índice específico não foi configurado nas configurações do projeto
* A imagem é uma imagem apenas de destino (nenhum índice gerado para destinos)

**Soluções:**

1. Verifique se a imagem foi processada (verifique a pasta de saída para arquivos processados)
2. Verifique as configurações do projeto para confirmar se os índices foram configurados
3. Reprocesse com os índices desejados ativados

### Camada errada exibida

**Problema**: A imagem abre em uma camada inesperada

**Causa**: A preferência de camada da imagem anterior foi transferida, mas essa camada não existe na imagem atual

**Solução:** O Chloros volta automaticamente para JPG quando a camada preferida não está disponível - este é um comportamento normal

### Não é possível ver os alvos de calibração

**Problema:** A camada RAW (alvo) não mostra a detecção do alvo

**Possíveis causas:**

* Os alvos não foram detectados durante o processamento
* A imagem não contém realmente alvos
* As configurações de detecção de alvos são muito rígidas

**Soluções:**

1. Verifique o log de depuração em busca de mensagens “Alvo encontrado”
2. Verifique se a imagem realmente contém alvos de calibração visíveis
3. Ajuste as configurações de detecção de alvos nas Configurações do projeto
4. Consulte [Escolhendo imagens-alvo](../processing-images-gui/choosing-target-images.md)

***

## Recursos relacionados

### Ferramentas do visualizador de imagens

Ao visualizar qualquer camada, você pode usar:

* **Controles de zoom**: amplie para inspecionar detalhes
* **Panorâmica**: clique e arraste para mover a imagem ampliada
* **Inspeção do valor do pixel**: veja os valores na localização do cursor
* **Setas de navegação**: mova-se entre as imagens mantendo a camada
* **Modo Porcentagem de Pixel**: alterne entre a exibição DN e porcentagem

Consulte [Abrindo uma imagem em tela cheia](opening-an-image-full-screen.md) para obter a documentação completa do Visualizador de imagens.

### Sandbox de índice/LUT

Para testes e visualização interativos de índices:

* **Cálculo de índice em tempo real**: teste diferentes fórmulas de índice
* **Mapeamento de cores LUT**: aplique gradientes de cores a índices em escala de cinza
* **Exportar visualizações**: salve imagens de índice coloridas

Consulte [Sandbox de índice/LUT](index-lut-sandbox.md) para obter detalhes.

***

## Próximos passos

Agora que você entende as camadas de imagem:

* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) - Guia completo do Visualizador de Imagens
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Visualização interativa de índices
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Referência de índices disponíveis
* [**Concluindo o processamento**](../processing-images-gui/finishing-the-processing.md) - Compreendendo os resultados processados
