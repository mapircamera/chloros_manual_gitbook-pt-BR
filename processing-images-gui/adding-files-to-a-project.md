# Adicionando arquivos a um projeto

Depois de criar ou abrir um projeto no Chloros, o próximo passo é adicionar suas imagens multiespectrais para iniciar o processamento. A guia “Navegador de arquivos”<img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> facilita a importação de imagens e o gerenciamento do seu conjunto de dados.

## Acessando o Navegador de arquivos

1. Abra ou crie um projeto no Chloros
2. Clique no ícone **Navegador de Arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> na barra lateral esquerda
3. O painel do Navegador de Arquivos exibirá a lista de arquivos do seu projeto

{% hint style="info" %}
**Tipos de arquivos compatíveis**:

* **Survey3W / Survey3N**: pares RAW+JPG e imagens JPG (recomenda-se RAW+JPG)
* **LATTICE**: capturas `.tif` / `.tiff` — gravadas pelo controle de câmera do Chloros ou por um hub LATTICE
* **Dados do sensor de luz**: gravações `.daq` (DAQ-U/M/E) e registros de luz descendente `.csv` do DAQ-M — importados junto com as imagens para realizar a calibração de refletância
{% endhint %}

***

## Adicionando imagens ao seu projeto

Existem duas maneiras principais de adicionar imagens ao seu projeto:

### Método 1: Adicionar arquivos

Use esta opção para importar arquivos de imagem individuais ou uma pequena seleção de arquivos.

1. Clique no botão **&quot;Adicionar arquivos&quot;** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na parte superior do painel Navegador de arquivos
2. Navegue até a pasta que contém suas imagens
3. Selecione um ou mais arquivos de imagem (mantenha pressionada a tecla **Ctrl** para selecionar vários arquivos)
4. Clique em **&quot;Abrir&quot;** para importar os arquivos selecionados

### Método 2: Adicionar pasta

Use esta opção para importar todas as imagens de uma pasta de uma só vez. Você pode selecionar **várias pastas** em uma única caixa de diálogo.

1. Clique no botão **“Adicionar pasta”** <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line">, na parte superior do painel Navegador de arquivos
2. Navegue até a(s) pasta(s) que contém(em) as imagens da sua sessão de captura e selecione-a(s)
3. Clique em **“Selecionar pasta”** para importar todas as imagens compatíveis

{% hint style="info" %}
**Os arquivos que não conseguem ser carregados são sinalizados.** Se uma pasta contiver arquivos que o Chloros reconhece, mas não consegue carregar, um aviso informará isso — as imagens não desaparecem silenciosamente da grade.
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

**Selecione “Adicionar pasta” na raiz das capturas** (`output/` acima). Quando a pasta selecionada não contém imagens, mas possui subpastas, o Chloros as percorre automaticamente — as subpastas do nível e a raiz `.daq` são todas capturadas de uma só vez.**Como as capturas são importadas:*** Cada captura é importada como uma **única imagem**, agrupada por captura (não uma entrada por nível). Os outros níveis da mesma captura aparecem como modos de visualização dessa única imagem.
* **O processamento sempre começa a partir do quadro bruto.** Os outros níveis podem ser visualizados, mas apenas o `raw` é processado — reprocessar um produto já processado aplicaria as correções duas vezes, por isso o Chloros recusa. Uma exportação reimportada nunca pode ocupar o espaço de um quadro bruto de uma captura.
* Uma pasta de captura salva **sem** quadros brutos é importada e exibida normalmente, mas o processamento a ignora e registra isso no log. (O sinalizador da CLI `--input-level` pode forçar um ponto de entrada para esse caso — consulte [a Referência da CLI](../reference/cli-reference.md#what-a-captures-folder-looks-like).)
* A partir da versão 1.3.0, uma pasta de capturas também pode conter uma pasta `clamp_masks/` (máscaras por pixel para os níveis de radiação e refletância — consulte [Formatos de imagem de saída](../output-image-formats.md#clamp-masks)). As máscaras não são imagens da cena, e o Chloros as ignora na importação, independentemente de onde estejam.

**As sessões do hub LATTICE** são importadas da mesma maneira: clique em “Adicionar pasta” na pasta da sessão copiada do hub (ela contém `raw/` e `previews/`), juntamente com qualquer registro de fluxo descendente do DAQ-M `.csv`. Se a calibração da câmera ou do DAQ ainda não estiver armazenada em cache no seu computador, o Chloros a busca automaticamente pelo número de série durante a importação (requer conexão com a internet uma vez).***

## Entendendo a tabela do navegador de arquivos

Depois que as imagens são importadas, elas aparecem em uma tabela com as seguintes colunas:

### Nome do arquivo

* Nome original do arquivo da câmera
* Mantém a convenção de nomenclatura da câmera (por exemplo, IMG\_0001.RAW ou capture\_20260816\_101500\_SN213800234\_raw.tif)

### Carimbo de data/hora

* Data e hora em que a imagem foi capturada
* Extraído dos metadados EXIF da imagem
* Utilizado para correspondência de sensores de luz, sincronização PPK e programação de alvos de calibração

### Modelo da câmera

* Configuração da câmera e do filtro detectada automaticamente
* Exemplos do Survey3: Survey3W_RGN, Survey3N_OCN, Survey3W_RGB
* Exemplos do LATTICE: LATT-M3M-L41-F550, LATT-M3C-L87-FRGN
* Usado para aplicar os perfis de processamento corretos

### Coluna “Alvo” (caixa de seleção)

* Marque esta caixa para imagens que contenham alvos de calibração
* Quando pelo menos uma imagem estiver marcada, **apenas as imagens marcadas serão verificadas** em busca de alvos
* Consulte [Escolhendo imagens-alvo](choosing-target-images.md) para obter detalhes

### Visualização dos metadados da imagem

Clicar no botão de alternância no canto superior direito, acima da tabela, exibe os metadados da imagem selecionada na área da grade de imagens.

<figure><img src="../.gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

***

## Arquivos do sensor de luz em seu projeto

* Os arquivos `.daq` e `.csv` aparecem na lista do Navegador de Arquivos, mas não são imagens clicáveis — eles fornecem a irradiância descendente para a calibração de refletância.
* Cada arquivo `.daq`/`.csv` importado é listado em **Configurações do projeto → Sensor de luz DAQ**, onde você pode verificar a correção da tampa difusora aplicada a cada arquivo. Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).
* As gravações feitas na guia **Sensores de luz** são adicionadas automaticamente ao projeto aberto — não é necessária nenhuma importação manual.***

## Gerenciando arquivos em seu projeto

### Remoção de arquivos

Para remover imagens indesejadas do seu projeto:

1. Selecione uma ou mais imagens na tabela do Navegador de Arquivos
2. Clique no botão **“Remover selecionados”** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
3. Confirme a remoção (os arquivos não são excluídos do disco, apenas removidos do projeto)

### Classificação e filtragem

* **Classificar por coluna**: clique em qualquer cabeçalho de coluna para classificar as imagens
* **Classificação por data e hora**: útil para organizar sequências cronológicas de captura
* **Filtro por modelo de câmera**: agrupe as imagens por tipo de câmera caso esteja usando várias câmeras***

## Visualização da imagem

### Exibição da imagem completa

Clique em qualquer miniatura de imagem no Navegador de Arquivos para exibi-la na área principal de visualização:

1. A imagem aparece no painel de visualização central
2. Use os controles de zoom para examinar os detalhes da imagem
3. Navegue entre as imagens usando as setas do teclado

### Navegação rápida

* **Imagem anterior**: Clique na seta para a esquerda ou pressione a tecla ←
* **Imagem seguinte**: Clique na seta para a direita ou pressione a tecla →
* **Aumentar/diminuir o zoom**: Use a roda do mouse ou os botões de zoom
* **Deslocamento**: Clique e arraste sobre a imagem quando estiver ampliada***

## Tratamento de arquivos duplicados

O Chloros detecta e ignora automaticamente arquivos duplicados:

* Arquivos com nomes idênticos são ignorados
* Evita o processamento acidental em duplicata
* É exibida uma mensagem de aviso quando são detectadas duplicatas

{% hint style="warning" %}
**Importante**: Não renomeie nem modifique seus arquivos de imagem originais antes da importação. O Chloros depende dos nomes de arquivo e metadados originais para o processamento adequado.
{% endhint %}

***

## Conjuntos de dados de câmeras mistas

Se seu projeto contiver imagens de várias câmeras MAPIR:

1. O Chloros detecta automaticamente cada modelo de câmera — Survey3, LATTICE ou uma combinação
2. Cada tipo de câmera é processado com seu perfil de calibração apropriado
3. O Navegador de Arquivos exibe o modelo da câmera na coluna “Modelo da Câmera”
4. Cada câmera recebe sua própria árvore de pastas de saída ao ser processada

**Exemplos de cenários**: configuração de câmera dupla Survey3W RGN + Survey3N OCN, ou um conjunto LATTICE com uma câmera principal RGB e vários módulos de banda estreita***

## Melhores práticas

### Organize antes da importação

* Mantenha as imagens de alvo de calibração na mesma pasta que as imagens de levantamento
* Mantenha os arquivos do sensor de luz `.daq` / `.csv` de cada sessão de captura junto com as imagens dessa sessão
* Mantenha a estrutura de pastas original da sua câmera/cartão SD/hub
* Não misture conjuntos de dados de sessões diferentes em um único projeto

### Nomeação de arquivos

* Preserve os nomes originais dos arquivos da câmera (IMG\_0001.RAW, capture\_..., etc.)
* Não renomeie os arquivos antes da importação
* Os nomes originais contêm metadados importantes

### Imagens de alvo de calibração

* Sempre inclua 1 a 2 imagens de alvo de calibração por sessão (no Survey3; no LATTICE, uma gravação DAQ pode substituí-las — consulte [Escolhendo imagens de alvo](choosing-target-images.md))
* Capture as imagens-alvo antes e depois da sessão de captura
* Coloque as imagens-alvo nas mesmas condições de iluminação da área de captura
* Marque as imagens-alvo usando a caixa de seleção “Alvo”

***

## Problemas comuns e soluções

### Imagens não aparecem após a importação

**Possíveis causas:**

* Formato de arquivo não compatível (consulte a lista de tipos compatíveis no topo desta página)
* As imagens são de câmeras que não são da MAPIR (consulte [Câmeras compatíveis](../supported-cameras.md))
* Arquivo corrompido ou transferência incompleta do cartão SD

**Solução**: Verifique a compatibilidade do formato de arquivo e do modelo da câmera e analise o aviso de carregamento de arquivos para identificar exatamente quais arquivos apresentaram falha

### Modelo da câmera não detectado

**Possíveis causas:**

* Metadados EXIF modificados
* Imagens editadas em software externo
* Transferência incompleta de arquivos

**Solução**: Reimporte os arquivos originais, não modificados, da câmera ou do cartão SD

### Falta de carimbos de data/hora

**Possíveis causas:**

* Relógio da câmera não configurado corretamente
* Dados EXIF removidos por software externo

**Solução**: Verifique se as configurações de hora da câmera estavam corretas durante a captura

### Projeto reaberto relata arquivos ausentes

Se os arquivos de origem foram movidos ou excluídos desde a última vez que o projeto foi aberto, o Chloros informa **quais**arquivos estão faltando, em vez de abrir uma grade em branco. Se a própria pasta do projeto tiver sido movida, o Chloros redireciona seus caminhos de origem para a pasta em que `project.json` está agora e mantém um backup (`project.json.before-relink-<date>-<time>`). Quando isso não os localiza, a janela**Arquivos de origem não encontrados**oferece a opção**Procurar**para o novo local e, em seguida,**Novo projeto**ou**Excluir projeto**. Na CLI, use `chloros-cli project open PATH --relink FOLDER`; no SDK, `open_project(path, relink=...)`.***

## Próximos passos

Depois que seus arquivos forem importados:

1. **Revise a lista de arquivos** — Certifique-se de que todas as imagens foram carregadas corretamente
2. **Verifique os modelos de câmera** — Confirme se a detecção da câmera está correta
3. **Marque as imagens-alvo** — Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
4. **Ajuste as configurações** — Configure as opções de processamento em [Configurações do projeto](adjusting-project-settings.md)
5. **Inicie o processamento** — Consulte [Iniciando o processamento](starting-the-processing.md)

Para obter informações detalhadas sobre a configuração do projeto, consulte [Ajustando as configurações do projeto](adjusting-project-settings.md).
