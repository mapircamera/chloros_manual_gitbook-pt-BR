# GUI: Navegação

Quando você inicia o Chloros e o Chloros (navegador) pela primeira vez, o backend é iniciado. Quando estiver pronto, o ícone do menu principal no canto superior esquerdo será exibido. <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

No menu principal, você pode iniciar um novo projeto, abrir um projeto existente ou abrir a pasta do projeto.

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Botão Reproduzir/Iniciar

Quando ativado, o botão Iniciar processamento inicia o pipeline de processamento de imagens.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de progresso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">No modo gratuito Chloros, que processa todos os arquivos sequencialmente, a barra de progresso mostrará duas etapas: Detecção do alvo e Processamento.

No modo licenciado pago Chloros+, que processa todos os arquivos simultaneamente, a barra de progresso mostra quatro etapas: Detecção, Análise, Calibração e Exportação. Se você passar o cursor do mouse sobre a barra de progresso Chloros+, um painel estendido com quatro barras de progresso será exibido para que você possa acompanhar o processo. Clicar na barra de progresso superior congelará o painel suspenso; clicar novamente o descongelará.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones para interagir:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configurações do projeto](project-settings/project-settings.md)

A guia Configurações do projeto permite ajustar as configurações globais e de processamento do projeto. Ajuste-as antes de iniciar o processamento dos seus arquivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de arquivos

Adicione arquivos/pastas e remova arquivos do projeto. Arquivos duplicados são ignorados. Marque a caixa da coluna de destino para qualquer imagem de destino, e o processamento analisará apenas as imagens marcadas como destinos, acelerando consideravelmente o tempo de processamento. Use o botão Imagem/Metadados para alternar entre a visualização da grade de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando uma imagem é clicada no visualizador de imagens principal, ela é aberta em tela cheia na guia Visualizador de imagens.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Visualize suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Compatível com Google Maps e provedores de blocos ESRI, selecionando automaticamente o melhor serviço para sua localização. Passe o mouse sobre os marcadores para ver as visualizações em miniatura das imagens.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro de depuração

Revise o registro para impressões de depuração quando ocorrerem problemas. Copie/baixe o registro e envie para o [Suporte MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Login do usuário](chloros+-login.md)

A barra lateral de login do usuário permite que você faça login na sua conta Chloros+ para desbloquear recursos avançados. Você também pode visualizar a versão atual do aplicativo, bem como ajustar o idioma do texto exibido na GUI Chloros e no CLI.
