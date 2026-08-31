# CLI Guia de Início Rápido (pool-*)

As unidades `chloros-cli` fornecidas controlam sensores DAQ por meio da **família de comandos `daq pool-*`** — clientes HTTP “leves” que operam o sensor por meio do pool de sensores persistente do backend Chloros. O backend é responsável pelo transporte, de modo que a GUI, o CLI e os scripts SDK compartilham um único identificador ativo, em vez de disputarem a porta. Tudo o que um cliente precisa está acessível por meio do `pool-*`: conectar, transmitir, gravar arquivos `.daq` calibrados e trocar perfis de cap.

O `pool-*` também é a **única** superfície DAQ nas versões lançadas. O `chloros-cli daq --help` lista os subcomandos do `pool-*`, e a invocação de um subcomando DAQ de hardware direto em uma versão lançada encerra a execução com um erro explícito indicando o pacote ausente e redirecionando você de volta para o `pool-*` — nada falha silenciosamente. (Os comandos de hardware direto são executados apenas a partir de um checkout do código-fonte do MAPIR; o `pip install chloros-sdk` também não os fornece.)

***

## Pré-requisitos

* **O backend Chloros deve estar em execução** — os comandos do `pool-*` são clientes do HTTP, não drivers de hardware. No Windows, inicie o aplicativo de desktop Chloros (ele inicia o backend). No Linux/Jetson sem interface gráfica, habilite o serviço: `sudo systemctl enable --now chloros-backend.service`.
* **Um login no Chloros+ (plano pago)**: execute primeiro o `chloros-cli login`. A verificação de acesso ocorre no lado do servidor — sem login, os comandos falham com o `401 AUTH_REQUIRED`; no plano gratuito (Iron), eles falham com o `403 PLAN_UPGRADE_REQUIRED`.
* Os comandos têm como alvo o `http://127.0.0.1:5000` por padrão; a família `daq pool-*` respeita a variável de ambiente `CHLOROS_BACKEND_URL` se o seu backend for executado em outro lugar.

***

## Uma sessão de cinco minutos

```bash
# 1. Connect a sensor into the backend pool (pick the line matching your model)
chloros-cli daq pool-connect                                  # smart-detect any DAQ
chloros-cli daq pool-connect --port COM3                      # DAQ-U on a specific COM port
chloros-cli daq pool-connect --mac AA:BB:CC:DD:EE:FF          # DAQ-M by BLE MAC
chloros-cli daq pool-connect --eth-host daq-e-def330.local    # DAQ-E by hostname (reliable)

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

## `pool-connect` — abre um sensor no pool

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

{% hint style="warning" %}
**Advertência sobre a descoberta automática do `--eth`.** Em um host com múltiplas interfaces de rede (mais de uma interface de rede ativa), o *primeiro* `pool-connect --eth` após a inicialização pode retornar vazio mesmo com o sensor em bom estado — a varredura de descoberta pode não detectar a interface do sensor enquanto o cache ARP estiver inativo. Se o `--eth` não encontrar nada, tente novamente ou pule a descoberta completamente com o `--eth-host <ip-or-hostname>`, que é o caminho confiável em máquinas com múltiplas interfaces de rede. O nome do host do DAQ-E é `daq-e-<id>.local` (por exemplo, `daq-e-def330.local`); seu endereço IP simples também funciona.
{% endhint %}

## `pool-list` — veja o que está conectado

Mostra todos os sensores no pool de back-end, incluindo o `sensor_id` necessário para todos os outros comandos:

| Modelo | Formato do `sensor_id` | Exemplo |
| --- | --- | --- |
| DAQ-U / DAQ-M | 5 octetos com hífen | `CB-7C-A8-2E-5F` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

## `pool-latest` — ler quadros de espectro

```bash
chloros-cli daq pool-latest --sensor-id daq-e-def330 --recent 10 --json
```

Retorna o quadro mais recente ou os quadros mais recentes; o `--recent N` emite uma saída legível por máquina para uso em scripts. Os quadros representam a irradiância espectral (W/m²/nm) calibrada radiometricamente em uma grade de 135 pontos, na faixa de 340 a 1.010 nm, com o perfil da tampa do sensor já aplicado. Para obter valores quantitativos de irradiância, calcule a média de pelo menos 15 segundos de quadros — trata-se de uma característica do instrumento, não de um defeito.

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
| `--output DIR` / `-o DIR` | `~/Documents/DAQ Live View/` | Diretório de saída, determinado **na máquina que executa o backend**. |
| `--device-name NAME` | — | Rótulo armazenado com a gravação. |
| `--stop` | — | Interrompe uma gravação em andamento. |

{% hint style="info" %}
A gravação ocorre no backend, portanto, o arquivo `.daq` é salvo no sistema de arquivos da **máquina do backend** — por padrão, no `~/Documents/DAQ Live View/`, e não necessariamente onde você executou o CLI. Os nomes dos arquivos incluem o ID do sensor e um carimbo de data/hora.
{% endhint %}

## `pool-set-cap` — declare a tampa instalada

```bash
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id sunshine_cosine
```

O ID da tampa seleciona o perfil de correção medido na fábrica aplicado a cada espectro e **deve corresponder à tampa fisicamente instalada no sensor** — nem o sensor nem o software podem detectar a tampa por conta própria, e a seleção é registrada em cada arquivo `.daq`. O padrão em todos os casos é `sunshine_cosine` (todos os DAQs vêm com a tampa corretora de cosseno Sunshine instalada, com atenuação de ~12× por padrão — uma troca não declarada da tampa corrige incorretamente os espectros por aproximadamente esse fator).

| `--cap-id` | Disponível em |
| --- | --- |
| `sunshine_cosine` (padrão) | DAQ-U, DAQ-M, DAQ-E |
| `fov_15`, `fov_45`, `fov_90` | DAQ-U, DAQ-E |
| `fov_30`, `fov_60` | Somente DAQ-U |
| `none` | Somente DAQ-E — veja a nota |

Um ID de tampa fora do conjunto do sensor é rejeitado na conexão, gerando um erro explícito. `none` (DAQ-E) significa que a tampa foi removida fisicamente — ele ainda aplica um perfil geométrico de fábrica para o difusor de vidro recuado do DAQ-E; portanto, não é uma operação nula, e um DAQ-E sem tampa é uma configuração de bancada, não uma configuração de campo suportada. (Um DAQ-U sem tampa é verdadeiramente “nu” e não precisa de nenhum perfil de correção; o DAQ-M é usado com sua tampa Sunshine.)

## `pool-disconnect` — liberar sensores

```bash
chloros-cli daq pool-disconnect --sensor-id daq-e-def330   # one sensor
chloros-cli daq pool-disconnect --all                      # everything in the pool
```

***

## Resumo dos comandos

| Comando | Finalidade |
| --- | --- |
| `daq pool-connect [--port P \| --ble \| --mac M \| --eth \| --eth-host H] [-t MS] [-f N] [--no-ae] [--no-stream] [--cap-id CAP]` | Abrir um sensor no pool de back-end. |
| `daq pool-list` | Exibir todos os sensores do pool com seu `sensor_id`. |
| `daq pool-latest --sensor-id ID [--recent N] [--json]` | N quadros de espectro calibrados mais recentes. |
| `daq pool-stream --sensor-id ID [--start \| --stop]` | Retomar/pausar a transmissão. |
| `daq pool-record --sensor-id ID [-d SEC] [-o DIR] [--device-name NAME] [--stop]` | Iniciar/parar uma gravação `.daq` (no back-end). |
| `daq pool-set-cap --sensor-id ID --cap-id CAP` | Trocar o perfil de correção de limite durante a execução. |
| `daq pool-disconnect --sensor-id ID [--all]` | Liberar um sensor ou todos eles. |

***

## Solução de problemas na primeira conexão do DAQ-E

1. O DAQ-E não possui LED de status — verifique se há alimentação por meio do indicador de PoE/link no switch ou na porta do injetor e aguarde alguns segundos após a ligação para que ele inicialize e se conecte à rede.
2. A máquina backend deve estar no **mesmo domínio de broadcast** que o sensor — o mDNS não atravessa roteadores.
3. No Windows, aceite a solicitação do firewall Defender na primeira execução (mDNS UDP 5353, dados do DAQ-E UDP 5002, PTP UDP 319/320).
4. Ainda não há resposta do `--eth`? Use o `--eth-host` com o nome de host da unidade (`daq-e-<id>.local`) ou o endereço IP — esse é o caminho mais confiável, especialmente em hosts com múltiplos endereços IP.

***{% hint style="info" %}**Dica para assistentes de IA.** Todas as páginas deste manual são servidas como Markdown bruto — acrescente `.md` ao slug em letras minúsculas de uma página, URL (esta página: `https://mapir.gitbook.io/chloros/daq/cli-quick-start.md`); o índice legível por máquina é `https://mapir.gitbook.io/chloros/llms.txt`. Para obter a documentação completa sobre os parâmetros de `chloros-cli daq` e todas as outras famílias de comandos, acesse a [Referência do CLI](../reference/cli-reference.md) (`https://mapir.gitbook.io/chloros/reference/cli-reference.md`); o caminho do Python é `chloros_sdk.connect_daq_sensor()` na [Referência do SDK](../reference/sdk-reference.md).
{% endhint %}
