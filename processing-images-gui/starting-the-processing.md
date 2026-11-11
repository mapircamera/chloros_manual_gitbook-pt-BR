# Iniciando o processamento

Depois de importar suas imagens, marcar seus alvos de calibração e definir as configurações do projeto, você estará pronto para iniciar o processamento. Esta página orienta você sobre como iniciar o fluxo de processamento do Chloros.

## Lista de verificação do pré-processamento

Antes de clicar no botão Iniciar, verifique se tudo está pronto:

* [ ] **Arquivos importados** — Todas as imagens aparecem no Navegador de Arquivos
* [ ] **Imagens-alvo marcadas** — Coluna “Alvo” verificada para imagens de calibração (ou uma gravação `.daq` importada para o LATTICE)
* [ ] **Modelos de câmera detectados** — A coluna “Modelo da câmera” mostra as câmeras corretas
* [ ] **Configurações definidas** — Configurações do projeto revisadas e ajustadas
* [ ] **Índices selecionados** — Índices multiespectrais desejados adicionados (se necessário)
* [ ] **Formato de exportação escolhido** - Formato de saída adequado ao seu fluxo de trabalho

{% hint style="info" %}
**Dica**: Clique em algumas imagens no Navegador de Arquivos para verificar se elas foram carregadas corretamente antes do processamento.
{% endhint %}

***

## Iniciando o processamento

### Localize o botão Iniciar

O botão Iniciar/Reproduzir está localizado na barra de cabeçalho superior do Chloros:

* Posição: parte superior central da janela
* Ícone: **botão Reproduzir/Iniciar** <img src="../.gitbook/assets/image (2) (1) (1).png" alt="" data-size="line">
* Status: o botão fica ativado (iluminado) quando estiver pronto para o processamento

### Clique para iniciar

1. Clique no **botão Reproduzir/Iniciar** na barra superior
2. O processamento começa imediatamente
3. O botão se transforma em um botão **Parar** durante o processamento
4. A barra de progresso é atualizada, mostrando o status do processamento

{% hint style="success" %}
**Processamento iniciado**: Após ser clicado, o Chloros lida automaticamente com todas as etapas de processamento — detecção de alvo, debayering, calibração, cálculo de índice e exportação. Ele detecta automaticamente se o seu projeto é Survey3, LATTICE ou uma combinação dos dois, e aplica o fluxo de trabalho correto a cada câmera.
{% endhint %}

***

## Entendendo os modos de processamento O

Chloros opera em dois modos de processamento diferentes, dependendo da sua licença:

### Modo Gratuito (Processamento Sequencial)

**Disponível para todos os usuários**

**Como funciona:**

* Processa as imagens uma de cada vez, sequencialmente
* Operação em thread único
* Menor uso de memória

**A barra de progresso mostra 2 etapas:**

1.**Detecção de alvos** — Varredura em busca de alvos de calibração
2. **Processamento** — Aplicação da calibração e exportação das imagens**Tempo de processamento:**

* Muito mais lento do que o modo paralelo do Chloros+
* Adequado para conjuntos de dados pequenos a médios (&lt; 200 imagens)

### Modo Chloros+ (Processamento Paralelo)

**Requer licença do Chloros+**

**Como funciona:**

* Processa várias imagens simultaneamente usando um [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md)
* A [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) seleciona automaticamente a estratégia ideal para o seu hardware no início da execução
* Aceleração por GPU (CUDA) com placas de vídeo NVIDIA
* **O número de workers se adapta ao hardware**: as estratégias de GPU executam**2 a 4 workers simultâneos** (escalonados pela VRAM — 3 em uma GPU de 2 a 12 GB, 2 a 4 em uma GPU de 12 GB ou mais); Sistemas apenas com CPU executam um worker por núcleo físico, menos um**A barra de progresso mostra 4 etapas** (correspondentes às 4 threads do pipeline):

1. **Detecção** (Thread 1) — Localização dos alvos de calibração
2. **Análise** (Thread 2) — Exame dos metadados da imagem e cálculo da calibração
3. **Calibração** (Thread 3) — Remoção do efeito bayesian, correção de vinheta, calibração, cálculo do índice
4. **Exportação** (Thread 4) — Salvamento das imagens processadas e dos índices**Interação com a barra de progresso:*** **Passe o mouse** sobre a barra para ver o painel suspenso detalhado com as 4 etapas
* **Clique** na barra de progresso para fixar o painel suspenso no lugar
* **Clique novamente** para desfixar e ocultar o painel**Tempo de processamento:**

* Significativamente mais rápido que o modo gratuito
* A aceleração por GPU melhora ainda mais a velocidade

{% hint style="info" %}
**Velocidade do Chloros+**: O processamento paralelo pode ser de 5 a 10 vezes mais rápido do que o modo sequencial para grandes conjuntos de dados. Um projeto de 500 imagens que leva 2 horas no modo gratuito pode ser concluído em 15 a 20 minutos com o Chloros+.
{% endhint %}

***

## O que acontece durante o processamento

### Etapa 1: Detecção de alvos

**O que o Chloros faz:**

* Analisa as imagens marcadas na coluna “Alvo” (todas as imagens, caso nenhuma esteja marcada)
* Identifica os painéis de calibração em cada alvo
* Extrai os valores de refletância dos painéis-alvo
* Registra os carimbos de data e hora dos alvos para o agendamento da calibração

**Duração:** 1 a 30 segundos (com alvos marcados), 5 a 30+ minutos (sem marcação)

### Etapa 2: Debayering (conversão RAW)

**O que o Chloros faz:**

* Converte dados RAW no padrão Bayer em imagens completas de 3 canais (os módulos mono LATTICE permanecem em banda única — o debayering é ignorado para eles, com uma nota no registro)
* Aplica o algoritmo de demosaicing selecionado
* Preserva a máxima qualidade de imagem e detalhes

**Duração:** Varia de acordo com o número de imagens e a velocidade da CPU/GPU

### Etapa 3: Calibração

**O que o Chloros faz:*** **Correção de vinheta**: Remove o escurecimento das bordas causado pela lente
* **Calibração de refletância**: Normaliza usando valores-alvo de refletância e/ou dados de radiação descendente do DAQ
* Aplica correções em todas as bandas/canais
* Utiliza a referência de calibração apropriada para cada imagem com base no carimbo de data/hora

**Duração:** A maior parte do tempo de processamento

### Etapa 4: Cálculo de índices

**O que o Chloros faz:**

* Calcula os índices multiespectrais configurados (NDVI, NDRE, etc.)
* Aplica cálculos matemáticos de bandas às imagens calibradas
* Gera imagens de índice para cada índice selecionado

**Duração:** Alguns segundos por imagem

### Etapa 5: Exportação

**O que o Chloros faz:**

* Salva as imagens processadas no formato selecionado
* **Fan-out do LATTICE**: cada quadro bruto do LATTICE é exportado como todos os produtos habilitados em uma única passagem — debayering, pré-visualização, radianância (sempre float32), refletância
* Grava os arquivos na árvore de saída do projeto: `<project>/<camera>/<format>/<Product>_Images/`
* **Mantém o nome do arquivo de origem** — a pasta identifica o produto, nenhum sufixo é adicionado**Duração:** Varia de acordo com o formato de exportação e o tamanho do arquivo***

## Comportamento do processamento

### Pipeline de processamento automático

Uma vez iniciado, todo o pipeline é executado automaticamente:

* Não é necessária interação do usuário
* Todas as etapas configuradas são executadas em sequência
* Atualizações de progresso exibidas em tempo real
* Os arquivos exportados são gravados no disco à medida que são concluídos — você pode abrir os resultados finalizados enquanto a execução continua

### Uso do computador durante o processamento

**Modo Livre:**

* Uso relativamente baixo da CPU (single-threaded)
* O computador permanece responsivo para outras tarefas
* É seguro minimizar o Chloros e trabalhar em outros aplicativos

**Modo Paralelo do Chloros+:**

* Alto uso da CPU em todo o conjunto de trabalhadores da estratégia
* Com aceleração por GPU: alto uso da GPU
* O computador pode ficar menos responsivo durante o processamento
* Evite iniciar outras tarefas que exijam muito da CPU

{% hint style="warning" %}
**Dica de desempenho**: Para obter o melhor desempenho do Chloros+, feche outros aplicativos e permita que o Chloros utilize todos os recursos do sistema.
{% endhint %}

### O processamento não pode ser pausado (mas a interrupção é definitiva)

* Uma vez iniciado, o processamento não pode ser pausado e retomado posteriormente
* Clicar em **Parar** encerra a execução de forma definitiva logo no primeiro clique
* Os produtos já exportados antes da interrupção permanecem no disco
* Uma execução interrompida relata com precisão o que foi concluído (consulte as linhas `[RUN-SUMMARY]` no log)
* Uma nova execução reinicia o pipeline desde o início

**Dica de planejamento:** Para projetos muito grandes, considere processar em lotes ou usar a CLI para obter melhor controle.***

## Monitorando seu processamento

Enquanto o processamento está em andamento, você pode:

* **Observar a barra de progresso** — Veja a porcentagem geral de conclusão
* **Visualizar o estágio atual** — Detecção, Análise, Calibração ou Exportação
* **Verificar a aba de log** — Veja mensagens detalhadas de processamento e avisos
* **Visualizar imagens concluídas** — Os arquivos exportados aparecem no disco durante o processamento

Para informações detalhadas sobre monitoramento, consulte [Monitorando o processamento](monitoring-the-processing.md).

***

## Interrompendo o processamento

Se você precisar interromper o processamento:

### Como interromper

1. Localize o **botão Interromper** (substitui o botão “Iniciar” durante o processamento)
2. Clique nele uma vez — a barra exibe **“Interrompendo...”** enquanto a imagem em processamento é finalizada
3. A execução termina em um estado de interrupção definitiva e o log gera um relatório detalhado `[RUN-SUMMARY]` do que foi concluído

### Quando interromper

**Motivos válidos para interromper:**

* Percebeu-se que foram utilizadas configurações incorretas
* Esqueceu-se de marcar as imagens-alvo
* Imagens erradas foram importadas
* O sistema está muito lento ou não responde

**Após a interrupção:**

* Os produtos exportados antes da interrupção permanecem no disco
* Revise e corrija quaisquer problemas, ajuste as configurações conforme necessário
* Reinicie o processamento — a execução recomeça do início

***

## Estimativas de tempo de processamento

O tempo real de processamento varia muito de acordo com:

* Número de imagens
* Resolução das imagens
* Formato de entrada RAW vs. JPG
* Modo de processamento (Gratuito vs. Chloros+)
* Velocidade da CPU e número de núcleos
* Disponibilidade da GPU (somente Chloros+)
* Número de índices a serem calculados
* Número de produtos de exportação habilitados (LATTICE)

### Estimativas aproximadas (Chloros+, imagens de 12 MP, CPU moderna)

| Número de imagens | Modo Gratuito | Chloros+ (CPU) | Chloros+ (GPU) |
| ----------- | --------- | -------------- | -------------- |
| 50 imagens   | 15-20 min | 5-8 min        | 3-5 min        |
| 100 imagens  | 30-40 min | 10-15 min      | 5-8 min        |
| 200 imagens  | 1–1,5 horas | 20–30 min      | 10–15 min      |
| 500 imagens  | 2–3 horas   | 45–60 min      | 20–30 min      |
| 1.000 imagens | 4 a 6 horas   | 1,5 a 2 horas      | 40 a 60 min      |

{% hint style="info" %}
**Primeira execução**: O processamento inicial pode demorar mais, pois o Chloros cria caches e perfis. O processamento subsequente de conjuntos de dados semelhantes será mais rápido.
{% endhint %}

***

## Problemas comuns na inicialização

### Botão Iniciar desativado (desativado, em cinza)

**Possíveis causas:**

* Nenhuma imagem importada
* Backend não totalmente iniciado
* Processamento anterior ainda em execução
* Projeto não totalmente carregado

**Soluções:**

1. Aguarde até que o backend seja totalmente inicializado (verifique o ícone no menu principal)
2. Verifique se as imagens foram importadas no Navegador de Arquivos
3. Reinicie o Chloros se o botão continuar desativado
4. Verifique o Log de Depuração em busca de mensagens de erro

### O processamento inicia, mas falha imediatamente

**Possíveis causas:**

* Não há imagens válidas no projeto
* Arquivos de imagem corrompidos
* Espaço em disco insuficiente
* Memória insuficiente (RAM)

**Soluções:**

1. Verifique o Log de Depuração <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> em busca de mensagens de erro
2. Verifique o espaço disponível em disco
3. Tente processar um subconjunto menor de imagens
4. Verifique se as imagens não estão corrompidas

### A execução é concluída, mas nenhuma imagem é gravada

Uma execução que solicitou produtos de imagem, mas não gravou nenhum, é tratada como uma **falha, não como um sucesso** — o Chloros relata isso de forma bem clara:

* O log da GUI exibe mensagens do tipo `[RUN-SUMMARY]` indicando a causa provável — nenhuma imagem importada, nenhum alvo detectado ou todos os produtos solicitados ignorados por serem inaplicáveis (por exemplo, solicitar radianância/refletância de câmeras que capturam apenas RGB)
* O equivalente na CLI (`chloros-cli process`) exibe `Processing finished but wrote no image products.` e **retorna um valor diferente de zero**, para que os scripts possam detectá-lo
* Uma execução deliberada apenas com metadados (todos os produtos de exportação desativados, sem índices) ainda é considerada um sucesso

Consulte [a Referência da CLI](../reference/cli-reference.md#a-run-that-writes-no-images-fails) para a semântica completa.

### A pasta do projeto está muito profunda para este computador

**Somente Windows, novidade na versão 1.3.0.** O Windows vem com caminhos longos desativados, e, portanto, nenhum caminho de arquivo pode ter mais de 260 caracteres. Os produtos da Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto; portanto, uma pasta de projeto mais profunda do que isso não poderia receber seus produtos. Em vez de ser executado e não gravar nada, o Chloros agora verifica antes do início do processamento:

* Clicar em Iniciar não inicia uma execução: a barra de progresso mostra o motivo, e o Log de Depuração exibe `[PROCESS] refused: The project folder is too deep for this computer…`. A mensagem informa o comprimento da pasta do projeto e a pasta de projeto mais longa que cabe.
* A criação ou abertura de um projeto em uma pasta muito profunda, mesmo para os próprios arquivos do projeto, também é recusada. Uma caixa de mensagem explica o motivo, indicando a pasta de projeto mais longa que cabe e como habilitar caminhos longos, e nada é criado ou aberto; o Log de Depuração registra isso como `[PROJECT] refused: …`. Duplicar um projeto com um nome mais longo também pode ultrapassar o limite: a cópia é feita, mas não é aberta, e a mesma mensagem explica o motivo.

**Soluções:**

1. Mantenha os projetos em uma pasta com nome mais curto — altere o**Diretório de Trabalho** em [Configurações do Projeto](../project-settings/project-settings.md) e crie o projeto novamente, ou
2. Ative os caminhos longos do Windows e reinicie o Chloros: execute novamente o instalador do Chloros com a opção **Ativar caminhos longos do Windows** marcada (ela vem marcada por padrão), ou use a Política de Grupo: *Configuração do Computador &gt; Modelos Administrativos &gt; Sistema &gt; Sistema de Arquivos &gt; Ativar caminhos longos do Win32*.

### Aviso “Nenhum alvo detectado”

**Possíveis causas:**

* Esqueceu de marcar as imagens-alvo
* As imagens-alvo não contêm alvos visíveis
* Configurações de detecção de alvos muito restritas

**Soluções:**

1. Consulte [Escolhendo imagens-alvo](choosing-target-images.md)
2. Marque as imagens apropriadas na coluna “Alvo”
3. Verifique se os alvos estão visíveis nas imagens marcadas
4. Ajuste as configurações de detecção de alvos, se necessário

***

## Dicas para um processamento bem-sucedido

### Antes de começar

1. **Teste primeiro com um pequeno subconjunto** — Processe de 10 a 20 imagens para verificar as configurações
2. **Verifique o espaço disponível em disco** — Garanta que haja espaço livre equivalente a 2 a 3 vezes o tamanho do conjunto de dados (mais se todos os produtos LATTICE estiverem ativados)
3. **Feche aplicativos desnecessários** — Libere recursos do sistema
4. **Verifique as imagens dos alvos** — Visualize os alvos marcados para garantir a qualidade
5. **Salve o projeto** — O projeto é salvo automaticamente, mas é recomendável salvá-lo manualmente

### Durante o processamento

1. **Evite que o sistema entre em modo de suspensão** — Desative os modos de economia de energia
2. **Mantenha o Chloros em primeiro plano** — Ou, pelo menos, visível na barra de tarefas
3. **Monitore o andamento ocasionalmente** — Verifique se há avisos ou erros
4. **Não carregue outros aplicativos pesados** — Especialmente no modo paralelo do Chloros+

### Aceleração por GPU do Chloros+

Se estiver usando aceleração por GPU da NVIDIA:

1. Atualize os drivers da NVIDIA para a versão mais recente
2. Certifique-se de que a GPU tenha 4 GB ou mais de VRAM (7 GB ou mais para debayering simultâneo com reconhecimento de textura)
3. Feche aplicativos que exigem muito da GPU (jogos, edição de vídeo)
4. Monitore a temperatura da GPU (garanta um resfriamento adequado)

***

## Próximos passos

Assim que o processamento for iniciado:

1. **Monitore o andamento** — Consulte [Monitorando o processamento](monitoring-the-processing.md)
2. **Aguarde a conclusão** — O processamento é executado automaticamente
3. **Analise os resultados** — Consulte [Concluindo o processamento](finishing-the-processing.md)

Para obter informações sobre o que fazer durante o processamento, consulte [Monitorando o processamento](monitoring-the-processing.md).
