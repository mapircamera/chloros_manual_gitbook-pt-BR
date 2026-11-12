---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>O

Chloros é um aplicativo da [MAPIR](https://www.mapir.camera) para processar imagens multiespectrais, controlar em tempo real o hardware da MAPIR e registrar dados de sensores. O Chloros 1.3.0 é compatível com toda a linha de produtos da MAPIR:

* **Câmeras Survey3** — processam capturas RAW+JPG, transformando-as em mapas calibrados de refletância e índices de vegetação. Consulte [Câmeras compatíveis](supported-cameras.md).
* **Câmeras LATTICE** — conectam os módulos de câmera multiespectral GigE em tempo real, individualmente ou como conjuntos sincronizados de várias câmeras: visualizam, capturam e processam em produtos calibrados de radianância e refletância. Consulte a [seção LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — Sensores espectrais DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E e DAQ-E-S (Ethernet): espectros calibrados em tempo real, gravações `.daq` e iluminação descendente para processamento de refletância. Consulte a [seção DAQ](daq/README.md).

{% hint style="success" %}
**Novidades no Chloros 1.3.0**: assistentes de IA podem controlar o Chloros por meio do `chloros-mcp`, e a CLI, o SDK e o `chloros-mcp` oferecem os mesmos recursos. Os produtos do LATTICE Index foram aprimorados, os arranjos de sensores completos estão mais rápidos e funcionam em qualquer rede, e o backend continua em execução entre os comandos. Consulte a lista de novidades abaixo e [baixe](download.md) os instaladores.
{% endhint %}

{% hint style="info" %}
**Está usando o Chloros com um assistente de IA?** Este manual foi feito para isso. Peça ao seu assistente para acessar:

* `https://mapir.gitbook.io/chloros/llms.txt` — índice legível por máquina de todas as páginas.
* Qualquer página como Markdown bruto — acrescente `.md` à sua URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência da CLI](reference/cli-reference.md) e a [Referência do SDK](reference/sdk-reference.md) — páginas de referência completas com valores exatos, escritas para uso por LLMs.

Exemplo de prompt: *“Leia https://mapir.gitbook.io/chloros/reference/cli-reference.md, e, em seguida, escreva um script que faça login e processe a pasta ~/flights/flight_001 em GeoTIFFs de refletância + NDVI.”*

Ou execute o Chloros diretamente: o `chloros-mcp`, um servidor MCP instalado com o SDK, funciona com o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP.

Guia completo: [Como usar o Chloros com assistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novidades no Chloros 1.3.0

* **Os assistentes de IA podem controlar o Chloros.** O `chloros-mcp`, um servidor MCP instalado com o SDK do Python, permite que o Claude Desktop, o Claude Code, o Cursor e outros clientes MCP processem imagens e operem câmeras LATTICE, matrizes e sensores de luz DAQ. Consulte [Como usar o Chloros com assistentes de IA](ai-assistants.md).
* **A CLI, o SDK e o `chloros-mcp` oferecem os mesmos recursos.** Apenas o login, o provisionamento do DAQ-E e os servidores DAQ diretos permanecem na CLI, por motivos de segurança. Alguns scripts da versão 1.2.0 precisam de alterações: consulte **Alterações na versão 1.3.0** no início da [Referência da CLI](reference/cli-reference.md) e da [Referência do SDK](reference/sdk-reference.md).
* **Produtos de índice LATTICE aprimorados.** O NDVI e os demais produtos de índice exportados das capturas do LATTICE foram aprimorados em todos os modos de processamento.
* **Opções do instalador do Windows.**Em**Configurações do sistema**, o instalador oferece as opções**Ativar caminhos longos do Windows**e**Excluir o backend do Chloros da verificação do Microsoft Defender**, ambas marcadas por padrão. A primeira permite que as pastas de projeto tenham nomes com mais de 260 caracteres; a segunda impede que a verificação em tempo real retarde a captura da câmera (cerca de 30% da taxa de captura em taxas elevadas). O instalador agora também adiciona a pasta CLI ao `PATH` apenas uma vez e remove as entradas duplicadas deixadas por instaladores anteriores. Consulte [Download](download.md).
* **Pastas de projeto com hierarquia muito profunda são rejeitadas logo de início.** Com os caminhos longos do Windows desativados (configuração padrão do Windows), uma pasta de projeto com hierarquia muito profunda para o limite de 260 caracteres no caminho é rejeitada antes que qualquer operação seja executada, exibindo a pasta de projeto mais longa que cabe e instruções sobre como ativar os caminhos longos. Anteriormente, essa execução não gerava nenhum produto. Com os caminhos longos ativados, os produtos em pastas muito profundas agora mantêm seus metadados (tags GPS e XMP).
* **Máscaras de limitação.** Quando a separação de bandas de um produto de radiação ou refletância do LATTICE produz um valor de banda negativo, esse valor é limitado a zero, e uma máscara por pixel em uma pasta `clamp_masks/` ao lado das pastas de produtos registra quais pixels e bandas foram afetados. O produto nomeia sua máscara na tag `Chloros:UnmixClampMask`. As visualizações em tempo real agora realizam o recorte exatamente da mesma forma que a exportação. Consulte [Formatos de imagem de saída](output-image-formats.md).
* **Os produtos se descrevem.** Os produtos de índice possuem suas próprias tags de processamento (`Chloros:ProcessingLevel=index`, a expressão do índice e se foram calculados a partir da radianância ou da refletância) em vez das do arquivo bruto de origem. Os produtos do Survey3 possuem tags de processamento do Chloros em todos os modos de processamento. Os arquivos brutos e debayered do LATTICE permanecem na orientação nativa do sensor e registram sua orientação no XMP; o Visualizador de Imagens os gira apenas para exibição.
* **Os pacotes para Linux são fornecidos descompactados.** O backend e a CLI são instalados como pastas dentro de `/usr/lib/chloros/`, de modo que não precisam mais se descompactar a cada inicialização. No x86_64, o backend não leva mais cerca de 100 segundos para iniciar, e o `.deb` tem aproximadamente metade do tamanho anterior. O ExifTool incluído agora funciona em uma instalação limpa do Ubuntu. O `.deb` agora também instala o Python SDK para o sistema `python3` (instale-o com o `sudo apt install ./chloros_1.3.0_<arch>.deb`), aumenta os buffers de recepção de soquetes do kernel necessários para os fluxos da câmera e adiciona um serviço de backend que é executado como seu usuário. O pacote Jetson, `chloros_1.3.0_arm64_jp6.deb`, foi desenvolvido para o JetPack 6 e processa na GPU do Jetson com a versão CUDA do PyTorch do JetPack. No Linux, cada imagem exportada agora chega ao disco uma única vez (gravada e marcada na memória e, em seguida, copiada para o local): em um Jetson, onde o disco determina o ritmo, uma execução do LATTICE com 288 imagens leva 14 minutos em vez de 24. Consulte [Instalação no Linux](linux/linux-installation.md).
* **As câmeras LATTICE OCN apresentam leituras mais precisas sob luz solar.** Radiação, refletância e índices recebem uma correção rápida após a separação de bandas, de modo que um alvo cinza apresenta uma leitura próxima ao NDVI neutro sob o sol. As câmeras RGN permanecem inalteradas.
* **Relatórios de bugs no aplicativo.** Um ícone de bug na parte superior da aba “Log de Depuração” envia ao MAPIR uma descrição, até 10 imagens, os logs e detalhes do sistema. Consulte [GUI: Navegação](navigation.md).
* **Matrizes LATTICE.**Por padrão, as matrizes mantêm o sensor completo, e o transporte compactado de 12 bits (`--packed`, padrão `auto`) aumenta as taxas de quadros do sensor completo em cerca de um terço; uma matriz funciona em qualquer rede na taxa de quadros que ela permitir (um link lento reduz a taxa de quadros em vez de rejeitar a matriz ou diminuir seus quadros); uma matriz cujas câmeras passam por um ciclo de reinicialização se recupera sozinha, com o PTP reativado; a exposição automática não reduz mais essa taxa em condições de pouca luz (ela preenche o período e, em seguida, aumenta o ganho); A captura contínua acompanha as matrizes rápidas; um controle de**Destaques** para câmeras RGB; e o `ArraySession` do SDK ganha o `grab()`, `frame_stream()` e `continuous()`. Consulte a [seção LATTICE](lattice/README.md).
* **Exposição automática mais estável.** A exposição permanece constante sob iluminação estável e se estabiliza em poucos segundos após uma mudança; pequenos realces brilhantes não escurecem mais a cena; uma exposição manual mantém o ganho no modo Automático; e uma câmera que deixa de responder sob sol forte é mantida no nível adequado à sua iluminação, lido pelo sensor de luz ou, na ausência de um sensor conectado, pelos próprios quadros da câmera. Consulte [Configurações da câmera](lattice/camera-settings.md).
* **Sensores de luz.** **Suporte ao DAQ-E-S:** seu difusor fixo não requer correção de tampa, e cada leitura inclui a inclinação, rotação e inclinação longitudinal da unidade. Uma gravação indica se a tampa foi selecionada ou presumida, e o processamento emite um aviso quando se trata de uma tampa presumida: o DAQ-U, o DAQ-M e o DAQ-E são fornecidos sem tampa; portanto, especifique o nome da tampa instalada. Gravações ativas do `.daq` sobrevivem a uma queda de energia (mantenha um arquivo `.daq-journal` junto com seu `.daq`), e um DAQ-E que para de transmitir é reconectado com a lacuna marcada em seu registro. Um projeto que contenha apenas gravações de sensores de luz as lista e processa a partir do aplicativo e da CLI, e uma gravação feita com o `chloros_scripts` é listada como o DAQ que a fez, com um limite que você pode definir. Cada captura salva a leitura do DAQ utilizada. Consulte a [seção DAQ](daq/README.md).
* **Processamento.**Os quadros do LATTICE iniciam o processamento durante a detecção do alvo. As referências do alvo e do DAQ são definidas por banda. Um projeto movido vincula novamente seus arquivos. As exportações de uma captura de 8 bits são em 8 bits, com um novo formato**TIFF (8 bits)**. Uma execução do Survey3 sem alvo de calibração é arquivada como seu produto alternativo (`Vignette_Corrected_Images` ou `Sensor_Response_Images`), e não como refletância. As tags DLS por banda de uma imagem de refletância são sempre as do próprio quadro. O recurso “Texture Aware” é rejeitado para capturas LATTICE, que sempre recebem o debayer padrão.
* **CLI e SDK.** Os caminhos de saída relativos agora são direcionados para a pasta a partir da qual você executa o comando ou script (um backend instalado costumava resolvê-los em sua própria pasta de instalação e falhava). O backend continua em execução entre comandos e scripts e se desliga automaticamente após 30 minutos sem uso; o `chloros-cli backend status` e o `backend stop` gerenciam isso. Os comandos de captura listam todos os arquivos que não conseguiram gravar e saem com o código 1; capturas repetidas são interrompidas quando o disco fica cheio. Uma execução bem-sucedida do `process` gera o relatório `Image products written: N (plus M clamp masks)`. No Windows, o `ChlorosLocal` do SDK / `process_folder` do SDK localizam o backend instalado (e utilizam um que já esteja em execução), e o `read_image_audit_tags` funciona a partir de um `pip install` simples.

***

## Novidades no Chloros 1.2.0

* **Controle de câmera ao vivo — nova aba “Câmeras”.** Conecte câmeras LATTICE uma de cada vez ou como conjuntos sincronizados de várias câmeras (sincronização de tempo PTP, captura acionada por hardware), com sobreposições de visualização ao vivo, histogramas por banda, exposição automática inteligente, um calculador de índice em tempo real e atualizações de firmware da câmera diretamente no aplicativo.
* **Sensores de luz — nova aba “Sensores de luz”.** Conecte sensores DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet); visualize espectros calibrados em tempo real (W/m²/nm), grave arquivos `.daq` em seu projeto, selecione perfis de correção de cap e atualize o firmware do DAQ-E pela rede.
* **Modos de captura e gravadores.** Captura única / contínua / por intervalo, além de um modo de captura mais rápida apenas para dados brutos; seleção por projeto das câmeras e dos tipos de exportação que o “Capturar tudo” produz; gravadores em matriz para vídeo indexado de nível de monitoramento e rajadas de dados brutos de nível de análise com compilações de vídeo offline.
* **Fluxo de processamento LATTICE.** Importe pastas de captura LATTICE e divida cada quadro bruto em produtos de remoção de bayering, pré-visualização, radianância float32 (W/m²/sr/nm) e refletância, com opções de ativação/desativação por produto. A refletância pode ser obtida a partir de um alvo de calibração no quadro ou da luz descendente do DAQ; o alinhamento da matriz é aplicado às exportações; a calibração de fábrica ausente é baixada automaticamente pelo número de série da câmera.
* **Os projetos lembram o hardware.** Câmeras e sensores de luz conectados são salvos com o projeto (`cameras.json` / `sensors.json`) e se reconectam com suas configurações salvas quando você reabre o projeto. Consulte [GUI: Projetos](projects.md).
* **Atualizações no visualizador de imagens.** Leitura de pixels/índice do cursor com escala de refletância correta por arquivo, histogramas de camadas, um controle deslizante de agrupamento de GSD, modos de grade “Por Gatilho” / “Por Câmera”, visualizações de produtos LATTICE e exportações de índice/LUT para o disco.
* **CLI e SDK, amplamente expandidos.** Novas famílias de comandos `lattice`, `daq pool-*`, `project` e `time-sync`; novas opções `process` (`--input-level`, alternâncias por produto, `--reflectance-source`, sinalizadores de alinhamento de matriz); identificadores de conexão inteligente do SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) que iniciam automaticamente o backend; Automação do `open_project()`; o pacote “SDK wheel” vem incluído nos instaladores e é publicado no PyPI como `chloros-sdk`.
* **Semântica de falha transparente.** Uma execução do `chloros-cli process` que solicitou produtos, mas não gerou nenhum, agora falha de forma evidente e retorna um valor diferente de zero; execuções bem-sucedidas informam quantos produtos de imagem foram gerados.
* **Novo layout de saída.** Os produtos são salvos em pastas `<project>/<camera>/<format>/<Product>_Images/` e mantêm o nome do arquivo de origem — a pasta, e não um sufixo no nome do arquivo, identifica o produto. Consulte [Formatos de imagem de saída](output-image-formats.md).
* **Mais entradas, planos e idiomas.** Suporte à entrada `.dng`; todos os 38 idiomas da interface totalmente implementados; limites de hardware por plano com uso gratuito (sem login) de até 4 câmeras e 2 sensores de luz.
* **Confiabilidade.** A opção “Interromper processamento” encerra a operação de forma limpa com um resumo honesto da execução; projetos com várias câmeras exportam todas as câmeras; e as atualizações do instalador não fazem mais com que você seja desconectado.

***

O Chloros está disponível em três interfaces de aplicativo:

## Chloros: Aplicativo GUI para desktop

Janela independente com todos os recursos, incluindo as guias “Câmeras ao vivo” e “Sensores de luz”. _Somente para Windows._

## [Chloros CLI: Interface de linha de comando](CLI.md)

Processamento em lote por linha de comando, além dos comandos em tempo real `lattice`, `daq pool-*`, `project` e `time-sync`. Perfeito para automação, criação de scripts e operação sem interface gráfica. Disponível no **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _O acesso à CLI requer um plano pago do Chloros+._

## [Chloros API: Python SDK](api-python-sdk.md)

Interface programática em Python para automação e fluxos de trabalho personalizados: processamento de pipeline completo, sessões ao vivo de câmera/matriz, sessões de sensores DAQ e automação de projetos salvos. Instalado com o pacote para desktop/CLI e também publicado como `pip install chloros-sdk`. _A API requer um plano pago do Chloros+ para acesso._

***

## Plataformas compatíveis

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Sim | Sim | Sim |
| **Linux amd64 (x86_64)** | Não | Sim | Sim |
| **Linux arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação no Linux, consulte a seção [Linux e computação de borda](linux/linux-overview.md).

***

## Comece em três etapas

1. **Instale** — baixe e execute o instalador para sua plataforma. Consulte [Download](download.md).
2. **Faça login (opcional para a GUI)** — a GUI processa imagens gratuitamente, sem necessidade de conta. Um [login no Chloros+](chloros+-login.md) desbloqueia o processamento paralelo, a aceleração por GPU, limites mais altos para dispositivos e acesso à CLI e ao SDK.
3. **Crie seu primeiro projeto** — abra o Chloros, crie um [Novo Projeto](projects.md), [adicione suas imagens](processing-images-gui/adding-files-to-a-project.md) e [inicie o processamento](processing-images-gui/starting-the-processing.md). Para controlar hardware em tempo real, abra a aba Câmeras ou Sensores de luz — consulte [GUI: Navegação](navigation.md).

***

## Chloros+

Embora o Chloros seja gratuito para a maioria das tarefas, talvez você perceba que precisa de mais recursos. É aí que uma licença paga do Chloros+ pode ser vantajosa para você. Com uma licença do Chloros+, você pode desbloquear novos recursos, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos maiores, processando imagens simultaneamente pelo pipeline.
* **Aceleração por GPU (CUDA)**: aproveite as opções atuais de maior memória de GPU para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Chloros+**[**CLI**](CLI.md) **Acesso**: execute o Chloros+ a partir da linha de comando para automatizar e integrar ao seu próprio software. Disponível em qualquer plano pago; aplicado no lado do servidor.
* **Chloros+**[**API**](api-python-sdk.md) **Acesso:** execute o Chloros+ a partir do Python para controle programático, permitindo integração perfeita com seus fluxos de pesquisa, fluxos de trabalho de análise de dados e aplicativos personalizados. Disponível em qualquer plano pago; aplicado no lado do servidor.
* **Limites de hardware mais altos**: conecte mais câmeras e sensores de luz ao mesmo tempo. Sem login, a interface gráfica (GUI) conecta até 4 câmeras e 2 sensores de luz DAQ; os planos pagos aumentam ambos os limites:

| Plano | Câmeras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sem login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uso em vários dispositivos**: cada licença do Chloros+ permite o registro de 2 ou mais dispositivos. Use sua conta do MAPIR Cloud para gerenciar os dispositivos registrados. Adicione suporte para mais dispositivos atualizando sua licença do Chloros+.
* **Método avançado de debayer com reconhecimento de textura:** um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que remove praticamente todo o ruído do processo de debayer.
* **Fórmulas personalizadas de índices multiespectrais:** insira índices multiespectrais personalizados nas calculadoras de raster do Chloros, tanto para processamento quanto para a área de teste de visualização de imagens.
* **Linux e computação de borda:** execute o Chloros em plataformas Linux x86_64 e ARM64, incluindo o NVIDIA Jetson, para processamento em campo e de borda. Consulte a [Visão geral do Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preços e inscrição no Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.3.0 build so the banner prints "Chloros CLI 1.3.0". -->
