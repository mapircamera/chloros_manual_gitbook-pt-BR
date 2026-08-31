# Sensores de luz DAQ

> **Procurando o hardware?**Os próprios sensores — modelos, montagem, tampas, portas, alimentação e o aplicativo SCANNER — estão documentados no**[manual do usuário do DAQ](https://mapir.gitbook.io/daq)**. Este capítulo aborda como utilizá-los a partir do Chloros.

Os sensores de luz **DAQ** do MAPIR medem a luz ambiente como espectros calibrados radiometricamente. No Chloros, eles desempenham duas funções:

* **Um instrumento espectral autônomo** — gráficos de espectro em tempo real, dados colorimétricos e gravações do `.daq`, tudo a partir da [guia Sensores de Luz](gui.md), o [CLI](cli-quick-start.md) ou o Python SDK.
* **Uma fonte de irradiância descendente para refletância** — durante o processamento, o Chloros interpola suas leituras do `.daq` para o carimbo de data e hora de exposição de cada capturae utiliza a luz descendente medida para converter a radiancia da câmera em refletância (`--reflectance-source daq`), sem necessidade de painel na cena para bandas calibradas.

<!-- SCREENSHOT-NEEDED: product photo of the DAQ-U, DAQ-M, and DAQ-E units side by side, each with its Sunshine cosine-corrector cap fitted (request from hardware team — no repo asset exists) -->

***

## Três modelos, um formato de dados

| Modelo | Transporte | Descoberta |
| --- | --- | --- |
| **DAQ-U** | USB (serial) | varredura pela porta serial |
| **DAQ-M** | Bluetooth Low Energy | varredura BLE por nome |
| **DAQ-E** | Ethernet (IPv4, alimentação por PoE) | mDNS `_daq-e._tcp` (nome do host `daq-e-<id>.local`) |

Todos os três utilizam o mesmo protocolo de comunicação e fornecem dados idênticos:

* Um **espectro de 135 pontos de 340 a 1010 nm em intervalos de 5 nm**, além dos valores tristimulares CIE XYZ, em cada quadro.
* **Irradiação espectral calibrada radiometricamente em W/m²/nm** — o pacote de calibração de fábrica de cada unidade (além de seu perfil de correção de limite ativo) é aplicado antes que os dados cheguem até você.
* O mesmo **formato de gravação `.daq`** (um arquivo SQLite). O processamento posterior é idêntico, independentemente do meio de transporte que tenha gerado o arquivo.

As pilhas de transporte (serial USB, BLE, mDNS/zeroconf) estão integradas no backend Chloros — não há nada a ser instalado para se comunicar com qualquer um dos três modelos pela interface gráfica (GUI) ou pelos comandos `pool-*` do CLI.

***

## Faixa calibrada: 340–1010 nm relatada, ~374–974 nm calibrada

O sensor relata a grade completa de 340–1010 nm, mas o ganho radiométrico rastreável pelo NIST abrange aproximadamente **374–974 nm**. O comando Chloros recusa a divisão de refletância absoluta para qualquer banda da câmera com menos da metade de seu peso espectral dentro desse intervalo calibrado; a banda ignorada é relatada com o motivo de ignoração `dls-uncalibrated-band-<nm>`.

Entre os SKUs de filtros LATTICE disponíveis para envio, apenas o **F988** é afetado:

A refletância do F988 é calibrada usando um painel de refletância na cena: a banda está além da faixa calibrada do sensor de luz do DAQ; portanto, o Chloros aplica sua captura mais recente do painel e a mantém entre as observações do painel.

Se uma captura do F988 for processada com apenas dados do DAQ disponíveis, o Chloros rejeita a refletância baseada no DAQ para essa faixa com o motivo de omissão `dls-uncalibrated-band-988` — o [fluxo de trabalho do painel de refletância](../calibration-targets.md) é o caminho compatível para o F988.

***

## IDs de sensores

Cada DAQ reporta um ID de sensor estável. Seu formato varia de acordo com o modelo:

| Modelo | Formato do ID | Exemplo |
| --- | --- | --- |
| DAQ-U | 5 octetos com hifenização | `CB-7C-A8-2E-5F` |
| DAQ-M | 5 octetos com hífens | `CB-74-02-30-6B` |
| DAQ-E | `daq-e-<6 hex digits>` | `daq-e-def330` |

O ID do sensor é:

* gravado em cada arquivo `.daq` que ele registra,
* a chave que o Chloros usa para buscar o pacote de calibração de fábrica daquela unidade,
* o valor que você passa para o `--sensor-id` nos comandos CLI e `pool-*`, e
* para o DAQ-E, também seu nome de host mDNS (`daq-e-def330.local`) — o valor que o `--eth-host` aceita.

***

## Calibração de fábrica e a nuvem

Cada unidade DAQ é calibrada individualmente na fábrica com uma cadeia radiométrica rastreável pelo NIST, e o Chloros carrega o pacote de calibração de cada unidade, identificado pelo ID do sensor. O relatório de calibração por unidade (PDF) pode ser baixado nas configurações do sensor na [guia Sensores de luz](gui.md).

{% hint style="warning" %}
**O DAQ-U e o DAQ-M requerem acesso à nuvem para calibração.**Nenhum dos dois modelos armazena nada internamente: seus pacotes de calibração de fábrica ficam na nuvem do MAPIR e são buscados pelo ID do sensor (sendo então armazenados em cache localmente). O Chloros precisa de uma conexão com a internet para fornecer dados calibrados em W/m²/nm a partir de um DAQ-U ou DAQ-M.**O DAQ-E é a exceção** — ele mantém sua calibração no próprio dispositivo.

<!-- PRE-PUBLISH-CHECK: LAUNCH item 3 (DAQ-M end-to-end connect smoke) was still unverified as of 2026-08-16 — re-confirm the DAQ-M cloud-calibration flow on the release build before publishing this page. -->

{% endhint %}***

## Onde os registros são armazenados

| Superfície | Destino padrão do `.daq` |
| --- | --- |
| GUI — guia Sensores de luz | `<project folder>/light_sensor/` (os registros concluídos são adicionados ao projeto automaticamente) |
| CLI — `daq pool-record` | `~/Documents/DAQ Live View/` na máquina que executa o backend |

Cada nome de arquivo `.daq` inclui o ID do sensor e um carimbo de data/hora.

***

## Neste capítulo

* [**A guia DAQ no Chloros**](gui.md) — o passo a passo completo da interface gráfica: conexão de cada modelo, configurações por sensor, gráficos de espectro, dados colorimétricos em tempo real, refletância de sensor duplo e gravação.
* [**Início rápido do CLI (pool-\*)**](cli-quick-start.md) — como acionar sensores DAQ a partir do `chloros-cli daq pool-*`, o caminho de linha de comando compatível.
* [**Perfis de Limite e Faixa Calibrada**](caps-and-range.md) — quais limites existem por modelo, como declará-los e a faixa espectral calibrada em detalhes.
* [**Gravação e o formato .daq**](recording.md) — o formato SQLite do `.daq` e os fluxos de trabalho de gravação.
* [**Rede DAQ-E e Sincronização de Tempo**](ethernet-ptp.md) — modos de transporte DAQ-E e sincronização de tempo PTP.
* [**Fluxos de trabalho de refletância**](reflectance.md) — uso de dados de radiação descendente do DAQ para calcular a refletância.
* Para obter a documentação completa sobre os sinalizadores, consulte a [Referência do CLI](../reference/cli-reference.md) (seção `chloros-cli daq`) e a [Referência do SDK](../reference/sdk-reference.md) (`chloros_sdk.connect_daq_sensor()`), ambas redigidas para serem diretamente utilizadas por assistentes de IA.
