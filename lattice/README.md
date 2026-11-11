# Câmeras LATTICE

O LATTICE é o sistema modular de câmeras multiespectrais dMAPIR para imagens agrícolas e científicas. Todas as câmeras LATTICE são equipadas com o sensor Sony IMX265 de obturador global (**3,1 MP, pixels de 3,45 µm**) e se conectam via Ethernet como um dispositivo**GigE Vision**. O

Chloros 1.3.0 controla as câmeras LATTICE em tempo real — descoberta, visualização ao vivo, captura e arranjos sincronizados de múltiplas câmeras — a partir de três interfaces:

| Interface    | Onde                                                          | Plataformas                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Guia **Câmeras** na barra lateral do Chloros                         | Windows 10/11 x64                                        |
| CLI        | Família de comandos `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86\_64, Linux aarch64 (Jetson) |

> **Procurando o hardware?**Módulos de câmera, lentes, filtros e bandas, estruturas e suportes, cabos, PoE e fiação de acionamento estão documentados no [**manual do usuário do LATTICE**](https://mapir.gitbook.io/lattice-camera). Este capítulo aborda o controle das câmeras a partir do Chloros.

As capturas do LATTICE são arquivos padrão `.tif`/`.tiff`, e o Chloros sempre as processa a partir da captura bruta. Consulte a [Referência da CLI](../reference/cli-reference.md) e a [Referência do SDK](../reference/sdk-reference.md) para obter a lista completa de comandos e a interface da API.

## Duas configurações de sensor

| Configuração | Sensor       | Filtro                                | O que uma câmera fornece                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Cor Bayer | filtro de passagem de banda tripla                |**Três bandas calibradas a partir de uma única exposição**                 |
| **M3M**| Monocromática   | filtro de interferência de banda estreita única |**Uma banda calibrada**; combine várias câmeras M3M para índices |

Como uma câmera M3M é monocromática por trás de um único filtro, cada banda recebe sua própria exposição. Uma câmera M3C cobre todas as suas três bandas com uma única exposição do sensor.

## Sequências de modelo e nomenclatura

Cada câmera armazena sua identidade no GenICam `DeviceUserID` como uma sequência de modelo:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

O Chloros exibe essa sequência com o prefixo `LATT-` (por exemplo, `LATT-M3M-L87-F450`). A mesma sequência `LATT-…` é gravada na tag EXIF `Model` de cada exportação e é usada como nome da pasta de saída da câmera em projetos processados.

| Componente | Valores                                                   | Significado                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Cor Bayer / monocromático                                                                          |
| Lente      | `L41` / `L87`                                            | O número representa o **campo de visão horizontal em graus**: L41 = estreito (41°), L87 = amplo (87°)    |
| Filtro    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ou `F<nm>` (M3M) | Consulte [Filtros e faixas espectrais](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

A string do modelo controla tudo o que vem a seguir: o Chloros determina o perfil do sensor, o layout das bandas e a calibração de fábrica a partir de `DeviceUserID` + `DeviceSerialNumber`. Não há nada a ser configurado por câmera — consulte [Conectando câmeras](connecting.md).

## Filtros e bandas

Os centros das bandas, as bordas FWHM e o catálogo completo de 23 SKUs do M3M são especificações do produto, portanto, estão no manual do hardware: [**Filtros e bandas espectrais**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

O que importa do ponto de vista do software: o código do filtro na string do modelo determina quais produtos a Chloros pode gerar. Câmeras com filtro RGB (`FRGB`) emitem apenas produtos sem debayering e de pré-visualização — a radianância e a refletância por banda não são significativas para um sensor de banda larga; por isso, o Chloros as ignora e informa isso. Todos os outros filtros geram a cadeia completa radianância → refletância → índice.

## Calibração radiométrica em resumo

Cada câmera LATTICE é calibrada individualmente na fábrica em relação a uma cadeia rastreável pelo NIST e é fornecida com um certificado específico para cada câmera. O que isso abrange, como é medido e a precisão que você pode citar estão no manual do hardware: [**Calibração radiométrica de fábrica**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

No que diz respeito ao software, o importante é que o Chloros determine a calibração correta quando uma câmera é conectada e fixe os coeficientes aplicados em cada exportação — consulte [Conectando câmeras](connecting.md).

## Neste capítulo

* [Conectando câmeras](connecting.md) — descoberta automática, a caixa de diálogo de conexão da GUI, equivalentes em CLI/SDK e como a calibração de fábrica é determinada (pacote na câmera vs. nuvem) quando uma câmera se conecta.
* [Configurações da câmera](camera-settings.md) — todos os controles na guia “Câmeras”: a lista de câmeras, o painel de configurações por câmera e o painel de configurações da matriz (incluindo o editor de ROI e o alinhamento).
* [Configurações e modos de captura](capture.md) — Captura total, captura única/contínua/por intervalo, tipos de exportação e a estrutura da pasta de capturas.
* [Matrizes Multicâmera](arrays.md) — o assistente de conexão da matriz, a sonda do cabo de sincronização, a largura de banda, o PTP e a visualização ao vivo da matriz.
* [Câmeras Mono e Índices de Vegetação](mono-indices.md) — câmeras M3M e o fluxo de trabalho alinhar → empilhar → indexar.

A lista completa de comandos está disponível na [Referência da CLI](../reference/cli-reference.md) e na [Referência do SDK](../reference/sdk-reference.md).
