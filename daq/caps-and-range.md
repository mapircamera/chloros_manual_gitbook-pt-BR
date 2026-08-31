# Perfis das tampas e faixa calibrada

> As próprias tampas — quais são fornecidas com quais sensores, como são montadas e seu comportamento óptico — estão documentadas no **[manual do usuário do DAQ](https://mapir.gitbook.io/daq)**. Esta página aborda a *declaração* da tampa instalada no Chloros, o que garante que a correção seja feita corretamente.

A calibração radiométrica de fábrica de cada sensor de luz do DAQ descreve o sensor *nu*. A tampa física instalada sobre o difusor altera a luz que o sensor capta; portanto, o Chloros aplica um **perfil de correção da tampa** medido na fábrica sobre o pacote de calibração. Declarar a tampa correta faz parte do processo de obtenção de dados calibrados — esta página aborda quais tampas existem por modelo, como declará-las e qual é, de fato, a faixa espectral calibrada do sensor.

## Disponibilidade de tampas por modelo

| Perfil da tampa (`cap_id`) | Tampa física | DAQ-U | DAQ-M | DAQ-E |
| --- | --- | --- | --- | --- |
| `sunshine_cosine` | Tampa corretora de cosseno para luz solar (**padrão em todos os modelos**) | Sim | Sim | Sim |
| `fov_15` / `fov_45` / `fov_90` | Cones de restrição do campo de visão (15° / 45° / 90°) | Sim | — | Sim |
| `fov_30` / `fov_60` | Cones de restrição do campo de visão (30° / 60°) | Sim | — | — |
| `none` | Sem tampa instalada | — | — | Sim |

Observações específicas do modelo:

* **O DAQ-M possui um único perfil de tampa: `sunshine_cosine`.** “Bare-plus-Sunshine-cap” é sua definição de produto, e um DAQ-M “bare” não necessita de perfil geométrico.
* **Um DAQ-U “bare” é verdadeiramente “bare”** — não necessita de nenhum perfil geométrico, razão pela qual não existe o perfil `none` para ele.
* **O `none` em um DAQ-E NÃO é uma operação nula.** O difusor recuado e coberto por vidro do DAQ-E possui uma correção geométrica real própria; portanto, “sem tampa” é, por si só, um perfil medido nesse modelo.
* Um **DAQ-E sem acessórios não consegue medir a luz solar direta em nenhuma elevação** — a tampa Sunshine é a configuração de campo. Não planeje trabalhos ao ar livre com um DAQ-E sem acessórios.

Nas configurações por sensor da GUI (ícone de engrenagem na aba “Sensores de Luz”), o menu suspenso **Tampa** também oferece a opção “Nenhuma (sensor sem tampa)” no DAQ-U e no DAQ-M — nesses dois modelos, “sem tampa” significa simplesmente que nenhuma correção de tampa é aplicada, conforme as notas acima. Selecione essa opção somente quando a tampa estiver fisicamente removida.

## Declarando a tampa — e por que isso é importante

**O código `cap_id` declarado deve corresponder à tampa que está fisicamente no sensor.** Nem o sensor nem o software conseguem detectar a tampa instalada. A declaração determina duas coisas:

1. A **correção em tempo real** aplicada a cada espectro.
2. O **código da tampa gravado em cada gravação `.daq`**, no qual o processamento de refletância posterior se baseia.

A tampa Sunshine atenua em aproximadamente **12× por padrão**; portanto, gravar com a tampa errada declarada distorce a escala dos espectros por esse fator. Declare alterações na tampa imediatamente.

### Configurando a tampa

GUI: guia “Sensores de luz” → ícone de engrenagem na linha do sensor → menu suspenso **“Tampa”**. O padrão para todos os modelos é `sunshine_cosine` (todos os sensores DAQ vêm com o corretor de cosseno instalado), e a seleção é mantida no projeto.

<!-- SCREENSHOT-NEEDED: DAQ tab per-sensor settings modal (gear icon) scrolled to the Cap dropdown, open to show the per-model choices with "Sunshine (cosine corrector)" selected. Use a connected DAQ-E so the Hostname/Firmware/PTP rows are also visible above it. -->

CLI (o backend deve estar em execução):

```bash
# Declare at connect time
chloros-cli daq pool-connect --eth-host daq-e-def330.local --cap-id sunshine_cosine

# Swap at runtime (after physically changing the cap)
chloros-cli daq pool-set-cap --sensor-id daq-e-def330 --cap-id fov_45
```

O CLI aceita sintaticamente toda a lista `cap_id` (`{none, fov_15, fov_30, fov_45, fov_60, fov_90, sunshine_cosine}`); cada perfil é validado em relação ao modelo do sensor na conexão; portanto, um ID de capacitância indisponível (por exemplo, um ID apenas E em um DAQ-U) resulta em falha com um erro claro, em vez de uma correção incorreta. O padrão do backend quando nada é passado é `sunshine_cosine`.

Python SDK observação: `cap_id` **não** é um controle SDK — `connect_daq_sensor()` / `DAQSensorSession` não expõem nenhum parâmetro de cap. Selecione o limite por meio dos comandos CLI acima ou do menu suspenso da GUI; consulte a [Referência do SDK](../reference/sdk-reference.md).

Avançado: os perfis são fornecidos na instalação do Chloros em `daq/cap_profiles/<u|m|e>/<cap_id>.json` e podem ser substituídos por usuário em `~/.chloros/daq_cap_profiles/<u|m|e>/<cap_id>.json`.

Independentemente dos limites, os sensores que nunca foram recalibrados recebem automaticamente um pequeno ajuste de desvio de escuro derivado da frota — sem necessidade de ação do usuário.

## Desempenho do limite de luz solar (a configuração para uso externo)

Números nos quais você pode basear seus procedimentos:

| Propriedade | Valor |
| --- | --- |
| Campo de visão | Hemisférico de 180° |
| Erro de resposta cosseno | ≤ ±4 % até 60° de incidência; ≤ ±4,5 % até 70° |
| Limite de sol baixo | Não recomendado abaixo de ~15° de elevação solar |
| Atenuação | ~12× (por projeto) |
| Repetibilidade da remontagem da capa | ≈ 1,5 % |
| Irradiação quantitativa | Média de **≥ 15 s** de leituras (característica do instrumento, não é um defeito) |

Para qualquer valor de irradiação quantitativa — incluindo referências de refletância —, utilize uma média de pelo menos 15 segundos de leituras, em vez de um único quadro.

## Faixa espectral calibrada

| Propriedade | Valor |
| --- | --- |
| Amostragem espectral | 340–1010 nm em intervalos de 5 nm (135 pontos) |
| Faixa calibrada radiometricamente | **~374–974 nm** (imposta pelo software) |

O sensor reporta a grade completa de 340–1010 nm, mas o ganho radiométrico rastreável ao NIST abrange ~374–974 nm. O Chloros **recusa a divisão por refletância absoluta** para qualquer banda da câmera com menos da metade de seu peso espectral dentro desse intervalo, relatando o motivo de omissão `dls-uncalibrated-band-<nm>` em vez de produzir um produto não calibrado. Entre os SKUs de câmeras disponíveis para envio, apenas o filtro F988 fica fora desse intervalo; ele utiliza, em vez disso, o fluxo de trabalho com painel de refletância — consulte [Fluxos de Trabalho de Refletância](reflectance.md).

Para modelos de sensores, transportes e IDs de sensores, consulte a [visão geral do DAQ](README.md). Para saber como o carimbo de limite máximo é consumido durante o processamento, consulte [Gravação e o formato .daq](recording.md).
