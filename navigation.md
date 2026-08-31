# GUI: Navegação

Ao iniciar o Chloros pela primeira vez, ele inicia o backend de processamento. Assim que o backend estiver pronto, o ícone do menu principal no canto superior esquerdo é exibido <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> e as guias “Câmeras” e “Sensores de luz” ficam ativas na barra lateral esquerda (elas permanecem desativadas até esse momento).

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Da esquerda para a direita, o cabeçalho superior contém:

### <img src=".gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> Menu principal

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

No menu principal, você pode:

* **Novo Projeto**— criar um novo projeto. Se você tiver modelos de projeto salvos, um menu suspenso**Selecionar Modelo** é exibido para que o novo projeto comece com as configurações de um modelo.
* **Abrir Projeto**— abrir um projeto existente. A lista inclui um botão**Abrir pasta do projeto** que abre a pasta dos projetos no seu explorador de arquivos.
* **Duplicar projeto** — copia o projeto atualmente aberto com um novo nome (sugere-se um nome livre, como “MeuProjeto (2)”) e abre a cópia. _(visível após a abertura de um projeto)_
* **Adicionar arquivos** — adiciona arquivos de imagem individuais ao projeto atual _(visível após a abertura de um projeto)_
* **Adicionar pasta** — adiciona uma ou mais pastas de imagens ao projeto atual _(visível após a abertura de um projeto)_
* **Iniciar processamento / Parar processamento** — inicia ou interrompe o fluxo de processamento de imagens _(ativado após a adição de arquivos)_
* **Conectar à câmera** — vai para a [guia Câmeras](lattice/) para conectar uma câmera ou matriz LATTICE. Funciona mesmo sem um projeto aberto.
* **Conectar ao sensor de luz** — acesse a [guia Sensores de luz](daq/) para conectar um sensor de luz DAQ. Funciona mesmo sem um projeto aberto.

{% hint style="info" %}
**Somente Windows**: A interface gráfica de usuário (GUI) do Chloros para desktop está disponível no Windows. Os usuários do Linux devem consultar a documentação do [CLI](CLI.md) e [Python SDK](api-python-sdk.md) para obter informações sobre o processamento sem interface gráfica.
{% endhint %}

### Botão “Reproduzir/Iniciar” do<img src=".gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">



Quando ativado, o botão “Iniciar processamento” inicia o fluxo de processamento de imagens.

### Barra de progresso do<img src=".gitbook/assets/image (4).png" alt="" data-size="line">

<img src=".gitbook/assets/image (5).png" alt="" data-size="line">

No modo gratuito do Chloros, que processa todos os arquivos sequencialmente, a barra de progresso exibirá duas etapas: Detecção de alvo e Processamento.

No modo Chloros+ licenciado, que processa todos os arquivos simultaneamente, a barra de progresso exibe 4 etapas: Detecção, Análise, Calibração e Exportação. Se você passar o cursor do mouse sobre a barra de progresso do Chloros+, será exibido um painel suspenso com as quatro etapas da barra de progresso para que você possa acompanhar o andamento. Clicar na barra de progresso superior congela o painel suspenso; clicar novamente o descongela.

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

## Menu lateral

O menu da barra lateral esquerda contém vários ícones com os quais você pode interagir, nesta ordem, de cima para baixo:

#### <img src=".gitbook/assets/icon_project-settings.JPG" alt="" data-size="line"> [Configurações do projeto](project-settings/project-settings.md)

A aba “Configurações do projeto” permite ajustar as configurações globais do projeto e as configurações de processamento. Ajuste essas configurações antes de iniciar o processamento dos seus arquivos.

#### <img src=".gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> Navegador de arquivos

Adicione arquivos/pastas e remova arquivos do projeto. Arquivos duplicados são ignorados. Marque a caixa da coluna “Destino” para qualquer imagem de destino, e o processamento analisará apenas as imagens marcadas como destinos, acelerando significativamente o tempo de processamento. Use o botão de alternância “Imagem/Metadados” para alternar entre a visualização da grade de miniaturas da imagem selecionada e uma tabela detalhada de metadados.

#### <img src=".gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> [Visualizador de Imagens](image-viewer-gui/opening-an-image-full-screen.md)

Quando uma imagem é clicada no visualizador de imagens principal, ela é aberta em tela cheia na guia Visualizador de Imagens.

#### <img src=".gitbook/assets/image (3) (1).png" alt="" data-size="line"> [Visualizador de Mapas](image-viewer-gui/map-markers.md)

Visualize suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Compatível com os provedores de blocos do Google Maps e da ESRI, selecionando automaticamente o melhor serviço para a sua localização. Passe o mouse sobre os marcadores para ver pré-visualizações em miniatura das imagens.

#### <img src=".gitbook/assets/image (17).png" alt="" data-size="line"> [Câmeras](lattice/)

Conecte e controle câmeras LATTICE ao vivo — uma de cada vez ou como conjuntos sincronizados de várias câmeras. A aba exibe blocos de visualização ao vivo com sobreposições e histogramas, configurações por câmera e por conjunto, além das Configurações de Captura que determinam quais câmeras e tipos de exportação a opção “Capturar tudo” gera. Disponível assim que o backend estiver pronto; consulte a [seção LATTICE](lattice/) para o passo a passo completo.

#### <img src=".gitbook/assets/image (23).png" alt="" data-size="line"> [Sensores de luz](daq/)

Conecte sensores de luz DAQ — DAQ-U (USB), DAQ-M (Bluetooth) e DAQ-E (Ethernet) — e visualize seus gráficos de espectro calibrados em tempo real em W/m²/nm. A partir daqui, você pode gravar arquivos `.daq` no projeto aberto, renomear sensores, selecionar perfis de correção de capacitância e atualizar o firmware do DAQ-E. Disponível assim que o backend estiver pronto; consulte a [seção DAQ](daq/) para obter o passo a passo completo.

#### Log de depuração do <img src=".gitbook/assets/icon_log.JPG" alt="" data-size="line">

Verifique o log em busca de mensagens de depuração quando ocorrerem problemas. Copie/baixe o log e envie para o [Suporte do MAPIR](https://www.mapir.camera/community/contact) para obter assistência.

#### [Login do usuário](chloros+-login.md) no <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line">

A barra lateral de login do usuário permite que você faça login na sua conta Chloros+ para desbloquear recursos avançados. Você também pode visualizar a versão atual do aplicativo, bem como ajustar o idioma do texto exibido na interface gráfica do Chloros e no CLI.
