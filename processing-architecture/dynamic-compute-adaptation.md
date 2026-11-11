# Adaptação dinâmica de computação O

Chloros 1.3.0 utiliza detecção de hardware e seleção automática de estratégia de processamento. O mecanismo de processamento se adapta ao seu hardware — desde um Jetson Orin Nano até uma estação de trabalho com várias GPUs — sem qualquer configuração manual.

***

## Como funciona

Quando o Chloros é iniciado, ele faz um perfil do seu sistema:

1. **Detecta o sistema operacional** — Windows ou Linux
2. **Identifica os núcleos da CPU e a RAM total**

3.**Detecta a presença de GPU** — compatibilidade com NVIDIA CUDA, VRAM, modelo
4. **Identifica o modelo do Jetson** (se aplicável) — via `/proc/device-tree/model`
5. **Verifica os sensores térmicos** (Jetson) — para processamento sensível à temperatura
6. **Seleciona a estratégia de computação** — com base em todo o hardware detectado
7. **Configura automaticamente o número de workers, o tipo de pipeline e a alocação de memória**

O perfil detectado é armazenado em cache para a sessão na memória e no disco, para que as execuções posteriores sejam iniciadas mais rapidamente:

| Plataforma | Perfil em cache |
| --- | --- |
| **Linux / Jetson** | `~/.config/chloros/system_config.json` (prioriza `XDG_CONFIG_HOME`) |
| **Windows** | `%LOCALAPPDATA%\Chloros\config\system_config.json` |

Exclua esse arquivo para forçar uma nova detecção — útil após adicionar uma GPU ou mais memória RAM. O Chloros também realiza uma nova detecção automaticamente quando o cache foi gravado por uma versão anterior incompatível.

***

## Estratégias de computação

O Chloros seleciona uma das três estratégias de computação com base no seu hardware:

| Estratégia | Selecionada quando | Trabalhadores | Executor | Pipeline |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`**| GPU CUDA relatando**12 GB ou mais de VRAM**(na memória unificada do Jetson, também requer 12 GB+ de RAM compartilhada total) | `min(4, ⌊(VRAM − 0.5GB) × 0.9 ÷ 3.5GB⌋)`, mínimo de 2 —**limitado a 2 no Jetson** | Processos de trabalho (contextos CUDA separados) | `fused_gpu` |
| **`GPU_SINGLE`**| GPU CUDA com**2 a 12 GB de VRAM**| 3 (sobreposição de E/S; acesso à GPU serializado por um semáforo).**1 (sequencial) em Jetsons com menos de 12 GB de RAM** | Processos de trabalho (contextos CUDA separados); sequencial no próprio processo em Jetsons com pouca RAM | `fused_gpu` / `tiled_gpu` |
| **`CPU_PARALLEL`** | Sem GPU CUDA ou com menos de 2 GB de VRAM | `max(2, physical cores − 1)` | `ThreadPoolExecutor` | `cpu_fallback` |

Exemplos práticos da fórmula de trabalhadores do `GPU_PARALLEL`: 12 GB de VRAM → 2 trabalhadores, 16 GB → 3 trabalhadores, 20 GB ou mais → 4 trabalhadores. Cada worker recebe uma cota de 3,5 GB, a necessidade estimada para uma tarefa com reconhecimento de textura, dentro de 90% da VRAM restante após uma reserva de 0,5 GB.

{% hint style="info" %}
**As estratégias de GPU exigem um PyTorch capaz de utilizar CUDA.** Uma máquina cujo PyTorch não possua um dispositivo CUDA utilizável recebe o `CPU_PARALLEL`, independentemente de sua GPU. O pacote Jetson 1.3.0 utiliza a versão do PyTorch para CUDA do JetPack 6; portanto, um Jetson se qualifica assim que as bibliotecas CUDA e cuDNN do JetPack forem instaladas. Um Jetson com menos de 12 GB de RAM sempre processa uma imagem por vez, em processo (registrado como `GPU_SINGLE`).
{% endhint %}

As estratégias de GPU executam um conjunto de processos de trabalho, cada um sendo um processo separado do sistema operacional com seu próprio contexto CUDA, que o Chloros inicia ao executar novamente seu próprio programa de backend instalado (não por meio do `multiprocessing` do Python, cuja inicialização dos processos de trabalho não funciona dentro do aplicativo empacotado). A estratégia de CPU utiliza um conjunto de threads (`ThreadPoolExecutor` do Python). O Chloros não utiliza nenhuma estrutura distribuída de terceiros (como o Ray).

### Tipos de pipeline

* **`fused_gpu`** — Caminho de processamento totalmente na GPU. As operações de debayer, correção e indexação são executadas na GPU em uma única passagem combinada. Apresenta a maior taxa de transferência, mas requer mais VRAM.
* **`tiled_gpu`** — Caminho de GPU com eficiência de memória. Processa imagens em blocos para caber na memória limitada da GPU. Menor rendimento, mas funciona em dispositivos com restrições de memória.
* **`cpu_fallback`** — Processamento exclusivamente na CPU usando paralelismo multithread. Utilizado quando não há GPU NVIDIA disponível e como alternativa de último recurso quando ambas as rotas da GPU falham.

A cadeia de alternativas em tempo de execução é sempre `fused_gpu` → `tiled_gpu` → `cpu_fallback`.

***

## Substituição manual da estratégia

Defina a variável de ambiente `CHLOROS_STRATEGY` para forçar uma estratégia específica — uma saída de emergência para especialistas, para quando a detecção automática escolher algo inadequado para a sua situação (por exemplo, manter a GPU livre para outras tarefas):

```bash
# Valid values: CPU_PARALLEL, GPU_SINGLE, GPU_PARALLEL
CHLOROS_STRATEGY=CPU_PARALLEL chloros-cli process ~/datasets/flight001
```

A variável é comparada sem distinção entre maiúsculas e minúsculas; qualquer valor que não seja um dos três nomes é ignorado e a detecção automática prossegue normalmente. Mesmo com a substituição, o Chloros ainda define o número de trabalhadores para você:

| Substituição | Número de trabalhadores usado |
| --- | --- |
| `CPU_PARALLEL` | `max(2, physical cores − 1)` |
| `GPU_SINGLE` | 3 |
| `GPU_PARALLEL` | `min(4, physical cores)` |

Uma substituição por `GPU_SINGLE` ou `GPU_PARALLEL` em uma máquina sem dispositivo CUDA utilizável é recusada com uma linha `[STRATEGY]` no log, e a execução utiliza o `CPU_PARALLEL`.

É preferível definir isso por comando, em vez de permanentemente, para que as execuções normais continuem se adaptando automaticamente.

***

## Comportamento específico da plataforma

| Plataforma | Estratégia | Trabalhadores | Pipeline | Observações |
| --- | --- | --- | --- | --- |
| **Jetson Orin Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequencial) | Modo com eficiência de memória, uma imagem por vez |
| **Jetson Orin NX 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (sequencial) | A RAM compartilhada inferior a 12 GB força o processamento sequencial |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Dispositivo de borda recomendado — limitado a 2 trabalhadores no Jetson |
| **Jetson AGX Orin 32–64 GB** | `GPU_PARALLEL` | 2 | `fused_gpu` (concorrente) | Desempenho máximo na borda (também limitado pelo Jetson a 2 workers) |
| **Desktop com GPU de 8 GB** | `GPU_SINGLE` | 3 | `fused_gpu` / `tiled_gpu` | 3 trabalhadores sobreponham E/S enquanto um semáforo serializa o acesso à GPU |
| **Desktop com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 2–4 | `fused_gpu` (concorrentes) | Desempenho ideal do desktop: 12 GB → 2 trabalhadores, 16 GB → 3, 20 GB+ → 4 |
| **Sistema apenas com CPU** | `CPU_PARALLEL` | núcleos físicos − 1 (mín. 2) | `cpu_fallback` | Não requer GPU, utiliza um pool de threads |

{% hint style="info" %}
**Memória unificada do Jetson**: os dispositivos Jetson compartilham a memória da GPU e da CPU. Um Jetson Orin NX de 16 GB apresenta cerca de 15,3 GB de VRAM, mas essa é a mesma RAM física usada pelo sistema operacional e pelos processos da CPU. É por isso que os Jetsons com 16 GB ou mais se qualificam para o `GPU_PARALLEL` da mesma forma que uma GPU de desktop com 12 GB ou mais, mas estão limitados a 2 trabalhadores — a GPU, os processos de trabalho e seus contextos CUDA por trabalhador utilizam o mesmo pool compartilhado.
{% endhint %}

### Orçamento da GPU por VRAM (GPUs discretas)

Em hosts x86_64 com uma GPU NVIDIA discreta, a VRAM detectada também define quanto do processamento da placa pode ser reivindicado e até que tamanho os lotes podem crescer:

| VRAM detectada | Limite máximo do orçamento da GPU | Multiplicador do tamanho do lote |
| --- | --- | --- |
| **8 GB ou mais** | 90% | ×2,0 |
| **6–8 GB** | 85% | ×1,75 |
| **3,5–6 GB** | 80% | ×1,5 |
| **2–3,5 GB** | 75% | ×1,25 |
| **Menos de 2 GB** | 70% | ×1,0 |

As GPUs discretas reservam apenas 0,5 GB para o sistema, pois não compartilham a memória RAM do sistema. Os perfis do Jetson reservam muito mais e têm um limite inferior — consulte o [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md#per-model-gpu-budget).

***

## Alocação dinâmica de memória da GPU

O Chloros utiliza um [pipeline de processamento de 4 threads](processing-pipeline.md):

* **Thread 1** (Detecção) — Carregamento de imagem, análise de EXIF, detecção de alvo
* **Thread 2** (Calibração) — Cálculo da calibração de refletância
* **Thread 3** (Processamento) — Debayer da GPU, correção de vinheta, cálculo de índice
* **Thread 4** (Exportação) — Gravação de arquivo, incorporação de metadados

Os threads 1, 2 e 4 consomem pouca GPU; A thread 3 é a que mais consome recursos. À medida que as threads anteriores do pipeline são concluídas, sua cota de GPU é **redistribuída para as threads ativas restantes**, de modo que a thread 3 recebe progressivamente mais memória à medida que a execução avança.

### Estágios de alocação

| Estágio | Threads ativas | Distribuição de memória da GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Dividida entre todas as threads, com a maior parte indo para a Thread 3 |
| **Início-meio** | 2, 3, 4 | A parcela da Thread 1 é redistribuída |
| **Fase intermediária-final** | 3, 4 | As parcelas das threads 1 e 2 são direcionadas para as threads 3 e 4 |
| **Final** | 3 ou 4 | A última thread ativa recebe sua alocação máxima |

Duas regras regem esses números:

* Uma thread que seja a **única** ativa recebe a alocação máxima do seu perfil.
* Quando mais de uma tarefa *pesada* da GPU está ativa, a alocação básica de cada tarefa pesada é dividida entre elas (nunca ficando abaixo do mínimo configurado).

O valor efetivamente utilizado em tempo de execução é o **menor** entre a alocação do perfil da plataforma e a recomendação em tempo real do monitor de memória da GPU; portanto, uma placa ocupada sempre prevalece sobre um perfil otimista.***

## Processamento com reconhecimento de textura

O debayer com reconhecimento de textura (**somente Chloros+** — `--debayer texture-aware`) executa um modelo de redução de ruído de IA/ML que requer aproximadamente 1,75 GB de VRAM em FP16 por cópia; portanto, ele utiliza muito mais memória da GPU do que o método padrão:

* Sistemas com **menos de 7 GB de VRAM**processam o Texture Aware em um**loop síncrono, uma imagem por vez** — várias cópias do modelo não cabem, e um pool de workers apenas aumentaria a contenda
* Sistemas com **mais de 7 GB de VRAM** podem processar o Texture Aware de forma simultânea, embora com um número reduzido de workers em comparação com o Standard
* No **Jetson**, o Texture Aware é sempre alocado a um único trabalhador
* Sem **GPU CUDA utilizável**, a execução usa o debayer padrão, e o backend registra uma linha `[DEBAYER] … needs a CUDA GPU …` explicando o motivo
* Em Jetsons de baixo consumo de energia (Nano, Orin Nano), a CLI aplica um limite de frequência da GPU quando você solicita o Texture Aware — consulte o [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md#gpu-frequency-cap-for-texture-aware-on-nano-and-orin-nano)

***

## Gerenciamento térmico (Jetson)

Os dispositivos Jetson têm restrições térmicas, especialmente em implantações em ambientes fechados ou a bordo de aeronaves. O Chloros monitora os sensores de temperatura integrados ao Jetson e dimensiona os tamanhos dos lotes automaticamente:

| Temperatura | Resposta |
| --- | --- |
| **< 70 °C** | Operação normal — velocidade total |
| **70 °C** (Aviso) | O tamanho do lote é reduzido progressivamente (100% → 50% entre 70 °C e 80 °C) |
| **80 °C** (Crítico) | Limitação agressiva (50% → 0% entre 80 °C e 90 °C) |
| **90 °C** (Desligamento) | Interrompa totalmente o processamento da GPU |

Em sistemas desktop com resfriamento adequado, a limitação térmica raramente é acionada.

***

## Gerenciamento da pressão de memória

O Chloros monitora a memória da GPU continuamente durante o processamento e reage em três níveis.

**Dimensionamento do lote.** Um lote começa com 8 imagens multiplicadas pelo fator de multiplicação da plataforma, conforme as tabelas acima. O Chloros, então, verifica a VRAM livre, reserva 20% dela para a sobrecarga do próprio PyTorch e considera aproximadamente 100 MB de memória da GPU por imagem de 12 MP — o tamanho do lote é o menor entre o limite derivado da memória e a base da plataforma. Ele nunca fica abaixo de 1.

**Redução preventiva.**Acima de**85% de utilização da VRAM**, os tamanhos dos lotes são reduzidos antes que ocorra qualquer falha.**Redução da alocação por thread.** À medida que a utilização em tempo real aumenta, o orçamento de GPU de cada thread é reduzido: ×0,75 acima de 80% de utilização, ×0,5 acima de 90%. Os intervalos do monitor são 70% (conservador), 85% (limite normal de operação) e 95% (risco de OOM).**Recuo e recuperação em caso de OOM.** Se, mesmo assim, ocorrer um evento de falta de memória:

* o tamanho do lote é **reduzido pela metade** e, novamente, pela metade a cada OOM consecutivo — cada lote subsequente bem-sucedido reduz essa penalidade em um nível
* as alocações das threads ativas são reduzidas para 70% de seu valor atual e o alocador muda para sua estratégia conservadora, relaxando novamente após uma sequência de alocações bem-sucedidas
* sob pressão severa, o pipeline recua de `fused_gpu` para `tiled_gpu` e, como último recurso, para `cpu_fallback`

**RAM do host (Jetson).** Antes do processamento, a CLI estima o pico de memória do host com base na contagem de imagens e no modo de debayer e emite um aviso se a RAM mais o swap baseado em arquivo forem provavelmente insuficientes, exibindo os comandos exatos para adicionar swap — consulte o [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md#swap-warning-and-recommendations).

***

## Monitoramento da adaptação de computação

### Diagnósticos do sistema

`chloros-cli selftest` é a maneira mais rápida de confirmar o que a camada de computação percebe:

```bash
chloros-cli selftest
```

Suas 7 verificações abrangem versão, disponibilidade de porta, inicialização do backend, `/api/test`, informações do sistema, presença do modelo de redução de ruído e prontidão do CUDA + redutor de ruído. A verificação 5 exibe diretamente a linha de hardware:

```
      GPU: NVIDIA RTX A4000, CUDA: True, PyTorch: 2.7.0
```

A verificação 7 exibe `CUDA: <bool>, Denoiser: <bool>` — ambas devem ser verdadeiras para que o Texture Aware possa ser utilizado.

### Logs do backend

A estratégia e o número de workers são escolhidos no backend no início de cada execução — não há nenhum banner da CLI que os anuncie. Quando algo se comporta de maneira inesperada (um caminho de GPU caindo em fallback, um OOM, um denoiser que não carrega), o log do backend dessa sessão é onde isso aparece:

| Plataforma | Local do log |
| --- | --- |
| **Linux/Jetson** | `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` (um arquivo por execução) |
| **Linux, backend iniciado pela CLI** | também `~/.chloros/backend.log` |
| **Windows** | `%LOCALAPPDATA%\Chloros\logs\` |

### Progresso em tempo real

Durante uma execução, a CLI mostra o progresso em tempo real por thread (Detectando, Analisando, Processando, Exportando) transmitido por meio de Server-Sent Events — a indicação prática de que a Thread 3 é o gargalo. Consulte [Pipeline de processamento](processing-pipeline.md).

***

## Próximos passos

* [Pipeline de processamento](processing-pipeline.md) — Compreendendo a arquitetura do pipeline de 4 threads
* [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implantação e otimização específicas para o Jetson
* [CLI: Linha de comando](../CLI.md) — O guia da CLI

* [Referência da CLI](../reference/cli-reference.md) — Lista completa de comandos para a versão 1.3.0
