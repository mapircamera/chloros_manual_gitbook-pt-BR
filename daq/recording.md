# Gravação e o formato .daq

Um arquivo `.daq` é o formato de gravação do sensor de luz do Chloros: um **banco de dados SQLite** contendo quadros espectrais calibrados de um sensor DAQ. Grave um arquivo durante uma sessão de captura e o pipeline de refletância poderá, posteriormente, dividir cada imagem pela irradiância descendente medida naquele exato momento.

## O que um arquivo .daq contém

| Propriedade | Valor |
| --- | --- |
| Contêiner | Banco de dados SQLite, um arquivo por sensor por gravação |
| Nome do arquivo | Inclui o **ID do sensor**e um**carimbo de data/hora**, por exemplo, `daq_data_daq-e-def330_2026_04_13_18h30m00.daq` |
| Espectro por quadro | 135 pontos, 340–1010 nm em intervalos de 5 nm, além do triestímulo CIE XYZ |
| Unidades | Irradiação espectral calibrada, **W/m²/nm** (pacote de calibração de fábrica + perfil da tampa aplicado) |
| Metadados gravados | ID do sensor (a chave para obter a calibração de fábrica dessa unidade) e o perfil de tampa em vigor — consulte [Perfis de tampa e faixa calibrada](caps-and-range.md) |

O formato é idêntico nos modelos DAQ-U, DAQ-M e DAQ-E, portanto, o processamento posterior não leva em conta qual dispositivo de captura realizou o registro.

O registro calibrado requer o pacote de calibração de fábrica do sensor. Para o DAQ-U e o DAQ-M, o backend busca o pacote na nuvem do MAPIR por ID do sensor (a gravação é recusada se isso não for possível); as unidades DAQ-E estão isentas, pois mantêm sua calibração no próprio dispositivo.

## Gravação pela interface gráfica (GUI)

A gravação na GUI requer um **projeto aberto** (caso contrário, os botões “Gravar” ficam desativados):

* **Gravar tudo / Parar tudo** — na parte superior da barra lateral dos Sensores de Luz; inicia ou interrompe uma gravação do `.daq` em todos os sensores conectados de uma só vez.
* **Gravar / Parar Gravação** — por sensor, na janela modal de configurações (ícone de engrenagem). Um indicador vermelho “REC” aparece nas linhas de informações em tempo real do sensor durante a gravação.

Os arquivos são gravados no `<project>/light_sensor/` e, quando uma gravação é interrompida — seja por meio de “Parar”, “Parar tudo” ou pela desconexão de um sensor de gravação —, o `.daq` finalizado é **adicionado automaticamente ao projeto aberto**. Ele aparece na lista de arquivos do projeto sem a necessidade de adição manual, já pronto para o processamento de refletância.

<!-- SCREENSHOT-NEEDED: Light Sensors tab with one DAQ sensor connected and recording: sidebar showing the red "Stop All" state of the Record All button, the sensor row, and the settings modal open with the red "REC" indicator visible in the live info rows. -->

<!-- SCREENSHOT-NEEDED: File Browser / project file list immediately after stopping a DAQ recording, showing the .daq file auto-added to the open project alongside imagery. -->

## Gravação a partir do CLI

O CLI grava por meio do conjunto de sensores do backend (o backend deve estar em execução — esses comandos são clientes HTTP leves):

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

Obter o valor do `--sensor-id` a partir do `chloros-cli daq pool-list`. Dois valores padrão que vale a pena conhecer:

| Opção | Padrão |
| --- | --- |
| `--duration` | `0` — gravar até `pool-record --stop` |
| `--output` / `-o` | `~/Documents/DAQ Live View/` no sistema de arquivos do **backend**, e não no do CLI |

A distinção entre diretórios de saída é importante quando o CLI tem como destino um backend em outra máquina: o arquivo é salvo onde o backend é executado.

## Gravação a partir do Python

O `DAQSensorSession` (retornado pelo `chloros_sdk.connect_daq_sensor()`) expõe a mesma gravação agrupada: O `record_start(output_dir=None, device_name=None)` retorna o caminho do arquivo, enquanto o `record_stop()` retorna o `{path, rows}`. Consulte a [Referência do SDK](../reference/sdk-reference.md) para obter a sessão completa do API. As classes de hardware direto do SDK (somente para instalações em desktop) gravam as gravações no `~/Documents/DAQ/` por padrão; para versões lançadas, o caminho agrupado acima é a rota suportada.

## Usando um arquivo .daq no momento do processamento

Para calcular a refletância a partir de imagens, o Chloros precisa da irradiância descendente correspondente a cada exposição:

* **Mantenha o `.daq` junto com as imagens.**No momento do processamento, o pipeline determina automaticamente a**irradiação descendente com carimbo de data/hora correspondente** a partir de um `.daq` gravado (qualquer modelo de DAQ) — ou a partir de um `.csv` nativo do DAQ-M — encontrado junto com as imagens. As gravações da interface gráfica atendem a esse requisito automaticamente, uma vez que são adicionadas ao projeto no momento em que são encerradas.
* **A calibração é obtida sob demanda.** Se um pacote de calibração de fábrica por câmera ou por DAQ ainda não estiver armazenado em cache localmente, o Chloros o obtém automaticamente da nuvem do MAPIR na primeira utilização (é necessária conexão com a internet uma vez; armazenado em cache sob o `~/.chloros/`).
* **As capturas ao vivo gravam seu próprio arquivo sidecar.** Para qualquer quadro de refletância capturado ao vivo, a leitura do DAQ efetivamente utilizada é salva como um arquivo sidecar `.daq` junto com as imagens, de modo que a captura possa ser reprocessada posteriormente sem a gravação original.

## Recuperando a irradiância

O processamento de um projeto também exporta todos os registros de sensores de luz que ele contém para uma
pasta `Light Sensor/` ao lado dos produtos de imagem. Isso **não** requer imagens: um
sensor de luz operado isoladamente já constitui uma captura completa, e uma pasta contendo apenas arquivos `.daq`
é uma entrada válida. A execução informa quantos produtos do sensor de luz foram gerados.

| Produto | O que é |
| --- | --- |
| `<name>_calibrated.daq` | Um arquivo reprocessável no mesmo esquema de um registro em tempo real, agora declarando o pacote de calibração que o produziu. Reimportá-lo **não** realiza uma segunda calibração. |
| `<name>_calibrated.csv` | Irradiação espectral em W/m²/nm na grade de comprimentos de onda do próprio sensor, uma linha por leitura, além de colunas fotométricas: potência total, lux fotópico e escotópico, PPFD com sua divisão em azul/verde/vermelho e comprimento de onda de pico. |

Um DAQ-U ou DAQ-M cujo pacote de calibração não possa ser obtido — você está offline ou
esse sensor não possui calibração registrada — é **ignorada com um motivo**, nunca sendo gravada
como um arquivo “calibrado” contendo contagens brutas. Conecte-se à internet e execute novamente. Um DAQ-E
possui sua própria calibração, portanto, só precisa disso quando a unidade não está conectada e
nada está armazenado em cache localmente.

### DAQ-A: contagens brutas e por que essa é a resposta certa

O **DAQ-A** é anterior ao sistema de pacotes de calibração por porta serial e não possui nenhum pacote para
ser obtido. Isso não é um descuido: um DAQ-A é calibrado em campo em relação a um
alvo de refletância, e a calibração baseada em alvo precisa apenas da resposta *relativa*
do sensor — que é exatamente o que representam suas contagens brutas. O Chloros é calibrado com elas atualmente.

Portanto, um registro do DAQ-A é exportado, mas com um nome diferente:

```
<project>/
└── Light Sensor/
    ├── <name>_raw.daq
    └── <name>_raw.csv
```

`_raw`, e não `_calibrated` — um nome de arquivo diferente, em vez de um sinalizador dentro do arquivo,
porque a informação precisa ser preservada mesmo quando o arquivo for enviado por e-mail apenas com o nome. O cabeçalho `.csv`
indica `raw spectral sensor counts (NOT irradiance)` e alerta que os valores são
comparáveis **dentro** do arquivo e não entre sensores. As colunas que só têm
significado para a irradiância real — potência total, lux, PPFD — são deixadas em branco, em vez de
serem calculadas a partir de contagens.

Gravações mais antigas do DAQ-A-SD (esquema v1.01 / v1.02) registram apenas a hora de gravação do arquivo, e não um
carimbo de data/hora por leitura. O Chloros não irá comparar as imagens com essas — associar um quadro a um
tempo de gravação seria incorreto, mesmo que não pareça errado —, mas a exportação as lê corretamente e
o CSV indica em qual relógio ele está.

Para conhecer toda a história da refletância — sensor único com uma câmera e sensor duplo (ambiente/objeto) — consulte [Fluxos de Trabalho de Refletância](reflectance.md).
