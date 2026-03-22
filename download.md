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
| **Internet**         | Necessária para a ativação da licença \[opcional] Chloros+ | Necessária para a ativação da licença \[opcional] Chloros+ |

#### Linux amd64 (x86\_64)

| Requisito       | Mínimo                    | Recomendado               |
| ----------------- | -------------------------- | ------------------------- |
| **Distribuição**  | Ubuntu 20.04+ / Debian 11+ | Ubuntu 22.04+             |
| **Processador**     | x86\_64 (Intel/AMD)        | Intel Core i7 ou superior   |
| **Memória (RAM)**  | 8 GB                        | 16 GB ou mais              |
| **Placa de vídeo** | Nenhuma (processamento pela CPU)      | GPU NVIDIA com 4 GB+ de VRAM |
| **Armazenamento**       | 2 GB de espaço livre             | SSD com 10 GB+ de espaço livre       |
| **Python**        | Python 3.7+ (para SDK)      | Python 3.10+              |

#### Linux arm64 (NVIDIA Jetson)

| Requisito      | Mínimo                      | Recomendado                     |
| ---------------- | ---------------------------- | ------------------------------- |
| **Plataforma**     | NVIDIA Jetson com JetPack 6 | Jetson Orin NX 16 GB ou AGX Orin |
| **Memória (RAM)** | 8 GB (GPU/CPU compartilhada)         | 16 GB+ compartilhada                    |
| **Armazenamento**      | 2 GB de espaço livre               | SSD NVMe com 10 GB+ de espaço livre        |
| **Python**       | Python 3.7+ (para SDK)        | Python 3.10+                    |

{% hint style="info" %}
**Aceleração por GPU**: Usuários do Chloros+ com GPUs NVIDIA podem usar a aceleração CUDA para um processamento significativamente mais rápido. Isso funciona tanto no Windows (GPUs de desktop) quanto no Linux (GPUs de desktop e NVIDIA Jetson). Os usuários do Chloros+ também ganham processamento multithread para velocidade máxima.
{% endhint %}

***

## Baixar Chloros

### Versão estável mais recente (23 de março de 2026): Versão 1.1.0

### <a href="https://drive.google.com/uc?export=download&#x26;id=1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4" class="button primary">Baixar Chloros para Windows (.exe)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1dB8-ke3wxNXpw_e1qJ4BhwBpCoNd4kLS" class="button primary">Baixar Chloros para Linux amd64 (.deb)</a>



### <a href="https://drive.google.com/uc?export=download&#x26;id=1d1OwdcYA4Rf4jkuPi2IBeWT2772_HnyO" class="button primary">Baixar Chloros para Linux arm64 / Jetson (.deb)</a>

#### Instalador Windows (GUI + CLI + Backend)

* **Tipo de arquivo**: .exe (Instalador Windows)**Etapas de instalação:**

1. Baixe o arquivo .exe acima
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\[USER]\Chloros\`)
5. Conclua a instalação e inicie o Chloros ou o Chloros CLI
6. Faça login com sua [conta MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

{% hint style="success" %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema para acesso via linha de comando.
{% endhint %}

#### Linux amd64 (Pacote .deb — CLI + Backend)

* **Tipo de arquivo**: .deb (pacote Debian/Ubuntu)
* **Arquitetura**: x86_64 (amd64)

```bash
sudo dpkg -i chloros-amd64.deb
chloros-cli --version  # Verify installation
```

#### Linux arm64 — NVIDIA Jetson (Pacote .deb — CLI + Backend)

* **Tipo de arquivo**: .deb (JetPack 6)
* **Arquitetura**: aarch64 (arm64)

```bash
sudo dpkg -i chloros-arm64-jp6.deb
chloros-cli --version  # Verify installation
```

Consulte [Instalação do Linux](linux/linux-installation.md) para obter instruções detalhadas de configuração e o [Guia do NVIDIA Jetson](linux/nvidia-jetson-guide.md) para orientações específicas sobre o Jetson.

#### Python SDK (Todas as plataformas)

```bash
pip install chloros-sdk
```

Consulte [API : Python SDK](api-python-sdk.md) para obter a documentação.

{% hint style="info" %}
**Usuários do Linux**: O pacote `.deb` instala o CLI e o backend. O Python SDK é instalado separadamente via pip. Não há GUI para o Linux — toda a interação é feita via CLI ou SDK.
{% endhint %}

***

## Recursos adicionais

### Python SDK

Para desenvolvedores e fluxos de trabalho de automação, instale o Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)**Requisitos**: O Chloros deve estar instalado (instalador do Windows ou pacote do Linux `.deb`), é necessário fazer login com a licença do Chloros+***

## O que está incluído

### Instalador Windows

* ✅ **Chloros GUI** - Interface gráfica com todos os recursos
* ✅ **Chloros CLI** - Interface de linha de comando (requer licença Chloros+)
* ✅ **Chloros Backend** - Mecanismo de processamento
* ✅ **Perfis de câmera** - Modelos de câmera MAPIR pré-configurados

### Pacote .deb Linux

* ✅ **Chloros CLI** - Interface de linha de comando (requer licença Chloros+)
* ✅ **Backend Chloros** - Mecanismo de processamento
* ✅ **Perfis de câmera** - Modelos de câmera MAPIR pré-configurados
* ❌ Sem GUI — Linux é apenas CLI/SDK sem interface gráfica

### Python SDK (pip, todas as plataformas)

* ✅ **Chloros SDK** - Python API (requer licença Chloros+)***

## Atualize para o Chloros+

Desbloqueie recursos avançados com uma assinatura do Chloros+:

* 🚀 **Processamento multithread** - Processe imagens em paralelo
* ⚡ **Aceleração por GPU (CUDA)** - Aproveite o poder da GPU NVIDIA
* 💻 **Acesso ao CLI** - Automatize com ferramentas de linha de comando
* 🐍 **Python SDK** - Acesso programático ao API
* 📱 **Vários dispositivos** - Use em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de debayer com reconhecimento de textura** - um debayer de alta qualidade com reconhecimento de bordas combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do debayer.
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Veja os planos e preços do Chloros+</a></p>***

## Ajuda para instalação

### Solução de problemas

**A instalação falha com a mensagem de erro:**

* Certifique-se de ter direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se você atende aos requisitos mínimos do sistema

**O aplicativo não inicia (Windows):**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os drivers de vídeo
* Verifique o Visualizador de Eventos do Windows para obter detalhes do erro
* Entre em contato com o suporte com os logs de erros

**O CLI não inicia (Linux):**

* Verifique se o pacote `.deb` está instalado corretamente: `dpkg -l | grep chloros`
* Verifique as permissões: `sudo chmod +x /usr/bin/chloros-cli`
* Execute o diagnóstico: `chloros-cli selftest`
* Verifique se há bibliotecas faltando: `ldd /usr/lib/chloros/chloros-backend | grep "not found"`

**Problemas com a ativação da licença:**

* Certifique-se de que a conexão com a Internet esteja ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se o firewall não está bloqueando o Chloros
* Consulte [Chloros+ Login](chloros+-login.md) para obter instruções detalhadas

### Obter suporte

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **FAQ**: [Perguntas frequentes](faq.md)***

## Registro de alterações

<details>

<summary>Versão 1.1.0 (Mais recente)</summary>

**Data de lançamento: março de 2026**

**Novos recursos*** **Suporte a Linux** — Suporte nativo a CLI e SDK para Linux amd64 (x86\_64) e arm64 (NVIDIA Jetson JetPack 6). Instale por meio dos pacotes `.deb`.
* **Suporte a NVIDIA Jetson** — Processamento otimizado para dispositivos de borda Jetson Nano, Orin Nano, Orin NX e AGX Orin.
* **Adaptação dinâmica de computação** — Detecção automática de hardware e otimização da estratégia de processamento. O Chloros se adapta ao seu hardware, desde um Jetson Nano até uma estação de trabalho com múltiplas GPUs.
* **Pipeline de processamento de 4 threads** — Threads simultâneos de detecção, calibração, processamento e exportação com alocação dinâmica de memória da GPU.
* **Novos comandos CLI** — `selftest` (diagnóstico do sistema) e `update` (gerenciamento de atualizações do Linux).
* **Novos sinalizadores de processo CLI** — `--debayer` (padrão/sensível à textura), `--indices` (especificar índices), `--target` (pesquisar primeiro na subpasta de destino para detecção mais rápida).
* **Novos itens do menu da GUI** — Adicionar arquivos, Adicionar pasta e Iniciar/Parar processamento agora acessíveis no menu suspenso principal.**Melhorias**

* Detecção automática de back-end multiplataforma (caminhos Windows e Linux)
* SDK e `get_status()` aprimorados com acompanhamento de progresso por thread
* Novas exceções SDK: `ChlorosConfigurationError`, `ChlorosAuthenticationError`
* Gerenciamento térmico e limitação adaptativa para NVIDIA Jetson
* Gerenciamento automático de memória com fallback OOM para processamento em mosaico da GPU

</details>

<details>

<summary>Versão 1.0.5</summary>

**Data de lançamento: 10 de fevereiro de 2026**

**Novos recursos*** **Método de debayer com reconhecimento de textura \[Somente Chloros+] -** O Texture Aware utiliza um debayer de alta qualidade com reconhecimento de bordas, combinado com um modelo de redução de ruído de IA/ML que remove quase todo o ruído do debayer.
* **Suporte para alvos de calibração T4P*** **Processamento de GPU Chloros+ mais rápido, melhor gerenciamento de memória**

**Correções de bugs*** Interface gráfica (GUI) totalmente nova, agora deve funcionar em todos os computadores Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

**Data de lançamento: 5 de janeiro de 2026**

**Novos recursos*** **Alternância entre imagem e metadados**: Adicionada opção no Navegador de Arquivos para visualizar os metadados da imagem selecionada em uma tabela, em vez da grade de imagens
* **Controle deslizante de zoom da grade de imagens**: Novo controle deslizante na interface para ajustar o tamanho das miniaturas (também suporta CTRL + roda do mouse)
* **Botões de exportação da grade de imagens**: Botões na linha superior para alternar as miniaturas de JPG para exportações processadas (Alvos, Refletância, Índice, LUT)
* **Guia Mapa**: Novo mapa 2D interativo exibindo marcadores de localização GPS das imagens
  * Compatível com Google Maps e blocos de mapa ESRI (seleciona automaticamente o melhor serviço de blocos com base na disponibilidade do nível de zoom)
  * Visualização da miniatura ao passar o mouse sobre os marcadores no mapa

**Correções de bugs*** Suporte aprimorado para a instalação do Chloros em computadores com idiomas diferentes do inglês

</details>

<details>

<summary>Versão 1.0.3</summary>

**Data de lançamento: 20 de dezembro de 2025**

**Novos recursos*** Lançamento inicial

**Melhorias*** Lançamento inicial

**Correções de bugs*** Lançamento inicial

**Problemas conhecidos*** Lançamento inicial

</details>***

## Contrato de Licença**Software Proprietário** - Copyright (c) 2026 MAPIR Inc.

É proibido o uso, a distribuição ou a modificação não autorizados.

**Versão gratuita**: Disponível para uso pessoal e comercial com limitações de recursos**Chloros+**: Licença por assinatura para recursos avançados e implantações comerciais
