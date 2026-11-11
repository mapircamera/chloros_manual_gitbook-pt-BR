# Gravação e o formato .daq

Um arquivo `.daq` é o formato de gravação do sensor de luz dChloros

: um **banco de dados SQLite** contendo quadros espectrais calibrados de um sensor DAQ. Grave um arquivo durante uma sessão de captura e o pipeline de refletância poderá, posteriormente, dividir cada imagem pela irradiância descendente medida exatamente naquele momento.

## O que um arquivo .daq contém

| Propriedade | Valor |
| --- | --- |
| Contêiner | Banco de dados SQLite, um arquivo por sensor por gravação |
| Nome do arquivo | Inclui o **ID do sensor**e um**carimbo de data/hora**, por exemplo, `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Espectro por quadro | 135 pontos, 340–1010 nm em intervalos de 5 nm, além do tristímulo CIE XYZ |
| Unidades | Irradiação espectral calibrada, **W/m²/nm** (pacote de calibração de fábrica + perfil da tampa aplicado) |
| Metadados registrados | ID do sensor (a chave para obter a calibração de fábrica dessa unidade) e o perfil de tampa em vigor — consulte [Perfis de tampa e faixa calibrada](caps-and-range.md) |

O formato é idêntico nos modelos DAQ-U, DAQ-M, DAQ-E e DAQ-E-S, portanto, o processamento posterior não se importa com qual dispositivo de aquisição de dados (DAQ) fez o registro. Um registro do DAQ-E-S também armazena a inclinação, o rolamento e o ângulo de inclinação da unidade ao lado de cada espectro.

**As gravações sobrevivem a uma queda de energia (1.3.0).**Uma gravação em tempo real é gravada com um diário e uma sincronização completa do disco a cada confirmação; portanto, uma queda de energia ou um processo encerrado prejudicam apenas os últimos 3 segundos, e não o arquivo. Uma gravação interrompida no meio da gravação pode deixar um arquivo `<name>.daq-journal` ao lado do `.daq`:**mantenha e copie os dois juntos.** O Chloros reverte o diário na primeira vez que lê o arquivo. Um disco ocupado atrasa o arquivo, mas nunca as leituras do sensor.

**Um DAQ-E que para de transmitir é reconectado (1.3.0).** Quando o fluxo de um DAQ-E fica inativo (reinicialização do switch, perda de conexão), o backend o reconecta com as configurações anteriores, e a gravação nele continua no mesmo arquivo `.daq`, com a lacuna marcada no arquivo.

A gravação calibrada requer o pacote de calibração de fábrica do sensor. Para o DAQ-U e o DAQ-M, o backend busca o pacote na nuvem dMAPIR

por meio do ID do sensor (a gravação é recusada se isso não for possível); as unidades DAQ-E estão isentas, pois mantêm sua calibração no próprio dispositivo.

## Gravação pela GUI

A gravação na GUI requer um **projeto aberto** (caso contrário, os botões “Gravar” ficam desativados):

* **Gravar tudo / Parar tudo** — na parte superior da barra lateral dos Sensores de Luz; inicia ou interrompe uma gravação `.daq` em todos os sensores conectados de uma só vez.
* **Gravar / Parar Gravação** — por sensor, na janela modal de configurações (ícone de engrenagem). Um indicador vermelho “REC” aparece nas linhas de informações em tempo real do sensor durante a gravação.

Os arquivos são gravados no `<project>/light_sensor/` e, quando uma gravação é interrompida — seja por meio de “Parar”, “Parar tudo” ou pela desconexão de um sensor de gravação —, o `.daq` concluído é **adicionado automaticamente ao projeto aberto**. Ele aparece na lista de arquivos do projeto sem a necessidade de adição manual, já pronto para o processamento de refletância.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->



## Gravação pela CLI



A CLI grava por meio do conjunto de sensores do backend (o backend deve estar em execução — esses comandos são clientes HTTP leves):

```bash
# Connect the sensor into the backend pool
chloros-cli daq pool-connect --eth-host daq-e-def330.local

# Record for 150 seconds, with a human-friendly device label
chloros-cli daq pool-record --sensor-id daq-e-def330 --duration 150 \
    -o ./out --device-name "rooftop-A"

# Or run open-ended and stop explicitly
chloros-cli daq pool-record --sensor-id daq-e-def330            # --duration defaults to 0 = run until --stop
chloros-cli daq pool-record --sensor-id daq-e-def330 --stop
```

Obter o valor de `--sensor-id` a partir de `chloros-cli daq pool-list`. Duas configurações padrão que vale a pena conhecer:

| Opção | Padrão |
| --- | --- |
| `--duration` | `0` — gravar até `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` no sistema de arquivos do **backend**, não no da CLI |

A distinção entre diretórios de saída é importante quando a CLI tem como destino um backend em outra máquina: o arquivo é salvo onde o backend é executado. Com o backend no mesmo computador, um `-o` relativo (como o `./out` acima) é resolvido em relação à pasta em que você executou o comando.

## Gravação a partir do Python

`DAQSensorSession` (retornado por `chloros_sdk.connect_daq_sensor()`) expõe a mesma gravação agrupada: `record_start(output_dir=None, device_name=None)` retorna o caminho do arquivo, `record_stop()` retorna `{path, rows}`. Consulte a [Referência do SDK](../reference/sdk-reference.md) para obter a API completa da sessão. As classes de hardware direto do SDK (apenas para instalações em desktop) gravam as gravações em `~/Documents/DAQ/` por padrão; para versões lançadas, o caminho agrupado acima é a rota suportada.

## Usando um arquivo .daq no momento do processamento

Para calcular a refletância a partir de imagens, o Chloros precisa da irradiância descendente correspondente a cada exposição:

* **Mantenha o `.daq` junto com as imagens.**No momento do processamento, o pipeline determina automaticamente a**irradiação descendente com carimbo de data/hora correspondente** a partir de um `.daq` gravado (qualquer modelo de DAQ) — ou a partir de um `.csv` nativo do DAQ-M — encontrado junto com as imagens. As gravações da GUI atendem a esse requisito automaticamente, uma vez que são adicionadas ao projeto no momento em que são encerradas.
* **A calibração é obtida sob demanda.** Se um pacote de calibração de fábrica por câmera ou por DAQ ainda não estiver armazenado em cache localmente, o Chloros o busca automaticamente na nuvem dMAPIR

na primeira utilização (é necessária conexão com a internet uma vez; armazenado em cache como `~/.chloros/`).
* **As capturas ao vivo gravam seu próprio arquivo sidecar.** Para qualquer quadro de refletância capturado ao vivo, a leitura do DAQ efetivamente utilizada é salva como um arquivo `.daq` na pasta `daq/` das capturas, de modo que a captura possa ser reprocessada posteriormente sem a gravação original.
* **Gravações dos gravadores `chloros_scripts` (1.3.0).**Um arquivo `.daq` gerado pelos gravadores públicos [`chloros_scripts`](https://github.com/mapircamera/chloros_scripts) é listado como o modelo de DAQ que o produziu, e seu limite máximo pode ser definido em**Configurações do Projeto → Sensor de Luz DAQ**, como em qualquer outra gravação. Esses gravadores registram as contagens brutas do sensor; o Chloros obtém a calibração de fábrica do sensor via porta serial e a aplica.
* **De quem é a leitura (1.3.0).**Ao lado de cada captura `.daq`, um `.daq.cameras.json` identifica a(s) câmera(s) cujos quadros essa leitura descreve, e o processamento a associa**apenas às imagens dessas câmeras**. Uma câmera autônoma sem sensor de luz próprio, portanto, não utiliza mais a leitura de outra câmera — vincule um sensor a ela ou grave uma leitura. Uma gravação feita com o próprio DAQ (Gravação na GUI, `daq pool-record`) e qualquer `.daq` sem esse arquivo (incluindo capturas mais antigas) ainda atendem a todas as câmeras.

## Recuperando a irradiância

O processamento de um projeto também exporta todas as gravações de sensores de luz que ele contém para uma
pasta `Light Sensor/` ao lado dos produtos de imagem. Isso **não** requer imagens: um
sensor de luz registrado isoladamente já é uma captura completa, e uma pasta contendo apenas arquivos `.daq`
é uma entrada válida. Um projeto que contenha apenas gravações de sensores de luz as lista e
processa tanto pelo aplicativo quanto pela CLI, e elas permanecem listadas após a execução. A execução
informa quantos produtos de sensores de luz foram gravados.

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | Um arquivo que pode ser reprocessado no mesmo esquema de uma gravação ao vivo, agora declarando o pacote de calibração que o produziu. Reimportá-lo **não** o calibra uma segunda vez. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grade de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas: potência total, lux fotópico e escotópico, PPFD com sua divisão em azul/verde/vermelho e comprimento de onda de pico. |

Um DAQ-U ou DAQ-M cujo pacote de calibração não possa ser obtido — você está offline ou
esse sensor não possui calibração registrada — é **ignorada com um motivo**, nunca sendo gravada
como um arquivo “calibrado” contendo contagens brutas. Conecte-se à internet e execute novamente. Um DAQ-E
possui sua própria calibração, portanto, só precisa disso quando a unidade não está conectada e
nada está armazenado em cache localmente.

### DAQ-A: contagens brutas e por que essa é a resposta correta

O **DAQ-A** é anterior ao sistema de pacotes de calibração por porta serial e não possui nenhum pacote para
ser obtido. Isso não é um descuido: um DAQ-A é calibrado em campo em relação a um
alvo de refletância, e a calibração baseada em alvo precisa apenas da resposta *relativa*
do sensor — que é exatamente o que representam suas contagens brutas. A Chloros a calibra com elas até hoje.

Portanto, um registro do DAQ-A é exportado, mas com um nome diferente:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, e não `_calibrated` — um nome de arquivo diferente, em vez de um sinalizador dentro do arquivo,
porque a informação precisa permanecer intacta mesmo quando o arquivo for enviado por e-mail apenas com o nome. O cabeçalho `.csv`
indica `raw spectral sensor counts (NOT irradiance)` e alerta que os valores são
comparáveis **dentro** do arquivo e não entre sensores. As colunas que só têm
significado para a irradiância real — potência total, lux, PPFD — são deixadas em branco, em vez de
serem calculadas a partir de contagens.

Gravações mais antigas do DAQ-A-SD (esquema v1.01 / v1.02) registram apenas a hora de gravação do arquivo, e não um
carimbo de data/hora por leitura. O Chloros não vai comparar imagens com esses dados — associar um quadro a um
tempo de gravação seria incorreto, mesmo que não pareça errado —, mas a exportação os lê corretamente e
o arquivo CSV indica em qual relógio ele está.

Para obter informações completas sobre refletância — sensor único com câmera e sensor duplo (ambiente/objeto) — consulte [Fluxos de Trabalho de Refletância](reflectance.md).
