# Ajustando as configurações do projeto

Antes de processar suas imagens, é importante configurar as configurações do projeto para atender aos requisitos do seu fluxo de trabalho. O painel Configurações do projeto <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> oferece controle abrangente sobre calibração, opções de processamento, índices multiespectrais e formatos de exportação.

## Acessando as configurações do projeto

1. Abra seu projeto no Chloros
2. Clique no ícone **Configurações do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Configurações do projeto exibe todas as opções de configuração

{% hint style=&quot;info&quot; %}
**As configurações são salvas automaticamente** com o seu projeto. Quando você reabre um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Configurações padrão (recomendadas para a maioria dos usuários)

Para fluxos de trabalho típicos da câmera MAPIR Survey3, as configurações padrão funcionam bem:

* ✅ **Correção de vinheta**: ativada
* ✅ **Calibração de refletância**: ativada (requer imagens de alvos MAPIR)
* ✅ **Método Debayer**: Alta qualidade (mais rápido)
* ✅ **Formato de exportação**: TIFF (16 bits)

Basta importar suas imagens e iniciar o processamento com essas configurações padrão.

***

## Visão geral das configurações do projeto

O painel Configurações do projeto está organizado em várias categorias. Abaixo está um resumo de cada seção. Para obter a documentação completa, consulte [Configurações do projeto](../project-settings/project-settings.md).

### Detecção de alvos

Controla como o Chloros identifica alvos de calibração em suas imagens.

**Configurações principais:**

* **Área mínima da amostra de calibração**: limite de tamanho para detecção de alvos (padrão: 25 pixels)
* **Agrupamento mínimo de alvos**: limite de similaridade para agrupar regiões-alvo (padrão: 60)

**Quando ajustar:**

* Aumente a área da amostra se houver detecções falsas
* Diminua se os alvos não estiverem sendo detectados
* Ajuste o agrupamento se os alvos estiverem sendo divididos em várias detecções

### Processamento

Principais opções de processamento e calibração de imagens.

**Configurações principais:**

* **Correção de vinheta**: Compensa o escurecimento das lentes nas bordas ✅ Recomendado
* **Calibração de refletância**: Normaliza os valores usando alvos de calibração ✅ Recomendado
* **Método Debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: tempo entre o uso de alvos de calibração (0 = usar todos)

**Configurações avançadas:**

* **Desvio de fuso horário do sensor de luz**: para sincronização de tempo PPK (padrão: 0)
* **Aplicar correções PPK**: usa dados de GPS/pino de exposição de arquivos .daq
* **Pino de exposição 1/2**: atribui câmeras a pinos de exposição para configurações de câmera dupla

### Índice (índices multiespectrais)

Configure quais índices de vegetação calcular e exportar.

**Como adicionar índices:**

1. Clique no botão **“Adicionar índice”**
2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as definições de visualização (cores LUT, intervalos de valores)
4. Adicione vários índices, conforme necessário

**Índices populares:**

* **NDVI**: Saúde geral da vegetação (mais comum)
* **NDRE**: Detecção precoce de estresse com RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com alto índice de área foliar (LAI)

**Fórmulas personalizadas (somente Chloros+):**

* Crie fórmulas de índice multiespectral personalizadas
* Use matemática de banda com todos os canais de imagem
* Salve fórmulas personalizadas para reutilização

Para todos os índices e fórmulas disponíveis, consulte [Fórmulas de Índice Multiespectral](../project-settings/multispectral-index-formulas.md).

### Exportar

Controla o formato e a qualidade do arquivo de saída.

**Formatos disponíveis:**

* **TIFF (16 bits)**: recomendado para GIS e análise científica (intervalo de 0 a 65.535)
* **TIFF (32 bits, porcentagem)**: valores de refletância de ponto flutuante (intervalo de 0,0 a 1,0)
* **PNG (8 bits)**: Compressão sem perdas para visualização (intervalo de 0 a 255)
* **JPG (8 bits)**: Arquivos menores, compressão com perdas (intervalo de 0 a 255)

***

## Salvando e carregando configurações

### Salvar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as configurações desejadas no painel Configurações do projeto
2. Role até a seção **“Salvar modelo do projeto”** na parte inferior
3. Digite um nome descritivo para o modelo (por exemplo, “Survey3N\_RGN\_Agricultura”)
4. Clique no ícone Salvar

**Benefícios:**

* Aplique configurações idênticas em vários projetos
* Compartilhe configurações com membros da equipe
* Mantenha a consistência para pesquisas repetidas

### Carregar modelo em novo projeto

Ao criar um novo projeto:

1. Selecione **“Novo projeto”** no menu principal
2. Escolha a opção **“Carregar do modelo”**
3. Selecione seu modelo salvo
4. Todas as configurações são aplicadas automaticamente

### Diretório de trabalho

A configuração **“Salvar pasta do projeto”** especifica onde novos projetos são criados por padrão:

* **Local padrão**: `C:\Users\[Username]\Chloros Projects`
* **Alterar local**: clique no ícone de edição e selecione uma nova pasta
* **Quando alterar**:
  * Unidade de rede para colaboração em equipe
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração PPK (cinemática pós-processada)

Se estiver usando gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Arquivo de log .daq com entradas de pinos de exposição
* Câmera conectada aos pinos de exposição DAQ durante a sessão de captura

### Etapas de configuração

1. Coloque o arquivo de log .daq na pasta do seu projeto
2. Em Configurações do projeto, marque a caixa de seleção **“Aplicar correções PPK”**
3. Defina **“Fuso horário do sensor de luz”**, se necessário (padrão: 0 para UTC)
4. Atribua câmeras aos pinos de exposição:
   * **Câmera única**: Atribuída automaticamente ao pino 1
   * **Câmeras duplas**: atribua manualmente cada câmera ao pino correto

**Atribuição do pino de exposição:**

* **Pino de exposição 1**: selecione o modelo da câmera no menu suspenso
* **Pino de exposição 2**: selecione a segunda câmera ou “Não usar”
* A mesma câmera não pode ser atribuída a ambos os pinos

{% hint style=&quot;warning&quot; %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às respectivas câmeras. A atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmeras

Ao processar imagens de várias câmeras MAPIR em um projeto:

1. Chloros detecta automaticamente cada modelo de câmera
2. Cada câmera recebe o perfil de processamento apropriado
3. PPK: atribua manualmente cada câmera ao pino de exposição correto
4. Todas as câmeras usam o mesmo formato de exportação e índices

**Exemplo**: Survey3W RGN + Survey3N OCN equipamento com duas câmeras

### Levantamentos com intervalos de tempo ou em várias datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com suas configurações padrão
2. Use uma configuração de alvo de calibração consistente em cada sessão
3. Processe cada data como um projeto separado
4. Use configurações idênticas para obter resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Grandes conjuntos de dados

Para projetos com muitas imagens (mais de 500):

* Considere dividir em projetos menores por data ou área
* Use o processamento paralelo Chloros+ para obter resultados mais rápidos
* Considere CLI ou API para automação em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de detecção do alvo

***

## Verificando suas configurações

Antes de iniciar o processamento, revise estas configurações importantes:

* [ ] Modelo da câmera detectado corretamente no Navegador de arquivos
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Pelo menos uma imagem de alvo de calibração importada
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação apropriado para o seu fluxo de trabalho
* [ ] Configurações PPK configuradas (se estiver usando .daq com eventos de exposição)

***

## Próximas etapas

Depois que suas configurações estiverem definidas:

1. **Marque as imagens de calibração** - Consulte [Escolhendo imagens de calibração](choosing-target-images.md)
2. **Inicie o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as configurações disponíveis, consulte a documentação de referência [Configurações do projeto](../project-settings/project-settings.md).
