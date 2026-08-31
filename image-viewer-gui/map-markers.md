# Marcadores do mapa

A aba “Mapa” plota suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Ela oferece uma visão geral geográfica de uma sessão de captura e é a maneira mais rápida, logo após a importação, de descartar imagens que você não deseja processar.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Acessando a aba “Mapa”

1. Abra ou crie um projeto no Chloros
2. Importe imagens que contenham metadados de GPS
3. Clique na aba **Mapa** <img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> na barra lateral esquerda
4. O mapa exibe um marcador na localização GPS de cada imagem

{% hint style="info" %}
**GPS obrigatório**: apenas imagens com coordenadas GPS em seus metadados EXIF aparecem no mapa. Uma imagem sem coordenadas continua no projeto e é processada normalmente — ela simplesmente não tem marcador.
{% endhint %}

***

## Ajustando imagens na aba “Mapa”

A aba **Mapa**<img src="../.gitbook/assets/image (3) (1).png" alt="" data-size="line"> possui os mesmos botões para adicionar <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> <img src="../.gitbook/assets/image (1) (1).png" alt="" data-size="line"> e remover <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line"> arquivos que a aba [**Navegador de arquivos**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">. Ela exibe a mesma lista de arquivos do projeto, com colunas geográficas:

| Coluna        | Conteúdo                                                           |
| ------------- | ------------------------------------------------------------------ |
| **Nome**      | O nome do arquivo tal como saiu da câmera                             |
| **Latitude**  | Graus decimais, seis casas decimais                                |
| **Longitude** | Graus decimais, seis casas decimais                                |
| **Altitude**  | Metros, uma casa decimal — `-` quando a imagem não contém altitude |

{% hint style="info" %}
Clique em qualquer cabeçalho de coluna para ordenar por ele; clique novamente para inverter a ordem.
{% endhint %}

{% hint style="warning" %}
**A altitude é a altura acima do nível do mar, não a altura acima do solo.** O valor provém da tag EXIF `GPSAltitude` da imagem, que é referenciada em relação ao nível médio do mar. Não se trata da altura de voo acima do terreno, e o Chloros não calculará a distância de amostragem do solo a partir desse valor — sobre um campo a 300 m acima do nível do mar, um drone a 100 m AGL registra aproximadamente 400 m neste campo. Use a coluna para identificar valores atípicos e confirmar uma altitude de voo consistente, não como uma medida AGL.
{% endhint %}

***

## Marcadores de imagem

Cada imagem com dados de GPS recebe um marcador em suas coordenadas.

### Exibição dos marcadores

* Os marcadores ficam nas coordenadas exatas registradas para cada captura
* Marcadores próximos uns dos outros podem se sobrepor visualmente quando se diminui o zoom — amplie a imagem para separá-los
* Os marcadores selecionados e destacados são exibidos acima dos demais

### Visualização ao passar o mouse

* **Passe o mouse** sobre qualquer marcador para exibir uma miniatura daquela imagem com o nome do arquivo
* **Clique**em um marcador para selecionar a imagem e**fixar** a janela pop-up aberta — ela permanece até você clicar em outro lugar. Enquanto uma janela pop-up estiver fixada, passar o mouse sobre outros marcadores não a fecha
* Essa é a maneira mais rápida de encontrar um quadro específico em uma sessão grande sem sair do mapa

<figure><img src="../.gitbook/assets/image (36).png" alt=""><figcaption><p>A aba “Mapa” exibe todas as imagens com geotags do projeto</p></figcaption></figure>### Superzoom

{% hint style="success" %}
**SUPERZOOM**: quando você atinge o zoom máximo para o qual o provedor de blocos de imagem possui imagens, ampliar ainda mais aumenta os blocos em vez de parar, permitindo que você diferencie marcadores que estão quase sobrepostos.
{% endhint %}

* O superzoom é ativado apenas quando você está **no** zoom máximo do provedor para aquele local e os blocos já terminaram de carregar. Abaixo desse nível, o zoom funciona normalmente
* O intervalo é de **1× a 32×** além do máximo do próprio provedor
* Um indicador no canto mostra o superzoom atual como uma porcentagem, e um botão **×** ao lado dele leva você de volta ao zoom normal com um clique
* Ao diminuir o zoom, a ação sempre é repassada ao próprio mapa, de modo que você nunca fica preso no superzoom
* Ao ampliar e deslocar a imagem enquanto estiver no superzoom, o deslocamento resultante é repassado ao mapa, de modo que a área descentrada para a qual você se deslocou continua solicitando blocos, em vez de ficar em branco
* Os marcadores são desenhados como elementos vetoriais, em vez de serem rasterizados, por isso permanecem nítidos em todos os níveis de superzoom

***

## Provedores de blocos de mapa

{% hint style="success" %}
**Seleção automática**: o Chloros escolhe o serviço de blocos que oferece o melhor nível de zoom para onde quer que suas imagens estejam. Você pode alternar manualmente a qualquer momento.
{% endhint %}

| Provedor        | Observações                                                                                                                                                             |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Google Maps** | Ampla cobertura mundial; suporta todos os quatro tipos de blocos                                                                                                            |
| **Esri ArcGIS**| Imagens aéreas frequentemente com resolução mais alta em determinadas regiões. O tipo de bloco**Terrain** não é oferecido para o Esri e seu botão fica desativado enquanto o Esri estiver selecionado |***

## Tipos de blocos de mapa

Escolha o tipo de camada do mapa usando os botões (da esquerda para a direita):

![](&lt;../.gitbook/assets/image (14).png&gt;)

| Tipo                 | Exibe                                                                |
| -------------------- | -------------------------------------------------------------------- |
| **Terreno**          | Sombreamento de elevação com detalhes do mapa (estradas, rótulos). Apenas Google       |
| **Mapa**              | Blocos padrão de mapa de ruas — a opção que requer menor largura de banda              |
| **Satélite**        | Imagens detalhadas de satélite, sem legendas — a opção que exige maior largura de banda |
| **Híbrido** (padrão) | Imagens de satélite com estradas e legendas sobrepostas                |

A guia “Mapa” abre na opção **Híbrido**. Sua escolha é aplicada à troca de provedor, desde que o provedor ofereça suporte a isso.***

## Navegação no mapa

* **Zoom**: roda de rolagem do mouse ou os botões de zoom no mapa
* **Deslocamento**: clique e arraste
* **Tela inteira**: o controle de tela inteira expande o mapa para ocupar toda a janela***

## Casos de uso

### Revisão da rota de voo

* Veja a área de cobertura de uma sessão de drone de relance
* Identifique lacunas onde uma passagem foi perdida
* Confirme se o voo seguiu o padrão planejado

### Revisão de levantamento terrestre

* Veja como as capturas terrestres estão distribuídas
* Localize os quadros-alvo de calibração em relação à área de levantamento
* Decida onde são necessárias capturas adicionais

### Controle de qualidade

* Encontre imagens capturadas em locais inesperados e remova-as antes do processamento
* Classifique por altitude para identificar um quadro capturado na altura errada ou em que a localização por GPS foi precária
* Compare as localizações das imagens com as anotações de campo

***

## Solução de problemas

### Não aparecem marcadores

**Possíveis causas**

* As imagens não contêm metadados de GPS
* O GPS estava desativado na câmera durante a captura
* Os dados EXIF foram removidos por outro software antes da importação

**O que fazer**: confirme se o GPS está ativado na câmera e reimporte os arquivos originais. Você pode verificar se um arquivo específico possui coordenadas procurando-o na tabela de arquivos da aba “Mapa” — uma imagem sem coordenadas não terá nenhuma linha nessa tabela.

### Os marcadores estão no lugar errado

**Possíveis causas**: uma localização por satélite insatisfatória no momento da captura ou desvio do GPS durante a sessão.**O que fazer**: trata-se de um problema relacionado ao momento da captura, e não de algo que o Chloros possa corrigir posteriormente. Para trabalhos de precisão, use um fluxo de trabalho com GPS PPK/RTK — consulte a configuração**Aplicar correções PPK** em [Configurações do projeto](../project-settings/project-settings.md).

### O mapa está em branco ou os blocos param de carregar

Os provedores de blocos são serviços on-line. Se os blocos pararem de chegar, verifique a conexão de rede do equipamento e, em seguida, tente trocar de provedor. Se você estava com o zoom muito ampliado, pressione o botão de reinicialização **×** para retornar a um nível normal de zoom e permitir que o mapa solicite os blocos novamente.***

## Páginas relacionadas

* [**Grade de imagens**](image-grid.md) — o mesmo conjunto de imagens usado como miniaturas
* [**Abrindo uma imagem em tela cheia**](opening-an-image-full-screen.md) — inspecionando uma imagem em detalhes
* [**Adicionando arquivos a um projeto**](../processing-images-gui/adding-files-to-a-project.md) — os botões para adicionar/remover arquivos que esta guia compartilha
