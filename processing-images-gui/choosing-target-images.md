# Escolha das imagens-alvo

Marcar quais imagens contêm alvos de calibração é uma etapa crucial que acelera significativamente o pipeline de processamento do Chloros. Ao pré-selecionar as imagens-alvo, você elimina a necessidade do Chloros verificar todas as imagens do seu conjunto de dados em busca de alvos de calibração.

## Por que marcar imagens-alvo?

### Velocidade de processamento

Sem marcar as imagens-alvo, o Chloros precisa:

* Verificar todas as imagens do seu projeto
* Executar algoritmos de detecção de alvos em cada imagem
* Verificar centenas ou milhares de imagens desnecessariamente

**Resultado**: o processamento pode demorar significativamente mais tempo, especialmente para conjuntos de dados grandes.

### Com imagens-alvo marcadas

Quando você marca a coluna Alvo para imagens específicas:

* O Chloros verifica apenas as imagens marcadas em busca de alvos
* A detecção de alvos é concluída muito mais rapidamente
* O tempo total de processamento é bastante reduzido

{% hint style=&quot;success&quot; %}
**Melhoria na velocidade**: marcar 2-3 imagens-alvo em um conjunto de dados de 500 imagens pode reduzir o tempo de detecção de alvos de mais de 30 minutos para menos de 1 minuto.
{% endhint %}

***

## Como marcar imagens-alvo

### Etapa 1: identifique suas imagens-alvo

Examine as imagens importadas no Navegador de arquivos e identifique quais imagens contêm alvos de calibração.

**Cenários comuns:**

* **Alvo pré-captura**: capturado antes de iniciar a sessão
* **Alvo pós-captura**: capturado após concluir a sessão
* **Alvos em campo**: alvos colocados dentro da área de captura
* **Vários alvos**: 2-3 imagens-alvo por sessão (recomendado)

### Etapa 2: verifique a coluna Alvo

Para cada imagem que contenha um alvo de calibração:

1. Localize a imagem na tabela do Navegador de arquivos
2. Encontre a coluna **Alvo** (coluna mais à direita)
3. Clique na caixa de seleção na coluna Alvo para essa imagem
4. Repita para todas as imagens que contenham alvos

### Etapa 3: verifique sua seleção

Antes do processamento, verifique novamente:

* [ ] Todas as imagens com alvos de calibração estão marcadas
* [ ] Nenhuma imagem que não seja alvo foi marcada acidentalmente
* [ ] Os alvos estão claramente visíveis nas imagens marcadas

***

## Melhores práticas para imagens-alvo

### Diretrizes para captura de alvos

**Tempo:**

* Capture imagens-alvo imediatamente antes e durante a sessão de captura
* Nas mesmas condições de iluminação do sensor de luz DAQ
* O ideal é capturar imagens de alvo com a maior frequência possível para obter os melhores resultados. Caso contrário, os dados do sensor de luz serão usados para ajustar a calibração ao longo do tempo.

**Posição da câmera:**

* Segure a câmera acima do alvo de forma que ele fique centralizado e ocupe cerca de 40-60% do centro da imagem.
* Mantenha a câmera paralela/nadir à superfície do alvo

**Iluminação:**

* Mesma iluminação ambiente do sensor de luz DAQ
* Evite sombras nas superfícies do alvo
* Não bloqueie a fonte de luz com o corpo, veículo ou vegetação
* Condições nubladas fornecem resultados mais consistentes

**Condição do alvo:**

* Mantenha os painéis do alvo limpos e secos
* Todos os 4 painéis devem estar claramente visíveis e desobstruídos
* Alvos perpendiculares/nadir à fonte de luz, se possível

### Quantas imagens do alvo?

**Mínimo:** 1 imagem alvo por sessão. **Recomendado:** 3-5 imagens alvo por sessão.

**Cronograma de melhores práticas:**

* 3-5 imagens capturadas logo após o sensor de luz começar a gravar
* Gire a câmera entre as capturas para obter os melhores resultados
* Opcional: periodicamente no meio da sessão, se as condições de iluminação mudarem constantemente

***

## Trabalhando com várias câmeras

### Configurações com duas câmeras

Se estiver usando duas câmeras MAPIR simultaneamente (por exemplo, Survey3W RGN + Survey3N OCN):

1. Capture imagens-alvo com **ambas as câmeras** ao mesmo tempo
2. Use o **mesmo alvo físico** para ambas as câmeras
3. Marque imagens-alvo para **ambos os tipos de câmera** no Navegador de Arquivos
4. O Chloros usará alvos apropriados para a calibração de cada câmera

### Coluna Modelo da câmera

A coluna **Modelo da câmera** ajuda a identificar quais imagens vieram de qual câmera:

* Survey3W\_RGN
* Survey3N\_OCN
* Survey3W\_RGB
* etc.

Use esta coluna para verificar se você marcou os alvos para cada tipo de câmera em seu projeto.

***

## Configurações de detecção de alvos

### Ajustando a sensibilidade de detecção

Se o Chloros não estiver detectando seus alvos corretamente, ajuste estas configurações em [Configurações do projeto](adjusting-project-settings.md):

**Área mínima da amostra de calibração:**

* **Padrão**: 25 pixels
* **Aumente** se estiver obtendo detecções falsas em pequenos artefatos
* **Diminua** se os alvos não estiverem sendo detectados

**Agrupamento mínimo de alvos:**

* **Padrão**: 60
* **Aumente** se os alvos estiverem sendo divididos em várias detecções
* **Diminua** se os alvos com variação de cor não estiverem sendo totalmente detectados

***

## Problemas comuns com imagens de alvos

### Problema: Nenhum alvo detectado

**Possíveis causas:**

* Imagens de alvos não marcadas no Navegador de arquivos
* Alvo muito pequeno no quadro (&lt; 30% da imagem)
* Iluminação inadequada (sombras, reflexos)
* Configurações de detecção de alvos muito rígidas

**Soluções:**

1. Verifique se a coluna Alvo está marcada para as imagens corretas
2. Revise a qualidade da imagem do alvo na visualização
3. Recapture os alvos se a qualidade for ruim
4. Ajuste as configurações de detecção de alvos, se necessário

### Problema: Detecções falsas de alvos

**Possíveis causas:**

* Edifícios brancos, veículos ou cobertura do solo confundidos com alvos
* Manchas brilhantes na vegetação
* Sensibilidade de detecção muito baixa

**Soluções:**

1. Marque apenas imagens de alvos reais para limitar o escopo de detecção
2. Aumente a área mínima de amostra de calibração
3. Aumente o valor mínimo de agrupamento de alvos
4. Certifique-se de que as imagens de alvos mostrem apenas o alvo (mínimo de interferência de fundo)

***

## Lista de verificação

Antes de iniciar o processamento, verifique sua seleção de imagens-alvo:

* [ ] Pelo menos 1 imagem-alvo marcada por sessão
* [ ] As caixas de seleção da coluna Alvo estão marcadas para todas as imagens-alvo
* [ ] Imagens-alvo capturadas no mesmo período da pesquisa
* [ ] Alvos claramente visíveis na visualização quando clicados
* [ ] Todos os 4 painéis de calibração visíveis em cada imagem-alvo
* [ ] Sem sombras ou obstruções nos alvos
* [ ] Para câmera dupla: alvos marcados para ambos os tipos de câmera

***

## Processamento sem alvos

### Processamento sem alvos de calibração

Embora não seja recomendado para trabalhos científicos, você pode processar sem alvos:

1. Deixe todas as caixas de seleção da coluna Alvo desmarcadas
2. **Desative** a “Calibração de refletância” nas configurações do projeto
3. A correção de vinheta ainda será aplicada
4. A saída não será calibrada para refletância absoluta

{% hint style=&quot;warning&quot; %}
**Não recomendado**: sem calibração de refletância, os valores de pixel representam apenas o brilho relativo, não medições científicas de refletância. Use alvos de calibração para obter resultados precisos e repetíveis.
{% endhint %}

***

## Próximos passos

Depois de marcar suas imagens-alvo:

1. **Revise suas configurações** - Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
2. **Inicie o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)

Para obter mais informações sobre as metas de calibração, consulte [Metas de calibração](../calibration-targets.md).
