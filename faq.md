---
description: Frequently Asked Questions
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/faq
---

# Perguntas frequentes

<details>

<summary>Posso processar imagens de câmeras que não sejam da marca MAPIR com o Chloros?</summary>

Não, o Chloros suporta apenas o processamento de imagens de câmeras MAPIR — das famílias Survey3 e LATTICE. Consulte a lista de [modelos de câmeras compatíveis](supported-cameras.md) para obter mais informações. Oferecemos o processamento de outras câmeras na nuvem MAPIR; veja a lista completa [aqui](https://mapir.gitbook.io/mapir-cloud/supported-cameras).

</details>

<details>

<summary>O Chloros é compatível com câmeras LATTICE?</summary>

Sim. O Chloros 1.2.0 oferece suporte completo aos módulos de câmera LATTICE M3C e M3M: **controle em tempo real**— descubra, conecte, visualize e capture a partir da guia “Câmeras” da interface gráfica do usuário (GUI), `chloros-cli lattice` ou no Python SDK, incluindo arranjos sincronizados de múltiplas câmeras com sincronização de tempo PTP — e**processamento radiométrico completo** das capturas (bruto → sem bayering → radianância → refletância → índice). Consulte [Câmeras compatíveis](supported-cameras.md) e o [guia do LATTICE](lattice/README.md).

</details>

<details>

<summary>Posso calibrar minhas imagens para refletância sem um alvo de calibração?</summary>

**Survey3:** Não. Sem uma imagem do alvo de calibração capturada ao mesmo tempo em que as imagens sem alvo são capturadas, você não conseguirá relacionar os valores dos pixels da imagem a uma porcentagem de refletância conhecida. Se você também não incluir o registro de um sensor de luz MAPIR, o espectro da luz ambiente não será medido e os resultados de refletância não serão precisos.**LATTICE:** Sim. A refletância pode ser referenciada à irradiância descendente medida por um sensor de luz DAQ em vez de um painel (ρ = π·L/E). Quando um alvo dentro do quadro que passou no controle de qualidade *está* presente, ele se torna a referência absoluta por padrão (`--reflectance-source auto`). Uma exceção: “A refletância F988 é calibrada usando um painel de refletância na cena: a banda está além da faixa calibrada do sensor de luz do DAQ; portanto, o Chloros aplica sua captura mais recente do painel e a mantém entre as observações do painel.” Consulte [Alvos de calibração](calibration-targets.md).

</details>

<details>

<summary>Preciso de um sensor de luz DAQ?</summary>

Não para radiação: as exportações de radiação do LATTICE provêm da calibração radiométrica de fábrica de cada câmera e não requerem nem um sensor DAQ nem um alvo. Para **refletância**, você precisa de uma referência para a luz ambiente — seja uma medição de luz descendente de um sensor de luz DAQ ou um alvo de calibração dentro do quadro. Um sensor DAQ permite que você produza refletância calibrada**sem colocar nenhum painel na cena**. Os arquivos `.daq` gravados são associados automaticamente às suas imagens por meio do carimbo de data e hora. Consulte [Alvos de calibração](calibration-targets.md) e a [Referência CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso usar o Chloros com um assistente de IA (Claude, ChatGPT, etc.)?</summary>

Sim — este manual e os arquivos CLI/SDK foram criados para isso:

* O índice completo do manual está disponível em `https://mapir.gitbook.io/chloros/llms.txt` para que os assistentes de IA possam localizar todas as páginas.
* O código Markdown bruto de cada página está disponível na página correspondente em letras minúsculas, URL, com `.md` anexado (por exemplo, `https://mapir.gitbook.io/chloros/reference/cli-reference.md`).
* A [Referência CLI](reference/cli-reference.md) e [Referência SDK](reference/sdk-reference.md) foram elaboradas para uso por LLMs: sinalizadores exatos, valores padrão, semântica de saída e comandos que podem ser copiados e colados.

Consulte [Assistentes de IA](ai-assistants.md) para saber como configurar seu assistente para o Chloros.

</details>

<details>

<summary>Para onde vão meus arquivos de saída processados?</summary>

Os produtos são salvos na pasta do projeto, agrupados por câmera e, em seguida, por formato de arquivo:

```
<project>/<camera-folder>/<format-folder>/<Product>_Images/
```

* **pasta-da-câmera** — `LATT-<sensor>-<lens>-F<filter>` para LATTICE, `<model>_<filter>` (por exemplo, `Survey3N_RGN`) para Survey3
* **pasta-formato** — `tiff16`, `tiff8`, `png8`, `jpg8` ou `tiff32`
* **pastas de produtos** — `Reflectance_Calibrated_Images/`, `Debayered_Images/`, `Preview_Images/`, `Radiance_Images/` (sempre na pasta `tiff32`), `<INDEX>_Index_Images/`**Os arquivos exportados mantêm o nome do arquivo de origem — a pasta identifica o produto, não um sufixo do nome do arquivo.**Com o CLI, a pasta do projeto é criada ao lado da pasta de entrada, a menos que você passe o `-o`. Observe que uma execução do `chloros-cli process` que solicitou produtos, mas não gravou nenhum, imprime `Processing finished but wrote no image products.` e**encerra com valor diferente de zero**, para que os scripts possam detectá-la. Consulte [Formatos de imagem de saída](output-image-formats.md) e a [Referência do CLI](reference/cli-reference.md).

</details>

<details>

<summary>Posso editar minhas imagens antes do processamento no Chloros?</summary>

Não. O Chloros pressupõe que os dados de entrada não foram modificados. Não altere os nomes dos arquivos.

</details>

<details>

<summary>Posso configurar minhas câmeras MAPIR e Survey3 para exposição automática e processar as imagens no Chloros?</summary>

Não. Os conjuntos de dados de imagem do Survey3 devem ter uma exposição fixa/bloqueada; portanto, não é permitido usar velocidade de obturador automática nem ISO automático. Todas as imagens do mesmo modelo de câmera devem ter velocidade de obturador e ISO (exposição) idênticos.

As câmeras LATTICE não têm essa restrição: o Chloros controla a exposição em tempo real (Smart AE), e cada captura registra a exposição e o ganho efetivamente utilizados, o que é levado em conta pelo pipeline radiométrico.

</details>

<details>

<summary>O Chloros pode processar ou analisar imagens ortomosaicas?</summary>

Não. Apenas imagens individuais de câmeras MAPIR são suportadas, não imagens unidas, como um mapa ortomosaico.

</details>

<details>

<summary>Como posso acelerar a etapa de detecção de alvos do Chloros?</summary>

Na tabela do navegador de arquivos, a pré-seleção das imagens-alvo na coluna da direita instruirá o Chloros a procurar alvos de calibração apenas nessas imagens, acelerando consideravelmente o processamento.

</details>

<details>

<summary>Se eu for enviar minhas imagens para <a href="https://www.mapir.camera/collections/software/products/mapir-cloud-subscription">o MAPIR Cloud,</a> devo processá-las no Chloros antes do envio?</summary>

Se você planeja enviar para nossa plataforma de processamento online [MAPIR Cloud](https://www.mapir.camera/collections/software/products/mapir-cloud-subscription), não edite as imagens antes do envio. O Cloud realizará todo o mesmo processamento e muito mais.

</details>

<details>

<summary>O MAPIR virá a oferecer suporte ao recurso X? Eu realmente gostaria que o MAPIR oferecesse o recurso X.</summary>

Estamos sempre interessados em receber feedback sobre nossos produtos. Se você encontrar algum problema com nossos produtos ou tiver uma sugestão de como podemos melhorá-los, por favor, [ENTRE EM CONTATO CONOSCO](https://www.mapir.camera/community/contact) para compartilhar suas ideias. Grande parte de nossa pesquisa e desenvolvimento (P&amp;D) é orientada por ouvir as principais necessidades de nossos clientes.

</details>

<details>

<summary>O Chloros está disponível para o Linux?</summary>

Sim! O Chloros 1.2.0 é compatível com o Linux amd64 (x86_64) e arm64 (NVIDIA Jetson JetPack 6) por meio dos pacotes `.deb`. O CLI e o Python SDK são totalmente compatíveis com o Linux, incluindo o controle em tempo real de câmeras LATTICE e sensores DAQ. Não há interface gráfica (GUI) para o Linux — toda a interação é feita por meio do [CLI](CLI.md) ou do [Python SDK](api-python-sdk.md). Consulte a [Visão geral do Linux](linux/linux-overview.md) para obter mais detalhes.

</details>

<details>

<summary>Posso executar o Chloros no NVIDIA Jetson?</summary>

Sim! O Chloros é compatível com plataformas NVIDIA Jetson, incluindo Jetson Nano, Orin Nano, Orin NX e AGX Orin com o JetPack 6. O Chloros detecta automaticamente o seu modelo de Jetson e otimiza sua estratégia de processamento. Consulte o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para obter instruções de configuração e implantação.

</details>

<details>

<summary>O Chloros otimiza automaticamente para o meu hardware?</summary>

Sim! O Chloros inclui a [Adaptação Dinâmica de Computação](processing-architecture/dynamic-compute-adaptation.md), que detecta automaticamente sua CPU, GPU, RAM e (no Jetson) sensores térmicos. Em seguida, ele seleciona a estratégia de processamento ideal — desde o `GPU_PARALLEL` em sistemas com muita memória até o `GPU_SINGLE` em dispositivos com recursos limitados e o `CPU_PARALLEL` em sistemas sem uma GPU NVIDIA. Não é necessária nenhuma configuração manual.

</details>

<details>

<summary>O que é o pipeline de processamento de 4 threads?</summary>

O Chloros utiliza uma arquitetura em pipeline de 4 threads para usuários do Chloros+: A thread 1 (Detecção) carrega imagens e detecta alvos de calibração; a thread 2 (Calibração) calcula a calibração de refletância; a thread 3 (Processamento) realiza o debayering acelerado por GPU e o cálculo de índice; e a thread 4 (Exportação) grava os arquivos de saída. Várias imagens podem estar em diferentes threads simultaneamente para obter o máximo rendimento. Consulte [Pipeline de processamento](processing-architecture/processing-pipeline.md) para obter detalhes.

</details>

<details>

<summary>Como faço para executar diagnósticos na minha instalação do Chloros?</summary>

Use o comando `selftest` para executar um teste de funcionamento de 7 etapas: versão, disponibilidade de porta, inicialização do backend, conectividade do API (`/api/test`), informações do sistema (`/api/system-info` — GPU/CUDA/PyTorch), presença do modelo de redução de ruído e prontidão do CUDA + redutor de ruído:

```bash
chloros-cli selftest
```

Isso é especialmente útil em sistemas Linux/Jetson para verificar a configuração da GPU e do CUDA.

</details>
