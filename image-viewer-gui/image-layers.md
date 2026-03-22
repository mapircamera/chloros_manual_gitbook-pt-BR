# Camadas de imagem

O menu suspenso “Camadas de imagem” no Visualizador de Imagens Chloros permite alternar rapidamente entre diferentes versões da mesma imagem — desde as capturas originais até resultados de refletância processados e imagens de índice calculadas.

## O que são camadas de imagem?

No Chloros, **camadas** referem-se às diferentes saídas de imagem disponíveis para uma única imagem de origem. Ao processar imagens, o Chloros cria várias versões:

* **Imagens originais** (arquivos JPG e RAW da sua câmera)
* Saídas **calibradas por refletância** (se a calibração por refletância estiver ativada)
* **Imagens-alvo** (se a imagem contiver alvos de calibração)
* **Imagens de índice** (NDVI, NDRE, GNDVI, etc., se os índices tiverem sido configurados)

O **menu suspenso Seletor de Camadas** no canto superior direito do Visualizador de Imagens permite alternar instantaneamente entre essas versões sem sair do visualizador.***

## Tipos de Camadas Disponíveis

### JPG

* A imagem de pré-visualização JPG original da sua câmera
* Sempre disponível para todas as imagens
* Não processada, tal como capturada pela câmera
* Mais rápida para carregar e exibir

**Quando visualizar:**

* Visualização rápida da captura original
* Verificação da composição e do enquadramento da imagem
* Verificação da qualidade da captura antes do processamento

### RAW (Original)

* Os dados originais do sensor RAW da sua câmera
* Desbayered, sem pós-processamento aplicado
* Maior profundidade de bits do que o JPG (normalmente dados do sensor de 12 ou 14 bits)

**Quando visualizar:**

* Inspecionar a qualidade dos dados originais do sensor
* Verificar se há problemas no sensor ou artefatos
* Comparar os resultados antes e depois do processamento

### RAW (Alvo)

* Aparece apenas para imagens identificadas como contendo alvos de calibração
* Mostra a imagem RAW original com o alvo detectado
* Usado para verificar se a detecção do alvo foi bem-sucedida

**Quando visualizar:**

* Para confirmar se os alvos de calibração foram detectados corretamente
* Para verificar a qualidade da imagem do alvo
* Para solucionar problemas de calibração

{% hint style="info" %}
**Camada do Alvo**: Esta camada só aparece no menu suspenso para imagens que contenham alvos de calibração. Imagens de captura regulares não terão esta opção.
{% endhint %}

### RAW (Refletância)

* A imagem de saída com refletância calibrada
* Correção de vinheta (se ativada no processamento)
* Refletância calibrada usando dados de alvo (se ativada)
* Multibanda TIFF com todos os canais da câmera
* Os valores dos pixels representam a porcentagem de refletância (ao usar o modo de porcentagem)
* Pronta para manipulação com o [Index/LUT Sandbox](index-lut-sandbox.md)

**Quando visualizar:**

* Inspecionar resultados calibrados
* Verificar a qualidade da calibração
* Verificar valores de pixels para precisão científica
* Comparar com o original para ver os efeitos da calibração

{% hint style="success" %}
**Recomendado**: Use a camada RAW (Refletância) ao verificar valores de pixels para medições e análises científicas.
{% endhint %}

### RAW (Índice NDVI)... e similares

* Imagem do índice de vegetação calculado (NDVI neste exemplo)
* O nome do índice muda de acordo com o índice configurado durante o processamento
* Exemplos: RAW (Índice NDVI), RAW (Índice NDRE), RAW (Índice GNDVI), etc.
* Imagem em escala de cinza de banda única mostrando os resultados do cálculo do índice
* Uma camada aparece para cada índice configurado nas Configurações do Projeto

**Nomes de índice possíveis:**

* RAW (Índice NDVI)
* RAW (Índice NDRE)
* RAW (Índice GNDVI)
* RAW (Índice OSAVI)
* RAW (Índice EVI)
* RAW (Índice SAVI)
* E muitos mais... (consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md))

**Quando visualizar:**

* Analisar resultados de cálculo de índices
* Verificar intervalos de valores de índices
* Identificar áreas de interesse
* Verificar imagens de índices antes de usá-las em GIS ou análises

***

## Usando o Seletor de Camadas

### Abrindo o menu suspenso

1. Abra uma imagem no modo de tela cheia (clique em qualquer miniatura no Visualizador de Imagens)
2. Localize o **menu suspenso de camadas** no canto superior direito do visualizador
3. O menu suspenso mostra a camada selecionada no momento (por exemplo, “JPG”)
4. Clique no menu suspenso para ver todas as camadas disponíveis

### Alternando entre camadas

1. Clique no menu suspenso de camadas para abrir a lista
2. Todas as camadas disponíveis para a imagem atual são exibidas
3. Clique em qualquer nome de camada para alternar para essa versão
4. A imagem é atualizada imediatamente para mostrar a camada selecionada

**Alternância rápida:**

* O menu suspenso lembra sua última seleção
* Ao navegar para a próxima imagem, o Chloros tenta mostrar o mesmo tipo de camada
* Se essa camada não existir na próxima imagem, o padrão será JPG

### Disponibilidade de camadas

Nem todas as camadas estão disponíveis para todas as imagens:

**Sempre disponíveis:*** ✅ JPG (todas as imagens têm uma pré-visualização em JPG)

**Disponíveis condicionalmente:**

* ⚠️ RAW (Original) - Apenas se a imagem tiver sido capturada no modo RAW ou RAW+JPG
* ⚠️ RAW (Alvo) - Apenas se a imagem contiver alvos de calibração detectados
* ⚠️ RAW (Refletância) - Somente após o processamento com a calibração de refletância ativada
* ⚠️ RAW (\[Índice] Índice) - Somente após o processamento com índices configurados

***

## Persistência da camada

### Navegando entre imagens

Quando você navega para uma imagem diferente (usando as setas do teclado ou clicando nas miniaturas):**A preferência da camada é preservada:**

* Se estiver visualizando &quot;RAW (Refletância)&quot;, a próxima imagem mostra &quot;RAW (Refletância)&quot; (se disponível)
* Se estiver visualizando &quot;RAW (NDVI Índice)&quot;, a próxima imagem mostra &quot;RAW (NDVI Índice)&quot; (se disponível)
* Se a mesma camada não existir, o padrão será JPG

**Exemplo de fluxo de trabalho:**

1. Abra a Imagem 1, alterne para RAW (Índice NDVI)
2. Pressione → para visualizar a Imagem 2
3. A Imagem 2 exibe automaticamente a camada RAW (Índice NDVI)
4. Continue navegando — todas as imagens mostram a camada NDVI
5. Muito eficiente para revisar resultados de índice em várias imagens

***

## Fluxos de trabalho comuns

### Fluxo de trabalho 1: Comparação antes/depois

**Objetivo**: Comparar a imagem original com a imagem calibrada

1. Abra a imagem processada no Visualizador de Imagens
2. Selecione **RAW (Original)** no menu suspenso
3. Observe o efeito de vinheta e os valores não calibrados
4. Mude para **RAW (Refletância)** no menu suspenso
5. Compare — efeito de vinheta removido, valores calibrados

### Fluxo de trabalho 2: Revisão do índice

**Objetivo**: Revisar rapidamente os resultados do NDVI em todo o conjunto de dados

1. Abra a primeira imagem processada
2. Selecione **RAW (Índice NDVI)** no menu suspenso
3. Use a tecla de seta para a direita para navegar até a próxima imagem
4. A camada NDVI é mantida automaticamente
5. Continue por todas as imagens, verificando os padrões NDVI
6. Alterne para **RAW (Índice NDRE)** para comparar

### Fluxo de trabalho 3: Verificação de alvos

**Objetivo**: Verificar se todas as imagens de alvo foram detectadas corretamente

1. Navegue até uma imagem de alvo
2. Selecione **RAW (Alvo)** no menu suspenso
3. Verifique se os alvos de calibração estão claramente visíveis e foram detectados
4. Navegue até a próxima imagem de alvo
5. Repita a verificação para todos os alvos

### Fluxo de trabalho 4: Inspeção do valor dos pixels

**Objetivo**: Verificar os valores de refletância para garantir a precisão científica

1. Abra a imagem processada
2. Selecione a camada **RAW (Refletância)**

3. Ative o modo**Porcentagem de pixels** (botão na barra de ferramentas superior direita)
4. Mova o cursor sobre as áreas de vegetação
5. Verifique se os valores dos pixels estão dentro dos intervalos esperados (30-70% para NIR, 5-15% para Red)
6. Verifique se as áreas de solo e água apresentam valores adequados

***

## Compreensão dos valores dos pixels por camada

Camadas diferentes apresentam intervalos de valores de pixels distintos:

### Camada JPG

* **Intervalo**: 0-255 (8 bits)
* **Significado**: Valores de exibição, com correção de gama
* **Uso**: Apenas para inspeção visual, não para medição científica

### RAW (Original)

* **Intervalo**: 0-65535 (16 bits)
* **Significado**: Números digitais brutos do sensor
* **Uso**: Verificação do desempenho do sensor, não calibrado

### RAW (Refletância)

* **Intervalo**: 0-65.535 (16 bits TIFF) ou 0,0-1,0 (32 bits em porcentagem)
* **Significado**: Refletância percentual calibrada
* **Uso**: Medições e análises científicas**Para 16 bits TIFF:**Divida por 65.535 para obter a refletância percentual**Para 32 bits Porcentagem:** Os valores representam diretamente a porcentagem (0,5 = 50% de refletância)

### RAW (Imagens de Índice)

* **Intervalo**: Varia de acordo com o índice (normalmente de -1,0 a +1,0 para índices normalizados)
* **Significado**: Resultado do cálculo do índice
* **Exemplos**:
  * NDVI: -1 a +1 (vegetação normalmente de 0,4 a 0,9)
  * NDRE: -1 a +1 (detecção de estresse)
  * EVI: 0 a 1 (vegetação aprimorada)

***

## Dicas e melhores práticas

### Troca eficiente de camadas

* **Conhecimento dos atalhos de teclado**: Embora não haja atalhos de teclado para camadas, as setas de navegação (←/→) funcionam em todas as camadas
* **Fluxos de trabalho consistentes**: Escolha uma camada (por exemplo, NDVI) e analise todo o conjunto de dados antes de alternar para outra
* **Comparações rápidas**: Alterne entre Original e Refletância para verificar a qualidade do processamento

### Considerações sobre desempenho

* **JPG carrega mais rápido**: use para navegação rápida por muitas imagens
* **Camadas RAW carregam mais devagar**: maior resolução e profundidade de bits
* **Camadas de índice**: velocidade semelhante às camadas de refletância
* **O primeiro carregamento é o mais lento**: visualizações subsequentes da mesma camada são armazenadas em cache e ficam mais rápidas

### Verificação de qualidade

* **Sempre verifique o RAW (Original)**: Verifique a qualidade dos dados de origem antes de confiar nos resultados processados
* **Compare camadas**: Use a alternância de camadas para validar se o processamento funcionou corretamente
* **Verifique os intervalos de índice**: Use o modo Porcentagem de Píxeis com camadas de índice para verificar se os valores são razoáveis***

## Solução de problemas

### Camada indisponível

**Problema**: A camada esperada não aparece no menu suspenso**Possíveis causas:**

* A imagem não foi processada (apenas JPG e RAW (Original) disponíveis)
* A calibração de refletância foi desativada durante o processamento
* O índice específico não foi configurado nas Configurações do Projeto
* A imagem é uma imagem apenas de alvo (nenhum índice gerado para alvos)

**Soluções:**

1. Verifique se a imagem foi processada (verifique a pasta de saída para arquivos processados)
2. Verifique as Configurações do Projeto para confirmar se os índices foram configurados
3. Reprocesse com os índices desejados ativados

### Camada errada exibida

**Problema**: A imagem abre na camada inesperada**Causa**: A preferência de camada da imagem anterior foi mantida, mas essa camada não existe na imagem atual**Solução**: O Chloros recorre automaticamente ao JPG quando a camada preferida não está disponível — esse é um comportamento normal

### Não é possível ver os alvos de calibração

**Problema**: A camada RAW (Alvo) não mostra a detecção de alvos**Possíveis causas:**

* Os alvos não foram detectados durante o processamento
* A imagem não contém, de fato, alvos
* Configurações de detecção de alvos muito rígidas

**Soluções:**

1. Verifique o Log de Depuração em busca de mensagens &quot;Alvo encontrado&quot;
2. Verifique se a imagem contém, de fato, alvos de calibração visíveis
3. Ajuste as configurações de detecção de alvos nas Configurações do projeto
4. Consulte [Escolhendo imagens de alvo](../processing-images-gui/choosing-target-images.md)

***

## Recursos relacionados

### Ferramentas do visualizador de imagens

Ao visualizar qualquer camada, você pode usar:

* **Controles de zoom**: amplie para inspecionar detalhes
* **Panorâmica**: Clique e arraste para se mover pela imagem ampliada
* **Inspeção do valor do pixel**: Veja os valores na localização do cursor
* **Setas de navegação**: Mova-se entre imagens mantendo a camada
* **Modo Porcentagem de Pixel**: Alterne entre a exibição em DN e em porcentagem

Consulte [Abrindo uma imagem em tela cheia](opening-an-image-full-screen.md) para obter a documentação completa do Visualizador de Imagens.

### Sandbox de Índice/LUT

Para testes interativos de índice e visualização:

* **Cálculo de índice em tempo real**: Teste diferentes fórmulas de índice
* **Mapeamento de cores LUT**: Aplique gradientes de cor a índices em escala de cinza
* **Exportar visualizações**: Salve imagens de índice coloridas

Consulte [Sandbox de Índice/LUT](index-lut-sandbox.md) para obter detalhes.

***

## Próximos passos

Agora que você entende as camadas de imagem:

* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) - Guia completo do Visualizador de Imagens
* [**Área de teste de Índice/LUT**](index-lut-sandbox.md) - Visualização interativa de índices
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) - Referência dos índices disponíveis
* [**Concluindo o processamento**](../processing-images-gui/finishing-the-processing.md) - Compreendendo os resultados processados
