# Interface gráfica (GUI): Projetos

O Chloros permite criar projetos que podem ser reabertos no futuro. Um projeto é uma pasta comum (dentro da sua Pasta de Projetos) que contém:

* `project.json` — configurações do projeto, lista de arquivos e preferências de exibição
* `cameras.json` — câmeras e matrizes conectadas enquanto o projeto estava aberto, com suas configurações
* `sensors.json` — sensores de luz DAQ conectados enquanto o projeto estava aberto, além das associações entre câmeras e sensores
* suas capturas, gravações `.daq` e pastas de saída processadas

Não há um formato proprietário de arquivo de projeto — a pasta e seus arquivos JSON constituem o projeto, o que também facilita copiar, arquivar e acessar os projetos a partir do [CLI](CLI.md) ou [Python SDK](api-python-sdk.md).

## Novo Projeto

<figure><img src=".gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>Selecione “Novo Projeto” no menu principal e insira um nome exclusivo para o seu projeto.

Se você tiver salvo algum modelo de projeto, um menu suspenso **Selecionar Modelo** aparecerá abaixo do campo de nome — ao escolher um, o novo projeto será iniciado com as configurações desse modelo. Os modelos são salvos em [Configurações do projeto](project-settings/project-settings.md): digite um nome no campo “Nome do modelo do projeto” e clique no ícone de salvar.

## Abrir projeto

<figure><img src=".gitbook/assets/v120-open-project.jpg" alt=""><figcaption><p>A opção “Abrir projeto” lista todos os projetos na sua pasta de projetos, com a opção <strong>“Abrir pasta de projetos”</strong> na parte inferior</p></figcaption></figure>Selecione “Abrir projeto” para ver uma lista dos projetos existentes na pasta de projetos. Se não houver projetos, o menu lateral secundário não será exibido. Você pode ver alguns projetos criados pela GUI (t1, t2, t3) listados na foto acima. Os projetos DATE\_TIME foram criados pelo CLI usando o esquema padrão de nomenclatura de projetos. Clicar em qualquer nome de projeto irá abri-lo.

Clicar no botão “Abrir pasta do projeto” abre o explorador de arquivos do seu computador no caminho do projeto. Você pode ajustar o caminho do projeto em [Configurações do projeto](project-settings/project-settings.md).

Se algum dos arquivos de imagem de origem do projeto tiver sido movido ou excluído desde a última vez em que foi aberto, o Chloros exibe uma caixa de diálogo listando exatamente quais arquivos estão faltando, em vez de abrir uma grade em branco.

## Duplicar projeto

Disponível assim que o projeto for aberto. Selecione “Duplicar projeto” para copiar o projeto atual com um novo nome — o Chloros sugere o próximo nome disponível (por exemplo, “MeuProjeto (2)”) — e a duplicata é aberta imediatamente.

## Adicionar arquivos

Depois que um projeto for aberto, selecione “Adicionar arquivos” no menu principal para adicionar arquivos de imagem individuais ao projeto atual. Isso funciona da mesma forma que a funcionalidade de adição do navegador de arquivos, mas pode ser acessado diretamente pelo menu principal para maior conveniência.

## Adicionar pasta

Depois que um projeto for aberto, selecione “Adicionar pasta” no menu principal para adicionar pastas de imagens ao projeto atual. Você pode selecionar várias pastas de uma só vez. Arquivos duplicados são ignorados.

## Iniciar/Interromper o processamento

Depois que os arquivos forem adicionados a um projeto, a opção “Iniciar processamento” fica disponível no menu principal. Essa ação é equivalente a clicar no botão Reproduzir/Iniciar no cabeçalho superior. Durante o processamento, o item do menu muda para “Interromper processamento” para permitir que você pare o fluxo de trabalho.

## Conectar à Câmera / Conectar ao Sensor de Luz

Na parte inferior do menu principal há dois atalhos de hardware, disponíveis com ou sem um projeto aberto:

* **Conectar à Câmera** — abre a [guia Câmeras](lattice/) para conectar uma câmera ou matriz LATTICE.
* **Conectar ao sensor de luz** — abre a [guia Sensores de luz](daq/) para conectar um sensor de luz DAQ.

Conectar o hardware enquanto um projeto está aberto salva a conexão no projeto (veja abaixo). Sem um projeto, as conexões são válidas apenas para a sessão.

{% hint style="info" %}
Os itens de menu “Adicionar arquivos”, “Adicionar pasta” e “Iniciar/Parar processamento” só ficam visíveis ou ativados quando um projeto está aberto e os arquivos foram adicionados. Eles oferecem acesso rápido a ações também disponíveis na barra lateral do Navegador de arquivos e nos botões do cabeçalho.
{% endhint %}

## Os projetos lembram-se do seu hardware

Novidade na versão 1.2.0: um projeto mantém o hardware conectado enquanto estiver aberto. Câmeras e matrizes (com suas configurações por câmera, nomes, cores e layout de grade) são salvas no `cameras.json`, e sensores de luz (com nomes, cores e associações de câmera) no `sensors.json` — automaticamente, à medida que você trabalha.

Quando você **reabre** um projeto, o Chloros não acessa imediatamente nenhum hardware. Cada metade se reconecta na primeira vez que você acessa a aba à qual pertence:

* Abrir a aba **Câmeras** reconecta as câmeras e matrizes salvas e reaplica suas configurações salvas.
* Abrir a aba **Sensores de Luz** reconecta os sensores DAQ salvos.

Dessa forma, abrir um projeto apenas para visualizar ou exportar imagens nunca ativa as câmeras para transmissão. Se um dispositivo salvo não for encontrado ao abrir sua aba, uma caixa de diálogo informa quais dispositivos estão indisponíveis para que você possa reconectá-los ou removê-los.

## Gravações DAQ e arquivos .daq em um projeto

* As gravações `.daq` feitas enquanto o projeto está aberto (na aba **Sensores de Luz**ou durante as capturas) são**adicionadas automaticamente ao projeto**.
* Os arquivos `.daq` importados e todas as gravações do projeto são listados na seção **Sensor de Luz DAQ** das [Configurações do Projeto](project-settings/project-settings.md), cada um com seu perfil de correção de cap.
* Durante o processamento, os arquivos `.daq` do projeto fornecem iluminação descendente para produtos de refletância — consulte [Formatos de imagem de saída](output-image-formats.md).

## Execução de um projeto salvo sem interface gráfica

Um projeto salvo pode ser executado sem a interface gráfica (GUI):

* **CLI**: O `chloros-cli project open / connect / capture / sensor / align / run` opera com o caminho da pasta do projeto — consulte a [Referência do CLI](reference/cli-reference.md).
* **SDK**: `chloros_sdk.open_project(path)` retorna um identificador do projeto; `connect_all()` coloca todas as câmeras e sensores salvos em operação com suas configurações salvas — consulte a [Referência do SDK](reference/sdk-reference.md).
