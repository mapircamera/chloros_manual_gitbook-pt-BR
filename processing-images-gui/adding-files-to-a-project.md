# Adicionando arquivos a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar suas imagens multiespectrais para iniciar o processamento. A guia Navegador de arquivos<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e o gerenciamento do seu conjunto de dados.

## Acessando o Navegador de Arquivos

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de Arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel Navegador de arquivos exibirá a lista de arquivos do seu projeto

{% dica style=&quot;info&quot; %}
**Tipos de arquivos compatíveis**: O Chloros é compatível com arquivos de imagem RAW+JPG e JPG das câmeras MAPIR Survey3W e Survey3N. Recomenda-se apenas RAW+JPG.
{% endhint %}

***

## Adicionando imagens ao seu projeto

Existem duas maneiras principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar arquivos

Use esta opção para importar arquivos de imagem individuais ou uma pequena seleção de arquivos.

1. Clique no botão **“Adicionar arquivos”** <img src="../.gitbook/assets/image.png" alt="" data-size="line"> na parte superior do painel Navegador de arquivos
2. Navegue até a pasta que contém suas imagens
3. Selecione um ou mais arquivos de imagem (mantenha pressionada a tecla **Ctrl** para selecionar vários arquivos)
4. Clique em **“Abrir”** para importar os arquivos selecionados

### Método 2: Adicionar pasta

Use esta opção para importar todas as imagens de uma pasta de uma só vez.

1. Clique no botão **“Adicionar pasta”** <img src="../.gitbook/assets/image (1).png" alt="" data-size="line"> na parte superior do painel Navegador de arquivos
2. Navegue até a pasta que contém as imagens da sua sessão de captura e selecione-a
3. Clique em **“Selecionar pasta”** para importar todas as imagens compatíveis dessa pasta***

## Entendendo a tabela do Navegador de arquivos

Depois que as imagens são importadas, elas aparecem em uma tabela com as seguintes colunas:

### Nome do arquivo

* Nome do arquivo original da câmera
* Mantém a convenção de nomenclatura da câmera (por exemplo, IMG\_0001.RAW)

### Carimbo de data/hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF da imagem
* Usado para sincronização PPK e detecção de alvo de calibração

### Modelo da câmera

* Configuração da câmera e do filtro detectada automaticamente
* Exemplos: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Usado para aplicar perfis de processamento corretos

### Coluna Alvo (caixa de seleção)

* Marque esta caixa para imagens que contêm alvos de calibração
* Acelera significativamente a detecção de alvos durante o processamento
* Consulte [Escolhendo imagens-alvo](choosing-target-images.md) para obter detalhes

***

## Gerenciando arquivos em seu projeto

### Removendo arquivos

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de arquivos
2. Clique no botão **“Remover selecionados”** <img src="../.gitbook/assets/image (2).png" alt="" data-size="line"> .
3. Confirme a remoção (os arquivos não são excluídos do disco, apenas removidos do projeto)

### Classificação e filtragem

* **Classificar por coluna**: clique em qualquer cabeçalho de coluna para classificar as imagens
* **Classificação por data e hora**: útil para organizar sequências de captura cronológicas
* **Filtro de modelo de câmera**: agrupe imagens por tipo de câmera se estiver usando várias câmeras***

## Visualização de imagem

### Exibindo a imagem completa

Clique em qualquer miniatura de imagem no Navegador de arquivos para exibi-la na área de visualização principal:

1. A imagem aparece no painel de visualização central
2. Use os controles de zoom para inspecionar os detalhes da imagem
3. Navegue entre as imagens usando as setas do teclado

### Navegação rápida

* **Imagem anterior**: clique na seta para a esquerda ou pressione a tecla ←
* **Próxima imagem**: clique na seta para a direita ou pressione a tecla →
* **Aumentar/diminuir zoom**: use a roda do mouse ou os botões de zoom
* **Panorâmica**: clique e arraste na imagem quando estiver com o zoom aumentado***

## Tratamento de arquivos duplicados

O Chloros detecta e ignora automaticamente arquivos duplicados:

* Arquivos com nomes idênticos são ignorados
* Evita o processamento duplo acidental
* Mensagem de aviso exibida quando duplicatas são detectadas

{% hint style=&quot;warning&quot; %}
**Importante**: Não renomeie nem modifique seus arquivos de imagem originais antes de importá-los. O Chloros depende dos nomes de arquivo e metadados originais para o processamento adequado.
{% endhint %}

***

## Conjuntos de dados de câmeras mistas

Se o seu projeto contiver imagens de várias câmeras MAPIR:

1. O Chloros detecta automaticamente cada modelo de câmera
2. Cada tipo de câmera é processado com seu perfil de calibração apropriado
3. O Navegador de arquivos exibe o modelo da câmera na coluna Modelo da câmera
4. O processamento aplica as configurações corretas para cada tipo de câmera

**Exemplo de cenário**: Survey3W RGN + Survey3N OCN configuração de câmera dupla***

## Melhores práticas

### Organize antes de importar

* Mantenha as imagens de alvo de calibração na mesma pasta que as imagens de levantamento
* Mantenha a estrutura original da pasta da sua câmera/cartão SD
* Não misture conjuntos de dados de sessões diferentes em um único projeto

### Nomeação de arquivos

* Preserve os nomes de arquivo originais da câmera (IMG\_0001.RAW, etc.)
* Não renomeie os arquivos antes da importação
* Os nomes originais contêm metadados importantes

### Imagens de alvo de calibração

* Sempre inclua 1-2 imagens de alvo de calibração por sessão
* Capture os alvos antes e depois da sessão de captura
* Coloque os alvos nas mesmas condições de iluminação da área de captura
* Marque as imagens de alvo usando a caixa de seleção Alvo para acelerar o processamento

***

## Problemas comuns e soluções

### Imagens não aparecem após a importação

**Possíveis causas:**

* Formato de arquivo não compatível (apenas RAW+JPG e JPG de câmeras MAPIR)
* As imagens são de câmeras que não são MAPIR (consulte [Câmeras compatíveis](../supported-cameras.md))
* Arquivo corrompido ou transferência incompleta do cartão SD

**Solução:** Verifique a compatibilidade do formato do arquivo e do modelo da câmera

### Modelo da câmera não detectado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência de arquivo incompleta

**Solução:** Reimporte os arquivos originais, não modificados, da câmera/cartão SD

### Carimbos de data/hora ausentes

**Possíveis causas:**

* Relógio da câmera não configurado corretamente
* Dados EXIF removidos por software externo

**Solução**: Verifique se as configurações de hora da câmera estavam corretas durante a captura***

## Próximos passos

Depois que seus arquivos forem importados:

1. **Revise a lista de arquivos** - Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos de câmera** - Verifique se a detecção da câmera está correta
3. **Marque as imagens de destino** - Consulte [Escolhendo imagens de destino](choosing-target-images.md)
4. **Ajuste as configurações** - Configure as opções de processamento em [Configurações do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).
