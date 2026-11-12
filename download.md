---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Baixe a versão mais recente do Chloros para começar a trabalhar com o processamento de imagens multiespectrais.

### Requisitos do sistema

#### Windows


| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operacional** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processador**        | Intel Core i5 ou equivalente                          | Intel Core i7 ou superior                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB ou mais                                         |
| **Placa de vídeo**    | Compatível com DirectX 11                                | GPU NVIDIA com 4 GB ou mais de VRAM                            |
| **Armazenamento**          | 6 GB de espaço livre                                       | SSD com 10 GB ou mais de espaço livre                            |
| **Tela**          | 1920x1080                                            | 2560x1440 ou superior                                  |
| **Internet**         | Necessária para a ativação da licença [opcional] do Chloros+ | Necessária para a ativação da licença [opcional] do Chloros+ |

#### Linux amd64 (x86_64)

| Requisitos       | Mínimo                    | Recomendado               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa de vídeo** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB ou mais de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB ou mais de espaço livre       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX de 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (compartilhada entre GPU e CPU)         | 16 GB ou mais compartilhada                    |
| **Armazenamento**      | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Usuários do Chloros+ com GPUs NVIDIA podem usar a aceleração CUDA para um processamento significativamente mais rápido. Isso funciona com GPUs de desktop da NVIDIA no Windows e no Linux amd64, e no NVIDIA Jetson com o JetPack 6. Os usuários do Chloros+ também contam com processamento multithread para velocidade máxima.
{% endhint %}

***

## Baixar o Chloros


### Versão estável mais recente: Versão 1.3.0 (11 de novembro de 2026)

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Baixe o Chloros para Windows (.exe)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Baixe o Chloros para Linux amd64 (.deb)</a>


### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Baixar o Chloros para Linux arm64 / Jetson (.deb)</a>


#### Instalador do Windows (GUI + CLI + Backend)

* **Tipo de arquivo**: .exe (Instalador do Windows)

**Etapas de instalação:**

1. Baixe o arquivo .exe acima
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\Chloros\`)
5. Na página **Selecionar tarefas adicionais**, mantenha ambas as opções de**Configurações do sistema** marcadas (recomendado — veja abaixo)
6. Conclua a instalação e inicie o Chloros ou o Chloros CLI
7. Faça login com sua [conta MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

**Opções de configurações do sistema** (ambas marcadas por padrão):

| Opção, conforme descrita no instalador | O que ela faz | Desativar instalação silenciosa |
| --- | --- | --- |
| **Ativar caminhos longos do Windows** (permite que as pastas de projeto tenham nomes com mais de 260 caracteres; recomendado) | Define a configuração `LongPathsEnabled` do Windows, válida para todo o sistema, como 1. O Windows vem com essa opção desativada, e, nesse caso, nenhum caminho de arquivo pode ter mais de 260 caracteres; os produtos Chloros ficam até cerca de 130 caracteres abaixo da pasta do projeto, portanto, com a opção de caminhos longos desativada, uma pasta de projeto mais profunda é rejeitada. A configuração entra em vigor para programas iniciados após a instalação. Ela permanece ativada após a desinstalação, pois outros programas podem depender dela. | `/MERGETASKS="!longpaths"` |
| **Excluir o backend do Chloros da verificação do Microsoft Defender** (captura de câmera muito mais rápida; recomendado) | Adiciona uma exclusão de processo do Microsoft Defender para `chloros-backend.exe` na pasta de instalação, de modo que a proteção em tempo real não verifique todos os arquivos que o backend grava, independentemente de onde ele os grave. Em altas taxas de captura, a verificação consumia cerca de 30% da taxa de captura (77,7 contra 52,7 capturas por segundo em uma captura contínua com 9 câmeras). Nenhum outro programa é excluído, e a exclusão é removida na desinstalação. Quando o Defender estiver ausente ou bloqueado por política, esta etapa falha silenciosamente e a instalação continua. | `/MERGETASKS="!defenderexclusion"` |

Uma instalação silenciosa (`/SILENT` ou `/VERYSILENT`) aplica ambas as opções, a menos que você opte por não participar com o valor `/MERGETASKS` exibido.

{% hint style="success" %}
O instalador adiciona a pasta CLI (`C:\Program Files\Chloros\cli\`) ao seu sistema `PATH` uma única vez — a reinstalação ou atualização não acrescenta mais uma entrada duplicada e remove as duplicatas adicionadas por instaladores anteriores. Abra um novo terminal após a instalação para usar o `chloros-cli`. O instalador também adiciona duas regras do Firewall do Windows para o backend, em todos os perfis de rede: **Backend do Chloros (UDP-In)**, para transmissões de câmera, PTP, mDNS e o fluxo de dados do DAQ-E, e**Backend do Chloros (TCP-In)**, limitado à sub-rede local, para uma atualização de firmware do DAQ-E, na qual o sensor se conecta de volta a este computador para coletar a imagem. Ambas são removidas na desinstalação.
{% endhint %}

#### Linux amd64 (pacote .deb — CLI + back-end)

* **Tipo de arquivo**: .deb (pacote Debian/Ubuntu)
* **Arquitetura**: x86\_64 (amd64)

```bash
sudo apt install ./chloros_1.3.0_amd64.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

#### Linux arm64 — NVIDIA Jetson (pacote .deb — CLI + Backend)

* **Tipo de arquivo**: .deb (JetPack 6)
* **Arquitetura**: aarch64 (arm64)

```bash
sudo apt install ./chloros_1.3.0_arm64_jp6.deb
chloros-cli --version     # Verify installation: "Chloros CLI 1.3.0"
```

Use o nome do arquivo que você realmente baixou e mantenha o `./` para que o `apt` instale esse arquivo em vez de procurar um pacote pelo nome. Instale com o `apt install`, não com o `dpkg -i`: o apt também instala os pré-requisitos do Python SDK, que o pacote recomenda em vez de exigir, e o `dpkg -i` os ignora. Desde a versão 1.3.0, o backend e a CLI são instalados descompactados no diretório `/usr/lib/chloros/`, de modo que são iniciados sem precisar se descompactar primeiro.

O pacote Jetson usa a versão do PyTorch compilada com CUDA do JetPack 6; portanto, o processing — incluindo o debayer Texture Aware — é executado na GPU do Jetson. Ele utiliza as bibliotecas CUDA e cuDNN instaladas pelo JetPack 6 (`sudo apt install nvidia-jetpack` em uma imagem que não as possua).

Consulte [Instalação no Linux](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas sobre o Jetson.

#### Python SDK (Todas as plataformas)

Cada instalador inclui um wheel `chloros_sdk` correspondente, de modo que a versão do SDK sempre corresponde à GUI/CLI/backend instalada. No Windows, o instalador o instala automaticamente no Python do seu sistema. No Linux, o `.deb` instala o SDK para o próprio `python3` do sistema (no `/usr/lib/python3/dist-packages`), de modo que o `import chloros_sdk` funciona logo após a instalação, sem pip e sem conexão de rede. Apenas um virtualenv, que não tem acesso aos pacotes do sistema, precisa que o wheel incluído seja instalado nele:

```bash
pip install /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para hosts que utilizam apenas o pip (sem pacote Chloros instalado; o SDK, nesse caso, aciona um backend em outra máquina), o SDK também está disponível no PyPI:

```bash
pip install chloros-sdk
```

Consulte [API: Python SDK](api-python-sdk.md) e a [Referência do SDK](reference/sdk-reference.md) para obter a documentação.

{% hint style="info" %}
**Usuários do Linux**: O pacote `.deb` instala a CLI, o backend e o SDK do Python. Não há GUI para o Linux — toda a interação ocorre via CLI ou SDK.
{% endhint %}

***

## Recursos adicionais

### SDK Python para Chloros


Para desenvolvedores e fluxos de trabalho de automação, o Chloros Python SDK vem com todos os instaladores: o instalador do Windows o instala no seu Python, e o `.deb` do Linux o instala para o sistema `python3`. Em uma máquina sem o pacote Chloros (aquela que controla um backend em outra máquina), instale-o a partir do PyPI:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)

**Requisitos**: um backend do Chloros para ser acionado (o instalador do Windows ou o pacote `.deb` para Linux instala um) e um login no Chloros+ em um plano pago

***

## O que está incluído

### Instalador do Windows


* ✅ **GUI do Chloros** — Interface gráfica completa
* ✅ **CLI do Chloros** — Interface de linha de comando (requer licença do Chloros+)
* ✅ **Backend do Chloros** — Mecanismo de processamento
* ✅ **Python SDK** — instalado no seu Python, caso esteja presente (requer licença do Chloros+)
* ✅ **Perfis de câmera** — modelos de câmera MAPIR pré-configurados

### Pacote .deb para Linux


* ✅ **Chloros CLI** — Interface de linha de comando (requer licença Chloros+)
* ✅ **Chloros Backend** — Mecanismo de processamento
* ✅ **Python SDK** — instalado para o sistema `python3` (requer licença Chloros+)
* ✅ **Perfis de câmera** — Modelos de câmera MAPIR pré-configurados
* ❌ Sem GUI — o Linux é apenas CLI/SDK sem interface gráfica

### Python SDK (PyPI, para máquinas sem o pacote Chloros)

* ✅ **Chloros SDK** — Python API (requer licença Chloros+)

***

## Atualize para o Chloros+

Desbloqueie recursos avançados com uma assinatura do Chloros+:

* 🚀 **Processamento multithread** — Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** - Aproveite o poder da GPU da NVIDIA
* 💻 **Acesso à CLI** - Automatize com ferramentas de linha de comando
* 🐍 **Python SDK** - Acesso programático à API

* 📱 **Vários dispositivos** - Use em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de remoção de pixelização com reconhecimento de textura** — um método de remoção de pixelização de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que elimina praticamente todo o ruído da remoção de pixelização.
* 🧮 **Fórmulas personalizadas** — Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Veja os planos e preços do Chloros+</a></p>

***

## Ajuda com a instalação

### Solução de problemas

**A instalação falha com a seguinte mensagem de erro:**

* Certifique-se de ter direitos de administrador
* Desative temporariamente o antivírus
* Verifique se você atende aos requisitos mínimos do sistema

**O aplicativo não inicia (Windows):**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os drivers de vídeo
* Verifique o Visualizador de Eventos do Windows para obter detalhes do erro
* Entre em contato com o suporte enviando os logs de erro

**A CLI não inicia (Linux):**

* Verifique se o pacote `.deb` foi instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se há bibliotecas ausentes: `ldd /usr/lib/chloros/backend/chloros-backend | grep "not found"` (a partir da versão 1.3.0, o `/usr/lib/chloros/chloros-backend` é um pequeno script de inicialização; o programa em si está no `/usr/lib/chloros/backend/`)

**Problemas com a ativação da licença:**

* Certifique-se de que a conexão com a internet esteja ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se o firewall não está bloqueando o Chloros

* Consulte [Chloros+ Login](chloros+-login.md) para obter instruções detalhadas

### Obtenha suporte

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)

***

## Atualizações de software O

Chloros verifica se há atualizações, avisa quando uma nova versão está disponível e fornece um link para esta página de download — você atualiza executando o novo instalador assinado. Suas configurações, projetos e dados de login são mantidos após as atualizações. No Linux e no Jetson, o comando ``chloros-cli update`` verifica se há uma versão mais recente e oferece a opção de baixar e instalar a versão correspondente ``.deb`` (esse comando é exclusivo para Linux).

***

## Registro de alterações**Versão 1.3.0 (Mais recente, 11 de novembro de 2026)**— consulte**Novidades no Chloros

1.3.0** na página [Introdução](./) para ver a lista completa.**Versão 1.2.0**— consulte**Novidades no Chloros 1.2.0** na página [Introdução](./).

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novos recursos*** **Método de debayer com reconhecimento de textura \[Somente Chloros+] -** O Texture Aware utiliza um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que remove quase todo o ruído do processo de debayer.
* **Suporte para alvos de calibração T4P*** **Processamento mais rápido do Chloros+ na GPU, melhor gerenciamento de memória**

**Correções de bugs*** Interface gráfica (GUI) totalmente nova; agora deve funcionar em todos os computadores com Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novos recursos*** **Alternância entre imagem e metadados**: Adicionada uma opção no Navegador de Arquivos para visualizar os metadados da imagem selecionada em uma tabela, em vez da grade de imagens
* **Controle deslizante de zoom da grade de imagens**: Novo controle deslizante na interface do usuário para ajustar o tamanho das miniaturas (também suporta CTRL + roda do mouse)
* **Botões de exportação da grade de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Guia Mapa**: Novo mapa 2D interativo exibindo marcadores de localização GPS das imagens
  * Compatível com o Google Maps e blocos de mapa da ESRI (seleciona automaticamente o melhor serviço de blocos com base na disponibilidade do nível de zoom)
  * Visualização das miniaturas ao passar o mouse sobre os marcadores no mapa

**Correções de bugs*** Suporte aprimorado para a instalação do Chloros em computadores com idiomas diferentes do inglês

</details>

<details>

<summary>Versão 1.0.3</summary>

**Data de lançamento: 20 de dezembro de 2025**

**Novos recursos*** Lançamento inicial

**Melhorias*** Lançamento inicial

**Correções de bugs*** Lançamento inicial

**Problemas conhecidos*** Lançamento inicial

</details>

***

## Contrato de Licença**Software proprietário** - Copyright (c) 2026 MAPIR Inc.

É proibido o uso, a distribuição ou a modificação não autorizados.

**Versão gratuita**: Disponível para uso pessoal e comercial com limitações de recursos**Chloros+**: Licença por assinatura para recursos avançados e implantações comerciais
