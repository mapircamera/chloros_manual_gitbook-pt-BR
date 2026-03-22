# Adaptação dinâmica de computação

O Chloros 1.1.0 apresenta detecção inteligente de hardware e seleção automática de estratégias de processamento. O mecanismo de processamento se adapta ao seu hardware — desde um Jetson Nano até uma estação de trabalho com várias GPUs — sem qualquer configuração manual.

***

## Como funciona

Quando o Chloros é iniciado, ele cria automaticamente um perfil do seu sistema:

1. **Detecta o sistema operacional** — Windows ou Linux
2. **Identifica os núcleos da CPU e a RAM total**

3.**Detecta a presença de GPU** — capacidade NVIDIA CUDA, VRAM, modelo
4. **Identifica o modelo Jetson** (se aplicável) — via `/proc/device-tree/model`
5. **Verifica os sensores térmicos** (Jetson) — para processamento sensível à temperatura
6. **Seleciona a estratégia de computação ideal** — com base em todo o hardware detectado
7. **Configura automaticamente o número de workers, o tipo de pipeline e a alocação de memória**O resultado é armazenado em cache para que as execuções subsequentes iniciem mais rapidamente. Se houver alterações no hardware (por exemplo, se uma GPU for adicionada), o Chloros gera um novo perfil na próxima inicialização.***

## Estratégias de computação

O Chloros seleciona uma das três estratégias de computação com base no seu hardware:

| Estratégia | GPU necessária | Trabalhadores | Pipeline | Ideal para |
| --- | --- | --- | --- | --- |
| **`GPU_PARALLEL`** | Sim (12 GB+ de VRAM ou 16 GB+ compartilhada) | 3-4 | `fused_gpu` | GPUs de desktop com 12 GB+, Jetson Orin NX 16 GB, AGX Orin |
| **`GPU_SINGLE`** | Sim (&lt; 12 GB de VRAM) | 1-3 | `tiled_gpu` | GPUs básicas, Jetson Nano, Orin Nano |
| **`CPU_PARALLEL`** | Não | núcleos - 1 | `cpu_fallback` | Sistemas sem GPU NVIDIA |

### Tipos de pipeline

* **`fused_gpu`** — Caminho de processamento completo da GPU. Todas as operações de debayer, correção e indexação são executadas na GPU em uma única passagem fusionada. Maior taxa de transferência, mas requer mais VRAM.
* **`tiled_gpu`** — Caminho de GPU com eficiência de memória. Processa imagens em blocos para caber na memória limitada da GPU. Menor taxa de transferência, mas funciona em dispositivos com restrições de memória.
* **`cpu_fallback`** — Processamento apenas na CPU usando paralelismo multithread. Usado quando nenhuma GPU NVIDIA está disponível.***

## Comportamento específico da plataforma

| Plataforma | Estratégia | Trabalhadores | Pipeline | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` (serializado) | Modo com eficiência de memória, processa uma imagem por vez |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` (concorrente) | Dispositivo de borda recomendado — processamento paralelo real da GPU |
| **Jetson AGX Orin 64 GB** | `GPU_PARALLEL` | 4 | `fused_gpu` (concorrente) | Desempenho máximo de borda |
| **Desktop com GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bom desempenho em desktop com blocos eficientes em termos de memória |
| **Desktop com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Desempenho ideal em desktop |
| **Sistema apenas com CPU** | `CPU_PARALLEL` | núcleos - 1 | `cpu_fallback` | Não requer GPU, usa ThreadPool |

{% hint style="info" %}
**Memória unificada do Jetson**: Os dispositivos Jetson compartilham a memória da GPU e da CPU. Um Jetson Orin NX de 16 GB relata ~15,3 GB de VRAM, mas essa é a mesma RAM física usada pelo sistema operacional e pelos processos da CPU. O Chloros leva isso em consideração ao definir limites de alocação de memória.
{% endhint %}

***

## Alocação dinâmica de memória da GPU

O Chloros utiliza um [pipeline de processamento de 4 threads](processing-pipeline.md):

* **Thread 1** (Detecção) — Carregamento de imagem, análise de EXIF, detecção de alvo
* **Thread 2** (Calibração) — Cálculo da calibração de refletância
* **Thread 3** (Processamento) — Debayer da GPU, correção de vinheta, cálculo de índice
* **Thread 4** (Exportação) — Gravação de arquivos, incorporação de metadados

À medida que os threads anteriores do pipeline concluem seu trabalho (por exemplo, todas as imagens foram detectadas), sua alocação de memória da GPU é liberada e **redistribuída para os threads ativos restantes**. Isso significa que a Thread 3 (a etapa que exige maior uso da GPU) recebe progressivamente mais memória à medida que o pipeline avança, melhorando a taxa de transferência para o trabalho que exige maior capacidade de computação.

### Etapas de alocação

| Etapa | Threads ativas | Distribuição de memória da GPU |
| --- | --- | --- |
| **Inicial** | 1, 2, 3, 4 | Dividida entre todas as threads |
| **Início-meio** | 2, 3, 4 | Memória da thread 1 redistribuída |
| **Meio-fim** | 3, 4 | Memória das threads 1+2 vai para 3+4 |
| **Fim** | 3 ou 4 | Memória máxima para a thread restante |***

## Processamento com reconhecimento de textura

O método de debayer com reconhecimento de textura (somente Chloros+) utiliza significativamente mais memória da GPU do que o método padrão devido ao modelo de redução de ruído de IA/ML:

* Sistemas com **&lt; 7 GB de VRAM** são forçados a um ciclo de processamento síncrono no modo com reconhecimento de textura (uma imagem por vez)
* Sistemas com **7 GB ou mais de VRAM** podem processar o modo Texture Aware simultaneamente, embora com um número reduzido de workers em comparação com o modo Standard***

## Gerenciamento térmico (Jetson)

Os dispositivos Jetson têm restrições térmicas, especialmente em implantações em ambientes fechados ou aéreos. O Chloros monitora as temperaturas da GPU e da CPU e ajusta automaticamente o processamento:

| Temperatura | Resposta |
| --- | --- |
| **&lt; 70 °C** | Operação normal — velocidade total |
| **70 °C** (Aviso) | Reduzir tamanho do lote |
| **80 °C** (Crítico) | Limitação agressiva — diminuir a simultaneidade e o número de trabalhadores |
| **90 °C** (Desligamento) | Interromper totalmente o processamento da GPU |

O monitoramento de temperatura utiliza o `tegrastats` em plataformas Jetson. Em sistemas desktop com refrigeração adequada, a limitação térmica raramente é acionada.

***

## Gerenciamento da pressão de memória

O Chloros monitora a pressão de memória do sistema durante o processamento:

* **Limite de memória**: 85% de utilização aciona um comportamento conservador
* **Redução por falta de memória**: Se ocorrer um evento de falta de memória, a alocação é reduzida em 25% (multiplicador de 0,75x)
* **Recurso de fallback do pipeline**: Sob pressão de memória severa, o pipeline recua automaticamente de `fused_gpu` para `tiled_gpu`
* **Recomendações de swap**: No Jetson, o Chloros avisa se o espaço de swap for insuficiente para o tamanho do seu conjunto de dados***

## Monitoramento da adaptação de computação

### Saída de status do CLI

Quando o processamento é iniciado, o CLI exibe o perfil de hardware detectado:

```

Chloros CLI 1.1.0
Platform: Linux aarch64 (Jetson Orin NX 16GB)
Strategy: GPU_PARALLEL | Workers: 3 | Pipeline: fused_gpu
CUDA: Available | GPU Memory: 15.3 GB (shared)
```

### Diagnósticos do sistema

Execute o `chloros-cli selftest` para ver um perfil completo de hardware e verificar os recursos de computação:

```bash
chloros-cli selftest
```

Isso verifica a disponibilidade do CUDA, a memória da GPU, os modelos de redução de ruído e a conectividade do backend.

***

## Próximos passos

* [Pipeline de processamento](processing-pipeline.md) — Compreendendo a arquitetura do pipeline de 4 threads
* [Guia NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Implantação e otimização específicas para Jetson
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
