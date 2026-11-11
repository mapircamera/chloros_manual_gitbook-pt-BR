# Referência da CLI Chloros

**Versão:**

1.3.0**Gerado em:**29/07/2026 19:19 ·**Revisado em:** 06/11/2026**Público-alvo:** Otimizado para consumo por LLM; legível por humanos.**Escopo:** Todos os subcomandos do `chloros-cli` voltados para o usuário, com opções e exemplos que podem ser copiados e colados.

Este documento é a referência completa para a ferramenta de linha de comando `chloros-cli` que acompanha o MAPIR Chloros. Ele é intencionalmente exaustivo para que um LLM (ou humano) possa compor qualquer fluxo de trabalho suportado a partir das listas abaixo sem precisar inspecionar o código-fonte.

Se você precisar apenas dos destaques, vá para:
- [Guia rápido de cinco minutos](#five-minute-quickstart)
- [Fluxo de trabalho de primeira conexão da câmera LATTICE](#lattice-camera-first-connect-workflow)
- [Fluxo de trabalho de primeira conexão do sensor DAQ](#daq-sensor-first-connect-workflow)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Modos de captura, gravadores e reprocessamento offline](#capture-modes-recorders--offline-reprocess)

> **Controlando o Chloros a partir de um assistente de IA?** O Claude Desktop, o Claude Code, o Cursor e outros clientes MCP podem operar o processamento, as câmeras e matrizes LATTICE, bem como os sensores de luz DAQ por meio do `chloros-mcp`, o servidor MCP que é instalado junto com o Python SDK (e com o .deb do Linux, no `/usr/bin`): `claude mcp add chloros -- chloros-mcp`. A configuração, as opções e as ferramentas estão na seção [Servidor MCP (assistentes de IA)](sdk-reference.md#mcp-server-ai-assistants). Um assistente com um shell também pode executar diretamente todos os comandos desta página.

> **Alterado na versão 1.3.0** (o que um script ou fluxo de trabalho existente da versão 1.2.0 percebe).
>
> **Alterações que um script da versão 1.2.0 pode precisar lidar:**
>
> - **Aplica-se o `--preset`.** `lattice capture`, `continuous`, `calibrate`, `multi-capture`, `multi-stream`, `align-calibrate`, `align-apply` e `align-stream` foram removidos na versão 1.2.0 e passou a funcionar com as configurações próprias da câmera. Agora, define o formato, a exposição, o ganho e a taxa de quadros da predefinição: `--preset high_speed` captura em 8 bits a 5 ms, 6 dB, 60 fps. O `lattice cam-connect --preset` também mantém o formato da predefinição (a versão 1.2.0 configurava a câmera para 12 bits após aplicar a predefinição). Consulte [Opções comuns](#common-options-shared-by-most-subcommands).
> - **O formato de pixel Bayer em uma câmera M3M (mono) é sua profundidade.** As predefinições nomeiam formatos Bayer, que um sensor mono não consegue capturar: na versão 1.2.0, a câmera rejeitou o formato e manteve o que já tinha, portanto, `--preset high_speed` não era de 8 bits nesse caso. Agora é o formato Mono com a mesma profundidade: `high_speed` e `--pixel-format BayerRG8` resultam em `Mono8`, `high_quality`, `triggered` e `BayerRG12` resultam em `Mono12`. Em uma matriz mista de M3C + M3M, cada câmera adota o formato compartilhado em sua própria família, e o projeto `network-analysis` / `analyze-array --pixel-format` projetam cada câmera nessa profundidade. O mesmo ocorre no sentido inverso: uma M3C solicitada para o formato Mono adota o formato Bayer dessa profundidade; portanto, uma matriz cujo mestre seja uma M3M não reabre mais suas câmeras M3C no modo Mono. Consulte [Câmeras Mono (M3M) de banda única](#mono-m3m-single-band-cameras).
> - **As exportações do `lattice array-capture` são alinhadas por padrão, assim como as do aplicativo.** Todos os arquivos `debayered`, `display`, `radiance`, `reflectance` e arquivos de índice são distorcidos para se adequarem ao quadro da câmera mestre e recortados de acordo com a sobreposição dos membros (menor que o sensor devido aos deslocamentos das câmeras, `XMP-mapir:AlignmentJSON` `"applied": true`), de modo que uma captura via CLI corresponde aos mesmos arquivos da captura da mesma cena feita pelo aplicativo Chloros; o `raw` permanece nativo. Antes, uma captura sem `--aligned` salvava todos os arquivos no tamanho nativo, sem distorção. `--no-aligned` mantém isso. A mesma configuração padrão se aplica a `--smart`, `--continuous` e `--interval`, bem como ao SDK e ao `chloros-mcp`. Consulte [modos do `array-capture`](#modo-de-captura-de-matriz).
> - **O `lattice array-capture --compression` usa como padrão o `none`.** Os arquivos são gravados sem compressão (cerca de 6,3 MB por quadro em resolução total) para obter a taxa sustentada mais rápida; a versão 1.2.0 gravava no modo `deflate` (cerca de 4,1 MB). Passe `--compression deflate` para obter arquivos menores. Ambos são sem perdas.
> - **Os formatos PNG e JPEG aceitam apenas os produtos `display` / `preview` e indexados.** Uma captura com `--format png` ou `jpg` com um nível que um arquivo de 8 bits não pode suportar é recusada antes que qualquer coisa seja capturada (código HTTP400, saída `1`), assim como a captura padrão `--processing all` com `--format png`/`jpg`. Consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default).
> - **O `lattice dls` requer o `--mac`.**Não há mais um MAC padrão (era o endereço de uma unidade); o `daq pool-discover --only ble` localiza o endereço de um sensor.**`lattice dls --camera SERIAL`** é novo.
> - **Mais falhas saem do `1`.** `process` quando as exportações de uma execução não puderam ser verificadas ou ele perdeu uma imagem, embora os outros produtos tenham sido gravados; um comando de captura com qualquer arquivo que ele não conseguiu gravar (os outros arquivos ainda são salvos); `lattice continuous` e `multi-stream` em uma unidade cheia; `array-burst` / `array-record` quando o gravador relata um erro; `array-build-video` quando um produto solicitado foi ignorado ou não foi criado conforme solicitado; `project burst` / `stream` quando uma captura falha no meio do processo; `project sensor log` quando a gravação não contém nenhuma leitura; `lattice exposure --set`, `gain --set` e `configure --set` com um valor que a câmera rejeita. `lattice network-analysis --json` é encerrado assim como o resumo (`1` para `needs_force_slip`). Ao contrário: o `daq pool-disconnect` de um sensor que não está mais no conjunto encerra o `0`. Consulte [Códigos de Saída](#exit-codes).
> - **Um `lattice array-connect` lento nunca falha com a matriz conectada.** Ele aguarda até 600 s (180 s na versão 1.2.0; `project connect` 90 s), exibindo a etapa do backend à medida que avança, e uma conexão que não for concluída até então é desfeita pelo backend — cada câmera é reposicionada como estava — portanto, um `Error:` (saída `1`) significa que nenhum conjunto está conectado. Na versão 1.2.0, uma conexão realizada logo após as câmeras serem ligadas poderia ultrapassar o tempo de espera e conectar o conjunto de câmeras após o comando ter falhado. Agora, Ctrl+C cancela a conexão. Consulte [Conexões Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-connect` mantém o sensor completo.** Sem `--width` / `--height` / `--binning`, todas as câmeras são configuradas para o sensor completo e a taxa de quadros cai para o que este computador e a rede oferecem; a saída de conexão lista opções mais rápidas (agrupamento ou uma ROI menor) com suas taxas projetadas. A versão 1.2.0 reduziu ou agrupou os quadros para manter a taxa elevada. Consulte [Opções do `array-connect`](#array-connect-options).
> - **Uma rede lenta não impede mais o funcionamento de um array.** Cada câmera é ajustada à sua parcela da largura de banda disponível e a taxa de quadros diminui proporcionalmente. Quando as câmeras não podem ser ajustadas a essa velocidade, o array opera em emissão serial (`slip-emit-and-capture`): todos os quadros chegam, mas as câmeras não são expostas simultaneamente. O `--force-tier sim-capture-sim-emit` exige exposição simultânea. Consulte [FPS do conjunto e modelo de disparo em rajada](#array-fps--burst-model).
> - **`lattice network-analysis` / `analyze-array` definem qual connect se aplica**: sem o `--pixel-format`, os formatos definidos pelo `connect` (Mono12 / BayerRG12, antes era BayerRG8) e a decisão sobre [transporte compactado de 12 bits](#packed-12-bit-transport---packed). O mesmo comando pode, portanto, indicar uma taxa de quadros diferente daquela da versão 1.2.0.
> - **Os caminhos relativos são referenciados a partir do local onde o comando é executado.** `-o` e os outros caminhos que o backend abre (`--input`, `--burst-dir`, `--profile`, `--save`, `--mask`), o `output` de uma etapa da receita, e o padrão `output/` de `lattice index --live` são resolvidos com base na pasta a partir da qual você executa o `chloros-cli`. Na versão 1.2.0, o backend os resolvia em sua própria pasta de instalação, o que, em uma compilação instalada, resultava na falha “Acesso negado”. Consulte [Convenções](#conventions).
> - **O `lattice index` é executado no backend em todas as formas**, assim como os outros comandos `lattice`; portanto, inicie o backend primeiro. O índice offline (`--input`) é calculado pelo backend, que lê o TIFF e grava os produtos ao lado dele ou em `-o`, e os comandos `--list-presets` / `--list-gradients` imprimem as tabelas do backend. No Windows, agora funcionam: antes, a CLI do Windows respondia a qualquer forma de `lattice index`, incluindo `--live`, com `Error: lattice_sdk not available`. No Linux, o índice offline costumava ser executado dentro da CLI sem backend e sem login; agora ele precisa de ambos, como qualquer outro comando da CLI. Consulte [Índice / Cálculos de Vegetação](#index--vegetation-maths).
> - **Novos arquivos ao lado dos produtos.** Cada produto de radianância, refletância e índice no qual a separação de bandas limitou um pixel recebe uma máscara na pasta `clamp_masks/`, ao lado da pasta do produto, e a captura imprime uma linha `clamp mask:`. Consulte [Máscara de limitação da separação de bandas](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on).
> - **Orientação da câmera.** Uma captura com uma única câmera salva os arquivos `raw` e `debayered` conforme o sensor os leu e registra a inversão/rotação da câmera no XMP; o `process` aplica isso aos produtos que gera. Os arquivos membros da matriz permanecem em formato nativo, a menos que você passe o novo `--orient` (`array-capture`, `array-burst`, `project capture` / `burst` / `stream`). Consulte [Orientação da câmera](#como-é-uma-pasta-de-capturas).
> - **Um nome de nível de exportação que não seja “um” é rejeitado.** `lattice capture --processing` e todas as entradas da lista `--levels` aceitam `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou um alias como `preview`; `--processing` e também `all`). Qualquer outro nome é rejeitado antes que a câmera seja acionada (400 HTTP `UNKNOWN_LEVEL`, saída `1`); 1.2.0 salvou `raw` para isso e saiu `0`.
> - **`daq pool-set-cap` precisa de `--cap-id`.** Sem ele, o comando encerra em `2`; 1.2.0 removeu a correção da tampa, deixando o sensor exposto. O `--cap-id none` solicita o sensor sem correção.
> - **O `lattice resolution --binning-mode` foi removido.** O binning é sempre “Média”; o sinalizador ainda é aceito, exibe um aviso e não realiza nenhuma ação. Consulte [Ajuste do sensor](#sensor-tuning).
> - **O `process --debayer texture-aware` é recusado para capturas LATTICE.** Para uma pasta contendo capturas LATTICE, o `process` encerra o `1` após a importação, informando quantas existem, antes de processar qualquer coisa. As capturas do LATTICE sempre recebem o demosaico padrão, portanto a versão 1.2.0 aceitou a opção e a ignorou; o aplicativo Chloros também não oferece essa opção para elas. Consulte [`chloros-cli process`](#chloros-cli-process).
> - **As câmeras voltam ao estado em que foram encontradas — a menos que seja `--keep-settings`.** `lattice array-disconnect`, todos os comandos transitórios (`multi-capture` sem matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`) e o `lattice continuous` restauram cada câmera ao estado em que a encontraram: a fiação do disparador e da linha de sincronização, o PTP e todas as configurações da câmera alteradas durante a sessão (exposição e ganho, alvo de AE, limites e amortecimento, balanço de branco, nível de preto, formato de pixel, binning e ROI, inversões, taxa de quadros). Já o `--keep-settings` salva as configurações de cada câmerana própria memória da câmera, para que ela seja inicializada com essas configurações; o `--clear-settings` redefine essa memória para as configurações de fábrica. De qualquer forma, as câmeras ficam operando autonomamente. Um projeto configura suas câmeras a partir de suas próprias configurações salvas ao conectá-las. Consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings).
> - **Todos os modos salvam um projeto da mesma maneira.** Qualquer alteração em uma câmera, um array ou um sensor de luz conectado a um projeto — seja pelo aplicativo, pela CLI, pelo SDK ou pelo servidor MCP — é salva no `cameras.json` / `sensors.json`, e abrir o projeto em qualquer modo configura os dispositivos a partir dele. Antes, apenas o aplicativo salvava o projeto, e seu próximo salvamento sobrescrevia tudo o mais. Consulte [Um registro de projeto, todos os modos](#one-project-record-every-mode).
> - **Arquivos criados a partir de várias câmeras nomeiam cada uma delas.** Uma pilha multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), um índice dinâmico criado a partir das bandas de várias câmeras e a imagem `composite/` de uma captura em matriz contêm o número de série, o modelo e o pacote de calibração de cada câmera de origem na forma de listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), e as listas EXIF `SerialNumber` apresentam os números de série separados por vírgulas. Antes, elas não continham nenhuma câmera ou apenas o número de série da primeira câmera. Consulte [Arquivos criados a partir de várias câmeras](#files-made-from-several-cameras).
> - **Apenas um arquivo `raw` contém as tags de calibração da câmera.** O conjunto compatível com MicaSense (`BlackLevel`, `RadiometricCalibration`, o `Band{R,G,B}RadiometricCalibration` por canal, os conjuntos `MicaSenseVignetting*`) e a carga útil `CalibrationJSON` descrevem os valores brutos do sensor; portanto, eles são incluídos apenas no arquivo de captura `raw`. Todos os demais arquivos — os arquivos de captura `debayered`, `display`, `radiance`, `reflectance` e arquivos de índice, os quadros `lattice continuous` e `multi-stream`, e cada produto `process` — contém a identidade da câmera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, em uma captura, o próprio registro do quadro (`XMP-mapir:FrameJSON`: exposição, ganho, temperatura, formato de pixel, hora da captura). Na versão 1.2.0, eles também incluíam o conjunto de dados brutos, que uma ferramenta que o aplica (Pix4D, Metashape) aplicava a pixels que não são valores brutos. O `process` identifica a calibração desse arquivo por meio de seu número de série e do `CalBundleSha256`.
>
> **Outras alterações:**
>
> - **A taxa de quadros automática de um array se ajusta ao que o array fornece.** Uma taxa que perde mais de 1% de seus disparos é reduzida, avaliada com base em várias análises prolongadas em vez de apenas uma, e a taxa da qual ela foi reduzida é tentada novamente de tempos em tempos. Uma taxa definida pelo usuário permanece inalterada.
> - **A exposição automática está mais estável.** Ela se mantém constante sob iluminação constante, se estabiliza em segundos após uma mudança, ignora pequenos realces e faz com que a câmera volte a funcionar quando ela para de responder sob sol forte. Veja [Como a exposição automática se comporta](#how-auto-exposure-behaves).
> - **O `process` conta apenas suas imagens e gravações.** Ao executar novamente em uma pasta que contém o `camera_cal/` de um projeto, esses arquivos de calibração não são mais contados (“Encontradas 96 imagens” em um conjunto de 90 imagens), e um projeto que já os tenha incluído os descarta.
> - **`lattice multi-test` parte do estado de ligação das câmeras.** Ele ativa o PTP das câmeras para a execução, aguarda que seus relógios sejam sincronizados e estabilizados antes do primeiro disparo — cada câmera fica sincronizada por 120 s, cerca de 2 minutos a partir da ligação (`--ptp-settle SECONDS` define isso), já que um relógio recém-sincronizado apresenta um desvio de até ~200 us por cerca de 2 minutos — e restabelece o PTP de cada câmera como estava. Uma câmera ainda sincronizada de um teste anterior não é aguardada novamente. Anteriormente, ele desativava o PTP em uma rede com um único switch na qual nenhum DAQ-E respondia e interpretava o relógio de funcionamento livre de cada câmera como um relógio PTP que não se sincronizaria (`PTP is not locked`, a cada execução). Consulte [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **Uma gravação `.daq` indica se a tampa foi escolhida ou presumida.** Os modelos DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e o Chloros não consegue detectar se uma está instalada; portanto, um sensor conectado sem uma tampa nomeada recebe `sunshine_cosine` por padrão, uma *suposição*. Agora, todo registro do Chloros (esquema 1.30) armazena o código `cap_id_source` junto com o `cap_id`: `operator` quando você nomeou uma tampa (incluindo `none`), e `auto_default` quando foi o padrão. Ao processar um registro cuja tampa foi presumida, com a configuração de tampa do DAQ do projeto deixada em “Auto”, exibe um `[daq-dls] WARNING` nomeando o arquivo: um DAQ-U sem ajuste, sob a correção de luz solar, apresenta um valor cerca de 30 vezes maior. **Nomeie o limite que você ajustou** ao conectar ou defina o limite de DAQ do projeto, e o aviso desaparecerá.
> - **Não há mais janelas de console pip no Windows.** Em um PC com o Python instalado, o backend e cada um de seus processadores de imagem executavam o `pip` uma vez na inicialização, e cada um abria uma janela de console. Isso não acontece mais.
> - **Gravações ao vivo do `.daq` sobrevivem a uma queda de energia.**Uma gravação é gravada com um diário e uma sincronização completa do disco a cada commit, por trás do sensor, portanto, um disco lento atrasa o arquivo, mas nunca as leituras. Uma gravação interrompida no meio de um commit (queda de energia, encerramento forçado) pode deixar o `<name>.daq-journal` ao lado dela:**copie os dois juntos.** O Chloros reverte o diário na primeira vez que lê o arquivo, de modo que a gravação abre com tudo que foi confirmado antes da interrupção (cerca dos últimos 3 s são perdidos); na versão 1.2.0, uma queda de energia no momento errado poderia deixar o arquivo inteiro ilegível (“a imagem do disco do banco de dados está corrompida”).
> - **Uma gravação em um disco ocupado não bloqueia mais o sensor.** O `daq pool-record` enfileira as leituras e as grava em uma thread própria; uma leitura que não pôde ser gravada é contabilizada (o `daq pool-list` exibe o `N write error(s)` com o último erro) em vez de travar o sensor (uma gravação levou 8,4 s ao lado de quatro câmeras salvando arquivos TIFF).
> - **O `lattice multi-capture` sem uma matriz não perde mais os quadros que salvou quando um disparador atinge o tempo limite.** O gatilho de uma câmera que atinge o tempo limite (`SC_ERR_TIMEOUT -1011`, observado com quatro câmeras em um switch de 1 GbE) encerra a execução naquele grupo de quadros: os grupos já salvos são listados com `Saved:`, seguido do erro, saída `1`. Na versão 1.2.0, ocorria um erro HTTP500 sem lista, embora os arquivos estivessem no disco.
> - **As câmeras acionadas por software não descartam mais cada segundo acionamento.** Uma câmera no modo de acionamento por software agora enfileira um acionamento que chega durante sua leitura (ou o Chloros aguarda até que ela esteja pronta), assim como as câmeras sincronizadas por cabo já faziam.
> - **`daq pool-discover` nomeia um DAQ-E-S `DAQ-E-S`** em sua coluna `MODEL` (e o `--json` do `model`). Ele indicava `DAQ-E` para cada sensor Ethernet; aquele que não responde à verificação de identidade em cerca de 2 s ainda é listado como `DAQ-E`.
> - **Produtos de índices LATTICE aprimorados** (NDVI e outros índices) a partir de `process`, em todos os modos de processamento.
> - **Câmeras OCN: produtos de radiação, refletância e índices aprimorados sob luz solar.** É aplicada uma correção de frota após a separação de bandas; as câmeras RGN permanecem inalteradas, e os valores OCN diferem dos da versão 1.2.0. Isso se aplica também a uma captura OCN feita com uma versão anterior, quando ela é processada em um computador que possui a calibração com a qual a captura foi realizada (o Chloros mantém a calibração da câmera assim que ela é conectada).
> - **O `lattice continuous` grava o quadro bruto.** Uma execução do `tiff` gerou quadros demosaicados em 3, dos quais o `process` não pode partir. Seus quadros são agora o arquivo `raw` que uma captura em matriz grava: um plano mosaico com os valores do próprio sensor, não compactado, com o pacote de calibração. Um script que lia três planos desses arquivos passa a ler apenas um.
>
> - **Cada captura salva a leitura do DAQ utilizada, assim como a GUI faz.** `lattice capture`, `lattice continuous` (um `.daq` por quadro), `array-capture` e `multi-capture` com uma matriz conectada (e o SDK) gravam o `.daq` e seu `.match.json` sempre que um DAQ é vinculado ou agrupado, incluindo uma captura apenas em formato bruto; na versão 1.2.0, uma captura bruta ou com debayering salva apenas um com o `--force-daq`. O `--no-daq` desativa isso, o `--daq-tolerance-ms MS` define a tolerância de correspondência para essa captura, e o novo `daq pool-bind` vincula uma câmera ou um array a um DAQ a partir da CLI. Com o `.daq`, uma captura salva é combinada, no `process`, apenas com as câmeras que ela nomeia. Agora, uma captura aguarda a próxima leitura do DAQ para que possa combinar as leituras de ambos os lados da exposição. Consulte [A leitura do DAQ que uma captura salva](#the-daq-reading-a-capture-saves) e [Uma leitura do DAQ distante da exposição](#a-daq-reading-far-from-the-exposure).
> - **`process --indices NAME`** adiciona o índice ao projeto para as faixas de cada câmera, incluindo um novo projeto. Na versão 1.2.0, um nome que ainda não constava no projeto era ignorado com o `Unknown index name(s) ignored`.
> - **`process` ao lado de uma matriz ativa:**ele não reinicia mais um backend que contém câmeras ou DAQs e aguarda até 15 s por uma execução que está chegando ao fim.**Ctrl+C** interrompe a execução no backend, não apenas na CLI. Consulte [Notas sobre o `process`](#notes).
> - **Uma pasta de projeto com nível de hierarquia muito profundo para o limite de 260 caracteres do caminho do Windows** é rejeitada imediatamente, sendo substituída pela pasta mais longa que couber, em vez de uma execução que não grave nada. O instalador do Windows agora oferece a opção “Ativar caminhos longos do Windows” e uma exclusão do Microsoft Defender para o backend, ambas marcadas por padrão. Consulte [Windows (.exe)](#windows-exe).
> - **`lattice array-capture --continuous`** executa a captura contínua do próprio back-end: uma linha de progresso a cada 2 s, um resumo em vez de uma listagem por arquivo e um código de saída diferente de zero caso haja algum arquivo não gravado ou a unidade esteja cheia. Consulte [Modos do `array-capture`](#array-capture-modes).
> - **O primeiro `array-capture` após o `array-connect`** aguarda até 20 s pelo primeiro grupo de quadros da matriz, em vez de apresentar falha com o código HTTP 503 “Matriz ainda em aquecimento”.
> - **Exposição da matriz em uma ROI:** uma ROI pequena não limita mais a exposição de um membro da matriz ao período de funcionamento livre do sensor (4,9 ms a 768 × 304). Consulte [Exposição em um membro da matriz](#exposure-on-an-array-member).
> - **A exposição automática não reduz mais a taxa de quadros automática de um array.** Um array sincronizado por cabo opera na taxa permitida pela rede e pela leitura das câmeras; a exposição automática preenche o período correspondente a essa taxa e, em seguida, aumenta o ganho. Uma exposição manual mais longa que o período (`lattice exposure --set`, `configure --set`) retarda o disparo para se adequar a ela, em vez de ser recusada, e a taxa é restabelecida quando o membro retorna ao modo automático. Consulte [Exposição em um membro da matriz](#exposure-on-an-array-member).
> - **As matrizes geram menos grupos de quadros incompletos** e menos erros 503 de captura (“Nenhum grupo de quadros sincronizado recente”): a entrada de acionamento de cada câmera escrava agora está livre de interferências.
> - **Transporte compactado de 12 bits** (`--packed`, padrão `auto`) aumenta as taxas de quadros da matriz de sensor completo em cerca de um terço. O `array-connect --packet-size` agora está em vigor (a versão 1.2.0 o aceitou, mas o ignorou). Consulte [Transporte compactado de 12 bits](#packed-12-bit-transport---packed).
> - **Uma matriz cujas câmeras se recuperam sozinhas após reinicialização.** Após a reinicialização do switch, a matriz do `array-connect`-abre as câmeras quando elas respondem e retoma a transmissão com o PTP reativado; na versão 1.2.0, ele parava definitivamente e exigia um novo `array-connect`. O `array-status` reporta o `ptp_enabled: False` (e `ptp_off:`) enquanto uma câmera está com o PTP desativado, quando antes relatava a configuração com a qual o conjunto estava conectado. Consulte [Conexões Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **`lattice array-record` grava sem a GUI aberta.** Com a visualização combinada da GUI fechada, o backend renderiza a própria visualização: oNDVI ou o índice que você passa com o novo `--expr`. Uma gravação que não grava nenhum quadro agora falha (`no-frames: …`, saída `1`). Antes, ela gravava apenas enquanto a visualização combinada da GUI estivesse aberta, e uma gravação que não gravava nada ainda saía do `0`. Consulte [`array-record`](#array-record--combined-index-videogif-monitoring-grade).
> - **Uma câmera que o Chloros já mantém não é aberta uma segunda vez.** Um comando que a abriria por conta própria retorna, em vez disso, um código HTTP 409 `CAMERA_HELD`, indicando o que a mantém e o comando que a libera. Isso inclui uma câmera que outro comando está usando no momento (um `multi-test` ou `multi-capture` abrindo suas câmeras, um `lattice capture`): o segundo comando é recusado até que o primeiro seja concluído. Abri-la novamente poderia interromper a transmissão do detentor ou causar uma falha no backend. Consulte [Conexões Persistentes](#persistent-connections-smart-prep-gui-equivalent-flow).
> - **Comandos transitórios para múltiplas câmeras transmitem todas as câmeras ou falham.** `lattice multi-capture` sem matriz conectada, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream` e `index --live` saem do `1` ao nomear uma câmera que não transmitiu nenhum quadro. Os fluxos pulam um grupo de quadros em que falta uma câmera e param após três consecutivos; o `multi-test` imprime uma linha por acionamento, o `(no frame)` indica onde uma câmera falhou; o `align-stream` também sai do `1` devido a um erro na tarefa. Cada execução, então, restabelece todas as câmeras como as encontrou: sua fiação de acionamento e linha de sincronização, seu PTP (`PtpEnable`) e todas as configurações da câmera alteradas pela execução — exposição e ganho, alvo de AE, limites e amortecimento, balanço de branco, nível de preto, formato de pixel, binning e ROI, inversões, taxa de quadros — a menos que `--keep-settings` / `--clear-settings` (consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings)). Antes, esses comandos mantinham todos os quadros recebidos e saíam de `0` (`align-stream` exibia um erro de tarefa como `Note:`), e deixavam todas as câmeras em `TriggerMode` `On`. Eles também definiam a exposição de cada câmera com base na cena anterior à captura de um quadro (no máximo alguns segundos), no alvo em que uma matriz conectada inicia cada câmera em: 40 multiespectral, 60 RGB (o `--brightness N` define isso) e mantém esses valores durante a execução; o mesmo ocorre com um único `lattice capture` de uma câmera que o Chloros não mantém. Antes, o primeiro quadro de cada câmera era capturado no limite de exposição da própria câmera, cerca de 100 ms, independentemente da iluminação, e frequentemente ficava cortado. Veja [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **Os comandos transitórios de multicâmera compartilham a rede da mesma forma que o `array-connect` faz e abrem apenas o `--serials`.** Cada câmera é ajustada à sua parcela do link do host; assim, quatro câmeras operam em um link de 1 Gbps; antes, as quatro solicitavam 125 MB/s entre elas, totalizando 1 GbE, e cada quadro perdia pacotes. Um conjunto que a rede não consegue suportar é recusado antes que qualquer câmera comece a transmitir. O `--serials` agora seleciona as câmeras, sendo a primeira a acionar o gatilho; antes, todo comando abria todas as câmeras na rede, independentemente do que o `--serials` indicasse. Uma falha que é perda de pacotes do GVSP agora indica isso, em vez de apontar para o cabo de sincronização M8. Consulte [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions).
> - **O `lattice array-disconnect` deixa as câmeras operando livremente, conforme encontrado.** `TriggerMode`, `Off` e todo o restante conforme constatado pela conexão — as configurações de disparo e da linha de sincronização (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`), PTP e as configurações da câmera alteradas pela sessão — de modo que um únicocomando de câmera em um membro anterior obtém os quadros de uma só vez, e o mestre não fica mais em `TriggerSource` `Software`. `--keep-settings` salva as configurações da sessão na memória de cada câmera, enquanto o `--clear-settings` reinicializa essa memória (consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings)). O `lattice continuous` também configura uma câmera que encontrar armada para o modo livre durante a execução (e a coloca de volta no modo normal após o término), e uma execução sem quadros sai do `1`.
> - **`lattice configure --get` / `lattice exposure`** imprimem um `note:` com aexposição real ou ganho enquanto o modo automático está em funcionamento. Um `lattice exposure --set` / `configure --set` rejeitado indica o intervalo aceito pela câmera e reativa o modo automático, em vez de desativar a exposição automática. Consulte [Ajuste do sensor](#sensor-tuning).
> - **Novos controles da câmera:**`lattice highlights` (reduz o brilho de superfícies brancas na renderização da tela RGB) e `lattice exposure --brightness-auto`.**`lattice multi-detect`** localiza as câmeras conectadas por cabo em uma rodada de sondagem e define a câmera RGB como mestre por padrão.
> - **Novos comandos DAQ:** `daq pool-bind`; `pool-enable-ptp`; e, para as configurações próprias do DAQ-E, `pool-info`, `pool-set-network`, `pool-set-auto-stream`, `pool-reboot` e `pool-update-firmware`. Consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings).
> - **Um DAQ-E que para de transmitir é reconectado pelo backend.** Quando o fluxo de um DAQ-E em pool fica inativo (uma reinicialização do switch que o alimenta, uma queda na conexão da câmera), o backend o reconecta com as configurações anteriores, tentando novamente em intervalos crescentes enquanto ele estiver indisponível. Uma gravação nele continua no mesmo `.daq` com a lacuna marcada no arquivo, e o `pool-record --stop` lista cada lacuna. Na versão 1.2.0, o sensor permanecia inativo até o `pool-disconnect` + `pool-connect`, o que também encerrava a gravação. Veja [Um DAQ-E que para de transmitir](#a-daq-e-that-stops-delivering).
> - **`time-sync status`** informa se os pacotes do grandmaster PTP chegam à rede (`health`), com um `WARNING:` no stderr quando isso não ocorre. Consulte [`time-sync`](#chloros-cli-time-sync).
> - **.deb para Linux:** aumenta os buffers de recepção de soquetes do kernel necessários para os fluxos de vídeo das câmeras (na configuração padrão do kernel, os arrays perdem quadros e taxa de quadros), adiciona um serviço de back-end que é executado como seu usuário e o `lattice network` verifica os buffers. A CLI nunca interrompe um backend executado por um serviço, e uma atualização reinicia os serviços de backend habilitados. Consulte [Executando o backend como um serviço](#running-the-backend-as-a-service-headless) e [Hosts Linux — buffers de recepção de soquetes](#linux-hosts--socket-receive-buffers).
> - **`lattice array-capture` informa quando não foi possível alinhar:**cada motivo pelo qual uma captura foi salva desalinhada (ausência de perfil de alinhamento, membros em resoluções diferentes, uma matriz não sincronizada com o cabo, um quadro no qual o perfil não pode ser mapeado) é exibida como uma linha `Alignment:`.**`chloros-cli daq help`** é `daq --help` (ele saiu de `2`), e no Linux a ajuda lista os comandos de hardware direto `daq` que essa compilação inclui.
> - **Jetson:** o pacote é o `chloros_1.3.0_arm64_jp6.deb`, para o JetPack 6, e seu processamento por GPU é executado no Orin com o CUDA PyTorch do JetPack. Consulte [Linux arm64 — Jetson](#linux-arm64--jetson-jetpack-6).

---

## Convenções

- Todos os comandos têm o prefixo `chloros-cli`. No Windows, o binário é `chloros-cli.exe`; no Linux/Jetson, é `chloros-cli`.
- Argumentos opcionais são apresentados como `--flag`. Argumentos posicionais obrigatórios são apresentados sem colchetes.
- Quando um valor padrão é fornecido, omitir o sinalizador faz com que esse valor seja usado.
- A CLI é um cliente HTTP leve sobre o backend Chloros (o servidor em `127.0.0.1:5000` que o aplicativo de desktop também utiliza). **Quais comandos a iniciam:**
  - Os comandos principais — `process`, `login`, `logout`, `status`, `export-status`, `time-sync` e `selftest` — utilizam um backend em execução ou iniciam um quando nenhum estiver em execução e o param novamente ao concluir suas tarefas.
  - **Os comandos `lattice`, `project` e `daq` nunca iniciam um.** Eles precisam de um backend que já esteja em execução: o do aplicativo para desktop do Windows, o [serviço de backend](#running-the-backend-as-a-service-headless) do Linux ou um que você mesmo inicie (consulte [Iniciar o Backend](#iniciar-o-backend)). Sem um backend, eles exibem o código de erro `Error: Chloros backend not reachable at http://127.0.0.1:5000`, acompanhado do comando que o inicia, e encerram com o código de erro `1`.
- `CHLOROS_BACKEND_URL=<url>` direciona as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para outro backend (o `lattice network-analysis`, por sua vez, direciona para o `--backend-url`). Os comandos principais fixam deliberadamente o `http://127.0.0.1:<port>` e o ignoram (o literal IPv4 evita a penalidade de cerca de 2 s por solicitação do Windows para `localhost`→`::1`). Consulte [Variáveis de ambiente](#environment-variables).
- **Os caminhos relativos são resolvidos em relação à pasta a partir da qual você executa o `chloros-cli`.** O backend é o processo que abre as pastas de saída e entrada; portanto, antes de enviar um caminho, a CLI o torna absoluto: `-o` / `--output` (incluindo seu padrão, `output`), `--input`, `--burst-dir`, `--profile`, `--save` e `--mask`, um etapa [receita](#recipe-dsl) `output` (padrão `output`), e a pasta padrão `output` de `lattice index --live`. `-o captures/`, portanto, grava em `<current folder>/captures/`, e os caminhos impressos são absolutos. `~` é expandido. Em relação a um backend remoto (`CHLOROS_BACKEND_URL` indicando outra máquina), os caminhos são enviados conforme digitados e resolvidos nessa máquina.
- É necessário um login com conta Chloros+ para todas as chamadas de SDK/CLI (execute o `chloros-cli login` uma vez por máquina; armazenado em cache no `~/.chloros/`).
- Os exemplos utilizam caminhos do Linux; no Windows, substitua `/home/user/...` por `C:/Users/.../...`. A porta serial de um DAQ-Ué do tipo `COM3` no Windows e `/dev/ttyUSB0` / `/dev/ttyACM0` no Linux.
- As sinopses escrevem sinalizadores mutuamente exclusivos como `{--a | --b}` e as opcionais como `[...]`.

---

## Sinopse de nível superior

```
chloros-cli [global options] COMMAND [command options]
```

### Opções globais

| Sinalizador | Descrição |
| --- | --- |
| `--backend-exe PATH` | Substitui o executável do backend detectado automaticamente. |
| `--port N` | Porta HTTP do backend (padrão: `5000`). Usada apenas pelos comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`); `lattice`, `project` e `daq` utilizam `CHLOROS_BACKEND_URL` (`http://127.0.0.1:5000` por padrão). |
| `-v, --verbose` | Ativa a saída detalhada. |
| `--restart` | Reinicia à força o backend (encerra qualquer instância em execução de `backend_server.py`). Um backend executado por um serviço do systemd (.deb do Linux) nunca é interrompido: o comando exibe `sudo systemctl restart <unit>` e encerra `1`. |
| `--version` | Exibe a versão (`Chloros CLI 1.3.0`). |
| `--help` | Exibe a ajuda de nível superior. |

### Índice de comandos

| Comando | Finalidade |
| --- | --- |
| [`process`](#chloros-cli-process) | Processar uma pasta de capturas do Survey3 ou do LATTICE de ponta a ponta. |
| [`login`](#chloros-cli-login) | Autenticar esta máquina com uma conta do Chloros+. |
| [`logout`](#chloros-cli-logout) | Limpar as credenciais armazenadas em cache. |
| [`status`](#chloros-cli-status) | Mostrar o status atual da licença/autenticação. |
| [`export-status`](#chloros-cli-export-status) | Exibir o andamento da exportação do Thread-4 em tempo real durante uma execução do `process`. |
| [`language`](#chloros-cli-language) | Definir ou listar o idioma de exibição da CLI (38 idiomas suportados). |
| [`set-project-folder`](#project-folder-commands) / [`get-project-folder`](#project-folder-commands) / [`reset-project-folder`](#project-folder-commands) | Pasta padrão do projeto (compartilhada com a GUI). |
| [`update`](#chloros-cli-update) | Verificar e instalar atualizações do Chloros (Linux/Jetson). |
| [`selftest`](#chloros-cli-selftest) | Diagnósticos do sistema + testes de funcionamento básico. |
| [`time-sync`](#chloros-cli-time-sync) | Status/controle do grandmaster PTP. |
| [`lattice`](#chloros-cli-lattice) | Controle e captura da câmera LATTICE (mais de 45 subcomandos). |
| [`daq`](#chloros-cli-daq) | Controle de sensores espectrais DAQ (DAQ-U / DAQ-M / DAQ-E). |
| [`project`](#chloros-cli-project) | Abrir e executar um projeto Chloros salvo (câmeras + DAQs). |

---

## Instalação

O `chloros-cli` vem incluído no instalador do Chloros para desktop em todas as plataformas compatíveis — não há download separado da CLI. A instalação do pacote da plataforma adiciona o `chloros-cli` ao seu `PATH`, juntamente com o aplicativo para desktop e o binário de back-end que ele controla.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

> O instalador também inclui scripts de inicialização práticos (`Chloros_CLI.bat` / `Chloros_CLI.ps1`, `Launch_CLI.*`, `chloros-cli.sh`) que abrem um shell de CLI pronto para uso.

### Windows (.exe)

1. Baixe o instalador do Windows na página de downloads.
2. Execute o `Chloros Installer 1.3.0.exe` e siga as instruções do assistente. O caminho de instalação padrão é `C:\Program Files\Chloros\` (o CLI fica em `C:\Program Files\Chloros\cli\`, que o instalador adiciona ao PATH).
3. Abra um novo terminal (`cmd.exe`, PowerShell ou Terminal do Windows) para que o `PATH` atualizado seja reconhecido.

```powershell
chloros-cli --version
```

O instalador adiciona automaticamente o `chloros-cli.exe` ao seu sistema `PATH` e inclui os drivers da câmera LATTICE. O backend é o `C:\Program Files\Chloros\backend\chloros-backend.exe` (a pasta `backend` da sua pasta de instalação), que não está no `PATH`.

**Configurações do sistema.** A página *Selecionar tarefas adicionais* do instalador possui duas opções em **Configurações do sistema**, ambas**marcadas por padrão**:

| Opção (conforme descrita no instalador) | O que ela faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** (permite que as pastas de projeto tenham mais de 260 caracteres; recomendado) | Define o parâmetro `LongPathsEnabled` do Windows, válido para todo o computador, como 1. O Windows vem com essa opção desativada, e, nesse caso, nenhum caminho pode ultrapassar 260 caracteres; os produtos da Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto, portanto, uma pasta de projeto mais profunda é rejeitada (consulte [Solução de problemas](#dicas-de-solução-de-problemas)). Entra em vigor para programas iniciados após a instalação. Permanece ativado na desinstalação, pois outros programas podem depender dessa configuração. | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender**(captura de imagem muito mais rápida; recomendado) | Adiciona uma exclusão de**processo** do Defender para o `chloros-backend.exe` na pasta de instalação, de modo que a proteção em tempo real não verifique cada arquivo que o backend grava, independentemente de onde ele seja gravado. Nenhum outro programa é excluído; a exclusão é removida na desinstalação. Quando o Defender estiver ausente ou bloqueado por política, a etapa falha silenciosamente e a instalação continua. | `/MERGETASKS="!defenderexclusion"` |**Por que a opção do Defender é importante:** em altas taxas de captura, a verificação em tempo real dos arquivos capturados consome cerca de 30% da taxa de captura (medido em uma captura contínua com 9 câmeras: 77,7 capturas/s excluídas, 52,7 verificadas). O `lattice array-capture --continuous` exibe uma nota quando detecta que o Defender está utilizando um núcleo da CPU ou mais durante a captura; execute novamente o instalador com a opção marcada.

Uma instalação silenciosa (`/SILENT` ou `/VERYSILENT`) aplica ambas as opções, a menos que sejam desativadas, por exemplo, `"Chloros Installer 1.3.0.exe" /VERYSILENT /MERGETASKS="!defenderexclusion"`.

**Firewall do Windows.** O instalador sempre adiciona duas regras de entrada para o `chloros-backend.exe`, em todos os perfis de rede, e o desinstalador as remove: `Chloros Backend (UDP-In)`, para fluxos de câmera, PTP, mDNS e o fluxo de dados do DAQ-E; e `Chloros Backend (TCP-In)`, limitada à sub-rede local, para uma atualização de firmware do DAQ-E, na qual o sensor se reconecta a este computador para coletar a imagem (consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings)). A API HTTP do backend permanece vinculada a `127.0.0.1`.

### Linux amd64 (.deb)

Para estações de trabalho com Ubuntu 22.04 LTS ou mais recente / baseadas em Debian com arquitetura x86_64.

> **O Ubuntu 20.04 não é compatível.** A lista de dependências do pacote é derivada
> do que o backend realmente vincula, e isso inclui o `libc6 (>= 2.35)`;
> o Focal vem com a glibc 2.31. O `apt` recusa a instalação em vez de permitir que ela falhe em
> tempo de execução. A lista também menciona o `libgl1`, que o OpenCV incluído carrega e que pode estar ausente em uma
> imagem mínima de servidor; o `apt install` o fornece.

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
```

O pacote .deb instala:
- `chloros-cli` até `/usr/bin/chloros-cli` e `chloros-backend` (inicia o backend) até o `/usr/bin/chloros-backend`
- O backend compilado e a CLI, descompactados, em `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`
- Os drivers da câmera LATTICE
- Modelos de redução de ruído, pacotes de calibração e configuração do canal de atualização
- O SDK Python para o sistema `python3` (`import chloros_sdk`). O `apt install` também traz seus pré-requisitos, `python3-numpy` e `python3-requests` (recomendados, não obrigatórios: os programas acima são autônomos; o `dpkg -i` os ignora, e a saída da instalação indica o que está faltando). Em um virtualenv: `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`
- `/etc/sysctl.d/60-chloros-gige.conf`, que aumenta os buffers de recepção de soquetes do kernel para fluxos de câmera para 32 MB e os aplica de uma só vez (um valor já definido como maior é mantido). A remoção do pacote remove o arquivo. Consulte [Hosts Linux — buffers de recepção de soquetes](#linux-hosts--socket-receive-buffers).
- Dois serviços opcionais do systemd para o backend, nenhum deles ativado (próxima seção)

#### Executando o backend como um serviço (sem interface gráfica)

Sem um serviço, o backend (e, com ele, o grandmaster PTP) é executado apenas enquanto algo o mantiver em execução: `chloros-backend` em um terminal, um script do SDK ou um comando principal `chloros-cli` durante todo o tempo em que estiver em execução. Para um backend sempre ativo em uma máquina sem interface gráfica, habilite o serviço **como seu usuário**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

O `chloros-backend@<user>.service` executa o backend como esse usuário. As capturas que ele grava pertencem a esse usuário, seu log detalhado está no `~/.cache/chloros/logs` dele, e ele usa o `chloros-cli login` (`~/.chloros`) desse usuário; portanto, faça login uma vez como esse usuário primeiro. O PTP não precisa de privilégios de root. Assim como todo backend instalado, ele escuta apenas no `127.0.0.1:5000`.

O `chloros-backend.service` executa o backend como **root**. Ele ainda está instalado para que uma máquina que o tenha habilitado continue funcionando, mas tudo o que um backend root grava pertence ao root, portanto, um usuário não pode excluir suas próprias capturas. Para alternar para o seu usuário:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

O log detalhado do serviço root está em `/var/log/chloros`, legível por membros do grupo `adm` (o serviço define `CHLOROS_LOG_DIR`; consulte [Variáveis de ambiente](#environment-variables)). A instalação do pacote exibe o comando `enable` com seu nome de usuário preenchido, bem como a opção de alternância quando o serviço root estiver habilitado.

**Habilite uma unidade de back-end.** Apenas um backend é executado por vez. Uma segunda unidade habilitada (`chloros-backend.service` além de `chloros-backend@<user>.service`, ou duas unidades `@` de usuários diferentes) se recusa a iniciar e permanece como `failed` com status de saída 98; o systemd não a reinicia. Seu `systemctl status` explica o motivo: `Another instance is already running. Exiting.`, ou seja, o que está retendo o backend (a porta 5000, ou o arquivo de bloqueio e o usuário a quem ele pertence), e, em seguida, o `systemd will not restart <unit> (exit status 98)`. Após uma reinicialização, qualquer uma das unidades pode ser a que será executada. Para alternar, desative aquela que você não usa, conforme descrito acima.**A CLI nunca interrompe o backend de um serviço.** O `chloros-cli` o utiliza como está, assim como a GUI e o SDK: o `process` é executado nele, carregando seu próprio projeto no lugar do último. Nos casos em que a CLI, de outra forma, reiniciaria o backend (`--restart` ou um `process` cuja importação não encontrou nada), ela nomeia a unidade, exibe `sudo systemctl restart <unit>` e encerra o `1`, pois um backend parado dessa forma permaneceria inativo. O `GET /api/status` nomeia a unidade como `service` (`null` para um backend no qual nenhum serviço está em execução).**Atualização e remoção.** A atualização do pacote interrompe todos os serviços de backend e, em seguida, reinicia cada um dos que estão habilitados, começando pelas unidades `@<user>` (um serviço iniciado manualmente, mas nunca habilitado, permanece parado, como após uma reinicialização). Com mais de um habilitado, apenas um é executado e os demais permanecem com falha, conforme descrito acima; a atualização exibe `Chloros: N backend units are enabled: …` e `Keep one enabled: sudo systemctl disable --now <unit>`. A remoção interrompe e desabilita esses serviços. A purga também exclui o `/var/log/chloros`.

### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Mesmo layout que o .deb para amd64, incluindo buffers de recepção e [serviços de backend](#running-the-backend-as-a-service-headless), para Jetson Orin / Orin NX / Orin Nano no **JetPack 6** (L4T R36). Seu PyTorch é a compilação CUDA do JetPack 6 para o Orin; portanto, o processamento por GPU — incluindo o debayer com reconhecimento de textura — é executado na GPU do Jetson. O pacote não inclui o CUDA em si: ele utiliza as bibliotecas CUDA e cuDNN instaladas pelo JetPack 6 (`sudo apt install nvidia-jetpack` em uma imagem que não as possua). Quando nenhuma GPU com CUDA estiver disponível, o processamento é executado na CPU e o processamento com reconhecimento de textura utiliza o debayer padrão (consulte [`process`](#chloros-cli-process)).

### Autenticação única por máquina

Todas as plataformas exigem um login único no Chloros+ antes que as chamadas do SDK ou da CLI funcionem:

```bash
chloros-cli login user@example.com 'YourPassword'
```

As credenciais são armazenadas em cache no `~/.chloros/user_session.json`.

### Iniciar o backend

Os comandos `lattice`, `project` e `daq` se comunicam com um backend que já está em execução; eles nunca iniciam um (consulte [Convenções](#convenções)). Mantenha um em execução enquanto os utiliza:

| Plataforma | Como |
| --- | --- |
| Windows | Abra o aplicativo de desktop Chloros (seu backend também atende à CLI), ou execute o `"C:\Program Files\Chloros\backend\chloros-backend.exe"` em um terminal próprio (na pasta `backend` da sua pasta de instalação) e deixe-o aberto. |
| Linux/Jetson | Execute o `chloros-backend` em um terminal próprio ou mantenha um sempre em execução com o [serviço de back-end](#running-the-backend-as-a-service-headless). |

Apenas um backend é executado por vez; todos os clientes (a GUI, a CLI e os scripts do SDK) o compartilham. Os comandos principais (`process`, `login`, `status`, …) também o utilizam e iniciam um quando nenhum estiver em execução.

**Um backend continua em execução entre os comandos**, de modo que o próximo não precisa pagar pelo seu tempo de inicialização novamente. Ele é interrompido quando você sai da conta (`chloros-cli logout`, a menos que o aplicativo Chloros esteja aberto ou uma execução ou compilação offline esteja em andamento), quando você executa o `chloros-cli backend stop`, ou automaticamente após 30 minutos sem que nada o esteja utilizando: nenhum comando, script ou aplicativo, nenhuma execução ou compilação offline, nenhuma câmera conectada ou sensor de luz. O `backend stop` recusa enquanto uma execução ou compilação offline estiver em andamento, indicando o nome; o `--force` o interrompe imediatamente e essa execução ou compilação é perdida (uma sequência é mantida, e o `array-build-video` a compila novamente). Configure o `CHLOROS_BACKEND_IDLE_MIN` antes que o backend inicie para alterar os 30 minutos (`0`: nunca); o [serviço backend](#running-the-backend-as-a-service-headless) nunca para por conta própria. O `chloros-cli backend status` mostra se há algum em execução, o que o está utilizando e se há alguma compilação offline em andamento. O `process` abre seu projeto no backend em execução, a menos que o aplicativo Chloros esteja aberto com um projeto que ele abriu: nesse caso, ele recusa, e você deve processar a partir do aplicativo ou fechar esse projeto primeiro. Um projeto que um `process` anterior ou um script do SDK tenha deixado aberto é simplesmente substituído.

### Verificar a instalação

```bash
chloros-cli --version           # prints "Chloros CLI 1.3.0"
chloros-cli selftest            # 7-step diagnostic (backend, GPU, models, CUDA)
chloros-cli status              # shows license tier + logged-in user
```

O `selftest` encerra o `0` somente quando todas as sete verificações forem aprovadas. A última delas requer uma GPU CUDA (uma GPU NVIDIA, ou de um Jetson Orin) e os modelos de redução de ruído; portanto, em um computador sem essa GPU, ele exibe o código `Failed: CUDA + Denoiser` e encerra o `1`; tudo, exceto o debayer sensível à textura, que depende exclusivamente da GPU, ainda funciona nesse caso.

> **É necessária uma assinatura do Chloros+.**A CLI requer um plano Chloros+ ativo.**Copper**é o nível básico do Chloros+ — todos os níveis pagos do Chloros+ têm acesso à CLI/SDK; apenas o nível gratuito**Iron** não tem. (Mapeamento de ID do plano: `0`=Iron/gratuito, `1`=Copper, `2`=Bronze, `3`=Silver, `4`=Gold.) Faça o upgrade em [`https://cloud.mapir.camera/pricing`](https://cloud.mapir.camera/pricing).
>
> Esse limite mínimo é imposto pelo backend, não apenas pela CLI: uma solicitação marcada pelo SDK/CLI sem um plano pago é rejeitada com o código `403 PLAN_UPGRADE_REQUIRED`, independentemente de vir do `chloros-cli`, do Python SDK ou de um cliente HTTP personalizado. Um usuário desconectado recebe o código de erro `401 AUTH_REQUIRED`. O acesso funciona offline durante o período de carência do plano (30 dias para planos mensais; até o vencimento para planos anuais) e é interrompido quando esse prazo expira; o código `chloros-cli status` continua funcionando para que o motivo fique visível (essa é a única rota de SDK/CLI isenta da restrição de nível — `GET /api/license-status`).

---

## Guia de Início Rápido em Cinco Minutos

```bash
# 1. Sign in once on this machine
chloros-cli login user@example.com 'YourPassword'

# 2. Survey3 / LATTICE folder → finished radiance + NDVI in one call
chloros-cli process "/home/user/captures/flight_001" \
  --vignette --reflectance --indices NDVI GNDVI OSAVI

# 3. The lattice / daq commands need a running backend: on Windows open the
#    Chloros desktop app; on Linux start one (or see "Start the Backend")
chloros-backend &

# 4. Take a single LATTICE photo with the first camera found
chloros-cli lattice capture -o output/

# 5. Connect a 4-cam LATTICE array with the GUI's smart-prep flow
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 6. Read a spectrum from a connected DAQ-U (Windows: --port COM3)
chloros-cli daq pool-connect --port /dev/ttyUSB0
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F   # id from 'daq pool-list'
```

O `process` (etapa 2) inicia um backend para sua própria execução quando nenhum estiver em execução e o interrompe quando terminar; portanto, inicie um a partir da etapa 4. Consulte [Iniciar o back-end](#start-the-backend).

---

## `chloros-cli process`

Processe uma pasta de imagens por meio do pipeline completo do Chloros (detecção de alvo → calibração → vinheta → refletância → exportação de índice).

### Sinopse

```
chloros-cli process INPUT [OPTIONS]
```

### Argumentos posicionais

| Argumento | Descrição |
| --- | --- |
| `INPUT` | Caminho para a pasta de entrada contendo os arquivos `.raw + .jpg` (Survey3), `.tif/.tiff` (LATTICE) ou `.dng`. |

### Opções comuns

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `-o, --output PATH` | uma nova pasta com o nome `-n` na pasta padrão do projeto | A pasta do projeto. O projeto recebe esse nome, e uma pasta que já contenha um `project.json` recebe um `_1` / `_2` em vez de ser sobrescrita. A pasta padrão do projeto é aquela escolhida pelo `set-project-folder` (ou pela interface gráfica), caso contrário, é o `Documents/MAPIR/Chloros Projects`. |
| `-n, --project-name NAME` | automático (carimbo de data/hora) | Nome do projeto, usado apenas **sem** `-o` (com `-o`, o projeto recebe o nome da pasta). |
| `--debayer {standard,texture-aware}` | `standard` | `texture-aware` utiliza um debayer neural do Chloros+ para imagens do Survey3; mais lento, mas com maior qualidade. É exclusivo para o Survey3: As capturas LATTICE sempre utilizam o demosaico padrão; portanto, para uma pasta contendo capturas LATTICE, o `process` rejeita o `texture-aware` após a importação, exibindo um erro que indica quantas imagens existem, e encerra o `1` antes de processar qualquer coisa (o aplicativo Chloros também não oferece o recurso “Texture Aware” para elas). É necessária uma GPU com CUDA (uma GPU NVIDIA no Windows/Linux x64, ou a GPU do Orin em um Jetson): sem ela, a execução usa o `standard`, e o backend registra uma linha no log do `[DEBAYER] … needs a CUDA GPU …` explicando o motivo (mostrado com o `--verbose`). |
| `--vignette / --no-vignette` | `--vignette` | Correção de vinheta. |
| `--reflectance / --no-reflectance` | `--reflectance` | Calibração de refletância (usa o alvo do painel, se encontrado; calibração NIST por série para LATTICE). Para o LATTICE multiespectral, isso também funciona como o **produto** de refletância — consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--ppk` | desativado | Aplica correções PPK GNSS a partir de arquivos sidecar. |
| `--exposure-pin-1 CAMERA_MODEL` | nenhum | O modelo da câmera conectado ao pino de exposição 1, para sincronização de tempo PPK. |
| `--exposure-pin-2 CAMERA_MODEL` | nenhum | O modelo da câmera conectado ao pino de exposição 2, para sincronização de tempo PPK. |
| `--recal-interval SECONDS` | `0` | Intervalo mínimo de recalibração, em segundos (o *Intervalo mínimo de recalibração* do projeto). |
| `--timezone-offset HOURS` | `0` | Desvio do fuso horário do sensor de luz em relação ao UTC, em horas (o *Desvio do fuso horário do sensor de luz* do projeto). |
| `--format FORMAT` | `TIFF (16-bit)` | Um dos seguintes: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)`. O Chloros nunca aumenta a profundidade de bits de uma imagem: em imagens totalmente de 8 bits, um formato com maior profundidade é gravado como `TIFF (8-bit)`, e o código indica isso uma vez. Em um projeto que mistura capturas de 8 bits e de 12/16 bits, cada captura é exportada com sua própria profundidade ou uma profundidade inferior. O Radiance é sempre de 32 bits em formato flutuante. |
| `--indices NAME [NAME ...]` | nenhum | Índices de vegetação por nome de predefinição (`NDVI`, `GNDVI`, `OSAVI`, `SAVI`, `EVI`, `GCI`, …), sem distinção entre maiúsculas e minúsculas. Cada um é adicionado ao projeto para as bandas da respectiva câmera, incluindo um novo projeto. Um nome que não seja uma predefinição, ou que as bandas da câmera não possam gerar (`NDRE` requer RE e NIR, que nenhuma câmera fornece sozinha), é indicado em um aviso e não recebe nenhuma imagem de índice. |
| `--input-level {auto,raw,debayered,processed}` | `auto` | Forçar o ponto de entrada do pipeline para arquivos TIFF do LATTICE (os arquivos .raw do Survey3 não são afetados). Além disso, a saída de emergência que permite que uma captura **sem raw** seja processada — consulte [Como é uma pasta de capturas](#what-a-captures-folder-looks-like). |
| `--debayered / --no-debayered` | ativado | Emite o produto com debayering linear (`Debayered_Images`). Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--preview / --no-preview` | ativado | Emite a pré-visualização da tela (`Preview_Images`): RGB = balanço de branco (iluminante DAQ, quando disponível; caso contrário, mundo cinza) + gama; multiespectral = alongamento de cores falsas. |
| `--radiance / --no-radiance` | ativado | Emite radiancia em float32 (`Radiance_Images`, W/m²/sr/nm). |
| `--reflectance-source {daq,target,auto}` | `auto` | Referência para o produto de refletância LATTICE: `auto` = a meta dentro do quadro aprovada no controle de qualidade (QA) é a referência absoluta (uma meta com um sinalizador de QA compete com o DAQ na pontuação medida); recurso de fallback para o DAQ de radiação descendente (ρ = π·L/E); `target` = estrito (sem substituição do DAQ); `daq` = autoridade do DAQ. Consulte [Opções de exportação por produto](#per-product-export-toggles-lattice-multispectral). |
| `--target-reflectance-dir DIR` | nenhum | Diretório de varreduras de refletância-alvo **medidas** por unidade (`<serial>.csv`); recorre aos espectros nominais T3/T4P em caso de falha. |
| `--array-alignment / --no-array-alignment` | ativado | Matrizes LATTICE: aplica o alinhamento módulo a módulo registrado no XMP de cada captura (`Chloros:Alignment*`) a todos os produtos processados (desbayering / visualização / radiação / refletância / índice). Não realiza nenhuma operação para imagens sem as tags. |
| `--array-alignment-crop / --no-array-alignment-crop` | recorte | Recorte as exportações alinhadas à região de sobreposição comum da matriz para que todos os módulos compartilhem uma única área de cobertura; `--no-…` mantém a tela completa do sensor (preenchimento em preto fora da fonte). |
| `--array-alignment-interp {bilinear,nearest,cubic}` | `bilinear` | Reamostragem para a distorção de alinhamento. `nearest` preserva os DNs exatos da fonte (sem mistura entre pixels de valores radiométricos). |

### Opções de detecção de alvo

| Sinalizador | Descrição |
| --- | --- |
| `--min-target-size PIXELS` | Tamanho mínimo do alvo de calibração em pixels para o detector. |
| `--target-clustering 0-100` | Porcentagem de agrupamento de alvos (padrão: `60`). |
| `--target / --targets` | Pesquisar alvos de calibração apenas em uma subpasta `target` ou `targets` da entrada (mais rápido). |

### Exemplos

```bash
# Simplest: defaults are good for most surveys
chloros-cli process "/home/user/images/survey_001"

# Multi-index with explicit format
chloros-cli process "/home/user/images/survey_001" \
  --vignette \
  --reflectance \
  --format "TIFF (32-bit, Percent)" \
  --indices NDVI GNDVI OSAVI

# Texture-aware debayer for highest quality (Chloros+ only)
chloros-cli process "/home/user/images/survey_001" \
  --debayer texture-aware \
  --indices NDVI

# Process LATTICE captures explicitly (auto-detects from EXIF normally)
chloros-cli process "/home/user/captures/lattice_flight" \
  --input-level processed

# LATTICE multispectral → float32 radiance only (no DAQ downwelling needed)
chloros-cli process "/home/user/captures/lattice_flight" \
  --no-debayered --no-preview --no-reflectance

# LATTICE reflectance anchored to an in-frame target (strict, no DAQ fallback),
# with per-unit measured target scans looked up by serial
chloros-cli process "/home/user/captures/lattice_flight" \
  --reflectance-source target --target-reflectance-dir "/home/user/target_scans"

# LATTICE array capture: keep native geometry (ignore stamped alignment)
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment

# Aligned, uncropped, value-preserving resampling
chloros-cli process "/home/user/captures/array_flight" \
  --no-array-alignment-crop --array-alignment-interp nearest

# Save to a project folder of your choice (the project is named after it)
chloros-cli process "C:/input" -o "C:/projects/Field_A_2026-05-26"

# Name the project and let it go to the default project folder
chloros-cli process "C:/input" -n "Field_A_2026-05-26"
```

### Opções de exportação por produto (LATTICE multiespectral)

O processamento do LATTICE se ramifica para **todos os produtos aplicáveis em uma única passagem**. As quatro opções por tipo — `--debayered`, `--preview`, `--radiance`, `--reflectance` — estão todos**ativados por padrão**; use o formulário `--no-<type>` para desativar um deles. As câmeras mestras RGB sempre emitem apenas imagens sem debayering + pré-visualização (sem radiação/refletância por banda), portanto, `--radiance`/`--reflectance` não têm efeito para elas. Os alternadores são ignorados para o Survey3 `.raw` (que segue o caminho padrão de refletância/alvo). *(O antigo sinalizador `--radiometric-output {reflectance,radiance,sensor-response}` foi **removido** e substituído por esses botões de alternância; não existe mais o nível `sensor-response`.)*

| Produto | Saída | É necessária aquisição de dados de downwelling? |
| --- | --- | --- |
| `--debayered` | Desmosaico linear (`Debayered_Images`): o mesmo arquivo que uma captura salva em seu nível `debayered` — o deslocamento de escuro (DSNU) do pacote de calibração aplicado ao raw, seguido do demosaico sensível a bordas do OpenCV; sem LUT de linearidade, campo plano ou separação de cores. | Não. |
| `--preview` | Visualização na tela (`Preview_Images`): RGB = WB + gama; multiespectral = alongamento em cores falsas. | Não. |
| `--radiance` | float32 W/m²/sr/nm proveniente da cadeia radiométrica completa (`Radiance_Images`). | N.º |
| `--reflectance` | uint16 refletância ρ (`32768` = 1,0), Compatível com Pix4D. | **Sim**, a menos que um alvo dentro do quadro aprovado no controle de qualidade (QA) o ancore (veja abaixo). |

`--reflectance-source` seleciona a referência de refletância:**`auto`**(padrão) define um alvo dentro do quadro que passou no controle de qualidade como a**referência absoluta**— as cadeias de linhas empíricas ancoradas no alvo são comparadas em painéis de teste e o resultado vencedor é aplicado, faixa por faixa: uma faixa que o DAQ não consegue calibrar, ou cujo ajuste de inclinação do DAQ falha, adota a própria linha do alvo (`ReflectanceElBandMethods` indica qual) — um alvo que passa no controle de qualidade com uma marcação (um painel descartado, uma banda fora de ordem) também compete com a divisão do DAQ com base nessa pontuação medida, e o quadro ao qual ele perde é marcado como `ReflectanceChain=daq` — recorrendo à divisão descendente do DAQ (ρ = π·L/E) quando não há alvo presente ou o controle de qualidade falha;**`target`**é estrito (sem substituição do DAQ);**`daq`**opta pelo comportamento autoritário do DAQ. A geometria do alvo (ArUco / ROI fixa / faixa) provém da configuração do alvo do projeto; `--target-reflectance-dir DIR` mantém, por unidade,**varreduras ** por unidade (`<serial>.csv`), consultados pelo número de série/QR da unidade-alvo, com os espectros nominais T3/T4P como alternativa.

O caminho de refletância do DAQ resolve automaticamente a **radiação descendente com carimbo de data/hora correspondente**a partir de um**`.daq`**(DAQ-U/M/E) registrado**ou um `.csv` nativo do DAQ-M**encontrado junto com as imagens. Se um pacote de calibração por câmera ou DAQ não estiver armazenado em cache localmente, o pipeline**o busca automaticamente na AWS** na primeira utilização (requer conexão com a internet uma vez; armazenado em cache sob o `~/.chloros/`).**De quem é essa leitura?**Um `.daq` salvo por uma captura é a leitura da(s) câmera(s) que ele descreve — nomeada(s) em um arquivo sidecar `<file>.daq.cameras.json` ao lado dele (`lattice capture`, `multi-capture`, `array-capture` e na GUI; uma sequência bruta nomeia a leitura de cada membro em seu manifesto) — e o `process` a associa**apenas às imagens dessas câmeras**. Uma câmera autônoma, sem leitura própria, portanto, não utiliza mais o DAQ de outra câmera; vincule um DAQ a ela ou faça uma gravação. A gravação própria de um DAQ (`daq pool-record`, ou uma gravação iniciada na GUI), ou qualquer `.daq` sem esse arquivo auxiliar (incluindo capturas mais antigas), ainda atende a todas as câmeras.

#### Leitura de pixels de refletância (Pix4D / Metashape / seus próprios scripts)

A refletância é armazenada como um DN inteiro, e **o DN que corresponde a ρ = 1,0 depende da câmera de origem**:

| Origem | ρ = 1,0 é | Como identificar |
| --- | --- | --- |
| LATTICE (M3C / M3M) | `32768` (margem até ρ 2,0) | O arquivo contém a marcação XMP `Chloros:PixelScale=32768`. |
| Survey3 | `65535` (limitado a ρ 1,0) | Não há tags XMP `Chloros:*` — essa ausência *é* o indício. |

**Leia `Chloros:PixelScale` e divida por esse valor**, em vez de assumir uma constante. A tag é definida no domínio uint16, portanto permanece como `32768` em formatos de saída que redimensionam — `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)`, `JPG (8-bit)` e `TIFF (32-bit, Percent)` são todos autodescritivos (normalize primeiro o tipo de dados armazenado de volta para uint16: ×257 a partir de 8 bits, ×65535 a partir de float).

> **Arquivos de compilações mais antigas podem não conter escala.** Um TIFF de refletância de 8 bits é redimensionado e rotulado como `Chloros:PixelScale`, assim como os demais. Compilações anteriores à versão 1.3.0, em vez disso, *recortavam* esse intervalo para 0..255; assim, todo valor acima de ρ≈0,008 era nivelado para 255, e omitiam tanto o `Chloros:PixelScale` quanto o tuplo `MicaSense:RadiometricCalibration`. **Se a tag estiver ausente em um arquivo de refletância LATTICE, não presuma uma escala — reprocesse-o** em vez de dividir pixels que nunca foram divisíveis.

#### Leitura de pixels de índice (índice NDVI e outros índices de vegetação)

**Um raster de índice inteiro é um alongamento de contraste por imagem, não um mapa no intervalo −1…1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — incluindo o arquivo TIFF de 8 bits no qual uma captura de 8 bits é gravada — armazenam

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

onde `IndexMin` / `IndexMax` são **os valores de índice finitos mais baixo e mais alto daquele quadro**. DN 0 não é −1, e a escala total não é +1.

O Chloros marca ambos os extremos para que o mapeamento permaneça invertível:

| Tag | Significado |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — a convenção abaixo |
| `Chloros:IndexMin` | o valor de índice armazenado como DN 0 |
| `Chloros:IndexMax` | o valor de índice armazenado como escala total |

Leia-os e inverta, normalizando o DN armazenado para 0…1 pelo máximo do tipo de dados (dtype) armazenado primeiro, de modo que uma única expressão abranja todos os formatos inteiros:

```python
import json, subprocess
import numpy as np, tifffile

tags = json.loads(subprocess.run(
    ["exiftool", "-j", "-XMP:all", path],
    capture_output=True, text=True).stdout)[0]

# float(): exiftool hands back a negative real as a quoted string, so
# IndexMin arrives as str on most real NDVI data while IndexMax is a number.
lo, hi = float(tags["IndexMin"]), float(tags["IndexMax"])

raw = tifffile.imread(path)                    # uint16 for TIFF (16-bit)
ndvi = lo + raw.astype(np.float64) / np.iinfo(raw.dtype).max * (hi - lo)
```

A leitura não requer `-config pix4d.config` — o exiftool lê essas propriedades XMP por conta própria. A configuração é necessária apenas para *gravá-las*.

> **`IndexMin == IndexMax`** significa que cada pixel daquele quadro possuía o mesmo valor de índice. A trama está preenchida na escala média e não há nada pelo qual dividir — use o valor extremo.

> **A ausência de tags `Chloros:Index*` significa que não há mapeamento definido.** Uma exportação LUT é uma renderização colorida cujos pixels são cores em gradiente vinculadas à sua janela de limiar LUT, e um quadro sem pixels de índice finito não possui intervalo a ser definido. Não presuma um mapeamento em nenhum dos casos — a mesma regra que para um `PixelScale` ausente.**Como o alongamento é por imagem, dois quadros de um voo não são comparáveis em termos de DN**— cada um carrega seus próprios pontos finais, de modo que o mesmo NDVI resulta em DNs diferentes em imagens diferentes. Para trabalhos quantitativos em um voo (estatísticas zonais, limiares fixos, séries temporais), exporte**`TIFF (32-bit, Percent)`**: as rasters de índice são gravadas ali como float32**os próprios valores de índice**, sem necessidade de reescalonamento nem tags. Apesar do nome do formato, os produtos de índice não são ×100 — esse escalonamento se aplica apenas à refletância com tipo de dado inteiro.

#### Máscara de limitação da separação espectral (em quais pixels uma banda foi limitada)

A separação espectral 3×3 pode tornar uma banda negativa — quando a luz mal contém essa banda ou quando a cena fica fora do que a matriz de separação da câmera descreve. Esse não é um valor real, portanto a exportação o fixa em 0 — e a visualização ao vivo mostra o mesmo 0. Uma banda pode ser limitada a alguns pixels ou a quase todos eles; a máscara registra qualquer uma das situações. O `Chloros:Band{R,G,B}UnmixClampFraction` indica em que proporção do quadro isso ocorreu; a máscara de limitação indica **quais pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— em uma pasta `clamp_masks/`**ao lado** da pasta do produto, nunca dentro dela (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`; uma exportação de `process` resulta em `tiff16/Reflectance_Calibrated_Images/x.tif` → `tiff16/clamp_masks/Reflectance_Calibrated_Images/x_clampmask.tif`) — para cada produto de radiação, refletância e índice em que pelo menos um pixel tenha sido limitado: tanto as capturas quanto as exportações (e os TIFFs por quadro). `uint8` de banda única, na própria grade de pixels do produto (incluindo orientação, ROI, binning e alinhamento da matriz). Cada máscara identifica a câmera do produto (`Make` / `Model` / número de série), a hora da captura e as tags da lente, além de listar as bandas representadas por seus bits (`XMP-Camera:BandName`, bit 0 primeiro); seus bits `ImageDescription` e `Software` a identificam como uma máscara de limitação, e ela não contém calibração.
- **Bits**: bit 0 = a banda no canal R do produto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Um bit definido significa que o valor do pixel nessa banda foi limitado a 0 — considere-o, e qualquer índice construído a partir dele, como de baixa confiança.
- **`Chloros:UnmixClampMask`** é gravado em todos os produtos que passaram pela separação de cores: o caminho da máscara em relação à própria pasta do produto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre separado por `/` — junte-o à pasta da imagem para abrir a máscara), ou `none` quando nenhum pixel foi limitado. A ausência de marcação significa que o produto nunca passou pelo processo de unmix (câmeras RGB ou monocromáticas, níveis raw/debayered).
- `lattice capture` / `array-capture` imprimem o caminho da máscara sob o arquivo salvo (`clamp mask: …`), e a lista de resultados da captura a indica como o `clamp_mask` do quadro.
- `lattice index --live --save-multiband` grava uma máscara por câmera de origem para a pilha multibanda — consulte [`lattice index` — máscaras de pilha multibanda](#lattice-index--multiband-stack-masks). Um vídeo não possui máscara: o `array-build-video` relata, em vez disso, as frações de limitação de cada quadro (consulte [`array-build-video`](#array-build-video--offline-reprocess-a-saved-burst)).

Como nenhuma pasta de produto contém uma máscara, em qualquer nível, uma pasta de produto (`radiance/`, `reflectance/`, a pasta `*_Images/` de uma exportação `process`) pode ser entregue ao Pix4D, Metashape ou outro software de fotogrametria tal como está — incluindo subpastas — e ele aceita apenas imagens. Envie a pasta do produto, não a pasta acima dela (essa também contém `clamp_masks/` e todos os outros tipos de exportação). O próprio importador do Chloros ignora as máscaras. Interpretação de uma máscara: `(mask & 1) != 0` significa “a banda do canal R foi limitada aqui”, `& 2` G, `& 4` B.

#### EXIF transferido para a exportação

`process` copia o **bloco GPS e seu ExifIFD** da captura de origem para cada produto; portanto, uma
exportação traz `FocalLength`, `FNumber`, `ExposureTime`, `ISO`, `DateTimeOriginal` e
`CameraSerialNumber` junto com a georreferência.

**O `FocalLength` não é opcional para fotogrametria.** O Pix4D calcula a distância de amostragem do solo a partir da
distância focal mais a altitude; sem essa tag, ele recorre a uma escala totalmente errada. Em um
voo sobre um pomar de laranjeiras com 49 capturas, a falta dessa tag transformou um local de 411 m × 160 m em uma reconstrução
de 47,8 km × 13 km — uma ortofoto de 455 MP composta principalmente por “nodata”, o que foi interpretado como um problema de mosaico e
um problema de BigTIFF antes que alguém verificasse a GSD. Se sua ortofoto for gerada em uma escala implausível, execute o `exiftool -FocalLength` no produto exportado primeiro.

A cópia **não** é deliberadamente o `-all:all`: asas tags estruturais do IFD0&#x27;
prejudicam a saída do LATTICE quando
copiadas, e o `ExifImageWidth` / `ExifImageHeight` foram excluídos porque descrevem a
captura *de origem* — uma exportação que já tenha sido redimensionada, de outra forma, apresentaria dimensões
contraditórias em relação à sua própria imagem rasterizada. O XMP é gravado diretamente, em vez de copiado, porque o ExifTool
descarta as tags XMP da mesma invocação quando o bloco XMP é copiado (o que eliminaria as tags de calibração do MAPIR
).

### Onde as saídas são salvas

Os produtos são gravados **na pasta do projeto, agrupados por câmera e, em seguida, por formato de arquivo**:

```
<project>/
└── LATT-M3M-L41-F550/                  # one folder per camera model+lens+filter
    ├── tiff16/
    │   ├── Reflectance_Calibrated_Images/
    │   ├── Debayered_Images/
    │   ├── Preview_Images/
    │   └── <INDEX>_Index_Images/        # e.g. NDVI_Index_Images
    └── tiff32/
        └── Radiance_Images/             # float32 radiance always lands here
```

A pasta da câmera é `LATT-<sensor>-<lens>-F<filter>` para o LATTICE (correspondendo ao EXIF da captura
`Model`) e `<model>_<filter>` para o Survey3 — duas câmeras que compartilham um sensor e um filtro, mas diferem
na lente, mantêm árvores separadas, pois a vinheta, o campo de visão e a distorção diferem. A pasta de formato
da pasta segue `--format`: `tiff16`, `tiff8`, `png8`, `jpg8`, ou `tiff32` para
`TIFF (32-bit, Percent)`.

> **Cada produto exportado mantém o nome do arquivo ORIGINAL.** Uma exportação de radiação de
> `capture_…_raw.tif` ainda é chamada de `capture_…_raw.tif` — ela apenas fica em
> `tiff32/Radiance_Images/`. **A pasta identifica o produto, não o nome do arquivo**, portanto, uma busca global
> por `*radiance*.tif` não encontra nada; em vez disso, faça a correspondência com o diretório.

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

O `process` também processa as gravações `.daq` na sua pasta de entrada e **não**
precisa de nenhuma imagem para fazer isso: um DAQ-U / DAQ-M / DAQ-E operado sozinho já constitui uma
captura completa, e uma pasta contendo apenas arquivos `.daq` é uma entrada válida.

Um DAQ pode ser gravado **sem** sua calibração — é isso que os gravadores públicos
[`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fazem por padrão: eles gravam as contagens brutas do sensor e marcam o arquivo para que
o Chloros busque a calibração de fábrica desse sensor **pela porta serial** (primeiro no cache local,
depois na nuvem MAPIR) e a aplique. O `process` grava o resultado de volta:

```
<project>/
└── Light Sensor/
    ├── <name>_calibrated.daq        # reprocessable archive, declares its bundle
    └── <name>_calibrated.csv        # W/m^2/nm per reading + photometric columns
```

O `.csv` contém uma linha por leitura: carimbo de data/hora UTC, tempo de integração, potência total,
lux fotópico/escotópico, PPFD (e sua divisão em azul/verde/vermelho), comprimento de onda de pico e, em seguida, o
espectro completo na grade de comprimentos de onda do próprio sensor. O `.daq` é reimportado sem precisar ser
calibrado uma segunda vez.

`<name>` é o próprio nome do arquivo da gravação. Duas gravações em pastas diferentes podem compartilhar
uma — leituras por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salvas no
mesmo milissegundo por duas câmeras — e cada uma ainda recebe seus próprios produtos: a primeira mantém
`<name>`, a seguinte acrescenta as pastas que a diferenciam **após** o nome
(`<name>__<capture>_daq_calibrated.daq`), de modo que a data e a hora da gravação continuam aparecendo primeiro.

Se for bem-sucedida, a execução gera o relatório `Light-sensor products written: N (calibrated .daq + .csv)`.
O texto entre parênteses descreve o que foi realmente gravado, portanto, a leitura é
`(RAW COUNTS — this sensor has no calibration bundle)` para um sensor sem pacote e
`(N calibrated, M raw counts)` para uma pasta que contém ambos. Os próprios títulos do backend,
`[DAQ-EXPORT]` e `[RUN-SUMMARY]`, derivam sua redação da mesma forma — nenhum dos
três pode classificar uma exportação bruta como calibrada.

Um registro DAQ-U / DAQ-M / DAQ-E cujo pacote de calibração não possa ser obtido — você está
offline ou aquele sensor não possui calibração registrada — é **ignorada com um motivo** em uma
linha `[DAQ-EXPORT]`, nunca gravada como um arquivo “calibrado” contendo contagens brutas.
Conecte-se à internet e execute novamente. O motivo é aquele que o leitor realmente
estabeleceu para aquele arquivo (esquema ilegível, ausência de pacote, erro de gravação), e o resumo da execução
lista motivos **distintos** — vinte arquivos ignorados por um único motivo são interpretados como um
único motivo, não como vinte repetições dele. A CLI nomeia cada gravação ignorada no final da
execução, com o respectivo motivo:

```
⚠ Light-sensor recording not exported: <file>.daq: no calibration bundle available for …
```

Uma pasta de gravações que não exporta **nenhuma** delas falha: veja
[Uma execução que não grava imagens falha](#a-run-that-writes-no-images-fails).

Uma gravação cujo nome termine em `_calibrated` ou `_raw` (`field_raw.daq`) **não é
importada**: esses são os nomes que o `process` atribui aos seus próprios produtos, e um produto lido
como uma gravação seria contado duas vezes. A CLI indica isso e nomeia o arquivo —
`Not imported as a light-sensor recording: field_raw.daq … Rename the file to import it.`
— em vez de relatar uma pasta vazia. Renomeie essa gravação antes de processá-la.

#### Gravações DAQ-A são exportadas como contagens brutas

A família **DAQ-A** é anterior ao sistema de pacotes por número de série e não possui pacote de calibração
para ser baixado — em vez disso, é calibrada em campo em relação a um alvo de refletância, e é
por isso que nunca precisou de um. Recusar essas gravações deixava-as sem nenhuma maneira de extrair seus
números, então elas são exportadas com um **nome diferente**:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq        # NOT _calibrated
    └── <name>_raw.csv        # raw spectral sensor counts, NOT irradiance
```

Um nome de arquivo diferente, em vez de um sinalizador dentro do arquivo, porque a informação precisa permanecer intacta
ao ser enviada por e-mail apenas como um nome simples. O cabeçalho `.csv` indica
`raw spectral sensor counts (NOT irradiance)` e avisa que os valores são comparáveis
**dentro** do arquivo — que é exatamente para isso que a calibração baseada em alvo os utiliza — e
não entre sensores. As colunas fotométricas dependentes de potência (potência total, lux fotópico e
escotópico, PPFD) são registradas como **NULL** em vez de serem integradas a partir de contagens, e o resumo
da execução indica `RAW COUNTS`; portanto, “exportado” em um log não pode ser interpretado como irradiância.

Gravações antigas **v1.01 / v1.02** (um DAQ-A-SD as grava) não contêm época por leitura,
apenas a hora de gravação do arquivo. O comparador de imagens↔radiação descendente ainda as rejeita — comparar um
quadro com uma hora de gravação seria um erro imperceptível —, mas o exportador as lê, e
o arquivo CSV exibe `clock=daq_created_on`, de modo que o produto indica em qual relógio ele se encontra.

### Notas

- O `process` detecta automaticamente se sua pasta é do tipo Survey3, “LATTICE” ou mista.
- O progresso é transmitido por meio de Server-Sent Events; a CLI mostra o progresso em tempo real por thread (Detectando, Analisando, Processando, Exportando).
- Para Linux/Jetson, a CLI verifica o swap e pode emitir um aviso antes de processar pastas grandes. O debayer sensível à textura também aplica automaticamente um limite de frequência da GPU em Jetsons de baixo consumo de energia (Nano, Orin Nano).
- Em caso de sucesso, a execução informa quantos produtos de imagem foram gravados (`Image products written: N`, com `(plus M clamp masks)` quando a separação de cores limitou algum pixel).
- **`process` compartilha um backend em execução com segurança.** Ele reinicia automaticamente um backend já em execução somente quando esse backend estiver ocioso, com um projeto anterior carregado e sem câmeras, matrizes ou DAQs. Um backend que contenha qualquer um desses elementos (a GUI transmitindo uma matriz, um script de SDK, uma gravação de DAQ) é usado tal como está, nunca reiniciado; assim, o `process` pode ser executado ao lado de uma matriz ativa. Se o backend relatar uma execução em andamento, o `process` aguarda até 15 s para que ela termine (uma execução que acabou de ser concluída leva cerca de um segundo para ser liberada), e então retorna um erro com o código `Backend is mid-run (processing). Wait for it to finish, or pass --restart to force a fresh backend (this kills the in-flight run).`. O sinalizador global `--restart` ainda força a inicialização de um backend novo e encerra qualquer sessão de câmera, matriz ou DAQ nele. Um backend executado como serviço do systemd no Linux nunca é reiniciado: `process` o utiliza, e o `--restart` exibe o `sudo systemctl restart <unit>` em vez disso (consulte [Executando o backend como um serviço](#running-the-backend-as-a-service-headless)). O mesmo vale para o próprio backend do aplicativo Chloros: com o aplicativo aberto, o `process` o utiliza, carregando seu projeto no lugar do do aplicativo (um backend iniciado em seu lugar bloquearia o acesso do aplicativo aberto).
- **Uma execução em andamento mantém seu projeto.** Enquanto uma execução de processamento estiver em andamento — iniciada pelo aplicativo, pelo `process`, por um script do SDK ou pelo `chloros-mcp` — o backend se recusa a criar, abrir, importar, adicionar ou remover arquivos, mover, excluir ou alterar as configurações de processamento de um projeto, com o código HTTP 409 `PROCESSING_IN_PROGRESS` e `A processing run is in progress on project '<name>', so ... now is refused`; um `process` que receba essa solicitação encerra o `1`. As configurações de exibição (tamanho da miniatura) continuam sendo salvas. O modo de processamento e as configurações de refletância-alvo do projeto são recusados da mesma forma (uma execução mantém o modo em que foi iniciada). Uma captura salva no projeto durante a execução (pelo aplicativo, pela CLI, pelo SDK ou pelo `chloros-mcp`) é salva normalmente, mas só é adicionada à lista de arquivos do projeto quando a execução termina; a resposta confirma isso (`project_add`), assim como uma gravação do sensor de luz interrompida no projeto. Antes, um segundo cliente podia alternar para o projeto aberto no meio da execução, e a execução em andamento era exportada para o outro projeto.
- **Ctrl+C interrompe a execução no backend, não apenas na CLI.** A CLI solicita ao backend que interrompa o processamento, exibe `Waiting for the backend to stop (Ctrl+C again to stop waiting)...` e aguarda até 120 s pelo `Processing stopped.`, para que o `chloros-cli process … && next` não inicie o `next` em um backend ainda ocupado com a execução. Um segundo Ctrl+C encerra a espera. `process` então encerra `130`, como qualquer comando interrompido faz.

#### Uma execução que não grava imagens falha

Se você solicitou produtos e a execução não gravou **nenhum** — apenas `project.json` e
`calibration_data.json` — o `process` trata isso como uma falha: ele exibe
`Processing finished but wrote no image products.` e **sai com valor diferente de zero**, de modo que um script possa
detectá-la. A mensagem indica o nome da pasta do projeto e as causas mais comuns:

- a pasta de entrada não foi reconhecida como uma captura (verifique o layout e o `--input-level`), ou
- todos os produtos solicitados foram ignorados por serem inaplicáveis para essas câmeras (por exemplo, solicitar
  radianância/refletância de câmeras apenas RGB).

Execute novamente com o `--verbose` e verifique o log do backend em busca das linhas `[LATTICE-EXPORT]` / `[EXPORT-CHECK]`,
que explicam as omissões por câmera que, de outra forma, não chegariam à saída da CLI.

Uma execução deliberada apenas com metadados — todos os produtos desativados e sem `--indices` — ainda é um
**sucesso**, pois uma saída de imagem vazia é o resultado correto nesse caso.

#### Uma execução cujas exportações não puderam ser verificadas falha

Após uma execução, o `process` verifica as tags dos arquivos exportados com o ExifTool. Se o ExifTool não puder ser executado, ele
exibe `Falha na verificação da exportação: o ExifTool não pôde ser executado, portanto as tags de N de M arquivos exportados
não foram verificadas (consulte as linhas [EXIFTOOL] no log do backend)` and `Os produtos são gravados e permanecem
no local; execute novamente para verificá-los.`, does not print `Processamento concluído!`, e **encerra com código diferente de zero**. Os
produtos são mantidos.

#### Quais capturas uma execução considera

As capturas concluídas antes da execução do `process` são incluídas. As capturas salvas durante a execução não são; o `process`
avisa quantas foram deixadas de fora, e executá-lo novamente as inclui.

#### Uma execução que perdeu uma imagem falha

Se uma imagem falhar durante uma execução, o `process` exibe o `N image(s) failed and were not exported` com a
causa mais comum, não exibe o `Processing complete!` e **encerra com valor diferente de zero**. Os demais produtos são
gravados e permanecem no local, e o log do backend registra cada falha.

O mesmo ocorre com uma **execução apenas com sensor de luz**: uma pasta de gravações `.daq` não contém imagens para exportar
por definição, e a execução é avaliada com base nos arquivos calibrados `.daq` / `.csv` que foram gravados em seu lugar.
Se não gravou **nenhuma** — todas as gravações foram ignoradas —, ele exibe
`Processing finished but wrote no light-sensor products.`, nomeia cada gravação com o
motivo pelo qual foi ignorada e **encerra com valor diferente de zero**.

---

## `chloros-cli login`

Autentique esta máquina com uma conta na nuvem do Chloros+. As credenciais são armazenadas em cache de forma segura no `~/.chloros/user_session.json`.

```
chloros-cli login EMAIL PASSWORD
```

### Exemplos

```bash
chloros-cli login user@example.com 'YourPassword'

# Passwords containing $ should use SINGLE quotes
chloros-cli login user@example.com 'my$ecret$pass'
```

> **PowerShell `$$` mangling is auto-corrected.** In double quotes PowerShell expands `$$` (removendo partes da senha ou duplicando-as). Em caso de erro 401, a CLI tenta automaticamente novamente com `$$` re, e depois com metade da senha sem repetições; se a nova tentativa for bem-sucedida, o sistema faz o login e exibe a sintaxe correta com aspas simples para ser usada na próxima vez.

> **Uso sem interface gráfica/por script: a ausência de sessão em cache significa que será exibido um prompt de login.** Um comando principal (`process`, `status`, `export-status`, `time-sync`, …) executado sem uma sessão em cache solicita `Email:` / `Password:` na entrada padrão (stdin) antes de prosseguir. Com o stdin fechado (cron, uma unidade do systemd), o programa falha imediatamente com o código de erro `login cancelled`; com o stdin em um terminal aberto ou pipe, ele aguarda a entrada. Execute o `chloros-cli login EMAIL PASSWORD` uma vez por máquina antes de agendar tarefas sem interface gráfica.

---

## `chloros-cli logout`

Limpa a sessão armazenada em cache e força um novo login na próxima chamada.

Sair enquanto uma execução de processamento estiver em andamento não a interrompe: a execução é concluída no modo de processamento em que foi iniciada, e a próxima execução utiliza o processamento padrão. O código `logout` indica isso (`The processing run in progress finishes in <mode> mode; the next run uses standard`), assim como o aplicativo ao sair de lá.

```bash
chloros-cli logout
```

---

## `chloros-cli status`

Mostra o nível de licença atual (Iron/Copper/Bronze/Silver/Gold), o usuário autenticado e a contagem de vinculações de dispositivos.

```bash
chloros-cli status
```

---

## `chloros-cli export-status`

Verifica o andamento da exportação do Thread-4 em tempo real. É seguro chamar **durante** a execução do `process` a partir de outro shell.

```bash
chloros-cli export-status
```

---

## `chloros-cli language`

Define o idioma de exibição da CLI (38 idiomas suportados, incluindo CJK, RTL e Indic). Recorre ao inglês de maneira adequada em consoles legados que não conseguem renderizar o script.

```
chloros-cli language [LANG_CODE] [--list]
```

### Exemplos

```bash
# List all available languages
chloros-cli language --list

# Switch to Spanish
chloros-cli language es

# Show the currently-active language
chloros-cli language
```

---

## Comandos da pasta de projetos

Esses comandos gerenciam o local padrão da pasta de projetos (compartilhado com a GUI).

```bash
chloros-cli set-project-folder "/home/user/Chloros Projects"
chloros-cli get-project-folder
chloros-cli reset-project-folder
```

`get-project-folder` exibe a pasta para a qual os novos projetos são direcionados. Se a pasta configurada não existir mais (foi excluída, renomeada ou está em uma unidade que não está conectada), a CLI, assim como a GUI, recorre à pasta padrão: a pasta padrão é exibida na saída padrão (stdout), um aviso informando o nome da pasta ausente é enviado para a saída de erro (stderr) e o código de saída é `0`.

---

## `chloros-cli update`

Somente Linux/Jetson. Verifica se existe o `version_url` a partir do `/etc/chloros/update.conf` e, quando há uma versão mais recente, baixa o `.deb` correspondente e o instala com o `sudo dpkg -i` imediatamente (sem solicitação de confirmação; o `sudo` pode solicitar sua senha). No Jetson, a atualização é oferecida apenas para sistemas com o JetPack 6. No Windows, o comando não executa nenhuma ação e exibe a mensagem `up to date`: atualize com um novo instalador.

```bash
chloros-cli update            # check + install
chloros-cli update --check    # check only
```

No Linux/Jetson, a CLI também executa uma **verificação automática de atualizações a cada inicialização** (sem bloqueio, nunca atrasa o comando): ela lê `/etc/chloros/update.conf`, armazena o resultado em cache por 1 hora em `~/.chloros/update_cache.json` e exibe `Update available: vX.Y.Z / Run: chloros-cli update` quando existe uma versão mais recente. Ignora silenciosamente qualquer erro e no Windows; `CHLOROS_DISABLE_UPDATE_CHECK=1` desativa essa função. Devido ao cache de 1 hora, uma versão lançada dentro dessa hora ainda pode ser considerada atualizada.

---

## `chloros-cli selftest`

Executa um teste de funcionamento de 7 etapas: versão, disponibilidade de porta, inicialização do backend, `/api/test`, `/api/system-info` (GPU/CUDA/PyTorch), presença do modelo de redução de ruído, prontidão do CUDA + denoiser. Ele utiliza um backend em execução ou inicia um para o teste e o encerra em seguida. Exibe `N/7 checks passed` e as verificações com falha, e só encerra `0` quando todas as sete forem aprovadas; a verificação de CUDA + denoiser falha em um computador sem uma GPU CUDA (consulte [Verificar a instalação](#verify-the-install)).

```bash
chloros-cli selftest
```

---

## `chloros-cli time-sync`

Status e controle do grandmaster PTP. O host Chloros executa o grandmaster PTP; as câmeras LATTICE e as unidades DAQ-E funcionam como escravas dele para garantir a sincronização de carimbos de data/hora entre dispositivos (cerca de 1 ms entre os dispositivos). O PTP alinha **carimbos de data/hora**: as câmeras de um conjunto expõem simultaneamente porque o disparador de hardware M8 as aciona (em menos de 100 µs entre os módulos), e não por causa do PTP. O comando `chloros-cli time-sync`, sem subcomando, executa o comando `status`.

| Subcomando | Descrição |
| --- | --- |
| `status` | Mostra o estado do grandmaster, as prioridades BMCA, a identidade do relógio e se seus pacotes estão chegando à rede: `health` é `OK` (`STOPPED` quando o grandmaster não está em execução), ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` com um `health_detail`, e, em seguida, uma linha `WARNING:` é impressa no stderr (o stdout permanece em JSON). `sync_sent` / `announce_sent` contam apenas as mensagens que foram enviadas; `*_send_errors` e `last_send_error` contam as que não foram. Após uma queda e recuperação da conexão da câmera, o grandmaster reconstrói seus soquetes sozinho (`rebinds`, `last_rebind`). |
| `peers` | Lista os escravos detectados via Delay_Req (câmeras + sensores DAQ-E). |
| `cameras` | Integridade do PTP por câmera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`). |
| `restart` | Reiniciar o processo do grandmaster. |
| `set-priority --priority1 N --priority2 N` | Substituir as prioridades do BMCA. |

O grandmaster faz o anúncio nas placas de rede das câmeras (qualquer adaptador com um endereço `169.254.x.x` incluído). Se nenhum estiver ativo, ele recorre ao adaptador da rota padrão somente quando este for com fio, nunca Wi-Fi ou um adaptador virtual (Tailscale, WSL), já que nenhuma câmera ou DAQ-E fica atrás de um. Sem nada para anunciar, ele roda sem soquetes (`health` `NO_SOCKETS`), e a próxima câmera ou DAQ que se conectar o reconstrói na placa de rede que estiver ativa naquele momento; `time-sync restart` faz isso imediatamente.

O grandmaster é executado em um processo próprio, ao lado do backend (`own_process` / `process_pid` em `time-sync status`), com alta prioridade no Windows (`process_priority`), de modo que um backend ocupado (uma execução de processamento, uma exportação, vários comandos ao mesmo tempo) não possa atrasar os horários que ele registra; antes, um atraso desse tipo poderia alterar o relógio de todas as câmeras de uma só vez. Esse processo termina com o backend; se ele não puder ser iniciado, o grandmaster é executado dentro do backend como antes (`own_process` false, e o log do backend indica isso).

### Exemplos

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
chloros-cli time-sync cameras
chloros-cli time-sync restart
chloros-cli time-sync set-priority --priority1 1 --priority2 1
```

---

## `chloros-cli lattice`

Controle de câmera LATTICE. Cada subcomando passa pelo backend Chloros; o backend é responsável pelo conjunto de câmeras, de modo que chamadas subsequentes da CLI reutilizam o mesmo identificador aberto.

### Opções comuns (compartilhadas pela maioria dos subcomandos)

| Sinalizador | Descrição |
| --- | --- |
| `-d, --device N` | Índice da câmera (padrão: 0). |
| `-s, --serial SN` | Número de série específico; substitui `--device`. |
| `--serials SN1,SN2,…` | Números de série separados por vírgulas para operação com múltiplas câmeras. Os [comandos transitórios para múltiplas câmeras](#multi-camera-transient-sessions) abrem apenas essas câmeras, sendo que a primeira aciona o gatilho. |
| `--all` | Operar em todas as câmeras detectadas. |
| `--exposure US` | Tempo de exposição em microssegundos. |
| `--gain DB` | Ganho em dB. |
| `--pixel-format FMT` | por exemplo, `BayerRG8`, `BayerRG12`; `Mono8`, `Mono12` em um M3M. O nome define a profundidade de bits na própria família da câmera: em um M3M, um nome do tipo Bayer define o formato Mono de sua profundidade (`BayerRG8` define `Mono8`); em um M3C, um nome Mono define o nome do tipo Bayer (`Mono12` define `BayerRG12`). |
| `--width N` / `--height N` | Dimensões da imagem. |
| `--preset {default,high_quality,high_speed,triggered}` | Aplicar uma predefinição de configurações: seu formato de pixel, exposição, ganho e taxa de quadros (`high_speed`: `BayerRG8`, 5 ms, 6 dB, 60 fps; `high_quality`: `BayerRG12`, 20 ms, 0 dB; `lattice presets` lista essas configurações). Em um M3M, o formato é o Mono com a mesma profundidade: `high_speed` captura `Mono8`, `high_quality` e `triggered` `Mono12`. Um sinalizador fornecido com ele (`--exposure`, `--gain`, `--pixel-format`) prevalece sobre a predefinição, e uma configuração não especificada na predefinição mantém a configuração padrão da câmera. Todas as execuções do, exceto `triggered`, que arma a câmera para uma mudança de nível de hardware na Linha 2 — sem nada acionando essa linha, ela ficará esperando indefinidamente em vez de capturar. |
| `-o, --output DIR` | Diretório de saída (padrão: `output`). |
| `--packet-size {auto,jumbo,standard,N}` | Tamanho do pacote GVSP. `auto` executa sondas ICMP+GVSP; `jumbo` = 9000; `standard` = 1500. |

### Fluxo de trabalho de primeira conexão da câmera LATTICE

```bash
# 0. A backend must be running: the Chloros desktop app (Windows),
#    `chloros-backend`, or the Linux service (see "Start the Backend")

# 1. Discover cameras on the network
chloros-cli lattice info

# 2. Single-cam smoke test: capture one frame.
#    By default this saves EVERY export type applicable to the cam
#    (raw, debayered, radiance, reflectance, preview). Pass e.g.
#    `--processing debayered` to save just one.
chloros-cli lattice capture -o output/

# 3. Connect a synchronized array (RECOMMENDED ENTRY POINT for arrays).
#    This is the same "smart-prep" flow the Chloros GUI uses:
#      - Network capability probe (ICMP DF ping + GVSP probe)
#      - Tier auto-pick (sim-emit / ftd-stagger / slip)
#      - Full sensor (no binning, no ROI); the frame rate falls to
#        what the host and wire deliver, and the faster options are
#        printed
#      - PTP enabled by default
#      - Per-cam pixel format auto-pick
#      - AE seeding from the cam's saved state
#      - GPIO trigger config on Line2
chloros-cli lattice array-connect \
  --serials 213800234,214000533,214701288,214701292

# 4. Capture one synced frame group from the live array.
#    Defaults to --processing all (one file per export type per cam);
#    pass a single level to narrow it, e.g. --processing reflectance.
chloros-cli lattice array-capture --processing reflectance -o output/

# 5. Live-preview one cam in your browser
chloros-cli lattice viewer --serial 213800234

# 6. Tear down when done
chloros-cli lattice array-disconnect
```

### Referência de subcomandos

#### Descoberta e informações

| Subcomando | Finalidade |
| --- | --- |
| `lattice info` | Listar câmeras conectadas (fabricante, modelo, número de série, IP, MAC). |
| `lattice probe [--pixel-format FMT] [--json] [--no-discover]` | Analisa o sistema host para obter a configuração ideal da câmera. O `--no-discover` ignora a descoberta da câmera (mais rápido, análise apenas pela placa de rede). |
| `lattice network [--fix] [--estimate] [--cameras N] [--pixel-format FMT]` | Verifica/corrige as configurações da placa de rede (no Linux, também os buffers de recepção de soquetes do kernel — consulte [hosts Linux](#linux-hosts--socket-receive-buffers)); estima a largura de banda/FPS. |
| `lattice network-analysis --master SN [--slaves SN1,SN2,…] [--width N] [--height N] [--pixel-format FMT] [--binning {1,2,4}] [--force-tier TIER] [--packed {auto,on,off}] [--backend-url URL] [--json]` | Recomendação de capacidade de rede do backend com esquema estável+ recomendação de matriz (retorna `status` ∈ `ok` / `auto_shrunk` / `auto_capped_fps` / `needs_force_slip` / `error`). `--width` / `--height` assumem por padrão o sensor completo, `2048` × `1536`. Ele sai de `0` para `ok`, `auto_capped_fps` e `auto_shrunk`, e `1` para `needs_force_slip` e `error`, com ou sem `--json`. Ele chega ao backend em `--backend-url` (padrão `http://127.0.0.1:5000`), e não em `CHLOROS_BACKEND_URL`. O `auto_capped_fps` mantém a resolução solicitada, mas limita a taxa de quadros alvo — leia o `recommended.recommended_target_fps` e passe-o como destino da conexão; trate isso como sucesso, não como um erro. `auto_shrunk` aparece apenas quando o quadro solicitado não pode ser entregue em nenhuma taxa de quadros; o connect nunca reduz quadros por conta própria. |
| `lattice analyze-array [--models M1,M2,…] [--binning {1,2,4}] [--n-active N] [--width N] [--height N] [--pixel-format FMT] [--force-tier TIER] [--packed {auto,on,off}] [--json]` | Análise de cenários hipotéticos sem abrir câmeras (`--width` / `--height` padrão `2048` × `1536`). Tanto este quanto o `network-analysis` projetam o que o connect transmitirá: sem o `--pixel-format`, os formatos por câmera que o Connect aplica (Mono12 / BayerRG12); e com o `--packed auto` (padrão), a decisão que o Connect tomaria neste computador [transporte compactado de 12 bits](#transportado-em-12-bits-compactado) que o Connect tomaria neste computador — portanto, o fps indicado é a taxa na qual a matriz opera. Ambos exibem a decisão de compactação. **`--n-active` é o número total de câmeras na rede, não apenas as desta matriz**— aumente-o quando câmeras autônomas transmitirem simultaneamente, ou quando a capacidade da rede for calculada com base em uma demanda que as subestime (padrão: `len(--models)`). Sempre exibe as linhas agregadas `Wire budget:` (MB/s solicitados vs. limite máximo seguro contra colisões) e `Max cameras:`, e sinaliza `** OVER-SUBSCRIBED**` quando o array sobrecarrega a largura de banda — consulte [Modelo de fps e burst da matriz](#array-fps--burst-model). |
| `lattice gpu` | Mostrar status da GPU. |
| `lattice firmware [--update] [--force] [-y\|--yes]` | Verificar ou atualizar o firmware da câmera. A seleção local do `.fwa` está fixada: o arquivo no `firmware/<MODEL_PREFIX>/` que corresponde ao `MIN_FIRMWARE_VERSION` da compilação é gravado quando presente (apenas a versão mais alta como alternativa), portanto, uma imagem do fabricante mais recente armazenada no disco permanece inativa até que esse pino seja alterado — versões deliberadamente mais recentes chegam às unidades por meio do manifesto assinado da AWS, o que é preferível quando há uma versão mais recente. |
| `lattice presets [--apply NAME]` | Listar ou aplicar predefinições da câmera. `--apply` define o que as predefinições representam (seu formato de pixel, exposição, ganho, taxa de quadros, acionamento) e exibe essas configurações conforme a câmera as lê após: em um M3M, o formato Mono da profundidade da predefinição; a taxa de quadros em que ela é executada, que pode ser inferior à solicitada pela predefinição (`high_speed` solicita 60 fps; uma câmera full-frame em um link de 1 Gbps opera a cerca de 35). Todas as demais configurações que seu comando “open” escreveu são restauradas conforme encontradas. `--apply default` redefine a câmera para seus padrões de fábrica: suas configurações de fábrica com exposição automática livre na Linha 2, o limite de taxa de transferência do link e a correção de defeitos desativados, e o equilíbrio de branco calibrado de uma câmera RGB mantido (unidade em qualquer outra). Nada é salvo na memória da câmera; portanto, na próxima inicialização, ela retoma a operação a partir de sua própria memória, como antes. Como o `--preset` de uma captura ou conexão, o `default` não é uma predefinição. |
| `lattice status` | Status da câmera ao vivo. |

#### Captura

| Subcomando | Finalidade |
| --- | --- |
| `lattice capture [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--processing LEVEL] [--levels L1,L2,…] [--no-daq] [--daq-tolerance-ms MS]` | Quadro único. **Salva todos os tipos de exportação por padrão**(`--processing all`); consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). `--processing` aceita `all`, `raw`, `debayered`, `radiance`, `reflectance` ou `display` (`preview` é aceito para `display`); qualquer outro nome é rejeitado antes que a câmera seja acionada (saída `1`). `--levels` salva um subconjunto explícito (substitui `--processing`). Uma câmera que o Chloros não possui é aberta para a captura, e sua exposição é definida a partir da cena inicialmente (no máximo alguns segundos), assim como fazem os [comandos transitórios](#multi-camera-transient-sessions); quando a captura termina, independentemente da forma como isso ocorra, essa câmera é restaurada ao estado em que foi encontrada (formato, exposição, pipeline de cores, todas as configurações alteradas pela captura — a lista em [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions)). A leitura do DAQ utilizada na captura é salva como um sidecar `.daq`, incluindo uma captura apenas em formato bruto; `--no-daq` / `--daq-tolerance-ms` ajustam isso ([O DAQ que lê uma captura salva](#the-daq-reading-a-capture-saves)). `--force-daq` ainda é aceito. `--jpeg-quality` = qualidade JPEG 1–100 (padrão 95).**Os arquivos salvos no projeto aberto são adicionados a ele.** Assim como no aplicativo, uma captura cujos arquivos sejam salvos na pasta do projeto aberto no Chloros os adiciona à lista de arquivos desse projeto; arquivos salvos em outros locais não são afetados. O comando exibe uma linha informando isso (`Project: ...`) ou, enquanto uma execução de processamento estiver em andamento, que eles estão sendo retidos e serão incorporados quando a execução terminar. O mesmo ocorre com `multi-capture`, `array-capture` (`--smart` também) e `project capture`. |
| `lattice continuous [--format tiff\|tif\|png\|jpg\|jpeg] [--jpeg-quality N] [--queue-depth N] [--no-daq] [--daq-tolerance-ms MS]` | Transfere para o disco até pressionar Ctrl+C. Uma execução do `tiff`são o arquivo `raw`, uma matriz que a captura grava daquela câmera: o mosaico do sensor, um plano com seus próprios valores, não compactado, com o pacote de calibração (um quadro de 2048 x 1536 de 12 bits tem 6,0 MiB mais o pacote, independentemente da cena), portanto, o `process` parte deles. `png` e `jpg` contêm 8 bits; portanto, essas sequências são quadros desmosaicados para visualização, dos quais o `process` não parte. Com um DAQ vinculado (`daq pool-bind`) ou agrupado, cada quadro recebe seus próprios `.daq` e `.match.json` na pasta `daq/` da sequência, já que o `lattice capture` salva um; o `--no-daq` desativa isso, e a parada exibe quantos. Cada quadro é gravado uma vez, com seus metadados completos; ao pressionar Ctrl+C, os quadros já capturados são gravados antes que a câmera seja liberada. A execução é interrompida quando o disco fica cheio, exibe `Failed:` / `Disk full:` e sai com o nome `1`; qualquer quadro que não tenha sido gravado também gera o arquivo `1`. Uma câmera deixada armada para um disparador de hardware (`TriggerMode` `On`) é configurada para operação livre durante a sessão. Ao término da sessão, a câmera é restaurada ao estado em que foi encontrada — suas configurações de acionamento e todas as configurações alteradas durante a sessão (exposição, ganho, formato, …) — a menos que seja executado o comando `--keep-settings`, que salva as configurações da execução na própria memória da câmera (modo livre), ou o comando `--clear-settings`, que redefine essa memória para as configurações de fábrica. Uma execução que não grava nenhum quadro é considerada uma falha: se não houver nenhum quadro por 10 s, ela se interrompe automaticamente, imprime `No frames:` com o motivo e sai com o código `1`. Os quadros que os gravadores não conseguiram acompanhar são contados como descartados, e a interrupção exibe `Dropped:` com o motivo (a taxa de quadros da câmera, dos gravadores e da fila); uma taxa de quadros mais baixa ou um formato de pixel de 8 bits mantém todos eles. O comando verifica o backend a cada 2 s; se ele falhar sem interromper a gravação (`kill -9`, falta de memória), o backend a interrompe 30 s após a última verificação e coloca a câmera de volta (`stop_reason` `client_gone`, e uma linha no log do backend). `multi-stream` e `align-stream` fazem o mesmo. Uma inicialização em que a câmera demora mais para abrir do que o tempo de espera do comando (60 s; 180 s para `multi-stream` / `align-stream`) — uma câmera ainda em inicialização — é revertida pelo backend: a câmera é recolocada e fechada, e nenhuma gravação permanece em execução (504 HTTP `CLIENT_DEADLINE_PASSED`). |
| `lattice viewer [--brightness N] [--ae-damping F] [--frame-rate FPS]` | Visualização MJPEG ao vivo no navegador: conecta a câmera, exibe a URL do stream e a abre no seu navegador. `--ae-damping` define o amortecimento da exposição automática (0,4–100). |

##### Transmissões ao vivo (MJPEG; sem RTSP)

O Chloros transmite vídeo ao vivo como **MJPEG por HTTP**(`multipart/x-mixed-replace`, limite `frame`) a partir do backend.**RTSP não é suportado.** Abra uma URL em um navegador ou leia-a com qualquer cliente MJPEG (`cv2.VideoCapture(url)` do OpenCV, ffmpeg, VLC). A câmera deve estar conectada no backend primeiro (`lattice cam-connect`, `lattice array-connect`, `lattice viewer` ou pela GUI).

| URL (`http://127.0.0.1:5000` + …) | Fluxo |
| --- | --- |
| `/api/camera/stream/<serial>` | A pré-visualização da tela, com até cerca de 30 fps. Para um elemento da matriz, `?max_h=` (padrão: `720`) e `?jpeg_q=` (padrão: `80`) definem a altura e a qualidade do JPEG. Esta é a URL que `lattice viewer` abre. |
| `/api/camera/<serial>/stream-annotated` | A mesma pré-visualização com as sobreposições da GUI desenhadas em: `zebra=1`, `crosshair=1`, `grid=3x3` (ou `9x9`), `histogram=1`, `peaking=1`, `spot=1`; sem solicitação, as configurações de sobreposição salvas no projeto para essa câmera. |
| `/api/camera/<serial>/stream-reflectance` | Refletância de 8 bits por banda: ρ = pixel ÷ a parte`X-Chloros-Pixel-Scale` (255, a menos que `refl_max` seja acionado). `fps` (padrão `10`), `jpeg_q` (padrão: `95`), `max_h` (padrão: `0` = nativo), `band=<name>`, `align=1`. JPEG, portanto com perda de dados. Recusado com código HTTP 409 quando a câmera não apresenta refletância (uma câmera RGB, sem pacote de calibração, sem leitura DAQ). |

- **Correspondência de quadros entre câmeras.**Cada parte enviada por um**membro da matriz**, nos três fluxos, contém os cabeçalhos `X-Chloros-Group-Id` (o contador de grupo sincronizado da matriz), os cabeçalhos `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Partes de câmeras diferentes com o mesmo `X-Chloros-Group-Id` provêm do mesmo gatilho. Uma câmera autônoma não possui grupo e não envia nenhum ID de grupo. Os navegadores ignoram os cabeçalhos de cada parte; um cliente que analisa o próprio fluxo multiparte os lê.
- **Pausado enquanto a visualização da matriz estiver aberta.** O `/api/camera/stream/<serial>` de um membro da matriz envia um quadro estático “pausado” enquanto o fluxo ao vivo combinado da matriz estiver aberto (por exemplo, a visualização da matriz na GUI) e retoma quando ele for fechado.

#### Ajuste do sensor

| Subcomando | Finalidade |
| --- | --- |
| `lattice configure [{--get N1 N2… \| --set N=V N=V… \| --dump}] [--json]` | Ler/gravar qualquer nó GenICam. `--set ExposureTime=…` (`Gain=…`) sem seus parâmetros `ExposureAuto` (`GainAuto`) que desativam o modo automático primeiro, como faz o `lattice exposure --set`; um valor que a câmera rejeita imprime `Failed to set <node>: …` no stderr e encerra o `1`. O `--get` e a visualização agrupada padrão exibem um `note:` sob `ExposureTime` / `Gain` enquanto o modo automático estiver em execução — consulte [Lendo a exposição e o ganho no modo automático](#reading-exposure-and-ganho-no-modo-automático). |
| `lattice exposure [{--auto \| --auto-once \| --off \| --set US}] [--brightness 0-255] [--brightness-auto] [--damping 0.4-100] [--upper-limit US]` | Exposição e AE. Uma exposição `--set` ou um alvo `--brightness` é mantido como o seu (o controle automático de exposição do Chloros não o altera); o ganho permanece como estava, portanto, no modo automático, ele continua se ajustando em direção ao alvo até que você defina o `lattice gain`. `--auto` devolve o controle da exposição, e `--brightness-auto` devolve o alvo para o alvo automático dChloros. Sem sinalizadores, ele lê os nós de exposição, com o mesmo `note:` que `configure --get`. Um valor `--set` que a câmera recusa imprime `Failed to set ExposureTime: ExposureTime=<value> refused (<camera error>); the camera accepts <min>..<max> now (ExposureAuto put back to <mode>)` no stderr — no modo automático, a gravação desativada é restaurada em vez de permanecer desativada — não imprime nenhuma linha `Exposure: … us (auto OFF)` e encerra o `1`. |
| `lattice gain [{--auto \| --off \| --set DB}]` | Ganho e ganho automático. Um valor `--set` que a câmera rejeita imprime `Failed to set Gain: …` no stderr, nenhuma linha `Gain: … dB (auto OFF)`, e sai do `1`. |
| `lattice resolution [--set WxH] [--offset X,Y] [--binning N]` | ROI do sensor e binning. O binning é sempre **Média** — veja abaixo. |
| `lattice format [--set FMT] [--list]` | Formato de pixel. |
| `lattice trigger [--mode On\|Off] [--source SRC] [--delay-us US] [--activation EDGE] [--list-sources] [--software]` | Acionamento por hardware/software. |
| `lattice white-balance [--auto] [--off] [--red R] [--blue B]` (sem sinalizadores = WB de disparo único) | Operações de WB. Apenas câmeras RGB/Bayer; uma operação nula (ignorada) em M3M monocromático. O one-shot mantém o equilíbrio de branco que calcula (`BalanceWhiteAuto` e os ganhos); todas as outras configurações cuja gravação aberta ocorre retornam como foram encontradas, e todas elas quando o one-shot não puder ser executado. |
| `lattice color-profile [--set raw\|linear\|natural\|enhanced\|custom_temp] [--cct 2000-10000] [--get]` | Pipeline de cores da exibição RGB. `natural` (padrão) é o acabamento ao vivo mais simples; `enhanced` adiciona remoção de franjas + vibração + contraste local CLAHE para o visual completo de paridade de hub a cerca de 2× o custo de processamento por quadro, resultando em uma taxa de quadros **ao vivo** mais baixa — as capturas salvas sempre recebem o processamento completo de qualquer maneira. Apenas câmeras RGB/Bayer; ignorado em M3M monocromático. |
| `lattice color [--saturation 0-200] [--contrast 0-200] [--reset] [--get]` | Saturação/contraste da pré-visualização ao vivo (câmeras com filtro RGB); `100` é neutro, `--reset` define ambos para 100. As capturas salvas não são afetadas. Ignorado na M3M monocromática. |
| `lattice highlights [{--set -100..0 \| --reset \| --get}]` (sem sinalizadores = leitura) | Exibir **Destaques** (câmeras com filtro RGB): reduzir os pixels mais brilhantes e mais brancos — por exemplo, uma superfície branca brilhante que é renderizada como uma mancha branca uniforme. `0` = desativado, `-100` = mais forte. A pré-visualização ao vivo e as capturas no nível de exibição seguem essa configuração; a radiação/refletância nunca o fazem, e ele não consegue recuperar dados do sensor que sofreram clipping (diminua a exposição para isso). Ignorado no M3M monocromático. |
| `lattice filter [--list]` | Lista os modelos de filtro que o Chloros conhece (`RGN-IMX265`, `OCN`, `NGB`, …). O filtro de uma câmera é lido a partir da string do modelo e não pode ser configurado: o `--set NAME` apenas procura um modelo e, em seguida, sai do `1` com o `setting the filter model is not supported`. |
| `lattice power [--sleep]` | Verifica os nós de alimentação/térmicos; alterna para o modo de inatividade de baixo consumo. `--sleep` deixa a câmera em modo de suspensão (seu limite de taxa de transferência de link ativado (`DeviceLinkThroughputLimitMode` `On`) no valor mínimo, o modo de economia de energia ativado, caso a câmera possua um) com todas as demais configurações restauradas exatamente como estavam inicialmente; um modo de suspensão que falha é restaurado na íntegra. O despertar restaura a ligação como estava no momento da suspensão (o limite, seu modo e o atraso de pacotes `GevSCPD`, que a câmera mantém em 0 enquanto o limite está ativado; o registro é mantido neste computador, de modo que um despertar após a reinicialização do backend também o encontre), ou, para uma câmera que este computador não colocou em modo de suspensão, mas que parece estar em suspensão (com o limite ativado no valor mínimo), com o limite desativado e um `GevSCPD` mantido em 0 a 80 (as configurações de fábrica); um limite definido acima do mínimo é mantido como está. A sonda e o despertar restauram o que foram gravados ao serem abertos. |

> **O binning é sempre Média.** `lattice resolution --binning N` define o binning Média e
> não oferece opção de mudança de modo. O binning por soma multiplica cada DN por N², custando aproximadamente dois stops de
> margem de realces por etapa de binning; o Chloros divide esse fator novamente antes da
> cadeia radiométrica, mas os pixels que a soma levou ao teto do sensor são cortados e não podem ser
> recuperados. Isso não traz nenhuma vantagem que a Média não traga — um índice é invariante em relação à escala e os escalares de radianância
> são calibrados na escala da Média. Se uma câmera chegar já configurada para Somar (deixada
> assim por outra ferramenta ou por uma redefinição de fábrica), o Chloros detecta isso a partir de uma leitura do nó e
> compensa; executar o `lattice resolution --binning N` a coloca de volta na “Average”. A versão 1.2.0
> `--binning-mode` ainda é aceito, de modo que scripts mais antigos são executados, mas ele apenas exibe um aviso:
> o binning permanece em Média.

> **Alterar o agrupamento, o formato de pixel e o gatilho em uma câmera não surte efeito.** Em uma câmera em que o
> backend permanece aberto e transmite dados (`cam-connect`, a GUI), `lattice resolution --binning`
> não surtem efeito (exibe que o binning ainda está em vigor, `Binning: 1x1`), e
> `lattice format --set` e `lattice trigger --mode` são rejeitados pela câmera
> (`SC_ERR_ERROR -1001`). Libere-o primeiro (`lattice cam-disconnect -s <SN>`) ou defina
> esses parâmetros na conexão (`cam-connect --pixel-format …`, `array-connect --binning …`).

##### Leitura da exposição e do ganho no modo automático

Enquanto o `ExposureAuto` (`GainAuto`) de uma câmera estiver em execução, seu nó `ExposureTime` (`Gain`) mantém o **último valor gravado**, e não o que a câmera está fazendo no momento. Para uma câmera cujo backend permanece aberto (`cam-connect`, `array-connect`, a GUI), o `lattice configure --get`, a visualização padrão `lattice configure` e o `lattice exposure` (sem sinalizadores) exibem, portanto, o valor do nó e, abaixo dele, uma nota com o quadro mais recente da câmera:

```

ExposureTime = 2000.448
  note: ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448); the camera's newest frame was at 4900 us.
```

Leia o valor do quadro a partir da nota; o valor do nó é apenas a última gravação. Por meio dHTTP, a mesma resposta `POST /api/camera/<serial>/read-nodes` mantém `nodes` inalterado e adiciona `frame_values` (`{"ExposureTime": <us>, "Gain": <dB>}`, do quadro mais recente) e `notes` (a frase acima). Ambos estão presentes apenas enquanto o modo automático não for `Off` e um quadro for conhecido; uma câmera aberta apenas para leitura não possui quadro e não recebe nenhum dos dois. `lattice gain` (sem sinalizadores) imprime apenas o nó — use `lattice configure --get Gain` para a nota.

##### Exposição em um membro da matriz

Enquanto uma matriz transmite, cada membro opera com **`AcquisitionFrameRateEnable` ativado**, a uma taxa de quadros um pouco acima da taxa de disparo da matriz: seu período é o período de disparo menos 15% (pelo menos 3 ms), ou a taxa máxima da própria câmera quando o disparo já estiver próximo a ela. Portanto, o `configure --get` em um membro é interpretado como `AcquisitionFrameRateEnable`, `True` e `AcquisitionFrameRate` nessa configuração; a ativação é desativada quando a matriz para.**Não grave esses dois nós em um membro da matriz** — a matriz os mantém sincronizados com seu gatilho e os reescreve.

Essa configuração também é o limite máximo de exposição do membro: cerca de 85 ms com um gatilho de 10 fps (período de 100 ms), em **qualquer ROI ou binning**. Antes da versão 1.3.0, uma ROI pequena limitava a exposição de um membro ao período de funcionamento livre do sensor (4,9 ms em 768 × 304), independentemente da lentidão do disparo da matriz.**Um comando manual `ExposureTime` mais longo que o limite máximo desacelera o gatilho para se adequar a ele** quando é gravado com `lattice exposure --set` ou `configure --set` (ou pela GUI). A matriz primeiro abre o limite máximo desse membro para o valor e permite que a taxa mais lenta execute alguns períodos de acionamento, de modo que a gravação pode demorar alguns segundos a mais; o acionamento então ocorre não mais rápido do que a exposição permite. Em uma matriz com taxa automática, a taxa em que ela funcionava anteriormente é restabelecida assim que todos os membros cuja exposição manual a desacelerou voltarem à exposição automática. Um valor que a própria câmera não pode aceitar ainda é recusado, com o intervalo aceitável indicado na mensagem de erro, e a exposição automática do membro é restabelecida se ela tiver sido desativada apenas para aquela gravação. Uma exposição da receita `apply` não diminui a velocidade do disparador: ela é mantida no limite máximo. O Chloros nunca encurta uma exposição manual para se adequar a um disparador mais rápido; uma taxa fixada (`array-connect --target-fps F`) mais rápida do que a exposição permite deixa para aquele membro um limite máximo de tempo suficiente para ela, e esse membro recebe disparos não mais rápidos do que sua exposição permite.

#### Calibração e Sensores

| Subcomando | Finalidade |
| --- | --- |
| `lattice calibrate [--filter NAME] [--attempts N] [--save PATH]` | Calibrar a partir de um alvo de refletância. São pesquisados até `--attempts` quadros (padrão: `30`) para isso; um quadro incompleto (perda de pacotes GVSP) custa uma tentativa, não a execução. A operação falha com `no target detected in N attempts`, ou com `no complete frame in N attempts: <last error>` quando nenhum quadro chega completo. |
| `lattice dls {--connect \| --spectrum \| --irradiance} --mac MAC [--filter NAME] [--camera SERIAL] [--json]` | O espectro e as magnitudes de banda de um DAQ-M, lidos via Bluetooth (BLE) pelo backend. `--mac` é o próprio MAC BLE do sensor e é necessário para qualquer uma das três ações: não há valor padrão, já que cada unidade tem o seu próprio (`daq pool-discover --only ble` os lista); sem ele, o comando sai com o código `1`. Sem nenhuma ação, ele exibe sua ajuda. `--camera SERIAL` integra as magnitudes de banda exibidas por meio da resposta espectral medida pela própria câmera, em vez da curva de frota do modelo de filtro — câmeras RGB, com o pacote de calibração da câmera carregado (conectado) ou armazenado em cache neste computador. O título indica qual curva foi usada: `Band magnitudes (camera <SERIAL>):`, ou `Band magnitudes (model <filter>):` quando a câmera não possui nenhuma (não é uma câmera RGB, ou nenhum pacote carregado ou armazenado em cache) e a curva padrão foi usada em seu lugar. Por meio de HTTP, `POST /api/dls/cli-read` recebe `camera_serial` e retorna `band_magnitudes_source`. |
| `lattice vignette --input DIR --output DIR [--lens-model KEY]` | Aplica correção de vinheta a imagens existentes, cada uma com o campo plano de sua própria lente (a partir do modelo EXIF). O `--lens-model` se aplica a imagens cujo modelo não especifique nenhuma câmera; uma imagem cujo modelo contradiga isso é ignorada. |

#### Multicâmera (Sessões Transitórias)

Um comando transitório abre as câmeras para uma execução e as fecha quando ela termina: `multi-capture` sem matriz persistente conectada, `multi-stream` e `multi-test` aqui, e `align-calibrate`, `align-apply`, `align-stream` e `index --live` abaixo. Essas regras se aplicam a todas elas:

- **O `--serials SN1,SN2,…` seleciona as câmeras.** Apenas essas são abertas, e a primeira aciona o gatilho (para `align-calibrate` e `--reference`, quando fornecidas). Uma câmera serial que não esteja na rede faz com que a execução falhe antes que qualquer câmera seja aberta. Sem o `--serials`, todas as câmeras da rede são abertas. Uma câmera que o Chloros já possui (na GUI, `cam-connect`, `array-connect`) recusa a execução, independentemente de o `--serials` indicá-la ou não: ela compartilha a rede para a qual a execução foi configurada. (O `index --live`, sem o `--serials`, abre, em vez disso, as câmeras habilitadas em seu perfil: consulte [Índice / Matemática da Vegetação](#index--vegetation-maths).)
- **As câmeras compartilham a rede da mesma forma que o `array-connect` a compartilha.** Cada câmera é ajustada à sua parcela do tráfego transportado pelo link do host (um atraso entre pacotes, não o limite de taxa de transferência da câmera, cujo piso de 31,25 MB/s já é excedido por quatro câmeras em 1 GbE), portanto, quatro câmeras operam em um link de 1 Gbps. Quando as câmeras não conseguem enviar todas ao mesmo tempo no ritmo mais lento que alcançam, elas são acionadas uma após a outra, cada uma com o link dedicado a si mesma, e o comando exibe `Serial emit: the wire cannot carry these cameras at once, so each is triggered <ms> ms after the one before; their frames are not exposed together`. Quando mesmo uma única câmera por vez não cabe, a execução é recusada antes que qualquer câmera transmita: `array over-subscribes the wire: <n> cameras demand <x> MB/s but the collision-safe ceiling is <y> MB/s. …`, HTTP 409, saída `1`.
- **Todas as câmeras devem transmitir, ou a execução falha.** Uma câmera que não transmite nenhum quadro é um erro: o comando exibe `Error: … no frame from <n> of <total> cameras: SN:<serial> (<why>), …`, onde `<why>` é o erro de busca dessa câmera, seguido da indicação da causa, e encerra com `1`. Um quadro que chegou com perda de pacotes GVSP (`IncompleteBufferError: GVSP packet loss …`) significa que a rede não está suportando a largura de banda combinada das câmeras (`Every missing frame arrived with GVSP packet loss: …`): verifique a velocidade do link do host e do uplink do switch (`lattice network`), use quadros jumbo, ou abra menos câmeras com o comando `--serials`. Uma câmera que não transmitiu nada geralmente não está recebendo o sinal de acionamento do mestre: verifique seu cabo de sincronização M8 e, em seguida, execute o comando `lattice multi-detect`. O que cada comando faz com um grupo de quadros em que falta uma câmera está descrito na respectiva linha.
- **Cada câmera é restaurada ao estado em que foi encontrada.** Quando a execução termina, todas as câmeras voltam ao estado em que estavam quando foram abertas: `TriggerMode`, `TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, e `LineMode` / `LineSource` em `Line2` e `Line3` (e a entrada de disparo, que elimina ruídos em uma matriz, fornece a linha de um escravo, `LineFilterSelector` / `LineFilterWidth`); seu `PtpEnable` (e o conjunto `PtpSlaveOnly` / `PtpDomainNumber` definido com ele); `ExposureAuto` / `GainAuto` (e `ExposureTime` / `Gain`, onde estava definido com um valor manual); `TargetBrightness`, `ExposureAutoLimitAuto` / `ExposureAutoUpperLimit` / `ExposureAutoLowerLimit`, `GainAutoUpperLimit`, `ExposureAutoDamping`; `BalanceWhiteAuto` (e cada `BalanceRatio` em que o equilíbrio de branco era manual); `BlackLevelAuto` / `BlackLevel`; `PixelFormat`, binning, `Width` / `Height` / `OffsetX` / `OffsetY`; `GammaEnable` (e `Gamma` em que o gama estava ativado), `LUTEnable`, `ColorTransformationEnable`; `ReverseX` / `ReverseY`; `UserSetSelector`; o tamanho do pacote (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`); o ritmo, `DeviceLinkThroughputLimitMode` (e `DeviceLinkThroughputLimit`, onde o modo era `On`), ode pacotes `GevSCPD` e o deslocamento de início `GevSCFTD` que uma matriz fornece a cada câmera; os blocos de dados que cada fluxo ativa (`ChunkModeActive`, cada `ChunkEnable` sob seu `ChunkSelector`); `AcquisitionFrameRateEnable` (e `AcquisitionFrameRate` onde a ativação estava ligada — com ela desligada, a taxa é somente para leitura e lê o máximo que o ritmo permite, portanto não é uma configuração). Apenas o que difere é gravado: a câmera parou primeiro, o ritmo e a taxa de quadros antes de uma exposição que requer seu período de quadro (uma câmera limita o `ExposureTime` a um período de quadro). Com o `--keep-settings`, a fiação do disparador e da linha de sincronização é desativada, a câmera opera em modo livre e o restante é salvo na própria memória da câmera conforme definido pela configuração de execução; o `--clear-settings` restaura as configurações de fábrica.
- **A exposição é definida a partir da cena antes que o primeiro quadro seja mantido.** A execução captura grupos de quadros sincronizados e os descarta enquanto define a exposição e o ganho de cada câmera a partir desses quadros. O objetivo é obter um brilho médio no alvo da câmera, com no máximo 1% do quadro cortado; uma cena com realces brilhantes fica aquém do alvo. Isso geralmente leva de 2 a 6 grupos de quadros e, no máximo, 5 s.
  - Cada câmera mira no alvo a partir do qual uma matriz conectada a inicia, por meio de seu filtro: TargetBrightness **40**em uma câmera multiespectral (RGN, OCN, NGB) e**60** em uma câmera RGB. Isso equivale à metade da meta de exposição fornecida pelo valor 80 (três quartos em RGB), de modo que um painel de refletância brilhante tem menos chances de sofrer recorte. `--brightness N` (0-255) define a meta de cada câmera (`multi-test` não possui `--brightness`: sua exposição automática está desativada). Os logs do backend nomeiam a meta de cada câmera: `[transient-ae] AE target per camera (filter default): SN:<serial> <target>, …`.
  - A exposição vem primeiro, até o limite de exposição automática da própria câmera. O ganho cobre apenas o que resta.
  - A exposição e o ganho determinados são mantidos durante toda a execução; portanto, todos os quadros de uma `multi-stream` são capturados com a mesma exposição. Uma captura que precise acompanhar mudanças de luz deve ser feita em uma matriz conectada (`array-connect`), cuja exposição automática continua se ajustando.
  - `--exposure` e `--gain` são mantidos exatamente como fornecidos. Com ambas especificadas, nenhum quadro é descartado.
  - Quando a execução termina, a exposição automática de cada câmera é reativada.
  - O log do backend relata o resultado: `[array-ae] exposure solved in <s> s (<n> warm-up groups): SN:<serial> <ms> ms <dB> dB mean <m>/<target>, …` ou `exposure NOT solved after …` com o motivo, e `(not settled)` para cada câmera que não foi incluída. Nesse caso, os quadros ainda são capturados.
  - `CHLOROS_TRANSIENT_AE_SETTLE_S` altera o limite de 5 s; `0` desativa essa função (consulte [Variáveis de ambiente](#environment-variables)).

**Enquanto o Chloros mantiver qualquer câmera** (a interface gráfica, o `cam-connect` ou um conjunto conectado), o `multi-stream`, o `multi-test`, `multi-detect`, `align-apply`, `align-stream` e `index --live` se recusam a ser executados, pois abrem todas as câmeras da rede e uma segunda abertura de uma câmera retida pode travar o backend. (O `index --live` recusa quando uma câmera que ele abriria está retida: seja o `--serials` ou as câmeras de seu perfil, e qualquer câmera de um perfil que não especifique nenhuma.) O backend responde com o código HTTP 409 para o `CAMERA_HELD`, indicando cada câmera bloqueada e os comandos de liberação (`chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`), e o comando é encerrado com o código `1`. Com uma matriz conectada, o comando `array-capture --continuous` a grava no lugar do comando `multi-stream`, e o comando `array-capture` (alinhado por padrão) salva um conjunto alinhado no lugar de `align-apply`; `multi-capture` e `align-calibrate` utilizam uma matriz conectada que contém suas câmeras em vez de abri-las.

| Subcomando | Finalidade |
| --- | --- |
| `lattice multi-info [--serials SN1,SN2,…]` | Lista todas as câmeras com funções de sincronização; com `--serials`, apenas essas; a primeira como `MASTER`. |
| `lattice multi-capture [--serials SN1,SN2,…] [--format FMT] [--jpeg-quality N] [--processing LEVEL] [--brightness N] [--no-daq] [--daq-tolerance-ms MS]` | Um quadro sincronizado de cada câmera. Salva **todos os tipos de exportação por padrão**quando uma matriz persistente está conectada, e a leitura do DAQ utilizada ([A leitura do DAQ que realiza uma captura salva](#the-daq-reading-a-capture-saves)); o fallback transitório sem matriz é**apenas despixelado** (execute `array-connect` primeiro para o restante). O `--brightness` se aplica apenas ao recurso de fallback: um array conectado mantém sua própria exposição automática. O recurso de fallback não salva o `.daq` (`--no-daq` / `--daq-tolerance-ms` não têm efeito ali), e o `multi-capture` não aceita o `--force-daq`. No fallback, uma câmera que não forneça nenhum quadro falha na captura (`Error: multi-capture failed: no frame from …`, sair de `1`) e nenhum arquivo desse grupo de quadros é salvo. |
| `lattice multi-stream [--serials SN1,SN2,…] [--fps F] [--count N] [--format FMT] [--jpeg-quality N] [--brightness N]` | Transmite quadros sincronizados (transitório), `--fps` padrão `3`, até Ctrl+C ou `--count` grupos. Um grupo de quadros para o qual uma câmera não enviou nada não é gravado, e três seguidos encerram o fluxo. Se algum grupo foi pulado, a execução termina imprimindo `Incomplete: <n> frame group(s) not written, a camera delivered nothing (SN:<serial> x<count>, …)`, além de `Error: the stream stopped: …` indicando o nome da câmera quando ela encerrou o fluxo, e sai do `1`X. Interrompe-se automaticamente quando a unidade fica cheia, exibe `Failed:` / `Disk full:` e sai com `1`; qualquer arquivo que não tenha sido possível gravar também encerra o `1`. |
| `lattice multi-test [--serials SN1,SN2,…] [--count N] [--ptp-settle SECONDS]` | Teste de temporização de sincronização GPIO (transitório), `--count` acionadores padrão `10`. Exibe como a rede é compartilhada (`Wire: each camera paced to <x> MB/s (<n> on a <y> MB/s ceiling, <link> Mbps link)`, e a linha `Serial emit:` quando as câmeras são acionadas uma após a outra, cujo desfasamento é então incluído nos carimbos de data/hora), uma linha `Sync group: master SN:<serial> -> SN:<serial>, …` por grupo de sincronização (`(no slaves: not synced)` para uma câmera sem cabo de sincronização, que é acionada por conta própria), em seguida, uma linha por acionamento: `[  0] SN:<serial>=<timestamp> \| …`, com `SN:<serial>=(no frame)` para uma câmera que não enviou nada para aquele acionamento. Em seguida, uma linha `Offsets from master SN:<serial> (<PTP / device (PTP off)> clock, <n> trigger(s)):` e uma linha `SN:<serial>: mean <±x> us, jitter <y> us` por câmera, cada câmera em relação ao mestre dentro de um ciclo de relógio: um acionamento cujos carimbos misturam os relógios dos dispositivos das câmeras e o PTP (PTP desativado ou ainda em sincronização) é deixado de fora, com um `Note:` contando-os. Qualquer quadro perdido faz com que o teste seja reprovado (`Error: multi-test failed: <k> of <n> frame groups incomplete (SN:<serial> missed <k>); …`, saída `1`); três gatilhos incompletos seguidos encerram o teste antecipadamente. Ele também falha (saída `1`) quando menos de dois gatilhos mantêm todas as câmeras em um único relógio (`… the clocks cannot be compared`), ou quando os relógios nunca coincidem: Relógios PTP com desvio superior a 1 ms (mais qualquer desfasamento da camada serial) em relação ao mestre (`… PTP is not locked`), ou relógios de dispositivos cujo desvio varie mais do que isso entre os gatilhos. Ele inicia o grandmaster PTP caso este não esteja em execução, ativa o PTP de todas as câmeras (como faz o `array-connect`, independentemente da configuração da rede) e, antes do primeiro gatilho, aguarda que o relógio PTP de todas as câmeras sincronize com o grandmaster e se estabilize: `PtpStatus` Escravo em um grandmaster, com desvio de até 100 us em relação a ele e estável por 3 s, e cada câmera sincronizada dessa forma, sem interrupção, por 120 s — cerca de 2 minutos após a ativação. Um relógio que acabou de se sincronizar apresenta um desvio de até ~200 us por cerca de 2 minutos após ser lido como Escravo, e o próprio `PtpOffsetFromMaster` da câmera não mostra isso; 120 s fazem com que as câmeras fiquem a ~10 us de um relógio estabilizado. Uma câmera ainda está sincronizada desde um teste anterior que deixou o PTP ativado, conforme observado por este backend, conta a partir desse sincronismo; portanto, um teste realizado logo após outro não precisa aguardar novamente; uma câmera que o teste liga, ou uma encontrada com o PTP ativado que este backend ainda não tenha visto sincronizada (ligada em outro lugar ou antes de uma reinicialização do backend), aguarda seus 120 s, e em uma matriz com ambas as câmeras recém-sincronizadas, elas ainda aguardam o tempo necessário. Uma câmera que sai do modo Escravo, ultrapassa 100 us ou segue outro grandmaster por 3 leituras consecutivas (1,5 s) durante a espera reinicia sua contagem; uma ou duas dessas leituras são registradas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e a contagem continua; um ID de grandmaster que não possa ser lido é considerado desconhecido, e não como uma alteração. O reinício simultâneo da contagem de todas as câmeras (em menos de um segundo) indica que o tempo do “grandmaster” está avançando, e não o das câmeras: o registro do backend mostra `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, `ptp_lock` lista isso em `grandmaster_steps`, e uma espera que então expira inicia seu erro com `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. O limite não é estendido para isso. `--ptp-settle SECONDS` define o tempo de estabilização para a execução (0-600; `0` para nenhuma espera de estabilização). A espera é limitada ao tempo de estabilização mais 90 s (210 s por padrão, no mínimo 90 s), e o comando aguarda o backend por 240 s mais o tempo de estabilização (360 s por padrão). Ele exibe `PTP: locked after <s> s (each camera locked <settle> s, already locked: SN:<serial>, …; from the grandmaster: SN:<serial> <±x> us, …)`, e o log do backend apresenta uma linha a cada 5 s com o estado e o deslocamento de cada câmera, há quanto tempo ela está travada e quanto tempo ainda resta: `[array-sync] PTP at <s> s: SN:<serial> Slave <±x> us for <s> s (<s> s left), … -- <s> s left, the wait gives up at <s> s`. Todas as câmeras são ligadas com o PTP desativado; a partir daí, os relógios se sincronizam em cerca de 10 s. Depois, cada câmera é restaurada ao estado em que o teste a encontrou — seu PTP, a exposição automática e o ganho, que o teste mantém desativados durante a execução, e o restante (a menos que `--keep-settings`) — portanto, uma câmera que ficou com o PTP ativado (`lattice configure -s <serial> --set PtpEnable=True` ou uma execução com `--keep-settings`) permanece sincronizada, e uma vez que este backend tenha constatado que ela está sincronizada pelo tempo de estabilização, um teste posterior não aguarda por ela. Se os relógios não se estabilizarem durante a espera, o teste falha antes de ser acionado (`PTP: NOT locked …`, saída `1`) e indica o que fazer: uma câmera que nunca ficou travada (`PtpStatus Listening`) significa que o grandmaster não está conseguindo acessá-la (`time-sync status`); uma que ficou caindo constantemente é identificada pelo tempo que permaneceu travada (`… had not been locked for the 120 s settle time … (SN:<serial> locked <s> s, <s> s to go)`); relógios ainda convergindo: verifique `time-sync status`, e, para que fiquem sincronizadas antes do teste, ative o PTP e deixe-o ativado (`lattice configure -s <serial> --set PtpEnable=True` para cada câmera, ou execute o teste com `--keep-settings`; o teste restaura o que encontrou, portanto ele permanece ativado), depois execute-o. |
| `lattice multi-detect [--line LINE] [--json]` | Encontre as câmeras na linha de sincronização M8 e um mestre padrão: a câmera RGB (modelo terminando em `-FRGB`) se estiver conectada por cabo; caso contrário, a primeira câmera conectada por cabo na ordem serial. Os cabos M8 se encontram em uma junção, portanto **qualquer câmera conectada pode ser a mestre** — o código impresso `MASTER` é um padrão, não uma característica da fiação. Cada câmera, por sua vez, aciona a linha e as outras são verificadas quanto ao seu pulso; uma câmera já encontrada no grupo conectado não é verificada novamente, portanto, um conjunto totalmente conectado leva uma rodada. Na ausência de `--line`, ele verifica `Line2`, depois o `Line3`. Imprime o `MASTER: SN:…`, um `SLAVE: SN:…` para cada outra câmera conectada e o `(probed on <line>)`; o `--json` soma o resultado completo (`master`, `slaves`, `no_cable` — câmeras que o pulso não atingiu — `line` e, por câmera, `results`). Ao terminar, cada câmera recupera as configurações de disparo e linha de sincronização que possuía (os mesmos nós mencionados acima). |

#### Alinhamento

| Subcomando | Finalidade |
| --- | --- |
| `lattice align-calibrate [--serials SN1,SN2,…] [--method orb\|akaze\|phase\|checkerboard\|manual] [--model translation\|rigid\|affine\|homography] [--frames N] [--checkerboard RxC] [--points PATH] [--reference SN] [--save PATH] [--preview] [--vignette] [--prefilter none\|gradient\|clahe\|blur\|hist_match] [--rms-threshold-px N] [--brightness N]` — além dos controles de detector/correspondência `[--max-features N] [--ratio-threshold F] [--matcher bf\|flann] [--knn-k N]`, parâmetros do RANSAC `[--ransac-threshold-px F] [--ransac-iters N] [--ransac-confidence F]`, combinação de múltiplos quadros `[--averaging mean\|median\|inlier_weighted]`, restrições geométricas `[--lock-rotation] [--lock-scale] [--lock-axis x\|y]`, restrição espacial `[--roi X0,Y0,X1,Y1] [--mask PATH]` e substituiçõessub-escravo `[--per-cam-override SN:KEY=VALUE]` (repetível) | Calcula o perfil de alinhamento a partir de câmeras em tempo real. Padrões: `--method orb`, `--model rigid`, `--frames 3`, `--save alignment_profile.json` (na pasta a partir da qual você o executa), `--max-features 2000`, `--ratio-threshold 0.75`, `--matcher bf`, `--knn-k 2`, `--ransac-threshold-px 3.0`, `--ransac-iters 2000`, `--ransac-confidence 0.99`, `--averaging mean`. (O `project align calibrate` possui seus próprios valores padrão; consulte [sua tabela](#project-align-calibrate-options).) O `--prefilter` usa como padrão o `gradient` (mapa de bordas; corresponde ao alinhador de GUI/matriz — as bordas permanecem entre as faixas espectrais). O `--matcher flann` apresenta bons resultados acima de ~5.000 características; o `--averaging median` é robusto contra uma captura com defeito; o `inlier_weighted` pondera pela contagem de correspondências; o `--lock-scale` projeta para a rotação mais próxima (sem escala), `--lock-axis` zera um componente de translação; O `--mask` se aplica a todas as câmeras (use o `--per-cam-override` para configurações específicas por câmera, por exemplo, o `--per-cam-override 214701292:method=phase`). `--rms-threshold-px` recusa salvar uma calibração cujo RMS de reprojeção exceda o limite. `--method manual` obtém seus pontos de `--points PATH`: um objeto JSON, o `{"<serial>": [[x, y], ...], ...}`, que fornece as coordenadas em pixels de cada câmera para as mesmas características, incluindo as da câmera de referência (pelo menos 2 por câmera); cada um dos `--frames` é calculado a partir desses pontos. Quando um array conectado contém as câmeras, a calibração utiliza os quadros em tempo real desse array, e o perfil também se torna o alinhamento do array para suas capturas. Caso contrário, ele abre um [array transitório](#multi-camera-transient-sessions): uma câmera que não forneça nenhum quadro em qualquer um dos `--frames` falha na calibração (sai do `1`, nomeando-a), e nenhum perfil é salvo. |
| `lattice align-apply --profile PATH [--serials SN1,SN2,…] [--format tiff\|png] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-camera] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode constant\|replicate\|reflect\|wrap] [--border-value N] [--brightness N]` | Captura um quadro multibanda alinhado. `--bit-depth`, por padrão, faz a correspondência com a câmera; `--no-crop` mantém o quadro completo (preenchido com preto); `--interpolation` (padrão `linear`) e `--border-mode`/`--border-value` (padrão: `constant`/0) controlam a distorção da CPU — o caminho da GPU é bilinear independentemente disso. Abre uma [matriz transitória](#multi-camera-transient-sessions): uma câmera que não forneça nenhum quadro falha na captura (sai de `1`, nomeando-a), e nada é gravado. |
| `lattice align-stream --profile PATH [--serials SN1,SN2,…] [--fps F] [--count N] [--bit-depth 8\|12\|16] [--bands NAMES] [--order NAMES] [--gpu\|--no-gpu] [--no-crop] [--per-band] [--vignette] [--interpolation nearest\|linear\|cubic\|lanczos] [--border-mode MODE] [--border-value N] [--brightness N]` | Transmite quadros multibanda alinhados (mesmos controles de warp que `align-apply`), `--fps` padrão `3`. Abre uma [matriz transitória](#multi-camera-transient-sessions): um grupo de quadros para o qual uma câmera não entregou nada não é salvo, e três seguidos encerram o fluxo. Se algum grupo foi pulado, a execução termina exibindo `Incomplete: <n> frame group(s) not saved, a camera delivered nothing (SN:<serial> x<count>, …)` e `Error: …`, e encerra com `1`. Um fluxo que termina devido a qualquer outro erro também exibe `Error: …` e encerra com `1`. |
| `lattice align-info --profile PATH [--json]` | Exibir detalhes do perfil. |
| `lattice align-reorder --profile PATH [--order NAMES] [--enable SERIALS] [--disable SERIALS]` | Alterar a ordem das camadas. |

#### Índice / Matemática da Vegetação

```bash
# Offline: compute NDVI from an aligned multi-band TIFF (an RGN camera's bands)
chloros-cli lattice index --input aligned.tif --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --output ndvi.tif --colorize --gradient RdYlGn

# Live: open the cameras, capture one aligned set with a profile made by
# `lattice align-calibrate`, and compute the index from it
chloros-cli lattice index --live --profile align.json --preset NDVI \
  --channel red=Red_660_RGN --channel nir=NIR_850_RGN \
  --save-multiband -o output/
```

**`--channel` mapeia os símbolos de uma predefinição para as faixas** (NDVI: `red`, `nir`), um `--channel SYMBOL=BAND` para cada um.

- **`--live`: apenas para um símbolo que a pilha não consegue responder.** Assim como no `--input`, um símbolo de uma predefinição que você omitir assumirá a única banda da pilha (as bandas das câmeras ativadas do perfil) cujo nome comece com ele. Isso é verificado antes que qualquer câmera seja aberta: um símbolo ao qual nenhuma banda, ou várias bandas, respondem é recusado nesse momento, nomeando-as, e sai do `1`. Um `--formula` personalizado precisa de um `--channel` para cada símbolo.
- **Os produtos `--live` indicam a quem pertencem.** `index.tif` e `index_LUT.png` contêm os dados da câmera da pilha (marca, modelo, número de série), sua hora de captura (exposição da câmera de referência, UTC) e a lente, e ambos nomeiam o índice como sua única banda (`XMP-Camera:BandName` `NDVI`; o PNG colorido nomeia o índice, não seus canais de cor), assim como faz um produto `--input`. Um índice proveniente das bandas de várias câmeras, por outro lado, nomeia cada uma delas, e seu `Model` é `LATTICE` — consulte [Arquivos criados a partir de várias câmeras](#files-made-from-several-cameras).
- **`--input`: apenas para um símbolo que o arquivo não consegue identificar.** Um símbolo que você omitir será atribuído à única banda do arquivo cujo nome comece com ele: `red` assume `Red_660` ou `Red_660_RGN`; `nir` assume `NIR_850`. Os nomes provêm do próprio arquivo: da lista de bandas da pilha ou, em uma captura de radianância/refletância do Chloros ou no produto `process`, suas tags `Chloros:Band{R,G,B}Name` / `XMP-Camera:BandName`. Portanto, o `lattice index --input <reflectance capture> --preset NDVI` não precisa de `--channel`.
- **Duas bandas que correspondem a um único símbolo não são distinguidas** (uma pilha de 9 câmeras possui duas bandas NIR): nomeie essa. Um símbolo ao qual nada corresponde é rejeitado com `channel_map missing entries`; saia de `1`.**Os nomes das bandas em uma pilha** indicam o filtro da câmera: as bandas de uma câmera M3C são `<band>_<filter>` — `NIR_850_RGN`, `Green_550_RGN`, `Red_660_RGN` em um RGN; `NIR_808_OCN`, `Cyan_490_OCN`, `Orange_615_OCN` em um OCN; `Blue_475_RGB`, `Green_550_RGB`, `Red_625_RGB` em um RGB; `Blue_475_NGB`, `Green_550_NGB`, `NIR_850_NGB` em um NGB — e a banda única da câmera M3Mbanda única é `Mono_<filter>` (`Mono_660`, `Mono_850`). `lattice align-info --profile PATH` lista as bandas de um perfil.

Conjunto completo de sinalizadores: `--input PATH | --live --profile PATH`, `--preset NAME` (NDVI/NDRE/EVI/SAVI/GNDVI/…), `--formula EXPR`, `--channel SYM=BAND` (repetível), `--capture-level raw|debayered|radiance|reflectance|unknown` (substitui o nível de captura registrado no arquivo TIFF de origem; padrão: lido nos metadados do TIFF, recorrendo ao  de um arquivo Chloros, `Chloros:ProcessingLevel`; assim, uma captura de refletância reporta `reflectance`), `--output PATH` / `-o` (offline: um caminho; com uma extensão de imagem, os arquivos são nomeados a partir de seu radical — `-o ndvi_FRGN.png` grava `ndvi_FRGN.tif` e `ndvi_FRGN_LUT.png` — e, sem ela, é um prefixo, `-o ndvi` -&gt; `ndvi.tif`, `ndvi_LUT.png`; padrão ao lado da entrada, como `<input name>_<preset>`; com `--live` e sem `-o`, uma pasta `output/` na pasta a partir da qual você executa o comando), `--output-format all|raw|tif|colorized|lut|png`, `--gradient NAME|JSON`, `--vmin/--vmax/--percentile LO,HI`, `--bg-mode clip|transparent|indexColor|backgroundColor`, `--colorize`, `--list-presets`, `--list-gradients`. Com o `--live`, os controles de distorção de alinhamento também se aplicam: `--save-multiband`, `--gpu/--no-gpu`, `--no-crop`, `--bit-depth 8|12|16`, `--vignette`, `--interpolation nearest|linear|cubic|lanczos`, `--border-mode constant|replicate|reflect|wrap`, `--border-value N`, `--brightness N` (a meta de exposição automática das câmeras; consulte as regras de transientes), `--serials SN1,SN2,…`. `--live` abre uma [matriz de transientes](#sessões-transitórias-multicâmera): uma câmera que não forneça nenhum quadro faz com que a execução falhe (saída de `1`, nomeando-a), e nenhuma pilha ou índice é gravado. Ele abre o `--serials` (o primeiro aciona o gatilho); caso contrário, as câmeras do perfil cuja entrada `layer_order` esteja habilitada, começando pela referência — mesmo que as bandas da referência estejam desativadas, já que as outras estão alinhadas a ela e aciona o gatilho. Uma câmera cujas bandas estejam desativadas nunca é calibrada; portanto, uma câmera RGB desativada no perfil não falha mais em um índice de refletância (`no-absolute-radiance-scale`). Um perfil que não nomeie nenhuma câmera abre todas as câmeras da rede, como antes.

Todos os formulários são executados no backend, portanto ele deve estar em execução: inicie-o a partir da GUI do Chloros ou com o comando que a CLI exibe quando não consegue acessá-lo. Offline, o backend lê o `--input` e grava o TIFF de precisão flutuante e o PNG colorido ele mesmo; portanto, um `--input` ou `-o` é resolvido em relação à pasta a partir da qual você executa o comando (consulte [Convenções](#conventions)). Ambos os produtos offline contêm os dados de registro da câmera da entrada, assim como uma exportação de índice `process`: Marca / Modelo / número de série, GPS, as tags da lente, a hora da captura (a própria da entrada; quando seus dados EXIF não indicam nada, o instante da exposição registrado ou o carimbo do nome do arquivo, em UTC) e a identidade de calibração MAPIR — nunca a calibração raw-DN, que não se aplica a um índice — além de `Chloros:ProcessingLevel=index` e da expressão do índice; o TIFF de tipo float nomeia uma banda, a predefinição. `--list-presets` e `--list-gradients` imprimem as tabelas de predefinições e gradientes do backend. Uma fórmula rejeitada pelo backend, ou uma banda `--channel` que o TIFF não possui, encerra o `1` com o motivo informado pelo backend.

> **Os símbolos do `--channel` diferenciam maiúsculas de minúsculas.** O lado do símbolo deve corresponder exatamente aos nomes dos canais da predefinição (as predefinições usam letras minúsculas, por exemplo, NDVI = `red`,`nir` — verifique `--list-presets`), e o lado da banda deve corresponder a um nome de banda na pilha alinhada (ou ser um índice de banda com base em 0 no modo offline). `--channel red=Red_660_RGN --channel nir=NIR_850_RGN` funciona; `--channel RED=660` falha com um erro `channel_map missing entries` .

#### `lattice index` — máscaras de pilha multibanda

Em `--capture-level radiance` ou `reflectance`, a separação 3×3 de cada câmera pode limitar uma banda a 0 (consulte [Máscara de limitação de desmistura](#unmix-clamp-mask-which-pixels-a-band-was-clamped-on)). Uma pilha multibanda contém bandas de várias câmeras, e cada câmera limita a sua própria banda; portanto, o `--live --save-multiband` grava **uma máscara por câmera de origem** ao lado da pilha:

```
output/
├── index_multiband.tif
└── clamp_masks/
    ├── index_multiband_SN<serial A>-<filter A>_clampmask.tif
    └── index_multiband_SN<serial B>-<filter B>_clampmask.tif
```

- Cada máscara é de banda única `uint8` na grade de pixels **da pilha** (por meio do mesmo alinhamento, distorção e recorte), com os próprios bits dessa câmera: bit 0 = a banda do canal R, bit 1 = G, bit 2 = B. Uma banda da pilha retirada do canal *c* de uma câmera é limitada onde o bit *c* dessa câmera (R = 0, G = 1, B = 2) está definido.
- A proveniência incorporada de cada banda indica o nome da máscara da respectiva câmera (`unmix_clamp_mask`, em relação à pasta da pilha, ou `none`) e o bit a ser lido (`unmix_clamp_mask_bit`).
- Uma câmera cuja separação de cores não limitou nada não recebe nenhum arquivo; uma câmera monocromática ou RGB não executa a separação de cores e também não recebe nenhum. A CLI exibe cada máscara sob a pilha (`clamp mask SN…: …`); a resposta as lista como `clamp_masks`.
- O TIFF/PNG de índice da mesma execução é calculado a partir da mesma pilha, na mesma grade, portanto as máscaras também o descrevem. `--capture-level raw` não executa o unmix e não grava nenhuma máscara.
- A pilha e o índice ficam diretamente em `--output`, portanto as máscaras vão para o próprio `clamp_masks/` de `--output` — nunca acima da pasta que você nomeou. Essa pasta mistura tipos de produtos (pilha, índice, renderização colorida): selecione seus arquivos, não passe a pasta inteira para o software de fotogrametria.

#### Conexões persistentes (Smart-Prep, fluxo equivalente à GUI)

Esses comandos mantêm as câmeras abertas no pool de back-end entre as invocações dCLI.

| Subcomando | Finalidade |
| --- | --- |
| `lattice cam-connect [--serial SN]` | Adiciona uma câmera ao pool (câmera única, sem matriz). As [opções comuns](#common-options-shared-by-most-subcommands) `--exposure`, `--gain`, `--preset`, `--pixel-format`, `--width` / `--height` e `--packet-size` (`standard` = 1500, `jumbo` = 9000 ou bytes; sondas `auto`) são aplicados na conexão (um `--exposure` ou `--gain` desativa o modo automático). Ele retorna assim que o fluxo da câmera for iniciado (aguardando cerca de 15 s para isso e, em seguida, 10 s pelo primeiro buffer completo) e exibe `stream: streaming` ou `stream: no_frames -- connected, no frames yet: …` quando o fluxo não entregar nada — para uma câmera com pacotes jumbo, com a recomendação de se reconectar com `--packet-size standard`. Enquanto uma câmera se conecta (a primeira conexão baixa sua calibração, cerca de 70 s), outro comando que precisa da câmera aguarda por ela no máximo 5 s, em seguida, falha com o código HTTP 503 `CAMERA_POOL_BUSY`, indicando a câmera que está se conectando; execute-o novamente em alguns instantes. `lattice status` e a lista de câmeras nunca aguardam: uma câmera no meio da conexão aparece como “conectando”, com sua fase. |
| `lattice cam-disconnect [--serial SN] [--all] [--keep-settings \| --clear-settings]` | Liberar. A câmera é restaurada ao estado em que foi encontrada na conexão (fiação do disparador, PTP, configurações); `--keep-settings` salva suas configurações em sua própria memória; `--clear-settings` redefine essa memória para as configurações de fábrica (consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings)). Uma câmera que seja membro de um conjunto libera todo o conjunto. |
| `lattice cam-list` | Lista as câmeras no pool. |
| **`lattice array-connect`**|**Conecta um conjunto sincronizado persistente (O ponto de entrada recomendado).** Executa todo o fluxo de preparação inteligente da GUI. Leva cerca de um minuto para 9 câmeras; logo após as câmeras serem ligadas, pode demorar vários minutos. Enquanto aguarda, exibe cada etapa alcançada pelo backend, como `    <s> s: <step>` (por exemplo, `    41 s: open SN:214000535 (4 of 9)`), e o log do backend apresenta uma linha de tempo por etapa (`[array-connect <id>] <step>: <s> s …`) e uma última linha com as etapas mais demoradas. Ele aguarda até 600 s. Uma conexão não concluída até esse momento é desfeita pelo backend: cada câmera que ele abriu é colocada de volta como estava quando foi encontrada e fechada; o comando exibe `Error:` com a etapa alcançada e encerra em `1`, e nenhuma matriz está conectada (504 HTTP `CLIENT_DEADLINE_PASSED`). Ctrl+C cancela a conexão da mesma forma (`Interrupted: …`); se a matriz já tivesse se conectado, a linha indica isso e nomeia o comando `array-disconnect` que a libera. Um comando encerrado enquanto aguarda (`kill -9`) faz com que a conexão seja desfeita quando o backend não receber resposta por 30 s. Um array que está ativo, mas ainda não possui um grupo de quadros sincronizado, continua sendo relatado como conectado, com seu fluxo indicado como `no_frames`. |
| `lattice array-disconnect [--array-id ID] [--all] [--keep-settings \| --clear-settings]` | Liberar uma matriz. Todas as câmeras ficam em modo de funcionamento livre (`TriggerMode` `Off`), com todas as demais configurações mantidas como estavam no momento da conexão — suas configurações de acionamento e linha de sincronização (a do mestre`TriggerSource` volta ao que era, sua linha de sincronização volta a ser uma entrada), seu PTP e as configurações da câmera alteradas pela sessão — para que um comando de câmera única em um membro funcione imediatamente. `--keep-settings` salva as configurações de cada câmeraconforme a sessão as deixou na própria memória da câmera; o `--clear-settings` redefine essa memória para as configurações de fábrica. Consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings). |
| `lattice array-list` | Listar matrizes conectadas. |
| `lattice array-status [--array-id ID]` | fps ao vivo, PTP, último erro. `ptp_enabled` é igual a `True` somente quando todos os membros apresentam PTP ativado; `ptp_off:` identifica um membro que não o faz. A linha `state:` indica o `restarting (applying settings)` durante uma reinicialização das configurações, o `recovering (…)` enquanto um membro que perdeu energia é reativado (veja a nota abaixo), `stalled (…)` enquanto o ciclo de captura está em execução, mas não publica nenhum grupo de quadros, e `stopped (…)` assim que o ciclo de captura desiste (reconecte o array). |
| `lattice array-capture [--processing LEVEL\|all] [--levels L1,L2,…] [--aligned\|--no-aligned] [--orient] [--index\|--no-index] [--no-daq] [--daq-tolerance-ms MS] [--smart] [--fastest] [--compression deflate\|none] [--continuous\|--interval S] [--count N] [--duration S]` | Uma captura sincronizada da matriz ativa — Única / Contínua / Intervalada / Mais rápido. **Padrão: `all`**(um arquivo por tipo de exportação aplicável por câmera),**alinhado** conforme a captura do aplicativo (`--no-aligned` para arquivos nativos). Câmeras ignoradas (por exemplo, RGB, excluídas da radianância/refletância) são relatadas com `Skipped: SN:<serial> (<reason>)`; a leitura do DAQ utilizada na captura é salva junto com ela, incluindo uma captura apenas em formato bruto, e relatada com `DAQ: <path>` ([O DAQ que lê uma captura salva](#o-daq-que-lê-uma-captura-salva)). Consulte [Modos de captura, gravadores e reprocessamento offline](#modos-de-captura-gravadores--reprocessamento-offline). |
| `lattice array-record [--fps F] [--duration S] [--gif] [--gif-only] [--expr EXPR]` | Grave a visualização do índice combinado em tempo real como vídeo/GIF (nível de monitoramento; sem nenhuma janela da GUI aberta, o backend renderiza o NDVI da matriz, ou `--expr`). |
| `lattice array-burst [--duration S] [--max-frames N] [--orient] [--build] [--products …]` | Série de imagens Bayer brutas com alta taxa de quadros por segundo (nível de análise; reprocessar offline). Assim como `array-record` e `array-capture --continuous`, ele se comunica com o backend a cada sondagem: encerrado sem interrupção (`kill -9`), o backend o interrompe 30 s após a última verificação (`stop_reason` `client_gone`; o manifesto da sequência é finalizado e a construção do índice combinado é iniciada, como ocorre em uma interrupção) e a matriz permanece conectada. |
| `lattice array-build-video --burst-dir DIR [--products …] [--fps F] [--save-tiffs] [--gif]` | Reprocessar uma sequência bruta salva em vídeo(s) calibrado(s). |

> **Uma câmera que perde energia ou sua conexão é restabelecida automaticamente.** Quando um membro para de responder (um ciclo de energia do switch reinicia todas as câmeras que ele alimenta), o conjunto aguarda por ela, reabre a conexão quando ela responde novamente, e restaura as configurações definidas por `array-connect`: PTP, linha de disparo e sincronização, ritmo de rede, formato de pixel e tamanho de quadro, além da exposição automática. Em seguida, ela retoma a transmissão sem nenhuma nova configuração de `array-connect`. Enquanto isso, o `array-status` exibe o `state: recovering (...)` e o `last_error` indica qual câmera está sendo aguardada; em seguida, ele imprime o `re-opened:`. Uma câmera que reiniciou retorna com o PTP desativado; portanto, até que seja reaberta, o `ptp_enabled` é o `False`, e o `ptp_off:` identifica-a.

> **Uma câmera retida nunca é aberta uma segunda vez.** Uma segunda abertura interrompe o fluxo do detentor e reescreve suas configurações, podendo causar uma falha no backend. Portanto, em uma câmera mantida pelo pool (a GUI ou `cam-connect`), um membro da matriz ou uma câmera mantida por uma tarefa em segundo plano (um `lattice continuous` em execução, `multi-stream` ou `align-stream`, seja ela própria ou de outro cliente), `lattice continuous`, `lattice capture`, `cam-connect`, `power --sleep`, `calibrate`, umúnica tentativa `white-balance` e `presets --apply` rejeitam com código HTTP 409 `CAMERA_HELD`, indicando o titular e o comando de liberação (`chloros-cli lattice cam-disconnect -s <SN>`, `chloros-cli lattice array-disconnect --array-id <ID>` ou, no caso de uma tarefa, seu ID e quem a iniciou) e encerrar `1`. O mesmo ocorre com uma câmera que outro comando esteja usando no momento (um `multi-test` ou `multi-capture` abrindo suas câmeras, outro comando único), para todos os comandos aqui: o erro indica que se deve aguardar até que esse comando seja concluído. O comando `lattice power`, sem o `--sleep`, verifica o próprio identificador do detentor, quando houver um (o pool, uma matriz, a câmera aberta de um trabalho), e é recusado se a câmera estiver sendo usada por outro comando. Cada resposta do `CAMERA_HELD` contém o código tanto no `code` quanto no `error_code`. Para os comandos que abrem todas as câmeras, consulte [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions).

##### Opções do `array-connect`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--serials SN1,SN2,…` | detectar automaticamente todas as câmeras LATTICE (requer ≥2) | A primeira serial é a MASTER. Quando omitida, a detecção filtra os modelos LATTICE (M3C / M3M) e conecta todos eles. |
| `--line {Line0,Line2,Line3}` | `Line2` | Linha de sincronização GPIO. |
| `--target-fps F` | automático | Taxa de disparo do mestre. |
| `--force-tier {sim-capture-sim-emit, sim-capture-ftd-stagger, slip-emit-and-capture}` | automático | Substituir o seletor de nível. |
| `--wire-ceiling-mbps MB_PER_S` | detectado automaticamente | **A largura de banda sustentada do host, em MB/s — o valor do qual depende toda a alocação do array.** Reduza-o quando a matriz relatar quadros corrompidos por GVSP: o valor automático é derivado da taxa de link anunciada pela placa de rede, que superestima adaptadores USB, pistas PCIe estreitas e estruturas compartilhadas ocupadas. Persiste no bloco de captura da matriz do projeto, portanto, uma reconexão por reabertura, CLI ou SDK o restaura. Consulte [Integridade da matriz](#array-health--which-subsystem-is-losing-frames). |
| `--binning {1,2,4}` | nenhum (sensor completo) | Agrupamento de pixels por hardware (sempre Média). |
| `--width N` / `--height N` / `--pixel-format FMT` | sensor completo; formato por câmera | Um tamanho de quadro ou formato de pixel explícito para cada câmera, em vez do sensor completo e da escolha do formato por câmera (Mono12 / BayerRG12). |
| `--no-recommend` | desativado | Ignora a etapa de análise de rede (cada câmera mantém então seu tamanho atual, em vez de ser configurada para o sensor completo). |
| `--no-ptp` | desativado | Desativar PTP (os carimbos de data/hora entre câmeras **não** são, então, comparáveis). |
| `--packed {auto,on,off}` | `auto` | Transmita quadros de 12 bits compactados e descompacte-os neste computador — mais quadros por segundo com o sensor completo, dados idênticos. `auto` compacta apenas quando isso for útil e este computador tiver recursos suficientes; `off` nunca compacta. Consulte [Transporte compactado de 12 bits](#packed-12-bit-transport---packed). |
| `--packet-size {auto,jumbo,standard,N}` | `auto` | Tamanho do pacote GVSP para cada câmera na matriz. `auto` O  permite que cada câmera faça um teste de rede; `jumbo`  = 9000, `standard`  = 1500 ou um tamanho em bytes o bloqueia. Antes da versão 1.3.0, o `array-connect` aceitava esse sinalizador e o ignorava. |

> **Sensor completo por padrão.** Sem `--width` / `--height` / `--binning` /
> `--pixel-format`, o `array-connect` configura todas as câmeras para o sensor completo — sem
> binning, sem recorte de ROI — e permite que a taxa de quadros caia para o que este computador e
> a rede suportarem, em vez de reduzir os quadros para manter a taxa alta. A
> linha de análise de rede mostra a taxa projetada, e a linha seguinte lista
> opções mais rápidas (binning ou um ROI menor) com suas taxas projetadas; passe
> `--binning` ou `--width`/`--height` para selecionar uma. O comando “Connect” nunca realiza binning ou recorte
> por conta própria.

##### Transporte compactado de 12 bits (`--packed`)

Com o sensor operando em capacidade total, a taxa de quadros de uma matriz é definida pela velocidade com que cada câmera pode enviar
seus quadros. Os formatos de 12 bits (`BayerRG12`, `Mono12`) normalmente são transmitidos como palavras de 16 bits;
o transporte **compactado** envia os mesmos 12 bits em 1,5 bytes por pixel e
os descompacta neste computador, de modo que cada quadro leva 25% menos tempo para ser enviado —
cerca de **33% mais quadros por segundo na resolução máxima do sensor** (medido: 10,7 → 14,0 fps
em um conjunto de 9 câmeras). **Os dados salvos são idênticos**: mesmos valores, mesmos
arquivos de 16 bits, mesmos metadados.

- O `auto` (padrão) compacta apenas quando isso ajuda **e** este computador tem capacidade para
  realizar a descompactação. Ao se conectar, o backend mede sua própria velocidade de descompactação, o número de núcleos da CPU,
  a carga atual da CPU e a largura de banda disponível, de modo que um desktop com núcleos disponíveis compacta
  com o sensor em capacidade total, enquanto um computador ocupado ou com baixo consumo de energia transmite os dados descompactados. ROIs pequenas e quadros agrupados ganham
  pouco.
- `off` nunca compacta — use-o para desativar a compactação durante o diagnóstico ou em um
  computador que precise manter toda a sua CPU para outras tarefas.
- `on` sempre compacta formatos de 12 bits.

A saída do `connect` mostra a decisão e o motivo (`compactado em 12 bits: em (automático:
projeção de 11,0 -&gt; 14,4 fps, …)`); `/api/camera/array/status` relata
`packed_transport` e `packed_decision`. Para definir um padrão para toda a máquina
em vez de passar o sinalizador a cada vez, defina `CHLOROS_ARRAY_PACKED12=off` (ou
`on`) no ambiente do backend — um `--packed on|off` explícito ainda prevalece.
Os formatos de 8 e 16 bits nunca são compactados.

### Um registro de projeto, todos os modos

Uma pasta de projeto contém as configurações de todos os dispositivos: `cameras.json` — de cada câmera (`cameras[].settings`, para uma única câmera e para cada membro da matriz) e as configurações válidas para toda a matriz (`arrays[].capture`: formato, ROI, binning, taxa de disparo, orçamento de fios) — e `sensors.json`, de cada sensor de luz (tempo de integração, média de quadros, limite). O aplicativo, a CLI, o SDK e o servidor MCP leem e gravam os mesmos arquivos, da mesma maneira:

- **Um dispositivo conectado a um projeto é salvo nele e registra todas as alterações nele.** “Vinculado a um projeto” significa com um projeto aberto no backend (do aplicativo) ou — a partir do SDK — por meio de um `ChlorosProject`, que nomeia sua própria pasta. Uma câmera, matriz ou sensor de luz que ainda não esteja no projeto é adicionada aos arquivos dele quando se conecta (`lattice cam-connect`, `array-connect`, `daq pool-connect`, o aplicativo, o SDK, `chloros-mcp`), com suas configurações. Qualquer alteração em qualquer modo (`lattice exposure`, `gain`, `configure --set`, `resolution`, ase a taxa de disparo, o limite do sensor de luz, os controles do aplicativo) é salva nos arquivos desse projeto um instante depois. Uma exposição, ganho ou meta de AE definida em uma câmera é salva como mantida; portanto, ao reabri-la, ela é mantida novamente.
- **Ao abrir um projeto, seus dispositivos são configurados com base nele**, em todos os modos: uma câmera, matriz ou sensor de luz conectada sem configurações próprias recebe as configurações salvas do projeto (`lattice cam-connect` / `array-connect` com um projeto aberto, `chloros-cli project connect`, `open_project().connect_all()`, ao reabrir o aplicativo). Um conjunto conectado com um novo ID atualiza o conjunto que o projeto salvou com as mesmas câmeras; um conjunto com outras câmeras é adicionado ao lado dele.
- **O próprio salvamento do aplicativo nunca sobrescreve as configurações ativas de um dispositivo** com sua cópia mais antiga e mantém as substituições por membro salvas por um chamador de CLI ou SDK.
- **O formato de pixel definido é aquele que o projeto mantém.** Um formato definido nas Configurações da Matriz, por meio de `--pixel-format` ou `connect_array(pixel_format=)`, incluindo o de 8 bits, é salvo e reconectado exatamente da mesma forma. Se nada for definido, cada câmera se conecta com sua configuração padrão de 12 bits (`BayerRG12`, `Mono12` em um sensor monocromático). O nome de um formato indica sua profundidade de bits; a família é a do sensor, portanto, uma câmera colorida nunca é configurada para um formato monocromático, nem uma câmera monocromática para um formato Bayer, e a linha `[PixelFormat <serial>] ... written` indica isso quando um formato foi mapeado. As exportações de uma captura de 8as exportações de uma captura de 8 bits são de 8 bits, incluindo a refletância: redimensionadas e marcadas com seu `PixelScale`.
- Um dispositivo conectado sem estar vinculado a nenhum projeto não grava nada.
- **Uma gravação segue um projeto criado depois dela; ao ser aberta, ela mantém o projeto original.**Quando um projeto é criado — um SDK `create_project()`, o `chloros-mcp` do `create_project` ou o**Novo Projeto**do aplicativo — ou duplicado (a opção**Duplicar**do aplicativo, que abre a cópia) enquanto uma gravação ainda está gravando no projeto que estava aberto anteriormente, a gravação continua, e o que é gravado vai para o novo projeto, no mesmo local dentro dele (abaixo). Ao abrir um projeto existente — `chloros-cli process`, `project devices --reconnect`, `ChlorosProject.process()`, a opção**Abrir Projeto** do aplicativo — deixa uma gravação ainda em andamento no projeto em que está sendo gravada: ela é concluída ali, e seus arquivos são adicionados à lista de arquivos desse projeto (o aplicativo interrompe todas as gravações antes de abrir um projeto).
  - `array-record` e `array-burst`: o vídeo, ou a pasta de sequência com todos os quadros, é movido inteiro para lá quando a gravação para (seu `--duration`, Ctrl+C). O `Burst dir` gerado é o novo, e o `--build` e o `array-build-video` operam a partir dele. Se a unidade recusar a transferência, a gravação permanece onde foi gravada; odo log explica o motivo, assim como a resposta HTTP da interrupção (`relocation_error`). Os quadros brutos de uma sequência interrompida e o `.daq` são adicionados à lista de arquivos do projeto no qual ela é incluída; um vídeo (um `array-record`, um vídeo gerado a partir de uma sequência) não é adicionado à lista de arquivos de um projeto.
  - `array-capture --continuous`: todas as capturas feitas após a alteração são salvas no novo projeto; as salvas anteriormente permanecem onde estão.
  - `daq pool-record` e `project sensor log` (e a gravação do próprio sensor de luz do aplicativo): o `.daq` é movido para a nova pasta do projetopasta `light_sensor` do novo projeto quando a gravação é interrompida e passa a fazer parte da lista de arquivos desse projeto.

  Uma gravação em uma pasta fora do projeto aberto (um `-o` em outro local) permanece onde está, e o `lattice continuous` e o `multi-stream` mantêm a pasta que lhes foi atribuída.
- **No aplicativo, as capturas feitas após a criação de um novo projeto são incluídas nele.** Criar ou duplicar um projeto mantém todas as sessões de captura e gravações em andamento, e cada captura feita a partir desse momento (Capturar Tudo, Iniciar Captura) é salva no novo projeto e passa a fazer parte de sua lista de arquivos. Abrir um projeto existente no aplicativo interrompe todas elas primeiro.
- **Um projeto não é movido enquanto estiver gravando.**A tentativa de mover o projeto aberto para outro diretório de trabalho (em**Configurações → Diretório de Trabalho** do aplicativo; nenhum comando de CLI move um projeto, mas um cliente HTTP que envia uma solicitação para `/api/move-project` sim) é recusado com o código HTTP 409 `PROJECT_RECORDING` enquanto uma gravação em matriz ou em rajada, uma captura contínua em matriz, um trabalho `lattice continuous` / `multi-stream` ou uma gravação do sensor de luz estiver gravando nele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.` O aplicativo recusa da mesma forma, antes de solicitar uma pasta, enquanto sua própria função “Capturar tudo”, “Iniciar captura” ou gravação do sensor de luz está em execução.

Isso é diferente do que fica definido nas câmeras ao serem desconectadas (consulte [Memória da câmera](#camera-memory----keep-settings----clear-settings)): um projeto define suas próprias câmeras ao ser aberto.

### Memória da câmera — `--keep-settings` / `--clear-settings`

Quando um conjunto é conectado, suas câmeras assumem as configurações do conjunto, e você pode alterá-las posteriormente. Quando o Chloros libera as câmeras — `array-disconnect`, o fim de um comando transitório (`multi-capture` sem matriz, `multi-stream`, `multi-test`, `align-calibrate`, `align-apply`, `align-stream`, `index --live`), a interrupção do `lattice continuous`, o fim de um `lattice capture` de uma câmera que não está conectada — ele repõe cada um na forma como estava antes da conexão: sua fiação de disparo e linha de sincronização, PTP e configurações da câmera (a lista completa está em [Multicâmera (Sessões Transitórias)](#multi-camera-transient-sessions)).

Para usar as câmeras fora de um projeto, dois sinalizadores em qualquer um desses comandos gravam na **própria memória** da câmera — aquilo com que ela inicializa, em qualquer software:

- **`--keep-settings`** salva as configurações de cada câmera exatamente como estavam no momento em que a sessão foi encerrada, em sua memória (UserSet2, que se torna o conjunto de inicialização). As câmeras continuam operando livremente (`TriggerMode` `Off`), com a fiação da linha de sincronização tal como encontrada. Uma câmera sem UserSet2 salva em UserSet1; este computador se lembra disso, de modo que a autocorreção da matriz dChloros — que redefine um UserSet1 que se desviou da linha de base do Chloros — não interfere nela.
- **`--clear-settings`** redefine a memória de cada câmera para as configurações de fábrica: a linha de base do Chloros, aquela que a autocorreção da matriz consolida — padrões de fábrica com `TriggerMode`, `Off`, `TriggerSource` e `Line2`, exposição automática, o equilíbrio de branco calibrado de uma câmera RGB mantido — no UserSet1, que compõe o conjunto de inicialização.
- Os dois não podem ser combinados. Cada câmera recebe uma linha: `SN:…: settings saved to the camera's memory (UserSet2); it powers up with them.` ou `SN:…: camera memory reset to the factory settings.`; a resposta `camera_memory` é a mesma para cada câmera.

O aplicativo não oferece esses sinalizadores.

**Qualquer saída reverte as configurações das câmeras.**A mesma reversão “como encontrada” é executada em cada caminho de saída, não apenas nos comandos acima: `cam-disconnect` (e a câmera**X**do aplicativo); o**X** do aplicativo em uma matriz ou na linha de qualquer membro da matriz (uma matriz não pode continuar sendo executada sem um membro, portanto, toda a matriz é liberada); ao fechar o aplicativo (ele dá ao backend até 20 s para restaurar todas as câmeras, matriz e o `lattice continuous` em execução de volta antes de interrompê-lo; um comando ainda em execução nesse momento — um comando transitório abrindo suas câmeras ou aguardando seus relógios PTP, um `lattice capture` de execução única, `power`, `white-balance` ou `presets --apply` — é interrompido no próximo ponto seguro, restabelece suas câmeras primeiro, por até 8 s, e responde com HTTP 503 `SHUTTING_DOWN`; uma conexão em array ainda em execução — `lattice array-connect`, a conexão do aplicativo — para antes de sua próxima câmera e fecha as câmeras que havia aberto, por até 10 s, e responde com HTTP 503 `SHUTTING_DOWN`; e uma que chega durante o desligamento não abre nada: HTTP 503 `SHUTTING_DOWN`); e um comando CLI que é encerrado ou cujo terminal é fechado (SIGTERM / SIGHUP interrompem sua tarefa — e recoloca suas câmeras no lugar — antes de sair (`143` / `129`, como Ctrl+C). Um `lattice continuous`, `multi-stream` ou `align-stream` encerrado imediatamente (SIGKILL), que não pode interromper nada, tem sua tarefa interrompida e suas câmeras repostas pelo backend 30 s após seu último check-in. Um backend que trava ou é encerrado à força não consegue restaurar nada; um reinício retorna cada câmera à sua memória, e a próxima conexão do Chloros reconfigura qualquer câmera que encontrar armada.**Um projeto configura suas próprias câmeras.** Quando um conjunto se conecta com um projeto aberto, cada membro recebe as configurações que o projeto salvou para ele — uma exposição, ganho ou meta de AE mantida nele (considerada como sua, de modo que a exposição automática não a altera) e seu modo de balanço de branco (correção do Chloros; o balanço de branco da própria câmera permanece desativado) — independentemente do que a memória da câmera contenha.

### Smart-AE / Smart-Capture

As matrizes LATTICE executam AE contínuo em segundo plano assim que são conectadas, mas uma cena recém-focalizada leva um momento para convergir. `array-capture --smart` é a **opção prática**: ele aguarda até que a AE se estabilize em todas as câmeras do conjunto e, então, aciona a captura. Use-o ao trocar de cena no meio de uma sessão.

```bash
# Connect once, then take settled captures whenever you re-point the rig
chloros-cli lattice array-connect --serials SN1,SN2,SN3,SN4
chloros-cli lattice array-capture --smart --processing reflectance -o pose_a/
# (move the rig)
chloros-cli lattice array-capture --smart --processing reflectance -o pose_b/
```

A política de estabilização é conservadora por padrão: tempo limite de 5 s, janela de estabilidade de 1,5 s, tolerância de variação de exposição de ±5 %. Ajuste por meio do SDK (`ArrayHandle.capture_smart(settle_timeout_s=…, stability_window_s=…, exposure_tolerance_pct=…)`) se precisar de um comportamento diferente da automação.

#### Como funciona a exposição automática

- **Exposição primeiro, ganho por último.** Com o ganho em “Auto”, ele permanece em 0 dB até que a exposição atinja o valor máximo permitido.
- **Uma exposição manual mantém o ganho em “Auto”** até que você altere o ganho ou o configure para “Manual”. De volta ao modo “Auto”, o limite do ganho é o mesmo de antes.
- **Pequenas áreas claras não escurecem a cena.** Um reflexo especular de até 1 % do quadro é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5 % de um painel.
- **O limite máximo de exposição do DAQ se aplica apenas enquanto um alvo de calibração estiver em campo de visão.** Ele evita que um painel branco sofra clipping; sem nenhum painel visível, a câmera expõe para a cena.
- **A luz constante permanece estável, e a luz alterada se estabiliza em segundos.** Com luz constante, a exposição e seu limite permanecem inalterados; após uma grande mudança na luz, a imagem se estabiliza em poucos segundos. Um clipping não limita mais a exposição de forma definitiva, e uma mudança na região de interesse (ROI) restabelece a exposição.
- **O alvo de brilho permanece ao alcance.** Ele não aumenta enquanto a câmera estiver no limite de exposição e seu ganho não puder subir; assim, uma luz que retorne não causa reflexos no quadro.
- **Uma câmera cuja exposição automática deixa de responder é mantida no nível adequado à iluminação em que se encontra.** Sob sol forte, uma câmera poderia cair para sua exposição mais curta e permanecer assim com um quadro escuro, com todas as suas configurações indicando valores normais. O Chloros monitora o quadro, mantém a câmera no nível que ela mantinha para aquela luz e a libera quando a luz muda. Um sensor de luz escuro ou coberto não interfere nisso.

### Níveis de captura e exportação (padrão do `all`)

A partir desta versão, `lattice capture`, `lattice multi-capture` e `lattice array-capture` **o padrão é `--processing all`** — um arquivo salvo por tipo de exportação que se aplica a cada câmera, correspondendo ao comportamento da opção “Capturar tudo” da interface gráfica. Os níveis são:

| Nível | Saída | Aplica-se a |
| --- | --- | --- |
| `raw` | Bayer de canal único (câmeras monocromáticas: a banda única) diretamente do sensor. | Todas as câmeras. |
| `debayered` | Cor de 3 canais, salva na ordem de canais **RGB**(um TIFF com interpretação fotométrica RGB; PNG/JPEG também abertos como RGB): o deslocamento de escuro (DSNU) do pacote de calibração é removido do raw, seguido pela demosaicização sensível a bordas do OpenCV— sem LUT de linearidade, campo plano ou desmistura (câmeras monocromáticas: 1 canal em escala de cinza). Os quadros sem bayering na memória que o SDK Python retorna (`grab()`, `frame_stream()`) estão no formato**BGR**, na ordem do OpenCV; apenas o arquivo salvo está no formato RGB. | Todas as câmeras. |
| `radiance` | float32 W/m²/sr/nm por meio da cadeia radiométrica completa. | Apenas multiespectral (M3C/M3M) — **omitido para câmeras com filtro RGB**. |
| `reflectance` | uint16 ρ (`32768` = 1,0), pronto para Pix4D. | Apenas multiespectral, e **somente quando um DAQ estiver vinculado + a câmera estiver calibrada**; caso contrário, ignorado. |
| `display` | Cadeia completa de pré-visualização na GUI (CCM + WB + gama de acordo com o perfil da câmera). Salvo em uma pasta `preview/`. `lattice capture` também aceita o nome `preview` para ela; `array-capture` / `multi-capture` aceitam apenas `display`. | Todas as câmeras. |

Passe um único nível para salvar apenas esse (`--processing debayered`). Quando você solicita o `all`, os níveis que não se aplicam a uma determinada came são ignorados (e relatados), mas não geram erro — uma câmera desconectada ou não calibrada ainda recebe `raw` / `debayered` / `preview`.

#### A leitura do DAQ em uma captura é salva

Sempre que um DAQ é vinculado à câmera ou ao conjunto de câmeras, ou incluído no pool, a leitura de radiação descendente utilizada na captura é gravada em um arquivo **`.daq`** na subpasta `daq/` da pasta de capturas, com seu `.match.json` (a distância da leitura em relação à exposição) e seu `.cameras.json` (de quem é a leitura), e relatada em uma linha `DAQ:`. Isso se aplica a todos os níveis, incluindo capturas brutas ou com debayering, de modo que qualquer captura pode ser reprocessada posteriormente em radianância, refletância ou um índice. É o mesmo arquivo, com o mesmo nome e arquivos sidecar, independentemente de a captura ter vindo da GUI, da CLI ou do SDK. Sem nenhuma leitura DAQ, não há `.daq`, e um `multi-capture` sem matriz conectada (o recurso de fallback transitório) não salva nada.

| Sinalizador | Efeito |
| --- | --- |
| `--no-daq` | Não salvar a leitura em uma captura cujos níveis não a utilizem (`raw`, `debayered`). Um nível `radiance`, `reflectance` ou `display` ainda a grava, pois a reimportação precisa da leitura com a qual o produto foi fabricado. |
| `--daq-tolerance-ms MS` | Quão longe da exposição a leitura pode estar e ainda ser considerada dentro da tolerância (padrão 500). Apenas o veredicto muda (o `.match.json` do `in_tolerance` e do `tolerance_ms`, o `DlsInTolerance` da refletância), nunca a leitura utilizada. Veja [Uma leitura DAQ distante da exposição](#a-daq-reading-far-from-the-exposure). |
| `--force-daq` | A maneira antiga de solicitar o `.daq` em uma captura apenas em formato raw. Atualmente, essa é a configuração padrão, e o `lattice capture` e o `array-capture` ainda aceitam o sinalizador (`multi-capture` não aceita). `--no-daq` tem prioridade sobre ele. |

Qual DAQ: aquele vinculado à câmera ou ao conjunto de sensores (a seleção de sensor de luz da GUI, uma vinculação salva do projeto ou o `chloros-cli daq pool-bind`), ou então o mais recente que o backend tiver. Uma sequência de imagens em formato raw (`array-burst`) mantém seu próprio `.daq` para cada leitura distinta na pasta de rajadas, como antes.

**Os formatos PNG e JPEG contêm 8 bits por canal; portanto, eles utilizam apenas os arquivos `display` / `preview` e os produtos de indexação.** `--format png` ou `jpg` com um nível que não podem suportar são rejeitados antes que qualquer coisa seja capturada, indicando o nível e o formato TIFF que o suporta:

- `radiance` (float32) e `reflectance` (uint16, `32768` = 1,0) — sempre; portanto, o `--processing all` (padrão) com `--format png`/`jpg` também é rejeitado: use `--format tiff` ou `--levels display`.
- `raw` e `debayered` de uma câmera transmitindo mais de 8 bits (os quadros M3C de 12 bits / Mono12 M3M) — a CLI não consegue identificar a profundidade de bits da câmera, portanto o backend rejeita esses quadros (HTTP 400). O únicoquadro `lattice capture --processing debayered` captura um quadro de 8 bits para PNG/JPEG e não é rejeitado, assim como um `multi-capture` sem uma matriz persistente (desbayering de 8 bits).

Todas as rotas de captura respondem da mesma forma (`error_code` `FORMAT_CANNOT_HOLD_LEVEL`): `lattice capture`, `multi-capture`, `array-capture`, `project capture` / `burst` e as rotas HTTP por trás delas.

### Como é a pasta de capturas

Cada tipo de exportação é armazenado em sua **própria subpasta**dentro de `-o`; assim, uma captura em vários níveis nunca mistura os tipos. O esboço mostra a**captura padrão `all` de uma câmera pelo `lattice capture`** — vários níveis por câmera, de modo que cada arquivo traz um sufixo `_<level>`:

```
output/
├── raw/           capture_<ts>_SN<serial>-<filter>_raw.tiff
├── debayered/     capture_<ts>_SN<serial>-<filter>_debayered.tiff
├── radiance/      capture_<ts>_SN<serial>-<filter>_radiance.tiff
├── reflectance/   capture_<ts>_SN<serial>-<filter>_reflectance.tiff
├── preview/       capture_<ts>_SN<serial>-<filter>_display.tiff
├── index/         per-camera vegetation-index (LUT) render, when --index is on
├── composite/     array foreground/background live-view composite, when produced
├── clamp_masks/   radiance/…_radiance_clampmask.tif, reflectance/…, index/… — the unmix clamp
│                  masks, one subfolder per product folder, only when a band clamped
└── daq/           daq_data_<sensor id>_<ts>.daq: the downwelling reading matched to the capture
                   (+ .daq.cameras.json: whose)
```

Como os nomes dos arquivos variam:

- **Campo da câmera — número de série, seguido do filtro:** cada arquivo de captura recebe o nome da câmera que o gravou como `SN<serial>-<filter>`, onde `<filter>` é a parte do filtro do modelo da câmera: `SN261902142-F450` para uma `M3M-L87-F450`, `SN213800234-FRGN` para uma `M3C-L87-FRGN`, `SN214701292-FRGB` para uma câmera RGB. O filtro une o número de série com `-`, e não com o `_`; portanto, um nome possui os mesmos campos separados por `_` de antes (prefixo, data, hora, milissegundos, câmera, nível). Uma câmera cujo modelo não possa ser lido (um DeviceUserID apagado) recebe apenas o nome `SN<serial>`. Capturas de versões anteriores não possuem filtro no nome; o Chloros lê ambas.
- **Prefixo:** `lattice capture` nomeia os arquivos `capture_…`; `array-capture`, `multi-capture` com uma matriz conectada, e `project capture` / `burst` / `stream` de uma matriz os nomeia como `sync_<ts>_SN<serial>-<filter>…`.
- **Sufixo de nível apenas com vários níveis.**Uma câmera que salva**vários**níveis em uma captura (`all` ou `--levels a,b`) adiciona `_<level>`. Uma câmera que salva**um** nível mantém o nome simples, sem sufixo — `--processing reflectance` grava `reflectance/sync_<ts>_SN<serial>-<filter>.tiff` — para que os scripts criados para capturas de nível único continuem funcionando. O nível ainda é indicado pela pasta.
- **As máscaras de limitação seguem o nome do produto:** `<product name>_clampmask.tif`. Uma captura de refletância de nível único grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`; uma captura `all` grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`.
- **A extensão** segue `--format`: o padrão `tiff` resulta em `.tiff`, `--format tif` resulta em `.tif` (as máscaras de restrição são sempre `.tif`).
- **Uma exceção:** `lattice capture --processing debayered` (ou `--levels debayered`) recebe um únicoe grava `capture_<ts>_SN<serial>-<filter>.tiff` diretamente em `-o` — sem subpasta.
- **Fluxos e rajadas** contêm o mesmo campo: `lattice continuous` grava `frame_<seq>_SN<serial>-<filter>.tiff`, `lattice multi-stream`, `stream_<seq>_SN<serial>-<filter>_<frame id>.tiff` e uma sequência bruta `<burst>_f<frame>_SN<serial>-<filter>_raw.tiff`.

`<ts>` é o carimbo de data/hora da captura e `<serial>` é o número de série da câmera; portanto, um grupo sincronizado compartilha um
carimbo de data/hora entre as câmeras. **Observe a única assimetria:** o nível `display` é armazenado em uma pasta
chamada `preview/`, enquanto os arquivos multinível mantêm `_display` no nome — a pasta e o sufixo diferem
apenas para esse nível. Níveis desconhecidos recorrem a uma pasta com seu próprio nome e, se a subpasta
não puder ser criada, o arquivo é gravado na raiz de saída, em vez de ser perdido.

**Reprocessamento de uma pasta de capturas:**aponte `chloros-cli process` para a**raiz de capturas**
(`output/`). O `process` normalmente importa apenas a pasta que você nomear, mas quando essa pasta não contém
imagens e possui subpastas, ele percorre automaticamente a hierarquia — de modo que as subpastas de nível e a
subpasta `daq/` são todas selecionadas de uma só vez. Cada nível de uma captura é importado como uma única imagem, com
os outros níveis disponíveis como modos, em vez de uma imagem por nível.

Nomear uma **subpasta de nível** diretamente (por exemplo, `output/raw/`) também funciona. Ao fazer isso, a pasta `daq/`
fica de fora; portanto, copie ou indique a leitura do DAQ junto quando você estiverderivar um produto radiométrico
a partir de `raw/` — caso contrário, a correspondência do carimbo de data/hora não terá nada com que se alinhar.

**O processamento sempre começa a partir de `raw`.** Dentro de cada captura, o quadro bruto é a fonte do pipeline;
`debayered`, `radiance`, `reflectance` e `preview` surgem como modos visualizáveis, mas nunca são realimentados
de volta ao pipeline. O reprocessamento de um produto derivado reaplicaria a vinheta, o CCM e
os cálculos de radiação que já estão incorporados em seus pixels; por isso, o Chloros, em vez de
fazer um processamento duplo, recusa essa operação. Duas consequências que vale a pena conhecer:

- As renderizações `index/` e `composite/` **nunca** processadas. São saídas, não capturas —
  uma renderização de LUT de NDVI não possui interpretação significativa de radiação.
- Uma pasta de capturas exportada **sem** o `raw` (por exemplo, `array-capture --processing reflectance`) não possui
  nenhuma fonte legítima no pipeline. Essas capturas são importadas e exibidas normalmente, mas o `process` as ignora
  e indica isso:

  ```
  [IMPORT-LEVEL] Skipping 4 already-processed file(s) with no raw source: capture_…_reflectance.tif
  [IMPORT-LEVEL] Processing starts from raw. Re-capture with --processing raw, or force an entry
                 point with --input-level.
  ```

  Se você realmente precisar passar um produto derivado — uma sessão de hub capturada com
  `demosaic` ativado, ou uma pasta legada — o `--input-level {raw,debayered,processed}` força o ponto de entrada
  e anula a omissão. Esse sinalizador é a saída de emergência deliberada; o `auto` (o padrão)
  nunca processa uma captura que não tenha dados brutos.

**Tags de calibração: somente no `raw`.** O arquivo `raw` de uma captura contém o pacote de calibração da câmera: a carga útil `CalibrationJSON` e as tags compatíveis com MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que descrevem os valores brutos do sensor. Todos os outros arquivos de nível (`debayered`, `display`, `radiance`, `reflectance`, a renderização `index/`) e cada quadro `lattice continuous` / `multi-stream` contém a identidade da câmera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e o próprio registro do quadro, `XMP-mapir:FrameJSON` (exposição, ganho, temperatura, formato de pixel, hora da captura, correspondência DAQ), já que cada produto `process` possui essa identidade. O `process` localiza a calibração desse arquivo por meio de seu número de série e o `CalBundleSha256` no cache de calibração do Chloros, baixando o pacote uma vez caso o computador não o tiver.

#### Arquivos criados a partir de várias câmeras

Um arquivo criado a partir das faixas de mais de uma câmera — uma pilha multibanda (`align-apply`, `align-stream`, `index --live --save-multiband`), um índice ao vivo gerado a partir de várias câmeras (`index.tif`, `index_LUT.png`), uma imagem de captura em matriz `composite/` — nomeia todas as câmeras que contribuíram para ela:

- **Listas por câmera, em ordem de banda** (a composição: camada de fundo, seguida da camada de primeiro plano; uma camada de índice, com as câmeras das quais suas bandas provêm): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (o pacote de calibração pelo qual as faixas de cada câmera passaram; caso contrário, — uma pilha `align-apply` / `align-stream`, cujas bandas não passam por nenhum — o pacote da câmera que o Chloros armazenou em cache, aquele cujo nome aparece nos arquivos de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, sendo que o item *n* de cada uma descreve a mesma câmera; `-`, onde uma câmera não possui valor (sem pacote de calibração, uma lente desconhecida). Cada câmera é **um** item: quando seu pacote possui vários valores para um campo — os certificados de lâmpadas halógenas de um pacote —, eles são unidos com `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca com uma vírgula, portanto, uma lista apresenta um item por câmera, independentemente de como for impressa.
- **EXIF `SerialNumber`** contém os mesmos números de série, separados por vírgulas (`214000535,214000551`).
- Ele **não** contém identidade de uma única câmera (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): esses identificam uma câmera e seu conjunto, e `process` determinaria a calibração do arquivo a partir deles. Um índice calculado a partir das faixas de várias câmeras (`index.tif`, `index_LUT.png`) contém EXIF `Model` `LATTICE` (o conjunto), e não o modelo de uma única câmera `LATT-…`.

Um arquivo com as faixas de **uma** câmera — um TIFF por banda (`--per-band`), um quadro `align-apply --per-camera`, uma pilha ou índice ao vivo de uma única câmera — contém as próprias tags de identidade dessa câmera, assim como qualquer outro arquivo de captura. A pilha do `align-apply`, os arquivos por banda e por câmera, e cada pilha do `align-stream` e seus arquivos por banda contêm a hora de captura do grupo (a exposição da câmera de referência, UTC), e um quadro `--per-camera` de uma câmera monocromática é um plano, em `--bit-depth`, assim como a pilha.**Orientação da câmera (inversão / rotação).** A configuração de orientação de uma câmera (as opções “Espelho H” /
Mirror V / rotação) da interface gráfica de usuário (GUI) determina o que é salvo na captura:

- **`raw` e `debayered` são salvos exatamente como o sensor os leu**, independentemente da configuração. Um
  mosaico de Bayer girado ou invertido seria um padrão de filtro de cor diferente, e os mapas de calibração
  (desvio escuro, campo plano) estão na própria grade do sensor.
- **`preview`/`display`, `radiance`, `reflectance` e a renderização `index/` estão orientados** — girados
  uma vez, após todos os cálculos de calibração, de modo que se abrem da mesma forma que a câmera foi enquadrada. Seus
  `clamp_masks/…_clampmask.tif` estão na mesma grade.
- **Cada arquivo de captura registra a configuração** no XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, no sentido horário, após
  as inversões) e `ChlorosOrientationApplied` — `false` em imagens brutas e sem debayering (pixels nativos),
  `true` nos produtos orientados. `ChlorosProcessing` lista as operações aplicadas aos
  pixels desse arquivo (`none` quando nativo). Isso inclui todos os quadros que o `lattice continuous` transmite (a
  própria configuração da câmera, não aplicada) e todos os quadros do `lattice multi-stream` (a identidade: seus
  quadros componentes são nativos, como os de uma captura em matriz). `lattice align-stream` / `align-apply`
  as pilhas registram a configuração da câmera mestre, não aplicada: suas bandas são distorcidas para o quadro da câmera mestre,
  mas não são giradas.
- **`process` aplica a orientação registrada** ao processar um arquivo bruto: deslocamento de escuro, campo plano
  , demosaico e desmistura nas coordenadas do sensor primeiro; em seguida, a distorção de alinhamento da matriz; e, por último, a
  orientação por último — de modo que os produtos de pré-visualização, radiação, refletância e índice saem exatamente como
  os da captura ao vivo e registram o `ChlorosOrientationApplied=true`. Os produtos de fallback não calibrados
  (`Sensor_Response_Images`, `Vignette_Corrected_Images`) são orientados da mesma forma, após
  o campo plano. O produto sem debayering permanece em formato nativo.
- **O visualizador de imagens Chloros mostra um arquivo bruto ou sem debayering girado de acordo com seu registro** (apenas para exibição —
  o arquivo não é alterado), de modo que se parece com seus produtos; a miniatura, a visualização em tamanho real, a
  leitura do cursor e a área de teste de Índice/LUT utilizam todos a mesma rotação. Um arquivo sem registro e
  um produto (já orientado) são exibidos conforme armazenados. Uma exportação para a área de teste registra a orientação que seus
  pixels carregam (`ChlorosOrientationApplied=true`; a identidade para uma camada sem registro).
- **Capturas sem o registro são processadas como nativas** (comportamento inalterado). Antes da versão 1.3.0, uma
  captura com uma única câmera com inversão ou rotação definida girava por conta própria seus arquivos raw e debayered;
  o Chloros não consegue distinguir um arquivo raw já-rotacionado de um arquivo nativo e não faz suposições, portanto,
  processa-o conforme armazenado. Os mapas de calibração estão na própria grade do sensor; portanto, para tal
  captura, use uma feita com a versão 1.3.0 quando a calibração for relevante.
- **As matrizes não giram nenhum arquivo membro por padrão** (a distorção de alinhamento e o XMP `Alignment*` estão nos
  próprios quadros das câmeras, sem rotação) e registram a identidade. O `array-capture --orient` gira os
  produtos de cada membro, após a transformação de alinhamento: de acordo com a orientação do **mestre** quando foram
  transformados para seu quadro (para que os membros permaneçam co-registrados), ou de acordo com a orientação de suas próprias câmeras. Cada
  arquivo registra a orientação do quadro em que seus próprios pixels se encontram — um arquivo bruto, da própria câmera —
  e um arquivo bruto que contém a transformação de alinhamento também registra `ChlorosAlignedFlipHorizontal` /
  `ChlorosAlignedFlipVertical` / `ChlorosAlignedRotation`: a do mestre, para um produto alinhado
  para o quadro da câmera principal. `process` gira cada produto de acordo com o que pertence aos seus pixels:
  o alinhado quando distorce o raw (`--array-alignment`, o padrão — os produtos da captura alinhada em tempo real
  ), o da própria câmera quando isso não ocorre (`--no-array-alignment` — os da captura ao vivo
  não alinhada). A mesma opção existe para `array-burst`, `project capture` / `burst` /
  `stream`, receitas e o SDK.

### Capturas ignoradas em matrizes com filtros mistos

Quando você mistura câmeras RGB e multiespectrais em uma matriz, o `array-capture --processing radiance` (ou `reflectance`) salva os quadros multiespectrais e **ignora** as câmeras RGB — a radiação por pixel Bayer não é significativa para um sensor de banda larga. A CLI exibe explicitamente cada arquivo salvo (com seu nível de exportação), cada `.daq` gravado e cada omissão, de modo que a contagem de arquivos não é surpreendente:

```
  Saved: …/output/reflectance/sync_…_SN213800234-FRGN.tiff [reflectance] (SN:213800234, fid:1)
  Saved: …/output/reflectance/sync_…_SN214000533-FOCN.tiff [reflectance] (SN:214000533, fid:1)
  Saved: …/output/reflectance/sync_…_SN214701288-FNGB.tiff [reflectance] (SN:214701288, fid:1)
  DAQ:   …/output/daq/daq_data_daq-e-54b5e0_….daq
  Skipped: SN:214701292 (reflectance-not-applicable-to-rgb-cam filter=RGB)

  3 synchronized frames captured. (1 skipped)
```

Os tokens de motivo de omissão seguem o padrão `<level>-not-applicable-to-rgb-cam`. Uma câmera cujo pacote de calibração não esteja carregado omite `radiance` / `reflectance` com `<level>-needs-calibration (connect the camera so its cal pack loads)`, e, na ausência de leitura de refletância utilizável do DAQ, pula com `reflectance-skipped-no-dls`. A refletância também pode ser pulada com `reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, e com `dls-uncalibrated-band-<nm>` quando a banda se encontra, em sua maior parte, fora da faixa radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm) — entre os SKUs disponíveis para envio, apenas o F988, cujo fluxo de trabalho compatível é o do painel de refletância.

Use o `--processing debayered` (ou `display`) para incluir todas as câmeras, independentemente do tipo de filtro, ou o padrão `all` para obter todos os níveis aplicáveis por câmera de uma só vez.

### Uma leitura do DAQ distante da exposição

Cada `.daq` que uma captura salva é a leitura do DAQ correspondente àquela exposição, e o `.match.json` ao lado registra a distância entre os dois (`gap_ms`) e se isso está dentro da tolerância (`in_tolerance`, 500 ms, a menos que a captura tenha definido `--daq-tolerance-ms`). Uma leitura fora desse intervalo ainda é utilizada, de modo que nenhum quadro é perdido, e ela é identificada: o arquivo XMP de refletância contém `DlsInTolerance=False` e `DlsMatchGapS`. `lattice capture`, `multi-capture` e `array-capture` também a imprimem sob o `.daq` a que se refere, com a falha de fluxo do DAQfalha de fluxo do DAQ quando se explica:

**Como uma leitura é associada a uma exposição.** Uma captura aguarda a primeira leitura do DAQ após a exposição, para que possa combinar as leituras de ambos os lados dela (`method` `interpolated`). Um DAQ-E envia uma leitura a cada 0,4 s à luz do dia e a cada 1-2 s em ambientes internos; portanto, a espera soma cerca de um intervalo e meio desses ao comando, e não à exposição. Não se aguarda um DAQ que tenha parado de enviar. Uma leitura do DAQ-E é a média da luz no intervalo desde sua leitura anterior; portanto, é colocada no meio desse intervalo: `gap_ms` é medido a partir da exposição até esse ponto médio, e `bracket_stamps_ns` fornece os próprios carimbos de data/hora das leituras. `array-capture --continuous` aguarda sem retardar a captura: seus arquivos brutos são gravados imediatamente e seus `.daq` assim que a leitura chega. Com o `--interval`, cada passagem aguarda como se fosse uma única captura.

```
  DAQ:   …/output/daq/daq_data_daq-e-54b774_….daq
         DAQ reading 114.9 s from the exposure (tolerance 500 ms): reflectance uses it, labelled DlsInTolerance=False
         DAQ stream fault: no frame accepted from daq-e-54b774.local in 25 s, after 1932 good frame(s). …
```

Quando nenhum arquivo de refletância foi gravado a partir dessa leitura, a primeira linha termina com `saved anyway; its .match.json says in_tolerance: false`.

A captura ainda sai com `0`: todos os arquivos foram gravados e indicam o que contêm. Um DAQ cujo fluxo tenha travado continua fornecendo sua última leitura; portanto, verifique-o com `chloros-cli daq pool-list` antes de confiar na refletância dessa captura. O backend reconecta um DAQ-E em pool que parou de enviar dados (consulte [Um DAQ-E que para de fornecer dados](#a-daq-e-that-stops-delivering)). Uma leitura fora da tolerância exibe apenas sua linha `DAQ:`, a menos que seja uma leitura mantida (abaixo).

**Uma leitura mantida após o DAQ ter saído de operação.** Quando nenhum DAQ pode fornecer uma leitura para uma exposição (o DAQ vinculado está indisponível ou nenhum DAQ em pool possui uma, por exemplo, após `daq pool-disconnect`), a captura reutiliza a última leitura que o Chloros possuía, e seu `.match.json` descreve essa leitura em relação a essa exposição: `method`, `retained`, `gap_ms` desta exposição para as amostras das quais a leitura foi feita, `in_tolerance` daquele intervalo e `retained_from_epoch_ns`, a exposição à qual a leitura foi inicialmente associada. Uma captura feita muito tempo depois que o DAQ foi desligado, portanto, imprime as linhas acima e, abaixo delas mais uma, que é impressa mesmo enquanto a leitura ainda está dentro da tolerância:

```
         This is the DAQ's last reading: the DAQ is no longer reporting, so each capture reuses it until a new one arrives
```

Seu `.daq` ainda identifica o DAQ que fez a leitura, com a calibração e a tampa sob as quais essa leitura foi feita, de modo que uma reimportação sabe qual DAQ, calibração e tampa a produziram.

### Arquivos que não conseguem ser gravados

Uma câmera cujo arquivo não pode ser gravado — um disco cheio, um erro de permissão — não impede que os arquivos das outras câmeras sejam salvos. Cada comando de captura (`lattice capture`, `multi-capture`, `array-capture`; `project capture`, `burst`, `stream`; e os `capture`, `burst` e `stream` de uma receita) imprimem cada arquivo com falha no stderr, em seguida **encerra o `1`**, mesmo que o restante tenha sido salvo:

```
  Failed: SN:214701292 [reflectance] OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

Todos os arquivos que uma captura deve gerar estão incluídos, incluindo o único`lattice capture --levels debayered` com debayering de quadro único. Uma leitura `.daq` cujo arquivo sidecar de proveniência `.match.json` não pôde ser gravado é listada como `[match]` (o próprio `.daq` foi salvo).

Em um disco cheio, uma captura repetida (`array-capture --continuous` / `--interval`, `project burst` / `stream`, uma receita `burst` / `stream`) **interrompe-se nessa passagem** em vez de não gravar nada na mesma unidade pelo restante da execução. Qualquer outra falha é relatada a cada passagem em que ocorre, e a execução continua até o fim, encerrando-se então em `1`. `array-capture --continuous` (a captura contínua do backend) não lista nenhum arquivo, incluindo os que falharam: ele os conta nas linhas de progresso e resumo (`N failed`, `N file(s) failed to write.`) e encerra o `1` — consulte [Modo contínuo](#continuous-mode---continuous).

Os gravadores — `array-burst` e `array-record` — relatam suas próprias falhas. O `array-burst` interrompe-se automaticamente quando a unidade fica cheia, ou pouco antes disso. Ambos continuam operando mesmo após qualquer outra falha de gravação. Quando qualquer um deles relata um erro, este é exibido após os contadores, e o comando **sai do `1`**:

```
  Recorder error: disk-full: OSError: [Errno 28] No space left on device
  Disk full: the drive the backend captures to has no space left. Free space on it (or capture to another drive) before capturing again.
```

`Disk low:` no lugar de `Disk full:` significa que a sequência foi interrompida antes de encher a unidade. Os quadros gravados antes da interrupção são mantidos. O `array-burst --build` ignora a compilação quando o espaço se esgota, pois os vídeos iriam para a mesma unidade. Libere espaço primeiro e, em seguida, execute o `array-build-video --burst-dir …`. Após qualquer outro erro, a compilação continua sendo executada.

Um `array-record` que não gravou nenhum quadro também é considerado uma gravação com falha (`Recorder error: no-frames: the combined view published no frame to record …`, saia do `1`) — consulte [`array-record`](#array-record--combined-index-videogif-monitoring-grade).

---

## Modos de captura, gravadores e reprocessamento offline

Todos eles operam em uma **matriz persistente** (execute primeiro o `array-connect`). Eles espelham o painel de captura da GUI.

### Modos do `array-capture`

O `array-capture` é um único comando com quatro modos de obturador, além de um conjunto de opções de exportação:

| Modo | Sinalizador | Comportamento |
| --- | --- | --- |
| **Único** *(padrão)* | (nenhum) | Um grupo de captura sincronizado e, em seguida, encerramento. |
| **Contínuo** | `--continuous` | Capturas consecutivas até que o `Ctrl+C`, o `--count N` capture, `--duration S` ou até que a unidade esteja cheia. Executa a captura contínua do próprio backend (o modo Contínuo da GUI) — consulte [Modo Contínuo](#continuous-mode---continuous) abaixo. |
| **Intervalo** | `--interval S` | Uma passagem a cada `S` segundos (medidos a partir do início de cada passagem), com os mesmos limites. |
| **Mais rápido** | `--fastest` | Brutosomente + a leitura DAQ atribuída + o composto de índice combinado; ignora os cálculos de radianância/refletância/exibição para que o quadro seja renderizado rapidamente. Implica `--processing raw --force-daq`. Reprocesse o `.daq` salvo em produtos calibrados posteriormente. |

Opções de exportação (combináveis com qualquer modo; todas compartilham a interface gráfica/ponto de extremidade SDK):

| Sinalizador | Efeito |
| --- | --- |
| `--array-id ID` | A matriz a ser capturada (omita quando apenas uma estiver conectada). |
| `-o, --output DIR` | Pasta de saída (padrão: `output`, na pasta em que o comando é executado). |
| `--format {tiff,tif,png,jpg,jpeg}` | Formato do arquivo (padrão: `tiff`). PNG/JPEG aceitam apenas `display` e a renderização do índice; consulte [Níveis de exportação de captura](#capture-export-levels-the-all-default). |
| `--processing LEVEL` | Nível único de exportação (`raw`, `debayered`, `radiance`, `reflectance`, `display`), ou `all` (padrão). |
| `--levels L1,L2,…` | Subconjunto explícito de tipos de exportação (por exemplo, `raw,radiance,reflectance`); **substitui `--processing`**. |
| `--aligned` / `--no-aligned` | **Padrão: alinhado, assim como a captura do mesmo array pelo aplicativo Chloros**(sua configuração de captura**Alinhada** por array, ativada a menos que você a desative). A exportação não bruta de cada membro (`debayered`, `display`, `radiance`, `reflectance`, a renderização do índice) é distorcida para o quadro da câmera principal e recortada para a sobreposição comum dos membros, de modo que os arquivos ficam alinhados em pixels e apresentam o mesmo tamanho, recorte e tags `Alignment*` que os do aplicativo — menores que o sensor (ou da ROI) pelos deslocamentos dos membros; sob uma ROI, a transformação é reajustada com base na janela. `XMP-mapir:AlignmentJSON` indica `"applied": true` e fornece o `crop_roi`. O arquivo bruto permanece sem distorção no tamanho nativo, mas carrega a transformação nos metadados (`"applied": false`). `--no-aligned` salva todos os arquivos no tamanho nativo, sem distorção. `--aligned` é a configuração padrão explicitada. Independentemente de a distorção ser aplicada ou não, cada arquivo membro registra a transformação `Alignment*` de sua câmera assim que a matriz tiver um perfil de alinhamento — exceto uma câmera que o perfil não alinha (omitida ou forçada à identidade por uma resolução malsucedida) e a câmera de referência quando nenhuma outra câmera está alinhada: essas não possuem tags `Alignment*`. Sem perfil na matriz, a captura é salva desalinhada, e a CLI exibe o motivo em uma linha `Alignment:`, assim como faz em todos os outros casos em que a distorção não pôde ser aplicada (membros com resoluções diferentes, uma matriz não sincronizada por cabo, um quadro no qual o perfil não pode ser mapeado). O alinhamento automático da matriz cria o perfil assim que a exposição se estabiliza; `align-calibrate` cria um sob demanda. |
| `--orient` | Transforme os **produtos**de cada membro — `display`, `radiance`, `reflectance`, a renderização `index/` e o `composite/` — pela orientação da câmera, após a distorção de alinhamento: um produto distorcido para o quadro do mestre (padrão) pela**orientação do**quadro mestre, de modo que os membros permaneçam co-registrados; um não alinhado (`--no-aligned`, ou uma captura à qual a distorção não pôde ser aplicada) pela orientação de sua própria câmera.**Padrão: desativado — todos os arquivos membros permanecem nativos**, como sempre acontece com as capturas em matriz. `raw` e `debayered` permanecem em formato nativo de qualquer maneira. Cada arquivo registra a orientação que seus produtos possuem (consulte [Orientação da câmera](#como-é-uma-pasta-de-capturas)); o XMP do `Alignment*` mantém a transformação do quadro nativo, aplicada antes da orientação, de modo que o `process` (distorção, seguida de orientação) reproduz o produto ao vivo. |
| `--index` / `--no-index` | Salvar / ignorar a sobreposição do índice de vegetação por câmera, quando configurada. Padrão: renderizá-la. |
| `--no-daq` | Ignora o arquivo sidecar `.daq` quando nenhum nível selecionado utiliza o DAQ (uma captura apenas em formato bruto). Por padrão, a leitura é salva, para que os quadros possam ser reprocessados em refletância/índice off-line. `--force-daq` ainda é aceito (o padrão). |
| `--daq-tolerance-ms MS` | A tolerância de correspondência do DAQ para essas capturas (padrão: 500 ms). |
| `--smart` | Aguardar que o AE se estabilize em todas as câmeras antes do acionamento (consulte [Smart-AE / Smart-Capture](#smart-ae--smart-capture)). |
| `--compression {deflate,none}` | Compressão de pixels TIFF. `none` (padrão desde a versão 1.3.0; a versão 1.2.0 gravava `deflate`) = sem compressão — os pixels do sensor exatamente como são lidos, ~6,3 MB por quadro em resolução total e a taxa sustentada mais rápida; `deflate` = zlib L1 sem perdas + preditor horizontal, ~4,1 MB por quadro, mais lento. Ambos são sem perdas e são lidos de forma idêntica na importação. |

> **TIFF de gravação única + o modelo de taxa sustentada.**Cada arquivo é gravado**uma vez**: pixels, o pacote XMP, IFD0 Marca/Modelo/Número de Série da Câmera e as tags EXIF `ExposureTime` / `SerialNumber` em uma única passagem, sem que nada seja reescrito posteriormente. `--continuous` captura grupos sincronizados consecutivos — cada captura recebe o grupo após o anterior, incluindo um publicado enquanto essa captura ainda estava sendo salva —, de modo que opera na taxa da matriz. O DEFLATE mantém o GIL do Python, portanto, as gravações compactadas**não**são paralelizadas entre as threads de gravação de cada câmera; a captura sustentada em resolução total na taxa do sensor requer gravações não compactadas (o padrão)**e** um disco da classe NVMe (~600 MB/s sustentados para um conjunto de 9 câmeras a ~10 fps). O mesmo controle é exposto como `compression` em `POST /api/camera/array/capture`.

```bash
# Interval timelapse: one reflectance pass every 10 s for 5 minutes
chloros-cli lattice array-capture --interval 10 --duration 300 \
  --processing reflectance -o timelapse/

# Fastest grab for a moving rig — raw + .daq now, calibrate later
chloros-cli lattice array-capture --fastest -o flightline/

# Co-registered multi-band export (aligned is the default; drop the index overlay)
chloros-cli lattice array-capture --processing reflectance --no-index -o out/

# Each member at its native size, unwarped
chloros-cli lattice array-capture --processing reflectance --no-aligned -o native/

# Continuous raw capture for 60 s, as fast as the array publishes
chloros-cli lattice array-capture --continuous --duration 60 --fastest -o flightline/
```

#### Modo contínuo (`--continuous`)

`array-capture --continuous` (sem `--interval` ou `--smart`) transfere a execução para a **própria captura contínua do backend** — a sessão em que o modo Contínuo da GUI é executado. O backend salva cada grupo sincronizado por conta própria, com duas capturas em andamento e sem solicitação da CLI entre os grupos, de modo que acompanha um array rápido, no qual uma solicitação por captura fica para trás (medido em 73–77 capturas/s em um array de 9 câmeras com uma ROI de 512 pixels). Cada opção de exportação acima se aplica a cada captura.

- **Limites:** `--count N` para após N capturas (uma captura = um grupo sincronizado: os arquivos de cada membro); o `--duration S` após S segundos. Sem nenhuma dessas opções, ele continua até o `Ctrl+C`, o disco ficar cheio, ocorrer um erro ou a matriz ser desativada. O `Ctrl+C` encerra a sessão do backend, em seguida, exibe o resumo.
- **Progresso** a cada 2 s, com `, N failed` e `, N groups skipped` anexados quando diferentes de zero (um grupo ignorado é aquele que a matriz publicou, mas para o qual não houve captura):

  ```
     12.0s  880 captures (73.3/s), 7920 files
  ```

- **Sem listagem por arquivo.** A execução termina com uma linha de resumo — capturas, taxa, arquivos e a pasta de saída — seguida por `N file(s) failed to write.` / `N group(s) skipped.` quando o valor for diferente de zero:

  ```
  Done — 880 captures (73.3/s), 7920 files in /home/user/flightline.
  ```

- **Interrompe** quando o disco fica cheio com `Continuous capture stopped: the drive is full.` e em caso de erro ou quando a matriz é desativada com `Continuous capture stopped: <reason>.` (ambos no stderr).
- **Código de saída:** `0` quando as capturas foram salvas e todos os arquivos foram gravados, incluindo o `Ctrl+C`; `1` caso haja qualquer arquivo cuja gravação tenha falhado, uma unidade cheia, um erro, a matriz sendo desativada ou nenhuma captura.
- **Microsoft Defender:** quando o Defender utiliza um núcleo da CPU ou mais para verificar os arquivos que a captura grava, a execução exibe uma vez, no stderr: `Note: Microsoft Defender is using N.N CPU cores scanning the capture files, CPU the capture needs. Re-run the Chloros installer with "Exclude the Chloros backend from Microsoft Defender scanning" checked.` Consulte as [Configurações do sistema](#windows-exe).
- Uma captura contínua por matriz por vez; uma segunda é recusada com o código `a continuous capture is already running for this array`. Por HTTP, a sessão é `POST /api/camera/array/continuous/start` (o corpo `/api/camera/array/capture` mais `max_captures` / `duration_s`), `GET /api/camera/array/continuous/status?array_id=ID` e `POST /api/camera/array/continuous/stop`.

**Intervalo**(`--interval S`, com ou sem `--continuous`) e**`--continuous --smart`** mantêm o ciclo por solicitação: uma solicitação de captura por passagem — em cada momento programado para o Intervalo, após a estabilização do AE para o `--smart` — com os arquivos de cada passagem listados como no modo Único, uma linha `Done — N capture pass(es).` no final e `--count N` contando as passagens.

### `array-record` — vídeo/GIF com índice combinado (nível de monitoramento)

Grava a **visualização ao vivo do índice combinado** da matriz em um `.avi` (e, opcionalmente, em um `.gif`). Com a visualização combinada da GUI aberta, ele grava essa visualização conforme exibida. Se nenhuma estiver aberta, o backend renderiza a própria visualização: o índice que você passa com `--expr` ou, por padrão, o NDVI da matriz, assim que a matriz estiver alinhada. O NDVI padrão utiliza as bandas vermelha e NIR mais próximas de 660 nm e 850 nm, assim como a visualização ao vivo. Uma visualização da GUI aberta durante a gravação é gravada enquanto permanecer aberta. Ela verifica o progresso a cada 2 s e interrompe em `--duration`, `Ctrl+C`, ou quando o gravador encerra automaticamente.

Uma gravação que não grava nenhum quadro falha: o gravador reporta `no-frames: …` (a matriz ainda pode estar se alinhando ou ajustando sua exposição) e o comando **sai com o código `1`**.

```bash
# 30-second combined-index clip at 10 fps, plus a GIF (no GUI needed)
chloros-cli lattice array-record --duration 30 --fps 10 --gif -o monitoring/

# Record a chosen index instead of the default NDVI
chloros-cli lattice array-record --duration 30 \
  --expr "(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)" -o monitoring/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino (omita se houver apenas uma conectada). |
| `-o, --output DIR` | `output` | Diretório de saída. Um caminho relativo está na pasta de onde você executar o comando. |
| `--fps F` | `10` | Taxa de quadros da gravação. |
| `--duration S` | até Ctrl+C | Parada automática após `S` segundos. |
| `--gif` | desativado | Grave também um GIF animado. |
| `--gif-only` | desativado | Gravar apenas um GIF (sem `.avi`). |
| `--expr EXPR` | o NDVI da matriz | Expressão de índice a ser registrada quando nenhuma visualização da GUI estiver aberta, nos nomes das bandas da matriz (por exemplo, `'(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)'`; consulte [nomes de bandas](#index--vegetation-maths)). Um nome de cor curto (`NIR`, `Red`) funciona apenas quando uma banda da matriz o possui. Uma visualização da GUI que esteja aberta é registrada conforme exibida. |

### `array-burst` — sequência de fotos raw-Bayer em alta taxa de quadros por segundo (nível de análise)

Lê diretamente o buffer do grupo sincronizado do loop de captura — **sem necessidade de cadeia de calibração, exiftool ou visualização ao vivo** — portanto, opera na taxa máxima de captura da câmera. Grava quadros raw + um manifesto por quadro + um `.daq` por leitura DLS distinta sob `<output>/bursts/<base>/`. Reprocesse offline (próximo comando) ou passe `--build` para fazer isso imediatamente ao parar.

```bash
# 5-second raw burst, then build the combined index video in one shot
chloros-cli lattice array-burst --duration 5 --build \
  --products combined:index --fps 10 -o capture/
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--array-id ID` | apenas matriz | Matriz de destino. |
| `-o, --output DIR` | `output` | Diretório de saída (os dados em rajada são armazenados em `<DIR>/bursts/<base>/`). |
| `--duration S` | até Ctrl+C | Parada automática após `S` segundos. |
| `--max-frames N` | ilimitado | Parada automáticaapós `N` quadros brutos. |
| `--orient` | desativado | Os produtos gerados por uma compilação a partir dessa sequência são girados de acordo com a orientação da câmera — os produtos de cada câmera são girados de acordo com a orientação de sua própria câmera, o índice combinado pela orientação da câmera mestre — como faz o `array-capture --orient`. Os quadros brutos permanecem em formato nativo; o `manifest.json` da sequência registra a orientação e a opção de cada membro. |
| `--build` | desativado | Após a parada, reprocesse imediatamente a sequência (igual ao `array-build-video`). |
| `--products …` | `combined:index` | Com `--build`: quais vídeos criar (veja abaixo). |
| `--fps F` | `10` | Com `--build`: fps do vídeo de saída. |
| `--save-tiffs` | desativado | Com `--build`: também salva arquivos TIFF calibrados por quadro (cada um com sua [máscara de limitação de desmistura](#unmix-clamp-mask-which-pixels-a-banda-foi-limitada) quando um pixel for limitado; o resultado da compilação os lista como `clamp_masks`). |
| `--gif` | desativado | Com `--build`: também grava(m) GIF(s) animado(s). |

### `array-build-video` — reprocessa offline uma sequência salva

Sincroniza temporalmente cada quadro bruto com a leitura `.daq` salva mais próxima e o submete à **mesma cadeia de radianância / refletância / índice que o pipeline de importação**, gerando um ou mais vídeos.

`--products` é uma lista separada por vírgulas de itens `kind:level`, onde `kind` ∈ `per_cam` | `combined` e `level` ∈ `radiance` | `reflectance` | `index`. Um `level` sem especificação (sem `kind:`) assume por padrão o valor de `per_cam`. O valor padrão é `combined:index`.

```bash
# Per-cam reflectance video for every member + one combined NDVI video
chloros-cli lattice array-build-video \
  --burst-dir "capture/bursts/2026-06-24_141500" \
  --products per_cam:reflectance,combined:index \
  --fps 10 --save-tiffs
```

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--burst-dir DIR` | (obrigatório) | Caminho para a pasta de burst (`…/bursts/<base>/`). |
| `--products …` | `combined:index` | Lista `kind:level`, conforme acima. |
| `--fps F` | `10` | FPS do vídeo de saída. |
| `--save-tiffs` | desativado | Também salvar arquivos TIFF calibrados por quadro junto com o(s) vídeo(s). |
| `--gif` | desativado | Também gravar GIFs animados. |

**Um vídeo não contém máscara de limitação** (um fluxo de imagens, 8 bits). Em vez disso, o resultado da compilação contém `clamp_fractions`: para cada produto de vídeo, a parcela de pixels de cada quadro que a separação de cores limitou na faixa contida em seu canal R/G/B — o `Chloros:Band{R,G,B}UnmixClampFraction` com o qual o produto de radiação/refletância desse quadro seria marcado (para o índice combinado, de cada câmera participante). A execução imprime o pior quadro por produto em uma linha `Clamp:`. `--save-tiffs` adiciona as máscaras por pixel para os TIFFs, em `clamp_masks/<level>/` ao lado da pasta `<level>/` dos TIFFs.**Uma compilação interrompida não deixa nenhum vídeo corrompido.** Cada vídeo é gravado como `<name>.partial.avi` e recebe seu nome definitivo somente quando estiver concluído; portanto, um vídeo mais antigo com esse nome permanece até que seu substituto esteja completo. Execute a compilação novamente para substituir um `.partial.avi` que tenha ficado para trás.

A execução lista todos os vídeos gravados e, em seguida, qualquer produto `Skipped:` (uma câmera/nível solicitado — ou o índice combinado — que não produziu vídeo, com o motivo — por exemplo, ausência de pacote de calibração, nenhuma leitura DLS para refletância ou nenhum par de bandas vermelha/NIR para o índice combinado) e qualquer `Note:`. Uma câmera RGB solicitada para radianância ou refletância recebe, em vez disso, seu vídeo de exibição, com balanço de branco quando o pacote de calibração da câmera estiver disponível; uma câmera renderizada sem ele é identificada em um `Note:`. Ele **retorna um valor diferente de zero** quando um produto solicitado foi ignorado ou não foi gerado conforme solicitado (uma expressão de índice configurada não é avaliada offline); os vídeos que foram gerados ainda são gravados e listados.

> **Escolha o gravador correto.** O `array-record` é de *nível de monitoramento* — ele grava a composição ao vivo renderizada: a visualização combinada da GUI ou, sem nenhuma GUI aberta, a própria visualização NDVI (ou `--expr`) do próprio backend. O `array-burst` → `array-build-video` é de *nível de análise* — ele salva os dados brutos do sensor na taxa máxima e reconstrói vídeos calibrados de radiação/refletância/índice posteriormente, sem a necessidade de visualização ao vivo.

### Câmeras monocromáticas (M3M) de banda única

A linha **M3M**é a versão monocromática da**M3C**da Bayer: um filtro de interferência de banda estreita por câmera (`M3M-<lens>-F<wavelength>`, por exemplo, `M3M-L87-F685`), portanto, o sensor fornece uma**única banda em escala de cinza**, sem mosaico Bayer. Não há nada para desmosaicar, nenhuma interferência entre canais para separar e nenhum equilíbrio de branco para definir — todo o fluxo de trabalho de cores da exibição RGB simplesmente não se aplica.

O que isso significa na CLI:

- **`lattice white-balance`, `lattice color-profile`, `lattice color`**detectam uma câmera monocromática e**ignoram com uma mensagem de uma linha**, em vez de enviar configurações sem sentido. Eles ainda funcionam normalmente com uma câmera RGB/Bayer M3C na mesma sessão.
- **Um formato de pixel Bayer indica sua profundidade.** Um M3M oferece apenas formatos mono; portanto, um formato Bayer proveniente de `--pixel-format` ou `--preset` define o formato mono com a mesma profundidade: `BayerRG8` e `--preset high_speed` resultam em `Mono8`; `BayerRG12`, `high_quality` e `triggered` resultam em `Mono12`; um nome compactado permanece compactado (`BayerRG12p` resulta em `Mono12p`). Os arquivos e o nome `lattice format` correspondem ao formato Mono. Uma matriz que mistura câmeras M3C e M3M adota um formato da mesma forma: Bayer em cada M3C, sua profundidade Mono em cada M3M. Um nome corresponde à sua profundidade de bits: o mesmo vale no sentido inverso, portanto, uma M3C nunca é colocada em um formato Mono (`--pixel-format Mono12` é `BayerRG12` em uma M3C, independentemente de qual câmera seja a mestre da matriz), e `lattice format --set` mapeia da mesma forma e imprime o formato definido.
- **`lattice calibrate` / `process --reflectance` / `array-capture --processing radiance`** ainda funcionam — a radianância e a refletância são mapas radiométricos *por banda* e estão perfeitamente bem definidas para uma única banda. Quadros monocromáticos contêm uma matriz de resposta do sensor **identitária** (sem separação 3×3), de modo que o plano passa pela matemática de calibração inalterado.
- **Uma única câmera monocromática não pode produzir um índice de vegetação.**NDVI/NDRE/etc. precisam de pelo menos duas bandas (por exemplo, Redro + NIR). Para obter um índice a partir de hardware monocromático, aponte**várias** câmeras M3M em diferentes comprimentos de onda, alinhe-as em uma pilha multibanda e calcule o índice *dessa* pilha:

```bash
# Red (660) + NIR (850) mono pair -> aligned 2-band stack -> NDVI
chloros-cli lattice index --live --profile align.json \
  --preset NDVI --channel red=Mono_660 --channel nir=Mono_850 \
  --save-multiband -o output/
```

`index --live` abre as câmeras por conta própria (uma [sessão transitória](#multi-camera-transient-sessions)), portanto, não conecte nada primeiro: `--serials`; caso contrário, as câmeras habilitadas no perfil (consulte [Índice / Matemática da Vegetação](#index--vegetation-maths)). Enquanto o Chloros mantiver qualquer câmera (a GUI, `cam-connect`, `array-connect`) a solicitação é recusada com o código HTTP 409 `CAMERA_HELD` e o programa é encerrado `1`; libere-as com `chloros-cli lattice cam-disconnect --all` / `chloros-cli lattice array-disconnect --all`.

Os símbolos `--channel` devem corresponder **exatamente** aos nomes dos canais da predefinição (distingue maiúsculas e minúsculas; os NDVIs são em minúsculas `red`, `nir` — consulte `--list-presets`), e os nomes laterais das bandas devem corresponder a uma banda na pilha alinhada (o modo offline também aceita índices de banda com base em 0, por exemplo, `--channel red=0 --channel nir=1`).

O discriminador em toda a pilha é o token `M3M` na string do modelo (ele nunca aparece em uma string `M3C`), apresentado à GUI/SDK como `is_mono`.

---

## Configuração e ajuste da placa de rede do host (matrizes LATTICE)

As câmeras LATTICE transmitem GVSP pela placa de rede Ethernet do host; portanto, para matrizes com várias câmeras, o **driver**e o**tamanho do anel de recepção** são tão importantes quanto a taxa de transmissão. Configurações incorretas aparecem como um portão `FRAMES WILL DROP` / `Reduce ROI to enable` no painel “Configurações do Array” (e no `lattice network-analysis` / no SDK `analyze_array_network()`), mesmo quando as próprias câmeras estão funcionando corretamente.

### Adaptadores USB 10GbE — Realtek RTL8157 (“Realtek USB 10GbE Family Controller”)

As configurações abaixo são propriedades do driver do Windows. No Linux, um adaptador USB também tem um limite (400 MB/s no x86_64, 200 MB/s no NVIDIA Jetson) (consulte [Modelo de fps e burst de matriz](#array-fps--burst-model)), e os buffers de recepção de soquete do kernel são importantes para todos os adaptadores — consulte [Hosts Linux](#linux-hosts--socket-receive-buffers).

| Item | Valor necessário | Por que é importante |
| --- | --- | --- |
| **Versão do driver**|**≥ v10.67 (janeiro de 2026)**, INF `rtump64x64sta.inf` | O driver legado de**2016**(v10.65, `rtump64x64.inf`) lida incorretamente com o desligamento e apresenta erros com**`DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`)**ao desligar, reiniciar ou entrar no modo de suspensão. A transição fica travada (tempo limite de ~5 min), o usuário desliga à força o computador, e desligamentos incorretos repetidos**corrompem o repositório WMI**(o PowerShell e outras ferramentas começam a apresentar falhas com o código `Invalid class`) e**bloqueiam a pilha USB** na próxima inicialização (a placa de rede não é ativada; as unidades USB param de ser reconhecidas). Faça a atualização no site realtek.com (ou junto ao fabricante do dongle) antes de contar com reinicializações corretas. |
| **Buffers de recepção**— palavra-chave `ReceiveBufferLen` |**256**(máximo do driver) | O anel de recepção (RX) da placa de rede. O padrão do driver de**32**deixa apenas ~0,26 MB de anel utilizável — muito pequeno para uma rajada de múltiplas câmeras —, de modo que o painel da matriz reporta `Sim-emit burst … exceeds NIC RX ring usable capacity 0.26 MB` e bloqueia a conexão. Com**256**o anel é grande (**cerca de 13,5 MB medidos no host de 10 GbE do laboratório**), proporcionando ao pipeline de recepção margem real para rajadas GVSP com múltiplas câmeras. (Se uma determinada configuração realmente *se conecta* é decidido por duas verificações — a verificação de **admissão com consciência de drenagem**e a verificação de**sobre-assinatura agregada** — e não por uma comparação bruta entre rajada e anel; consulte [Modelo de fps e rajadas do array](#array-fps--burst-model).) |
| **URBs de recepção**— palavra-chave `PendingReceives` |**64** (máx.) | Blocos de solicitação USB em trânsito; aumente juntamente com os Buffers de Recepção para absorção de rajadas. |
| **Quadro Jumbo** — palavra-chave `*JumboPacket` | **9014** | Necessário para pacotes GVSP de 9.000 bytes (6 vezes menos pacotes por quadro do que com 1.500). |

> ⚠️ **Uma atualização do driver da placa de rede REINICIA essas propriedades avançadas para os valores padrão.**Após atualizar ou substituir o driver do adaptador,**reaplique** `ReceiveBufferLen=256` e `PendingReceives=64`, ou o painel do array entrará em modo de bloqueio novamente, mesmo que “nada tenha mudado no hardware”. Essa é a principal causa de um equipamento que antes funcionava se recusar repentinamente a se conectar.

Aplique a partir de um PowerShell com **privilégios elevados** (substitua pelo nome do seu adaptador, por exemplo, `"Ethernet 5"`):

```powershell
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen -RegistryValue 256
Set-NetAdapterAdvancedProperty -Name "Ethernet 5" -RegistryKeyword PendingReceives  -RegistryValue 64
Get-NetAdapterAdvancedProperty  -Name "Ethernet 5" -RegistryKeyword ReceiveBufferLen,PendingReceives   # verify
```

> **`lattice network --fix` abrange adaptadores USB 10GbE.** Agora ele detecta o tipo de adaptador e ajusta a palavra-chave correta para o anel de recepção: `*ReceiveBuffers`→2048 para placas de rede PCIe (Intel I219, etc.), ou `ReceiveBufferLen`→256 + `PendingReceives`→64 para o controlador **USB** 10GbE da Realtek (que não expõe `*ReceiveBuffers`). Os valores-alvo são limitados ao máximo relatado por cada driver (`NumericParameterMaxValue`), de modo que nunca seja gravado um valor fora do intervalo. Execute a partir de um terminal com **privilégios elevados**; como qualquer ajuste baseado no Registro, a alteração entra em vigor após a reinicialização do adaptador ou do sistema. Os comandos manuais `Set-NetAdapterAdvancedProperty` acima continuam sendo uma boa alternativa — eles são aplicados em tempo real (reassociam o adaptador) sem a necessidade de reinicialização.

### Máquinas Linux
— buffers de recepção de soquetes

No Linux
, incluindo o NVIDIA Jetson, os buffers de recepção de soquetes do kernel são tão importantes quanto o anel de recepção da placa de rede. Na configuração padrão do kernel (`net.core.rmem_max` / `net.core.rmem_default` = 212.992 bytes), uma sequência de pacotes GVSP da câmera causa estouro no soquete de fluxo antes que o Chloros
os leia. Os pacotes são descartados no kernel, embora a própria placa de rede não relate nenhuma perda, e um conjunto de câmeras perde grupos de quadros e reduz sua taxa de quadros. Medido em um conjunto de 4 câmeras conectado a um adaptador USB 10 GbE, aumentar ambos para 32 MB elevou a taxa de quadros do conjunto de 4,96 para 7,52 fps após a conexão e a captura bruta contínua de 4,7 para 7,6 capturas/s, sem que nenhum pacote fosse descartado no kernel.

- **O pacote .deb define esses valores.** A instalação do pacote grava `/etc/sysctl.d/60-chloros-gige.conf` (ambos em `33554432`) e aplica-os imediatamente. Um valor já definido como maior é mantido inalterado.
- **`lattice network` verifica esses valores.** Quando qualquer um dos valores estiver mais baixo, ele lista `socket receive buffers too small for GigE Vision (…)` em *Problemas encontrados*, e o log do backend exibe o mesmo aviso na primeira conexão da câmera ou do conjunto de câmeras.
- **Para aumentá-los manualmente** (até a próxima reinicialização; o arquivo do pacote os aplica a cada inicialização):

  ```bash
  sudo sysctl -w net.core.rmem_max=33554432 net.core.rmem_default=33554432
  ```

  `lattice network --fix` também aumenta e mantém esses valores, mas somente quando o backend é executado como root. Um backend executado como seu usuário (iniciado com `chloros-backend` ou [`chloros-backend@<user>`](#executando-o-backend-como-um-serviço-sem-interface-gráfica)) não pode alterar as configurações do kernel e, em vez disso, exibe a linha `sudo sysctl`.

**O ajuste automático do adaptador afeta apenas os adaptadores conectados.** Na primeira conexão da câmera, um backend executado como root ajusta os adaptadores com fio cujo anel de recepção é pequeno. Agora, ele ajusta apenas adaptadores com um link ativo; portanto, uma porta integrada desconectada mantém seu MTU, e os buffers de recepção não são alterados. O `lattice network` não lista mais o MTU de um adaptador desconectadocomo um problema. `lattice network`, `--fix`, `lattice probe` e o ajuste de adaptadores consideram apenas portas Ethernet com fio, não o barramento CAN do Jetson (`can0`) nem as conexões no modo de dispositivo USB (`usb0`, `usb1`) que ele oferece a um PC host; um adaptador Ethernet USB para câmeras ainda é considerado.

### Noções básicas de rede (todos os links da LATTICE)

- **Endereçamento:** endereço local de link `169.254.0.0/16` (GigE Vision LLA). O host recebe um endereço estático `169.254.x.x/16`; as câmeras e o DAQ-E se autoatribuem endereços no mesmo intervalo. Não é necessário DHCP/gateway.
- **Tamanho do pacote:**prefira jumbo (9.000), mas deixe que a detecção automática o determine — ela ignora o limite de 1.500 bytes do ICMP da câmera por meio de um teste GVSP, de modo que define o tamanho como jumbo sempre que a conexão realmente o suportar.**Cada câmera é medida uma vez por sessão de backend e novamente quando seu link muda (outro adaptador, outra velocidade de link ou um link que caiu, como ocorre ao trocar um switch); as reconexões reutilizam o resultado. Após uma alteração atrás de um switch que permaneceu ativo, reinicie o backend para medir novamente.**Uma primeira conexão pode, ocasionalmente, levar alguns minutos nessa verificação, e o log indica isso quando ocorre. Use o pin com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` somente quando tiver mais certeza do que o probe, e prefira o uso por comando em vez de permanente: um pin ignora o probe, portanto, se o caminho não puder realmente suportar 9000,**todas** as capturas expiram com o `SC_ERR_TIMEOUT -1011` (consulte [Variáveis de Ambiente](#environment-variables)).
- **O anel RX é dimensionado com `ReceiveBufferLen`:**no valor padrão `32`, o anel utilizável é de ~0,26 MB (muito pequeno para qualquer rajada com várias câmeras); no valor máximo `256`, ele é grande (~13,5 MB medidos no host de 10 GbE do laboratório), proporcionando margem real. A conexão de uma configuração é então decidida pela verificação de admissão com consideração de drenagem**e** pela verificação de sobrescrita agregada abaixo — não por uma comparação bruta entre burst e anel.

### fps do Array e modelo de burst

Como interpretar o painel de configurações do Array (e o `lattice analyze-array` / o `analyze_array_network` do SDK
):

- **O burst é somado por câmera no formato de pixel real de cada câmera.**Câmeras Mono**M3M**transmitem em**Mono12 (2 B/px)**ou, com o `--pixel-format`, no formato Mono de sua profundidade (`BayerRG8` é `Mono8`, 1 B/px); as câmeras**M3C**Bayer transmitem em 8 ou 12 bits (o firmware M3C emite silenciosamente BayerRG12 mesmo quando BayerRG8 é solicitado). Portanto, um quadro em resolução total com 4 câmeras tem**~12,6 MB se todas forem de 8 bits, mas ~25 MB com três câmeras mono de 12 bits**. A projeção determina o formato de cada câmera a partir de seu modelo (cache de identidade), de modo que o burst corresponde ao que o cabo realmente transmite — e não a uma suposição genérica de BayerRG8.
- **Um adaptador Ethernet USB tem limite máximo de 200 MB/s, independentemente da especificação nominal.** A tabela de eficiência que transforma uma taxa de link em um valor sustentado é derivada do PCIe; uma placa de rede USB anuncia sua taxa de link *Ethernet*, mas é limitada pelo barramento USB e seu driver. Um dongle USB 10GbE costumava atingir ~1.063 MB/s “sustentada” — um número que nunca foi comprovado — e o ritmo resultante corrompia de 6 a 18 % dos quadros, embora ainda relatasse uma taxa de quadros por segundo (fps) alvo satisfatória. As placas de rede conectadas via USB agora têm um limite absoluto: **200 MB/s** no Windows
e no NVIDIA Jetson, **400 MB/s** no Linux
x86_64 (no Linux
, um adaptador conectado a uma porta USB 2.0 tem capacidade limitada a **30 MB/s**, e as especificações recomendam conectá-lo a uma porta USB 3) (o limite é determinado pelo barramento USB e pelo driver do host, portanto não varia de acordo com a especificação nominal; um adaptador USB 1 GbE atinge cerca de 80 MB/s e não é afetado). No Linux
, o adaptador é identificado pelo sysfs (recorrendo a `ethtool -i` como `bus-info`). Cada valor é medido: no Windows
, o mesmo conjunto de 4 câmeras corrompeu de 6 a 10 % dos quadros acima de 200 MB/s; no Linux
x86_64, o mesmo adaptador funcionou sem erros em todos os limites até 480 MB/s (273 MB/s entregues, o limite das próprias câmeras). No NVIDIA Jetson, o mesmo adaptador funcionou sem erros apenas a 200 MB/s e perdeu quadros intermitentemente acima dessa velocidade. O `wire_ceiling_source` no registro de capacidade indica isso por escrito, e o `nic_is_usb` sinaliza essa informação. Substitua qualquer uma das opções com `--wire-ceiling-mbps`.
- **A admissão leva em conta o consumo de recursos, não a comparação entre burst completo e anel.** Um burst simultâneo precisa apenas caber no *backlog transitório* = `max(0, Σ per-cam arrival − host drain) × emit_window`, não no burst inteiro. Em uma malha de host rápido / câmera lenta (um **PCIe**de 10G + 4× cams de 1 GbE: chegada ≈ 320 MB/s, drenagem ≈ 1063 MB/s), o host esvazia mais rápido do que as câmeras enchem; o backlog é ≈ 0; portanto, a emissão simulada em resolução total**é permitida**, mesmo que o burst de 25 MB exceda o anel de 13,5 MB. Coloque as mesmas quatro câmeras atrás de um adaptador**USB** de 10 GbE no Windows
e a taxa de escoamento será de 200 MB/s, e não 1.063 — a chegada ultrapassa essa taxa, e a perda aparece como quadros corrompidos, em vez de uma taxa de quadros mais baixa. Em um host de 1 GbE, o limite mínimo de DLThr de 31,25 MB/s das câmeras faz com que a chegada ultrapasse a taxa de drenagem → ele **bloqueia** corretamente (para *essa* classe de bloqueio, reduza a ROI ou use binning ≥ 2). A admissão é uma das **duas** portas de conexão — a outra é a verificação de sobrescrita agregada abaixo.
- **Os fps projetados correspondem ao ciclo da câmera mais lenta.**Na emissão simultânea, todas as câmeras enviam seus quadros ao mesmo tempo; assim, o array opera no ciclo correspondente ao maior entre o tempo de leitura do sensor e o tempo de emissão de seu membro mais lento (bytes do quadro ÷ sua taxa de emissão: o**link de acesso** da câmera, 1 GbE ≈ 80 MB/s, ou sua parcela da largura de banda máxima do host quando esta for menor). Cada exposição se sobrepõe à leitura do quadro anterior; portanto, a exposição não entra no ciclo: a exposição automática preenche o período determinado pela taxa, depois aumenta o ganho, e uma cena escura não reduz mais a taxa (na versão 1.2.0 isso acontecia). Apenas uma exposição manual mais longa que o período retarda o disparo (consulte [Exposição em um membro da matriz](#exposure-on-an-array-member)). Adicionar câmeras não retarda a matriz até que a largura de banda compartilhada entre elas diminua. Na emissão serial (`slip-emit-and-capture`), os tempos de emissão das câmeras são somados, e a exposição de cada membro é considerada no escalonamento. Medido com sensor completo, 12 bits, em uma matriz de 9 câmeras com um host PCIe de 10 GbE: 10,7 fps, ou 14,0 fps com [transporte compactado](#packed-12-bit-transport---packed).
- **Após uma captura contínua, a projeção indica o valor mantido.** Uma captura de 30 s ou mais em uma taxa automática registra a taxa em que se estabilizou para aquele tamanho de matriz neste computador; as breves tentativas do controlador de taxa de uma velocidade mais rápida são desconsideradas. A partir daí, a linha do `array-connect` e o painel “Configurações da Matriz” indicam essa taxa quando ela está abaixo da do modelo, com a causa do limite: `clock-bound` (a matriz operou na taxa em que se estabilizou, o loop de captura aguardando seu relógio de acionamento), `loop-bound` (o loop de captura não tinha mais tempo), `capture-bound` (a captura ficou para trás e pulou grupos) ou `measured` (não há indicação de qual). O valor do modelo está em `wire_fps_max`. A matriz ainda atinge o limite que a conexão suporta.
- **Um array funciona em qualquer rede — a taxa de quadros é que determina o limite.**Cada câmera recebe uma parcela igual do**limite máximo da conexão à prova de colisões**(`sustained × sim_emit_factor`), sincronizada com o GevSCPD, e todas elas ainda expõem no mesmo gatilho. Em um cabo lento, a parcela é pequena e a taxa de quadros cai com ela: 8 câmeras de 12 bits em resolução total em 1 GbE sem quadros jumbo recebem 52 / 8 = 6,5 MB/s cada, cerca de**1 fps**(cerca de 1,3 fps compactados), enquanto um host de 10G as opera a 11-14. Se as câmeras não puderem ser sincronizadas tão lentamente — verifique o intervalo GevSCPD de cada câmera para descobrir —, o conjunto muda para emissão**serial**(`slip-emit-and-capture`): as câmeras se alternam na linha na taxa total do link, acionadas uma após a outra, de modo que todos os quadros ainda chegam, mas os membros**não são expostos simultaneamente**. `analyze-array` e a saída do `connect` indicam o modo; defina `--force-tier sim-capture-sim-emit` para recusar o fallback. O tamanho do quadro não influencia esse compartilhamento, portanto,**o binning e uma ROI menor aumentam a taxa de quadros, mas não alteram se uma matriz cabe**. Apenas um cabo que não consiga transportar a matriz, mesmo em modo serial, está**sobrecarregado**— por exemplo, muito mais câmeras do que um cabo pode suportar, ou muitas outras câmeras transmitindo nele — e, nesse caso, o sintoma é a perda de pacotes GVSP, não uma taxa mais baixa: O `analyze-array` zera os valores de fps alcançáveis e exibe o `** OVER-SUBSCRIBED**`; a conexão prossegue com um aviso sonoro, e uma conexão HTTP
com o `pin_resolution: true` (modo estrito) **é recusada**. As soluções são: reduzir o número de câmeras, usar quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` reclassifica a recusa para um aviso sonoro durante o trabalho de teste — consulte [Variáveis de Ambiente](#environment-variables).

### Integridade do array — qual subsistema está perdendo quadros

O `GET /api/camera/array/<array_id>/capability` de um array conectado contém um bloco
`health` em tempo real, reavaliado em uma janela contínua de **10 segundos**. Ele divide a perda de quadros
em duas causas que exigem correções opostas, em vez de relatar uma taxa “incompleta”
que não especifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serial) | O quadro **chegou e estava estruturalmente corrompido**— perda de pacotes GVSP. |**Rede**: largura de banda, ritmo de transmissão, anel de recepção da placa de rede, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nem chegou**— a câmera não disparou ou nada saiu dela. |**Acionamento/sincronização**: cabo M8, `--line`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | A pior taxa de cada câmera. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmera (ambas as causas juntas). | — |
| `stable_for_seconds` | Por quanto tempo cada câmera permaneceu abaixo de 0,01 %. | — |

Acima de 5%, o backend registra uma linha `[array-health <id>] WARN` indicando a divisão — na
primeira violação, em uma mudança de faixa de gravidade, uma vez por minuto enquanto persistir e uma vez quando
for resolvida. A metade corrompida grava `[gvsp-corrupt <SN>]` no arquivo de log do backend na
primeira ocorrência por câmera e motivo, e depois um resumo a cada 60 s; o painel de log do aplicativo exibe a
linha `[array-health <id>]`. Todas as avaliações ainda são registradas no arquivo de log do backend;
os contadores são atualizados a cada buffer, independentemente do que for impresso.

O mesmo registro informa o número ao qual toda a alocação está vinculada:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de banda sustentado do host em vigor, em MB/s. |
| `wire_ceiling_source` | De onde veio esse número, em palavras — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; PnPDeviceID=USB\VID_0BDA&PID_815A)` (no Linux
x86_64, `USB-capped 400 MB/s (…; sysfs: bus=usb (usb2/2-1/2-1:1.0))`) ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando definido por `--wire-ceiling-mbps` (ou pelo campo **Orçamento de banda** da GUI). |
| `nic_is_usb` | `true` para um adaptador Ethernet USB — consulte o limite de USB acima. |

**Interpretação:** `gvsp_corrupt_rate_pct` diferente de zero com `never_arrived_rate_pct` igual a 0
significa que o disparo e a sincronização do cabo estão perfeitos e que 100% da perda está no caminho da rede
— reduza o valor de `--wire-ceiling-mbps` e reconecte. O padrão inverso aponta para o
cabo de sincronização ou para a linha de disparo.

> **O `--target-fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido
> uma vez na conexão; portanto, diminuir a taxa de acionamento altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na demanda não produziu melhora;
> reduzir o limite máximo da linha de 240 para 200 MB/s levou o mesmo equipamento de 10,4 %
> de quadros corrompidos para 0,00 %.

> **Um array em operação não pode se replanejar sozinho no firmware LATTICE (M3C / M3M).** Ela
> não consegue corrigir isso sozinha; desconecte e reconecte para que o seletor de tempo de conexão possa
> se replanejar com o novo limite máximo.

### Sintoma → correção

| Sintoma (Configurações da matriz / conexão / `analyze_array_network`) | Causa | Correção |
| --- | --- | --- |
| `FRAMES WILL DROP … exceeds NIC RX ring usable capacity 0.26 MB`, `Reduce ROI to enable` | `ReceiveBufferLen` reajustado para 32 (normalmente após uma atualização de driver) | Defina `ReceiveBufferLen`→256, `PendingReceives`→64; reabra o painel (reinicie o backend caso ele tenha armazenado em cache o tamanho antigo do anel) |
| Travamento ao reiniciar/desligar; posteriormente, erros WMI relacionados a `Invalid class`, a placa de rede não é ativada, drives USB não são reconhecidos | Driver antigo Realtek USB 10GbE de 2016 → BSOD `0x9F` → desligamentos forçados | Atualize o driver do adaptador para ≥ v10.67 (2026) e, em seguida, reaplique as configurações do anel de recepção acima |
| O array se conecta com o sensor no máximo, mas apresenta baixa taxa de quadros | O modo automático mantém o sensor no máximo e permite que a taxa caia para o que a linha suporta | Escolha uma das opções mais rápidas listadas na saída de conexão (`--binning 2` / `4`, ou um `--width`/`--height`), habilite quadros jumbo ou atualize o link |
| O Array relata uma taxa de quadros alvo válida, mas entrega apenas uma fração dela; `health.gvsp_corrupt_rate_pct` diferente de zero, `never_arrived_rate_pct` 0 | O orçamento de banda inferido do host superestima o que ele realmente sustenta (típico em um adaptador Ethernet USB, uma pista PCIe estreita ou uma malha compartilhada) | Reconecte-se com um valor menor de `--wire-ceiling-mbps` e verifique novamente o bloco de integridade. **Não** `--target-fps` — O ritmo do GevSCPD é fixado na conexão |
| Linux
: um array perde grupos de quadros e reduz sua taxa de quadros, enquanto a placa de rede não relata perdas; `lattice network` lista `socket receive buffers too small` | Os buffers de recepção de soquetes do kernel estão em seus valores padrão, portanto, os pacotes da câmera são descartados no kernel | Aumente o valor de `net.core.rmem_max` / `rmem_default` para 32 MB — consulte [hosts Linux
](#linux-hosts--socket-receive-buffers). O pacote .deb faz isso durante a instalação |
| Câmeras ausentes dos grupos publicados; `health.never_arrived_rate_pct` diferente de zero, `gvsp_corrupt_rate_pct` 0 | Acionamento / caminho de sincronização — as câmeras não estão disparando, não é um problema de rede | Verifique o cabo de sincronização M8 e o `--line`; confirme se todos os membros estão armados (`TriggerMode=On`) |
| `**OVER-SUBSCRIBED**` / `Wire budget` excedido em `analyze-array`, ou recusa de conexão no modo estrito (`pin_resolution`; `array over-subscribes the wire`) | O cabo não consegue suportar a matriz em qualquer ritmo que as câmeras atinjam, seja simultaneamente ou alternadamente — há muito mais câmeras do que o cabo pode suportar, ou outras câmeras transmitindo nele | Menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida. **ROI/binning NÃO ajudará** (o compartilhamento é independente do tamanho do quadro). `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` é substituído no ambiente de teste (aceita a perda de pacotes) |
| A matriz se conecta no `slip-emit-and-capture` (serial) em uma rede lenta | As câmeras não podem ter uma cadência lenta o suficiente para compartilhar o cabo simultaneamente, então elas se revezam; os quadros não são expostos no mesmo instante | Quadros jumbo ou uma placa de rede mais rápida restauram a captura simultânea; `--force-tier sim-capture-sim-emit` insiste nisso (e, em vez disso, perde pacotes) |

---

## `chloros-cli daq`

Comandos do sensor espectral. Os comandos `pool-*` são clientes HTTP
“leves” que acionam o sensor por meio do pool persistente do backend; portanto, precisam de um backend em execução (consulte [Iniciar o Backend](#iniciar-o-backend)). O backend é responsável pelo transporte, de modo que os scripts da CLI
e do SDK
compartilham um único identificador ativo, em vez de disputarem a porta serial. O `pool-discover` é a única maneira de descobrir o MAC BLE de um DAQ-M.

> **`chloros-cli daq --help`** (ou `-h`, ou `daq help`) lista os subcomandos do `pool-*` (no Linux
, juntamente com os comandos diretos de hardware que a compilação inclui), e o `chloros-cli daq <subcommand> --help` (ou `daq help <subcommand>`) apresenta as opções de cada um. Elas abrangem conexão, transmissão, gravação de arquivos `.daq` calibrados, troca de perfis de capacitância e vinculação de um DAQ a uma câmera; e, para um DAQ-E, suas configurações de rede, PTP, reinicialização e atualizações de firmware. O DAQ também pode ser controlado a partir do Python
com o comando `chloros_sdk.connect_daq_sensor()`, que utiliza o mesmo caminho agrupado.
>
> **Utilize os comandos `pool-*` em todas as plataformas.** Eles se comportam da mesma forma no Windows
, Linux
e Jetson. No Windows
, `daq discover` e `daq list` são aliases de `pool-discover`, e qualquer outro comando `daq` que não seja `pool-*` é executado como `1` com `DAQ direct-hardware commands require the full daq package`. No Linux
e no Jetson, os comandos `daq discover`, `daq list` e os demais comandos que não sejam `pool-*` e `daq` são os comandos diretos: eles abrem o sensor no próprio processo da CLI
, fora do pool do backend (portanto, não em um sensor mantido pelo backend, pela GUI ou por um script), e lá aceitam os comandos `daq discover`, `--only` e `--timeout`, mas não o comando `--json`. Esta referência documenta apenas os comandos `pool-*`.

### Fluxo de trabalho da primeira conexão do sensor DAQ

```bash
# 1. Scan every transport, showing the address to connect with.
#    This is how you find a DAQ-M's BLE MAC — unlike a DAQ-E hostname or a
#    DAQ-U serial port, a MAC isn't printed on the device or listed by the OS.
chloros-cli daq pool-discover
chloros-cli daq pool-discover --only ble           # BLE only
chloros-cli daq pool-discover --json               # machine-readable

# 2. Open a persistent pool session (handle stays alive across CLI calls)
chloros-cli daq pool-connect                                   # smart-detect
chloros-cli daq pool-connect --port /dev/ttyUSB0               # DAQ-U on a serial port (Windows: COM3)
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF           # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-xxx.local        # DAQ-E by hostname

# 3. List what's in the pool, including the sensor_id you'll use next
#    (DAQ-U ids look like 'CB-7C-A8-2E-5F'; DAQ-E ids like 'daq-e-def330')
chloros-cli daq pool-list

# 4. Read the latest spectrum frame
chloros-cli daq pool-latest --sensor-id CB-7C-A8-2E-5F

#    Bind it to a camera, so that camera's captures use and save its readings
chloros-cli daq pool-bind --camera 213800234 CB-7C-A8-2E-5F

# 5. Record a calibrated .daq file for 60s
chloros-cli daq pool-record --sensor-id CB-7C-A8-2E-5F --duration 60 \
  -o ~/Documents/spectra --device-name "field-A"

# 6. Release
chloros-cli daq pool-disconnect --sensor-id CB-7C-A8-2E-5F
```

### Referência do `pool-*`

| Subcomando | Finalidade |
| --- | --- |
| `daq pool-connect` (smart-detect) | Abre um sensor no pool de back-end. |
| `daq pool-connect --port PORT` (`-p`) | DAQ-U em uma porta serial específica. |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC detectado automaticamente. |
| `daq pool-connect --mac MAC` (`-m`) | DAQ-M via BLE em um MAC conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E via Ethernet com nome de host ou IP conhecido (implica `--eth`). |
| `daq pool-connect --eth` | DAQ-E via Ethernet, host detectado automaticamente (mDNS + fallback ARP; funciona com cache ARP vazio no Windows
e no Linux
). |
| `daq pool-connect --integration-time MS (-t) --frame-avg N (-f) --no-ae` | Ajustar janela de integração / estado do AE. |
| `daq pool-connect --no-stream` | Conectar, mas não iniciar a transmissão ainda (retomar com `pool-stream --start`). |
| `daq pool-connect --cap-id {none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}` | Perfil de correção de limite. O padrão no backend é `sunshine_cosine`; `none` é um sensor sem capa. Um [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor) o ignora (difusor fixo). **Os sensores são fornecidos sem tampa; portanto, nomeie a tampa que você instalou:** o padrão é registrado como (`cap_id_source=auto_default`) e o processamento emite um aviso a respeito, já que um DAQ-U sem difusor sob a luz solar direta apresenta uma leitura cerca de 30 vezes maior. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Verifique cada transporte em busca de sensores aos quais você possa se conectar, sem efetuar a conexão. **É assim que você encontra o MAC BLE de um DAQ-M.** `--timeout` é por transporte (padrão 5 s, 1-20). Os sensores já abertos no pool não são listados — um DAQ-M conectado para de anunciar — portanto, use o comando `pool-list` para esses casos. `usb` lista apenas dispositivos seriais USB: as portas seriais do próprio computador (`COM1`, `/dev/ttyS*` do Linux
) não podem ser sensores e são excluídas. |
| `daq pool-list` | Mostra todos os sensores no pool de back-end. |
| `daq pool-bind (--camera SERIAL \| --array ARRAY_ID) {SENSOR \| --clear}` | Vincula uma câmera ou uma matriz a um DAQ, da mesma forma que a seleção de sensores de luz da GUI faz: suas capturas utilizam, correspondem e salvam as leituras desse DAQ. `SENSOR` é um ID de sensor de `pool-list` ou o endereço no qual o DAQ é acessado. `--clear` (sem `SENSOR`) cancela a vinculação, e as capturas voltam a usar o DAQ mais recente. A vinculação permanece até que o backend seja reiniciado ou até que a câmera, o conjunto de sensores ou o DAQ seja desconectado: uma desconexão (aqui, do SDK
ou no aplicativo) interrompe todas as vinculações do hardware desconectado, independentemente de quem as tenha definido, incluindo os membros de um conjunto de sensores junto com o próprio conjunto. A câmera deve estar conectada, no projeto aberto, ou ter sido aberta neste computador anteriormente, e a matriz conectada ou no projeto aberto; qualquer outro número de série ou ID é rejeitado por nome e nada é vinculado. |
| `daq pool-disconnect {--sensor-id ID \| --all}` | Liberação — quem quer que tenha aberto o sensor; assim, também libera um sensor que um script com falha deixou no pool (qualquer gravação nele é concluída primeiro). Um sensor que não esteja no pool não tem nada a ser liberado: ele exibe `ID: already released (not in the pool)` e encerra com `0`; uma liberação que o backend recusou ou não confirmou encerra com `1`. SDK
: `DAQSensorSession.force_disconnect()`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Os N quadros de espectro mais recentes (padrão: 1). |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [--duration SEC] [-o DIR] [--device-name NAME] [--confirm-after S] [--heartbeat S]` / `--stop` | Iniciar/parar uma gravação .daq. `-o` é definido por padrão como `~/Documents/DAQ Live View` no computador do backend (um caminho relativo está na pasta a partir da qual você executa o comando); `--duration` (`-d`, padrão `0` = até `--stop`). `--confirm-after S` emite um aviso quando nenhuma leitura chega S segundos após o início (padrão 20; 0 desativa o recurso); com `--duration`, `--heartbeat S` exibe a contagem de leituras a cada S segundos e emite um aviso quando ela para de crescer (padrão 30; 0 desativa o recurso). Um registro em um DAQ-E que fica em silêncio continua no mesmo arquivo assim que o backend se reconectar, e o `--stop` lista as lacunas ([Um DAQ-E que para de enviar dados](#a-daq-e-that-stops-delivering)). Ctrl+C encerra antecipadamente uma gravação do `--duration` e informa o ocorrido; sem o `--duration`, a gravação continua rodando após o Ctrl+C (sair do `130`) até o `--stop`. |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Troca o perfil de correção de capacitância em tempo de execução. É necessário o `--cap-id`; o `--cap-id none` remove a correção de capacitância (um sensor sem capa). Um DAQ-E-S mantém o `as_recorded` independentemente da configuração. |
| `daq pool-enable-ptp --sensor-id ID` | Reativa o PTP em um DAQ-E cujo PTP esteja desativado. A configuração é salva na unidade, portanto, uma unidade deixada desativada permanece desativada — seus carimbos de data/hora não são sincronizados por PTP — até que este comando seja executado. `pool-list` sinaliza tal unidade (`!! PTP is switched OFF`) e imprime este comando para ela. O Chloros
nunca reativa uma unidade por conta própria, pois o desligamento pode ser intencional. |
| `daq pool-info --sensor-id ID [--json]` | As configurações próprias de um DAQ-E: seu firmware (e se esta versão do Chloros
traz uma versão mais recente), seu endereço e como ele o obteve (DHCP / link-local / estático), a configuração de rede salva até que o sensor a execute (aquela salva com o `--no-reboot` incluída), o fluxo na inicialização e o PTP. Em um sensor cujo fluxo não esteja chegando a este computador, ele lê o sensor em uma nova conexão e informa isso. |
| `daq pool-set-network --sensor-id ID {--auto \| --static IP [--mask M] [--gateway G]} [--no-reboot] [--force] [--timeout S]` | Configure um DAQ-E para DHCP (`--auto`) ou um endereço estático (`--mask` padrão `255.255.255.0`). O sensor reinicia para aplicar a configuração e é reconectado com as configurações anteriores, aguardando até `--timeout` segundos (padrão: 45) para isso; `--no-reboot` apenas salva a configuração. Consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings). |
| `daq pool-set-auto-stream --sensor-id ID {--on \| --off}` | Se um DAQ-E inicia a transmissão ao ser ligado sem nenhum software conectado (salvo no sensor). O Chloros
inicia a transmissão por conta própria de qualquer maneira. |
| `daq pool-reboot --sensor-id ID [--timeout S]` | Reinicie um DAQ-E e reconecte-o com suas configurações anteriores (aguarda até `--timeout`, padrão 45 s). |
| `daq pool-update-firmware --sensor-id ID [--timeout S]` | Atualize o firmware do DAQ-E incluído nesta versão do Chloros
em um sensor que esteja executando uma versão mais antiga e, em seguida, reconecte-o (cerca de um minuto; não desligue a alimentação; `--timeout` padrão de 45 s para que ele volte a funcionar). Um sensor que já esteja atualizado não é alterado. |

### Um DAQ-E que para de transmitir

Um DAQ-E no conjunto que não transmite nenhum espectro por 20 segundos é reconectado pelo backend. Isso é feito por meio de um ciclo de reinicialização do switch que alimenta o sensor: o sensor reinicia com ele e não transmite novamente até que receba a instrução para (a menos que `pool-set-auto-stream --on`).

- **Primeiro, ele verifica se é o mesmo sensor.** Ele consulta, no endereço que o sensor possuía — sem alterar nada —, o nome, o MAC e o número de série do sensor. Ele também consulta pelo nome `daq-e-<id>.local` do sensor, mas só confia nessa resposta quando ela vem do próprio adaptador de rede (MAC) do sensor. Nenhum outro dispositivo que responder será reconectado ou terá dados gravados.
- **Ele se reconecta com as configurações da sessão:** tempo de integração, média de quadros, exposição automática e perfil de cap, exatamente como estavam, e não com os padrões de inicialização do sensor. Um sensor que estava com o PTP ativado e voltou com ele desativado é reativado. Em seguida, ele reinicia a transmissão e o receptor do computador para ela.
- **Ele tenta novamente em intervalos crescentes:** imediatamente, depois após 5, 10, 20 e 40 segundos, e então a cada 60 segundos enquanto o sensor estiver indisponível. Cada tentativa apenas pergunta ao sensor qual é a sua identificação. Um sensor que responde, mas ainda assim não envia nada após três tentativas de reconexão, é deixado em paz e reportado (`auto-reconnect GAVE UP`). `pool-stream --start` tenta novamente.
- **`pool-list` mostra em que ponto está.** Enquanto o sensor permanece em silêncio, sua linha `STREAM FAULT` indica o que a tentativa de reconexão está fazendo. A linha é apagada com o primeiro espectro, e o sensor então exibe `reconnected automatically N time(s)`.
- **A gravação continua no mesmo `.daq`.** O arquivo marca cada lacuna com duas linhas: `event_type` 5 no último espectro antes da lacuna e 6 no primeiro depois dela, cada uma com apenas um carimbo de data/hora. `pool-record --duration` imprime `!! STREAM LOST …` enquanto o sensor está ausente e `stream resumed after …` quando ele retorna. `pool-record --stop` lista cada lacuna e emite um aviso, assim como faz para um arquivo vazio (um registro que não contém nenhuma leitura sai de `1`).
- **Um registro mantém uma calibração.** Se o sensor retornar com uma calibração diferente enquanto um registro está em execução, o restante do registro mantém a calibração com a qual começou, e o aviso de parada informa isso. `pool-disconnect` + `pool-connect` adota a nova calibração.
- **Seus comandos prevalecem.** Um sensor que você interrompe com `pool-disconnect` ou pausa com `pool-stream --stop` nunca é reconectado, mesmo quando uma reconexão já estiver em andamento.
- **`pool-info` em um sensor silencioso** lê o sensor em uma nova conexão e adiciona `!! stream NOT reaching this computer`. Um sensor que não responde é relatado imediatamente. O comando `pool-list` não exibe mais um estado PTP para um sensor que parou de responder.
- O DAQ-U e o DAQ-M não estão incluídos. Um DAQ-M se reconecta via Bluetooth por conta própria, e um DAQ-U desconectado do USB precisa do comando `pool-connect` novamente.

### DAQ-E-S — difusor fixo e sensor de inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças relevantes para o software. Todo o restante — descoberta, conexão, PTP, gravação e as configurações do dispositivo abaixo — funciona exatamente como no DAQ-E.**Seu difusor é permanente, portanto não requer correção de tampa.** O difusor do tipo Sunshine é integrado e já estava na unidade quando sua calibração de fábrica foi medida; assim, essa calibração já indica a irradiância descendente real. O Chloros
define sua tampa como `as_recorded` (sem perfil), independentemente do que for solicitado: um limite nomeado para ele (`sunshine_cosine`, `fov_*`, `none`) é ignorado com uma mensagem, e seus registros armazenam `cap_id_source=model`, de modo que o aviso de limite presumido nunca é acionado para ele. Também não aplique um limite aos dados manualmente: um limite de `sunshine_cosine` no topo resultaria em uma leitura cerca de 11 vezes maior.

**Ele possui um acelerômetro de 3 eixos.** A resposta cosseno do difusor de luz solar só é válida enquanto ele estiver nivelado; portanto, um sensor inclinado relata uma irradiância incorreta — e a refletância herda o erro — sem nenhum outro sintoma. Cada leitura do DAQ-E-S, portanto, carrega sua atitude:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ângulos em graus. `None` quando o firmware marca o ângulo como indefinido (queda livre, um eixo inativo). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleração em mili-g. Um valor de `mag_mg` muito distante de ~1000 significa que a unidade estava em movimento e os ângulos não são significativos para essa amostra. |
| `cal_applied` | Definido: os eixos e ângulos estão no sistema de referência do difusor (a calibração de montagem da unidade é aplicada). Limpo: sistema de referência bruto da placa, que pode apresentar até ~2,3° de inclinação que não é real. |
| `sample_age_ms` | Tempo decorrido desde a amostra do acelerômetro até o envio do espectro. |
| `temp_raw` | **Tendência** da temperatura do chip não calibrada — apenas diferenças, nunca uma leitura do termômetro. |

O `daq pool-latest --json` retorna esses valores sob a chave `imu` de cada quadro (`null` em um DAQ-U, DAQ-M ou DAQ-E comum). Um `.daq` registrado a partir de um DAQ-E-S armazena os mesmos valores em suas colunas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ao lado de cada espectro, para que o leitor possa distinguir uma nuvem de um tripé que tenha sido derrubado. Em todos os outros modelos, essas colunas ficam vazias.

O fluxo de atitude é relatado separadamente: o `daq pool-list` imprime o `!! ATTITUDE STREAM FAULT (spectra unaffected)` enquanto está desligado e quantas falhas ele recuperou posteriormente. Uma falha nesse fluxo nunca causa uma lacuna no registro espectral. `daq pool-discover` lista a unidade como `DAQ-E-S` (como `DAQ-E` se ela não responder à verificação de identidade em cerca de 2 s).

### Configurações do dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330

# A fixed address on the network this computer is on (192.168.1.x)
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1

# Back to the factory setting: DHCP, falling back to a 169.254.x.x
# link-local address when no DHCP server answers
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto

chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

- **O `--static` sem o `--gateway` não salva nenhum gateway** (qualquer gateway definido anteriormente é descartado), o que é adequado para uma rede de câmeras sem roteador.
- **É necessário que haja um endereço estático na rede deste computador.** O Chloros
rejeita qualquer endereço cuja sub-rede não inclua o endereço deste computador na rede do sensor: após a reinicialização, nada conseguiria acessar o sensor para reverter a configuração. Utilize o `--force` somente quando este computador for reendereçado ou roteado para a nova sub-rede.
- **A reinicialização é recusada enquanto o sensor estiver gravando.** Interrompa a gravação primeiro (`pool-record --stop`). Uma gravação iniciada no momento anterior à reinicialização é encerrada por ela, e os códigos `pool-set-network`, `pool-reboot` e `pool-update-firmware` terminam com uma linha `Warning:` indicando o nome do arquivo da gravação (mesmo quando o sensor não retorna).
- **Após uma reinicialização, o Chloros
reconecta apenas o próprio sensor.** Ele aguarda no local onde o sensor estava — e, após `--auto`, também sob seu nome `daq-e-<id>.local` — por uma resposta com o nome do sensor, nome do host e modelo, a partir do adaptador de rede (MAC) que ele possuía, e o reconecta no endereço que respondeu. Uma resposta com esse nome só é aceita com aquele MAC; portanto, de um computador que acessa o sensor por meio de um roteador, ou do WSL2 em sua configuração de rede NAT padrão, um sensor que o DHCP moveu para um novo endereço não é reconectado: o comando indica isso e exibe o comando `pool-connect --eth-host <its address> …`, que o traz de volta com as configurações de limite e exposição que ele possuía.
- **`pool-update-firmware` rejeita um sensor cuja versão de firmware não consegue ler.** Um sensor que demora mais do que o tempo de espera para responder após o flash não apresentou falha: assim que responde, ele é reconectado com as configurações do novo firmware preenchidas, e o comando avisa que ele demorou para responder.
- **Uma atualização de firmware requer uma conexão TCP de entrada.** O sensor se reconecta a este computador para coletar a imagem. O Chloros
fica à escuta na placa de rede que encaminha para o sensor (em todas as placas, apenas quando não consegue identificar qual) e atende apenas ao endereço do próprio sensor; qualquer outra conexão é fechada sem ser atendida. No Windows
, a regra de firewall `Chloros Backend (TCP-In)` do instalador permite a entrada (consulte [Windows
(.exe)](#windows-exe)); um firewall que bloqueia a conexão faz com que a atualização falhe com o código de erro `… device never connected back on TCP <port> …`.
- **No WSL2, use rede espelhada** (`networkingMode=mirrored` em `%UserProfile%\.wslconfig`). No modo NAT padrão, o endereço deste computador na rede do sensor é o próprio do WSL; portanto, todos os endereços estáticos são recusados (exceto `--force`), a transmissão ao vivo não chega ao WSL e a atualização do firmware não pode ser concluída, pois o sensor se conecta de volta ao computador para receber a imagem.
- **O ID do sensor e o nome do host não mudam.** Seja qual for seu endereço, um DAQ-E continua sendo `daq-e-<id>` e `daq-e-<id>.local`, portanto, o `--eth-host daq-e-<id>.local` continua encontrando-o.
- **A GUI mantém as mesmas configurações**: guia “Sensores de luz”, configurações do sensor (engrenagem), “Rede”. A GUI mantém sua própria conexão com um sensor; portanto, ao alterar um sensor que a GUI tenha aberto a partir da própria GUI: uma reinicialização pela CLI
encerra a sessão da GUI com ele.
- Os equivalentes no SDK
são `DAQSensorSession.device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`.

---

## `chloros-cli project`

Abrir, conectar-se e executar um projeto Chloros
salvo (uma pasta contendo `cameras.json` + `sensors.json` + `project.json`). Tudo passa pelo backend, de modo que a GUI e a CLI
apresentam o mesmo estado do hardware.

### Referência de subcomandos

| Subcomando | Finalidade |
| --- | --- |
| `project open PATH [--relink FOLDER]` | Imprime o manifesto de dispositivos do projeto (câmeras, matrizes, sensores), lê a partir de seus arquivos e verifica seus arquivos-fonte (consulte [Um projeto movido](#a-moved-project---relink)). Ele não abre o projeto no backend: o projeto, já aberto lá, bem como seus dispositivos e gravações, permanecem como estão. Encerra o `1` quando nenhum dos arquivos-fonte do projeto puder ser encontrado. |
| `project devices PATH [--reconnect]` | Listar ou reexecutar a descoberta. |
| `project connect PATH [--cameras-only] [--sensors-only] [--packed {auto,on,off}]` | Conectar todas as câmeras / matrizes / sensores salvos. Cada matriz se conecta da mesma forma que o `lattice array-connect`: até 600 s, com suas etapas impressas no stderr (`[array <id>]    <s> s: <step>`; o stdout é o relatório em JSON
), e um conjunto não conectado nesse tempo é revertido — suas câmeras são repostas como foram encontradas — e relatado sob `errors`, nunca permanecendo conectado. `--packed` define o transporte compactado de 12 bits de cada conjunto (padrão `auto`; um projeto não salva configurações por matriz; consulte [Transporte compactado de 12 bits](#packed-12-bit-transport---packed)). Cada sensor de luz é aberto no pool de DAQ do backend em seu endereço salvo, com seu tempo de integração, média de quadros, exposição automática e limite salvos: um sensor sem limite de exposição definido recebe o valor `sunshine_cosine`; um sensor salvo sem configuração (`none`) permanece sem configuração; e um DAQ-E-S nunca recebe um limite de exposição. O painel DAQ da GUI não faz parte do conjunto: feche primeiro um sensor por lá. |
| `project capture PATH NAME [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Captura única de uma câmera ou matriz nomeada (`--prefix` por padrão, `capture`). `--orient` (matrizes): a matriz salva os dados brutos, que permanecem no formato nativo — isso registra que os produtos que o `process` gera a partir deles devem ser orientados, conforme o `array-capture --orient`. Padrão: nativo. `project burst` / `stream` e os parâmetros `capture` / `burst` / `stream` (`orient: true`) de uma receita têm a mesma opção. |
| `project burst PATH NAME [-n N] [-i S] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--prefix P] [--orient]` | Série de N quadros de uma câmera ou matriz especificada (`-n/--count` padrão 5; `-i/--interval` intervalo entre quadros em segundos, padrão 0). As sequências de matrizes eliminam a duplicação de grupos sincronizados repetidos (vigilância de obsolescência), de modo que uma matriz com ciclo parcial não possa devolver N cópias de um quadro; imprime os resultados por iteração. Uma captura que falha no meio interrompe a sequência e sai do `1`, informando quantos quadros foram salvos antes disso. |
| `project stream PATH NAME [-n N] [--fps F] [-o DIR] [--format FMT] [--exposure US] [--gain DB] [--orient]` | Captura em disco em um loop: o `--fps F` captura um segundo (padrão 3), um quadro por captura de uma câmera, um grupo de quadros sincronizados de uma matriz (com o mesmo watchdog de obsolescência que o `project burst`), até que o `-n N` capture (padrão `0` = até Ctrl+C, o que exibe quantos foram salvos e encerra o `0`, ou o `1` quando nenhum foi salvo ou houve falha no arquivo; um Ctrl+C antes de o `-n N` capturar encerra o `130`). Uma captura que falha no meio interrompe o fluxo e encerra o `1`, informando quantos quadros foram salvos antes disso. O `--poll-interval` está obsoleto e é ignorado: ele ainda é aceito para que scripts mais antigos sejam executados, com uma nota no stderr. |
| `project sensor read PATH NAME [--json]` | Quadro de espectro mais recente. |
| `project sensor log PATH NAME [--seconds SEC] [-o DIR] [--device-name NAME]` | Grava um arquivo .daq para `--seconds` (padrão 10). Uma gravação que não contenha nenhuma leitura é relatada no stderr e encerra o `1`. |
| `project run PATH RECIPE.yaml` | Executar uma receita de captura YAML/JSON
. `--dry-run` valida sem executar. |
| `project align calibrate PATH NAME [--method M] [--model M] [--frames N] [--reference SN] [--max-features N] [--ratio-threshold F] [--ransac-threshold-px F] [--min-matches N] [--max-reproj-err-px F] [--checkerboard RxC] [--name PROFILE]` | Calcula o alinhamento para uma matriz — consulte [a tabela de sinalizadores abaixo](#project-align-calibrate-options). |
| `project align status PATH NAME [--json]` | Imprime o perfil de alinhamento atual. |
| `project align clear PATH NAME` | Limpa o perfil armazenado em cache. |
| `project align tweak PATH NAME --serial SN [--dx N] [--dy N] [--rotation-deg N] [--scale N]` | Ajusta a transformação de um escravo (padrões `0`, `0`, `0`, `1.0`: sem alteração). |
| `project align export PATH NAME --to FILE` | Salvar o perfil em JSON
. |
| `project align import PATH NAME --from FILE [--no-validate]` | Carregar um perfil salvo. |

#### Opções do `project align calibrate`

| Sinalizador | Padrão | Descrição |
| --- | --- | --- |
| `--method {feature_orb, feature_akaze, phase_correlation, checkerboard, manual}` | `feature_orb` | Método de alinhamento. **Essas grafias diferem das de `lattice align-calibrate`**, que aceita as formas abreviadas `orb` / `akaze` / `phase`; os dois comandos não são intercambiáveis neste sinalizador. |
| `--model {translation, rigid, affine, homography}` | `affine` | Transformar o modelo para se ajustar. |
| `--frames N` | `1` | Instantâneos de quadros sincronizados com média. |
| `--reference SN` | a câmera mestre | Número de série da câmera de referência; todos os demais membros são distorcidos para se alinharem a ela. |
| `--max-features N` | `5000` | Limite de contagem de características ORB. |
| `--ratio-threshold F` | `0.75` | Teste de. |
| `--ransac-threshold-px F` | `3.0` | Limite de pontos válidos do RANSAC. |
| `--min-matches N` | `15` | **Limite de qualidade** — rejeitar a solução se o número de correspondências válidas for inferior a este valor. |
| `--max-reproj-err-px F` | `4.0` | **Limite de qualidade** — rejeitar a solução acima desse erro de reprojeção RMS. |
| `--checkerboard RxC` | — | Geometria da placa para `--method checkerboard`, por exemplo, `9x6`. |
| `--name PROFILE` | vazio | Nome do perfil incorporado no JSON
salvo. **Não é o nome da matriz** — esse é o `NAME` posicional. |

Os dois critérios de qualidade são a razão pela qual uma calibração pode ser bem-sucedida na resolução e ainda assim
ser recusada para salvamento: um perfil que falhar em qualquer um deles causaria, silenciosamente, um desalinhamento em todas as
capturas posteriores; por isso, ele é recusado em vez de ser salvo.

### Um projeto movido (`--relink`)

O `project.json` de um projeto nomeia cada arquivo de origem — todas as imagens e registros do sensor de luz
— por seu caminho completo. Se você mover a pasta do projeto, ou toda a pasta de projetos,
para outra unidade, esses caminhos ainda apontarão para o local antigo.

Ao abrir um projeto, o que for possível é corrigido automaticamente: um arquivo que estava dentro da pasta do projeto
é procurado na pasta em que o projeto se encontra agora, e cada caminho encontrado ali
é redirecionado e salvo (o `project.json` é primeiro copiado como
`project.json.before-relink-<date>-<time>`). Isso ocorre a cada abertura — do aplicativo,
`chloros-cli project open`, `chloros-cli process` em um projeto existente, os do SDK

e os do `chloros-mcp`. Um caminho só é redirecionado para um arquivo que esteja presente ali.

Arquivos que estavam fora da pasta do projeto — uma sessão do Hub, uma importação via “Adicionar Pasta” —
não podem ser encontrados dessa forma. Indique o nome da pasta em que eles se encontram agora:

```bash
chloros-cli project open "N:/Chloros Projects/Field_A" --relink "E:/backup/Hub Projects"
```

`--relink` considera a pasta para a qual os arquivos foram movidos, ou qualquer pasta acima dela: a pasta de cada
arquivo ausente é procurada nessa hierarquia com base nos nomes das pastas finais
(`Hub Projects/<session>/raw/<serial>`, depois `<session>/raw/<serial>`, ...), e cada
arquivos dessa pasta são obtidos do local que encontrar o maior número deles. `project open` exibe
o que foi redirecionado, quantos arquivos de origem foram encontrados e as pastas em que os demais estavam
localizados. Ainda faltam alguns: um aviso, saia do `0` — o projeto abre com o restante. Nenhum
encontrado: um erro indicando essas pastas e `--relink`, saia do `1`; o backend também
se recusa a abrir tal projeto para `process` ou `project devices --reconnect` (409 HTTP
`PROJECT_SOURCES_MISSING`).

### Exemplos

```bash
# Open a project and see what it knows about
chloros-cli project open "/home/user/Chloros Projects/Field_A"

# Its hub captures moved to another drive: point it at them
chloros-cli project open "/home/user/Chloros Projects/Field_A" --relink "/media/backup/Hub Projects"

# Connect everything saved in the project
chloros-cli project connect "/home/user/Chloros Projects/Field_A"

# Capture from a named camera (defined in cameras.json)
chloros-cli project capture "/home/user/Chloros Projects/Field_A" FrontLeft \
  -o output/ --format tiff

# Capture from a named array
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  -o output/ --format tiff

# Capture with overrides
chloros-cli project capture "/home/user/Chloros Projects/Field_A" main_rig \
  --exposure 5000

# Read a spectrum
chloros-cli project sensor read "/home/user/Chloros Projects/Field_A" Sky --json

# Record a DAQ log
chloros-cli project sensor log "/home/user/Chloros Projects/Field_A" Sky \
  --seconds 120 -o ~/Documents/spectra/

# Align an array (live)
chloros-cli project align calibrate "/home/user/Chloros Projects/Field_A" main_rig
chloros-cli project align status "/home/user/Chloros Projects/Field_A" main_rig

# Run a recipe
chloros-cli project run "/home/user/Chloros Projects/Field_A" recipe.yaml
```

### DSL de receitas

O `project run RECIPE.yaml` aceita um arquivo YAML ou JSON que descreva uma sequência de ações:

```yaml
# recipe.yaml
overrides:
  cameras:
    FrontLeft:
      exposure_us: 5000
      target_brightness: 80

stop_on_error: true
actions:
  - apply:
      name: FrontLeft
      settings:
        auto_exposure: "Off"
        gain_db: 6.0
        auto_gain: "Off"
  - wait: 2s
  - capture:
      name: FrontLeft
      output: pose_a/
      format: tiff
  - stream:
      name: main_rig
      count: 60
      fps: 5
      output: stream/
  - burst:
      name: main_rig
      count: 10
      interval: 0.5
      output: burst_a/
      format: tiff
  - sensor:
      name: Sky
      action: read
```

Ações suportadas: `apply`, `wait`, `capture`, `stream`, `burst`, `sensor`. `apply` utiliza as chaves `cameras.json`, `settings` — `exposure_us`, `auto_exposure`, `gain_db`, `auto_gain`, `target_brightness`, … (os nomes antigos `exposure_time`, `exposure_auto`, `gain`, `gain_auto` são aceitos como aliases). Uma exposição, ganho ou alvo definido dessa forma é mantido como seu: o controle automático de exposição do Chloros (incluindo a exposição medida pelo DAQ) não o altera. Uma exposição manual mantém o ganho como estava: no modo automático, a câmera continua ajustando o ganho (em toda a sua faixa) em direção ao brilho-alvo até que você defina `gain_db` ou `auto_gain: "Off"`. Um comando posterior como `auto_exposure: "Continuous"` devolve a exposição ao modo automático, e `target_brightness_auto: true` devolve um alvo mantido ao alvo automático dChloros. A ação `burst` aceita `name` (obrigatório), `count` (padrão 5), `interval` (segundos, padrão 0), `output`, `format`, e `settings` (mesma configuração por câmera que `apply`); as sequências em matriz utilizam o mesmo watchdog de grupo recém-sincronizado que `project burst`.

Um `capture`, `burst` ou `stream`, o valor padrão do `output` é `output`. Um `output` relativo — o padrão incluído — é resolvido com base na pasta a partir da qual você executa o `chloros-cli project run`, e não na pasta do arquivo de receita ou na pasta do projeto (na versão 1.2.0, ele foi colocado na pasta de instalação do backend).

Execute-o:

```bash
chloros-cli project run "/path/to/project" recipe.yaml

# Dry-run to validate without firing hardware
chloros-cli project run "/path/to/project" recipe.yaml --dry-run
```

**Cada execução é verificada primeiro em relação ao projeto.**Cada ação deve ter exatamente um dos verbos acima como chave — `- capture: {name: FrontLeft}`, e não `- action: capture` ou `- type: capture` — com um `name`, o projeto possui `cameras.json` / `sensors.json`, uma câmera ou matriz para `apply` / `capture` / `burst` / `stream` e um sensor de luz para o `sensor`; um `stream` precisa que `count` &gt; 0 e um `wait` precisa de uma duração. Uma receita com qualquer um desses erros**não** executa nenhuma ação (`[recipe] error: …`, `[recipe] not run: …`, saída `1`), mesmo com `stop_on_error: false`. As teclas que uma etapa não utiliza são um `[recipe] warning:`, assim como qualquer matriz ou sensor de luz que a receita controle e que não esteja conectado no momento. Uma receita nunca conecta dispositivos: execute `chloros-cli project connect PATH` primeiro. `--dry-run` imprime a receita, depois os mesmos erros e avisos, e `[recipe] valid: …` / `NOT valid: …`, não executa nada e encerra o `1` quando a receita apresenta um erro.

---

## Variáveis de ambiente

| Variável | Efeito |
| --- | --- |
| `CHLOROS_BACKEND_URL` | Substitui a URL do backend (padrão `http://127.0.0.1:5000`) — **respeitada apenas pelas famílias de comandos `lattice`, `project` e `daq pool-*`** (o comando `lattice network-analysis` aceita o parâmetro `--backend-url`). Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) fixam o `http://127.0.0.1:<port>` e ignoram essa variável (o literal IPv4 contorna a penalidade de Windows `localhost`→`::1` de cerca de 2 s por solicitação), de modo que sempre têm como alvo a máquina local. |
| `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED` | Definido no ambiente do backend. `1` reduz a recusa por supersubscrição — uma conexão em array com `pin_resolution`, ou um [comando de multicâmera transitória](#multi-camera-transient-sessions) (`error_code` `ARRAY_OVERSUBSCRIBED`) — para um aviso sonoro, aceitando a perda de pacotes GVSP. Apenas para testes — consulte [Modelo de fps e burst da matriz](#array-fps--burst-model). |
| `CHLOROS_ARRAY_PACKED12` | Definido no ambiente do backend: `on` ou `off` define o padrão como [transporte compactado de 12 bits](#packed-12-bit-transport---packed) para todas as matrizes (padrão: `auto`). Uma especificação explícita de `--packed` ainda prevalece. |
| `CHLOROS_DISABLE_UPDATE_CHECK` | `1` (ou `true` / `yes`) desativa a [verificação automática de atualizações](#chloros-cli-update) da CLI do Linux na inicialização. |
| `CHLOROS_CLI_MODE` | Definido pela própria CLI; instrui o backend a habilitar o processamento paralelo. |
| `CHLOROS_GVSP_PROBE_FALLBACK` | `0` ignora o teste de fallback do GVSP (apenas resultados ICMP). **Isso desativa o jumbo, não apenas silencia o log** — a câmera responde a pings DF apenas até 1500 em cada caminho, portanto, esse teste é a única forma de detectar o jumbo. Economiza ~1 s por câmera por conexão; custa ~1,45× o limite da largura de banda se a rede *pudesse* ter transportado pacotes jumbo. O SDK avisa quando você define essa opção. |
| `CHLOROS_GVSP_SESSION_REUSE` | Configure no ambiente do backend: `0` mede o tamanho do pacote a cada conexão, em vez de uma vez por câmera por sessão do backend. |
| `CHLOROS_GVSP_PACKET_SIZE_FORCE` | Fixa o tamanho do pacote GVSP em N bytes; ignora completamente a sondagem. É preferível usar por comando (`CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 chloros-cli …`) em vez de definir permanentemente: um tamanho fixado deixa de se adaptar à rede à sua frente, e fixar 9000 em um caminho que não suporta pacotes jumbo faz com que **todas** as capturas atinjam o tempo limite com `SC_ERR_TIMEOUT -1011`. |
| `CHLOROS_TRANSIENT_AE_SETTLE_S` | Defina no ambiente do backend: o número máximo de segundos que um [comando transitório de múltiplas câmeras](#multi-camera-transient-sessions), ou um `lattice capture` de uma câmera que não mantém nada, gasta ajustando a exposição de cada câmera a partir da cena anterior antes de manter um quadro (padrão `5`). `0` desativa essa função: a exposição automática de cada câmera passa a ser executada a partir do primeiro quadro, começando pelo seu limite de exposição. |
| `CHLOROS_LOG_DIR` | Definido no ambiente do backend: a pasta para a qual o log detalhado é enviado. Padrão: `%LOCALAPPDATA%\Chloros\logs` no Windows, `~/.cache/chloros/logs` no Linux (`$XDG_CACHE_HOME/chloros/logs` quando definido). O serviço raiz do .deb define `/var/log/chloros`, legível pelo grupo `adm` — consulte [Executando o backend como um serviço](#running-the-backend-as-a-service-headless). |
| `TMPDIR` (Linux) | Onde os arquivos temporários são armazenados. A CLI usa automaticamente o `/mnt/ssd/tmp`, se estiver presente. |

---

## Códigos de saída

| Código | Significado |
| --- | --- |
| `0` | Sucesso. |
| `1` | Falha genérica (a maioria dos erros de subcomandos). Um comando de captura também encerra com o código `1` quando qualquer arquivo que deveria ser gravado falha na gravação, mesmo que os demais tenham sido salvos, e com o código `array-burst` / `array-record` encerram o `1` quando o gravador relata um erro — consulte [Arquivos cuja gravação falhou](#files-that-fail-to-write). `process` também encerra o `1` quando as exportações de uma execução não puderam ser verificadas ou quando perdeu uma imagem, embora os outros produtos tenham sido gravados. |
| `2` | Erro de argumento (um sinalizador desconhecido ou ausente; também `time-sync set-priority` sem nenhuma das prioridades). |
| `130` | Interrompido por Ctrl+C: qualquer comando que seja interrompido — `lattice`, `project`, `daq` e `process` (`process` após o backend ter interrompido a execução; consulte [Notas sobre `process`](#notes)). Um comando cuja interrupção normal é por Ctrl+C — `lattice array-capture --continuous` / `--interval`, `lattice viewer`, `lattice continuous`, `multi-stream`, `align-stream`, `array-record`, `array-burst` e `project stream` com `-n 0` — encerra-se com base no que foi feito: `0` quando tudo foi gravado, `1` quando algo falhou. |
| `143` / `129` | Encerrado por SIGTERM / SIGHUP (128 + o sinal), após a execução da limpeza do comando: uma tarefa em execução foi interrompida e suas câmeras foram recolocadas, uma execução de processamento foi interrompida no backend. Uma gravação do tipo `lattice continuous` ou outra cuja parada normal seja feita com Ctrl+C também encerra o `143`, e não o `0`. |

O `lattice network-analysis` encerra o `1` para os veredictos `needs_force_slip` e `error`, e o `selftest` encerra o `1`, a menos que todas as sete verificações sejam aprovadas.

---

## Dicas para solução de problemas

- **“É necessário fazer login”** → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina.
- **“A pasta do projeto está muito profunda para este computador”** → Os caminhos longos do Windows estão desativados nesta máquina (configuração padrão do Windows), portanto, nenhum caminho pode ter mais de 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. Nada foi processado. A mensagem indica a pasta de projeto mais longa que cabe: use uma mais curta (`-o` ou `chloros-cli set-project-folder`) ou ative os caminhos longos do Windows e reinicie o Chloros. O instalador do Windows oferece essa opção como “Ativar caminhos longos do Windows”, marcada por padrão; caso contrário, use a Política de Grupo: *Configuração do Computador &gt; Modelos Administrativos &gt; Sistema &gt; Sistema de Arquivos &gt; Ativar caminhos longos do Win32*.
- **“backend inacessível”** → Inicie o aplicativo de desktop do Chloros (Windows) ou inicie o backend manualmente: `"C:\Program Files\Chloros\backend\chloros-backend.exe"` no Windows (a pasta `backend` da sua pasta de instalação; não está em `PATH`), `chloros-backend` no Linux. Ou verifique `CHLOROS_BACKEND_URL` se estiver em acesso remoto.
- **Os comandos `lattice` falham com a mensagem “Drivers da câmera LATTICE não encontrados”** → O backend não conseguiu carregar o runtime da câmera LATTICE com o qual está instalado. Todos os comandos `lattice` são executados no backend; a própria CLI não contém drivers de câmera. Reinstale o Chloros: o instalador do Windows e o .deb instalam o runtime junto com o backend.
- **“Array connect” / “Array Settings” exibe “FRAMES WILL DROP” ou “Reduza a ROI para habilitar”** → O anel de recepção da placa de rede do host é muito pequeno (geralmente reajustado para 32 após uma atualização do driver da placa de rede). Consulte [Configuração e ajuste da placa de rede do host](#host-nic-setup--tuning-lattice-arrays) — defina `ReceiveBufferLen=256`, `PendingReceives=64`.
- **A máquina trava ao reiniciar/desligar; em seguida, o WMI apresenta o código `Invalid class` / a placa de rede não é ativada / unidades USB ausentes** → Driver desatualizado do adaptador USB 10 GbE causando o código `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o driver do adaptador — consulte [Configuração e ajuste da NIC do host](#host-nic-setup--tuning-lattice-arrays).
- **Linux: as capturas pertencem ao root, portanto você não pode excluí-las, e o log do backend não está no seu `~/.cache`** → O backend é executado como root por meio do `chloros-backend.service`. Mude para o `chloros-backend@<user>.service`, que o executa como você — consulte [Executando o backend como um serviço](#running-the-backend-as-a-service-headless). Até então, o log do serviço como root está em `/var/log/chloros` (legível pelo grupo `adm`).
- **Linux: uma unidade de backend é `failed` com a mensagem “Outra instância já está em execução” (código de saída 98)** → Outro backend já ocupa a porta: apenas um pode ser executado por vez. Se duas unidades de backend estiverem habilitadas, mantenha apenas uma: `sudo systemctl disable --now <unit>` para aquela que você não usa — consulte [Executando o backend como um serviço](#running-the-backend-as-a-service-headless). Se apenas esta unidade estiver habilitada, um backend iniciado fora do systemd (por `chloros-backend`, `chloros-cli` ou um script do SDK) está ocupando a porta: interrompa esse backend e, em seguida, execute `sudo systemctl start <unit>`.
- **Aviso de swap do Jetson** → Adicione swap baseado em arquivo; a CLI exibe os comandos exatos `fallocate` / `swapon`.
- **“backend não acessível” a partir de um comando `lattice`, `project` ou `daq`** → nenhum backend está em execução: esses comandos nunca iniciam um. Execute o comando `chloros-cli backend status` e consulte [Iniciar o backend](#start-the-backend).
- **&quot;O comando `daq net ...` não está disponível no Chloros&quot;** (Linux e Jetson) → Algumas configurações do próprio DAQ-E são reservadas para o serviço MAPIR. Configurações de rede, reinicializações e atualizações de firmware são `pool-set-network`, `pool-reboot` e `pool-update-firmware`; entre em contato com o suporte da MAPIR para o restante.
- **&quot;Os comandos diretos de hardware do DAQ exigem o pacote daq completo&quot;** (Windows) → Use `pool-*`, que controla o mesmo sensor por meio do backend, ou `chloros_sdk.connect_daq_sensor()` em Python.

---

## Veja também

- [Referência do SDK do Python](sdk-reference.md) — equivalente programático de todos os comandos da CLI.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) no manual do Chloros — os sensores, seus limites e faixa calibrada, gravação, rede DAQ-E.
- [Câmeras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) no manual do Chloros — conexão, configurações da câmera, modos de captura, matrizes.
- Documentação online: `https://mapir.gitbook.io/chloros/cli`
