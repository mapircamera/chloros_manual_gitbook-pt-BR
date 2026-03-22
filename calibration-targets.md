---
description: Lab-measured panels used to calibrate captured data in post processing
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/calibration-targets
---

# Alvos de calibração

A MAPIR oferece diversos alvos de calibração para atender a uma ampla gama de aplicações. O compacto T4-R50, mostrado abaixo, contém 4 painéis cuja refletância da luz foi medida na faixa de 250 a 2.500 nm.

<figure><img src=".gitbook/assets/t4-r50_2.jpg" alt=""><figcaption><p>MAPIR T4-R50</p></figcaption></figure>Os alvos de referência difusa T4 apresentam as seguintes curvas de refletância, [baixe os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (250-2500nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 250-2.500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4 (400-1000nm).png" alt=""><figcaption><p>MAPIR Refletância T4 :: 400-1.000 nm</p></figcaption></figure>Os alvos de referência difusos T4P apresentam as seguintes curvas de refletância, [baixe os dados aqui](https://cdn.shopify.com/s/files/1/0972/5566/files/MAPIR_Diffuse_Reflectance_Standard_Calibration_Target_Data_T4.xlsx?v=1741759157):

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 350-2500nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 250-2500 nm</p></figcaption></figure>

<figure><img src=".gitbook/assets/MAPIR Diffuse Reflectance Standard Calibration Target Data T4P -- 400-1000nm.jpg" alt=""><figcaption><p>MAPIR Refletância T4P :: 400-1000 nm</p></figcaption></figure>Ao observar o gráfico de refletância, você pode ver que os valores representam o comprimento de onda (eixo x) em função da porcentagem de refletância (eixo y). Quando capturamos uma imagem do alvo de calibração, criamos uma relação entre o valor do pixel e a porcentagem de refletância, dentro do espectro ao qual cada uma das bandas do sensor da câmera é sensível.

Isso significa que, para cada imagem capturada com nossas câmeras, você pode usar uma foto de nossos alvos de refletância, como o [T4-R50](https://www.mapir.camera/collections/calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t3-r50) ou o [T4-R125](https://www.mapir.camera/collections/multispectral-reflectance-reference-calibration-targets/products/diffuse-reflectance-standard-calibration-target-package-t4-r125), para calibrar as imagens quanto à refletância. Uma vez calibrada, cada pixel da imagem corresponde a uma porcentagem de refletância.

Se você exportar as imagens calibradas no Chloros como um JPG comum ou no TIFF, a porcentagem de refletância é calculada dividindo-se o valor do pixel pela profundidade de bits do formato da imagem. Portanto, para JPG, divida por 255, e para TIFF, divida por 65.535. Você também pode escolher a saída no formato PERCENT no Chloros; nesse caso, cada pixel terá um valor percentual entre 0,0 e 1,0 (0% a 100% de refletância). Lembre-se de que alguns aplicativos de imagem não aceitam imagens em formato percentual (ponto flutuante) e que elas ocupam muito espaço de armazenamento.

<div><figure><img src=".gitbook/assets/t3-125.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_2.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure> <figure><img src=".gitbook/assets/t3-125_closed.jpg" alt=""><figcaption><p>T4-R125</p></figcaption></figure></div>
