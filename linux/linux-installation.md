# Instalação do Linux

O Chloros é distribuído para o Linux na forma de pacotes `.deb` que instalam o CLI e o backend. O Python SDK é instalado separadamente via pip.

***

## Linux amd64 (x86_64)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribuição** | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+ |
| **Processador** | x86_64 (Intel/AMD) | Intel Core i7 ou superior |
| **Memória (RAM)** | 8 GB | 16 GB ou mais |
| **Placa de vídeo** | Nenhuma (processamento pela CPU) | GPU NVIDIA com 4 GB+ de VRAM |
| **Armazenamento** | 2 GB de espaço livre | SSD com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para SDK) | Python 3.10+ |

### Instalação

Baixe o pacote `.deb` e instale:

```bash
sudo dpkg -i chloros-amd64.deb
```

Verifique a instalação:

```bash
chloros-cli --version
```

***

## Linux arm64 (NVIDIA Jetson)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 mais recente |
| **Memória (RAM)** | 8 GB (GPU/CPU compartilhada) | 16 GB+ compartilhada |
| **Armazenamento** | 2 GB de espaço livre | SSD NVMe com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para SDK) | Python 3.10+ |

### Instalação

Baixe o pacote JetPack 6 `.deb` e instale:

```bash
sudo dpkg -i chloros-arm64-jp6.deb
```

Verifique a instalação:

```bash
chloros-cli --version
```

Para obter informações detalhadas sobre a configuração do Jetson, incluindo gerenciamento térmico e implantação em campo, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalação do Python SDK (Todos os Linux)

O Python SDK é instalado separadamente via pip e funciona tanto em amd64 quanto em arm64:

```bash
pip install chloros-sdk
```

Para incluir suporte opcional a streaming de progresso:

```bash
pip install chloros-sdk[progress]
```

Verifique o SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
O pacote `.deb` instala o Chloros, o CLI e o backend. O Python SDK é um pacote pip separado que se comunica com o backend por meio de um HTTP API local.
{% endhint %}

***

## Diretórios de configuração

O Chloros no Linux segue a [Especificação de Diretório Base XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html):

| Finalidade | Linux Caminho | Windows Equivalente |
| --- | --- | --- |
| **Configuração** | `~/.config/chloros/` | `%APPDATA%\Chloros\` |
| **Dados / Projetos** | `~/.local/share/chloros/` | `%LOCALAPPDATA%\Chloros\` |
| **Cache / Credenciais** | `~/.cache/chloros/` | `%APPDATA%\Chloros\cache\` |

## Locais dos executáveis do backend

O pacote `.deb` instala o backend em um local padrão. O CLI e o SDK detectam automaticamente o caminho do backend:

| Método de instalação | Caminho do backend |
| --- | --- |
| Pacote `.deb` | `/usr/lib/chloros/chloros-backend` |
| Manual / personalizado | `/opt/mapir/chloros/backend/chloros-backend` |

Você pode substituir o caminho do backend com o sinalizador `--backend-exe` CLI ou o parâmetro do construtor `backend_exe` SDK.

***

## Configuração inicial

### 1. Ative sua licença

É necessária uma licença Chloros+ para acesso ao CLI e ao SDK:

```bash
chloros-cli login your@email.com 'your-password'
```

### 2. Verifique o status da sua licença

```bash
chloros-cli status
```

### 3. Processe seu primeiro conjunto de dados

```bash
chloros-cli process ~/datasets/flight001
```

### 4. Execute o diagnóstico do sistema

Verifique se o seu sistema está configurado corretamente:

```bash
chloros-cli selftest
```

Isso executa 7 verificações de diagnóstico, incluindo versão, inicialização do backend, conectividade e disponibilidade de CUDA/GPU.

***

## Exemplos de scripts Bash

### Processar vários conjuntos de dados

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    chloros-cli process "$dataset" --format tiff-32
    echo "Done: $(basename "$dataset")"
done
```

### Processar com configurações personalizadas

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format tiff-32 \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

### Processamento automatizado com Cron

Adicione ao seu crontab (`crontab -e`) para processar novos conjuntos de dados automaticamente:

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplo de Python e SDK

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "NDRE"],
    export_format="TIFF (32-bit, Percent)"
)
```

***

## Solução de problemas

### CLI Não encontrado após a instalação

Se o `chloros-cli` não for encontrado após a instalação do pacote `.deb`:

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# If not in PATH, check the installation
dpkg -L chloros-amd64  # or chloros-arm64-jp6

# Reload your shell
source ~/.bashrc
```

### Permissão negada

```bash
# Ensure the binary is executable
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### Falha ao iniciar o backend

```bash
# Check if port 5000 is already in use
lsof -i :5000

# Kill any existing process on port 5000
kill $(lsof -t -i :5000)

# Try starting with a different port
chloros-cli --port 5001 process ~/datasets/flight001
```

### CUDA não detectado

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

### Bibliotecas compartilhadas ausentes

```bash
# Install common dependencies
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

***

## Atualizando o Chloros no Linux

Use o comando de atualização integrado para verificar e instalar atualizações:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

***

## Próximos passos

* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — Otimização e implantação específicas para Jetson
* [CLI : Linha de comando](../CLI.md) — Referência completa de comandos do CLI
* [API : Python SDK](../api-python-sdk.md) — Referência completa do SDK
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Como o Chloros se adapta ao seu hardware
