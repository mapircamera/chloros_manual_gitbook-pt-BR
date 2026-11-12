# Matriz de câmeras

Uma **matriz**LATTICE consiste em duas ou mais câmeras LATTICE conectadas como uma única unidade sincronizada. Uma câmera é a**mestre**: ela emite um pulso de acionamento GPIO de hardware em uma linha de sincronização compartilhada (padrão**Linha 2**), de modo que todos os membros capturam o mesmo instante. O Chloros adiciona sincronização de tempo PTP, uma pré-visualização ao vivo (mosaicos por câmera ou uma única imagem composta multibanda alinhada) e captura sincronizada — cada passagem de captura produz um **grupo de quadros** no qual todas as câmeras compartilham o mesmo carimbo de data/hora e ID de quadro (relatado como `fid:N` na saída de captura).

As matrizes são a forma como as câmeras monocromáticas (M3M) produzem índices de vegetação — uma câmera contribui com uma banda, e a matriz as alinha em uma pilha multibanda. Consulte [Câmeras Mono e Índices de Vegetação](mono-indices.md).

Existem três maneiras equivalentes de conectar uma matriz, e todas elas executam o mesmo fluxo “smart-prep”:

| Superfície | Ponto de entrada |
| --- | --- |
| GUI | Guia Câmeras → **Conectar Matriz** (botão azul) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primeiro número de série = mestre) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primeiro número de série = mestre) |

O smart-prep realiza: um teste de capacidade de rede (ping ICMP DF + teste GVSP), seleção da camada de sincronização, uma projeção da taxa de quadros no tamanho de quadro solicitado, a decisão de transporte compactado de 12 bits (consulte [Transporte compactado de 12 bits](#packed-12-bit-transport)), ativação do PTP, seleção automática do formato de pixel por câmera, inicialização da exposição automática a partir do estado salvo de cada câmera e configuração do gatilho GPIO na Linha 2.

**Sensor completo por padrão (novidade na versão 1.3.0).** O Connect mantém o tamanho de quadro solicitado — o sensor completo de 2048×1536, sem binning nem recorte, a menos que você escolha o contrário — e permite que a taxa de quadros caia para o que o host e a rede fornecem. Ele nunca realiza binning ou recorte por conta própria (a versão 1.2.0 ativava o binning automaticamente abaixo de 1,5 fps). O binning ou uma ROI menor ficam a seu critério: na caixa de diálogo “Conectar”, escolha uma opção e o “Resultado Projetado” mostra a taxa de quadros resultante; o comando ``array-connect`` exibe as opções mais rápidas com suas taxas projetadas.

{% hint style="info" %}
As câmeras devem estar acessíveis no link antes que qualquer uma dessas funções funcione — consulte [Conectando câmeras](connecting.md) para descoberta, endereçamento e download da calibração na primeira conexão. Para configurações com várias câmeras, as configurações do anel de recepção da placa de rede do host são tão importantes quanto a velocidade do link; a tabela completa de sintomas → soluções está em [Referência da CLI § Configuração e Ajuste da Placa de Rede do Host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## A caixa de diálogo “Conectar Matriz”

A guia “Câmeras” → **Conectar Matriz**abre um assistente de três etapas:**Selecionar → Modo de exibição → Configurações**.

### Etapa 1 — Selecionar mestre e escravos

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered (an FRGB among them). Table showing Camera / Serial / IP / Master radio / Slave checkbox columns in camera order (M3C first, FRGB at the top), with the FRGB pre-selected as master and the green "Sync cable found on N cameras. Any of them can be master; the RGB camera (else the first) is pre-selected." probe banner visible above the table. -->



A caixa de diálogo varre a rede assim que é aberta (“Varredura da rede...”) e, em seguida, verifica os cabos de sincronização (“Verificando cabos de sincronização...”) para identificar quais câmeras estão na linha de sincronização M8. São necessárias pelo menos **2 câmeras** para formar um array.

Os cabos de sincronização M8 se encontram em um ponto de junção, portanto, **qualquer câmera conectada pode ser a mestre**. O teste preenche previamente as funções — a câmera RGB (`-FRGB`) como mestre quando estiver conectada; caso contrário, a primeira câmera conectada; e as demais câmeras conectadas como escravas — e exibe uma das três mensagens:

| Mensagem | Significado |
| --- | --- |
| “Cabo de sincronização encontrado em {n} câmeras. Qualquer uma delas pode ser mestre; a câmera RGB (ou a primeira, caso contrário) está pré-selecionada.” (verde) | O probe encontrou as câmeras na linha de sincronização; as caixas de seleção “mestre” e “escravo” já estão marcadas. |
| “Nenhum mestre detectado — verifique o cabo GPIO” (laranja) | Nenhuma câmera detectou um pulso de acionamento; verifique o cabeamento de sincronização. As funções ainda estão pré-definidas (a câmera RGB, ou a primeira câmera da lista, como mestre) e você pode alterá-las. |
| “Sem cabo de sincronização: {seriais}” (laranja) | As câmeras listadas não têm nenhum cabo de sincronização conectado. |

A tabela de câmeras possui as colunas **Câmera / Serial / IP / Mestre (rádio) / Escravo (caixa de seleção)**. As linhas são listadas na**ordem das câmeras** — câmeras M3C (Bayer) antes das M3M (mono); entre as M3C, FRGB, FRGN, FOCN e, em seguida, FNGB; as M3M por comprimento de onda do filtro, do menor para o maior (F450 antes de F808). A lista de câmeras na barra lateral e o layout padrão da grade usam a mesma ordem.

* Escolha exatamente **um mestre**e**um ou mais escravos**. Clicar novamente no botão de opção do mestre atual o desmarca. Escolher um mestre diferente é feito com um clique: o novo mestre deixa os escravos e o mestre anterior passa a ser um escravo (a menos que não tenha cabo de sincronização).
* Uma câmera marcada como **“Sem cabo de sincronização”** nunca pode ser selecionada como escrava — uma escrava sem fiação de acionamento ficaria esperando pela linha de sincronização para sempre e transmitiria um sinal inativo. Conecte essa câmera como uma câmera autônoma, em vez disso.
* Câmeras já conectadas de forma autônoma *não* são desativadas: a conexão em matriz libera a sessão autônoma e reabre a câmera dentro da matriz.

**Próximo: Modo de Exibição →**fica habilitado assim que um mestre e pelo menos um escravo forem escolhidos.**Reexaminar** executa novamente a descoberta e a sondagem de fiação.

{% hint style="warning" %}
**Cancelar** fica desativado enquanto uma varredura ou sondagem estiver em andamento — cancelar no meio da sondagem pode causar falha no SDK da câmera no firmware da câmera LATTICE. Aguarde até que o indicador de carregamento termine.
{% endhint %}

### Etapa 2 — Modo de exibição



<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

| Modo | O que você obtém |
| --- | --- |
| **Câmeras separadas** | Um bloco ao vivo por câmera, todos acionados juntos para que os quadros permaneçam sincronizados. Cada câmera mantém sua própria cor e configurações. |
| **Câmeras combinadas** *(padrão)* | Um único bloco exibindo a composição multibanda alinhada de NDVI/índice. As câmeras compartilham a cor do conjunto. |

O modo de exibição altera apenas a apresentação da pré-visualização ao vivo — o comportamento de captura é o mesmo em ambos.

### Etapa 3 — Configurações do conjunto e resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Trigger Rate / Wire Budget controls (1.3.0 has no "Pin resolution" checkbox), right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

projetado

Ao entrar nessa cena, o Chloros solicita ao backend uma **recomendação**e a aplica: o**sensor completo — sem binning, sem recorte de ROI — sempre que qualquer taxa de quadros permitir**, independentemente da taxa que o host e a conexão forneçam. Somente quando nenhuma taxa puder suportar o sensor completo é que a recomendação reduz o quadro, e, nesse caso, prefere o binning ao recorte de ROI, pois o binning preserva o campo de visão completo. A partir daí, você troca resolução por taxa de quadros. Cada alteração que você faz reexecuta a análise em tempo real e atualiza o painel**Resultado Projetado** à direita.

Coluna esquerda — configurações:

| Controle | Opções | Padrão | Observações |
| --- | --- | --- | --- |
| **ROI (Campo de Visão)** | Completo (2048×1536) / Metade (1024×768) / Um quarto (512×384) | Completo | Recorte do sensor: recorte de metade ou um quarto para uma região menor com o passo de pixel nativo. |
| **Binning** | 1× / 2× (soma 2×2) / 4× (soma 4×4) | 1× | Binning de hardware: 2×2 = campo de visão completo com um quarto do custo de transmissão; 4×4 = campo de visão completo com 1/16. Ocultado se as câmeras não suportarem binning. |
| **Imagem no lado do cabo** (leitura) | — | — | A largura × altura pós-binning efetivamente enviada pelo cabo, redimensionada para múltiplos de 16 (mínimo 64). |
| **Taxa de disparo** | 0,5–60 fps, passo de 0,1 | vazio = automático | A taxa de disparo do mestre. Deixe em branco para que o Chloros a calcule. |
| **Capacidade da conexão**| 20–2.000 MB/s, em incrementos de 10 | vazio = automático | Quanto o host pode realmente absorver, em MB/s —**o único número do qual depende toda a alocação da matriz.** Detectado automaticamente a partir do adaptador de rede. Reduza-o se a matriz relatar quadros corrompidos: o valor detectado superestima os adaptadores USB e os switches compartilhados. Alterá-lo reexecuta a projeção em tempo real. O campo abre com o orçamento com o qual a última matriz foi conectada (salvo com o projeto), portanto, um orçamento que você corrigiu uma vez é mantido. |

A caixa de seleção **Resolução de pinos** da versão 1.2.0 foi removida: a conexão mantém sempre o tamanho de quadro escolhido e, em vez disso, reduz a taxa de quadros.

Coluna direita — **Resultado projetado**:

* **Nível de sincronização** — “Captura simultânea” (verde), “Captura simultânea (emissão escalonada FTD)” (verde), “Captura escalonada (desvio de 100 ms)” (âmbar) ou “Configuração muito grande” (vermelho). O Chloros escolhe o nível: captura simultânea sempre que o cabo puder ser compartilhado a uma taxa que as câmeras possam acompanhar (a taxa de quadros diminui à medida que a parcela de cada câmera diminui) e captura escalonada apenas quando isso não for possível — assim, um conjunto de câmeras se conecta a qualquer rede, na taxa de quadros que a rede permitir.
* **Projeção de fps** — a taxa de quadros na qual se espera que o conjunto funcione. Com a captura simultânea, trata-se de um único número: a taxa vem primeiro, e a exposição automática preenche o período que ela determina, depois aumenta o ganho, de modo que uma cena escura não a reduza (consulte [Como a exposição automática distribui exposição e ganho](camera-settings.md)). Com a captura escalonada, ela é mostrada como um intervalo (“escuro → claro”), pois a exposição de cada câmera prolonga o escalonamento em uma cena escura.
* **Linha da NIC** — velocidade do link e taxa de transferência sustentada (“NIC {mbps} Mbps · sustentada {N} MB/s”).
* **Verificação de rajada de emissão simulada** — a NIC do anel de recepção do host consegue receber uma rajada simultânea de todas as câmeras (“Rajada de emissão simulada: X MB · Anel de NIC utilizável: Y MB ✓/✗”).
* **Verificação do orçamento de banda** — demanda agregada em estado estacionário versus o limite máximo de banda seguro contra colisões (“Orçamento de banda: {demanda} MB/s exigidos por {n} câmeras · limite máximo {limite} MB/s ✓/✗ sobrecarregado”).
* **“Número máximo de câmeras nesta linha: {n} — definido pelo limite mínimo de largura de banda por câmera, portanto o agrupamento não aumenta esse número.”** — exibido quando você está próximo (ou acima) do limite máximo de câmeras.
* **“QUADROS SERÃO PERDIDOS com essas configurações."**— aviso em vermelho com o motivo do backend, além de uma lista de bloqueadores e**sugestões de correção** em azul (“Para acomodar esta matriz na rede” / “Para habilitar a captura simultânea”).**Aplicar e Conectar** fica bloqueado até que exista uma projeção, e seu rótulo indica por que está sendo recusado:

| Rótulo do botão | Significado | O que realmente ajuda |
| --- | --- | --- |
| “Analisando...” | Análise ainda em andamento. | Aguarde. |
| **“Câmeras demais para esta rede"**| A conexão não consegue suportar o conjunto nem mesmo uma câmera por vez (falha na verificação agregada): há muito mais câmeras do que ela consegue processar, ou outras câmeras transmitindo pela mesma conexão. | Menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.**Uma área de interesse (ROI) menor NÃO ajudará** — veja abaixo. |
| **“Reduzir a área de interesse (ROI) para habilitar”** | Os quadros seriam perdidos com essas configurações (falha na verificação de burst/ring). | Reduza a área de interesse (ROI), aumente o binning ou conserte o anel de recepção da placa de rede. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->



Durante a conexão, um **painel de download de calibração** verde pode aparecer com uma barra de progresso por porta serial: na primeira vez que uma câmera é conectada a uma máquina, o Chloros baixa seu pacote de calibração de fábrica de ~3,8 MB da câmera via GigE (aproximadamente 70 segundos por câmera). Câmeras armazenadas em cache nunca exibem esse painel; com mais de seis câmeras, suas linhas rolam. Consulte [Conectando câmeras](connecting.md).

### Transporte compactado de 12 bits

Os formatos de 12 bits (`BayerRG12`, `Mono12`) normalmente são transmitidos como palavras de 16 bits. **O transporte**compactado**envia os mesmos 12 bits em 1,5 bytes por pixel e os descompacta no host, de modo que cada quadro leva 25% menos tempo para ser enviado — com o sensor operando em capacidade total, aproximadamente um terço a mais de quadros por segundo.**Os dados salvos são idênticos.** A conexão em matriz decide isso automaticamente (`auto`): ela compacta apenas quando isso é vantajoso e o host dispõe de CPU disponível para a descompactação. A GUI não possui opção para isso; na CLI, use `array-connect --packed {auto,on,off}` (padrão: `auto`). Os formatos de 8 e 16 bits nunca são compactados.

## Largura de banda: quantas câmeras cabem

A capacidade de um array depende da largura de banda da conexão, não do Chloros; portanto, os números de planejamento constam no manual do hardware: **[Planejamento de largura de banda do array](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

O que o Chloros faz com esses dados: a caixa de diálogo de conexão executa um teste de rede, projeta a taxa de quadros alcançável e seleciona um nível adequado. Uma matriz se conecta a qualquer rede: em um link lento, cada câmera é ajustada à sua parcela da largura de banda e a taxa de quadros diminui, em vez de a matriz ser recusada ou seus quadros serem reduzidos. Somente um array que a linha não consiga suportar nem mesmo uma câmera por vez é recusado (**Muitas câmeras para esta rede**), em vez de descartar pacotes silenciosamente — consulte o painel de resultados projetados descrito acima.

## Quando os quadros desaparecem

Uma câmera pode estar ausente de um grupo publicado por dois motivos completamente diferentes,
e eles exigem soluções opostas. O Chloros os contabiliza separadamente, em vez de relatar um
número “incompleto” que não especifica nenhum dos dois:

| O que aconteceu | O que isso significa | Onde procurar |
| --- | --- | --- |
| **Corrompido**— o quadro chegou e estava estruturalmente corrompido | Perda de pacotes do GVSP no caminho da rede | O**orçamento de banda**, o anel de recepção da placa de rede, quadros jumbo, o switch |
| **Nunca chegou**— nenhum quadro chegou | A câmera não disparou, ou nada saiu dela | O**cabo de sincronização M8**, a linha de sincronização, se todos os membros estão armados |

A divisão é reavaliada a cada 10 segundos enquanto o conjunto de câmeras transmite. Acima de 5 %, ela é
registrada com os dois números indicados, e cada buffer corrompido é relatado na primeira vez em que
ocorre por câmera; em seguida, os dados são agrupados uma vez por minuto para que uma sessão longa permaneça legível.

**Quadros corrompidos com zero “nunca chegou” significam que o acionamento e a sincronização do cabo estão perfeitos**e que todos os quadros perdidos estão no caminho da rede. A solução é diminuir o**Wire Budget** e
reconectar.

{% hint style="warning" %}
**Reduzir a taxa de acionamento não ajuda com quadros corrompidos.** O ritmo de pacotes
da câmera é definido uma vez, na conexão. Reduzir a taxa de acionamento altera a frequência com que uma rajada
ocorre, não a velocidade com que a própria rajada chega ao cabo. Em um sistema medido com 4 câmeras, uma
redução de 5× na taxa de disparo não alterou nada, enquanto diminuir o orçamento de banda de 240 para
200 MB/s reduziu a taxa de quadros corrompidos daquele mesmo sistema de 10,4% para zero.
{% endhint %}

Um array em operação não pode se replanejar — desconecte e reconecte para que o seletor de tempo de conexão
possa operar com base no novo orçamento.

### Adaptadores de rede USB têm limite

Um adaptador Ethernet USB anuncia sua taxa de link *Ethernet*, mas o que ele pode realmente
manter é limitado pelo barramento USB e seu driver. Um dongle USB 10GbE costumava ser creditado
com aproximadamente 1.000 MB/s de taxa de transferência — um valor que nunca havia sido medido — e ajustar
quatro câmeras com base nesse limite imaginário corrompeu de 6 a 18 % dos quadros, enquanto o array
ainda relatava uma taxa de quadros alvo satisfatória. Os adaptadores conectados via USB agora têm um limite de
**200 MB/s** no Windows e no NVIDIA Jetson, e **400 MB/s** no Linux x86_64. O limite é um valor absoluto, e não uma porcentagem, pois o restritor é o
barramento: um adaptador USB de 1 GbE atinge cerca de 80 MB/s e não é afetado.

Se o seu host for realmente mais rápido do que o limite, aumente o **Wire Budget** para refletir isso.

## Sincronização de tempo PTP

A *sincronização* de quadros vem do gatilho de hardware; o **PTP** (IEEE 1588 PTPv2) fornece *carimbos de data/hora* comparáveis em todos os dispositivos. Ele é ativado por padrão na conexão do array:

* O **backend do host Chloros executa o grandmaster PTP**. As câmeras LATTICE e os sensores de luz DAQ-E funcionam como escravos dele no domínio 0, de modo que os carimbos de data/hora das imagens e os espectros do DAQ ficam em um único relógio (~1 ms).
* `--no-ptp` (CLI) desativa essa função para trabalhos em bancada — os carimbos de data/hora entre câmeras, então, **não** são comparáveis.
* Verifique o estado da sincronização pela CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity, health (OK, or why its packets are not reaching the network)
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

A guia “Câmeras” em si não possui indicador de PTP; as informações de sincronização por câmera exibidas ali são: **Função**(Mestre/Escravo) — somente leitura,**Linha de Sincronização**e o nível de**Capacidades** do conjunto. O estado PTP do DAQ-E é exibido nos detalhes do sensor na guia “Sensores de Luz”.

## A visualização

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows in camera order (M3C before M3M, monos by wavelength), and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

ao vivo da matriz A área principal de exibição oferece dois layouts (alterne na barra superior): **visualização em grade**(cada bloco é uma célula; arraste para reordenar quando o cadeado da grade estiver destravado) e**visualização em lista**(linhas de largura total: uma matriz**combinada**ocupa uma linha — seu bloco composto —, enquanto uma matriz**separada**dá a**cada membro sua própria linha**, com seu próprio feed ao vivo e sua própria tabela de Alvo de Calibração; abaixo das matrizes, uma câmera independente — a ativa). O controle deslizante**Zoom do Feed** ajusta o tamanho dos blocos; abaixo de 200 px de largura de célula, as sobreposições de nome/fps se ocultam automaticamente. As câmeras membros aparecem na barra lateral e no layout padrão da grade na ordem das câmeras (M3C antes de M3M, FRGB primeiro, monocromáticas por comprimento de onda); desbloquear a grade e arrastar ainda assim substitui a ordem da grade.

<!-- SCREENSHOT-NEEDED: list view of a connected SEPARATE array (1.3.0): one full-width row per member camera, each with its live feed and its own Calibration Target table below it. -->



O **modo separado** mostra um bloco por câmera. Cada bloco exibe:

* o nome da câmera (canto superior esquerdo),
* uma **leitura de fps** (canto superior direito) — essa é a *taxa real de captura* da câmera informada pelo backend, não a taxa de atualização da pré-visualização (a pré-visualização ao vivo é limitada a 30 fps, independentemente da taxa de captura),
* um ponto de status — verde (transmissão) / âmbar (carregando) / vermelho (erro),
* um **indicador de quadro desatualizado** quando nenhum quadro novo chega por 2 s — normal por cerca de 5 s após qualquer conexão/desconexão, enquanto o backend reequilibra a largura de banda entre as câmeras.

O **modo combinado**exibe um único bloco composto: o backend realiza o debayering, redimensiona, alinha, remove ruídos, converte para radiação por banda (mais refletância DLS quando um sensor de luz está vinculado), avalia a expressão de índice da matriz, aplica a LUT e transmite o resultado como MJPEG. Até que o primeiro quadro alinhado seja renderizado, o bloco indica seu estado: “Preparando matriz…”, “Calibrando o alinhamento…”, “Aguardando o primeiro quadro…”, ou — se as tentativas de alinhamento automático do bloco (cerca de um minuto) se esgotarem — “Alinhamento necessário” com um botão**Calibrar alinhamento**.

A exibição de fps do bloco da matriz também informa o estado da transmissão:**“Aplicando configurações…”**enquanto uma alteração nas configurações (formato, binning, ROI) reinicia as câmeras,**“Parado”**quando a matriz está ativa, mas parou de enviar grupos de quadros, e**“Parado"**quando ele interrompeu a transmissão — nesse caso, reconecte o conjunto. Quando a separação espectral precisou limitar uma banda exibida na imagem composta em uma grande parcela de seus pixels (25 % ou mais), o bloco exibe**“Dados da banda com baixa confiança · N%"**: os valores de índice calculados com base nessa banda são suspeitos (geralmente, uma banda recebeu pouca luz).

Informações úteis sobre o modo combinado:

* A imagem composta é alinhada ao quadro da câmera **mestre**. O direcionamento AE-ROI e a medição pontual na imagem composta são exatos para a câmera mestre e aproximados para as escravas; use**Visão Dividida** (configurações do conjunto → “Mostrar câmeras membros”) para blocos por câmera com precisão de pixel sem abrir conexões extras de câmera.
* **Camadas de exibição** (configurações da matriz; desativado por padrão) permite escolher uma camada de primeiro plano e uma de fundo — qualquer câmera membro ou**Índice**. Com primeiro plano = Índice, os pixels fora do Min/Max da LUT exibem a camada de fundo.
* **Resolução de renderização** (padrão 720p) define a altura da transmissão ao vivo *e* o tamanho da exportação da imagem composta salva. As imagens por câmera são sempre exportadas em resolução total.
* O alinhamento é calculado por sessão e nunca é salvo. O Chloros alinha **todas as matrizes automaticamente após a conexão, em ambos os modos de exibição**, assim que a exposição automática dos membros se estabilizar (ele continua tentando por até 5 minutos). A seção**Alinhamento**do painel de configurações do conjunto mostra os resíduos RMS e o botão**Recalibrar alinhamento** — também para conjuntos separados, que não possuem bloco composto.

## Captura: monitoramento x análise

As superfícies de captura do conjunto se dividem claramente em **nível de monitoramento**(grava o que você vê) e**nível de análise** (grava dados brutos, calibra posteriormente):

| Fluxo de trabalho | Nível | O que é salvo | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Captura**(imagens estáticas) | Análise | Um grupo de quadros sincronizados por passagem; arquivos por câmera em todos os níveis de exportação selecionados (bruto/sem debayering/radiação/refletância/visualização/índice) + arquivo sidecar `.daq` |**Botão **Capturar tudo** + Configurações de captura | `lattice array-capture` |
| **Gravar vídeo de índice** | Monitoramento | A composição do índice combinado ao vivo conforme exibida — 8 bits, resolução de pré-visualização, LUT incorporada. Na CLI, sem nenhuma janela da GUI aberta, o backend renderiza o NDVI da matriz (ou `--expr`) assim que a matriz estiver alinhada; uma gravação que não grava nenhum quadro falha | ● Gravar vídeo de índice (matrizes combinadas) | `lattice array-record` |
| **Sequência bruta → criar vídeo**| Análise | Quadros brutos do sensor na taxa máxima de captura + manifesto + `.daq`, seguida de reconstrução offline em vídeo calibrado de radiação / refletância / índice, sincronizado com as leituras do DAQ | ⦿ Gravar sequência bruta →**Criar vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regra geral: se os pixels forem fornecer *medições*, use a captura ou o burst (nível de análise); se você precisar apenas *assistir ou demonstrar* o que o array captou, grave o vídeo indexado (nível de monitoramento).

### Configurações de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->



A engrenagem ao lado de **Capturar tudo** abre o painel de configurações de captura (requer um projeto aberto — as capturas são salvas nele):

* **Modo de captura**:**Único**(uma passagem) /**Contínuo**(consecutivo; limitado por um número de capturas, padrão 1, ou por uma duração, padrão 10 s) /**Intervalo** (timelapse: N capturas a cada intervalo de X para um total de Y; padrão: 1 a cada 5 s por 1 minuto).
* **Tipos de exportação por câmera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — todas as opções aplicáveis estão ativadas por padrão. Radiance/Reflectance ficam ocultas para câmeras com filtro RGB; **Refletância aparece apenas quando a câmera possui um sensor de luz DAQ** (próprio ou herdado da matriz); Índice requer uma expressão de índice configurada.
* **Alinhado**(por matriz, padrão**ativado**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz, de modo que as exportações sejam registradas por pixel. O formato Raw permanece sempre sem distorção, mas carrega a transformação nos metadados. Os arquivos dos membros da matriz são salvos na orientação nativa de cada câmera (qualquer configuração de Espelhamento/Rotação é registrada no arquivo, mas não aplicada).
* **Captura mais rápida** (alternar): somente raw + a leitura DAQ atribuída + a composição de índice combinado livre, pulando os cálculos de calibração no momento da captura para obter a taxa máxima — reconstrua a radiação/refletância/índice posteriormente a partir do `.daq` salvo.
* As seleções permanecem com o projeto. Câmeras ocultas ou pausadas são ignoradas.
* **Contínuo** em matrizes é executado no backend do Chloros (novidade na versão 1.3.0): quando todas as câmeras da lista pertencem a uma matriz, o backend executa ele mesmo o ciclo de captura, gravando um grupo enquanto seleciona o próximo, sem necessidade de solicitação por captura. Um disco cheio interrompe a execução (“Interrompido: disco cheio. Salvo N, F falhou”). Consulte [Configurações e modos de captura](capture.md#capture-modes) para ver a nota do Microsoft Defender sobre altas taxas de captura.

A CLI equivalente (mesmo endpoint do backend, mesma semântica):

```bash
# One synced group, every applicable export level per camera (the default)
chloros-cli lattice array-capture -o output/

# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# 30-second monitoring clip of the combined index view, plus a GIF
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# 5-second analysis-grade raw burst, then build the combined index video
chloros-cli lattice array-burst --duration 5 --build --products combined:index --fps 10 -o capture/
```

A compressão TIFF para capturas é `deflate` (sem perdas, padrão) ou `none` — as tabelas completas de sinalizadores, o layout da pasta de captura e as regras de reprocessamento estão na [Referência da CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparelhamento de um sensor de luz DAQ

As visualizações prévias com correção de refletância e iluminação precisam de dados de luz descendente de um sensor DAQ (conectado na guia **Sensores de Luz**):

* A **linha da matriz**na barra lateral exibe um**botão “DAQ · ligado/desligado”** — *ligado* quando um sensor de luz no nível da matriz está configurado **ou** quando qualquer câmera membro possui seu próprio sensor; sua dica de ferramenta lista exatamente qual sensor alimenta qual câmera.
* Atribua a configuração para toda a matriz nas configurações da matriz → **Sensor de Luz Ambiente**→ menu suspenso**Sensor de Luz**. A seleção permanece com o projeto, se propaga para todas as câmeras do conjunto, e as câmeras individuais ainda podem substituí-la pelo próprio sensor.
* A linha de status abaixo dela informa o estado em tempo real: **Desativado**→ “Aguardando o primeiro espectro…” →**“Ativo — todas as câmeras do conjunto têm a iluminação corrigida”** → ou, se nenhum novo espectro chegou nos últimos 3 s, um aviso de dados desatualizados — a última leitura continua sendo usada (as leituras nunca expiram no caminho de captura).

Com um sensor atribuído: o tipo de exportação “Refletância” fica disponível, as visualizações em tempo real são corrigidas quanto à iluminação, a exposição automática preditiva pode usar o espectro e cada captura de refletância grava a leitura do DAQ efetivamente usada como um **arquivo `.daq`** na pasta `daq/` para que a captura possa ser reprocessada posteriormente. Ao lado dele, um arquivo `.daq.cameras.json` identifica as câmeras cujos quadros essa leitura descreve; ao reprocessar, o Chloros combina a leitura apenas com imagens dessas câmeras.

## Opções de CLI do `array-connect`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectar automaticamente todas as câmeras LATTICE (requer ≥2) | **O primeiro número de série é o MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | auto | Taxa de disparo do mestre. |
| `--binning {1,2,4}` | auto | Agrupamento de pixels por hardware. “Auto” significa o sensor completo (sem agrupamento) sempre que qualquer taxa de quadros o suportar; “connect never bins” mantém a taxa elevada sem agrupamento. |
| `--packed {auto,on,off}` | `auto` | [Transporte compactado de 12 bits](#packed-12-bit-transport): O `auto` compacta apenas quando isso ajuda e o host tem capacidade para descompactar. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituição avançada do seletor de camada de sincronização. |
| `--wire-ceiling-mbps MB_PER_S` | detectado automaticamente | Orçamento de banda do host em MB/s — a forma de CLI do campo **Orçamento de banda**. Reduza-o se o array relatar quadros corrompidos. Salvo com o projeto, de modo que uma reconexão posterior o restaure. |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativa o PTP (os carimbos de data/hora entre câmeras não serão comparáveis). |

`lattice array-list`, `array-status` e `array-disconnect` gerenciam a sessão persistente. A referência completa aos subcomandos, incluindo alinhamento (`align-calibrate` / `align-apply`) e as ferramentas de rede, encontra-se na [Referência da CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); os equivalentes no SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) estão na [Referência do SDK](../reference/sdk-reference.md). No Python, o orçamento de transmissão é `connect_array(..., wire_ceiling_mbps=120)`, e a divisão entre dados corrompidos em tempo real e dados que nunca chegaram está em [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
