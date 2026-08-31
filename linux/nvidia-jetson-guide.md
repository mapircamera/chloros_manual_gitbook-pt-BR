# Guia do NVIDIA Jetson O

Chloros

no NVIDIA Jetson permite o processamento de imagens multiespectrais na borda — em campo, em UAVs e em instalações remotas. OChloros

1.2.0 detecta o modelo do seu Jetson na inicialização e otimiza sua estratégia de processamento para o hardware identificado. **Não é necessário nenhum ajuste manual.**

***

## Modelos Jetson compatíveis

| Modelo                | RAM            | Estratégia de processamento                                     | Uso recomendado                                          |
| -------------------- | -------------- | ------------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB compartilhados | `GPU_PARALLEL` (2 workers)                              | Desempenho máximo, grandes conjuntos de dados                      |
| **Jetson Orin NX**   | 8–16 GB compartilhados  | `GPU_PARALLEL` (2 trabalhadores, 16 GB) / `GPU_SINGLE` (8 GB)   | Recomendação principal para implantação aérea e em campo |
| **Jetson Orin Nano** | 8 GB compartilhados     | `GPU_SINGLE` (1 trabalhador, sequencial)                     | Computação de ponta de nível básico                                 |

{% hint style="info" %}
O pacote arm64 doLinux

requer o **JetPack 6**, que está disponível na família Jetson Orin. Modelos mais antigos (Nano, TX2, Xavier NX) não podem executar o JetPack 6 e não são compatíveis com o pacote atual.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (recomenda-se a versão mais recente)
* **NVIDIA CUDA** (incluída no JetPack)
* **Plano pagoChloros

+** — nível Copper ou superior (necessário para todo o acesso aCLI

/SDK

; aplicado no lado do servidor)

## Instalação

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f

# Verify installation
chloros-cli --version    # prints "Chloros CLI 1.2.0"

# Install Python SDK (optional) — the bundled wheel always matches this build
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl

# Run system diagnostics
chloros-cli selftest
```

Para obter detalhes gerais sobre a instalação doLinux

, localização de arquivos e solução de problemas, consulte [Linux

Instalação](linux-installation.md).

{% hint style="info" %}
**Coloque o diretório de extração em um armazenamento rápido.** Os binários compilados se descompactam automaticamente em um diretório temporário a cada inicialização — o que é extremamente lento quando feito a partir de um cartão SD. OChloros

usa o `/mnt/ssd/tmp` automaticamente quando ele existe; caso contrário, defina o `TMPDIR` como um caminho no seu NVMe (`export TMPDIR=/mnt/nvme/tmp`).
{% endhint %}

***

## Adaptação Dinâmica de Computação no Jetson

### Como Funciona

Na inicialização, oChloros

analisa o perfil do seu sistema:

1. **Detecta o modelo do Jetson** por meio de `/proc/device-tree/model`
2. **Lê a memória compartilhada disponível da GPU/CPU** (o Jetson usa memória unificada)
3. **Seleciona uma estratégia de processamento** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Define automaticamente o número de workers, o tipo de pipeline e a alocação de memória**A decisão é determinada pela**RAM compartilhada total**, e não pelo nome do modelo:

* **Com menos de 12 GB de RAM total**(todos os Jetsons de 8 GB): `GPU_SINGLE` com**1 trabalhador — processamento sequencial deliberado**. A memória é insuficiente para trabalhadores simultâneos, portanto, as imagens são processadas uma de cada vez. Nos Jetsons com**8 GB ou menos**, o Thread 3 ignora completamente o conjunto de workers e executa seu trabalho por imagem dentro do próprio processo.
* **12 GB ou mais**(Orin NX 16 GB, AGX Orin): a memória unificada atende aos requisitos para o `GPU_PARALLEL`, mas o número de workers está**limitado a 2 no Jetson** — a GPU, a RAM dos processos de trabalho e seus contextos CUDA por trabalhador utilizam o mesmo pool compartilhado; portanto, um número maior de trabalhadores aumenta o risco de falhas por falta de memória.

É possível substituir a escolha automática com a variável de ambiente `CHLOROS_STRATEGY` — consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md#manual-strategy-override).

### Comportamento por modelo

| Modelo Jetson                | Estratégia       | Trabalhadores | Execução                                      |
| --------------------------- | -------------- | ------- | ---------------------------------------------- |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | Loop sequencial no próprio processo (`tiled_gpu` sob pressão de memória) |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 1       | Loop sequencial no próprio processo                     |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 2       | Processos de trabalho simultâneos, caminho `fused_gpu`  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 2       | Processos de trabalho simultâneos, caminho `fused_gpu`  |

A principal diferença entre as plataformas é a **memória**. Um Jetson de 8 GB precisa processar imagens uma de cada vez usando uma abordagem em blocos, eficiente em termos de memória, quando a carga é alta, enquanto um Orin de 16 GB ou mais pode processar duas imagens simultaneamente pela GPU usando o pipeline fusionado de maior rendimento.

### Orçamento de GPU por modelo

Cada modelo Jetson também possui um perfil de hardware que limita a quantidade de memória que o processamento do pool compartilhado pode ocupar e dimensiona os tamanhos dos lotes:

| Modelo | Limite máximo do orçamento de GPU | Multiplicador do tamanho do lote | Reservado para sistema/tela |
| --- | --- | --- | --- |
| **Jetson Orin Nano** | 70% | ×0,8 | 2,0 GB |
| **Jetson Orin NX** | 75% | ×1,0 | 3,0 GB |
| **Jetson AGX Orin** | 80% | ×1,5 | 4,0 GB |

A RAM detectada ajusta o perfil: um Jetson que informe **16 GB ou mais** tem seu multiplicador de lote aumentado para ×1,2. O tamanho básico do lote antes dos multiplicadores é de 8 imagens.

Para a referência completa sobre adaptação de computação, consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md).

***

## Limite de frequência da GPU para o Texture Aware no Nano e no Orin Nano

O processador de remoção de padrão de pixel do Texture Aware executa inferência de rede neural na GPU, o que pode acionar **alertas de sobrecorrente**em modelos Jetson de baixo consumo de energia (classe de 10 a 15 W) quando a GPU opera em velocidade máxima. Antes do processamento do Texture Aware em um**Jetson Nano ou Orin Nano**, o `Chloros

` verifica a frequência máxima da GPU e a limita a **510 MHz** (510000000) caso ela esteja atualmente mais alta:

* Se o `CLI

` puder gravar no nó sysfs da frequência da GPU, o limite é **aplicado automaticamente** e uma confirmação é exibida.
* Caso contrário (requer acesso root), oCLI

exibe o comando exato `sudo` para aplicar o limite manualmente, aguarda um momento para que você possa lê-lo e, em seguida, continua — o processamento ainda é executado, mas pode exibir avisos de sobrecorrente.

Para aplicar o limite você mesmo antes do processamento:

```bash
echo 510000000 | sudo tee /sys/devices/platform/bus@0/17000000.gpu/devfreq/17000000.gpu/max_freq
```

Modelos de maior potência (Orin NX 25W, AGX Orin 60W) operam na velocidade máxima da GPU; nenhum limite é aplicado. O debayer padrão nunca aciona o limite em nenhum modelo.

{% hint style="info" %}
**O Texture Aware no Jetson sempre processa uma imagem por vez.** Cada worker precisaria de seu próprio contexto CUDA (~1 GB), além de sua própria cópia do modelo de redução de ruído, o que a memória unificada não suporta — portanto, no Jetson, a rota do Texture Aware é fixada a um único trabalhador com acesso à GPU serializado. É de se esperar que o Texture Aware seja significativamente mais lento que o Standard em qualquer Jetson.
{% endhint %}

***

## Gerenciamento Térmico

Os dispositivos Jetson têm margem térmica limitada, especialmente em implantações em ambientes fechados ou a bordo de aeronaves. OChloros

monitora a temperatura do SoC e limita automaticamente o tamanho dos lotes:

| Temperatura         | Ação                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Operação normal — velocidade total de processamento          |
| **70 °C** (Aviso)  | O tamanho do lote é reduzido progressivamente (100% → 50% entre 70 °C e 80 °C) |
| **80 °C** (Crítico) | Limitação agressiva (50% → 0% entre 80 °C e 90 °C) |
| **90 °C** (Desligamento) | Interrupção total do processamento da GPU — é necessário resfriamento |

{% hint style="warning" %}
**Garanta ventilação e dissipação de calor adequadas** para processamento contínuo, especialmente em gabinetes de campo fechados ou sistemas aéreos. A limitação térmica reduzirá a taxa de processamento para proteger o hardware.
{% endhint %}

***

## Gerenciamento de memória

Os dispositivos Jetson utilizam **memória unificada** — a GPU e a CPU compartilham a mesma RAM física. A VRAM informada (por exemplo, ~15,3 GB em um Orin NX de 16 GB) não é memória dedicada à GPU; trata-se da mesma RAM que o sistema operacional e todos os outros processos estão utilizando.

### Aviso e recomendações sobre swap

Antes do processamento no Jetson, o `CLI

` conta as imagens RAW na sua pasta de entrada (`.tif`, `.tiff`, `.raw`, `.dng` — as visualizações em JPG não são contadas), estima o pico de memória necessário para a execução e **avisa antes de iniciar** se for provável que a RAM + swap sejam insuficientes. O aviso tem como título `LOW MEMORY WARNING - Jetson Detected`, exibe a contagem de imagens, a RAM, o swap atual e o pico estimado e, em seguida, fornece os comandos exatos `fallocate` / `chmod` / `mkswap` / `swapon` dimensionados para o seu projeto (nunca menores que 8 GB). Ele faz uma pausa de alguns segundos para que a mensagem não se perca na rolagem do histórico, e então o processamento continua.**Estimativas de memória utilizadas pelo aviso:**

| Modo de debayer | Base | Por imagem |
| --- | --- | --- |
| Padrão | ~1,5 GB | ~10 MB |
| Sensível à textura | ~2,5 GB (tempo de execução do modelo + “Python

”) | ~15 MB |

O aviso é acionado quando o pico estimado excede a RAM + swap menos uma margem de segurança de 1 GB, e ele conta apenas o swap **baseado em arquivo** — uma configuração apenas com zram ainda será sinalizada.

Para adicionar swap manualmente (exemplo: 8 GB):

```bash
# Check current memory and swap
free -h

# Create a swap file
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```



<!-- SCREENSHOT-NEEDED: Terminal on a Jetson Orin (SSH session) showing the full "LOW MEMORY WARNING - Jetson Detected" block printed by `chloros-cli process` on a large folder: the image count and debayer mode line, RAM / current swap / estimated peak figures, and the fallocate/chmod/mkswap/swapon command block it recommends -->

### Tratamento de OOM (Out of Memory)

Durante o processamento, oChloros

monitora a memória da GPU e reduz a carga de trabalho de forma suave, em vez de travar:

1. Quando a utilização da memória da GPU excede **85%**, os tamanhos dos lotes são reduzidos de forma preventiva
2. Se ainda assim ocorrer um evento de falta de memória, o tamanho do lote é **reduzido pela metade** e, novamente, pela metade a cada OOM consecutivo; cada lote subsequente bem-sucedido reverte essa penalidade em um passo
3. Sob pressão sustentada, o pipeline recua do `fused_gpu` para o caminho `tiled_gpu`, que é mais eficiente em termos de memória, e para o processamento pela CPU como último recurso

***

## Implantação em campo

### Considerações sobre energia

| Modelo Jetson     | Consumo típico de energia | Observações                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Orin Nano | 7–15 W              | Conector cilíndrico CC          |
| Jetson Orin NX   | 10–25 W             | Conector cilíndrico CC          |
| Jetson AGX Orin  | 15–60 W             | USB-C PD ou conector cilíndrico |

Planeje seu orçamento de energia para processamento contínuo — o consumo máximo de energia ocorre durante o Thread 3 (Processamento), que exige uso intensivo da GPU.

### Recomendações de armazenamento

* **SSD NVMe** fortemente recomendado para implantações em arm64
* Cartões SD são muito lentos para processamento — use-os apenas como mídia de inicialização
* Planeje 2 a 3 vezes o tamanho dos dados brutos da imagem para a saída processada

### Operação sem interface gráfica viaSSH



Chloros

CLI

é ideal para implantações do Jetson sem interface gráfica:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format "TIFF (32-bit, Percent)"

# Monitor export progress
chloros-cli export-status
```

### Backend sempre ativo para sincronização de tempo do LATTICE / DAQ-E

Se o seu Jetson controla câmeras LATTICE ou sensores de luz DAQ-E sem monitor, habilite o serviço systemd de backend para que o grandmaster PTP funcione continuamente (a unidade está instalada, mas não habilitada por padrão):

```bash
sudo systemctl enable --now chloros-backend.service
chloros-cli time-sync status
```

Consulte [Instalação doLinux

](linux-installation.md#always-on-ptp-for-headless-hosts) para obter detalhes, incluindo como o pacote torna as portas PTP 319/320 vinculáveis sem privilégios de root.

### Processamento automatizado com o systemd

Crie um serviço systemd para processamento automatizado:

```ini
# /etc/systemd/system/chloros-process.service
[Unit]
Description=Chloros Automated Processing
After=network.target

[Service]
Type=oneshot
User=chloros
ExecStart=/usr/bin/chloros-cli process /data/incoming --output /data/processed
StandardOutput=append:/var/log/chloros-process.log
StandardError=append:/var/log/chloros-process.log

[Install]
WantedBy=multi-user.target
```

O `chloros-cli process` retorna um valor diferente de zero quando uma execução que solicitou produtos não grava nenhuma imagem; portanto, o status de falha do systemd é relevante para o monitoramento.

Combine com um temporizador do systemd para processamento programado:

```ini
# /etc/systemd/system/chloros-process.timer
[Unit]
Description=Run Chloros Processing Every Hour

[Timer]
OnCalendar=hourly
Persistent=true

[Install]
WantedBy=timers.target
```

```bash
sudo systemctl enable chloros-process.timer
sudo systemctl start chloros-process.timer
```

***

## Exemplos de fluxos de trabalho

### Processamento básico no Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI
```

###Python

SDK

no Jetson

```python
from chloros_sdk import ChlorosLocal

with ChlorosLocal() as chloros:
    chloros.create_project("field_survey_042")
    chloros.import_images("/data/flights/flight_042")
    chloros.configure(
        indices=["NDVI", "NDRE", "GNDVI"],
        export_format="TIFF (32-bit, Percent)",
        reflectance_calibration=True
    )
    chloros.process(mode="parallel")

print("Processing complete!")
```

### Processamento em lote de vários voos

```bash
#!/bin/bash
# Process all flight datasets in a directory
for flight in /data/flights/*/; do
    name=$(basename "$flight")
    echo "Processing $name..."
    chloros-cli process "$flight" \
        --output "/data/processed/$name" \
        --format "TIFF (32-bit, Percent)" \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para uso em campo

Para implantações em campo e aéreas, considere estas opções de placas portadoras Jetson Orin NX de 16 GB:

* **Aéreo/drone**: Sistemas com classificação de resistência à vibração (MIL-STD), leves (menos de 300 g) e com resfriamento passivo
* **Campo robusto**: Caixas à prova d’água IP67/IP69K com conectividade para câmera GigE via PoE
* **Mínimo/orçamento reduzido**: kits de desenvolvedor com caixas adicionais

Entre em contato com o [Suporte daMAPIR

](https://www.mapir.camera/community/contact) para obter recomendações específicas de hardware para o seu cenário de implantação.

***

## Próximos passos

* [Instalação doLinux

](linux-installation.md) — Detalhes gerais sobre a instalação doLinux


* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Referência completa sobre estratégias de computação
* [Pipeline de processamento](../processing-architecture/processing-pipeline.md) — Noções básicas sobre o pipeline de 4 threads
* [CLI

: Linha de comando](../CLI.md) — O guia doCLI


* [API

:Python

SDK

](../api-python-sdk.md) — O guiaSDK


* [Referência doCLI

](../reference/cli-reference.md) e [Referência doSDK

](../reference/sdk-reference.md) — Listas completas de comandos/API

s para a versão 1.2.0
