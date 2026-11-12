# Conclusão do processamento

Assim que o Chloros concluir o processamento, é hora de analisar seus resultados, verificar a qualidade da saída e preparar suas imagens processadas para uso em seu fluxo de trabalho. Esta página orienta você nas etapas finais e nas próximas ações.

## Indicação de conclusão do processamento

Quando o processamento for concluído com sucesso, você verá vários indicadores:

* ✅ **Barra de progresso**: Atinge 100% de conclusão
* ✅ **Log de depuração**: Mostra as linhas finais `[RUN-SUMMARY]` com contagens (imagens, grupos de câmeras, alvos, imagens calibradas, arquivos gravados)
* ✅ **Botão Iniciar**: Torna-se ativo novamente (pronto para a próxima execução do processamento)
* ✅ **Arquivos de saída**: Todas as imagens processadas são salvas na árvore de saída do projeto (abaixo)

{% hint style="warning" %}
**Uma execução que não grava imagens é considerada uma falha.** Se você solicitou produtos de imagem e a execução não gravou nenhum, o Chloros relata isso como uma falha — o `[RUN-SUMMARY]` no nome do log sugere a causa provável (nada importado, nenhum alvo detectado ou todos os produtos solicitados ignorados por serem inaplicáveis). O equivalente na CLI retorna um valor diferente de zero. Uma execução deliberada apenas com metadados (todos os produtos de exportação desativados, sem índices) ainda é considerada bem-sucedida. Consulte [a Referência da CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localizando suas imagens processadas

### Abrindo a pasta de saída

1. Clique no ícone do **Menu Principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line">
(canto superior esquerdo)
2. Selecione **“Abrir pasta do projeto”**

3. Seu explorador de arquivos abre no diretório do projeto
4. Localize seu projeto pelo nome

### A árvore de saída

Os produtos são salvos **na pasta do projeto, agrupados por câmera e, em seguida, por formato de arquivo**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one folder per selected index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

* **Pasta da câmera**: `LATT-<sensor>-<lens>-F<filter>` para LATTICE (correspondendo ao EXIF da captura `Model`), `<model>_<filter>` para Survey3 (por exemplo, `Survey3N_RGN`). Duas câmeras que compartilham um sensor e um filtro, mas possuem lentes diferentes, mantêm árvores separadas — a vinheta, o campo de visão e a distorção diferem.
* **Pasta de formato**: segue sua configuração de formato de exportação — `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` para TIFF (32 bits, Porcentagem). A radianância é sempre float32 e sempre fica na pasta `tiff32`.
* **Pastas de produtos**:
  * `Reflectance_Calibrated_Images/` — refletância calibrada
  * `Debayered_Images/` — sem bayering linear (LATTICE): desvio de escuro removido e sem bayering, sem campo plano nem desmistura, na orientação nativa do sensor
  * `Preview_Images/` — visualização na tela (LATTICE)
  * `Radiance_Images/` — radiação espectral em float32, W/m²/sr/nm (multiespectral LATTICE)
  * `Vignette_Corrected_Images/` **ou** `Sensor_Response_Images/` — o valor de fallback não calibrado para quadros sem referência de refletância; existe exatamente um dos dois por execução, escolhido pela configuração de correção de vinheta
  * `<INDEX>_Index_Images/` — uma pasta por índice selecionado (por exemplo, `NDVI_Index_Images`)
* **`clamp_masks/`** (novidade na versão 1.3.0, apenas para câmeras multiespectrais M3C da LATTICE): aparece em uma pasta de formato, ao lado das pastas de produtos, quando a separação de bandas corta valores negativos para zero em um produto de radiação ou refletância. Ela contém uma máscara por pixel para cada produto afetado, por exemplo, `tiff16/clamp_masks/Reflectance_Calibrated_Images/<name>_clampmask.tif`. As máscaras nunca estão dentro de uma pasta de produto; portanto, uma pasta de produto pode ser entregue ao software de fotogrametria tal como está. Consulte [Máscaras de limitação](../output-image-formats.md#clamp-masks).

{% hint style="info" %}
**Todo produto exportado mantém o nome do arquivo de ORIGEM.**Uma exportação de radianância de `capture_..._raw.tif` ainda se chama `capture_..._raw.tif` — ela apenas fica na pasta `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do arquivo**; portanto, procurar por `*radiance*.tif` não resulta em nada; em vez disso, faça a correspondência com o diretório.
{% endhint %}

<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-M3C multispectral camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images, clamp_masks) and tiff32 (Radiance_Images, clamp_masks) subfolders visible -->


### Quantos arquivos devem existir?

Não conte com base em fórmulas — o número de saídas depende de quais produtos foram habilitados e quais se aplicam a cada câmera (por exemplo, câmeras RGB não recebem radiação/refletância). A contagem oficial está no log: a linha final `[RUN-SUMMARY]` informa exatamente quantos arquivos foram gravados, e as linhas de dica explicam tudo o que foi ignorado.

***

## Revisão das imagens processadas

### Visualização rápida no Explorador de Arquivos

**Visualização integrada do Windows:**

1. Navegue até uma pasta de produtos (por exemplo, `tiff16/Reflectance_Calibrated_Images/`)
2. Selecione um arquivo de imagem
3. A pré-visualização aparece no painel de pré-visualização do Explorador do Windows

4. Use as setas do teclado para percorrer as imagens

### Pré-visualização em visualizadores de imagem externos

**Visualizadores recomendados:*** **QGIS** – Software GIS gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** – Visualizador de imagens rápido e leve (compatível com TIFF)
* **Adobe Photoshop** – Edição profissional (compatível com TIFF)
* **GIMP** – Alternativa gratuita ao Photoshop
* **Fotos do Windows** – Visualização básica (pode não ser compatível com TIFF de 16 bits)

### Visualização no visualizador de imagens do Chloros


Use o visualizador de imagens integrado do Chloros para visualização avançada:

1. Clique na miniatura de uma imagem no Navegador de Arquivos
2. A imagem é aberta na área principal de visualização
3. Clique na aba **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line">
na barra lateral esquerda
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de Imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Leitura dos valores de refletância dos pixels (GIS / Pix4D / Scripts)

A refletância é armazenada como um valor inteiro DN, e **o valor de DN que corresponde a ρ = 1,0 depende da câmera de origem**:

| Origem          | ρ = 1,0 é | Como identificar                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margem de segurança até ρ 2,0) | A tag XMP `Chloros:PixelScale=32768` está marcada no arquivo |
| Survey3          | **65535** (limitado a ρ 1,0)     | Não há tag `Chloros:PixelScale` — essa ausência é o indicador (os produtos Survey3 contêm outras tags de processamento `Chloros:*`) |

**Leia a tag `Chloros:PixelScale` e divida pelo valor dela**, em vez de assumir um valor genérico de 65535 — dividir a refletância do LATTICE por 65535 reduz silenciosamente pela metade todos os valores. Um arquivo de refletância anterior à versão 1.3.0, gravado em 8 bits, foi cortado, não redimensionado, e não possui tag de escala: reexporte-o em 16 bits ou 32 bits em vez de dividi-lo. Consulte [Formatos de imagem de saída](../output-image-formats.md) para obter mais detalhes.

***

## Metadados transferidos para as exportações

Cada produto mantém o **bloco GPS**da captura original e seu**sub-IFD EXIF**; portanto, uma
exportação transporta `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber`, além da georreferência.

{% hint style="warning" %}
**Se um ortomosaico for gerado em uma escala absurda, verifique primeiro o `FocalLength`.**
O Pix4D calcula a distância entre amostras no solo a partir da distância focal mais a altitude. Sem a tag, ele
recorre a uma escala totalmente errada — em um voo medido com 49 capturas, um pomar de laranjas de 411 m × 160 m
foi reconstruído como 47,8 km × 13 km, produzindo uma ortomosaico de 455 megapixels composta principalmente
por espaço vazio. A lentidão na divisão em blocos e um arquivo inesperadamente grande são sintomas disso, não
problemas separados.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nem *todas* as tags são copiadas. As tags estruturais do IFD0 são deliberadamente deixadas de fora (copiá-las
corrompe a saída do LATTICE), e `ExifImageWidth` / `ExifImageHeight` são excluídas
porque descrevem a captura original — uma exportação redimensionada, caso contrário,
reivindicaria dimensões que sua própria imagem rasterizada contradiz.

O Chloros também grava suas próprias tags XMP `Chloros:*`, que descrevem o que foi feito em cada produto
(por exemplo, `Chloros:ProcessingLevel`, `Chloros:CalibrationApplied`,
`Chloros:VignetteCorrected`). A partir da versão 1.3.0, os produtos do Survey3 as incluem em todos os modos de processamento; os produtos de índice descrevem o próprio índice (`Chloros:ProcessingLevel=index`, a
fórmula e se foi calculado a partir da refletância ou da radianância), em vez de repetir as
tags do arquivo bruto de origem; e os produtos em pastas com nomes com mais de 260 caracteres também mantêm seus metadados. Consulte [Formatos de imagem de saída](../output-image-formats.md#index-products).

***

## Analisando o log de depuração

### Verifique se há avisos ou erros

1. Abra a guia **Log de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

2. Percorra as mensagens
3. Procure por avisos em amarelo ou erros em vermelho
4. Leia as linhas `[RUN-SUMMARY]` e quaisquer dicas
5. Entre em contato com o suporte da MAPIR para obter assistência

### Salvando o log

Para manter um registro do processamento ou enviar ao suporte da MAPIR:

1. Clique no botão **“Copiar”**ou**“Baixar”**

2. Salve como arquivo de texto na pasta do projeto
3. Inclua na documentação do projeto
4. Envie ao suporte da MAPIR caso encontre problemas

Para enviar um problema diretamente à MAPIR, use o ícone de bug (**Relatar um bug**) na parte superior da guia “Log de Depuração” — consulte [Como relatar um bug a partir da guia de log](monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

***

## Problemas comuns de saída e soluções

### Problema: Arquivos de saída ausentes

**Possíveis causas:**

* O produto não se aplica àquela câmera (por exemplo, radianância/refletância para câmeras RGB — o log indica isso)
* Faltou uma referência obrigatória (por exemplo, refletância sem alvo e sem `.daq` de radiação descendente)
* A caixa de seleção de exportação do produto estava desativada nas Configurações do Projeto
* O espaço em disco se esgotou durante a exportação

**Soluções:**

1. Verifique as dicas `[RUN-SUMMARY]` e as linhas `[EXPORT-CHECK]` no Log de Depuração — elas explicam as omissões por câmera
2. Verifique as caixas de seleção de exportação do produto em [Configurações do projeto](adjusting-project-settings.md)
3. Verifique se havia espaço em disco suficiente
4. Reprocesse após corrigir a causa

### Problema: Bordas escuras ou claras (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmera/lente não consta no banco de dados de perfis dChloros

* Vinheta extrema, além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta está ativada nas Configurações do projeto
2. Verifique se o modelo da câmera foi detectado corretamente
3. Entre em contato com o suporte da MAPIR se a vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Nenhum alvo de calibração detectado
* Modelo incorreto de alvo de calibração selecionado
* Calibração de refletância desativada
* Imagens de alvo de baixa qualidade

**Soluções:**

1. Verifique se a calibração de refletância estava ativada
2. Verifique as mensagens “Alvo encontrado” no Log de Depuração
3. Analise a qualidade das imagens dos alvos
4. Reprocesse com os alvos corretos marcados

### Problema: Valores de NDVI parecem incorretos

**Intervalos esperados para o NDVI:*** **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação esparsa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9**Se os valores estiverem fora desses intervalos:**

1. Verifique se a calibração de refletância foi aplicada
2. Verifique se o registro do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detectados
4. Certifique-se de que o modelo correto da câmera foi detectado
5. Analise o momento e as condições de captura das imagens-alvo
6. Se você mesmo estiver calculando os índices a partir dos arquivos de refletância, confirme se dividiu pelo valor `Chloros:PixelScale` do arquivo (veja acima)

***

## Como usar suas imagens processadas

### Para fotogrametria / criação de ortomosaicos

**Fluxo de trabalho recomendado:**

1.**Importe imagens de refletância calibradas** para um software de fotogrametria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Mantenha os metadados EXIF**: certifique-se de que os dados de GPS sejam preservados para georreferenciamento
3. **Fluxos de trabalho calibrados**: use imagens de refletância para precisão científica — a refletância LATTICE contém as tags de calibração XMP que o Pix4D lê
4. **Processe mosaicos de índice**: crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exportar GeoTIFF georreferenciado**: Para uso em aplicativos SIG

### Para análise SIG

**Fluxo de trabalho recomendado:**

1.**Carregar no QGIS, ArcGIS ou similar**

2.**Use imagens de refletância em formato TIFF de 16 bits** para análise multibanda (divida pelo valor `Chloros:PixelScale` do arquivo)
3. **Use imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas para uso
4. **Calculadora raster**: combine bandas para análises personalizadas
5. **Exportar**: crie mapas de classificação, detecção de alterações e mapas de saúde da vegetação

### Para análise direta / relatórios

**Fluxo de trabalho recomendado:**

1.**Utilize imagens de índice com cores LUT** para relatórios visuais
2. **Extrair estatísticas**: NDVI médio por campo/parcela
3. **Séries temporais**: Comparar índices entre várias sessões
4. **Gerar relatórios**: Incluir mapas, estatísticas e visualizações***

## Arquivamento e backup

### Estratégia de backup recomendada

**O que salvar:*** ✅ **Imagens RAW/JPG originais ou capturas brutas do LATTICE** — Arquive em uma unidade separada ou na nuvem; os arquivos brutos são a fonte do pipeline e tudo o mais pode ser regenerado a partir deles
* ✅ **Arquivos do sensor de luz `.daq` / `.csv`** — Necessários para recalcular a refletância posteriormente
* ✅ **Resultados processados** — Mantenha as imagens calibradas e os índices
* ✅ **Pasta do projeto** (`project.json` e arquivos associados) — Contém todas as configurações para reprocessamento, se necessário
* ✅ **Log de depuração** — Documenta os detalhes do processamento
* ✅ **Imagens de alvo de calibração** — Para verificação e reprocessamento**Recomendações de armazenamento:*** **Backup imediato**: Disco rígido externo
* **Arquivamento de longo prazo**: Armazenamento em nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: Mantenha 2 a 3 cópias em locais diferentes***

## Próximas execuções de processamento

### Reutilização das configurações do projeto

Se for processar conjuntos de dados semelhantes no futuro:

1. **Salve o modelo do projeto** (caso ainda não tenha feito isso)
2. **Crie um novo projeto** usando o modelo salvo
3. **Importe novas imagens**

4.**Processe**com configurações idênticas para garantir a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:**Opção 1: GUI – Vários projetos**

* Crie um projeto separado para cada sessão
* Use configurações consistentes do modelo
* Processe um de cada vez

**Opção 2: Chloros CLI (somente Chloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte a [Documentação da CLI](../CLI.md) e a [Referência da CLI](../reference/cli-reference.md)

**Opção 3: SDK do Python (somente Chloros+)**

* Controle programático
* Integração com pipelines de análise
* Consulte a [Documentação da API](../api-python-sdk.md) e a [Referência do SDK](../reference/sdk-reference.md)

***

## Solução de problemas no pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Mantenha as imagens originais (nunca as exclua)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel “Configurações do projeto”
4. Processe novamente — os resultados são salvos nas mesmas pastas do produto, portanto, os arquivos com o mesmo nome da execução anterior serão substituídos

### Processamento de um subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que precisam ser reprocessadas
3. Use o mesmo modelo de configurações
4. Processe um conjunto de dados menor

### Obtenha ajuda

Se você encontrar problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o log de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Perguntas frequentes**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)

***

## Resumo: Fluxo de trabalho completo

Você concluiu agora todo o fluxo de trabalho de processamento do Chloros:

1. ✅ **Projeto criado** - Consulte [Projetos](../projects.md)
2. ✅ **Adicionou arquivos** – Consulte [Adicionando arquivos](adding-files-to-a-project.md)
3. ✅ **Ajustou as configurações** — Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
4. ✅ **Marcou os alvos** — Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
5. ✅ **Iniciado o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
6. ✅ **Monitorado o andamento** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página**Suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Recursos avançados

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) — Visualização e análise interativas
* [**Área de testes de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) — Testes personalizados de índices
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — Referência completa de índices

### Automação e integração

* [**Documentação da CLI**](../CLI.md) — Processamento em lote por linha de comando
* [**Python SDK**](../api-python-sdk.md) — Automação programática
* [**Recursos do Chloros+**](../#chloros) — Recursos avançados de processamento

### Suporte e Aprendizado

* [**Perguntas frequentes**](../faq.md) — Respostas às perguntas mais comuns
* [**Alvos de calibração**](../calibration-targets.md) - Noções básicas sobre calibração de refletância
* [**Câmeras compatíveis**](../supported-cameras.md) - Hardware compatível
