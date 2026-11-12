# Guia de Início Rápido do CLI (pool-*)

As unidades `chloros-cli` fornecidas controlam os sensores DAQ por meio da família de comandos **`daq pool-*`** — clientes HTTP “leves” que operam o sensor por meio do pool de sensores persistente do backend do Chloros. O backend é responsável pelo transporte, de modo que a GUI, a CLI e os scripts do SDK compartilham um único identificador ativo, em vez de disputarem a porta. Tudo o que o cliente precisa está acessível por meio do `pool-*`: conectar, transmitir, gravar arquivos `.daq` calibrados e trocar perfis de capacitância.

O `pool-*` é a interface DAQ suportada nas versões lançadas. O `chloros-cli daq --help` lista os subcomandos do `pool-*`, e o `daq discover` / `daq list` executam a mesma varredura que o `pool-discover`. Quando um subcomando de DAQ direto de hardware não estiver disponível, o programa será encerrado com um erro explícito indicando o pacote ausente e direcionando você de volta ao `pool-*` — nada falha silenciosamente. (O `pip install chloros-sdk` também não fornece os comandos diretos de hardware.)

***

## Pré-requisitos

* **O backend do Chloros deve estar em execução** — os comandos `pool-*` são clientes HTTP, não drivers de hardware. No Windows, inicie o aplicativo de desktop do Chloros (ele inicia o backend). Em um Linux/Jetson sem interface gráfica, habilite o serviço como seu usuário: `sudo systemctl enable --now chloros-backend@$USER.service` (consulte [Instalação no Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts)).
* **Login no Chloros+ (plano pago)**: execute primeiro o comando `chloros-cli login`. A verificação de acesso ocorre no lado do servidor — sem um login, os comandos falham com o `401 AUTH_REQUIRED`; no plano gratuito (Iron), eles falham com o `403 PLAN_UPGRADE_REQUIRED`.
* Os comandos têm como alvo o `http://127.0.0.1:5000` por padrão; a família `daq pool-*` respeita a variável de ambiente `CHLOROS_BACKEND_URL` caso seu backend seja executado em outro local.

***

## Uma sessão de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)
#    (a DAQ-M's BLE MAC is not printed anywhere: find it with `chloros-cli daq pool-discover --only ble`)

# 2. List the pool — this shows the sensor_id used by every command below
chloros-cli daq pool-list

# 3. Read the most recent calibrated spectrum frame (add --json for scripting)
chloros-cli daq pool-latest --sensor-id daq-e-def330 --json

# 4. Record a calibrated .daq file for 60 seconds
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 60 \
  --device-name "field-A"

# 5. Release the sensor when done
chloros-cli daq pool-disconnect --sensor-id daq-e-def330
```

***

## `pool-connect` — abrir um sensor no pool

| Variante | Significado |
| --- | --- |
| `daq pool-connect` | Detecção inteligente: localiza qualquer DAQ nesta máquina. |
| `daq pool-connect --port PORT` | DAQ-U em uma porta serial específica (por exemplo, `COM3`, `/dev/ttyUSB0`). |
| `daq pool-connect --ble` | DAQ-M via BLE, MAC detectado automaticamente. |
| `daq pool-connect --mac MAC` | DAQ-M em um MAC BLE conhecido (implica `--ble`). |
| `daq pool-connect --eth-host HOST` | DAQ-E com nome de host ou IP conhecido — **o caminho confiável**. |
| `daq pool-connect --eth` | DAQ-E com descoberta automática (mDNS, com fallback para ARP). Veja a observação abaixo. |

Sinalizadores de ajuste, todos opcionais:

| Sinalizador | Significado |
| --- | --- |
| `--integration-time MS` / `-t MS` | Tempo de integração manual em milissegundos. |
| `--frame-avg N` / `-f N` | Média de quadros por espectro relatado. |
| `--no-ae` | Desativar exposição automática (AE está ativada por padrão). |
| `--no-stream` | Conectar sem iniciar a transmissão (retomar mais tarde com `pool-stream --start`). |
| `--cap-id CAP` | Perfil de correção de cap; o padrão do backend é `sunshine_cosine`. Consulte [`pool-set-cap`](#pool-set-cap-declare-the-fitted-cap). |

Conectar um DAQ-E que já esteja no pool sob outro de seus endereços — seu nome de host `daq-e-<id>.local` ou seu IP — retorna a entrada existente no pool em vez de abrir a unidade pela segunda vez.

{% hint style="warning" %}
**Observação sobre a descoberta automática do `--eth`.** Em um host com múltiplas interfaces de rede (mais de uma interface de rede ativa), o *primeiro* `pool-connect --eth` após a inicialização pode não encontrar nada, mesmo com o sensor em bom estado — a varredura de descoberta pode não detectar a interface do sensor enquanto o cache ARP estiver vazio. Se o `--eth` não encontrar nada, tente novamente ou pule a descoberta completamente com o `--eth-host <ip-or-hostname>`, que é o caminho confiável em máquinas com múltiplas interfaces de rede. O nome do host do DAQ-E é `daq-e-<id>.local` (por exemplo, `daq-e-def330.local`); seu endereço IP simples também funciona.
{% endhint %}

## `pool-discover` — varredura sem conexão

```bash
chloros-cli daq pool-discover                  # USB, BLE and Ethernet
chloros-cli daq pool-discover --only ble       # BLE only: this is how you find a DAQ-M's MAC
chloros-cli daq pool-discover --json           # machine-readable
```

Vara em busca de sensores aos quais você poderia se conectar e mostra o endereço para conexão, sem estabelecer a conexão. `--only` aceita um subconjunto separado por vírgulas de `usb,ble,eth`; `--timeout SEC` define o tempo de varredura por transporte (padrão: 5 s, limitado a 1–20 s). Os sensores já abertos no pool não são listados (um DAQ-M conectado deixa de anunciar) — use o `pool-list` para esses casos. Em uma versão comercializada, `chloros-cli daq discover` e `daq list` executam essa mesma varredura.

## `pool-list` — veja o que está conectado

Mostra todos os sensores no pool de back-end, incluindo o `sensor_id` necessário para todos os outros comandos:

| Modelo | Formato do `sensor_id` | Exemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos com hífen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

Um DAQ-E cujo PTP foi desativado (a configuração é salva na unidade) é sinalizado em `pool-list` com `!! PTP is switched OFF on this unit (saved in its flash): its timestamps are not PTP-synced.` e o comando que o reativa — consulte [`pool-enable-ptp`](#pool-enable-ptp-switch-a-daq-es-ptp-back-on).

## `pool-latest` — ler quadros de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Retorna o quadro mais recente ou os quadros mais recentes `--recent N`; o `--json` emite uma saída legível por máquina para uso em scripts. Os quadros representam a irradiância espectral (W/m²/nm) calibrada radiometricamente em uma grade de 135 pontos, na faixa de 340 a 1010 nm, com o perfil da tampa do sensor já aplicado. Para obter valores quantitativos de irradiância, calcule a média de pelo menos 15 segundos de quadros — trata-se de uma característica do instrumento, não de um defeito.

## `pool-stream` — pausar ou retomar a transmissão

```bash
chloros-cli daq pool-stream --sensor-id daq-e-def330 --stop    # pause
chloros-cli daq pool-stream --sensor-id daq-e-def330 --start   # resume
```

## `pool-record` — gravar um arquivo `.daq`

```bash
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
  --output ~/Documents/spectra --device-name "rooftop-A"
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

| Sinalizador | Padrão | Significado |
| --- | --- | --- |
| `--duration SEC` / `-d SEC` | `0` | Duração da gravação em segundos; `0` significa executar até que você emita o comando `--stop`. |
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Diretório de saída. Com o backend neste computador, um caminho relativo é resolvido em relação à pasta na qual você executou o comando; com um backend remoto, o caminho é resolvido **na máquina que está executando o backend**. |
| `--device-name NAME` | — | Rótulo armazenado com a gravação. |
| `--stop` | — | Interromper uma gravação em andamento. |
| `--confirm-after SEC` | `20` | Após o início, aguarde até esse tempo para que a primeira linha seja gravada e emita um aviso sonoro (`!! WARNING: no rows written after …s. This recording is producing an EMPTY file.`) se nenhuma chegar. `0` desativa a verificação. |
| `--heartbeat SEC` | `30` | Com `--duration`, imprima a contagem de linhas a cada SEC segundos e avise se ela parar de aumentar. `0` desativa essa verificação. |

{% hint style="info" %}
O registro ocorre no backend, portanto, o arquivo `.daq` é salvo no sistema de arquivos da **máquina de backend** — por padrão, em `~/Documents/DAQ Live View/`, não necessariamente onde você executou a CLI. Os nomes dos arquivos incluem o ID do sensor e um carimbo de data/hora.
{% endhint %}

## `pool-set-cap` — declare a tampa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

O ID da tampa seleciona o perfil de correção medido na fábrica aplicado a cada espectro e **deve corresponder à tampa fisicamente instalada no sensor** — nem o sensor nem o software podem detectar a tampa por conta própria, e a seleção é registrada em cada arquivo `.daq`. O padrão em todos os casos é `sunshine_cosine` (todos os DAQs são fornecidos com a tampa corretora de cosseno Sunshine instalada, com atenuação de aproximadamente 12× por padrão — uma troca não declarada da tampa corrige incorretamente os espectros por aproximadamente esse fator).

| `--cap-id` | Disponível em |
| --- | --- |
| `sunshine_cosine` (padrão) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Somente DAQ-U |
| `none` | Somente DAQ-E — veja a nota |

Um ID de tampa fora do conjunto do sensor é rejeitado na conexão com um erro explícito. `none` (DAQ-E) significa que a tampa foi removida fisicamente — ele ainda aplica um perfil geométrico de fábrica para o difusor de vidro recuado do DAQ-E; portanto, não é uma operação nula, e um DAQ-E sem tampa é uma configuração de bancada, não uma configuração de campo suportada. (Um DAQ-U sem tampa é totalmente sem tampa e não precisa de nenhum perfil de correção; o DAQ-M é usado com sua tampa Sunshine.)

## `pool-disconnect` — liberar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

`pool-disconnect` libera um sensor independentemente de quem o tenha aberto; portanto, também libera um sensor que um script com falha tenha deixado no pool (desde que a gravação nele seja concluída primeiro). Um sensor que não esteja no pool não tem nada para liberar: o comando exibe `<sensor-id>: already released (not in the pool)` e encerra com `0`. Uma liberação recusada ou não confirmada pelo backend encerra com `1`.

## `pool-enable-ptp` — reativar o PTP de um DAQ-E

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
```

Um DAQ-E mantém sua configuração de ativação/desativação do PTP em sua própria memória; portanto, uma unidade cujo PTP foi desativado permanece desativada — e seus carimbos de data/hora não são sincronizados via PTP — até que seja reativada. O comando `pool-list` sinaliza essa unidade e exibe este comando para ela. O Chloros nunca reativa uma unidade por conta própria, pois a desativação pode ser intencional. Novidade na versão 1.3.0.

## Configurações do dispositivo DAQ-E

```bash
chloros-cli daq pool-info --sensor-id daq-e-def330
chloros-cli daq pool-set-network --sensor-id daq-e-def330 \
  --static 192.168.1.50 --gateway 192.168.1.1
chloros-cli daq pool-set-network --sensor-id daq-e-def330 --auto
chloros-cli daq pool-set-auto-stream --sensor-id daq-e-def330 --on
chloros-cli daq pool-reboot --sensor-id daq-e-def330
chloros-cli daq pool-update-firmware --sensor-id daq-e-def330
```

`pool-info` exibe o firmware do sensor (e se esta versão do Chloros traz uma versão mais recente), seu endereço e como ele o obteve, a configuração de rede salva caso difira da atual, streaming na inicialização e PTP. `pool-set-network` reinicia o sensor para aplicar a alteração e o reconecta com as configurações anteriores; `--no-reboot` apenas salva as configurações. Um endereço estático deve estar na rede deste computador; caso contrário, o Chloros o rejeita; o `--force` ignora essa restrição, mas apenas para um computador que será reendereçado ou roteado para ele. As reinicializações são recusadas enquanto o sensor estiver gravando. `pool-update-firmware` é o equivalente à opção **Atualizar para \<version\>**</version\> da interface gráfica (GUI) <version\>para a linha de comando e a maneira de atualizar um DAQ-E no Linux, que não possui interface gráfica. Novidade na versão 1.3.0.

***

## Resumo dos comandos

| Comando | Finalidade |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir um sensor no pool de back-end. |
| `daq pool-discover [--only usb,ble,eth] [--timeout SEC] [--json]` | Procurar sensores sem se conectar (localiza o MAC BLE de um DAQ-M). |
| `daq pool-list` | Exibe todos os sensores no pool com seu `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | Os N quadros de espectro calibrados mais recentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--confirm-after SEC] [--heartbeat SEC] [--stop]` | Iniciar/parar uma gravação `.daq` (no backend). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de limite durante a execução. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar um sensor ou todos eles. |
| `daq pool-enable-ptp --sensor-id ID` | Reativar o PTP para um DAQ-E cujo PTP esteja desativado. |
| `daq pool-info --sensor-id ID [--json]` | Firmware, endereço, configuração de rede salva, transmissão ao ligar e PTP de um DAQ-E. |
| `daq pool-set-network --sensor-id ID (--auto \| --static IP [--mask M] [--gateway G]) [--no-reboot] [--force]` | DHCP ou endereço estático; reinicia e reconecta o sensor. |
| `daq pool-set-auto-stream --sensor-id ID (--on \| --off)` | Transmissão ao ligar. |
| `daq pool-reboot --sensor-id ID` | Reinicia um DAQ-E e o reconecta. |
| `daq pool-update-firmware --sensor-id ID` | Atualiza o firmware do DAQ-E desta versão em um sensor mais antigo. |

***

## Solução de problemas na primeira conexão do DAQ-E

1. O DAQ-E não possui LED de status — verifique se há alimentação por meio do indicador PoE/link no switch ou na porta do injetor e aguarde alguns segundos após a inicialização para que ele inicialize e se conecte à rede.
2. A máquina de back-end deve estar no **mesmo domínio de broadcast** que o sensor — o mDNS não atravessa roteadores.
3. Um firewall deve permitir a passagem de mDNS (UDP 5353), dados do DAQ-E (UDP 5002) e PTP (UDP 319/320). O instalador do Windows adiciona uma regra para isso, e o `.deb` do Linux abre essas portas em um `ufw` / `firewalld` que estava ativo no momento da instalação; um firewall de terceiros, ou um ativado posteriormente, precisa permitir o acesso a elas.
4. Ainda nada do `--eth`? Use o `--eth-host` com o nome do host da unidade (`daq-e-<id>.local`) ou o IP, o que ignora a descoberta.

***
{% hint style="info" %}
**Dica para assistentes de IA.** Todas as páginas deste manual são fornecidas como Markdown bruto — acrescente `.md` ao slug da URL em letras minúsculas de uma página (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); o índice legível por máquina é `https://mapir.gitbook.io/chloros/llms.txt`. Para obter a documentação completa sobre os parâmetros de `chloros-cli daq` e todas as outras famílias de comandos, consulte a [Referência da CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); o caminho do Python é `chloros_sdk.connect_daq_sensor()` na [Referência do SDK](../reference/sdk-reference.md).
{% endhint %}
</version\>
