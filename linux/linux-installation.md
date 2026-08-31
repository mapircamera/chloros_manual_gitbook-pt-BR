# Instalação do Linux

O Chloros é distribuído para o Linux como pacotes `.deb` que instalam o CLI e o servidor backend. O Python SDK é um pacote pip separado (também incluído no `.deb` como um wheel com versão compatível).

Os nomes dos arquivos dos pacotes indicam a versão e a arquitetura: `chloros_1.2.0_amd64.deb` para x86_64 e `chloros_1.2.0_arm64_jp6.deb` para compilações do JetPack 6 para Jetson. Substitua o arquivo que você realmente baixou nos comandos abaixo.

***

## Linux amd64 (x86_64)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribuição** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processador** | x86_64 (Intel/AMD) | Intel Core i7 ou superior |
| **Memória (RAM)** | 8 GB | 16 GB ou mais |
| **Placa de vídeo** | Nenhuma (processamento pela CPU) | GPU NVIDIA com 4 GB ou mais de VRAM (12 GB ou mais desbloqueia o `GPU_PARALLEL`; 7 GB ou mais mantém o Texture Aware desativado no caminho de imagem única) |
| **Armazenamento** | 2 GB de espaço livre | SSD com 10 GB ou mais de espaço livre |
| **Python** | Python 3.7 ou superior (para o SDK) | Python 3.10+ |

> **O Ubuntu 20.04 e o Debian 11 não são suportados.** A lista de dependências do `.deb` é
> derivada do que o backend Chloros realmente vincula, e isso inclui
> o `libc6 (>= 2.34)`. Tanto o Focal quanto o bullseye vêm com a glibc 2.31; por isso, o `apt` recusa a
> instalação imediatamente, em vez de permitir que ela falhe posteriormente durante a execução.

### Instalação

```bash
sudo dpkg -i chloros_1.2.0_amd64.deb
sudo apt-get install -f    # pulls the declared dependencies (libibverbs1, libcap2-bin)
```

{% hint style="info" %}
O `dpkg -i` não resolve as dependências. Se ele relatar a falta de pacotes, o `sudo apt-get install -f` (ou o `sudo apt --fix-broken install`) conclui a instalação — esse é o caminho normal, não um erro.
{% endhint %}

Verifique a instalação:



<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo dpkg -i chloros_1.2.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block naming the bundled wheel path under /usr/lib/chloros/sdk/, any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend.service" hint -->

```bash
chloros-cli --version    # prints "Chloros CLI 1.2.0"
```***

## Linux arm64 (NVIDIA Jetson)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 mais recente |
| **Memória (RAM)** | 8 GB (compartilhada entre GPU e CPU) | 16 GB ou mais compartilhados (12 GB ou mais é o limite mínimo para workers paralelos da GPU) |
| **Armazenamento** | 2 GB de espaço livre | SSD NVMe com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para o SDK) | Python 3.10+ |

### Instalação

```bash
sudo dpkg -i chloros_1.2.0_arm64_jp6.deb
sudo apt-get install -f
chloros-cli --version
```

Mesmo layout que o amd64 `.deb`, com uma compilação CUDA otimizada para Jetson Orin / Orin NX / Orin Nano. Para obter informações sobre o comportamento da memória, térmica e de implantação em campo do Jetson, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md).

***

## Instalação do Python e do SDK (todos os Linux)

O SDK é um-Python HTTP para o backend; portanto, o mesmo pacote funciona em amd64 e arm64. Duas fontes:**Do PyPI** — a versão estável publicada:

```bash
pip install chloros-sdk
```

**Do wheel incluído** — garantidamente compatível com o CLI/backend que você acabou de instalar (use isso quando o seu `.deb` for mais recente que o do PyPI):

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

{% hint style="warning" %}
**Distribuições PEP 668** (Ubuntu 23.10+, Debian 12+) não permitem instalações via pip em todo o sistema. Use o `pip install --user …`, um ambiente virtual ou o `sudo pip install --break-system-packages …`. O instalador de pacotes nunca instala automaticamente o SDK no seu sistema Python — essa escolha fica a seu critério.
{% endhint %}

Extras opcionais:

| Extra | Comando | Adiciona |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para transmissão de progresso em tempo real |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Verifique o SDK:

```bash
python -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
O `.deb` instala o Chloros, o CLI e o backend. O Python e o SDK se comunicam com esse backend por meio de uma conexão local HTTP e API (`http://127.0.0.1:5000`) e o inicia automaticamente quando necessário. Sempre use o endereço IPv4 literal em vez de `localhost` — `localhost` pode ser resolvido como `::1` e levar cerca de dois segundos por solicitação.
{% endhint %}

***

## Configuração inicial

### 1. Faça login

O acesso a CLI e SDK requer um plano pago Chloros+ (**Copper** ou superior), aplicada no lado do servidor: um chamador desconectado recebe `401 AUTH_REQUIRED`, e um chamador do plano gratuito (Iron) recebe `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

As credenciais são armazenadas em cache no `~/.chloros/user_session.json`.

{% hint style="warning" %}
**É necessário fazer login novamente após cada instalação ou atualização.** O script `prerm` do pacote limpa deliberadamente o `~/.chloros/user_session.json` e a licença armazenada em cache para todos os usuários da máquina, de modo que uma nova compilação sempre revalida a licença em vez de confiar em um cache desatualizado.
{% endhint %}

### 2. Verifique o status da sua licença

```bash
chloros-cli status
```

O `chloros-cli status` funciona em qualquer plano (incluindo o gratuito), para que você sempre possa verificar por que o acesso está ou não disponível.

### 3. Execute o diagnóstico do sistema

```bash
chloros-cli selftest
```

Sete verificações são executadas em sequência, e o comando retorna um valor diferente de zero se alguma delas falhar:

| # | Verificação | O que ela comprova |
| --- | --- | --- |
| 1 | **Versão** | O CLI informa sua versão (`v1.2.0`). |
| 2 | **Porta disponível** | A porta 5000 está livre *ou* já foi respondida por um backend Chloros em bom estado (o que conta como aprovação). |
| 3 | **Inicialização do backend** | O binário do backend é iniciado. |
| 4 | **Teste do API (`/api/test`)** | O backend responde com `status: ok`. |
| 5 | **Informações do sistema** | Exibe `GPU: <name>, CUDA: <bool>, PyTorch: <version>` a partir de `/api/system-info`. |
| 6 | **Modelos do denoiser** | Encontra os modelos `*.pth.enc` (em Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Redutor de ruído**| O Texture Aware está de fato pronto para uso — requer CUDA**e** pelo menos um arquivo de modelo. |

A execução termina com `N/7 checks passed`, listando todas as falhas por nome.

### 4. Processe seu primeiro conjunto de dados

```bash
chloros-cli process ~/datasets/flight001
```

***

## Arquivos e diretórios

### Por usuário

O Chloros mantém suas credenciais e a configuração do CLI em um único diretório multiplataforma, **`~/.chloros/`** (no Windows, `%USERPROFILE%\.chloros\`). Dois caches específicos do Linux seguem as convenções XDG — estes respeitam as configurações do `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando definidas.

| Caminho | Finalidade |
| --- | --- |
| `~/.chloros/user_session.json` | Cache da sessão de login gravado por `chloros-cli login` (limpo a cada instalação/atualização de pacote) |
| `~/.chloros/working_directory.txt` | Substituição da pasta padrão do projeto (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferência de idioma do CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuração de idioma compartilhada com a interface gráfica do Windows — um valor de `language` aqui tem prioridade sobre o de `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de uma hora para a verificação de atualizações na inicialização do Linux/Jetson |
| `~/.chloros/backend.log` | Log do backend quando este foi iniciado pelo CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacotes de calibração LATTICE armazenados em cache por câmera, indexados por número de série e hash do pacote |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Substituições opcionais do usuário para perfis de correção de limite do DAQ |
| `~/.config/chloros/system_config.json` | Perfil de hardware armazenado em cache proveniente da Adaptação Dinâmica de Computação — exclua-o para forçar uma nova detecção de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Logs do servidor backend, um arquivo por inicialização |
| `~/Chloros Projects/` | Pasta padrão do projeto quando nenhuma substituição estiver definida |

### Em todo o sistema

| Caminho | Finalidade |
| --- | --- |
| `/usr/bin/chloros-cli` | Script wrapper — define `LD_LIBRARY_PATH` para as bibliotecas nativas incluídas no pacote e, em seguida, executa o binário real |
| `/usr/bin/chloros-backend` | Script wrapper — o mesmo, além de `CHLOROS_PRODUCTION=1` para que o mecanismo de autenticação do backend nunca possa se desativar silenciosamente |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Os binários compilados |
| `/usr/lib/chloros/arena_runtime/` | Ambiente de execução Arena SDK exigido pelas câmeras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos criptografados de redução de ruído usados pelo debayer sensível à textura |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Python e SDK – conjunto de pacotes correspondente exatamente a esta compilação |
| `/usr/lib/chloros/exiftool` | exiftool incluído (com link simbólico para o `/usr/local/bin/exiftool` somente se não houver um exiftool no sistema) |
| `/etc/chloros/update.conf` | Configuração do canal de atualização lida pelo `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Define o `net.ipv4.ip_unprivileged_port_start = 319` para que o backend possa vincular as portas PTP sem privilégios de root |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Direciona o carregador dinâmico para o `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede ao usuário conectado acesso à ponte serial USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend.service` | Ativa o serviço de backend sempre ativo (instalado, **não habilitado**) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada no menu do aplicativo “Chloros CLI” que abre um terminal |

## Localização do executável de back-end

O CLI e o SDK detectam automaticamente o back-end:

| Componente | Caminho |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` |

Substitua o caminho do backend com o sinalizador `--backend-exe` CLI ou o parâmetro do construtor `backend_exe` SDK, e a porta com `--port` (padrão: `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` aponta para as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para um backend remoto. Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoram-no deliberadamente e sempre têm como alvo o `http://127.0.0.1:<port>`.
{% endhint %}

***

## Câmeras LATTICE e sensores de luz DAQ no Linux

Todas as famílias de comandos de hardware em tempo real funcionam no Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — detecta, conecta, configura e captura imagens de câmeras LATTICE e matrizes sincronizadas. O `.deb` inclui o runtime Arena SDK necessário e o registra no carregador dinâmico.
* **`chloros-cli daq pool-*`** — conectar sensores de luz DAQ-U/M/E por meio do pool de back-end, transmitir espectros calibrados e gravar arquivos `.daq`. O CLI compilado inclui apenas a família `pool-*`: `pool-connect`, `pool-disconnect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`.
* **`chloros-cli project`** — executa um projeto salvo (suas câmeras, sensores e configurações de processamento) sem interface gráfica.
* **`chloros-cli time-sync`** — inspecionar o “grandmaster” PTP no qual o backend Chloros é executado para câmeras LATTICE e sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

O `--sensor-id` é exigido pelo `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; o `pool-list` mostra os IDs atualmente no pool.

{% hint style="info" %}
**Prefira o `--eth-host` para a primeira conexão do DAQ-E em uma máquina com múltiplos endereços de rede.** A descoberta automática vasculha o mDNS e pode não detectar a interface do sensor devido a um cache ARP vazio; portanto, o primeiro `pool-connect --eth` após a inicialização pode falhar, mesmo que o sensor esteja em perfeitas condições. Passar o IP ou o nome do host do sensor ignora completamente a descoberta.
{% endhint %}

**As permissões seriais do DAQ-U** são gerenciadas pela regra udev instalada (`uaccess` + grupo `dialout`). Se um sensor que já estava conectado permanecer inacessível, recarregue as regras ou reconecte-o:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulte a [referência do CLI](../CLI.md) para obter o conjunto completo de comandos.

### PTP sempre ativo para hosts sem interface gráfica

Na primeira instalação, a unidade systemd `chloros-backend.service` é gerada, mas **não está habilitada**. Em um Jetson sem monitor ou servidor que deva manter a sincronização de tempo PTP em execução contínua para sensores DAQ-E e câmeras LATTICE, habilite-a:

```bash
sudo systemctl enable --now chloros-backend.service
sudo systemctl status chloros-backend.service
```

Sem isso, o PTP é executado apenas enquanto o backend Chloros estiver em execução — ou seja, durante uma sessão ativa do CLI/SDK.

A unidade vincula o backend às configurações de ambiente `127.0.0.1:5000` (configurações de ambiente `CHLOROS_HOST` / `CHLOROS_PORT` dentro da unidade; substitua por `sudo systemctl edit chloros-backend.service`) e o reinicia em caso de falha após 5 segundos.

**Como o PTP obtém suas portas.** O PTP usa as portas UDP 319/320, ambas abaixo do limite normal de 1.024 para portas privilegiadas. O pacote `postinst` grava `/etc/sysctl.d/60-chloros-ptp.conf` com `net.ipv4.ip_unprivileged_port_start = 319`, o que permite que o backend as vincule enquanto é executado como seu usuário. Ele também aplica o `setcap cap_net_bind_service,cap_net_raw=+ep` ao binário do backend como uma medida de segurança — é por isso que o `libcap2-bin` é uma dependência declarada do pacote.***

## Exemplos de scripts em Bash

{% hint style="info" %}
**Códigos de saída compatíveis com scripts.**O `chloros-cli process` encerra o `0` em caso de sucesso e**com valor diferente de zero em caso de falha — incluindo uma execução que solicitou produtos de imagem, mas não gravou nenhum** (ele exibe o código `Processing finished but wrote no image products.` e indica o nome da pasta do projeto e as causas habituais). Execuções bem-sucedidas informam quantos produtos de imagem foram gerados (`Image products written: N`). Códigos de saída: `0` sucesso, `1` falha, `2` erro de argumento, `130` interrompido.
{% endhint %}

### Processar vários conjuntos de dados

```bash
#!/bin/bash
for dataset in ~/datasets/2026/*/; do
    echo "Processing $(basename "$dataset")..."
    if chloros-cli process "$dataset" --format "TIFF (32-bit, Percent)"; then
        echo "Done: $(basename "$dataset")"
    else
        echo "FAILED: $(basename "$dataset")" >&2
    fi
done
```

### Processar com configurações personalizadas

```bash
#!/bin/bash
chloros-cli process ~/datasets/field_a \
    --output ~/output/field_a \
    --format "TIFF (32-bit, Percent)" \
    --indices NDVI NDRE GNDVI \
    --debayer texture-aware \
    --no-vignette
```

Os valores válidos para `--format` são exatamente quatro e contêm espaços — sempre coloque-os entre aspas:

| Valor `--format` | Pasta de saída |
| --- | --- |
| `TIFF (16-bit)` *(padrão)* | `tiff16` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` aceita `standard` (padrão) ou `texture-aware` (Chloros+).

### Processamento automatizado com Cron

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

### CLI não encontrado após a instalação

```bash
# Check if the binary exists
which chloros-cli
ls -la /usr/bin/chloros-cli

# List everything the package installed
dpkg -L chloros

# Reload your shell
source ~/.bashrc
```

### Permissão negada

```bash
sudo chmod +x /usr/bin/chloros-cli
sudo chmod +x /usr/lib/chloros/chloros-backend
```

### “setcap falhou” durante a instalação

O `.deb` aplica o `cap_net_bind_service` ao `/usr/lib/chloros/chloros-backend` para que ele possa vincular as portas PTP 319/320 sem privilégios de root. Se o `libcap2-bin` não estiver presente no momento da instalação, a chamada é ignorada. Instale-o e reinstale o pacote:

```bash
sudo apt install libcap2-bin
sudo apt reinstall chloros
```

### O PTP não inicia / não consegue vincular a porta 319

Confirme se o limite mínimo de portas sem privilégios foi reduzido e, caso não tenha sido, reaplique-o para a inicialização atual:

```bash
sysctl net.ipv4.ip_unprivileged_port_start     # expect 319
sudo sysctl -w net.ipv4.ip_unprivileged_port_start=319
```

Em seguida, verifique o grandmaster:

```bash
chloros-cli time-sync status
chloros-cli time-sync peers
```

### “Drivers da câmera LATTICE não encontrados”

O tempo de execução do Arena SDK não está sendo resolvido. Confirme se a configuração do carregador que o pacote grava está presente e atualizada:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
sudo ldconfig
ls /usr/lib/chloros/arena_runtime | head
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

Os logs do backend relativos à falha na inicialização estão em `~/.cache/chloros/logs/`.

### CUDA não detectado

```bash
# Check NVIDIA driver installation
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check JetPack version
cat /etc/nv_tegra_release
```

O `chloros-cli selftest` relata o mesmo problema em uma linha: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Bibliotecas compartilhadas ausentes

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries
ldd /usr/lib/chloros/chloros-backend | grep "not found"
```

### Inicialização lenta em sistemas com cartão SD

Os binários compilados se extraem para um diretório temporário a cada inicialização. Se o `/mnt/ssd/tmp` existir, o Chloros o utiliza automaticamente; caso contrário, configure o `TMPDIR` para um sistema de arquivos rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Atualização do Chloros no Linux

O comando `update` é exclusivo para o Linux/Jetson. Ele verifica a versão publicada no canal de atualização configurado em `/etc/chloros/update.conf` e oferece o download e a instalação do `.deb` correspondente:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

No Linux/Jetson, o CLI também executa uma verificação de atualização sem bloqueio a cada inicialização (resultado armazenado em cache por uma hora no `~/.chloros/update_cache.json`) e exibe o `Update available: vX.Y.Z` quando existe uma versão mais recente. Suas configurações e projetos são preservados durante a atualização; você precisará fazer login novamente após a atualização.

## Desinstalação

```bash
sudo apt remove chloros
```

A remoção interrompe o `chloros-backend.service`, restaura o limite mínimo padrão para portas sem privilégios (1024), remove o link simbólico do exiftool incluído e a configuração do carregador Arena, além de limpar as credenciais armazenadas em cache. Seus projetos e arquivos de dados `~/.chloros/` não são alterados.

***

## Próximos passos

* [Guia NVIDIA Jetson](nvidia-jetson-guide.md) — otimização e implantação específicas para o Jetson
* [CLI : Linha de comando](../CLI.md) — o guia CLI
* [API : Python SDK](../api-python-sdk.md) — o guia SDK
* [Referência do CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas completas de comandos/API para a versão 1.2.0
* [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware
