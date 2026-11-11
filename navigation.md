# GUI: Navegação

Ao iniciar o Chloros pela primeira vez, ele inicia seu backend de processamento. Assim que o backend estiver pronto, o ícone do menu principal no canto superior esquerdo é exibido <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> e as guias “Câmeras” e “Sensores de luz” ficam ativas na barra lateral esquerda (elas permanecem desativadas até esse momento).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>No menu principal, você pode:

* **Novo Projeto**— criar um novo projeto. Se você tiver modelos de projeto salvos, um menu suspenso**Selecionar Modelo** será exibido para que o novo projeto comece com as configurações de um modelo.
* **Abrir Projeto**— abrir um projeto existente. A lista inclui um botão**Abrir pasta do projeto** que abre a pasta de projetos no seu explorador de arquivos.
* **Duplicar projeto** — copia o projeto atualmente aberto com um novo nome (sugere-se um nome livre, como “MeuProjeto (2)”) e abre a cópia. _(visível após a abertura de um projeto)_
* **Adicionar arquivos** — adiciona arquivos de imagem individuais ao projeto atual _(visível após a abertura de um projeto)_
* **Adicionar pasta** — adiciona uma ou mais pastas de imagens ao projeto atual _(visível após a abertura de um projeto)_
* **Iniciar processamento / Parar processamento** — inicia ou interrompe o fluxo de processamento de imagens _(ativado após a adição de arquivos)_
* **Conectar à câmera** — vai para a [guia Câmeras](lattice/) para conectar uma câmera ou matriz LATTICE. Funciona sem um projeto aberto.
* **Conectar ao sensor de luz** — acesse a [guia Sensores de luz](daq/) para conectar um sensor de luz DAQ. Funciona mesmo sem um projeto aberto.

{% hint style="info" %}
**Somente Windows**: A interface gráfica do Chloros

Desktop está disponível no Windows

. Usuários do Linux

devem consultar a documentação da [CLI

](CLI.md) e do [Python

SDK

](api-python-sdk.md) para processamento sem interface gráfica.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">

Botão Reproduzir/Iniciar

Quando ativado, o botão de início do processamento inicia o fluxo de processamento de imagens.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line">

Barra de progresso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">



No modo gratuito do Chloros

, que processa todos os arquivos sequencialmente, a barra de progresso exibirá duas etapas: Detecção de alvo e Processamento.

No modo pago do Chloros

+, que processa todos os arquivos simultaneamente, a barra de progresso exibe quatro etapas: Detecção, Análise, Calibração e Exportação. Se você passar o cursor do mouse sobre a barra de progresso do Chloros

+, será exibido um painel suspenso com as quatro etapas da barra de progresso para que você possa acompanhar o andamento. Clicar na barra de progresso superior congela o painel suspenso; clicar novamente o descongela.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu Lateral

O menu da barra lateral esquerda contém vários ícones com os quais você pode interagir, nesta ordem, de cima para baixo:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configurações do Projeto](project-settings/project-settings.md)

A aba “Configurações do Projeto” permite ajustar as configurações globais do projeto e as de processamento. Ajuste essas configurações antes de iniciar o processamento dos seus arquivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de arquivos

Adicione arquivos/pastas e remova arquivos do projeto. Arquivos duplicados são ignorados. Marque a caixa da coluna “Destino” para qualquer imagem de destino, e o processamento analisará apenas as imagens marcadas como destinos, acelerando significativamente o tempo de processamento. Use o botão de alternância “Imagem/Metadados” para alternar entre a visualização da grade de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de Imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando uma imagem é clicada no visualizador de imagens principal, ela é aberta em tela cheia na guia “Visualizador de Imagens”.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visualizador de Mapas](image-viewer-gui/map-markers.md)

Visualize suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Compatível com os provedores de mosaicos do Google Maps e da ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o mouse sobre os marcadores para ver pré-visualizações em miniatura das imagens.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Câmeras](lattice/)

Conecte e controle câmeras LATTICE ao vivo — uma de cada vez ou como conjuntos sincronizados de várias câmeras. A aba exibe blocos de visualização ao vivo com sobreposições e histogramas, configurações por câmera e por conjunto, além das Configurações de Captura que determinam quais câmeras e tipos de exportação a opção “Capturar tudo” gera. Disponível assim que o backend estiver pronto; consulte a [seção LATTICE](lattice/) para o passo a passo completo.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensores de luz](daq/)

Conecte sensores de luz DAQ — DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet) — e visualize seus gráficos de espectro calibrados em tempo real em W/m²/nm. A partir daqui, você pode gravar arquivos `.daq` no projeto aberto, renomear sensores, selecionar perfis de correção de teto e atualizar o firmware do DAQ-E. Disponível assim que o backend estiver pronto; consulte a [seção DAQ](daq/) para obter o passo a passo completo.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro de depuração

Analise o registro em busca de mensagens de depuração quando ocorrerem problemas. Copie/baixe o registro e envie para o [Suporte da MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

Novidade na versão 1.3.0: o ícone de bug na parte superior da aba de log (**Relatar um bug**) envia à MAPIR um relatório com sua descrição, até 10 imagens, os logs do Chloros e detalhes do sistema. É necessário estar conectado para enviar um relatório. Consulte [Como relatar um bug a partir da aba de log](processing-images-gui/monitoring-the-processing.md#reporting-a-bug-from-the-log-tab).

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Login do usuário](chloros+-login.md)

A barra lateral de login do usuário permite que você faça login na sua conta do Chloros+ para desbloquear recursos avançados. Você também pode visualizar a versão atual do aplicativo, bem como ajustar o idioma do texto exibido na interface gráfica (GUI) e na linha de comando (CLI) do Chloros.
