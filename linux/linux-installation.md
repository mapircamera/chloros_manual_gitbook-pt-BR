# Instalação no Linux

O Chloros é distribuído para o Linux como pacotes `.deb`, que instalam a CLI, o servidor backend e o SDK do Python (para o sistema `python3`, além de um wheel compatível com a versão para virtualenvs).

Os nomes dos arquivos dos pacotes indicam a versão e a arquitetura: `chloros_1.3.0_amd64.deb` para x86_64, e `chloros_1.3.0_arm64_jp6.deb` para arm64 (NVIDIA Jetson, JetPack 6). Substitua o nome do arquivo que você realmente baixou nos comandos abaixo.

{% hint style="info" %}
**Novidade na versão 1.3.0: os programas são fornecidos descompactados.** O backend e a CLI são instalados como pastas, `/usr/lib/chloros/backend/` e `/usr/lib/chloros/cli/`, em vez de arquivos autoextraíveis individuais. Eles não se descompactam mais em uma pasta temporária a cada inicialização, portanto, iniciam muito mais rapidamente (no x86_64, o backend costumava levar cerca de 100 segundos para se descompactar a cada inicialização) e não precisam mais de vários GB de espaço livre em `/tmp`. A versão amd64, `.deb`, também tem cerca de metade do tamanho anterior.
{% endhint %}

***

## Linux amd64 (x86_64)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Distribuição** | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS |
| **Processador** | x86_64 (Intel/AMD) | Intel Core i7 ou superior |
| **Memória (RAM)** | 8 GB | 16 GB ou mais |
| **Placa de vídeo** | Nenhuma (processamento pela CPU) | GPU NVIDIA com 4 GB ou mais de VRAM (12 GB ou mais desbloqueia o `GPU_PARALLEL`; 7 GB ou mais mantém o Texture Aware fora do caminho de imagem única) |
| **Armazenamento** | 2 GB de espaço livre | SSD com 10 GB ou mais de espaço livre |
| **Python** | Python 3.7 ou superior (para o SDK) | Python 3.10 ou superior |

> **O Ubuntu 20.04 e o Debian 11 não são suportados.** A lista de dependências do `.deb` é
> derivada do que o backend Chloros realmente vincula, e isso inclui um
> `libc6` mais recente do que a glibc 2.31 fornecida pelas versões focal e bullseye; por isso, o `apt` recusa a
> instalação imediatamente, em vez de permitir que ela falhe posteriormente durante a execução.

### Instalação

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
```

{% hint style="info" %}
**Use o `apt install ./…`, não o `dpkg -i`.** O `apt` resolve as dependências do pacote e também instala o que recomenda: os requisitos do Python SDK, `python3-numpy` e `python3-requests`. O `dpkg -i` não instala nenhum dos dois. A CLI e o backend são autônomos e funcionam sem eles, mas o `import chloros_sdk` precisa deles; a saída da instalação indica quais estão faltando. Mantenha o `./`, para que o `apt` instale o arquivo em vez de procurar um pacote com esse nome. O pacote também declara o `libgl1`, que o OpenCV incluído carrega e que pode estar ausente em uma imagem mínima de servidor.
{% endhint %}

Verifique a instalação:

```bash
chloros-cli --version    # prints "Chloros CLI 1.3.0"
```

<!-- SCREENSHOT-NEEDED: Terminal on Ubuntu 22.04 immediately after `sudo apt install ./chloros_1.3.0_amd64.deb`, showing the full postinst output: the "Chloros installed successfully!" banner, the Usage lines, the "Python SDK:" block ("chloros_sdk 1.3.0 for python3" and the "In a virtualenv: pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl" line), any "GPU Acceleration:" detection line, and the closing "Systemd Service (optional): sudo systemctl enable --now chloros-backend@<user>.service" hint -->

***

## Linux arm64 (NVIDIA Jetson)

### Requisitos do sistema

| Requisito | Mínimo | Recomendado |
| --- | --- | --- |
| **Plataforma** | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **JetPack** | JetPack 6.x | JetPack 6 mais recente |
| **Memória (RAM)** | 8 GB (compartilhada entre GPU e CPU) | 16 GB+ compartilhados (12 GB+ é o limite mínimo para processamento paralelo) |
| **Armazenamento** | 2 GB de espaço livre | SSD NVMe com 10 GB+ de espaço livre |
| **Python** | Python 3.7+ (para o SDK) | Python 3.10+ |

### Instalação

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version
```

Mesmo layout que o amd64 `.deb` (Python SDK, buffers de recepção e serviços de backend incluídos), para Jetson Orin / Orin NX / Orin Nano no **JetPack 6** (L4T R36). Seu PyTorch é a compilação CUDA do JetPack 6 para o Orin; portanto, o processamento na GPU, incluindo o debayer Texture Aware, é executado na GPU do Jetson. O pacote não inclui o CUDA em si: ele utiliza as bibliotecas CUDA e cuDNN instaladas pelo JetPack 6 (`sudo apt install nvidia-jetpack` em uma imagem que não as possua). Quando nenhuma GPU com CUDA está disponível, o processamento é executado na CPU e o `--debayer texture-aware` utiliza o debayer padrão (o backend registra uma linha `[DEBAYER] … needs a CUDA GPU …` explicando o motivo). Para obter informações sobre o comportamento do Jetson em relação à memória, temperatura e implantação em campo, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md).***

## Instalação do Python SDK (Todos os sistemas Linux)**O `.deb` instala o SDK para você.** Ambos os pacotes (amd64 e Jetson) o instalam para o sistema `python3`, de modo que o `import chloros_sdk` funciona imediatamente após a instalação, sem necessidade de pip nem conexão de rede. Seus pacotes (`chloros_sdk`, `lattice_sdk`, `lattice_cam`, `arena_api` e os carregadores de projeto) vão para o `/usr/lib/python3/dist-packages`, de propriedade do pacote: uma atualização os substitui, e a remoção do pacote os remove. A versão sempre corresponde à CLI e ao backend que você instalou.

Os próprios pré-requisitos do SDK, `python3-numpy` e `python3-requests`, são **recomendados, não obrigatórios** pelo pacote: o `sudo apt install ./chloros_<version>_<arch>.deb` os instala, enquanto o `dpkg -i` não. A saída da instalação informa a versão do SDK que o `python3` importa, qualquer requisito que esteja faltando, e qualquer `chloros_sdk` em outro local no caminho do `python3` que substitua a cópia do pacote (como um instalado anteriormente com o `pip install --user`).**Em um virtualenv**, que não tem acesso aos pacotes do sistema, instale o wheel incluído nele (totalmente offline):

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

**Do PyPI** — apenas para uma máquina sem o pacote Chloros instalado, cujo SDK aciona um backend em outra máquina (consulte [Modo de Backend Remoto](../reference/sdk-reference.md#remote-backend-mode-pip-only-host-via-tunnel)):

```bash
pip install chloros-sdk
```

Extras opcionais, para uma instalação via pip (em um virtualenv ou no PyPI):

| Extra | Comando | Adiciona |
| --- | --- | --- |
| `progress` | `pip install chloros-sdk[progress]` | `sseclient-py` para transmissão de progresso em tempo real |
| `camera` | `pip install chloros-sdk[camera]` | `bleak` para transporte BLE (DAQ-M) |

Verifique o SDK:

```bash
python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"
```

{% hint style="info" %}
O SDK do Python se comunica com o backend que o `.deb` instala por meio de uma HTTP API local (`http://127.0.0.1:5000`). O `ChlorosLocal` e os auxiliares de conexão o iniciam automaticamente quando necessário; o `connect_all()` de um projeto salvo nunca o faz, portanto, em uma máquina sem interface gráfica, execute o back-end como um [serviço](#always-on-ptp-for-headless-hosts). Sempre use o endereço IPv4 literal em vez de `localhost` — o `localhost` pode ser resolvido como `::1` e levar cerca de dois segundos por solicitação.
{% endhint %}

***

## Configuração inicial

### 1. Faça login

O acesso à CLI e ao SDK requer um plano pago do Chloros+ (**Copper** ou superior), aplicado no lado do servidor: um usuário desconectado recebe `401 AUTH_REQUIRED`, e um usuário do plano gratuito (Iron) recebe `403 PLAN_UPGRADE_REQUIRED`.

```bash
chloros-cli login your@email.com 'your-password'
```

As credenciais são armazenadas em cache em `~/.chloros/user_session.json`.

{% hint style="info" %}
**As atualizações mantêm seu login; a remoção o apaga.** A instalação de uma versão mais recente do `.deb` sobre uma instalação existente (inclusive por meio do `chloros-cli update`) mantém o `~/.chloros/user_session.json` e a licença armazenada em cache, de modo que as tarefas CLI/SDK sem interface gráfica continuam funcionando. A remoção do pacote (`sudo apt remove chloros`) apaga o login salvo e o cache de licença de todos os usuários na máquina; portanto, uma nova instalação exigirá o `chloros-cli login` novamente.
{% endhint %}

### 2. Verifique o status da sua licença

```bash
chloros-cli status
```

O `chloros-cli status` funciona em qualquer plano (incluindo o gratuito), portanto, você sempre pode verificar por que o acesso está ou não disponível.

### 3. Execute o diagnóstico do sistema

```bash
chloros-cli selftest
```

Sete verificações são executadas em ordem, e o comando retorna um valor diferente de zero se alguma delas falhar:

| # | Verificação | O que ela comprova |
| --- | --- | --- |
| 1 | **Versão** | A CLI informa sua versão (`v1.3.0`). |
| 2 | **Porta disponível** | A porta 5000 está livre, *ou* já foi respondida por um backend do Chloros em bom estado (o que conta como aprovado). |
| 3 | **Inicialização do backend** | O binário do backend é iniciado. |
| 4 | **Teste de API (`/api/test`)** | O backend responde com `status: ok`. |
| 5 | **Informações do sistema** | Exibe `GPU: <name>, CUDA: <bool>, PyTorch: <version>` a partir de `/api/system-info`. |
| 6 | **Modelos do denoiser** | Encontra modelos `*.pth.enc` (no Linux: `/usr/lib/chloros/models`). |
| 7 | **CUDA + Redutor de ruído**| O Texture Aware está de fato pronto para uso — requer CUDA**e** pelo menos um arquivo de modelo. |

A execução termina com o `N/7 checks passed`, listando todas as falhas por nome.

### 4. Processe seu primeiro conjunto de dados

```bash
chloros-cli process ~/datasets/flight001
```

***

## Arquivos e diretórios

### Por usuário O

Chloros mantém suas credenciais e configuração da CLI em um único diretório multiplataforma, **`~/.chloros/`** (no Windows, `%USERPROFILE%\.chloros\`). Dois caches específicos do Linux seguem as convenções XDG — estes respeitam `XDG_CONFIG_HOME` / `XDG_CACHE_HOME` quando definidos.

| Caminho | Finalidade |
| --- | --- |
| `~/.chloros/user_session.json` | Cache da sessão de login gravado por `chloros-cli login` (mantido durante atualizações do pacote; limpo quando o pacote é removido) |
| `~/.chloros/working_directory.txt` | Substituição da pasta padrão do projeto (`chloros-cli set-project-folder` / `get-project-folder` / `reset-project-folder`) |
| `~/.chloros/cli_language.json` | Preferência de idioma da CLI (`chloros-cli language <code>`) |
| `~/.chloros/user.json` | Configuração de idioma compartilhada com a interface gráfica do Windows — um `language` aqui tem prioridade sobre o `cli_language.json` |
| `~/.chloros/update_cache.json` | Cache de uma hora para a verificação de atualizações na inicialização do Linux/Jetson |
| `~/.chloros/backend.log` | Log do backend quando este foi iniciado pela CLI |
| `~/.chloros/camera_cal/<serial>/<bundle_sha>/` | Pacotes de calibração LATTICE armazenados em cache por câmera, indexados por número de série e hash do pacote |
| `~/.chloros/daq_cap_profiles/<u\|m\|e>/<cap_id>.json` | Substituições opcionais do usuário para perfis de correção de cap do DAQ |
| `~/.config/chloros/system_config.json` | Perfil de hardware armazenado em cache da Adaptação Dinâmica de Computação — exclua-o para forçar uma nova detecção de hardware |
| `~/.cache/chloros/logs/backend_<YYYYMMDD_HHMMSS>.log` | Logs do servidor backend, um arquivo por inicialização |
| `~/Documents/MAPIR/Chloros Projects/` | Pasta padrão do projeto quando nenhuma substituição estiver definida |

### Em todo o sistema

| Caminho | Finalidade |
| --- | --- |
| `/usr/bin/chloros-cli` | Script wrapper — define `LD_LIBRARY_PATH` para as bibliotecas nativas incluídas (e de `TMPDIR` para `/mnt/ssd/tmp` quando essa pasta existe) e, em seguida, executa o binário real |
| `/usr/bin/chloros-backend` | Script wrapper — o mesmo para o backend, que ele marca como uma compilação de produção instalada |
| `/usr/lib/chloros/cli/`, `/usr/lib/chloros/backend/` | A CLI e o backend compilados, instalados descompactados (novidade na versão 1.3.0); os programas são `cli/chloros-cli` e `backend/chloros-backend` |
| `/usr/lib/chloros/chloros-cli`, `/usr/lib/chloros/chloros-backend` | Scripts de inicialização de duas linhas que iniciam os programas reais acima — mantidos nesses caminhos porque o SDK, o CLI e os scripts mais antigos os procuram aqui |
| `/usr/lib/chloros/arena_runtime/` | Tempo de execução do SDK da Arena exigido pelas câmeras LATTICE |
| `/usr/lib/chloros/models/*.pth.enc` | Modelos criptografados de redução de ruído usados pelo debayer Texture Aware |
| `/usr/lib/python3/dist-packages/chloros_sdk/` (e `lattice_sdk`, `lattice_cam`, `arena_api`, os carregadores de projeto) | O SDK Python, instalado para o sistema `python3` e pertencente ao pacote |
| `/usr/lib/chloros/sdk/chloros_sdk-*.whl` | Wheel do Python SDK correspondente exatamente a esta compilação, para um virtualenv |
| `/usr/lib/chloros/exiftool` | Iniciador do ExifTool incluído (com link simbólico para `/usr/local/bin/exiftool` apenas se não houver um exiftool do sistema). A partir da versão 1.3.0, o script ExifTool vem com sua própria biblioteca Perl em `/usr/lib/chloros/exiftool-dist/`, portanto, ele é executado sem o pacote do sistema `libimage-exiftool-perl` |
| `/etc/chloros/update.conf` | Configuração do canal de atualização lida pelo `chloros-cli update` |
| `/etc/sysctl.d/60-chloros-ptp.conf` | Define o `net.ipv4.ip_unprivileged_port_start = 319` para que o backend possa vincular as portas PTP sem privilégios de root |
| `/etc/sysctl.d/60-chloros-gige.conf` | Aumenta os buffers de recepção de soquetes do kernel (`net.core.rmem_max`, `net.core.rmem_default`) para 32 MB para fluxos de câmera LATTICE e os aplica na instalação (um valor já definido como maior é mantido). Com a configuração padrão do kernel, os pacotes da câmera são descartados no kernel e um array perde quadros e taxa de quadros; `chloros-cli lattice network` avisa quando os buffers estão baixos. Consulte [Hosts Linux — buffers de recepção de soquetes](../reference/cli-reference.md#linux-hosts--socket-receive-buffers) |
| `/etc/ld.so.conf.d/Arena_SDK.conf` | Direciona o carregador dinâmico para `/usr/lib/chloros/arena_runtime` |
| `/lib/udev/rules.d/70-chloros-daq.rules` | Concede ao usuário conectado acesso à ponte serial USB DAQ-U (CP2102N, `10c4:ea60`) |
| `/lib/systemd/system/chloros-backend@.service` | Serviço de back-end sempre ativo (opt-in) que executa o back-end como o usuário que você nomear (instalado, **não habilitado**) — este é o que deve ser habilitado; consulte [PTP sempre ativo](#always-on-ptp-for-headless-hosts) |
| `/lib/systemd/system/chloros-backend.service` | O mesmo serviço que executa o backend como **root**(instalado,**não ativado**), mantido para que uma máquina que o tenha ativado continue em execução |
| `/var/log/chloros/` | O log detalhado do serviço root, legível pelo grupo `adm` (excluído quando o pacote é removido) |
| `/usr/share/applications/chloros-cli.desktop` | Entrada no menu de aplicativos “Chloros CLI” que abre um terminal |

**Firewall (novidade na versão 1.3.0).** Se o `ufw` ou o `firewalld` estiver ativo no momento da instalação, o pacote abre as portas UDP `5002` (dados DAQ-E), `5353` (descoberta mDNS), `319` e `320` (PTP), e remove essas regras quando o pacote é desinstalado. Se um firewall estiver instalado, mas não estiver ativo, ele não adiciona nada e exibe as portas a serem permitidas caso você o habilite posteriormente. Ele não abre nenhuma porta TCP: uma atualização de firmware do DAQ-E, na qual o sensor se conecta novamente a este computador via TCP para coletar a imagem, falha com o código de erro `… device never connected back on TCP <port> …` enquanto um firewall ativo bloqueia essa conexão.

## Localização do executável do backend

A CLI e o SDK detectam automaticamente o backend:

| Componente | Caminho |
| --- | --- |
| CLI | `/usr/bin/chloros-cli` |
| Backend | `/usr/lib/chloros/chloros-backend` (um iniciador para o `/usr/lib/chloros/backend/chloros-backend`) |

Substitua o caminho do backend com o sinalizador da CLI `--backend-exe` ou o parâmetro do construtor do SDK `backend_exe`, e a porta com `--port` (padrão: `5000`).

{% hint style="info" %}
`CHLOROS_BACKEND_URL` aponta para as famílias de comandos **`lattice`**,**`project`**e**`daq pool-*`** para um backend remoto. Os comandos principais (`process`, `login`, `logout`, `status`, `export-status`, `time-sync`, `selftest`) ignoram-no deliberadamente e sempre têm como alvo o `http://127.0.0.1:<port>`.
{% endhint %}

***

## Câmeras LATTICE e sensores de luz DAQ no Linux

Todas as famílias de comandos live-hardware funcionam no Linux (amd64 e Jetson):

* **`chloros-cli lattice`** — detecta, conecta, configura e captura imagens de câmeras LATTICE e matrizes sincronizadas. O `.deb` inclui o ambiente de execução do SDK do Arena necessário e o registra no carregador dinâmico.
* **`chloros-cli daq pool-*`** — conecta sensores de luz DAQ-U/M/E por meio do pool de back-end, transmite espectros calibrados, grava arquivos `.daq` e troca perfis de cap, e, para um DAQ-E, altera suas configurações de rede, reinicia-o e atualiza seu firmware. O `chloros-cli daq discover` procura sensores que você possa conectar, e o `chloros-cli daq --help` lista os subcomandos do `pool-*`.
* **`chloros-cli project`** — executa um projeto salvo (suas câmeras, sensores e configurações de processamento) sem interface gráfica.
* **`chloros-cli time-sync`** — inspecionar o grandmaster PTP que o backend Chloros executa para câmeras LATTICE e sensores DAQ-E.

```bash
# DAQ-E at a known address — the reliable path on multi-homed hosts
chloros-cli daq pool-connect --eth-host 192.168.2.50

# DAQ-U over USB serial
chloros-cli daq pool-connect --port /dev/ttyUSB0

# What is connected, then the latest calibrated spectrum as JSON
chloros-cli daq pool-list
chloros-cli daq pool-latest --sensor-id daq-e-a1b2c3 --json
```

O comando `--sensor-id` é pré-requisito para os comandos `pool-latest`, `pool-stream`, `pool-record` e `pool-set-cap`; o `pool-list` exibe os IDs atualmente no pool.

{% hint style="info" %}
**`--eth-host` ignora a descoberta.** `pool-connect --eth` localiza um DAQ-E por mDNS, com um fallback de ARP que funciona a partir de um cache de ARP vazio; ao passar o IP ou o nome do host do sensor com `--eth-host`, ele se conecta diretamente a ele.
{% endhint %}

**As permissões seriais do DAQ-U** são gerenciadas pela regra udev instalada (`uaccess` + grupo `dialout`). Se um sensor que já estava conectado permanecer inacessível, recarregue as regras ou reconecte-o:

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger --subsystem-match=tty
```

Consulte a [referência da CLI](../CLI.md) para obter o conjunto completo de comandos.

### PTP sempre ativo para hosts sem monitor

O pacote instala dois serviços do systemd para o backend, nenhum deles ativado. Em um Jetson sem monitor ou servidor que deva manter a sincronização de tempo PTP em execução contínua para sensores DAQ-E e câmeras LATTICE, ou onde scripts do SDK conectam projetos salvos (o `connect_all()` nunca inicia um back-end), habilite o serviço **como seu usuário**:

```bash
sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
systemctl status chloros-backend@$USER.service
```

O `chloros-backend@<user>.service` executa o back-end como esse usuário: as capturas que ele grava pertencem a ele, seu log detalhado está no `~/.cache/chloros/logs` dele e ele usa o `chloros-cli login` (`~/.chloros`), portanto, faça login uma vez como esse usuário primeiro. A instalação do pacote exibe este comando com seu nome de usuário preenchido.

Sem um serviço, o backend (e, com ele, o grandmaster PTP) só é executado enquanto algo o mantiver em execução — um comando `chloros-cli` ou um script do SDK.

A unidade vincula o backend às configurações de ambiente `127.0.0.1:5000` (configurações de ambiente `CHLOROS_HOST` / `CHLOROS_PORT` dentro da unidade; substitua por `sudo systemctl edit chloros-backend@.service`) e o reinicia em caso de falha após 5 segundos.

**O serviço root.**O `chloros-backend.service` executa o backend como**root**. Ele continua instalado para que uma máquina que o tenha habilitado continue funcionando, mas tudo o que um backend como root grava pertence ao root, de modo que um usuário não pode excluir suas próprias capturas; seu log está em `/var/log/chloros`, legível pelo grupo `adm`. Para mudar para o seu usuário:

```bash
sudo systemctl disable --now chloros-backend.service
sudo systemctl enable --now chloros-backend@$USER.service
```

**O backend de um serviço é compartilhado e nunca é interrompido por um cliente.** Os scripts `chloros-cli` e SDK o utilizam conforme o encontram: o `process` é executado nele, carregando seu próprio projeto no lugar do último. Onde a CLI, em outras circunstâncias, reiniciaria o backend (`--restart`, ou um `process` cuja importação não encontrou nada), ela identifica a unidade, exibe `sudo systemctl restart <unit>` e encerra o `1`, pois um backend parado dessa forma permaneceria inativo. Reinicie-o manualmente com o comando `sudo systemctl restart chloros-backend@$USER.service`. O backend de um serviço nunca se interrompe sozinho quando está ocioso, e o comando `chloros-cli backend stop` recusa isso, indicando o comando `systemctl stop`.**Como o PTP obtém suas portas.** O PTP usa as portas UDP 319/320, ambas abaixo do limite normal de 1024 para portas privilegiadas. O comando `postinst` do pacote grava `/etc/sysctl.d/60-chloros-ptp.conf` com `net.ipv4.ip_unprivileged_port_start = 319`, o que permite que o backend as vincule enquanto é executado como seu usuário. A partir da versão 1.3.0, o pacote não define nenhuma capacidade de arquivo (`setcap`) no backend, e `libcap2-bin` não é mais uma dependência: no backend descompactado, uma capacidade de arquivo impediria sua inicialização, portanto, não adicione uma por conta própria.***

## Exemplos de scripts em Bash

{% hint style="info" %}
**Códigos de saída compatíveis com scripts.**`chloros-cli process` encerra o `0` em caso de sucesso e**retorna um valor diferente de zero em caso de falha — incluindo uma execução que solicitou produtos de imagem, mas não gerou nenhum** (ele exibe `Processing finished but wrote no image products.` e indica o nome da pasta do projeto e as causas habituais). Execuções bem-sucedidas informam quantos produtos de imagem foram gravados (`Image products written: N`, além de `(plus M clamp masks)` quando máscaras foram gravadas). Códigos de saída: `0` sucesso, `1` falha, `2` erro de argumento, `130` interrompido.
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
    --indices NDVI GNDVI OSAVI \
    --debayer texture-aware \
    --no-vignette
```

Os valores válidos para `--format` são exatamente quatro e contêm espaços — sempre coloque-os entre aspas:

| Valor `--format` | Pasta de saída |
| --- | --- |
| `TIFF (16-bit)` *(padrão)* | `tiff16` |
| `TIFF (8-bit)` | `tiff8` |
| `TIFF (32-bit, Percent)` | `tiff32` |
| `PNG (8-bit)` | `png8` |
| `JPG (8-bit)` | `jpg8` |

`--debayer` aceita `standard` (padrão) ou `texture-aware` (Chloros+).

### Processamento automatizado com Cron

```cron
# Process any new datasets at 2 AM daily
0 2 * ** /usr/bin/chloros-cli process /data/incoming --output /data/processed >> /var/log/chloros.log 2>&1
```

### Exemplo de SDK do Python

```python
from chloros_sdk import process_folder

# One-line processing
result = process_folder(
    "/home/user/datasets/flight001",
    indices=["NDVI", "OSAVI"],
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
sudo chmod +x /usr/lib/chloros/chloros-backend /usr/lib/chloros/backend/chloros-backend
```

### O backend não inicia após a adição de capacidades de arquivo

Não execute o `setcap` no backend. A partir da versão 1.3.0, o pacote não define recursos de arquivo, e as portas 319/320 do PTP provêm da configuração `sysctl` descrita em [Como o PTP obtém suas portas](#always-on-ptp-for-headless-hosts). Uma capacidade de arquivo coloca o backend descompactado em um modo de execução segura, no qual ele não consegue localizar suas próprias bibliotecas, portanto não inicia. Se você adicionou uma, remova-a:

```bash
sudo setcap -r /usr/lib/chloros/backend/chloros-backend
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

### Os comandos `lattice` falham com a mensagem “Drivers da câmera LATTICE não encontrados”

O backend não conseguiu carregar o runtime da câmera LATTICE com o qual está instalado. Todos os comandos `lattice` são executados no backend; a própria CLI não contém drivers de câmera. Reinstale o Chloros (`sudo apt install --reinstall ./chloros_1.3.0_<arch>.deb`): o `.deb` instala o ambiente de execução junto ao backend, no `/usr/lib/chloros/arena_runtime`, e o registra no carregador dinâmico. Para verificar esse registro:

```bash
cat /etc/ld.so.conf.d/Arena_SDK.conf     # expect /usr/lib/chloros/arena_runtime
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

Os logs do backend relativos à falha na inicialização estão em `~/.cache/chloros/logs/`. Se a porta 5000 pertencer a um serviço de backend (`systemctl status chloros-backend@$USER.service`), reinicie o serviço em vez de encerrar seu processo.

### CUDA não detectada

```bash
# Check NVIDIA driver installation (x86_64)
nvidia-smi

# Check CUDA availability
nvcc --version

# On Jetson, check the L4T release (JetPack 6 is R36) and that JetPack's CUDA libraries are installed
cat /etc/nv_tegra_release
sudo apt install nvidia-jetpack
```

O `chloros-cli selftest` relata a mesma informação em uma única linha: `GPU: <name>, CUDA: <bool>, PyTorch: <version>`.

### Bibliotecas compartilhadas ausentes

```bash
sudo apt-get update
sudo apt-get install -f

# Check for missing libraries (the program itself, not the launcher script)
ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"
```

### Inicialização lenta em sistemas com cartão SD

A partir da versão 1.3.0, o backend e a CLI são instalados descompactados, de modo que não precisam mais se extrair para um diretório temporário a cada inicialização. Os arquivos temporários são direcionados automaticamente para o `/mnt/ssd/tmp` quando essa pasta existe; caso contrário, você pode direcioná-los para um sistema de arquivos rápido:

```bash
export TMPDIR=/mnt/nvme/tmp
```

***

## Atualizando o Chloros no Linux

O comando `update` é exclusivo para Linux/Jetson. Ele verifica a versão publicada no canal de atualização configurado em `/etc/chloros/update.conf` e oferece o download e a instalação do `.deb` correspondente:

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

No Linux/Jetson, a CLI também executa uma verificação de atualização não bloqueante a cada inicialização (resultado armazenado em cache por uma hora em `~/.chloros/update_cache.json`) e exibe `Update available: vX.Y.Z` quando existe uma versão mais recente. Suas configurações, projetos e dados de login são preservados após a atualização.

A atualização do pacote interrompe todos os serviços de back-end durante a instalação; assim, um script em execução perde suas câmeras e sensores e, em seguida, reinicia cada um dos que estiverem habilitados (um script iniciado manualmente, mas nunca habilitado, permanece parado, como após uma reinicialização).

## Desinstalação

```bash
sudo apt remove chloros
```

A remoção interrompe e desativa todos os serviços de back-end, restaura o limite padrão de portas sem privilégios (1024), remove o arquivo de configuração do buffer de recepção (os valores em execução permanecem até a reinicialização), o link simbólico do exiftool incluído, a configuração do carregador Arena, as regras de firewall adicionadas pelo pacote e o PythonSDK instalado, além de limpar as credenciais armazenadas em cache. Seus projetos e arquivos de dados do `~/.chloros/` não são afetados. O `sudo apt purge chloros` também exclui o `/var/log/chloros`.

***

## Próximos passos

* [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) — Otimização e implantação específicas para o Jetson
* [CLI: Linha de comando](../CLI.md) — o guia da CLI
* [API: Python SDK](../api-python-sdk.md) — o guia do SDK
* [Referência da CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas completas de comandos/API para a versão 1.3.0
* [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware
