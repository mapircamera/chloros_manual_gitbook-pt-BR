# Câmeras monocromáticas e índices de vegetação

## Uma câmera = uma banda

A câmera **M3M**é a versão monocromática da**M3C**da Bayer: um sensor IMX265 monocromático protegido por um único filtro de interferência de banda estreita. A sequência do modelo indica o nome da banda — `M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685` (exibido em Chloros como `LATT-M3M-L87-F685`). O sensor fornece uma**única banda em escala de cinza** sem mosaico Bayer: não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de branco para ajustar.

Consequências que vale a pena conhecer antes de planejar um sistema monocromático:

* **A radianância e a refletância são totalmente definidas por banda.**São mapas radiométricos por banda; portanto, uma câmera M3M produz radiancia calibrada em float32 (W/m²/sr/nm) e refletância em uint16 (`32768` = ρ 1,0) exatamente como uma banda M3C. Os quadros monocromáticos contêm uma matriz de resposta do sensor**identitária** — não é necessário nem aplicado nenhum processo de desmistura 3×3.
* **Uma única câmera mono não pode produzir um índice de vegetação.** NDVI, NDRE e similares precisam de pelo menos duas bandas. Para calcular índices a partir de hardware mono, é preciso combinar várias câmeras M3M — veja abaixo.
* As câmeras M3M transmitem **Mono12** (12 bits, 2 bytes/pixel na transmissão), o que é importante para o [planejamento da largura de banda do array](arrays.md#bandwidth-the-rules-of-thumb).

## O que o Chloros ignora no modo mono — e como ele avisa

As etapas do pipeline de cores simplesmente não se aplicam a um sensor de banda única. O Chloros **os ignora com uma mensagem de uma linha** em vez de gerar erro, e ainda assim os executa normalmente para qualquer câmera M3C (Bayer) na mesma sessão:

| Etapa | Comportamento Mono (M3M) | Comportamento M3C |
| --- | --- | --- |
| Demosaico / debayer | Ignorado — o nível de exportação do `debayered` é uma imagem em escala de cinza de 1 canal. | Demosaico de 3 canais. |
| Equilíbrio de branco (`lattice white-balance`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Perfil de cor (`lattice color-profile`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Saturação/contraste (`lattice color`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Desmistura de interferência espectral | Identidade (sem matriz 3×3). | Matriz 3×3 aplicada por câmera. |
| Radiação / refletância | **Executado** — por banda, totalmente calibrado. | Executado por banda. |

A interface gráfica aplica o mesmo filtro: para uma câmera monocromática, o painel de configurações por câmera oculta as linhas exclusivas do RGB (Equilíbrio de branco, Gama, Perfil de cor, Saturação, Contraste, divisões de canal), e o histograma em tempo real fica restrito a um único traço **MONO**. O discriminador em toda a pilha é o token `M3M` na string do modelo, exibido na GUI/SDK como `is_mono`.

## Os índices precisam de ≥ 2 bandas: alinhar → empilhar → indexar

O fluxo de trabalho do índice mono segue sempre as mesmas três etapas:

1. **Alinhar** — apontar várias câmeras M3M em diferentes comprimentos de onda (por exemplo, uma F650 “Red” e uma F850 “NIR”), conecte-as como um [arranjo multicâmera](arrays.md) e deixe que o Chloros calcule a distorção de co-registro entre as câmeras.
2. **Pilha** — os quadros alinhados se tornam uma imagem multibanda (cada câmera contribui com uma banda nomeada).
3. **Índice** — avalie uma fórmula de índice sobre as bandas da pilha, opcionalmente renderizando-a por meio de uma LUT.

Na interface gráfica, toda essa cadeia corresponde ao modo de exibição **Câmeras Combinadas**: a composição ao vivo já está alinhada, e a Calculadora de Índice da matriz (abaixo) define a fórmula que ela renderiza. As exportações capturadas podem ser distorcidas para o mesmo alinhamento com a opção de captura**Alinhada**.

## A Calculadora de Índice

A Calculadora de Índice cria a expressão de índice usada pela visualização ao vivo e pelas exportações de índice por câmera. Trata-se de uma superfície compartilhada, acessível a partir de dois locais na barra lateral da aba “Câmeras”:

* **Por câmera**— Visualização ao Vivo → ícone de engrenagem**Índice** (apenas câmeras Bayer RGN/OCN/NGB; uma câmera monocromática isolada não possui controle de índice, pois uma única banda não permite a formação de um índice).
* **Por conjunto**— configurações do conjunto → Visualização ao Vivo → ícone de engrenagem**Índice**. Este é o caminho para câmeras monocromáticas: a lista de bandas abrange**todas as câmeras do conjunto**, de modo que um par monocromático contribui com suas duas bandas aqui.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Seus controles, de cima para baixo:

* **Chips de banda** (“Bandas — clique para adicionar à expressão”) — um botão por banda disponível, rotulado com o nome da cor + comprimento de onda em nm (nomes de cores duplicados são diferenciados como, por exemplo, “Cor 850”). Clicar insere o token da banda na posição do cursor. As bandas de câmeras que não conseguem produzir radiação por banda (RGB/FRGB) são filtradas.
* **Botões de operadores e funções** — `+ - * / ( ) ^ ,` e `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Área de texto para expressões** — fórmula digitada livremente; o espaço reservado exibe o formato clássico NDVI de `(NIR - Red) / (NIR + Red)`. Uma pré-visualização tokenizada somente para leitura acima dela renderiza os chips de banda, números e sinalizadores como tokens desconhecidos.
* **Banner de validade**— cinza “Vazio — nenhum índice será aplicado”; verde “Expressão válida”; vermelho com o erro de análise específico (banda desconhecida, banda ambígua captada por múltiplas câmeras, parêntese ausente, …); ou âmbar quando a expressão é válida, mas**constante** (por exemplo, `X/X`, ou um denominador NDVI digitado com `−` em vez de `+`) — uma constante mapeia o quadro inteiro para uma única cor.
* Um aviso âmbar separado aparece se a expressão aplicada estiver correta, mas o **quadro ao vivo for uniforme** (cena plana ou saturada) — o colapso do histograma é detectado automaticamente para você.
* **Aplicar LUT**(padrão: ativado; desativado = alongamento em escala de cinza),**Nível**de 2/3/5/7 stops (padrão: 7 stops) e entradas**Mín. / Máx.**ao lado da barra de gradiente. O valor padrão de Mín. é**0,2**— ele amplia a rampa de cores para a faixa relevante para a vegetação, enquanto valores abaixo disso são exibidos em escala de cinza; defina Mín. como −1 para a faixa completa do índice (o botão**Redefinir** restaura o intervalo de −1 a +1). O valor padrão de Máx. é 1.
* **Histograma em tempo real** da distribuição do índice — barras em escala de raiz quadrada, linhas âmbar nos percentis p2/p98, uma linha mediana branca e indicações nas extremidades fora do intervalo (“◀ N% &lt; lo” / “hi &lt; N% ▶&quot;) que ficam âmbar acima de 1 % como indicação para ampliar a janela de Mín./Máx.
* **Aplicar**confirma a expressão no fluxo ao vivo; ajustes de LUT são aplicados em tempo real sem precisar pressionar Aplicar. As expressões são deliberadamente**exclusivas da sessão** — elas não são mantidas entre sessões.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->

## O caminho CLI

A mesma cadeia de alinhamento → pilha → índice, programável de ponta a ponta:

```bash
chloros-cli lattice array-connect --serials SN_RED,SN_NIR
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`--channel` mapeia os símbolos de uma predefinição para os nomes das faixas da pilha. Duas regras evitam uma execução com falha:

* **Os símbolos diferenciam maiúsculas de minúsculas** e devem corresponder exatamente aos nomes dos canais da predefinição — as predefinições usam letras minúsculas (as do NDVI são `red`,`nir`; verifique `--list-presets`). `--channel red=Red_660` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.
* O lado da banda deve nomear uma banda na pilha alinhada (`lattice align-info --profile align.json` as lista). O modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`.

O `lattice index` também é executado totalmente offline em relação a um TIFF multibanda alinhado e salvo:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Predefinições de índice

O `lattice index --preset` (e a [Área de teste de índice/LUT](../image-viewer-gui/index-lut-sandbox.md) da aba Imagem, que utiliza o mesmo mecanismo) inclui estas **22 predefinições**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Execute o `chloros-cli lattice index --list-presets` para acessar a fórmula e os símbolos de canal de cada predefinição, e o `--list-gradients` para ver os gradientes de cor disponíveis. Fórmulas personalizadas utilizam o `--formula EXPR` com a mesma sintaxe da Calculadora de Índice. Observe que esta lista de predefinições é específica do mecanismo de índices LATTICE — o menu suspenso de processamento em “Configurações do Projeto” para imagens importadas apresenta uma lista diferente (consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md)).

O conjunto completo de sinalizadores (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, controles deslizantes de alinhamento e distorção para `--live`, e outros) está documentado na [Referência do CLI § Índice / Matemática da Vegetação](../reference/cli-reference.md#index--vegetation-maths); os equivalentes do SDK encontram-se na [Referência do SDK](../reference/sdk-reference.md).

## Captura de produtos de índice a partir de uma matriz mono

Com uma matriz conectada e uma expressão de índice aplicada, o `array-capture` (ou a opção **Capturar tudo** da interface gráfica) salva os níveis de exportação por câmera *e* a renderização do índice — o `--index`/`--no-index` alterna essa opção no CLI, e a captura inclui, por padrão, todos os níveis aplicáveis. A contribuição de uma câmera mono para cada grupo de captura é sua única banda nos níveis bruto/despixelação (escala de cinza)/radiação/refletância, além da composição de índice combinado compartilhada quando a matriz opera no modo combinado. Consulte [Matrizes com várias câmeras § Captura](arrays.md#capturing-monitoring-vs-analysis).
