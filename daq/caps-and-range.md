# Perfis das tampas e faixa calibrada

> As próprias tampas — quais são fornecidas com quais sensores, como são montadas e seu comportamento óptico — estão documentadas no **[manual do usuário do DAQ](https://mapir.gitbook.io/daq)**. Esta página aborda a *declaração* da tampa instalada ao Chloros, o que garante que a correção seja feita corretamente.

A calibração radiométrica de fábrica de cada sensor de luz do DAQ descreve o sensor *nu*. A tampa física instalada sobre o difusor altera a luz que o sensor capta; por isso, o Chloros aplica um **perfil de correção da tampa** medido na fábrica sobre o pacote de calibração. Declarar a tampa correta faz parte do processo de obtenção de dados calibrados — esta página aborda quais tampas existem por modelo, como declará-las e qual é, de fato, a faixa espectral calibrada do sensor.

## Disponibilidade de tampas por modelo

| Perfil da tampa (`cap_id`) | Tampa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tampa corretora de cosseno para luz solar (**padrão em todos os modelos**) | Sim | Sim | Sim |
| `fov_15` / `fov_45` / `fov_90` | Cones de restrição do campo de visão (15° / 45° / 90°) | Sim | — | Sim |
| `fov_30` / `fov_60` | Cones de restrição do campo de visão (30° / 60°) | Sim | — | — |
| `none` | Sem tampa instalada | — | — | Sim |

Observações específicas do modelo:

* **O DAQ-M possui um único perfil de tampa: `sunshine_cosine`.** “Bare-plus-Sunshine-cap” é sua definição de produto, e um DAQ-M “bare” não necessita de perfil geométrico.
* **Um DAQ-U “bare” é verdadeiramente “bare”** — ele não necessita de nenhum perfil geométrico, razão pela qual não existe o perfil `none` para ele.
* **O `none` em um DAQ-E NÃO é uma operação nula.** O difusor recuado e coberto por vidro do DAQ-E possui uma correção geométrica real própria; portanto, “sem tampa” é, por si só, um perfil medido nesse modelo.
* Um **DAQ-E sem acessórios não consegue medir a luz solar direta em nenhuma elevação** — a tampa Sunshine é a configuração de campo. Não planeje trabalhos ao ar livre com um DAQ-E sem acessórios.

Nas configurações por sensor da interface gráfica (ícone de engrenagem na aba “Sensores de Luz”), o menu suspenso **Tampa** também oferece a opção “Nenhuma (sensor sem tampa)” no DAQ-U e no DAQ-M — nesses dois modelos, “sem tampa” significa simplesmente que nenhuma correção de tampa é aplicada, conforme as notas acima. Selecione essa opção somente quando a tampa estiver fisicamente removida.

## Declarando a tampa — e por que isso é importante

**O código `cap_id` declarado deve corresponder à tampa que está fisicamente no sensor.** Nem o sensor nem o software conseguem detectar a tampa instalada. A declaração determina duas coisas:

1. A **correção em tempo real** aplicada a cada espectro.
2. O **código da tampa gravado em cada registro `.daq`**, no qual o processamento de refletância posterior se baseia.

A tampa Sunshine atenua em aproximadamente **12× por padrão**; portanto, registrar com a tampa errada declarada distorce a escala dos espectros em cerca desse fator. Declare alterações na tampa imediatamente.

### Configurando a tampa

GUI: guia “Sensores de luz” → ícone de engrenagem na linha do sensor → menu suspenso **“Tampa”**. O padrão para todos os modelos é `sunshine_cosine`, uma suposição (os modelos DAQ-U, DAQ-M e DAQ-E são fornecidos sem tampa, e um registro indica que a tampa foi assumida até que você escolha uma), e a seleção permanece válida para o projeto.



CLI<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->
(o backend deve estar em execução):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

A CLI aceita sintaticamente a lista completa `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil é validado em relação ao modelo do sensor no momento da conexão; portanto, um ID de cap inválido (por exemplo, um ID exclusivo do E em um DAQ-U) resulta em uma falha com um erro claro, em vez de uma correção incorreta. O padrão do backend, quando nada é passado, é `sunshine_cosine`.

Python SDK: nomeie o cap no momento da conexão com `connect_daq_sensor(cap_id="sunshine_cosine")` ou troque-o em tempo de execução com `DAQSensorSession.set_cap("fov_45")`; consulte a [Referência do SDK](../reference/sdk-reference.md#daq-sensor-sessions).

Avançado: os perfis são fornecidos na instalação do Chloros em `daq/cap_profiles/<u|m|e>/<cap_id>.json` e podem ser substituídos por usuário em `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Independentemente dos limites, os sensores que nunca foram recalibrados recebem automaticamente um pequeno ajuste de desvio de escuro derivado da frota — sem necessidade de ação do usuário.

## DAQ-E-S: difusor fixo e inclinação

O **DAQ-E-S** é um DAQ-E com duas diferenças importantes para o Chloros. Todo o restante (detecção, conexão, PTP, gravação, configurações do dispositivo) funciona exatamente como em um DAQ-E.

* **Seu difusor é permanente, portanto não requer correção de tampa.** O difusor já estava na unidade quando sua calibração de fábrica foi medida; portanto, essa calibração já indica a irradiância descendente real. O Chloros não aplica nenhum perfil de limite a ele, independentemente do que for solicitado: um limite nomeado para ele é ignorado com uma mensagem, seus registros indicam que o limite veio do modelo e o aviso de limite presumido nunca é acionado para ele. Em Configurações do Projeto → Sensor de Luz DAQ, sua linha não exibe nenhum limite, e a substituição de limite para todo o projeto não se aplica a ele. Não aplique um limite aos seus dados manualmente: a correção Sunshine resultante apresentaria um valor cerca de 11 vezes maior.
* **Ele possui um acelerômetro de 3 eixos.** A resposta cosseno do difusor só é válida enquanto ele estiver nivelado; portanto, um sensor inclinado relata uma irradiância incorreta sem apresentar nenhum outro sintoma. Cada leitura do DAQ-E-S, portanto, contém seus dados de inclinação, rotação e inclinação longitudinal, e um `.daq` registrado a partir de um deles os armazena ao lado de cada espectro (`imu_tilt_deg`, `imu_roll_deg`, `imu_pitch_deg`, …), permitindo distinguir uma nuvem de uma montagem deslocada.

Para cada campo de atitude e para saber como uma falha no fluxo de atitude é relatada, consulte [DAQ-E-S na Referência da CLI](../reference/cli-reference.md#daq-e-s--fixed-diffuser-and-tilt-sensor).

## Desempenho do Sunshine Cap (configuração para uso ao ar livre)

Números nos quais você pode basear seus procedimentos:

| Propriedade | Valor |
| --- | --- |
| Campo de visão | Hemisférico de 180° |
| Erro de resposta cosseno | ≤ ±4 % até 60° de incidência; ≤ ±4,5 % até 70° |
| Limite de sol baixo | Não recomendado abaixo de ~15° de elevação solar |
| Atenuação | ~12× (por projeto) |
| Repetibilidade da remontagem da tampa | ≈ 1,5 % |
| Irradiação quantitativa | Média de **≥ 15 s** de leituras (característica do instrumento, não um defeito) |

Para qualquer valor de irradiação quantitativa — incluindo referências de refletância —, utilize uma média de pelo menos 15 segundos de leituras, em vez de um único quadro.

## Faixa espectral calibrada

| Propriedade | Valor |
| --- | --- |
| Amostragem espectral | 340–1010 nm em intervalos de 5 nm (135 pontos) |
| Faixa calibrada radiometricamente | **~374–974 nm** (imposta pelo software) |

O sensor reporta a grade completa de 340–1010 nm, mas o ganho radiométrico rastreável ao NIST abrange ~374–974 nm. O Chloros **recusa a divisão de refletância absoluta** para qualquer banda da câmera com menos da metade de seu peso espectral dentro desse intervalo, relatando o motivo de omissão `dls-uncalibrated-band-<nm>` em vez de produzir um resultado não calibrado. Entre os SKUs de câmeras disponíveis para envio, apenas o filtro F988 fica fora desse intervalo; ele utiliza, em vez disso, o fluxo de trabalho com painel de refletância — consulte [Fluxos de Trabalho de Refletância](reflectance.md).

Para modelos de sensores, transportes e IDs de sensores, consulte a [visão geral do DAQ](README.md). Para saber como o carimbo de limite é consumido durante o processamento, consulte [Gravação e o formato .daq](recording.md).
