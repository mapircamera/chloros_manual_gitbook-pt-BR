# Conclusão do processamento

Assim que o Chloros concluir o processamento, é hora de analisar os resultados, verificar a qualidade da saída e preparar as imagens processadas para uso em seu fluxo de trabalho. Esta página orienta você nas etapas finais e nas próximas ações.

## Indicação de processamento concluído

Quando o processamento for concluído com sucesso, você verá vários indicadores:

* ✅ **Barra de progresso**: Atinge 100% de conclusão
* ✅ **Log de depuração**: Mostra as linhas finais do `[RUN-SUMMARY]` com as contagens (imagens, grupos de câmeras, alvos, imagens calibradas, arquivos gravados)
* ✅ **Botão Iniciar**: fica habilitado novamente (pronto para a próxima execução do processamento)
* ✅ **Arquivos de saída**: todas as imagens processadas são salvas na árvore de saída do projeto (abaixo)

{% hint style="warning" %}
**Uma execução que não grava imagens é considerada uma falha.** Se você solicitou produtos de imagem e a execução não gravou nenhum, o Chloros reporta isso como uma falha — o `[RUN-SUMMARY]` sugere no nome do log a causa provável (nada importado, nenhum alvo detectado ou todos os produtos solicitados ignorados por serem inaplicáveis). O equivalente CLI retorna um valor diferente de zero. Uma execução deliberada apenas com metadados (todos os produtos de exportação desativados, sem índices) ainda é considerada um sucesso. Consulte [a Referência do CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails).
{% endhint %}

***

## Localizando suas imagens processadas

### Abrindo a pasta de saída

1. Clique no ícone do **Menu Principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (canto superior esquerdo)
2. Selecione **&quot;Abrir pasta do projeto&quot;**

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
* **Pasta de formato**: segue sua configuração de formato de exportação — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32` até TIFF (32 bits, porcentagem). A radiancia é sempre do tipo float32 e sempre é armazenada em `tiff32`.
* **Pastas de produtos**:
  * `Reflectance_Calibrated_Images/` — refletância calibrada
  * `Debayered_Images/` — sem bayering linear (LATTICE)
  * `Preview_Images/` — visualização na tela (LATTICE)
  * `Radiance_Images/` — radiancia espectral float32, W/m²/sr/nm (LATTICE multiespectral)
  * `Vignette_Corrected_Images/` **ou** `Sensor_Response_Images/` — o valor de fallback não calibrado para quadros sem referência de refletância; existe exatamente um dos dois por execução, escolhido pela configuração de correção de vinheta
  * `<INDEX>_Index_Images/` — uma pasta por índice selecionado (por exemplo, `NDVI_Index_Images`)

{% hint style="info" %}
**Cada produto exportado mantém o nome do arquivo de ORIGEM.**Uma exportação de radiação de `capture_..._raw.tif` ainda é chamada de `capture_..._raw.tif` — ela apenas fica na pasta `tiff32/Radiance_Images/`.**A pasta identifica o produto, não o nome do arquivo**; portanto, procurar por `*radiance*.tif` não resulta em nada; em vez disso, procure pelo diretório.
{% endhint %}



<!-- SCREENSHOT-NEEDED: Windows Explorer open on a processed project folder showing the tree: a LATT-… camera folder expanded with tiff16 (Reflectance_Calibrated_Images, Debayered_Images, Preview_Images, NDVI_Index_Images) and tiff32 (Radiance_Images) subfolders visible -->### Quantos arquivos devem existir?

Não conte com base em fórmulas — o número de saídas depende de quais produtos foram habilitados e quais se aplicam a cada câmera (por exemplo, as câmeras RGB não recebem radiação/refletância). A contagem oficial está no log: a linha final `[RUN-SUMMARY]` informa exatamente quantos arquivos foram gravados, e as linhas de dica explicam tudo o que foi ignorado.

***

## Revisão das imagens processadas

### Visualização rápida no Explorador de Arquivos

**Visualização integrada do Windows:**

1. Navegue até uma pasta de produto (por exemplo, `tiff16/Reflectance_Calibrated_Images/`)
2. Selecione um arquivo de imagem
3. A pré-visualização aparece no painel de pré-visualização do Windows Explorer
4. Use as setas do teclado para percorrer as imagens

### Pré-visualização em visualizadores de imagem externos

**Visualizadores recomendados:*** **QGIS** - Software SIG gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** - Visualizador de imagens rápido e leve (compatível com TIFF)
* **Adobe Photoshop** - Edição profissional (compatível com TIFF)
* **GIMP** - Alternativa gratuita ao Photoshop
* **Windows Photos** - Visualização básica (pode não oferecer suporte a TIFF de 16 bits)

### Visualização no visualizador de imagens do Chloros

Use o Visualizador de Imagens integrado do Chloros para visualização avançada:

1. Clique na miniatura de uma imagem no Navegador de Arquivos
2. A imagem é aberta na área principal de visualização
3. Clique na guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Leitura dos valores de refletância dos pixels (GIS / Pix4D / Scripts)

A refletância é armazenada como um valor inteiro de DN, e **o valor de DN que corresponde a ρ = 1,0 depende da câmera de origem**:

| Origem          | ρ = 1,0 corresponde a | Como identificar                                        |
| --------------- | ---------- | -------------------------------------------------- |
| LATTICE (M3C/M3M) | **32768** (margem de até ρ 2,0) | A tag XMP `Chloros:PixelScale=32768` é inserida no arquivo |
| Survey3         | **65535** (cortado em ρ 1,0)     | Não há tags XMP `Chloros:*` — essa ausência é o sinal |

**Leia a tag `Chloros:PixelScale` e divida pelo valor dela** em vez de assumir um valor genérico de 65535 — dividir a refletância do LATTICE por 65535 reduz silenciosamente pela metade todos os valores. Um caso extremo não carrega nenhuma escala por design: uma captura de fonte de 8 bits gravada como saída de 8 bits é cortada, não redimensionada, e deliberadamente não recebe nenhuma tag de escala — reexporte em 16 bits ou 32 bits em vez de dividi-la. Consulte [Formatos de imagem de saída](../output-image-formats.md) para obter a explicação completa.***

## Metadados transferidos para as exportações

Cada produto mantém o **bloco GPS**da captura de origem e seu**sub-IFD EXIF**; portanto, uma
exportação carrega `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber`, além da georreferência.

{% hint style="warning" %}
**Se um ortomosaico for gerado em uma escala absurda, verifique primeiro o `FocalLength`.**
O Pix4D calcula a distância entre amostras no solo a partir da distância focal mais a altitude. Sem essa informação,
ele recorre a uma escala totalmente errada — em um voo medido com 49 capturas, um pomar de laranjas de 411 m × 160 m
foi reconstruído como 47,8 km × 13 km, produzindo uma ortomosaica de 455 megapixels composta principalmente
por espaço vazio. A lentidão na divisão em blocos e um arquivo inesperadamente grande são sintomas disso, não
problemas separados.

```bash
exiftool -FocalLength -GPSLatitude "YourProject/.../some_export.tif"
```
{% endhint %}

Nem *todas* as tags são copiadas. As tags estruturais do IFD0 são deliberadamente deixadas de fora (copiá-las
corrompe a saída do LATTICE), e `ExifImageWidth` / `ExifImageHeight` são excluídas
porque descrevem a captura original — uma exportação redimensionada, caso contrário,
reivindicaria dimensões que sua própria malha contradiz.

***

## Analisando o Log de Depuração

### Verifique se há avisos ou erros

1. Abra a guia **Log de depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">
2. Percorra as mensagens
3. Procure por avisos em amarelo ou erros em vermelho
4. Leia as linhas `[RUN-SUMMARY]` e quaisquer dicas
5. Entre em contato com o suporte da MAPIR para obter assistência

### Salvando o log

Para manter um registro do processamento ou para enviar ao suporte da MAPIR:

1. Clique no botão **“Copiar”**ou**“Baixar”**

2. Salve como arquivo de texto na pasta do projeto
3. Inclua na documentação do projeto
4. Envie ao suporte do MAPIR caso encontre problemas

***

## Problemas comuns de saída e soluções

### Problema: Arquivos de saída ausentes

**Possíveis causas:**

* O produto não é compatível com essa câmera (por exemplo, radianância/refletância para câmeras RGB — o log indica isso)
* Faltou uma referência obrigatória (por exemplo, refletância sem alvo e sem radiação descendente do `.daq`)
* A caixa de seleção de exportação do produto estava desativada nas Configurações do Projeto
* O espaço em disco esgotou-se durante a exportação

**Soluções:**

1. Verifique as dicas `[RUN-SUMMARY]` e as linhas `[EXPORT-CHECK]` no Log de Depuração — elas explicam as omissões por câmera
2. Verifique as caixas de seleção de produtos de exportação em [Configurações do Projeto](adjusting-project-settings.md)
3. Verifique se havia espaço em disco suficiente
4. Reprocesse após corrigir a causa

### Problema: Bordas escuras ou claras (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmera/lente não consta no banco de dados de perfis do Chloros
* Vinheta extrema, além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta está ativada nas Configurações do projeto
2. Verifique se o modelo da câmera foi detectado corretamente
3. Entre em contato com o suporte do MAPIR se a vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Nenhum alvo de calibração detectado
* Modelo incorreto de alvo de calibração selecionado
* Calibração de refletância desativada
* Imagens de alvo de baixa qualidade

**Soluções:**

1. Verifique se a calibração de refletância está ativada
2. Verifique as mensagens “Alvo encontrado” no Log de Depuração
3. Analise a qualidade das imagens dos alvos
4. Reprocesse com os alvos corretos marcados

### Problema: Os valores do NDVI parecem incorretos

**Intervalos esperados para NDVI:*** **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação esparsa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9**Se os valores estiverem fora desses intervalos:**

1. Verifique se a calibração de refletância foi aplicada
2. Verifique se o registro do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detectados
4. Certifique-se de que o modelo correto da câmera foi detectado
5. Revise o momento e as condições de captura da imagem-alvo
6. Se você mesmo estiver calculando os índices a partir dos arquivos de refletância, confirme se você dividiu pelo valor `Chloros:PixelScale` do arquivo (veja acima)

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
3. **Fluxos de trabalho calibrados**: use imagens de refletância para precisão científica — a refletância do LATTICE contém as tags de calibração XMP que o Pix4D lê
4. **Processe mosaicos de índice**: crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exporte GeoTIFF georreferenciados**: para uso em aplicativos SIG

### Para análise SIG

**Fluxo de trabalho recomendado:**

1.**Carregue no QGIS, ArcGIS ou similar**

2.**Utilize imagens de refletância de 16 bits TIFF** para análise multibanda (dividir pelo valor do arquivo `Chloros:PixelScale`)
3. **Utilizar imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas para uso
4. **Calculadora raster**: Combine bandas para análises personalizadas
5. **Exportar**: Crie mapas de classificação, detecção de mudanças e mapas de saúde da vegetação

### Para análise direta / geração de relatórios

**Fluxo de trabalho recomendado:**

1.**Use imagens de índice com cores LUT** para relatórios visuais
2. **Extraia estatísticas**: Média de NDVI por campo/parcela
3. **Série temporal**: Compare índices entre várias sessões
4. **Gere relatórios**: Inclua mapas, estatísticas e visualizações***

## Arquivamento e backup

### Estratégia de backup recomendada

**O que salvar:*** ✅ **Imagens RAW/JPG originais ou capturas brutas do LATTICE** – Arquive em uma unidade separada ou na nuvem; o arquivo bruto é a fonte do pipeline e tudo o mais pode ser regenerado a partir dele
* ✅ **Arquivos do sensor de luz `.daq` / `.csv`** — Necessários para recalcular a refletância posteriormente
* ✅ **Resultados processados** — Mantenha imagens calibradas e índices
* ✅ **Pasta do projeto** (`project.json` e arquivos associados) — Contém todas as configurações para reprocessamento, se necessário
* ✅ **Log de depuração** – Documenta os detalhes do processamento
* ✅ **Imagens-alvo de calibração** – Para verificação e reprocessamento**Recomendações de armazenamento:*** **Backup imediato**: Disco rígido externo
* **Arquivamento de longo prazo**: Armazenamento na nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: mantenha 2 a 3 cópias em locais diferentes***

## Próximas execuções de processamento

### Reutilização das configurações do projeto

Caso seja necessário processar conjuntos de dados semelhantes no futuro:

1. **Salve o modelo do projeto** (caso ainda não tenha feito isso)
2. **Crie um novo projeto** usando o modelo salvo
3. **Importe novas imagens**

4.**Processe**com configurações idênticas para garantir a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:**Opção 1: GUI – Vários projetos**

* Crie um projeto separado para cada sessão
* Use configurações consistentes no modelo
* Processe uma por vez

**Opção 2: Chloros CLI (somente a partir da versão Chloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte a [Documentação do CLI](../CLI.md) e a [Referência do CLI](../reference/cli-reference.md)

**Opção 3: Python SDK (somente Chloros+)**

* Controle programático
* Integração com pipelines de análise
* Consulte a [Documentação do API](../api-python-sdk.md) e a [Referência do SDK](../reference/sdk-reference.md)

***

## Solução de problemas no pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Guarde as imagens originais (nunca as exclua)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel “Configurações do projeto”
4. Processe novamente — os resultados são salvos nas mesmas pastas de produto, portanto, os arquivos com o mesmo nome da execução anterior serão substituídos

### Processamento de um subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que precisam ser reprocessadas
3. Use o mesmo modelo de configurações
4. Processe um conjunto de dados menor

### Obtendo ajuda

Caso encontre problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o log de depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Perguntas frequentes**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)***

## Resumo: Fluxo de trabalho completo

Você concluiu agora todo o fluxo de trabalho de processamento do Chloros:

1. ✅ **Projeto criado** – Consulte [Projetos](../projects.md)
2. ✅ **Adicionou arquivos** — Consulte [Adicionando arquivos](adding-files-to-a-project.md)
3. ✅ **Ajustou as configurações** — Consulte [Ajustando as configurações do projeto](adjusting-project-settings.md)
4. ✅ **Alvos marcados** – Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
5. ✅ **Processamento iniciado** – Consulte [Iniciando o processamento](starting-the-processing.md)
6. ✅ **Progresso monitorado** — Consulte [Monitorando o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** — Esta página**Suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Recursos avançados

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) — Visualização e análise interativas
* [**Área de testes de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) — Testes personalizados de índices
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — Referência completa de índices

### Automação e integração

* [**Documentação do CLI**](../CLI.md) — Processamento em lote via linha de comando
* [**Python SDK**](../api-python-sdk.md) - Automação programática
* [**Recursos do Chloros+**](../#chloros) - Recursos avançados de processamento

### Suporte e Aprendizado

* [**Perguntas frequentes**](../faq.md) - Respostas às perguntas mais comuns
* [**Alvos de calibração**](../calibration-targets.md) - Entendendo a calibração de refletância
* [**Câmeras compatíveis**](../supported-cameras.md) - Hardware compatível
