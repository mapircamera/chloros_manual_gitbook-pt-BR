# Marcadores do mapa

A guia Mapa exibe suas imagens em um mapa 2D interativo com base nas coordenadas GPS. Isso fornece uma visão geral geográfica da sua sessão de captura e ajuda a visualizar a cobertura espacial. Também é útil ao importar suas imagens pela primeira vez para remover rapidamente quaisquer imagens que você não precise processar.

## Acessando a guia Mapa

1. Abra ou crie um projeto no Chloros
2. Importe imagens que contenham metadados GPS
3. Clique na guia **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na barra lateral esquerda
4. O mapa exibirá marcadores na localização GPS de cada imagem

{% hint style=&quot;info&quot; %}
**GPS necessário**: Somente imagens com coordenadas GPS incorporadas em seus metadados EXIF aparecerão no mapa. Certifique-se de que sua câmera tenha o GPS ativado durante a captura.
{% endhint %}

***

## Ajustando imagens na guia Mapa

A guia **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> tem as mesmas opções de adicionar  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  e remover  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que a guia [**Navegador de arquivos**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Ele também mostra a mesma lista de arquivos de projeto, mas com cabeçalhos de coluna diferentes:

### Nome do arquivo

* Nome do arquivo original da câmera
* Mantém a convenção de nomenclatura da câmera (por exemplo, IMG\_0001.RAW)

### Latitude

* A latitude da imagem

### Longitude

* A longitude da imagem

### Altitude

* Altitude da imagem

{% hint style=&quot;info&quot; %}
Clicar nos cabeçalhos das colunas da tabela também classifica os dados da linha
{% endhint %}

***

## Marcadores de imagem

Cada imagem com dados GPS é representada por um marcador no mapa:

### Exibição do marcador

* Os marcadores indicam as coordenadas GPS exatas onde cada imagem foi capturada
* Os marcadores agrupados podem se agrupar quando o zoom é reduzido
* Aumente o zoom para ver as localizações individuais das imagens

{% hint style=&quot;success&quot; %}
SUPER-ZOOM: Quando você atinge o nível máximo de zoom do provedor de blocos do mapa, o bloco é ampliado ao aumentar ainda mais o zoom, permitindo que você veja os marcadores que estão próximos uns dos outros.
{% endhint %}

### Visualização ao passar o mouse

* **Passe o mouse** sobre qualquer marcador para ver uma pré-visualização em miniatura dessa imagem
* Isso permite uma identificação visual rápida sem sair da visualização do mapa
* Útil para localizar imagens específicas em uma grande sessão de captura

***

## Provedores de blocos de mapa

{% hint style=&quot;success&quot; %}
**Seleção automática**: Chloros escolhe automaticamente o serviço de mosaicos que oferece o melhor nível de zoom para a sua localização atual no mapa. Você pode alternar manualmente entre os provedores, se desejar.
{% endhint %}

A guia Mapa oferece suporte a dois provedores de mosaicos para as imagens de fundo do mapa:

### Google Maps

* Imagens padrão de satélite e mapa do Google
* Ideal para cobertura geral em todo o mundo

### ESRI

* Imagens de satélite e aéreas do ESRI ArcGIS
* Geralmente fornece imagens de alta resolução em determinadas regiões

***

## Tipos de mosaicos de mapa

Você pode escolher o tipo de camada do mapa (da esquerda para a direita):

 <img src="../.gitbook/assets/image (23).png" alt="" data-size="original">### Terreno

Mostra perfis de elevação e blocos de mapa com detalhes (estradas, etc.)

### Mapa

Mostra blocos de mapa padrão (baixa largura de banda) com detalhes (estradas, etc.)

### Satélite

Mostra blocos de mapa de satélite detalhados (alta largura de banda)

### Híbrido

Mostra blocos de mapa de satélite com detalhes adicionais (estradas, etc.)

***

## Navegação no mapa

### Controles de zoom

* **Aumentar/diminuir zoom**: use a roda do mouse ou os botões de zoom
* **Tela inteira**: exibe o mapa em tela inteira

### Controles de panorâmica

* **Panorâmica**: clique e arraste para se mover pelo mapa***

## Casos de uso

### Visualização da trajetória de voo

* Visualize a área de cobertura das sessões de captura do drone
* Identifique lacunas na cobertura da imagem
* Verifique a execução da trajetória de voo

### Revisão da pesquisa em terra

* Veja a distribuição espacial das capturas em terra
* Localize imagens-alvo de calibração em relação à área de pesquisa
* Planeje locais de captura adicionais

### Controle de qualidade

* Identifique rapidamente imagens capturadas em locais inesperados
* Verifique a precisão do GPS em todo o conjunto de dados
* Cruze as localizações das imagens com as notas de campo

***

## Resolução de problemas

### Não aparecem marcadores

**Possíveis causas:**

* As imagens não contêm metadados GPS
* O GPS estava desativado na câmera durante a captura
* Os dados EXIF foram removidos por um software externo

**Solução**: verifique se o GPS está ativado na sua câmera e reimporte os arquivos originais

### Marcadores em localização incorreta

**Possíveis causas:**

* O GPS da câmera tinha uma fixação de satélite ruim
* Desvio do GPS durante a captura

**Solução**: isso geralmente é um problema relacionado ao tempo de captura; considere usar GPS PPK/RTK para aplicações de precisão
