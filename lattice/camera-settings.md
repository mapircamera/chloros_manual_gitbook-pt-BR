# Configurações da câmera

A guia **Câmeras**é a superfície de controle em tempo real do Chloros para câmeras LATTICE: uma área de exibição principal que mostra todas as câmeras conectadas como blocos em tempo real e uma barra lateral que alterna entre três páginas — a**lista de câmeras**, um**painel de configurações**(configurações por câmera, de matriz ou de captura — uma por vez) e a**Calculadora de Índice**. Esta página documenta todos os controles da lista de câmeras, do painel de configurações por câmera e do painel de configurações de matriz. Os modos de captura, a seleção do tipo de exportação e o fluxo “Capturar Tudo” estão na página complementar [Configurações e Modos de Captura](capture.md).

A guia “Câmeras” aparece na barra lateral assim que o backend Chloros estiver pronto. Todos os controles abaixo se comunicam com o backend local por meio do `127.0.0.1:5000`; as alterações se aplicam imediatamente à câmera ao vivo, salvo indicação em contrário.

## Tipos de câmera utilizados nesta página

Os controles são exibidos ou ocultados de acordo com o tipo de câmera selecionado. O manual utiliza estes termos ao longo do texto:

| Termo | Significado | Canais do filtro |
| --- | --- | --- |
| **Câmera RGB** | LATTICE M3C com filtro FRGB (o modelo contém `-FRGB`) | Red / Green / Blue |
| **Multiespectral Bayer** | LATTICE M3C com FRGN, FOCN, ou FNGB | FRGN: Red / Green / NIR · FOCN: Orange / Cyan / NIR · FNGB: NIR / Green / Blue |
| **Mono (M3M)** | LATTICE M3M — um filtro de banda estreita, uma banda calibrada | Banda única |
| **Membro da matriz** | Uma câmera conectada como parte de uma matriz sincronizada (exibição combinada ou separada) | Conforme seu filtro |

As câmeras RGB passam por processamento fotométrico (equilíbrio de branco, perfis de cor, gama); câmeras multiespectrais e monocromáticas passam pela cadeia radiométrica e ignoram os controles fotométricos. Os membros da matriz transferem as configurações no nível do fluxo (formato de pixel, resolução, binning, acionamento, taxa de quadros) para a matriz — essas linhas passam a ser somente de leitura no painel de cada câmera e são transferidas para o painel de configurações da matriz.

## A área

<!-- SCREENSHOT-NEEDED: Cameras tab with 2+ cameras connected in grid view — live tiles visible with name and fps overlays, sidebar camera list open on the right. -->

principal de feed Sem câmeras conectadas, a área de feed exibe uma tela inicial com a mensagem **“Conecte uma câmera para começar”**, com dois botões:**Conectar Câmera**(verde, abre a caixa de diálogo de conexão de uma única câmera) e**Conectar Matriz** (azul, abre a caixa de diálogo de conexão do conjunto). As próprias caixas de diálogo de conexão estão documentadas em [Conectando Câmeras](connecting.md); conceitos de conjuntos (sincronização, camadas, largura de banda) em [Arrays de várias câmeras](arrays.md). Ao abrir um projeto salvo que contenha câmeras, a tela inicial exibe um indicador de carregamento com a mensagem “Reabrindo N câmeras salvas…” enquanto o Chloros restaura os fluxos da última sessão.

<!-- SCREENSHOT-NEEDED: Cameras tab empty state — the "Connect a camera to get started" splash with the green Connect Camera and blue Connect Array buttons. -->

### Barra superior

| Controle | O que faz |
| --- | --- |
| **Alternar modo de exibição**| Alterna entre a**visualização em grade**(todos os blocos como células) e a**visualização em lista** (matrizes em largura total na parte superior, UMA câmera ativa abaixo). Dicas de ferramenta: “Alternar para visualização em grade” / “Alternar para a visualização em lista”. |
| **Bloqueio da grade**(cadeado) | Padrão**bloqueado** — blocos fixos no lugar. Desbloqueie para arrastar e reordenar os blocos em qualquer posição (os espaços vazios são preservados). A grade se bloqueia automaticamente sempre que uma nova câmera se conecta. Dicas de ferramenta: “Desbloquear grade (ativar arrastar de blocos)” / “Bloquear grade (congelar blocos no lugar)”. |
| Controle deslizante **Zoom do feed** | Tamanho do bloco, de 60 px até a largura total do contêiner. As células mantêm a proporção 4:3. Abaixo de 200 px de largura de célula, as sobreposições de nome e fps são ocultadas para manter o bloco limpo. |

### Blocos do feed

Cada câmera renderiza um bloco composto ao vivo; uma câmera pode, além disso, exibir três blocos em escala de cinza com **divisão por canal** (consulte [Divisões de canal](#display-overlays-drawn-over-the-live-feed)), e os conjuntos de câmeras renderizam um bloco combinado. O bloco ativo exibe um anel de seleção na cor da câmera (ou do conjunto de câmeras).

Ao passar o mouse sobre um bloco, um botão de fechamento **X** é exibido:

* Fechar um bloco **composto** enquanto suas divisões de canal permanecem visíveis apenas oculta o bloco composto.
* Fechar o **último bloco visível de uma câmera independente** desconecta essa câmera.
* **Blocos divididos de membros de uma matriz combinada nunca desconectam** a câmera — eles apenas a ocultam.

Com a grade desbloqueada, arraste qualquer bloco para qualquer slot; o layout é salvo com o projeto.

## Barra lateral — lista

<!-- SCREENSHOT-NEEDED: sidebar camera list pane showing a standalone camera row and an ARRAY group with indented member rows, the DAQ on/off pill visible on the array row, plus the Connect Camera / Connect Array / Capture All buttons at the top. -->

de câmeras A primeira página da barra lateral lista todas as câmeras e matrizes conectadas:

* **Conectar Câmera**(verde) /**Conectar Matriz** (azul, exibe “Detectando...” durante a varredura). Ambos ficam desativados enquanto uma caixa de diálogo de conexão estiver aberta.
* **Capturar tudo**(vermelho) — captura todas as câmeras listadas com os tipos de exportação escolhidos nas**Configurações de captura**. Requer um projeto aberto. Totalmente documentado em [Configurações e modos de captura](capture.md).
* **Engrenagem de Configurações de Captura** (ao lado de “Capturar tudo”) — abre o [painel de Configurações de Captura](capture.md#the-capture-settings-pane). Desativado sem um projeto ou durante a captura.

### Linhas de câmera

Cada linha de câmera exibe uma borda codificada por cores (a cor personalizada da câmera), um rótulo “CAM” — com uma letra azul **M**(mestre) ou verde**S** (escravo) indicando a função dos membros da matriz — e o nome de exibição. O nome padrão é `LATTICE-MODEL (serial)`; renomeie-o no painel de configurações de cada câmera. Botões da linha:

| Botão | Efeito |
| --- | --- |
| **Olho**| Alternar visibilidade. Câmeras ocultas saem da grade e são**excluídas da opção “Capturar tudo”**. |
| **Engrenagem** | Abre o painel de configurações específicas de cada câmera (próxima seção). |
| **Pausar / Reproduzir**| Congela a pré-visualização ao vivo**apenas no lado da exibição** — a captura no backend continua em execução. Câmeras pausadas não podem capturar. |
| **X** | Desconectar. A interface do usuário é atualizada imediatamente (estimativa otimista); a desconexão no back-end pode levar de 10 a 30 s para ser concluída. |

### Linhas da matriz

Uma linha da matriz exibe um ícone “ARRAY” na cor da matriz, o nome da matriz (que pode ser renomeado nas configurações da matriz) e um botão **DAQ · ligado/desligado**—**ligado** quando o sensor de luz no nível da matriz está configurado *ou* qualquer membro possui um sensor por câmera; sua dica de ferramenta lista exatamente qual sensor alimenta o quê. As câmeras membros são listadas recuadas abaixo, em suas próprias linhas. Botões da linha da matriz: **olho**(oculta/exibe TODOS os membros juntos),**engrenagem**(painel de configurações da matriz),**X**(desconecta a matriz inteira).

O status do sensor de luz (DLS) usado nas linhas da matriz e no painel de configurações da matriz tem quatro estados:**desligado**,**em espera**(ainda sem espectro),**ativo**(um espectro chegou nos últimos 3 s) e**desatualizado** — nenhum espectro novo nos últimos 3 s, mas a última leitura *ainda é usada* (as leituras do DAQ nunca expiram no caminho de captura).

Você pode arrastar câmeras independentes e grupos inteiros de matrizes uns sobre os outros na barra lateral para reordenar a lista; os membros da matriz não podem ser arrastados individualmente.

## Painel de configurações por câmera

Abra clicando na **engrenagem** na linha de uma câmera. O painel desliza sobre a lista de câmeras.

<!-- SCREENSHOT-NEEDED: per-camera settings pane, top portion — header with color swatch, camera name, rename pencil and close X; live histogram with the orange dashed AE-target line and green mean-luma line; the RGB per-band toggle button visible top-right of the histogram. -->

**Cabeçalho**: a**amostra de cor**da câmera (clique para abrir um seletor de cores nativo — define a cor da borda da barra lateral e do anel de seleção de blocos), o**nome**com um botão de lápis**Renomear**(salvar um nome vazio reverte para o padrão `MODEL (serial)`)e**×** para fechar.

### Histograma ao vivo

Na parte superior do painel há um histograma de luma ao vivo, calculado a partir da pré-visualização JPEG a ~8 Hz. A média é ponderada por Bayer — (R+2G+B)/4 — para corresponder à medição de AE da própria câmera.

* **Linha tracejada Orange**= a meta de AE.**Arraste-a horizontalmente para redefinir a meta** — um comando é enviado ao soltar, e o arrastar alterna o modo da meta de AE para Manual.
* **Linha contínua do Green** = a média real de luma (o que o AE está fornecendo no momento).
* **Botão RGB** (canto superior direito): alterna entre histogramas sobrepostos por banda, coloridos pelo filtro da câmera (por exemplo, em FRGN: cinza NIR, verde, vermelho). Em câmeras monocromáticas (M3M), o botão exibe “MONO” e fica desativado — o modo monocromático sempre mostra o histograma de luminância de banda única.
* Os rótulos do eixo X seguem a profundidade de bits do sensor do formato de pixel atual: 0..255, 0..1023, 0..4095 ou 0..65535.

### Linhas

<!-- SCREENSHOT-NEEDED: per-camera settings info rows — Model, Radiometric Calibration "Active" badge with the tier/sha/date caption, Calibration Report Download button, Serial, Firmware row showing the "Up to date" state, IP, Temperature readout, Calibration Target checkbox, Light Sensor dropdown. -->

de informações da câmera | Linha | Comportamento |
| --- | --- |
| **Modelo** | Somente leitura (por exemplo, `LATT-M3C-L87-FRGN`). |
| **Calibração radiométrica**| Emblema**“Ativo”**do Green com uma legenda que mostra o nível de calibração, o hash, a data da calibração e a lista de bandas, carregados a partir do pacote de calibração da câmera (consulte [Calibração radiométrica de fábrica](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration)).**Oculto para câmeras RGB** — elas possuem uma calibração fotométrica de balanço de branco, e não de radiação por banda. |
| **Relatório de calibração**| Botão**Baixar** — abre o PDF do certificado de calibração NIST por número de série da câmera no visualizador do seu sistema operacional. Se o certificado ainda não estiver armazenado em cache, o Chloros exibe uma dica em seu lugar. |
| **Número de série** | Somente leitura. |
| **Firmware**| Mostra a versão atual e, em seguida, determina a versão disponível para este modelo (armazenada em cache por modelo — um conjunto de N câmeras verifica o servidor uma única vez). Exibe: “Verificando…” → Botão**“Atualizar para X”**→ “Atualizando…” → “Atualizado de A para B” / “Falha: …” / “Ignorado: …” / verde**“Atualizado”**. Dica do botão de atualização: “Redefinição de fábrica + atualização de firmware + reprogramação do UserSet1. ~2–3 minutos; não desconecte.” |
| **IP** | Somente leitura. |
| **Temperatura** | Somente leitura, atualizada a cada 3 s. Fica laranja a partir de ≥65 °C e vermelha com um ⚠ a partir de ≥75 °C. |
| Caixa de seleção **Meta de calibração** | Ativa a detecção da meta de refletância ArUco com uma tabela de validação NDVI por painel abaixo da transmissão ao vivo (visualização em lista). Apenas para a sessão — sempre abre desativada. |
| Menu suspenso **Sensor de luz** | Vincula um sensor de luz DAQ (DAQ-E/M/U, da lista da guia Sensores de luz) a esta câmera para correção de iluminação por luz descendente (DLS) e exposição automática preditiva. “Nenhum” cancela a vinculação. Se nenhum sensor estiver conectado, o menu suspenso exibe “(nenhum sensor conectado — abra a guia DAQ)”. A vinculação é salva com o projeto. |

### Exposição e ganho

<!-- SCREENSHOT-NEEDED: per-camera Exposure & Gain section — Exposure (us) and Gain (dB) rows with Auto/Manual toggles, AE Target Brightness, AE Smoothing slider, AE Region of Interest row with the Aim button, and (on an array camera) AE Tune Speed and Highlight Protection rows. -->

Todas as entradas numéricas aqui usam controles giratórios do tipo “manter pressionado para acelerar”: toque = ±1, mantenha pressionado por &gt;1,5 s = ±10, mantenha pressionado por &gt;3 s = ±100. O valor é enviado para a câmera quando você solta.

| Controle | Faixa / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Exposição (us)**| Mínimo/máximo em tempo real da câmera | Automático | Todos | Tempo de exposição em microssegundos, com um botão para alternar entre**Automático/Manual**. Automático = exposição automática contínua pela câmera. |
| **Ganho (dB)**| Mínimo/máximo em tempo real da câmera (por exemplo, até 48 dB) | Manual (desativado) | Todos | Ganho analógico/digital com seu próprio botão**Automático/Manual**. |
| **Brilho-alvo da AE**| 0–255 | 80, modo**Automático**| Todos (editável quando a AE ou o ganho automático estão ativados) | O brilho que a AE busca atingir. No modo**Automático**(padrão), um controlador interno baseado em histograma seleciona a meta automaticamente, mantendo a exposição entre 60% e 75% do máximo do sensor. Digitar um valor ou arrastar a linha laranja do histograma alterna para**Manual**. |
| **Suavização do AE** | 0,5–40, passo de 0,1 | 8,0 | Todos | Amortecimento do AE. Dica de ferramenta: “Valor mais baixo = o AE reage mais rápido (pode apresentar oscilações em altas taxas de quadros). Valor mais alto = mais suave/mais lento.” Valores muito abaixo do padrão podem fazer com que a AE oscile e desestabilize a transmissão em altas taxas de quadros; 8,0 é o padrão estável. |
| **Região de Interesse da AE**| Caixa de seleção “Ativar” + botão**Mirar**| Desativado | Todos | Quando ativado, a AE mede apenas a região verde tracejada em vez de todo o quadro.**Mirar**ativa o recurso “clicar para posicionar” na transmissão ao vivo: um clique centraliza uma região em 30 % do quadro; clicar e arrastar traça um retângulo personalizado (mínimo de 5% × 5%). O**Aim** se desativa automaticamente após uma colocação. A região é remapeada para as coordenadas nativas da câmera, considerando qualquer rotação ou espelhamento que você tenha definido, e é salva com o projeto. |
| **Velocidade de Ajuste do AE** | 0,1–5, intervalo de 0,1 | 1,0 | Apenas para membros do Array | A rapidez com que o alvo do AE automático acompanha as mudanças de brilho da cena; 1,0× verifica novamente a cada 2,5 s. |
| **Proteção de realces** | Rigorosa (1 %) / Normal (5 %) / Flexível (15 %) | Rigorosa | Câmeras que expõem a configuração | Quanto do quadro pode sofrer clipping para branco antes que o AE escureça a imagem. |

{% hint style="info" %}
**Requisito de iluminação para câmeras multiespectrais Bayer (RGN / OCN / NGB):** a cena deve ter luz suficiente em todos os três canais; caso contrário, a calibração não funcionará corretamente — uma única exposição do sensor abrange todos os três espectros. Use um sensor de luz DAQ para medir a iluminação ou opte pelo modo totalmente monocromático (M3M) para que cada banda receba sua própria exposição. Se uma captura violar essa regra, o Chloros detecta isso e avisa você (a notificação “unmix-clamp”).
{% endhint %}

### Formato de pixel e resolução**Membros da

<!-- SCREENSHOT-NEEDED: per-camera Pixel Format & Resolution section on a STANDALONE camera — Pixel Format, Resolution, and Binning dropdowns plus the Current WxH readout. A second capture on an array member showing the read-only "Set in array settings" state would also be useful. -->

matriz** exibem as linhas “Atual” (formato + LxA) e “Binning”, somente para leitura, com a observação “Definido nas configurações da matriz” — reiniciar o fluxo em um membro quebraria a sincronização, portanto, esses parâmetros são gerenciados no [painel de configurações da matriz](#array-settings-pane).**Câmeras autônomas** apresentam:

| Controle | Opções | O que faz |
| --- | --- | --- |
| **Formato de pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de pixel do sensor (profundidade de bits). |
| **Resolução** | Completa / Metade / Um quarto | Relativa ao binning atual: Completa = 2048/N × 1536/N para binning N×N. |
| **Agrupamento** | 1x1 (nenhum) / 2x2 / 4x4 | Agrupamento N×N por hardware — valores maiores reduzem a resolução, mas aumentam a relação sinal-ruído (SNR) e a taxa de quadros. Alterar essa configuração reinicia a transmissão e redefine qualquer região de interesse (ROI) para o novo campo de visão completo. |
| **Atual** | somente leitura | As dimensões reais LxA e o deslocamento (x, y) em vigor. |

### Visualização ao vivo

Tudo nesta seção é **apenas para exibição**— altera o que você vê na transmissão ao vivo, enquanto as capturas salvas permanecem lineares e inalteradas — com uma exceção:**Vignette** é radiométrica e afeta também as exportações (destacado abaixo).

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on an RGB (FRGB) camera — Render resolution, White Balance mode, Gamma, Denoise, Sharpness, Vignette, Color Profile dropdown open showing Raw/Linear/Natural/Enhanced/Custom Temperature, Saturation, Contrast, Mirror H/V and Rotation. -->

<!-- SCREENSHOT-NEEDED: per-camera Live Preview section on a Bayer multispectral (e.g. FRGN) camera — showing the Index row with its gear button (and the absence of the RGB-only White Balance / Gamma / Color Profile / Saturation / Contrast rows). -->

| Controle | Intervalo / opções | Padrão | Aplica-se a | O que faz |
| --- | --- | --- | --- | --- |
| **Resolução de renderização** | 360p (mais rápida) / 480p / 720p / 1080p / Resolução nativa do sensor (mais lenta) | 720p | Todos | A resolução na qual o backend executa a cadeia de pré-visualização radiométrica. Valores mais baixos aumentam a taxa de quadros sem alterar o campo de visão. |
| **Índice**| Caixa de seleção “Ativar” + ícone de engrenagem | Desativado | Apenas multiespectral Bayer,**não** para membros de matriz combinada | Visualização ao vivo do índice de vegetação. O ícone de engrenagem abre a [Calculadora de Índice](#index-calculator-pane) compartilhada, pré-carregada com as bandas naturais do filtro da câmera (por exemplo, `Red_660_RGN`, `Green_550_RGN`, `NIR_850_RGN`). A expressão personalizada mais a LUT (ligada/desligada, nível padrão 3, mínimo padrão 0,2, máximo padrão 1) é calculada em cada quadro de visualização. Membros de matriz combinada ocultam esta linha — a matriz possui um único índice compartilhado. |
| **Equilíbrio de branco** | Desligado / Uma vez / Contínuo + um botão de recaptura | Contínuo | Somente RGB | Equilíbrio de branco em tempo real. O botão de atualização recaptura o equilíbrio de branco a partir do espectro DLS atual (desativado quando o modo está desligado). |
| **Gama** | Ligado / Desligado | Ligado | Somente RGB | Exibe a gama (γ = 2,2 LUT) na pré-visualização ao vivo. As capturas salvas permanecem lineares. |
| **Redução de ruído** | Caixa de seleção + intensidade 0–100 | Desligado / 50 | Todos (por câmera, mesmo dentro de matrizes) | Filtro bilateral na pré-visualização ao vivo. Valores mais altos = detalhes mais suaves, mas menos nítidos. |
| **Nitidez** | Caixa de seleção + intensidade 0–100 | Desligado / 30 | Todos | Máscara de nitidez na visualização ao vivo, aplicada por último. Pode amplificar o ruído. Apenas na pré-visualização. |
| **Vinheta**| Caixa de seleção + intensidade 0–100 | Desativado / 0 | Todas | Remoção manual de vinheta residual (clareia os cantos), sobreposta à estimativa de Vinheta Inteligente do conjunto de câmeras.**Radiométrico — afeta a visualização em tempo real E as exportações**, ao contrário de Redução de ruído/Nitidez. |
| **Perfil de cor** | Raw / Linear / Natural / Aprimorado / Temperatura personalizada | Natural | Apenas RGB | Veja abaixo. |
| **Temperatura de cor** | 2.000–10.000 K, em incrementos de 100 | 5.500 K | Apenas RGB, perfil de temperatura personalizada | Fixa o balanço de branco em uma temperatura de cor correlacionada fixa (entrada DLS ignorada). O último valor em Kelvin selecionado é lembrado mesmo ao trocar de perfil. |
| **Saturação** | 0–200 (100 = neutro) | 100 | Apenas RGB | Saturação HSV na visualização ao vivo. |
| **Contraste** | 0–200 (100 = neutro) | 100 | Apenas RGB | Contraste linear em torno do cinza médio na visualização ao vivo. |
| **Espelhar H / Espelhar V** | Caixas de seleção | Desativado | Todos | Inverte a pré-visualização horizontalmente / verticalmente. |
| **Rotação**| 0° / 90° / 180° / 270° | 0° | Todas | Gira a pré-visualização. A orientação é aplicada no final da cadeia de pré-visualização do backend —**as capturas salvas permanecem na orientação nativa da câmera**, e as visualizações compostas em matriz a ignoram. |**Semântica do perfil de cor** (câmeras RGB):

* **Raw** — ignora completamente a cadeia de processamento.
* **Linear** — sinal escuro + campo plano + balanço de branco; sem matriz de cor, sem gama.
* **Natural** *(padrão)* — linear mais a matriz de correção de cor medida e uma curva de tom adaptável à cena.
* **Aprimorado**— Natural mais vibração e contraste local CLAHE. O custo extra se aplica**apenas à visualização ao vivo** — as imagens salvas sempre recebem o acabamento completo, independentemente do perfil.
* **Temperatura personalizada** — Natural com o equilíbrio de branco fixado no valor em Kelvin de sua escolha.

{% hint style="warning" %}
Para os modos Natural, Aprimorado e Temperatura personalizada, o painel exibe uma observação sobre o tom: os quadros são clareados de acordo com sua própria cena, portanto, as imagens *exibidas* salvas não são comparáveis entre si. **Exporte a radiação ou a refletância para medições.**
{% endhint %}

### Sobreposições na tela (desenhadas sobre a transmissão ao vivo)

Elas são exclusivas do front-end — aplicadas sobre o vídeo, sem nunca afetar a transmissão ou as capturas.

<!-- SCREENSHOT-NEEDED: a live feed tile with overlays active — zebra stripes on clipped sky, 3x3 grid, focus peaking in the default orange, and the on-feed histogram strip; the overlays section of the settings pane visible alongside. -->

| Sobreposição | Controles | Padrão | O que faz |
| --- | --- | --- | --- |
| **Zebra** | Caixa de seleção + limite 200–255 | Desativado / 250 | Listras diagonais magenta nos pixels cortados. |
| **Mira** | Caixa de seleção | Desativado | Marca no centro do quadro. |
| **Grade** | Desativado / 3 × 3 / 9 × 9 | Desativado | Grade de composição. |
| **Histograma** | Caixa de seleção + largura de 0,10–0,90 do quadro | Desativado / 0,25 | Uma faixa de histograma na transmissão. |
| **Pico de foco** | Caixa de seleção + limite 20–200 + amostra de cor | Desativado / 80 / `#ff5722` | Destaque de borda Sobel para foco. |
| **Divisões de canal** | “Mostrar divisões (Red / Green / NIR)&quot; / botão “Ocultar divisões” | Oculto | Adiciona três blocos independentes em escala de cinza por canal ao lado da imagem composta (o rótulo do botão segue os canais de filtro da câmera). Cada bloco dividido pode ser arrastado e compartilha a cor da borda da câmera. Não disponível em câmeras monocromáticas. Salvo com o projeto. |

### Medidor pontual

* Caixa de seleção **Clique para amostrar**: clique na imagem ao vivo para amostrar um único pixel (uma cruz marca o local) ou clique e arraste uma região para obter uma média de pixels.**Limpar**apaga a amostra e a cruz. É mutuamente exclusivo com o modo**Mirar** do AE-ROI.
* Menu suspenso **Mostrar**:**Raw (profundidade de bits)**— valores digitais nativos na profundidade de bits do sensor (por exemplo, 12 bits → 0..4095) — ou**Exibição (8 bits)** (padrão). Quando um índice em tempo real está ativo, a opção Exibição mostra o valor do índice calculado (por exemplo, NDVI) em vez disso.
* O painel de leitura lista coordenadas de pixels, tamanho do quadro, formato de pixel, profundidade de bits e uma tabela de canais (Canal / Valor / %) com rótulos de bandas e comprimentos de onda; os pares verdes do Bayer são calculados como média; amostras de região exibem “Média de N px”.

O estado do medidor pontual é válido apenas para a sessão.

<!-- SCREENSHOT-NEEDED: Spot Meter in use — reticle placed on the live feed, readout panel showing the per-channel value table with band wavelength labels. -->

### Exposição Automática Preditiva (controlada por DLS)

Esta seção só aparece quando **pelo menos um sensor de luz DAQ estiver conectado** — o solucionador precisa de um espectro descendente em tempo real para operá-la.

<!-- SCREENSHOT-NEEDED: Predictive Auto-Exposure (DLS-driven) section with a DAQ connected — Enable checkbox, Smoothing (α) slider at 0.30, and the "Recalibrate ρ" button. -->

| Controle | Faixa | Padrão | O que faz |
| --- | --- | --- | --- |
| **Ativar** | Caixa de seleção | Ativado (câmeras autônomas) | Um solucionador deutiliza o espectro do DLS mais os escalares do pacote de calibração da câmera para ajustar a faixa mais clara próxima da saturação, mantendo a faixa mais escura acima do limite mínimo de SNR — uma única exposição por cálculo, sem loop de estabilização. Projetado para timelapse com energia solar, onde cada foto deve ser corretamente exposta. O backend recorre silenciosamente à AE reativa sempre que a leitura do DLS estiver desatualizada/ausente ou o pacote de calibração não estiver carregado. |
| **Suavização (α)** | 0,05–1,0, passo de 0,05 | 0,3 | Suavização de soluções preditivas sucessivas (valor menor = maior suavização). |
| **Refletância da cena**| Botão**Recalibrar ρ** | — | Reestima o fator de refletância da cena usado pelo solucionador. |

{% hint style="info" %}
**A conexão em matriz desativa a AE preditiva por padrão** — para matrizes, Chloros o AE inteligente, juntamente com a exposição automática do lado da câmera, controla a exposição (com proteção contra saturação), e a estimativa única de refletância da cena do AE preditivo não é segura em cenas mistas. Você pode reativá-lo por câmera aqui se desejar especificamente uma exposição radiométrica orientada por DLS.
{% endhint %}

**Limite de exposição controlado por DAQ e AE fixada na luz incidente.**Independentemente da caixa de seleção acima, quando um sensor de luz DAQ é atribuído a uma câmera RGB, o Chloros calcula — a partir da irradiância descendente absoluta medida — a exposição máxima × ganho na qual uma superfície com 100% de refletância permanece abaixo do clipping e a aplica como um**limite máximo**na exposição automática. Enquanto o limite máximo estiver ativo, a câmera fica**fixada na luz incidente**: ela opera em malha aberta na exposição medida pela luz incidente com ganho em 0 dB — a exposição segue a luz medida, não o conteúdo da cena. Como o limite máximo só pode reduzir a exposição, ele não pode, por si só, causar clipping. O limite máximo é desativado automaticamente — e a AE normal da cena é retomada — sempre que a leitura do DAQ estiver ausente, desatualizada (&gt;30 s), ou escura, ou se ≥15 % do quadro sofrer clipping na exposição fixada (o que significa que o sensor e a câmera estão captando iluminação diferente). Não há opção na interface gráfica; esse é o comportamento padrão sempre que uma câmera RGB tem um DAQ vinculado.

### Os membros da matriz de aquisição e acionamento

<!-- SCREENSHOT-NEEDED: Acquisition & Trigger section on a standalone camera — Trigger Mode, Trigger Source, and the Frame Rate row in Auto mode showing live fps; ideally a second capture on an array member showing the read-only Role/Sync Line/Peers rows. -->

exibem adicionalmente as linhas **Função**(somente leitura; Mestre em azul / Escravo em verde),**Linha de Sincronização**e**Parceiros**.

| Controle | Opções | Padrão | Observações |
| --- | --- | --- | --- |
| **Modo de disparo** | Desativado / Ativado | Ativado | Desativado para membros da matriz (a matriz gerencia o acionamento). |
| **Fonte de acionamento** | Software / Linha 0 (M8) / Linha 1 / Linha 2 | Linha 0 | Oculto enquanto o Modo de acionamento estiver desativado; desativado para membros da matriz. A Linha 0 é a entrada de acionamento externa opto-isolada M8. |
| **Taxa de quadros**| Automático / Manual + valor | Automático |**Automático**: o limite de taxa de quadros da câmera está desativado — a exposição determina os fps, e a caixa exibe a taxa real em tempo real.**Manual**: você define o limite de fps com um controle deslizante (de 1 até o máximo limitado pela largura de banda), com base na taxa real atual. Os membros da matriz veem um valor somente leitura “N fps (ao vivo)” com a indicação “Definido nas configurações da matriz”. |

### Rede / Transporte

| Coluna | Comportamento |
| --- | --- |
| **Tamanho do pacote**| 1500 (Padrão) / 9000 (Jumbo) — padrão**Jumbo**. |
| **Largura de banda** | Limite de largura de banda do link somente leitura em MB/s. O backend reequilibra esse valor entre todas as câmeras conectadas a cada conexão/desconexão. |
| **Gerenciamento de buffer** | Modo de gerenciamento de buffer somente leitura. |

### Captura

O painel termina com um botão **“Abrir configurações de captura…”** que leva ao [painel Configurações de captura](capture.md#the-capture-settings-pane) (desativado até que um projeto seja aberto — “Crie ou abra um projeto para salvar capturas”). Se a câmera estiver oculta ou pausada, uma dica lembra que você deve exibi-la/retomá-la antes de capturar.

## Painel de configurações da matriz

Abra clicando na **engrenagem**em uma linha da MATRIZ. Cabeçalho: nome da matriz com um lápis para renomear e**×** para fechar. As seções abaixo marcadas como *somente combinadas* aparecem apenas para matrizes conectadas no modo de exibição combinado.

<!-- SCREENSHOT-NEEDED: array settings pane, top portion — array name header, Sync section (Master/Slaves/Sync Line), and Ambient Light Sensor section with the Light Sensor dropdown and the green "Active — all cameras in the array are illumination-corrected" status line. -->

### Sincronização

Linhas **Mestre**,**Escravos**e**Linha de Sincronização** somente para leitura.

### Sensor de luz ambiente

Exibido tanto para matrizes combinadas quanto separadas:

* Caixa de seleção **Alvo de calibração** — “Detectar alvo ArUco MAPIR e validar NDVI em relação à LUT de refletância do painel”; controla a sobreposição do alvo e a tabela de validação do bloco combinado.
* Menu suspenso **Sensor de luz**menu suspenso — vincula um DAQ a toda a matriz. A seleção é aplicada imediatamente, se propaga ao menu suspenso**Sensor de luz** de cada câmera da matriz (ainda é possível sobrescrever por câmera) e começa a encaminhar espectros para a matriz.
* Linha de **Status** em tempo real: Desligado · “Aguardando o primeiro espectro…” · “Ativo — todas as câmeras da matriz estão com correção de iluminação” · “Nenhum espectro novo nos últimos 3 s — ainda usando a última leitura (sem tempo limite para dados obsoletos)…”.
* Observação no painel: “Correção radiométrica em toda a matriz. As configurações por câmera substituem esta.”

### Captura — configurações uniformes do sensor *(apenas combinadas)*

Essas configurações se aplicam uniformemente a todos os membros (alterações por membro quebrariam a sincronização). As edições são preparadas e aplicadas juntas.

<!-- SCREENSHOT-NEEDED: array settings Capture section — Pixel Format, Binning, Resolution preset, the ROI crop W/H/X/Y fields with the "max WxH" hint and Reset button, Trigger Rate row in Auto showing the derived fps, and the Apply/Cancel buttons; ideally with the live orange crop-preview box visible on the array tile. -->

| Controle | Opções / intervalo | O que faz |
| --- | --- | --- |
| **Formato de pixel** | BayerRG8 / BayerRG10 / BayerRG12 / BayerRG16 / Mono8 | Formato de sensor uniforme para todos os membros. |
| **Binning** | 1x1 / 2x2 / 4x4 | Binning de hardware — mantém o campo de visão completo enquanto aumenta a relação sinal-ruído (SNR) e a taxa de quadros. Alterá-lo redefine os campos de ROI para o novo campo de visão completo. |
| **Predefinição de resolução** | Completa / Metade / Quarto | Relativa ao binning; preenche os campos de ROI com um recorte centralizado. |
| **Recorte de ROI (px)**| Campos numéricos L / A / X / Y | Recorte do sensor. Largura/altura se ajustam a múltiplos de 16 (mínimo 64); os deslocamentos se ajustam a múltiplos de 4. Uma dica “max WxH” mostra o limite máximo e**Redefinir** retorna ao campo de visão completo. Durante a edição, uma caixa laranja de pré-visualização do recorte em tempo real é desenhada no bloco da matriz (incluindo um esquema do sensor completo ao ampliar o recorte para fora). |
| **Taxa de disparo**| Alternância entre Automático / Manual + fps 0,5–10, passo de 0,5 |**Automático**(padrão): o backend calcula a taxa de disparo a partir da resolução e da largura de banda — a entrada fica desativada e exibe o valor calculado.**Manual**: fixa seu valor ao clicar em**Aplicar**. |

Nota no painel: “Alterações de formato/resolução reiniciam todas as câmeras brevemente. A taxa de disparo é aplicada em tempo real.” Os botões **Aplicar / Cancelar** ficam na parte inferior do painel.

### Alinhamento (co-registro) *(somente combinado)*

<!-- SCREENSHOT-NEEDED: array settings Alignment section after a successful calibration — green "RMS x.xx px" residual pill, "✓ All cameras aligned (N)" summary, the per-camera table with px error / match count / NCC columns, the Recalibrate alignment button and the "Auto-expose cameras for alignment" checkbox. -->



* **Residual**: “RMS x,xx px” — verde se for inferior a 1 px, âmbar se for inferior a 3 px, vermelho nos demais casos ou se alguma câmera falhar; “sem perfil” antes da primeira resolução.
* Linha de resumo: “✓ Todas as câmeras alinhadas (N)” / “⚠ p/N câmeras alinhadas — <serial (filter)="">falha” / “Recorte ativo — Recalibrar para alinhar (usa o sensor completo)” / “Aguardando a estabilização da exposição…”.
* Tabela por câmera: câmera (últimos 4 dígitos do número de série + filtro), erro de reprojeção em px com contagem de correspondências (“ref” para a câmera mestre) e a pontuação de correlação cruzada normalizada da sobreposição em relação ao limite mínimo de aprovação de 0,35.
* Botão **Recalibrar alinhamento** (exibe “Calibrar alinhamento” antes do primeiro perfil) — reexecuta o co-registro em quadros novos.
* Caixa de seleção **“Exposição automática das câmeras para alinhamento”** (marcada por padrão) — clareia temporariamente câmeras escuras ou sem contraste (primeiro a exposição, depois o ganho) para que apresentem textura para correspondência e, em seguida, restaura a exposição automática (AE).

A visualização combinada se alinha automaticamente ao ser aberta; recalibre se o foco ou a profundidade da cena tiverem mudado. O alinhamento é **projetado para valer apenas para a sessão** — ele nunca é salvo em um perfil, pois depende da distância da cena naquele momento. As capturas ainda podem ser exportadas com registro de pixels (consulte [Exportações alinhadas](capture.md#per-array-controls)).

### Vinheta inteligente

* Caixa de seleção **Ativar correção**— aplica a estimativa de vinheta por câmera à cadeia radiométrica (em tempo real**e** nas exportações).
* **Calibrar a partir da visualização atual**— aponte o conjunto de câmeras primeiro para um alvo uniforme (painel plano, parede ou céu); cada câmera é nivelada individualmente e o status informa “n/N câmeras · −x,x %” de ganho de nivelamento.**Limpar** remove a estimativa.
* Ajuste com precisão por câmera usando o controle deslizante **Vignette** específico para cada câmera na [Pré-visualização ao vivo](#live-preview).

### Pré-visualização ao vivo *(somente combinada)** **Índice**: marque a caixa de seleção + ícone de engrenagem — abre a [Calculadora de Índice](#index-calculator-pane) compartilhada com faixas desenhadas a partir de**todas** as câmeras participantes. Uma linha de pré-visualização de expressão abaixo mostra a expressão atual (“Nenhuma expressão definida — abra a calculadora para criar uma”), atualizada a cada segundo.
* Menu suspenso **Resolução de renderização**(mesmas predefinições por câmera, padrão 720p): a altura do stream de visualização ao vivo**e** o tamanho da exportação da composição salva. Observação no painel: “Pré-visualização + tamanho da composição salva. As imagens por câmera sempre são exportadas em resolução total.”

### Camadas de exibição *(somente combinadas)** Caixa de seleção **Ativar** (padrão desativado — a câmera principal é exibida diretamente; ativada = composição em camadas).
* Menus suspensos **Primeiro plano**/**Fundo**: cada câmera membro (por nome) ou**Índice**. Quando o Primeiro plano é definido como Índice, os pixels fora dos valores Mín./Máx. da LUT exibem a camada de Fundo.

### Visualização dividida *(apenas combinada)*

**“Mostrar câmeras membros”**— um botão**Dividir/Ocultar câmeras membros** que adiciona a transmissão ao vivo de cada câmera membro como blocos separados na grade ao lado da composição. Os blocos leem o buffer de quadros existente da matriz (sem conexão extra de câmera). Apenas na visualização em grade; salva por matriz junto com o projeto.

### Recursos

Um painel somente leitura atualizado a cada 5 s:

* **Rótulo de nível**: “Captura simultânea” (verde) · “Captura simultânea (emissão escalonada FTD)” (verde) · “Captura escalonada (desvio de 100 ms)” (âmbar) · “Configuração muito grande” (vermelho).
* **Integridade do quadro**: “x,xx % incompleto” — verde abaixo de 1 %, âmbar abaixo de 5 %, vermelho a partir de 5 %.
* **Linha de link**: “NIC {mbps} Mbps - sustentada {MB/s} MB/s”.

Este é o orçamento de largura de banda em tempo real da matriz. Para obter informações sobre o modelo de fps e de rede subjacente — e o que alterar quando o nível ficar âmbar ou vermelho — consulte [Matrizes com várias câmeras](arrays.md) e a [Referência CLI](../reference/cli-reference.md).



<!-- SCREENSHOT-NEEDED: array settings Capabilities panel showing a green "Simultaneous capture" tier, the frame-health percentage, and the NIC/sustained-throughput line. -->## Painel “Calculadora de Índice”

A terceira página da barra lateral, compartilhada entre a configuração de Índice por câmera e a configuração de Índice da matriz combinada (uma de cada vez — o título é “Calculadora de Índice — <camera name="">” ou “Calculadora de Índice —<array name="">

”). Ela recebe a lista de bandas (as bandas naturais do filtro da câmera ou todas as bandas dos membros do conjunto), a expressão atual e a configuração da LUT (ativada/desativada, nível — padrão 3, mínimo — padrão 0,2, máximo — padrão 1), além de um histograma de índice em tempo real. **Aplicar** confirma a expressão; as alterações na LUT são aplicadas em tempo real à visualização.

<!-- SCREENSHOT-NEEDED: Index Calculator pane open for a combined array — band buttons for all member cameras, an NDVI-style expression in the editor, LUT controls, and the live index histogram. -->

## Configurações por câmera vs. gerenciadas pela matriz

Referência rápida sobre o que fica onde quando uma câmera é membro de uma matriz:

| Gerenciado pela matriz (somente leitura no painel da câmera) | Ainda por câmera dentro de uma matriz |
| --- | --- |
| Formato de pixel, resolução, binning | Exposição automática (exposição, ganho, alvo, suavização, ROI) |
| Modo/fonte de disparo, taxa de quadros | Redução de ruído, nitidez, vinheta |
| | Orientação (espelhamento/rotação), sobreposições de exibição, medidor pontual |
| | Índice (matrizes com exibição separada), vinculação de sensor de luz |

Outros comportamentos transversais:

* **Exibição combinada vs. separada** é escolhida na conexão da matriz: combinada = um bloco composto alinhado (feeds dos membros apenas via Split View); separado = cada membro renderiza seu próprio bloco sincronizado. Uma câmera nunca exibe simultaneamente um feed independente e um bloco da matriz.
* **Reconexão automática**: abrir um projeto salvo restaura suas câmeras e matrizes e reaplica todas as configurações salvas ao backend antes que as transmissões sejam retomadas.
* **Controle de captura**: câmeras ocultas ou pausadas são excluídas da opção “Capturar tudo”; uma matriz é totalmente bloqueada somente quando TODOS os membros estiverem ocultos/pausados. Consulte [Configurações e modos de captura](capture.md).

## Como as configurações são mantidas

O estado da aba “Câmera” é salvo **com o projeto**, não no navegador:

* Cada alteração reativa captura instantâneos das câmeras e matrizes no arquivo `cameras.json` do projeto (com tempo de espera de 500 ms). Isso inclui nomes e cores das câmeras, configurações de exposição/ganho/AE, formato de pixel/resolução/binning, taxa de disparo, configurações de pré-visualização (resolução de renderização, redução de ruído, nitidez, vinheta, perfil de cor, saturação/contraste), orientação, sobreposições, divisões de canal, configuração de índice, configurações de AE preditivo, ROI de AE, nomes de matrizes, modo de exibição, configurações de captura de matrizes (incluindo a posição de recorte da ROI) e o bloco de grade (zoom do feed, modo de visualização, bloqueio de grade, ordem manual dos blocos, câmeras ocultas, blocos fechados, câmera ativa).
* As configurações dos sensores de luz são salvas no arquivo `sensors.json` do projeto.
* Ao reabrir o projeto, o hardware é reconectado e todas as configurações são reaplicadas.
* **Sem projeto aberto = apenas sessão**: sem projeto, nada é mantido após o fechamento do Chloros.
* Apenas sessão, independentemente do projeto: estado de pausa, amostras do medidor pontual, a caixa de seleção “Alvo de calibração” por câmera (sempre desmarcada ao abrir) e o perfil de alinhamento da matriz (recalculado por sessão, conforme projetado).
* Uma exceção: as seleções de exportação das **Configurações de Captura** e o modo de captura persistem por projeto no armazenamento local do aplicativo, em vez de no `cameras.json` — consulte [Configurações e Modos de Captura](capture.md).</array></camera></serial>
