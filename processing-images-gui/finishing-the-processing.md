# Conclusão do processamento

Assim que o Chloros concluir o processamento, é hora de analisar os resultados, verificar a qualidade da saída e preparar as imagens processadas para uso em seu fluxo de trabalho. Esta página orienta você nas etapas finais e nas próximas ações.

## Indicação de processamento concluído

Quando o processamento for concluído com sucesso, você verá vários indicadores:

* ✅ **Barra de progresso**: Atinge 100% de conclusão
* ✅ **Registro de depuração**: Exibe a mensagem “Processamento concluído”
* ✅ **Botão Iniciar**: Torna-se ativo novamente (pronto para a próxima execução do processamento)
* ✅ **Arquivos de saída**: Todas as imagens processadas salvas na subpasta do modelo da câmera***

## Localizando suas imagens processadas

### Abrindo a pasta de saída

1. Clique no ícone do **Menu principal** <img src="../.gitbook/assets/image (1) (1) (1) (1).png" alt="" data-size="line"> (canto superior esquerdo)
2. Selecione **&quot;Abrir pasta do projeto&quot;**

3. Seu explorador de arquivos abre no diretório do projeto
4. Localize seu projeto pelo nome

***

## Revisando imagens processadas

### Visualização rápida no Explorador de Arquivos

**Visualização integrada do Windows:**

1. Navegue até a subpasta do modelo da câmera
2. Selecione um arquivo de imagem
3. A visualização aparece no painel de visualização do Windows Explorer
4. Use as setas do teclado para percorrer as imagens

### Visualização em visualizadores de imagem externos

**Visualizadores recomendados:*** **QGIS** - Software GIS gratuito (ideal para análise multiespectral georreferenciada)
* **IrfanView** - Visualizador de imagens rápido e leve (compatível com TIFF)
* **Adobe Photoshop** - Edição profissional (compatível com TIFF)
* **GIMP** - Alternativa gratuita ao Photoshop
* **Windows Photos** - Visualização básica (pode não ser compatível com TIFF de 16 bits)

### Visualização no Visualizador de Imagens do Chloros

Use o Visualizador de Imagens integrado do Chloros para visualização avançada:

1. Clique em uma miniatura de imagem no Navegador de Arquivos
2. A imagem abre na área de visualização principal
3. Clique na guia **Visualizador de Imagens** <img src="../.gitbook/assets/icon_image-viewer.JPG" alt="" data-size="line"> na barra lateral esquerda
4. Use [Index/LUT Sandbox](../image-viewer-gui/index-lut-sandbox.md) para análise interativa

Consulte [Visualizador de Imagens](../image-viewer-gui/opening-an-image-full-screen.md) para obter instruções detalhadas.

***

## Analisando o Log de Depuração

### Verifique se há avisos ou erros

1. Abra a guia **Registro de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> 2. Percorra as mensagens
3. Procure por avisos em amarelo ou erros em vermelho
4. Analise quaisquer problemas observados
5. Entre em contato com o suporte da MAPIR para obter assistência

### Salvando o log

Para manter um registro do processamento ou enviar ao suporte da MAPIR:

1. Clique no botão **&quot;Copiar&quot;**ou**&quot;Baixar&quot;**

2. Salve como arquivo de texto na pasta do projeto
3. Inclua na documentação do projeto
4. Envie ao suporte da MAPIR caso encontre problemas

***

## Problemas comuns de saída e soluções

### Problema: Arquivos de saída ausentes

**Possíveis causas:**

* Os arquivos não atenderam aos critérios de processamento
* Imagens apenas de destino (excluídas da exportação)
* Espaço em disco esgotado durante a exportação
* Corrupção de arquivo durante o processamento

**Soluções:**

1. Verifique o Log de Depuração para mensagens de omissão/erro
2. Verifique se havia espaço em disco suficiente
3. Conte os arquivos: deve corresponder a (contagem original - contagem de destino) × (índices + 1)
4. Reimporte e reprocesse quaisquer arquivos ausentes

### Problema: Bordas escuras ou claras (vinheta ainda visível)

**Possíveis causas:**

* Correção de vinheta desativada
* Câmera/lente não consta no banco de dados de perfis do Chloros
* Vinheta extrema além da capacidade de correção

**Soluções:**

1. Verifique se a correção de vinheta foi ativada nas Configurações do Projeto
2. Verifique se o modelo da câmera foi detectado corretamente
3. Entre em contato com o suporte do MAPIR se o efeito de vinheta persistir

### Problema: Cores ou valores incorretos

**Possíveis causas:**

* Nenhum alvo de calibração detectado
* Modelo de alvo de calibração incorreto selecionado
* Calibração de refletância desativada
* Imagens de alvo de baixa qualidade

**Soluções:**

1. Verifique se a calibração de refletância foi ativada
2. Verifique as mensagens “Alvo encontrado” no Log de Depuração
3. Analise a qualidade da imagem do alvo
4. Reprocesse com os alvos corretos marcados

### Problema: Os valores do NDVI parecem incorretos

**Intervalos esperados do NDVI:*** **Água, rochas, solo**: -0,1 a 0,2
* **Vegetação esparsa/doente**: 0,2 a 0,4
* **Vegetação moderada**: 0,4 a 0,6
* **Vegetação saudável e densa**: 0,6 a 0,9**Se os valores estiverem fora desses intervalos:**

1. Verifique se a calibração de refletância foi aplicada
2. Verifique se o registro do sensor de luz foi incluído
3. Verifique se os alvos de calibração foram detectados
4. Certifique-se de que o modelo correto da câmera foi detectado
5. Revise o momento e as condições de captura da imagem-alvo

***

## Usando suas imagens processadas

### Para fotogrametria / criação de ortomosaicos

**Fluxo de trabalho recomendado:**

1.**Importe imagens de refletância calibradas** para o software de fotogrametria:
   * Pix4Dmapper
   * Agisoft Metashape
   * DroneDeploy
   * WebODM
2. **Mantenha os metadados EXIF**: Certifique-se de que os dados de GPS sejam preservados para georreferenciamento
3. **Fluxos de trabalho calibrados**: Use imagens de refletância para precisão científica
4. **Processe mosaicos de índice**: Crie ortomosaicos NDVI a partir de imagens de índice individuais
5. **Exporte GeoTIFF georreferenciados**: Para uso em aplicações GIS

### Para análise GIS

**Fluxo de trabalho recomendado:**

1.**Carregue no QGIS, ArcGIS ou similar**

2.**Use imagens de refletância de 16 bits TIFF** para análise multibanda
3. **Use imagens de índice** (NDVI, NDRE) como camadas de vegetação prontas para uso
4. **Calculadora raster**: Combine bandas para análise personalizada
5. **Exportar**: Crie mapas de classificação, detecção de alterações e mapas de saúde da vegetação

### Para análise direta / relatórios

**Fluxo de trabalho recomendado:**

1.**Use imagens de índice com cores LUT** para relatórios visuais
2. **Extraia estatísticas**: Média NDVI por campo/parcela
3. **Série temporal**: Compare índices entre várias sessões
4. **Gere relatórios**: Inclua mapas, estatísticas e visualizações***

## Arquivamento e backup

### Estratégia de backup recomendada

**O que salvar:*** ✅ **Imagens RAW/JPG originais** - Arquive em unidade separada/nuvem
* ✅ **Resultados processados** - Mantenha imagens calibradas e índices
* ✅ **Arquivo do projeto** - Contém todas as configurações para reprocessamento, se necessário
* ✅ **Registro de depuração** - Documenta detalhes do processamento
* ✅ **Imagens de alvo de calibração** - Para verificação e reprocessamento**Recomendações de armazenamento:*** **Backup imediato**: Disco rígido externo
* **Arquivamento de longo prazo**: Armazenamento em nuvem (Google Drive, Dropbox, etc.)
* **Dados críticos**: Mantenha 2-3 cópias em locais diferentes***

## Próximas execuções de processamento

### Reutilização das configurações do projeto

Se for processar conjuntos de dados semelhantes no futuro:

1. **Salve o modelo do projeto** (se ainda não tiver feito isso)
2. **Crie um novo projeto** usando o modelo salvo
3. **Importe novas imagens**

4.**Processe**com configurações idênticas para garantir a consistência

### Processamento em lote de várias sessões

Para várias sessões/conjuntos de dados:**Opção 1: GUI - Vários projetos**

* Crie um projeto separado para cada sessão
* Use configurações de modelo consistentes
* Processe um de cada vez

**Opção 2: Chloros CLI (somente Chloros+)**

* Automatize o processamento em lote
* Processe várias pastas com scripts
* Consulte a [Documentação do CLI](../CLI.md)

**Opção 3: Python SDK (somente Chloros+)**

* Controle programático
* Integração com pipelines de análise
* Consulte a [Documentação do API](../api-python-sdk.md)

***

## Solução de problemas no pós-processamento

### Reprocessamento com configurações diferentes

Se os resultados não forem satisfatórios:

1. Mantenha as imagens originais (nunca as exclua)
2. Abra o mesmo projeto no Chloros
3. Ajuste as configurações no painel Configurações do projeto
4. Processe novamente — os resultados substituirão os anteriores

### Processamento de um subconjunto de imagens

Para reprocessar apenas imagens específicas:

1. Crie um novo projeto
2. Importe apenas as imagens que precisam ser reprocessadas
3. Use o mesmo modelo de configurações
4. Processe um conjunto de dados menor

### Obtenha ajuda

Se você encontrar problemas:

* 📧 **E-mail**: info@mapir.camera (inclua o Log de Depuração)
* 🌐 **Suporte**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Perguntas frequentes**: [Perguntas frequentes](../faq.md)
* 📖 **Documentação**: [Manual do Chloros](../)***

## Resumo: Fluxo de trabalho completo

Você concluiu agora todo o fluxo de trabalho de processamento do Chloros:

1. ✅ **Projeto criado** - Consulte [Projetos](../projects.md)
2. ✅ **Adicionou arquivos** - Consulte [Adicionando arquivos](adding-files-to-a-project.md)
3. ✅ **Ajustou configurações** - Consulte [Ajustando configurações do projeto](adjusting-project-settings.md)
4. ✅ **Marcou alvos** - Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
5. ✅ **Iniciou o processamento** - Consulte [Iniciando o processamento](starting-the-processing.md)
6. ✅ **Monitorou o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
7. ✅ **Resultados revisados** - Esta página**Suas imagens multiespectrais calibradas e com correção de refletância estão prontas para análise!**

***

## Recursos adicionais

### Recursos avançados

* [**Visualizador de imagens**](../image-viewer-gui/opening-an-image-full-screen.md) - Visualização e análise interativas
* [**Área de testes de índices/LUT**](../image-viewer-gui/index-lut-sandbox.md) - Testes de índices personalizados
* [**Fórmulas de índices multiespectrais**](../project-settings/multispectral-index-formulas.md) - Referência completa de índices

### Automação e integração

* [**Documentação do CLI**](../CLI.md) - Processamento em lote via linha de comando
* [**Python SDK**](../api-python-sdk.md) - Automação programática
* [**Recursos do Chloros+**](../#chloros) - Recursos avançados de processamento

### Suporte e Aprendizagem

* [**Perguntas frequentes**](../faq.md) - Respostas às perguntas mais comuns
* [**Alvos de calibração**](../calibration-targets.md) - Entendendo a calibração de refletância
* [**Câmeras compatíveis**](../supported-cameras.md) - Hardware compatível
