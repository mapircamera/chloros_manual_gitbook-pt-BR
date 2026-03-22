# Marcadores do mapa

A guia Mapa exibe suas imagens em um mapa 2D interativo com base em suas coordenadas GPS. Isso oferece uma visão geral geográfica da sua sessão de captura e ajuda a visualizar a cobertura espacial. Também é útil ao importar suas imagens pela primeira vez para remover rapidamente aquelas que não precisam ser processadas.

<figure><img src="../.gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

## Acessando a guia Mapa

1. Abra ou crie um projeto no Chloros
2. Importe imagens que contenham metadados de GPS
3. Clique na guia **Mapa** <img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> na barra lateral esquerda
4. O mapa exibirá marcadores na localização GPS de cada imagem

{% hint style="info" %}
**GPS necessário**: Somente imagens com coordenadas GPS incorporadas em seus metadados EXIF aparecerão no mapa. Certifique-se de que o GPS da sua câmera esteja ativado durante a captura.
{% endhint %}

***

## Ajustando imagens na guia Mapa

A guia **Mapa**<img src="../.gitbook/assets/image (3).png" alt="" data-size="line"> possui os mesmos botões de adicionar  <img src="../.gitbook/assets/image.png" alt="" data-size="line">   <img src="../.gitbook/assets/image (1).png" alt="" data-size="line">  e remover  <img src="../.gitbook/assets/image (2).png" alt="" data-size="line">  que a guia [**Navegador de arquivos**](../processing-images-gui/adding-files-to-a-project.md) <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line"> . Ela também exibe a mesma lista de arquivos do projeto, mas com cabeçalhos de coluna diferentes:

### Nome do arquivo

* Nome original do arquivo da câmera
* Mantém a convenção de nomenclatura da câmera (por exemplo, IMG\_0001.RAW)

### Latitude

* A latitude da imagem

### Longitude

* A longitude da imagem

### Altitude

* A altitude da imagem

{% hint style="info" %}
Clicar nos cabeçalhos das colunas da tabela também ordena os dados das linhas
{% endhint %}

***

## Marcadores de imagem

Cada imagem com dados de GPS é representada por um marcador no mapa:

### Exibição de marcadores

* Os marcadores indicam as coordenadas GPS exatas onde cada imagem foi capturada
* Marcadores agrupados podem se juntar quando o zoom é reduzido
* Aumente o zoom para ver as localizações individuais das imagens

{% hint style="success" %}
SUPER-ZOOM: Quando você atinge o nível máximo de zoom do provedor de blocos do mapa, o bloco é ampliado ao aumentar ainda mais o zoom, permitindo que você veja marcadores que estão próximos uns dos outros.
{% endhint %}

### Visualização ao passar o mouse

* **Passe o mouse** sobre qualquer marcador para ver uma pré-visualização em miniatura dessa imagem
* Isso permite uma identificação visual rápida sem sair da visualização do mapa
* Útil para localizar imagens específicas dentro de uma grande sessão de captura

***

## Provedores de blocos de mapa

{% hint style="success" %}
**Seleção automática**: o Chloros escolhe automaticamente o serviço de blocos que oferece o melhor nível de zoom para a sua localização atual no mapa. Você pode alternar manualmente entre os provedores, se desejar.
{% endhint %}

A guia Mapa oferece suporte a dois provedores de blocos para as imagens de fundo do mapa:

### Google Maps

* Imagens padrão de satélite e mapa do Google
* Ideal para cobertura geral mundial

### ESRI

* Imagens de satélite e aéreas do ESRI ArcGIS
* Frequentemente oferece imagens de maior resolução em determinadas regiões

***

## Tipos de blocos de mapa

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

* **Aumentar/Diminuir zoom**: Use a roda de rolagem do mouse ou os botões de zoom
* **Tela cheia**: Exiba o mapa em tela cheia

### Controles de panorâmica

* **Panorâmica**: Clique e arraste para se deslocar pelo mapa***

## Casos de uso

### Visualização da trajetória de voo

* Visualize a área de cobertura das sessões de captura por drone
* Identifique lacunas na cobertura de imagens
* Verifique a execução da trajetória de voo

### Revisão do levantamento terrestre

* Veja a distribuição espacial das capturas terrestres
* Localize imagens de alvos de calibração em relação à área de levantamento
* Planeje locais de captura adicionais

### Controle de qualidade

* Identifique rapidamente imagens capturadas em locais inesperados
* Verifique a precisão do GPS em todo o conjunto de dados
* Compare as localizações das imagens com as anotações de campo

***

## Solução de problemas

### Marcadores não aparecem

**Possíveis causas:**

* As imagens não contêm metadados de GPS
* O GPS estava desativado na câmera durante a captura
* Os dados EXIF foram removidos por um software externo

**Solução**: Verifique se o GPS está ativado na sua câmera e reimporte os arquivos originais

### Marcadores na localização errada

**Possíveis causas:**

* O GPS da câmera teve uma fixação de satélite deficiente
* Desvio do GPS durante a captura

**Solução**: Isso geralmente é um problema relacionado ao momento da captura; considere usar GPS PPK/RTK para aplicações de precisão
