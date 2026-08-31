---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Alvos de calibração

A MAPIR oferece diversos alvos de calibração para atender a uma ampla gama de aplicações. O compacto T4-R50, mostrado abaixo, contém 4 painéis cuja refletância da luz foi medida na faixa de 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Os alvos de referência difusos T4 apresentam as seguintes curvas de refletância, [baixe os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 250-2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 400–1.000 nm</p></figcaption></figure>Os alvos de referência difusos T4P apresentam as seguintes curvas de refletância, [baixe os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 250–2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 400–1.000 nm</p></figcaption></figure>Ao observar o gráfico de refletância, é possível perceber que os valores representam o comprimento de onda (eixo x) em função da porcentagem de refletância (eixo y). Quando capturamos uma imagem do alvo de calibração, criamos uma relação entre o valor do pixel e a porcentagem de refletância, dentro do espectro ao qual cada uma das faixas do sensor da câmera é sensível.

Isso significa que, a cada imagem capturada com nossas câmeras, você pode usar uma foto de nossos alvos de refletância, como o [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), para calibrar as imagens quanto à refletância. Uma vez calibrada, cada pixel da imagem corresponde a uma porcentagem de refletância.

Para as **saídas do Survey3** , se você exportar as imagens calibradas no Chloros como um JPG comum ou no TIFF, a porcentagem de refletância é calculada dividindo-se o valor do pixel pela profundidade de bits do formato da imagem. Portanto, para JPG, divida por 255; e para TIFF, divida por 65.535. Você também pode escolher a saída no formato PERCENT no Chloros; nesse caso, cada pixel terá um valor percentual variando de 0,0 a 1,0 (0% a 100% de refletância). Lembre-se de que alguns aplicativos de imagem não aceitam imagens em porcentagem (ponto flutuante), e elas ocupam muito espaço de armazenamento.

{% hint style="info" %}
**A refletância LATTICE utiliza uma escala de pixels diferente.** A refletância LATTICE é armazenada com DN 32.768 = 100% de refletância (e não 65.535), e cada arquivo contém uma tag XMP `Chloros:PixelScale` indicando sua escala. Leia a tag e divida pelo valor indicado, em vez de supor uma constante — consulte [Formatos de imagem de saída](output-image-formats.md).
{% endhint %}

## Alvos de calibração com câmeras LATTICE

Com as câmeras LATTICE, um alvo de calibração é **opcional** para a refletância: o Chloros pode, em vez disso, referenciar a refletância à irradiância descendente medida por um sensor de luz DAQ (ρ = π·L/E). A referência é escolhida com a configuração da fonte de refletância (Configurações do Projeto na interface gráfica; `--reflectance-source` no CLI; `reflectance_source` no SDK):

| Valor | Comportamento |
| --- | --- |
| `auto` *(padrão)* | Um alvo dentro do quadro que passou no controle de qualidade (QA) é a **referência absoluta**; quando não há alvo presente ou o controle de qualidade falha, o Chloros recorre à divisão de downwelling do DAQ. |
| `target` | Exclusivamente alvo — sem substituição por DAQ. |
| `daq` | Autoridade do DAQ — a medição descendente é sempre a referência. |

Comportamento adicional do alvo para o LATTICE:

* **Geometrias de alvo** — painéis marcados com ArUco, painéis com ROI fixa e alvos em faixa são todos suportados; a geometria é definida pela configuração de alvo do projeto.
* **Dados de alvo medidos por unidade** — `--target-reflectance-dir DIR` aponta para um diretório de varreduras de refletância de alvo medidas por unidade (`<serial>.csv`, consultadas pelo número de série/QR da unidade do alvo). Em caso de falha, o Chloros recorre aos espectros nominais T3/T4P.
* **Ancoragem temporal** — um alvo detectado calibra os quadros ao seu redor e é mantido entre as observações do alvo.

A semântica completa dos sinalizadores e exemplos estão na [Referência do CLI](reference/cli-reference.md) (consulte “Opções de exportação por produto”).

### F988

“A refletância F988 é calibrada usando um painel de refletância na cena: a banda está fora da faixa calibrada do sensor de luz do DAQ; portanto, o Chloros aplica sua captura mais recente do painel e a mantém entre as observações do painel.”

Se o F988 for executado com calibração apenas por DAQ, o Chloros rejeita a refletância baseada no DAQ para essa banda e explica o motivo (motivo da omissão: `dls-uncalibrated-band-988`); o fluxo de trabalho com painel é o caminho recomendado.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
