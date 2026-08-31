# Câmeras LATTICE

LATTICE é o sistema modular de câmeras multiespectrais da MAPIR para imagens agrícolas e científicas. Todas as câmeras LATTICE são baseadas no sensor de obturador global Sony IMX265 (**3,1 MP, pixels de 3,45 µm**) e se conectam via Ethernet como um dispositivo**GigE Vision**.

O Chloros 1.2.0 controla as câmeras LATTICE em tempo real — descoberta, visualização ao vivo, captura e arranjos sincronizados de múltiplas câmeras — a partir de três interfaces:

| Interface    | Onde                                                          | Plataformas                                                |
| ---------- | -------------------------------------------------------------- | -------------------------------------------------------- |
| GUI        | Abas **Câmeras** na barra lateral do Chloros                         | Windows 10/11 x64                                        |
| CLI        | Família de comandos `chloros-cli lattice`                           | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |
| Python SDK | `chloros_sdk.connect_camera()` / `chloros_sdk.connect_array()` | Windows 10/11 x64, Linux x86_64, Linux aarch64 (Jetson) |

> **Procurando o hardware?**Módulos de câmera, lentes, filtros e bandas, estruturas e suportes, cabos, PoE e fiação de acionamento estão documentados no [**manual do usuário da LATTICE**](https://mapir.gitbook.io/lattice-camera). Este capítulo aborda o controle das câmeras a partir do Chloros.

As capturas do LATTICE são arquivos padrão do tipo `.tif`/`.tiff`, e o Chloros sempre as processa a partir da captura bruta. Consulte a [Referência do CLI](../reference/cli-reference.md) e a [Referência do SDK](../reference/sdk-reference.md) para obter o comando completo e a interface do API.

## Duas configurações de sensores

| Configuração | Sensor       | Filtro                                | O que uma câmera fornece                                          |
| ------------- | ------------ | ------------------------------------- | ----------------------------------------------------------------- |
| **M3C**| Cor Bayer | filtro passa-banda triplo                |**Três bandas calibradas a partir de uma única exposição**                 |
| **M3M**| Monocromática   | filtro de interferência de banda estreita única |**Uma banda calibrada**; combine várias câmeras M3M para índices |

Como uma câmera M3M é monocromática por trás de um único filtro, cada banda recebe sua própria exposição. Uma câmera M3C cobre todas as suas três bandas com uma única exposição do sensor.

## Strings de modelo e nomenclatura

Cada câmera armazena sua identidade no GenICam `DeviceUserID` como uma string de modelo:

```
<sensor>-<lens>-F<filter>       e.g.  M3C-L41-FRGN,  M3M-L87-F450
```

Chloros exibe-a com o prefixo `LATT-` (por exemplo, `LATT-M3M-L87-F450`). A mesma sequência `LATT-…` é gravada na tag EXIF `Model` de cada exportação e é usada como o nome da pasta de saída da câmera em projetos processados.

| Componente | Valores                                                   | Significado                                                                                            |
| --------- | -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Sensor    | `M3C` / `M3M`                                            | Cor Bayer / monocromático                                                                          |
| Lente      | `L41` / `L87`                                            | O número representa o **campo de visão horizontal em graus**: L41 = estreito (41°), L87 = amplo (87°)    |
| Filtro    | `FRGB` / `FRGN` / `FOCN` / `FNGB` (M3C) ou `F<nm>` (M3M) | Consulte [Filtros e bandas espectrais](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands) |

A sequência do modelo determina tudo o que vem a seguir: Chloros determina o perfil do sensor, o layout das bandas e a calibração de fábrica a partir de `DeviceUserID` + `DeviceSerialNumber`. Não há nada a ser configurado por câmera — consulte [Conectando câmeras](connecting.md).

## Filtros e bandas

Os centros das bandas, as bordas do FWHM e o catálogo completo de 23 SKUs do M3M são especificações do produto, portanto, constam no manual do hardware: [**Filtros e bandas espectrais**](https://mapir.gitbook.io/lattice-camera/hardware/filters-and-bands).

O que importa no lado do software: o código do filtro na sequência do modelo determina quais produtos o Chloros pode gerar. As câmeras com filtro RGB (`FRGB`) emitem apenas produtos sem debayering e de pré-visualização — a radianância e a refletância por banda não são significativas para um sensor de banda larga, portanto, o Chloros as ignora e indica isso. Todos os outros filtros produzem a cadeia completa de radiação → refletância → índice.

## Calibração radiométrica em resumo

Cada câmera LATTICE é calibrada individualmente na fábrica com base em uma cadeia rastreável pelo NIST e é fornecida com um certificado específico para cada unidade. O que isso abrange, como é medido e a precisão que você pode citar estão no manual do hardware: [**Calibração radiométrica de fábrica**](https://mapir.gitbook.io/lattice-camera/calibration/factory-radiometric-calibration).

No que diz respeito ao software, o importante é que o Chloros determine a calibração correta quando uma câmera é conectada e fixe os coeficientes aplicados em todas as exportações — consulte [Conectando câmeras](connecting.md).

## Neste capítulo

* [Conectando câmeras](connecting.md) — descoberta automática, a caixa de diálogo de conexão da GUI, equivalentes a CLI/SDK e como a calibração de fábrica é determinada (pacote na própria câmera vs. nuvem) quando uma câmera se conecta.

Outros tópicos do LATTICE — configurações da câmera e controle ao vivo, modos de captura, arranjos com múltiplas câmeras e processamento mono (M3M) e índices — são abordados em seções específicas deste manual, e a superfície de comandos completa está disponível na [Referência do CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md).
