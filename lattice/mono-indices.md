# Câmeras monocromáticas e índices de vegetação

## Uma câmera = uma banda

A câmera **M3M**é a versão monocromática da**M3C** da Bayer: um sensor IMX265 monocromático atrás de um único filtro de interferência de banda estreita. A sequência do modelo indica o nome da banda — `M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685` (exibida no Chloros como `LATT-M3M-L87-F685`). O sensor oferece uma **única banda em escala de cinza**, sem mosaico Bayer: não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de branco para ajustar.

Consequências que vale a pena conhecer antes de planejar um sistema monocromático:

* **A radianância e a refletância são totalmente definidas por banda.**São mapas radiométricos por banda; portanto, uma câmera M3M produz radiancia calibrada em float32 (W/m²/sr/nm) e refletância em uint16 (`32768` = ρ 1,0) exatamente como uma banda M3C. Os quadros monocromáticos contêm uma matriz de resposta do sensor**identitária** — não é necessária nem aplicada nenhuma separação 3×3.
* **Uma única câmera monocromática não pode produzir um índice de vegetação.** O NDVI, o NDRE e outros índices semelhantes requerem pelo menos duas bandas. Para calcular índices a partir de hardware monocromático, é preciso combinar várias câmeras M3M — veja abaixo.
* As câmeras M3M transmitem **Mono12** (12 bits: 2 bytes/pixel na transmissão, ou 1,5 bytes/pixel quando um array usa [transporte compactado de 12 bits](arrays.md#packed-12-bit-transport)), o que é importante para [o planejamento da largura de banda do array](arrays.md#bandwidth-how-many-cameras-fit).

## O que o Chloros ignora no modo mono — e como ele avisa

Os estágios do pipeline de cores simplesmente não se aplicam a um sensor de banda única. O Chloros **os ignora com uma mensagem de uma linha** em vez de gerar um erro, e ainda assim os executa normalmente para qualquer câmera M3C (Bayer) na mesma sessão:

| Etapa | Comportamento no modo monocromático (M3M) | Comportamento no M3C |
| --- | --- | --- |
| Demosaico / debayer | Ignorado — o nível de exportação `debayered` é uma imagem em escala de cinza de 1 canal. | Demosaico de 3 canais. |
| Equilíbrio de branco (`lattice white-balance`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Perfil de cor (`lattice color-profile`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Saturação/contraste (`lattice color`) | Ignorado com uma mensagem de uma linha. | Executa normalmente. |
| Separação de interferência espectral | Identidade (sem matriz 3×3). | Matriz 3×3 aplicada por câmera. |
| Radiação / refletância | **Executa** — por banda, totalmente calibrado. | Executa por banda. |

A interface gráfica aplica o mesmo filtro: para uma câmera monocromática, o painel de configurações por câmera oculta as linhas exclusivas de RGB (Equilíbrio de Brancos, Gama, Perfil de Cor, Saturação, Contraste, divisões de canal), e o histograma em tempo real é restrito a um único traço **MONO**. O discriminador em toda a pilha é o token `M3M` na string do modelo, exibido na GUI/no SDK como `is_mono`.

## Os índices precisam de ≥ 2 bandas: alinhar → empilhar → indexar

O fluxo de trabalho do índice mono segue sempre as mesmas três etapas:

1. **Alinhar** — apontar várias câmeras M3M em diferentes comprimentos de onda (por exemplo, uma F650 “Red” e uma F850 “NIR”), conectá-las como um [arranjo multicâmera](arrays.md) e deixe o Chloros calcular a distorção de co-registro entre as câmeras.
2. **Empilhar** — os quadros alinhados se tornam uma imagem multibanda (cada câmera contribui com uma banda nomeada).
3. **Índice** — avalie uma fórmula de índice sobre as bandas da pilha, opcionalmente renderizando-a por meio de uma LUT.

Na interface gráfica, toda essa cadeia corresponde ao modo de exibição de matriz **Câmeras Combinadas**: a composição ao vivo já está alinhada, e a Calculadora de Índice da matriz (abaixo) define a fórmula que ela renderiza. As exportações capturadas podem ser ajustadas para o mesmo alinhamento com a opção de captura**Alinhado**.

## A Calculadora de Índice

A Calculadora de Índice cria a expressão de índice usada pela visualização ao vivo e pelas exportações de índice por câmera. Trata-se de uma interface compartilhada, acessível a partir de dois locais na barra lateral da aba Câmeras:

* **Por câmera**— Visualização ao vivo → ícone de engrenagem**Índice** (somente câmeras Bayer RGN/OCN/NGB; uma única câmera monocromática não possui controle de índice, pois uma única banda não permite a criação de um índice).
* **Por matriz**— configurações da matriz → Visualização ao vivo → ícone de engrenagem**Índice**. Esse é o caminho para câmeras monocromáticas: a lista de bandas abrange**todas as câmeras da matriz**, portanto, um par monocromático contribui com suas duas bandas aqui.

<!-- SCREENSHOT-NEEDED: Index Calculator pane opened for a combined array of two mono cameras (e.g. F650 + F850): band chips row showing the two bands with wavelength labels, the operator buttons, the expression textarea containing "(NIR - Red) / (NIR + Red)", the green "Valid expression" banner, the LUT controls (Apply LUT checked, Level 7-stop, Min 0.2 / Max 1), and the live histogram with p2/p98 percentile lines. -->

Seus controles, de cima para baixo:

* **Chips de banda** (“Bandas — clique para adicionar à expressão”) — um botão por banda disponível, rotulado com o nome da cor + comprimento de onda em nm (nomes de cores duplicados são diferenciados como, por exemplo, “Cor 850”). Clicar insere o token da banda no cursor. As bandas de câmeras que não conseguem produzir radiação por banda (RGB/FRGB) são filtradas.
* **Botões de operadores e funções** — `+ - * / ( ) ^ ,` e `abs() sqrt() log() log10() exp() min() max() pow()`.
* **Área de texto para expressões** — fórmula digitada livremente; o espaço reservado exibe o formato clássico do NDVI: `(NIR - Red) / (NIR + Red)`. Uma pré-visualização tokenizada somente para leitura, localizada acima, exibe os chips de banda, números e sinalizadores como tokens desconhecidos.
* **Banner de validade**— cinza “Vazio — nenhum índice será aplicado”; verde “Expressão válida”; vermelho com o erro de análise específico (banda desconhecida, banda ambígua captada por várias câmeras, parêntese ausente, …); ou âmbar quando a expressão é válida, mas**constante** (por exemplo, `X/X`, ou um denominador NDVI digitado como `−` em vez de `+`) — uma constante mapeia todo o quadro para uma única cor.
* Um aviso âmbar separado aparece se a expressão aplicada estiver correta, mas o **quadro ao vivo for uniforme** (cena plana ou saturada) — o colapso do histograma é detectado automaticamente.
* **Aplicar LUT**(padrão: ativado; desativado = alongamento da escala de cinza),**Nível**de 2/3/5/7 stops (padrão: 7 stops) e entradas**Mín. / Máx.**ao lado da barra de gradiente. O valor padrão de Mín. é**0,2**— ele amplia a rampa de cores para a faixa relevante para a vegetação, enquanto valores abaixo disso passam como escala de cinza; defina Mín. como −1 para a faixa completa do índice (o botão**Redefinir** restaura o intervalo de −1 a +1). O valor padrão de Máx. é 1.
* **Histograma em tempo real** da distribuição do índice — barras em escala de raiz quadrada, linhas âmbar nos percentis p2/p98, uma linha mediana branca e indicações nas extremidades fora do intervalo (“◀ N% < lo” / “hi < N% ▶”) que ficam âmbar acima de 1 % como indicação para ampliar a janela de Mín./Máx.
* **Aplicar** confirma a expressão no fluxo ao vivo; os ajustes de LUT são aplicados em tempo real sem precisar pressionar Aplicar. Com um projeto aberto, as configurações de expressão e LUT são salvas com o projeto e restauradas quando você o reabre; sem projeto, elas permanecem apenas durante a sessão.

<!-- SCREENSHOT-NEEDED: Combined-array live tile rendering NDVI from a mono pair through the default 7-stop LUT, with the array name pill and fps readout visible — the result of applying the expression from the previous screenshot. -->



## O caminho da CLI



A mesma cadeia alinhar → empilhar → indexar, programável de ponta a ponta:

```bash
chloros-cli lattice align-calibrate --serials SN_RED,SN_NIR --save align.json
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Red_660 --channel nir=NIR_850 \
  --save-multiband -o output/
```

`align-calibrate` usa uma matriz conectada que contém suas câmeras ou as abre por conta própria para uma única execução. O `index --live` sempre abre as câmeras por conta própria (um [array transitório](../reference/cli-reference.md#multi-camera-transient-sessions)) e não possui o `--serials`: ele abre todas as câmeras da rede. Portanto, libere todas as câmeras que o Chloros mantém antes disso — a GUI, o `cam-connect`, o `array-connect` (`chloros-cli lattice array-disconnect --all`): abrir uma câmera retida pela segunda vez pode interromper o fluxo de seu detentor ou causar uma falha no backend. Uma câmera que não forneça nenhum quadro faz com que a execução falhe, e nenhuma pilha ou índice é gravado.

`--channel` mapeia os símbolos de uma predefinição para os nomes de bandas da pilha. Duas regras evitam que a execução falhe:

* **Os símbolos diferenciam maiúsculas de minúsculas** e devem corresponder exatamente aos nomes dos canais da predefinição — as predefinições usam letras minúsculas (as do NDVI são `red`,`nir`; verifique `--list-presets`). `--channel red=Red_660` funciona; `--channel RED=660` falha com um erro `channel_map missing entries`.
* O lado da banda deve indicar o nome de uma banda na pilha alinhada (`lattice align-info --profile align.json` as lista). O modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`.

O `lattice index` também é executado offline em um arquivo TIFF multibanda alinhado salvo. Cada formulário é executado no backend; portanto, ele deve estar em execução e conectado a um plano pago do Chloros+; um `--input` ou `--output` relativo é resolvido com base na pasta a partir da qual você executa o comando:

```bash
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --output ndvi.tif --colorize --gradient RdYlGn
```

### Predefinições de índice

O `lattice index --preset` (e a opção [Index/LUT sandbox](../image-viewer-gui/index-lut-sandbox.md) da aba Imagem, que utiliza o mesmo mecanismo) inclui estas **22 predefinições**:

`NDVI, GNDVI, BNDVI, NDRE, ENDVI, SAVI, OSAVI, MSAVI, EVI, EVI2, CVI, MSR, TDVI, LAI, GLI, NGRDI, VARI, TGI, EXG, CIRE, CIGREEN, NDWI`

Execute o comando `chloros-cli lattice index --list-presets` para obter a fórmula e os símbolos de canal de cada predefinição, e o comando `--list-gradients` para ver os gradientes de cor disponíveis. Fórmulas personalizadas utilizam o comando `--formula EXPR` com a mesma sintaxe da Calculadora de Índice. Observe que esta lista de predefinições é específica do mecanismo de índices LATTICE — o menu suspenso “Processamento” nas Configurações do Projeto para imagens importadas apresenta uma lista diferente (consulte [Fórmulas de Índices Multiespectrais](../project-settings/multispectral-index-formulas.md)).

O conjunto completo de sinalizadores (`--output-format`, `--vmin/--vmax/--percentile`, `--bg-mode`, os controles deslizantes de alinhamento e distorção para `--live` e outros) está documentado na [Referência da CLI § Matemática de Índices/Vegetação](../reference/cli-reference.md#index--vegetation-maths); os equivalentes do SDK estão na [Referência do SDK](../reference/sdk-reference.md).

## Captura de produtos de índice a partir de uma matriz mono

Com uma matriz conectada e uma expressão de índice aplicada, o `array-capture` (ou a opção da GUI **Capturar Tudo**) salva os níveis de exportação por câmera *e* a renderização do índice — `--index`/`--no-index` alterna essa opção na CLI e, por padrão, captura todos os níveis aplicáveis. A contribuição de uma câmera mono para cada grupo de captura é sua única banda nos níveis bruto/despixelação (escala de cinza)/radiação/refletância, além da composição de índice combinado compartilhada quando a matriz opera no modo combinado. Consulte [Matrizes com várias câmeras § Captura](arrays.md#capturing-monitoring-vs-analysis).
