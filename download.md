---
metaLinks:
  alternates:
    - https://app.gitbook.com/s/o044KN3Ws0uIDvOmSkcR/download
---

# Download

Faça o download da versão mais recente do Chloros para começar a usar o processamento de imagens multiespectrais.

### Requisitos do sistema

| Requisito          | Mínimo                                              | Recomendado                                          |
| -------------------- | ---------------------------------------------------- | ---------------------------------------------------- |
| **Sistema operacional** | Windows 10 (64 bits)                                  | Windows 11 (64 bits)                                  |
| **Processador**        | Intel Core i5 ou equivalente                          | Intel Core i7 ou superior                              |
| **Memória (RAM)**     | 8 GB                                                  | 16 GB ou mais                                         |
| **Placa gráfica**    | Compatível com DirectX 11                                | GPU NVIDIA com 4 GB+ de VRAM                            |
| **Armazenamento**          | 6 GB de espaço livre                                       | SSD com 10 GB+ de espaço livre                            |
| **Tela**          | 1920x1080                                            | 2560x1440 ou superior                                  |
| **Internet**         | Necessária para ativação da licença \[opcional] Chloros+ | Necessária para ativação da licença \[opcional] Chloros+ |

{% hint style="info" %}
**Aceleração GPU**: Os usuários do Chloros+ com GPUs NVIDIA podem usar a aceleração CUDA para um processamento significativamente mais rápido. Os usuários do Chloros+ também ganham processamento multithread para velocidade máxima.
{% endhint %}

***

## Baixar Chloros

### <a href="https://drive.google.com/file/d/1HjwrUY4M7HGxDbMybO7iPe_6JoHnUGr4/view?usp=drive_link" class="button primary">Baixar Chloros aqui</a>

### Última versão estável

**Chloros Instalador para Windows*** **Versão**: 1.0.5
* **Data de lançamento**: 10 de fevereiro de 2026
* **Tamanho do arquivo (download)**: 1,6 GB
* **Tamanho do arquivo (instalado)**: 5,7 GB
* **Tipo de arquivo**: .exe (Instalador Windows)

#### **Etapas de instalação:**

1. Baixe o arquivo `CHLOROS INSTALLER - CURRENT VERSION.exe`
2. Clique duas vezes no instalador para iniciar a instalação
3. Siga as instruções do assistente de instalação
4. Escolha o diretório de instalação (padrão: `C:\Program Files\[USER]\Chloros\`)
5. Conclua a instalação e inicie o Chloros ou Chloros CLI
6. Faça login com sua conta [MAPIR Cloud Chloros+](https://cloud.mapir.camera/pricing) (ou continue com a versão gratuita)

{% hint style="success" %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema para acesso à linha de comando.
{% endhint %}

***

## Recursos adicionais

### Python SDK

Para desenvolvedores e fluxos de trabalho de automação, instale o Chloros Python SDK:

```bash
pip install chloros-sdk
```

**Documentação**: [API: Python SDK](api-python-sdk.md)**Requisitos**: Chloros Desktop deve estar instalado, Chloros+ login de licença necessário***

## O que está incluído

A instalação do Chloros inclui:

* ✅ **Chloros** - Interface gráfica completa (GUI)
* ✅ **Chloros CLI** - Interface de linha de comando (requer licença Chloros+)
* ✅ **Chloros SDK** - Python API (requer licença Chloros+)
* ✅ **Perfis de câmera** - Modelos de câmera MAPIR pré-configurados***

## Atualize para Chloros+

Desbloqueie recursos avançados com uma assinatura Chloros+:

* 🚀 **Processamento multithread** - Processe imagens em paralelo
* ⚡ **Aceleração GPU (CUDA)** - Aproveite o poder da GPU NVIDIA
* 💻 **Acesso CLI** - Automatize com ferramentas de linha de comando
* 🐍 **Python SDK** - Acesso programático ao API
* 📱 **Vários dispositivos** - Use em 2 a 10 ou mais dispositivos (dependendo do plano)
* **🐻 Método avançado de debayer com reconhecimento de textura** - um debayer de alta qualidade com reconhecimento de bordas combinado com um modelo de redução de ruído AI/ML que remove quase todo o ruído do debayering. 
* 🧮 **Fórmulas personalizadas** - Crie índices multiespectrais personalizados

<p align="center"><a href="https://cloud.mapir.camera/pricing" class="button primary">Veja os planos e preços do Chloros+</a></p>***

## Ajuda para instalação

### Solução de problemas

**A instalação falha com a mensagem de erro:**

* Certifique-se de que você tem direitos de administrador
* Desative temporariamente o software antivírus
* Verifique se você atende aos requisitos mínimos do sistema

**O aplicativo não inicia:**

* Verifique se o Windows 10/11 (64 bits) está instalado
* Atualize os drivers gráficos
* Verifique o Windows Visualizador de Eventos para obter detalhes sobre o erro
* Entre em contato com o suporte com os registros de erros

**Problemas com a ativação da licença:**

* Certifique-se de que a conexão com a Internet esteja ativa
* Verifique as credenciais em [https://cloud.mapir.camera](https://cloud.mapir.camera)
* Verifique se o firewall não está bloqueando o Chloros
* Consulte [Chloros+ Login](chloros+-login.md) para obter instruções detalhadas

### Obtendo suporte

Precisa de ajuda com a instalação ou configuração?

* 📧 **E-mail**: info@mapir.camera
* 🌐 **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* 📚 **Documentação**: [Introdução](./)
* ❓ **Perguntas frequentes**: [Perguntas frequentes](faq.md)***

## Registro de alterações

<details>

<summary>Versão 1.0.5</summary>

#### **Data de lançamento**: 10 de fevereiro de 2026**Novos recursos*** **Método Debayer com reconhecimento de textura \[Chloros+ Somente] -** O reconhecimento de textura usa um debayer de alta qualidade com reconhecimento de bordas combinado com um modelo de redução de ruído AI/ML que remove quase todo o ruído do debayering.
* **Suporte para alvos de calibração T4P*** **Processamento mais rápido da GPU Chloros+, melhor gerenciamento de memória**

**Correções de bugs*** Front-end (GUI) completamente novo, agora deve funcionar em todos os computadores Windows.

</details>

<details>

<summary>Versão 1.0.4</summary>

#### **Data de lançamento**: 5 de janeiro de 2026**Novos recursos*** **Alternância de imagem/metadados**: adicionada alternância no Navegador de arquivos para visualizar os metadados da imagem selecionada em uma tabela em vez da grade de imagens
* **Controle deslizante de zoom da grade de imagens**: novo controle deslizante da interface do usuário para ajustar o tamanho das miniaturas (também suporta CTRL + roda do mouse)
* **Botões de exportação da grade de imagens**: botões na linha superior para alternar as miniaturas de JPG para exportações processadas (alvos, refletância, índice, LUT)
* **Guia Mapa**: Novo mapa 2D interativo que mostra marcadores de localização GPS da imagem
  * Suporta Google Maps e blocos de mapa ESRI (seleciona automaticamente o melhor serviço de blocos com base na disponibilidade do nível de zoom)
  * Visualização da miniatura ao passar o mouse sobre os marcadores do mapa

**Correções de bugs*** Suporte aprimorado para a instalação do Chloros em computadores que não estejam em inglês

</details>

<details>

<summary>Versão 1.0.3</summary>

#### **Data de lançamento**: 20 de dezembro de 2025**Novos recursos*** Lançamento inicial

**Melhorias*** Lançamento inicial

**Correções de bugs*** Lançamento inicial

**Problemas conhecidos*** Lançamento inicial

</details>***

## Contrato de licença**Software proprietário** - Copyright (c) 2026 MAPIR Inc.

É proibido o uso, distribuição ou modificação não autorizados.

**Versão gratuita**: Disponível para uso pessoal e comercial com limitações de recursos**Chloros+**: Licença baseada em assinatura para recursos avançados e implantações comerciais
