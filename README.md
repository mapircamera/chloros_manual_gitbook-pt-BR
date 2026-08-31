---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>O Chloros é um aplicativo de software da [MAPIR](https://www.mapir.camera) para processar imagens multiespectrais, controlar em tempo real o hardware MAPIR e registrar dados de sensores. O Chloros 1.2.0 oferece suporte a toda a família de produtos MAPIR:

* **Câmeras Survey3** — processam capturas em formato RAW+JPG, transformando-as em mapas calibrados de refletância e índices de vegetação. Consulte [Câmeras compatíveis](supported-cameras.md).
* **Câmeras LATTICE** — conecte os módulos de câmera multiespectral GigE em tempo real, individualmente ou como conjuntos sincronizados de várias câmeras: visualize, capture e processe em produtos calibrados de radiação e refletância. Consulte a [seção LATTICE](lattice/README.md).
* **Sensores de luz DAQ** — sensores espectrais DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet): espectros calibrados em tempo real, gravações `.daq` e iluminação descendente para processamento de refletância. Consulte a [seção DAQ](daq/README.md).

{% hint style="success" %}
**Novidades no Chloros 1.2.0**: controle em tempo real da câmera e da matriz LATTICE, integração do sensor de luz com o DAQ, modos de captura e gravadores, um fluxo completo de processamento radiométrico do LATTICE, automação de projetos a partir do CLI/SDK e muito mais. Confira a lista de novidades abaixo e [baixe](download.md) o registro de alterações.
{% endhint %}

{% hint style="info" %}
**Está usando o Chloros com um assistente de IA?** Este manual foi feito para isso. Indique ao seu assistente:

* `https://mapir.gitbook.io/chloros/llms.txt` — índice legível por máquina de todas as páginas.
* Qualquer página como Markdown bruto — acrescente `.md` ao seu URL (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência CLI](reference/cli-reference.md) e [Referência SDK](reference/sdk-reference.md) — páginas de referência completas com valores exatos, escritas para uso por LLMs.

Exemplo de prompt: *“Leia o https://mapir.gitbook.io/chloros/reference/cli-reference.md, e, em seguida, escreva um script que faça login e processe a pasta ~/flights/flight_001 em arquivos GeoTIFF de refletância + NDVI.”*

Guia completo: [Como usar o Chloros com assistentes de IA](ai-assistants.md).
{% endhint %}

***

## Novidades no Chloros 1.2.0

* **Controle de câmera em tempo real — nova aba “Câmeras”.** Conecte câmeras LATTICE uma de cada vez ou como conjuntos sincronizados de várias câmeras (sincronização de tempo PTP, captura acionada por hardware), com sobreposições de visualização ao vivo, histogramas por banda, exposição automática inteligente, um calculador de índice em tempo real e atualizações de firmware da câmera diretamente no aplicativo.
* **Sensores de luz — nova aba “Sensores de luz”.** Conecte sensores DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet); visualize espectros calibrados em tempo real (W/m²/nm), grave arquivos `.daq` em seu projeto, selecione perfis de correção de cap e atualize o firmware do DAQ-E pela rede.
* **Modos de captura e gravadores.** Captura única / contínua / por intervalo, além de um modo de captura mais rápida (Fastest Capture) apenas para dados brutos; seleção por projeto das câmeras e dos tipos de exportação que o “Capture All” produz; gravadores em matriz para vídeo indexado de nível de monitoramento e rajadas de dados brutos de nível de análise com compilações de vídeo offline.
* **Fluxo de processamento LATTICE.** Importe pastas de captura LATTICE e divida cada quadro bruto em produtos debayered, de pré-visualização, radiação em float32 (W/m²/sr/nm) e refletância, com opções de ativação/desativação por produto. A refletância pode ser obtida a partir de um alvo de calibração no quadro ou da luz descendente do DAQ; o alinhamento da matriz é aplicado às exportações; a calibração de fábrica ausente é baixada automaticamente pelo número de série da câmera.
* **Os projetos lembram-se do hardware.** As câmeras e sensores de luz conectados são salvos com o projeto (`cameras.json` / `sensors.json`) e se reconectam com suas configurações salvas quando você reabre o projeto. Consulte [GUI: Projetos](projects.md).
* **Atualizações do visualizador de imagens.** Leitura de pixels/índice do cursor com escalonamento correto de refletância por arquivo, histogramas de camadas, um controle deslizante de agrupamento de GSD, modos de grade “Por Gatilho” / “Por Câmera”, visualizações de produtos LATTICE e exportações de índice/LUT para o disco em ambiente de teste.
* **CLI e SDK, significativamente ampliados.** Novas famílias de comandos `lattice`, `daq pool-*`, `project` e `time-sync`; novas opções do `process` (`--input-level`, opções de ativação/desativação por produto, `--reflectance-source`, sinalizadores de alinhamento de matriz); identificadores de conexão inteligente do SDK (`connect_camera` / `connect_array` / `connect_daq_sensor`) que iniciam automaticamente o backend; Automação do `open_project()`; o pacote SDK vem incluído nos instaladores e é publicado no PyPI como `chloros-sdk`.
* **Semântica de falha transparente.** Uma execução do `chloros-cli process` que solicitou produtos, mas não gerou nenhum, agora falha de forma evidente e retorna um valor diferente de zero; execuções bem-sucedidas informam quantos produtos de imagem foram gerados.
* **Novo layout de saída.** Os produtos são salvos em pastas `<project>/<camera>/<format>/<Product>_Images/` e mantêm o nome do arquivo de origem — a pasta, e não um sufixo do nome do arquivo, identifica o produto. Consulte [Formatos de imagem de saída](output-image-formats.md).
* **Mais entradas, planos e idiomas.** Suporte à entrada `.dng`; todos os 38 idiomas da interface totalmente preenchidos; limites de hardware por plano com uso gratuito (sem login) de até 4 câmeras e 2 sensores de luz.
* **Confiabilidade.** A opção “Interromper processamento” encerra a operação de forma limpa com um resumo preciso da execução; projetos com várias câmeras exportam todas as câmeras; e as atualizações do instalador não fazem mais com que você seja desconectado.***

O Chloros está disponível em três interfaces de aplicativo:

## Chloros: Aplicativo GUI para desktop

Janela independente com todos os recursos, incluindo as guias “Câmeras ao vivo” e “Sensores de luz”. _Somente para Windows._

## [Chloros CLI: Interface de linha de comando](CLI.md)

Processamento em lote por linha de comando, além dos comandos ao vivo `lattice`, `daq pool-*`, `project` e `time-sync`. Perfeito para automação, criação de scripts e operação sem interface gráfica. Disponível em **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _O CLI requer um plano pago Chloros+ para acesso._

## [Chloros API: Python SDK](api-python-sdk.md)

Interface programática Python para automação e fluxos de trabalho personalizados: processamento de pipeline completo, sessões ao vivo com câmera/matriz, sessões com sensores DAQ e automação de projetos salvos. Instalada com o pacote para desktop/CLI e também publicada como `pip install chloros-sdk`. _A API requer um plano Chloros+ pago para acesso._

***

## Plataformas compatíveis

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11 (x64)** | Sim | Sim | Sim |
| **Linux amd64 (x86_64)** | Não | Sim | Sim |
| **Linux arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação do Linux, consulte a seção [Linux e Computação de Borda](linux/linux-overview.md).

***

## Comece em três etapas

1. **Instalar** — baixe e execute o instalador para sua plataforma. Consulte [Download](download.md).
2. **Faça login (opcional para a interface gráfica)** — a interface gráfica processa imagens gratuitamente, mesmo sem uma conta. Um [login no Chloros+](chloros+-login.md) desbloqueia o processamento paralelo, aceleração por GPU, limites mais altos de dispositivos e acesso a CLI/SDK.
3. **Crie seu primeiro projeto** — abra o Chloros, crie um [Novo Projeto](projects.md), [adicione suas imagens](processing-images-gui/adding-files-to-a-project.md) e [inicie o processamento](processing-images-gui/starting-the-processing.md). Para controlar hardware em tempo real, abra a aba Câmeras ou Sensores de luz — consulte [GUI: Navegação](navigation.md).***

## Chloros+

Embora o Chloros seja gratuito para a maioria das tarefas, talvez você perceba que precisa de mais recursos. É aí que uma licença paga para o Chloros+ pode ser útil. Com uma licença do Chloros+, você pode desbloquear novos recursos, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos maiores, processando-as simultaneamente ao longo do pipeline.
* **Aceleração por GPU (CUDA)**: aproveite as opções atuais de maior memória de GPU para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Chloros+**[**CLI**](CLI.md)**Acesso**: execute o Chloros+ a partir da linha de comando para automatizar e integrar ao seu próprio software. Disponível em qualquer plano pago; aplicado no lado do servidor.
* **Chloros+**[**API**](api-python-sdk.md)**Acesso:** execute o Chloros+ a partir do Python para controle programático, permitindo integração perfeita com seus pipelines de pesquisa, fluxos de trabalho de análise de dados e aplicativos personalizados. Disponível em qualquer plano pago; aplicado no lado do servidor.
* **Limites de hardware mais altos**: conecte mais câmeras e sensores de luz ao mesmo tempo. Sem login, a interface gráfica (GUI) conecta até 4 câmeras e 2 sensores de luz DAQ; os planos pagos aumentam ambos os limites:

| Plano | Câmeras | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito, sem login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

* **Uso em vários dispositivos**: cada licença Chloros+ permite o registro de 2 ou mais dispositivos. Use sua conta MAPIR Cloud para gerenciar os dispositivos registrados. Adicione suporte para mais dispositivos atualizando sua licença Chloros+.
* **Método avançado de debayer com reconhecimento de textura:** um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que remove quase todo o ruído do processo de debayer.
* **Fórmulas personalizadas de índices multiespectrais:** insira índices multiespectrais personalizados nas calculadoras de raster do Chloros, tanto para processamento quanto para a área de teste de visualização de imagens.
* **Linux e computação de ponta:** execute o Chloros nas plataformas Linux x86_64 e ARM64, incluindo o NVIDIA Jetson, para processamento de campo e de ponta. Consulte [Visão geral do Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preços e inscrição no Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: cli.JPG shows the 1.1.0 CLI banner. Re-shoot a terminal running `chloros-cli --version` + `chloros-cli status` on the 1.2.0 build so the banner prints "Chloros CLI 1.2.0". -->
