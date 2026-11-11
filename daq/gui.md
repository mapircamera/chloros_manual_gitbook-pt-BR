# A guia DAQ no Chloros

A guia DAQ — identificada como **Sensores de luz** na barra lateral do Chloros — é a interface de controle em tempo real para [os sensores de luz DAQ-U, DAQ-M e DAQ-E](README.md): conecte sensores por qualquer protocolo de transporte, visualize espectros calibrados em tempo real, calcule a refletância em tempo real a partir de um par de sensores e grave arquivos `.daq` diretamente no seu projeto.

A aba fica disponível assim que o backend do Chloros terminar de iniciar. Os gráficos da aba são alimentados pelo serviço DAQ do Chloros por meio de uma conexão ativa que se reconecta automaticamente (intervalo de 2 a 10 s) caso seja interrompida; enquanto o serviço estiver inacessível, a linha de status de um sensor exibe **Sem servidor**.

O layout consiste em uma **barra lateral de sensores**(uma linha por sensor conectado) e uma**área de gráficos** (um bloco de gráfico por sensor ou grupo).

<!-- SCREENSHOT-NEEDED: full DAQ (Light Sensors) tab in list view with one DAQ-E connected — sensor sidebar on the left (Connect Sensor + Record All buttons, one sensor row), spectrum chart with rainbow fill in the main area, live data table below the chart -->

***

## Conectando um sensor

Clique em **Conectar sensor**na parte superior da barra lateral. A caixa de diálogo de conexão é aberta na área principal (ou como uma sobreposição ao adicionar outro sensor — nesse caso, um botão**Cancelar** é exibido).

| Controle | Comportamento |
| --- | --- |
| **Tipo de dispositivo** | `DAQ-U (USB)` (padrão), `DAQ-M (Bluetooth)` ou `DAQ-E (Ethernet)`. A alteração reinicia a varredura para o transporte recém-selecionado. |
| **Porta / Dispositivo BLE / Nome do host / IP** | Lista os dispositivos detectados como `device - description`; a primeira entrada reconhecida como sensor é selecionada automaticamente. Durante a varredura, exibe `Scanning...` (USB), `Scanning (N)...` com uma contagem regressiva de 8 segundos (BLE) ou `Discovering ethernet sensors (N)...` com uma contagem regressiva de 5 segundos (Ethernet). Resultados vazios são exibidos como `No ports` / `No BLE devices` / `No ethernet sensors found`. |
| **↻ Atualizar** | Realiza uma nova varredura no transporte selecionado imediatamente (desativado durante a varredura BLE/Ethernet). |
| **Conectar sensor** | Ativado assim que um dispositivo é selecionado; renomeia para `Connecting...` enquanto a conexão é estabelecida. |

A descoberta só é executada **enquanto a caixa de diálogo de conexão estiver na tela** e se repete a cada 15 segundos apenas para o transporte selecionado — simplesmente abrir a aba não inicia a varredura. Em caso de falha, a caixa de diálogo exibe: *“Falha na conexão. Tente desconectar e reconectar o sensor e, em seguida, clique em Conectar novamente.”*

A barra lateral se abre automaticamente quando o primeiro sensor se conecta.

{% hint style="info" %}
**O DAQ-E não está aparecendo?**O DAQ-E não possui LED de status — verifique o indicador de PoE/link no switch ou na porta do injetor em que ele está conectado e aguarde alguns segundos após a inicialização para que ele seja inicializado. A máquina Chloros deve estar no mesmo domínio de difusão (o mDNS não atravessa roteadores). A regra de firewall do**Backend do Chloros (UDP-In)** do instalador permite a passagem de mDNS (UDP 5353), dados do DAQ-E (UDP 5002) e PTP (UDP 319/320); um firewall de terceiros precisa ter essas mesmas portas liberadas. Duas unidades DAQ-E em uma LAN são detectadas separadamente, cada uma com seu próprio nome de host `daq-e-<id>.local`.
{% endhint %}

<figure><img src="../.gitbook/assets/v120-daq-device-type.png" alt=""><figcaption>O tipo de dispositivo oferece DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet)</figcaption></figure>

***

## A barra lateral de sensores

Cada sensor conectado recebe uma linha (mais uma linha por grupo “Ambiente+Objeto”). As linhas podem ser reordenadas por arrastar, e sua ordem também reordena os blocos do gráfico. Clique em uma linha para tornar esse sensor/grupo o gráfico ativo na visualização em lista.

| Elemento | Significado |
| --- | --- |
| Borda esquerda colorida | A cor do gráfico do sensor. |
| Emblema de transporte | `DAQ-U` / `DAQ-M` / `DAQ-E`, ou um ícone verde `REF` para um grupo de refletância do Ambient+Objeto. |
| Nome do dispositivo | Por padrão, é o número de série do sensor (sua identidade estável para calibração, nomes de arquivos `.daq` e correspondência de importação); nomes personalizados são mantidos por projeto. |
| **Calibrado** (verde) | Exibido quando o pacote de calibração de fábrica do sensor é carregado, ou seja, os espectros estão na unidade correta W/m²/nm. |
| Ícone **Atualização disponível** (âmbar, somente DAQ-E) | O firmware em execução é mais antigo do que a imagem incluída nesta versão do Chloros

. Durante uma atualização, ele mostra o progresso em tempo real (`Flashing… N%`, `Restarting sensor…`, depois `Updated X → Y` ou `Failed`). |
| Olho | Alterna a visibilidade deste sensor em seu gráfico. |
| Engrenagem | Abre a janela modal de configurações por sensor (abaixo). |
| ✕ (vermelho) | Desconecta o sensor ou remove um grupo “Ambiente+Objeto”. |

Acima das linhas, há dois botões:

* **Conectar Sensor** — abre a caixa de diálogo de conexão (mudam para `Connecting...` enquanto estiver ocupado).
* **Gravar tudo / Parar tudo**— inicia ou interrompe uma gravação `.daq` em**todos**os sensores conectados. Requer pelo menos um sensor**e um projeto aberto** (dica de ferramenta: “Abra um projeto para gravar”); fica vermelho enquanto qualquer gravação estiver em andamento.

Quando vazio, exibe “Nenhum sensor conectado”.

<!-- SCREENSHOT-NEEDED: sensor sidebar with three rows — a DAQ-E showing both the green Calibrated pill and the amber Update Available pill, a DAQ-U row, and a green REF group row — plus the Connect Sensor and Record All buttons -->

***

## Configurações por sensor (a janela modal da engrenagem)

Abra clicando no ícone de engrenagem na linha de um sensor. Conteúdo em ordem:

* **Linhas de informações** — Tipo de dispositivo (DAQ-U/M/E), Conexão (`Serial (USB)` / `Bluetooth` / `Ethernet`), porta (porta COM, endereço BLE ou host) e serial.
* **Relatório de calibração: Baixar** — busca o certificado de calibração rastreável pelo NIST (PDF) desta unidade e o abre em seu visualizador de PDF. Disponível assim que o número de série for conhecido; o certificado é armazenado em cache na primeira conexão.
* **Nome do dispositivo** — clique no lápis para renomear; permanece por projeto.
* **Cor da linha do gráfico** — amostra de cor; permanece por projeto.
* **Tempo de integração (ms)**— controle deslizante + número,**1–500 ms**, padrão**32 ms**. Desativado enquanto a AE estiver ativada.
* **Média de quadros**— controle deslizante + número,**1–50 quadros**, padrão**20**.
* **AE: LIGADO/DESLIGADO**— botão para ativar/desativar a exposição automática;**padrão LIGADO** na conexão. Desative-o para definir o tempo de integração manualmente.
* **Interromper transmissão / Iniciar transmissão** — pausa ou retomada da transmissão ao vivo.
* **Gravar / Parar de gravar** — gravação do `.daq` por sensor (requer um projeto aberto).
* **Cap** — o perfil de correção de cap (próxima seção).
* **Linhas de informações em tempo real** — Tempo de integração (ms), FPS, Amostras, Gravação (vermelho `REC` ou `Off`) e Status (`Streaming` / `Paused` / `SATURATED` / `No Server`).

### Apenas DAQ-E: linhas de rede, firmware e PTP

* **Nome do host / IP** — o endereço atual da unidade.
* **Firmware** — versão atual do firmware, além de uma célula de ação: um<version\>

botão</version\>

**Atualizar para \<version\>** aparece quando essa compilação do Chloros

inclui uma imagem de firmware mais recente para o DAQ-E. A atualização é transferida pela rede em cerca de 30 segundos; o sensor reinicia e se reconecta automaticamente, e uma transferência interrompida mantém o firmware atual intacto. O progresso é transmitido em tempo real (`Flashing… N%` → `Restarting sensor…` → `Updated X → Y`), e a célula exibe `Up to date` quando atualizada.
* **Sincronização PTP** — o estado PTP em tempo real (reverte para `unknown`). O firmware DAQ-E v1.2.0+ participa do IEEE 1588 PTPv2 como um relógio exclusivamente escravo; o backend do host Chloros

é o grandmaster PTP, e todas as câmeras DAQ-E e LATTICE na LAN funcionam como escravas dele no domínio 0, mantendo os carimbos de data/hora com uma precisão de aproximadamente 1 ms.
* **Rede**— o endereço em que a unidade opera e como o obteve (DHCP, link-local ou estático), além de uma configuração salva que se aplica após uma reinicialização.**Alterar…**abre um formulário:**Automático (DHCP)**ou**IP estático**com um endereço IP, uma máscara de sub-rede e um gateway opcional.**Aplicar e reiniciar** salva as configurações, reinicia o sensor e o reconecta. Um endereço estático fora da rede deste computador é recusado, pois o Chloros

não conseguiria acessar o sensor após a reinicialização; uma caixa de diálogo para permitir isso mesmo assim é exibida, para um computador que será reendereçado ou roteado para essa rede. **Transmissão ao ligar**define se o sensor inicia a transmissão assim que é ligado, antes que qualquer software se conecte.**Reiniciar sensor** reinicia o sensor e o reconecta com suas configurações atuais. A reinicialização é recusada enquanto o sensor estiver gravando. Novidade na versão 1.3.0.

Para um grupo “Ambient+Object”, a janela modal da engrenagem mostra apenas os sensores de fonte do grupo, o nome do dispositivo e a cor da linha do gráfico.

<!-- SCREENSHOT-NEEDED: per-sensor settings modal for a DAQ-E — info rows, Calibration Report Download, Hostname/IP + Firmware row with an "Update to <ver>" button, PTP Sync row, Integration Time / Frame Average sliders, AE ON toggle, and the Cap dropdown all visible (scrolled composite acceptable) -->



### Seleção da tampa

O menu suspenso **Tampa** informa ao Chloros

qual tampa física está instalada sobre o difusor do sensor e aplica o perfil de correção medido na fábrica dessa tampa a todos os espectros. As opções dependem do modelo:

| Modelo | Opções de tampa |
| --- | --- |
| DAQ-U | Nenhuma (sensor sem tampa), FOV 15°, FOV 30°, FOV 45°, FOV 60°, FOV 90°, Sunshine (corretor cosseno) |
| DAQ-M | Nenhuma (sensor sem tampa), Sunshine (corretor de cosseno) |
| DAQ-E | Nenhuma (sensor sem tampa), FOV 15°, FOV 45°, FOV 90°, Sunshine (corretor de cosseno) |

**A configuração padrão para todos os modelos é “Sunshine” (corretor de cosseno)** — a MAPIR

fornece todos os DAQs com a tampa “Sunshine” instalada, e essa é a configuração padrão para uso ao ar livre: uma visão hemisférica de 180° com erro de cosseno ≤ ±4 % até 60° e ≤ ±4,5 % até 70° (não recomendado abaixo de ~15° de elevação solar), com atenuação por design (~12×). Sua seleção é mantida no projeto.

{% hint style="warning" %}
**A seleção da tampa deve corresponder à tampa física.**Nem o sensor nem o software conseguem detectar qual tampa está instalada. A seleção determina tanto a correção em tempo real quanto o carimbo gravado em cada arquivo `.daq` — com a atenuação de ~12× da tampa Sunshine, uma troca de tampa não declarada corrige incorretamente os espectros aproximadamente por esse fator. (Remover e remontar a mesma tampa repete o erro em cerca de 1,5 %.) Escolha**Nenhuma (sensor sem tampa)** somente quando a tampa estiver fisicamente removida; em um DAQ-E, “Nenhum” ainda aplica um perfil geométrico de fábrica para seu difusor de vidro recuado — não é uma operação nula — e um DAQ-E sem difusor é uma configuração de bancada, não uma configuração de campo suportada.
{% endhint %}

{% hint style="info" %}
Atualização em relação a um manual anterior: a opção “Difusor de luz solar instalado” no navegador, presente na versão 1.1.0, foi removida. O gerenciamento da tampa agora é feito por meio deste perfil de tampa por sensor, aplicado no servidor.
{% endhint %}

***

## A área do gráfico

Uma barra fixa na parte superior contém um **botão para alternar entre visualização em lista ⇄ grade**e um controle deslizante de**zoom do gráfico** (tamanho dos blocos: 200–2.000 px). A visualização muda automaticamente para a grade quando há mais de um grupo de gráficos e volta para a lista quando há um ou menos. O modo de visualização e o tamanho do gráfico são mantidos por projeto.

O **gráfico de espectro** de cada sensor mostra:

* **Eixo X** — Comprimento de onda (nm). A grade do sensor varia de 340 a 1.010 nm em intervalos de 5 nm (135 pontos), interpolada para 1 nm para exibição.
* **Eixo Y**— para um sensor calibrado (pastilha verde**Calibrado**),**Potência (W/m²/nm)**— irradiância espectral calibrada radiometricamente em todos os três sistemas de unidades — com um prefixo SI automático (m/µ/n) escolhido a partir do pico. Um sensor cujo pacote de calibração não esteja carregado plota suas leituras brutas como**Intensidade (contagens)**, sem escala. Quando um gráfico sobrepõe os dois tipos, a potência fica no eixo esquerdo e as contagens no eixo direito.
* Um preenchimento espectral em tons de arco-íris sob um único traço; vários sensores em um único gráfico são sobrepostos como linhas coloridas com preenchimentos esmaecidos.
* **Passe o mouse**— um cursor vertical com comprimento de onda e valor por sensor (W/m²/nm para um sensor calibrado; contagens totais nos demais casos);**arraste** para ampliar (um botão de redução aparece enquanto estiver ampliado).
* Um botão **+** (somente na visualização em grade) para adicionar um sensor a este gráfico ou criar um grupo (abaixo).
* O nome do dispositivo centralizado na parte superior e um indicador giratório até que o primeiro quadro seja carregado.

A **saturação** não é indicada no próprio gráfico: um sensor saturado exibe o texto de status `SATURATED` em vermelho e uma linha `Saturated: Yes` em vermelho na tabela de dados em tempo real. Reduza o tempo de integração ou reative o AE para limpar isso.

<!-- SCREENSHOT-NEEDED: grid view with at least two chart tiles visible, the Chart Zoom slider and list/grid toggle in the top bar, and the "+" add-sensor button visible on one tile -->



***

## Tabela de dados em tempo real (visualização em lista)

Abaixo do gráfico, na visualização em lista, atualizada a cada 500 ms:

* **Todos os modelos**: Amostra de cor da luz (sRGB a partir de CIE XYZ), Saturado (Sim/Não), CIE 1931 X/Y/Z, Cromaticidade x/y, CIE u′/v′, CCT (K), CRI (Ra), comprimento de onda dominante (nm), comprimento de onda de pico (nm), pureza de excitação, Duv, CIE L\*/a\*/b\* e Munsell H/V/C.
* **Apenas sensores calibrados**(qualquer um dos modelos DAQ-U / DAQ-M / DAQ-E, desde que seu pacote de calibração de fábrica esteja carregado — o selo verde**Calibrado** na linha do sensor é o indicador): Potência total (W/m²), Lux fotópico (lx), Lux escotópico (lx), Relação S/P, PPFD mais PPFD Red

/Green

/Blue

(µmol/m²/s) e as irradiâncias opicas — cone S, melanópico, rodópico, cone M, cone L (todas em W/m²).

<!-- SCREENSHOT-NEEDED: list view live data table for a DAQ-E showing both the colorimetric rows and the power-calibrated rows (Total Power, Photopic/Scotopic Lux, PPFD, opic irradiances) -->

***

## Grupos de refletância (Ambiente + Objeto)

Dois sensores conectados podem ser combinados para formar uma exibição de refletância em tempo real — sem o uso de câmera:

1. Na visualização em grade, clique em **+**em um bloco de gráfico e selecione**Combinar Ambiente + Objeto**.
2. Selecione um sensor de **Fonte de Luz Ambiente**e um sensor de**Scanner de Objeto**(dois sensores distintos) e, em seguida, clique em**Criar**. O

Chloros

a calcula R(λ) = objeto(λ) / ambiente(λ) por comprimento de onda a partir dos dois fluxos em tempo real (0 quando ambiente ≤ 0). O rótulo do grupo segue a classe de calibração dos sensores:

* Ambos os sensores calibrados (pacote carregado) → **“Refletância aparente”**.
* Qualquer um dos sensores não calibrado → **“Refletância Relativa”**.

O grupo aparece como uma linha verde `REF` na barra lateral e em seu próprio gráfico (preenchimento arco-íris, valores exibidos ao passar o mouse com 4 casas decimais, zoom por arrastar).

O menu **+**também oferece a opção**Adicionar novo sensor** com três opções de posicionamento: *Combinar novo sensor* (incorporar a este gráfico), *Mover sensor existente para cá* ou *Visualizar novo sensor* (em seu próprio gráfico).

<!-- SCREENSHOT-NEEDED: the "+" add-sensor overlay open on a chart tile showing the menu (Add New Sensor / Combine Ambient + Object / Cancel), and the Ambient + Object sub-dialog with its two sensor selects -->



### Tabela de índices de vegetação

Na visualização em lista, uma tabela de índices de vegetação fica abaixo do gráfico de um grupo de refletância, calculada a partir da refletância em tempo real nos centros das bandas **azul 450 / verde 550 / vermelho 670 / NIR

800 nm** (valores com 4 casas decimais; `---` quando não calculável; passe o mouse sobre o nome de um índice para ver seu nome completo):

* **Sempre exibido** (invariante em escala, qualquer combinação de sensores): NDVI

, GNDVI

, ENDVI, WDRVI

, GRVI

, CVI

, GCI

, MSR.
* **Somente quando ambos os sensores estiverem calibrados em potência** (ambos os pacotes carregados): EVI

, SAVI

, OSAVI

, GSAVI, GOSAVI, MSAVI2, RDVI, TDVI, LAI

, NLI, MNLI, FCI, GEMI

.

<!-- SCREENSHOT-NEEDED: an Ambient+Object reflectance group in list view — reflectance chart labeled "Apparent Reflectance" with the vegetation index table below it showing live NDVI etc. -->

***

## Gravação de arquivos `.daq`

* A gravação requer um **projeto aberto** — caso contrário, tanto a opção “Gravar tudo” (na barra lateral) quanto o botão “Gravar” de cada sensor ficam desativados.
* Os arquivos são gravados em **`<project folder>/light_sensor/`**; os nomes dos arquivos contêm o ID do sensor e um carimbo de data/hora, e o nome do dispositivo é armazenado junto com a gravação.
* Quando uma gravação é interrompida (Parar, Parar tudo ou uma desconexão no meio da gravação), o arquivo `.daq` concluído é **adicionado automaticamente ao projeto aberto** — ele aparece na lista de arquivos do projeto sem necessidade de adição manual, pronto para servir como dados de irradiância descendente para [processamento de refletância](README.md).
* Um indicador vermelho `REC` é exibido nas linhas em tempo real da janela modal de configurações durante a gravação.

Para valores quantitativos de irradiância, calcule a média de pelo menos 15 segundos de dados — trata-se de uma característica do instrumento, não de um defeito.

<!-- SCREENSHOT-NEEDED: recording in progress — sidebar Stop All button in its red state and the settings modal live rows showing Recording: REC -->



***

## Layouts multissensores e persistência do projeto

* Combine vários sensores em um único gráfico (eixos compartilhados), mantenha gráficos separados (layout de grade automático), mova sensores entre gráficos, reordene linhas/blocos arrastando-os e oculte sensores individuais com o botão de ativação/desativação.
* Por projeto, o Chloros

mantém: nomes de dispositivos, cores dos gráficos, tamanho do gráfico, modo de visualização e as configurações de cada sensor (tempo de integração, média de quadros, estado do AE, seleção de tampa).
* **Ao reabrir um projeto, seus sensores são reconectados automaticamente** por endereço — porta COM para o DAQ-U, dispositivo BLE para o DAQ-M, nome de host mDNS para o DAQ-E (resolvido mesmo que o IP da unidade tenha mudado) — e reaplica o perfil de tampa salvo de cada sensor, a média de quadros, o estado AE e o tempo de integração manual.***

## Emparelhamento de câmera (DLS)

Não há nada para emparelhar. Ao contrário dos fluxos de trabalho de DLS com drones, que vinculam um sensor de luz a uma câmera antecipadamente, o Chloros

combina os dados do DAQ com as imagens posteriormente: no momento da importação/processamento, as leituras do `.daq` são interpoladas para o carimbo de data/hora de exposição de cada captura. Grave com qualquer sensor conectado (o `.daq` é adicionado automaticamente ao projeto), e o processamento de refletância encontra as leituras corretas por tempo — consulte [Sensores de Luz DAQ](README.md) para saber como os dados de luz descendente são utilizados. Para capturas LATTICE **ao vivo**, escolha o sensor no menu suspenso**Sensor de luz** da guia Câmeras; cada captura, então, salva a leitura utilizada, associada apenas às câmeras dessa captura (consulte [Fluxos de Trabalho de Refletância](reflectance.md)).</version\>
