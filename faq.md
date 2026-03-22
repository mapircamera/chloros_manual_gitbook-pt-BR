---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmeras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros suporta apenas o processamento de imagens de câmeras MAPIR. Consulte a lista de [modelos de câmeras compatíveis](supported-cameras.md) para obter mais informações. Oferecemos o processamento de imagens de outras câmeras no MAPIR Cloud; veja a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>Posso calibrar minhas imagens para refletância sem um alvo de calibração?</summary>

Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo que as imagens sem alvo, você não poderá relacionar os valores de pixel da imagem a uma porcentagem de refletância conhecida. Se você também não incluir o registro de um sensor de luz MAPIR, o espectro de luz ambiente não será medido e os resultados de refletância não serão precisos.

</details>

<details>

<summary>Posso editar minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros pressupõe que os dados de entrada não foram modificados. Não altere os nomes dos arquivos.

</details>

<details>

<summary>Posso configurar minhas câmeras MAPIR e Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagem do Survey3 devem ter uma exposição fixa/bloqueada, portanto, sem velocidade de obturador automática ou ISO automático. Todas as imagens do mesmo modelo de câmera devem ter velocidade de obturador e ISO (exposição) idênticos.

</details>

<details>

<summary>O Chloros pode processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas imagens individuais de câmera do MAPIR são suportadas, não imagens costuradas como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de detecção de alvos do Chloros?</summary>

Na tabela do navegador de arquivos, pré-selecionar as imagens-alvo na coluna da direita instruirá o Chloros a procurar alvos de calibração apenas nessas imagens, acelerando significativamente o processamento.

</details>

<details>

<summary>Se eu for enviar minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">o MAPIR Cloud,</a> devo processá-las no Chloros antes do envio?</summary>

Se você planeja enviar para nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes do envio. O Cloud realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>O MAPIR algum dia oferecerá suporte ao recurso X? Eu realmente gostaria que o MAPIR oferecesse X.</summary>

Estamos sempre interessados em receber feedback sobre nossos produtos. Se você encontrar algum problema com nossos produtos ou tiver uma sugestão sobre como podemos melhorá-los, por favor, [ENTRE EM CONTATO CONOSCO](https://www.mapir.camera/community/contact) para compartilhar suas ideias. A maior parte de nossa P&amp;D é orientada por ouvir as principais necessidades de nossos clientes.

</details>

<details>

<summary>O Chloros está disponível para o Linux?</summary>

Sim! O Chloros 1.1.0 oferece suporte ao Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) por meio dos pacotes `.deb`. O CLI e o Python SDK são totalmente compatíveis com o Linux. Não há GUI para o Linux — toda a interação é feita via [CLI](CLI.md) ou [Python SDK](api-python-sdk.md). Consulte [Visão geral do Linux](linux/linux-overview.md) para obter detalhes.

</details>

<details>

<summary>Posso executar o Chloros no NVIDIA Jetson?</summary>

Sim! O Chloros 1.1.0 é compatível com plataformas NVIDIA Jetson, incluindo Jetson Nano, Orin Nano, Orin NX e AGX Orin executando o JetPack 6. O Chloros detecta automaticamente o seu modelo Jetson e otimiza sua estratégia de processamento. Consulte o [Guia NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obter instruções de configuração e implantação.

</details>

<details>

<summary>O Chloros otimiza automaticamente para o meu hardware?</summary>

Sim! O Chloros 1.1.0 inclui a [Adaptação Dinâmica de Computação](processing-architecture/dynamic-compute-adaptation.md), que detecta automaticamente sua CPU, GPU, RAM e (no Jetson) sensores térmicos. Em seguida, ele seleciona a estratégia de processamento ideal — desde o `GPU_PARALLEL` em sistemas com muita memória até o `GPU_SINGLE` em dispositivos com recursos limitados e o `CPU_PARALLEL` em sistemas sem uma GPU NVIDIA. Não é necessária nenhuma configuração manual.

</details>

<details>

<summary>O que é o pipeline de processamento de 4 threads?</summary>

O Chloros 1.1.0 utiliza uma arquitetura em pipeline de 4 threads para usuários do Chloros+: O Thread 1 (Detecção) carrega imagens e detecta alvos de calibração, o Thread 2 (Calibração) calcula a calibração de refletância, o Thread 3 (Processamento) realiza o debayering acelerado por GPU e o cálculo de índice, e o Thread 4 (Exportação) grava os arquivos de saída. Várias imagens podem estar em threads diferentes simultaneamente para obter o máximo rendimento. Consulte [Pipeline de Processamento](processing-architecture/processing-pipeline.md) para obter detalhes.

</details>

<details>

<summary>Como faço para executar diagnósticos na minha instalação do Chloros?</summary>

Use o comando `selftest` para executar 7 diagnósticos do sistema, incluindo verificação de versão, disponibilidade de porta, inicialização do backend, conectividade do API, informações do sistema, modelos de denoiser e disponibilidade de CUDA:

```bash
chloros-cli selftest
```

Isso é especialmente útil em sistemas Linux/Jetson para verificar a configuração da GPU e do CUDA.

</details>
