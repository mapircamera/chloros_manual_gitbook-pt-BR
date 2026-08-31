# Adicionando arquivos a um projeto

Depois de criar ou abrir um projeto no Chloros, a próxima etapa é adicionar suas imagens multiespectrais para iniciar o processamento. A guia “Navegador de arquivos” <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e o gerenciamento do seu conjunto de dados.

## Acessando o Navegador de arquivos

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de Arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel do Navegador de Arquivos exibirá a lista de arquivos do seu projeto

{% hint style="info" %}
**Tipos de arquivos compatíveis**:

* **Survey3W / Survey3N**: pares RAW+JPG e imagens JPG (recomenda-se RAW+JPG)
* **LATTICE**: capturas `.tif` / `.tiff` — gravadas pelo controle de câmera Chloros ou por um hub LATTICE
* **Dados do sensor de luz**: gravações `.daq` (DAQ-U/M/E) e registros de radiação descendente do DAQ-M `.csv` — importados juntamente com as imagens para realizar a calibração de refletância
{% endhint %}

***

## Adicionando imagens ao seu projeto

Existem duas maneiras principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar arquivos

Use esta opção para importar arquivos de imagem individuais ou uma pequena seleção de arquivos.

1. Clique no botão **“Adicionar arquivos”** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na parte superior do painel Navegador de arquivos
2. Navegue até a pasta que contém suas imagens
3. Selecione um ou mais arquivos de imagem (mantenha pressionada a tecla **Ctrl** para selecionar vários arquivos)
4. Clique em **“Abrir”** para importar os arquivos selecionados

### Método 2: Adicionar pasta

Use esta opção para importar todas as imagens de uma pasta de uma só vez. Você pode selecionar **várias pastas** em uma única caixa de diálogo.

1. Clique no botão **“Adicionar pasta”** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> na parte superior do painel Navegador de arquivos
2. Navegue até e selecione a(s) pasta(s) que contém(em) as imagens da sua sessão de captura
3. Clique em **“Selecionar pasta”** para importar todas as imagens compatíveis

{% hint style="info" %}
**Os arquivos que não conseguem ser carregados são relatados.** Se uma pasta contiver arquivos que o Chloros reconhece, mas não consegue carregar, um aviso informará isso — as imagens não desaparecem silenciosamente da grade.
{% endhint %}

***

## Importação de pastas de captura do LATTICE

As capturas do LATTICE são salvas com **uma subpasta por nível de exportação** — por exemplo, `raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` — com o arquivo de fluxo descendente correspondente `.daq` na raiz:

```
output/
├── raw/           capture_<timestamp>_SN<serial>_raw.tif
├── debayered/     capture_<timestamp>_SN<serial>_debayered.tif
├── preview/       capture_<timestamp>_SN<serial>_display.tif
└── *.daq          the downwelling reading matched to the capture
```

**Selecione “Adicionar pasta” na raiz das capturas** (`output/` acima). Quando a pasta selecionada não contém imagens, mas possui subpastas, o Chloros percorre-as automaticamente — as subpastas do nível e a raiz `.daq` são todas capturadas de uma só vez.**Como as capturas são importadas:*** Cada captura é importada como uma **única imagem**, agrupada por captura (não uma entrada por nível). Os outros níveis da mesma captura aparecem como modos de visualização dessa única imagem.
* **O processamento sempre começa a partir do quadro bruto.** Os outros níveis podem ser visualizados, mas apenas o `raw` é encaminhado pelo pipeline — reprocessar um produto já processado aplicaria as correções duas vezes, por isso o Chloros é rejeitado. Uma exportação reimportada nunca pode ocupar o espaço de quadro bruto de uma captura.
* Uma pasta de captura salva **sem** importações de quadros brutos é exibida normalmente, mas o processamento a ignora e registra isso no log. (O sinalizador CLI `--input-level` pode forçar um ponto de entrada para esse caso — consulte [a Referência do CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)**As sessões do hub LATTICE** são importadas da mesma maneira: selecione “Adicionar Pasta” na pasta da sessão copiada do hub (ela contém `raw/` e `previews/`), juntamente com qualquer registro de downwelling do DAQ-M `.csv`. Se a calibração da câmera ou do DAQ ainda não estiver armazenada em cache na sua máquina, o Chloros a busca automaticamente pelo número de série durante a importação (requer conexão com a internet uma vez).***

## Entendendo a tabela do navegador de arquivos

Depois que as imagens são importadas, elas aparecem em uma tabela com as seguintes colunas:

### Nome do arquivo

* Nome do arquivo original da câmera
* Mantém a convenção de nomenclatura da câmera (por exemplo, IMG\_0001.RAW ou capture\_20260816\_101500\_SN213800234\_raw.tif)

### Carimbo de data/hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF da imagem
* Utilizado para correspondência de sensores de luz, sincronização PPK e programação de alvos de calibração

### Modelo da câmera

* Configuração da câmera e do filtro detectada automaticamente
* Exemplos Survey3: Survey3W\_RGN, Survey3N\_OCN, Survey3W\_RGB
* Exemplos LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Usado para aplicar os perfis de processamento corretos

### Coluna de alvos (caixa de seleção)

* Marque esta caixa para imagens que contenham alvos de calibração
* Quando pelo menos uma imagem estiver marcada, **apenas as imagens marcadas serão verificadas** em busca de alvos
* Consulte [Escolhendo imagens-alvo](choosing-target-images.md) para obter detalhes

### Visualização dos metadados da imagem

Clicar no botão de alternância no canto superior direito, acima da tabela, exibe os metadados da imagem selecionada na área da grade de imagens.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Arquivos do sensor de luz em seu projeto

* Os arquivos `.daq` e `.csv` aparecem na lista do Navegador de Arquivos, mas não são imagens clicáveis — eles fornecem a irradiância descendente para a calibração de refletância.
* Cada arquivo `.daq`/`.csv` importado é listado em **Configurações do Projeto → Sensor de Luz DAQ**, onde você pode verificar a correção da tampa difusora aplicada a cada arquivo. Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).
* Os registros feitos na guia **Sensores de luz** são adicionados automaticamente ao projeto aberto — não é necessária nenhuma importação manual.***

## Gerenciando arquivos em seu projeto

### Remoção de arquivos

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de Arquivos
2. Clique no botão **“Remover selecionados”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirme a remoção (os arquivos não são excluídos do disco, apenas removidos do projeto)

### Classificação e filtragem

* **Classificar por coluna**: clique em qualquer cabeçalho de coluna para classificar as imagens
* **Classificação por data e hora**: útil para organizar sequências cronológicas de captura
* **Filtro por modelo de câmera**: agrupe imagens por tipo de câmera caso esteja usando várias câmeras***

## Visualização da imagem

### Exibição da imagem completa

Clique em qualquer miniatura de imagem no Navegador de Arquivos para exibi-la na área principal de visualização:

1. A imagem aparece no painel de visualização central
2. Use os controles de zoom para inspecionar os detalhes da imagem
3. Navegue entre as imagens usando as setas do teclado

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou pressione a tecla ←
* **Imagem seguinte**: Clique na seta para a direita ou pressione a tecla →
* **Aumentar/diminuir o zoom**: Use a roda do mouse ou os botões de zoom
* **Deslocamento**: Clique e arraste sobre a imagem quando estiver ampliada***

## Tratamento de arquivos duplicados

O Chloros detecta e ignora automaticamente arquivos duplicados:

* Arquivos com nomes idênticos são ignorados
* Evita o processamento duplo acidental
* É exibida uma mensagem de aviso quando são detectadas duplicatas

{% hint style="warning" %}
**Importante**: Não renomeie nem modifique seus arquivos de imagem originais antes da importação. O Chloros depende dos nomes de arquivo e metadados originais para o processamento adequado.
{% endhint %}

***

## Conjuntos de dados de câmeras mistas

Se o seu projeto contiver imagens de várias câmeras MAPIR:

1. O Chloros detecta automaticamente cada modelo de câmera — Survey3, LATTICE ou uma combinação
2. Cada tipo de câmera é processado com seu perfil de calibração apropriado
3. O Navegador de Arquivos exibe o modelo da câmera na coluna “Modelo da Câmera”
4. Cada câmera recebe sua própria árvore de pastas de saída ao ser processada

**Exemplos de cenários**: Survey3W RGN + Survey3N OCN — configuração com duas câmeras, ou uma matriz LATTICE com um RGB mestre e vários módulos de banda estreita***

## Melhores práticas

### Organize antes da importação

* Mantenha as imagens de alvo de calibração na mesma pasta que as imagens de levantamento
* Mantenha os arquivos dos sensores de luz `.daq` / `.csv` de cada sessão de captura junto com as imagens dessa sessão
* Mantenha a estrutura de pastas original da sua câmera/cartão SD/hub
* Não misture conjuntos de dados de sessões diferentes em um único projeto

### Nomeação de arquivos

* Preserve os nomes originais dos arquivos da câmera (IMG\_0001.RAW, capture\_..., etc.)
* Não renomeie os arquivos antes da importação
* Os nomes originais contêm metadados importantes

### Imagens-alvo de calibração

* Sempre inclua 1 a 2 imagens-alvo de calibração por sessão (Survey3; para o LATTICE, uma gravação DAQ pode substituí-las — consulte [Escolhendo imagens-alvo](choosing-target-images.md))
* Capture os alvos antes e depois da sessão de captura
* Coloque os alvos nas mesmas condições de iluminação da área de captura
* Marque as imagens-alvo usando a caixa de seleção “Alvo”

***

## Problemas comuns e soluções

### Imagens que não aparecem após a importação

**Possíveis causas:**

* Formato de arquivo não compatível (consulte a lista de tipos compatíveis no topo desta página)
* As imagens são de câmeras que não são do modelo MAPIR (consulte [Câmeras compatíveis](../supported-cameras.md))
* Arquivo corrompido ou transferência incompleta do cartão SD

**Solução**: Verifique a compatibilidade do formato do arquivo e do modelo da câmera e confira o aviso de carregamento de arquivos para identificar exatamente quais arquivos apresentaram falha

### Modelo da câmera não detectado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência incompleta de arquivos

**Solução**: Reimporte os arquivos originais, não modificados, da câmera ou do cartão SD

### Falta de registros de data e hora

**Possíveis causas:**

* Relógio da câmera não configurado corretamente
* Dados EXIF removidos por software externo

**Solução**: Verifique se as configurações de hora da câmera estavam corretas durante a captura

### Relatórios de projeto reabertos indicam arquivos ausentes

Se os arquivos de origem foram movidos ou excluídos desde a última vez que o projeto foi aberto, o código de erro Chloros informa **quais** arquivos estão faltando, em vez de abrir uma grade em branco. Restaure os arquivos em seus caminhos originais ou remova as entradas ausentes e reimporte-os.***

## Próximos passos

Depois que seus arquivos forem importados:

1. **Revise a lista de arquivos** – Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos das câmeras** – Confirme se a detecção das câmeras está correta
3. **Marque as imagens de destino** — Consulte [Escolhendo imagens de destino](choosing-target-images.md)
4. **Ajuste as configurações** — Configure as opções de processamento em [Configurações do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** – Consulte [Iniciando o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).
