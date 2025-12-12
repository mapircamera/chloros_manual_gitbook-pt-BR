---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmeras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros só suporta o processamento de imagens de câmeras MAPIR. Consulte a lista de [modelos de câmeras compatíveis](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmeras no MAPIR Cloud. Consulte a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Posso calibrar minhas imagens para refletância sem um alvo de calibração?</summary>

Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo que as imagens não alvo, você não poderá relacionar os valores de pixel da imagem a uma porcentagem de refletância conhecida. Se você também não incluir o registro de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Posso editar minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros pressupõe que os dados de entrada não foram modificados. Não altere os nomes dos arquivos.

</details>

<details>

<summary>Posso configurar minhas câmeras MAPIR Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagens Survey3 devem ter uma exposição fixa/bloqueada, portanto, sem velocidade do obturador automática ou ISO automático. Todas as imagens do mesmo modelo de câmera devem ter velocidade do obturador e ISO (exposição) idênticos.

</details>

<details>

<summary>O Chloros pode processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas imagens individuais da câmera MAPIR são suportadas, não imagens unidas como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de detecção de alvos do Chloros?</summary>

Na tabela do navegador de arquivos, pré-selecionar as imagens-alvo na coluna à direita indicará ao Chloros para procurar apenas nessas imagens os alvos de calibração, acelerando consideravelmente o processamento.

</details>

<details>

<summary>Se eu for enviar minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">o MAPIR Cloud,</a> devo processá-las no Chloros antes do envio?</summary>

Se você planeja fazer o upload para nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes do upload. A nuvem realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>O MAPIR algum dia oferecerá suporte ao recurso X? Eu realmente gostaria que o MAPIR oferecesse o X.</summary>

Estamos sempre interessados em receber feedback sobre nossos produtos. Se você encontrar algum problema com nossos produtos ou tiver alguma sugestão sobre como podemos melhorá-los, entre em contato conosco para compartilhar suas ideias. A maior parte de nossa pesquisa e desenvolvimento é orientada por ouvir as principais necessidades de nossos clientes.

</details>
