# Pipeline de processamento O

Chloros 1.3.0 utiliza um pipeline de processamento com 4 threads que funciona como uma linha de montagem em etapas. Cada thread lida com uma fase distinta do fluxo de trabalho, de modo que várias imagens podem estar em processamento em diferentes etapas ao mesmo tempo.

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

***

## Arquitetura do pipeline

```

Images In → [Thread 1: Detection] → [Thread 2: Calibration] → [Thread 3: Processing] → [Thread 4: Export] → Files Out
```

Cada imagem passa por todas as quatro threads em ordem. Com o processamento multithread do Chloros+, várias imagens ocupam diferentes threads simultaneamente — enquanto a Thread 3 processa uma imagem, a Thread 1 pode estar detectando a próxima, a Thread 2 calibrando outra e a Thread 4 gravando uma imagem finalizada no disco.

O progresso é relatado por thread e transmitido por meio de Server-Sent Events (o backend os publica no `/api/events`). Na exibição de progresso em tempo real da CLI, os quatro estágios são identificados como **Detecção, Análise, Processamento, Exportação**.***

## Detalhes das threads

### Thread 1: Detecção

**Objetivo**: Carregar imagens e detectar alvos de calibração.

* Lê arquivos de imagem do disco — pares Survey3 `.raw`+`.jpg`, capturas LATTICE `.tif`/`.tiff` e `.dng`
* Extrai metadados EXIF (GPS, modelo da câmera, registros de data e hora, exposição)
* Detecta alvos de calibração: geometrias de alvos marcados com ArUco para capturas do LATTICE e o detector de painel clássico para fotos de alvos de calibração do Survey3
* Resultados: dados de imagem + metadados + resultados da detecção de alvos

É principalmente uma thread limitada por E/S e CPU.

### Thread 2: Calibração

**Objetivo**: Calcular parâmetros de calibração a partir dos alvos detectados.

* Calcula coeficientes de calibração de refletância a partir das imagens dos alvos
* Calcula parâmetros de correção de vinheta
* Determina curvas de calibração por banda
* Saídas: parâmetros de calibração para cada imagem

Um thread de computação limitado pela CPU. Em uma execução do Survey3 ou mista, o Thread 3 aguarda sua conclusão quando a calibração de refletância está ativada, de modo que seus coeficientes estejam prontos antes que qualquer imagem seja processada. Em uma execução apenas do LATTICE, o Thread 3 é executado durante a detecção: cada quadro é iniciado assim que sua âncora de alvo estiver finalizada. Um quadro considera o alvo mais próximo no tempo, em qualquer direção, portanto, ele aguarda até que todos os quadros de sua câmera mais próximos no tempo do que seu alvo mais próximo atual tenham sido digitalizados. Os produtos são os mesmos, apenas mais rápidos.

### Thread 3: Processamento (GPU)

**Objetivo**: Aplicar correções e calcular índices de vegetação.**Esta é a thread que exige maior capacidade de computação.*** **Debayering**: converte dados RAW Bayer em imagens multicanais
  * Padrão (Rápido, Qualidade Média) — padrão, `--debayer standard`
  * Sensível à Textura (Lento, Qualidade Máxima) — somente Chloros+, `--debayer texture-aware`, utiliza um modelo de redução de ruído baseado em IA/ML
  * As capturas LATTICE mono (M3M) são de banda única: as etapas de demosaico e equilíbrio de branco são ignoradas para elas (com uma mensagem de log de uma linha), enquanto quaisquer imagens M3C/Bayer na mesma execução ainda passam por essas etapas
* **Correção de vinheta**: aplica correção de vinheta da lente em toda a imagem
* **Calibração de refletância**: aplica coeficientes de calibração para converter em valores de refletância
* **Cálculo de índices**: calcula índices de vegetação (NDVI, NDRE, GNDVI, …)
* Resultados: dados de imagem processados, prontos para exportação

Esta thread é a que mais se beneficia da aceleração por GPU, e é a thread que a [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) ajusta.

### Thread 4: Exportação

**Objetivo**: Gravar imagens processadas no disco.

* Grava arquivos de saída no formato selecionado — `TIFF (16-bit)`, `TIFF (8-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)`
* Incorpora metadados nos arquivos de saída (GPS, registros de data e hora, parâmetros de processamento) — incluindo as tags de processamento `Chloros:*`, que, a partir da versão 1.3.0, são gravadas em todos os modos de processamento, tanto para o Survey3 quanto para o LATTICE
* Organiza a saída na pasta do projeto como `<camera>/<format>/<Product>_Images/` — por exemplo, `LATT-M3M-L41-F550/tiff16/Reflectance_Calibrated_Images/`. **Os arquivos exportados mantêm o nome do arquivo de origem; a pasta identifica o produto.**
* Para capturas do LATTICE, um quadro de origem pode ser dividido em vários produtos (Debayered, Preview, Radiance, Reflectance, Index), cada um em sua própria pasta de produto. Os produtos Index são calculados a partir do produto de refletância gerado pela execução (quando não houver nenhum, a partir de outro produto gerado pela execução, como o de radianância). Os produtos de radianância e refletância de câmeras multiespectrais M3C também podem receber uma máscara de limitação por pixel em `<format>/clamp_masks/` (consulte [Formatos de imagem de saída](../output-image-formats.md#clamp-masks))
* Saídas: arquivos finais no disco

Principalmente uma thread limitada pela E/S — o armazenamento em SSD melhora seu desempenho de forma perceptível.

***

## Por trás dos bastidores: Executores

Dentro da Thread 3, o trabalho por imagem é paralelizado da seguinte forma:

* **Estratégias de GPU**(`GPU_SINGLE`, `GPU_PARALLEL`) utilizam um conjunto de**processos de trabalho**, cada um sendo um processo separado do sistema operacional com seu próprio contexto CUDA. O Chloros inicia cada trabalhador executando novamente seu próprio programa de backend instalado — e não por meio do `multiprocessing` do Python, cuja inicialização do trabalhador não funciona dentro do aplicativo empacotado
* **`CPU_PARALLEL`** utiliza um conjunto de threads (o `ThreadPoolExecutor` do Python) — o NumPy e o OpenCV liberam o GIL, portanto, as threads são suficientes
* Dispositivos Jetson com 8 GB ou menos de RAM compartilhada ignoram completamente o executor e processam internamente, de forma sequencial
* O Texture Aware em uma GPU com menos de 7 GB de VRAM também é executado sequencialmente — o modelo de redução de ruído não cabe mais de uma vez O

Chloros não utiliza nenhuma estrutura distribuída de terceiros (como o Ray). Consulte [Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) para saber como a estratégia e o número de trabalhadores são escolhidos.

***

## Processamento sequencial vs. em pipeline

### Modo Livre (Sequencial)

Na versão gratuita do Chloros, as imagens são processadas **uma de cada vez**, sequencialmente, passando por todas as quatro etapas:

```

Image 1: [Detect] → [Calibrate] → [Process] → [Export]
                                                         Image 2: [Detect] → [Calibrate] → [Process] → [Export]
```

A interface gráfica exibe uma barra de progresso simplificada no modo gratuito; suas fases seriais são indicadas como **Detecção de Alvo**e, em seguida,**Processamento**.

### Modo Chloros+ (em pipeline)

Com uma licença do Chloros+, todas as quatro threads operam **simultaneamente** em imagens diferentes:

```

Thread 1: [Image 1] [Image 2] [Image 3] [Image 4] ...
Thread 2:           [Image 1] [Image 2] [Image 3] ...
Thread 3:                     [Image 1] [Image 2] ...
Thread 4:                               [Image 1] ...
```

A barra de progresso da GUI mostra as quatro etapas; passe o mouse sobre ela para ver o progresso por thread. Na CLI, as mesmas quatro etapas são exibidas em tempo real como **Detectando, Analisando, Processando, Exportando**.

{% hint style="info" %}
**Um rótulo, dois nomes.** A CLI denomina a etapa 3 de _Processamento_. O feed de progresso do modo premium do backend — aquele que a barra de progresso da GUI exibe — rotula a mesma etapa como _Calibração_. Trata-se da mesma thread realizando o mesmo trabalho (Thread 3: debayer, correções, índices).
{% endhint %}

{% hint style="success" %}
**O processamento em pipeline com o Chloros+** pode ser de 3 a 5 vezes mais rápido do que o processamento sequencial, dependendo do seu hardware e do tamanho do conjunto de dados. O aumento de velocidade é maior em sistemas com GPUs e SSDs rápidos.
{% endhint %}

***

## Progresso da exportação da thread 4

A thread de exportação possui seu próprio acompanhamento de progresso, que você pode consultar separadamente:**Interface de CLI**:

```bash
chloros-cli export-status
```

**SDK**:

```python
status = chloros.get_status()
print(f"Export: {status['export']['percent']}% - Phase: {status['export']['phase']}")
```

O processamento está concluído quando a Thread 4 atinge 100%.

{% hint style="info" %}
**Uma execução que não grava imagens é considerada uma falha.**Em caso de sucesso, `chloros-cli process` informa quantos produtos de imagem foram gravados (`Image products written: N`). Se produtos foram solicitados e**nenhum**foi gravado — apenas `project.json` e `calibration_data.json` —, a CLI exibe `Processing finished but wrote no image products.` e**encerra com valor diferente de zero**, indicando o nome da pasta do projeto e as causas habituais (a pasta de entrada não foi reconhecida como uma captura — verifique o layout e `--input-level` — ou todos os produtos solicitados eram incompatíveis com essas câmeras). Os scripts podem se basear no código de saída.
{% endhint %}

***

## Relação com a Adaptação Dinâmica de Computação

[Adaptação Dinâmica de Computação](dynamic-compute-adaptation.md) afeta principalmente a **Thread 3 (Processamento)**:

* **`GPU_PARALLEL`**: A Thread 3 processa várias imagens simultaneamente na GPU usando o pipeline `fused_gpu`
* **`GPU_SINGLE`**: A Thread 3 serializa o acesso à GPU com um semáforo enquanto os processos de trabalho realizam E/S sobrepostas, utilizando o pipeline `fused_gpu` ou o pipeline `tiled_gpu`, que é eficiente em termos de memória
* **`CPU_PARALLEL`**: A thread 3 utiliza processamento baseado na CPU com paralelismo multithread

A alocação de memória da GPU da thread 3 também aumenta à medida que as threads 1 e 2 são concluídas — consulte [Alocação dinâmica de memória da GPU](dynamic-compute-adaptation.md#dynamic-gpu-memory-allocation).

***

## Próximos passos

* [Adaptação dinâmica de computação](dynamic-compute-adaptation.md) — Como o Chloros seleciona a estratégia ideal para o seu hardware
* [Guia do NVIDIA Jetson](../linux/nvidia-jetson-guide.md) — Comportamento do pipeline específico da plataforma no Jetson
* [Monitoramento do processamento](../processing-images-gui/monitoring-the-processing.md) — Monitoramento do andamento pela interface gráfica
* [Referência da CLI](../reference/cli-reference.md) — `process`, `export-status`, códigos de saída e layout de saída
