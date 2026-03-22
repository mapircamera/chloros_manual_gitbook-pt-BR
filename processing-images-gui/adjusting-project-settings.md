# Ajustando as configurações do projeto

Antes de processar suas imagens, é importante configurar as definições do projeto de acordo com os requisitos do seu fluxo de trabalho. O painel Configurações do projeto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferece controle abrangente sobre calibração, opções de processamento, índices multiespectrais e formatos de exportação.

## Acessando as configurações do projeto

1. Abra seu projeto no Chloros
2. Clique no ícone **Configurações do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Configurações do projeto exibe todas as opções de configuração

{% hint style="info" %}
**As configurações são salvas automaticamente** com o seu projeto. Ao reabrir um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Configurações padrão (recomendadas para a maioria dos usuários)

Para fluxos de trabalho típicos de câmeras MAPIR e Survey3, as configurações padrão funcionam bem:

* ✅ **Correção de vinheta**: Ativada
* ✅ **Calibração de refletância**: Ativada (requer imagens de alvos MAPIR)
* ✅ **Método de debayer**: Padrão (Rápido, Qualidade Média)
* ✅ **Formato de exportação**: TIFF (16 bits)

Basta importar suas imagens e iniciar o processamento com essas configurações padrão.

***

## Visão geral das configurações do projeto

O painel Configurações do projeto está organizado em várias categorias. Abaixo está um resumo de cada seção. Para a documentação completa, consulte [Configurações do projeto](../project-settings/project-settings.md).

### Detecção de alvos

Controla como o Chloros identifica alvos de calibração em suas imagens.

**Configurações principais:*** **Área mínima da amostra de calibração**: Limite de tamanho para detecção de alvos (padrão: 25 pixels)
* **Agrupamento mínimo de alvos**: Limite de similaridade para agrupar regiões-alvo (padrão: 60)**Quando ajustar:**

* Aumente a área da amostra se houver detecções falsas
* Diminua se os alvos não estiverem sendo detectados
* Ajuste o agrupamento se os alvos estiverem sendo divididos em múltiplas detecções

### Processamento

Principais opções de processamento de imagem e calibração.

**Configurações principais:*** **Correção de vinheta**: Compensa o escurecimento da lente nas bordas ✅ Recomendado
* **Calibração de refletância**: Normaliza valores usando alvos de calibração ✅ Recomendado
* **Método de debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo entre o uso de alvos de calibração (0 = usar todos)**Configurações avançadas:*** **Deslocamento de fuso horário do sensor de luz**: Para sincronização de tempo PPK (padrão: 0)
* **Aplicar correções PPK**: Usa dados de GPS/pinos de exposição de arquivos .daq
* **Pino de exposição 1/2**: Atribui câmeras aos pinos de exposição para configurações com duas câmeras

### Método de debayer

Atualmente, oferecemos 2 métodos de debayer no Chloros:

#### Padrão (Rápido, Qualidade Média)

O debayer padrão processa rapidamente, mas apresenta ruído de cor no debayering, resultando em imagens menos precisas e com mais ruído.

#### Sensível à textura (Lento, Qualidade máxima) \[Apenas no Chloros+]

O Sensível à textura utiliza um debayer de alta qualidade sensível às bordas, combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do debayering. O modelo Sensível à Textura requer memória da GPU (VRAM) para ser executado. Recomendamos usá-lo quando você tiver &gt;4 GB de VRAM disponível para um processamento mais rápido.

### Índice (Índices Multiespectrais)

Configure quais índices de vegetação calcular e exportar.

**Como adicionar índices:**

1. Clique no botão**&quot;Adicionar índice&quot;**

2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as definições de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices conforme necessário

**Índices populares:*** **NDVI**: Saúde geral da vegetação (mais comum)
* **NDRE**: Detecção precoce de estresse com RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com alto índice de área foliar (LAI)**Fórmulas personalizadas (somente Chloros+):**

* Crie fórmulas de índices multiespectrais personalizadas
* Use cálculos matemáticos de bandas com todos os canais da imagem
* Salve fórmulas personalizadas para reutilização

Para todos os índices e fórmulas disponíveis, consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla o formato e a qualidade do arquivo de saída.

**Formatos disponíveis:*** **TIFF (16 bits)**: Recomendado para GIS e análise científica (intervalo de 0 a 65.535)
* **TIFF (32 bits, Porcentagem)**: Valores de refletância em ponto flutuante (intervalo de 0,0 a 1,0)
* **PNG (8 bits)**: Compressão sem perdas para visualização (intervalo de 0 a 255)
* **JPG (8 bits)**: Arquivos menores, compressão com perdas (intervalo de 0 a 255)***

## Salvando e carregando configurações

### Salvar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as definições desejadas no painel **Configurações do projeto**

2. Role até a seção**&quot;Salvar modelo do projeto&quot;** na parte inferior
3. Insira um nome descritivo para o modelo (por exemplo, &quot;Survey3N\_RGN\_Agricultura&quot;)
4. Clique no ícone de salvar

**Benefícios:**

* Aplique configurações idênticas em vários projetos
* Compartilhe configurações com membros da equipe
* Mantenha a consistência em pesquisas repetidas

### Carregar modelo em novo projeto

Ao criar um novo projeto:

1. Selecione **&quot;Novo projeto&quot;** no menu principal
2. Escolha a opção **&quot;Carregar a partir do modelo&quot;**

3. Selecione o modelo salvo
4. Todas as configurações são aplicadas automaticamente

### Diretório de trabalho

A configuração **&quot;Pasta para salvar projeto&quot;** especifica onde novos projetos são criados por padrão:

* **Localização padrão**: `C:\Users\[Username]\Chloros Projects`
* **Alterar localização**: Clique no ícone de edição e selecione uma nova pasta
* **Quando alterar**:
  * Unidade de rede para colaboração em equipe
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração PPK (Cinemática Pós-Processada)

Se estiver usando gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Arquivo de log .daq com entradas de pinos de exposição
* Câmera conectada aos pinos de exposição do DAQ durante a sessão de captura

### Etapas de configuração

1. Coloque o arquivo de log .daq na pasta do seu projeto
2. Em Configurações do projeto, marque a caixa de seleção **&quot;Aplicar correções PPK&quot;**

3. Defina**&quot;Desvio de fuso horário do sensor de luz&quot;**, se necessário (padrão: 0 para UTC)
4. Atribua câmeras aos pinos de exposição:
   * **Câmera única**: Atribuída automaticamente ao Pino 1
   * **Câmeras duplas**: Atribua manualmente cada câmera ao pino correto**Atribuição de pinos de exposição:*** **Pino de exposição 1**: Selecione o modelo da câmera no menu suspenso
* **Pino de exposição 2**: Selecione a segunda câmera ou &quot;Não usar&quot;
* A mesma câmera não pode ser atribuída a ambos os pinos

{% hint style="warning" %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às respectivas câmeras. A atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmeras

Ao processar imagens de várias câmeras MAPIR em um projeto:

1. O Chloros detecta automaticamente cada modelo de câmera
2. Cada câmera recebe o perfil de processamento apropriado
3. PPK: Atribua manualmente cada câmera ao pino de exposição correto
4. Todas as câmeras usam o mesmo formato de exportação e índices

**Exemplo**: Survey3W RGN + Survey3N OCN equipamento com duas câmeras

### Levantamentos em time-lapse ou com várias datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com suas configurações padrão
2. Use uma configuração consistente do alvo de calibração em cada sessão
3. Processe cada data como um projeto separado
4. Use configurações idênticas para obter resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Conjuntos de dados grandes

Para projetos com muitas imagens (mais de 500):

* Considere dividir em projetos menores por data ou área
* Use o processamento paralelo Chloros+ para resultados mais rápidos
* Considere o CLI ou o API para automação em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de detecção do alvo

***

## Verificando suas configurações

Antes de iniciar o processamento, verifique estas configurações principais:

* [ ] Modelo da câmera detectado corretamente no Navegador de Arquivos
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Pelo menos uma imagem de alvo de calibração importada
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação adequado para seu fluxo de trabalho
* [ ] Configurações PPK definidas (se estiver usando .daq com eventos de exposição)

***

## Próximos passos

Depois de configurar suas definições:

1. **Marque as imagens-alvo de calibração** - Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
2. **Inicie o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as configurações disponíveis, consulte a documentação de referência [Configurações do projeto](../project-settings/project-settings.md).
