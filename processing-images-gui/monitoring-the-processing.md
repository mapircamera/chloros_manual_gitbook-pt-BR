# Monitoramento do processamento

Assim que o processamento for iniciado, o Chloros oferece várias maneiras de monitorar o andamento, verificar se há problemas e entender o que está acontecendo com seu conjunto de dados. Esta página explica como acompanhar seu processamento e interpretar as informações fornecidas pelo Chloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o status do processamento em tempo real e a porcentagem de conclusão. O progresso é transmitido ao vivo do backend por meio de Server-Sent Events (SSE), de modo que a barra reflete o que o pipeline está realmente fazendo.

### Barra de progresso no modo gratuito

Para usuários sem licença dChloros

+:

**Exibição de progresso em 2 etapas:**

1.**Detecção de alvos** — Localização de alvos de calibração nas imagens
2. **Processamento** — Aplicação de correções e exportação**A barra de progresso mostra:**

* Porcentagem geral de conclusão (0-100%)
* Nome da etapa atual
* Visualização simples em forma de barra horizontal

### Barra de Progresso do Chloros+

Para usuários com licença do Chloros+:

**Exibição de progresso em 4 etapas:**

1.**Detecção** – Localização de alvos de calibração
2. **Análise** – Exame das imagens e preparação do pipeline
3. **Calibração** – Aplicação de correções de vinheta e refletância
4. **Exportação** – Salvamento dos arquivos processados**Recursos interativos:*** **Passe o mouse** sobre a barra de progresso para ver o painel expandido de 4 etapas
* **Clique** na barra de progresso para congelar/fixar o painel expandido
* **Clique novamente** para desbloquear e ocultar automaticamente ao afastar o mouse
* Cada etapa mostra o progresso individual (0-100%)

{% hint style="info" %}
**Paridade com a CLI**: durante uma execução do `chloros-cli process`, as mesmas quatro threads são exibidas como “Detectando”, “Analisando”, “Processando” e “Exportando”, e o `chloros-cli export-status` mostra o progresso da exportação da Thread-4 em tempo real a partir de outro terminal. Consulte a [Referência da CLI](../reference/cli-reference.md).
{% endhint %}

***

## Entendendo cada etapa do processamento

{% hint style="info" %}
**Arquitetura de pipeline**: Essas 4 etapas da GUI correspondem ao [pipeline de processamento de 4 threads](../processing-architecture/processing-pipeline.md). Em sistemas com aceleração por GPU, a Thread 3 (Calibração) se beneficia da [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md), que otimiza o processamento para o seu hardware específico.
{% endhint %}

### Etapa 1: Detecção (Detecção de Alvo)

**O que está acontecendo:**

* O Chloros examina as imagens que você marcou com a caixa de seleção “Alvo” (todas as imagens, caso nenhuma esteja marcada)
* Algoritmos de visão computacional identificam os painéis de calibração
* Valores de refletância extraídos de cada painel
* Carimbos de data e hora dos alvos registrados para o agendamento adequado da calibração

**Duração:**

* Com alvos marcados: 10 a 60 segundos
* Sem alvos marcados: 5 a 30+ minutos (analisa todas as imagens)

**Indicador de progresso:**

* Detectando: 0% → 100%
* Número de imagens analisadas (conta apenas as imagens que estão realmente sendo analisadas)
* Contagem de alvos encontrados

**O que observar:**

* Deve ser concluído rapidamente se os alvos estiverem devidamente marcados
* Se demorar muito, os alvos podem não estar marcados
* Verifique o Log de Depuração em busca de mensagens do tipo “Alvo encontrado”

### Etapa 2: Análise

**O que está acontecendo:**

* Leitura dos metadados EXIF das imagens (carimbos de data/hora, configurações de exposição)
* Determinação da estratégia de calibração com base nos carimbos de data/hora dos alvos e nos dados de radiação descendente do DAQ disponíveis
* Organização da fila de processamento de imagens
* Preparação dos trabalhadores de processamento paralelo (Somente Chloros+)

**Duração:** 5 a 30 segundos**Indicador de progresso:**

* Analisando: 0% → 100%
* Etapa rápida, geralmente é concluída rapidamente

**O que observar:**

* O progresso deve ocorrer de forma constante, sem pausas
* Avisos sobre metadados ausentes aparecerão no Log de Depuração

### Etapa 3: Calibração

**O que está acontecendo:*** **Debayering**: Conversão do padrão RAW Bayer em 3 canais (ignorada para módulos LATTICE mono, com uma observação)
* **Correção de vinheta**: Remoção do escurecimento nas bordas da lente
* **Calibração de refletância**: Normalização com valores-alvo e/ou downwelling do DAQ
* **Cálculo de índices**: Cálculo de índices multiespectrais
* Processamento de cada imagem por todo o pipeline

**Duração:** A maior parte do tempo total de processamento (60-80%)**Indicador de progresso:**

* Calibrando: 0% → 100%
* Imagem atual sendo processada
* Imagens concluídas / Total de imagens

**Comportamento do processamento:*** **Modo livre**: Processa uma imagem por vez, sequencialmente
* **Modo Chloros+**: Executa um conjunto de trabalhadores adaptável ao hardware — de 1 a 4 trabalhadores simultâneos em sistemas com GPU (dependendo da VRAM), um trabalhador por núcleo físico (menos um) em sistemas apenas com CPU. Consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md)
* **Aceleração por GPU**: Acelera significativamente esta etapa**O que observar:**

* Progresso constante na contagem de imagens
* Verifique o Log de Depuração para mensagens de conclusão por imagem
* Avisos sobre problemas de qualidade de imagem ou calibração

### Etapa 4: Exportação

**O que está acontecendo:**

* Gravação das imagens processadas no disco no formato selecionado, à medida que são concluídas
* **LATTICE**: cada quadro é distribuído para todos os produtos habilitados (debayering / visualização / radiação / refletância)
* Exportação de imagens de índice multiespectral com cores LUT
* Criação da árvore de saída `<project>/<camera>/<format>/<Product>_Images/` — os arquivos exportados mantêm o nome do arquivo de origem; a pasta identifica o produto

**Duração:** 10-20% do tempo total de processamento**Indicador de progresso:**

* Exportando: 0% → 100%
* Arquivos sendo gravados
* Formato de exportação e destino

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de arquivos
* Conclusão de todas as saídas configuradas

***

## Guia “Log de depuração”

O “Log de depuração” fornece informações detalhadas sobre o andamento do processamento e quaisquer problemas encontrados. As mensagens de inicialização do backend também são reproduzidas no console de log; portanto, o log conta toda a história, mesmo que você o abra posteriormente.

### Acessando o Log de Depuração

1. Clique no ícone **Log de Depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line">

na barra lateral esquerda
2. O painel de log é aberto, exibindo mensagens de processamento em tempo real
3. A rolagem automática exibe as mensagens mais recentes

<!-- SCREENSHOT-NEEDED: Debug Log tab open at the end of a completed run, showing real backend log lines including the [RUN-SUMMARY] lines (images / camera groups / targets / calibrated / files written), with the 1.3.0 bug icon (Report a bug) visible at the top of the tab -->

### Entendendo as mensagens do log As linhas do log do

Chloros são precedidas por tags entre colchetes que identificam o subsistema — por exemplo, `[PROCESSING]`, `[RUN-SUMMARY]`, `[LATTICE-EXPORT]`, `[EXPORT-CHECK]`, `[IMPORT-LEVEL]`. O mais importante a se saber é o **resumo da execução**, exibido no final de cada execução (incluindo as interrompidas):

```
[RUN-SUMMARY] 49 image(s) in 2 camera group(s); 4 target(s) detected; 45 image(s) calibrated; 180 file(s) written.
```

Linhas de dica adicionais, como `[RUN-SUMMARY]`, aparecem sempre que algo precisa ser explicado — por exemplo, uma execução que não produziu nada, ou uma câmera cujo produto solicitado foi ignorado por ser considerado inaplicável. As linhas `[EXPORT-CHECK]` explicam as omissões por câmera (por exemplo, por que uma câmera RGB não obteve nenhum produto de radiação).

Os níveis de gravidade gerais das mensagens (os exemplos abaixo são ilustrativos, não literais):

#### Mensagens de Informação (Branco/Cinza)

Atualizações normais do processamento: processamento iniciado, alvos detectados (com contagem de painéis), progresso da calibração por imagem, arquivos exportados, processamento concluído.

#### Mensagens de aviso (amarelas)

Problemas não críticos que não interrompem o processamento — por exemplo, dados de GPS ausentes em um quadro, uma grande lacuna de data e hora entre imagens-alvo ou baixo contraste em um painel de calibração.

**Ação:** Revise os avisos após o processamento, mas não interrompa

#### Mensagens de erro (em Red)

Problemas críticos que podem causar falha no processamento — por exemplo, disco cheio, um arquivo de imagem corrompido ou nenhum alvo detectado enquanto a calibração de refletância foi solicitada.

**Ação:** Interrompa o processamento, resolva o erro e reinicie

### Situações comuns no log

| Situação                             | Significado                                       | Ação necessária                                         |
| ------------------------------------- | --------------------------------------------- | ----------------------------------------------------- |
| Alvo detectado em \[nome do arquivo]        | Alvo de calibração encontrado com sucesso         | Nenhuma — normal                                         |
| Linhas de progresso por imagem              | Atualização do progresso atual                       | Nenhuma — normal                                         |
| Nenhum alvo encontrado                      | Nenhum alvo de calibração detectado               | Marcar imagens de alvo ou desativar a calibração de refletância |
| Espaço em disco insuficiente               | Armazenamento insuficiente para saída                 | Liberar espaço em disco                                    |
| Ignorando arquivo corrompido               | O arquivo de imagem está danificado                         | Recopie o arquivo do cartão SD                             |
| `[IMPORT-LEVEL] Skipping ... no raw source` | Uma captura sem um quadro raw não pode ser processada | Recapture com raw, ou use a CLI `--input-level`  |
| `[RUN-SUMMARY] ... 0 file(s) written` | A execução não produziu nenhum produto de imagem — relatado como uma falha com dicas | Leia as linhas de dicas; verifique o que foi ignorado e por quê |
| `[PROCESS] refused: The project folder is too deep for this computer…` | Os caminhos longos do Windows estão desativados e a pasta do projeto está muito profunda para o limite de 260 caracteres do caminho; nada foi processado | Use uma pasta de projeto mais curta (a mensagem indica o caminho mais longo que cabe), ou ative os caminhos longos do Windows e reinicie o Chloros — consulte [Iniciando o processamento](starting-the-processing.md#the-project-folder-is-too-deep-for-this-computer) |
| `[unmix-clamp-mask] …` | Não foi possível gravar a máscara de clamp de um produto LATTICE; o produto em si está correto e exibe a mensagem `Chloros:UnmixClampMask=unavailable` | Verifique o espaço em disco e as permissões da pasta e, em seguida, reprocesse se precisar da máscara |

### Copiando dados do log

Para copiar o log para fins de solução de problemas ou suporte:

1. Abra o painel “Log de depuração”
2. Clique no botão **Copiar**(copia o log para a área de transferência) ou em**Baixar** para salvá-lo como um arquivo de texto
3. Cole em um arquivo de texto ou em um e-mail
4. Envie para o suporte da MAPIR, se necessário

### Relatando um bug na guia “Log de depuração”

Novidade na versão 1.3.0: o ícone de bug (**Relatar um bug**) na parte superior da guia “Log de depuração” envia um relatório diretamente para a MAPIR.

1. Clique no ícone de bug na parte superior da guia “Log de depuração”
2. Descreva o problema: o que você estava fazendo, o que esperava e o que aconteceu na verdade (obrigatório)
3. Opcionalmente, marque **Adicionar algumas imagens (até 10)**e selecione imagens do projeto aberto, ou use**Procurar...** para qualquer outro arquivo de imagem
4. Clique em **Enviar**

O relatório inclui sua descrição, quaisquer imagens anexadas, os logs do Chloros e detalhes do sistema (versão do Chloros, sistema operacional, GPU e configurações do projeto). Você precisa estar conectado ao Chloros (aba “Usuário”) para enviar o relatório. Você pode fechar a janela enquanto o envio ocorre; quando terminar, uma referência como `Bug report sent. Reference: BR-…` será registrada no log — mencione-a ao entrar em contato com a MAPIR. Se o relatório não puder ser enviado (por exemplo, por falta de conexão com a internet), o Chloros o salva no seu computador e mostra o caminho para que você possa enviá-lo por e-mail para support@mapir.camera.

***

## Monitoramento de Recursos do Sistema

### Utilização da CPU

**Modo Livre:**

* 1 núcleo da CPU a ~100%
* Outros núcleos ociosos ou disponíveis
* O sistema continua responsivo

**Modo Paralelo do Chloros+:**

* Vários núcleos com alta utilização — o número depende da estratégia escolhida pela [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md)
* O sistema pode parecer menos responsivo

**Para monitorar:**

* Gerenciador de Tarefas do Windows (Ctrl+Shift+Esc)
* Guia Desempenho → seção CPU
* Procure pelos processos “Chloros” ou “chloros-backend”

### Utilização de memória (RAM)

**Utilização típica:**

* Projetos pequenos (< 100 imagens): 2–4 GB
* Projetos médios (100–500 imagens): 4–8 GB
* Projetos grandes (mais de 500 imagens): 8–16 GB
* O modo paralelo do Chloros+ utiliza mais RAM

**Se a memória estiver baixa:**

* Processe lotes menores
* Feche outros aplicativos
* Aumente a RAM se processar regularmente grandes conjuntos de dados

### Uso da GPU (Chloros+ com CUDA)

Quando a aceleração por GPU está ativada:

* A GPU NVIDIA apresenta alta utilização (60–90%)
* O uso de VRAM aumenta (requer 4 GB ou mais de VRAM; 7 GB ou mais para debayering simultâneo com reconhecimento de textura)
* A etapa de calibração fica significativamente mais rápida

**Para monitorar:**

* Ícone da NVIDIA na bandeja do sistema
* Gerenciador de Tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitoramento semelhante

### E/S de disco

**O que esperar:**

* Alta taxa de leitura do disco durante a etapa de análise
* Alta taxa de gravação do disco durante a etapa de exportação
* SSD significativamente mais rápido que HDD

**Dica de desempenho:**

* Use um SSD para a pasta do projeto sempre que possível
* Evite unidades de rede para conjuntos de dados grandes
* Certifique-se de que o disco não esteja quase cheio (isso afeta a velocidade de gravação)

***

## Detecção de problemas durante o processamento

### Sinais de alerta

**O progresso fica estagnado (sem alterações por mais de 5 minutos):**

* Verifique o log de depuração em busca de erros
* Verifique o espaço disponível em disco
* Verifique o Gerenciador de Tarefas para garantir que o Chloros esteja em execução

**Mensagens de erro aparecem com frequência:**

* Interrompa o processamento e analise os erros
* Causas comuns: espaço em disco, arquivos corrompidos, problemas de memória
* Consulte a seção “Solução de problemas” abaixo

**O sistema deixa de responder:**

* O modo paralelo do Chloros está consumindo muitos recursos
* Considere reduzir o número de tarefas simultâneas ou atualizar o hardware
* O modo livre consome menos recursos

### Quando interromper o processamento

Interrompa o processamento se observar:

* ❌ Erros do tipo “Disco cheio” ou “Não é possível gravar o arquivo”
* ❌ Erros repetidos de corrupção de arquivos de imagem
* ❌ Sistema completamente travado (sem resposta)
* ❌ Percebeu que foram configuradas opções incorretas
* ❌ Imagens erradas importadas

**Como interromper:**

1. Clique no**botão Interromper** (que substitui o botão Iniciar) — basta uma vez
2. A barra exibe “Interrompendo...” enquanto a imagem em processamento é concluída; em seguida, a execução termina no estado interrompido
3. Os produtos já exportados permanecem no disco; o log exibe um código `[RUN-SUMMARY]` que indica o que foi concluído
4. Corrija os problemas e reinicie — a execução recomeça do início

***

## Solução de problemas durante o processamento

### O processamento está muito lento

**Possíveis causas:**

* Imagens-alvo não marcadas (digitalização de todas as imagens)
* Armazenamento em HDD em vez de SSD
* Recursos do sistema insuficientes
* Muitos índices configurados
* Acesso a unidade de rede

**Soluções:**

1. Se acabou de iniciar e está na fase de detecção: interrompa, marque os alvos e reinicie
2. Para o futuro: use SSD, reduza o número de índices, atualize o hardware
3. Considere usar a CLI para processamento em lote de grandes conjuntos de dados

### Avisos de “Espaço em disco”

**Soluções:**

1. Libere espaço em disco imediatamente
2. Mova o projeto para uma unidade com mais espaço
3. Reduza o número de índices a serem exportados
4. Desative os produtos de exportação do LATTICE que você não precisa (Configurações do projeto → Processamento)
5. Use o formato JPG em vez de TIFF (arquivos menores)

### Mensagens frequentes de “Arquivo corrompido”

**Soluções:**

1. Recopie as imagens do cartão SD para garantir a integridade
2. Verifique se há erros no cartão SD
3. Remova os arquivos corrompidos do projeto
4. Continue processando as imagens restantes

### Superaquecimento / Limitação do sistema

**Soluções:**

1. Garanta ventilação adequada
2. Limpe a poeira das aberturas de ventilação do computador
3. Reduza a carga de processamento (use o modo “Free” em vez do “Chloros+”)
4. Processe durante os horários mais frescos do dia

***

## Notificação de processamento concluído

Quando o processamento terminar:

* A barra de progresso chega a 100%
* As linhas `[RUN-SUMMARY]` aparecem no Log de Depuração com as contagens finais
* O botão Iniciar fica ativo novamente
* Todos os arquivos de saída estão na árvore de saída do projeto, organizada por câmera: `<project>/<camera>/<format>/<Product>_Images/`

***

## Próximos passos

Assim que o processamento for concluído:

1. **Analise os resultados** — Consulte [Concluindo o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** — Confirme se todos os arquivos foram exportados corretamente
3. **Analise o log de depuração** — Verifique se há avisos ou erros
4. **Visualize as imagens processadas** — Use o Visualizador de Imagens ou um software externo

Para obter informações sobre como analisar e utilizar os resultados processados, consulte [Concluindo o processamento](finishing-the-processing.md).
