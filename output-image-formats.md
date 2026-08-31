---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/output-image-formats
---

# Formatos de imagem de saída

O Chloros exporta produtos processados em quatro formatos de arquivo. Selecione o formato nas Configurações do Projeto (GUI), com o `--format` (CLI) ou com o `export_format` (SDK). O CLI e o SDK aceitam as sequências de caracteres exatas abaixo.

| String de formato | Extensão | Tipo de pixel | Intervalo de pixels | Observações |
| --- | --- | --- | --- | --- |
| `TIFF (16-bit)` *(padrão)* | `.tif` | número digital uint16 | 0 – 65535 | Recomendado para fotogrametria / SIG. |
| `TIFF (32-bit, Percent)` | `.tif` | float32 | 0,0 – 1,0 | 1,0 = 100% de refletância. Alguns aplicativos não conseguem ler arquivos TIFF com pontos flutuantes; os arquivos são maiores. |
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

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para a LATTICE e `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para a Survey3. **Cada produto exportado mantém o nome do arquivo de origem — a pasta identifica o produto, não um sufixo do nome do arquivo.** Consulte [Onde os resultados são salvos](reference/cli-reference.md) na Referência do CLI para conhecer todas as regras.

## Produtos LATTICE (níveis de captura e exportação)

Um quadro bruto do LATTICE é dividido em todos os produtos solicitados em uma única passagem. Cada tipo de produto possui seu próprio botão de ativação (caixas de seleção na interface gráfica ou CLI `--debayered` / `--preview` / `--radiance` / `--reflectance`, todos ativados por padrão):

| Nível | Conteúdo | Tipo de dados |
| --- | --- | --- |
| `raw` | Dados Bayer diretamente do sensor (câmeras monocromáticas: a única banda). O processamento sempre começa a partir dos dados brutos. | Conforme capturado |
| `debayered` | Desmosaico linear — 3 canais para M3C, escala de cinza de 1 canal para M3M. | DN linear |
| `radiance` | Radiação espectral absoluta da cadeia radiométrica completa, em **W/m²/sr/nm**. Sempre gravado como TIFF de 32 bits (`tiff32/Radiance_Images/`), independentemente do formato de exportação selecionado. | float32 |
| `reflectance` | Refletância ρ, onde **DN 32768 = ρ 1,0 (100%)** com margem até ρ 2,0. Compatível com Pix4D. | uint16 |
| `preview` | Renderização pronta para exibição: RGB = balanço de branco + gama; multiespectral = expansão de cores falsas. | exibição de 8 bits |

## Leitura dos valores de refletância dos pixels

A refletância é armazenada como um número digital inteiro, e **o DN que corresponde a ρ = 1,0 (100% de refletância) depende da câmera de origem**:

| Câmera de origem | ρ = 1,0 corresponde a DN | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem de segurança até ρ 2,0) | A tag XMP `Chloros:PixelScale=32768` está gravada no arquivo. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há tags XMP `Chloros:*` — essa ausência é o indício. |

**Leia a tag XMP `Chloros:PixelScale` e divida por ela**, em vez de supor uma constante. A tag é definida no domínio uint16, portanto permanece `32768` em formatos de saída que redimensionam — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float32).

{% hint style="warning" %}
**Um caso não possui escala, por padrão.** Quando uma captura de fonte de 8 bits (BayerRG8) é gravada como TIFF de 8 bits, o pipeline limita o valor a 0–255 em vez de reescalar, portanto, o arquivo não possui escala — o Chloros omite deliberadamente o `Chloros:PixelScale` nesse caso. Se a tag estiver ausente em um arquivo de refletância do LATTICE, não presuma que haja escala; reexporte em 16 bits ou 32 bits.
{% endhint %}

Para as regras completas (incluindo as tags compatíveis com MicaSense), consulte **“Leitura de pixels de refletância”** na [Referência do CLI](reference/cli-reference.md).
