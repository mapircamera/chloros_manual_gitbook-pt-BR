---
metaLinks: {}
---

# Introdução

<div data-full-width="false"><figure><img src=".gitbook/assets/chloros_logo_transparent.png" alt=""><figcaption></figcaption></figure></div>O Chloros é um aplicativo de software da [MAPIR](https://www.mapir.camera) para processar imagens e outros dados de sensores.

***{% hint style="success" %}**Novidades no Chloros 1.1.0**: Suporte nativo ao Linux (amd64 e arm64), computação de ponta NVIDIA Jetson, Adaptação Dinâmica de Computação, pipeline de processamento de 4 threads, novos comandos e opções do CLI. Consulte [Download](download.md) para ver o changelog completo.
{% endhint %}

O Chloros está disponível em 3 modos de aplicação:

## Chloros: Aplicação GUI para desktop

Janela separada independente com todos os recursos. _Somente para Windows._

## [Chloros CLI: Interface de linha de comando](CLI.md)

Processamento em lote por linha de comando. Perfeito para automação, scripts e operação sem interface gráfica. Disponível em **Windows, Linux amd64 e Linux arm64 (NVIDIA Jetson)**. _A CLI requer uma licença Chloros+ para acesso._

## [Chloros API: Python SDK](api-python-sdk.md)

Interface programática Python para automação e fluxos de trabalho personalizados. Perfeita para pipelines de pesquisa, integração com aplicativos Python existentes e criação de ferramentas personalizadas. Disponível em **todas as plataformas** via `pip install chloros-sdk`. _A API requer uma licença Chloros+ para acesso._***

## Plataformas compatíveis

| Plataforma | GUI | CLI | Python SDK |
| --- | --- | --- | --- |
| **Windows 10/11** | Sim | Sim | Sim |
| **Linux amd64 (x86_64)** | Não | Sim | Sim |
| **Linux arm64 (NVIDIA Jetson)** | Não | Sim | Sim |

Para obter instruções de instalação do Linux, consulte a seção [Linux e Computação de Borda](linux/linux-overview.md).

***

## Chloros+

Embora o Chloros seja gratuito para a maioria das tarefas, você pode perceber que precisa de mais recursos. É aí que uma licença paga para o Chloros+ pode ser vantajosa para você. Com uma licença do Chloros+, você pode desbloquear novos recursos, tais como:

* **Processamento multithread**: acelere significativamente o processamento de imagens em projetos maiores, processando imagens simultaneamente através do pipeline.
* **Aceleração por GPU (CUDA)**: aproveite as opções atuais de maior memória de GPU para acelerar ainda mais o pipeline de processamento de imagens. Recomendamos 4 GB ou mais de VRAM para obter os melhores resultados.
* **Chloros+**[**CLI**](CLI.md)**Acesso**: execute o Chloros+ a partir da linha de comando para automatizar e integrar em seu próprio software.
* **Chloros+**[**API**](api-python-sdk.md)**Acesso:** execute o Chloros+ a partir do Python para controle programático, permitindo integração perfeita com seus pipelines de pesquisa, fluxos de trabalho de análise de dados e aplicativos personalizados.
* **Uso em vários dispositivos**: cada licença do Chloros+ permite o registro de 2 ou mais dispositivos. Use sua conta na nuvem do MAPIR para gerenciar os dispositivos registrados. Adicione suporte para mais dispositivos atualizando sua licença do Chloros+.
* **Método avançado de debayer com reconhecimento de textura:** um debayer de alta qualidade com reconhecimento de bordas combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do processo de debayering. 
* **Fórmulas personalizadas de índices multiespectrais:** insira índices multiespectrais personalizados nas calculadoras de raster do Chloros, tanto para processamento quanto para a área de teste de visualização de imagens.
* **Linux e computação de ponta:** execute o Chloros nas plataformas Linux x86_64 e ARM64, incluindo o NVIDIA Jetson, para processamento de campo e de ponta. Consulte [Visão geral do Linux](linux/linux-overview.md).

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary" data-icon="envira">Preços e inscrição do Chloros+</a></p>

<figure><img src=".gitbook/assets/plus_prog.JPG" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_zoom.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_mode.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_grid_meta.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/chloros_map_markers.gif" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>
