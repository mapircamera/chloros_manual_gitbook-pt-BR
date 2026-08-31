# Camadas de imagem

O **menu suspenso de camadas** no canto superior direito do Visualizador de Imagens permite alternar entre todas as versões da imagem que você está visualizando — desde a captura original, passando por cada produto processado, até as imagens de índice calculadas — sem sair do visualizador.

## O que são camadas de imagem?

Uma “camada” no Chloros é um **arquivo de produto**registrado em relação a uma imagem original. A importação fornece os arquivos originais; o processamento adiciona uma camada para cada produto gerado pela execução. Os arquivos exportados mantêm o nome do arquivo de origem — é a**pasta** que identifica o produto, e o nome da camada é o rótulo do Chloros para essa pasta.

<!-- SCREENSHOT-NEEDED: Image Viewer full screen with the layer dropdown open on a processed LATTICE multispectral image, showing the full list: TIFF base, RAW (Original), RAW (Debayered), RAW (Preview), RAW (Radiance), RAW (Reflectance), and one RAW (NDVI Index) entry. -->

***

## A lista de camadas

### Sempre presente

| Camada | O que é |
| --- | --- |
| **JPG**(ou**PNG**/**TIFF**) | O arquivo base que veio junto com a captura. O Survey3 importa um `.JPG` ao lado de cada `.RAW`; As capturas LATTICE trazem uma pré-visualização de exibição do tipo PNG ou TIFF. Rotulada de acordo com o que foi realmente importado |
| **RAW (Original)** | O quadro bruto de origem, submetido a débayering para exibição, sem correções aplicadas. Disponível desde o momento da importação — não requer processamento |

Uma captura do LATTICE cujo arquivo base **é** seu quadro bruto não possui entrada base separada: o `RAW (Original)` já a abrange.

### Produtos de processamento do Survey3

| Camada | Gravada em | Existe quando |
| --- | --- | --- |
| **RAW (Alvo)** | — | O quadro foi identificado como contendo um alvo de calibração |
| **RAW (Refletância)** | `Reflectance_Calibrated_Images/` | A calibração de refletância foi executada com sucesso neste quadro |
| **Correção de vinheta**| `Vignette_Corrected_Images/` | Não foi possível realizar a calibração de refletância do quadro**e** a *correção de vinheta* estava ativada |
| **Resposta do sensor**| `Sensor_Response_Images/` | Não foi possível calibrar o quadro quanto à refletância**e** a *correção de vinheta* estava desativada |
| **Equilíbrio de branco** | `White_Balanced_Images/` | Um produto com equilíbrio de branco foi gerado |

{% hint style="info" %}
**A correção de vinheta e a resposta do sensor são alternativas; nunca ambas ao mesmo tempo.** Existe exatamente um produto de fallback não calibrado por execução, para cada modelo de câmera, e a opção *Correção de vinheta* determina qual deles será usado. Consulte [Configurações do projeto](../project-settings/project-settings.md).
{% endhint %}

### Níveis do LATTICE

O LATTICE captura o fan-out nesses níveis em uma única etapa de processamento. Quais deles existem depende das opções de exportação por produto nas Configurações do Projeto e do que se aplica à câmera.

| Camada | Gravada em | Aplica-se a |
| --- | --- | --- |
| **RAW (sem bayering)** | `Debayered_Images/` | RGB e multiespectral |
| **RAW (Pré-visualização)** | `Preview_Images/` | Multiespectral (estiramento de cores falsas) |
| **Equilíbrio de branco** | `Preview_Images/` | Câmeras principais RGB — a pré-visualização RGB está registrada sob este nome para que se alinhe com a camada Survey3 de mesmo nome |
| **RAW (Radiação)** | `Radiance_Images/` | Apenas multiespectral |
| **RAW (Radiação)** | `Reflectance_Calibrated_Images/` | Apenas multiespectral, e somente quando um registro descendente `.daq` correspondente ou um alvo no quadro aprovado no controle de qualidade (QA) cobrir o quadro |

As câmeras principais RGB não possuem radiometria por banda; portanto, a radiação e a refletância são ignoradas para elas como **não aplicáveis** — o log informa isso em vez de apresentar uma falha silenciosa.

### Camadas de índice, LUT e sandbox

| Padrão da camada | Exemplo | De onde vem |
| --- | --- | --- |
| **RAW (`<INDEX>` Índice)** | `RAW (NDVI Index)` | Um por índice configurado nas Configurações do Projeto, calculado durante o processamento |
| **`<INDEX>` LUT** | `NDVI LUT` | A versão com mapeamento de cores de um índice |
| **Sandbox (`<Name>` `<Index\|LUT>` `<NNN>`)** | `Sandbox (NDVI LUT 003)` | Uma por execução de exportação do [Índice/LUT Sandbox](index-lut-sandbox.md) |

Se o mesmo nome de índice for configurado mais de uma vez com configurações diferentes, o segundo e os seguintes recebem um número no nome (`RAW (NDVI2 Index)`) para que as camadas permaneçam distinguíveis.

***

## Usando o seletor de camadas

1. Abra uma imagem em tela cheia clicando em uma miniatura na grade
2. Clique no **menu suspenso de camadas** no canto superior direito do visualizador
3. Escolha uma camada — a imagem é atualizada imediatamente

O menu suspenso exibe **JPG, RAW (Original), RAW (Destino), RAW (Refletância)** em primeiro lugar, nessa ordem, e lista todas as demais opções a seguir, na ordem em que os produtos foram registrados.

### Preferência de camada ao navegar

Pressionar **←**/**→** avança para a próxima imagem e tenta manter você na mesma camada:

1. **Correspondência exata primeiro** — se a próxima imagem tiver uma camada com o mesmo nome, você a visualiza. É isso que mantém você na camada `RAW (NDVI Index)` ao percorrer um conjunto inteiro
2. **Em seguida, correspondência por tipo** — uma camada de índice procura por qualquer camada de índice, uma LUT por qualquer LUT, refletância por refletância, alvo por alvo, original por original, base por base
3. **Em seguida, apenas para camadas de exportação** — o nome é mantido mesmo que a lista de camadas ainda não tenha sido atualizada, pois o arquivo já existe no disco. É isso que permite revisar os produtos enquanto uma execução ainda os está gravando
4. **Caso contrário** — a primeira camada disponível, que normalmente é a imagem base

Os arquivos sidecar `.daq` e `.csv` no projeto são ignorados pela navegação com as setas, de modo que a navegação pelas imagens nunca chega a uma gravação do sensor de luz.

O zoom e a panorâmica também se aplicam às imagens, o que facilita a comparação “antes e depois” da mesma posição no campo.

***

## Compreendendo os valores dos pixels por camada

O [painel Valores do Cursor](opening-an-image-full-screen.md#cursor-values) exibe o valor real por canal sob o cursor, na unidade em que a camada está armazenada. Suas colunas variam de acordo com a camada:

| Camada | Unidade exibida | Observações |
| --- | --- | --- |
| Base (JPG / PNG / pré-visualização do TIFF) | DN, 0–255 | Valores de exibição, com correção de gama no RGB. Apenas para inspeção visual |
| RAW (Original) | DN | Valores digitais brutos do sensor. O eixo do histograma indica a profundidade: 255 (8 bits), 4095 (12 bits) ou 65535 (16 bits) |
| RAW (sem debayering) | DN | Linear, sem expansão de exibição |
| RAW (Pré-visualização) / Equilíbrio de branco | DN | Produto de exibição — expandido ou com correção de gama. Não serve para medição |
| RAW (Radiação) | **W/m²/sr/nm** | Radiação física em Float32. Sem coluna DN |
| RAW (Refletância) | DN **e %** | Porcentagem calculada com a escala própria do arquivo — veja abaixo |
| Exportações de índice / LUT / sandbox | Valor de índice ou componentes RGB | Um arquivo de índice de canal único informa o valor do índice; um arquivo LUT com mapeamento de cores relata os componentes Red/Green/Blue |

### Refletância: a escala é por arquivo

{% hint style="warning" %}
**“Dividir por 65.535” só está correto para Survey3.** A refletância do LATTICE é armazenada em uma escala diferente, e misturar os dois divisores é a maneira mais comum de obter valores de refletância que são exatamente a metade do que deveriam ser.
{% endhint %}

| Fonte | DN que equivale à refletância 1,0 | Identificado por |
| --- | --- | --- |
| **LATTICE**(M3C / M3M) |**32768** | A tag XMP `Chloros:PixelScale=32768` incluída em toda exportação de refletância do LATTICE. A margem de 2× significa que ρ acima de 1,0 é representável, em vez de ser cortado |
| **Survey3**|**65535** | Sem tag de escala XMP Chloros — a calibração Survey3 grava ρ × dtype-max e clipa em 1,0 |

Para GIS e scripts: leia `Chloros:PixelScale` do arquivo e divida pelo valor correspondente. Se a tag estiver ausente, o arquivo está na escala Survey3 (65535). O visualizador, a área de teste de índice/LUT e a exportação de índice resolvem a escala dessa mesma maneira; portanto, o número exibido no cursor é o número utilizado nos cálculos do índice.

Armazenamento específico do formato além dessa escala:

* **TIFF (32 bits, porcentagem)** armazena DN / 65535 como um número de ponto flutuante
* **PNG (8 bits)**e**JPG (8 bits)** armazenam DN × 255 / 65535
* Uma **exportação de 8 bits TIFF de uma captura com fonte de 8 bits** é limitada ao intervalo de 0 a 255, em vez de ser redimensionada, e deliberadamente não contém nenhuma tag de escala. O painel exibe apenas o DN para esses arquivos, sem a coluna de porcentagem

### Intervalos de valores de índice

| Família de índices | Intervalo típico | Leitura |
| --- | --- | --- |
| Diferença normalizada (NDVI, GNDVI, NDRE, ENDVI…) | −1 a +1 | Vegetação saudável geralmente entre 0,4 e 0,9; solo nu próximo a 0; água com valor negativo |
| Ajustado ao solo (SAVI, OSAVI, MSAVI2…) | aproximadamente de −1 a +1,5 | Valor semelhante ao de NDVI com o fundo do solo suprimido |
| Razão (GRVI, GCI, MSR, CIRE…) | sem limite superior | As razões crescem sem limite à medida que a faixa do denominador se aproxima de zero |
| EVI / LAI | 0 a ~1, 0 a ~3,5 | Nuvens e outros pixels saturados fazem com que ambos saiam da faixa — máscara-os primeiro |

Consulte [Fórmulas de índices multiespectrais](../project-settings/multispectral-index-formulas.md) para conhecer a fórmula exata por trás de cada predefinição.

***

## Fluxos de trabalho comuns

### Comparação antes/depois

1. Selecione **RAW (Original)** e observe o efeito de vinheta e os valores não calibrados
2. Alterne para **RAW (Refletância)**

3. Compare — efeito de vinheta removido, valores calibrados. O zoom e a panorâmica permanecem fixos, para que você esteja vendo a mesma área

### Revise um índice em todo o conjunto

1. Abra a primeira imagem processada e selecione a camada de índice
2. Pressione **→** repetidamente — a camada de índice acompanha você de imagem em imagem
3. Observe o histograma na barra lateral à medida que avança: um quadro cuja distribuição apresente picos merece uma análise mais detalhada

### Verifique os alvos de calibração

1. Selecione **RAW (Alvo)** em um quadro-alvo
2. Confirme se o alvo está claramente visível e foi detectado
3. Passe para o próximo quadro-alvo — a camada de alvo acompanha

### Verifique a precisão dos valores de refletância

1. Selecione **RAW (Refletância)**

2. Leia a coluna**%** no painel “Valores do cursor” — ela já está dimensionada corretamente para esse arquivo
3. Faça uma verificação de validade comparando com materiais conhecidos no quadro: vegetação saudável apresenta valores altos em NIR e baixos no vermelho; um alvo de calibração deve apresentar valores próximos à sua refletância publicada

***

## Solução de problemas

### Uma camada que eu esperava não está no menu suspenso

**Possíveis causas**

* A imagem nunca foi processada — existem apenas a camada base e a `RAW (Original)`
* A opção de exportação do produto está desmarcada nas Configurações do Projeto
* O produto não se aplica a essa câmera (radianância e refletância em uma câmera mestre RGB; qualquer índice em uma câmera M3M monocromática de banda única)
* A calibração de refletância não teve dados para trabalhar — sem cobertura descendente de `.daq` e sem alvo no quadro aprovado no controle de qualidade —, portanto, o quadro recorreu à correção de vinheta ou à resposta do sensor

**O que fazer**

1. Verifique o log da execução: o Chloros indica quando um produto de exportação solicitado foi impossível e o motivo
2. Verifique as opções de exportação por produto em [Configurações do projeto](../project-settings/project-settings.md)
3. Confirme se a pasta do produto existe na árvore de saída do projeto
4. Reprocesse com o produto habilitado

### A lista de camadas parece desatualizada

O Chloros refaz a varredura das pastas de produtos do projeto enquanto uma execução está em andamento e corrige registros de camadas ausentes com base no que realmente está no disco; portanto, uma camada cuja exportação foi concluída normalmente aparece por conta própria em uma varredura. Sair da imagem e voltar força uma nova resolução.

### Os valores de refletância parecem ser metade do que deveriam ser

É quase certo que você esteja dividindo um arquivo LATTICE por 65535. Use o `Chloros:PixelScale` (32768) ou consulte a coluna **%**, que já aplicou esse valor.

### A camada de índice existe, mas a imagem está em branco

O índice precisa de faixas que sua camada não possui — por exemplo, um índice que lê um terceiro canal aplicado a um arquivo de um ou dois canais. Mude para uma camada multibanda (refletância ou sem bayering) ou escolha um índice compatível com o filtro da câmera.

***

## Próximos passos

* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) — leitura do cursor, histograma e controle de GSD
* [**Área de testes de índices/LUT**](index-lut-sandbox.md) — visualização interativa de índices e exportação
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) — a referência de índices
* [**Conclusão do processamento**](../processing-images-gui/finishing-the-processing.md) — a árvore de pastas de saída para a qual essas camadas apontam
