# Guia do NVIDIA Jetson

XPROTX000047 O XPROTX no NVIDIA Jetson permite o processamento de imagens multiespectrais na borda — em campo, em UAVs e em instalações remotas. XPROTX000048 O XPROTX detecta automaticamente o seu modelo Jetson e otimiza sua estratégia de processamento para o seu hardware.

***

## Modelos Jetson compatíveis

| Modelo                | RAM            | Estratégia de processamento                                   | Uso recomendado                                          |
| -------------------- | -------------- | ----------------------------------------------------- | -------------------------------------------------------- |
| **Jetson AGX Orin**  | 32-64 GB compartilhados | `GPU_PARALLEL` (4 workers)                            | Desempenho máximo, grandes conjuntos de dados                      |
| **Jetson Orin NX**   | 8-16 GB compartilhada  | `GPU_PARALLEL` (3 workers, 16 GB) / `GPU_SINGLE` (8 GB) | Recomendação principal para implantação aérea e em campo |
| **Jetson Orin Nano** | 8 GB compartilhados     | `GPU_SINGLE` (1 trabalhador)                               | Computação de ponta de nível básico                                 |
| **Jetson Nano**      | 4-8 GB compartilhados   | `GPU_SINGLE` (1 trabalhador)                               | Nível básico, com restrições de memória                          |

{% hint style="info" %}
**Modelos Jetson antigos** (TX2, TX1, Xavier NX) podem não ser compatíveis. O desempenho varia de acordo com a memória de GPU disponível e os recursos CUDA.
{% endhint %}

***

## Requisitos

* **JetPack 6.x** (recomenda-se a versão mais recente)
* **NVIDIA CUDA** (incluído no JetPack)
* **Licença Chloros+** (necessária para acesso a CLI/SDK)

## Instalação

```bash
# Install the JetPack 6 .deb package
sudo dpkg -i chloros-arm64-jp6.deb

# Verify installation
chloros-cli --version

# Install Python SDK (optional)
pip install chloros-sdk

# Run system diagnostics
chloros-cli selftest
```

Para obter detalhes gerais sobre a instalação do Linux, consulte [Instalação do Linux](linux-installation.md).

***

## Adaptação dinâmica de computação no Jetson

O Chloros detecta automaticamente seu modelo Jetson e seleciona a estratégia de processamento ideal. **Não é necessário nenhum ajuste manual.**

### Como funciona

Na inicialização, o Chloros cria um perfil do seu sistema:

1. **Detecta o modelo Jetson** via `/proc/device-tree/model`
2. **Lê a GPU disponível/memória compartilhada**

3.**Seleciona uma estratégia de processamento** (`GPU_PARALLEL`, `GPU_SINGLE` ou `CPU_PARALLEL`)
4. **Define automaticamente o número de workers, o tipo de pipeline e a alocação de memória**

### Comportamento por modelo

| Modelo Jetson                | Estratégia       | Trabalhadores | Pipeline                       | Concorrência |
| --------------------------- | -------------- | ------- | ------------------------------ | ----------- |
| **Jetson Nano 8 GB**         | `GPU_SINGLE`   | 1       | `tiled_gpu` (eficiência de memória) | Serializado  |
| **Jetson Orin Nano 8 GB**    | `GPU_SINGLE`   | 1       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 8 GB**      | `GPU_SINGLE`   | 2       | `tiled_gpu`                    | Serializado  |
| **Jetson Orin NX 16 GB**     | `GPU_PARALLEL` | 3       | `fused_gpu` (caminho completo da GPU)    | Concorrente  |
| **Jetson AGX Orin 32-64 GB** | `GPU_PARALLEL` | 4       | `fused_gpu`                    | Concorrente  |

{% hint style="success" %}
**Jetson Orin NX 16 GB** é o ponto ideal para implantação de ponta — ele recebe a estratégia `GPU_PARALLEL` com 3 trabalhadores simultâneos, oferecendo processamento paralelo real da GPU em um formato compacto.
{% endhint %}

A principal diferença entre as plataformas é a **memória**. Um Jetson Nano com 8 GB de memória compartilhada precisa processar imagens uma de cada vez usando uma abordagem em blocos eficiente em termos de memória, enquanto um Orin NX com 16 GB pode processar 3 imagens pela GPU simultaneamente usando o pipeline fusionado de maior throughput.

Para a referência completa sobre adaptação de computação, consulte [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md).

***

## Gerenciamento térmico

Os dispositivos Jetson têm margem térmica limitada, especialmente em implantações em ambientes fechados ou aéreos. O Chloros inclui monitoramento térmico automático e limitação de desempenho:

| Temperatura         | Ação                                            |
| ------------------- | ------------------------------------------------- |
| **&lt; 70 °C**          | Operação normal — velocidade total de processamento          |
| **70 °C** (Aviso)  | Reduzir o tamanho do lote automaticamente                   |
| **80 °C** (Crítico) | Limitação agressiva — menor simultaneidade         |
| **90 °C** (Desligamento) | Interromper totalmente o processamento da GPU — resfriamento necessário |

{% hint style="warning" %}
**Garanta ventilação e dissipação de calor adequadas** para processamento contínuo, especialmente em gabinetes de campo fechados ou sistemas aéreos. A limitação térmica reduzirá a taxa de processamento para proteger o hardware.
{% endhint %}

***

## Gerenciamento de memória

Os dispositivos Jetson utilizam **memória unificada** — a GPU e a CPU compartilham a mesma RAM física. Isso significa que a VRAM informada (por exemplo, 15,3 GB no Orin NX de 16 GB) não é memória dedicada à GPU; ela é compartilhada com o sistema operacional e outros processos.

### Recomendações de swap

Para conjuntos de dados grandes ou processamento de debayer com reconhecimento de textura, o Chloros pode recomendar a criação de espaço de swap:

```bash
# Check current memory and swap
free -h

# Create a swap file (example: 8GB)
sudo fallocate -l 8G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make persistent across reboots
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

**Estimativas de memória por imagem:**

* Debayer padrão: ~10 MB por imagem
* Debayer com reconhecimento de textura: ~15 MB por imagem

O Chloros calcula automaticamente a memória necessária com base no tamanho do seu conjunto de dados e avisa se o swap for recomendado.

### Fallback OOM (Out of Memory)

Se uma condição de falta de memória for detectada durante o processamento:

1. O Chloros reduz automaticamente a contagem de workers da GPU
2. Recorre do pipeline `fused_gpu` para o `tiled_gpu` (mais eficiente em termos de memória)
3. Continua o processamento com taxa de transferência reduzida, em vez de travar

***

## Implantação em campo

### Considerações sobre energia

| Modelo Jetson     | Consumo típico de energia | Observações                   |
| ---------------- | ------------------ | ----------------------- |
| Jetson Nano      | 5-10 W              | USB-C ou conector cilíndrico    |
| Jetson Orin Nano | 7-15 W              | Conector cilíndrico CC          |
| Jetson Orin NX   | 10-25 W             | Conector cilíndrico CC          |
| Jetson AGX Orin  | 15-60 W             | USB-C PD ou conector cilíndrico |

Planeje seu orçamento de energia para processamento contínuo — o pico de consumo de energia ocorre durante o Thread 3 (Processamento), que exige uso intensivo da GPU.

### Recomendações de armazenamento

* **SSD NVMe** altamente recomendado para implantações em arm64
* Cartões SD são muito lentos para processamento — use-os apenas como mídia de inicialização
* Planeje 2 a 3 vezes o tamanho dos dados brutos da imagem para a saída processada

### Operação sem monitor via SSH

Chloros CLI é ideal para implantações Jetson sem monitor:

```bash
# SSH into the Jetson
ssh user@jetson-hostname

# Process a dataset
chloros-cli process /data/datasets/flight001 --format tiff-32

# Monitor export progress
chloros-cli export-status
```

### Processamento automatizado com systemd

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

## Fluxos de trabalho de exemplo

### Processamento básico do Jetson

```bash
#!/bin/bash
# Process a drone flight dataset on Jetson
chloros-cli process /data/flights/flight_042 \
    --output /data/processed/flight_042 \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI
```

### Python SDK no Jetson

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
        --format tiff-32 \
        --indices NDVI NDRE
    echo "Completed $name"
done
```

***

## Sistemas Jetson recomendados para uso em campo

Para implantações em campo e aéreas, considere estas opções de placas de suporte Jetson Orin NX de 16 GB:

* **Aéreo/drone**: Sistemas com classificação de vibração (MIL-STD), leves (menos de 300 g) e resfriamento passivo
* **Campo robusto**: Gabinetes à prova d&#x27;água IP67/IP69K com conectividade para câmera PoE GigE
* **Mínimo/econômico**: kits de desenvolvedor com gabinetes adicionais

Entre em contato com o [Suporte da MAPIR](https://www.mapir.camera/community/contact) para obter recomendações específicas de hardware para o seu cenário de implantação.

***

## Próximos passos

* [Linux Instalação](linux-installation.md) — Detalhes gerais da instalação do Linux
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Referência completa da estratégia de computação
* [Pipeline de processamento](../processing-architecture/processing-pipeline.md) — Entendendo o pipeline de 4 threads
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
* [API : Python SDK](../api-python-sdk.md) — Referência completa do SDK
