# Visão geral do Linux

O Chloros 1.3.0 oferece suporte nativo ao Linux para a **CLI**e o**Python SDK** — processamento de imagens multiespectrais sem interface gráfica, além de controle em tempo real de câmeras LATTICE e sensores de luz DAQ — em estações de trabalho, servidores e dispositivos de borda NVIDIA Jetson com Linux.

{% hint style="info" %}
**Não há GUI para desktop no Linux.**A GUI do Chloros Desktop é exclusiva para Windows. Os usuários do Linux interagem com o Chloros por meio da [CLI](../CLI.md) e do [Python SDK](../api-python-sdk.md). O `.deb` adiciona uma entrada**Chloros CLI** ao menu de aplicativos — ele simplesmente abre um emulador de terminal executando o `chloros-cli`.
{% endhint %}

***

## Matriz de compatibilidade de plataformas

| Recurso | Windows (GUI) | Windows (CLI/SDK) | Linux amd64 (CLI/SDK) | Linux arm64 / Jetson (CLI/SDK) |
| --- | --- | --- | --- | --- |
| **GUI para desktop** | Sim | N/A | Não | Não |
| **Interface de linha de comando (CLI)** (`chloros-cli`) | Sim | Sim | Sim | Sim |
| **SDK Python para Python** (`chloros-sdk`) | Sim | Sim | Sim | Sim |
| **Pipeline de processamento de imagem** | Sim | Sim | Sim | Sim |
| **Controle da câmera LATTICE (ao vivo)** | Sim (guia Câmeras) | Sim (`chloros-cli lattice`, SDK) | Sim | Sim |
| **Sensores de luz DAQ (ao vivo)** | Sim (guia “Sensores de luz”) | Sim (`chloros-cli daq pool-*`, SDK) | Sim | Sim |
| **Sincronização de tempo PTP (o host é o grandmaster)** | Sim | Sim (`chloros-cli time-sync`) | Sim | Sim |
| **Aceleração por GPU (CUDA)** | Sim | Sim | Sim | Sim (JetPack 6) |
| **Debayer com reconhecimento de textura** (requer uma GPU com CUDA) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) | Sim (Chloros+) |
| **Adaptação dinâmica de computação** | Sim | Sim | Sim | Sim |
| **Backend como serviço do sistema** (`chloros-backend@<user>.service`) | Não | Não | Sim (opcional) | Sim (opcional) |
| **Atualizador no local** (`chloros-cli update`) | Não (execute o instalador) | Não (execute o instalador) | Sim | Sim |

***

## Arquiteturas compatíveis

| Arquitetura | Descrição | Pacote |
| --- | --- | --- |
| **amd64 (x86_64)** | Processadores padrão para desktop/servidor (Intel, AMD) | `chloros_<version>_amd64.deb` |
| **arm64 (aarch64)** | Processadores ARM — família NVIDIA Jetson Orin (JetPack 6) | `chloros_<version>_arm64_jp6.deb` |

## Distribuições do Linux compatíveis

* **Ubuntu 22.04 LTS ou mais recente** (amd64)
* **Debian 12 ou mais recente** (amd64)
* **NVIDIA JetPack 6** (arm64 — plataformas Jetson Orin)

***

## O que os usuários do Linux obtêm

* **Chloros CLI** — a interface completa de linha de comando para processamento em lote, automação e criação de scripts
* **Chloros Python SDK** — interface programática em Python para pipelines de pesquisa e ferramentas personalizadas, instalada pelo `.deb` para o sistema `python3` (um wheel com versão compatível para virtualenvs está no `/usr/lib/chloros/sdk/`; o PyPI é para máquinas sem o pacote Chloros)
* **Controle de câmeras LATTICE** — descubra, conecte, configure e capture imagens de câmeras LATTICE e conjuntos sincronizados de múltiplas câmeras por meio do `chloros-cli lattice` e do SDK; o `.deb` inclui o runtime do SDK Arena necessário para as câmeras
* **Controle de sensores de luz DAQ** — conecte sensores DAQ-U/M/E, transmita espectros calibrados e grave arquivos `.daq` por meio do `chloros-cli daq pool-*` e do SDK
* **Sincronização de tempo PTP** — o backend do Chloros executa o grandmaster PTP ao qual as câmeras LATTICE e os sensores DAQ-E se submetem; verifique-o com o `chloros-cli time-sync` e mantenha-o em execução sem interface gráfica com a unidade systemd `chloros-backend@<user>.service` (consulte [Instalação no Linux](linux-installation.md#always-on-ptp-for-headless-hosts))
* **Automação de projetos** — execute projetos salvos em modo headless com o `chloros-cli project` e o `open_project` do SDK
* **Aceleração por GPU** — processamento acelerado por CUDA em GPUs NVIDIA (GPUs de desktop em amd64 e a GPU do Jetson com o JetPack 6)
* **Adaptação dinâmica de computação** — detecção automática de hardware e seleção da estratégia de processamento, com a opção de substituição `CHLOROS_STRATEGY` como uma saída de emergência para especialistas
* **Todos os recursos de processamento** — o mesmo fluxo de trabalho do Windows: calibração, correção de vinheta, índices de vegetação e todos os formatos de exportação
* **Recursos do Chloros+** — processamento multithread (em pipeline), debayer com reconhecimento de textura (em uma GPU CUDA) e índices personalizados, com um plano pago do Chloros+

## O que os usuários do Linux não têm

* **GUI para desktop** — sem interface gráfica; toda a interação ocorre via CLI ou Python SDK
* **Visualizador de imagens** — sem visualizador de imagens interativo, visualização em grade ou marcadores de mapa
* **Gerenciamento visual de projetos** — os projetos são criados e executados por meio de comandos de CLI e chamadas de SDK (o próprio hardware — câmeras, sensores, captura — permanece totalmente controlável a partir do terminal)

***

## Requisitos de licença

O acesso à CLI e ao SDK requer um **plano pago do Chloros+ — Copper ou superior**(Copper, Bronze, Silver, Gold). O plano gratuito**Iron** não oferece acesso à CLI/SDK. Essa restrição é aplicada pelo back-end, não apenas pela CLI:

| Situação | Resposta do back-end |
| --- | --- |
| Não conectado | `401` com `error_code: AUTH_REQUIRED` |
| Conectado no plano gratuito Iron | `403` com `error_code: PLAN_UPGRADE_REQUIRED` |

O `chloros-cli status` funciona em qualquer plano — é a única rota isenta do filtro —, portanto, o motivo da recusa está sempre visível.

***

## Introdução no Linux

1. **Instale o Chloros** — consulte [Instalação no Linux](linux-installation.md) para a instalação do `.deb`
2. **Verifique** — o `chloros-cli --version` exibe o `Chloros CLI 1.3.0`; o `chloros-cli selftest` executa o diagnóstico de 7 etapas
3. **Verifique o SDK Python** — o `.deb` o instala: `python3 -c "import chloros_sdk; print(chloros_sdk.__version__)"` (em um virtualenv, `pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl`)
4. **Faça login** — `chloros-cli login your@email.com 'your-password'` (uma vez por máquina; as atualizações do pacote mantêm o login, enquanto a remoção do pacote o apaga)
5. **Processe seu primeiro conjunto de dados** — `chloros-cli process ~/datasets/flight001`

Para o NVIDIA Jetson, consulte o [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) dedicado para configuração específica da plataforma, comportamento térmico e implantação em campo.

***

## Próximos passos

* [Instalação no Linux](linux-installation.md) — instalação detalhada, localização de arquivos e solução de problemas para amd64 e arm64
* [Guia do NVIDIA Jetson](nvidia-jetson-guide.md) — configuração específica para Jetson, comportamento térmico e de memória, implantação em campo
* [CLI: Linha de comando](../CLI.md) — o guia da CLI
* [API: Python SDK](../api-python-sdk.md) — o guia do SDK
* [Referência da CLI](../reference/cli-reference.md) e [Referência do SDK](../reference/sdk-reference.md) — listas completas de comandos/API para a versão 1.3.0
* [Adaptação Dinâmica de Computação](../processing-architecture/dynamic-compute-adaptation.md) — como o Chloros se adapta ao seu hardware

{% hint style="info" %}
**Lendo este manual programaticamente.** Cada página também é disponibilizada como Markdown bruto em seu próprio URL, acrescentando-se `.md` (por exemplo, `https://mapir.gitbook.io/chloros/linux/linux-installation.md`), e um índice de todo o manual está publicado em [`https://mapir.gitbook.io/chloros/llms.txt`](https://mapir.gitbook.io/chloros/llms.txt).
{% endhint %}
