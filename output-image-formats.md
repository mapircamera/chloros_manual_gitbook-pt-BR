---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagem de saída O

Chloros exporta produtos processados em quatro formatos de arquivo. Selecione o formato nas Configurações do Projeto (GUI), com `--format` (CLI) ou com `export_format` (SDK). A CLI e o SDK aceitam as sequências exatas abaixo.

| String de formato | Extensão | Tipo de pixel | Intervalo de pixels | Observações |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(padrão)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametria / SIG. |
| `TIFF (8-bit)` | `.tif` | número digital uint8 | 0 – 255 | Para todas as capturas de 8 bits, que oferecem apenas os formatos de 8 bits: o Chloros não aumenta a profundidade de bits. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% de refletância. Alguns aplicativos não conseguem ler arquivos TIFF de ponto flutuante; os arquivos são maiores. |
| `PNG (8-bit)` | `.png` | número digital uint8 | 0 – 255 | Compressão sem perdas, adequada para visualização na web e em outros meios. |
| `JPG (8-bit)` | `.jpg` | número digital uint8 | 0 – 255 | Compressão com perda, arquivos menores. |

## Onde os arquivos de saída são salvos

Os arquivos são gravados na pasta do projeto, agrupados por câmera e, em seguida, por formato de arquivo:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera (model+lens+filter)
    ├── tiff16/                          # follows --format: tiff16, tiff8, png8, jpg8, or tiff32
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── NDVI_Index_Images/           # one <INDEX>_Index_Images/ folder per requested index
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para a Survey3. **Cada produto exportado mantém o nome do arquivo de origem — a pasta identifica o produto, não um sufixo do nome do arquivo.** Consulte [Onde os arquivos de saída são salvos](reference/cli-reference.md) na Referência da CLI para conhecer todas as regras.

## Produtos LATTICE (níveis de captura e exportação)

Um quadro bruto do LATTICE é dividido em todos os produtos solicitados em uma única passagem. Cada tipo de produto possui sua própria opção de ativação/desativação (caixas de seleção na GUI ou comandos CLI: `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos ativados por padrão):

| Nível | Conteúdo | Tipo de dados |
| --- | --- | --- |
| `raw` | Dados Bayer diretamente do sensor (câmeras monocromáticas: a única banda). O processamento sempre começa a partir dos dados brutos. | Conforme capturado |
| `debayered` | Desmosaico linear — 3 canais para M3C, 1 canal em escala de cinza para M3M: o desvio de escuro do pacote de calibração é removido, seguido de um desmosaico sensível a bordas, sem correção de linearidade, campo plano ou separação de cores. Uma exportação `process` grava o mesmo arquivo que uma captura salva neste nível, em todos os modos de processamento. | DN linear |
| `radiance` | Radiação espectral absoluta da cadeia radiométrica completa, em **W/m²/sr/nm**. Sempre gravado como TIFF de 32 bits (`tiff32/Radiance_Images/`), independentemente do formato de exportação selecionado. | float32 |
| `reflectance` | Refletância ρ, onde **DN 32768 = ρ 1,0 (100%)** com margem até ρ 2,0. Compatível com Pix4D. | uint16 |
| `preview` | Renderização pronta para exibição: RGB = balanço de branco + gama; multiespectral = expansão de cores falsas. | exibição de 8 bits |

### Orientação

Os arquivos `raw` e `debayered` permanecem sempre na orientação **nativa** do sensor; a configuração de inversão/rotação da câmera é aplicada apenas aos outros produtos (visualização, radiação, refletância, índice). As capturas em matriz mantêm os arquivos de cada membro na orientação nativa, a menos que você solicite a orientação (`--orient` na CLI, `orient=True` no SDK). Cada arquivo registra a orientação correspondente aos seus pixels no XMP — `ChlorosRotation`, `ChlorosFlipHorizontal`, `ChlorosFlipVertical` — e `ChlorosOrientationApplied` indica se seus próprios pixels contêm essa informação (`false` em arquivos raw e debayered). O Visualizador de Imagens Chloros converte arquivos raw e sem bayering com base nesse registro apenas para exibição; os próprios arquivos nunca são reescritos. Um arquivo sem esse registro (criado antes da versão 1.3.0 ou por outra ferramenta) é exibido tal como está armazenado.

## Máscaras de limitação

Os produtos de radiação e refletância de uma câmera multiespectral M3C passam por uma separação de bandas 3×3. Quando a separação produz um valor negativo em uma banda, esse valor é limitado a zero (as visualizações em tempo real são limitadas exatamente da mesma maneira). Novidade na versão 1.3.0: um produto no qual pelo menos um pixel foi cortado recebe uma **máscara de limitação** associada que indica exatamente quais pixels e bandas:

* **Onde:**em uma pasta `clamp_masks/`**ao lado** das pastas de produtos, nunca dentro delas: `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif` e `tiff32/Radiance_Images/x.tif` → `tiff32/clamp_masks/Radiance_Images/x_clampmask.tif`.
* **O quê:** um arquivo TIFF de banda única `uint8` na própria grade de pixels do produto (incluindo orientação, ROI, binning e alinhamento da matriz). O bit 0 (`mask & 1`) é definido onde a banda no canal R do produto foi cortada; o bit 1 (`& 2`) no canal G, e o bit 2 (`& 4`) no canal B. A proporção de pixels cortados por banda também é indicada no produto como `Chloros:BandRUnmixClampFraction`, `Chloros:BandGUnmixClampFraction` e `Chloros:BandBUnmixClampFraction`.
* **Como isso é indicado no produto:** a tag XMP `Chloros:UnmixClampMask` contém o caminho da máscara em relação à pasta do produto (`../clamp_masks/<product folder>/<name>_clampmask.tif`, sempre separado por `/`), `none` quando nenhum pixel foi cortado (nenhuma máscara é gravada) ou `unavailable` quando não foi possível gravar uma máscara. Um produto sem tag nunca executou a separação de cores (câmeras RGB e monocromáticas, níveis raw e pós-processamento de bayering).

Como nenhuma pasta de produto contém uma máscara, você pode passar uma pasta de produto como `Reflectance_Calibrated_Images/` para o Pix4D, Metashape ou outro software de fotogrametria tal como está, incluindo as subpastas — passe a pasta de produto, não a pasta de formato acima dela. A importação do próprio Chloros ignora máscaras de limitação. A CLI as conta separadamente: `Image products written: N (plus M clamp masks)`.

## Produtos de índice

Cada índice é armazenado em sua própria pasta `<INDEX>_Index_Images/` (por exemplo, `NDVI_Index_Images/`).

* **Formatos inteiros**(`TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`) armazenam um**estiramento linear por imagem**, e não um mapeamento de −1…1: DN 0 é o valor de índice mais baixo daquele quadro e a escala total, o mais alto. O arquivo contém `Chloros:IndexScaling=linear_minmax`, `Chloros:IndexMin` e `Chloros:IndexMax`, portanto, também contém `value = IndexMin + DN / full_scale × (IndexMax − IndexMin)`. Como cada quadro tem seus próprios pontos extremos, compare os quadros por seus valores, e não por seus DNs.
* **`TIFF (32-bit, Percent)`** armazena os próprios valores de índice como float32, sem reescalonamento — a melhor opção para trabalhos quantitativos ao longo de um voo.

Novidade na versão 1.3.0: um produto de índice descreve **a si mesmo**, em vez de repetir as tags do dado bruto de origem. Suas tags de GPS e de câmera são copiadas do produto a partir do qual foi calculado (o produto de refletância, quando houver) e, em seguida:

| Tag | Valor |
| --- | --- |
| `Chloros:ProcessingLevel` | `index` (portanto, o arquivo nunca é reimportado como uma captura) |
| `Chloros:IndexApplied` / `Chloros:IndexExpression` | `True` / a fórmula utilizada |
| `Chloros:IndexLevel` | `reflectance` ou `radiance` — a partir de que o índice foi calculado (ausente quando nenhum dos dois) |
| `Chloros:IlluminationInvariant` | `True` para um índice calculado a partir da refletância, `False` a partir da radianância ou dos valores do sensor |
| `Chloros:ChlorosPipelineVersion` | a versão do Chloros que gerou o índice |

`Chloros:PixelScale` foi removido dos produtos de índice — use `IndexMin` / `IndexMax` acima. A receita completa de leitura está em “Um raster de índice inteiro é um alongamento de contraste por imagem” na [Referência CLI](reference/cli-reference.md).

## Leitura dos valores de refletância dos pixels

A refletância é armazenada como um número digital inteiro, e **o DN que corresponde a ρ = 1,0 (100% de refletância) depende da câmera de origem**:

| Câmera de origem | ρ = 1,0 corresponde a DN | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem de segurança até ρ 2,0) | A tag XMP `Chloros:PixelScale=32768` está gravada no arquivo. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há tag `Chloros:PixelScale` — essa ausência é o indicador. (Os produtos Survey3 contêm outras tags de processamento `Chloros:*`, como a `Chloros:CalibrationApplied`, em todos os modos de processamento a partir da versão 1.3.0.) |

**Leia a tag XMP `Chloros:PixelScale` e divida por ela**, em vez de assumir um valor constante. A tag é definida no domínio uint16, portanto permanece como `32768` em todos os formatos de saída que redimensionam — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float32).

{% hint style="warning" %}
**Capturas de 8 bits.** As exportações de uma captura de 8 bits são de 8 bits. Sua refletância é redimensionada e marcada como `Chloros:PixelScale=32768`, de modo que a escala se mantém; produtos DN de 8 bits e arquivos de refletância gravados antes da versão 1.3.0 são limitados ao intervalo de 0 a 255 e não contêm nenhuma marca de escala. Se a marca estiver ausente em um arquivo de refletância do LATTICE, não presuma que haja uma escala; em vez disso, reexporte em 16 bits ou 32 bits.
{% endhint %}

Para conhecer as regras completas (incluindo as tags compatíveis com MicaSense), consulte **“Lendo pixels de refletância”** na [Referência da CLI](reference/cli-reference.md).
