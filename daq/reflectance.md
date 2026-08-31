# Fluxos de trabalho de refletância

Um sensor de luz DAQ converte imagens radiométricas em refletância. Existem dois fluxos de trabalho distintos:

1. **Sensor único** — um DAQ mede a irradiação descendente enquanto uma câmera captura a imagem, e o Chloros divide a radiancia da câmera por essa referência.
2. **Sensor duplo** — dois sensores DAQ, um monitorando o céu e outro monitorando um objeto, produzem uma curva de refletância espectral em tempo real sem o uso de câmera.

## Sensor único + câmera (referência descendente)

O DAQ atua como o sensor de luz descendente (DLS): a câmera mede a radianância ascendente **L**(W/m²/sr/nm), o DAQ mede a irradiância descendente**E** (W/m²/nm) e o Chloros calcula a refletância por banda da seguinte forma:

> ρ = π · L / E

A leitura do DAQ está sempre **sincronizada com a exposição por meio do carimbo de data/hora** — é por isso que o DAQ e as câmeras compartilham um relógio sincronizado por PTP (consulte [Rede e Sincronização de Tempo do DAQ-E](ethernet-ptp.md)). Ajuste o cofre cosseno Sunshine para trabalhos ao ar livre e declare-o corretamente; a declaração do cofre dimensiona diretamente E (consulte [Perfis de cofre e faixa calibrada](caps-and-range.md)). Para trabalhos quantitativos, lembre-se da característica do instrumento: a irradiância quantitativa é obtida a partir de uma média de pelo menos 15 s de leituras.

### Captura ao vivo

Vincule o DAQ a uma câmera na aba “Câmeras”: o painel de configurações de cada câmera possui um menu suspenso **Sensor de Luz**que lista todos os DAQs conectados (DAQ-U/M/E) da aba “Sensores de Luz”; para um conjunto sincronizado, a seleção do**Sensor de Luz** para todo o conjunto se propaga a todos os membros (câmeras individuais ainda podem substituir essa configuração). Uma vez vinculado, os espectros do sensor alimentam o slot DLS da câmera e as exportações de refletância são divididas pela leitura correspondente.

<!-- SCREENSHOT-NEEDED: Cameras tab per-camera settings panel showing the "Light Sensor" dropdown open, with a connected DAQ sensor listed and selected. -->

Dois comportamentos que vale a pena conhecer:

* **Sem DAQ vinculado → a refletância é rejeitada, não simulada.** O Chloros rejeita o produto de refletância e registra o motivo da omissão, em vez de devolver silenciosamente um produto inferior.
* **A leitura utilizada é preservada.** Para cada quadro de refletância, a leitura do DAQ efetivamente aplicada é gravada como um sidecar `.daq` ao lado das imagens, para que a captura possa ser reprocessada posteriormente ([Gravação e o formato .daq](recording.md)).

### Processamento de imagens gravadas

Para o processamento pós-voo, grave um `.daq` durante a sessão e mantenha-o junto com as imagens — o pipeline resolve automaticamente a luz descendente com carimbo de data/hora correspondente, buscando qualquer calibração de fábrica ausente na nuvem do MAPIR na primeira utilização. As gravações da GUI são adicionadas automaticamente ao projeto aberto assim que são encerradas.

A referência de refletância pode ser selecionada no momento do processamento — `--reflectance-source` no `chloros-cli process`, ou a configuração da fonte de refletância nas Configurações do Projeto da GUI:

| Valor | Comportamento |
| --- | --- |
| `auto` (padrão) | Um alvo de calibração dentro do quadro aprovado no controle de qualidade (QA) é a referência absoluta; a radiação descendente do DAQ (ρ = π·L/E) é a opção alternativa |
| `daq` | Autoridade do DAQ |
| `target` | Alvo estritamente dentro do quadro; sem substituição pelo DAQ |

Consulte [Alvos de calibração](../calibration-targets.md) para fluxos de trabalho de alvos e o [capítulo LATTICE](../lattice/README.md), além da [Referência CLI](../reference/cli-reference.md) para o pipeline de processamento completo. Ao ler os pixels de refletância exportados, use a escala indicada (LATTICE: 32768 = ρ 1,0, XMP `Chloros:PixelScale`; Survey3: 65535) — consulte [Formatos de imagem de saída](../output-image-formats.md).

### Bandas fora da faixa calibrada do DAQ

A faixa radiometricamente calibrada do DAQ é de ~374–974 nm. O Chloros rejeita a refletância baseada no DAQ para qualquer banda da câmera com menos da metade de seu peso espectral dentro desse intervalo, relatando o motivo de omissão `dls-uncalibrated-band-<nm>`. Entre os SKUs disponíveis, isso afeta apenas a F988: a refletância da F988 é calibrada usando um painel de refletância na cena; como a banda está fora da faixa calibrada do sensor de luz do DAQ, o Chloros aplica sua captura mais recente do painel e a mantém entre as observações do painel. Se uma câmera F988 for operada apenas com o DAQ, o código Chloros rejeita a refletância baseada no DAQ para essa banda com o motivo de omissão `dls-uncalibrated-band-988` — o fluxo de trabalho com o painel é o caminho recomendado.

## Sensor duplo (ambiente + objeto)

Dois sensores de DAQ — qualquer par, em qualquer meio de transporte — fornecem um espectro de refletância em tempo real sem câmera: um sensor está voltado para o céu (**Fonte de Luz Ambiente**), outro para o objeto (**Scanner de Objeto**), e o Chloros calcula por comprimento de onda:

> R(λ) = objeto(λ) / ambiente(λ)

(zero quando ambiente ≤ 0).

### Na interface gráfica (GUI)

Com ambos os sensores conectados na aba “Sensores de luz”, abra a sobreposição para adicionar sensores (o botão “+” em um bloco de gráfico na visualização em grade) e escolha **Combinar ambiente + objeto**. Selecione os dois sensores nos menus suspensos “Fonte de Luz Ambiente” e “Scanner de Objetos” e clique em “Criar”. O grupo aparece como um gráfico próprio e como uma linha na barra lateral com um ícone verde**REF**.

<!-- SCREENSHOT-NEEDED: The add-sensor overlay's "Combine Ambient + Object" panel with two connected DAQ sensors selected in the Ambient Light Source and Object Scanner dropdowns, Create button enabled. -->

<!-- SCREENSHOT-NEEDED: A live Apparent Reflectance chart from an Ambient+Object DAQ pair in list view, with the vegetation-index table visible below the chart (NDVI etc. showing live values). -->

Abaixo do gráfico de refletância (visualização em lista), uma **tabela de índices de vegetação** em tempo real calcula os índices a partir da curva usando os centros de banda em azul 450 / verde 550 / vermelho 670 / NIR 800 nm. Índices baseados em razões que anulam a escala absoluta (NDVI, GNDVI, ENDVI, WDRVI, GRVI, CVI, GCI, MSR) são sempre exibidos; índices que exigem refletância absoluta (EVI, SAVI, OSAVI, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) aparecem apenas quando ambos os sensores são modelos com calibração de potência.

### Aparente vs. Relativo — a regra de rotulagem

O Chloros rotula a saída do par de sensores de acordo com o que o par de sensores pode realmente indicar:

| Par de sensores | Rotulagem |
| --- | --- |
| Ambos os sensores calibrados — pacote de fábrica carregado | **Refletância aparente** |
| Qualquer um dos sensores não calibrado | **Refletância Relativa** |

Todos os três modelos são radiométricos: uma vez carregado o pacote de calibração de fábrica de um sensor, seus espectros são absolutos em W/m²/nm; portanto, um par de sensores calibrados resulta em uma refletância aparente absoluta — o transporte não determina isso. Um sensor que ainda esteja transmitindo contagens brutas (sem pacote acessível) reclassifica o resultado para uma curva relativa (a forma espectral continua válida). Ambos os sensores devem possuir limites declarados corretamente ([Perfis de Limite e Faixa Calibrada](caps-and-range.md)).

### De Python

Não há uma chamada dedicada para dois sensores na superfície agrupada SDK: abra duas sessões com o `chloros_sdk.connect_daq_sensor()` e compare você mesmo seus espectros `latest()`, aplicando a mesma convenção de nomenclatura. (Também existe uma ferramenta de gravação com dois sensores na superfície interna de hardware direto do MAPIR, listada na [Referência do CLI](../reference/cli-reference.md) para fins de completude — ela não faz parte do CLI fornecido; o fluxo de trabalho da GUI acima é o caminho ativo suportado.)
