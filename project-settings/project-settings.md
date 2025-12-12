Configurações do projeto

A barra lateral Configurações do projeto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral do Chloros permitem configurar todos os aspectos do processamento de imagens, detecção de alvos de calibração, cálculos de índices multiespectrais e opções de exportação para o seu projeto. Essas configurações são salvas com o seu projeto e podem ser salvas como modelos para reutilização em vários projetos.

## Acessando as configurações do projeto

Para acessar as configurações do projeto:

1. Abra um projeto no Chloros
2. Clique na guia **Configurações do projeto**  <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel de configurações exibirá todas as opções de configuração disponíveis organizadas por categoria

***

## Detecção de alvos

Essas configurações controlam como o Chloros detecta e processa alvos de calibração em suas imagens.

### Área mínima da amostra de calibração (px)

* **Tipo**: Número
* **Intervalo**: 0 a 10.000 pixels
* **Padrão**: 25 pixels
* **Descrição**: Define a área mínima (em pixels) necessária para que uma região detectada seja considerada uma amostra válida de alvo de calibração. Valores menores detectarão alvos menores, mas podem aumentar os falsos positivos. Valores maiores exigem regiões-alvo maiores e mais nítidas para detecção.
* **Quando ajustar**:
  * Aumente se você estiver obtendo detecções falsas em pequenos artefatos de imagem.
  * Diminua se seus alvos de calibração parecerem pequenos em suas imagens e não estiverem sendo detectados.

### Agrupamento mínimo de alvos (0-100)

* **Tipo**: Número
* **Intervalo**: 0 a 100
* **Padrão**: 60
* **Descrição**: Controla o limite de agrupamento para agrupar regiões de cores semelhantes ao detectar alvos de calibração. Valores mais altos exigem que cores mais semelhantes sejam agrupadas, resultando em uma detecção de alvo mais conservadora. Valores mais baixos permitem mais variação de cor dentro de um grupo de alvos.
* **Quando ajustar**:
  * Aumente se os alvos de calibração estiverem sendo divididos em várias detecções
  * Diminua se os alvos de calibração com variação de cor não estiverem sendo totalmente detectados

***

## Processamento

Essas configurações controlam como o Chloros processa e calibra suas imagens.

### Correção de vinheta

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Aplica correção de vinheta para compensar o escurecimento da lente nas bordas das imagens. A vinheta é um fenômeno óptico comum em que os cantos e bordas de uma imagem parecem mais escuros do que o centro devido às características da lente.
* **Quando desativar**: Desative apenas se a combinação de câmera/lente já tiver aplicado a correção de vinheta ou se você quiser corrigir manualmente a vinheta no pós-processamento.

### Calibração de refletância/equilíbrio de branco

* **Tipo**: Caixa de seleção
* **Padrão**: Ativado (marcado)
* **Descrição**: Ativa a calibração automática de refletância usando alvos de calibração detectados em suas imagens. Isso normaliza os valores de refletância em todo o seu conjunto de dados e garante medições consistentes, independentemente das condições de iluminação.
* **Quando desativar**: desative apenas se desejar processar imagens brutas e não calibradas ou se estiver usando um fluxo de trabalho de calibração diferente.

### Método Debayer

* **Tipo**: seleção suspensa
* **Opções**:
  * Alta qualidade (mais rápido) - Atualmente, a única opção disponível
* **Padrão**: Alta qualidade (mais rápido)
* **Descrição**: Seleciona o algoritmo de demosaicing usado para converter dados brutos do sensor de padrão Bayer em imagens coloridas. O método “Alta qualidade (mais rápido)” oferece um equilíbrio ideal entre velocidade de processamento e qualidade de imagem.
* **Observação**: Métodos de debayer adicionais podem ser adicionados em versões futuras do Chloros.

### Intervalo mínimo de recalibração

* **Tipo**: Número
* **Intervalo**: 0 a 3.600 segundos
* **Padrão**: 0 segundos
* **Descrição**: Define o intervalo de tempo mínimo (em segundos) entre o uso de alvos de calibração. Quando definido como 0, o Chloros utilizará todos os alvos de calibração detectados. Quando definido para um valor mais alto, o Chloros utilizará apenas alvos de calibração que estejam separados por pelo menos esse número de segundos, reduzindo o tempo de processamento para conjuntos de dados com capturas frequentes de alvos de calibração.
* **Quando ajustar**:
* Defina como 0 para obter a máxima precisão de calibração quando as condições de iluminação variam
  * Aumente (por exemplo, para 60-300 segundos) para um processamento mais rápido quando a iluminação for consistente e você tiver imagens frequentes de alvos de calibração

### Desvio de fuso horário do sensor de luz

* **Tipo**: Número
* **Intervalo**: -12 a +12 horas
* **Padrão**: 0 horas
* **Descrição**: Especifica o deslocamento do fuso horário (em horas a partir do UTC) para os carimbos de data/hora dos dados do sensor de luz. Isso é usado ao processar arquivos de dados PPK (cinemática pós-processada) para garantir a sincronização correta entre as capturas de imagem e os dados GPS.
* **Quando ajustar**: Defina isso para o deslocamento do fuso horário local se seus dados PPK usarem a hora local em vez do UTC. Por exemplo:
  * Hora do Pacífico: -8 ou -7 (dependendo do horário de verão)
  * Hora do Leste: -5 ou -4 (dependendo do horário de verão)
  * Hora da Europa Central: +1 ou +2 (dependendo do horário de verão)

### Aplicar correções PPK

* **Tipo**: Caixa de seleção
* **Padrão**: Desativado (desmarcado)
* **Descrição**: Habilita o uso de correções cinemáticas pós-processadas (PPK) dos gravadores MAPIR DAQ que contêm um GPS (GNSS). Quando habilitado, o Chloros usará quaisquer arquivos de log .daq que contenham dados de pinos de exposição no diretório do seu projeto e aplicará correções precisas de geolocalização às suas imagens.
* **Requisito**: O arquivo de log .daq com entradas de pinos de exposição deve estar presente no diretório do seu projeto
* **Quando ativar**: Recomenda-se sempre ativar a correção PPK se você tiver entradas de feedback de exposição no seu arquivo de log .daq.

### Pino de exposição 1

* **Tipo**: Seleção suspensa
* **Visibilidade**: Visível apenas quando “Aplicar correções PPK” está ativado E os dados de exposição estão disponíveis para o pino 1
* **Opções**:
  * Nomes de modelos de câmera detectados no projeto
  * “Não usar” - Ignorar este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao Pino de exposição 1 para sincronização de tempo PPK. O pino de exposição registra o momento exato em que o obturador da câmera é acionado, o que é fundamental para a geolocalização PPK precisa.
* **Comportamento de seleção automática**:
  * Câmera única + pino único: Seleciona automaticamente a câmera
  * Câmera única + dois pinos: o pino 1 é atribuído automaticamente à câmera
  * Várias câmeras: seleção manual necessária

### Pino de exposição 2

* **Tipo**: seleção suspensa
* **Visibilidade**: visível apenas quando “Aplicar correções PPK” está ativado E os dados de exposição estão disponíveis para o pino 2
* **Opções**:
  * Nomes de modelos de câmera detectados no projeto
  * “Não usar” - Ignora este pino de exposição
* **Padrão**: Selecionado automaticamente com base na configuração do projeto
* **Descrição**: Atribui uma câmera específica ao pino de exposição 2 para sincronização de tempo PPK ao usar uma configuração de câmera dupla.
* **Comportamento de seleção automática**:
  * Câmera única + pino único: O pino 2 é automaticamente definido como “Não usar”
  * Câmera única + dois pinos: o pino 2 é automaticamente definido como “Não usar”
  * Várias câmeras: seleção manual necessária
* **Observação**: a mesma câmera não pode ser atribuída simultaneamente ao pino 1 e ao pino 2.

***

## Índice

Essas configurações permitem que você configure índices multiespectrais para análise e visualização.

### Adicionar índice

* **Tipo**: Painel de configuração de índice especial
* **Descrição**: Abre um painel interativo onde você pode selecionar e configurar índices multiespectrais de vegetação (NDVI, NDRE, EVI, etc.) para calcular durante o processamento da imagem. Você pode adicionar vários índices, cada um com suas próprias configurações de visualização.
* **Índices disponíveis**: O sistema inclui mais de 30 índices multiespectrais predefinidos, incluindo:
  * NDVI (Índice de Vegetação por Diferença Normalizada)
  * NDRE (Diferença Normalizada RedEdge)
  * EVI (Índice de Vegetação Aprimorado)
  * GNDVI, SAVI, OSAVI, MSAVI2
  * E muitos mais (consulte [Fórmulas de Índice Multiespectral](multispectral-index-formulas.md) para obter a lista completa)
* **Recursos**:
  * Selecione entre fórmulas de índice predefinidas
  * Configure gradientes de cores de visualização (LUT - Tabelas de Consulta)
  * Defina valores limite para análise
  * Crie fórmulas de índice personalizadas

### Fórmulas personalizadas (Recurso Chloros+)

* **Tipo**: Matriz de definições de fórmulas personalizadas
* **Descrição**: Permite criar e salvar fórmulas de índice multiespectral personalizadas usando matemática de banda. As fórmulas personalizadas são salvas com as configurações do seu projeto e podem ser usadas da mesma forma que os índices integrados.
* **Como criar**:
  1. No painel de configuração do Índice, procure a opção de fórmula personalizada
  2. Defina sua fórmula usando identificadores de banda (por exemplo, NIR, Red, Green, Blue)
  3. Salve a fórmula com um nome descritivo
* **Sintaxe da fórmula**: Operações matemáticas padrão são suportadas, incluindo:
* Aritmética: `+`, `-`, `*`, `/`
  * Parênteses para ordem das operações
  * Referências de banda: NIR, Red, Green, Blue, RedEdge, Cyan, Orange, NIR1, NIR2

***

## Exportar

Essas configurações controlam o formato e a qualidade das imagens processadas exportadas.

### Formato de imagem calibrado

* **Tipo**: Seleção suspensa
* **Opções**:
  * **TIFF (16 bits)** - Formato TIFF de 16 bits não compactado
  * **TIFF (32 bits, porcentagem)** - TIFF de 32 bits com valores de refletância em porcentagem
  * **PNG (8 bits)** - Formato PNG de 8 bits compactado
  * **JPG (8 bits)** - Formato JPEG de 8 bits compactado
* **Padrão**: TIFF (16 bits)
* **Descrição**: Seleciona o formato de arquivo para salvar imagens processadas e calibradas.
* **Recomendações de formato**:
  * **TIFF (16 bits)**: Recomendado para análises científicas e fluxos de trabalho profissionais. Preserva a máxima qualidade dos dados sem artefatos de compactação. Ideal para análises multiespectrais e processamento adicional em software GIS.
  * **TIFF (32 bits, porcentagem)**: Ideal para fluxos de trabalho que exigem valores de refletância em porcentagem (0-100%). Oferece precisão máxima para medições radiométricas.
  * **PNG (8 bits)**: Ideal para visualização na web e visualização geral. Tamanhos de arquivo menores com compactação sem perdas, mas faixa dinâmica reduzida.
  * **JPG (8 bits)**: Tamanhos de arquivo menores, ideal apenas para visualizações e exibição na web. Utiliza compactação com perdas, o que não é adequado para análises científicas.

***

## Salvar modelo de projeto

Este recurso permite salvar as configurações atuais do projeto como um modelo reutilizável.

* **Tipo**: Entrada de texto + botão Salvar
* **Descrição**: Digite um nome descritivo para o seu modelo de configurações e clique no ícone Salvar. O modelo armazenará todas as configurações atuais do seu projeto (detecção de alvo, opções de processamento, índices e formato de exportação) para fácil reutilização em projetos futuros.
* **Casos de uso**:
  * Crie modelos para diferentes sistemas de câmera (RGB, multiespectral, NIR)
  * Salve configurações padrão para tipos específicos de culturas ou fluxos de trabalho de análise
  * Compartilhe configurações consistentes com toda a equipe
* **Como usar**:
  1. Configure todas as configurações desejadas do projeto
  2. Digite um nome para o modelo (por exemplo, “RedEdge Survey3 NDVI Padrão”)
  3. Clique no ícone Salvar
  4. Agora, o modelo pode ser carregado ao criar novos projetos

***

## Pasta Salvar Projeto

Esta configuração especifica onde os novos projetos são salvos por padrão.

* **Tipo**: Exibição do caminho do diretório + botão Editar
* **Padrão**: `C:\Users\[Username]\Chloros Projects`
* **Descrição**: Mostra o diretório padrão atual onde os novos projetos Chloros são criados. Clique no ícone Editar para selecionar um diretório diferente.
* **Quando alterar**:
  * Defina como uma unidade de rede para colaboração em equipe
  * Altere para uma unidade com mais espaço de armazenamento para grandes conjuntos de dados
  * Organize projetos por ano, cliente ou tipo de projeto em pastas diferentes
* **Observação**: alterar essa configuração afeta apenas projetos NOVOS. Os projetos existentes permanecem em seus locais originais.

***

## Persistência das configurações

Todas as configurações do projeto são salvas automaticamente com o arquivo do projeto (formato de projeto `.mapir`). Quando você reabre um projeto, todas as configurações são restauradas exatamente como você as deixou.

### Hierarquia das configurações

As configurações são aplicadas na seguinte ordem:

1. **Padrões do sistema** - Padrões integrados definidos pelo Chloros
2. **Configurações do modelo** - Se você carregar um modelo ao criar um projeto
3. **Configurações do projeto salvas** - Configurações salvas com o arquivo do projeto
4. **Ajustes manuais** - Quaisquer alterações feitas durante a sessão atual

### Configurações e processamento de imagens

A maioria das alterações nas configurações (especialmente nas categorias Processamento e Exportação) acionará o reprocessamento das imagens para refletir as novas configurações. No entanto, algumas configurações são “somente para exportação” e não requerem reprocessamento imediato:

* Salvar modelo do projeto
* Diretório de trabalho
* Formato de imagem calibrado (aplica-se ao exportar)

***

## Práticas recomendadas

1. **Comece com as configurações padrão**: as configurações padrão funcionam bem para a maioria dos sistemas de câmera MAPIR e fluxos de trabalho típicos.
2. **Crie modelos**: depois de otimizar as configurações para um fluxo de trabalho ou câmera específico, salve-as como um modelo para garantir a consistência entre os projetos.
3. **Teste antes do processamento completo**: Ao experimentar novas configurações, teste em um pequeno subconjunto de imagens antes de processar todo o conjunto de dados.
4. **Documente suas configurações**: use nomes de modelos descritivos que indiquem o sistema de câmera, o tipo de processamento e o uso pretendido (por exemplo, “Survey3\_RGB\_NDVI\_Agricultura”).
5. **Seleção do formato de exportação**: Escolha o formato de exportação com base no uso final:
   * Análise científica → TIFF (16 bits ou 32 bits)
   * Processamento GIS → TIFF (16 bits)
   * Visualização rápida → PNG (8 bits)
   * Compartilhamento na web → JPG (8 bits)

***

Para obter mais informações sobre índices multiespectrais em Chloros, consulte a página [Fórmulas de índices multiespectrais](multispectral-index-formulas.md).
