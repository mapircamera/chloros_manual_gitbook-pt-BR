# Matriz de câmeras

Uma **matriz**LATTICE consiste em duas ou mais câmeras LATTICE conectadas como uma única unidade sincronizada. Uma câmera atua como**mestre**: ela emite um pulso de acionamento GPIO de hardware em uma linha de sincronização compartilhada (padrão**Linha 2**), de modo que todas as câmeras captam o mesmo instante. O Chloros adiciona sincronização de tempo PTP, uma visualização ao vivo (blocos por câmera ou uma única imagem composta multibanda alinhada) e captura sincronizada — cada passagem de captura produz um**grupo de quadros** no qual todas as câmeras compartilham o mesmo carimbo de data/hora e ID de quadro (relatado como `fid:N` na saída de captura).

Os conjuntos de câmeras são a forma como as câmeras mono (M3M) produzem índices de vegetação — uma câmera contribui com uma banda, e o conjunto as alinha em uma pilha multibanda. Consulte [Câmeras Mono e Índices de Vegetação](mono-indices.md).

Existem três maneiras equivalentes de conectar uma matriz, e todas elas executam o mesmo fluxo de “preparação inteligente”:

| Superfície | Ponto de entrada |
| --- | --- |
| GUI | Guia “Câmeras” → **Conectar Matriz** (botão azul) |
| CLI | `chloros-cli lattice array-connect --serials SN1,SN2,…` (primeiro número de série = mestre) |
| Python SDK | `connect_array(serials=[…])` → `ArraySession` (primeiro número de série = mestre) |

O Smart-prep executa, na seguinte ordem: um teste de capacidade de rede (ping ICMP DF + teste GVSP), seleção da camada de sincronização, redução automática do tamanho do quadro para se adequar à linha, ativação do PTP, seleção automática do formato de pixel por câmera, inicialização automática da exposição a partir do estado salvo de cada câmera e configuração do gatilho GPIO na Linha 2.

{% hint style="info" %}
As câmeras devem estar acessíveis no link antes que qualquer uma dessas funções funcione — consulte [Conectando Câmeras](connecting.md) para descoberta, endereçamento e download da calibração na primeira conexão. Para configurações com várias câmeras, as configurações do anel de recepção da placa de rede do host são tão importantes quanto a velocidade do link; a tabela completa de sintomas→soluções está na [CLI Referência § Configuração e Ajuste da NIC do Host](../reference/cli-reference.md#host-nic-setup--tuning-lattice-arrays).
{% endhint %}

## A caixa de diálogo “Conectar Matriz”

A guia “Câmeras” → **Conectar Matriz**abre um assistente de três etapas:**Selecionar → Modo de exibição → Configurações**.

### Etapa 1 — Selecionar mestre e escravos

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Select scene, with 3-4 LATTICE cameras discovered. Table showing Camera / Serial / IP / Master radio / Slave checkbox columns, with the green "GPIO master detected — selections pre-populated" probe banner visible above the table. -->

A caixa de diálogo varre a rede assim que é aberta (“Varrendo a rede...”); em seguida, verifica a fiação do gatilho GPIO (“Verificando a fiação GPIO...”). São necessárias pelo menos **2 câmeras** para formar um array.

A verificação da fiação preenche automaticamente a seleção de funções quando possível e exibe uma das três mensagens a seguir:

| Mensagem | Significado |
| --- | --- |
| “Mestre GPIO detectado — seleções preenchidas automaticamente” (verde) | A verificação encontrou a topologia de acionamento; as caixas de seleção de rádio mestre e escravo já estão marcadas. |
| “Nenhum mestre detectado — verifique o cabo GPIO” (laranja) | Nenhuma câmera detectou um pulso de acionamento; verifique o cabeamento de sincronização. Você ainda pode selecionar as funções manualmente. |
| “Sem cabo de sincronização: {seriais}” (laranja) | As câmeras listadas não têm nenhum cabo de sincronização conectado. |

A tabela de câmeras possui as colunas **Câmera / Serial / IP / Mestre (rádio) / Escravo (caixa de seleção)**:

* Selecione exatamente **um mestre**e**um ou mais escravos**. Clicar novamente no rádio do mestre atual o desmarca.
* Uma câmera marcada como **“Sem cabo de sincronização”** nunca pode ser selecionada como escrava — uma escrava sem fiação de disparo ficaria esperando na linha de sincronização para sempre e transmitiria um sinal inativo. Conecte essa câmera como uma câmera autônoma, em vez disso.
* Câmeras já conectadas de forma autônoma *não* são desativadas: a conexão em matriz libera a sessão autônoma e reabre a câmera dentro da matriz.

**Próximo: Modo de exibição →**é ativado assim que um mestre e pelo menos um escravo forem escolhidos.**Reexaminar** reexecuta a descoberta e o teste de fiação.

{% hint style="warning" %}
**Cancelar** fica desativado enquanto uma varredura ou sondagem estiver em andamento — cancelar no meio de uma sondagem pode travar a câmera SDK no firmware da câmera LATTICE. Aguarde até que o indicador de carregamento termine.
{% endhint %}

### Etapa 2 — Modo

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Display Mode scene, showing the two selectable cards ("Separate Cameras" and "Combined Cameras") with Combined selected/highlighted as the default. -->

de exibição | Modo | O que você obtém |
| --- | --- |
| **Câmeras separadas** | Um bloco ao vivo por câmera, todos acionados juntos para que os quadros permaneçam sincronizados. Cada câmera mantém sua própria cor e configurações. |
| **Câmeras combinadas** *(padrão)* | Um único bloco exibindo a composição alinhada multibanda NDVI/índice. As câmeras compartilham a cor da matriz. |

O modo de exibição altera apenas a apresentação da pré-visualização ao vivo — o comportamento de captura é o mesmo em ambos.

### Etapa 3 — Configurações da matriz e resultado

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, healthy state: left column with ROI / Binning / Pin resolution / Trigger Rate controls, right "Projected Outcome" column showing green "Simultaneous capture" tier, an fps range, the NIC line, the "Sim-emit burst" line, and the "Wire budget" line with a checkmark. -->

projetado Ao entrar nessa cena, o Chloros solicita ao backend uma **recomendação**e aplica automaticamente uma combinação de ROI + binning que se adapta ao anel de recepção da sua placa de rede (ele prefere o binning ao recorte de ROI, pois o binning preserva o campo de visão completo). Cada alteração que você fizer reexecuta a análise em tempo real e atualiza o painel**Resultado Projetado** à direita.

Coluna esquerda — configurações:

| Controle | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **ROI (Campo de Visão)** | Completo (2048×1536) / Metade (1024×768) / Um quarto (512×384) | Completo | Recorte do sensor: recorte de metade/quarto para uma região menor com o passo de pixel nativo. |
| **Binning** | 1× / 2× (soma 2×2) / 4× (soma 4×4) | 1× | Agrupamento por hardware: 2×2 = campo de visão completo a um quarto do custo de transmissão; 4×4 = campo de visão completo a 1/16. Oculto se as câmeras não suportarem agrupamento. |
| **Imagem no lado do cabo** (leitura) | — | — | A largura × altura pós-binning efetivamente enviada pelo cabo, redimensionada para múltiplos de 16 (mínimo 64). |
| **Resolução dos pinos**| caixa de seleção | desativada | O Chloros normalmente ativa o binning automaticamente na conexão quando a taxa projetada fica abaixo de**1,5 fps**. A fixação mantém o tamanho de quadro escolhido e aceita a taxa mais baixa — e transforma uma configuração com excesso de demanda em uma recusa de conexão definitiva, em vez de uma redução automática. |
| **Taxa de disparo** | 0,5–60 fps, passo de 0,1 | vazio = automático | A taxa de disparo do mestre. Deixe em branco para permitir que o Chloros a calcule. |
| **Capacidade de banda**| 20–2000 MB/s, incremento de 10 | vazio = automático | Quanto o host pode realmente absorver, em MB/s —**o único número do qual depende toda a alocação da matriz.** Detectado automaticamente a partir do adaptador de rede. Reduza-o se o array relatar quadros corrompidos: o valor detectado superestima a capacidade de adaptadores USB e switches compartilhados. Alterá-lo faz com que a projeção seja executada novamente em tempo real. |

Coluna da direita — **Resultado projetado**:

* **Nível de sincronização** — “Captura simultânea” (verde), “Captura simultânea (emissão escalonada FTD)” (verde), “Captura escalonada (desvio de 100 ms)” (âmbar) ou “Configuração muito grande” (vermelho).
* **projeção de fps** — exibida como um intervalo (“escuro → claro”), pois a taxa de uma matriz sincronizada é limitada pela exposição da câmera mais lenta.
* **Linha da NIC** — velocidade do link e taxa de transferência sustentada (“NIC {mbps} Mbps · sustentada {N} MB/s”).
* **Verificação de rajada de emissão simulada** — o anel de recepção da NIC do host consegue absorver uma rajada simultânea de todas as câmeras (“Rajada de emissão simulada: X MB · anel da NIC disponível: Y MB ✓/✗”).
* **Verificação do orçamento de banda** — demanda agregada em estado estacionário versus o limite máximo de banda à prova de colisões (“Orçamento de banda: {demanda} MB/s exigidos por {n} câmeras · limite máximo {limite} MB/s ✓/✗ sobrecarregado”).
* **“Número máximo de câmeras neste cabo: {n} — definido pelo limite mínimo de largura de banda por câmera, portanto o agrupamento não aumenta esse valor.”** — exibido quando você está próximo (ou acima) do limite máximo de número de câmeras.
* **“HAVERÁ PERDA DE QUADROS com essas configurações.”**— aviso em vermelho com o motivo indicado pelo backend, além de uma lista de bloqueadores e**sugestões de correção** em azul (“Para acomodar esta matriz na rede” / “Para habilitar a captura simultânea”).**Aplicar e Conectar** fica bloqueado até que haja uma projeção, e o texto do botão explica por que a operação está sendo recusada:

| Texto do botão | Significado | O que realmente ajuda |
| --- | --- | --- |
| “Analisando...” | Análise ainda em andamento. | Aguarde. |
| **“Câmeras demais para esta rede”**| O conjunto sobrecarrega a banda (falha na verificação agregada). | Menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.**Um ROI menor NÃO ajudará** — veja abaixo. |
| **“Reduzir o ROI para habilitar”** | Os quadros seriam perdidos com essas configurações (falha na verificação de burst/anel). | Reduza a ROI, aumente o binning ou conserte o anel de recepção da placa de rede. |

<!-- SCREENSHOT-NEEDED: Array Connect dialog, Settings scene, over-subscribed state: red "Wire budget ... over-subscribed" line, the "Max cameras on this wire" hint, and the Apply button reading "Too many cameras for this network". Reproduce by configuring more cameras than the 1 GbE ceiling (e.g. 7+ cams at 1500 MTU) or with CHLOROS-simulated models via `lattice analyze-array`. -->

Durante a conexão, um **painel de download de calibração** verde pode aparecer com uma barra de progresso por porta serial: na primeira vez que uma câmera é conectada a uma máquina, o Chloros obtém seu pacote de calibração de fábrica de ~3,8 MB da câmera via GigE (aproximadamente 70 segundos por câmera). Câmeras armazenadas em cache nunca exibem esse painel. Consulte [Conectando câmeras](connecting.md).

## Largura de banda: quantas câmeras cabem

A capacidade de uma matriz depende da largura de banda da conexão, não do Chloros; portanto, os valores de planejamento constam no manual do hardware: **[Planejamento de largura de banda da matriz](https://mapir.gitbook.io/lattice-camera/setup/array-bandwidth-planning)**.

O que o Chloros faz com esses dados: a caixa de diálogo de conexão executa um teste de rede, projeta a taxa de quadros alcançável e seleciona um nível adequado. Se a matriz sobrecarregar o cabo, ela se recusa a se conectar em vez de descartar pacotes silenciosamente — consulte o painel de resultados projetados descrito acima.

## Quando os quadros desaparecem

Uma câmera pode estar ausente de um grupo publicado por dois motivos completamente diferentes,
e eles exigem soluções opostas. O Chloros os contabiliza separadamente, em vez de relatar um
número “incompleto” que não especifica nenhum dos dois:

| O que aconteceu | O que isso significa | Onde procurar |
| --- | --- | --- |
| **Corrompido**— o quadro chegou, mas estava estruturalmente corrompido | Perda de pacotes do GVSP no caminho de rede | O**orçamento de linha**, o anel de recepção da placa de rede, quadros jumbo, o switch |
| **Nunca chegou**— nenhum quadro chegou | A câmera não disparou, ou nada saiu dela | O**cabo de sincronização M8**, a linha de sincronização, se todos os membros estão armados |

A divisão é reavaliada a cada 10 segundos enquanto o conjunto de câmeras transmite. Acima de 5%, ela é
registrada com os dois números indicados, e cada buffer corrompido é relatado na primeira vez em que
ocorre por câmera; em seguida, os dados são agrupados uma vez por minuto para que uma sessão longa permaneça legível.

**Quadros corrompidos com zero “nunca chegou” significam que o disparo e a sincronização do cabo estão perfeitos**e que todos os quadros perdidos estão no caminho da rede. A solução é diminuir o**Wire Budget** e
reconectar.

{% hint style="warning" %}
**Reduzir a taxa de acionamento não ajuda com quadros corrompidos.** O ritmo de pacotes
da câmera é definido uma única vez, na conexão. Reduzir a taxa de acionamento altera a frequência com que uma rajada
ocorre, não a velocidade com que a própria rajada chega ao cabo. Em um equipamento medido com 4 câmeras, uma
redução de 5× na taxa de disparo não alterou nada, enquanto diminuir o orçamento de banda de 240 para
200 MB/s levou o mesmo equipamento de 10,4 % de quadros corrompidos para zero.
{% endhint %}

Um array em execução não pode se replanejar — desconecte e reconecte para que o seletor de tempo de conexão
possa funcionar com base no novo orçamento.

### Adaptadores de rede USB têm limite de 200 MB/s

Um adaptador Ethernet USB anuncia sua taxa de link *Ethernet*, mas o que ele pode realmente
manter é limitado pelo barramento USB e seu driver. Um dongle USB 10GbE costumava ser creditado
com cerca de 1.000 MB/s de taxa de transferência — um número que nunca havia sido medido — e sincronizar
quatro câmeras com base nessa margem fantasmagórica corrompia de 6 a 18 % dos quadros, enquanto o array
ainda relatava uma taxa de quadros alvo satisfatória. Os adaptadores conectados via USB agora estão limitados a
**200 MB/s**. O limite é um valor absoluto, e não uma porcentagem, pois o restritor é o
barramento: um adaptador USB 1 GbE atinge cerca de 80 MB/s e não é afetado.

Se o seu host for realmente mais rápido que o limite, aumente o **Wire Budget** para refletir isso.

## Sincronização de tempo PTP

A *sincronização* de quadros vem do gatilho de hardware; o **PTP** (IEEE 1588 PTPv2) fornece *carimbos de data/hora* comparáveis em todos os dispositivos. Ele é ativado por padrão ao conectar a matriz:

* O **backend do host Chloros executa o grandmaster PTP**. As câmeras LATTICE e os sensores de luz DAQ-E funcionam como escravos dele no domínio 0, de modo que os carimbos de data/hora das imagens e os espectros do DAQ ficam em um único relógio (~1 ms).
* O `--no-ptp` (CLI) desativa essa função para trabalhos em bancada — os carimbos de data/hora entre câmeras, então, **não** são comparáveis.
* Verifique o estado da sincronização com o CLI:

```bash
chloros-cli time-sync status     # grandmaster state, clock identity
chloros-cli time-sync peers      # slaves seen (cameras + DAQ-E sensors)
chloros-cli time-sync cameras    # per-camera PtpStatus / PtpOffsetFromMaster / PtpMeanPathDelay
```

A guia “Câmeras” em si não possui indicador PTP; as informações de sincronização por câmera exibidas ali são: **Função**(Mestre/Escravo) — somente leitura,**Linha de Sincronização**e o nível de**Capacidades** do conjunto. O estado PTP do DAQ-E é exibido nos detalhes do sensor na guia “Sensores de Luz”.

## A visualização

<!-- SCREENSHOT-NEEDED: Cameras tab with a connected combined array: sidebar showing the ARRAY row (color badge, array name, "DAQ · on" pill) with indented member camera rows, and the main area showing the combined index composite tile with the LUT-colored NDVI render, top-left array name pill, and top-right fps readout. -->

ao vivo da matriz A área principal de exibição oferece dois layouts (alterne na barra superior): **visualização em grade**(cada bloco é uma célula; arraste para reordenar quando o cadeado da grade estiver destravado) e**visualização em lista**(matrizes em largura total na parte superior, uma câmera ativa abaixo). O controle deslizante**Zoom da exibição** ajusta o tamanho dos blocos; abaixo de 200 px de largura de célula, as sobreposições de nome/fps são ocultadas automaticamente.

O **modo separado** exibe um bloco por câmera. Cada bloco exibe:

* o nome da câmera (canto superior esquerdo),
* uma **leitura de fps** (canto superior direito) — essa é a *taxa real de captura* da câmera informada pelo backend, não a taxa de atualização da pré-visualização (a pré-visualização ao vivo é limitada a 30 fps, independentemente da taxa de captura),
* um ponto de status — verde (transmissão) / âmbar (carregando) / vermelho (erro),
* um **ícone giratório de quadro desatualizado** quando nenhum quadro novo chega por 2 s — normal por cerca de 5 s após qualquer conexão/desconexão, enquanto o backend reequilibra a largura de banda entre as câmeras.

O **modo combinado**exibe um único bloco composto: o backend realiza o debayering, redimensiona, alinha, remove ruídos, converte para radiação por banda (mais refletância DLS quando um sensor de luz está vinculado), avalia a expressão de índice da matriz, aplica a LUT e transmite o resultado como MJPEG. Até que o primeiro quadro alinhado seja renderizado, o bloco indica seu estado: “Preparando matriz…”, “Calibrando alinhamento…”, “Aguardando o primeiro quadro…” ou — se o tempo de tentativa de realinhamento automático (~30 s) se esgotar — “Alinhamento necessário”, com um botão**Calibrar alinhamento**.

Informações úteis sobre o modo combinado:

* A imagem composta é registrada em relação ao quadro da câmera **mestre**. O foco AE-ROI e a medição pontual na imagem composta são exatos para a câmera mestre e aproximados para as escravas; use**Visão Dividida** (configurações da matriz → “Mostrar câmeras membros”) para blocos por câmera com precisão de pixel sem abrir conexões extras de câmera.
* **Exibir camadas**(configurações da matriz; desativado por padrão) permite escolher uma camada de primeiro plano e uma de fundo — qualquer câmera membro ou**Índice**. Com primeiro plano = Índice, os pixels fora do Min/Max da LUT mostram a camada de fundo.
* **Resolução de renderização** (padrão 720p) define a altura da transmissão ao vivo *e* o tamanho da exportação da composição salva. As imagens por câmera são sempre exportadas em resolução total.
* O alinhamento é calculado por sessão e nunca é mantido — consulte a seção de alinhamento do painel de configurações do conjunto para ver os resíduos RMS e o botão “Recalibrar”.

## Captura: monitoramento x análise

As superfícies de captura da matriz se dividem claramente em **nível de monitoramento**(grava o que você vê) e**nível de análise** (grava dados brutos, calibra posteriormente):

| Fluxo de trabalho | Nível | O que é salvo | GUI | CLI |
| --- | --- | --- | --- | --- |
| **Captura**(imagens estáticas) | Análise | Um grupo de quadros sincronizados por passagem; arquivos por câmera em todos os níveis de exportação selecionados (bruto/sem debayering/radiação/refletância/visualização/índice) + arquivo sidecar `.daq` |**Botão **Capturar tudo** + Configurações de captura | `lattice array-capture` |
| **Gravar vídeo de índice** | Monitoramento | A composição combinada do índice ao vivo conforme exibida — 8 bits, resolução de pré-visualização, LUT incorporada; requer que a transmissão ao vivo esteja aberta | ● Gravar vídeo de índice (matrizes combinadas) | `lattice array-record` |
| **Sequência bruta → criar vídeo**| Análise | Quadros brutos do sensor na taxa máxima de captura + manifesto + `.daq`; em seguida, reconstrução off-line em vídeo calibrado de radianância / refletância / índice, sincronizado com as leituras do DAQ | ⦿ Gravar sequência bruta →**Criar vídeo** | `lattice array-burst` → `lattice array-build-video` |

Regra geral: se os pixels forem fornecer *medições*, use a captura ou o burst (nível de análise); se você só precisar *assistir ou demonstrar* o que o array captou, grave o vídeo de índice (nível de monitoramento).

### Configurações de captura (GUI)

<!-- SCREENSHOT-NEEDED: Capture Settings pane (gear next to Capture All) with a connected array: capture-mode buttons (Single/Continuous/Interval), the bulk export-type toggle row, the Fastest Capture toggle, and the per-array group card showing the Aligned checkbox and the "Record index video" / "Record raw burst" buttons. -->

A engrenagem ao lado de **Capturar tudo** abre o painel Configurações de captura (requer um projeto aberto — as capturas são salvas nele):

* **Modo de captura**:**Único**(uma passagem) /**Contínuo**(consecutivamente; limitado por uma contagem de capturas, padrão 1, ou por uma duração, padrão 10 s) /**Intervalo** (timelapse: N capturas a cada intervalo de X para um total de Y; padrão: 1 a cada 5 s por 1 minuto).
* **Tipos de exportação por câmera**: Raw, Debayered, Radiance, Reflectance, Preview, Index — todas as opções aplicáveis estão ativadas por padrão. Radiance/Reflectance ficam ocultos para câmeras RGB-filter;**Reflectance aparece apenas quando a câmera possui um sensor de luz DAQ** (próprio ou herdado da matriz); Index requer uma expressão de índice configurada.
* **Alinhado**(por matriz, padrão**ativado**): distorce as exportações dos membros de acordo com o perfil de alinhamento da matriz para que as exportações sejam registradas por pixel. O formato Raw permanece sempre sem distorção, mas carrega a transformação nos metadados.
* **Captura mais rápida** (alternar): apenas dados brutos + a leitura DAQ atribuída + o composto de índice combinado livre, pulando os cálculos de calibração no momento da captura para obter a taxa máxima — reconstrua a radianância/refletância/índice posteriormente a partir do `.daq` salvo.
* As seleções permanecem com o projeto. Câmeras ocultas ou pausadas são ignoradas.

O equivalente a CLI (mesmo ponto final de back-end, mesma semântica):

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

A compactação para capturas no TIFF é `deflate` (sem perdas, padrão) ou `none` — tabelas de sinalizadores completas, o layout da pasta de captura e as regras de reprocessamento estão na [Referência do CLI](../reference/cli-reference.md#capture-modes-recorders--offline-reprocess).

## Emparelhamento de um sensor de luz DAQ

As visualizações prévias com correção de refletância e iluminação requerem dados de luz incidente provenientes de um sensor DAQ (conectado na guia **Sensores de luz**):

* A **linha da matriz**na barra lateral exibe um**botão &quot;DAQ · ligado/desligado&quot;** — *ligado* quando um sensor de luz no nível da matriz está configurado **ou** quando qualquer câmera membro possui o seu próprio; sua dica de ferramenta lista exatamente qual sensor alimenta qual câmera.
* Atribua para toda a matriz nas configurações da matriz → **Sensor de Luz Ambiente**→ menu suspenso**Sensor de luz**. A seleção permanece no projeto, é aplicada a todas as câmeras do array, e as câmeras individuais ainda podem substituí-la pelo próprio sensor.
* A linha de status abaixo dela informa o estado atual: **Desligado**→ “Aguardando o primeiro espectro…” →**“Ativo — todas as câmeras do conjunto têm a iluminação corrigida”** → ou, se nenhum espectro novo chegou nos últimos 3 s, um aviso de dados desatualizados — a última leitura continua sendo usada (as leituras nunca expiram no caminho de captura).

Com um sensor atribuído: o tipo de exportação “Refletância” fica disponível, as visualizações em tempo real são corrigidas quanto à iluminação, a exposição automática preditiva pode usar o espectro e cada captura de refletância grava a leitura do DAQ efetivamente usada como um **`.daq` sidecar** ao lado da imagem, para que a captura possa ser reprocessada posteriormente.

## Opções do `array-connect` CLI

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectar automaticamente todas as câmeras LATTICE (requer ≥2) | **A primeira porta serial é o MASTER.** |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | automático | Taxa de disparo do mestre. |
| `--binning {1,2,4}` | automático | Agrupamento por hardware. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituição avançada do seletor de camadas. |
| `--wire-ceiling-mbps MB_PER_S` | detectado automaticamente | Orçamento de banda do host em MB/s — a forma CLI do campo **Orçamento de banda**. Reduza-o se o array relatar quadros corrompidos. Salvo com o projeto, portanto, uma reconexão posterior o restaura. |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede. |
| `--no-ptp` | desativado | Desativa o PTP (os carimbos de data/hora entre câmeras não serão comparáveis). |

`lattice array-list`, `array-status` e `array-disconnect` gerenciam a sessão persistente. A referência completa aos subcomandos, incluindo o alinhamento (`align-calibrate` / `align-apply`) e as ferramentas de rede, encontra-se na [Referência do CLI § chloros-cli lattice](../reference/cli-reference.md#chloros-cli-lattice); os equivalentes do SDK (`connect_array`, `ArraySession`, `attach_array`, `analyze_array_network`) estão na [Referência SDK](../reference/sdk-reference.md). A partir de Python, o orçamento de fiação é `connect_array(..., wire_ceiling_mbps=120)`, e a divisão entre “corrupto ativo” e “nunca chegou” está em [`/api/camera/array/<id>/capability`](../reference/sdk-reference.md#array-health--which-subsystem-is-losing-frames).
