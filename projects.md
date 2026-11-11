# GUI: Projetos O

Chloros permite que você crie projetos que possam ser reabertos no futuro. Um projeto é uma pasta comum (dentro da sua Pasta de Projetos) que contém:

* `project.json` — configurações do projeto, lista de arquivos e preferências de exibição
* `cameras.json` — câmeras e matrizes conectadas enquanto o projeto estava aberto, com suas configurações
* `sensors.json` — sensores de luz DAQ conectados enquanto o projeto estava aberto, além das associações entre câmeras e sensores
* suas capturas, gravações `.daq` e pastas de saída processadas

Não há um formato proprietário de arquivo de projeto — a pasta e seus arquivos JSON constituem o projeto, o que também facilita copiar, arquivar e executar projetos a partir da [CLI](CLI.md) ou do [Python SDK](api-python-sdk.md).

A menos que você altere isso nas [Configurações do projeto](project-settings/project-settings.md), a pasta do projeto é `Documents\MAPIR\Chloros Projects` na sua pasta Documentos (`~/Documents/MAPIR/Chloros Projects` no Linux).

{% hint style="warning" %}
**Windows: mantenha as pastas de projeto curtas, a menos que os caminhos longos estejam ativados.**O Windows vem com os caminhos longos desativados; portanto, nenhum caminho de arquivo pode ter mais de 260 caracteres. Os produtos processados ficam até cerca de 130 caracteres abaixo da pasta do projeto; portanto, a partir da versão 1.3.0, o Chloros rejeita uma pasta de projeto que seja muito profunda — ao criá-la, abri-la ou iniciar o processamento — em vez de executar e não gravar nada. A mensagem indica a pasta de projeto mais longa que cabe. Use uma pasta de projeto mais curta ou ative os caminhos longos do Windows (a opção**Ativar caminhos longos do Windows** do instalador, marcada por padrão) e reinicie o Chloros. Consulte [Iniciando o processamento](processing-images-gui/starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer).
{% endhint %}

## Novo projeto

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecione “Novo projeto” no menu principal e insira um nome exclusivo para o seu projeto.

Se você tiver salvo algum modelo de projeto, um menu suspenso **Selecionar modelo** aparecerá abaixo do campo de nome — ao escolher um, o novo projeto será iniciado com as configurações desse modelo. Os modelos são salvos em [Configurações do projeto](project-settings/project-settings.md): digite um nome no campo “Nome do modelo de projeto” e clique no ícone de salvar.

## Abrir projeto

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>A opção “Abrir projeto” lista todos os projetos na sua pasta de projetos, com a opção <strong>Abrir pasta de projetos</strong> na parte inferior</p></figcaption></figure>

Selecione “Abrir projeto” para ver uma lista dos projetos existentes na pasta de projetos. Se não houver projetos, o menu lateral secundário não será exibido. Você pode ver alguns projetos criados pela interface gráfica (t1, t2, t3) listados na foto acima. Os projetos DATE\_TIME foram criados pela CLI usando o esquema padrão de nomenclatura de projetos. Clicar em qualquer nome de projeto irá abri-lo.

Clicar no botão “Abrir pasta do projeto” abre o explorador de arquivos do seu computador no caminho do projeto. Você pode ajustar o caminho do projeto em [Configurações do projeto](project-settings/project-settings.md).

Se algum dos arquivos de imagem de origem do projeto tiver sido movido ou excluído desde a última vez em que foi aberto, o Chloros exibe uma caixa de diálogo listando exatamente quais arquivos estão faltando, em vez de abrir uma grade em branco.

## Duplicar projeto

Disponível assim que o projeto estiver aberto. Selecione “Duplicar projeto” para copiar o projeto atual com um novo nome — o Chloros sugere o próximo nome disponível (por exemplo, “MeuProjeto (2)”) — e a duplicata é aberta imediatamente.

## Adicionar arquivos

Depois que um projeto for aberto, selecione “Adicionar arquivos” no menu principal para adicionar arquivos de imagem individuais ao projeto atual. Isso funciona da mesma forma que a funcionalidade de adição do navegador de arquivos, mas pode ser acessado diretamente pelo menu principal para maior conveniência.

## Adicionar pasta

Depois que um projeto for aberto, selecione “Adicionar pasta” no menu principal para adicionar pastas de imagens ao projeto atual. Você pode selecionar várias pastas de uma só vez. Arquivos duplicados são ignorados.

## Iniciar/Interromper o processamento

Depois que os arquivos forem adicionados a um projeto, a opção “Iniciar o processamento” fica disponível no menu principal. Essa ação é equivalente a clicar no botão Reproduzir/Iniciar no cabeçalho superior. Durante o processamento, o item do menu muda para “Interromper o processamento” para permitir que você pare o fluxo de trabalho.

## Conectar à câmera / Conectar ao sensor de luz

Na parte inferior do menu principal, há dois atalhos de hardware, disponíveis com ou sem um projeto aberto:

* **Conectar à câmera** — abre a [aba Câmeras](lattice/) para conectar uma câmera ou matriz LATTICE.
* **Conectar ao sensor de luz** — abre a [guia Sensores de luz](daq/) para conectar um sensor de luz DAQ.

Conectar o hardware enquanto um projeto está aberto salva a conexão no projeto (veja abaixo). Sem um projeto, as conexões são válidas apenas para a sessão.

{% hint style="info" %}
Os itens de menu “Adicionar arquivos”, “Adicionar pasta” e “Iniciar/Parar processamento” só ficam visíveis ou habilitados quando um projeto está aberto e os arquivos foram adicionados. Eles oferecem acesso rápido a ações também disponíveis na barra lateral do Navegador de arquivos e nos botões do cabeçalho.
{% endhint %}

## Os projetos lembram seu hardware

Novidade na versão 1.2.0: um projeto mantém o hardware conectado enquanto estiver aberto. Câmeras e matrizes (com suas configurações por câmera, nomes, cores e layout de grade) são salvas em `cameras.json`, e sensores de luz (com nomes, cores e associações de câmera) em `sensors.json` — automaticamente, à medida que você trabalha.

Quando você **reabre** um projeto, o Chloros não acessa imediatamente nenhum hardware. Cada parte se reconecta na primeira vez que você acessa a aba correspondente:

* Abrir a aba **Câmeras** reconecta as câmeras e matrizes salvas e reaplica suas configurações salvas.
* Abrir a aba **Sensores de Luz**reconecta os sensores DAQ salvos e reaplica suas configurações salvas, incluindo a tampa. Um sensor configurado como**Nenhum (sensor sem capa)** no menu Capa permanece sem capa. Um projeto salvo por uma versão anterior do Chloros registrou um sensor sem capa como a única opção disponível; portanto, ele retorna com a correção do cosseno da luz solar; configure-o como Nenhum mais uma vez e salve o projeto.

Dessa forma, abrir um projeto apenas para visualizar ou exportar imagens nunca ativa as câmeras para transmissão. Se um dispositivo salvo não for encontrado ao abrir sua aba, uma caixa de diálogo informa quais dispositivos estão indisponíveis para que você possa reconectá-los ou removê-los.

## Gravações DAQ e arquivos .daq em um projeto

* As gravações `.daq` feitas enquanto o projeto está aberto (na aba “Sensores de Luz” ou durante as capturas) são **adicionadas automaticamente ao projeto**.
* Os arquivos `.daq` importados e todas as gravações do projeto são listados na seção **Sensor de Luz DAQ** das [Configurações do Projeto](project-settings/project-settings.md), cada um com seu perfil de correção de cap.
* Durante o processamento, os arquivos `.daq` do projeto fornecem iluminação descendente para produtos de refletância — consulte [Formatos de imagem de saída](output-image-formats.md).

## Executando um projeto salvo sem interface gráfica

Um projeto salvo pode ser executado sem a interface gráfica (GUI):

* **CLI**: o `chloros-cli project open / connect / capture / sensor / align / run` opera com o caminho da pasta do projeto — consulte a [Referência da CLI](reference/cli-reference.md).
* **SDK**: o `chloros_sdk.open_project(path)` retorna um identificador de projeto; `connect_all()` coloca todas as câmeras, matrizes e sensores de luz salvos em operação com suas configurações salvas (sensores de luz por meio de um backend Chloros em execução) — consulte a [Referência do SDK](reference/sdk-reference.md).
