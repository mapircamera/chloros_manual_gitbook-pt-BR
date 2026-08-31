# Escolha das imagens-alvo

Marcar quais imagens contêm alvos de calibração indica ao Chloros exatamente onde procurá-las. Quando pelo menos uma imagem está marcada na coluna “Alvo”, o Chloros digitaliza **apenas as imagens marcadas** — portanto, marcar os alvos é tanto uma forma de acelerar o processamento quanto de evitar que imagens de levantamento sejam confundidas com um alvo.

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## Por que marcar imagens-alvo?

### A marcação controla a varredura

Quando você marca a coluna “Alvo” para imagens específicas:

* O Chloros varre apenas as imagens marcadas em busca de alvos
* A detecção de alvos é concluída muito mais rapidamente
* Imagens de levantamento topográfico não podem gerar detecções falsas de alvos

Se **nenhuma** imagem estiver marcada, o Chloros volta a varrer todas as imagens do projeto:

* Os algoritmos de detecção de alvos são executados em cada imagem
* Centenas ou milhares de imagens são verificadas desnecessariamente
* O processamento demora significativamente mais, especialmente para grandes conjuntos de dados

{% hint style="success" %}
**Melhoria na velocidade**: Marcar 2 a 3 imagens-alvo em um conjunto de dados de 500 imagens pode reduzir o tempo de detecção de alvos de mais de 30 minutos para menos de 1 minuto.
{% endhint %}

***

## Como marcar imagens-alvo

### Passo 1: Identifique suas imagens-alvo

Examine as imagens importadas no Navegador de Arquivos e identifique quais imagens contêm alvos de calibração.

**Cenários comuns:*** **Alvo pré-captura**: Capturado antes do início da sessão
* **Alvo pós-captura**: Capturado após a conclusão da sessão
* **Alvos no campo**: Alvos posicionados dentro da área de captura
* **Vários alvos**: 2 a 3 imagens de alvo por sessão (recomendado)

### Etapa 2: Verifique a coluna **Alvo** <img src="../.gitbook/assets/image (33).png" alt="" data-size="original">

Para cada imagem que contenha um alvo de calibração:

1. Localize a imagem na tabela do Navegador de Arquivos
2. Encontre a coluna **Alvo** (coluna mais à direita)
3. Marque a caixa de seleção na coluna **Alvo** correspondente àquela imagem
4. Repita o procedimento para todas as imagens que contenham alvos

### Etapa 3: Verifique sua seleção

Antes do processamento, verifique novamente:

* [ ] Todas as imagens com alvos de calibração estão marcadas
* [ ] Nenhuma imagem que não seja um alvo está marcada acidentalmente
* [ ] Os alvos estão claramente visíveis nas imagens marcadas

***

## LATTICE: Os alvos são opcionais quando um DAQ está gravando

Para câmeras multiespectrais LATTICE, um alvo de calibração no quadro é **uma das duas** referências de refletância possíveis:

* **Alvo dentro do quadro**: quando uma imagem de alvo marcada passa pelos filtros de qualidade (QA) dChloros, o alvo se torna a**referência absoluta de refletância** para as imagens ao seu redor.
* **Irradiação descendente do DAQ**: quando não há alvo presente (ou a QA falha), o Chloros calcula a refletância a partir da irradiância descendente do sensor de luz do DAQ (ρ = π·L/E). Se uma gravação do `.daq` ou do DAQ-M `.csv` abranger suas capturas, você obterá a refletância calibrada**sem nenhuma imagem-alvo**.

Esse comportamento automático é o padrão. No CLI / SDK, isso corresponde ao `--reflectance-source auto`; você também pode forçar o `target` (rigoroso — sem substituição do DAQ) ou o `daq` (autoridade do DAQ). Consulte a [Referência do CLI](../reference/cli-reference.md#per-product-export-toggles-lattice-multispectral).

**Geometrias de alvos do LATTICE**: além da detecção clássica por painéis usada para Survey3, o processamento do LATTICE suporta**alvos marcados com ArUco**,**alvos com ROI fixa**e**alvos em faixa**, configurados por projeto. Varreduras de refletância**medida** do alvo por unidade podem ser fornecidas por número de série (CLI: `--target-reflectance-dir`, um `<serial>.csv` por unidade-alvo), com os espectros nominais T3/T4P como alternativa.

{% hint style="info" %}
**Módulo F988**: a refletância do F988 é calibrada usando um painel de refletância no local: a banda está além da faixa calibrada do sensor de luz do DAQ; portanto, Chloros aplica sua captura mais recente do painel e a mantém entre as observações do painel. Se um módulo F988 for processado apenas com o DAQ, o Chloros rejeita a refletância baseada no DAQ para essa banda (motivo da omissão: `dls-uncalibrated-band-988`) — o fluxo de trabalho com o painel é o caminho recomendado.
{% endhint %}

***

## Melhores práticas para imagens do alvo

### Diretrizes para captura do alvo

**Tempo:**

* Capture imagens do alvo imediatamente antes e durante toda a sessão de captura
* Nas mesmas condições de iluminação do sensor de luz do DAQ
* O ideal é capturar imagens do alvo com a maior frequência possível para obter os melhores resultados. Caso contrário, os dados do sensor de luz serão usados para ajustar a calibração ao longo do tempo.

**Posição da câmera:**

* Segure a câmera acima do alvo de forma que ele fique centralizado e ocupe cerca de 40 a 60% do centro da imagem.
* Mantenha a câmera paralela/nadir à superfície do alvo

**Iluminação:**

* A mesma iluminação ambiente do seu sensor de luz do DAQ
* Evite sombras nas superfícies do alvo
* Não bloqueie a fonte de luz com seu corpo, veículo ou vegetação
* Condições de céu nublado proporcionam resultados mais consistentes

**Condição do alvo:**

* Mantenha os painéis do alvo limpos e secos
* Todos os painéis do seu alvo (por exemplo, todos os 4 em um T4) devem estar claramente visíveis e desobstruídos
* Posicione os alvos perpendicularmente ou na direção nadir em relação à fonte de luz, se possível

### Quantas imagens do alvo?

**Mínimo:**1 imagem do alvo por sessão.**Recomendado:** 3 a 5 imagens do alvo por sessão.**Cronograma recomendado:**

* 3 a 5 imagens capturadas logo após o sensor de luz iniciar a gravação
* Gire a câmera entre as capturas para obter os melhores resultados
* Opcional: periodicamente no meio da sessão, se as condições de iluminação mudarem constantemente

***

## Trabalhando com várias câmeras

### Configurações com duas câmeras

Se estiver usando duas câmeras MAPIR simultaneamente (por exemplo, Survey3W RGN + Survey3N OCN):

1. Capture imagens-alvo com **ambas as câmeras** ao mesmo tempo
2. Use o **mesmo alvo físico** para ambas as câmeras
3. Marque as imagens dos alvos para **ambos os tipos de câmera** no Navegador de Arquivos
4. O Chloros utilizará os alvos apropriados para a calibração de cada câmera

### Coluna “Modelo da Câmera”

A coluna **Modelo da Câmera** ajuda a identificar quais imagens vieram de qual câmera:

* Survey3W\_RGN
* Survey3N\_OCN
* LATT-M3M-L41-F550
* LATT-M3C-L87-FRGN
* etc.

Use essa coluna para verificar se você marcou alvos para cada tipo de câmera em seu projeto.

***

## Configurações de detecção de alvos

### Ajustando a sensibilidade de detecção

Se o Chloros não estiver detectando seus alvos corretamente, ajuste estas configurações em [Configurações do projeto](adjusting-project-settings.md):**Área mínima da amostra de calibração (px):*** **Padrão**: 25 pixels
* **Aumente** se houver detecções falsas em pequenos artefatos
* **Diminua** se os alvos não estiverem sendo detectados**Agrupamento mínimo de alvos (0-100):*** **Padrão**: 60
* **Aumente** se os alvos estiverem sendo divididos em várias detecções
* **Diminua** se alvos com variação de cor não estiverem sendo totalmente detectados

{% hint style="info" %}
**Dica do CLI**: o `chloros-cli process` aceita os mesmos parâmetros (`--min-target-size`, `--target-clustering`), e seu sinalizador `--target`/`--targets` marca uma pasta de entrada inteira como “somente painel de alvos”. Consulte a [Referência do CLI](../reference/cli-reference.md).
{% endhint %}

***

## Problemas comuns com imagens-alvo

### Problema: Nenhum alvo detectado

**Possíveis causas:**

* Imagens-alvo não marcadas no Navegador de Arquivos
* Alvo muito pequeno no quadro (&lt; 30% da imagem)
* Iluminação inadequada (sombras, reflexos)
* Configurações de detecção de alvo muito restritas

**Soluções:**

1. Verifique se a coluna “Alvo” está marcada para as imagens corretas
2. Verifique a qualidade da imagem do alvo na visualização
3. Refaça a captura dos alvos se a qualidade for ruim
4. Ajuste as configurações de detecção de alvos, se necessário

### Problema: Detecções Falsas de Alvos

**Possíveis causas:**

* Prédios brancos, veículos ou cobertura do solo confundidos com alvos
* Manchas claras na vegetação
* Sensibilidade de detecção muito baixa

**Soluções:**

1. Marque apenas as imagens de alvo reais — somente as imagens marcadas serão analisadas
2. Aumente a área mínima da amostra de calibração
3. Aumente o valor mínimo de agrupamento de alvos
4. Certifique-se de que as imagens de alvo mostrem apenas o alvo (mínimo de interferência no fundo)

***

## Lista de verificação

Antes de iniciar o processamento, verifique sua seleção de imagens de alvo:

* [ ] Pelo menos 1 imagem de alvo marcada por sessão (ou, para o LATTICE, uma gravação `.daq`/`.csv` cobrindo a sessão)
* [ ] As caixas de seleção da coluna “Alvo” estão marcadas para todas as imagens-alvo
* [ ] Imagens-alvo capturadas no mesmo intervalo de tempo da pesquisa
* [ ] Alvos claramente visíveis na visualização prévia ao clicar neles
* [ ] Todos os painéis de calibração visíveis em cada imagem-alvo
* [ ] Sem sombras ou obstruções nos alvos
* [ ] Para câmera dupla: alvos marcados para ambos os tipos de câmera

***

## Processamento sem alvos

### LATTICE: com uma gravação DAQ

Se um sensor de luz DAQ registrou a irradiação descendente durante suas capturas do LATTICE, não é necessário nenhum alvo:

1. Importe o arquivo `.daq` (ou DAQ-M `.csv`) com as imagens
2. Deixe a coluna “Alvo” desmarcada
3. A refletância é calculada automaticamente a partir da referência de irradiância descendente do DAQ
4. A radiancia nunca precisa de um alvo ou de um DAQ — ela é obtida exclusivamente a partir da calibração radiométrica de fábrica da câmera

### Processamento sem nenhuma referência

Você também pode processar sem alvos e sem um DAQ:

1. Deixe todas as caixas de seleção da coluna “Alvo” desmarcadas
2. **Desative** “Calibração de refletância / balanço de branco” nas Configurações do Projeto — a detecção de alvo será, então, totalmente ignorada
3. A correção de vinheta ainda será aplicada
4. A saída não será calibrada para refletância absoluta (o LATTICE multiespectral ainda exporta produtos sem debayering, de visualização e de radiancia)

{% hint style="warning" %}
**Não recomendado para trabalhos científicos dSurvey3**: sem a calibração de refletância, os valores de píxeis dSurvey3 representam apenas brilho relativo, e não medições científicas de refletância. Use alvos de calibração (ou, para o LATTICE, um sensor de luz DAQ) para obter resultados precisos e repetíveis.
{% endhint %}

***

## Próximos passos

Depois de marcar suas imagens-alvo:

1. **Verifique suas configurações** — Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
2. **Inicie o processamento** — consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o andamento** — Consulte [Monitorando o processamento](monitoring-the-processing.md)

Para obter mais informações sobre os alvos de calibração em si, consulte [Alvos de calibração](../calibration-targets.md).
