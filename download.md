---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Baixe a versão mais recente doChloros

para começar a trabalhar com o processamento de imagens multiespectrais.

### Requisitos do sistema

####Windows



| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operacional** |Windows

10 (64 bits)                                  |Windows

11 (64 bits)                                  |
| **Processador**        | Intel Core i5 ou equivalente                          | Intel Core i7 ou superior                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB ou mais                                         |
| **Placa de vídeo**    | Compatível com DirectX 11                                | GPU NVIDIA com 4 GB ou mais de VRAM                            |
| **Armazenamento**          | 6 GB de espaço livre                                       | SSD com 10 GB ou mais de espaço livre                            |
| **Tela**          | 1920x1080                                            | 2560x1440 ou superior                                  |
| **Internet**         | Necessária para a ativação da licença do \[opcional]Chloros

+ | Necessária para a ativação da licença do \[opcional]Chloros

+ |

####Linux

amd64 (x86_64)

| Requisito       | Mínimo                    | Recomendado               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 22.04 LTS+ / Debian 12+ | Ubuntu 24.04 LTS      |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa de vídeo** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB ou mais de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB ou mais de espaço livre       |
| **Python**        |Python

3.7+ (paraSDK

)      |Python

3.10+              |

####Linux

arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (compartilhada entre GPU e CPU) | 16 GB+ compartilhada                    |
| **Armazenamento** | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       |Python

3.7+ (paraSDK

)        |Python

3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Usuários doChloros

e versões superiores com GPUs NVIDIA podem usar a aceleração CUDA para um processamento significativamente mais rápido. Isso funciona tanto noWindows

(GPUs de desktop) quanto noLinux

(GPUs de desktop e NVIDIA Jetson). Os usuários doChloros

+ também contam com processamento multithread para obter velocidade máxima.
{% endhint %}

***

## Baixe oChloros

### Versão estável mais recente: 1.2.0

<!-- NOLAN: replace installer links + release date for 1.2.0 — the three download buttons below still point at the 1.1.0 Google Drive files, and the release date needs to be added to the heading above. -->



### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Baixe o Chloros para Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Baixe o Chloros para Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Baixe Chloros para Linux arm64 / Jetson (.deb)</a>



#### Instalador doWindows

(GUI +CLI

+ Backend)

* **Tipo de arquivo**: .exe (Instalador doWindows

)

**Etapas de instalação:**

1. Baixe o arquivo .exe acima
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\MAPIR\Chloros\`)
5. Conclua a instalação e abraChloros

ouChloros

CLI

6. Faça login com sua [conta doMAPIR

CloudChloros

+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

{% hint style="success" %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema para acesso pela linha de comando.
{% endhint %}

####Linux

amd64 (Pacote .deb —CLI

+ Backend)

* **Tipo de arquivo**: .deb (pacote Debian/Ubuntu)
* **Arquitetura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

####Linux

arm64 — NVIDIA Jetson (pacote .deb —CLI

+ Backend)

* **Tipo de arquivo**: .deb (JetPack 6)
* **Arquitetura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consulte [Instalação doLinux

](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas sobre o Jetson.

####Python

SDK

(Todas as plataformas)

Cada instalador inclui um pacote “wheel” `chloros_sdk` correspondente, de modo que a versãoSDK

sempre corresponde à GUI/CLI

/backend instalado. EmWindows

, o instalador instala automaticamente no seu sistemaPython

; emLinux

, o `.deb` coloca o wheel em `/usr/lib/chloros/sdk/` e exibe o comando de instalação:

```bash
pip install --user /usr/lib/chloros/sdk/chloros_sdk-*.whl
```

Para hosts que utilizam apenas o pip (sem o pacoteChloros

instalado), oSDK

também está disponível no PyPI:

```bash
pip install chloros-sdk
```

Consulte [API

:Python

SDK

](api-python-sdk.md) e a [Referência doSDK

](reference/sdk-reference.md) para obter documentação.

{% hint style="info" %}
**Usuários doLinux**: O pacote `.deb` instala oCLI

e o backend. Não há interface gráfica (GUI) para oLinux

— toda a interação é feita por meio deCLI

ouSDK

.
{% endhint %}

***

## Recursos adicionais

###Python

SDK



Para desenvolvedores e fluxos de trabalho de automação, instale oChloros

Python

SDK

:

```bash
pip install chloros-sdk
```

**Documentação**: [API

:Python

SDK

](api-python-sdk.md)

**Requisitos**: É necessário ter oChloros

instalado (instaladorWindows

ou pacoteLinux

`.deb`), é necessário fazer login com licença emChloros

+

***

## O que está incluído

### InstaladorWindows



* ✅ **Interface gráfica doChloros** – Interface gráfica com todos os recursos
* ✅ **Chloros

CLI

** — Interface de linha de comando (requerChloros

+ licença)
* ✅ **BackendChloros** — Mecanismo de processamento
* ✅ **Perfis de câmera** — Modelos de câmera pré-configuradosMAPIR



### Pacote .deb doLinux



* ✅ **Chloros

CLI

** - Interface de linha de comando (requerChloros

+ licença)
* ✅ **Chloros

Backend** - Mecanismo de processamento
* ✅ **Perfis de câmera** - Modelos de câmera pré-configuradosMAPIR


* ❌ Sem GUI — oLinux

é headless, apenasCLI

/SDK



###Python

SDK

(pip, todas as plataformas)

* ✅ **Chloros

SDK

** -Python

API

(requer licençaChloros

+)

***

## Atualize paraChloros

+

Desbloqueie recursos avançados com uma assinatura doChloros

+:

* 🚀 **Processamento multithread** — Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** — Aproveite o poder da GPU da NVIDIA
* 💻 **Acesso aoCLI** — Automatize com ferramentas de linha de comando
* 🐍 **Python

SDK

** — Acesso programático aoAPI


* 📱 **Vários dispositivos** — Use em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de debayering com reconhecimento de textura** — um debayering de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que remove quase todo o ruído do processo de debayering.
* 🧮 **Fórmulas personalizadas** — Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Veja Chloros+ Planos e preços</a></p>***

## Ajuda com a instalação

### Solução de problemas

**A instalação falha com a mensagem de erro:**

* Verifique se você possui direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se você atende aos requisitos mínimos do sistema

**O aplicativo não inicia (Windows

):**

* Verifique se oWindows

10/11 (64 bits) está instalado
* Atualize os drivers de vídeo
* Verifique o Visualizador de Eventos emWindows

para obter detalhes do erro
* Entre em contato com o suporte enviando os logs de erro

**OCLI

não inicia (Linux

):**

* Verifique se o pacote `.deb` foi instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se há bibliotecas ausentes: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas com a ativação da licença:**

* Certifique-se de que a conexão com a internet esteja ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se o firewall não está bloqueandoChloros

* Consulte [Chloros

+ Login](chloros+-login.md) para obter instruções detalhadas

### Obter suporte

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)***

## Atualizações de software O

Chloros

verifica se há atualizações, avisa quando uma nova versão está disponível e fornece um link para esta página de download — você atualiza executando o novo instalador assinado. Suas configurações e projetos são mantidos após as atualizações. NoLinux

e no Jetson, o comando ``chloros-cli update`` verifica se há uma versão mais recente e oferece a opção de baixar e instalar o `.deb` correspondente (esse comando é exclusivo dLinux

).

***

## Registro de alterações**Versão 1.2.0 (Mais recente)**— consulte**Novidades noChloros

1.2.0** na página [Introdução](./) para obter a lista completa de recursos.

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novos recursos*** **Método de debayer com reconhecimento de textura \[Chloros

+ apenas] -** O Texture Aware utiliza um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído baseado em IA/ML que remove quase todo o ruído do processo de debayering.
* **Suporte para alvos de calibração T4P*** **Chloros

mais rápido + processamento por GPU, melhor gerenciamento de memória**

**Correções de bugs*** Interface de usuário (GUI) totalmente nova; agora deve funcionar em todos os computadoresWindows

.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novos recursos*** **Alternância entre imagem e metadados**: Adicionada opção no Navegador de Arquivos para visualizar os metadados da imagem selecionada em uma tabela, em vez da grade de imagens
* **Controle deslizante de zoom da grade de imagens**: Novo controle deslizante na interface para ajustar o tamanho das miniaturas (também suporta CTRL + roda do mouse)
* **Botões de exportação da grade de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Guia “Mapa”**: Novo mapa 2D interativo que mostra marcadores de localização GPS nas imagens
  * Compatível com o Google Maps e blocos de mapa da ESRI (seleciona automaticamente o melhor serviço de blocos com base na disponibilidade do nível de zoom)
  * Visualização das miniaturas ao passar o mouse sobre os marcadores no mapa

**Correções de bugs*** Suporte aprimorado para a instalação do Chloros em computadores que não estejam configurados para o idioma inglês

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
