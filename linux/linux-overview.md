# Visão geral do Linux

O Chloros 1.1.0 oferece suporte nativo ao Linux para o **CLI**e o**Python SDK**, permitindo o processamento de imagens multiespectrais sem interface gráfica em estações de trabalho, servidores e dispositivos de borda NVIDIA Jetson.

{% hint style="info" %}
**Sem GUI no Linux.** A GUI do Chloros Desktop está disponível apenas no Windows. Os usuários do Linux interagem com o Chloros por meio do [CLI](../CLI.md) e do [Python SDK](../api-python-sdk.md).
{% endhint %}

***

## Matriz de suporte de plataformas

| Recurso | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI de desktop** | Sim | N/A | Não | Não |
| **CLI** | Sim | Sim | Sim | Sim |
| **Python SDK** | Sim | Sim | Sim | Sim |
| **Aceleração por GPU (CUDA)** | Sim | Sim | Sim | Sim (JetPack 6) |
| **Debayer com reconhecimento de textura** | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) |
| **Adaptação dinâmica de computação** | Sim | Sim | Sim | Sim |***

## Arquiteturas compatíveis

| Arquitetura | Descrição | Método de instalação |
| --- | --- | --- |
| **amd64 (x86_64)** | Processadores padrão para desktop/servidor (Intel, AMD) | Pacote `.deb` |
| **arm64 (aarch64)** | Processadores baseados em ARM, principalmente NVIDIA Jetson | Pacote `.deb` (JetPack 6) |

## Distribuições Linux compatíveis

* **Ubuntu 20.04+** (amd64)
* **Debian 11+** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson)***

## O que os usuários do Linux obtêm

* **Chloros CLI** — Interface de linha de comando completa para processamento em lote, automação e scripts
* **Chloros Python SDK** — Interface programática Python (`pip install chloros-sdk`) para integração em pipelines de pesquisa e ferramentas personalizadas
* **Aceleração por GPU** — Processamento acelerado por CUDA em GPUs NVIDIA (desktop e Jetson)
* **Adaptação Dinâmica de Computação** — Detecção automática de hardware e otimização da estratégia de processamento
* **Todos os Recursos de Processamento** — O mesmo pipeline de processamento multiespectral do Windows (calibração, correção de vinheta, índices de vegetação, todos os formatos de exportação)
* **Recursos do Chloros+** — Processamento multithread, debayer sensível à textura, índices personalizados (com licença do Chloros+)

## O que os usuários do Linux não têm

* **GUI de desktop** — Sem interface gráfica; toda a interação é feita via CLI ou Python SDK
* **Visualizador de imagens** — Sem visualizador de imagens interativo, visualização em grade ou marcadores de mapa
* **Gerenciamento visual de projetos** — Os projetos são gerenciados por meio de comandos CLI e chamadas SDK***

## Introdução ao Linux

1. **Instale o Chloros** — Consulte [Instalação do Linux](linux-installation.md) para a instalação do pacote `.deb`
2. **Instale o Python SDK** (opcional) — `pip install chloros-sdk`
3. **Ative sua licença** — `chloros-cli login your@email.com 'password'`
4. **Processe seu primeiro conjunto de dados** — `chloros-cli process ~/datasets/flight001`

Para usuários do NVIDIA Jetson, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) dedicado para configuração e otimização específicas da plataforma.

***

## Próximos passos

* [Instalação do XPROTX](linux-installation.md) — Instruções detalhadas de instalação para amd64 e arm64
* [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) — Configuração específica para Jetson, gerenciamento térmico e implantação em campo
* [CLI : Linha de comando](../CLI.md) — Referência completa do CLI
* [API : Python SDK](../api-python-sdk.md) — Referência completa do SDK
* [Adaptação dinâmica de computação](../processing-architecture/dynamic-compute-adaptation.md) — Como o Chloros se adapta ao seu hardware
