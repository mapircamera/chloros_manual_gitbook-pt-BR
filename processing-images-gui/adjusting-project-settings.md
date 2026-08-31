# Ajustando as configurações do projeto

Antes de processar suas imagens, é importante configurar as definições do projeto de modo a atender aos requisitos do seu fluxo de trabalho. O painel “Configurações do projeto” (<img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line">) oferece controle abrangente sobre calibração, opções de processamento, índices multiespectrais e formatos de exportação.

## Acessando as configurações do projeto

1. Abra seu projeto no Chloros
2. Clique no ícone **Configurações do projeto** <img src="../.gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel “Configurações do projeto” exibe todas as opções de configuração

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption><p>O painel **Configurações do projeto** — Exibição, detecção de alvos e processamento</p></figcaption></figure>{% hint style="info" %}**As configurações são salvas automaticamente** com o seu projeto. Ao reabrir um projeto, todas as configurações são restauradas.
{% endhint %}

***

## Configuração rápida para fluxos de trabalho comuns

### Configurações padrão (recomendadas para a maioria dos usuários)

As configurações padrão funcionam bem para fluxos de trabalho típicos do Survey3 e do LATTICE:

* ✅ **Correção de vinheta**: Ativada
* ✅ **Calibração de refletância / balanço de branco**: Ativada (utiliza alvos MAPIR e/ou dados do sensor de luz DAQ)
* ✅ **Método de debayer**: Padrão (Rápido, Qualidade Média)
* ✅ **Formato de exportação**: TIFF (16 bits)
* ✅ **Todos os produtos de exportação**: Ativado (o LATTICE captura automaticamente a distribuição em debayering, pré-visualização, radiação e refletância)

Basta importar suas imagens e iniciar o processamento com essas configurações padrão.

***

## Visão geral das configurações do projeto

O painel **Configurações do projeto**está organizado nas seções abaixo. Duas seções adicionais —**Sensor de luz DAQ**e**Alinhamento da matriz** — aparecem automaticamente quando seu projeto contém os arquivos relevantes. Para obter a documentação completa, consulte [Configurações do projeto](../project-settings/project-settings.md).

### Exibição

* **Resolução da miniatura da imagem**: Resolução das miniaturas da grade de imagens. Opções:**Padrão (512 px)**,**1024 px**,**2048 px**,**Resolução total**. Apenas para exibição — nunca afeta o processamento. Valores mais altos proporcionam uma imagem mais nítida ao ampliar, mas demoram mais para carregar.

### Detecção de alvos

Controla como o Chloros identifica os alvos de calibração em suas imagens.

**Principais configurações:*** **Área mínima da amostra de calibração (px)**: Limite de tamanho para detecção de alvos (padrão:**25**, intervalo 0–10.000)
* **Agrupamento mínimo de alvos (0–100)**: Limite de similaridade para agrupar regiões-alvo (padrão:**60**)**Quando ajustar:**

* Aumente a área de amostragem se houver detecções falsas
* Diminua se os alvos não estiverem sendo detectados
* Ajuste o agrupamento se os alvos estiverem sendo divididos em várias detecções

{% hint style="info" %}
Essas configurações ficam desativadas quando a **calibração de refletância / balanço de branco** está desativada — com ela desativada, a detecção de alvos simplesmente não é executada.
{% endhint %}

### Processamento

Principais opções de processamento de imagem e calibração.

**Configurações principais:*** **Correção de vinheta**: Compensa o escurecimento da lente nas bordas ✅ Recomendado
* **Calibração de refletância / balanço de branco**: Calibra as imagens usando alvos detectados (Survey3) e/ou dados do sensor de luz DAQ (LATTICE) ✅ Recomendado
* **Método de debayer**: Algoritmo para converter RAW em multiespectral de 3 canais
* **Intervalo mínimo de recalibração**: Tempo mínimo, em segundos, entre o uso de alvos de calibração (padrão:**0** = usar todos, intervalo de 0 a 3.600)**Produtos alternativos não calibrados:**Quando um quadro não pode ser calibrado por refletância (sem alvo disponível ou calibração desativada), ele é exportado como um dos dois produtos alternativos —**existe exatamente um do par por execução**, escolhido pela opção de correção de vinheta:

* **Exportar resposta do sensor**: grava `Sensor_Response_Images` — usado quando a correção de vinheta está**desativada*** **Exportar com correção de vinheta**: grava `Vignette_Corrected_Images` — usado quando a correção de vinheta está**ativada**A caixa de seleção que não está em uso fica desativada. Desmarcar a caixa ativa impede que esse arquivo seja gravado.**Produtos de exportação do LATTICE** (exibidos para todos os projetos; aplicam-se às capturas do LATTICE):

* **Exportar sem debayering**: a imagem linear sem debayering (`Debayered_Images`). Aplica-se a RGB e aos módulos multiespectrais.
* **Exportar pré-visualização**: a pré-visualização exibida (`Preview_Images`). RGB = equilíbrio de branco (iluminante DAQ, quando disponível; caso contrário, mundo cinza) + gama; multiespectral = expansão de cores falsas.
* **Exportação de radianância**: radianância espectral float32 (`Radiance_Images`, W/m²/sr/nm). Apenas módulos multiespectrais — não se aplica aos mestres RGB.
* ****Exportar refletância**: refletância uint16 (`Reflectance_Calibrated_Images`, DN 32768 = ρ 1,0) quando uma leitura descendente `.daq` ou um alvo dentro do quadro cobre o quadro. Apenas módulos multiespectrais.

Todos os quatro estão **ativados por padrão**— um quadro bruto LATTICE importado é distribuído para todos os produtos habilitados e aplicáveis em uma única etapa de processamento. A caixa de seleção**Exportar refletância** fica desativada quando a calibração de refletância está desativada. Configurações cujo botão pai as torna impossíveis estão sempre desativadas, com uma dica de ferramenta indicando o botão a ser alterado.**Configurações avançadas:*** **Deslocamento de fuso horário do sensor de luz**: Horas em relação ao UTC para sincronização do tempo do sensor de luz (padrão: 0, intervalo de −12 a +12)
* **Aplicar correções PPK**: utiliza dados de GPS/pinos de exposição dos arquivos `.daq` (padrão: desativado)
* **Pino de exposição 1/2**: atribui câmeras aos pinos de exposição para configurações com duas câmeras

{% hint style="info" %}
**O nível de entrada do LATTICE é automático.** As capturas do LATTICE carregam seu nível de processamento nos metadados XMP, e o processamento sempre entra no pipeline a partir do quadro bruto — não há nada a ser configurado na interface gráfica. (O sinalizador CLI `--input-level` existe como uma substituição para usuários avançados em capturas com metadados perdidos; consulte a [Referência do CLI](../reference/cli-reference.md).)
{% endhint %}

### Método de remoção de pixelização

Atualmente, oferecemos dois métodos de remoção de pixelização no Chloros:

#### Padrão (Rápido, Qualidade Média)

O método padrão processa rapidamente, mas apresenta ruído de cor na remoção de pixelização, resultando em imagens menos precisas e com mais ruído.

#### Sensível à textura (Lento, qualidade máxima) \[Apenas Chloros+]

O método “Sensível à textura” utiliza um debayer de alta qualidade, sensível a bordas, combinado com um modelo de redução de ruído baseado em IA/ML, que remove quase todo o ruído do processo de remoção do padrão de Bayer. O modelo requer memória da GPU (VRAM) para ser executado: com **7 GB ou mais de VRAM**, ele pode processar várias imagens simultaneamente; com menos de 7 GB, ele processa uma imagem por vez (de forma visivelmente mais lenta). Consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md).

{% hint style="info" %}
**As capturas do LATTICE sempre utilizam o demosaico padrão.**Não existe um modelo Texture Aware treinado para o LATTICE; portanto, a opção não é oferecida para imagens do LATTICE — imagens**Survey3** no mesmo projeto ainda podem utilizá-la.
{% endhint %}

### Índice (Índices Multiespectrais)

Configure quais índices de vegetação devem ser calculados e exportados. O menu suspenso da interface gráfica oferece **27 fórmulas de índices predefinidas**.**Como adicionar índices:**

1. Clique no botão**“Adicionar índice”**

2. Selecione um índice no menu suspenso (NDVI, NDRE, GNDVI, etc.)
3. Configure as opções de visualização (cores da LUT, intervalos de valores)
4. Adicione vários índices conforme necessário

**Índices populares:*** **NDVI**: Saúde geral da vegetação (o mais comum)
* **NDRE**: Detecção precoce de estresse com o RedEdge
* **GNDVI**: Sensível à concentração de clorofila
* **OSAVI**: Funciona bem com solo visível
* **EVI**: Regiões com alto índice de área foliar (LAI)**Fórmulas personalizadas:**

* Crie fórmulas personalizadas de índices multiespectrais com cálculos entre bandas em todos os canais da imagem
* Salve fórmulas personalizadas para reutilização
* As fórmulas personalizadas são um recurso do Chloros+; a disponibilidade depende do seu plano

Para todos os índices e fórmulas disponíveis — incluindo quais nomes são exclusivos da interface gráfica (GUI) e quais também funcionam no CLI/SDK — consulte [Fórmulas de índices multiespectrais](../project-settings/multispectral-index-formulas.md).

### Exportação

Controla o formato do arquivo de saída.

**Formatos disponíveis**(configuração:**Formato de imagem calibrada**, padrão**TIFF (16 bits)**):

* **TIFF (16 bits)**: Recomendado para SIG e análises científicas
* **TIFF (32 bits, porcentagem)**: Valores de ponto flutuante
* **PNG (8 bits)**: Compressão sem perdas para visualização
* **JPG (8 bits)**: Arquivos menores, compressão com perdas

Os arquivos de saída são gravados na pasta do projeto, agrupados por câmera e formato: `<project>/<camera>/<format>/<Product>_Images/`. O Radiance é **sempre** gravado como float32 na pasta `tiff32`, independentemente dessa configuração. Os arquivos exportados mantêm o nome do arquivo de origem — a pasta identifica o produto. Consulte [Concluindo o processamento](finishing-the-processing.md) para ver a árvore completa de saídas.

{% hint style="warning" %}
**Lendo valores de refletância**: o DN que significa ρ = 1,0 depende da câmera de origem — o LATTICE usa 32768 (marcado como XMP `Chloros:PixelScale`), enquanto o Survey3 usa 65535. Leia a tag em vez de supor um valor constante. Consulte [Formatos de imagem de saída](../output-image-formats.md).
{% endhint %}

### Sensor de luz DAQ

Esta seção lista todos os arquivos de radiação descendente do DAQ (`.daq` / `.csv`) em seu projeto, uma linha por arquivo, mostrando o modelo do sensor, o nome do arquivo e a correção da **tampa** do difusor em vigor para esse arquivo.

* **Substituição do limite (todos os arquivos)**: um único menu suspenso para todo o projeto. A opção**Auto** (padrão) utiliza o limite registrado em cada arquivo — presume-se luz solar quando nada foi registrado, já que todos os DAQs MAPIR vêm com o corretor de luz solar. A escolha de um limite substitui todos os arquivos: as gravações brutas são corrigidas com ele, e as gravações que já possuem um limite são reajustadas (a correção registrada é desfeita, e o limite selecionado é aplicado).
* As linhas exibem um aviso quando um limite registrado foi o padrão assumido pelo hub, em vez de ter sido confirmado pelo operador, e quando o limite selecionado não possui perfil para aquele modelo de dispositivo (a substituição é recusada para esse arquivo).

As gravações do DAQ feitas na aba “Sensores de Luz” são adicionadas automaticamente ao projeto aberto, e os arquivos `.daq` / `.csv` importados aparecem aqui assim que são adicionados.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Configurações inferiores do projeto — Índice, formato de exportação, a seção DAQ de sensores de luz e os controles de modelo/pasta do projeto</p></figcaption></figure>### Alinhamento de matriz

Esta seção aparece **apenas** quando pelo menos uma imagem no projeto contém a transformação de alinhamento módulo a módulo que as matrizes LATTICE aplicam na captura (`Chloros:Alignment*` XMP). Ela mostra quantas imagens contêm tags e qual câmera é a referência, com estes controles:

* **Aplicar alinhamento de matriz** (padrão: ativado): distorce cada produto processado (debayering / visualização / radianância / refletância / índice) para a geometria de referência compartilhada da matriz. Desativado = exporta na geometria nativa do sensor.
* **Recortar para sobreposição comum** (padrão: ativado): recorta as exportações alinhadas à região compartilhada por todos os módulos, de modo que cada banda tenha a mesma área de cobertura. Desativado mantém a tela completa do sensor (preenchimento preto fora da fonte).
* **Reamostragem**:**Bilinear (suave, padrão)**,**Mais próximo (preserva valores exatos)**— sem mistura entre pixels, para análise radiométrica rigorosa — ou**Cúbica (mais nítida)**.***

## Salvando e carregando configurações

### Salvar modelo de projeto

Crie modelos reutilizáveis para fluxos de trabalho consistentes:

1. Configure todas as opções desejadas no painel “Configurações do projeto”
2. Role até a seção **“Salvar modelo de projeto”** na parte inferior
3. Digite um nome descritivo para o modelo (por exemplo, “Survey3N\_RGN\_Agricultura”)
4. Clique no ícone de salvar

**Benefícios:**

* Aplique configurações idênticas em vários projetos
* Compartilhe configurações com os membros da equipe
* Mantenha a consistência em pesquisas repetidas

### Carregar modelo em um novo projeto

Ao criar um novo projeto:

1. Selecione **“Novo projeto”** no menu principal
2. Escolha um modelo de projeto no seletor de modelos opcional
3. Todas as configurações do modelo são aplicadas automaticamente

### Diretório de Trabalho

A configuração **&quot;Diretório de Trabalho&quot;** especifica onde os novos projetos são criados por padrão:

* **Localização padrão**: `C:\Users\[Username]\Chloros Projects`
* **Alterar localização**: Clique no ícone de edição e selecione uma nova pasta
* **Compartilhado com o CLI**: O `chloros-cli` usa a mesma configuração padrão de pasta de projeto
* **Quando alterar**:
  * Unidade de rede para colaboração em equipe
  * Unidade diferente com mais espaço de armazenamento
  * Estrutura de pastas organizada por ano/cliente

***

## Configuração do PPK (Cinética Pós-Processada)

Se estiver usando gravadores DAQ MAPIR com GPS para geolocalização precisa:

### Pré-requisitos

* DAQ MAPIR com módulo GPS (GNSS)
* Arquivo de log .daq com entradas dos pinos de exposição
* Câmera conectada aos pinos de exposição do DAQ durante a sessão de captura

### Etapas de configuração

1. Coloque o arquivo de log .daq na pasta do seu projeto
2. Em Configurações do projeto, marque a caixa de seleção **“Aplicar correções PPK”**

3. Defina o**“Desvio de fuso horário do sensor de luz”**, se necessário (padrão: 0 para UTC)
4. Atribua as câmeras aos pinos de exposição:
   * **Câmera única**: atribuída automaticamente ao Pino 1
   * **Câmeras duplas**: atribua manualmente cada câmera ao pino correto**Atribuição dos pinos de exposição:*** **Pino de exposição 1**: Selecione o modelo da câmera no menu suspenso
* **Pino de exposição 2**: Selecione a segunda câmera ou “Não usar”
* A mesma câmera não pode ser atribuída a ambos os pinos

{% hint style="warning" %}
**Importante**: Os pinos de exposição devem ser atribuídos corretamente às respectivas câmeras. Uma atribuição incorreta resultará em dados de geolocalização errados.
{% endhint %}

***

## Cenários avançados

### Projetos com várias câmeras

Ao processar imagens de várias câmeras MAPIR em um único projeto:

1. O Chloros detecta automaticamente cada modelo de câmera (tanto Survey3 quanto LATTICE)
2. Cada câmera recebe os perfis de processamento adequados e cada uma tem sua própria árvore de pastas de saída
3. PPK: Atribua manualmente cada câmera Survey3 ao pino de exposição correto
4. Todas as câmeras utilizam o mesmo formato de exportação e os mesmos índices

**Exemplos**: Survey3W, RGN + Survey3N, OCN (conjunto de duas câmeras), ou um arranjo LATTICE que combina um mestre RGB com módulos de banda estreita

### Levantamentos em time-lapse ou com várias datas

Para levantamentos repetidos da mesma área ao longo do tempo:

1. Crie um modelo com suas configurações padrão
2. Use uma configuração consistente do alvo de calibração a cada sessão
3. Processe cada data como um projeto separado
4. Use configurações idênticas para obter resultados comparáveis
5. Exporte no mesmo formato para análise temporal

### Grandes conjuntos de dados

Para projetos com muitas imagens (mais de 500):

* Considere dividir em projetos menores por data ou área
* Use o processamento paralelo Chloros+ para resultados mais rápidos
* Considere o CLI ou o API para automação em lote
* Ajuste o intervalo mínimo de recalibração para reduzir o tempo de detecção do alvo

***

## Verificando suas configurações

Antes de iniciar o processamento, verifique estas configurações importantes:

* [ ] Modelo da câmera detectado corretamente no Navegador de Arquivos
* [ ] Correção de vinheta ativada
* [ ] Calibração de refletância ativada
* [ ] Para o Survey3: pelo menos uma imagem de alvo de calibração importada e verificada; para o LATTICE: um alvo e/ou um registro de luz descendente do `.daq` presente
* [ ] Índices multiespectrais desejados adicionados
* [ ] Formato de exportação adequado ao seu fluxo de trabalho
* [ ] Configurações de PPK definidas (se estiver usando arquivos .daq com eventos de exposição)

***

## Próximos passos

Depois de definir suas configurações:

1. **Marque as imagens-alvo de calibração** — Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
2. **Inicie o processamento** — Consulte [Iniciando o processamento](starting-the-processing.md)
3. **Acompanhe o andamento** — Consulte [Acompanhamento do processamento](monitoring-the-processing.md)

Para obter detalhes completos sobre todas as configurações disponíveis, consulte a documentação de referência [Configurações do projeto](../project-settings/project-settings.md).
