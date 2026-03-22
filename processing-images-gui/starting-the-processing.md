# Iniciando o processamento

Depois de importar suas imagens, marcar seus alvos de calibração e definir as configurações do projeto, você estará pronto para iniciar o processamento. Esta página orienta você sobre como iniciar o fluxo de processamento Chloros.

## Lista de verificação do pré-processamento

Antes de clicar no botão Iniciar, verifique se tudo está pronto:

* [ ] **Arquivos importados** - Todas as imagens aparecem no Navegador de Arquivos
* [ ] **Imagens-alvo marcadas** - Coluna Alvo marcada para imagens de calibração
* [ ] **Modelos de câmera detectados** - A coluna Modelo de Câmera mostra as câmeras corretas
* [ ] **Configurações definidas** - Configurações do projeto revisadas e ajustadas
* [ ] **Índices selecionados** - Índices multiespectrais desejados adicionados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado para o seu fluxo de trabalho

{% hint style="info" %}
**Dica**: Clique em algumas imagens no Navegador de Arquivos para verificar se elas foram carregadas corretamente antes do processamento.
{% endhint %}

***

## Iniciando o processamento

### Localize o botão Iniciar

O botão Iniciar/Reproduzir está localizado na barra de cabeçalho superior do Chloros:

* Posição: Parte superior central da janela
* Ícone: **Botão Reproduzir/Iniciar** <img src="../.gitbook/assets/image (2) (1).png" alt="" data-size="line">
* Status: O botão fica ativado (iluminado) quando pronto para processar

### Clique para iniciar

1. Clique no **botão Reproduzir/Iniciar** na barra superior
2. O processamento começa imediatamente
3. O botão fica desativado (esmaecido) durante o processamento
4. A barra de progresso é atualizada, mostrando o status do processamento

{% hint style="success" %}
**Processamento iniciado**: Uma vez clicado, o Chloros lida automaticamente com todas as etapas de processamento — detecção de alvo, debayering, calibração, cálculo de índice e exportação.
{% endhint %}

***

## Entendendo os modos de processamento

O Chloros opera em dois modos de processamento diferentes, dependendo da sua licença:

### Modo Gratuito (Processamento Sequencial)

**Disponível para todos os usuários**

**Como funciona:**

* Processa imagens uma de cada vez, sequencialmente
* Operação em thread único
* Menor uso de memória

**A barra de progresso mostra 2 etapas:**

1.**Detecção de alvo** - Varredura em busca de alvos de calibração
2. **Processamento** - Aplicação da calibração e exportação de imagens**Tempo de processamento:**

* Muito mais lento do que o modo paralelo do Chloros+
* Adequado para conjuntos de dados de pequeno a médio porte (&lt; 200 imagens)

### Modo Chloros+ (Processamento Paralelo)

**Requer licença Chloros+**

**Como funciona:**

* Processa várias imagens simultaneamente usando um [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md)
* A [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) seleciona automaticamente a estratégia ideal para o seu hardware
* Aceleração por GPU (CUDA) com placas de vídeo NVIDIA (desktop e Jetson)
* Escalável de um Jetson Nano (1 worker) a um desktop com GPU de 12 GB ou mais (3-4 workers)

**A barra de progresso mostra 4 etapas** (correspondentes às 4 threads do pipeline):

1. **Detecção** (Thread 1) - Localização de alvos de calibração
2. **Análise** (Thread 2) - Exame dos metadados da imagem e cálculo da calibração
3. **Calibração** (Thread 3) - Debayering da GPU, correção de vinheta, cálculo do índice
4. **Exportação** (Thread 4) - Salvando imagens processadas e índices**Interação com a barra de progresso:*** **Passe o mouse** sobre a barra para ver o painel suspenso detalhado de 4 etapas
* **Clique** na barra de progresso para fixar o painel suspenso no lugar
* **Clique novamente** para desfixar e ocultar o painel**Tempo de processamento:**

* Significativamente mais rápido que o modo gratuito
* Escala com o número de núcleos da CPU
* A aceleração por GPU melhora ainda mais a velocidade

{% hint style="info" %}
**Chloros+ Velocidade**: O processamento paralelo pode ser 5 a 10 vezes mais rápido do que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que leva 2 horas no modo gratuito pode ser concluído em 15 a 20 minutos com o Chloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Etapa 1: Detecção de alvos

**O que o Chloros faz:**

* Analisa imagens de alvos marcados (ou todas as imagens, se nenhuma estiver marcada)
* Identifica os 4 painéis de calibração em cada alvo
* Extrai valores de refletância dos painéis de alvo
* Registra os carimbos de data/hora dos alvos para agendamento da calibração

**Duração:** 1-30 segundos (com alvos marcados), 5-30+ minutos (sem marcação)

### Etapa 2: Debayering (Conversão RAW)

**O que o Chloros faz:**

* Converte dados RAW no padrão Bayer em imagens RGB completas
* Aplica algoritmo de demosaicing de alta qualidade
* Preserva a máxima qualidade e detalhes da imagem

**Duração:** Varia de acordo com o número de imagens e a velocidade da CPU

### Etapa 3: Calibração

**O que o Chloros faz:*** **Correção de vinheta**: Remove o escurecimento das bordas causado pela lente
* **Calibração de refletância**: Normaliza usando valores de refletância de referência
* Aplica correções em todas as bandas/canais
* Utiliza alvo de calibração apropriado para cada imagem com base no carimbo de data/hora

**Duração:** A maior parte do tempo de processamento

### Etapa 4: Cálculo de Índices

**O que o Chloros faz:**

* Calcula índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplica cálculos matemáticos de banda às imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Etapa 5: Exportação

**O que o Chloros faz:**

* Salva imagens calibradas no formato selecionado
* Exporta imagens de índice com cores LUT configuradas
* Grava arquivos em subpastas de modelos de câmera
* Preserva nomes de arquivos originais com sufixos

**Duração:** Varia de acordo com o formato de exportação e o tamanho do arquivo***

## Comportamento do processamento

### Pipeline de processamento automático

Uma vez iniciado, todo o pipeline é executado automaticamente:

* Não é necessária interação do usuário
* Todas as etapas configuradas são executadas em sequência
* Atualizações de progresso exibidas em tempo real

### Utilização do computador durante o processamento

**Modo Livre:**

* Uso relativamente baixo da CPU (single-threaded)
* O computador permanece responsivo para outras tarefas
* É seguro minimizar o Chloros e trabalhar em outros aplicativos

**Chloros+ Modo Paralelo:**

* Alto uso da CPU (multi-threaded, até 16 núcleos)
* Com aceleração por GPU: Alto uso da GPU
* O computador pode ficar menos responsivo durante o processamento
* Evite iniciar outras tarefas que exijam muito da CPU

{% hint style="warning" %}
**Dica de desempenho**: Para obter o melhor desempenho do Chloros+, feche outros aplicativos e deixe o Chloros usar todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado

**Limitações importantes:**

* Uma vez iniciado, o processamento não pode ser pausado
* Você pode cancelar o processamento, mas o progresso será perdido
* Resultados parciais não são salvos
* É necessário reiniciar do início se for cancelado

**Dica de planejamento:** Para projetos muito grandes, considere processar em lotes ou usar o CLI para um melhor controle.***

## Monitorando seu processamento

Enquanto o processamento está em execução, você pode:

* **Observar a barra de progresso** - Ver a porcentagem geral de conclusão
* **Visualizar o estágio atual** - Detectar, Analisar, Calibrar ou Exportar
* **Verificar a guia de log** - Ver mensagens e avisos detalhados do processamento
* **Visualizar imagens concluídas** - Alguns arquivos de exportação podem aparecer durante o processamento

Para informações detalhadas sobre o monitoramento, consulte [Monitorando o processamento](monitoring-the-processing.md).

***

## Cancelando o processamento

Se você precisar interromper o processamento:

### Como cancelar

1. Localize o **botão Parar/Cancelar** (substitui o botão Iniciar durante o processamento)
2. Clique no botão Parar
3. O processamento é interrompido imediatamente
4. Os resultados parciais são descartados

### Quando cancelar

**Motivos válidos para cancelar:**

* Percebeu que foram utilizadas configurações incorretas
* Esqueceu de marcar as imagens de destino
* Importou imagens erradas
* O sistema está muito lento ou não responde

**Após o cancelamento:**

* Revise e corrija quaisquer problemas
* Ajuste as configurações conforme necessário
* Reinicie o processamento desde o início
* Para uma experiência mais limpa, feche completamente o Chloros e reinicie

{% hint style="warning" %}
**Sem resultados parciais**: O cancelamento descarta todo o progresso. O Chloros não salva imagens parcialmente processadas.
{% endhint %}

***

## Estimativas de tempo de processamento

O tempo real de processamento varia muito com base em:

* Número de imagens
* Resolução da imagem
* Formato de entrada RAW vs JPG
* Modo de processamento (Free vs Chloros+)
* Velocidade da CPU e número de núcleos
* Disponibilidade da GPU (somente Chloros+)
* Número de índices a calcular
* Complexidade do formato de exportação

### Estimativas aproximadas (Chloros+, imagens de 12 MP, CPU moderna)

| Número de imagens | Modo Gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagens  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagens  | 1-1,5 h | 20-30 min      | 10-15 min      |
| 500 imagens  | 2-3 horas   | 45-60 min      | 20-30 min      |
| 1000 imagens | 4-6 horas   | 1,5-2 horas      | 40-60 min      |

{% hint style="info" %}
**Primeira execução**: O processamento inicial pode demorar mais, pois o Chloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns na inicialização

### Botão Iniciar desativado (desativado)

**Possíveis causas:**

* Nenhuma imagem importada
* Backend não totalmente iniciado
* Processamento anterior ainda em execução
* Projeto não totalmente carregado

**Soluções:**

1. Aguarde a inicialização completa do backend (verifique o ícone no menu principal)
2. Verifique se as imagens foram importadas no Navegador de Arquivos
3. Reinicie o Chloros se o botão permanecer desativado
4. Verifique o Log de Depuração para mensagens de erro

### O processamento inicia e falha imediatamente

**Possíveis causas:**

* Não há imagens válidas no projeto
* Arquivos de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o Log de Depuração <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> em busca de mensagens de erro
2. Verifique o espaço em disco disponível
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### Aviso “Nenhum alvo detectado”

**Possíveis causas:**

* Esqueceu de marcar as imagens-alvo
* As imagens-alvo não contêm alvos visíveis
* Configurações de detecção de alvos muito rígidas

**Soluções:**

1. Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
2. Marque as imagens apropriadas na coluna &quot;Alvo&quot;
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as configurações de detecção de alvos, se necessário

***

## Dicas para um processamento bem-sucedido

### Antes de começar

1. **Teste primeiro com um pequeno subconjunto** - Processe 10 a 20 imagens para verificar as configurações
2. **Verifique o espaço disponível em disco** - Garanta que haja espaço livre equivalente a 2 a 3 vezes o tamanho do conjunto de dados
3. **Feche aplicativos desnecessários** - Libere recursos do sistema
4. **Verifique as imagens-alvo** - Visualize os alvos marcados para garantir a qualidade
5. **Salve o projeto** - O projeto é salvo automaticamente, mas é recomendável salvar manualmente

### Durante o processamento

1. **Evite que o sistema entre em modo de suspensão** - Desative os modos de economia de energia
2. **Mantenha o Chloros em primeiro plano** - Ou, pelo menos, visível na barra de tarefas
3. **Monitore o progresso ocasionalmente** - Verifique se há avisos ou erros
4. **Não carregue outros aplicativos pesados** - Especialmente com o modo paralelo do Chloros+

### Aceleração por GPU do Chloros+

Se estiver usando a aceleração por GPU da NVIDIA:

1. Atualize os drivers da NVIDIA para a versão mais recente
2. Certifique-se de que a GPU tenha 4 GB ou mais de VRAM
3. Feche aplicativos que exigem muito da GPU (jogos, edição de vídeo)
4. Monitore a temperatura da GPU (garanta um resfriamento adequado)

***

## Próximos passos

Assim que o processamento tiver iniciado:

1. **Monitore o progresso** - Consulte [Monitorando o processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** — O processamento é executado automaticamente
3. **Analise os resultados** — Consulte [Concluindo o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorando o processamento](monitoring-the-processing.md).
