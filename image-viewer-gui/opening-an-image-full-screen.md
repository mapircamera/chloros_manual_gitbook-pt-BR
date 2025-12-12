# Abrindo uma imagem em tela cheia

O Visualizador de Imagens Chloros oferece uma interface dedicada em tela cheia para visualizar, analisar e manipular suas imagens multiespectrais. Seja para visualizar imagens originais ou resultados processados, o Visualizador de Imagens oferece ferramentas poderosas para inspeção e análise.

## Acessando o Visualizador de Imagens

### A partir do Navegador de Arquivos

A maneira mais comum de abrir uma imagem no Visualizador de Imagens:

1. Certifique-se de estar na guia **Navegador de Arquivos** <img src="../.gitbook/assets/icon_file-browser.JPG" alt="" data-size="line">
2. Clique em qualquer **miniatura de imagem** na grade de imagens
3. A imagem é aberta na **área de visualização principal** (centro da tela)
4. A imagem agora está carregada e pronta para visualização em tela cheia

### Abrindo a guia Visualizador de Imagens

Depois que uma imagem é carregada na área de visualização:

1. Clique no ícone **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
2. A guia Visualizador de imagens é aberta, exibindo a imagem selecionada em tela cheia
3. Ferramentas avançadas de visualização e análise ficam disponíveis na barra lateral esquerda

***

## Visão geral da interface do Visualizador de imagens

### Área de exibição principal

A maior parte da tela mostra sua imagem:

* **Resolução total**: imagens exibidas na resolução nativa
* **Zoom**: use os controles ou a roda do mouse para ampliar
* **Panorâmica**: clique e arraste para se mover quando ampliar
* **Proporção mantida**: as imagens são dimensionadas proporcionalmente

***

## Opções de visualização

### Navegação básica pela imagem

#### Navegar pelas imagens

Navegue pelo conjunto de imagens usando atalhos do teclado ou botões:

* **Próxima imagem**: clique no botão → ou pressione a tecla **→** (seta para a direita)
* **Imagem anterior**: clique no botão ← ou pressione a tecla **←** (seta para a esquerda)
* **Ir para uma imagem específica**: volte ao Navegador de arquivos e clique na miniatura desejada

#### Controles de zoom

Ajuste a ampliação para inspecionar os detalhes da imagem:

**Aumentar:**

* Clique no botão **+** (Mais)
* Pressione a tecla **+** ou **=**
* Role a roda do mouse **para cima**

**Diminuir:**

* Clique no botão **−** (Menos)
* Pressione a tecla **−** (Menos)
* Role a roda do mouse **para baixo**

**Ajustar à tela:**

* Clique no botão **↔** (Ajustar)
* Pressione a tecla **0** (Zero)
* Clique duas vezes na imagem

#### Panorâmica ao ampliar

Ao ampliar além do tamanho da tela:

1. Mova o cursor do mouse sobre a imagem
2. Clique e **mantenha pressionado o botão esquerdo do mouse**
3. **Arraste** para mover a imagem
4. Solte para parar a panorâmica

**Alternativa**: Use as teclas de seta para fazer uma panorâmica em pequenos incrementos

***

## Inspeção do valor dos pixels

### Visualização dos valores dos pixels no cursor

À medida que move o cursor do mouse sobre a imagem, os valores dos pixels são exibidos em tempo real:

**Localização da exibição do valor:**

* **Número flutuante e linha vermelha na legenda do gradiente LUT do índice do lado direito**
* **Ao ampliar ainda mais, valor flutuante próximo ao cursor e pixel destacado**
* Mostra os valores dos pixels **sob o cursor ou destacados**
* Atualiza conforme você move o mouse

***

## Tipos de imagens que você pode visualizar

### Imagens originais (pré-processamento)

**Imagens RAW + JPG da câmera:**

* Exibe os dados RAW conforme visualizados
* Mostra os valores originais, sem correção
* Útil para verificar a qualidade da imagem antes do processamento

### Imagens de refletância calibradas

**Após o processamento:**

* Vinheta corrigida
* Refletância calibrada
* Multibanda TIFF (Red, Green, NIR, etc.)
* Dados científicos prontos para análise

### Imagens de índice

**NDVI, NDRE, GNDVI, etc. (arquivos \_NDVI.tif):**

* Imagens em escala de cinza de banda única
* Os valores dos pixels representam os resultados do cálculo do índice
* Faixa normalmente de -1 a +1 para índices normalizados
* É possível aplicar LUTs de cores para visualização

***

## Aplicação de índices e LUTs

Aplique índices multiespectrais e tabelas de consulta de cores:

1. Localize **Index/LUT Sandbox** na <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral
2. Selecione o índice de vegetação (NDVI, NDRE, etc.)
3. Selecione a fórmula multiespectral ou crie sua própria fórmula personalizada (somente Chloros+)
4. Aplique o gradiente LUT de cores para visualização
5. Ajuste os intervalos de valores e os limites

Consulte [Index/LUT Sandbox](index-lut-sandbox.md) para obter instruções detalhadas.

***

## Atalhos de teclado

### Navegação

* **→** (Seta para a direita): Próxima imagem
* **←** (Seta para a esquerda): Imagem anterior
* **Home**: Primeira imagem da lista
* **End**: Última imagem da lista

### Zoom

* **+** ou **=**: Ampliar
* **−**: Reduzir
* **0** (Zero): Ajustar à tela
* **Roda do mouse**: Ampliar/reduzir

### Controles de visualização

* **P**: Alternar modo de porcentagem de pixels
* **L**: Alternar painel de camadas
* **Esc**: Fechar tela cheia ou retornar ao Navegador de arquivos

### Outros

* **Ctrl+S**: Salvar imagem atual
* **F**: Modo de tela cheia (se disponível)

***

### Verificando cálculos de índice

Verifique se os índices foram calculados corretamente:

1. Abra NDVI ou outra imagem de índice
2. Verifique as áreas de vegetação:
   * **NDVI**: Deve mostrar 0,4-0,9 para plantas saudáveis
   * **NDRE**: Valores mais altos para crescimento vigoroso
   * **GNDVI**: Semelhante ao NDVI, mas sensível à clorofila
3. Verifique a não vegetação:
   * **Solo**: Perto de 0 ou ligeiramente negativo
   * **Água**: Valores negativos (-0,5 a 0)

***

## Solução de problemas de visualização

### A imagem não abre

**Possíveis causas:**

* Arquivo corrompido durante o processamento
* Formato de arquivo não compatível
* Memória insuficiente para imagens grandes

**Soluções:**

1. Tente abrir em um visualizador externo para verificar a integridade do arquivo
2. Verifique se o formato do arquivo corresponde ao tipo esperado
3. Feche outros aplicativos para liberar memória
4. Tente uma imagem menor/diferente

### Exibição de imagem em preto ou branco

**Possíveis causas:**

* Intervalo de valores fora da capacidade de exibição
* Imagem flutuante de 32 bits com valores incomuns
* Erro de cálculo do índice

**Soluções:**

1. Verifique os valores dos pixels - se todos forem muito baixos ou muito altos, ajuste o intervalo de exibição
2. Tente abrir no QGIS ou similar com ajuste automático do intervalo
3. Verifique o log de depuração do processamento para erros

### Os valores dos pixels parecem errados

**Possíveis causas:**

* Visualização da imagem errada (original vs processada)
* A calibração não foi aplicada corretamente
* Os dados do sensor de luz não foram incluídos na entrada
* O modo de porcentagem foi alternado incorretamente

**Soluções:**

1. Verifique se você está visualizando a saída processada (verifique o sufixo do nome do arquivo)
2. Verifique o estado do botão do modo de porcentagem
3. Compare com imagens conhecidas como boas do mesmo conjunto de dados

***

## Próximos passos

Agora que você pode visualizar imagens em tela cheia:

* [**Camadas de imagem**](image-layers.md) - Saiba mais sobre visualização multibanda
* [**Index/LUT Sandbox**](index-lut-sandbox.md) - Aplique índices personalizados e mapeamento de cores
* [**Fórmulas de índice multiespectral**](../project-settings/multispectral-index-formulas.md) - Entenda os índices disponíveis

Para o fluxo de trabalho de processamento, consulte:

* [**Processamento de imagens (GUI)**](../processing-images-gui/adding-files-to-a-project.md) - Guia completo de processamento
