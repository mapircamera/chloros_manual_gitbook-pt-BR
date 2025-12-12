# Monitoramento do processamento

Depois que o processamento é iniciado, o Chloros oferece várias maneiras de monitorar o progresso, verificar se há problemas e entender o que está acontecendo com seu conjunto de dados. Esta página explica como acompanhar seu processamento e interpretar as informações fornecidas pelo Chloros.

## Visão geral da barra de progresso

A barra de progresso no cabeçalho superior mostra o status do processamento em tempo real e a porcentagem de conclusão.

### Barra de progresso do modo gratuito

Para usuários sem licença do Chloros+:

**Exibição do progresso em duas etapas:**

1. **Detecção de alvo** - Localização de alvos de calibração nas imagens
2. **Processamento** - Aplicação de correções e exportação

**A barra de progresso mostra:**

* Porcentagem geral de conclusão (0-100%)
* Nome da etapa atual
* Visualização simples em barra horizontal

### Barra de progresso do Chloros+

Para usuários com licença Chloros+:

**Exibição de progresso em 4 etapas:**

1. **Detecção** - Localização de alvos de calibração
2. **Análise** - Exame de imagens e preparação do pipeline
3. **Calibração** - Aplicação de correções de vinheta e refletância
4. **Exportação** - Salvamento de arquivos processados

**Recursos interativos:**

* **Passe o mouse sobre** a barra de progresso para ver o painel expandido de 4 estágios
* **Clique** na barra de progresso para congelar/fixar o painel expandido
* **Clique novamente** para descongelar e ocultar automaticamente ao sair com o mouse
* Cada estágio mostra o progresso individual (0-100%)

***

## Entendendo cada estágio do processamento

### Estágio 1: Detecção (detecção de alvo)

**O que está acontecendo:**

* O Chloros digitaliza imagens marcadas com a caixa de seleção Alvo
* Algoritmos de visão computacional identificam os 4 painéis de calibração
* Valores de refletância extraídos de cada painel
* Registros de data e hora dos alvos para programação adequada da calibração

**Duração:**

* Com alvos marcados: 10-60 segundos
* Sem alvos marcados: 5-30+ minutos (digitaliza todas as imagens)

**Indicador de progresso:**

* Detectando: 0% → 100%
* Número de imagens digitalizadas
* Contagem de alvos encontrados

**O que observar:**

* Deve ser concluído rapidamente se os alvos estiverem devidamente marcados
* Se demorar muito, os alvos podem não estar marcados
* Verifique o Log de Depuração para mensagens “Alvo encontrado”

### Etapa 2: Análise

**O que está acontecendo:**

* Lendo metadados EXIF da imagem (carimbos de data/hora, configurações de exposição)
* Determinando a estratégia de calibração com base nos carimbos de data/hora dos alvos
* Organizando a fila de processamento de imagens
* Preparando os trabalhadores de processamento paralelo (somente Chloros+)

**Duração:** 5 a 30 segundos

**Indicador de progresso:**

* Analisando: 0% → 100%
* Etapa rápida, geralmente concluída rapidamente

**O que observar:**

* Deve progredir de forma constante, sem pausas
* Avisos sobre metadados ausentes aparecerão no Log de Depuração

### Etapa 3: Calibrando

**O que está acontecendo:**

* **Debayering**: Convertendo o padrão RAW Bayer para 3 canais
* **Correção de vinheta**: Remoção do escurecimento das bordas da lente
* **Calibração de refletância**: Normalização com valores-alvo
* **Cálculo de índice**: Cálculo de índices multiespectrais
* Processamento de cada imagem através do pipeline completo

**Duração:** Maioria do tempo total de processamento (60-80%)

**Indicador de progresso:**

* Calibração: 0% → 100%
* Imagem atual sendo processada
* Imagens concluídas / Total de imagens

**Comportamento do processamento:**

* **Modo livre**: processa uma imagem por vez sequencialmente
* **Modo Chloros+**: processa até 16 imagens simultaneamente
* **Aceleração da GPU**: acelera significativamente esta etapa

**O que observar:**

* Progresso constante através da contagem de imagens
* Verifique o Log de Depuração para mensagens de conclusão por imagem
* Avisos sobre qualidade da imagem ou problemas de calibração

### Etapa 4: Exportação

**O que está acontecendo:**

* Gravação de imagens calibradas no disco no formato selecionado
* Exportação de imagens de índice multiespectral com cores LUT
* Criação de subpastas do modelo da câmera
* Preservação dos nomes de arquivos originais com sufixos apropriados

**Duração:** 10-20% do tempo total de processamento

**Indicador de progresso:**

* Exportação: 0% → 100%
* Arquivos sendo gravados
* Formato e destino da exportação

**O que observar:**

* Avisos de espaço em disco
* Erros de gravação de arquivo
* Conclusão de todas as saídas configuradas

***

## Guia Log de depuração

O Log de depuração fornece informações detalhadas sobre o andamento do processamento e quaisquer problemas encontrados.

### Acessando o Log de depuração

1. Clique no ícone **Log de depuração** <img src="../.gitbook/assets/icon_log.JPG" alt="" data-size="line"> na barra lateral esquerda
2. O painel de log é aberto, exibindo mensagens de processamento em tempo real
3. A rolagem automática exibe as mensagens mais recentes

### Entendendo as mensagens de log

#### Mensagens de informação (branco/cinza)

Atualizações normais de processamento:

```
[INFO] Processing started
[INFO] Target detected in IMG_0015.RAW - 4 panels found
[INFO] Calibrating IMG_0234.RAW
[INFO] Exported NDVI image: IMG_0234_NDVI.tif
[INFO] Processing complete
```

#### Mensagens de aviso (amarelo)

Problemas não críticos que não interrompem o processamento:

```
[WARN] No GPS data found in IMG_0145.RAW
[WARN] Target image timestamp gap > 30 minutes
[WARN] Low contrast in calibration panel - results may vary
```

**Ação:** Revise os avisos após o processamento, mas não interrompa

#### Mensagens de erro (Red)

Problemas críticos que podem causar falha no processamento:

```
[ERROR] Cannot write file - disk full
[ERROR] Corrupted image file: IMG_0299.RAW
[ERROR] No targets detected - enable reflectance calibration or mark target images
```

**Ação:** Interrompa o processamento, resolva o erro e reinicie.

### Mensagens comuns do log

| Mensagem                          | Significado                                | Ação necessária                                         |
| -------------------------------- | -------------------------------------- | ----------------------------------------------------- |
| “Alvo detectado em \[nome do arquivo]” | Alvo de calibração encontrado com sucesso  | Nenhuma - normal                                         |
| “Processando imagem X de Y”        | Atualização do progresso atual                | Nenhuma - normal                                         |
| “Nenhum alvo encontrado”               | Nenhum alvo de calibração detectado        | Marque as imagens de alvo ou desative a calibração de refletância |
| “Espaço em disco insuficiente”        | Armazenamento insuficiente para saída          | Libere espaço em disco                                    |
| “Ignorando arquivo corrompido”        | Arquivo de imagem está danificado                  | Recopie o arquivo do cartão SD                             |
| “Dados PPK aplicados”               | Correções GPS do arquivo .daq aplicadas | Nenhum - normal                                         |

### Copiando dados de log

Para copiar o log para solução de problemas ou suporte:

1. Abra o painel Log de depuração
2. Clique no botão **“Copiar log”** (ou clique com o botão direito → Selecionar tudo)
3. Cole em um arquivo de texto ou e-mail
4. Envie para o suporte MAPIR, se necessário

***

## Monitoramento de recursos do sistema

### Uso da CPU

**Modo livre:**

* 1 núcleo da CPU a ~100%
* Outros núcleos ociosos ou disponíveis
* O sistema permanece responsivo

**Chloros+ Modo paralelo:**

* Vários núcleos a 80-100% (até 16 núcleos)
* Alta utilização geral da CPU
* O sistema pode parecer menos responsivo

**Para monitorar:**

* Gerenciador de Tarefas do Windows (Ctrl+Shift+Esc)
* Guia Desempenho → seção CPU
* Procure pelos processos “Chloros” ou “chloros-backend”

### Uso da memória (RAM)

**Uso típico:**

* Projetos pequenos (&lt; 100 imagens): 2-4 GB
* Projetos médios (100-500 imagens): 4-8 GB
* Projetos grandes (mais de 500 imagens): 8-16 GB
* O modo paralelo Chloros+ usa mais RAM

**Se a memória estiver baixa:**

* Processe lotes menores
* Feche outros aplicativos
* Atualize a RAM se processar regularmente grandes conjuntos de dados

### Uso da GPU (Chloros+ com CUDA)

Quando a aceleração da GPU está ativada:

* A GPU NVIDIA apresenta alta utilização (60-90%)
* O uso da VRAM aumenta (requer 4 GB+ de VRAM)
* A etapa de calibração é significativamente mais rápida

**Para monitorar:**

* Ícone da bandeja do sistema NVIDIA
* Gerenciador de tarefas → Desempenho → GPU
* GPU-Z ou ferramenta de monitoramento semelhante

### E/S do disco

**O que esperar:**

* Alta leitura do disco durante a fase de análise
* Alta gravação do disco durante a fase de exportação
* SSD significativamente mais rápido que HDD

**Dica de desempenho:**

* Use SSD para a pasta do projeto, quando possível
* Evite unidades de rede para grandes conjuntos de dados
* Certifique-se de que o disco não esteja perto da capacidade máxima (afeta a velocidade de gravação)

***

## Detectando problemas durante o processamento

### Sinais de alerta

**O progresso fica parado (sem alterações por mais de 5 minutos):**

* Verifique se há erros no log de depuração
* Verifique o espaço disponível em disco
* Verifique o Gerenciador de Tarefas para garantir que o Chloros esteja em execução

**Mensagens de erro aparecem com frequência:**

* Interrompa o processamento e revise os erros
* Causas comuns: espaço em disco, arquivos corrompidos, problemas de memória
* Consulte a seção Solução de problemas abaixo

**O sistema fica sem resposta:**

* O modo paralelo Chloros+ está usando muitos recursos
* Considere reduzir as tarefas simultâneas ou atualizar o hardware
* O modo livre consome menos recursos

### Quando interromper o processamento

Interrompa o processamento se você observar:

* ❌ Erros “Disco cheio” ou “Não é possível gravar o arquivo”
* ❌ Erros repetidos de corrupção de arquivos de imagem
* ❌ Sistema completamente travado (não responde)
* ❌ Percebeu que configurações erradas foram definidas
* ❌ Imagens erradas importadas

**Como interromper:**

1. Clique no **botão Parar/Cancelar** (substitui o botão Iniciar)
2. O processamento é interrompido e o progresso é perdido
3. Corrija os problemas e reinicie do início

***

## Solução de problemas durante o processamento

### O processamento está muito lento

**Possíveis causas:**

* Imagens de destino não marcadas (digitalização de todas as imagens)
* Armazenamento em HDD em vez de SSD
* Recursos do sistema insuficientes
* Muitos índices configurados
* Acesso à unidade de rede

**Soluções:**

1. Se acabou de começar e está na fase de detecção: cancele, marque os alvos e reinicie
2. Para o futuro: use SSD, reduza os índices, atualize o hardware
3. Considere o CLI para processamento em lote de grandes conjuntos de dados

### Avisos de “Espaço em disco”

**Soluções:**

1. Libere espaço em disco imediatamente
2. Mova o projeto para uma unidade com mais espaço
3. Reduza o número de índices a exportar
4. Use o formato JPG em vez de TIFF (arquivos menores)

### Mensagens frequentes de “Arquivo corrompido”

**Soluções:**

1. Recopie as imagens do cartão SD para garantir a integridade
2. Teste o cartão SD para verificar se há erros
3. Remova os arquivos corrompidos do projeto
4. Continue processando as imagens restantes

### Superaquecimento/limitação do sistema

**Soluções:**

1. Garanta ventilação adequada
2. Limpe a poeira das aberturas de ventilação do computador
3. Reduza a carga de processamento (use o modo Livre em vez de Chloros+)
4. Processe durante os períodos mais frios do dia

***

## Notificação de processamento concluído

Quando o processamento terminar:

* A barra de progresso atinge 100%
* A mensagem **“Processamento concluído”** aparece no Log de depuração
* O botão Iniciar fica ativado novamente
* Todos os arquivos de saída estão na subpasta do modelo da câmera

***

## Próximas etapas

Quando o processamento for concluído:

1. **Revise os resultados** - Consulte [Concluindo o processamento](finishing-the-processing.md)
2. **Verifique a pasta de saída** - Verifique se todos os arquivos foram exportados corretamente
3. **Revise o log de depuração** - Verifique se há avisos ou erros
4. **Visualize as imagens processadas** - Use o Visualizador de Imagens ou um software externo

Para obter informações sobre como revisar e usar os resultados processados, consulte [Concluindo o processamento](finishing-the-processing.md).
