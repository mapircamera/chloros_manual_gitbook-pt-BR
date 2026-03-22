# GUI: Navegação

Ao iniciar o Chloros e o Chloros (Navegador) pela primeira vez, o backend será iniciado. Assim que estiver pronto, o ícone do menu principal no canto superior esquerdo será exibido <img src=".gitbook/assets/image (1) (1) (1).png" alt="" data-size="line"> .

<figure><img src=".gitbook/assets/header.JPG" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

<figure><img src=".gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

No menu principal, você pode:

* **Novo Projeto** — criar um novo projeto
* **Abrir Projeto** — abrir um projeto existente
* **Abrir Pasta do Projeto** — abrir a pasta do projeto no seu explorador de arquivos
* **Adicionar Arquivos** — adicionar arquivos de imagem individuais ao projeto atual _(visível após a abertura de um projeto)_
* **Adicionar Pasta** — adicionar uma pasta de imagens ao projeto atual _(visível após a abertura do projeto)_
* **Iniciar Processamento / Parar Processamento** — iniciar ou parar o fluxo de processamento de imagens _(ativado após a adição dos arquivos)_

{% hint style="info" %}
**Apenas Windows**: A interface gráfica do Chloros está disponível no Windows. Os usuários do Linux devem consultar o [CLI](CLI.md) e [Python SDK](api-python-sdk.md) para processamento sem interface gráfica.
{% endhint %}

### <img src=".gitbook/assets/image (2) (1).png" alt="" data-size="line"> Botão Reproduzir/Iniciar

Quando ativado, o botão Iniciar processamento inicia o fluxo de processamento de imagens.

### <img src=".gitbook/assets/image (4).png" alt="" data-size="line"> Barra de progresso <img src=".gitbook/assets/image (5).png" alt="" data-size="line">No modo gratuito Chloros, que processa todos os arquivos sequencialmente, a barra de progresso exibirá 2 etapas: Detecção de Alvo e Processamento.

No modo licenciado Chloros+, que processa todos os arquivos simultaneamente, a barra de progresso exibe 4 etapas: Detecção, Análise, Calibração e Exportação. Se você passar o cursor do mouse sobre a barra de progresso do Chloros+, um painel suspenso com as 4 etapas da barra de progresso será exibido para que você possa acompanhar o andamento. Clicar na barra de progresso superior congela o painel suspenso; clicar novamente o descongela.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones para interação:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configurações do projeto](project-settings/project-settings.md)

A guia Configurações do projeto permite ajustar as configurações globais e de processamento do projeto. Ajuste-as antes de iniciar o processamento dos seus arquivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de arquivos

Adicione arquivos/pastas e remova arquivos do projeto. Arquivos duplicados são ignorados. Marque a caixa da coluna de destino para qualquer imagem de destino, e o processamento analisará apenas as imagens marcadas como destinos, acelerando significativamente o tempo de processamento. Use o botão de alternância Imagem/Metadados para alternar entre a visualização da grade de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de Imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando uma imagem é clicada no visualizador de imagens principal, ela é aberta em tela cheia na guia Visualizador de Imagens.

#### <img src=".gitbook/assets/image (7).png" alt="" data-size="line"> [Mapa](image-viewer-gui/map-markers.md)

Visualize suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Compatível com os provedores de mapas Google Maps e ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o mouse sobre os marcadores para ver pré-visualizações das miniaturas das imagens.

#### <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line"> Registro de Depuração

Analise o registro para ver as saídas de depuração quando ocorrerem problemas. Copie/baixe o registro e envie para o [Suporte MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> [Login do usuário](chloros+-login.md)

A barra lateral de login do usuário permite que você faça login na sua conta Chloros+ para desbloquear recursos avançados. Você também pode visualizar a versão atual do aplicativo, bem como ajustar o idioma do texto exibido na interface gráfica do Chloros e no CLI.
