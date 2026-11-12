# Configurações da câmera

A aba **Câmeras** é a interface de controle em tempo real do Chloros para as câmeras LATTICE: uma área de feed principal que exibe todas as câmeras conectadas como blocos dinâmicos e uma barra lateral que alterna entre três páginas — a **lista de câmeras**, um**painel de configurações**(configurações por câmera, de matriz ou de captura — uma de cada vez) e a**Calculadora de Índice**. Esta página documenta todos os controles da lista de câmeras, do painel de configurações por câmera e do painel de configurações de matriz. Os modos de captura, seleção do tipo de exportação e o fluxo “Capturar Tudo” estão na página complementar [Configurações e Modos de Captura](capture.md).

A aba “Câmeras” aparece na barra lateral assim que o backend do Chloros estiver pronto. Todos os controles abaixo se comunicam com o backend local por meio do `127.0.0.1:5000`; as alterações são aplicadas imediatamente à câmera ao vivo, salvo indicação em contrário.

## Tipos de câmera utilizados nesta página

Os controles são exibidos ou ocultados de acordo com o tipo de câmera selecionado. O manual utiliza estes termos ao longo do texto:

| Termo | Significado | Canais de filtro |
| --- | --- | --- |
| **Câmera RGB** | LATTICE M3C com filtro FRGB (o modelo contém `-FRGB`) | Red / Green / Blue |
| **Multiespectral Bayer** | LATTICE M3C com FRGN, FOCN ou FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — um filtro de banda estreita, uma banda calibrada | Banda única |
| **Membro da matriz** | Uma câmera conectada como parte de uma matriz sincronizada (exibição combinada ou separada) | De acordo com seu filtro |

Câmeras RGB passam por processamento fotométrico (equilíbrio de branco, perfis de cor, gama); câmeras multiespectrais e monocromáticas passam pela cadeia radiométrica e ignoram os controles fotométricos. Os membros da matriz passam as configurações no nível do fluxo (formato de pixel, resolução, agrupamento de pixels, acionamento, taxa de quadros) para a matriz — essas linhas passam a ser somente de leitura no painel por câmera e são transferidas para o painel de configurações da matriz.

## A área principal de feed



<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

Sem câmeras conectadas, a área de feed exibe uma tela inicial com a mensagem **“Conecte uma câmera para começar”**e dois botões:**Conectar Câmera**(verde, abre a caixa de diálogo de conexão de uma única câmera) e**Conectar Matriz** (azul, abre a caixa de diálogo de conexão da matriz). As próprias caixas de diálogo de conexão estão documentadas em [Conectando Câmeras](connecting.md); os conceitos da matriz (sincronização, camadas, largura de banda) em [Matrizes de múltiplas câmeras](arrays.md). Ao abrir um projeto salvo que contenha câmeras, a tela inicial exibe, em vez disso, um indicador de carregamento com a mensagem “Reabrindo N câmeras salvas…” enquanto o Chloros restaura os fluxos da última sessão.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superior

| Controle | O que faz |
| --- | --- |
| **Alternar modo de visualização**| Alterna entre a**visualização em grade**(todos os blocos como células) e a**visualização em lista**(linhas de largura total: cada matriz combinada como uma linha composta, cada membro de uma matriz**separada** como sua própria linha com seu próprio feed e tabela de alvo de calibração, e, em seguida, UMA câmera independente — a ativa — abaixo). Dicas de ferramenta: “Alternar para a visualização em grade” / “Alternar para a visualização em lista”. |
| **Bloqueio da grade**(cadeado) | Padrão**bloqueado** — blocos fixos no lugar. Desbloqueie para arrastar e reordenar os blocos em qualquer posição (os espaços vazios são preservados). A grade se bloqueia automaticamente toda vez que uma nova câmera se conecta. Dicas de ferramenta: “Desbloquear grade (ativar arrastar de blocos)” / “Bloquear grade (congelar blocos no lugar)”. |
| Controle deslizante **Zoom da transmissão** | Tamanho do bloco, de 60 px até a largura total do contêiner. As células mantêm a proporção 4:3. Abaixo de 200 px de largura de célula, as sobreposições de nome e fps são ocultadas para manter o bloco limpo. |

### Blocos de feed

Cada câmera renderiza um bloco ao vivo composto; uma câmera pode, adicionalmente, exibir três blocos em escala de cinza **divididos por canal** (consulte [Divisões de canal](#display-overlays-drawn-over-the-live-feed)), e os conjuntos de câmeras renderizam um bloco combinado. O bloco ativo exibe um anel de seleção na cor da câmera (ou do conjunto).

Passar o mouse sobre um bloco revela um botão de fechamento **X**:

* Fechar um bloco **composto** enquanto suas divisões de canal permanecem visíveis apenas oculta o bloco composto.
* Fechar o **último bloco visível de uma câmera independente** desconecta essa câmera.
* **Os blocos de divisão de membros de um conjunto combinado nunca desconectam** a câmera — eles apenas a ocultam.

Com a grade desbloqueada, arraste qualquer bloco para qualquer espaço; o layout é salvo com o projeto.

## Barra lateral — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

de câmeras A primeira página da barra lateral lista todas as câmeras e matrizes conectadas:

* **Conectar Câmera**(verde) /**Conectar Matriz** (azul, exibe “Detectando...” durante a varredura). Ambos ficam desativados enquanto uma caixa de diálogo de conexão estiver aberta.
* **Capturar Tudo**(vermelho) — captura todas as câmeras listadas com os tipos de exportação escolhidos nas**Configurações de Captura**. Requer um projeto aberto. Totalmente documentado em [Configurações e modos de captura](capture.md).
* **Engrenagem de configurações de captura** (ao lado de Capturar tudo) — abre o [painel de configurações de captura](capture.md#the-capture-settings-pane). Desativado sem um projeto ou durante a captura.

### Linhas de câmera

Cada linha de câmera exibe uma borda codificada por cores (a cor personalizada da câmera), um rótulo “CAM” — com uma letra **M**azul (mestre) ou**S** verde (escravo) para membros da matriz — e o nome de exibição. O nome padrão é `LATTICE-MODEL (serial)`; renomeie-o no painel de configurações de cada câmera. Botões da linha:

| Botão | Efeito |
| --- | --- |
| **Olho**| Alterna a visibilidade. Câmeras ocultas saem da grade e são**excluídas da função “Capturar tudo”**. |
| **Engrenagem** | Abre o painel de configurações de cada câmera (próxima seção). |
| **Pausar / Reproduzir**| Congela a pré-visualização ao vivo**apenas no lado da tela** — a captura no backend continua em execução. Câmeras em pausa não podem capturar. |
| **X** | Desconectar. A interface do usuário é atualizada imediatamente (na melhor das hipóteses); a desconexão do back-end em si pode levar de 10 a 30 s para ser concluída. |

### Linhas da matriz

Uma linha da matriz exibe um ícone “ARRAY” na cor da matriz, o nome da matriz (que pode ser renomeado nas configurações da matriz) e um botão **DAQ · ligado/desligado**—**ligado** quando o sensor de luz no nível da matriz está configurado *ou* qualquer membro possui um sensor por câmera; sua dica de ferramenta lista exatamente qual sensor alimenta o quê. As câmeras membros são listadas recuadas abaixo, em suas próprias linhas. Botões da linha da matriz: **olho**(oculta/exibe TODOS os membros juntos),**engrenagem**(painel de configurações da matriz),**X**(desconecta toda a matriz).

O status do sensor de luz (DLS) usado nas linhas da matriz e no painel de configurações da matriz possui quatro estados:**desligado**,**em espera**(ainda sem espectro),**ativo**(um espectro chegou nos últimos 3 s) e**desatualizado** — nenhum espectro novo nos últimos 3 s, mas a última leitura *ainda é usada* (as leituras do DAQ nunca expiram no caminho de captura).

Até que você reordene a lista, ela exibe primeiro os arranjos, depois as câmeras independentes, com as câmeras (e os membros de cada arranjo) na **ordem das câmeras**: M3C (Bayer) antes de M3M (mono); entre M3C, FRGB, FRGN, FOCN e, em seguida, FNGB; M3M por comprimento de onda do filtro, do menor para o maior. O layout padrão da grade segue a mesma ordem. Você pode arrastar câmeras independentes e grupos inteiros de matrizes uns sobre os outros na barra lateral para reordenar a lista; os membros da matriz não podem ser arrastados independentemente.

## Painel de configurações por câmera

Abra clicando na **engrenagem** na linha de uma câmera. O painel desliza sobre a lista de câmeras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->



**Cabeçalho**: a**amostra de cor**da câmera (clique para abrir um seletor de cores nativo — define a cor da borda da barra lateral e do anel de seleção de blocos), o**nome**com um botão de lápis**Renomear**(salvar um nome vazio reverte para o padrão `MODEL (serial)`) e**×** para fechar.

### Histograma em tempo real

Na parte superior do painel há um histograma de luminância em tempo real calculado a partir da pré-visualização em JPEG a ~8 Hz. A média é ponderada pelo método Bayer — (R+2G+B)/4 — para corresponder à medição de AE da própria câmera.

* **Linha tracejada Orange**= a meta de AE. **Arraste-a horizontalmente para redefinir a meta** — um comando é enviado ao soltar, e o ato de arrastar alterna o modo de meta de AE para Manual.
* **Linha contínua Green** = a média real de luma (o que o AE está fornecendo no momento).
* **Botão RGB** (canto superior direito): alterna entre histogramas sobrepostos por banda, coloridos de acordo com o filtro da câmera (por exemplo, no FRGN: NIR cinza, verde, vermelho). Em câmeras mono (M3M), o botão exibe “MONO” e fica desativado — o modo mono sempre mostra o histograma de luminância de banda única.
* Os rótulos do eixo X seguem a profundidade de bits do sensor do formato de pixel atual: 0..255, 0..1023, 0..4095 ou 0..65535.

### Linhas

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

de informações da câmera

| Linha | Comportamento |
| --- | --- |
| **Modelo** | Somente leitura (por exemplo, `LATT-M3C-L87-FRGN`). |
| **Calibração radiométrica** | Emblema Green**"Ativo"**com uma legenda mostrando o nível de calibração, o hash, a data da calibração e a lista de bandas, carregados a partir do pacote de calibração da câmera (consulte [Calibração radiométrica de fábrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Ocultado para câmeras RGB** — elas possuem uma calibração fotométrica de balanço de branco, e não radiação por banda. |
| **Relatório de calibração**| Botão**Baixar** — abre o PDF do certificado de calibração NIST específico para cada número de série da câmera no visualizador do seu sistema operacional. Se o certificado ainda não estiver armazenado em cache, o Chloros exibe uma dica. |
| **Número de série** | Somente leitura. |
| **Firmware**| Mostra a versão atual e, em seguida, identifica a versão disponível para este modelo (armazenada em cache por modelo — um conjunto de N câmeras verifica o servidor uma única vez). Estados: “Verificando…” → botão**“Atualizar para X”**→ “Atualizando…” → “Atualizado de A para B” / “Falha: …” / “Ignorado: …” / verde**“Atualizado”**. Dica de ferramenta do botão de atualização: “Redefinição de fábrica + flash + reprogramação do UserSet1. ~2–3 minutos; não desconecte.” |
| **IP** | Somente leitura. |
| **Temperatura** | Somente leitura, atualizada a cada 3 s. Fica laranja a partir de ≥65 °C e vermelha com um ⚠ a partir de ≥75 °C. |
| **Alvo de calibração** caixa de seleção | Ativa a detecção de alvo de refletância ArUco com uma tabela de validação NDVI por painel abaixo da transmissão ao vivo (visualização em lista) — consulte [A tabela de alvo de calibração](#the-calibration-target-table). Apenas para a sessão — abre sempre desativada. |
| Menu suspenso **Sensor de luz** | Vincula um sensor de luz DAQ (DAQ-E/M/U, da lista da aba Sensores de luz) a esta câmera para correção de iluminação de luz descendente (DLS) e exposição automática preditiva. ““Nenhum” cancela a vinculação. Se nenhum sensor estiver conectado, o menu suspenso exibe “(nenhum sensor conectado — abra a aba DAQ)”. A vinculação é salva com o projeto. |

#### A tabela Alvo de Calibração

A tabela é uma **leitura apenas para validação** (“apenas validação — não modifica a calibração”). Seu cabeçalho indica o número de série da câmera, o filtro, o número do alvo e o tamanho do marcador em pixels, e possui uma linha por painel:

* **Painel**, seguido de uma coluna por banda com o valor do painel — uma câmera Bayer exibe suas três bandas por nome e comprimento de onda (por exemplo, FRGN: Red 660 nm / Green 550 nm / NIR 850 nm); uma **câmera mono (M3M) exibe sua única banda** (por exemplo, `F750`) e não possui colunas de NDVI.
* **Sat** — a proporção de pixels do painel que estão saturados (âmbar a partir de 1 %, vermelho a partir de 5 %).
* **NDVI Medido**,**NDVI Teórico** (o NDVI de refletância de referência do painel) e **Δ**— apenas para câmeras multiespectrais Bayer. Uma pequena marca após “NDVI Medido” indica a partir de que o NDVI medido foi calculado:**ρ**(refletância — há leitura disponível do sensor de luz),**L**(radianância — sem leitura do sensor de luz) ou**DN** (valores brutos da câmera). Apenas um NDVI**ρ** é comparável ao NDVI Teórico: para L e DN, a coluna Δ fica desativada e uma nota indica isso (“Sem leitura do sensor de luz: o NDVI é derivado da radianância, portanto Δ não é uma verificação de calibração” / “O NDVI é derivado dos valores brutos da câmera, portanto Δ não é uma verificação de calibração”). Com ρ, Δ fica verde abaixo de 0,02, âmbar abaixo de 0,05 e vermelho nos demais casos.

<!-- SCREENSHOT-NEEDED: a 1.3.0 Calibration Target table under a live feed in list view — a Bayer camera's table showing the band columns, Sat, and "NDVI Meas ρ" / NDVI Theo / Δ; ideally next to a mono camera's single-band table with no NDVI columns. -->

### Exposição e Ganho

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Todas as entradas numéricas aqui usam controles giratórios do tipo “manter pressionado para acelerar”: tocar = ±1, manter pressionado por mais de 1,5 s = ±10, manter pressionado por mais de 3 s = ±100. O valor é enviado para a câmera quando você solta.

| Controle | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Exposição (us)**| Mín./máx. em tempo real da câmera | Automático | Todos | Tempo de exposição em microssegundos, com um botão de alternância**Automático/Manual**. Automático = exposição automática contínua pela câmera. Ao definir uma exposição manual, o**Ganho** permanece em Automático (se já estivesse), permitindo usar toda a faixa de ganho da câmera, até que você edite o ganho ou o defina como Manual. |
| **Ganho (dB)**| Mínimo/máximo em tempo real da câmera (por exemplo, até 48 dB) | Manual (desativado) | Todos | Ganho analógico/digital com seu próprio botão de alternância**Automático/Manual**. |
| **Brilho-alvo da AE**| 0–255 | 80, modo**Automático**| Todos (editável quando a AE ou o ganho automático estão ativados) | O brilho que a AE busca atingir. No modo**Automático**(padrão), um controlador interno baseado no histograma seleciona a meta automaticamente, mantendo a exposição entre 60% e 75% do máximo do sensor. Digitar um valor ou arrastar a linha laranja do histograma alterna para**Manual**. |
| **Suavização de AE** | 0,5–40, passo de 0,1 | 8,0 | Todos | Amortecimento do AE. Dica de ferramenta: “Valor mais baixo = a AE reage mais rápido (pode apresentar oscilações em altas taxas de quadros). Valor mais alto = mais suave / mais lento.” Valores muito abaixo do padrão podem fazer com que a AE oscile e desestabilize a transmissão em altas taxas de quadros; 8,0 é o padrão estável. |
| **Região de Interesse da AE**| Caixa de seleção “Ativar” + botão**Mirar**| Desativado | Todos | Quando ativado, o AE mede apenas a região verde tracejada em vez de todo o quadro.**Mirar** ativa a função “clicar para posicionar” na transmissão ao vivo: um clique centraliza uma região em 30% do quadro; clicar e arrastar traça um retângulo personalizado (mínimo 5% × 5%). O “Aim” se desativa automaticamente após um posicionamento. A região é remapeada para as coordenadas nativas da câmera de acordo com qualquer rotação/espelhamento que você tenha definido e é salva com o projeto. |
| **Velocidade de Ajuste do AE** | 0,1–5, passo de 0,1 | 1,0 | Apenas para membros da matriz | A rapidez com que o alvo do AE automático acompanha as mudanças de brilho da cena; 1,0× verifica novamente a cada 2,5 s. |
| **Proteção de realces** | Rigorosa (1 %) / Normal (5 %) / Flexível (15 %) | Rigorosa | Câmeras que expõem a configuração | Quanto do quadro pode ficar clipeado em branco antes que o AE escureça a imagem. |

{% hint style="info" %}
**Requisito de iluminação para câmeras multiespectrais Bayer (RGN / OCN / NGB):** a cena deve ter luz suficiente nos três canais; caso contrário, a calibração não funcionará corretamente — uma única exposição do sensor abrange todos os três espectros. Use um sensor de luz DAQ para medir a iluminação ou opte pelo modo totalmente monocromático (M3M) para que cada banda receba sua própria exposição. Se uma captura violar essa regra, o Chloros detecta isso e avisa você (a notificação de limitação da separação de canais). Em tempo real, um bloco de divisão de canais exibe **“Dados da banda com baixa confiança · N%”** quando o unmix limitou essa banda em 25% ou mais de seus pixels.
{% endhint %}

{% hint style="info" %}
**Como a exposição automática distribui a exposição e o ganho (1.3.0).** Com o ganho em “Auto”, o Chloros mantém o ganho em 0 dB até que a exposição atinja o valor máximo permitido (o limite superior da AE, o período de disparo de uma matriz ou o teto de exposição do DAQ); somente então o ganho aumenta. Em uma matriz, a taxa de disparo tem prioridade: a exposição automática preenche o período determinado pela taxae, em seguida, aumenta o ganho; assim, uma cena que escurece não reduz mais a taxa de disparo automática (na versão 1.2.0 isso acontecia). Uma exposição **manual**em um membro da matriz com duração superior ao período de disparo desacelera o disparo da matriz para se adequar a ela; a taxa automática é restabelecida assim que todos os membros que a retardaram voltarem à exposição automática. Com**Exposição**e**Brilho-alvo da AE** ambos em Automático e um sensor de luz DAQ fornecendo uma leitura, a exposição de cada câmera é mantida assim que a exposição automática se estabilizar, desde que a luz medida permaneça dentro de cerca de ±25 %; uma variação de luz maior e sustentada, uma reinicialização das configurações (por exemplo, uma alteração na ROI) ou uma alteração de exposição feita por você reativa o sistema.
{% endhint %}

{% hint style="info" %}
**Como a exposição automática se comporta (1.3.0).**

* Uma exposição manual mantém o ganho em Automático até que você altere o ganho.
* Pequenos realces não escurecem a cena: um realce especular de até 1% do quadro é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5% de um painel.
* O limite máximo de exposição do DAQ se aplica apenas enquanto um alvo de calibração estiver em vista; sem nenhum painel visível, a câmera expõe para a cena.
* A exposição permanece estável sob luz constante e se estabiliza em poucos segundos após uma grande mudança. Um clipe não limita mais a exposição permanentemente, e uma alteração na região de interesse (ROI) restabelece a exposição.
* O alvo de brilho não ultrapassa o alcance da câmeraalcance da câmera, de modo que o retorno da luz não causa reflexos no quadro.
* Uma câmera cuja exposição automática deixa de responder sob sol forte é mantida no nível adequado à luz em que se encontra, lido pelo sensor de luz ou, na ausência de um sensor conectado, pelos próprios quadros da câmera, e é liberado quando a luz muda. Um sensor de luz escuro ou coberto não interfere nisso.
{% endhint %}

### Formato de Píxeis e Resolução

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

**Os membros da matriz** exibem as linhas “Atual” (formato + LxA) e “Binning”, somente para leitura, com a observação “Definido nas configurações da matriz” — reiniciar a transmissão em um membro quebraria a sincronização, portanto, esses parâmetros são gerenciados no [painel de configurações da matriz](#painel-de-configurações-da-matriz).**Câmeras autônomas** apresentam:

| Controle | Opções | O que faz |
| --- | --- | --- |
| **Formato de Píxeis** | 8 bits / 10 bits / 12 bits / 16 bits | Profundidade de bits do sensor; a família de formatos é a do sensor. O padrão é 12 bits, e um projeto salva o que você definir, incluindo 8 bits, e se reconecta com essa configuração. |
| **Resolução** | Completa / Metade / Um quarto | Relativa ao binning atual: Completa = 2048/N × 1536/N para binning N×N. |
| **Binning** | 1x1 (nenhum) / 2x2 / 4x4 | Binning N×N por hardware — valores maiores reduzem a resolução, mas aumentam a relação sinal-ruído (SNR) e a taxa de quadros. Alterá-lo reinicia o fluxo e redefine qualquer ROI para o novo campo de visão total. |
| **Atual** | somente leitura | As dimensões reais LxA e o deslocamento (x, y) em vigor. |

### Visualização ao vivo

Tudo nesta seção é **apenas para exibição**— altera o que você vê na transmissão ao vivo, enquanto as capturas salvas permanecem lineares e inalteradas — com uma exceção: o**Vignette** é radiométrico e afeta as exportações também (conforme destacado abaixo).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->



| Controle | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Resolução de renderização** | 360p (mais rápida) / 480p / 720p / 1080p / Resolução nativa do sensor (mais lenta) | 720p | Todos | A altura na qual o backend executa a cadeia de pré-visualização radiométrica. Um valor menor aumenta a taxa de quadros sem alterar o campo de visão. |
| **Índice**| Caixa de seleção “Ativar” + engrenagem | Desativado | Apenas multiespectral Bayer,**não** membros de matriz combinada | Pré-visualização ao vivo do índice de vegetação. A engrenagem abre a [Calculadora de Índice](#index-calculator-pane) compartilhada, pré-carregada com as bandas naturais do filtro da câmera (por exemplo, `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). A expressão personalizada mais a LUT (ligado/desligado, nível padrão de 7 stops, mínimo padrão de 0,2, máximo padrão de 1) é calculada em cada quadro de pré-visualização. Os membros da matriz combinada ocultam esta linha — a matriz possui um único índice compartilhado. |
| **Equilíbrio de Brancos** | Desligado / Uma vez / Contínuo + um botão de recaptura | Contínuo | Somente RGB | Equilíbrio de branco em tempo real. O botão de atualização recaptura o equilíbrio de branco a partir do espectro DLS atual (desativado quando o modo está Desligado). |
| **Gama** | Ligado / Desligado | Ligado | Somente RGB | Exibe a gama (γ = 2,2 LUT) na pré-visualização ao vivo. As capturas salvas permanecem lineares. |
| **Redução de ruído** | Caixa de seleção + intensidade 0–100 | Desligado / 50 | Todos (por câmera, mesmo dentro de matrizes) | Filtro bilateral na visualização ao vivo. Valor mais alto = detalhes mais suaves, mas menos nítidos. |
| **Nitidez** | Caixa de seleção + intensidade 0–100 | Desligado / 30 | Todos | Máscara de nitidez na visualização ao vivo, aplicada por último. Pode amplificar o ruído. Apenas na visualização. |
| **Vinheta**| Caixa de seleção + intensidade 0–100 | Desativado / 0 | Todas | Remoção manual de vinheta residual (clareia os cantos), sobreposta à estimativa de Vinheta Inteligente da matriz.**Radiométrico — afeta a visualização ao vivo E as exportações**, ao contrário de Redução de ruído/Nitidez. |
| **Perfil de cor** | Raw / Linear / Natural / Aprimorado / Temperatura personalizada | Natural | Apenas RGB | Veja abaixo. |
| **Temperatura de cor** | 2.000–10.000 K, intervalo de 100 | 5.500 K | Apenas RGB e perfil de temperatura personalizada | Fixa o equilíbrio de branco em uma temperatura de cor correlacionada fixa (a entrada DLS é ignorada). O último valor em Kelvin selecionado é lembrado mesmo ao alternar entre perfis. |
| **Destaques** | −100 a 0, em incrementos de 1 | 0 (desativado) | Somente RGB | Novidade na versão 1.3.0. Escurece as áreas mais claras e brancas da imagem (0 = inalterado, −100 = intensidade máxima). Aplica-se à visualização ao vivo **e às capturas no nível da tela (Pré-visualização)**, que registram a configuração (`Chloros:DisplayHighlights`); as exportações de radiação e refletância nunca são afetadas. Não é possível recuperar dados do sensor que sofreram clipping — reduza a exposição para isso. A sobreposição zebra acompanha a imagem escurecida. Desativado com o perfil de cor Raw (“Não aplicado com o perfil de cor Raw.”). |
| **Saturação** | 0–200 (100 = neutro) | 100 | Somente RGB | Saturação HSV na pré-visualização ao vivo. |
| **Contraste** | 0–200 (100 = neutro) | 100 | Somente RGB | Contraste linear em torno do cinza médio na visualização ao vivo. |
| **Espelhar H / Espelhar V** | Caixas de seleção | Desativado | Todas | Inverte a visualização na horizontal / na vertical. |
| **Rotação**| 0° / 90° / 180° / 270° | 0° | Todas | Gira a visualização. A orientação é aplicada no final da cadeia de pré-visualização do backend.**Capturas salvas (1.3.0):**os produtos Pré-visualização, Radiação, Refletância e Índice de uma câmera autônoma são girados por Espelhamento/Rotação, enquanto**os arquivos Raw e Debayered permanecem na orientação nativa da câmera**; cada arquivo de captura registra a configuração em seu XMP, de modo que o processamento posterior de um arquivo raw a aplica da mesma forma.**Os arquivos membros da matriz permanecem na orientação nativa** (a configuração é registrada, não aplicada), e as visualizações compostas da matriz a ignoram. |

**Semântica dos perfis de cor** (câmeras RGB):

* **Raw** — ignora completamente a cadeia de processamento.
* **Linear** — sinal escuro + campo plano + balanço de branco; sem matriz de cor, sem gama.
* **Natural** *(padrão)* — linear mais a matriz de correção de cor medida e uma curva de tom adaptativa à cena.
* **Aprimorado**— Natural mais vibração e contraste local CLAHE. O custo extra se aplica**apenas à visualização ao vivo** — as capturas salvas sempre recebem o acabamento completo, independentemente do perfil.
* **Temperatura personalizada** — Natural com o equilíbrio de branco fixado no valor em Kelvin escolhido por você.

{% hint style="warning" %}
Para os modos Natural, Aprimorado e Temperatura personalizada, o painel exibe uma observação sobre o tom: os quadros são clareados de acordo com sua própria cena, portanto, as imagens *exibidas* salvas não são comparáveis entre si. **Exporte a radiação ou a refletância para medições.**
{% endhint %}

### Sobreposições de exibição (desenhadas sobre a transmissão ao vivo)

Elas são exclusivas do front-end — são sobrepostas ao vídeo, sem nunca afetar a transmissão ou as capturas.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sobreposição | Controles | Padrão | O que faz |
| --- | --- | --- | --- |
| **Zebra** | Caixa de seleção + limite 200–255 | Desativado / 250 | Listras diagonais magenta nos pixels cortados. |
| **Mira** | Caixa de seleção | Desativado | Marca no centro do quadro. |
| **Grade** | Desativado / 3 × 3 / 9 × 9 | Desativado | Grade de composição. |
| **Histograma** | Caixa de seleção + largura de 0,10–0,90 do quadro | Desativado / 0,25 | Uma faixa de histograma no feed. |
| **Pico de foco** | Caixa de seleção + limite 20–200 + amostra de cor | Desativado / 80 / `#ff5722` | Destaque de bordas Sobel para foco. |
| **Divisões de canal** | "Mostrar divisões (Red / Green / NIR)" / botão “Ocultar divisões” | Oculto | Adiciona três blocos independentes em escala de cinza, um por canal, ao lado da imagem composta (o rótulo do botão segue os canais de filtro da câmera). Cada bloco dividido pode ser arrastado e compartilha a cor da borda da câmera. Não disponível em câmeras monocromáticas. Salvo com o projeto. |

### Medidor pontual

* **Caixa de seleção “Clique para amostrar”**: clique na imagem ao vivo para amostrar um único pixel (uma cruz marca o ponto), ou clique e arraste uma região para obter uma média de pixels.**Limpar**apaga a amostra e a cruz. É mutuamente exclusivo com o modo**Aim** do AE-ROI.
* **Menu suspenso**Mostrar**:**Raw (profundidade de bits)**— valores digitais nativos na profundidade de bits do sensor (por exemplo, 12 bits → 0..4095) — ou**Exibição (8 bits)** (padrão). Quando um índice em tempo real está ativo, a opção Exibição mostra o valor do índice calculado (por exemplo, NDVI) em vez disso.
* O painel de leitura lista coordenadas de pixels, tamanho do quadro, formato de pixel, profundidade de bits e uma tabela de canais (Canal / Valor / %) com rótulos de banda e comprimentos de onda; os pares verdes do Bayer são calculados como média; as amostras da região exibem “Média de N px”.

O estado do medidor pontual é válido apenas para a sessão.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposição automática preditiva (controlada por DLS)

Esta seção só aparece quando **pelo menos um sensor de luz DAQ estiver conectado** — o solucionador precisa de um espectro de radiação descendente em tempo real para operá-la.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->



| Controle | Faixa | Padrão | O que faz |
| --- | --- | --- | --- |
| **Ativar** | Caixa de seleção | Ligado (câmeras autônomas) | Um solucionador de forma fechada usa o espectro do DLS mais os escalares do pacote de calibração da câmera para ajustar a faixa mais clara perto da saturação, mantendo a faixa mais escura acima do limite mínimo de SNR — uma única gravação de exposição por resolução, sem ciclo de estabilização. Projetado para timelapse com energia solar, onde cada foto deve ter a exposição correta. O backend recorre silenciosamente ao AE reativo sempre que a leitura do DLS estiver desatualizada/ausente ou o pacote de calibração não estiver carregado. |
| **Suavização (α)** | 0,05–1,0, passo de 0,05 | 0,3 | Suavização de soluções preditivas sucessivas (valor mais baixo = suavização maior). |
| **Refletância da cena**| Botão**Recalibrar ρ** | — | Reestima o fator de refletância da cena usado pelo solucionador. |

{% hint style="info" %}
**A conexão em matriz desativa a AE preditiva por padrão** — para matrizes, a AE inteligente do Chloros, juntamente com a exposição automática do lado da câmera, controla a exposição (com proteção contra saturação), e a estimativa de refletância de cena única da AE preditivanão é segura em cenas mistas. Você pode reativá-la por câmera aqui se desejar especificamente uma exposição radiométrica orientada por DLS.
{% endhint %}

**Limite máximo de exposição orientado por DAQ e AE fixada na luz incidente.** Independentemente da caixa de seleção acima, quando um sensor de luz DAQ é atribuído a uma câmera RGB, o Chloros calcula — a partir da irradiância descendente absoluta medida — a exposição máxima × ganho na qual uma superfície com 100% de refletância permanece abaixo do clipping e aplica isso como um **limite**na exposição automática. Desde a versão 1.3.0, o limite se aplica apenas enquanto um alvo de calibração estiver no campo de visão daquela câmera (visível em seu quadro nos últimos 10 s); sem nenhum alvo no campo de visão, a exposição automática normal da câmera, com sua própria proteção contra realces, é acionada, de modo que o restante da cena não fique mais escuro do que o necessário. Enquanto o limite máximo estiver ativo, a câmera fica**fixada na luz incidente**: ela opera em malha aberta com a exposição medida pela luz incidente e ganho em 0 dB — a exposição segue a luz medida, não o conteúdo da cena. Como o limite máximo só pode reduzir a exposição, ele não pode, por si só, causar clipping. Desde a versão 1.3.0, o limite máximo é calculado com a constante de exposição da própria câmera, proveniente de seu pacote de calibração (um valor médio da frota apenas quando o pacote não contém um). O limite máximo é desativado automaticamente — e a exposição automática (AE) normal da cena é retomada — sempre que nenhum alvo de calibração estiver no campo de visão, a leitura do DAQ estiver ausente, desatualizada (>30 s), ou escura, ou se ≥15 % do quadro sofrer clipping na exposição fixada (o que significa que o sensor e a câmera estão captando iluminação diferente). Não há opção na interface gráfica; esse é o comportamento padrão sempre que uma câmera RGB tem um DAQ vinculado.

### Aquisição e Acionamento

Os membros da<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

matriz exibem adicionalmente as linhas de **Função**(somente para leitura; Mestre em azul / Escravo em verde),**Linha de Sincronização**e**Parceiros**.

| Controle | Opções | Padrão | Notas |
| --- | --- | --- | --- |
| **Modo de acionamento** | Desligado / Ligado | Ligado | Desativado para membros da matriz (a matriz gerencia o acionamento). |
| **Fonte de acionamento** | Software / Linha 0 (M8) / Linha 1 / Linha 2 | Linha 0 | Oculto enquanto o Modo de acionamento estiver desativado; desativado para membros da matriz. A Linha 0 é a entrada de acionamento externa optoisolada M8. |
| **Taxa de quadros**| Automático / Manual + valor | Automático |**Automático**: o limite de taxa de quadros da câmera está desativado — a exposição determina os fps, e a caixa exibe a taxa real em tempo real.**Manual**: você define o limite de fps com um controle deslizante (de 1 até o máximo permitido pela largura de banda), com base na taxa real atual. Os membros da matriz veem um valor somente leitura “N fps (ao vivo)” com a indicação “Definido nas configurações da matriz”. |

### Rede / Transporte

| Coluna | Comportamento |
| --- | --- |
| **Tamanho do pacote**| 1500 (Padrão) / 9000 (Jumbo) — padrão**Jumbo**. |
| **Taxa de transferência** | Limite de taxa de transferência do link, somente leitura, em MB/s. O back-end reequilibra esse valor entre todas as câmeras conectadas a cada conexão/desconexão. |
| **Gerenciamento de buffer** | Modo de gerenciamento de buffer, somente leitura. |

### Captura

O painel termina com um botão **“Abrir configurações de captura…”** que leva ao [painel Configurações de captura](capture.md#the-capture-settings-pane) (desativado até que um projeto seja aberto — “Crie ou abra um projeto para salvar as capturas”). Se a câmera estiver oculta ou em pausa, uma dica lembra que você deve exibi-la/retomá-la antes de capturar.

## Painel de configurações da matriz

Abra clicando na **engrenagem**em uma linha de ARRAY. Cabeçalho: nome do array com um lápis para renomear e**×** para fechar. As seções abaixo marcadas como *somente combinadas* aparecem apenas para arrays conectados no modo de exibição combinada.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->



### Sincronização

Linhas **Mestre**,**Escravos**e**Linha de Sincronização** somente para leitura.

### Sensor de luz ambiente

Exibido tanto para matrizes combinadas quanto separadas:

* Menu suspenso **Sensor de luz**— vincula um DAQ a toda a matriz. A seleção é aplicada imediatamente, propaga-se para o menu suspenso**Sensor de luz** de cada câmera membro (você ainda pode substituir a configuração por câmera) e começa a encaminhar espectros para a matriz.
* Linha **Status** em tempo real: Desligado · “Aguardando o primeiro espectro…” · “Ativo — todas as câmeras na matriz estão com correção de iluminação” · “Nenhum espectro novo nos últimos 3 s — ainda usando a última leitura (sem tempo limite para dados obsoletos)…”.
* Nota no painel: “Correção radiométrica em toda a matriz. As configurações por câmera substituem essa configuração.”

### Alvo de calibração

Seu próprio grupo logo após o Sensor de Luz Ambiente, tanto para matrizes combinadas quanto separadas:

* Caixa de seleção **Ativar** — “Detectar alvo ArUco do MAPIR e validar NDVI versus LUT de refletância do painel”; controla a sobreposição do alvo e as tabelas de validação do bloco combinado. Em uma matriz **separada**, ela ativa a [tabela de alvo de calibração](#the-calibration-target-table) de cada membro (uma por linha de câmera na visualização em lista).

### Captura — configurações uniformes do sensor

Exibido tanto para matrizes combinadas quanto separadas (o cabeçalho do grupo diz “Captura (uniforme entre as câmeras)”). Essas configurações se aplicam uniformemente a todos os membros (alterações por membro quebrariam a sincronização). As edições são preparadas e aplicadas juntas com **Aplicar**— exceto o botão de alternância**ROI ativado / ROI desativado**, que se aplica imediatamente.

<!-- SCREENSHOT-NEEDED: array settings Capture section (1.3.0) — Pixel Format, Binning, Resolution preset, the ROI crop row with the "ROI on" toggle, the "Select" button, the "max WxH" hint and Reset button, the W/H/X/Y fields, the full-sensor ROI map below them with the orange crop box and its "W×H @ X,Y" label above the box, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons. -->

| Controle | Opções / intervalo | O que faz |
| --- | --- | --- |
| **Formato de pixel** | 8 bits / 10 bits / 12 bits / 16 bits | Profundidade de bits uniforme para todos os membros; cada câmera adota o formato dessa profundidade para seu próprio sensor. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning por hardware — mantém o campo de visão completo enquanto aumenta a relação sinal-ruído (SNR) e a taxa de quadros. Alterá-lo redefine os campos de ROI para o novo campo de visão completo. |
| Predefinição de **resolução** | Completa / Metade / Um quarto | Relativa ao binning; preenche os campos de ROI com um recorte centralizado. |
| **Recorte de ROI (px)**| Botão de alternância**ROI ativado / desativado**, botão**Selecionar**, campos L / A / X / Y, mapa do sensor completo | Recorte do sensor — consulte [O editor de ROI](#the-roi-editor). Largura/altura se ajustam a múltiplos de 16 (mínimo 64); os deslocamentos se ajustam a múltiplos de 4. Uma dica “max WxH” mostra o limite máximo e**Redefinir** retorna os campos ao campo de visão completo. Durante a edição, a caixa de recorte laranja acompanha todas as alterações nos campos no mapa e no bloco da matriz. |
| **Taxa de disparo**| Alternância Automático / Manual + fps 0,5–10, passo de 0,5 |**Automático**(padrão): o backend calcula a taxa de disparo com base na resolução e na largura de banda — a entrada fica desativada e exibe o valor calculado.**Manual**: fixa seu valor ao clicar em**Aplicar**. Uma taxa fixada que um novo tamanho de quadro não consiga suportar é mantida na taxa mais rápida que esse tamanho permite e volta ao normal quando o tamanho do quadro permitir novamente. |

Observação no painel: “Alterações de formato/resolução reiniciam todas as câmeras brevemente. A taxa de disparo é aplicada em tempo real.” Os botões **Aplicar / Cancelar** ficam na parte inferior do painel.

#### O editor de ROI

Novidade na versão 1.3.0. A linha de recorte de ROI e o mapa abaixo dela funcionam em conjunto:

* **ROI ativado / ROI desativado**— ativa ou desativa o recorte do sensor**imediatamente**(as câmeras reiniciam brevemente; o bloco da matriz exibe “Aplicando configurações…”).**Desativado**lê o sensor completo e**lembra do seu recorte**— os campos continuam exibindo-o;**Ativado** o aplica novamente. Ativar a ROI sem nenhum recorte definido aplica um recorte centralizado de metade do tamanho (“Nenhum recorte foi definido, portanto, um recorte centralizado de metade do tamanho foi aplicado. Ajuste-o com os campos.”).
* **Selecionar**— desenha o recorte no mapa: arraste pelo mapa ou clique em dois cantos opostos (Esc cancela). O retângulo é registrado nos campos L/A/X/Y, alinhado às etapas acima; nada é aplicado até você pressionar**Aplicar**, para que você possa ajustá-lo com precisão usando as setas dos campos primeiro. Um retângulo por vez ao pressionar Selecionar.
* **O mapa**mostra o recorte em todo o sensor, para que você possa ampliá-lo novamente: “Sensor completo, ao vivo” enquanto as câmeras leem o sensor completo, ou “Instantâneo em quadro completo de {hora}. As câmeras leem apenas o recorte.” enquanto estiverem recortadas. Se o conjunto de câmeras só tiver transmitido imagens recortadas, o mapa exibirá “Ainda sem visualização em quadro completo: este conjunto de câmeras só transmitiu imagens recortadas.” com um botão**Mostrar quadro completo**, que desativa a ROI. O rótulo `W×H @ X,Y` do recorte fica fora da caixa (acima dela ou abaixo, quando a caixa toca a parte superior), no mapa e no bloco ao vivo.
* **Aplicar**confirma os campos: um recorte ativa a ROI; o quadro completo (após**Redefinir**) a desativa, mantendo o recorte anterior memorizado; uma ROI que está desativada e cujos campos ainda mostram o recorte memorizado permanece desativada quando você aplica outra alteração (formato, binning, taxa).
* **Em um conjunto alinhado, a janela de recorte de cada membro é posicionada pelo alinhamento**, de modo que todas as câmeras enquadram a mesma área da cena. Um conjunto recortado antes mesmo de ser alinhado é alinhado primeiro no sensor completo (o recorte é removido, o alinhamento é resolvido e o recorte é restaurado com janelas específicas para cada câmera).
* Após uma alteração no recorte, a exposição automática se reajusta para o novo enquadramento. Sob um recorte, a exposição de um membro ainda pode durar até o período de disparo.
* O estado de ativação/desativação da ROI e o recorte memorizado são salvos com o projeto.

### Alinhamento (co-registro)

Exibido para **matrizes combinadas e separadas** (a versão 1.2.0 exibia apenas para matrizes combinadas); o cabeçalho do grupo exibe “Alinhamento”.

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* Caixa **Residual**: “RMS x,xx px” — verde se for inferior a 1 px, âmbar se for inferior a 3 px, vermelho nos demais casos ou se alguma câmera falhar; “sem perfil” antes da primeira resolução.
* Linha de resumo: “✓ Todas as câmeras alinhadas (N)” / “⚠ p/N câmeras alinhadas — <serial (filter)="">falha” / “Recorte ativo — Recalibrar para alinhar (usa o sensor completo)” / “Aguardando a estabilização da exposição…” / “Aplicando configurações…” (as câmeras estão reiniciando) / “Matriz parou de enviar quadros — reconecte a matriz”.
* Tabela por câmera: câmera (últimos 4 dígitos do número de série + filtro), erro de reprojeção em px com contagem de correspondências (“ref” para a mestre) e a pontuação de correlação cruzada normalizada da sobreposição em relação ao limite mínimo de aprovação de 0,35.
* **Recalibrar alinhamento** (exibe “Calibrar alinhamento” antes do primeiro perfil) — reexecuta o co-registro em quadros novos.
* **Caixa de seleção “Exposição automática das câmeras para alinhamento”** (marcada por padrão) — clareia temporariamente câmeras escuras ou sem contraste (primeiro a exposição, depois o ganho) para que apresentem textura para correspondência e, em seguida, restaura a exposição automática.

O Chloros alinha cada matriz automaticamente após a conexão, em ambos os modos de exibição, assim que a exposição automática dos membros se estabilizar (ele continua tentando por até 5 minutos); a pré-visualização combinada também é alinhada ao ser aberta. Uma matriz separada não possui bloco composto; portanto, esta seção verifica o estado do alinhamento e executa a recalibração por conta própria. Recalibre se o foco ou a profundidade da cena tiverem mudado. O alinhamento é **projetado para valer apenas para a sessão** — ele nunca é salvo em um perfil, pois depende da distância da cena naquele momento. As capturas ainda podem ser exportadas com registro de pixels (consulte [Exportações alinhadas](capture.md#per-array-controls)).

### Vinheta Inteligente

* Caixa de seleção **Ativar correção**— aplica a estimativa de vinheta por câmera à cadeia radiométrica (em tempo real**e** nas exportações).
* **Calibrar a partir da visualização atual**— aponte a matriz primeiro para um alvo uniforme (painel plano, parede ou céu); cada câmera é nivelada individualmente e o status informa “n/N câmeras · −x,x %” de ganho de nivelamento.**Limpar** remove a estimativa.
* Status: “Calibrado”, “Não calibrado” ou “Parcialmente calibrado — execute novamente” quando apenas as estimativas de alguns membros ainda se aplicam ao enquadramento atual (por exemplo, após uma “Recalibração” ter movido a janela de recorte de um membro) — execute a “Calibração” novamente.
* Faça o ajuste fino por câmera com o controle deslizante **Vignette** específico para cada câmera na [Pré-visualização ao vivo](#live-preview).

### Visualização ao vivo

* Caixa de seleção **Histograma**+ controle deslizante de largura (0,10–0,90 do quadro) — a faixa de histograma na transmissão ao vivo em todos os blocos de cada membro de uma só vez (incluindo blocos de canais divididos), a [sobreposição de histograma](#display-overlays-drawn-over-the-live-feed) definida para todo o conjunto. Ela aparece como ativada quando todos os membros já a possuem. Em um conjunto**separado**, esta é a única linha do grupo.
* *Apenas combinado:* **Índice**: caixa de seleção “Ativar” + ícone de engrenagem — abre a [Calculadora de Índice](#index-calculator-pane) compartilhada com faixas traçadas a partir de**todas** as câmeras dos membros. Uma linha de visualização da expressão abaixo mostra a expressão atual (“Nenhuma expressão definida — abra a calculadora para criar uma”), atualizada a cada segundo.
* *Apenas combinado:* Menu suspenso **Resolução de renderização**(mesmas predefinições que por câmera, padrão 720p): a altura do stream de visualização ao vivo**e** o tamanho da exportação composta salva. Observação no painel: “Pré-visualização + tamanho da composição salva. Imagens por câmera sempre são exportadas em resolução total.”

### Camadas de exibição *(somente no modo combinado)** Caixa de seleção **Ativar** (padrão desativado — a câmera principal é exibida diretamente; ativada = composição em camadas).
* **Menus suspensos**Primeiro plano**/**Fundo**: cada câmera membro (por nome) ou**Índice**. Quando o Primeiro plano é Índice, os pixels fora do Min/Max da LUT exibem a camada de Fundo.

### Visualização dividida *(somente combinada)*

**“Mostrar câmeras membros”**— um botão**Dividir / Ocultar câmeras membros** que adiciona a transmissão ao vivo de cada câmera membro como blocos separados na grade ao lado da composição. Os blocos leem o buffer de quadros existente da matriz (sem conexão extra de câmera). Apenas na visualização em grade; salvo por matriz junto com o projeto.

### Recursos

Um painel somente leitura atualizado a cada 5 s:

* **Rótulo de nível**: “Captura simultânea” (verde) · “Captura simultânea (emissão escalonada FTD)” (verde) · “Captura escalonada (desvio de 100 ms)” (âmbar) · “Configuração muito grande” (vermelho).
* **Integridade do quadro**: “x,xx % incompleto” — verde abaixo de 1 %, âmbar abaixo de 5 %, vermelho a partir de 5 %.
* **Linha de link**: “NIC {mbps} Mbps - sustentada {MB/s} MB/s”.

Este é o orçamento de largura de banda em tempo real do array. Para obter informações sobre o modelo de fps e de rede subjacente — e o que alterar quando o nível ficar âmbar ou vermelho — consulte [Matrizes com várias câmeras](arrays.md) e a [Referência da CLI](../reference/cli-reference.md).

<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->

## Painel Calculadora de Índice

A terceira página da barra lateral, compartilhada entre a engrenagem de Índice por câmera e a engrenagem de Índice da matriz combinada (uma de cada vez — o título exibe “Calculadora de Índice — <camera name="">” ou “Calculadora de Índice — <array name="">”). Ela recebe a lista de faixas (as faixas naturais do filtro da câmera— bandas naturais ou todas as bandas dos membros da matriz), a expressão atual e a configuração da LUT (ativada/desativada, nível — padrão 7 stops, mínimo — padrão 0,2, máximo — padrão 1), além de um histograma de índice em tempo real. **Aplicar** confirma a expressão; as alterações na LUT são aplicadas em tempo real à visualização.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Configurações por câmera vs. gerenciadas pela matriz

Referência rápida sobre o que fica onde quando uma câmera é membro de uma matriz:

| Gerenciado pela matriz (somente leitura no painel da câmera) | Ainda por câmera dentro de uma matriz |
| --- | --- |
| Formato de pixel, resolução, binning | Exposição automática (exposição, ganho, alvo, suavização, ROI) |
| Modo/fonte de disparo, taxa de quadros | Redução de ruído, nitidez, vinheta |
| | Orientação (espelhamento/rotação), sobreposições de tela (configurações da matriz → a Visualização ao Vivo define o histograma em todos os membros de uma só vez), medidor pontual |
| | Índice (matrizes com exibição separada), vinculação do sensor de luz |

Outros comportamentos transversais:

* **Exibição combinada vs. separada** é escolhida na conexão da matriz: combinada = um bloco composto alinhado (as câmeras transmitem apenas via Visualização Dividida); separada = cada câmera renderiza seu próprio bloco sincronizado. Uma câmera nunca exibe simultaneamente uma transmissão independente e um bloco da matriz.
* **Reconexão automática**: abrir um projeto salvo restaura suas câmeras e matrizes e reaplica todas as configurações salvas ao backend antes que as transmissões sejam retomadas.
* **Controle de captura**: câmeras ocultas ou pausadas são excluídas da opção “Capturar tudo”; uma matriz é totalmente bloqueada somente quando TODOS os membros estiverem ocultos/pausados. Consulte [Configurações e modos de captura](capture.md).

## Como as configurações são mantidas

O estado da aba da câmera é salvo **com o projeto**, não no navegador:

* Cada alteração reativa captura instantâneos das câmeras e matrizes no `cameras.json` do projeto (com tempo de espera de 500 ms). Isso inclui nomes e cores das câmeras, configurações de exposição/ganho/AE, formato de pixel/resolução/binning, taxa de disparo, configurações de visualização (resolução de renderização, redução de ruído, nitidez, vinheta, perfil de cor, realces, saturação/contraste), orientação, sobreposições, divisões de canais, configuração de índice, configurações de AE preditivo, ROI de AE, nomes de matrizes, modo de exibição, configurações de captura de matrizes (incluindo o recorte da ROI, se a ROI está ativada e o recorte memorizado quando ela está desativada), o Wire Budget da matriz e o bloco de grade (zoom de feed, modo de visualização, bloqueio de grade, ordem manual de blocos, câmeras ocultas, blocos fechados, câmera ativa).
* As configurações dos sensores de luz são salvas no arquivo `sensors.json` do projeto.
* Ao reabrir o projeto, o hardware é reconectado e todas as configurações são reaplicadas.
* **Sem projeto aberto = apenas sessão**: sem projeto, nada sobrevive ao fechamento do Chloros.
* Apenas para a sessão, independentemente do projeto: estado de pausa, amostras do medidor pontual, a caixa de seleção “Alvo de calibração” por câmera (sempre desmarcada por padrão) e o perfil de alinhamento da matriz (recalculado por sessão, conforme o projeto).
* Uma exceção: as seleções de exportação das **Configurações de Captura** e o modo de captura persistem por projeto no armazenamento local do aplicativo, em vez de no `cameras.json` — consulte [Configurações e Modos de Captura](capture.md).</array></camera></serial>
