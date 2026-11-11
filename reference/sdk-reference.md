# Referência do Chloros Python SDK

**Versão:**

1.3.0**Gerado em:**29/07/2026 19:19 ·**Revisado em:** 06/11/2026**Pacote:** `chloros-sdk` (PyPI)**Público-alvo:** Otimizado para uso com LLM; legível por humanos.**Escopo:** Todas as classes, funções e auxiliares públicas expostas pelo `import chloros_sdk`, com exemplos que podem ser copiados e colados, abrangendo processamento de imagens, controle de câmera única, matrizes sincronizadas, sensores DAQ e automação de projetos, além do servidor `chloros-mcp` para assistentes de IA.

Se você precisar apenas dos destaques, vá para:
- [Instalação e Introdução Rápida](#instalação)
- [Smart-Connect para matrizes LATTICE](#smart-connect-para-câmeras-lattice)
- [Sessões de sensores DAQ](#sessões-de-sensores-daq)
- [Utilitários de câmera](#utilitários-de-câmera)
- [Matrizes Transitórias, Perfis de Alinhamento e Índices](#matrizes-transitórias-perfis-de-alinhamento--índices)
- [Pasta do Projeto e Receitas](#pasta-do-projeto--receitas)
- [Automação de projetos](#project-automation--chlorosproject)
- [Smart-AE / Smart-Capture](#smart-ae--smart-capture)
- [Quadros ao vivo em seu próprio pipeline](#live-frames-in-your-own-pipeline)
- [Servidor MCP (assistentes de IA)](#mcp-server-ai-assistants)

> **Alterações na versão 1.3.0** (o que um script existente da versão 1.2.0 detecta).
>
> **Alterações que um script da versão 1.2.0 pode precisar lidar:**
>
> - **Agora é gerado um erro quando o backend não confirma uma liberação.** `CameraSession.disconnect()`, `ArraySession.disconnect()` / `force_disconnect()` em um array aberto pela sessão, `DAQSensorSession.disconnect()` / `force_disconnect()` em um sensor aberto por ela, e `ChlorosProject.disconnect_all()` geram o erro `ChlorosConnectError` quando o backend recusa a liberação ou não a confirma a tempo, pois o dispositivo ainda pode estar ocupado. Na versão 1.2.0, eles retornavam silenciosamente. A saída de um bloco `with` ainda nunca gera um erro.
> - **Um dispositivo emprestado nunca é liberado.** `with attach_array(...)` e um `disconnect()` explícito nele mantêm a matriz em execução (na versão 1.2.0, sair do bloco liberava o dispositivo, mesmo quando a GUI o possuía); chame `force_disconnect()` quando quiser encerrá-lo. Da mesma forma, um `connect_daq_sensor()` que encontrou o sensor já no pool, incluindo a detecção inteligente com sinalizador zero, o mantém aberto no `disconnect()` e ao sair de seu bloco `with`.
> - **Os formatos PNG e JPEG aceitam apenas os produtos `display` e indexados.** `capture()` com `format="png"` / `"jpg"` (`ext=".png"` / `".jpg"`) e um nível `radiance` ou `reflectance`, ou `processing="all"`, gera um `ValueError` antes de qualquer solicitação; um `raw` / `debayered` de 12 bits é recusado pelo backend. Consulte [PNG / JPEG](#png--jpeg-take-the-display-and-index-products-only).
> - **`connect_daq_sensor(sensor_id=...)` abre esse sensor ou gera** o `ChlorosConnectError` listando o que encontrou; ele nunca recorre a outro sensor. Consulte [Transporte Fixado](#pinned-transport).
> - **`frame_stream()` O  gera um erro em vez de ficar em silêncio:** `FrameFetchError` O , após uma recusa, repetiria no próximo tick (uma leitura de refletância sem nenhum DAQ incluído), após 60 s sem quadro e após 60 s sem novo gatilho. Consulte [Um gatilho por leitura](#one-trigger-per-read--group_id).
> - **Um tempo limite de leitura ou uma conexão interrompida gera o erro `ChlorosConnectError`** em todas as chamadas smart-connect e o erro `analyze_array_network()`, onde a versão 1.2.0 gerava o erro de tempo limite ou de conexão subjacente.
> - **O `connect_array()` aguarda 600 s por padrão (`timeout`, 180 s na versão 1.2.0), e uma conexão que ele desiste de estabelecer é desfeita.** A chamada envia seu `timeout` para o backend, que repõe todas as câmeras como estavam e não estabelece nenhuma conexão quando a conexão não é concluída a tempo; a chamada gera o erro `ChlorosConnectError` (504 HTTP `CLIENT_DEADLINE_PASSED`). Na versão 1.2.0, uma conexão realizada logo após as câmeras serem ligadas poderia demorar mais do que a chamada e conectar a matriz após a exceção ter sido gerada. `connect_all()` aguarda cada matriz salva da mesma forma (120 s na versão 1.2.0). Consulte [Assinatura do `connect_array()`](#connect_array-signature).
> - **O `connect_array()` mantém o sensor completo.** Sem o `width` / `height` / `binning`, todas as câmeras são configuradas para o sensor completo e a taxa de quadros cai para o que o host e a rede fornecem; na versão 1.2.0 reduziu ou agrupou os quadros para manter a taxa elevada. Consulte [Matriz Sincronizada](#synchronized-array--arraysession-smart-prep).
> - **Um formato de pixel Bayer em uma câmera M3M (mono) é sua profundidade.** Uma M3M oferece apenas formatos Mono, e as predefinições nomeiam os formatos Bayer: na versão 1.2.0, a câmera rejeitou o formato e manteve o seu próprio, enquanto o `settings.pixel_format` nomeava o formato Bayer. `LatticeCamera(preset=...)`, `apply_preset()`, `set_format()`, um `CameraSettings(pixel_format=...)`, `connect_camera(preset=...)` e uma matrizformato compartilhado agora definem o formato Mono da mesma profundidade em um M3M: `"BayerRG8"` e `high_speed` resultam em `Mono8`; `"BayerRG12"`, `high_quality` e `triggered` resultam em `Mono12`; `cam.settings.pixel_format` nomeia-o. O mesmo vale no sentido inverso: um M3C nunca é colocado no formato Mono, portanto, `"Mono12"` — a partir de qualquer um desses, `connect_array(pixel_format=...)`, `set_settings(pixel_format=...)` ou o formato salvo de uma matriz — resulta em `BayerRG12` em um M3C, independentemente de qual câmera seja a mestre da matriz. Um `CameraSettings` que você passar mantém seu nome; portanto, o mesmo objeto ainda solicita a cada câmera essa profundidade em sua própria família. O `connect_camera(preset=...)` mantém o formato da predefinição (a versão 1.2.0 configura a câmera para 12 bits após aplicar a predefinição), e uma câmera aberta em uma predefinição não grava mais o que capturou no `PRESETS`. Qualquer outro formato que uma câmera recuse (que não conste em sua lista `PixelFormat`, como `RGB8` em um M3M ou `BayerRG16` em um firmware que não o possua) também não recebe um nome próprio: após uma conexão com `LatticeCamera`, `apply_settings()` ou `set_format()`, `cam.settings.pixel_format` e `packed_transport` indicam o formato que a câmera transmite, e uma linha impressa indica o formato que foi rejeitado.
> - **`analyze_array_network()` indica a qual conexão se aplica:** `pixel_format=None` (anteriormente `"BayerRG8"`) significa os formatos definidos pela conexão (Mono12 / BayerRG12), e `packed="auto"` indica a decisão de transporte compactado, de modo que os fps indicados podem diferir dos valores da versão 1.2.0'. Um `pixel_format` que você passa projeta um M3M nessa profundidade. Consulte [Auxiliar de Análise de Rede](#network-analysis-helper).
> - **`CameraSession.capture()` retorna um `CaptureResult`** (ainda uma lista de dicionários de quadros) com `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. Um resultado `ArraySession.capture()` também contém `.alignment` (informando se a distorção de alinhamento foi aplicada e, caso contrário, o motivo) e `.oriented`.
> - **As capturas de matriz são alinhadas por padrão, assim como as do aplicativo.** `ArraySession.capture()`, `continuous()`, `capture_repeated()` e `ArrayHandle.capture()` / `capture_smart()` distorcem os arquivos `debayered`, `display`, `radiance`, `reflectance` e os arquivos de índice no quadro da câmera mestre e os recortam de acordo com a sobreposição dos membros (menor que o sensor devido aos deslocamentos das câmeras, `XMP-mapir:AlignmentJSON` `"applied": true`), de modo que sejam os mesmos arquivos que a captura da mesma cena pelo aplicativo Chloros; `raw` permanece no formato nativo. Antes, uma captura sem o `aligned=True` salvava todos os arquivos no tamanho nativo, sem correção de distorção. O `aligned=False` mantém isso. Os`capture_array` e `start_capture_run` adotam o mesmo padrão. Consulte [Métodos do `ArraySession`](#arraysession-methods).
> - **`connect_all()` identifica um backend que rejeita o SDK.** Um backend em execução que esteja desconectado ou em um plano inferior ao Copper é relatado em `errors` com seu próprio motivo (HTTP 401 / 403), e não como “nenhum backend do Chloros está em execução”; além disso, cada chamada do `connect_all()` verifica o backend novamente, de modo que chamá-lo após o login funciona no mesmo objeto de projeto.
> - **Um `output_dir` relativo funciona com um backend local.** Os caminhos relativos são convertidos em absolutos em relação ao diretório de trabalho do seu script antes de serem enviados. Na versão 1.2.0, o backend os resolvia em sua própria pasta de instalação, o que resultava na falha “Acesso negado”.
> - **A chamada `numpy` agora é um requisito** do SDK (as chamadas de live-frame retornam matrizes numpy).
> - **Um nome de nível de exportação que não seja “one” é rejeitado.** `processing=` e `levels=` aceitam `raw`, `debayered`, `radiance`, `reflectance` ou `display` (ou um alias como `preview`; `processing=` e também `"all"`). Qualquer outro nome é rejeitado antes que a câmera seja acionada (400 HTTP `UNKNOWN_LEVEL`: `capture()` gera `ChlorosConnectError`, `grab()` / `frame_stream()` `FrameFetchError`); a versão 1.2.0 salvou ou retornou `raw` para isso.
> - **O recurso “Texture Aware” é rejeitado para capturas LATTICE.** Em um projeto que contém capturas LATTICE, o `configure(debayer="Texture Aware (Slow, Highest Quality)")` gera o `ChlorosConfigurationError`, assim como o `process()` após um `configure()` que solicitou por isso antes da importação. Um valor de “Texture Aware” que o projeto salvou anteriormente é definido como “Standard”, com um `UserWarning`. As capturas LATTICE sempre recebem o demosaic padrão, então a versão 1.2.0 aceitou a configuração e a ignorou; o aplicativo Chloros também não oferece essa opção para elas. Consulte [Valores suportados](#supported-values).
> - **As câmeras voltam ao estado em que foram encontradas — a menos que haja um `keep_settings=True`.** `ArraySession.disconnect()` / `force_disconnect()`, o final de cada função transitória (`capture_cameras_transient()`, `start_multi_camera_stream()`, `test_array_sync()`, `calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index()`) e a interrupção de `start_camera_continuous()` restauram cada câmera ao estado em que foi encontrada: sua fiação de disparo e linha de sincronização, seu PTP e todas as configurações da câmera alteradas durante a sessão (exposição e ganho, alvo de AE, limites e amortecimento, balanço de branco, nível de preto, formato de pixel, binning e ROI, inversões, taxa de quadros). Já o comando `keep_settings=True` salva as configurações de cada câmera na própria memória da câmera, para que ela seja inicializada com essas configurações; o comando `clear_settings=True` redefine essa memória para as configurações de fábrica. De qualquer forma, as câmeras continuam operando autonomamente. Um projeto configura suas câmeras a partir de suas próprias configurações salvas ao conectá-las. Consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings).
> - **Todos os modos salvam um projeto da mesma maneira.** Qualquer alteração em uma câmera, um conjunto de câmeras ou um sensor de luz conectado a um projeto — seja pelo SDK, pelo aplicativo, pela CLI ou pelo servidor MCP — é salva no arquivo `cameras.json` / `sensors.json`, e ao abrir o projeto em qualquer modo, os dispositivos são configurados a partir dele; um `ChlorosProject` nomeia sua própria pasta no backend. Antes, apenas o aplicativo salvava o projeto. Consulte [Um registro de projeto, todos os modos](#one-project-record-every-mode).
> - **Arquivos criados a partir de várias câmeras indicam cada uma delas.** Uma pilha multibanda, um índice ao vivo criado a partir das bandas de várias câmeras e a imagem de uma captura em matriz `composite/` contêm o número de série, o modelo e o pacote de calibração de cada câmera de origem como listas (`XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceCalBundleSha256s`, …), e as listas EXIF `SerialNumber` listam os números de série separados por vírgulas. Anteriormente, elas não continham nenhuma informação sobre a câmera ou apenas o número de série da primeira câmera. Consulte [Arquivos criados a partir de várias câmeras](#files-made-from-several-cameras).
> - **Apenas um arquivo `raw` contém as tags de calibração da câmera.** O conjunto compatível com MicaSense (`BlackLevel`, `RadiometricCalibration`, os conjuntos`Band{R,G,B}RadiometricCalibration` por canal, os conjuntos `MicaSenseVignetting*`) e a carga útil `CalibrationJSON` descrevem os valores brutos do sensor; portanto, eles são incluídos apenas no arquivo de captura `raw`. Todos os demais arquivos — um arquivo de captura`debayered`, `display`, `radiance`, `reflectance` e arquivos de índice, quadros de gravação contínua e de fluxos de múltiplas câmeras— e todos os produtos `process()` — contém a identidade da câmera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e, em uma captura, os próprios dados do quadro (`XMP-mapir:FrameJSON`: exposição, ganho, temperatura, formato de pixel, hora da captura). Na versão 1.2.0, eles também incluíam o conjunto de dados brutos, que uma ferramenta de processamento (Pix4D, Metashape) aplicava aos pixels que não eram valores brutos. O Processing identifica a calibração desse arquivo por meio de seu número de série e do `CalBundleSha256`.
>
> **Outras alterações:**
>
> - **A taxa de quadros automática de um array se ajusta ao que o array fornece.** Uma taxa que perde mais de 1% de seus disparos é reduzida, avaliada com base em várias observações prolongadas em vez de apenas uma, e a taxa da qual ela foi reduzida é tentada novamente de tempos em tempos. Uma taxa definida pelo usuário permanece inalterada.
> - **A exposição automática está mais estável.** Ela se mantém constante sob iluminação estável, se estabiliza em segundos após uma mudança, ignora pequenos destaques e faz com que a câmera volte a funcionar quando ela para de responder sob sol forte. Veja [Como a exposição automática se comporta](#how-auto-exposure-behaves).
> - **O `import_images(recursive=True)` conta apenas suas imagens e gravações.** Uma pasta que contenha o `camera_cal/` de um projeto não tem mais esses arquivos de calibração contabilizados como imagens e gravações, e um projeto que já os tenha incluído os descarta.
> - **`configure(radiometric_output=...)` se aplica.** Agora, ele define os produtos `Export radiance` / `Export reflectance` do projeto; antes, as configurações de produto específicas de cada projeto prevaleciam sobre ele, de modo que o `radiance`, o `sensor-response` e o `all` exportavam radiação e refletância.
> - **O `test_array_sync()` é acionado a partir do estado de ligação das câmeras.** Ele ativa o PTP das câmeras para a execução, aguarda que seus relógios sejam sincronizados e estabilizados antes do primeiro disparo (`ptp_lock`) — cada câmera fica sincronizada por 120 s, cerca de 2 minutos a partir da ativação (`ptp_settle_s=` define isso), já que um relógio recém-sincronizado apresenta um desvio de até ~200 us por cerca de 2 minutos — e restabelece o PTP de cada câmera como estava. Uma câmera ainda sincronizada de um teste anterior não é aguardada novamente. Ele costumava deixar o PTP desativado em uma rede com um único switch, na qual nenhum DAQ-E respondia, e lia o relógio de funcionamento livre de cada câmera como um relógio PTP que não se sincronizava (“PTP não está sincronizado”, a cada execução).
> - **Uma gravação `.daq` indica se sua tampa foi selecionada ou presumida.** O DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e o Chloros não consegue detectar se há uma instalada; portanto, um sensor conectado sem uma tampa especificada recebe o nome `sunshine_cosine` por padrão, uma *suposição*. Cada registro do Chloros (esquema 1.30) agora armazena o `cap_id_source` ao lado do `cap_id`: `operator` quando você nomeou uma tampa (incluindo o `none`), `auto_default` quando era o padrão. O processamento de um registro cujo capacitor foi presumido, com o capacitor DAQ do projeto deixado em “Auto”, gera um `[daq-dls] WARNING` nomeando o arquivo: um DAQ-U sem filtro, sob a correção de luz solar, apresenta uma leitura cerca de 30 vezes maior. **Nomeie o filtro que você instalou** ao conectar (`connect_daq_sensor(cap_id=...)` ou `DAQSensorSession.set_cap()`, `chloros-cli daq pool-connect --cap-id` / `daq pool-set-cap`, ou a lista suspensa de capacitores da GUI), ou defina o capacitor do DAQ do projeto, e o aviso desaparecerá.
> - **Não há mais janelas de console pip no Windows.** Em um PC com o Python instalado, o backend e cada um de seus processadores de imagem executavam o `pip` uma vez na inicialização, e cada um abria uma janela de console. Isso não acontece mais.
> - **Gravações ao vivo do `.daq` sobrevivem a uma queda de energia.**Uma gravação é gravada com um diário e uma sincronização completa do disco a cada commit, por trás do sensor; assim, um disco lento atrasa o arquivo, mas nunca as leituras. Uma gravação interrompida no meio de um commit (queda de energia, encerramento forçado) pode deixar o `<name>.daq-journal` ao lado dela:**copie os dois juntos.** O Chloros reverte o diário na primeira vez que lê o arquivo, de modo que a gravação é aberta com tudo que foi confirmado antes da interrupção (cerca dos últimos 3 s são perdidos); na versão 1.2.0, uma queda de energia no momento errado poderia deixar o arquivo inteiro ilegível (“a imagem do disco do banco de dados está corrompida”).
> - **O `DAQSensorSession.record_start()` em um disco ocupado não bloqueia mais o sensor**: as leituras são enfileiradas e gravadas em uma thread própria, e aquela que não pôde ser gravada é contabilizada no `status()` do `record_errors` (junto com o `record_last_error`).
> - **`LatticeCamera` direto:**um quadro sem exposição/ganho por bloco agora relata a exposição e o ganho**reais** da câmera (o mecanismo de fallback lia um valor armazenado em cache que poderia permanecer congelado durante toda uma execução de exposição automática), e o `apply_settings(trigger_source='LineN')` / `set_trigger_source('LineN')` aplicam a mesma correção de ruído de entrada de 5 µs que os escravos de cabo do Chloros recebem (sem filtragem, um escravo disparava quadros sem que nenhum gatilho fosse enviado).
> - **Câmeras acionadas por software não ignoram mais um em cada dois sinais de acionamento.** Uma câmera no modo de acionamento por software agora enfileira um sinal de acionamento que chega durante sua leitura (ou o Chloros aguarda até que ela esteja pronta), assim como as câmeras sincronizadas por cabo já faziam.
> - **O `discover_daq_sensors()` nomeia um DAQ-E-S como `DAQ-E-S`.** Seu `model` era `DAQ-E` para cada sensor Ethernet; aquele que não responder à verificação de identidade em cerca de 2 s ainda é listado, como `DAQ-E`.
> - **Novo: `chloros-mcp`, um servidor MCP para assistentes de IA.** O Claude Desktop, o Claude Code, o Cursor e outros clientes MCP podem processar imagens (criar um projeto, importar capturas, escolher configurações, processar, gerar relatórios sobre o que foi exportado) e operar câmeras LATTICE, matrizes e sensores de luz DAQ: conectar, visualizar, definir exposição e cor, capturar, gravar. Suas ferramentas abrangem o que este SDK faz. Ele é instalado junto com o SDK, e o pacote .deb para Linux o coloca em `/usr/bin`. Consulte [Servidor MCP (Assistentes de IA)](#mcp-server-ai-assistants).
> - **Novo: utilitários de câmera**, os comandos `chloros-cli lattice` como chamadas do SDK: perfil de cor, saturação/contraste, realces, balanço de branco (único, automático, proporções fixas), predefinições, modelos de filtro, status e atualização do firmware, energia (verificação, suspensão, ativação), calibração do alvo, adaptadores de rede (verificação, estimativa, correção), correção de vinheta em arquivos, leitura DLS e verificação do host. Consulte [Utilitários da Câmera](#camera-utilities).
> - **Novo: o restante da CLI como chamadas SDK**, de modo que a CLI, o SDK e o `chloros-mcp` oferecem os mesmos recursos (a CLI mantém apenas o login, o provisionamento de um DAQ-E e os servidores DAQ diretos, por motivos de segurança): câmeras abertas por um único comando (`start_camera_continuous()`, `start_multi_camera_stream()`, `capture_cameras_transient()`, `test_array_sync()`, `detect_array_master()`, `analyze_array_models()`), arquivos de perfil de alinhamento (`calibrate_alignment_profile()`, `capture_aligned_multiband()`, `start_aligned_stream()`, `alignment_profile_info()`, `reorder_alignment_profile()`), índices (`index_catalog()`, `compute_index()`, `capture_live_index()`), o grandmaster do PTP (`time_sync_status()` e similares), a pasta do projeto (`project_folder()`, `set_project_folder()`, `reset_project_folder()`) e as receitas (`run_project_recipe()`, o executador próprio da CLI). Para sensores de luz: `connect_daq_sensor(cap_id=...)`, `DAQSensorSession.set_cap()`, `daq_bindings()` / `set_daq_bindings()`, `daq_band_presets()` e `light_sensor_reflectance()` (refletância e índices ambientais + de objeto de dois sensores); para uma câmera: `CameraSession.status()` e `capture_repeated()`; para um conjunto: `ArraySession.capability()` e `set_member_settings()` (exposição, ganho, etc., de um membro). Consulte [Matrizes Transitórias, Perfis de Alinhamento e Índices](#transient-arrays-alignment-profiles--indices), [Pasta do Projeto e Receitas](#project-folder--recipes) e [Sessões de Sensores DAQ](#daq-sensor-sessions).
> - **Produtos de índices LATTICE aprimorados** (NDVI e outros índices) do `process()`, em todos os modos de processamento.
> - **Câmeras OCN do LATTICE: produtos de radianância, refletância e índices aprimorados sob luz solar.** Uma correção de frota é aplicada após a separação de bandas — em tempo real, nas capturas e no `process()`; as câmeras RGN permanecem inalteradas. Consulte [Saída radiométrica](#radiometric-output-lattice-multispectral-pipeline).
> - **Cada captura salva a leitura do DAQ utilizada, assim como a GUI.** `CameraSession.capture()` e `ArraySession.capture()` / `continuous()` gravam o `.daq` e seu `.match.json` sempre que um DAQ é vinculado ou agrupado, incluindo uma captura apenas em formato bruto; na versão 1.2.0, uma captura bruta ou sem debayering era salva apenas com o `force_daq=True`. O `daq=False` desativa isso e o `daq_tolerance_ms=` define a tolerância de correspondência; ambos os resultados indicam o que foi salvo no `.daq_files` e `.daq_matches`. Agora, uma captura aguarda a próxima leitura do DAQ para que possa combinar as leituras de ambos os lados da exposição. Consulte [A leitura do DAQ que uma captura salva](#the-daq-reading-que-uma-captura-salva--arquivos-da-DAQ-e-correspondências-da-DAQ).
> - **`ArraySession` possui quadros em tempo real e uma captura contínua no lado do servidor.** `grab()` e `frame_stream()` retornam quadros numpy de um gatilho (a mesma leitura que as APIs do projeto `ArrayHandle.grab()` / `frame_stream()`), e o `continuous()` salva cada grupo sincronizado no backend e retorna um [`ContinuousHandle`](#continuoushandle). Consulte [Frames em tempo real em seu próprio pipeline](#live-frames-in-your-own-pipeline).
> - **Um array `grab()` é vinculado a um gatilho** (`ArrayHandle.grab()` e o novo `ArraySession.grab()`). Na versão 1.2.0, 12 das 20 câmerascâmeras `ArrayHandle.grab()` abrangiam dois gatilhos. Agora, cada quadro contém `group_id`, e uma matriz `frame_stream()` gera cada gatilho uma vez. Consulte [Um disparador por leitura](#one-trigger-per-read--group_id).
> - **Quadros por banda e alinhados:** `grab()` / `frame_stream()` recebem `band=`, `align=`, `bit_depth=` e `refl_max=` em identificadores de câmera e matriz. Consulte [Exportando uma única banda](#exporting-a-single-band--band).
> - **Novos métodos e argumentos do `ArraySession`:** `set_target_fps()`, `force_disconnect()`, `capture(after_seq=..., orient=...)` (com `CaptureResult.group_seq` e `.failed`), `continuous()`, `connect_array(packed=...)`; `capture_repeated()` encadeia grupos consecutivos e para quando a unidade estiver cheia.
> - **`ArraySession.record()` grava sem a GUI aberta.** Com a visualização combinada da GUI fechada, o backend renderiza a própria visualização: oNDVI ou o índice que você passa como `expr=`. `RecorderHandle.stop()` gera uma exceção `ChlorosConnectError` para uma gravação que falhou, na qual nenhum quadro foi gravado, e mantém o resultado da interrupção em `.result` (deixar um bloqueio em `with` não gera exceção: verifique `rec.result`). Consulte [`RecorderHandle`](#recorderhandle).
> - **Novos métodos `DAQSensorSession`:** `enable_ptp()`, `force_disconnect()` e, para as configurações próprias do DAQ-E, `device_info()`, `set_network()`, `set_auto_stream()`, `reboot()` e `update_firmware()`. `record_stop()` retorna `warning` e `gaps`. Uma reinicialização que encerrou uma gravação nomeia-a em sua resposta ou, quando a reinicialização falha, no `ChlorosConnectError` (`recording_closed`; consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings)). Consulte [Métodos do `DAQSensorSession`](#daqsensorsession-methods).
> - **Um DAQ-E em pool que para de transmitir é reconectado pelo backend**, com as configurações que possuía, e uma gravação nele continua no mesmo `.daq` com a lacuna marcada no arquivo; O `record_stop()` retorna as lacunas como `gaps`. Na versão 1.2.0, o sensor permanecia inativo até ser desconectado e reconectado, o que encerrava a gravação. Consulte a seção [Um DAQ-E que para de funcionar](cli-reference.md#a-daq-e-that-stops-delivering).
> - **O `open_project()` funciona a partir de um SDK instalado.** O wheel (o instaladore do PyPI) agora contém os carregadores de projeto necessários; antes, o `open_project()` gerava o `ImportError`, nomeando o `project_cameras` / `project_sensors` em qualquer SDK instalado via pip. O `connect_all()` conecta as câmeras, matrizes e sensores de luz salvos por meio de um backend em execução (sensores de luz por meio de seu pool de DAQ), e um dispositivo salvo que ele não tem como localizar é listado em `errors` com o motivo, em vez de ser ignorado sem qualquer aviso. Uma matriz salva que já esteja ativa no backend (da GUI, ou do `chloros-cli project connect`) é conectada e mantida em execução, enquanto antes era relatada como um erro (“A câmera  <sn>já está na matriz<id>”). Consulte [Métodos do `ChlorosProject`](#chlorosproject-methods).
> - **`ChlorosLocal()`, `process_folder()` e `process_lattice_capture()` localizam o Chloros no Windows.** Eles procuram onde o instalador coloca o backend (`<install folder>\backend\`, `C:\Program Files\Chloros\backend\` por padrão, uma pasta personalizada a partir do próprio registro do instalador) e se conectam a um backend que já esteja em execução, sem precisar procurar por um. Na versão 1.2.0, eles exibiam a mensagem “Backend do Chloros não encontrado” em uma máquina instalada, a menos que você passasse o `backend_exe=`.
> - **Um backend que encerra na inicialização é relatado imediatamente.** Quando o `ChlorosLocal()` ou uma chamada de conexão inteligente inicia um backend que encerra antes de responder, a chamada gera, em questão de segundos, o código de erro `ChlorosBackendError` proveniente de `ChlorosLocal` e `ChlorosConnectError` proveniente das chamadas smart--connect, com o código de saída, o comando executado e as últimas linhas exibidas pelo backend. A versão 1.2.0 aguardava o tempo limite de inicialização de 60 s e não informava o motivo. Consulte [Inicialização automática do backend](#backend-auto-start).
> - **O backend continua em execução entre os scripts.** Ao sair de um bloco `with ChlorosLocal(...)`, `process_folder()` / `process_lattice_capture()` retornando e `chloros-mcp` saindo não interrompe mais um backend iniciado pelo SDK; o próximo script o reutiliza. Ele é interrompido no `logout()` (a menos que o aplicativo Chloros esteja aberto ou uma execução esteja em andamento), no `shutdown_backend()` ou automaticamente após 30 minutos sem que nada o esteja utilizando (o `CHLOROS_BACKEND_IDLE_MIN` no ambiente do backend altera isso; `0`: nunca).
> - **Um backend iniciado pelo SDK é solicitado a ser desligado, não encerrado à força.** `shutdown_backend()` permite que o backend libere suas câmeras e sensores de luz e conclua qualquer gravação; ele só é encerrado se ainda estiver em execução 10 s depois. A versão 1.2.0 o encerrava imediatamente (no Windows, um encerramento forçado), o que poderia deixar um sensor de luz DAQ conectado a esse backend sem resposta. Um backend que o SDK encontrou em execução continua sendo ignorado.
> - **No Linux, a inicialização automática nunca executa um `backend_server.py` a partir da pasta atual.** Sem nenhum backend instalado, um script iniciado em uma pasta que contivesse um o executava; agora, apenas o backend instalado é iniciado (ou, para o `ChlorosLocal`, aquele nomeado pelo `backend_exe=`).
> - **`configure(indices=...)`**adiciona cada predefinição ao projeto para as faixas de sua câmera, incluindo um novo projeto, e exibe um aviso (`UserWarning`) caso não seja possível adicionar um nome.**`import_images()`** relata as gravações `.daq` como `scan_count` e não avisa mais sobre uma pasta de captura LATTICE.
> - **O `read_image_audit_tags()` funciona a partir de uma instalação via pip.** Ele lê os metadados da imagem por conta própria; na versão 1.2.0, ele retornava o `{}` sem os componentes internos da instalação para desktop no caminho do Python.
> - **O `capture()`, logo após o `connect_array()`**, aguarda até 20 s pelo primeiro grupo de quadros da matriz, em vez de apresentar falha com o código HTTP 503 “Matriz ainda em aquecimento”.
> - **`status()` relata uma paralisação.** O novo campo `stalled` fica como True enquanto o loop de captura é executado, mas não publica nada, e `actual_fps` fica então igual a 0. Na versão 1.2.0, `actual_fps` mantinha seu último valor durante uma paralisação; portanto, uma matriz paralisada exibia o valor `actual_fps > 0`, enquanto as capturas falhavam com o erro 503 “Nenhum grupo de quadros sincronizado recente”.
> - **Menos conjuntos de quadros incompletos e erros 503 de captura:** a entrada de disparo de cada câmera escrava agora está livre de interferências.
> - **Transporte compactado de 12 bits** (`connect_array(packed="auto")`) aumenta as taxas de quadros da matriz de sensor completo em cerca de um terço. Consulte [Transporte compactado de 12 bits](#packed-12-bit-transport-packed).
> - **Exposição da matriz em uma ROI:** uma ROI pequena não limita mais a exposição de um membro ao período de funcionamento livre do sensor (4,9 ms em 768 × 304). Consulte [Exposição nos membros da matriz](#exposure-on-array-members).
> - **A exposição automática não reduz mais a taxa de quadros automática de uma matriz.** Uma matriz sincronizada por cabo opera na taxa permitida pela rede e pela leitura das câmeras; a exposição automática preenche o período que essa taxa permite e, em seguida, aumenta o ganho. Uma exposição manual mais longa que o período diminui a velocidade do disparador para se adequar a ela, em vez de ser rejeitada. Consulte [Exposição nos membros da matriz](#exposure-on-array-members).
> - **`read_nodes()`**adiciona `frame_values` e `notes` enquanto a exposição automática está em execução (consulte [`read_nodes()` sob exposição automática](#read_nodes-under-auto-exposure--frame_values-and-notes)). Quando a câmera rejeita uma exposição ou ganho,**`set_settings()`** indica o intervalo que ela aceita e reativa o modo automático que havia sido desativado para esse valor; `set_settings(target_brightness_auto=True)` é novo.
> - **`orient=`** em `ArraySession.capture()` / `capture_fastest()` / `continuous()` / `burst()` e `ArrayHandle.capture()` / `capture_smart()` gira os produtos de acordo com a orientação da câmera. Consulte [Métodos do `ArraySession`](#arraysession-methods).
> - **O `DLS()` requer o próprio `mac_address` do sensor.** Não há mais um MAC padrão (era o endereço de uma unidade); o `DLS()` sem um gera o `DLSError`. Consulte [magnitudes da banda DAQ-M](#daq-m-band-magnitudes--dlsband_magnitudes).
> - **O instalador do Windows** oferece as opções “Habilitar caminhos longos do Windows” e “Excluir o backend do Chloros da verificação do Microsoft Defender”, ambas marcadas por padrão. Consulte [Etapa 1](#step-1--install-the-chloros-platform-package).
> - **O .deb do Linux** instala o Python SDK para o sistema `python3` (antes, ele apenas colocava o wheel no disco), aumenta os buffers de recepção de soquetes do kernel necessários para os fluxos de câmera e adiciona um serviço de backend que é executado como seu usuário. O pacote Jetson é `chloros_1.3.0_arm64_jp6.deb`, para o JetPack 6, com o PyTorch CUDA do JetPack. Consulte [Passo 1](#step-1--install-the-chloros-platform-package) e [Passo 2](#linux-deb).

---

## Arquitetura em 60 segundos

O SDK é uma camada fina de Python sobre o backend do Chloros (o mesmo servidor Flask que a GUI de desktop e a CLI utilizam). Para automação, você importa o `chloros_sdk` e chama métodos de alto nível; nos bastidores, cada chamada se transforma em uma solicitação HTTP para o backend local na porta 5000 — `http://127.0.0.1:5000/api/...` (deliberadamente não `localhost`, que resolve para `::1` primeiro no Windows e leva cerca de 2 s por solicitação em um backend apenas IPv4). O backend é responsável pelo conjunto de recursos de hardware — câmeras, sensores DAQ, perfis de alinhamento, buffers de quadros — para que os scripts do SDK possam coexistir com a GUI sem disputar portas seriais ou largura de banda da placa de rede.

Existem três interfaces que você utilizará:

1. **`ChlorosLocal` + funções livres** (`process_folder`, `process_lattice_capture`) — Pipeline de processamento de imagens. Execute uma pasta inteira passando pelos processos de calibração, debayer e exportação de índice a partir de uma única chamada em Python.
2. **Manipuladores Smart-connect** (`connect_camera`, `connect_array`, `connect_daq_sensor`) — Abra uma sessão de backend persistente para hardware ativo. O mesmo fluxo “smart-prep” da GUI: sonda de rede, seleção automática de camada, PTP, inicialização de AE, configuração de acionamento por GPIO.
3. **`ChlorosProject` / `open_project`** — Carrega um projeto salvo (pasta com `cameras.json` + `sensors.json` + `project.json`), conecta suas câmeras, matrizes e sensores de luz de uma só vez e aciona as capturas com identificadores nomeados.

As superfícies 1 e 2 **iniciam automaticamente um backend local** caso ainda não haja um em execução (o mesmo binário incluído que a GUI e a CLI executam) — assim, um script simples funciona a partir de um shell novo sem que você precise iniciar um backend primeiro: `ChlorosLocal` (e `process_folder()` / `process_lattice_capture()`), `connect_camera()`, `connect_array()`, `connect_daq_sensor()`, `discover_lattice_cameras()` e `discover_daq_sensors()`. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` nunca são iniciados: eles precisam de um backend em execução. Passe `auto_start_backend=False` para desativar (por exemplo, ao apontar para um backend remoto, que nunca é iniciado). Consulte [Inicialização automática do backend](#backend-auto-start). O Surface 3 se comporta de maneira diferente: o `open_project()` não aceita o parâmetro `auto_start_backend`, e o `connect_all()` nunca inicia um backend — ele tenta acessar o `http://127.0.0.1:5000` uma vez e, se não houver resposta, recorre ao controle direto (sem backend) do dispositivo `lattice_sdk`. Isso requer o runtime do SDK da Arena e o OpenCV (`cv2`) importável em seu Python, o que a roda do SDK não instala; sem eles, cada câmera e matriz salvas são relatadas como um erro, com o motivo. Os sensores de luz de um projeto não têm um caminho sem backend: sem nenhum backend em execução, cada sensor salvo é relatado como um erro, com o motivo (consulte [`SensorHandle`](#sensorhandle)). Apenas o `proj.process()` e o `stream(..., overlays=True)` constroem de forma preguiçosa um `ChlorosLocal()` (que inicia automaticamente).

Todos os três são auth: execute o `chloros-cli login` uma vez na máquina ou faça login pela interface gráfica da área de trabalho. O que uma chamada sem uma sessão válida gera depende da interface (consulte [Exceções](#exceptions)): o `ChlorosAuthenticationError` a partir do `ChlorosLocal` (e do `process_folder()` / `process_lattice_capture()`), `ChlorosConnectError` proveniente das chamadas smart-connect e suas sessões, `status: "error"` com um `error_code` proveniente de `analyze_array_network()`, e uma entrada sob `errors` no relatório `connect_all()`.

Requisitos:
- Python 3.7+ (conforme declarado pelo pacote; desenvolvido/testado na versão 3.10)
- Chloros Desktop instalado localmente (o binário de backend vem incluído no instalador)
- Login ativo no Chloros+ em **qualquer plano pago do Chloros+ (Copper ou superior)**; o plano gratuito**Iron**não tem acesso a SDK/CLI. Isso é imposto**pelo lado do servidor**:** toda solicitação marcada com SDK/CLI deve conter tanto uma sessão ativa quanto um plano pago. Um plano abaixo do Copper recebe o código de erro `403` com `error_code: PLAN_UPGRADE_REQUIRED` (`ChlorosLicenseError` a partir do `ChlorosLocal`, e `ChlorosConnectError` para chamadas smart-connect). Um chamador desconectado recebe `401` / `AUTH_REQUIRED` em vez disso (`ChlorosAuthenticationError` de `ChlorosLocal`, `ChlorosConnectError` das chamadas do Smart-Connect) — os dois são distintos porque a reexecução de `chloros-cli login` corrige o `401`, mas não consegue corrigir o `403`, que requer uma atualização do plano.
- O uso offline é permitido dentro do período de carência do plano: o nível de acesso é lido do cache de validação do servidor (5 minutos) ou do cache de licença assinada e vinculada à máquina (30 dias para planos mensais; até o vencimento da assinatura para planos anuais). Quando esse período de carência expira, o plano passa para a versão gratuita e o acesso ao SDK/CLI é interrompido até que a máquina consiga se conectar ao servidor pelo menos uma vez. O erro `chloros-cli status` (`GET /api/license-status`) permanece acessível no nível gratuito, de modo que o motivo fica visível.
- Windows 10/11 de 64 bits, **Ubuntu 22.04 LTS ou mais recente**, ou Jetson (JetPack 6). O Ubuntu 20.04**não** é compatível: as dependências do `.deb` derivam do que o backend vincula, incluindo o `libc6 (>= 2.35)`, e o Focal vem com a glibc 2.31.

---

## Instalação

O SDK do Python é uma camada fina de Python sobre o backend Chloros. A menos que você execute um backend em outra máquina (consulte [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel)), é necessário ter o **pacote Chloros para desktop instalado localmente** (instalador para Windows ou Linux `.deb`), que inclui os sensores de luz DAQ. O pacote fornece o binário do backend, o tempo de execução do SDK Arena para câmeras LATTICE e os pacotes de calibração.

Downloads mais recentes: [`https://mapir.gitbook.io/chloros/download`](https://mapir.gitbook.io/chloros/download)

### Etapa 1 — Instale o pacote da plataforma Chloros

#### Windows (.exe)

1. Baixe o `Chloros Installer 1.3.0.exe` na página de downloads.
2. Execute o instalador e siga as instruções do assistente. O caminho padrão de instalação é `C:\Program Files\Chloros\`.
3. Inicie o Chloros pelo menos uma vez e faça login com sua conta Chloros+.

O instalador permite que o backend (`chloros-backend.exe`, e nada mais) atravessar o Firewall do Windows em todos os perfis de rede: entrada UDP, para transmissões de câmera, PTP, mDNS e a transmissão do DAQ-E, e entrada TCP apenas da sub-rede local, para a conexão que um DAQ-E estabelece de volta ao computador durante uma atualização de firmware ([`update_firmware()`](#daq-e-device-settings)). O desinstalador remove ambas as regras. A API HTTP do backend permanece no `127.0.0.1` de qualquer maneira.

O grupo **Configurações do sistema** do assistente possui duas opções, ambas marcadas por padrão:

| Opção | O que faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** | Permite que as pastas de projeto fiquem em níveis mais profundos do que o limite de 260 caracteres do Windows. Com os caminhos longos desativados (padrão do Windows), uma pasta de projeto muito profunda para seus produtos é rejeitada antes que qualquer coisa seja processada — consulte [Solução de problemas](#dicas-de-solução-de-problemas). | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender** | Impede a verificação em tempo realde cada arquivo gravado por uma captura de câmera. Em altas taxas de captura, a verificação consome cerca de 30% da taxa: medido em ROI 512 em um conjunto de 9 câmeras, a captura contínua em formato raw atingiu 52,7 capturas/s com a verificação e 77,7 capturas/s com o backend excluído. | `/MERGETASKS="!defenderexclusion"` |

Uma captura contínua relata a carga do Defender como `defender_cores` (consulte [`ContinuousHandle`](#continuoushandle)).

#### Linux amd64 (.deb)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Instale com o `apt install ./…`, e não com o `dpkg -i`: o apt também instala os pré-requisitos do Python SDK, que o pacote recomenda em vez de exigir (consulte [Etapa 2](#linux-deb)).

#### Linux arm64 — Jetson (JetPack 6)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
chloros-cli login user@example.com 'YourPassword'
```

Para Jetson Orin / Orin NX / Orin Nano no JetPack 6 (L4T R36). Seu PyTorch é a versão CUDA do JetPack 6, portanto, o processamento por GPU é executado na GPU do Orin; ele utiliza as bibliotecas CUDA e cuDNN que o JetPack 6 instala (`sudo apt install nvidia-jetpack` em uma imagem que não as possua).

#### Linux: buffers de recepção e o serviço de back-end

Ambos os pacotes .deb aumentam para 32 MB os buffers de recepção de soquetes do kernel necessários para os fluxos das câmeras LATTICE (`/etc/sysctl.d/60-chloros-gige.conf`). Na configuração padrão do kernel, os pacotes da câmera são descartados no kernel e um array perde quadros e taxa de quadros; o `chloros-cli lattice network` avisa quando os buffers estão baixos. Consulte a seção [Hosts Linux — buffers de recepção de soquete](cli-reference.md#linux-hosts--socket-receive-buffers) da referência da CLI.

Em uma máquina sem monitor, execute o backend como um serviço **como seu usuário**, para que um script encontre um em execução (o `connect_all()` nunca inicia um backend):

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
```

O backend então é executado como você: as capturas que ele grava são suas, e ele usa seu `chloros-cli login`. O `chloros-backend.service` do root ainda está instalado para máquinas que o habilitaram, mas todos os arquivos que um backend do root grava pertencem ao root. Consulte a seção [Executando o backend como um serviço](cli-reference.md#running-the-backend-as-a-service-headless) da referência da CLI.

Um backend executado pelo serviço é compartilhado e nunca é interrompido por um cliente: os scripts do SDK e o `chloros-cli` o utilizam conforme o encontram (o `ChlorosLocal.shutdown_backend()` interrompe apenas um backend que o próprio `ChlorosLocal` iniciou), e o `GET /api/status` nomeia a unidade como `service`. Reinicie-o com o `sudo systemctl restart chloros-backend@$USER.service`. A atualização do pacote interrompe os serviços de backend durante a instalação, portanto, um script em execução perde suas câmeras e sensores e, em seguida, reinicia todos os que estão habilitados; a remoção do pacote os interrompe e desabilita os que estão habilitados. Apenas um backend é executado por vez; portanto, habilite apenas uma unidade: uma segunda unidade se recusa a iniciar e permanece como `failed` (código de saída 98, não reiniciado), e seu `systemctl status` explica o motivo. Para alternar entre unidades, desative aquela que você não estiver usando.</id></sn>

### Etapa 2 — Instalar o Python SDK

**O instalador do Chloros inclui um pacote “wheel” do SDK correspondente.** Todo instalador do Windows e todo pacote .deb do Linux instala no disco um `chloros_sdk-X.Y.Z-py3-none-any.whl` que corresponde exatamente à versão da GUI, da CLI e do backend. Você não precisa ficar acompanhando o PyPI para manter a sincronização.

#### Windows

O instalador executa automaticamente o `pip install` com o pacote “wheel” incluído, utilizando o Python do seu sistema (o executável `py.exe` é preferencial, mas, se não estiver disponível, ele recorre ao `python -m pip`). Não é necessária nenhuma ação — o `import chloros_sdk` funciona no seu ambiente Python após uma instalação bem-sucedida. Se não houver Python instalado no sistema, o instalador ignora essa etapa silenciosamente, e a GUI + CLI continuam funcionando.

#### Linux (.deb)

Ambos os pacotes .deb (x86_64 e Jetson) instalam o SDK para o sistema `python3`, de modo que o `import chloros_sdk` funciona após a instalação, sem necessidade de pip nem conexão de rede. Os pacotes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e os carregadores de projeto) vão para o `/usr/lib/python3/dist-packages`, e o comando `chloros-mcp` do SDK (o [servidor MCP](#mcp-server-ai-assistants)) no `/usr/bin`, todos pertencentes ao pacote: uma atualização os substitui e a remoção do pacote os remove.

Os próprios pré-requisitos do SDK, `python3-numpy` e `python3-requests`, são **recomendados, não obrigatórios** (a GUI, a CLI e o backend são independentes): O `sudo apt install ./chloros_<version>_amd64.deb` (`…_arm64_jp6.deb` em um Jetson) os instala, enquanto o `dpkg -i` não. A saída da instalação informa a versão do SDK que o `python3` importa, quaisquer dependências que estejam faltando, e um `chloros_sdk` em outro local no caminho do `python3` que substitua a cópia do pacote (como um instalado anteriormente com o `pip install --user`).

Em um virtualenv, que não tem acesso aos pacotes do sistema, instale o wheel incluído nele:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para implantações do Jetson em ambiente isolado (air-gapped), isso ocorre totalmente offline — o SDK e o wheel já estão no disco.

#### PyPI público

Para hosts que utilizam apenas o pip (sem o pacote de desktop do Chloros instalado; o SDK, nesse caso, controla um backend em outra máquina, consulte [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

O PyPI disponibiliza a versão estável mais recente. Mantenha o SDK na mesma versão que o backend do Chloros que ele controla (consulte [Versões e compatibilidade](#versioning--compatibility)). O auxiliar DAQ-M `DLS` também precisa do `bleak`: `pip install "chloros-sdk[camera]"` (ou `pip install bleak`).

#### Verificar

```python
import chloros_sdk
print(chloros_sdk.__version__)
print("CAMERA_AVAILABLE =", chloros_sdk.CAMERA_AVAILABLE)
print("PROJECT_AVAILABLE =", chloros_sdk.PROJECT_AVAILABLE)
```

> **É necessária uma assinatura do Chloros+.** Todas as chamadas do SDK exigem um login ativo no Chloros+. Execute o `chloros-cli login user@example.com 'YourPassword'` uma vez por máquina; as credenciais são armazenadas em cache no `~/.chloros/`.

### Preciso do Pacote para Desktop?

O pacote pip por si só **não** é suficiente para a maioria dos fluxos de trabalho. Veja a seguir o que cada interface do SDK precisa:

| Interface do SDK | Precisa do Pacote para Desktop? | Por quê |
| --- | --- | --- |
| `ChlorosLocal`, `process_folder`, `process_lattice_capture` | **Sim**(para iniciar um)**/ Não** (um backend já acessível) | Use um backend em execução (o do aplicativo para desktop ou outro script iniciado); caso contrário, inicie automaticamente o instalado: `/usr/lib/chloros/chloros-backend` (Linux) ou `<install folder>\backend\chloros-backend.exe` (em Windows; `C:\Program Files\Chloros\backend\` por padrão, uma pasta de instalação personalizada é lida a partir do registro do instalador). Passe `backend_exe=` para nomeá-lo você mesmo. |
| `chloros-mcp` ([servidor MCP](#mcp-server-ai-assistants)) | **Sim**(para iniciar um)**/ Não** (um backend já está acessível) | Baseado em `ChlorosLocal`, portanto, igual à linha acima. |
| `read_image_audit_tags` | **Não** | Lê os metadados da própria imagem processada (somente na biblioteca padrão do Python). |
| `connect_camera`, `connect_array`, `connect_daq_sensor`, `analyze_array_network`, `list_*`, `discover_*` | **Sim**(local)**/ Não**(remoto) | Clientes HTTP puros no backend. Backend local → pacote para desktop necessário. Backend remoto → `backend_url=`**por meio de um túnel** (consulte Modo de Backend Remoto — os backends fornecidos se ligam apenas ao loopback). |
| `ChlorosProject` / `open_project` | **Sim** | Acessa projetos salvos por meio do backend. |
| Classes LATTICE diretas (`LatticeCamera`, `CameraPool`, `Calibration`, `DLS`, …) | **Sim** | Requer o runtime do Arena SDK, que vem incluído no pacote para desktop, e o OpenCV (`cv2`) no seu Python, que nem o pacote para desktop nem o SDK instalam (`pip install opencv-python`). Caso contrário, o `CAMERA_AVAILABLE` é o `False` na importação. O `DLS` também precisa do `bleak`. |

### Modo de back-end remoto (host somente pip, via túnel)

> **O back-end instalado não está acessível pela LAN.** Ele escuta
> apenas no loopback (`127.0.0.1` / `::1`), portanto, o `backend_url="http://<lan-ip>:5000"`
> **não consegue acessar um Chloros instalado**. Para operar um backend em outra
> máquina, redirecione você mesmo a porta de loopback dele e configure o SDK para o
> túnel:

```bash
# on the pip-only host: forward local 5000 to the Chloros machine's loopback
ssh -N -L 5000:127.0.0.1:5000 user@chloros-host
```

```python
import chloros_sdk

BACKEND = "http://127.0.0.1:5000"     # the tunnel endpoint
serials = ["213800234", "214000533"]   # master first

# auto_start_backend=False: the tunnel URL looks local, so without it a tunnel
# that is down would start this machine's own backend, if one is installed.
arr = chloros_sdk.connect_array(serials, backend_url=BACKEND,
                                auto_start_backend=False)
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                     backend_url=BACKEND,
                                     auto_start_backend=False)

# The backend opens every path on ITS machine: pass absolute paths valid there.
# (A relative path would be made absolute against this machine's working folder.)
arr.capture("/home/chloros/captures/run1", processing="raw")
```

Hosts sem interface gráfica / CI / robótica podem manter uma máquina com a instalação completa da área de trabalho como o “servidor Chloros” e o `pip install chloros-sdk` em todos os outros locais — mas o transporte entre eles é o túnel configurado pelo usuário acima, não um URL direto da LAN.

> **`ChlorosLocal` em um host apenas com pip.** O `ChlorosLocal(backend_url=BACKEND)` se conecta a um backend que pode ser acessado sem a instalação da área de trabalho: ele procura um binário de backend local somente quando precisa INICIAR um (nenhum respondendo e `auto_start_backend=True`). Até a versão 1.2.0, ele procurava o binário primeiro e gerava o erro `ChlorosBackendError` (“Backend Chloros não encontrado…”) em um host apenas com o pip, mesmo com um backend acessível. As pastas que você passa (`process_folder`, `import_images`) são verificadas nesta máquina e lidas pelo backend; portanto, com um backend remoto, elas devem indicar a mesma pasta em ambos, como uma unidade compartilhada.

### Fluxo de trabalho apenas com DAQ (host apenas com pip)

O `pip install chloros-sdk` por si só não pode controlar um sensor DAQ: um sensor DAQ é sempre controlado por meio de um backend do Chloros com o `connect_daq_sensor()`:

- **Sensor nesta máquina:** instale o pacote de desktop do Chloros aqui. O `connect_daq_sensor()` inicia seu backend se nenhum estiver em execução.
- **Sensor em outra máquina:** essa máquina precisa do pacote para desktop; acesse seu backend por meio do túnel no [Modo Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel). O backend abre o sensor em sua própria máquina, portanto, um DAQ-U deve estar conectado a essa máquina, um DAQ-M deve estar dentro do alcance do Bluetooth dela e um DAQ-E deve estar em uma rede que ela possa acessar.

```python
import chloros_sdk

# http://127.0.0.1:5000 is the local desktop install's backend, or the tunnel.
with chloros_sdk.connect_daq_sensor() as daq:   # smart-detect USB / BLE / ETH
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])
```

De qualquer forma, o backend aplica o plano Chloros+: faça login na máquina que executa o backend (`chloros-cli login` ou o aplicativo para desktop) com qualquer plano pago (Copper ou superior). Um backend sem sessão ativa e um plano inferior ao Copper geram o código de erro `ChlorosConnectError`, indicando o motivo do backend (`AUTH_REQUIRED` / `PLAN_UPGRADE_REQUIRED`).

---

## Introdução rápida

```python
import chloros_sdk

# === Image processing ===
results = chloros_sdk.process_folder(
    "C:/DroneImages/Flight001",
    indices=["NDVI", "GNDVI", "OSAVI"],
)

# === Live LATTICE single-cam ===
with chloros_sdk.connect_camera("213800234") as cam:
    cam.set_settings(exposure_time=10000, gain=0.0)
    cam.capture("output/")

# === Live LATTICE synchronized array (GUI smart-prep flow) ===
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    arr.capture("output/", processing="reflectance")

# === Live DAQ spectral sensor ===
with chloros_sdk.connect_daq_sensor() as daq:    # smart-detect USB / BLE / ETH
    for frame in daq.latest(n=5):
        print(frame["spectrum"][:10])

# === Drive a saved project end-to-end ===
proj = chloros_sdk.open_project("/path/to/project")
proj.connect_all()
proj.arrays["main_rig"].capture("output/", processing="reflectance")
proj.disconnect_all()
```

---

## Índice da API de nível superior

```python
import chloros_sdk

# === Image processing (full pipeline) ===
chloros_sdk.ChlorosLocal                          # class
chloros_sdk.process_folder(...)                   # one-shot helper
chloros_sdk.process_lattice_capture(...)          # LATTICE-friendly defaults
chloros_sdk.read_image_audit_tags(path)           # post-run audit

# === Live cameras (persistent backend pool) ===
chloros_sdk.connect_camera(serial, ...)           # → CameraSession
chloros_sdk.connect_array(serials, ...)           # → ArraySession (smart-prep)
chloros_sdk.attach_array(serials_or_id, ...)      # → ArraySession (attach without re-connecting)
chloros_sdk.list_cameras()
chloros_sdk.list_arrays()
chloros_sdk.discover_lattice_cameras()
chloros_sdk.analyze_array_network(...)            # network capability + recommendation
chloros_sdk.CaptureResult                         # list subclass returned by CameraSession.capture / ArraySession.capture
chloros_sdk.RecorderHandle                        # handle for an array record()/burst() job
chloros_sdk.ContinuousHandle                      # handle for an array continuous() capture
chloros_sdk.GrabResult                            # dict returned by grab() / yielded by frame_stream()
chloros_sdk.FrameFetchError                       # raised by grab()/frame_stream() on a refusal

# === Live DAQ sensors (persistent backend pool) ===
chloros_sdk.connect_daq_sensor(...)               # → DAQSensorSession
chloros_sdk.discover_daq_sensors()                # scan USB/BLE/ETH (finds a DAQ-M MAC)
chloros_sdk.list_daq_sensors()
chloros_sdk.daq_bindings() / set_daq_bindings(cameras=, arrays=)   # which DAQ feeds which camera
chloros_sdk.daq_band_presets(filter_code=None)    # band sets for Ambient + Object reflectance
chloros_sdk.light_sensor_reflectance(ambient_id, object_id, ...)   # two DAQs -> reflectance + indices
chloros_sdk.time_sync_status() / time_sync_peers() / time_sync_cameras()   # PTP grandmaster
chloros_sdk.restart_time_sync() / set_time_sync_priority(priority1=, priority2=)

# === Camera utilities (the CLI's lattice commands; by serial) ===
chloros_sdk.camera_color_profile(serial) / set_camera_color_profile(serial, ...)
chloros_sdk.camera_color_adjust(serial) / set_camera_color_adjust(serial, ...)
chloros_sdk.camera_highlights(serial) / set_camera_highlights(serial, value)
chloros_sdk.camera_white_balance_once(serial) / set_camera_white_balance(serial, ...)
chloros_sdk.camera_presets() / apply_camera_preset(serial, preset)
chloros_sdk.camera_filter_models() / camera_filter_model_info(name)
chloros_sdk.camera_firmware_status() / update_camera_firmware(serial)
chloros_sdk.camera_power_probe(serial) / sleep_camera(serial) / wake_camera(serial)
chloros_sdk.calibrate_camera_with_target(serial, ...)
chloros_sdk.network_adapters(...) / fix_network_settings()
chloros_sdk.apply_vignette_correction(input_dir, output_dir, ...)
chloros_sdk.read_dls(mac, ...) / probe_system(...) / system_info()

# === Transient cameras, alignment profiles, indices (cameras opened for one command) ===
chloros_sdk.start_camera_continuous(serial, ..., daq=None, daq_tolerance_ms=None)  # → CliJob; a DAQ bound to the camera records one .daq per frame (daq=False: none)
chloros_sdk.start_multi_camera_stream(serials, ...)   # → CliJob
chloros_sdk.capture_cameras_transient(serials, ...)
chloros_sdk.test_array_sync(serials, count, ptp_settle_s=None) / detect_array_master(line) / analyze_array_models(models)
chloros_sdk.calibrate_alignment_profile(serials, save_path, ...)
chloros_sdk.capture_aligned_multiband(profile_path, ...) / start_aligned_stream(profile_path, ...)  # → dict / CliJob
chloros_sdk.alignment_profile_info(path) / reorder_alignment_profile(path, ...)
chloros_sdk.index_catalog() / compute_index(input_path, ...) / capture_live_index(profile_path, ...)
chloros_sdk.CliJob                                # stats() / stop() / wait()

# === Project lifecycle ===
chloros_sdk.open_project(path, relink=None)       # → ChlorosProject
chloros_sdk.project_folder() / set_project_folder(path) / reset_project_folder()
chloros_sdk.run_project_recipe(recipe, project_path)  # `chloros-cli project run`
chloros_sdk.ChlorosProject                        # class
chloros_sdk.AlignmentSpec                         # dataclass
chloros_sdk.ArrayHandle, CameraHandle, SensorHandle

# === Direct-hardware (no-backend) classes (from lattice_sdk) ===
chloros_sdk.LatticeCamera, CameraSettings, PRESETS, CameraPool
chloros_sdk.Calibration, CalibrationCoefficients, FilterModel, list_filters
chloros_sdk.DLS, NetworkDiagnostics               # DLS: DAQ-M over Bluetooth; needs bleak

# === Exceptions ===
chloros_sdk.ChlorosError                          # base
chloros_sdk.ChlorosBackendError
chloros_sdk.ChlorosLicenseError
chloros_sdk.ChlorosConnectionError
chloros_sdk.ChlorosProcessingError
chloros_sdk.ChlorosProjectFilesMissingError      # a ChlorosProcessingError
chloros_sdk.ChlorosAuthenticationError
chloros_sdk.ChlorosConfigurationError
chloros_sdk.ChlorosConnectError                   # raised by smart-connect surface
chloros_sdk.LatticeError, CameraNotFoundError, ...  # from lattice_sdk

# === Availability flags ===
chloros_sdk.CAMERA_AVAILABLE     # True iff lattice_sdk imported cleanly
chloros_sdk.PROJECT_AVAILABLE    # True when chloros_sdk.project imported; open_project()
                                 # still raises ImportError without the project loaders
```

---

## Processamento de imagens — `ChlorosLocal`

A classe principal do pipeline. Inicia o backend ao ser instanciada (se nenhum estiver respondendo e `auto_start_backend=True`), cria/configura projetos, monitora o andamento e retorna resumos pós-execução.

### Construtor

```python
ChlorosLocal(
    api_url="http://127.0.0.1:5000",   # backend URL (also: backend_url=)
    auto_start_backend=True,            # spawn backend if not running
    backend_exe=None,                   # override backend binary path
    timeout=30,                         # request timeout seconds
    backend_startup_timeout=60,         # backend boot timeout
    processing_timeout=14400,           # hard cap on process() (4 h)
    processing_stuck_timeout=1800,      # no-progress threshold (30 min)
)
```

### Métodos

| Método | Descrição |
| --- | --- |
| `create_project(project_name, camera=None)` | Cria um novo projeto (opcionalmente com um modelo de câmera, como `"Survey3N_RGN"`). |
| `open_project(project_path, relink=None)` | Abre um projeto salvo (sua pasta ou seu nome na pasta de projetos) e o torna o projeto aberto. Os caminhos de um projeto movido são redirecionados para a pasta em que ele se encontra agora; `relink` indica o nome da pasta em que os demais arquivos ausentes estão agora. Retorna a resposta do backend com `source_files`; gera o erro `ChlorosProjectFilesMissingError` quando nenhum arquivo for encontrado. Consulte **Um projeto movido** abaixo. |
| `import_images(folder_path, recursive=False)` | Importa imagens RAW/TIF/JPG/DNG **e `.daq` gravações do sensor de luz**. Retorna `count` (imagens) e `scan_count` (gravações). Avisará apenas se a pasta não contiver nenhum dos dois. |
| `export_light_sensor(daq=True, csv=True)` | Grava os arquivos calibrados `.daq` + `.csv` para cada gravação do sensor de luz do projeto, na pasta `<project>/Light Sensor/`. Consulte [Gravações do sensor de luz](#light-sensor-recordings--calibrated-daq--csv). |
| `configure(debayer=..., vignette_correction=..., reflectance_calibration=..., indices=[...], export_format=..., ppk=..., daq_log_path=..., input_level=..., radiometric_output=..., array_alignment=..., array_alignment_crop=..., array_alignment_interpolation=..., custom_settings=None)` | Definir controles de processamento. `indices` são nomes de predefinições (`NDVI`, `GNDVI`, `OSAVI`, …), cada um adicionado ao projeto para as faixas de sua câmera; um nome que não seja uma predefinição, ou que as faixas não possam formar (`NDRE` requer RE e NIR, o que nenhuma câmera sozinha fornece), recebe um nome do tipo `UserWarning` e nenhuma imagem de índice. `daq_log_path` designa uma gravação do sensor de luz (um arquivo `.daq` ou `.csv`, ou uma pasta com esses arquivos) com a qual cada quadro é comparado, tanto no LATTICE quanto no Survey3, em vez das gravações importadas junto com as imagens; se um caminho estiver ausente ou não contiver nenhuma gravação, as gravações importadas serão utilizadas. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executa o pipeline. Retorna `{"status": "complete", "async": False}`, além de uma chave `summary` quando o backend fornecer uma — consulte [Resumo pós-execução e dicas](#post-run-summary--hints) — `export_check_failure` quando as exportações da execução não puderam ser verificadas e `image_failure` quando uma imagem foi perdida. `wait=False` retorna `{"status": "started", "async": True}` de uma só vez. |
| `get_config()` / `get_status()` / `status()` | Inspecione o estado do backend. |
| `logout()` | Sair: limpa as credenciais armazenadas em cache e libera o slot de dispositivo desta máquina (`device_deregistration` no resultado). Uma execução de processamento em andamento não é interrompida: ela é concluída no modo em que foi iniciada (`run_in_progress: true` e `run_mode` no resultado), e a próxima execução utiliza o processamento padrão. |
| `shutdown_backend()` | Interrompe o backend se esta instância o tiver iniciado; um backend encontrado em execução não é afetado. Deixar um bloco `with` ou a instância sendo submetida à coleta de lixo não o interrompe: o backend continua em execução para o próximo script e se desliga automaticamente após 30 minutos, caso não haja nada utilizando-o. O backend é solicitado a desligar-se, portanto, ele libera suas câmeras e sensores de luz e conclui qualquer gravação em primeiro lugar; ele só é encerrado se ainda estiver em execução 10 s depois. |
| `discover_cameras()` | Descobrir câmeras LATTICE **por meio do backend desta instância** (`/api/camera/discover`). Retorna uma lista de dicionários (`serial`, `model`, `ip`, …) — com o mesmo formato que a GUI e a CLI veem. Lista vazia se nenhuma for encontrada ou se o backend estiver inacessível. |
| `camera_capture(output_dir, format="tiff", **settings)` | Captura um único quadro**por meio do backend**(iniciado automaticamente por este identificador) para que receba a mesma preparação que a GUI/CLI (padrão de 12 bits, reutilização de pool, metadados de cal embutidos). Resolva o alvo com `serial=` ou `device_index=`; passe `exposure`/`gain`/`pixel_format`/`preset` como `**settings`. Retorna o dicionário de metadados legados (`filepath`, `width`, `height`, `pixel_format`, `exposure_time`, `gain`, `timestamp`). |
| `camera_stream(serial, *, fps=10.0, overlay=None, decode=True, connect_timeout=10.0, read_timeout=15.0)` | Gera quadros de pré-visualização compostos por sobreposição a partir de uma câmera agrupada — cliente MJPEG leve pela rota `/api/camera/<serial>/stream-annotated` do backend (zebra / grade / mira / histograma / pico / ponto desenhado no lado do servidor). `decode=True` gera matrizes BGR e requer o OpenCV (`cv2`), que o SDK não instala; o `False` gera bytes JPEG brutos. Também acessível por projeto como `ChlorosProject.stream(overlays=True)`. |

**Uma execução em andamento mantém seu projeto.** Enquanto uma execução do Processing estiver em andamento — seja deste script, do aplicativo, do `chloros-cli process` ou do `chloros-mcp` —, o backend se recusa a criar, abrir, importar ou alterar as configurações de processamento de um projeto (409 HTTP `PROCESSING_IN_PROGRESS`): `create_project()` e `import_images()` geram `ChlorosProcessingError`, e o `configure()` gera o `ChlorosConfigurationError`, e o `process()` com um modo diferente do que está em execução gera o `ChlorosProcessingError` (uma execução mantém o modo em que foi iniciada), cada um com a mensagem do backend. Uma captura salva no projeto é então anexada a ele quando a execução termina (a resposta `project_add` indica isso). Aguarde até que a execução termine (`get_status()`) e, em seguida, continue.**Um projeto movido.** O `project.json` de um projeto nomeia cada arquivo de origem (todas as imagens e gravações do sensor de luz) pelo caminho completo; portanto, mover a pasta do projeto — ou toda a pasta de projetos para outra unidade — faz com que eles continuem indicando o caminho antigo. Cada vez que o projeto é aberto, ele corrige o que pode por conta própria: um arquivo que estava dentro da pasta do projeto é procurado na pasta em que o projeto está agora, e cada caminho encontrado ali é redirecionado e salvo (o `project.json` é primeiro copiado como `project.json.before-relink-<date>-<time>`). Um caminho só é redirecionado para um arquivo que realmente esteja presente. Arquivos que estavam fora da pasta do projeto — uma sessão do hub, uma importação via “Adicionar Pasta” — são encontrados apenas com `relink=<the folder they are in now>` (ou qualquer pasta acima dela). `open_project()` retorna `source_files`: `total`, `found`, `missing`, `missing_roots` (as pastas nas quais os arquivos ausentes estavam), `missing_outside_project` e `relink` (o que foi redirecionado, de onde para onde). Um projeto com alguns arquivos ausentes é aberto com o restante; um projeto em que nenhum arquivo foi encontrado não é aberto, e o `open_project()` gera o `ChlorosProjectFilesMissingError` (`.missing_roots`, `.audit`; HTTP 409 `PROJECT_SOURCES_MISSING`). Os erros `project open --relink` e `chloros-mcp` da CLI`open_project` (`relink_folder`) da CLI fazem o mesmo.

Use como gerenciador de contexto para garantir a limpeza:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26", camera="Survey3N_RGN")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI"],
        export_format="TIFF (16-bit)",
    )
    results = cl.process(mode="parallel", wait=True)
print(results["summary"])
```

### Gravações do sensor de luz — calibradas `.daq` + `.csv`

Um DAQ-U / DAQ-M / DAQ-E pode ser gravado **sem** seu pacote de calibração. Isso é
o que os gravadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts)
(`record_daq.py`) fazem por padrão: eles gravam as contagens brutas dos sensores e marcam o
arquivo para que o Chloros busque a calibração de fábrica daquele sensor **pelo número de série** — primeiro no cache local
primeiro, depois na nuvem MAPIR — e a aplica na importação.

O Chloros grava o resultado de volta como dois produtos por gravação, sob
`<project>/Light Sensor/`:

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | O arquivo reprocessável — mesmo esquema de uma gravação em tempo real, agora declarando o pacote que o produziu. Reimportá-lo **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grade de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas (potência total, lux fotópico/escotópico, PPFD e sua divisão em azul/verde/vermelho, comprimento de onda de pico). |
| `<name>_raw.daq` / `<name>_raw.csv` | **Apenas sensores sem pacote (DAQ-A).** Contagens espectrais brutas do sensor — *não* irradiação. Veja abaixo. |

`<name>` é o nome do próprio arquivo da gravação. Duas gravações em pastas diferentes podem compartilhar
uma — leitura por captura (`<capture>/daq/daq_data_<sensor id>_<ts>.daq`) salva no
mesmo milissegundo por duas câmeras — e cada uma ainda recebe seus próprios produtos: a primeira mantém
`<name>`; a seguinte acrescenta as pastas que a diferenciam **após** o nome
(`<name>__<capture>_daq_calibrated.daq`), de modo que a data e a hora da gravação continuam aparecendo primeiro.

O `process()` realiza essa exportação como uma de suas etapas. Ele **não** requer imagens:
um sensor de luz operado sozinho já constitui um fluxo de trabalho de primeira classe, e tal projeto tem zero
imagens por definição.

**As gravações do DAQ-A são exportadas como contagens brutas.** A família DAQ-A é anterior ao sistema de pacotes por série
e não possui nenhum pacote a ser buscado — ela é calibrada em campo em relação a um
alvo de refletância; é por isso que nunca precisou de um. Essas gravações são exportadas
com o prefixo `_raw` em vez de `_calibrated`: um nome de arquivo diferente em vez de um sinalizador
dentro do arquivo, pois a informação precisa permanecer intacta ao ser enviada por e-mail apenas como um nome simples. O
cabeçalho `.csv` indica `raw spectral sensor counts (NOT irradiance)` e avisa que os
valores são comparáveis **dentro** do arquivo — exatamente para isso serve a calibração baseada em alvo
para isso — e não entre sensores. As colunas fotométricas dependentes de potência (potência total,
lux fotópico/escotópico, PPFD) retornam **NULL** em vez de serem integradas a partir das contagens.

Um DAQ-U / DAQ-M / DAQ-E cujo pacote simplesmente não pôde ser obtido ainda é **ignorada**,
não gravada em formato bruto: nesse caso, o pacote existe e “reconectar e reprocessar” é um conselho válido.

Gravações legadas **v1.01 / v1.02** (um DAQ-A-SD as grava) não contêm época por leitura,
apenas a hora de gravação do arquivo. O comparador de imagem↔fluxo descendente ainda as rejeita — comparar um
quadro com uma data de gravação seria um erro imperceptível — mas o exportador as lê, e o
arquivo CSV imprime `clock=daq_created_on`, de modo que o produto indica em qual relógio está.

```python
import chloros_sdk

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("DAQ-U_2026-08-26")
    cl.import_images("C:/Flights/raw_daq")     # .daq only — no camera involved
    result = cl.export_light_sensor()          # or just cl.process()

for rec in result["exported"]:
    print(rec["csv"])
for rec in result["skipped"]:
    print("skipped", rec["source"], "--", rec["reason"])
```

Uma gravação cujo pacote de calibração não pode ser obtido (offline ou um sensor sem
calibração registrada) é relatada sob `skipped` **com o motivo**. Ela nunca é
gravada como um arquivo “calibrado” contendo contagens brutas — conecte-se à internet e
execute novamente, e a exportação será concluída.

### Retornos de chamada de progresso

```python
def show_progress(percent, message):
    print(f"[{percent:3d}%] {message}")

with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("FieldA")
    cl.import_images("C:/DroneImages/Flight001")
    cl.configure(indices=["NDVI"])
    cl.process(progress_callback=show_progress, poll_interval=1.0)
```

### Resumo pós-execução e dicas

Após a conclusão, o `process()` busca o `GET /api/processing-summary` e anexa o corpo como `result["summary"]`. A busca é feita com o melhor esforço possível e nunca bloqueia um retorno bem-sucedido — se o resumo não estiver disponível, o `process()` recorre ao formato simples do `{"status": "complete", "async": False}`. Cada entrada no `summary["hints"]` — frases completas com a correção sugerida, por exemplo, por que uma execução produziu saída zero — também é reemitida como um `UserWarning` em Python, de modo que execuções com saída zero são autodiagnósticas, mesmo que você nunca inspecione o dicionário:

```python
result = cl.process()
for hint in result.get("summary", {}).get("hints", []):
    print("HINT:", hint)
# hints also arrive on the warnings channel:
#   python -W always::UserWarning your_script.py
```

`summary["totals"]` é a parte legível por máquina:

| Chave | O que conta |
| --- | --- |
| `models` | Grupos de câmeras na execução. |
| `images_in_groups` | Imagens de origem nesses grupos. |
| `targets_found` | Alvos de refletância detectados. |
| `images_calibrated` | Imagens para as quais a execução gerou um produto de refletância. |
| `images_failed` | Imagens que falharam e não foram exportadas. |
| `exported_files` | **Arquivos de produtos de imagem gerados pela execução**: todos os produtos (desbayering, pré-visualização, radianância, refletância, índice, renderizações de alvos), em todos os formatos. |
| `reflectance_files` | Os produtos de refletância entre eles (`Reflectance_Calibrated_Images`), independentemente de terem sido calibrados pelo alvo ou pela irradiância do sensor de luz (DAQ). Uma execução sem alvo que ainda assim gerou refletância indica isso em uma dica; apenas uma execução que não gerou nada recebe a mensagem de que suas saídas de refletância estão ausentes. |
| `clamp_masks_written` | As [máscaras de limitação de desmistura](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) indicadas ao lado delas — são complementos, não produtos, portanto não constam em `exported_files` (a CLI as contabiliza separadamente da mesma forma). |
| `captures_left_out` | Quadros brutos na pasta `captures` do projeto que esta execução não capturou. Uma execução aguarda, ao iniciar, o que foi capturado anteriormente; portanto, esse valor é 0, a menos que uma captura tenha sido salva durante a execução ou que o tempo de espera tenha expirado. `summary["captures_left_out_note"]` indica o que fazer com elas. |
| `daq_recordings_exported` / `daq_recordings_skipped` | Gravações do sensor de luz, contadas separadamente propositalmente — elas provêm de uma etapa diferente e existem para execuções sem nenhuma imagem; portanto, incluí-las faria com que uma execução apenas de DAQ parecesse ter exportado imagens. |

Juntamente com elas: `summary["output_dirs"]` (todos os diretórios nos quais foram gravados dados),
`summary["light_sensor_export"]`, `summary["stopped"]` (verdadeiro quando o usuário interrompeu a
execução, para que contagens parciais não sejam interpretadas como uma execução concluída com produção insuficiente), e
`summary["groups"]` (a divisão por grupo).

`exported_files` é registrado pelo pipeline **à medida que grava**, e não extraído dos
objetos de imagem do projeto posteriormente. As estratégias paralela e de GPU criam seus próprios objetos de imagem
(em subprocessos de trabalho para os caminhos de GPU), de modo que a varredura antiga relatava
`0 file(s) written` para cada uma dessas execuções e, em seguida, emitia a dica de zero exportações — em execuções
nas quais tudo havia funcionado. Se você criar um script com base nesse número, uma execução paralela válida agora
relata uma contagem diferente de zero.

Os saltos do sensor de luz relatam o motivo que o leitor realmente identificou para cada arquivo — um
esquema ilegível, um pacote ausente, um erro de gravação — **deduplicado**; assim, vinte arquivos
ignorados por um único motivo são interpretados como um único motivo, em vez de vinte repetições dele.

> **`process()` não é acionado quando uma execução não produz imagens.** Esse é um dos dois pontos em que o SDK e
> a CLI diferem deliberadamente: o `chloros-cli process` trata “produtos foram solicitados, nenhum foi
> gravado” como uma falha e encerra com valor diferente de zero, enquanto o SDK retorna normalmente e relata a
> condição por meio do `summary` / dicas. Se seu pipeline deve parar em uma execução vazia, verifique isso
> você mesmo — inspecione o `summary` (ou conte os arquivos na pasta do projeto) em vez de confiar na
> ausência de uma exceção. As causas mais comuns são uma pasta de entrada que não foi reconhecida como uma
> captura e produtos ignorados por serem inaplicáveis às câmeras presentes (por exemplo, radiação de câmeras apenas RGB
>).
>
> O `process()` também não é gerado quando as exportações de uma execução não puderam ser verificadas: ele emite um aviso e define
> o `result["export_check_failure"]`. Os produtos são gravados. Para interromper a execução nesse caso, verifique essa chave. Uma execução que
> perdeu uma imagem funciona da mesma forma: ela emite um aviso e define `result["image_failure"]`, o que faz com que `chloros-cli process`
> retorne um valor diferente de zero.

### Funções de conveniência

```python
# One-call process: project + import + configure + process
results = chloros_sdk.process_folder(
    folder_path="C:/DroneImages/Flight001",
    project_name="FieldA_2026-05-26",
    camera="Survey3N_RGN",
    indices=["NDVI", "GNDVI", "OSAVI"],
    vignette_correction=True,
    reflectance_calibration=True,
    export_format="TIFF (16-bit)",
    mode="parallel",
    debayer="High Quality (Faster)",      # or "Texture Aware (Slow, Highest Quality)"
    ppk=False,
    recursive=False,
    processing_timeout=14400,
)

# LATTICE-friendly defaults (no panel-target detection, standard debayer)
results = chloros_sdk.process_lattice_capture(
    folder_path="C:/Captures/2026-05-13_Field",
    indices=["NDVI"],
)

# process_folder() / process_lattice_capture() also take input_level=,
# array_alignment=, array_alignment_crop=, array_alignment_interpolation=,
# progress_callback=, poll_interval=, timeout=, processing_stuck_timeout=,
# backend_exe= and api_url= (NOT backend_url=). indices defaults to ["NDVI"].

# Audit which calibration sources were applied to a processed image
tags = chloros_sdk.read_image_audit_tags("output/Reflectance_Calibrated/x.tif")
print(tags["CalibrationSource"])   # 'per_serial' / 'legacy_lookup' / 'none'
print(tags["VignetteSource"])      # 'per_serial' / 'legacy_polynomial' / 'none'
```

Cada uma é executada dentro de seu próprio `with ChlorosLocal(...)`; um backend que ela precisou iniciar continua em execução quando ela retorna, para a próxima chamada, e se encerra automaticamente após 30 minutos sem que nada a esteja utilizando. Nenhuma delas aceita `auto_start_backend`, `radiometric_output` ou `daq_log_path`; use `ChlorosLocal` e `configure()` para esses casos. `process_lattice_capture()` não aceita `camera`, `debayer` ou `ppk`: ele ignora a detecção de painel e usa o debayer padrão.

### Valores suportados

```python
# export_format
"TIFF (16-bit)"           # default, recommended
"TIFF (32-bit, Percent)"  # reflectance percentage as float32
"TIFF (8-bit)"            # all-8-bit images are written as this; never raised
"PNG (8-bit)"
"JPG (8-bit)"

# debayer
"High Quality (Faster)"               # standard, default
"Texture Aware (Slow, Highest Quality)"  # neural debayer for Survey3 imagery, Chloros+ only.
                                         # Survey3 only: on a project holding LATTICE captures
                                         # (which always use the standard demosaic) configure()
                                         # or process() raises ChlorosConfigurationError; a
                                         # Texture Aware value the project saved earlier is set
                                         # to Standard with a UserWarning, as the app does. Needs a
                                         # CUDA GPU the backend's PyTorch can use (an NVIDIA
                                         # GPU, or a Jetson Orin's). Without one, process()
                                         # uses the standard debayer, and the backend prints
                                         # one "[DEBAYER] ... needs a CUDA GPU" line saying why.
"Standard (Fast, Medium Quality)"      # alias used internally for LATTICE

# input_level (LATTICE only; Survey3 .raw ignores)
"auto"        # default — infers from each file's XMP ProcessingLevel tag
"raw"         # force-treat as raw Bayer
"debayered"   # force-treat as Chloros's saved debayered (3-channel) product
"processed"   # force-treat as already-calibrated radiance

# array_alignment / array_alignment_crop (LATTICE arrays; None = keep saved setting)
True          # backend default — apply the module-to-module transform stamped
              # in each capture's Chloros:Alignment* XMP to every product
False         # export in native sensor geometry / skip the common-overlap crop

# array_alignment_interpolation (alignment warp resampling)
"bilinear"    # backend default
"nearest"     # preserves exact source DNs (no inter-pixel value mixing)
"cubic"
```

#### Saída radiométrica (pipeline multiespectral LATTICE)

O nível de exportação multiespectral LATTICE (M3C/M3M) do pipeline `process` — `reflectance` (padrão), `radiance`, `sensor-response` ou `all` (cada modo aplicável por imagem) — define os produtos **“Exportar radianância”**e**“Exportar refletância”****: `radiance` exporta apenas radianância, `reflectance` apenas refletância, `sensor-response` nenhuma das duas (apenas os produtos sem bayering e de visualização), enquanto o `all` exporta ambos. Os produtos sem bayering e de pré-visualização não podem ser alterados neste nível. O `configure()` possui uma palavra-chave dedicada para isso:

```python
with chloros_sdk.ChlorosLocal() as cl:
    cl.create_project("Field_A")
    cl.import_images("C:/Captures/lattice_flight")
    cl.configure(
        radiometric_output="radiance",   # reflectance (default) / radiance / sensor-response / all
        export_format="TIFF (32-bit, Percent)",
    )
    cl.process()
```

A saída de emergência avançada — definir os produtos por meio de `custom_settings` — ainda funciona, mas lembre-se de que ela substitui todo o bloco de configurações (veja o aviso abaixo). Defina as chaves `"Export <product>"`: as configurações do próprio projeto em `"Export radiance"` / `"Export reflectance"` de um projeto prevalecem sobre a chave mais antiga `"Radiometric output"`; portanto, essa chave por si só não altera nada.

```python
cl.configure(custom_settings={
    "Project Settings": {
        "Processing": {"Export radiance": True, "Export reflectance": False},
        "Export": {"Calibrated image format": "TIFF (32-bit, Percent)"},
    }
})
```

`reflectance` (o padrão) divide a radiação da câmera pelo **fluxo descendente do DAQ com carimbo de data/hora correspondente**, resolvido automaticamente a partir de um `.daq` (DAQ-U/M/E) gravado**ou um `.csv` nativo do DAQ-M**encontrado junto com as imagens; qualquer pacote de calibração por câmera ou DAQ ausente localmente é**buscado automaticamente da AWS** na primeira utilização. A CLI expõe isso de acordo comopções de tipo de produto no `chloros-cli process`: `--radiance`/`--no-radiance`, `--reflectance`/`--no-reflectance`, `--debayered`, `--preview`.**De quem é essa leitura?**A leitura salva por uma captura `.daq` é a leitura da(s) câmera(s) que ela descreve — nomeadas em um arquivo sidecar `<file>.daq.cameras.json` ao lado dela (capturas únicas, sincronizadas e em matriz; uma sequência bruta nomeia a leitura de cada membro em seu manifesto) — e é associada**apenas a imagens dessas câmeras**. Uma câmera autônoma, sem leitura própria, portanto, não utiliza mais o DAQ de outra câmera; vincule um DAQ a ela ou grave uma. A própria gravação de um DAQ (`chloros-cli daq pool-record`, ou uma gravação iniciada na GUI), ou qualquer `.daq` sem esse arquivo sidecar (incluindo capturas mais antigas), ainda atende a todas as câmeras.**Câmeras OCN sob luz solar.** Todo pacote de calibração OCN é construído sob luz de banda estreita, com um LED por banda, enquanto a luz solar mistura as bandas. O Chloros 1.3.0 aplica uma correção rápida com base na luz solar após a separação das bandas, o que melhora os produtos de radianância, refletância e índice OCN sob luz solar. Ela é executada em qualquer lugar onde a separação de bandas ocorre — `process()`, capturas, `grab()` / `frame_stream()` e a visualização ao vivo — para pacotes OCN que oferecem suporte a ela; as câmeras RGN permanecem inalteradas. Imagens OCN capturadas com uma versão anterior também recebem essa correção quando processadas em um computador que contenha a calibração com a qual foram capturadas (o Chloros mantém a calibração da câmera assim que ela é conectada); caso contrário, elas são processadas como antes. Os valores de radiação, refletância e índice do OCN, portanto, diferem dos da versão 1.2.0.

> `custom_settings` **substitui** todo o bloco de configurações calculadas (ele ignora as outras palavras-chave e a validação do `configure()` por padrão). Ao usá-lo, inclua todas as chaves `Project Settings` que forem relevantes para você, como no exemplo acima.

#### Leitura de pixels de índice (índice NDVI e outros índices de vegetação)

**Um raster de índice inteiro é um alongamento de contraste por imagem, não um mapa no intervalo de −1 a 1.** `TIFF (16-bit)`, `TIFF (8-bit)`, `PNG (8-bit)` e `JPG (8-bit)` — incluindo o arquivo TIFF de 8 bits no qual uma captura de 8 bits é gravada — armazenam

```

DN = (value − IndexMin) / (IndexMax − IndexMin) × full_scale
```

onde `IndexMin` / `IndexMax` são **os valores de índice finitos mais baixo e mais alto daquele quadro**. DN 0 não é −1, e a escala total não é +1.

A Chloros marca ambos os extremos para que o mapeamento permaneça invertível:

| Tag | Significado |
| --- | --- |
| `Chloros:IndexScaling` | `linear_minmax` — a convenção abaixo |
| `Chloros:IndexMin` | o valor de índice armazenado como DN 0 |
| `Chloros:IndexMax` | o valor do índice armazenado como escala total |

Leia-os e inverta, normalizando o DN armazenado para 0…1 pelo máximo do tipo de dados (dtype) armazenado, de modo que uma única expressão abranja todos os formatos inteiros:

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

> **`IndexMin == IndexMax`** significa que cada pixel daquele quadro possuía o mesmo valor de índice. A trama é preenchida em escala média e não há nada pelo qual dividir — use o valor extremo.

> **A ausência de tags `Chloros:Index*` significa que não há mapeamento definido.** Uma exportação de LUT é uma renderização colorida cujos pixels são cores em gradiente vinculadas à sua janela de limiar de LUT, e um quadro sem pixels de índice finito não possui intervalo a ser declarado. Não presuma um mapeamento em nenhum dos casos — a mesma regra que se aplica à ausência de `PixelScale`.**Como o alongamento é por imagem, dois quadros de um voo não são comparáveis em termos de DN**— cada um carrega seus próprios pontos finais, de modo que o mesmo NDVI resulta em valores de DN diferentes em imagens diferentes. Para trabalhos quantitativos em um voo (estatísticas zonais, limiares fixos, séries temporais), exporte**`TIFF (32-bit, Percent)`**: as rasters de índice são gravadas ali como float32**os próprios valores de índice**, sem necessidade de reescalonamento nem tags. Apesar do nome do formato, os produtos de índice não são ×100 — essa escala se aplica apenas à refletância com tipo de dado inteiro.

#### Máscara de limitação da separação espectral — em quais pixels uma banda foi limitada

A separação espectral 3×3 pode tornar uma banda negativa — onde a luz mal contém essa banda, ou onde a cena fica fora do que a matriz de separação da câmera descreve. Esse não é um valor real, portanto a exportação o fixa em 0 — e a visualização ao vivo mostra o mesmo 0. Uma banda pode ser limitada a alguns pixels ou a quase todos eles; a máscara registra qualquer uma das situações. `Chloros:Band{R,G,B}UnmixClampFraction` indica em que proporção do quadro isso ocorreu; a máscara de limitação indica **quais pixels**:

- **`clamp_masks/<product folder>/<product name>_clampmask.tif`**— em uma pasta `clamp_masks/`**ao lado** da pasta do produto, nunca dentro dela (`radiance/x_radiance.tif` → `clamp_masks/radiance/x_radiance_clampmask.tif`) — para cada produto de radiação, refletância e índice no qual pelo menos um pixel sofreu limitação. `uint8` de banda única, na própria grade de pixels do produto (orientação, ROI, binning e a distorção de alinhamento da matriz incluídos). Cada máscara nomeia a câmera do seu produto (`Make` / `Model` / serial), a hora da captura e as tags da lente, além de listar as bandas que seus bits representam (`XMP-Camera:BandName`, bit 0 primeiro); seus bits `ImageDescription` e `Software` a identificam como uma máscara de limitação, e ela não contém calibração.
- **Bits**: bit 0 = a banda no canal R do produto (`Chloros:BandRName`), bit 1 = G (`BandGName`), bit 2 = B (`BandBName`). Um bit definido significa que o valor do pixel nessa banda foi limitado a 0 — trate-o, e qualquer índice construído a partir dele, como de baixa confiança.
- **`Chloros:UnmixClampMask`** é gravado em todos os produtos que executaram a separação de cores: o caminho da máscara em relação à própria pasta do produto (`../clamp_masks/<product folder>/<product name>_clampmask.tif`, sempre `/`separado — junte-o à pasta da imagem para abrir a máscara), ou `none` quando nenhum pixel foi limitado. A ausência de marcação significa que o produto nunca executou o processo de separação de cores (câmeras RGB ou monocromáticas, níveis raw/sem debayering).
- Os resultados da captura apresentam isso como o **`clamp_mask`** do quadro (o caminho da máscara, ou `None` quando nada foi limitado) — nos resultados `CameraSession.capture()`, os quadros `ArraySession.capture()`'nos quadros `CaptureResult` e nas respostas de captura HTTP. A compilação offline lista suas máscaras como `clamp_masks`.
- A pilha multibanda de um índice ao vivo (`/api/camera/cli-array-index` com `save_multiband`) recebe uma máscara por câmera de origem, na própria pasta de saída `clamp_masks/` — consulte a referência da CLI`lattice index`. Um vídeo não possui máscara: o resultado da compilação offline relata, em vez disso, a fração de limitação de cada quadro.

Como nenhuma pasta de produto contém uma máscara, em qualquer nível hierárquico, uma pasta de produto pode ser entregue ao software de fotogrametria tal como está — incluindo subpastas — e ele processa apenas as imagens. Entregue a pasta de produto, não a pasta acima dela. O próprio importador do Chloros ignora as máscaras.

```python
import os, numpy as np, tifffile

mask = tifffile.imread(
    "clamp_masks/radiance/capture_<ts>_SN<serial>-<filter>_radiance_clampmask.tif")
# or, from the image: os.path.join(os.path.dirname(image_path), tag), with
# tag = the image's Chloros:UnmixClampMask
r_clamped = (mask & 1) != 0      # the band Chloros:BandRName names
g_clamped = (mask & 2) != 0
b_clamped = (mask & 4) != 0
```

---

## Smart-Connect para câmeras LATTICE

Sessões persistentes de backend para hardware em tempo real. Os mesmos endpoints que a GUI utiliza, portanto o comportamento é idêntico entre SDK, CLI e GUI.

### Câmera única — `CameraSession`

```python
import chloros_sdk

# Open by serial; reuses existing pool entry if one exists
with chloros_sdk.connect_camera("213800234") as cam:
    # cam is a CameraSession; supports context manager + manual disconnect
    cam.set_settings(
        exposure_time=10000,    # microseconds
        gain=0.0,               # dB
        pixel_format="BayerRG12",
        target_brightness=80,
        ae_damping=8.0,
    )
    cam.capture("output/", ext=".tiff")
```

A exposição, o ganho e o alvo definidos aqui são mantidos como seus: o controle automático de exposição do Chloros (incluindo a exposição medida pelo DAQ) não os altera. `cam.set_settings(exposure_auto="Continuous")` (ou `gain_auto=`) devolve a exposição (ganho), e o `cam.set_settings(target_brightness_auto=True)` devolve o alvo ao sistema automático de alvo dChloros.

#### `connect_camera()` Assinatura

```python
connect_camera(
    serial,
    *,
    preset=None,                       # "default" | "high_quality" | "high_speed" | "triggered"
    settings=None,                     # dict overlaid on the preset
    backend_url="http://127.0.0.1:5000",  # deliberately not 'localhost' (::1-first on Windows ≈ 2 s/request)
    timeout=60.0,
    auto_start_backend=True,           # spawn a local backend if none is running
) -> CameraSession
```

- **Ele retorna com o fluxo iniciado.** O backend aguarda, com um limite (cerca de 15 s para o stream iniciar e, em seguida, 10 s para seu primeiro buffer completo — não para o primeiro quadro de pré-visualização, que surge cerca de 1,5 s depois), portanto, um `status()` logo após indica `streaming`. O `connect_stream` da sessão é o bloco `stream` da resposta: `state` `streaming`, `no_frames` (o fluxo foi iniciado e não entregou nada — o `note` indica isso e, para uma câmera com pacotes jumbo, para se reconectar com um tamanho de pacote de 1.500 bytes), `starting` ou `start_failed`, com `waited_s`. `None` para uma câmera que já estava conectada. Uma pré-visualização (`get_camera_preview`, `/api/camera/jpeg/<serial>`) solicitada logo após a espera, limitada a 4 s, pelo primeiro quadro de pré-visualização da câmera, em vez de responder “Ainda não há quadro disponível”; uma câmera cujo fluxo não tenha transmitido nada (`no_frames`, ou armada para um gatilho de hardware) ainda responde imediatamente.
- **O tamanho do pacote é definido por você.** `settings={"gev_packet_size": 1500, "gev_packet_size_auto": False}` abre a câmera com pacotes de 1.500 bytes, sem sondagem (`chloros-cli lattice cam-connect --packet-size standard` envia o mesmo).
- **Apenas uma conexão por vez ocupa o pool de câmeras**: a abertura, a verificação do tamanho do pacote, um primeiro download de calibração (cerca de 70 s para uma câmera nova neste computador) e o início da transmissão. Uma solicitação que precise de uma câmera nesse meio tempo (`read_nodes`, `set_settings`, uma captura, uma segunda conexão) aguarda por ela por no máximo 5 s; em seguida, falha com o código HTTP 503 `CAMERA_POOL_BUSY`, indicando o nome da câmera que está se conectando (`ChlorosConnectError`); pergunte novamente em alguns instantes. As listagens nunca esperam: `list_cameras()` (`GET /api/camera/pool`), `list_arrays()` e `CameraSession.status()` respondem imediatamente. Uma câmera em meio à conexão é listada sob a resposta do pool `connecting` (`serial`, `phase` — `opening the camera`, `rebalancing the pool bandwidth`, `loading its calibration`, `starting its stream`, `waiting for its stream` — e `elapsed_s`), e seu `status()` indica o `connecting: True` com seu `phase`; as demais câmeras são lidas em tempo real quando o pool está livre; caso contrário, respondem com o valor da última leitura (`live: False`).

#### Métodos `CameraSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=15.0)` | O estado ao vivo da câmera, conforme lido por `chloros-cli lattice status`: `streaming`, `stream_state` (`starting`, `no_frames`, `streaming`, `start_failed` ou `stopped`), `device` (modelo, número de série, firmware, temperatura, `is_mono`) e suas configurações atuais. |
| `read_nodes(names, enum_names=(), timeout=30.0)` | Lê os nós GenICam; retorna `{nodes, errors, enums, device, was_in_pool, in_array_id}`, além de `frame_values` e `notes` enquanto a exposição automática estiver em execução (veja abaixo). |
| `set_settings(**kwargs)` | Grava nós pelo nome amigável (`exposure_time`, `gain`, `pixel_format`, `width`, `height`, `target_brightness`, `target_brightness_auto`, `ae_damping`, `ae_upper_limit`, `trigger_mode`, `trigger_source`, …). Um `exposure_time` (`gain`) é uma exposição manual (ganho): escrito sem `exposure_auto` (`gain_auto`), o modo automático deve ser desativado primeiro, para que o valor não seja rejeitado pelo modo automático em execução. O resultado, `applied`, mantém o que a câmera capturou; uma gravação rejeitada pela câmera está em `errors` e não é mantida. O erro indica o intervalo que a câmera aceita agora (um valor máximo em `ExposureTime` segue a taxa de quadros da câmera), e um modo automático que foi desativado apenas para o valor recusado é reativado; `restored` lista isso (por exemplo, `{"ExposureAuto": "Continuous"}`). |
| `capture(output_dir="output", ext=".tiff", jpeg_quality=95, processing=None, levels=None, force_daq=None, daq=None, daq_tolerance_ms=None, settings=None, timeout=None)` | Captura um **único** quadro. Retorna um `CaptureResult`: uma lista de dicionários de metadados de quadros (um por nível salvo), com `.skipped`, `.failed`, `.daq_files` e `.daq_matches`. (A captura em sequência/multi-quadro foi removida — chame `capture()` em um loop se precisar de uma série.) |
| `capture_repeated(output_dir="output", *, count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Capturas consecutivas (`interval_s=0`) ou com intervalo desta câmera, para capturas `count` e/ou `duration_s` (pelo menos uma, para que termine), assim como o `ArraySession.capture_repeated()` faz para uma matriz. Retorna a lista de `CaptureResult`s; ela para após uma captura que informe que a unidade está cheia. `on_capture(i, result)` é executado após cada uma; outras palavras-chave vão para `capture()`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Liberação do pool; a câmera é restaurada ao estado em que foi encontrada na conexão (fiação do disparador, PTP, configurações); o `keep_settings=True` salva suas configurações em sua própria memória; o `clear_settings=True` reinicializa essa memória (consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings)); retorna `camera_memory` quando qualquer uma das opções for solicitada. Uma câmera que seja membro de um array libera todo o array. Não realiza nenhuma operação se estivermos conectados a uma sessão já aberta. Retorna assim que o backend confirmar a liberação ou quando não houver nada a ser liberado (a câmera já foi removida ou nenhum backend está em execução); uma liberação que atingiu o tempo limite ou que foi recusada pelo backend gera o erro `ChlorosConnectError`, pois a câmera ainda pode estar ocupada e a próxima conexão entraria em conflito com ela. A saída do bloco `with` nunca gera erro. |

Atributos: `serial`, `device` (o registro de descoberta da câmera) e `backend_url`.

Controles de exportação do `capture()` (mesmo modelo da matriz + GUI):

- `processing` / `levels` — `processing="all"` salva todos os tipos de exportação aplicáveis; `levels=["raw","radiance"]` salva apenas esses (substitui `processing`). Omita ambos para usar o padrão do backend.
- `daq` — a leitura do DAQ usada na captura é salva como um arquivo sidecar `.daq` por padrão, incluindo uma captura apenas em formato bruto, para que o quadro possa ser reprocessado posteriormente em refletância/índice. `daq=False` ignora isso quando nenhum nível o utiliza. `force_daq=True` é a grafia antiga do padrão. Sem DAQ, sem `.daq`.
- `daq_tolerance_ms` — a distância máxima que a leitura pode estar da exposição e ainda estar dentro da tolerância (padrão 500). Apenas o veredicto muda.

#### `read_nodes()` na exposição automática — `frame_values` e `notes`

Enquanto o `ExposureAuto` (`GainAuto`) está em execução, o nó `ExposureTime` (`Gain`) lê de volta o **último valor gravado**, e não o que a câmera está fazendo. O `nodes` ainda mantém esse valor do nó. Quando você solicita `ExposureTime` ou `Gain` em uma câmera conectada cujo modo automático não seja `Off`, a resposta acrescenta:

- `frame_values` — o valor contido no quadro mais recente da câmera, sob o nome do nó (`ExposureTime` em µs, `Gain` em dB);
- `notes` — uma frase por cada um desses nós, informando ambos os valores.

```python
r = cam.read_nodes(["ExposureTime", "Gain"])
print(r["nodes"]["ExposureTime"])                  # last value written, e.g. 2000.448
print(r.get("frame_values", {}).get("ExposureTime"))  # what the newest frame used, e.g. 19600.0
for name, note in r.get("notes", {}).items():
    print(note)
# ExposureAuto is Continuous: ExposureTime holds the last value written (2000.448);
# the camera's newest frame was at 19600 us.
```

Ambas as chaves estão ausentes quando o modo automático é `Off` (o nó é, então, ovalor real da câmera) e para uma câmera que não esteja conectada ao backend. Para uma matriz, `ArraySession.status()["member_ae"]` fornece os valores de quadro de cada membro de uma só vez.

#### Limite máximo de exposição medido pelo DAQ (HTTP)

Com um sensor de luz DAQ conectado, o Chloros limita a exposição × ganho de uma câmera com base na luz do dia medida, para que um painel de referência branco não sofra clipping. O limite se aplica apenas enquanto um alvo de calibração estiver no campo de visão daquela câmera (detectado em seu quadro nos últimos 10 s); sem nenhum alvo no campo de visão, a exposição automática normal da câmera, com sua própria proteção contra destaques, é ativada, para que o restante da cena não fique mais escuro do que o necessário. Não há um wrapper de SDK; leia ou altere essa configuração por câmera usando `GET` / `POST /api/camera/<serial>/daq-exposure-cap` (o corpo da solicitação POST com `{"enabled": false}` desativa essa função para essa câmera). A resposta contém `enabled`, `cap_us` (o limite máximo em tempo real; `None` quando inativo, inclusive quando nenhum alvo de calibração está no campo de visão), `k_exp_g` (a constante com a qual o limite máximo é calculado) e:

| Campo | Significado |
| --- | --- |
| `k_exp_g_source` | Qual constante está em uso: `camera` — a própria desta câmera, calculada a partir do ponto de laboratório em seu pacote de calibração (câmeras RGB); `fleet` — a constante da frota, usada quando o pacote não possui ponto de laboratório; `override` — uma substituição de campo definida no backend. |
| `k_exp_g_fleet` | A constante da frota, para comparação. |

### Matriz Sincronizada — `ArraySession` (Smart-Prep)

O `connect_array` é **o ponto de entrada recomendado** para configurações com várias câmeras. Ele executa, nos bastidores, todo o fluxo do Smart-Prep por meio da interface gráfica do usuário (GUI):

1. **Análise de rede**(`/api/camera/array/recommend`) — configura o**sensor completo** (sem binning, sem recorte de ROI) e permite que a taxa de quadros caia para o que este host e a rede oferecem, projetada com os formatos de pixel e o empacotamento que o Connect utilizará. Os quadros nunca são reduzidos para manter a taxa alta: passe um `width` menor / `height` ou `binning=2` / `4` para trocar resolução por taxa de quadros (a lista do endpoint`alternatives` lista essas opções com seus fps projetados).
2. **Seleção automática de nível** — `sim-capture-sim-emit` (todas as câmeras expõem no mesmo gatilho, com emissão sincronizada para compartilhar a linha, e a taxa de quadros cai à medida que o compartilhamento diminui) em qualquer rede em que as câmeras possam ser sincronizadas para compartilhar; `slip-emit-and-capture` (as câmeras se revezam na banda, sem exposição simultânea) apenas quando isso não for possível.
3. **PTP habilitado**por padrão — os carimbos de data/hora entre câmeras se alinham a um relógio compartilhado com precisão de**~1 ms**, o [grandmaster PTP](#ptp-grandmaster--time_sync_status) do backend. A exposição simultânea é proporcionada pelo disparador de hardware M8 (**< 100 µs** entre módulos), e não pelo PTP: o PTP alinha *carimbos de data/hora*, não exposições.
4. **Seleção automática do formato de pixel por câmera** — monocromático → `Mono12`, colorido → `BayerRG12`.
5. **Inicialização do AE** — captura instantânea do estado atual do AE de cada câmera para que a conexão não reinicie a exposição durante a operação.
6. **Configuração do disparador GPIO** — `connect_array` arma todas as câmeras (`TriggerMode=On`, `TriggerSource=Line2`) para que o pulso da mestreimpulso do mestre acione as escravas pelo cabo M8. Esta é uma etapa exclusiva para matrizes: uma única câmera aberta com `LatticeCamera` opera em modo livre. Ao liberar a matriz (`disconnect()` / `force_disconnect()`, `chloros-cli lattice array-disconnect`) faz com que todas as câmeras voltem ao modo de funcionamento livre (`TriggerMode=Off`), mantendo todas as demais configurações exatamente como estavam no momento da conexão: as configurações de disparo e da linha de sincronização (`TriggerSource`, `TriggerActivation`, `TriggerOverlap`, `TriggerDelay`, `LineMode` / `LineSource`; a linha de sincronização da câmera principal de volta para uma entrada), PTP e as configurações da câmera alteradas durante a sessão — a menos que `keep_settings=True` / `clear_settings=True` (consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings)).

```python
import chloros_sdk

# First serial is the MASTER (fires the trigger pulse); rest are slaves.
with chloros_sdk.connect_array(
        ["213800234", "214000533", "214701288", "214701292"]) as arr:
    print(arr.array_id, arr.sync_mode, arr.ptp_enabled)
    arr.capture("output/", processing="reflectance")
```

Atributos: `array_id`, `master_serial`, `slave_serials`, `ptp_enabled`, `sync_mode`, `packed_transport` e `packed_reason` (os quatro últimos são `None` em um identificador de `attach_array`), `connect_stream` e `backend_url`. `connect_stream` é o bloco `stream` da resposta de conexão, a forma da conexão única mais `members` (`{serial: {state, why?}}`): `streaming` assim que o primeiro grupo sincronizado da matriz chegar (o backend aguarda por ele, com limite de cerca de 10 s), ou `no_frames` com a última falha de busca de cada membro como seu `why` e, para pacotes jumbo, o fallback de 1.500 bytes no `note`. `None` em um identificador `attach_array`.

#### Assinatura do `connect_array()`

```python
connect_array(
    serials,                              # list[str]; serials[0] = master
    *,
    line="Line2",                         # GPIO sync line: Line0 | Line2 | Line3
    target_fps=None,                      # pin the master trigger rate (auto if None); see set_target_fps
    force_tier=None,                      # override tier picker; see below
    wire_ceiling_mbps=None,               # host sustained wire budget, MB/s (auto if None)
    width=None,                           # explicit frame size; skips network analysis
    height=None,
    pixel_format=None,
    binning=None,
    recommend=True,                       # set False to skip the recommend step
    ptp_enable=True,                      # set False to disable PTP
    packed="auto",                        # packed 12-bit transport: "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",  # same IPv6-avoidance default as connect_camera
    timeout=600.0,                        # s; also the backend's deadline (see below)
    auto_start_backend=True,              # spawn a local backend if none is running
) -> ArraySession
```

O `timeout` representa o tempo que a chamada aguarda pela conexão e é enviado ao
backend como seu prazo limite. Uma conexão não concluída até esse momento é revertida: todas as câmeras que ela
abriu são restauradas ao estado em que foram encontradas e fechadas, nenhum array é conectado, e a
chamada gera o código de erro `ChlorosConnectError` (HTTP 504 `CLIENT_DEADLINE_PASSED`, indicando a
etapa em que a conexão havia chegado). Portanto, uma chamada que gera exceção nunca deixa uma matriz
conectada a ela. Um conjunto de 9 câmeras se conecta em cerca de um minuto; logo após
as câmeras serem ligadas, isso pode levar vários minutos. Um conjunto que está ativo, mas ainda não possui
um grupo de quadros sincronizado, é devolvido com o código de erro `connect_stream["state"]`
`no_frames`. O log do backend contém uma linha de tempo por etapa de conexão
(`[array-connect <id>] <step>: <s> s …`) e uma última linha indicando a mais lenta.

Valores de `force_tier`:
- `"sim-capture-sim-emit"` — verdadeiramente simultâneo (todas as câmeras disparam na mesma borda do clock).
- `"sim-capture-ftd-stagger"` — escalonamento flexível no domínio do tempo (as câmeras disparam em momentos ligeiramente deslocados, de modo que os pacotes são serializados na linha).
- `"slip-emit-and-capture"` — captura sequencial por câmera (sem sincronização temporal). Escolhida automaticamente apenas quando as câmeras não podem ser ajustadas a uma velocidade baixa o suficiente para compartilhar a linha simultaneamente.

`wire_ceiling_mbps` substitui o **orçamento sustentado de banda do host** em MB/s — o único
número do qual toda a alocação da matriz depende. Deixe como `None` para usar o valor detectado automaticamente. Reduza-o quando o array relatar quadros corrompidos pelo GVSP: o valor automático é derivado
da taxa de link anunciada pela placa de rede, que superestima adaptadores USB, pistas PCIe estreitas e
estruturas compartilhadas ocupadas — e essa superestimativa se manifesta como quadros corrompidos, em vez de como um
link visivelmente lento. O valor é mantido no bloco de captura da matriz do projeto; portanto, uma
reabertura ou um `connect_array` posterior o restaura como qualquer outra configuração da matriz.
Consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames).

#### Transporte compactado de 12 bits (`packed=`)

Com todos os sensores em operação, a taxa de quadros de uma matriz é definida pela velocidade com que cada câmera pode enviar
seus quadros. Formatos de 12 bits (`BayerRG12`, `Mono12`) normalmente são transmitidos como palavras de 16 bits;
o transporte compactado envia os mesmos 12 bits em 1,5 bytes por pixel e
os descompacta no host, de modo que cada quadro leva 25% menos tempo para ser enviado — cerca de
**33% mais quadros por segundo com o sensor operando em capacidade total** (medido: 10,7 → 14,0 fps em um
conjunto de 9 câmeras). **Os dados salvos são idênticos**: mesmos valores, mesmos arquivos de 16 bits, mesmos metadados.

- O `packed="auto"` (padrão) compacta apenas quando isso ajuda **e** o host tem
  recursos para a descompactação: na conexão, o backend mede sua própria velocidade de descompactação,
  o número de núcleos da CPU, a carga atual e a largura de banda disponível. Um desktop com núcleos de sobra
  compacta na velocidade máxima do sensor; um host ocupado ou com baixo consumo de energia (computadores de placa única,
  controladores embarcados) transmite sem compactação. Pequenas ROIs e quadros agrupados ganham pouco com isso.
- `packed="off"` (ou `False`) nunca compacta — para descartar essa opção durante o diagnóstico,
  ou em um host que precise reservar sua CPU para outras tarefas.
- `packed="on"` (ou `True`) sempre compacta formatos de 12 bits.
- Qualquer outro valor gera o código de erro `ValueError`. Formatos de 8 e 16 bits nunca são
  compactados.

O `analyze_array_network(..., packed="auto")` projeta com a mesma decisão,
portanto, seu fps é a taxa na qual a matriz será executada (a resposta contém
`packed_transport` e `packed_reason`); seu `pixel_format` assume como padrão o que
o `connect` aplica (Mono12 / BayerRG12) em vez de 8 bits.

A decisão e sua justificativa estão na sessão:

```python
arr = chloros_sdk.connect_array(serials)          # packed="auto"
print(arr.packed_transport, arr.packed_reason)
# True  auto: projected 11.0 -> 14.4 fps, unpack 0.16 of 1.50 budgeted cores

arr = chloros_sdk.connect_array(serials, packed="off")
```

`ChlorosProject.connect_all(packed=...)` aplica a mesma substituição a todas as
matrizes de um projeto (padrão `None`, o que significa `"auto"`: um projeto não salva nenhuma
configuração compactada por matriz).
Para um padrão válido para toda a máquina, defina `CHLOROS_ARRAY_PACKED12=off` (ou `on`) no
ambiente do backend; uma configuração explícita como `packed="on"` / `"off"` ainda prevalece.
`/api/camera/array/status` reporta `packed_transport` e `packed_decision`.

#### Taxas de quadros esperadas

Taxas de quadros da matriz medidas (sincronizadas por hardware, 12 bits; sensor completo = 2048 × 1536). Use-as para verificar a validade do `status()["actual_fps"]`; o `analyze_array_network()` projeta a taxa para o seu próprio host e rede.

| Configuração | Taxa de quadros da matriz |
| --- | --- |
| Sensor completo, placa de rede PCIe 10 GbE no host, transporte compactado (`packed="auto"` em um desktop) | ~14 fps |
| Sensor completo, placa de rede do host PCIe 10 GbE, sem compactação (`packed="off"`) | ~10,5 fps |
| Sensor completo, 8 câmeras em um adaptador USB 10 GbE | ~3,8 fps |
| Sensor completo, link host de 1 GbE | 1,2–1,7 fps |
| Binning 2 × 2 (`binning=2`), placa de rede host PCIe 10 GbE | ~30–33 fps |

- Em um link PCIe 10 GbE, as **câmeras** são o limite na configuração de sensor completo, não a rede: 4 e 8 câmeras operam na mesma taxa.
- **`target_fps` é a taxa de disparo, e o conjunto nunca opera mais rápido.** `target_fps=1` opera o conjunto a 1 fps; `None` (o padrão) opera-o na taxa mais alta que o host e a rede suportam. Qualquer valor acima dessa taxa é reduzido para ela (consulte [`set_target_fps`](#interval-capture-for-drone-surveys)).
- **`link_speed_mbps` de `4294` em `status()` não é uma taxa de link.** Isso significa que o Windows não conseguiu ler a velocidade do adaptador (o driver informou o maior valor de 32 bits), o que é comum em adaptadores Ethernet USB. A taxa utilizável de um adaptador USB no Windows é limitada a 200 MB/s, independentemente da situação (consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames)); para ajustar um adaptador, consulte a seção “Adaptadores USB 10GbE” da referência da CLI em [Configuração e ajuste da NIC do host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **Um limite para USB também se aplica ao Linux: 400 MB/s no x86_64, 200 MB/s no NVIDIA Jetson.** O adaptador é identificado a partir do sysfs (recorrendo ao `ethtool -i`), portanto, um adaptador USB de 5 GbE ou 10 GbE é limitado a esse limite, e não à sua capacidade nominal de 500 / ~1060 MB/s. O valor para x86_64 foi medido no Linux: o mesmo adaptador que corrompe quadros acima de 200 MB/s no Windows operou um conjunto de 4 câmeras sem erros em todos os limites até 480 MB/s. No NVIDIA Jetson, o mesmo adaptador funcionou sem erros apenas a 200 MB/s e perdeu quadros de forma intermitente acima desse valor. Se o seu host absorver mais sem que haja perda de quadros com o `gvsp_corrupt`, aumente-o com o `wire_ceiling_mbps`. No Linux, um adaptador conectado a uma porta USB 2.0 tem uma largura de banda limitada a 30 MB/s; conecte-o a uma porta USB 3.

#### Sobrescrição (o limite mínimo por câmera)

Um array funciona em qualquer rede; a taxa de quadros é o que determina o desempenho. O ritmo de emissão simulada aloca a cada câmera uma parcela igual do orçamento de banda à prova de colisões, e todas elas ainda disparam no mesmo gatilho; portanto, em uma conexão lenta, a parcela — e a taxa de quadros — é pequena: 8 câmeras deem resolução total de 12 bits em 1 GbE sem quadros jumbo recebem 6,5 MB/s cada, cerca de **1 fps**(cerca de 1,3 fps compactados). A parcela tem um limite mínimo na taxa mais lenta na qual as câmeras podem ser sincronizadas (`per_cam_floor_bps`: lido na faixa GevSCPD de cada câmera na conexão, e nunca abaixo de 1 MB/s). Se a conexão exigir uma velocidade menor do que essa, a conexão recorre à transmissão**serial**(`slip-emit-and-capture`): as câmeras se alternam na taxa máxima do link, de modo que todos os quadros chegam, mas os membros**não são expostos simultaneamente**— o nível de resposta e os bloqueadores (`SERIAL: ...`) indicam isso; passe `force_tier="sim-capture-sim-emit"` para recusar o recurso de fallback. O tamanho do quadro não influencia o compartilhamento, portanto**o binning e a ROI aumentam a taxa de quadros, mas não alteram se uma matriz cabe**. Apenas um canal que não consiga transportar a matriz em qualquer ritmo, seja simultaneamente ou alternadamente,**fica sobrecarregado** — o modo de falha é a perda de pacotes GVSP, não uma taxa de quadros mais baixa. Soluções: menos câmeras, quadros jumbo de ponta a ponta ou uma placa de rede mais rápida.

- As respostas `analyze_array_network()` e `/api/camera/array/connect` transportam `oversubscribed`, `aggregate_demand_bps`, `collision_safe_ceiling_bps`, `max_cams_collision_safe` e `per_cam_floor_bps`. Quando `oversubscribed` é verdadeiro, a projeção **zerou os campos de fps** (`achievable_fps_max` / `fps_bright` / `fps_dark`) em vez de relatar uma taxa enganosa de “lenta, mas funcionando”.
- `POST /api/camera/array/connect` aceita um parâmetro de corpo `pin_resolution` (**somente HTTP — não é um kwarg do SDK**; o `connect_array` não o expõe). Trata-se do modo estrito: uma conexão com excesso de assinantes e o parâmetro `pin_resolution` definido é**rejeitada categoricamente** com um erro que indica todas as soluções possíveis. Sem ele, a conexão prossegue com um aviso de que a matriz perderá pacotes. De qualquer forma, a conexão mantém o tamanho de quadro solicitado — ela nunca agrupa ou corta por conta própria.
- Apenas para testes: defina `CHLOROS_ARRAY_ALLOW_OVERSUBSCRIBED=1` no ambiente do backend para rebaixar a recusa a um aviso sonoro — você se conecta de qualquer maneira e aceita a perda de pacotes.

#### Integridade da matriz — qual subsistema está perdendo quadros

`ArraySession.capability()` (`GET /api/camera/array/<array_id>/capability`) carrega um bloco `health` ativo em uma
matriz conectada, reavaliado em uma janela contínua de **10 segundos** . Ele divide a perda de quadros
em duas causas que exigem correções opostas, em vez de uma única taxa de “incompletude” que
não especifica nenhuma delas:

| Campo | O que significa | Qual subsistema |
| --- | --- | --- |
| `gvsp_corrupt_rate_pct` (por serial) | O quadro **chegou e estava estruturalmente corrompido**— perda de pacotes GVSP. |**Rede**: largura de banda disponível, ritmo de transmissão, anel de recepção da NIC, MTU |
| `never_arrived_rate_pct` (por número de série) | O quadro **nem chegou**— a câmera não disparou ou nada saiu dela. |**Disparador / sincronização**: cabo M8, `line=`, `TriggerMode` |
| `worst_gvsp_corrupt_pct` / `worst_never_arrived_pct` | Pior taxa de cada câmera cada uma. | — |
| `per_cam_rate_pct` | Taxa combinada de falhas por câmera (ambas as causas juntas). | — |
| `stable_for_seconds` | Por quanto tempo cada câmera permaneceu abaixo de 0,01 %. | — |

Juntamente com `health`, o mesmo registro informa o número pelo qual toda a alocação fica pendente:

| Campo | O que significa |
| --- | --- |
| `wire_ceiling_mbps` | O orçamento de banda sustentado do host em vigor, em MB/s. |
| `wire_ceiling_source` | De onde esse número veio, em palavras — por exemplo, `USB-capped 200 MB/s (was theoretical 1062; …)` ou `user override 120 MB/s (auto said 200)`. |
| `wire_ceiling_is_user_set` | `true` quando `wire_ceiling_mbps=` o definiu. |
| `nic_is_usb` | `true` para um adaptador Ethernet USB. |

```python
import chloros_sdk

serials = ["213800234", "214000533"]           # master first
arr = chloros_sdk.attach_array(serials)
h = arr.capability()

health = h.get("health", {})
print("wire ceiling:", h["wire_ceiling_mbps"], "MB/s", h["wire_ceiling_source"])
print("corrupt (network) :", health.get("worst_gvsp_corrupt_pct"), "%")
print("absent  (trigger) :", health.get("worst_never_arrived_pct"), "%")

if (health.get("worst_gvsp_corrupt_pct") or 0) > 1.0:
    # Network path. Reconnect with a lower budget -- NOT a lower target_fps.
    # force_disconnect(), not disconnect(): this handle came from attach_array,
    # and disconnect() never tears down an array the session only attached to.
    # It returns only once the backend confirms the release; a teardown that
    # timed out or was refused raises ChlorosConnectError instead, so the
    # reconnect below never races a half-released array.
    arr.force_disconnect()
    arr = chloros_sdk.connect_array(serials, wire_ceiling_mbps=120)
```

**Interpretação:** um valor diferente de zero para `gvsp_corrupt_rate_pct` com `never_arrived_rate_pct` igual a 0 significa
que o disparo e a sincronização do cabo estão perfeitos e que 100% da perda ocorre no caminho da rede — diminua
`wire_ceiling_mbps` e reconecte. O padrão inverso aponta para o cabo de sincronização ou para a
linha de disparo.

> **O `target_fps` não é o fator determinante para quadros corrompidos.** O ritmo do GevSCPD é definido uma vez na
> conexão; portanto, diminuir a taxa de disparo altera o ciclo de trabalho e não a
> taxa de rajada de emissão simultânea. Uma redução medida de 5× na demanda não produziu melhora, enquanto
> reduzir o limite máximo da conexão de 240 para 200 MB/s levou o mesmo equipamento de 10,4 % de quadros corrompidos para
> 0,00 %.

> **Um array em execução não pode se replanejar sozinho no firmware LATTICE (M3C / M3M).** Ele não pode
> corrigir isso sozinho; desconecte e reconecte para que o seletor de tempo de conexão se replaneje de acordo com
> o novo limite máximo.

Um **adaptador Ethernet USB é limitado a 200 MB/s** pela sonda, independentemente de sua
placa de identificação: a tabela de eficiência que transforma uma taxa de link em um valor sustentado é
derivada do PCIe, e uma placa de rede USB anuncia sua taxa de link Ethernet, embora esteja limitada pelo
barramento USB e seu driver. O limite é absoluto, não uma fração — um adaptador USB 1 GbE
atinge ~80 MB/s e não é afetado. Isso se aplica ao Windows e ao Linux (incluindo o Jetson);
consulte [Taxas de quadros esperadas](#expected-frame-rates) para saber onde isso foi medido.

#### Métodos `ArraySession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Estado em tempo real da matriz em execução: `target_fps` / `target_fps_auto` (a taxa de disparo e se o controlador de taxa a possui), `target_fps_clamped_reason` (por que uma taxa fixada é mantida mais baixa; caso contrário, `None`), `actual_fps` (medido; 0 assim que o loop de captura desistir ou travar), `recommended_fps`, `streaming` (False assim que o loop de captura desistir — verifique isso, não apenas `last_error`; permanece True após uma reinicialização das configurações, o que é sinalizado por `restarting`), `stalled` (True enquanto o loop de captura estiver em execução, mas não tiver publicado nenhum grupo de quadros por cerca de dez períodos de quadro, 2–30 s; `last_error` indica isso somente após 60 s), `last_frame_age_s`, `last_error`, `serials`, `ptp_enabled` (`True` apenas quando a matriz estava conectada via PTP **e** todos os membros leram `PtpEnable` pela última vez; não é o mesmo que “bloqueado”, o que `ptp_epoch` mede), `ptp_configured` (o que a conexão solicitou), `ptp_off_members` / `ptp_unread_members` (membros que leram `PtpEnable` como desativado pela última vez / não responderam), `member_recovery` (`None`, ou um membro que perdeu alimentação ou seu link: `state`, `waiting` enquanto não responde, `reopening`, em seguida, `recovered`, com `members` e `why` — a matriz o reabre quando ele responde novamente e devolve seu PTP de tempo de conexão, configurações de acionamento e fiação da linha de sincronização, ritmo e câmera; em seguida, transmite novamente sem novos `connect_array`) e `ptp_epoch` (medido), `sync_mode` / `sync_tier`, `packed_transport` / `packed_decision`, `per_cam_emit_mbps`, `link_speed_mbps` (`4294` significa que o Windows não conseguiu ler a velocidade do adaptador — consulte [Taxas de quadros esperadas](#expected-frame-rates)), `trigger_pipeline` / `pipeline_tune`, `radiometric_chain` (se a cadeia de radiação por banda está utilizável e qual membro a interrompeu) e `band_clamp` — enquanto uma visualização ao vivo renderiza a matriz: a proporção de pixels iluminados que a separação de cores limitou a 0, por elemento (`members`: `{serial: [band0, band1, band2]}` na ordem das bandas R, G, B da câmera) e, por banda, o que a imagem composta mostra (`composite`: `{band name: share}`), com `warn`, a proporção que a interface gráfica sinaliza como “Dados de banda de baixa confiança” (medidos aproximadamente uma vez por segundo; `None` quando não há visualização ao vivo em execução), e `member_ae` — por membro, a exposição (`exposure_us`) e o ganho (`gain_db`) que seu último quadro publicado continha, a partir do próprio quadro (`read_nodes` de `ExposureTime` / `Gain` retorna o último valor gravado enquanto a exposição automática está em execução, não o que está sendo feito). |
| `set_member_settings(serial, **kwargs)` | Grava as configurações de uma câmera membro, conforme `CameraSession.set_settings()` as obtém (exposição, ganho, brilho alvo, ...); o membro permanece na matriz. Gera uma exceção `ChlorosConnectError` para uma série que não seja um membro. Consulte [Exposição em membros da matriz](#exposure-on-array-members). |
| `capability(timeout=10.0)` | O que o smart-prep definiu (nível de captura, tamanho do quadro, `pixel_format`), o bloco `health` ativo e o orçamento de banda do host. Consulte [Integridade do array](#array-health--which-subsystem-is-losing-frames). |
| `set_target_fps(fps, timeout=10.0)` | Altere a taxa de acionamento enquanto o array estiver em operação, sem reconectar. Um número a fixa (o controlador de taxa não a altera); `None` o retorna para o modo automático. Retorna `{requested, target_fps, target_fps_auto, clamped}` — `clamped` é True quando a taxa foi reduzida para o que o host e a rede podem suportar. Consulte [Captura por intervalo para levantamentos com drones](#interval-capture-for-drone-surveys). |
| `grab(processing="debayered", with_metadata=False, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, bit_depth=None, band=None, align=False, refl_max=None)` | Um quadro de cada câmera, **todos a partir de um único acionador**, como matrizes numpy (sem gravação de arquivos). Retorna um `GrabResult` (um `dict`): `{serial: array}`, ou `{serial: (array, metadata)}` com `with_metadata=True`. `.group_id` é o disparador que os quadros compartilham; `.errors` mapeia cada câmera excluída para o motivo (uma câmera que não pode ser atendida nesse disparador é omitida, nunca preenchida por outra). `processing` é `raw` / `debayered` / `radiance` / `reflectance` / `display`. A mesma leitura que a [`ArrayHandle.grab()`](#one-trigger-per-read--group_id) da API do projeto — uma implementação compartilhada — portanto, todas as palavras-chave presentes ali (`band`, `align`, `bit_depth`, `refl_max`) funciona aqui. Requer `numpy`. |
| `frame_stream(*, processing="debayered", fps=5.0, duration=None, count=None, apply_calibration=True, apply_white_balance=True, apply_index=False, index_expression=None, with_metadata=False, bit_depth=None, band=None, align=False, refl_max=None)` | Iterador que retorna um resultado `grab()` por tick (somente argumentos de palavra-chave), com ritmo definido por `fps`, limitado por `duration` (segundos) e/ou `count` (`None` = ilimitado). **Cada conjunto retornado é um novo gatilho:** um tick cujo gatilho mais recente seja o conjunto que ele já gerou aguarda o próximo gatilho em vez de repeti-lo; assim, o fluxo é executado na menor das seguintes velocidades: `fps` ou a própria taxa de quadros da matriz. Gera o erro `FrameFetchError` em vez de ficar inativo: imediatamente quando todas as câmeras recusarem, por algum motivo, a repetição do próximo tick; após 60 s de ticks sem quadro; e após 60 s sem nenhum novo gatilho (“nenhum novo gatilho por 60 s”). Mesma implementação que `ArrayHandle.frame_stream()`. Consulte [Quadros ao vivo em seu próprio pipeline](#live-frames-in-your-own-pipeline). |
| `continuous(output_dir="output", *, format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, max_captures=None, duration_s=None, daq=None, daq_tolerance_ms=None, timeout=30.0)` | Inicia a captura contínua do próprio backend (o modo Contínuo da GUI) → [`ContinuousHandle`](#continuoushandle). Salva **todos** os grupos sincronizados que a matriz publica, da mesma forma que o `capture()` os salvaria (mesmos controles de exportação), até ser interrompido ou até que o `max_captures` / `duration_s` (`None` = até ser interrompido). Gera uma exceção `ChlorosConnectError` se já houver um em execução para essa matriz. |
| `capture(output_dir="output", format="tiff", processing="debayered", levels=None, aligned=None, render_index=None, force_daq=None, orient=None, smart=False, after_seq=None, daq=None, daq_tolerance_ms=None, timeout=300.0)` | Um grupo de captura sincronizado. Retorna um `CaptureResult` (lista de dicionários de quadros + `.skipped` + `.failed` + `.group_seq` + `.daq_files` + `.daq_matches` + `.alignment` + `.oriented`). Passe um resultado anterior`group_seq` como `after_seq` para obter o próximo grupo sincronizado após ele, em vez de aguardar um novo. Controles de exportação abaixo. |
| `capture(..., smart=True)` | **Captura inteligente** — aguarda a estabilização do AE em todas as câmeras e, em seguida, aciona. |
| `capture_fastest(output_dir="output", force_daq=True, render_index=True, orient=None, timeout=120.0)` | Captura mais rápida: somente dados brutos + a leitura DAQ atribuída (+ o índice combinado livre). Espelha o botão “Captura mais rápida” da GUI. |
| `capture_repeated(output_dir="output", count=None, duration_s=None, interval_s=0.0, on_capture=None, **capture_kwargs)` | Única / Contínua / Intervalo em um loop delimitado. Retorna `list[CaptureResult]`.**Requer `count` e/ou `duration_s`** para que seja encerrado (o SDK não possui Ctrl+C). Também termina antecipadamente, após a passagem, quando uma passagem relata uma unidade cheia (uma entrada `.failed` com `disk_full`); essa passagem é o último resultado. |
| `record(output_dir="output", fps=10.0, duration_s=None, video=True, gif=False, expr=None, timeout=30.0)` | Iniciar a gravação da visualização combinada do índice em tempo real como vídeo/GIF → `RecorderHandle`. O vídeo é um Motion-JPEG `.avi` (sem necessidade de ffmpeg). Sem nenhuma visualização da GUI aberta, o backend o renderiza: `expr` (uma expressão de índice nos nomes das bandas da matriz, por exemplo, `"(NIR_850_RGN-Red_660_RGN)/(NIR_850_RGN+Red_660_RGN)"`) ou o NDVI da matriz por padrão. Um gravador composto por matriz. |
| `burst(output_dir="output", duration_s=None, max_frames=None, index_config=None, serial_index_config=None, orient=None, timeout=30.0)` | Inicie uma sequência de fotos raw-Bayer com alta taxa de quadros → `RecorderHandle`. Reprocesse offline com `build_video()`. `orient=True`: os produtos gerados pelo `build_video()` são girados de acordo com a orientação da câmera (os produtos de cada câmera seguem a orientação da respectiva câmera, enquanto o índice combinado segue a orientação da câmera mestre); os quadros brutos permanecem em seu formato nativo e o manifesto da sequência registra a orientação de cada membro. |
| `build_video(burst_dir, products=None, fps=10.0, video=True, gif=False, save_tiffs=False, wait=True, poll_s=2.0, timeout=1800.0)` | Reprocessa offline uma sequência raw salva, convertendo-a em vídeo(s) calibrado(s). Bloqueia até a conclusão (`wait=True`) e retorna o resultado da compilação: `{success, outputs, errors, skipped, substitutions, notes, degraded, combined}` — consulte [o resultado da compilação](#the-build_video-result) abaixo. |
| `build_video_status(job_id, timeout=15.0)` | Verifica um trabalho de compilação offline: `{running, result, error, burst_dir}`. |
| `disconnect(keep_settings=False, clear_settings=False)` | Libera toda a matriz — **se esta sessão a tiver aberto** (`connect_array`): passa então a ser `force_disconnect()` e gera uma exceção `ChlorosConnectError` quando a liberação não for confirmada (a saída do bloco `with` nunca gera um erro). Em um identificador proveniente de `attach_array`, ele deixa a matriz em execução e imprime uma linha no stderr informando isso: a matriz pode pertencer à GUI ou a outro script. Cada câmera é deixada em funcionamento livre e, de resto, da mesma forma que a conexão a encontrou; o `keep_settings=True` salva suas configurações na própria memória da câmera, enquanto o `clear_settings=True` redefine essa memória para as configurações de fábrica (consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings)). |
| `force_disconnect(timeout=60.0, keep_settings=False, clear_settings=False)` | Libera todo o conjunto, independentemente de quem o tenha aberto. Use-o em um identificador conectado quando a intenção for encerrar o conjunto (por exemplo, para reconectá-lo com configurações diferentes). Retorna assim que o backend confirmar a liberação ou quando não houver nada a ser liberado (a matriz já foi removida ou nenhum backend está em execução); uma liberação que atingiu o tempo limite ou que foi recusada pelo backend gera o erro `ChlorosConnectError`. Com qualquer um dos sinalizadores, retorna o que a memória de cada câmera recebeu (`camera_memory`); caso contrário, retorna `None`. |

Controles de exportação do `capture()` (mesmo endpoint usado pela GUI/CLI):

- `processing` / `levels` — `processing="all"` (ou `levels=["raw","radiance",…]`) salva todos os tipos de exportação aplicáveis por câmera; um único valor `processing` salva apenas esse nível.
- `aligned` — **ativado por padrão (`None`), já que a captura do mesmo array pelo aplicativo Chloros**(sua configuração de captura**Alinhada** por matriz). A exportação não bruta de cada membro é alinhada ao quadro da câmera mestre pelo [perfil de alinhamento](#array-alignment) e recortada para a sobreposição comum dos membros, de modo que os arquivos são alinhados por pixel e apresentam o tamanho, o recorte e as tags `Alignment*` que o aplicativo utiliza (dentro de uma ROI, a transformação é reajustada com base na janela); o `XMP-mapir:AlignmentJSON` indica o `"applied": true` com seu `crop_roi`. O arquivo Raw permanece sem distorção no tamanho nativo, mas carrega a transformação nos metadados. `aligned=False` salva todos os arquivos no tamanho nativo, sem distorção; `aligned=True` é a configuração padrão explicitada. `continuous()`, `capture(smart=True)`, `capture_repeated()` e os parâmetros no nível do projeto `ArrayHandle.capture()` / `capture_smart()` adotam o mesmo padrão. Independentemente de a distorção ser aplicada ou não, cada arquivo membro registra a transformação `Alignment*` de sua câmera assim que a matriz possui um perfil de alinhamento — exceto uma câmera cujo perfil não esteja alinhado (omitida ou forçada à identidade devido a uma resolução malsucedida) e a câmera de referência quando nenhuma outra câmera estiver alinhada: essas não possuem tags `Alignment*`. Sem perfil na matriz, a captura é salva desalinhada, e o resultado indica isso: `result.alignment` é `{"requested": True, "applied": False, "warnings": [...]}`, um aviso por motivo pelo qual a distorção não pôde ser aplicada (ausência de perfil, membros com resoluções diferentes, uma matriz não, um quadro no qual o perfil não pode ser mapeado). `applied` é `True` somente quando todas as exportações que solicitaram a correção de distorção a obtiveram. O alinhamento automático da matriz cria o perfil assim que a exposição se estabiliza; o `ArrayHandle.calibrate_alignment()` / o `align-calibrate` da CLI criam um sob demanda. `result.oriented` indica se o `orient=True` gerou os produtos.
- `render_index=False` — ignora a sobreposição do índice de vegetação por câmera; por padrão, ela é renderizada onde estiver configurada.
- `daq` / `daq_tolerance_ms` — a leitura do DAQ é salva como um arquivo sidecar `.daq` por padrão, mesmo quando nenhum nível selecionado precisa dela; `daq=False` a ignora nesse caso, e o `daq_tolerance_ms` define a tolerância de correspondência (padrão 500). O `force_daq=True` é a grafia antiga do padrão. Consulte [A leitura do DAQ que uma captura salva](#o-daq-lendo-uma-captura-salva--arquivos-daq-e-correspondências-daq).
- `orient=True` — transforma os **produtos**de cada membro (`display`, `radiance`, `reflectance`, o índice por câmera e o composto) pela orientação da câmera, após a distorção de alinhamento: pela orientação da**câmera principal**quando foram distorcidos para seu quadro (o padrão `aligned` — os membros permanecem co-registrados), caso contrário, pela orientação de suas próprias câmeras.**Padrão: cada arquivo de membro em formato nativo**, como sempre ocorre com capturas em matriz; `raw` e `debayered` permanecem no formato nativo de qualquer maneira. Cada arquivo registra a orientação do quadro em que seus pixels estão (`XMP-Camera:ChlorosRotation` / `ChlorosFlip*` / `ChlorosOrientationApplied` — um arquivo raw, da própria câmera), e o XMP `Alignment*` mantém a transformação do quadro nativo, aplicada antes da orientação; um arquivo raw que contém essa transformação também registra `ChlorosAligned*` (a do quadro mestre), a orientação de um produto distorcido para o quadro mestre. O processamento gira cada produto de acordo com a orientação que pertence aos seus pixels: a alinhada quando distorce o arquivo raw (alinhamento de matriz ativado), a da própria câmera quando não o faz. `capture_repeated()` e `capture_fastest()` a transmitem; os parâmetros no nível do projeto `ArrayHandle.capture()` / `capture_smart()` também a aceitam.

**Compressão TIFF (opção exclusiva para HTTP):** `ArraySession.capture()` não envia nenhuma chave `compression`, portanto, aplica-se o padrão do backend — o `POST /api/camera/array/capture` lê um parâmetro de corpo do `compression`, o `"none"`, por padrão: sem compressão, os pixels do sensor exatamente como são lidos (~6,3 MB por quadro em resolução total). `"deflate"` (zlib L1 sem perdas + preditor horizontal, ~4,1 MB/quadro) grava arquivos menores mais lentamente — ambos são sem perdas e são lidos de forma idêntica na importação. O SDK não expõe nenhum kwarg para isso; a saída alternativa é `chloros-cli lattice array-capture --compression deflate` ou HTTP bruto. O DEFLATE ocupa o GIL do Python, portanto, as gravações compactadas não são paralelizadas entre as threads de gravação por câmera — a captura sustentada em resolução total na taxa do sensor requer o padrão não compactado. Detalhes: [Referência da CLI → array-capture](cli-reference.md).**Grupos consecutivos (protocolo HTTP):** o endpoint retorna `group_seq`; um loop que o reenvia como `after_seq` na próxima solicitação obtém o próximo grupo sincronizado — incluindo aquele publicado enquanto a captura anterior estava sendo salva — em vez de aguardar um grupo publicado após a solicitação, e `skipped_groups` relata qualquer grupo que o loop tenha perdido. `capture_repeated`, o `--continuous` da CLI e o modo Contínuo da GUI fazem exatamente isso; as passagens de intervalo permanecem atualizadas a cada momento programado.**Substituições de exportação por membro (somente HTTP):**o mesmo endpoint também aceita `exclude_serials` (lista — excluir membros do conjunto salvo; a matriz ainda é acionada como um grupo sincronizado e os membros excluídos são retornados em `excluded`), `serial_levels` (substituições no nível de câmera `{serial: [level tokens]}`) e `serial_index` (substituições de sobreposição de índice por câmera `{serial: bool}`). Esses são parâmetros de corpo com paridade de GUI e**ainda não são argumentos-chave do SDK**; membros ausentes dos mapas recorrem aos valores globais da matriz `levels` / `render_index`.

##### Inspecionando câmeras ignoradas — `CaptureResult.skipped`

`ArraySession.capture()` retorna um `CaptureResult`, que é uma subclasse de `list`: itere por ele, indexe-o, `len()` — todos os padrões existentes continuam funcionando. O novo código pode inspecionar o atributo `.skipped` para verificar quais câmeras foram excluídas e por quê. O caso mais comum são as câmeras RGB em uma matriz de filtros mistos quando você solicita `processing="radiance"` ou `"reflectance"` — a radiação por pixel Bayer não faz sentido para um sensor de banda larga, então o backend ignora essas câmeras em vez de produzir resultados sem sentido.

```python
with chloros_sdk.connect_array(serials) as arr:
    result = arr.capture("output/", processing="reflectance")

    # Back-compat: iterate as a plain list
    for frame in result:
        print(frame["filepath"], frame["serial"])

    # New: see why N-1 cams were saved
    for skip in result.skipped:
        print(f"skipped SN:{skip['serial']} reason={skip['reason']}")
        # e.g. {'serial': '214701292', 'level': 'reflectance',
        #       'reason': 'reflectance-not-applicable-to-rgb-cam',
        #       'filter': 'RGB'}
```

Os tokens de motivo seguem o padrão `<level>-not-applicable-to-rgb-cam` (uma entrada por nível ignorado, cada uma contendo `level`). Uma câmera cujo pacote de calibração não esteja carregado ignora `radiance` / `reflectance` com `<level>-needs-calibration (connect the camera so its cal pack loads)`, e, na ausência de uma leitura de refletância utilizável do DAQ, pula com `reflectance-skipped-no-dls`. Os outros pulos específicos de refletância são `reflectance-skipped-no-fresh-dls` (nenhuma leitura recente de radiação descendente disponível), `reflectance-skipped-bound-daq-unavailable (…)` (não foi possível acessar o DAQ vinculado) e `dls-uncalibrated-band-<nm>` — a banda fica em grande parte fora da faixa radiometricamente calibrada do sensor de luz do DAQ (~374–974 nm), portanto, a divisão absoluta de refletância baseada no DAQ é recusada e o quadro reverte abertamente para a resposta do sensor. Entre os SKUs em estoque, apenas o F988 aciona isso; o caminho suportado por essa câmera é o fluxo de trabalho com painel de refletância.

##### Arquivos que não puderam ser gravados — `CaptureResult.failed`

Uma captura que conseguiu gravar apenas alguns de seus arquivos ainda retorna: a lista contém todos os arquivos que **foram** gravados, e o `.failed` lista o restante, um dicionário por arquivo com o `serial` (`None` para o arquivo composto combinado e o sidecar de uma matriz, `.daq`), `level` (`raw`, `radiance`, …, `index`, `composite`, `daq`, ou `match` — a proveniência do `.daq` do `.match.json`, quando o `.daq` foi gravado, mas seu sidecar não) e `error`. Em uma unidade cheia, a entrada também contém o `disk_full: True`; qualquer captura posterior nessa unidade falharia da mesma forma, e é por isso que o `capture_repeated()` para por aí. Nenhuma exceção é gerada para uma captura parcial, e um arquivo em `.failed` não foi deixado no disco parcialmente gravado. `.failed` fica vazio em uma captura limpa.

```python
result = arr.capture("output/", processing="all")
for f in result.failed:
    print(f"SN:{f['serial']} {f['level']} not written: {f['error']}")
if any(f.get("disk_full") for f in result.failed):
    raise SystemExit("drive full -- free space and capture again")
```

##### O DAQ que lê uma captura salva — `.daq_files` e `.daq_matches`

Sempre que um DAQ está vinculado à câmera ou ao conjunto de câmeras, ou em pool, cada captura salva a leitura descendente que utilizou: um `.daq` na subpasta `daq/` da pasta de captura, com seu `.match.json` e seu `.cameras.json` (de quem é a leitura). Isso vale também para capturas em formato bruto ou com debayering, e o arquivo é o mesmo independentemente de a captura ter sido feita pela GUI, pela CLI ou pelo SDK. O `daq=False` ignora isso para uma captura cujos níveis não utilizem o DAQ (um nível do tipo `radiance`, `reflectance` ou `display` ainda o grava); o `daq_tolerance_ms` desloca a linha de tolerância para essa captura e nunca a leitura utilizada. Qual DAQ: aquele vinculado à câmera ou ao array (a seleção do sensor de luz na GUI, uma vinculação do projeto ou `chloros-cli daq pool-bind`), caso contrário, o mais recente que o backend tiver. A CLI especifica os controles `--no-daq` e `--daq-tolerance-ms` ([Referência da CLI](cli-reference.md#the-daq-reading-a-capture-saves)).

```python
r = cam.capture("out", levels=["raw"])      # a CameraSession; ArraySession results match
for m in r.daq_matches:                      # one per .daq in r.daq_files
    print(m["daq_file"], m["gap_ms"], m["in_tolerance"])
```

Uma captura salva a leitura do sensor de luz utilizada como um `.daq`, acompanhada de um `<file>.daq.match.json` que indica como a leitura foi vinculada à exposição: `method`, `gap_ms` (da exposição à leitura DAQ mais próxima) e `in_tolerance` (dentro de 500 ms, ou o `daq_tolerance_ms` da captura). O XMP do arquivo de refletância apresenta o mesmo veredicto que `Chloros:DlsMatchMethod`, `Chloros:DlsMatchGapS` e `Chloros:DlsInTolerance`.

Uma captura aguarda a primeira leitura do DAQ após a exposição, para que possa combinar as leituras de ambos os lados dela (`method` `interpolated`). Um DAQ-E envia uma leitura a cada 0,4 s à luz do dia e a cada 1–2 s em ambientes internos; portanto, o `capture()` retorna até cerca de um intervalo e meio depois; a exposição em si não sofre atraso. Não se aguarda um DAQ que tenha parado de enviar dados. Uma leitura do DAQ-E corresponde à média da luz no intervalo desde sua leitura anterior, portanto, ela é posicionada no meio desse intervalo: o `gap_ms` é medido nesse ponto médio, e o `.match.json`'`bracket_stamps_ns` fornece os próprios carimbos de data/hora das leituras. O `ArraySession.continuous()` aguarda sem desacelerar a captura. As passagens de volta do `capture_repeated()`não esperam, portanto mantêm sua taxa e são associadas à leitura mais próxima antes da exposição.

Uma leitura nunca expira, portanto nenhum quadro é perdido por causa de uma leitura atrasada — em vez disso, ela é identificada como:

- **O fluxo do DAQ está paralisado** — sua leitura mais recente é usada, e o `gap_ms` indica há quanto tempo ela foi registrada (`in_tolerance: false` há mais de 500 ms).
- **Nenhum DAQ para leitura**— desconectado (`disconnect()`, `chloros-cli daq pool-disconnect`) ou o DAQ vinculado não pode ser acessado — a última leitura da câmera é reutilizada e descrita em relação a**esta** exposição: `method`, `retained` e `gap_ms` desta exposição, `in_tolerance` é recalculado, e `retained_from_epoch_ns` indica a exposição à qual foi inicialmente associado. Uma câmera que nunca teve uma leitura continua sem refletância, e o resultado explica o motivo (`reflectance-skipped-no-fresh-dls` / `reflectance-skipped-bound-daq-unavailable (…)`, ou a recusa da captura).

Os arquivos que uma captura salva dentro da pasta do projeto aberto no Chloros passam a fazer parte da lista de arquivos desse projeto, assim como acontece com uma captura feita no aplicativo (arquivos salvos em outros locais não são afetados): `CameraSession.capture()`, `ArraySession.capture()` (`smart=True` também) e `capture_cameras_transient()`. O resultado indica isso em `.project_add` (`capture_cameras_transient()`: o `project_add` do dicionário) -- `held`, `files` e o `message` do backend; `held` é `True` enquanto uma execução de processamento estiver em andamento, e os arquivos são reunidos assim que ela terminar. `None` quando nenhum arquivo foi adicionado ao projeto.

`CameraSession.capture()` e `ArraySession.capture()` retornam-nos como o resultado `.daq_files` e `.daq_matches`, já que as respostas de captura HTTP (`/api/camera/<serial>/cli-capture`, `/api/camera/capture`, `/api/camera/capture-all-synced`, `/api/camera/array/capture`) contêm `daq_files` e `daq_matches`, um por `.daq`: `daq_file`, `cameras`, `method`, `gap_ms`, `in_tolerance`, `tolerance_ms` e `stream_fault` — falha no próprio fluxo do DAQ, consultada apenas para leituras fora da tolerância. Antes de confiar na refletância de uma captura, verifique o `in_tolerance` em seu `.match.json` ou XMP, ou os níveis [`status()`](#daqsensorsession-methods).

Níveis de `processing`:

| Nível | Saída |
| --- | --- |
| `"raw"` | Bayer de canal único (câmeras monocromáticas: a banda única) diretamente do sensor. |
| `"debayered"` *(padrão do SDK)* | Cor de 3 canais: o desvio de escuro (DSNU) do pacote de calibração retirado do raw, seguido do demosaico sensível a bordas do OpenCV — sem LUT de linearidade, campo plano ou separação de cores (câmeras monocromáticas: escala de cinza de 1 canal). O arquivo TIFF salvo está na ordem de canais **RGB**(RGB fotométrico, de modo que qualquer visualizador de imagens exiba as cores reais); os arrays na memória que `grab()` / `frame_stream()` retornam estão na ordem**BGR** do OpenCV — inverta os canais (`frame[..., ::-1]`) para comparar com um arquivo salvo. `chloros-cli process` grava o mesmo resultado que `Debayered_Images`. |
| `"radiance"` | float32 W/m²/sr/nm por meio da cadeia radiométrica completa. Apenas multiespectral — câmeras RGBs são ignoradas. |
| `"reflectance"` | uint16 0..32768 (compatível com Pix4D); requer um emparelhamento DAQ ativo para referência absoluta. Apenas multiespectral. |
| `"display"` | Cadeia completa correspondente à visualização da GUI (CCM + WB + gama de acordo com o perfil da câmera). |
| `"all"` | **Um arquivo por nível aplicável** para cada câmera (correspondendo à opção “Capturar tudo” da GUI / padrão da CLI). O `CaptureResult` retornado contém, então, um dicionário de quadro por `(cam, level)`, com o nível em cada dicionário; os níveis inaplicáveis aparecem em `.skipped`. A leitura do DAQ utilizada na captura é salva como um arquivo sidecar `.daq`. |

> **Observação — o padrão difere da CLI.** O `ArraySession.capture()` tem como padrão o `processing="debayered"`; o comando `chloros-cli lattice array-capture` tem como padrão o `processing="all"`. Passe o parâmetro `processing="all"` explicitamente a partir do SDK para espelhar o salvamento em vários níveis da CLI/GUI.

##### Nomes dos arquivos de captura de matriz

Cada nível é salvo em sua própria subpasta de `output_dir` (`raw/`, `debayered/`, `radiance/`, `reflectance/`, `preview/` para `display`, `index/`, `composite/`). Cada arquivo nomeia sua câmera como `SN<serial>-<filter>`: o número de série, seguido da parte do filtro do modelo da câmera — `SN214000533-FOCN` para uma `M3C-L87-FOCN`, `SN261902142-F450` para uma `M3M-L87-F450` — junto com `-`, portanto, um nome possui os mesmos campos separados por `_` de antes. Uma câmera cujo modelo não pode ser lido é apenas `SN<serial>`, e as capturas de versões anteriores não possuem filtro no nome. O nome do arquivo depende de quantos níveis foram solicitados ao membro:

| Nível solicitado | Nome do arquivo | Exemplo (`reflectance`, padrão `format="tiff"`) |
| --- | --- | --- |
| **Um nível** (`processing="reflectance"`, ou um item único `levels`) | `sync_<timestamp>_SN<serial>-<filter>.<format>` — sem sufixo de nível, como nas versões anteriores, para que os scripts existentes continuem funcionando | `reflectance/sync_<ts>_SN214000533-FOCN.tiff` |
| **Vários níveis** (`processing="all"` ou `levels` com mais de um item) | `sync_<timestamp>_SN<serial>-<filter>_<level>.<format>` | `reflectance/sync_<ts>_SN214000533-FOCN_reflectance.tiff` |

A [máscara de limitação](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) vem após o nome do produto, `clamp_masks/<product folder>/<product name>_clampmask.tif`: uma captura de refletância decaptura de refletância de nível único grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_clampmask.tif`, enquanto `processing="all"` grava `clamp_masks/reflectance/sync_<ts>_SN<serial>-<filter>_reflectance_clampmask.tif`. A regra do sufixo é determinada para cada membro a partir dos níveis solicitados; portanto, um nível ignorado (por exemplo, refletância em uma câmera RGB) não a altera. Leia os caminhos a partir dos quadros retornados, em vez de construí-los. Uma captura com uma única câmera (`CameraSession.capture()`) sempre adiciona o nível: `capture_<ts>_SN<serial>-<filter>_<level>.tiff`.

##### PNG/JPEG: aceitam apenas produtos de exibição e de índice

Um arquivo PNG ou JPEG armazena 8 bits por canal. A solicitação de um arquivo com um nível que ele não pode armazenar é recusada antes que qualquer coisa seja capturada, com uma mensagem indicando o nível e o formato TIFF como o que o suporta:

- **`radiance`**(float32) e**`reflectance`**(uint16, `32768` = 1,0) — e, portanto, `processing="all"`, que os inclui. O SDK verifica isso nos argumentos: `capture()` gera**`ValueError`** sem uma solicitação (`CameraSession`, `ArraySession`, `CameraHandle`, `ArrayHandle`, `capture_smart`).
- **`raw` / `debayered` provenientes de uma câmera transmitindo mais de 8 bits** (M3C de 12 bits, Mono12 M3M) — o backend rejeita esses dados, conhecendo a profundidade de bits de cada câmera: 400 HTTP com `error_code` e `FORMAT_CANNOT_HOLD_LEVEL`, gerado como `ChlorosConnectError` (`CameraSession` / `ArraySession`) ou `RuntimeError` (`CameraHandle` / `ArrayHandle`). A captura de nível único `debayered` de uma câmera captura um quadro de 8 bits para PNG/JPEG e não é rejeitada.

Salve os dados radiométricos como `format="tiff"` (`ext=".tiff"`); use PNG/JPEG para `display` e renderizações indexadas.

#### Como funciona a exposição automática

- **Exposição primeiro, ganho por último.** Com o ganho em “Auto”, ele permanece em 0 dB até que a exposição atinja o valor máximo permitido.
- **Uma exposição manual mantém o ganho em Automático** até que você altere o ganho ou o configure para manual. De volta ao modo Automático, o limite do ganho é o mesmo de antes.
- **Pequenos realces não escurecem a cena.** Um realce especular de até 1 % do quadro é ignorado e, nos painéis de um alvo de calibração conhecido, até 0,5 % de um painel.
- **O limite máximo de exposição do DAQ se aplica apenas enquanto um alvo de calibração estiver em vista.** Isso evita que um painel branco sofra clipping; quando nenhum painel está visível, a câmera expõe de acordo com a cena.
- **A luz constante permanece estável, e mudanças na luz se estabilizam em segundos.** Com luz constante, a exposição e seu limite permanecem inalterados, e após uma grande mudança na luz, a imagem se estabiliza em poucos segundos. Um clipping não limita mais a exposição de forma definitiva, e uma alteração na região de interesse (ROI) restabelece a exposição.
- **O alvo de brilho permanece ao alcance.** Ele não aumenta enquanto a câmera estiver no limite de exposição e seu ganho não puder subir; assim, uma luz que retorne não causa reflexos no quadro.
- **Uma câmera cuja exposição automática deixa de responder é mantida no nível correspondente à iluminação em que se encontra.** Sob sol forte, uma câmera poderia cair para sua exposição mais curta e permanecer assim com um quadro escuro, embora todas as suas configurações apresentem valores normais. O Chloros monitora o quadro, mantém a câmera no nível que ela mantinha para aquela iluminação e a libera quando a luz muda. Um sensor de luz escuro ou coberto não interfere nisso.

#### Exposição nos membros da matriz

Uma câmera LATTICE limita o `ExposureTime` a um período de sua taxa de quadros. Enquanto uma matriz transmite, o Chloros executa cada membro com o `AcquisitionFrameRateEnable` ativado, a uma taxa ligeiramente acima da taxa de disparo, de modo que o limite máximo de exposição de um membro fique um pouco abaixo do período de disparo, **em qualquer ROI**:

- O período de quadro de um membro é o período de disparo menos uma margem de 15% do período (no mínimo 3 ms, no máximo metade do período), e seu limite máximo de exposição é esse período de quadro menos cerca de 0,1 ms. Exemplo: um acionamento de 10 fps (100 ms) resulta em uma taxa de membro de cerca de 11,8 fps e um limite máximo de exposição de cerca de 85 ms.
- **Um `ExposureTime` manual mais longo do que esse limite máximo desacelera o gatilho para se adequar a ele.** O Chloros primeiro abre o limite máximo desse membro para o valor e permite que a taxa mais lenta execute alguns períodos de gatilho (a gravação pode levar alguns segundos a mais), depois grava o valor, e o disparador não dispara mais rápido do que a exposição permite. Em uma matriz de taxa automática, a taxa em que funcionava anteriormente é restabelecida assim que todos os membros cuja exposição manual o desacelerou voltarem à exposição automática. Um valor que a própria câmera não pode aceitar ainda é recusado, com o intervalo que ela aceita agora indicado na mensagem de erro, e a exposição automática do membro é restabelecida se ela tiver sido desativada apenas para essa gravação. Antes da versão 1.3.0, uma pequena ROI limitava cada membro ao próprio período de funcionamento livre do sensor, independentemente da taxa de disparo: 4,9 ms em 768 × 304, de modo que nenhum membro pudesse ser clareado.
- O Chloros nunca reduz uma exposição manual para se adequar a um disparo mais rápido. Uma taxa automática permanece dentro do que as exposições manuais permitem; uma taxa fixada mais rápida do que uma exposição permite deixa aquele membro com um limite máximo longo o suficiente para ela, e ele aceita disparos não mais rápidos do que sua exposição permite. Diminua a exposição ou a taxa fixada quando todos os membros precisarem realizar todos os disparos.
- **Leitura de retorno:** a leitura dos nós de um membro mostra `AcquisitionFrameRateEnable` True e `AcquisitionFrameRate` na taxa do membro; a ativação é desativada quando a matriz para.
- **A matriz possui esses dois nós. Não grave `AcquisitionFrameRateEnable` ou `AcquisitionFrameRate` em um membro da matriz** — defina acom `target_fps` / `set_target_fps()`.

Grave a exposição de um membro com `arr.set_member_settings(serial, exposure_time=20000)` (as palavras-chave de `CameraSession.set_settings()`; `chloros-cli lattice exposure --serial` faz o mesmo). O membro permanece na matriz. Não abra um membro da matriz com `connect_camera()`: o `connect` recusa uma câmera que já esteja em uma matriz, e o ``CameraSession`` que ele retornaria remove a câmera de sua matriz ao se desconectar. O ``ArraySession.status()["member_ae"]`` relata a exposição de cada membroexposição e ganho de cada membro a partir de seu quadro mais recente.

### Um registro de projeto, para todos os modos

Uma pasta de projeto contém as configurações de todos os dispositivos: `cameras.json` — de cada câmera (`cameras[].settings`, para uma única câmera e para cada membro da matriz) e as configurações válidas para toda a matriz (`arrays[].capture`: formato, ROI, binning, taxa de disparo, orçamento de banda) — e `sensors.json`, as de cada sensor de luz (tempo de integração, média de quadros, limite). O aplicativo, a CLI, o SDK e o servidor MCP leem e gravam os mesmos arquivos, da mesma maneira:

- **Um dispositivo conectado a um projeto é salvo nesse projeto e registra todas as alterações nele.** “Vinculado a um projeto” significa com um projeto aberto no backend (do aplicativo) ou — por meio de um `ChlorosProject` (`open_project()`), que indica sua própria pasta ao backend (`project_path`). Uma câmera, matriz ou sensor de luz que o projeto ainda não possua é adicionado aos seus arquivos quando se conecta (`connect_camera()`, `connect_array()`, a conexão do sensor de luz, a CLI, o aplicativo, `chloros-mcp`), com suas configurações. Uma alteração em qualquer modo (`set_settings()`, `ArraySession.set_member_settings()`, `set_target_fps()`, as configurações da matriz, `set_cap()`, o CLI, os controles do aplicativo) é salva nos arquivos desse projeto logo em seguida. Uma exposição, ganho ou meta de AE definida em uma câmera é salva como mantida; portanto, ao reabri-la, ela é mantida novamente.
- **Ao abrir um projeto, seus dispositivos são configurados a partir dele**, em todos os modos: uma câmera, matriz ou sensor de luz conectada sem configurações próprias recebe as(`lattice cam-connect` / `array-connect` com um projeto aberto, `chloros-cli project connect`, `open_project().connect_all()`, ao reabrir o aplicativo). Um array conectado com um novo ID atualiza o array que o projeto salvou com as mesmas câmeras; um array com outras câmeras é adicionado ao lado dele.
- **O próprio salvamento do aplicativo nunca sobrescreve as configurações ativas de um dispositivo** com sua cópia mais antiga e mantém as substituições por membro salvas por um chamador de CLI ou SDK.
- **O formato de pixel definido é aquele que o projeto mantém.** Um formato definido nas Configurações da Matriz, por meio de `--pixel-format` ou `connect_array(pixel_format=)`, incluindo o de 8 bits, é salvo e reconectado exatamente. Se nenhum formato for definido, cada câmera se conecta com sua configuração padrão de 12 bits (`BayerRG12`, `Mono12` em um sensor monocromático). O nome do formato indica sua profundidade de bits; a família é a do sensor; portanto, uma câmera colorida nunca é configurada para um formato monocromático, nem uma câmera monocromática para um formato Bayer, e a linha `[PixelFormat <serial>] ... written` indica isso quando um formato foi mapeado. As exportações de uma captura de 8 bits são de 8 bits, incluindo a refletância: redimensionadas e marcadas com seu `PixelScale`.
- Um dispositivo conectado sem nenhum projeto não registra nada.
- **Uma gravação segue um projeto criado depois dela; ao ser aberta, ela mantém o projeto.**Quando um projeto é criado — `create_project()`, o `chloros-mcp` do `create_project` ou o**Novo Projeto**do aplicativo — ou duplicado (a opção**Duplicar**do aplicativo, que abre a cópia) enquanto uma gravação ainda está gravando no projeto que estava aberto anteriormente, a gravação continua, e o que é gravado vai para o novo projeto, no mesmo local dentro dele (abaixo). Ao abrir um projeto existente — `ChlorosProject.process()` (que abre seu projeto no backend), `chloros-cli process`, a opção**Abrir Projeto** do aplicativo — deixa uma gravação ainda em andamento com o projeto no qual ela está sendo gravada: ela é concluída ali, e seus arquivos são adicionados à lista de arquivos desse projeto (o aplicativo interrompe todas as gravações antes de abrir um projeto).
  - `ArraySession.record()` e `ArraySession.burst()`: o vídeo, ou a pasta de sequência com todos os quadros, é movido inteiro para lá quando a gravação é interrompida. O resultado da interrupção informa o novo caminho (`out_dir`, `video_path`), e uma compilação da sequência é gerada a partir dele. Se a unidade recusar a transferência, a gravação permanece onde foi gravada, e o resultado indica isso (`relocation_error`). Os quadros brutos de uma sequência interrompida e o arquivo `.daq` são adicionados à lista de arquivos do projeto no qual ela é incorporada; um vídeo não é adicionado à lista de arquivos do projeto.
  - `ArraySession.continuous()`: todas as capturas após a alteração são salvas no novo projeto; as salvas anteriormente permanecem onde estão.
  - `DAQSensorSession.record_start()` (e a gravação do sensor de luz do próprio aplicativo): o `.daq` é movido para a pasta `light_sensor` do novo projeto quando a gravação é interrompida; o `record_stop()` retorna o novo caminho, e o arquivo passa a fazer parte da lista de arquivos desse projeto.

  Uma gravação em uma pasta fora do projeto aberto permanece onde está, e o `start_camera_continuous()` / tarefas com múltiplos fluxos mantêm a pasta que lhes foi atribuída.
- **No aplicativo, as capturas feitas após a criação de um novo projeto vão para ele.** Criar ou duplicar um projeto mantém todas as execuções de captura e gravações em andamento, e cada captura feita a partir desse momento (Capturar Tudo, Iniciar Captura) é salva no novo projeto e passa a fazer parte de sua lista de arquivos. Abrir um projeto existente no aplicativo interrompe todas elas primeiro.
- **Um projeto não é movido enquanto está gravando.**Mover o projeto aberto para outro diretório de trabalho (em**Configurações → Diretório de Trabalho** do aplicativo; nenhuma chamada ao SDK move um projeto, um cliente HTTP que envia dados para `/api/move-project` o faz) é recusada com o código HTTP 409 `PROJECT_RECORDING` enquanto uma gravação em matriz ou em rajada, uma captura contínua em matriz, um trabalho contínuo/multistream da câmera ou uma gravação do sensor de luz estiver gravando nele: `Project '<name>' is recording (<what>), so moving the project now is refused. Stop the recording, then try again.`

Isso é diferente do que ocorre com as câmeras no momento da desconexão (consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings)): um projeto configura suas próprias câmeras ao ser aberto.

### Memória da câmera — `keep_settings` / `clear_settings`

Quando um conjunto é conectado, suas câmeras assumem as configurações do conjunto, e você pode alterá-las posteriormente. Quando o Chloros libera as câmeras — `ArraySession.disconnect()` / `force_disconnect()`, no final de uma função transitória, a interrupção do `start_camera_continuous()` —, ele restaura cada uma delas ao estado em que se encontrava antes da conexão: sua fiação de disparo e linha de sincronização, PTP e configurações da câmera — `ExposureAuto` / `GainAuto` (e um modo manual `ExposureTime` / `Gain`), `TargetBrightness` e os limites e amortecimento do AE, balanço de branco, nível de preto, `PixelFormat`, agrupamento de pixels e a ROI, o pipeline de cores (`GammaEnable` / `Gamma`, `LUTEnable`, `ColorTransformationEnable`), `ReverseX` / `ReverseY`, `UserSetSelector`, o PTP (`PtpSlaveOnly` / `PtpDomainNumber`), o tamanho do pacote (`GevSCPSPacketSize` / `DeviceStreamChannelPacketSize`), o ritmo (`DeviceLinkThroughputLimitMode` / `DeviceLinkThroughputLimit`, `GevSCPD`, `GevSCFTD`), os dados do bloco (`ChunkModeActive`, cada `ChunkEnable` sob seu `ChunkSelector`), a remoção de ruídos da entrada de disparo — uma matriz fornece a linha de sincronização de um escravo (`LineFilterSelector` / `LineFilterWidth`) e a taxa de quadros de aquisição (`AcquisitionFrameRate` apenas quando sua ativação estava ligada: com ela desligada, a taxa é a máxima permitida pelo ritmo, não uma configuração). Apenas o que difere é registrado: a câmera parou primeiro, o ritmo e a taxa de quadros antes de uma exposição que requer seu período de quadro.

Para usar as câmeras fora de um projeto, dois argumentos gravam na **própria memória** da câmera — o que ela carrega ao ser ligada, em qualquer software (em `disconnect()` / `force_disconnect()`, todas as funções transitórias, `start_camera_continuous()` e `ChlorosProject.disconnect_all()`; MCP: os mesmos argumentos de `disconnect_device` e das ferramentas transitórias; CLI: `--keep-settings` / `--clear-settings`):

- **`keep_settings=True`** salva as configurações de cada câmera exatamente como estavam no momento em que a sessão foi encerrada (UserSet2, definido como conjunto de inicialização). As câmeras continuam operando livremente (`TriggerMode` `Off`), com a fiação da linha de sincronização tal como foi encontrada. Uma câmera sem UserSet2 salva em UserSet1; este computador lembra disso, portanto, a autocorreção da matriz dChloros — que redefine um UserSet1 que se desviou da linha de base dChloros — não a altera.
- **`clear_settings=True`** redefine a memória de cada câmera para as configurações de fábrica: a linha de base do Chloros, aquela que a autocorreção da matriz incorpora — padrões de fábrica com `TriggerMode`, `Off`, `TriggerSource`, `Line2`, exposição automática, o equilíbrio de branco calibrado de uma câmera RGB mantido — no UserSet1, que compõe o conjunto de inicialização.
- Não ambos (`ValueError`; o backend responde com 400). A resposta contém `camera_memory`, por câmera `{"saved_to": "UserSet2" | "UserSet1" | None, "errors": [...]}` ou `{"cleared": bool, "errors": [...]}` — o `force_disconnect()` o retorna; para uma tarefa, ele vem com o `stop()`.

O aplicativo não oferece esses valores.

**Cada saída restaura as câmeras.**A restauração conforme encontrado é executada em cada caminho de saída: `CameraSession.disconnect()` (a câmera**X**do aplicativo), o**X** do aplicativo em uma matriz ou em qualquer membro da matriz (a matriz inteira é liberada), fechando o aplicativo (o backend tem até 20 s para restaurar todas as câmeras, matrizes e gravações contínuas em andamento; uma função transitória ou uma chamada única ainda em execução para no próximo ponto seguro, restaura suas câmeras primeiro, por até 8 s, e responde com HTTP 503 `SHUTTING_DOWN`; um `connect_array()` ainda em execução para antes de sua próxima câmera e restaura as câmeras que abriu, por até 10 s, e gera `ChlorosConnectError` (HTTP 503 `SHUTTING_DOWN`); e uma que chega durante o desligamento não abre nada: HTTP 503 `SHUTTING_DOWN`), e um comando `chloros-cli` que é encerrado ou cujo terminal é fechado (sua tarefa é interrompida e suas câmeras são fechadas primeiro). Um backend que trava ou é encerrado à força não pode fechar nada; reiniciar o sistema retorna cada câmera à sua memória.**Um projeto configura suas próprias câmeras.** Quando um conjunto se conecta por meio de um projeto — o projeto aberto do aplicativo ou um `ChlorosProject`, que se identifica para o backend (`project_path`) — cada membro recebe as configurações que o projeto salvou para ele: uma exposição, ganho ou meta de AE mantida nele (considerada como sua, de modo que a exposição automática não a altera) e seu modo de balanço de branco (correção de Chloros; o próprio balanço de branco da câmera permanece desativado), independentemente do que a memória da câmera contenha.

### Modos de Captura e Gravadores

A superfície da matriz reflete o painel de captura da interface gráfica: modos de obturador Único / Contínuo / Intervalo / Mais rápido, além de dois gravadores (vídeo composto ao vivo e rajada em raw → reprocessamento offline).

```python
import time, chloros_sdk

with chloros_sdk.connect_array(serials) as arr:
    # Single (default) — one synced group
    arr.capture("out/", processing="reflectance")

    # Fastest — raw + .daq + combined index now, calibrate later
    arr.capture_fastest("flightline/")

    # Interval — one reflectance pass every 2 s, 5 passes (bounded so it ends)
    arr.capture_repeated("timelapse/", count=5, interval_s=2.0,
                         processing="reflectance",
                         on_capture=lambda i, r: print(f"pass {i}: {len(r)} frames"))

    # Continuous — every synced group for 60 s, saved by the backend itself
    with arr.continuous("run1/", processing="raw", duration_s=60) as run:
        run.wait()
    print(run.result["captures"], run.result["captures_per_s"],
          run.result["stop_reason"])

    # Combined-index video/GIF recorder: the GUI's combined view, or with no
    # GUI open the array's NDVI (pass expr="..." for another index)
    with arr.record("monitoring/", fps=10, gif=True) as rec:
        time.sleep(30)
    if rec.result and rec.result.get("error"):   # leaving the block does not raise
        print("recording failed:", rec.result["error"])
    else:
        print(rec.result["video_path"])

    # Raw-Bayer burst → offline reprocess into calibrated video(s)
    with arr.burst("capture/", duration_s=5) as b:
        pass
    out = arr.build_video(b.result["out_dir"], products=[
        {"kind": "per_cam", "level": "reflectance"},
        {"kind": "combined", "level": "index"}])
    print(out["outputs"])
```

- **`capture_repeated`**é o loop Contínuo/Intervalo do SDK. Como não há `Ctrl+C` para interrompê-lo a partir de um script, você**deve** passar `count` e/ou `duration_s` (ele para quando qualquer um deles for atingido). `interval_s` é medido a partir do início de cada passagem (em consonância com a GUI). As passagens contínuas (`interval_s=0`) encadeiam `group_seq` → `after_seq`, portanto são grupos sincronizados consecutivos na taxa da matriz. Os kwargs restantes são repassados diretamente para `capture()`.
- **`continuous`**é a captura contínua do próprio backend — o modo Contínuo da interface gráfica.**Use-o em vez de `capture_repeated()` quando quiser que todos os gatilhos sejam gravados em disco com uma alta taxa de quadros:** O `capture_repeated()` faz uma solicitação HTTP por grupo a partir do seu script, enquanto o `continuous()` mantém duas capturas em andamento dentro do backend, sem ida e volta entre os grupos. Medido no ROI 512 em um conjunto de 9 câmeras: 73–77 capturas/s, contra um conjunto que publica cerca de 78. Mantenha o `capture_repeated()` para captura por intervalo (`interval_s`) e para as passagens que você inspeciona uma a uma. No Windows, exclua o backend do Microsoft Defender para taxas elevadas (consulte [Etapa 1](#step-1--install-the-chloros-platform-package)).
- **`record`** é de *nível de monitoramento*: ele grava o composto de índice combinado renderizado em tempo real. Com a visualização combinada da GUI aberta, ele grava essa visualização conforme exibida. Sem nenhuma aberta, o backend renderiza a própria visualização: `expr` se você passar uma, caso contrário, o NDVI da matriz (as bandas vermelha e NIR mais próximas de 660 nm e 850 nm, conforme a visualização ao vivo as seleciona), assim que a matriz estiver alinhada. Nomeie as bandas em `expr` conforme a matriz as nomeia: `<band>_<filter>` para uma câmera M3C (`NIR_850_RGN`, `Red_660_RGN`, `Orange_615_OCN`), `Mono_<filter>` para uma M3M (`Mono_850`); um nome curto de cor (`NIR`, `Red`) funciona apenas quando uma banda da matriz o possui. Uma visualização da GUI aberta durante a gravação é gravada enquanto permanecer aberta. Uma gravação que não grava nenhum quadro falha com `error` e `"no-frames: …"` (a matriz ainda pode estar se alinhando ou ajustando sua exposição), e o `stop()` é acionado (consulte [`RecorderHandle`](#recorderhandle)). Um gravador composto por matriz (é acionado se já houver um em execução).
- **`burst` → `build_video`** é de *nível de análise*: `burst` grava quadros brutos + um manifesto por quadro + um `.daq` por leitura DLS distinta sob `<output>/bursts/<base>/` na taxa máxima do loop de captura (sem cadeia, sem exiftool, sem visualização ao vivo). `build_video` sincroniza cada quadro com o `.daq` mais próximo e reexecuta a cadeia de radianância/refletância/índice do pipeline de importação. O `products` é uma lista de `{"kind": "per_cam"|"combined", "level": "radiance"|"reflectance"|"index"}` (padrão: o índice combinado). O `burst().stop()` também inicia automaticamente a construção de um índice combinado, retornada como `build_job` no resultado de parada.

##### O resultado `build_video`

| Chave | Significado |
| --- | --- |
| `success` | `True` somente quando todos os produtos solicitados foram gerados conforme solicitado: pelo menos uma saída, nada em `skipped`, e nada em `degraded`. |
| `outputs` | Caminhos dos vídeos (e GIFs) gravados. |
| `errors` | Problemas, em texto. |
| `skipped` | Um `{"serial", "level", "reason"}` por produto solicitado que **não** produziu vídeo — seja por causa de uma câmera (ausência de pacote de calibração para essa câmera, ausência de leitura DLS para refletância, falha na cadeia de processamento) ou do índice combinado, que contém `"kind": "combined"`, `serial` e `None` (nenhum par de bandas vermelha/NIR entre as câmeras, nenhum grupo com radiancia para ambas as bandas). Qualquer entrada gera `success` e `False`; os demais vídeos ainda são gravados e listados. |
| `substitutions` | Câmeras renderizadas em um nível diferente do solicitado, e o motivo — uma câmera RGB que solicitou radianância/refletância recebe seu vídeo de exibição, com balanço de branco ajustado quando o pacote de calibração da câmera está disponível. Não é uma falha. |
| `notes` | Texto informativo, não problemas — por exemplo, uma câmera sem expressão de índice específica não possui vídeo de índice, ou o vídeo de exibição de uma câmera RGB não tem balanço de branco (ausência de pacote de calibração ou um sem balanço de branco de fábrica), indicando o nome da câmera. Não é uma falha. |
| `degraded` | `True` quando um produto foi gerado, mas não é o solicitado: o índice combinado offline é a diferença normalizada integrada entre vermelho e NIR, portanto, uma expressão de índice configurada não é avaliada. Gera `success` `False`. |
| `combined` | Resumo do índice combinado: as bandas utilizadas, o intervalo da LUT, os quadros criados. |
| `clamp_masks` | Caminhos das [máscaras de limitação de desmistura](#unmix-clamp-mask--which-pixels-a-band-was-clamped-on) dosTIFFs calibrados por quadro (`save_tiffs=True`), em `clamp_masks/<level>/` ao lado da pasta `<level>/` dos TIFFs, um por TIFF no qual um pixel foi limitado. Um vídeo não contém nenhum. |
| `clamp_fractions` | O que um vídeo contém em vez de uma máscara: um `{"kind", "serial", "level", "frames"}` por produto de vídeo (`kind` `"combined"` para as câmeras integrantes do índice combinado), `frames` uma lista de `[frame, R, G, B]` — a parcela de pixels daquele quadro que o desmisturador limitou na faixa que seu canal R / G / B ocupa (`Chloros:Band{R,G,B}UnmixClampFraction`). |

#### `RecorderHandle`

Retornado por `ArraySession.record()` e `ArraySession.burst()`. Use-o como um gerenciador de contexto para parar automaticamente ao sair do escopo ou acione-o manualmente.

| Membro | Descrição |
| --- | --- |
| `job_id` | ID da tarefa de backend (str). |
| `kind` | `"composite"` (de `record`) ou `"raw"` (de `burst`). |
| `start_stats` | O dicionário retornado pela chamada `start`. |
| `result` | `None` durante a execução; o dicionário final do resultado da parada após a interrupção. |
| `stats(timeout=10.0)` | Estatísticas do trabalho em tempo real (quadros gravados, fps alcançados, tempo decorrido). |
| `stop(timeout=60.0)` | Interrompe o gravador; retorna e armazena em cache o resultado final (`.result` permanece como `None` se a própria solicitação de interrupção falhar). Para uma gravação `record()` que falhou — aquela que não gravou nenhum quadro (`error` `"no-frames: …"`) — ele armazena o resultado em cache em `.result` e, em seguida, gera o erro `ChlorosConnectError`. Idempotente (uma segunda chamada retorna o resultado armazenado em cache, sem gerar erro). Sair de um bloco `with` interrompe o gravador sem gerar exceção; portanto, verifique `rec.result.get("error")` após isso. |

```python
rec = arr.burst("capture/")
# ... drive manually ...
print(rec.stats()["frames"])
result = rec.stop()
print(result["out_dir"], result.get("build_job"))
```

#### `ContinuousHandle`

Retornado por `ArraySession.continuous()`. Use-o como um gerenciador de contexto — sair do bloco `with` interrompe a captura se ela ainda estiver em execução — ou acione-o com `status()` / `wait()` / `stop()`. É executada uma captura contínua por matriz por vez.

| Membro | Descrição |
| --- | --- |
| `start_status` | O dicionário de status retornado pela chamada de início. |
| `result` | `None` durante a execução; o dicionário de status final após o término da captura (definido por `stop()`, ou por um `status()` / `wait()` que detectou seu término). |
| `status(timeout=10.0)` | Dicionário de status ativo (campos abaixo). |
| `wait(timeout=None, poll_s=0.5)` | Bloqueia até que a captura termine por si mesma — quando `max_captures` ou `duration_s` for atingido, a matriz for esgotada, a unidade estiver cheia ou ocorrer um erro — e retorna o status final. Com `timeout`, retorne o status mais recente após esse número de segundos, mesmo que ainda esteja em execução: verifique `running`. |
| `stop(timeout=30.0)` | Interrompe a captura e retorna seu status final. Idempotente (uma segunda chamada retorna o resultado armazenado em cache). |

Campos de status:

| Campo | Significado |
| --- | --- |
| `running` | Verdadeiro enquanto a captura estiver em execução. |
| `captures` | Grupos sincronizados salvos. |
| `files` | Arquivos gravados. |
| `failed_files` | Arquivos que não puderam ser gravados (conforme em [`CaptureResult.failed`](#arquivos-que-não-puderam-ser-gravados--resultado-da-captura-falhou)). |
| `failed_captures` | Tentativas de captura que falharam por completo, incluindo aquelas que foram repetidas (um erro HTTP503 enquanto o array não tinha nenhum novo grupo a tempo). |
| `skipped_groups` | Grupos publicados pela matriz que não foram salvos (a captura ficou atrasada). |
| `captures_per_s` | Taxa média de salvamento desde o início. |
| `elapsed_s` / `started_at` | Segundos de execução até o momento; hora de início (segundos da época Unix). |
| `last_group_seq` | O grupo sincronizado mais recente salvo. |
| `last_error` | O erro mais recente, ou `None`. |
| `stop_reason` | Uma vez concluído: `count` (`max_captures` atingido), `duration` (alcançado o `duration_s`), `stopped` (chamado o `stop()`), `array_gone` (o array foi desconectado), `error` (20 falhas consecutivas ou uma solicitação rejeitada pelo backend), `disk_full` (um arquivo para o qual a unidade não tinha espaço). `None` durante a execução. |
| `max_captures` / `duration_s` | Os limites com os quais foi iniciado (`0` = nenhum). |
| `defender_cores` | Número de núcleos da CPU utilizados pelo Microsoft Defender nos últimos 10 segundos; `None` até que um seja medido e sempre `None` em um host sem o Defender. |
| `defender_warned` | Verdadeiro assim que a captura registrar que o Defender está causando lentidão (em 1 núcleo ou mais). Exclua o backend da verificação do Defender — consulte [Etapa 1](#step-1--install-the-chloros-platform-package). |

```python
run = arr.continuous("captures/run1", processing="raw", max_captures=5000)
try:
    while True:
        st = run.wait(timeout=5)              # returns every 5 s while running
        print(f"{st['captures']} captures, {st['captures_per_s']:.1f}/s, "
              f"{st['skipped_groups']} skipped")
        if not st["running"]:
            break
finally:
    final = run.stop()                        # no-op once it has ended
if final["failed_files"] or final["stop_reason"] in ("disk_full", "error"):
    raise SystemExit(f"continuous capture: {final['stop_reason']}: "
                     f"{final['last_error']}")
```

### Quadros em tempo real em seu próprio pipeline

Para enviar quadros sincronizados para seu próprio código — aprendizado de máquina, um visualizador personalizado, seu próprio gravador — use `ArraySession.frame_stream()`. Cada tick gera um `GrabResult`: `{serial: numpy array}` para cada câmera, **tudo a partir de um único gatilho**, com o ID desse gatilho no `.group_id`. É a mesma implementação do `ArrayHandle.frame_stream()` da API do projeto, portanto, tudo o que está descrito em [Um gatilho por leitura](#one-trigger-per-read--group_id), [`band`](#exporting-a-single-band--band), [`align`](#aligning-the-streams--align) e [`bit_depth`](#bit_depth--trading-precision-for-bandwidth) se aplica a ambos.

```python
import chloros_sdk
from chloros_sdk import FrameFetchError

serials = ["213800234", "214000533", "214701288", "214701292"]   # master first

with chloros_sdk.connect_array(serials) as arr:
    try:
        for frames in arr.frame_stream(processing="raw", fps=10, duration=300,
                                       with_metadata=True):
            if not frames:              # no camera answered this tick
                continue
            if frames.errors:           # cameras missing from this trigger
                print("left out:", {s: str(e) for s, e in frames.errors.items()})
            for serial, (img, hdr) in frames.items():
                # img: numpy array (raw: one Bayer or mono plane)
                # hdr: group_id, frame_id, timestamp_ns, exposure_us, gain_db, ...
                my_pipeline(frames.group_id, serial, img, hdr["timestamp_ns"])
    except FrameFetchError as exc:      # the stream cannot continue
        print("stream ended:", exc.status, exc)
```

- **Cada conjunto gerado é um novo gatilho.** `frames.group_id` nunca se repete dentro de um fluxo; quando você solicita mais `fps` do que a velocidade de execução da matriz, o fluxo aguarda o próximo gatilho, de modo que ele é executado na velocidade mais lenta entre as duas.
- **Câmeras ausentes são identificadas, não preenchidas posteriormente.** Uma câmera que não pôde ser atendida naquele gatilho fica ausente do dicionário e é listada em `frames.errors`.
- **Ele termina de forma evidente.** O `FrameFetchError`, em caso de recusa, repetir-se-ia no próximo tick, após 60 s sem nenhum quadro ou após 60 s sem nenhum novo gatilho.**O que ele oferece.**O `frame_stream()` transfere quadros numpy inteiros pela conexão HTTP local e atinge um pico próximo a**170 MB/s**. Isso é suficiente para o processamento em tempo real de quadros brutos, ou de uma matriz agrupada ou de ROI, mas não para cada gatilho em resolução total — para manter**todos** os gatilhos, salve no disco com [`continuous()`](#continuoushandle). Medido em um conjunto de quatro câmeras (FOCN, FRGN, FRGB, F450) com transporte compactado; cada tick correspondia a um conjunto completo de um gatilho:

| | Sensor completo (matriz 14,2–14,4 fps) | Binning 2 × 2 (matriz ~33 fps) |
| --- | --- | --- |
| `frame_stream(processing="raw")` | 6,7 conjuntos/s | 22,2 conjuntos/s |
| `frame_stream(processing="debayered")` | 2,3 conjuntos/s | 9,3 conjuntos/s |
| `frame_stream(processing="display")` | 0,8 conjuntos/s | 3,0 conjuntos/s |
| MJPEG por câmera, 4 fluxos em 720p | 13,5 fps cada; cada quadro correspondente a um gatilho | ~25 fps cada; ~45 % dos gatilhos presentes em todos os 4 |

Use o `processing="raw"` para taxa de transferência e demosaico do seu lado; o nível `debayered` também remove o desvio de escuro do pacote de calibração, o que não ocorre com a desmosaicização do `raw`.

#### Fluxos MJPEG por câmera (sem RTSP)

**O RTSP não é compatível.** A interface de streaming é MJPEG sobre HTTP (`multipart/x-mixed-replace`, um JPEG por parte, cada parte com um `Content-Length`), um URL por câmera:

| URL | O que ele fornece |
| --- | --- |
| `GET /api/camera/stream/<serial>` | Visualização de 8 bits, até ~30 fps. Para um membro da matriz, `max_h` (padrão 720, 64–2160) e `jpeg_q` (padrão 80) definem o tamanho e a qualidade. |
| `GET /api/camera/<serial>/stream-annotated` | A mesma pré-visualização com as sobreposições da interface gráfica (zebra, grade, mira, histograma, pico, medidor pontual). O `ChlorosLocal.camera_stream()` faz a leitura para você. |
| `GET /api/camera/<serial>/stream-reflectance` | Refletância de 8 bits por banda — consulte [Fluxo de refletância ao vivo](#live-reflectance-stream-mjpeg). |

- **Cabeçalhos de disparo.** Para um membro da matriz, cada parte de todos os três fluxos contém `X-Chloros-Group-Id`, `X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Partes de câmeras diferentes com o mesmo `X-Chloros-Group-Id` correspondem ao mesmo acionador: reúna as partes e agrupe-as com base nesse cabeçalho para remontar um conjunto. Uma câmera autônoma não possui grupo: suas partes de pré-visualização não contêm nenhum desses cabeçalhos (suas peças de refletância possuem um `X-Chloros-Group-Id` vazio).
- **O fluxo de pré-visualização perde quadros sob carga.** Ele transmite o quadro mais recente e ignora aqueles que não conseguiu enviar a tempo; portanto, em altas taxas de matriz, nem todo gatilho chega ao fluxo de todas as câmeras (consulte a tabela acima). Para cada acionamento, use `frame_stream()` ou `continuous()`.
- **O `/api/camera/stream/<serial>` é pausado enquanto o fluxo ao vivo combinado da matriz estiver aberto** (`/api/camera/array/stream/<array_id>` — a visualização da matriz na GUI o abre). A transmissão de cada membro envia então um quadro de preenchimento “Pausado (transmissão da matriz ativa)”, sem cabeçalhos `X-Chloros-*`, e nada mais até que o fluxo combinado seja fechado; em seguida, ele retoma automaticamente. `stream-annotated` e `stream-reflectance` não são pausados. Feche a visualização de matriz da GUI quando um script ler os fluxos de pré-visualização de cada câmera.
- **Para números, e não imagens, use `frame_stream()`.**O formato JPEGso é com perdas e de 8 bits; o `frame_stream()` retorna os mesmos resultados sem nenhum codec no caminho.

### Captura em intervalos para levantamentos com drones

Um voo de levantamento precisa de um grupo de imagens sincronizadas a cada poucos metros de percurso, e não de todos os quadros que a matriz pode produzir. Duas configurações controlam isso:**`target_fps`**define a frequência com que as câmeras capturam um quadro, e**`capture_repeated(interval_s=…)`** define a frequência com que um quadro é salvo.**As câmeras continuam transmitindo entre os salvamentos.** A cada intervalo, o grupo sincronizado mais recente é salvo; os quadros intermediários não são gravados. A exposição automática se ajusta a cada quadro, de modo que a exposição já está estabilizada quando o salvamento ocorre. Não há modo em que as câmeras parem entre as fotos e sejam ativadas para cada uma delas: cada foto teria, então, que aguardar o início da transmissão e a estabilização da exposição.**Defina a taxa de quadros de acordo com a necessidade do intervalo.**Se deixada no modo automático, uma matriz opera na taxa mais alta que o host e a rede suportam. Definir uma taxa mais baixa reduz o aquecimento da câmera, a carga na rede e a carga na CPU do host sem alterar o que é salvo. Escolha uma taxa que proporcione**2 a 4 quadros por intervalo**:

- Cada salvamento aguarda o próximo quadro após ser solicitado; portanto, um grupo salvo chega até um período de quadro após o momento programado. Uma taxa mais alta mantém essa diferença pequena.
- A exposição automática se ajusta uma vez por quadro; portanto, uma taxa muito baixa diminui a rapidez com que a exposição acompanha uma nuvem que passa.

**Calcule o intervalo a partir do plano de voo:**

```
spacing    = along-track footprint × (1 − forward overlap)
interval_s = spacing / ground speed
```

Por exemplo, uma área de cobertura de 60 m com 80% de sobreposição para frente corresponde a um espaçamento de 12 m; a 10 m/s, isso resulta em `interval_s=1.2`, e um `target_fps=3` fixado resulta em 3 a 4 quadros por intervalo. A captura é baseada no tempo: o SDK não possui modo acionado por distância ou GPS; portanto, mantenha uma velocidade em relação ao solo constante nas linhas de levantamento.

**Salve os dados brutos na aeronave e calibre posteriormente.** O `interval_s` é medido a partir do início de cada salvamento; portanto, um salvamento que demore mais do que o intervalo faz com que o próximo comece atrasado e o espaçamento se desvie. Salvar os dados brutos juntamente com a leitura do sensor de luz (`processing="raw"`; a leitura é salva por padrão) mantém cada salvamento curto e garante que todos os quadros possam ser convertidos em refletância posteriormente.

```python
import chloros_sdk

INTERVAL_S = 1.2      # from the flight plan
FLIGHT_S = 20 * 60    # bounds the loop; it also stops at count= if given

with chloros_sdk.connect_array(serials, target_fps=3.0) as arr:
    groups = arr.capture_repeated(
        "survey/", interval_s=INTERVAL_S, duration_s=FLIGHT_S,
        processing="raw", render_index=False,
        on_capture=lambda i, r: print(f"group {i}: {len(r)} frames"))
```

- **`connect_array(target_fps=…)`**fixa a taxa desde o início;**`arr.set_target_fps(fps)`**altera-a durante o voo sem reconectar, e**`arr.set_target_fps(None)`** a retorna para o modo automático. Uma taxa acima do que o host e a rede podem suportar é reduzida para esse limite máximo — verifique `clamped` no resultado — e a taxa mínima é de 0,1 fps.
- **Não use `burst()` para reduzir o número de quadros.** Ele grava todos os quadros que a matriz produz; ele segue um `target_fps` fixado, portanto, é a ferramenta certa apenas quando todos os quadros são desejados.
- **O `target_fps` não corrige quadros corrompidos.** Reduzir esse valor diminui o número de quadros produzidos, não a velocidade com que cada um é enviado; para quadros corrompidos pelo GVSP, reduza o valor de `wire_ceiling_mbps` (consulte [Integridade da Matriz](#array-health--which-subsystem-is-losing-frames)).

```python
r = arr.set_target_fps(5.0)
if r["clamped"]:
    print(f"asked for {r['requested']} fps, running at {r['target_fps']}")
```

### Conectando-se a um array já conectado — `attach_array`

Se o array já estiver ativo (a GUI a abriu ou uma sessão anterior do SDK chamou `connect_array`), use `attach_array` para obter um identificador dela, em vez de se reconectar. `connect_array` sempre gera o erro “A câmera  <sn>já está na matriz<id>", nessa situação, porque enviar um POST com `/array/connect` para um membro do pool não é idempotente; `attach_array` lê `/api/camera/array/list` e faz a correspondência por array_id ou por números de série.

```python
import chloros_sdk

# By serials (matches if every serial is a member of one existing array)
arr = chloros_sdk.attach_array(
    ["213800234", "214000533", "214701288", "214701292"])

# By array_id (when you've already noted it down)
arr = chloros_sdk.attach_array("array-1779862544497")

# attach_array returns the same ArraySession as connect_array
arr.capture("output/", processing="reflectance")
```

Padrão: os scripts do SDK que compartilham o ambiente com a GUI da área de trabalho devem tentar `attach_array` primeiro e recorrer a `connect_array` se ainda não houver nenhuma matriz no pool.

```python
import chloros_sdk

try:
    arr = chloros_sdk.attach_array(serials)
except chloros_sdk.ChlorosConnectError:
    arr = chloros_sdk.connect_array(serials)
```

> **Um array anexado é emprestado, não pertence ao usuário.** Um identificador de `attach_array` nunca desativa a matriz: nem a saída do bloco `with` nem uma chamada explícita de `disconnect()` a libera (a chamada explícita imprime uma linha no stderr informando que deixou a matriz em execução). A mesma regra se aplica a `CameraSession` / `DAQSensorSession`. Portanto, o uso conjunto com a GUI é seguro, com ou sem `with`:
>
> ```python
> with chloros_sdk.attach_array(serials) as arr:
>     arr.capture("output/", processing="reflectance")
> # … the array stays up for the GUI
> ```
>
> Quando você realmente quiser encerrá-la — para se reconectar com um `wire_ceiling_mbps` diferente, por exemplo — chame `arr.force_disconnect()`. A reconexão após um `disconnect()` simples em um identificador conectado falha com a mensagem “A câmera <sn>já </sn>está <sn>na matriz<id>”, pois nada foi liberado.

### Auxiliar de Análise de Rede

Útil antes de abrir a matriz — verifica se as configurações propostas serão compatíveis:

```python
result = chloros_sdk.analyze_array_network(
    master_serial="214701288",
    slave_serials=["213800234", "214000533", "214701162"],
    width=2048, height=1536,
    pixel_format="BayerRG12",
    binning=1,
)

if result["status"] == "ok":
    print("Use the requested settings.")
elif result["status"] == "auto_capped_fps":
    r = result["recommended"]
    print(f"Keep the resolution; cap the trigger rate at {r['recommended_target_fps']} fps")
elif result["status"] == "auto_shrunk":
    r = result["recommended"]
    # width / height are sensor pixels BEFORE binning (what connect_array
    # takes with binning=); out_width / out_height is what is streamed.
    print(f"Connect with width={r['width']} height={r['height']} "
          f"binning={r['binning']} ({r['out_width']}x{r['out_height']} streamed)")
elif result["status"] == "needs_force_slip":
    print("Sim-sync impossible on this wire; force_tier='slip-emit-and-capture' required")
```

```python
analyze_array_network(
    master_serial,
    slave_serials=None,
    width=2048, height=1536,              # sensor pixels before binning
    pixel_format=None,                    # None = what connect applies (Mono12 / BayerRG12)
    binning=1,                            # 1 | 2 | 4
    force_tier=None,
    packed="auto",                        # "auto" | "on" | "off"
    backend_url="http://127.0.0.1:5000",
    timeout=30.0,
) -> dict
```

Nunca inicia um backend: um que não responde gera o erro `ChlorosConnectError`, enquanto uma solicitação recusada retorna como resultado. Um `packed` inválido gera o erro `ValueError`.

`status` é um dos seguintes: `ok` / `auto_capped_fps` / `auto_shrunk` / `needs_force_slip` (caso contrário, `error` — incluindo uma solicitação recusada: sem login, `error_code: AUTH_REQUIRED`, ou um plano abaixo do limite mínimo do SDK, `PLAN_UPGRADE_REQUIRED`). `auto_capped_fps` significa que a resolução solicitada se encaixa no anel RX apenas a uma taxa de disparo limitada — mantenha a resolução e passe de `target_fps=result["recommended"]["recommended_target_fps"]` para `connect_array` (consulte [Exemplo 6](#6-capability-probe-before-connecting-a-4-cam-array)). `auto_shrunk` aparece apenas quando o quadro solicitado não pode ser entregue em nenhuma taxa de quadros; o connect nunca reduz quadros por conta própria, portanto, aplique `recommended` você mesmo.

**Como interpretar a projeção** (mesmo modelo do painel “Configurações da Matriz” da GUI):

- **Burst (`frame_bytes_total`) é somado por câmera no formato de pixel real de cada câmera.**As câmeras**M3M**transmitem Mono12 (2 B/px) ou, com um `pixel_format`, o formato Mono de sua profundidade (`"BayerRG8"` é `Mono8`, 1 B/px); portanto, sem ele, um quadro de resolução total com 4 câmeras tem**~25 MB** com três câmeras mono, e não os ~12,6 MB que uma suposição de 8 bits em todos os canais indicaria. O backend determina o formato de cada câmera a partir de seu modelo.
- **A admissão (`burst_fits_nic_ring`) leva em conta o escoamento**, não se trata de uma comparação entre burst completo e anel: o sim-emit se aplica quando o host esvazia o anel de recepção mais rápido do que as câmeras o preenchem. Um host de 10G + câmeras de 1 GbE**admite** resolução total mesmo quando o burst excede o anel; um host de 1 GbE bloqueia (`needs_force_slip` / `auto_shrunk`).
- **`achievable_fps_max` é um limite máximo conservador para recuperação serial** — `max(readout+emit, N×emit)` com emissão por câmera limitada ao link de 1 GbE, independente da exposição. Por exemplo, ~2,8 fps para um conjunto de 4 câmeras em resolução total de 12 bits (corresponde aos ~2,7–3,0 medidos no tempo de execução). Modelo completo: [Referência CLI → Modelo de fps e rajada do conjunto](cli-reference.md#array-fps--burst-model).
- **Após uma captura contínua, a projeção indica o valor mantido.** Uma captura de 30 s ou mais em uma taxa automática registra a taxa em que se estabilizou para aquele tamanho de matriz neste computador; os breves testes do controlador de taxa com uma taxa mais rápida são desconsiderados. A partir daí, a linha do `array-connect`, a análise de `connect_array()` e o painel “Configurações da Matriz” citam essa taxa quando ela está abaixo da do modelo, com o que a limitou em `cycle_limiter`: `"clock"` (a matriz operou na taxa em que se estabilizou, com o loop de captura aguardando seu relógio de acionamento), `"loop"` (o loop de captura não tinha mais tempo), `"capture"` (a captura ficou para trás e pulou grupos) ou `"measured"` (não há indicação de qual). O valor do modelo está em `wire_fps_max`. A matriz ainda atinge o valor que a linha de comunicação transmite.
- **A supersubscrição (`oversubscribed: true`) significa que a linha de comunicação não consegue transmitir a matriz em nenhuma taxa de atualização alcançada pelas câmeras** — os campos de fps (`achievable_fps_max` / `fps_bright` / `fps_dark`) exibem 0, e o binning não consegue resolver isso (isso reduz os bytes por quadro, não os bytes sincronizados por segundo). Um cabo lento por si só não causa isso: em vez disso, a taxa de quadros cai e, se as câmeras não puderem ser sincronizadas a uma velocidade baixa o suficiente, o nível passa a ser `slip-emit-and-capture` (serial). As soluções são: usar menos câmeras, quadros jumbo ou uma placa de rede mais rápida; o `max_cams_collision_safe` indica o limite máximo para sua conexão e câmeras. A resposta também contém `aggregate_demand_bps`, `collision_safe_ceiling_bps` e `per_cam_floor_bps`. Consulte [Superscrição](#over-subscription-the-per-cam-floor).

### Descoberta e listagem

```python
chloros_sdk.discover_lattice_cameras()   # list all cams visible to the backend
chloros_sdk.list_cameras()               # cams currently in the pool
chloros_sdk.list_arrays()                # active arrays in the pool
```

Cada uma aceita apenas palavras-chave — `backend_url=` e `timeout=30.0` — e retorna uma lista de dicionários. `discover_lattice_cameras()` (`serial`, `ip`, `mac`, `model`, `lattice_model`, `vendor`, `connected`, `array_id`, …) também aceita `auto_start_backend=True`; as chamadas `list_*` nunca iniciam um backend.

---

## Smart-AE / Smart-Capture

As matrizes LATTICE executam AE contínuo em segundo plano assim que são conectadas, mas uma cena recém-focalizada leva um momento para convergir. **Smart-capture** é a solução prática integrada: ela verifica a exposição de cada câmera, aguarda até que a matriz esteja estável em toda a janela e, então, aciona a captura. É equivalente à interface gráfica: o botão de captura “inteligente” do aplicativo para desktop aciona o mesmo endpoint do backend.

```python
import chloros_sdk

with chloros_sdk.connect_array([
        "213800234", "214000533", "214701288", "214701292"]) as arr:
    # Initial pose
    arr.capture("pose_a/", processing="reflectance", smart=True)
    input("Move the rig, then press Enter...")
    # New pose — smart-capture waits for AE to re-settle automatically
    arr.capture("pose_b/", processing="reflectance", smart=True)
```

Ao utilizar o `ChlorosProject` (próxima seção), você tem mais opções de ajuste:

```python
proj.arrays["main_rig"].capture_smart(
    output_dir="out/",
    processing="reflectance",
    settle_timeout_s=5.0,           # max wait
    stability_window_s=1.5,         # exposure must hold steady this long
    exposure_tolerance_pct=5.0,     # %-spread allowed within the window
)
```

A política de AE inteligente é conservadora por padrão. Torne o `exposure_tolerance_pct` mais restritivo para trabalhos radiométricos exigentes; torne-o mais flexível para cenas que mudam rapidamente, nas quais você apenas deseja um resultado “suficientemente próximo”.

---

## Sessões de sensores DAQ

Pool de back-end persistente para sensores espectrais (DAQ-U via USB, DAQ-M via BLE, DAQ-E via Ethernet). Espelha a superfície da câmera: detecção inteligente, reutilização do pool, conexão idempotente.

### Detecção Inteligente (Configuração Zero)

```python
import chloros_sdk

with chloros_sdk.connect_daq_sensor() as daq:
    print(daq.model, daq.transport, daq.address)
    for frame in daq.latest(n=10):
        spectrum = frame["spectrum"]   # list[float] (W/m²/nm if calibrated)
        is_sat = frame["is_saturated"]
        x, y, z = frame["x"], frame["y"], frame["z"]
        print(len(spectrum), is_sat)
```

Prioridade: Ethernet → BLE → USB. Passe qualquer indicação explícita para fixar o transporte.

### Transporte Fixado

```python
# DAQ-U on a specific serial port
daq = chloros_sdk.connect_daq_sensor(transport="usb", port="COM3")

# DAQ-M over BLE by MAC (implies transport="ble")
daq = chloros_sdk.connect_daq_sensor(mac="AA:BB:CC:DD:EE:FF")

# DAQ-E over Ethernet by hostname (implies transport="eth")
daq = chloros_sdk.connect_daq_sensor(eth_host="daq-e-xxx.local")

# A specific sensor by id (as list_daq_sensors() shows it; ':' or '-' alike)
daq = chloros_sdk.connect_daq_sensor(sensor_id="CB-7C-A7-5A-73")

# Tuning knobs
daq = chloros_sdk.connect_daq_sensor(
    port="COM3",
    integration_time=64,      # ms
    frame_avg=20,
    enable_ae=True,
    start_streaming=True,
)
```

`sensor_id=` nomeia um sensor, e somente esse sensor é aberto. Ele é comparado com os sensores já abertos; em seguida, o nome de host ou IP de um DAQ-E (um DAQ-E que a varredura de rede detecta apenas pelo IP é consultado, em modo somente leitura, para obter seu nome, MAC e número de série, de modo que qualquer um deles também o localize), o MAC BLE de um DAQ-M e o ID que um DAQ-U ou DAQ-M reporta ao ser aberto (qualquer outro aberto para verificação é fechado novamente). Se não for encontrado, `connect_daq_sensor` gera o `ChlorosConnectError` listando o que foi encontrado; incluindo também um endereço (`port=` / `mac=` / `eth_host=`) que contenha um sensor diferente, também gera um erro. Ele nunca recorre ao sensor que estiver visível — é para isso que serve a chamada com o sinalizador zero.</id></sn></id></sn>

### Assinatura `connect_daq_sensor()`

```python
connect_daq_sensor(
    *,                                    # every argument is keyword-only
    sensor_id=None,                       # one named sensor (see above)
    transport=None,                       # "usb" | "ble" | "eth"; None = smart-detect
    port=None,                            # DAQ-U serial port ("COM3", "/dev/ttyUSB0")
    mac=None,                             # DAQ-M BLE MAC (implies transport="ble")
    eth_host=None,                        # DAQ-E hostname or IP (implies transport="eth")
    integration_time=None,                # ms; None = the sensor's own
    frame_avg=None,
    enable_ae=None,
    cap_id=None,                          # cap correction: "sunshine_cosine", "fov_30", ..., "none" (bare)
    start_streaming=True,
    backend_url="http://127.0.0.1:5000",
    timeout=60.0,
    auto_start_backend=True,              # spawn a local backend if none is running
) -> DAQSensorSession
```

Um sensor que já está no conjunto é devolvido tal como está (emprestado): `integration_time`, `frame_avg` e `enable_ae` não são aplicados a ele. Desconecte-o primeiro para abri-lo com suas configurações. O `cap_id` é a exceção: em um sensor emprestado, ele é definido com o `set_cap()`. Se omitido, prevalece o padrão do backend (consulte a nota abaixo da tabela de métodos).

### Métodos `DAQSensorSession`

| Método | Descrição |
| --- | --- |
| `status(timeout=10.0)` | Resumo da entrada no pool (estado de streaming/gravação, faixa de comprimento de onda, sha de calibração, tempo de integração, frame_avg, estado de AE). Para um DAQ-E cujo fluxo ficou silencioso: `reconnect` (onde a reconexão automática do backend ocorre: `state`, `attempts`, `next_attempt_in_s`, `last_error`; `None` enquanto estiver transmitindo), `stream_lost_s`, `reconnects` e `last_reconnect`, e `record_gaps` para a gravação em andamento; `rx_stream_fault` indica o estado de reconexão até que o primeiro espectro chegue. |
| `latest(*, n=1, timeout=10.0)` | Retorna até N quadros de espectro mais recentes (cada um `ts_ns`, `spectrum`, `is_saturated`, `integration_time`, `x`, `y`, `z`). |
| `stream_start()` / `stream_stop()` | Retomar / pausar a transmissão (o identificador permanece aberto). |
| `record_start(*, output_dir=None, device_name=None, timeout=15.0)` | Iniciar a gravação de um arquivo .daq. Retorna o caminho do arquivo. É recusado para DAQ-U/M sem um pacote de calibração AWS (DAQ-E está isento). |
| `record_stop()` | Interrompe a gravação. Retorna `{path, rows, warning, gaps}`; `warning` é `None` para uma gravação limpa, caso contrário, o diagnóstico do backend (por exemplo, um arquivo vazio ou uma lacuna no fluxo), que também é impresso no stderr. `gaps` lista cada trecho em que o fluxo não entregou nada: `start_utc`, `end_utc` (`None`: ainda perdido na parada), `duration_s`, `reconnected`, `marked_in_file`. Fica vazio para uma gravação contínua. |
| `set_cap(cap_id, timeout=15.0)` | Define o perfil de correção de limite aplicado às leituras a partir da próxima: `"sunshine_cosine"`, `"fov_15"` ... `"fov_90"` ou `"none"` (ou `None`) para um sensor sem difusor. Retorna o, cujo `cap_id` o lê de volta. Um DAQ-E-S mantém o `as_recorded` independentemente do que for solicitado (seu difusor é integrado). Aciona o `ChlorosConnectError` para um perfil que não carrega. CLIm: `chloros-cli daq pool-set-cap`. |
| `enable_ptp(timeout=15.0)` | Apenas DAQ-E: reativa o PTP para uma unidade cujo PTP esteja desativado (`status()["rx_ptp_enabled"]` é `False`; a configuração é salva na unidade, portanto ela permanece desativada até ser reativada). Retorna o status atualizado. Gera o código de erro `ChlorosConnectError` para um DAQ-U / DAQ-M, ou quando a unidade não confirma. O Chloros nunca reativa uma unidade por conta própria, pois o desligamento pode ser intencional. CLI: `chloros-cli daq pool-enable-ptp`. |
| `device_info(timeout=15.0)` | Apenas DAQ-E: as próprias configurações da unidade. `firmware` (`running`, `bundled`, `update_available`), `ip` e `ip_mode` (`dhcp` / `linklocal` / `static`: o que está em execução no momento), `network` (a configuração salva: `mode`, além de `ip` / `mask` / `gateway` quando estático), `network_change_pending` (válido até que a unidade execute a configuração salva, uma configuração salva que inclua `set_network(..., reboot=False)`), `auto_stream`, `ptp`. `firmware["update_available"]` é igual a `None` quando a versão da unidadenão foi possível ler a versão da unidade. Enquanto o fluxo da unidade não estiver chegando a este computador, ele é lido em uma nova conexão e o dicionário adiciona `stream_silent` (`True`), `stream_fault` e `reconnect`; uma unidade que não responde gera um erro. CLI: `pool-info`. |
| `set_network(mode="auto", *, ip=None, mask=None, gateway=None, reboot=True, force=False, comeback_timeout=45.0)` | DAQ-E apenas: DHCP (`"auto"`) ou um endereço estático. Reinicia a unidade para aplicar a configuração e a reconecta; esta sessão continua funcionando e o `address` acompanha a unidade. O `reboot=False` apenas salva a configuração. Consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings). CLI: `pool-set-network`. |
| `set_auto_stream(enabled, timeout=15.0)` | Apenas DAQ-E: transmite ao ligar, sem nenhum software conectado (salvo na unidade). Retorna a configuração confirmada. CLI: `pool-set-auto-stream`. |
| `reboot(*, comeback_timeout=45.0)` | Apenas DAQ-E: reinicie a unidade e reconecte-a com as configurações anteriores. CLI: `pool-reboot`. |
| `update_firmware(*, comeback_timeout=45.0)` | Apenas DAQ-E: atualize o firmware incluído nesta versão do Chloros em uma unidade que esteja executando uma versão mais antiga e, em seguida, reconecte-a (cerca de um minuto; não desligue a alimentação). Retorna `updated`, `fw_before`, `fw_after`, `bundled` e um `warning` quando a unidade demorou para retornar ou retornou com outra versão; `updated` é `False` para uma unidade que já está atualizada. Gera o código de erro `ChlorosConnectError` para uma unidade cuja versão não pode ser lida. CLI: `pool-update-firmware`. |
| `disconnect()` | Liberação do pool. Nãooperação para identificadores anexados que não pertencem ao usuário. Em um sensor aberto por esta sessão, é `force_disconnect()`, e gera o erro `ChlorosConnectError` quando a liberação não é confirmada (a saída do bloco `with` nunca gera exceção). |
| `force_disconnect(timeout=30.0)` | Libera o sensor independentemente de quem o abriu — por exemplo, um sensor que um script com falha deixou no pool, ao qual um novo script acessa apenas como um identificador anexado cujo `disconnect()` não realiza nenhuma ação. Encerrar o sensor de outro cliente interrompe qualquer gravação que ele esteja realizando. Retorna assim que o backend confirmar a liberação ou quando não houver nada a ser liberado (o sensor não está no pool ou nenhum backend está em execução); uma liberação que atingiu o tempo limite ou que foi recusada pelo backend gera o erro `ChlorosConnectError`. CLI: `chloros-cli daq pool-disconnect --sensor-id ID`. |

Atributos: `sensor_id`, `model`, `transport`, `address` (segue um DAQ-E que o `set_network()` reendereçou), `info` e `backend_url`.

> **Identifique a tampa.** Os modelos DAQ-U, DAQ-M e DAQ-E são fornecidos sem capacitor, e o Chloros não consegue detectar se há um instalado: um sensor conectado sem o `cap_id` recebe o `sunshine_cosine`, uma suposição que seus registros indicam ter sido feita (`cap_id_source=auto_default`). Um DAQ-U sem tampa, sob essa correção, apresenta uma leitura cerca de 30 vezes maior; portanto, passe `cap_id="none"` para um sensor sem tampa (ou `set_cap("none")`), e o perfil da tampa que você instalou, caso contrário.

### DAQ-E-S — difusor fixo e sensor de inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças relevantes para o software. Todo o restante — descoberta, conexão, PTP, gravação e as configurações do dispositivo abaixo — funciona exatamente como em um DAQ-E.**Seu difusor é permanente, portanto, não requer correção de tampa.** O difusor do tipo Sunshine é integrado e já estava na unidade quando sua calibração de fábrica foi medida; portanto, essa calibração já indica a irradiância descendente real. O Chloros define seu limite como `as_recorded` (sem perfil), independentemente do que for solicitado: um limite com esse nome (`sunshine_cosine`, `fov_*`, `none`) é ignorado com uma mensagem, e seus registros armazenam `cap_id_source=model`; portanto, o aviso de limite presumido nunca é acionado para ele. Também não aplique um limite aos dados manualmente: o `sunshine_cosine`, por exemplo, apresentaria uma leitura cerca de 11 vezes maior.

**Ele possui um acelerômetro de 3 eixos.** A resposta cosseno do difusor de luz solar só é válida enquanto ele estiver nivelado; portanto, um sensor inclinado relata uma irradiância incorreta — e a refletância herda o erro — sem apresentar nenhum outro sintoma. Cada leitura do DAQ-E-S, portanto, carrega sua atitude:

| Campo | Significado |
| --- | --- |
| `tilt_deg`, `roll_deg`, `pitch_deg` | Ângulos em graus. `None` quando o firmware marca o ângulo como indefinido (queda livre, um eixo inoperante). |
| `x_mg`, `y_mg`, `z_mg`, `mag_mg` | Aceleração em mili-g. `mag_mg` distante de ~1000 significa que a unidade estava em movimento e os ângulos não são significativos para essa amostra. |
| `cal_applied` | Definido: os eixos e ângulos estão no sistema de referência do difusor (a calibração de montagem da unidade é aplicada). Desmarcado: sistema de referência bruto da placa, que pode apresentar até ~2,3° de inclinação que não é real. |
| `sample_age_ms` | Tempo decorrido desde a amostra do acelerômetro até o envio do espectro. |
| `temp_raw` | **Tendência** da temperatura do chip não calibrada — apenas diferenças, nunca uma leitura do termômetro. |

O `DAQSensorSession.latest()` retorna esses valores sob a chave `imu` de cada quadro (`None` em um DAQ-U, DAQ-M ou DAQ-E padrão). Um `.daq` registrado a partir de um DAQ-E-S armazena os mesmos valores em suas colunas `imu_*` (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, `imu_mag_mg`, `imu_cal_applied`, …) ao lado de cada espectro, para que o leitor possa distinguir uma nuvem de um tripé que tenha sido derrubado. Em todos os outros modelos, essas colunas ficam vazias.

O fluxo de atitude relata seus próprios campos em `status()` (`rx_imu_stream_fault`, `rx_imu_stream_faults`, `rx_last_imu_stream_fault`), separados dos campos espectrais para que uma falha no acelerômetro nunca seja interpretada como uma lacuna no registro. O `discover_daq_sensors()` e o `model` da sessão indicam o `DAQ-E-S`.

### Configurações do dispositivo DAQ-E

```python
with chloros_sdk.connect_daq_sensor(eth_host="daq-e-def330.local") as daq:
    info = daq.device_info()
    print(info["ip"], info["ip_mode"], info["firmware"])

    # A fixed address on the network this computer is on
    daq.set_network("static", ip="192.168.1.50", gateway="192.168.1.1")
    print(daq.address)              # 192.168.1.50: the session follows it

    if info["firmware"]["update_available"]:
        daq.update_firmware()
```

- **Deve haver um endereço estático na rede deste computador.** O `set_network` gera o erro `ChlorosConnectError` para quem tiver uma sub-rede que não inclua o endereço deste computadorna rede do sensor: após a reinicialização, nada conseguiria acessar o sensor para reverter a configuração. Aplique o código `force=True` somente quando este computador for reendereçado ou roteado para a nova sub-rede.
- **`set_network`, `reboot` e `update_firmware` reiniciam o sensor** e geram o código de erro `ChlorosConnectError` enquanto ele grava, e quando ele não retornar dentro de `comeback_timeout` segundos. O ID do sensor e o nome do host `daq-e-<id>.local` nunca mudam.
- **Uma gravação iniciada pouco antes da reinicialização é encerrada por ela e recebe um nome.** Quando o sensor retorna, o dicionário retornado contém `recording_closed` (o caminho do arquivo) e um `warning` indicando isso. Quando ele não retorna, ou a transferência do firmware falha, o `ChlorosConnectError` indica isso: sua mensagem termina com o aviso, e `e.recording_closed` / `e.warning` são definidos (ambos, `None`, quando nenhuma gravação foi encerrada). Reinicie a gravação assim que o sensor estiver de volta.
- **Apenas o próprio sensor é reconectado.** Após a reinicialização, a unidade deve responder onde estava — e, após o `set_network("auto")`, também sob seu nome `.local` — com seu nome, nome do host e modelo, a partir do adaptador de rede (MAC) que possuía; `address` torna-se o IP que respondeu. Uma resposta sob o nome só é válida com aquele MAC; portanto, a partir de um computador que acessa o sensor por meio de um roteador ou da rede NAT padrão do WSL2, um sensor que foi movido pelo DHCP não é reconectado (`ChlorosConnectError`; reconecte-o em seu novo endereço).
- **Uma atualização de firmware requer que o sensor se reconecte.** `update_firmware()` convida a unidade, e a unidade abre uma conexão TCP de volta para este computador para a imagem, em uma porta escolhida pelo sistema operacional. O backend escuta no adaptador que se conecta à unidade (em todos os adaptadores, apenas quando não é possível identificar qual) e atende apenas essa unidade: qualquer outra conexão é recusada (`[DAQ-FW] … refused a firmware connection from <ip>` na saída do backend) enquanto ele permanece aguardando. Um firewall que bloqueia a conexão faz com que a atualização falhe com o código `device never connected back on TCP <port>`. O instalador do Windows adiciona a regra para isso (consulte [Etapa 1](#windows-exe)); em outro firewall, permita o tráfego TCP de entrada para o backend proveniente da sub-rede do sensor.
- **No WSL2, use rede espelhada** (`networkingMode=mirrored` em `.wslconfig`): no modo NAT padrão, todos os endereços estáticos são recusados (o próprio endereço do WSL não está na rede do sensor), o fluxo não chega ao WSL e uma atualização de firmware não pode ser concluída.
- A GUI mantém sua própria conexão com um sensor: uma reinicialização a partir de um script encerra a sessão da GUI com ele. A GUI possui as mesmas configurações (aba “Sensores de Luz”, configurações do sensor, “Rede”).

### Descoberta — localizando um endereço para se conectar

O `discover_daq_sensors()` varre as interfaces USB / BLE / ETH em busca de sensores que você *poderia* abrir. É a contraparte DAQ do `discover_lattice_cameras()` e a única maneira de obter o **MAC BLE de um DAQ-M** — um DAQ-E possui um nome de host e um DAQ-U, uma porta COM, mas o MAC não está impresso no dispositivo nem listado pelo sistema operacional.

```python
for s in chloros_sdk.discover_daq_sensors():
    print(s["transport"], s["address"], s["model"], s["extra"])
# ble  C3:D8:85:E0:0A:19  DAQ-M  {'name': 'NSP32_SPECTRUM'}
# usb  COM3               DAQ-U  {'vid': '0403', 'pid': '6001', 'manufacturer': 'FTDI'}

# `address` is exactly what connect_daq_sensor wants:
for s in chloros_sdk.discover_daq_sensors(transports=["ble"]):
    if s["model"] == "DAQ-M":
        daq = chloros_sdk.connect_daq_sensor(mac=s["address"])
```

| Campo | Descrição |
| --- | --- |
| `transport` | `usb` \| `ble` \| `eth`. |
| `address` | Porta COM / MAC BLE / nome do host — repassar para `connect_daq_sensor` como `port=` / `mac=` / `eth_host=`. |
| `display` | Rótulo legível por humanos. |
| `model` | `DAQ-U` \| `DAQ-M` \| `DAQ-E` \| `DAQ-E-S`, ou `None`. Um sensor Ethernet é o `DAQ-E-S` ou o `DAQ-E`, conforme ele mesmo se identifica quando solicitado, em modo somente leitura, durante a varredura (`DAQ-E` quando não responde a tempo). Uma porta serial USB exibe o valor `DAQ-U` quando seu chip de ponte é um dos utilizados pelo DAQ-U (FTDI, CH340, CP210x, Prolific) — uma dica da descrição da porta, não um probe — e `None` para qualquer outro dispositivo USB, que ainda está listado para que você possa passá-lo como `port=`. Uma porta serial sem ID de fornecedor USB e sem esse chip de ponte (um UART da placa-mãe, como o `/dev/ttyS*` do Linux, um link serial Bluetooth, Intel AMT) não pode ser um DAQ-U e não está listado. |
| `extra` | Detalhes por transporte (nome anunciado do BLE; USB `vid`, `pid`, `manufacturer`, `serial_number`; DAQ-E ip/fw/…). Valores vazios são omitidos. |

| Parâmetro | Padrão | Descrição |
| --- | --- | --- |
| `transports` | todos os três | Sequência (ou string CSV) que limita a varredura. Vale a pena especificar quando você sabe o que deseja — o BLE é o elo mais lento. |
| `scan_timeout` | 5 | Janela de varredura por transporte em segundos; o backend limita o valor a 1–20. |
| `timeout` | 60,0 | Limite de HTTP para toda a chamada (como em outras partes do SDK). |
| `auto_start_backend` | `True` | Inicia um backend local se nenhum estiver em execução. Nunca é iniciado para um `backend_url` remoto. |

> **Sensores já abertos no pool não aparecem.** Um periférico BLE conectado interrompe a transmissão de anúncios e uma porta COM aberta não pode ser detectada; portanto, a descoberta lista o que está *disponível para conexão*. É esperado um resultado vazio logo após você conectar algo — use `list_daq_sensors()` para o que você já possui. Transportes cuja varredura não pode ser executada (sem bleak/zeroconf instalados) são ignorados em vez de gerar erros, de modo que uma máquina sem Bluetooth ainda recebe suas respostas de USB e ETH.

### Listagem

```python
for s in chloros_sdk.list_daq_sensors():
    print(s["sensor_id"], s["model"], s["transport"], s["wavelength_range"])
```

### Qual DAQ alimenta qual câmera — `daq_bindings()`

A refletância de uma captura e o `.daq` que ela salva utilizam o DAQ vinculado àquela câmera ou matriz; caso contrário, utilizam o DAQ mais recente disponível no backend. A seleção do sensor de luz na GUI e o `chloros-cli daq pool-bind` definem isso; assim como:

```python
chloros_sdk.set_daq_bindings(cameras={"214701281": "COM7"},          # a sensor id or an address
                             arrays={"array-1779862544497": "daq-e-def330"})
chloros_sdk.set_daq_bindings(cameras={"214701281": None})            # None clears it
for serial, b in chloros_sdk.daq_bindings()["cameras"].items():
    print(serial, b["binding"], b["source"], b["resolved"]["in_pool"])
```

`daq_bindings()` retorna `{"cameras": {serial: b}, "arrays": {array_id: b}}`: cada `b` contém `binding` (o valor definido), `source` (`project`, do `sensors.json` do projeto ou do `runtime`), `known` / `known_as` (se a chave da câmera ou da matriz nomeia algo que Chloros conhece, e como: `connected`, `project` — o `cameras.json` do projeto aberto — ou, para uma câmera, `seen`: já aberto neste computador anteriormente; `known: False` para uma chave que não nomeia nada) e `resolved` (o sensor de pool ao qual ele está mapeado agora: `in_pool`, `sensor_id`, `address`, `model`, `streaming`). Uma ligação em tempo de execução substitui as do projetodurante a sessão; o arquivo do projeto não é alterado. Desconectar uma câmera, um array ou um DAQ cancela todas as ligações desse hardware, independentemente de quem as tenha definido (os membros de um array são ligados junto com o próprio array).

`set_daq_bindings()` vincula apenas uma câmera que esteja conectada, no projeto aberto ou que já tenha sido aberta neste computador anteriormente, e uma matriz que esteja conectada ou no projeto aberto (ou uma tecla já vinculada). Qualquer outra tecla gera o erro `ChlorosConnectError`, nomeando-a (`400`, `code`, `UNKNOWN_BINDING_TARGET`, com as teclas `unknown`), e **nada** nessa chamada é vinculado. `None` limpa qualquer chave.

### Refletância ambiente + objeto — `light_sensor_reflectance()`

Dois DAQs agrupados, um voltado para o céu (ambiente) e outro para o alvo (objeto), fornecem a refletância do alvo e os índices de vegetação a partir de suas últimas leituras: o par Ambiente + Objeto da GUI e o `chloros-cli daq reflectance`, sem precisar abrir os sensores uma segunda vez. Ambos devem estar conectados e transmitindo dados.

```python
r = chloros_sdk.light_sensor_reflectance("daq-e-def330", "daq-e-54b774")
print(r["reflectance_type"])                       # "Apparent Reflectance" when both are calibrated
for name, band in r["bands"].items():
    print(name, band["rho"] if band["valid"] else band["reason"])
for idx in r["indices"]:
    print(idx["name"], idx["value"])
```

```python
light_sensor_reflectance(
    ambient_sensor_id, object_sensor_id, *,
    bands=None,          # [{"name", "center_nm", "fwhm_nm", "role"}]; role blue / green / red / nir
    preset=None,         # a preset id from daq_band_presets(); default: the suggested one, else "GENERIC"
    expressions=None,    # custom indices: [{"name": "MyNDVI", "expr": "(NIR_850 - Red_660) / (NIR_850 + Red_660)"}]
    builtins=True,       # include the built-in indices
    backend_url="http://127.0.0.1:5000", timeout=30.0,
) -> dict
```

- **Bandas.** A refletância de cada banda é a irradiância integrada da banda do objetosobre a do ambiente. O `daq_band_presets(filter_code=None)` lista os conjuntos de bandas: um por filtro LATTICE M3C (`RGN`, `OCN`, `NGB`, o modelo de bandas da própria câmera; portanto, um NDVI do DAQ e um NDVI da câmera representam a mesma grandeza) e a banda larga `GENERIC`. Seu `suggested` é a predefinição correspondente a `filter_code`; caso contrário, é o filtro da primeira câmera conectada (`suggested_from` indica qual).
- **Retorna** `bands` (por banda: `rho`, `valid`, `reason`, `e_ambient`, `e_object`), `indices` (os integrados), `custom` (cada um com `value`, ou `valid: False` e o `error`), `invalid_bands`, `reflectance_type`, `preset` e as duas leituras de `timestamps` (`ts_ns`). Uma faixa que o sensor ambiental não conseguiu medir (fora de sua faixa de medição ou no nível de ruído de fundo) é considerada inválida, com a respectiva justificativa, e todos os índices que dependem dela são omitidos, nunca sendo calculados com base em zero.
- **Aparente ou relativa.** Os índices dependentes de escala (SAVI, OSAVI, EVI, MSAVI2, RDVI, TDVI, LAI, NLI, MNLI, FCI, GEMI) precisam de uma razão absoluta; portanto, só são gerados quando ambos os sensores estão calibrados em potência (cada um possui um `calibration_bundle_sha`): `"Apparent Reflectance"`. Caso contrário, o resultado é `"Relative Reflectance"`, contendo apenas os índices invariantes em relação à escala.
- **Duas grades de comprimentos de onda diferentes** são comparadas quanto à sua sobreposição: os pontos do sensor ambiente dentro do alcance do objeto, com o espectro do objeto interpolado sobre eles.
- **Dois sensores, nunca apenas um.** Nomear um sensor como ambos (o mesmo ID, em qualquer grafia que o pool aceite) gera o erro `ChlorosConnectError`: a leitura de um sensor sobre si mesmo é 1,0 em todas as bandas, o que não mede nada. O backend também rejeita isso (`POST /api/daq/indices` responde a `400`, `code` e `SAME_SENSOR`, enquanto o `ambient_sensor_id` e o `object_sensor_id` enviados pelo SDK correspondem a um único sensor agrupado, por exemplo, seu ID e sua porta).

### Compartilhamento com a CLI e outros scripts

O pool de sensores DAQ do backend é compartilhado por todos os clientes CLI e SDK na máquina. Se um sensor já estiver no pool — aberto por `chloros-cli daq pool-connect` ou por outro script —, chamar `connect_daq_sensor(port="COM3")` a partir do Python retorna um identificador emprestado: seu `disconnect()` é uma operação nula, de modo que seu script não retira o sensor do cliente que o abriu ao sair do escopo. O mesmo se aplica a `sensor_id=` (digitado com `:` ou `-`; qualquer um dos dois é válido), para um DAQ-E identificado pelo IP quando foi aberto pelo nome do host (ou vice-versa), e para a detecção inteligente com sinalizador zero quando o backend responde com um sensor que já possuía — um bloco `with` só libera um sensor que ele mesmo abriu.

A GUI do Chloros **não** faz parte desse pool: seu painel DAQ mantém sua própria conexão com o sensor, portanto o `connect_daq_sensor` nunca utiliza um sensor que a GUI tenha aberto. Não utilize o mesmo sensor pela GUI e por um script ao mesmo tempo. Um DAQ-U ou DAQ-M mantido pela GUI normalmente simplesmente não consegue ser aberto, mas um DAQ-E aceita uma segunda sessão: a conexão do script altera as configurações de aquisição da unidade na GUI, e seu `disconnect()` interrompe o fluxo da unidade, o que encerra a visualização ao vivo da GUI e qualquer gravação que ela esteja realizando. Feche o sensor na GUI primeiro.

### PTP Grandmaster — `time_sync_status()`

O backend executa o PTP grandmaster do qual as câmeras LATTICE e as unidades DAQ-E obtêm seus carimbos de data/hora. Ele é iniciado sob demanda: `connect_camera()`, `connect_array()`, `connect_daq_sensor()` e `connect_all()` o iniciam caso ele não esteja em execução — incluindo uma conexão que falha em seguida, como o `connect_all()` tentando acessar um DAQ-E que não está presente — e o mesmo ocorre com o `test_array_sync()` e todas as chamadas que abrem câmeras para um comando (`capture_cameras_transient()`, `start_multi_camera_stream()`, as chamadas de alinhamento, `capture_live_index()`).

- **Onde ele se anuncia.** Nos adaptadores de rede com fio conectados às câmeras (incluindo um endereço link-local `169.254.x.x`); caso contrário, no adaptador da rota padrão, se este for com fio. Nunca em Wi-Fi ou em um adaptador virtual (Tailscale, WSL): nenhuma câmera ou DAQ-E está conectada a um desses. Sem nenhum adaptador desse tipo ativo, ele continua anunciando sem destino (`health` `NO_SOCKETS`) e vincula-se ao adaptador assim que a próxima câmera ou DAQ se conectar.
- **Após uma queda de conexão.** Quando o adaptador da câmera perde a conexão e a recupera (ao desconectar um cabo, substituir um switch ou reenumerar um adaptador USB), o grandmaster reconstrói seus soquetes sozinho, sem reinicialização, e verifica novamente todos os DAQ-E conectados, reativando aquele que ficou travado na lacuna. Ele também se reconstrói quando todas as câmeras e DAQ-E que estava atendendo ficam inativos por 15 s.
- **Está alcançando o cabo?** Leia `time_sync_status()` (`GET /api/time-sync/status`). `state` apenas indica que o thread do grandmaster está em execução; `health` indica se seus pacotes estão sendo enviados: `OK` (nenhuma falha detectada), `STOPPED`, `NO_SOCKETS`, `SENDS_FAILING`, `DEGRADED` (alguns adaptadores com falha) ou `PEERS_SILENT` (as câmeras e os DAQ-Es que ele atendia pararam de responder), com uma mensagem em `health_detail`. `announce_sent` / `sync_sent` / `delay_resps_sent` contam apenas as mensagens que foram enviadas; `announce_send_errors`, `sync_send_errors`, `follow_up_send_errors`, `delay_resp_send_errors`, `last_send_error`, `failing_interfaces` e `last_delay_req_age_s` descrevem as que não foram enviadas, e `rebinds` / `last_rebind` as reconstruções. `sync_sends_slow` conta as sincronizações cujo envio demorou mais de 200 us, e `sync_send_max_us` é a mais demorada; o tempo de origem de cada Follow_Up conta, no máximo, 200 us de seu envio. O grandmaster é executado em um processo próprio (`own_process`, `process_pid`), com alta prioridade no Windows (`process_priority`), de modo que um backend ocupado não possa atrasar os horários que registra e ajustar o relógio de todas as câmeras de uma só vez; se esse processo não puder ser iniciado, ele é executado dentro do backend como antes (`own_process` false).

```python
st = chloros_sdk.time_sync_status()
if st.get("health") not in (None, "OK", "STOPPED"):
    print("PTP:", st["health"], "-", st.get("health_detail"))
    chloros_sdk.restart_time_sync()
```

| Função | O que faz |
| --- | --- |
| `time_sync_status()` | O estado do grandmaster, sua integridade, identidade do relógio, prioridades e contadores de mensagens (acima). `chloros-cli time-sync status`. |
| `time_sync_peers()` | Os escravos PTP que ele detecta (câmeras, DAQ-Es): `peers`, cada um dos `port_identity`, `last_seen_ip`, `last_delay_req_age_ms`, `delay_req_count`. `time-sync peers`. |
| `time_sync_cameras()` | Cada câmera de matriz conectada: `PtpStatus`, `PtpOffsetFromMaster` e `PtpMeanPathDelay` (uma câmera cujo firmware não possui nós PTP tem apenas o número de série). `time-sync cameras`. |
| `restart_time_sync()` | Interrompe e reinicia o grandmaster; o PTP de cada DAQ-E agrupado é verificado novamente após (`daq_ptp_rechecked`). Gera o evento `ChlorosConnectError` quando não retornar. `time-sync restart`. |
| `set_time_sync_priority(priority1=None, priority2=None)` | Suas prioridades PTP (0-255; quanto menor, maior a chance de vencer a eleição de best-master), para uma rede com outro mestre PTP. É necessário que ele esteja em execução. `time-sync set-priority`. |

Cada um recebe `backend_url=` e `timeout=` e inicia um backend local que não está em execução.

---

## Utilitários da câmera

O que o `chloros-cli lattice` faz além de conectar e capturar, como funções simples no `chloros_sdk` (módulo `chloros_sdk.camera_utils`). Cada um chama a rota do backend que o comando da CLI usa, seguindo as regras da CLI, de modo que as duas se comportam da mesma forma. Elas recebem um **número de série** da câmera, não uma sessão, e as mesmas palavras-chave `backend_url=` / `timeout=` que as chamadas de conexão inteligente; um backend que não esteja em execução é iniciado, assim como o `connect_camera()` faz. Uma recusa gera o `ChlorosConnectError` com o motivo do backend.

```python
import chloros_sdk as c

c.set_camera_color_profile("214701281", profile="natural")
c.set_camera_highlights("214701281", -40)
print(c.camera_firmware_status()["cameras"])
st = c.network_adapters(estimate=True, cameras=4, pixel_format="BayerRG12")
```

| Função | O que faz |
| --- | --- |
| `camera_color_profile(serial)` / `set_camera_color_profile(serial, profile=, custom_cct_k=)` | O perfil de cor da tela: `raw`, `linear`, `natural`, `enhanced` ou `custom_temp`, e sua temperatura de cor (limitada a 2.000–10.000 K; lida quando o perfil é `custom_temp`). |
| `camera_color_adjust(serial)` / `set_camera_color_adjust(serial, saturation=, contrast=, reset=False)` | Saturação e contraste da tela, 0 a 200 % cada (100 = inalterado); `reset=True` redefine ambos para 100. |
| `camera_highlights(serial)` / `set_camera_highlights(serial, value)` | Reduz a intensidade dos pixels mais brilhantes e mais brancos da renderização da tela: -100..0 (truncado em direção a zero e, em seguida, limitado; 0 = desativado). |
| `camera_white_balance_once(serial, settle_s=None)` | Equilíbrio de branco de uma única vez: a câmera transmite brevemente, converge e retorna `red_ratio` / `blue_ratio` (`ratios_read` False quando não consegue reportá-los). Ela mantém o equilíbrio de branco calculado; todas as outras configurações que seu comando “open” escreveu são restauradas como estavam. |
| `set_camera_white_balance(serial, mode=None, red=None, blue=None)` | `mode="auto"` (contínuo) ou `"off"`; ou proporções de equilíbrio fixas `red` / `blue`, que desativam o modo automático primeiro. Uma relação que a câmera rejeita é identificada como `errors`. |
| `camera_presets()` / `apply_camera_preset(serial, preset)` | As predefinições da câmera; ao aplicar uma delas, a câmera mantém as configurações indicadas pelos nomes das predefinições (a resposta é `settings`, conforme a câmera os lê posteriormente, por exemplo, a taxa de quadros em que opera; `None` para um campo que a predefinição não nomeia), e todas as outras configurações que seu arquivo de configuração abriu voltam ao estado em que foram encontradas. `"default"` redefine a câmera para seus padrões de fábrica (suas configurações de fábrica com exposição automática livre, o limite de taxa de transferência do link e a correção de defeitos desativados, mantendo o equilíbrio de branco calibrado de uma câmera RGB), sem salvar na memória: a resposta contém `factory_defaults: True` e todos os campos de `settings`. Em `camera_presets()`, `default` é igual a `{factory_defaults: True, description}`. |
| `camera_filter_models()` / `camera_filter_model_info(name)` | Os modelos de filtro e os detalhes de um modelo (bandas, número de condição de resposta espectral). O modelo de filtro de uma câmera é lido a partir de sua string de modelo; ele não é definido manualmente. |
| `camera_firmware_status(serial=None)` | O firmware de cada câmera: `current_version`, `available_version`, `update_offered`, `needs_minimum_update` e `minimum_firmware`. |
| `update_camera_firmware(serial, force=False)` | Grave o firmware desta versão do Chloros em uma câmera: redefinição de fábrica, gravação, reinicialização; as configurações básicas do Chloros são reescritas; a calibração da câmera é mantida. Vários minutos; a câmera deve permanecer ligada. `force=True` atualiza novamente uma câmera já em uso. |
| `camera_power_probe(serial)` | Os nós de alimentação e térmicos da câmera e sua temperatura. |
| `sleep_camera(serial)` / `wake_camera(serial)` | O modo de inatividade de baixo consumo de energia da câmera (ela permanece em modo de suspensão até ser ativada: seu limite de taxa de transferência de link ativado (`DeviceLinkThroughputLimitMode` `On`) no valor mínimo, o modo de economia de energia ativado, caso a câmera possua um), com a temperatura antes e depois; o despertar restabelece a conexão exatamente como estava quando entrou em modo de suspensão (o limite, seu modo e o atraso de pacotes `GevSCPD`, que a câmera mantém em 0 enquanto o limite está ativado; o registro é mantido neste computador, de modo que um despertar de um backend posterior também o encontre), ou, para uma câmera que este computador não colocou em modo de suspensão, mas que parece estar em suspensão (com o limite ativado no valor mínimo), com o limite desativado e um `GevSCPD` mantido em 0 a 80 (as configurações de fábrica); um limite definido acima do valor mínimo é mantido como está. Apenas o estado de energia é mantido; todas as outras configurações cujos valores foram alterados voltam ao que foram encontrados. |
| `calibrate_camera_with_target(serial, filter=None, attempts=30, settings=None)` | Capturar quadros até que um alvo de calibração MAPIR esteja em campo de visão e ajustar os coeficientes por canal a partir de seus painéis: `target_id`, `panel_dns`, `coefficients`. |
| `network_adapters(estimate=False, cameras=1, pixel_format="BayerRG8")` | Adaptadores Ethernet deste computador para GigE Vision (MTU, velocidade de link, quadros jumbo) e qualquer `issues`; `estimate=True` adiciona a taxa de quadros e a largura de banda que as câmeras `cameras` obteriam. |
| `fix_network_settings()` | Aplique as configurações do adaptador GigE Vision ao. Isso altera as configurações do adaptador e funciona apenas quando o Chloros é executado como Administrador; caso contrário, exibe um log com o que foi encontrado. |
| `apply_vignette_correction(input_dir, output_dir, lens_model=None)` | Corrija a vinheta das imagens em uma pasta, cada uma com o mapa de campo plano de sua própria lente: `processed`, `lens_models` (a lente para a qual cada arquivo foi corrigido), `skipped` (com justificativas). Sem o `lens_model`, a lente de cada arquivo é determinada a partir do campo “Modelo” do EXIF; um `lens_model` se aplica a arquivos cujo Modelo não especifica nenhuma câmera, e um arquivo cujo Modelo contradiga isso é ignorado. Apenas arquivos. |
| `read_dls(mac, filter=None, camera_serial=None)` | Uma leitura de um DLS (sensor Bluetooth de luz descendente): espectro, irradiância por banda e magnitudes de banda por meio da própria resposta da câmera quando o `camera_serial` é fornecido. O `discover_daq_sensors(transports=["ble"])` identifica o MAC. |
| `probe_system(pixel_format="BayerRG12", discover=True)` / `system_info()` | O hardware do host para câmeras e a configuração recomendada (`report_text`), além da CPU, GPU e tempo de execução do PyTorch do sistema. |

- **Algumas precisam da câmera exclusivamente para si.** `camera_white_balance_once()`, `apply_camera_preset()`, `sleep_camera()` / `wake_camera()`, `calibrate_camera_with_target()` e `update_camera_firmware()` abrem a câmera. Enquanto o Chloros a mantém ocupada (conectada no aplicativo, com o `connect_camera()`, como membro de uma matriz ou em uso por outro comando no momento, como um teste de matriz ou uma captura abrindo suas câmeras), o backend recusa com o código HTTP 409 `CAMERA_HELD` (tanto no `code` quanto no `error_code`), indicando como liberá-lo. As configurações de cor, destaques, `camera_power_probe()` e as leituras funcionam em uma câmera conectada.
- **Cor, destaques e balanço de branco são para câmeras RGB.** Em uma câmera monocromática de banda única (M3M), os setadores não alteram nada e retornam `{"skipped": True, "reason": ...}`, assim como o CLI. São configurações de exibição: a visualização ao vivo e as capturas no nível de exibição, nunca radiação ou refletância.

---

## Matrizes transitórias, perfis de alinhamento e índices

O que o `chloros-cli lattice` faz sem uma conexão persistente, assim como as funções no `chloros_sdk` (módulo `chloros_sdk.array_utils`): `continuous`, `multi-capture`, `multi-stream`, `multi-test`, `multi-detect`, `analyze-array`, `align-calibrate`, `align-apply`, `align-stream`, `align-info`, `align-reorder` e `index`. Cada um envia a solicitação que o comando da CLI envia, com os padrões da CLI.

```python
import chloros_sdk as c

prof = c.calibrate_alignment_profile(["214000535", "214000551", "214000556"],
                                     "D:/rig/alignment_profile.json", method="orb", model="rigid")
print({sn: t["reprojection_error"] for sn, t in prof["transforms"].items()})

c.capture_aligned_multiband("D:/rig/alignment_profile.json", "D:/rig/out")
idx = c.capture_live_index("D:/rig/alignment_profile.json", preset="NDVI",
                           output_dir="D:/rig/ndvi", capture_level="reflectance")

with c.start_multi_camera_stream(["214000535", "214000551"], "D:/rig/stream", fps=5, count=50) as job:
    print(job.wait()["final_stats"])          # waits for the 50 groups, then closes the cameras
```

- **Eles abrem as câmeras por conta própria.** Cada comando abre suas câmeras, executa sua tarefa e as fecha; portanto, precisa de câmeras que o Chloros não esteja mantendo (conectadas no aplicativo, com o `connect_camera()` ou como membro de uma matriz); o backend recusa com o código HTTP 409 `CAMERA_HELD` e indica como liberar uma. Uma abertura transitória leva algum tempo (cerca de 40 s para algumas câmeras). Para capturas repetidas, `connect_array()` uma única vez é a opção mais rápida e economiza em todos os tipos de exportação, enquanto uma captura transitória salva uma imagem sem debayering. `calibrate_alignment_profile()` é a exceção: em uma matriz conectada dessas câmeras, ele resolve a partir dos próprios quadros da matriz (`used_live_array`).
- **`serials`** são as câmeras a serem abertas, sendo que a primeira aciona o gatilho; `None` abre todas as câmeras da rede.
- **`settings`** é um dicionário de substituições de configurações de câmera, os campos definidos pelos sinalizadores da CLI: `exposure_time` (µs), `gain` (dB), `pixel_format`, `width`, `height`, `target_brightness`, `ae_damping`, `ae_upper_limit`, `frame_rate`, `gev_packet_size` e `preset` (uma predefinição de câmera por nome). Um valor de exposição ou ganho especificado desativa esse controle automático, assim como os sinalizadores.
- **`gpu`**: `True` força o warp da GPU, `False` o da CPU (`--gpu` / `--no-gpu`); `None` permite que o backend escolha.
- **`keep_settings`**/**`clear_settings`** (todas as funções que abrem câmeras, padrão: `False`): no final, cada câmera é restaurada ao estado em que foi encontrada; `keep_settings=True` salva as configurações da execução na própria memória da câmera (funcionamento livre), `clear_settings=True` redefine essa memória para as configurações de fábrica — consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings). A resposta (para uma tarefa, a do `stop()`) contém o `camera_memory`.
- **Os caminhos são do backend**: os relativos são transformados em absolutos em relação ao seu diretório de trabalho quando o backend está nesta máquina. As funções do arquivo de perfil de alinhamento (`alignment_profile_info()`, `reorder_alignment_profile()`) leem o arquivo neste computador, assim como a CLI.

| Função | O que faz |
| --- | --- |
| `start_camera_continuous(serial, output_dir="output", *, format="tiff", jpeg_quality=95, queue_depth=64, settings=None)` | Grava uma câmera continuamente até ser interrompida. Retorna um `CliJob`. Os quadros de uma execução do `tiff` são o arquivo `raw`, uma matriz que a captura grava dessa câmera (um plano de mosaico nospróprios valores, sem compressão); `png` e `jpg` contêm 8 bits; portanto, essas execuções são quadros desmosaicados para visualização. Um DAQ vinculado ou agrupado grava um `.daq` por quadro napasta `daq/` da sequência (`daq_files` em suas estatísticas). Cada quadro é gravado uma vez com seus metadados completos (modelo, hora da captura, lente, bandas, calibração em um quadro bruto); o `stop()` retorna assim que todos os quadros já capturados forem gravados (até 25 s), e somente então a câmera é desligada. Uma câmera deixada armada para um gatilho de hardware (`TriggerMode` `On`) é configurada para operação livre durante a execução (a resposta de início, `trigger`, informa o que foi encontrado). Ao parar, a câmera é restaurada ao estado em que foi encontrada — suas configurações de acionamento e todas as configurações alteradas pela execução —, a menos que seja chamado o `keep_settings=True`, que salva as configurações da execução na própria memória da câmera (modo livre), ou o `clear_settings=True`, que reinicializa essa memória. Uma execução que não grava nenhum quadro é encerrada automaticamente após 10 s com o `stop_reason` e o `no_frames`, e o motivo é indicado no `last_error`, e seu `stop()` retorna esse motivo como `error`. Os quadros perdidos porque os gravadores não conseguiram acompanhar são contados, com o `drop_reason` indicando o motivo. Para uma câmera que o Chloros mantém, use `CameraSession.capture_repeated()`. `lattice continuous`. |
| `start_multi_camera_stream(serials=None, output_dir="output", *, fps=3.0, count=None, format="tiff", jpeg_quality=95, settings=None)` | Transmite grupos de quadros sincronizados em `fps`, para grupos `count` ou até ser interrompido. Retorna um `CliJob` (seu `start_reply` contém `n_cams` e o wire `pacing`); suas estatísticas finais contabilizam os grupos capturados, gravados e descartados, os arquivos que o gravador não conseguiu gravar e os grupos para os quais uma câmera não enviou nada. `lattice multi-stream`. |
| `capture_cameras_transient(serials=None, output_dir="output", *, format="tiff", jpeg_quality=95, settings=None)` | Uma captura sincronizada, uma imagem sem bayering por câmera: `results` (`filepath`, `serial`, `frame_id`) e `pacing`. Um grupo sem uma câmera é exibido, indicando primeiro quantos arquivos foram salvos. `lattice multi-capture` sem matriz conectada. |
| `test_array_sync(serials=None, count=10, ptp_settle_s=None, timeout=None)` | `count` capturas sincronizadas com exposição fixa: `sync_groups`, o `master`, uma linha `group_timestamps` por disparo (uma câmera que perdeu um disparo está ausente de sua linha) e `sync`: o deslocamento de cada segunda câmera em relação à mestre, disparo a disparo (`offsets_us`: por câmera `mean_us`, `min_us`, `max_us`, `jitter_us`, `groups`), em UM relógio (`clock_domain`: `ptp` ou `device` — PTP desativado, o próprio relógio de tempo de atividade de cada câmera, de modo que seu deslocamento corresponde ao momento em que ela foi inicializada e apenas o jitter mede a sincronização). Um disparador cujos carimbos misturam o relógio do dispositivo e o PTP (PTP desativado ou ainda em sincronização) é deixado de fora e contado (`groups_mixed_domain`), assim como aquele em que um relógio PTP ainda está a mais de `agree_within_us` do mestre (`groups_clocks_disagree`); `notes` indicam isso. `clock_offset_mean_us` / `clock_offset_jitter_us` representam a maior média e o maior jitter por câmera. `success` é falso, assim como `error`, quando um gatilho foi perdido, quando menos de dois gatilhos têm todas as câmeras no mesmo relógio ou quando os relógios nunca coincidem (`sync.problems`); as linhas são retornadas de qualquer maneira. Ele inicia o grandmaster PTP se este não estiver em execução (`time_sync` indica `grandmaster_running` e `grandmaster_started_by_this_test`), ativa o PTP de todas as câmeras (como faz o `connect_array()`) e, antes do primeiro gatilho, aguarda até que o relógio PTP de todas as câmeras sincronize com o grandmaster e se estabilize: `PtpStatus` Escravo em um mestre, com desvio de até 100 us em relação a ele e estável por 3 s, e cada câmera sincronizada dessa forma, sem interrupção, por 120 s — cerca de 2 minutos após a ativação. Um relógio que acabou de se sincronizar apresenta um desvio de até ~200 us por cerca de 2 minutos após ser lido como “Slave”, e o próprio `PtpOffsetFromMaster` da câmera não mostra isso; 120 s fazem com que as câmeras fiquem a cerca de ~10 us de um relógio estabilizado. Uma câmera que ainda esteja sincronizada desde um teste anterior que deixou o PTP ativado, conforme observado por este backend, conta a partir dessa sincronização, portanto, um teste realizado logo após outro não espera novamente; uma câmera que o teste liga, ou uma encontrada com o PTP ativado que este backend ainda não tenha visto sincronizada (ligada em outro lugar ou antes de uma reinicialização do backend), aguarda seus 120 s, e em um conjunto de ambas as câmeras que acabaram de sincronizar ainda aguardam o tempo necessário. Uma câmera que sai do modo Escravo, ultrapassa 100 us ou segue outro grandmaster por 3 leituras consecutivas (1,5 s) durante a espera, reinicia sua contagem; uma ou duas dessas leituras são registradas (`[array-sync] PTP SN:<serial> off lock at <s> s (<n> of 3): <status> <±x> us gm <id> -- locked <s> s, counting on`) e a contagem continua, e um ID de grandmaster que não pode ser lido é considerado desconhecido, não como uma alteração. O reinício simultâneo da contagem de todas as câmeras (em menos de um segundo) indica que o tempo do “grandmaster” está avançando, e não o das câmeras: o log do backendregistro do backend indica `[array-sync] PTP grandmaster step: all <n> cameras restarted at <s> s …`, o `ptp_lock` lista isso no `grandmaster_steps`, e uma espera que então se esgota inicia seu erro com `every camera's count restarted together at <s> s -- the PTP grandmaster's time moved under them …`. O limite não é estendido para ele. `ptp_settle_s` (0-600; `0` para ausência de espera de estabilização) define o tempo de estabilização para a execução; outro valor é recusado (400 HTTP). A espera é limitada ao tempo de estabilização mais 90 s (210 s por padrão, no mínimo 90 s), e a chamada aguarda o backend `timeout` s: por padrão, 240 mais o tempo de estabilização (360 s), assim como o `test_array_sync` do `chloros-mcp`. O `ptp_lock` possui o `converged`, `waited_s`, `timeout_s`, `settle_s`, `locked_s` (há quanto tempo cada câmera está travada e estável em conjunto), `found_settled` (as câmeras já estavam travadas pelo tempo de estabilização quando foram encontradas) e, por câmera, `status`, `offsets_ns`, `slave_s` (há quanto tempo está sincronizada; Nenhuma: não sincronizada) e `remaining_s` (Nenhuma quando o PTP de nenhuma câmera está ativado). O log do backend contém uma linha a cada 5 s com o estado de cada câmera, há quanto tempo está sincronizada e quanto tempo ainda resta. Todas as câmeras são ligadas com o PTP desativado; a partir daí, os relógios se sincronizam em cerca de 10 s. Depois disso, cada câmera é colocada de volta no estado em que o teste a encontrou — seu PTP, a exposição automática e o ganho, que o teste mantém desativados durante sua execução (`ExposureAuto` / `GainAuto` e um modo manual `ExposureTime` / `Gain`), e o restante, a menos que seja o caso de `keep_settings=True` — ou seja, uma câmera que permaneceu com o PTP ativado (`PtpEnable=True`, ou uma execução com `keep_settings=True`) permanece travada, e uma vez que esse backend a tenha visto travada pelo tempo de estabilização, um teste posterior não espera por ela. Se os relógios não se estabilizarem, `success` é falso antes de qualquer acionamento, e `error` indica o que fazer: uma câmera que nunca travou (`not_slave`) significa que o grandmaster não está conseguindo acessá-la (`time_sync_status()`); uma que ficou caindo é identificada pelo tempo que permaneceu travada (`remaining_s`); relógios ainda convergindo (`far`): verifique `time_sync_status()` e, para mantê-los sincronizados antes do teste, ative o PTP e deixe-o ativado (`PtpEnable=True` em cada câmera, ou `keep_settings=True`; o teste restaura o que encontrou, portanto ele permanece ativado), em seguida, execute-o. `lattice multi-test`. |
| `detect_array_master(line=None)` | As câmeras na linha de sincronização e um mestre padrão: `master_serial`, `slave_serials`, o `line` foi verificado. `lattice multi-detect`. |
| `analyze_array_models(camera_models=None, *, width=2048, height=1536, n_active=None, force_tier=None, binning=1, packed="auto", pixel_format=None)` | Que variedade de modelos dessas câmeras (nomes de produtos, `"LATT-M3C-L87-FOCN"`) que apareceria na rede deste computador: `tier`, `achievable_fps_max`, o orçamento de cabos (`max_cams_collision_safe`, `oversubscribed`), `cam_caps` e `fix_suggestions`. Por padrão, os modelos das câmeras detectadas; nenhuma câmera está aberta. Para as câmeras que você possui, com as configurações que uma conexão usaria, consulte [`analyze_array_network()`](#network-analysis-helper). `lattice analyze-array`. |
| `calibrate_alignment_profile(serials=None, save_path="alignment_profile.json", *, frames=3, method="orb", model="rigid", reference_serial=None, checkerboard=None, control_points=None, preview=False, vignette=False, roi=None, mask_path=None, rms_threshold_px=None, per_cam_overrides=None, settings=None, ...)` | Resolva um perfil de alinhamento a partir de quadros em tempo real e salve-o. `method`: `orb`, `akaze`, `phase`, `checkerboard` (com o `checkerboard="9x6"`, os cantos internos) ou o `manual` (com o `control_points={serial: [[x, y], ...]}`, as câmeras de referência também). `model`: `translation`, `rigid`, `affine` ou `homography`. As opções mais detalhadas são os sinalizadores da CLI: `max_features`, `ratio_threshold`, `matcher` (`bf` / `flann`), `knn_k`, `prefilter` (`none` / `gradient` / `clahe` / `blur` / `hist_match`), `ransac_threshold_px`, `ransac_max_iters`, `ransac_confidence`, `averaging`, `lock_rotation`, `lock_scale`, `lock_axis`; `per_cam_overrides` é igual a `{serial: {option: value}}`. `rms_threshold_px` se recusa a salvar uma solução pior. Retorna `save_path`, `output_width` / `output_height`, `crop_roi`, `reference_serial`, `transforms` (por câmera `reprojection_error`, `num_matches`) e `preview_path`. `lattice align-calibrate`. |
| `capture_aligned_multiband(profile_path, output_dir="output", *, serials=None, format="tiff", bit_depth=None, gpu=None, no_crop=False, bands=None, order=None, vignette=False, per_band=False, per_camera=False, interpolation="linear", border_mode="constant", border_value=0, settings=None)` | Uma imagem multibanda alinhada por meio do perfil: `files` (o arquivo TIFF multibanda com seu `bands` e tamanho, mais um por banda com `per_band` e um por câmera com `per_camera`) e `vignette_corrected`. `bands` seleciona e `order` ordena as bandas por nome (uma lista ou uma sequência separada por vírgulas). `lattice align-apply`. |
| `start_aligned_stream(profile_path, output_dir="output", *, serials=None, fps=3.0, count=None, ...)` | Alinha imagens multibanda em fluxo (as mesmas opções de warping que `capture_aligned_multiband()`), para `count` ou até ser interrompido. Retorna um `CliJob`; suas estatísticas finais incluem o `frames_written` e qualquer `frame_groups_incomplete`. `lattice align-stream`. |
| `alignment_profile_info(path)` | Um arquivo de perfil resumido como `align-info` o imprime: `name`, `created`, `method`, `reference_serial`, `sensor_resolution`, `output_width` / `output_height`, `crop_roi`, `camera_serials`, `transforms` (por câmera `model`, `reprojection_error`, `num_matches`, `reference`), `camera_models`, `layer_order` (por câmera: `band_names`, `wavelengths_nm`, `enabled`); `profile` é o arquivo completo. |
| `reorder_alignment_profile(path, order=None, enable=None, disable=None)` | Altere o arquivo de perfil no local: `order` são nomes de faixas (cada uma move a camada de sua câmera para cima, nessa ordem), `enable` / `disable` são números de série. Retorna o novo `layer_order`. `lattice align-reorder`. |
| `index_catalog()` | As predefinições de índice (`presets`: cada `name`, `channels`, `formula`) e de cor `gradients`. `lattice index --list-presets` / `--list-gradients`. |
| `compute_index(input_path, *, preset=None, formula=None, channel_map=None, output=None, capture_level="unknown", output_format="all", colorize=False, gradient="RdYlGn", vmin=None, vmax=None, percentile=None, bg_mode="clip")` | Um índice sobre um arquivo TIFF multibanda alinhado: um `preset` (`NDVI`, `NDRE`, ...) ou um `formula` sobre os símbolos `channel_map` mapeia para bandas (`{"NIR": "NIR_850", "Red": 0}`, por nome ou índice). Uma predefinição’símbolo `channel_map` omite e seleciona a única banda do arquivo cujo nome começa com ele (`red` -> `Red_660`, `nir` → `NIR_850_RGN`), nomeada pelo próprio arquivo (sua lista de bandas ou o `Chloros:Band{R,G,B}Name` de uma captura ou produto do Chloros), portanto, o NDVI em uma captura de refletância do Chloros não precisa de `channel_map`; duas bandas correspondentes a um símbolo são rejeitadas, não inferidas. `capture_level` indica o que as bandas contêm: `reflectance` ou `radiance` tornam os valores quantitativos; uma pilha não calibrada (`align-apply`) fornece valores qualitativos; `"unknown"` lê isso do arquivo, incluindo o `Chloros:ProcessingLevel` de um arquivo Chloros. Retorna `stats` (forma, o intervalo do 1º ao 99º percentil, o intervalo da LUT, o `capture_level` utilizado) e o `files` gravado ao lado da entrada (`<input stem>_<preset>.tif`, `..._LUT.png`) ou em `output`, um caminho: com uma extensão de imagem, os arquivos são nomeados a partir de seu nome base (`output="D:/out/ndvi_FRGN.png"` gera `ndvi_FRGN.tif` e `ndvi_FRGN_LUT.png`); sem ela, trata-se de um prefixo (`D:/out/ndvi` → `ndvi.tif`, `ndvi_LUT.png`); leia os caminhos a partir de `files`. Eles contêm os dados de registro da câmera da entrada (marca/modelo/número de série, GPS, lente, hora da captura em UTC, a identidade de calibração, mas nunca sua calibração raw-DN), `Chloros:ProcessingLevel=index` e o índice como sua única banda (`XMP-Camera:BandName`, o TIFF e o PNG da LUT da mesma forma). Apenas arquivos; sem câmera. `lattice index --input`. |
| `capture_live_index(profile_path, *, preset=None, formula=None, channel_map=None, serials=None, output_dir="output", output_prefix="index", capture_level="reflectance", save_multiband=False, ...)` | Captura, calibra (`reflectance` ou `radiance`; `raw` ignora essa etapa), alinhar e calcular um índice em uma única etapa; `save_multiband=True` mantém a pilha calibrada, que `compute_index()` pode utilizar posteriormente. Retorna o `stats` (com o `serials` cujas bandas foram utilizadas) e o `files`. Ele abre o `serials` (o primeiro aciona o disparador); caso contrário, as câmeras do perfil cuja entrada `layer_order` esteja habilitada, começando pela de referência (aberta mesmo com suas bandas desativadas); uma câmera cujas bandas estejam desativadas nunca é calibrada. O `channel_map` é necessário apenas para um símbolo ao qual a pilha não consegue responder, como em `compute_index()`: o símbolo de uma predefinição utiliza a única banda ativada cujo nome comece com ele, verificada antes de qualquer câmera ser aberta (se nenhuma ou várias responderem, a resposta é recusada, indicando-as); um `formula` personalizado é necessário para cada símbolo. `index.tif` e a LUT contêm a câmera da pilha, o tempo de captura (a exposição da câmera de referência, UTC) e a lente, e ambas nomeiam o índice como sua única banda (`XMP-Camera:BandName`, por exemplo, `NDVI`: o PNG colorido indica o índice, não seus canais de cor). `lattice index --live`. |

### `CliJob`

O identificador retornado pelos três fluxos. O trabalho é executado no back-end, que mantém o controle das câmeras até ser interrompido.

| Membro | Descrição |
| --- | --- |
| `stats(timeout=10.0)` | `running`, `state`, `stats` (quadros salvos / gravados, descartados, `fps`, `elapsed`, falhas de gravação, `stop_reason`; em uma tarefa contínua, também `frames_grabbed` e `frames_queued` — cada quadro capturado é gravado, colocado na fila, descartado ou apresenta falha — e `drop_reason`, `writers` e `write_ms`, o tempo médio que levou para gravar um quadro) e `output_dir`. `state` é `running`, `ended` (ele parou sozinho e ainda não foi interrompido), `stopping` (uma instância está gravando sua fila: estatísticas em tempo real e `stopped_by`) ou `finished` (suas estatísticas finais, o `error` e o `stopped_by` da interrupção). O registro de um trabalho interrompido é mantido por 10 minutos (no máximo os últimos 32), para cada cliente: outro script ou `chloros-mcp` lê o mesmo estado e as estatísticas finais. Um trabalho `chloros-cli` iniciado (`lattice continuous`, `multi-stream`, `align-stream`) é interrompido pelo backend quando a CLI fica 30 s sem enviar atualizações — encerrado, com a máquina em modo de suspensão — com `stop_reason`, `client_gone` e `stopped_by` no backend; a leitura de suas estatísticas a partir daqui não o mantém ativo. Um trabalho iniciado a partir do SDK é executado até ser interrompido. Cada inicialização envia seu `timeout` como prazo do backend (`client_deadline_s`): uma câmera que é aberta depois disso (uma que ainda está inicializando) é recolocada e fechada, e nenhum trabalho é iniciado, e a chamada gera o `ChlorosConnectError` (`CLIENT_DEADLINE_PASSED`). Uma câmera mantida por um trabalho é recusada a todas as outras abertas (`connect_camera()`, outra inicialização, uma captura transitória) com o código HTTP 409 `CAMERA_HELD`, indicando o nome da tarefa e quem a iniciou. |
| `stop(timeout=30.0)` | Interrompa-a e feche suas câmeras: `final_stats`, `output_dir` e `error` quando a interrupção não ocorreu corretamente. Uma segunda chamada retorna a mesma resposta, assim como uma interrupção iniciada por outro cliente: uma enviada enquanto a primeira ainda está gravando sua fila aguarda a conclusão (por até 25 s; após esse tempo, ela gera uma exceção, e as estatísticas finais são exibidas em `stats()`). |
| `wait(timeout=None, poll_s=0.5)` | Aguarde até que ele termine por conta própria (quando seu `count` for atingido, a unidade estiver cheia ou ocorrer um erro) e, em seguida, execute o `stop()` para coletar as contagens finais. `timeout` gera o `TimeoutError` enquanto ainda está em execução; um trabalho sem `count` só termina quando é interrompido. |
| `job_id`, `kind`, `output_dir`, `start_reply`, `stopped` | Seu ID e tipo (`continuous`, `multi-stream`, `align-stream`), onde ele grava, a resposta da rota inicial e se o `stop()` foi executado. |

Use-o como um gerenciador de contexto para interrompê-lo ao sair do escopo.

---

## Pasta do projeto e receitas

**A pasta do projeto** é a pasta na qual novos projetos são criados, compartilhada pelo aplicativo Chloros, pela CLI e pelo SDK (módulo `chloros_sdk.system_utils`; `chloros-cli project-folder`):

```python
print(chloros_sdk.project_folder())                # the configured folder, else the default
chloros_sdk.set_project_folder("D:/Flights")       # created if missing
chloros_sdk.reset_project_folder()                 # -> {"path": <default>, "was_set": True}
```

Uma pasta configurada que não exista mais recorre à padrão (`Documents/MAPIR/Chloros Projects`), assim como o aplicativo. Um caminho relativo fornecido para `set_project_folder()` é obtido a partir do seu diretório de trabalho quando o backend está nesta máquina.

**Uma receita** é a lista de ações em YAML/JSON que o `chloros-cli project run PATH RECIPE` executa em um projeto salvo. O `run_project_recipe()` a executa com o próprio executador da CLI(instalado com o SDK), de modo que uma receita se comporta da mesma forma em qualquer um dos seguintes:

```python
r = chloros_sdk.run_project_recipe("D:/campaign/morning.yaml", "D:/Projects/Field_A")
print(r["ok"], r["actions"])
print(r["log"])                                    # what the CLI would have printed

chloros_sdk.run_project_recipe({"actions": [{"apply": {"name": "FrontLeft", "settings": {"exposure_us": 5000}}},
                                            {"wait": "2s"},
                                            {"capture": {"name": "FrontLeft", "output": "pose_a/", "format": "tiff"}}]},
                               "D:/Projects/Field_A")
```

```python
run_project_recipe(recipe, project_path, *, dry_run=False, echo=False,
                   backend_url="http://127.0.0.1:5000") -> dict
```

- **`recipe`** é um arquivo `.yaml` / `.yml` / `.json` ou a própria receita: um dicionário com `actions` (`apply`, `capture`, `burst`, `stream`, `sensor`, `wait`) e, opcionalmente, `overrides` e `stop_on_error` (padrão true: a execução é interrompida na primeira ação que falhar). As ações são as descritas na referência da CLI [Recipe DSL](cli-reference.md#recipe-dsl); um `output` relativo é resolvido em relação ao diretório de trabalho do seu script.
- **Retorna** `ok` (todas as ações foram bem-sucedidas), `actions` (quantas), `log` (a saída do executador; o `echo=True` também a exibe durante a execução) e `validation`. Um arquivo de receita ausente ou que não seja analisado corretamente gera o `ChlorosConnectError`.
- **Verificado antes da execução.**`validation` é `{"ok", "errors", "warnings", "steps"}`: cada ação deve ter exatamente um verbo conhecido como chave (`{"capture": {"name": ...}}`, não `{"action": "capture"}` ou `{"type": "capture"}`) que identifique uma câmera, matriz ou sensor do projeto, do tipo correto. Uma receita com erro**não** executa nenhuma ação (`ok` false, os erros em `log`), independentemente do que `stop_on_error` indique. Chaves não utilizadas, e todos os arrays ou sensores de luz controlados pela receita que não estejam conectados no momento, são `warnings`. Uma receita não conecta dispositivos: conecte o primeiro do projeto (`ChlorosProject.connect_all()`, `chloros-cli project connect`, ou a ferramenta MCP `connect_project_devices`). `dry_run=True` não executa nada e retorna `{"recipe": ..., "validation": ...}`.
- Apenas uma receita é executada por vez em um processo. Uma receita YAML requer o PyYAML.

---

## Automação de Projetos — `ChlorosProject`

Um projeto Chloros salvo é uma pasta contendo `cameras.json` + `sensors.json` + `project.json`. `open_project(path, relink=None)` carrega o manifesto (`relink`: a pasta onde agora se encontram os arquivos-fonte ausentes de um projeto movido, usada quando o backend o carrega — `process()` — consulte **Um projeto movido** em `ChlorosLocal`), e o `connect_all` coloca todas as câmeras, matrizes e sensores de luz salvos em operação com suas configurações salvas — o mesmo estado de hardware que a GUI produziria. Os sensores de luz se conectam por meio do [pool de sensores DAQ](#daq-sensor-sessions), portanto, precisam de um backend em execução (consulte [`SensorHandle`](#sensorhandle)).

### Exemplo mínimo

```python
import chloros_sdk

proj = chloros_sdk.open_project("/home/user/Chloros Projects/Field_A")
report = proj.connect_all(verbose=True)
print(report)  # {'cameras': {...}, 'arrays': {...}, 'sensors': {...}}

# Cameras and arrays are addressable by name OR serial / array_id
cam = proj.cameras["FrontLeft"]
cam.capture("./out", format="tiff", processing="reflectance")

arr = proj.arrays["main_rig"]
arr.capture("./out", format="tiff", processing="reflectance")

# Light sensors are addressable by name OR slot_id
reading = proj.sensors["Sky"].read()   # (spectrum, is_saturated, ...) or None

# One frame from every camera and array, one spectrum from every sensor
proj.capture_all("./out")

proj.disconnect_all()
```

Ou como um gerenciador de contexto:

```python
with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    proj.arrays["main_rig"].capture("./out", processing="reflectance")
```

### Métodos do `ChlorosProject`

| Método | Descrição |
| --- | --- |
| `connect_all(*, cameras=True, arrays=True, sensors=True, verbose=False, align=None, packed=None)` | Descobre e conecta todas as câmeras, matrizes e sensores de luz salvos. Retorna um relatório de conexão por classe (`{"cameras": {...}, "arrays": {...}, "sensors": {...}}`, cada um com `connected`, `missing` e `errors`; `arrays` também inclui o `alignment`). Com um backend em escuta no `127.0.0.1:5000`, as câmeras e matrizes salvas são localizadas e conectadas por meio dele. Na ausência de um backend, esse processo as localiza e as controla por conta própria por meio do `lattice_sdk`, que requer o ambiente de execução do SDK da Arena e o OpenCV (`cv2`) — o pacote do SDK não instala nenhum dos dois; quando isso não é possível, todas as câmeras e matrizes salvas são listadas em `errors` com o motivo, e `missing` permanece vazio. Um backend que está em execução, mas recusa o SDK (desconectado: HTTP 401; um plano abaixo do Copper: 403) não é considerado “sem backend”: todas as câmeras, matrizes e sensores de luz salvos são listados em `errors` com o motivo do backend, e nada é aberto nesse processo. Cada chamada verifica o backend novamente; portanto, o `connect_all()`, após o login ou após iniciar o Chloros, se conecta ao mesmo objeto de projeto. Uma matriz salva que já esteja ativa no backend com as mesmas câmeras (da GUI ou do `chloros-cli project connect`) é anexada como está, sem reaplicar suas configurações salvas, e o `disconnect_all()` o mantém em execução. Os sensores de luz sempre passam pelo backend (consulte [`SensorHandle`](#sensorhandle)). Ele nunca inicia um backend. |
| `disconnect_all()` | Desmontar tudo. Todos os dispositivos são liberados primeiro; depois, se o backend não confirmou a liberação de uma câmera, matriz ou sensor de luz (o tempo de espera da desmontagem expirou ou foi recusado), um `ChlorosConnectError` nomeia cada um deles. Um dispositivo já encerrado, ou sem backend em execução, permanece em silêncio. Uma matriz ou um sensor de luz que outro cliente tenha aberto permanece em execução. A saída do projeto `with` nunca é acionada. |
| `capture_all(output_dir=".")` | Um quadro de cada câmera + matriz conectada, e um espectro (`SensorHandle.read()`) de cada sensor de luz conectado, capturados em sequência (câmeras, depois matrizes, depois sensores), não em um único instante. |
| `stream(camera, overlays=False, fps=10.0)` | Gerador que produz quadros BGR `numpy` de uma câmera (ou matriz) nomeada. O `overlays=False` lê quadros com o identificador`grab()` no nível `debayered`: pelo backend quando o projeto está conectado por meio dele; caso contrário, um loop de captura direto `lattice_sdk` (matrizes geram dicionários `{serial: frame}`). `overlays=True` é roteado por meio de `ChlorosLocal.camera_stream()` → o feed MJPEG do backend, com o bloco `ui.overlay` salvo pela câmera sendo passado como parâmetros de consulta. Requer o modo backend, OpenCV (`cv2`) e uma **câmera autônoma**: uma câmera em modo direto gera o erro `RuntimeError` (o backend não consegue acessar uma câmera pertencente a este processo) e uma matriz gera o erro `NotImplementedError` (sobrepõe o composto por câmera — transmite um membro pelo nome). Equivalente de execução única: `CameraHandle.capture(annotated=True)`. |
| `align_arrays(align=True, verbose=False)` | Executar o alinhamento em todas as matrizes conectadas no momento. |
| `process(mode="parallel", wait=True, progress_callback=None, poll_interval=2.0)` | Executar o pipeline de calibração/indexação nas imagens do projeto (envolve `ChlorosLocal.process`; esses quatro são os **únicos** kwargs aceitos — `indices=` etc. gera exceção `TypeError`; define índices via `ChlorosLocal.configure()`). Constrói de forma preguiçosa um `ChlorosLocal()`, que inicia automaticamente um backend. |

Atributos:
- `proj.cameras` — `Dict[str, CameraHandle]` indexado por nome E número de série.
- `proj.arrays` — `Dict[str, ArrayHandle]` indexado por nome E array_id.
- `proj.sensors` — `Dict[str, SensorHandle]` indexado por nome E slot_id.
- `proj.config` — `project.json["config"]` (dicionário).

### `CameraHandle`

```python
cam = proj.cameras["FrontLeft"]

# Save a frame to disk (processing-aware)
filepath = cam.capture(
    output_dir="./out",
    format="tiff",
    processing="radiance",           # see the level table below
    apply_calibration=True,          # DSNU + flat + 3x3 unmix + NIST
    apply_white_balance=True,        # DLS-aware WB
    apply_index=False,
    index_expression=None,
)

# In-memory grab (numpy array)
frame = cam.grab(processing="debayered")
frame, header = cam.grab(processing="radiance", with_metadata=True)

# Frame iterator (generator)
for arr in cam.frame_stream(processing="debayered", fps=5, count=100):
    my_analysis(arr)
```

**Níveis de processamento.** `capture()`, `grab()` e `frame_stream()` utilizam todos o mesmo token `processing`, e a cadeia é cumulativa — cada nível executa tudo o que está acima dele:

| Nível | Saída | Observações |
| --- | --- | --- |
| `raw` | Bayer de 1 canal, nativo do sensor | Sem demosaico. Sobreposições não estão disponíveis neste nível. |
| `debayered` | Cor de 3 canais (**padrão** para `CameraHandle` e todos os `grab()` / `frame_stream()`; `ArrayHandle.capture()`, `capture_smart()` e `stream()` têm como padrão o `raw`): BGR na memória (`grab()` / `frame_stream()`), salvo como RGB pelo `capture()` | No modo backend, como uma captura: o deslocamento de escuro (DSNU) do pacote de calibração aplicado ao raw, seguido do demosaico sensível a bordas do OpenCV — sem LUT de linearidade, campo plano ou desmistura. O único nível que funciona sem o modo backend é o demosaico bilinear simples da biblioteca da câmera (sem desvio de escuro). O arquivo TIFF salvo é RGB fotométrico; portanto, inverta os canais de um quadro na memória (`frame[..., ::-1]`) para compará-lo com um arquivo. |
| `radiance` | float32, W/m²/sr/nm, **um plano por banda** | Cadeia radiométrica completa: demosaico + separação 3×3 (multiespectral) + DSNU + campo plano + escala NIST, com exposição × ganho eliminados para que os valores sejam absolutos. |
| `reflectance` | uint16, 32768 = 1,0 (ou uint8, 255 = 1,0 — consulte `bit_depth`), **um plano por banda** | Radiação dividida pela irradiância descendente (ρ = π·L/E). Requer uma leitura do DLS/DAQ — veja a nota abaixo. |
| `display` | 8 bits, semelhante ao sRGB | Renderização equivalente à GUI: CCM + balanço de branco + gama por meio do perfil de cor ativo da câmera. |

Somente no modo direto (sem backend), o `debayered` está disponível: o `grab()`, em outro nível, ignora
`NotImplementedError`, `capture()` ignora `processing` e `frame_stream()` eleva
`RuntimeError` em todos os níveis. `reflectance` precisa de uma leitura de downwelling utilizável — o ponto final do quadro puxa
o DAQ agrupado para o slot DLS da câmera automaticamente, mas sem um DAQ vinculado, a cadeia recusa a
saída de refletância e registra honestamente o rebaixamento nos metadados retornados, em vez de silenciosamente
devolver um produto de qualidade inferior.

> **Escala DN de refletância — não a codifique de forma rígida.** A refletância do LATTICE usa `32768` = ρ 1,0 e registra
> XMP `Chloros:PixelScale=32768`; a refletância do Survey3 usa `65535` = ρ 1,0 e não contém
> tags `Chloros:*`. Leia a tag e divida por ela. Ela está definida no domínio uint16, portanto permanece
> `32768` para todos os formatos que redimensionam (TIFF de 16 bits, TIFF de 8 bits, PNG/JPG de 8 bits, porcentagem de 32 bits)
> — normalize primeiro o tipo de dados armazenado de volta para uint16 (×257 a partir de 8 bits, ×65535 a partir de float). A única
> exceção é um arquivo de uma compilação anterior à versão 1.3.0: um TIFF de 8 bits era *recortado* naquela época, não redimensionado, portanto
> nenhuma escala o descreve, e ele não contém nenhuma tupla `PixelScale` ou MicaSense. Trate uma tag ausente em um
> arquivo de refletância LATTICE como “nenhuma escala válida”, e não como um padrão.

**Tags de calibração: apenas no `raw`.** O arquivo `raw` de uma captura contém o pacote de calibração da câmera: a carga útil `CalibrationJSON` e as tags compatíveis com MicaSense (`BlackLevel`, `RadiometricCalibration`, `Band{R,G,B}RadiometricCalibration`, `MicaSenseVignetting*`), que descrevem os valores brutos do sensor. Todos os demais arquivos de nível (`debayered`, `display`, `radiance`, `reflectance`, a renderização `index/`) e cada quadro de gravação contínua/fluxo multicâmera contém a identidade da câmera (`XMP-mapir:CameraSerial`, `CameraModel`, `FilterType`, `LensModel`, `CalBundleSha256`, ...) e os próprios dados do quadro, `XMP-mapir:FrameJSON` (exposição, ganho, temperatura, formato de pixel, hora da captura, correspondência DAQ), já que cada produto `process()` contém essa identidade. O processamento localiza a calibração desse arquivo por meio de seu número de série e do `CalBundleSha256` no cache de calibração do Chloros, baixando o pacote uma vez caso este computador não o possua.

#### Arquivos criados a partir de várias câmeras

Um arquivo criado a partir das bandas de mais de uma câmera — uma pilha multibanda (`capture_aligned_multiband()`, `start_aligned_stream()`, `capture_live_index(save_multiband=True)`, `Registration.save_multiband()`), um índice ao vivo gerado a partir de várias câmeras (`index.tif`, `index_LUT.png`), uma imagem de captura em matriz `composite/` — nomeia todas as câmeras que contribuíram para ela:

- **Por-listas por câmera, em ordem de banda** (a composição: camada de fundo, seguida da camada de primeiro plano; uma camada de índice, as câmeras das quais suas bandas provêm): `XMP-mapir:SourceCameraSerials`, `SourceCameraModels`, `SourceSensorProfiles`, `SourceProductLines`, `SourceFilterTypes`, `SourceLensModels`, `SourceCalDateUTCs`, `SourceCalBundleSha256s` (o pacote de calibração pelo qual as faixas de cada câmera passaram; caso contrário — uma pilha `capture_aligned_multiband()` / `start_aligned_stream()`, cujas faixas não passam por nenhum — o pacote da câmera que o Chloros armazenou em cache, aquele cujo nome corresponde ao dos arquivos de captura), `SourceLightboxCertIds`, `SourceHalogenLampCertIds`, `SourcePackTiers` — listas paralelas, em que o item *n* de cada uma descreve a mesma câmera; `-`, onde uma câmera não possui valor (sem pacote de calibração, uma lente desconhecida). Cada câmera é **um** item: quando seu pacote possui vários valores para um campo — os certificados de lâmpadas halógenas de um pacote — eles são unidos com `;` (`F-1479-1;F-1480-1;F-1481-1`), nunca com vírgula; assim, uma lista exibe um item por câmera, independentemente de como for impressa.
- **EXIF `SerialNumber`** contém os mesmos números de série, separados por vírgulas (`214000535,214000551`).
- Ele **não** contém identidade de câmera individual (`XMP-mapir:CameraSerial`, `CalBundleSha256`, …): esses identificam uma câmera e seu conjunto, e o processamento determinaria a calibração do arquivo a partir deles. Um índice calculado a partir das faixas de várias câmeras (`index.tif`, `index_LUT.png`) contém os dados EXIF `Model` `LATTICE` (o conjunto de câmeras), e não o modelo `LATT-…` de uma única câmera.

Um arquivo com as bandas de **uma** câmera — um TIFF por banda, um quadro `capture_aligned_multiband(per_camera=True)`, uma pilha ou um índice em tempo real de uma única câmera — contém as próprias tags de identidade dessa câmera, assim como qualquer outro arquivo de captura. A pilha do `capture_aligned_multiband()`, os arquivos por banda e por câmera, e cada pilha do `start_aligned_stream()` e seus arquivos por banda contêm a hora de captura do grupo (a exposição da câmera de referência, UTC), e um quadro do `per_camera` de uma câmera monocromática é um plano, no `bit_depth`, assim como a pilha.**Orientação da câmera (inversão/rotação).** Os quadros de `grab()` / `frame_stream()` estão sempre na
orientação nativa da câmera, em todos os níveis. As capturas salvas seguem a configuração de orientação
da câmera (as opções Espelhar H / Espelhar V / rotação da interface gráfica):

- **Os arquivos `raw` e `debayered` representam exatamente o que o sensor leu**, independentemente da configuração — um mosaico Bayer
  girado ou invertido seria um padrão de filtro de cor diferente, e os mapas de calibração estão
  na própria grade do sensor.
- **Os arquivos `display`, `radiance`, `reflectance` e os arquivos de índice são orientados** — girados uma vez, após toda a
  cálculos de calibração. Suas máscaras de limitação estão na mesma grade.
- **Cada arquivo de captura registra a configuração** no XMP: `XMP-Camera:ChlorosFlipHorizontal` /
  `ChlorosFlipVertical` (`true`/`false`), `ChlorosRotation` (0 / 90 / 180 / 270, no sentido horário, após
  as inversões) e `ChlorosOrientationApplied` — `false` em imagens brutas e sem debayering, `true` nos produtos orientados. `ChlorosProcessing` lista as operações aplicadas aos pixels desse arquivo (`none` quando
  nativo). Isso inclui todos os quadros de uma gravação contínua (a própria configuração da câmera,
  não aplicada) e de um fluxo multicâmera (a identidade: seus quadros componentes são nativos, como os de uma
  captura em matriz).
- **O processamento de um projeto aplica a orientação registrada** aos produtos que ele gera a partir de um
  (primeiro a calibração nas coordenadas do sensor, depois a distorção de alinhamento da matriz e, por último, a orientação),
  para que correspondam aos produtos da captura ao vivo — incluindo os produtos de fallback com resposta do sensor não calibrada e correção de vinheta;
  seu produto sem debayering permanece nativo.
- **O visualizador de imagens do Chloros mostra um arquivo bruto ou sem debayering girado de acordo com seu registro** — apenas para exibição,
  o arquivo não é alterado; um arquivo sem o registro é exibido tal como está armazenado.
- **Capturas sem o registro são processadas como nativas.** Antes da versão 1.3.0, uma captura com uma única câmera
  convertia ela mesma seus arquivos raw e sem debayering; o Chloros não consegue distinguir um arquivo raw desse tipo de um nativo e
  não faz suposições.
- **As capturas em matriz mantêm todos os arquivos membros na forma nativa por padrão** e registram a identidade;
  O `ArraySession.capture(orient=True)` converte seus produtos (consulte os controles de captura em matriz).

#### Saída por banda e matrizes mistas

Nos níveis `radiance` e `reflectance`, uma câmera retorna **um plano por banda dessa
câmera** — três para uma M3C (suas três bandas não misturadas, ordem dos canais BGR), um para um M3M (seu
único filtro de banda estreita). Os planos replicados da câmera monocromática são agrupados, de modo que uma **matriz mista
M3C/M3M produz formatos de matriz diferentes por série**. Leia a chave `bands` nos metadados do quadro
para saber o que é cada plano; não deduza isso a partir da contagem de canais:

```python
for serial, (arr, hdr) in arr_handle.grab(processing="reflectance",
                                          with_metadata=True).items():
    print(serial, arr.shape, hdr["bands"])   # ('…', (1536, 2048, 3), ['nir','green','red'])
                                             # ('…', (1536, 2048, 1), ['F685'])
    rho = arr.astype("float32") / hdr["pixel_scale"]
```

Sempre verifique `hdr["processing_level"] == "reflectance"` antes de usar os valores. Uma rebaixamento
(sem leitura DAQ, sem pacote de calibração, layout de banda desconhecido) é relatado em `processing_level` e
`radiometric_refused`, em vez de ser sinalizado — a matriz que você recebe ainda é um quadro válido, apenas
não é o produto que você solicitou. Uma captura salva que foi rebaixada contém o mesmo código em seu XMP
que `Chloros:RadiometricRefused` (por exemplo, `no-band-layout`, `no-dls-snapshot`).

#### Um disparador por leitura — `group_id`

**Os membros da matriz são atendidos a partir do grupo de quadros sincronizados da matriz, e não do próprio
buffer de captura de cada câmera.** As câmeras de uma matriz sincronizada por hardware expõem juntas, mas N leituras por câmera
não *amostram* juntas — a leitura do quadro mais recente armazenado no buffer de cada câmera ultrapassa os limites do disparador, e um único quadro perdido dessincroniza o conjunto sem que ninguém perceba.

O fornecimento a partir do grupo é necessário, mas não suficiente por si só: cada solicitação ainda
captura *o mais recente* grupo em seu próprio instante, e o loop de captura pode publicar entre um e outro.
Medido em um conjunto de 4 câmeras ao vivo, **12 de 20 leituras paralelas abrangeram dois grupos** — sempre
exatamente adjacentes. Portanto, a leitura é **fixada**:

1. Cada quadro carrega `group_id`, o contador monotônico de publicações do conjunto.
2. `grab()` lê a câmera mestre (ou, se a mestre não puder ser atendida, a próxima câmera disponível),
   em seguida, fixa as câmeras restantes ao ID do grupo em que foi direcionada (`?group_id=N` no endpoint HTTP).
3. O backend mantém os grupos recentes por **pelo menos meio segundo**, de modo que um irmão já
   ultrapassado por publicações posteriores ainda seja atendido por esse gatilho — tempo suficiente para que a
   exportação de refletância por câmera, mesmo em um conjunto de 9 câmeras com sensores completos. Uma atribuição que o backend não
   mantenha mais é recusada com `409` e `group_aged_out: true`, em vez de ser respondida silenciosamente
   com um gatilho diferente — o `grab()` relê o conjunto.

Remensurado com fixação: **20 de 20 leituras retornaram um único gatilho.** O `grab()` faz isso por você;
você só precisa do `group_id=` ao acessar diretamente o endpoint HTTP.

- Uma câmera que não pode ser atendida no grupo fixado — ausente dele ou recusando a solicitação
  (um `band` que ela não possui, um nível que não pode produzir) — é **omitida em vez de preenchida retroativamente**
  a partir de um gatilho diferente, e as outras câmeras ainda são retornadas.
- O dicionário que o `grab()` retorna é um `GrabResult` (uma subclasse simples de `dict`;
  `from chloros_sdk import GrabResult`): `.errors` mapeia cada
  número de série omitido para a exceção que sua leitura gerou, e `.group_id` é o grupo compartilhado. Um
  `.errors` vazio significa que todas as câmeras compõem esse grupo.
- Uma câmera autônoma não possui grupo, reporta `group_id: None` e **rejeita** um `group_id=`
  pin em vez de ignorá-lo.

```python
frames = arr_handle.grab(processing="reflectance")
for serial, err in frames.errors.items():
    print(f"SN:{serial} omitted: {err}")       # e.g. status=400 ... band 'F685' not in [...]

frames = arr_handle.grab(processing="reflectance", with_metadata=True)
ids = {hdr["group_id"] for _a, hdr in frames.values()}
assert len(ids) == 1, f"straddled triggers: {ids}"     # belt and braces
```

O mesmo contador é publicado em **cada fluxo MJPEG por câmera** de um membro da matriz — a
visualização (`/api/camera/stream/<serial>`), a visualização anotada (`/stream-annotated`) e o
fluxo de refletância (`/stream-reflectance`) — como `X-Chloros-Group-Id` em cada parte, além de
`X-Chloros-Frame-Id` e `X-Chloros-Timestamp-Ns`. Os fluxos são contínuos, e não
pontuais; portanto, são combinados por meio da coleta de partes e do agrupamento com base nesse cabeçalho — não há
pino a ser configurado. Uma câmera autônoma não possui grupo. Consulte
[Fluxos MJPEG por câmera](#per-camera-mjpeg-streams-no-rtsp).

**`frame_stream()` nunca fica em silêncio.** Um tick que falha por um motivo que pode ser resolvido por conta própria
é ignorado — um fluxo de câmera não retorna nada para ele, um fluxo de matriz retorna as câmeras que
responderam (possivelmente nenhuma). Esses motivos são: ainda não há quadro; a thread de captura da câmera foi pausada por um
segundo ou dois (`404 not streaming`, ao qual o backend responde sempre que outra câmera se conecta
ou se desconecta, ou quando uma alteração de configuração reequilibra o pool); uma reinicialização do backend ou `5xx`; o alinhamento
ainda está se estabilizando (`409 … retry shortly`); e uma recusa de refletância porque o DAQ ainda não tem nenhuma leitura
(`no-dls-snapshot`, uma leitura desatualizada ou vazia). Uma recusa no próximo tick apenas se repetiria — um
`band` desconhecido, `bit_depth=8` na radianância, um pacote de calibração sem escala de radianância, uma recusa de refletância
com **nenhum DAQ** (nenhum vinculado à câmera ou à sua matriz, nenhum conectado ao
backend: o `dls_source` da recusa é `"none"`), qualquer outro `4xx` —
gera `chloros_sdk.FrameFetchError` (um `RuntimeError`, contendo `.status` e `.body`) de
uma só vez. O mesmo ocorre com **60 segundos** de ticks pulados sem nenhum quadro entre eles, seja qual for o `fps` — tempo
suficiente para superar a estabilização do alinhamento e uma reinicialização do backend, mas curto o suficiente para que uma câmera que esteja
realmente fora de serviço encerre a transmissão. Esse erro de desistência também é um `FrameFetchError`, tanto para uma câmera quanto para
uma matriz, com a própria exceção da última leitura em `.errors` (e, para uma câmera, como `__cause__`). Para
um array, “recusado” significa que todas as câmeras foram recusadas; a exceção `.errors` contém o
motivo de cada câmera.

**Um array `frame_stream()` gera cada gatilho uma vez** (tanto `ArrayHandle` quanto `ArraySession`). Um
tick cujo conjunto mais recente seja o gatilho que ele já produziu aguarda o próximo gatilho em vez de
produzir o mesmo conjunto novamente; assim, o fluxo é executado na taxa mais lenta entre `fps` e a própria taxa de quadros da matriz, e conjuntos consecutivos nunca compartilham um `group_id`. Uma matriz que para de publicar gera
`FrameFetchError` (“nenhum novo gatilho por 60 s”) em vez de repetir seu último conjunto. Sem isso, um
fluxo que solicitasse mais do que a matriz publicava — ou que fosse iniciado logo após a conexão — geraria
repetições: em uma execução medida, 20 conjuntos continham 11 IDs de grupo distintos.

```python
from chloros_sdk import FrameFetchError

try:
    for arr in cam.frame_stream(processing="radiance", bit_depth=8):
        my_analysis(arr)
except FrameFetchError as exc:
    print(exc.status, exc.body)        # the HTTP status and the backend's reason
```

#### Exportação de uma única banda — `band`

`grab()` / `frame_stream()` utilizam o `band="<name>"` para exportar apenas uma das bandas de uma câmera como um
matriz `(H, W, 1)` de canal único — o mesmo formato que uma câmera monocromática produz; portanto, a saída de uma única banda é
tratada de forma idêntica, independentemente de ter vindo de um M3M ou da divisão de um M3C. A banda vermelha de um FRGN:

```python
red = cam.grab(processing="reflectance", band="red")        # (1536, 2048, 1)
```

As grafias aceitas são o nome completo do layout (`Red_660`, `NIR_850`, `Green_550`) ou seu
nome abreviado (`red`, `nir`, `green`), sem distinção entre maiúsculas e minúsculas. Um nome desconhecido é rejeitado com a lista de
nomes disponíveis.

> **A seleção é feita pelo nome, não pelo número do canal — deliberadamente.** “Canal 1” é `Green_550` na
> ordem dos canais BGR do array, mas *vermelho* na ordem das letras do código do filtro(`RGN`), e os dois
> não coincidem. Escolher o errado resulta em um quadro perfeitamente plausível da banda errada, que
> nada a jusante consegue detectar. Não há índice numérico por esse motivo.

#### Alinhamento dos fluxos — `align`

`align=True` transforma o quadro em seu quadro de referência comum da matriz. Cada câmera é transformada
por meio da **mesma** resolução de perfil e área de interesse (ROI) de recorte, de modo que os fluxos por câmera abertos independentemente
saem **mutuamente registrados** — você não precisa solicitá-los juntos:

```python
frames = arr_handle.grab(processing="reflectance", align=True)   # {serial: array}
# or one MJPEG stream per camera, each co-registered on its own:
#   GET /api/camera/<serial>/stream-reflectance?align=1
```

Requisitos e comportamento:

- A câmera deve estar em uma matriz **sincronizada por hardware** (cabo de sincronização M8) que possua um perfil de alinhamento
  — o co-registro pressupõe que os membros sejam expostos simultaneamente, o que uma matriz de
  matriz em operação livre não oferece. Caso contrário, será recusado com o código `409`.
- **Recusado; nunca fica desalinhado silenciosamente.** O caminho de captura pode salvar imagens desalinhadas com um aviso
  porque um arquivo mantém suas tags de proveniência e pode ser re-registrado posteriormente; uma transmissão ao vivo não tem
  essa segunda chance, portanto, a ausência de um perfil é um erro, e não um recurso alternativo.
- **Solicitar o alinhamento inicia o processo.** Solicitar `align` em um array sem perfil inicia a
  calibração e retorna `409` com o que está aguardando (por exemplo, *"aguardando que a exposição
  se estabilize em 214000551 (1/3)"*) — tente novamente e a operação será bem-sucedida, normalmente em poucas dezenas de segundos.
  Isso é importante no modo sem interface gráfica: o alinhamento automático no momento da conexão desiste após 30 s, e todas as tentativas
  dentro desse intervalo é ignorada enquanto o smart-AE ainda está se estabilizando; assim, uma matriz cuja exposição
  converge lentamente costumava ficar sem perfil para toda a sessão e sem maneira de obter
  um sem abrir a visualização combinada da área de trabalho. O `alignment_status` também relata
  `ae_align_holdouts`, indicando as câmeras que ainda estão se estabilizando e seu progresso.
- Atribua a cada fluxo de uma matriz o **mesmo `max_h`**, ou seus recortes ficarão em grades de pixels
  diferentes.
- O alinhamento é aplicado *antes* da codificação da profundidade de bits, de modo que a distorção interpola valores com precisão total.

#### `bit_depth` — trocando precisão por largura de banda

`grab()` e `frame_stream()` aceitam `bit_depth=16` (padrão) ou `bit_depth=8`:

| `bit_depth` | Codificação de refletância | Custo |
| --- | --- | --- |
| `16` | uint16, 32768 = ρ 1,0, margem até ρ 2,0 | Precisão total. |
| `8` | uint8 abrangendo `0..refl_max` (padrão ρ 1,0) | Metade dos bytes; quantização de ~0,004 ρ no limite superior padrão. Valores acima do limite superior são cortados — veja abaixo. |

##### O limite de 8 bits — `refl_max`

`refl_max` define o ρ que o intervalo de 8 bits abrange, portanto, em `pixel_scale = 255 / refl_max`, ρ ainda é
`pixel / pixel_scale`. `grab()` e `frame_stream()` o consideram como uma palavra-chave tanto na câmera quanto
identificadores de matriz (`?refl_max=` no endpoint HTTP). Aumente esse valor para obter margem para realces especulares,
em passos proporcionalmente mais grosseiros:

```python
frame = cam.grab(processing="reflectance", bit_depth=8, refl_max=1.5)   # 0..1.5 in 0..255
```

| `refl_max` | `pixel_scale` | passo |
| --- | --- | --- |
| `1.0` (padrão) | 255 | 0,0039 ρ |
| `1.5` | 170 | 0,0059 ρ |
| `2.0` | 127,5 | 0,0078 ρ |

> **A exposição automática não consegue corrigir um clipping de 8 bits, e tentar usá-la só piora os dados.**
> ρ = π·L/E com a exposição e o ganho eliminados, portanto ρ é uma propriedade da cena, não da
> configuração da câmera — medido em um array ao vivo, reduzir a exposição pela metade alterou a média de ρ em 1,3%. Aumentar
> a exposição para “ajustar” a faixa, ao contrário, leva os pixels à saturação do **sensor**, o que
> subestima L e distorce ρ para **valores baixos** sem que seja sinalizado nenhum erro (medido: ρ máximo 1,203 → 0,985 com
> exposição 2×, média −23%). Escolha a faixa com `refl_max`; manter o sensor fora da
> saturação é o que o AE já faz, e isso protege a *validade* de ρ, e não sua faixa.

**Leia `reflectance_clipped_fraction`, não o sinalizador.** O `reflectance_clipped_at_1_0` dispara com um
único pixel — em uma cena real, ele relatou `True` enquanto a fração era `0.000001`, um pixel
em um milhão. A fração é o que indica se você deve elevar o limite máximo.

**`bit_depth=8` refere-se apenas à refletância.** Solicitar isso para `radiance` é rejeitado em vez de
atendido: a refletância possui um intervalo fixo de 0 a 1 que se mantém em uma codificação de 8 bits, enquanto a radianância não tem
intervalo fixo; portanto, para encaixá-la em 8 bits, é necessário um alongamento percentual por quadro, cujo mapeamento
muda a cada quadro — trata-se de uma imagem, não de uma medição.

#### Fluxo de refletância ao vivo (MJPEG)

O `GET /api/camera/<serial>/stream-reflectance` fornece refletância de 8 bits por banda como
`multipart/x-mixed-replace` — um M3C como JPEG de 3 canais, um M3M em escala de cinza. Parâmetros de consulta:
`fps` (padrão 10), `jpeg_q` (padrão 95), `max_h` (0 = nativo), `band` (uma banda nomeada, conforme
acima), `align` (`1` para co-registro — mesmas regras que o `align` do SDK), `refl_max` (o limite máximo de codificação de 8 bits,
como acima). Cada parte contém
`X-Chloros-Group-Id`, `X-Chloros-Frame-Id`, `X-Chloros-Timestamp-Ns`, `X-Chloros-Bands`,
`X-Chloros-Pixel-Scale`, `X-Chloros-Aligned` e `X-Chloros-Reflectance-Clipped`.
**Partes de câmeras diferentes que contêm o mesmo `X-Chloros-Group-Id` correspondem ao mesmo gatilho** —
é assim que vários fluxos por câmera de um array são remontados, e isso é garantido
no lado do servidor, pois os membros do array são fornecidos a partir do grupo de quadros sincronizados (consulte
[Um gatilho por leitura](#um-gatilho-por-leitura--group_id)). Ele retorna `409` quando a refletância está
indisponível no momento da conexão e encerra o fluxo se ela se tornar indisponível durante a execução; portanto, um fluxo
paralisado significa “não há mais refletância”, e não “ainda está tudo bem, apenas estático”.

> **O JPEG é com perdas — esses pixels são uma medição.** Erros de compressão afetam diretamente os
> valores de refletância e são mais graves nas bordas, que é onde estão os limites dos materiais. A qualidade
> é padronizada em 95 por esse motivo. Se você precisar que os números permaneçam intactos, use
> `grab(processing="reflectance")` / `frame_stream(...)`, que retornam o mesmo produto sem
> codec no caminho (e `bit_depth=8`, caso o custo da conexão, e não o codec, seja a restrição).

> **EXIF transferido para a exportação.** `process()` copia o bloco GPS da captura de origem
> **e seu ExifIFD** para cada produto, de modo que as exportações contêm `FocalLength`, `FNumber`,
> `ExposureTime`, `ISO`, `DateTimeOriginal` e `CameraSerialNumber`, bem como a
> georreferenciamento. O `FocalLength` é a base a partir da qual o Pix4D calcula a distância entre amostras no solo — sem ele,
> a reconstrução recorre a uma escala totalmente errada (em um caso medido, um local de 411 m
> foi transformado em um de 47,8 km). A cópia não é deliberadamente `-all:all`: as tags estruturais do IFD0 quebram
> a saída do LATTICE, e `ExifImageWidth`/`Height` são excluídos porque descrevem a captura da fonte
> em vez da imagem raster exportada.

Sub-sinalizadores da etapa de captura (aplicáveis aos níveis radiométricos — `radiance`, `reflectance`, `display`):

| Sinalizador | Padrão | Significado |
| --- | --- | --- |
| `apply_calibration` | `True` | DSNU + campo plano + separação 3x3 + escala radiométrica NIST. |
| `apply_white_balance` | `True` | LUT de balanço de branco. Compatível com DLS quando um DAQ está vinculado à câmera. |
| `apply_index` | `False` | Avaliação do índice de vegetação. |
| `index_expression` | `None` | Substituir fórmula. Sevazio → ativa automaticamente o índice. |
| `annotated` | `False` | Sobreposição de decorações da GUI (zebra/grade/pico). Não disponível para `raw`. |

### `ArrayHandle`

```python
arr = proj.arrays["main_rig"]

# Single synced capture group
files = arr.capture("./out", format="tiff", processing="reflectance")
# → {"213800234": "/path/to/x.tif", "214000533": "/path/to/y.tif", ...}

# Multi-level: each serial's value becomes an ordered LIST, not a str
files = arr.capture("./out", processing="all")
# → {"213800234": ["/raw.tif", "/debayered.tif", ...], "combined": "/idx.tif"}

# Smart capture (wait for AE to settle)
result = arr.capture_smart(
    "./out", processing="reflectance",
    settle_timeout_s=5.0,
    stability_window_s=1.5,
    exposure_tolerance_pct=5.0,
)
print(result["frames"], result["settle"])

# In-memory grab: {serial: numpy array}
frames = arr.grab(processing="debayered")
frames = arr.grab(processing="radiance", with_metadata=True)

# Stream-to-disk loop
arr.stream(count=60, output_dir="./stream", fps=5, processing="raw")

# Frame-iterator (tolerates per-cam drops; great for downstream analysis pipelines)
for frames in arr.frame_stream(processing="radiance", fps=5, count=100):
    if "213800234" in frames:
        my_analysis_pipeline(frames["213800234"])

# Preview iterator (live MJPEG-equivalent; tolerates partial cycles)
counts = arr.preview_stream("./preview", fps=3.0, duration=30.0)
print(counts)  # frames written per serial
```

> **O valor de retorno é um `chloros_sdk.CapturePaths`, do tipo `CapturePathMap`, e não `Dict[str, str]`.**
> `chloros_sdk.CapturePathMap` é `Dict[str, Union[str, List[str]]]`: um caminho de nível único
> `processing` fornece a cada série um caminho, enquanto `processing="all"` fornece a
> **lista ordenada** de todos os produtos salvos para essa câmera. `ArrayHandle.capture()` não aceita
> nenhum `levels=` e ignora argumentos-chave que não reconhece (`levels`, `daq`,
> `render_index`, `force_daq`), portanto, passe apenas os que estão mostrados aqui (`aligned=False`
> salva arquivos nativos, como no `ArraySession.capture()`); use
> `ArraySession.capture()` para esses controles. Um composto combinado ao vivo, caso esteja sendo transmitido, chega sob a chave extra
> chave `"combined"`, em vez de sob um serial. Código que pressupõe `str` falha na
> forma de lista sem que nenhum verificador de tipos se oponha — a anotação indicava `Dict[str, str]`
> por algum tempo após o lançamento da forma de lista, e é por isso que o alias existe. Normalize
> quando quiser a forma plana:
>
> ```python
> paths = arr.capture(processing="all")
> flat = [p for v in paths.values()
>         for p in (v if isinstance(v, list) else [v])]
> ```

> **O que não poderia ser escrito: `.failed`.** O mapa que `capture()` retorna é um
> `chloros_sdk.CapturePaths` — uma subclasse simples de `dict`, portanto tudo acima funciona
> sem alterações — que também contém `.failed`: os arquivos que o backend não conseguiu gravar,
> as mesmas entradas que [`CaptureResult.failed`](#arquivos-que-não-puderam-ser-gravados--falha-na-captura)
> (`serial`, `level`, `error` e `disk_full: True` em um disco cheio). Uma captura parcial
> ainda retorna todos os caminhos que foram gravados; em um disco cheio, o mapa fica
> vazio e o `.failed` explica o motivo. `.skipped` lista os saltos por câmera e
> `.group_seq` o grupo sincronizado, e `.alignment` / `.oriented` são os
> do backend, assim como em `CaptureResult`. `capture_smart()` os retorna como
> `result["frames"].failed` e `result["failed"]`.
>
> ```python
> paths = arr.capture("./out", processing="all")
> if any(f.get("disk_full") for f in paths.failed):
>     raise SystemExit("drive full -- free space and capture again")
> ```

### Alinhamento de matrizes

`ArrayHandle` expõe toda a superfície de alinhamento. Por padrão, os perfis são válidos apenas para a sessão — chame `export_alignment()` explicitamente para torná-los persistentes.

```python
from chloros_sdk import AlignmentSpec

arr = proj.arrays["main_rig"]

# Defaults: ORB / affine / one synced snapshot — same as the GUI's auto-cal
result = arr.calibrate_alignment()
print(result["profile"]["rms_residual_px"])

# Custom spec for tough scenes (low-contrast canopy)
spec = AlignmentSpec(
    method="feature_orb",         # feature_orb / feature_akaze / phase_correlation / checkerboard / manual
    model="rigid",                # translation / rigid / affine / homography
    num_frames=5,
    max_features=8000,
    ratio_threshold=0.7,
    ransac_threshold_px=2.0,
    min_matches=30,
    max_reproj_err_px=2.0,
)
arr.calibrate_alignment(spec)

# Or tweak one knob at a time
arr.calibrate_alignment(num_frames=3, model="affine")

# Manual control points: each camera's pixel coordinates of the same
# features, the reference camera's included (>= 2 per camera). Every one
# of num_frames is solved from these points.
arr.calibrate_alignment(method="manual", model="rigid", control_points={
    "214701173": [[412, 300], [1630, 310], [1020, 1180]],   # reference (master)
    "214701292": [[418, 297], [1637, 306], [1027, 1177]],
})

# Inspect / manipulate
status = arr.alignment_status()
arr.tweak_alignment("214701292", dx=2.5, dy=-1.0, rotation_deg=0.0, scale=1.0)
arr.export_alignment("/tmp/main_rig_alignment.json")
arr.import_alignment("/tmp/main_rig_alignment.json", validate=True)
arr.clear_alignment()
```

#### Alinhamento no momento da conexão

O `connect_all(align=...)` pode alinhar automaticamente todas as matrizes no momento da conexão:

```python
# Align every array with defaults
proj.connect_all(align=True)

# Per-array control
proj.connect_all(align={
    "main_rig": AlignmentSpec(num_frames=5, model="affine"),
    "side_rig": True,             # use defaults
    "verify_rig": False,          # skip
})
```

Recorre ao `project.json["config"]["auto_align_on_connect"]` quando não especificado.

### `SensorHandle`

Um sensor de luz salvo (DAQ-U, DAQ-M ou DAQ-E), aberto por `connect_all()` no [pool de sensores DAQ](#daq-sensor-sessions) — o mesmo pool usado por `connect_daq_sensor()` e `chloros-cli daq pool-connect`.

```python
reading = proj.sensors["Sky"].read()
if reading is not None:
    spectrum, is_saturated, integration_time, x, y, z = reading
```

- `read()` aguarda o **próximo** espectro do sensor — um já armazenado em buffer no momento da chamada não conta — e retorna `(spectrum, is_saturated, integration_time, x, y, z)`, ou `None` se nenhum chegar em até 5 s. Um fluxo pausado é retomado. `spectrum` está em W/m²/nm quando o sensor está calibrado.
- `.sensor` é o [`DAQSensorSession`](#sessões do sensor de aquisição de dados); para todos os outros casos: `latest(n=...)`, `status()`, `record_start()` / `record_stop()`.
- `.slot_id`, `.model`, `.serial`, `.name` e `.settings` são as entradas salvas a partir de `sensors.json`.

**O que o `connect_all` restaura.** Cada sensor é aberto em seu endereço salvo — sua porta COM, endereço Bluetooth ou host DAQ-E — e, quando o projeto registrou o ID do sensor, deve ser esse sensor: uma unidade diferente nesse endereço é rejeitada, não conectada. Seu tempo de integração salvo, média de quadros e exposição automática são aplicados, assim como sua correção de tampa:

| Cap salvado | Conectado com |
| --- | --- |
| Um perfil de cap (por exemplo, `fov_30`) | Esse perfil. |
| Nenhuma escolhida | `sunshine_cosine`, o padrão de conexão, registrado conforme presumido (`cap_id_source=auto_default`), portanto o processamento emite um aviso. Os sensores são fornecidos sem capa: escolha a capa que estiver instalada. |
| `none` — selecionado como **Nenhum (sensor sem tampa)** na interface gráfica | Sem correção de tampa. |

Um DAQ-E-S não aceita nenhum perfil de tampa, independentemente do que tenha sido salvo: seu difusor faz parte de sua calibração (consulte [DAQ-E-S](#daq-e-s--fixed-diffuser-and-tilt-sensor)). Um projeto salvo por um Chloros anterior registrou um sensor sem tampa como opção padrão; portanto, ele se reconecta com o `sunshine_cosine`; escolha **Nenhum (sensor sem tampa)** na interface gráfica mais uma vez e salve o projeto.

Um sensor que outro cliente já tenha aberto no pool (CLI, outro script) é conectado como está, sem reaplicar configurações, e o `disconnect_all()` o mantém em execução; um sensor aberto pelo `connect_all()` é liberado. Assim como no `connect_daq_sensor()`, a GUI do Chloros **não** faz parte do pool: feche um sensor na GUI antes de conectar o projeto a partir de um script.**Requisitos e o relatório.** Os sensores de luz precisam de um backend do Chloros em execução, conectado a um plano pago do Chloros+ (Copper ou superior), e o `connect_all()` nunca inicia um. Sem nenhum em execução, todos os sensores salvos são listados no `report["sensors"]["errors"]`, indicando isso. Um sensor que o backend não consegue abrir também é listado ali, com o motivo indicado pelo backend; o `report["sensors"]["missing"]` permanece vazio para sensores de luz.

---

## Hardware direto (sem backend)

Quando você quiser zerar a dependência do backend para câmeras LATTICE (CI, robôs sem cabeçote, embarcadas), importe `lattice_sdk` diretamente — ele é reexportado por `chloros_sdk`. Atenção ao usar o `CAMERA_AVAILABLE`: O `lattice_sdk` está no wheel do SDK, mas requer a presença do runtime do Arena SDK e do OpenCV (`cv2`), e o wheel não instala nenhum dos dois. Os sensores de luz passam pelo backend — [`connect_daq_sensor()`](#daq-sensor-sessions) ou pelo `proj.sensors` de um projeto — com uma exceção: o auxiliar de magnitude de banda do DAQ-M [`DLS`](#daq-m-band-magnitudes--dlsband_magnitudes), que também requer o `bleak`.

```python
from chloros_sdk import (
    # cameras
    LatticeCamera, CameraSettings, PRESETS, CameraPool,
    Calibration, CalibrationCoefficients, FilterModel, list_filters,
    DLS, NetworkDiagnostics, gpu_info, gpu_available,
    # discovery
    discover_cameras, discover_cameras_via_backend,
    # exceptions
    LatticeError, CameraNotFoundError, StreamError, CaptureError,
    CalibrationError, NetworkError, DLSError,
)

# Find a camera and capture in one go
cams = discover_cameras(timeout_ms=3000)
print(cams)

settings = PRESETS["high_quality"]
with LatticeCamera(serial="213800234", settings=settings) as cam:
    result = cam.capture(output_dir="./out", format="tiff")
    print(result.filepath, result.width, result.height)
```

##### Predefinições e o gatilho

Três das quatro predefinições são do tipo **free-run**: a câmera expõe continuamente e um
`capture()` retorna o próximo quadro. O `triggered` é a exceção — ela arma a
câmera para uma flutuação de hardware na Linha 2, portanto não captura nada até que ocorra uma.

| Predefinição | Gatilho | Use quando |
| --- | --- | --- |
| `default` | free-run | uso geral |
| `high_speed` | free-run | 8 bits, limite de 60 fps, exposição curta |
| `high_quality` | free-run | 12 bits, sem limite de fps — a escolha usual para fotos |
| `triggered` | **armada, Linha 2** | a câmera está conectada a um cabo de sincronização M8 e é acionada por outro dispositivo |

Se você escolher `triggered` (ou configurar `trigger_mode="On"` por conta própria) sem nada
acionando a Linha 2, todos os `capture()` entrarão em timeout — corretamente, já que você solicitou
que a câmera aguardasse. O SDK explica isso quando isso ocorre; consulte
[SC_ERR_TIMEOUT durante a captura](#direct-hardware-backend-free).

> **Observação — as mensagens “GVSP probe” / `SC_ERR_TIMEOUT -1011` na conexão não são erros.**
> Na conexão, o SDK tenta negociar**quadros jumbo** (pacotes GVSP de 9.000 bytes) para obter maior taxa de transferência. Em um link direto ponto a ponto de placa de rede (por exemplo, um endereço `169.254.x.x` local do link), a rede geralmente não consegue transportar quadros jumbo; portanto, essa sonda expira e registra linhas como:
>
> ```
> [Network] GVSP probe: unexpected error (TimeoutError: ... SC_ERR_TIMEOUT -1011)
> [Network] GVSP probe at 9000 did not deliver a complete buffer; reverting to ICMP-chosen size
> [Network] GVSP packet size: 1500 bytes (standard)
> ```
>
> Esse é o **plano de contingência previsto**: o SDK reverte automaticamente para pacotes padrão de 1.500 bytes e a câmera continua se conectando normalmente (as linhas `[chunk-enable …]` que se seguem fazem parte da sequência normal de conexão). A captura continua funcionando.
>
> Você pode ignorar essa sonda, mas **ela não serve apenas para silenciar os registros — ela desativa os quadros jumbo.** A câmera responde a pings “Don't-Fragment” apenas até 1.500 bytes, independentemente da qualidade da sua rede; portanto, o teste de ping por si só nunca consegue detectar quadros jumbo; este teste é a única forma de fazê-lo. Desative-o e a câmera utilizará pacotes padrão de 1.500 bytes para sempre, em qualquer rede:
>
> ```bash
> CHLOROS_GVSP_PROBE_FALLBACK=0   # gives up jumbo — see the warning it prints
> ```
>
> Só vale a pena em uma rede que você *saiba* que não suporta quadros jumbo, onde isso economiza cerca de um segundo no tempo de conexão por câmera. Como se trata de uma troca real, e não apenas cosmética, o SDK agora avisa quando você usa essa opção:
>
> ```
> [Network] ⚠️ GVSP probe disabled (CHLOROS_GVSP_PROBE_FALLBACK=0) — staying at
> 1500 bytes, jumbo NOT tested. … if this network does carry it, you are giving
> up ~1.45x wire ceiling. Unset the variable to test for jumbo.
> ```
>
> **Não altere essa configuração, a menos que tenha um motivo.**Se permanecer ativado, cada câmera é medida uma vez por sessão de backend e novamente quando seu link muda (outro adaptador, outra velocidade de link ou um link que caiu, como acontece ao trocar um switch); as reconexões reutilizam o resultado.**Após uma alteração atrás de um switch que permaneceu ativo, reinicie o backend para medir novamente.** A primeira conexão pode, ocasionalmente, levar alguns minutos para essa verificação, e o log indica isso quando ocorre.
>
> Se você *quiser* a taxa de transferência jumbo, habilite o jumbo de ponta a ponta (MTU da placa de rede de 9.000 + um switch que os transmita), ou fixe-o com `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000` quando souber que o link suporta isso — embora seja preferível usar o comando `CHLOROS_GVSP_PACKET_SIZE_FORCE=9000 python …` por comando em vez de configurá-lo permanentemente, já que um tamanho fixado ignora o teste de detecção e interrompe a adaptação à rede à sua frente. **Todos** os dispositivos no caminho precisam transmitir pacotes jumbo — incluindo qualquer divisor ou injetor PoE, que é o motivo comum pelo qual uma configuração que, de outra forma, seria compatível com pacotes jumbo, não consegue transmiti-los.

> **`SC_ERR_TIMEOUT -1011` durante `capture()` / `grab*()` é um problema diferente — esse é um erro real.**
> A observação acima refere-se apenas ao `-1011` registrado pela**sonda de tempo de conexão**. O mesmo erro gerado a partir de uma**captura** significa que a câmera se conectou corretamente, mas não está enviando imagens:
>
> ```
> File ".../lattice_sdk/camera.py", line ..., in grab_frame_with_metadata
>   buffer = self._get_buffer(timeout)
> lattice_sdk.exceptions.CaptureError: Capture failed: ... SC_ERR_TIMEOUT -1011
> ```
>
> O indício revelador é uma câmera cujo canal de *controle* está funcionando corretamente — a detecção funciona, as configurações e as gravações do `[chunk-enable …]` são bem-sucedidas — enquanto *todos* os quadros atingem o tempo limite.
>
> **A causa mais comum é que a câmera está configurada para um disparador de hardware.** Com os códigos `trigger_mode="On"` e `trigger_source="Line2"`, a câmera não emite absolutamente nada até que uma flutuação elétrica chegue ao cabo de sincronização M8. Se não houver nenhum cabo acionando essa linha, cada captura fica esperando indefinidamente. A câmera não está com defeito e a rede está funcionando corretamente — ela está fazendo exatamente o que lhe foi solicitado.
>
> `CameraSettings()` e os modelos `default` / `high_speed` / `high_quality` são predefinições de funcionamento livre, e uma captura que atinge o tempo limite enquanto está armada exibe uma mensagem explicativa em vez de imprimir apenas um `-1011`. O `PRESETS["triggered"]` arma a Linha 2, conforme projetado.
>
> Para forçar qualquer câmera a operar em modo livre:
>
> ```python
> import copy
> settings = copy.deepcopy(PRESETS["high_quality"])   # PRESETS entries are shared
> settings.trigger_mode = "Off"        # free-run; don't wait for an M8 edge
> ```
>
> Se ainda assim ocorrer tempo limite com o comando `trigger_mode="Off"`, a câmera realmente não está enviando dados — envie-nos o log e o comando `ip link show`.

#### Perfis de cor (visualização ao vivo em RGB) — `set_color_profile`

`LatticeCamera.set_color_profile(profile, custom_cct_k=None)` seleciona o perfil de cor da tela para a **visualização ao vivo** em câmeras RGB (câmeras multiespectrais ignoram essa configuração):

| Perfil | Significado |
| --- | --- |
| `raw` | Ignora completamente a cadeia radiométrica. |
| `linear` | DSNU + flat + WB, sem CCM, sem gama. |
| `natural` | Linear + CCM medido + gama sRGB, apenas com o acabamento básico (suavização de crominância + dessaturação de realces) — a configuração padrão realista. |
| `enhanced` | `natural` mais o acabamento completo com paridade de hub (remoção de franjas, vibração, contraste local CLAHE). Aparência mais rica com aproximadamente **o dobro do custo de acabamento por quadro**, portanto, uma taxa de quadros LIVE mais baixa. |
| `custom_temp` | `natural`, mas com o equilíbrio de branco fixado em `custom_cct_k` Kelvin (DLS ignorado; limitado a 2.000–10.000 K no backend). |

O perfil é um controle de velocidade/aparência **exclusivo para visualização ao vivo**: as capturas salvas sempre recebem o acabamento rico completo, independentemente do perfil selecionado; portanto, escolher `natural` para recuperar tempo de quadro não reduz a qualidade do que é gravado no disco. Um perfil desconhecido aciona o `ValueError`; quando um backend chloros está acessível, a alteração também é enviada por POST para ele, de modo que o próximo quadro de pré-visualização reflita essa mudança (usuários diretos do SDK sem um backend ainda recebem a alteração nas configurações).

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_color_profile("enhanced")            # richer look, lower LIVE fps
    cam.set_color_profile("custom_temp", custom_cct_k=5600)
```

#### Destaques (tela RGB) — `set_highlights`

`LatticeCamera.set_highlights(value)` reduz os pixels mais brilhantes e mais brancos da **tela** em câmeras RGB — a correção para uma superfície branca brilhante que é renderizada como uma mancha branca sem detalhes. `value` é `-100`..`0` (limitado); o `0` está desativado, e a renderização fica exatamente como seria sem a configuração. Configurações mais intensas reduzem o branco da tela (para cerca de 60% no `-100`) e ampliam a curva de tom, de modo que uma superfície que o brilho automático da pré-visualização levou ao branco puro recupera seus detalhes.

Trata-se de um controle de exibição, mas, ao contrário da saturação e do contraste (que alteram apenas a pré-visualização ao vivo), ele também altera o que você salva: a pré-visualização ao vivo e as capturas **no nível da tela** seguem essa configuração, de modo que esses arquivos também ficam mais escuros (eles contêm `Chloros:DisplayHighlights`); as exportações de radiação e refletância nunca o fazem, e ele não consegue recuperar dados do sensor que sofreram clipping — reduza a exposição para isso. Câmeras multiespectrais e monocromáticas ignoram essa configuração. O valor é armazenado em `settings.highlights` e enviado via POST para um backend chloros acessível, exatamente como `set_color_profile`; o equivalente na CLI é `lattice highlights --set N`.

```python
with LatticeCamera(serial="214701292") as cam:   # RGB cam
    cam.set_highlights(-60)                      # 0 = off, -100 = strongest
```

#### Câmeras monocromáticas (M3M) e `Calibration`

Uma câmera monocromática **M3M** (`M3M-<lens>-F<wavelength>`) é de banda única: um plano em escala de cinza, sem mosaico de Bayer, sem matriz de interferência espectral 3×3. O `Calibration` a reconhece e expõe um sinalizador `is_mono`. A refletância ainda se aplica como um mapa radiométrico por banda (a desmistura é a matriz identidade), mas os cálculos multibanda em uma única câmera geram resultados válidos em vez de retornarem valores sem sentido:

```python
from chloros_sdk import Calibration, CalibrationError

calib = Calibration("M3M-L87-F685")
print(calib.is_mono)        # True  (False for any M3C / RGN Bayer cam)
print(calib.filter_type)    # 'mono'  (sentinel; not a real crosstalk key)

# NDVI needs two bands (Red + NIR); one mono band can't supply both.
try:
    calib.compute_ndvi(reflectance_frame)
except CalibrationError as e:
    print(e)   # "...single-band mono (M3M) camera. Combine multiple..."
```

Para construir um índice de vegetação a partir de hardware monocromático, combine várias câmeras M3M em diferentes comprimentos de onda em uma pilha multibanda alinhada (consulte [Alinhamento de Matriz](#array-alignment)) e calcule o índice nessa pilha, em vez de em uma única câmera.

#### Magnitudes de banda do DAQ-M — `DLS.band_magnitudes`

O `DLS(mac_address, filter_model="RGN-IMX265")` se conecta a um DAQ-M via Bluetooth por meio de seu próprio MAC BLE. É a única classe de sensor de luz que se comunica diretamente com o sensor, em vez de por meio do backend; portanto, ela precisa do que as classes de câmera direta exigem (`CAMERA_AVAILABLE`: o runtime do SDK da Arena e o OpenCV), além do `bleak` para Bluetooth (`pip install "chloros-sdk[camera]"` ou `pip install bleak`; sem ele, o `DLS` gera o `DLSError`). Um DAQ-M conectado em outro lugar (no pool do backend, em um projeto, na GUI) para de anunciar; portanto, libere-o primeiro nesse local. **`mac_address` é obrigatório**: não há valor padrão, já que cada unidade tem o seu próprio, e `DLS()`, sem um, gera o erro `DLSError` antes que qualquer conexão seja tentada. Encontre o MAC com `discover_daq_sensors(transports=["ble"])` (consulte [Descoberta](#discovery--finding-an-address-to-connect-with)) ou `chloros-cli daq pool-discover --only ble`.

`DLS.band_magnitudes(spectrum=None, *, spectral_response=None)` integra um espectro DAQ-M (a leitura mais recente quando `spectrum` é `None`; `DLSError`, caso ainda não haja nenhuma) por meio das respostas dos três canais de uma câmera e retorna `{"red": …, "green": …, "nir": …}` — as chaves nomeiam os canais nessa ordem; com as curvas de uma câmera RGB, o terceiro (`nir`) é o canal azul.

- Sem `spectral_response`, ele usa a curva padrão do modelo de filtro (a `filter_model` com a qual o `DLS` foi construído), que foi medida em uma unidade.
- O `spectral_response` utiliza, em vez disso, as **próprias** curvas medidas da câmera: um dicionário `{"wavelengths": [...], "R": [...], "G": [...], "B": [...]}`, conforme armazenado no pacote de calibração dessa câmera (`color_correction.spectral_response_smoothed` — câmeras RGB; pacotes multiespectrais não contêm tal curva). Se não for possível usar esse dicionário, ele recorre à curva da frota.

```python
from chloros_sdk import DLS

# camera_curves: {"wavelengths": [...], "R": [...], "G": [...], "B": [...]}
with DLS(mac_address="AA:BB:CC:DD:EE:FF") as dls:
    reading = dls.acquire()
    fleet = dls.band_magnitudes(reading)                         # fleet curve
    own = dls.band_magnitudes(reading,
                              spectral_response=camera_curves)   # the camera's own
```

O `chloros-cli lattice dls --camera SERIAL` da CLI faz o mesmo por meio do backend, localizando a curva da câmera em seu pacote de calibração carregado ou armazenado em cache, e informa qual curva foi utilizada. Por HTTP, isso corresponde a `POST /api/dls/cli-read` com `mac` (obrigatório: sem ele, a resposta é HTTP 400, `code` `MAC_REQUIRED`) e `camera_serial` no corpo; o `band_magnitudes_source` da resposta é `camera <serial>` quando a curva própria da câmera foi usada, ou `model <filter>` para a curva da frota (uma câmera sem curva: não é RGB ou nenhum pacote carregado ou armazenado em cache).

---

## Exceções

Intercepte a classe base para lidar com “qualquer coisa que deu errado no Chloros”:

```python
import chloros_sdk

try:
    chloros_sdk.process_folder("/path/to/folder")
except chloros_sdk.ChlorosAuthenticationError:
    print("Run `chloros-cli login` first.")
except chloros_sdk.ChlorosLicenseError:
    print("Chloros+ subscription required.")
except chloros_sdk.ChlorosError as e:
    print(f"Chloros error: {e}")
```

> `ChlorosAuthenticationError` e `ChlorosConfigurationError` são atributos de `chloros_sdk` (e de `chloros_sdk.exceptions`), mas não em `__all__`; portanto, `from chloros_sdk import *` não os inclui: importe-os pelo nome. A família `ChlorosError` é derivada de `ChlorosLocal` e das funções construídas sobre ela (`process_folder()`, `process_lattice_capture()`); a superfície smart-connect herda de `ChlorosConnectError`.

Hierarquia:

```

ChlorosError
├── ChlorosBackendError           (backend binary not found / failed to start)
├── ChlorosConnectionError        (HTTP transport failure; no backend running
│                                  with auto_start_backend=False)
├── ChlorosLicenseError           (403 PLAN_UPGRADE_REQUIRED: plan below Copper)
├── ChlorosAuthenticationError    (401 AUTH_REQUIRED: not logged in)
├── ChlorosConfigurationError     (bad configure() inputs)
└── ChlorosProcessingError        (pipeline failed; create/import refused)
    └── ChlorosProjectFilesMissingError  (open_project(): none of the
                                  project's source files found, even after
                                  the relink; .missing_roots, .audit)

ChlorosConnectError                (the smart-connect surface: connect_*, attach_array,
                                    list_*, discover_*, every CameraSession /
                                    ArraySession / DAQSensorSession method,
                                    RecorderHandle, ContinuousHandle,
                                    disconnect_all(), and analyze_array_network()
                                    when no backend answers; 401 and 403 included.
                                    Derives from plain Exception, NOT from
                                    ChlorosError, so `except ChlorosError` will
                                    not catch it. .warning: a warning the
                                    backend sent beside its error (also at the
                                    end of the message); .recording_closed: the
                                    recording a failed DAQ-E restart closed;
                                    both None otherwise)

FrameFetchError (RuntimeError)     (grab() / frame_stream() refusals; .status,
                                    .body, .errors)
ValueError                         (capture() arguments: PNG/JPEG with a level they
                                    cannot hold; a bad packed=)
FileNotFoundError / ImportError    (open_project(): no such folder / no project
                                    loaders)
RuntimeError                       (CameraHandle / ArrayHandle: a backend refusal)

lattice_sdk exceptions:
LatticeError
├── CameraNotFoundError
├── CameraConnectionError
├── StreamError
├── CaptureError
├── CalibrationError
├── NetworkError
└── DLSError
```

---

## Exemplos de ponta a ponta

### 1. Processar uma pasta com uma barra de progresso personalizada

```python
from chloros_sdk import ChlorosLocal

def progress(percent, message):
    bar = "#" * (percent // 5)
    print(f"\r[{bar:<20s}] {percent:3d}% {message}", end="", flush=True)

with ChlorosLocal() as cl:
    cl.create_project("FieldA_2026-05-26")
    cl.import_images("C:/DroneImages/Flight001", recursive=True)
    cl.configure(
        debayer="High Quality (Faster)",
        vignette_correction=True,
        reflectance_calibration=True,
        indices=["NDVI", "GNDVI", "OSAVI", "SAVI"],
        export_format="TIFF (16-bit)",
    )
    cl.process(progress_callback=progress)
print()
```

### 2. Matriz LATTICE em tempo real → Refletância + Referência DAQ

```python
import chloros_sdk

# Open a paired sensor first so the array's reflectance step has an
# absolute reference. Smart-detect picks USB / BLE / ETH automatically.
with chloros_sdk.connect_daq_sensor() as daq:
    with chloros_sdk.connect_array([
            "213800234", "214000533", "214701288", "214701292"
    ]) as arr:
        # Smart capture: wait for AE to settle, then snap
        arr.capture("./out", processing="reflectance", smart=True)

        # Record the corresponding DAQ frames as ground truth
        daq.record_start(output_dir="./out", device_name="sky-reference")
        # ... do whatever capture campaign ...
        info = daq.record_stop()
        print(info["path"], info["rows"])
```

### 3. Campanha de captura orientada a projeto

```python
import time, chloros_sdk

with chloros_sdk.open_project("/home/user/Chloros Projects/Field_A") as proj:
    report = proj.connect_all(verbose=True, align=True)
    if report["arrays"]["errors"]:
        raise SystemExit(f"Array(s) failed to connect: {report['arrays']['errors']}")

    rig = proj.arrays["main_rig"]

    # Re-align right before the campaign
    rig.calibrate_alignment(num_frames=5)
    rig.export_alignment("./alignments/main_rig.json")

    # 50 sequential single-frame captures at 2 fps
    for i in range(50):
        frames = rig.capture(
            output_dir=f"./out/frame_{i:04d}",
            processing="reflectance",
            apply_calibration=True,
            apply_white_balance=True,
        )
        time.sleep(0.5)

    # End-of-day: process the captured folder. process() accepts only
    # mode/wait/progress_callback/poll_interval — indices come from the
    # project's saved config (or set them via ChlorosLocal.configure()).
    proj.process()
```

### 4. Fluxo de quadros de múltiplas câmeras → Pipeline NumPy

```python
import chloros_sdk
import numpy as np

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    rig = proj.arrays["main_rig"]

    for frames in rig.frame_stream(
            processing="radiance",
            fps=5.0, count=300,
            apply_calibration=True,
            apply_white_balance=True):
        # frames is {serial: numpy_array}; cams not delivering this tick are omitted
        for serial, frame in frames.items():
            print(serial, frame.shape, frame.dtype, frame.mean())
```

### 5. Script de captura direta no hardware (sem backend) sem interface gráfica

```python
from chloros_sdk import LatticeCamera, PRESETS, discover_cameras

cams = discover_cameras(timeout_ms=3000)
print(f"Found {len(cams)} cams")

settings = PRESETS["high_quality"]
for c in cams:
    with LatticeCamera(serial=c.serial, settings=settings) as cam:
        result = cam.capture(output_dir="./out", format="tiff")
        print(c.serial, result.filepath)
```

### 6. Verificação de capacidade antes de conectar uma matriz de 4 câmeras

```python
import chloros_sdk

serials = ["214701288", "213800234", "214000533", "214701162"]

probe = chloros_sdk.analyze_array_network(
    master_serial=serials[0],
    slave_serials=serials[1:],
    width=2048, height=1536,
    pixel_format="BayerRG12",
)

if probe["status"] == "ok":
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12")
elif probe["status"] == "auto_capped_fps":
    r = probe["recommended"]
    print(f"Keeping resolution; capping trigger rate at "
          f"{r['recommended_target_fps']} fps")
    arr = chloros_sdk.connect_array(
        serials, width=2048, height=1536, pixel_format="BayerRG12",
        target_fps=r["recommended_target_fps"])
elif probe["status"] == "auto_shrunk":
    r = probe["recommended"]
    print(f"Auto-shrinking to {r['out_width']}x{r['out_height']} "
          f"binning={r['binning']} for sim-sync")
    arr = chloros_sdk.connect_array(
        serials,
        width=r["width"], height=r["height"],     # before binning
        pixel_format=r["pixel_format"], binning=r["binning"])
elif probe["status"] == "needs_force_slip":
    print("Wire can't sustain sim-sync; falling back to slip mode")
    arr = chloros_sdk.connect_array(
        serials, force_tier="slip-emit-and-capture")
else:
    raise RuntimeError(f"Probe error: {probe.get('error')}")
```

### 7. Equivalente à receita de captura (Python puro)

`run_project_recipe()` executa uma receita de CLI tal como está (consulte [Pasta do projeto e receitas](#project-folder--recipes)). Escritas em Python, as mesmas etapas são:

```python
import time, chloros_sdk

with chloros_sdk.open_project("/path/to/proj") as proj:
    proj.connect_all()
    cam = proj.cameras["FrontLeft"]
    rig = proj.arrays["main_rig"]
    sky = proj.sensors["Sky"]

    # apply: connect_all() applied each camera's saved settings. CameraHandle
    # has no settings method; a CameraSession on the same pooled camera does
    # (it is borrowed, so it never releases the camera)
    chloros_sdk.connect_camera(cam.serial).set_settings(
        exposure_time=5000, gain=6.0)

    # wait
    time.sleep(2)

    # capture
    cam.capture("pose_a/", format="tiff", processing="radiance")

    # stream
    rig.stream(count=60, fps=5, output_dir="stream/", processing="raw")

    # sensor read
    print(sky.read())
```

---

## Servidor MCP (assistentes de IA)

O `chloros-mcp` permite que um assistente de IA que utilize o [Protocolo de Contexto do Modelo](https://modelcontextprotocol.io) (MCP), como o Claude Desktop, o Claude Code ou o Cursor, controle o Chloros: processe imagens (crie um projeto, importe uma pasta de capturas, escolher configurações, processar e relatar o que a execução exportou) e operar câmeras LATTICE, matrizes e sensores de luz DAQ (conectar, visualizar, definir exposição e cor, capturar, gravar e atualizar firmware). Suas ferramentas abrangem o que este SDK faz. Você faz a solicitação em linguagem simples e o assistente aciona as ferramentas do servidor:

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

The assistant calls:
  create_project          {"project_name": "Flight_2026-10-17"}
  import_images           {"folder_path": "D:\\Flights\\2026-10-17"}
  configure_processing    {"indices": ["NDVI"]}
  start_processing        {}
  get_processing_status   {}      (every 30-60 s until its state is "finished")

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.

The assistant calls:
  list_devices            {}
  get_camera_preview      {"array_id": "array-1779862544497"}
  start_capture_run       {"array_id": "array-1779862544497", "mode": "interval",
                           "interval_s": 10, "duration_s": 300,
                           "processing": "reflectance"}
  get_job_status          {"job_id": "..."}   (until its state is "finished")
```

O servidor é construído com base neste SDK: as ferramentas de processamento chamam [`ChlorosLocal`](#image-processing--chloroslocal) e as ferramentas de hardware são acionadas pelas [chamadas do smart-connect](#smart-connect-for-lattice-cameras), [sessões DAQ](#daq-sensor-sessions), `ChlorosProject` e os [utilitários de câmera](#camera-utilities), todos se comunicando com o mesmo backend que o aplicativo Chloros usa. Suas solicitações contêm `X-Chloros-Client: sdk`, portanto, o backend aplica as verificações de login e de plano do SDK, e as regras do SDK são aplicadas automaticamente (o Texture Aware é recusado para capturas LATTICE, por exemplo).

### Instalação

O servidor vem no pacote `chloros-sdk`; não há mais nada para instalar. O SDK instala um comando `chloros-mcp`, que é o mesmo que `python -m chloros_sdk.mcp_server`:

```bash
pip install chloros-sdk        # not needed where the Chloros installer set up the SDK (Step 2)
chloros-mcp --help
```

- **Windows:** o instalador do Chloros instala o SDK via pip no seu Python, o que coloca o `chloros-mcp.exe` na pasta `Scripts` desse Python.
- **Linux (.deb):** o pacote instala o SDK no sistema `python3` e coloca o comando `chloros-mcp` no `/usr/bin`, de modo que ele fica no `PATH`. Em um virtualenv, o `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl` adiciona o `chloros-mcp` ao `bin` do virtualenv.

O servidor utiliza apenas a biblioteca padrão, além do SDK (Python 3.7+). Um cliente MCP o inicia e se comunica com ele por meio de stdin/stdout; você não precisa executá-lo manualmente, exceto com o `--help`.

### Configuração do cliente

**Claude Desktop:**adicione o servidor ao `claude_desktop_config.json` (no Windows, `%APPDATA%\Claude\claude_desktop_config.json`; acesse-o em**Configurações > Desenvolvedor > Editar Configuração**), depois reinicie o Claude Desktop:

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

**Claude Code:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Cursor:** a mesma entrada `mcpServers` do Claude Desktop, em `~/.cursor/mcp.json` (em todos os projetos; `%USERPROFILE%\.cursor\mcp.json` no Windows) ou `.cursor/mcp.json` em um projeto.

As opções devem ser inseridas em `args` ou após o comando para o Claude Code (`claude mcp add chloros -- chloros-mcp --no-auto-start`). Para usar um SDK específico do Python em vez do `chloros-mcp` no `PATH`, execute o módulo com esse Python (seu caminho completo como `command`, caso não seja o primeiro `python3` no `PATH`):

```json
{
  "mcpServers": {
    "chloros": {"command": "python3", "args": ["-m", "chloros_sdk.mcp_server"]}
  }
}
```

Um cliente inicia o servidor sem o seu shell; portanto, se ele informar que o servidor não conseguiu iniciar, provavelmente o `chloros-mcp` não está no `PATH` que o cliente vê. Forneça o caminho completo (`where chloros-mcp` no Windows, `which chloros-mcp` no Linux, onde o arquivo .deb está localizado), ou execute o módulo com o Python que contém o SDK.

### Opções

| Opção | Padrão | O que faz |
| --- | --- | --- |
| `--backend-url URL` | `http://127.0.0.1:5000` | O backend do Chloros a ser usado. |
| `--no-auto-start` | desativado | Nunca inicia um backend: usa um que já esteja em execução, como o do aplicativo Chloros, o serviço de backend do Linux ou um túnel para a (consulte [Modo de backend remoto](#remote-backend-mode-pip-only-host-via-tunnel); os caminhos das pastas devem, nesse caso, indicar a mesma pasta em ambas as máquinas). |
| `--backend-exe PATH` | onde o Chloros o instala | O backend a ser iniciado quando nenhum estiver em execução, para um Chloros instalado em um local onde o SDK não procura. |
| `--log-level LEVEL` | `WARNING` | `DEBUG`, `INFO`, `WARNING` ou `ERROR`. Os logs são direcionados para stderr, que a maioria dos clientes exibe em seu log do MCP; stdout transporta apenas mensagens de protocolo. |

### Ferramentas

Os argumentos de cada ferramenta são os da chamada do SDK que ela encapsula; o cliente lista cada um com seu esquema completo. Os caminhos são absolutos.

**Processamento**

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `create_project` | `ChlorosLocal.create_project` | Cria um projeto e o torna o projeto aberto, sobre o qual as ferramentas de processamento atuam. Um nome simples cria a pasta na pasta padrão do projeto; um caminho absoluto a cria nesse local. |
| `open_project` | `ChlorosLocal.open_project` | Abre um projeto salvo e o torna o projeto aberto. Os caminhos de um projeto movido são redirecionados para a pasta em que ele se encontra agora; `relink_folder` (absoluto) indica a pasta em que os demais arquivos ausentes estão agora. Retorna `source_files` e `image_count`. Nenhum encontrado: uma lista de erros indicando onde estavam e nomeando `relink_folder`; nada está aberto. |
| `import_images` | `import_images` | Importa pares Survey3 `.RAW`+`.JPG`, LATTICE `.tif` capturas e DAQ `.daq` gravações; retorna `count` (imagens) e `scan_count` (gravações). |
| `configure_processing` | `configure` | Altera as configurações passadas e nenhuma outra (valores: [Valores suportados](#supported-values)); `custom_settings` envia toda a configuração tal como fornecida. |
| `get_project_settings` | `get_config` | Lê as configurações salvas de processamento, exportação e índice do projeto aberto. |
| `start_processing` | `process(wait=False)` | Inicia o processamento do projeto aberto e retorna imediatamente. |
| `get_processing_status` | `get_status` | Relata `state` (`idle`, `starting`, `processing` ou `finished`) e o andamento; uma vez concluída, a execução gera os relatórios `summary`, `export_check_failure` quando as exportações não puderam ser verificadas e `image_failure` quando as imagens foram perdidas: os demais produtos são gravados; registre a execução como falhada. |
| `process_folder` | `process_folder` / `process_lattice_capture` | Cria um projeto para uma pasta, importa-o, configura e inicia a execução em uma única chamada (`lattice: true` para capturas LATTICE). Enquanto uma execução está em andamento, ela é recusada antes que qualquer coisa seja criada, e o backend também recusa uma alteração no projeto a partir de qualquer ferramenta nesse momento. |
| `export_light_sensor` | `export_light_sensor` | Grava os arquivos calibrados `.daq` e `.csv` para as gravações do sensor de luz do projeto sem processar imagens, obtendo a calibração de fábrica da MAPIR Cloud quando ela não estiver armazenada em cache (consulte [Gravações do sensor de luz](#light-sensor-recordings--calibrated-daq--csv)). |
| `read_image_audit_tags` | `read_image_audit_tags` | Lê os metadados de auditoria do Chloros de uma imagem processada (fonte, calibração, estágios aplicados). |

**Câmeras e matrizes**

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `list_devices` | `list_cameras`, `list_arrays`, `list_daq_sensors` | O que está conectado no momento, em qualquer cliente, cada um com `connected_here`; além dos trabalhos em execução no servidor. |
| `discover_devices` | `discover_lattice_cameras`, `discover_daq_sensors` | Câmeras e sensores de luz que poderiam estar conectados. |
| `get_device_status` | `CameraSession.status`, `ArraySession.status` / `capability`, `DAQSensorSession.status` | Estado em tempo real de uma câmera, de um conjunto de câmeras (com suas capacidades e integridade) ou de um sensor de luz. |
| `analyze_array_network` | `analyze_array_network` | O que este host e esta rede podem suportar para um conjunto proposto. |
| `connect_camera` / `connect_array` | `connect_camera` / `connect_array` | Conecte uma câmera ou um conjunto sincronizado (mestre primeiro). |
| `connect_project_devices` | `open_project(...).connect_all` | Conecte todos os dispositivos memorizados em um projeto salvo, com suas configurações. |
| `disconnect_device` | `disconnect` / `force_disconnect` | Desconecte uma câmera, um conjunto ou um sensor de luz (consulte Comportamento). O `keep_settings: true` em um conjunto salva as configurações de cada câmera em sua própria memória; o `clear_settings: true` reinicia essa memória (seu resultado leva o código `camera_memory`). |
| `disconnect_project_devices`, `align_project_arrays`, `capture_project_devices` | `disconnect_all`, `align_arrays`, `capture_all` | Os dispositivos do projeto, juntos. |
| `read_camera_settings` / `set_camera_settings` | `CameraSession.read_nodes` / `set_settings` | Ler ou gravar configurações da câmera, por nome amigável ou nome de nó GenIC. |
| `set_array_frame_rate` | `ArraySession.set_target_fps` | Definir a taxa de um array ou restaurá-la para a taxa automática. |
| `capture_camera` / `capture_array` | `CameraSession.capture` / `ArraySession.capture` | Uma captura (`fastest` e `wait_for_exposure` são `capture_fastest` e a captura com exposição ajustada). Na ausência de `output_dir`, ele salva na pasta aberta do projetopasta `captures` do projeto aberto, e os arquivos são incorporados ao projeto (retidos até que uma execução de processamento seja concluída); a resposta `project_add` confirma isso. |
| `start_capture_run` | `ArraySession.continuous`, `capture_repeated` | Capturas contínuas ou em intervalos, como uma tarefa. |
| `start_array_recording` | `ArraySession.record` / `burst` | Um vídeo com índice combinado ou uma sequência bruta, como uma tarefa. |
| `build_burst_video` | `ArraySession.build_video` | Reprocessar uma sequência em vídeo calibrado, como uma tarefa. |
| `get_job_status` / `stop_job` | | Acompanhar ou interromper uma tarefa. |
| `get_camera_preview` | | Visualização ao vivo de cada câmera, retornada como uma imagem. |
| `grab_frames` | `ArraySession.grab`, `CameraHandle.grab` | Um quadro (ou um conjunto sincronizado) em um nível de processamento, sem salvar: estatísticas por plano e um arquivo PNG de cada um. |
| `calibrate_array_alignment`, `get_array_alignment`, `clear_array_alignment`, `nudge_array_alignment`, `export_array_alignment`, `import_array_alignment` | `ArrayHandle.calibrate_alignment`, `alignment_status`, `clear_alignment`, `tweak_alignment`, `export_alignment`, `import_alignment` | Alinhamento de uma matriz. |

**Sensores de luz**

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `connect_light_sensor` | `connect_daq_sensor` | Conecta um DAQ-U, DAQ-M ou DAQ-E (`cap_id` define sua correção de limite). |
| `get_light_sensor_reading` | `DAQSensorSession.latest` | As leituras mais recentes: carimbo de data/hora, saturação, faixa e pico do espectro, e a média do espectro em intervalos de comprimento de onda (`full_spectrum` para cada ponto). |
| `set_light_sensor_streaming` | `stream_start` / `stream_stop` | Iniciar ou interromper o fluxo. |
| `start_light_sensor_recording` / `stop_light_sensor_recording` | `record_start` / `record_stop` | Gravar um arquivo `.daq` como uma tarefa; `duration_s` o interrompe automaticamente. |
| `set_light_sensor_cap` | `DAQSensorSession.set_cap` | A correção de tampa (`none` para um sensor sem cobertura). |
| `get_light_sensor_bindings` / `set_light_sensor_bindings` | `daq_bindings` / `set_daq_bindings` | Qual sensor fornece a refletância de cada câmera e matriz. |
| `get_light_sensor_band_presets` / `get_light_sensor_reflectance` | `daq_band_presets` / `light_sensor_reflectance` | Os conjuntos de bandas e a refletância e os índices “Ambiente + Objeto” de dois sensores de transmissão contínua. |
| `get_light_sensor_device_info`, `set_light_sensor_network`, `set_light_sensor_auto_stream`, `enable_light_sensor_ptp`, `reboot_light_sensor`, `update_light_sensor_firmware` | `device_info`, `set_network`, `set_auto_stream`, `enable_ptp`, `reboot`, `update_firmware` | Configurações próprias de um DAQ-E, salvas na unidade. |

**Utilitários da câmera** ([Utilitários da câmera](#camera-utilities))

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `get_camera_color_settings` / `set_camera_color_settings` | `camera_color_profile`, `camera_color_adjust`, `camera_highlights` e seus setadores | Cores de exibição de uma câmera RGB: perfil e temperatura de cor, saturação/contraste, realces. |
| `run_camera_white_balance` / `set_camera_white_balance` | `camera_white_balance_once` / `set_camera_white_balance` | Equilíbrio de branco de disparo único, ou automático / desligado / proporções fixas. |
| `list_camera_presets` / `apply_camera_preset` | `camera_presets` / `apply_camera_preset` | Predefinições da câmera. |
| `list_filter_models` | `camera_filter_models`, `camera_filter_model_info` | Os modelos de filtro ou um por nome. |
| `get_camera_firmware_status` / `update_camera_firmware` | `camera_firmware_status` / `update_camera_firmware` | Versões de firmware e um flash (uma tarefa por padrão). |
| `get_camera_power_info` / `set_camera_power` | `camera_power_probe`, `sleep_camera`, `wake_camera` | Estado de energia e térmico; modo de suspensão ou ativo. |
| `calibrate_camera_with_target` | `calibrate_camera_with_target` | Coeficientes de um alvo de calibração na tela. |
| `get_network_adapters` / `fix_network_settings` | `network_adapters` / `fix_network_settings` | Os adaptadores de rede da câmera, uma estimativa e a correção (Somente para administradores). |
| `apply_vignette_correction` | `apply_vignette_correction` | Corrigir a vinheta de uma pasta de imagens. |
| `read_dls` | `read_dls` | Uma leitura DLS. |
| `probe_system` / `get_system_info` | `probe_system` / `system_info` | Hardware e tempo de execução do host. |

**Câmeras transitórias, perfis de alinhamento e índices** ([Matrizes transitórias, perfis de alinhamento e índices](#transient-arrays-alignment-profiles--indices)). Todas as ferramentas aqui que abrem câmeras utilizam `keep_settings` / `clear_settings` (padrão: false): consulte [Memória da câmera](#camera-memory--keep_settings--clear_settings).

| Ferramenta | Chamada do SDK | O que faz |
| --- | --- | --- |
| `start_camera_continuous` / `start_multi_camera_stream` | `start_camera_continuous` / `start_multi_camera_stream` | Grava uma câmera ou transmite várias em sincronia, como uma tarefa (`duration_s` interrompe automaticamente). |
| `capture_cameras_transient` | `capture_cameras_transient` | Uma captura sincronizada sem conectar um array. |
| `test_array_sync` / `detect_array_master` | `test_array_sync` / `detect_array_master` | Verificação da sincronização do disparador e quais câmeras estão na linha de sincronização. |
| `analyze_array_models` | `analyze_array_models` | O que um conjunto desses modelos obteria nesta rede. |
| `calibrate_alignment_profile` | `calibrate_alignment_profile` | Resolver e salvar um arquivo de perfil de alinhamento (`options` contém as opções mais detalhadas do solucionador). |
| `capture_aligned_multiband` / `start_aligned_stream` | `capture_aligned_multiband` / `start_aligned_stream` | Uma imagem multibanda alinhada ou um fluxo delas como uma tarefa. |
| `read_alignment_profile` / `reorder_alignment_profile` | `alignment_profile_info` / `reorder_alignment_profile` | O conteúdo de um arquivo de perfil; sua ordem de bandas e câmeras ativadas. |
| `list_index_presets`, `compute_index`, `capture_live_index` | `index_catalog`, `compute_index`, `capture_live_index` | Predefinições e gradientes de índice; um índice para um arquivo TIFF multibanda; captura para índice em uma única etapa. |

**Sincronização de tempo, pasta do projeto e receitas** ([PTP Grandmaster](#ptp-grandmaster--time_sync_status), [Pasta do projeto e receitas](#project-folder--recipes))

| Ferramenta | Chamada SDK | O que faz |
| --- | --- | --- |
| `get_time_sync_status` | `time_sync_status`, `time_sync_peers`, `time_sync_cameras` | O estado do grandmaster PTP, de seus pares e o estado PTP de cada câmera do array. |
| `restart_time_sync` / `set_time_sync_priority` | `restart_time_sync` / `set_time_sync_priority` | Reinicie-o; defina suas prioridades. |
| `get_project_folder` / `set_project_folder` / `reset_project_folder` | `project_folder` / `set_project_folder` / `reset_project_folder` | A pasta na qual novos projetos são criados. |
| `run_project_recipe` | `run_project_recipe` | Execute a receita de um projeto salvo como uma tarefa com seu log. Verificação inicial: uma receita com erro é rejeitada e nada é executado; matrizes/sensores não conectados são sinalizados (eles não são conectados: `connect_project_devices` primeiro). `dry_run` retorna a verificação e nada é executado. |

Não há ferramenta para abrir um projeto salvo para processamento: cada tarefa de processamento começa com `create_project` (ou `process_folder`).

### Requisitos

- **Chloros instalado no mesmo computador.** O backend do Chloros realiza o processamento. O servidor pode, em vez disso, usar um backend acessado por meio de um túnel (`--backend-url` com `--no-auto-start`, consulte [Opções](#options)).
- **Estar conectado nesse computador**, no aplicativo Chloros ou com o `chloros-cli login`. O servidor não pode fazer o login por você: uma ferramenta chamada enquanto você estiver desconectado responde “Não está conectado ao Chloros” com o motivo do backend, e o assistente informa a você.
- **Qualquer plano pago do Chloros+ (Copper ou superior).** O servidor é acessível via SDK, e o plano gratuito Iron não oferece esse acesso: uma ferramenta chamada no plano gratuito responde “Não disponível neste plano do Chloros” com o motivo fornecido pelo backend.
- Python 3.7 ou mais recente com o SDK e um cliente MCP que inicie servidores locais (stdio).

### Comportamento

- **O backend.** Iniciar o servidor ou listar suas ferramentas não inicia nada; a primeira chamada à ferramenta estabelece a conexão. Se o aplicativo Chloros (ou o serviço de backend do Linux) estiver em execução, o servidor usa seu backend e o mantém em execução. Se nenhum backend estiver em execução, o servidor inicia o que está instalado, sem janela. Quando o servidor é encerrado (você sai do cliente ou ele reinicia o servidor), ele interrompe os trabalhos de captura e as gravações do sensor de luz que iniciou e libera os dispositivos aos quais se conectou; o backend continua em execução, de modo que uma execução de processamento inacabada continua até o fim e ele se desliga sozinho após 30 minutos sem que nada o esteja utilizando. Com o `--no-auto-start`, o servidor nunca inicia um backend: uma ferramenta chamada sem nenhum backend em execução solicita que você abra o aplicativo Chloros, e a próxima chamada após você fazer isso estabelece a conexão.
- **Um projeto aberto.** O backend tem um projeto aberto, compartilhado com o aplicativo Chloros quando este está aberto; portanto, não trabalhe em outro projeto no aplicativo enquanto o assistente estiver em funcionamento.
- **Apenas as configurações passadas são alteradas.** O `configure_processing` mantém todas as configurações que não forem fornecidas em seus valores salvos (o SDK do `configure()` redefine o debayer, a correção de vinheta, a calibração de refletância, o PPK e o formato de exportação para seus padrões quando não são passados; o servidor as preenche a partir do projeto). Se não conseguir ler as configurações salvas do projeto, ele não altera nada e informa isso, a menos que todas as cinco sejam fornecidas. O `indices` substitui a lista de índices do projeto. O resultado lista o que foi enviado como `settings_sent`, e nomeia sob `warnings` qualquer índice que as faixas da câmera não possam formar.
- **O processamento é do tipo “iniciar e depois consultar”.** `start_processing` retorna assim que a execução começa, o que leva de minutos a horas, e o assistente segue com `get_processing_status`. `state` é `idle` (sem processamento), o `starting` (o backend ainda não informou a execução, por até 20 s), o `processing` ou o `finished`, com o [resumo](#post-run-summary--hints): o que foi exportado e dicas quando a exportação foi menor do que o esperado (sem imagens, nenhum alvo de calibração encontrado). Os produtos são gravados na pasta do projeto, assim como no aplicativo. O `start_processing` é rejeitado enquanto uma execução estiver em andamento, e um backend que deixa de responder é relatado como um erro, não como uma execução concluída.
- **O que o assistente vê.** Os avisos do SDK aparecem no resultado da ferramenta sob o código `warnings`, e as falhas (saída da conta, plano, caminho incorreto) são exibidas como texto de erro que ele pode ler e repassar a você. Uma lista com mais de 20 itens é reduzida aos primeiros 20, com sua extensão completa indicada como `<key>_total`. Os caminhos devem ser absolutos.
- **Os dispositivos são compartilhados.** Câmeras, matrizes e sensores de luz ficam no backend; assim, o assistente pode controlar um deles pelo aplicativo Chloros, e as alterações feitas aparecem no aplicativo. `disconnect_device` exibe o que o servidor conectou; um dispositivo conectado por outro cliente (o aplicativo, `chloros-cli`, um script, outro `chloros-mcp`) é liberado apenas com o `force: true`, que é o `force_disconnect` do SDK, e a recusa indica quem o conectou quando o backend sabe; um dispositivo que o backend não possui de forma alguma é relatado como não conectado (`released: false`, `connected: false`, nada para liberar). Uma gravação ou transmissão executada pelo backend gera sua visualização `get_job_status` com `counts` — `frames_written`, `frames_dropped`, `frames_failed`, `drop_reason` (grupos de quadros para uma transmissão com várias câmeras) — e uma linha única `summary` (“220 quadros gravados, 55 descartados, 0 com falha: ...”), enquanto está em execução e, ao terminar, seus números finais; a listagem de outro cliente sobre isso (`other_clients_jobs`) apresenta os mesmos dados (`list_cameras()`, `list_arrays()` e `list_daq_sensors()` apresentam, cada um, os dados do respectivo dispositivo`connected_by`: `client`, `label`, `name`, `at`). Cada `chloros-mcp` se identifica para o backend como `chloros-mcp (pid N)` (`chloros_sdk.connect.CLIENT_LABEL`, enviado como `X-Chloros-Client-Label`; qualquer script SDK pode defini-lo). Interromper a gravação do sensor de luz de outro cliente também envolve o `force`.
- **Trabalho prolongado é uma tarefa.** `start_capture_run`, `start_array_recording`, `build_burst_video`, `start_light_sensor_recording`, `start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`, `run_project_recipe`, e qualquer captura, resolução ou indexação solicitada por `background` retorna um `job_id` imediatamente; `get_job_status` vem em seguida e `stop_job` encerra a sequência. Uma execução continua até ser interrompida, a menos que você forneça `max_captures`, `duration_s` (ou, para uma sequência contínua, `max_frames`); o comando `min_free_gb` se recusa a iniciar quando a unidade tem pouco espaço livre. Nenhum limite é aplicado, a menos que você defina um. Uma gravação ou transmissão executada pelo backend (`start_camera_continuous`, `start_multi_camera_stream`, `start_aligned_stream`) mantém o ID da tarefa do backend, de modo que outro `chloros-mcp` — iniciado posteriormente — o localiza por meio desse ID: `get_job_status` o acompanha (e lista tarefas como `other_clients_jobs`, a partir de `GET /api/camera/cli-jobs`, cada uma com `started_by`), e o `stop_job` o encerra somente com o `force: true`. Qualquer outro trabalho (uma execução de captura, um registro em matriz ou em rajada, uma captura em segundo plano) é executado no `chloros-mcp` que o iniciou, e somente esse servidor tem conhecimento disso: outro servidor que o solicitou informa isso. Um processo interrompido lê o `stopping` enquanto grava sua fila e, em seguida, o `finished` com suas contagens finais, por 10 minutos e a partir de qualquer `chloros-mcp`: aquele que iniciou o processo percebe a parada forçada de outro da mesma forma. O `get_job_status` nunca aguarda uma parada: durante uma (seja a do próprio `stop_job`, a de outro clienteou a interrupção que libera as câmeras de um trabalho que terminou por conta própria), ele responde imediatamente com o `stopping` e as contagens em tempo real, e o `stop_job` de um trabalho já em interrupção informa isso. Um trabalho que terminou indica o motivo no `stop_reason`: seu próprio motivo (`no_frames`, `disk_full`, `member_missing`, ou, para uma execução contínua em matriz, `count`, `duration`, `array_gone`), `limit reached` apenas quando suas contagens indicam seu `count` (ou o `max_frames` de uma sequência, o`duration_s`) for atingido, `error`, `stopped` (com `stopped_by`) quando um cliente a interrompeu; caso contrário, `finished`.
- **Para onde vão as capturas.** Para a pasta `captures` do projeto aberto, onde o aplicativo as salva, a menos que `output_dir` seja especificado.
- **Imagens.** `get_camera_preview` e `grab_frames` retornam imagens que o assistente pode visualizar, além de seus dados em JSON. As imagens `grab_frames` têm o contraste ajustado para visualização; suas estatísticas são os valores reais.
- **Quando o servidor é encerrado**, as tarefas que ele iniciou são interrompidas e uma gravação do sensor de luz iniciada por ele é encerrada. Se o servidor iniciou o backend, ele também libera os dispositivos aos quais se conectou antes de interromper o backend; no backend do aplicativo, eles permanecem conectados.
- **Protocolo.** Apenas stdio (sem endpoint HTTP), apenas ferramentas (sem recursos ou solicitações), resultados em texto e imagem, versões do protocolo MCP de 05/11/2024 a 25/11/2025.**O que foi deixado de fora.** Fazer login ou logout (a conta é sua; o servidor não pode fazer login) e parar um backend que o servidor não iniciou. Todo o restante que este SDK faz com processamento e hardware possui uma ferramenta. (O provisionamento de um DAQ-E — seu nome, token de autenticação, senha OTA, exigência de assinatura, redefinição de fábrica — e os servidores DAQ diretos são exclusivos do `chloros-cli`, também no SDK: um nome ou token incorreto bloqueia todas as instalações do Chloros na unidade.)

---

## Inicialização automática do back-end

Os pontos de entrada do smart-connect — `connect_camera`, `connect_array`, `connect_daq_sensor`, `discover_lattice_cameras` e `discover_daq_sensors` — são clientes HTTP leves que presumem que um backend esteja escutando no `127.0.0.1:5000` (a URL padrão da interface smart-connect). Quando a GUI ou a CLI já estiverem em execução, um deles já estará. A partir de um script simples, pode não haver nenhum — portanto, essas funções **inicializam automaticamente o binário do backend incluído** (sem janela, da mesma forma que o `ChlorosLocal` faz) antes de sua primeira chamada e, em seguida, aguardam até 60 s para que ele seja iniciado. `list_cameras()`, `list_arrays()`, `list_daq_sensors()`, `attach_array()` e `analyze_array_network()` nunca iniciam um.

Regras:

- **Apenas um URL local é gerado.** Um `backend_url` apontando para `localhost` / `127.0.0.1` / `[::1]` é elegível; qualquer outro host é considerado como sendo a máquina de outra pessoa e nunca é gerado.
- **Um backend que seja encerrado antes de responder encerra a espera.** A chamada retorna imediatamente com o código de saída, o comando executado e as últimas linhas exibidas pelo backend, em vez de aguardar os 60 s. O código de saída 98 é a exceção: outro backend já detém a porta ou o arquivo de bloqueio (do aplicativo Chloros, do serviço de backend ou de outro script que ainda está iniciando), de modo que a chamada continua aguardando que esse responda. `ChlorosLocal` faz o mesmo.
- **O backend é mantido em execução para reutilização** (da mesma forma que na CLI) — não há desligamento implícito quando seu script é encerrado. Ao executar o script novamente, o backend ativo é reutilizado.
- **Desative com `auto_start_backend=False`** em qualquer uma dessas chamadas (por exemplo, quando você tiver apontado para um backend remoto ou gerenciar o ciclo de vida do backend por conta própria).

```python
import chloros_sdk

# Fresh shell, no backend running, no GUI open — this still works:
with chloros_sdk.connect_camera("213800234") as cam:   # spawns the backend
    cam.capture("output/")

# Remote backend (via tunnel — see Remote-Backend Mode): don't spawn one locally
arr = chloros_sdk.connect_array(serials,
                                backend_url="http://127.0.0.1:5000",
                                auto_start_backend=False)
```

Se o binário incluído não puder ser localizado ou iniciado, a chamada HTTP subsequente gera um `ChlorosConnectError` acionável e **sensível à plataforma**, em vez de um simples rastreamento de conexão recusada — no Windows, ele direciona você para o aplicativo de desktop ou para um comando `chloros-cli`; no Linux (sem GUI), ele direciona você para um comando `chloros-cli` ou para o `.deb`.

---

## Ambiente e cabeçalhos

O SDK marca todas as chamadas HTTP do backend com `X-Chloros-Client: sdk`. O backend aplica as regras de licenciamento do SDK/CLI (é necessário fazer login **e** ter um plano pago do Chloros+) em vez do caminho da camada gratuita da GUI. Isso é definido automaticamente no momento da importação — você não precisa fazer nada.

O cabeçalho é adicionado a todas as chamadas `requests` / `urllib` cuja URL contenha `localhost` ou `127.0.0.1`, em qualquer porta — incluindo seus próprios serviços locais — e substitui um valor definido por você. Chamadas para outros hosts não são alteradas.

Substitua a URL do backend passando `backend_url=` (`ChlorosLocal` aceita `backend_url=` ou `api_url=`; `process_folder()` e `process_lattice_capture()` aceitam apenas `api_url=`):

```python
chloros_sdk.connect_camera("213800234", backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_array(serials, backend_url="http://127.0.0.1:5000")
chloros_sdk.connect_daq_sensor(eth_host="daq-e-1.local",
                                backend_url="http://127.0.0.1:5000")
chloros_sdk.ChlorosLocal(backend_url="http://127.0.0.1:5000")
```

(Um backend instalado escuta apenas no loopback, portanto, um `backend_url` da LAN não consegue acessá-lo; consulte [Modo de Backend Remoto](#remote-backend-mode-pip-only-host-via-tunnel) para o padrão de túnel.)

---

## Versões e compatibilidade

- A versão do SDK é exposta como `chloros_sdk.__version__`.
- O SDK vincula o comportamento à versão do backend incluída no pacote. Combinar um SDK mais antigo com um backend mais recente geralmente funciona (pontos de extremidade compatíveis com versões futuras), mas combinar um SDK mais recente com um backend mais antigo pode gerar erros `404` em novos pontos de extremidade — atualize o aplicativo para desktop para que fique compatível.
- A interface do smart-connect (`connect_camera` / `connect_array` / `connect_daq_sensor`) e o endpoint de análise de rede retornam esquemas JSON estáveis; novos campos são adicionais.

---

## Dicas para solução de problemas

- **`ChlorosAuthenticationError: Login required`** (de `ChlorosLocal`; as chamadas do Smart-Connect geram o erro `ChlorosConnectError: Login required. …` pelo mesmo motivo) → Execute o `chloros-cli login EMAIL PASSWORD` uma vez nesta máquina ou faça login pelo aplicativo de desktop do Chloros.
- **`ChlorosProcessingError: Failed to start processing: … The project folder is too deep for this computer …`** (ou `Failed to create project: …`, proveniente de `create_project()`) → Os caminhos longos do Windows estão desativados nesta máquina (configuração padrão do Windows), portanto, nenhum caminho pode ter mais de 260 caracteres, e os produtos ficam até cerca de 130 caracteres abaixo da pasta do projeto. Nada foi processado. A mensagem indica a pasta de projeto mais longa que cabe: use uma mais curta ou ative os caminhos longos do Windows e reinicie o Chloros. O instalador do Windows oferece essa opção como “Ativar caminhos longos do Windows”, marcada por padrão; caso contrário, use a Política de Grupo: *Configuração do Computador > Modelos Administrativos > Sistema > Sistema de Arquivos > Ativar caminhos longos do Win32*.
- **`ChlorosConnectError: No Chloros backend is running …`** → O smart-connect chama a inicialização automática de um backend local; portanto, isso só aparece quando o binário incluído não pode ser encontrado/iniciado (por exemplo, um host apenas pipsem pacote de desktop). A mensagem varia de acordo com a plataforma: no Windows, abra o aplicativo de desktop ou execute qualquer comando `chloros-cli`; no Linux, execute um comando `chloros-cli` (não há GUI disponível) ou instale o `.deb`. Para um backend remoto, passe `backend_url=` (e `auto_start_backend=False`).
- **`CAMERA_AVAILABLE == False`** na importação → Falha ao carregar `lattice_sdk`: o runtime do SDK da Arena ou o OpenCV (`cv2`), não pode ser importado neste Python. A superfície orientada por backend (`connect_*`, projetada por meio de um backend, `ChlorosLocal`) ainda funciona.
- **O Array se conecta com o sensor completo, mas com baixa taxa de quadros**→ Por padrão: o modo automático mantém o sensor completo e permite que a taxa caia para o que a conexão suporta. Passe `binning=2` / `4` ou um valor menor, como `width` / `height`, para obter mais quadros por segundo, ou habilite quadros jumbo / atualize o link. O binning e a ROI**não** cobrem a superscrição agregada (`oversubscribed: true`, campos de fps 0): o excesso de câmeras para a largura de banda não pode ser resolvido por binning/ROI — reduza o número de câmeras, habilite quadros jumbo ou mude para uma placa de rede mais rápida (consulte [Superscrição](#over-subscription-the-per-cam-floor)).
- **O código `analyze_array_network()` indica que o anel de recepção da placa de rede está muito pequeno (~0,26 MB) / portas de conexão com a mensagem “FRAMES WILL DROP”** → O anel de recepção da placa de rede do host está em seu valor padrão (geralmente redefinido para 32 após uma atualização do driver da placa de rede). Em um adaptador USB 10 GbE da Realtek, defina `ReceiveBufferLen=256` e `PendingReceives=64` (com privilégios elevados) e, em seguida, reinicie o backend para que ele releia o anel. Procedimento completo: [Referência da CLI → Configuração e ajuste da placa de rede do host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **O host trava ao reiniciar/desligar; posteriormente, erros WMI `Invalid class` / a placa de rede não é ativada** → Driver USB 10GbE desatualizado causando o erro `DRIVER_POWER_STATE_FAILURE` (BSOD `0x9F`). Atualize o driver do adaptador para uma versão atual (≥ 2026) e reaplique as configurações do anel de recepção. Consulte [Referência da CLI → Configuração e ajuste da NIC do host](cli-reference.md#host-nic-setup--tuning-lattice-arrays).
- **A captura do array falha com o código 503 do HTTP: “Nenhum novo grupo de quadros sincronizado”** → O array não publicou nenhum novo grupo de quadros durante o tempo de espera. Verifique se o código `arr.status()`: `stalled: True` (ou `streaming: False`) com `actual_fps` 0, o que significa que o ciclo de captura parou de publicar — desconecte e reconecte o array. Se `stalled` for False e `actual_fps` for maior que 0, tente novamente. Logo após `connect_array()`, uma captura já aguarda até 20 s pelo primeiro grupo.
- **`status()` exibe `link_speed_mbps: 4294`** → O Windows não conseguiu ler a velocidade do adaptador; isso é comum em adaptadores Ethernet USB e não constitui uma falha. Consulte [Taxas de quadros esperadas](#expected-frame-rates).
- **`update_firmware()` falha com `device never connected back on TCP <port>`** → Um firewall bloqueou a conexão do DAQ-E de volta a este computador. O instalador do Windows adiciona a regra (consulte [Configurações do dispositivo DAQ-E](#daq-e-device-settings)); com outro firewall, permita o tráfego TCP de entrada para o backend a partir da sub-rede do sensor.
- **Refletância recusada** → Um DAQ ativo deve estar vinculado à câmera (ou matriz) para obter a refletância em escala absoluta. Faça a vinculação por meio da interface gráfica ou use o `processing="radiance"` (W/m²/sr/nm), que não requer um sensor emparelhado.
- **A captura do `smart=True` demora mais do que o esperado** → A convergência do AE depende da dinâmica da cena. Com o `ArrayHandle.capture_smart()`, amplie o `exposure_tolerance_pct`, reduza o tempo de `stability_window_s` ou diminua o valor de `settle_timeout_s` para obter um acionamento mais rápido e menos estabilizado.

---

## Veja também

- [Referência da CLI](cli-reference.md) — cada subcomando da CLI corresponde a uma chamada do SDK.
- [Sensores de luz DAQ](https://mapir.gitbook.io/chloros/daq-light-sensors) no manual do Chloros — os sensores, seus limites e faixa calibrada, gravação, rede DAQ-E.
- [Câmeras LATTICE](https://mapir.gitbook.io/chloros/lattice-cameras/lattice) no manual do Chloros — conexão, configurações da câmera, modos de captura, matrizes.
- Documentação online: `https://mapir.gitbook.io/chloros/api-python-sdk`
