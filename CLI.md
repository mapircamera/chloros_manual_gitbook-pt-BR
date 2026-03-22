# CLI : Linha de comando

<figure><img src=".gitbook/assets/cli.JPG" alt=""><figcaption></figcaption></figure>O **Chloros CLI** oferece acesso poderoso via linha de comando ao mecanismo de processamento de imagens Chloros, permitindo automação, criação de scripts e operação sem interface gráfica para seus fluxos de trabalho de imagem.

### Principais recursos

* 🚀 **Automação** - Processamento em lote por script de vários conjuntos de dados
* 🔗 **Integração** - Incorporação em fluxos de trabalho e pipelines existentes
* 💻 **Operação sem interface gráfica** - Execução sem GUI
* 🌍 **Multilíngue** - Suporte para 38 idiomas
* ⚡ **Processamento paralelo** - [Adaptação dinâmica de computação](processing-architecture/dynamic-compute-adaptation.md) otimiza automaticamente para o seu hardware

### Requisitos

| Requisito          | Detalhes                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Sistema operacional** | Windows 10/11 (64 bits), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Licença**          | Chloros+ ([plano pago necessário](https://cloud.mapir.camera/pricing)) |
| **Memória**           | Mínimo de 8 GB de RAM (recomendado 16 GB)                                  |
| **Internet**         | Necessária para ativação da licença                                     |
| **Espaço em disco**       | Varia de acordo com o tamanho do projeto                                              |

{% hint style="warning" %}
**Requisito de licença**: O CLI requer uma assinatura paga do Chloros+. Os planos padrão (gratuitos) não têm acesso ao CLI. Acesse [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para fazer o upgrade.
{% endhint %}

## Início rápido

### Instalação

#### Windows

O CLI é incluído automaticamente no instalador do Chloros:

1. Baixe e execute o **Instalador Chloros.exe**

2. Conclua o assistente de instalação
3. O CLI foi instalado em: `C:\Program Files\Chloros\resources\cli\chloros-cli.exe`

{% hint style="success" %}
O instalador adiciona automaticamente o `chloros-cli` ao PATH do seu sistema. Reinicie o terminal após a instalação.
{% endhint %}

#### Linux

Instale o pacote `.deb` para a sua arquitetura:

```bash
# Linux amd64
sudo dpkg -i chloros-amd64.deb

# Linux arm64 (NVIDIA Jetson, JetPack 6)
sudo dpkg -i chloros-arm64-jp6.deb
```

Para obter detalhes sobre a configuração do Linux, consulte [Instalação do Linux](linux/linux-installation.md).

### Configuração inicial

Antes de usar o CLI, ative sua licença Chloros+:

**Windows:**

```powershell
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
# Login with your Chloros+ account
chloros-cli login user@example.com 'your_password'

# Check license status
chloros-cli status

# Process your first project
chloros-cli process ~/images/dataset001
```

### Uso básico

Processe uma pasta com as configurações padrão:

**Windows:**

```powershell
chloros-cli process "C:\Images\Dataset001"
```

**Linux:**

```bash
chloros-cli process ~/images/dataset001
```

***

## Referência de comandos

### Sintaxe geral

```
chloros-cli [global-options] <command> [command-options]
```

***

## Comandos

### `process` - Processar imagens

Processa imagens em uma pasta com calibração.

**Sintaxe:**

```bash
chloros-cli process <input-folder> [options]
```

**Exemplos:**

```bash
# Windows
chloros-cli process "C:\Datasets\Survey_001" --vignette --reflectance

# Linux
chloros-cli process ~/datasets/survey_001 --vignette --reflectance
```

#### Opções do comando Processar

| Opção                | Tipo    | Padrão        | Descrição                                                                            |
| --------------------- | ------- | -------------- | -------------------------------------------------------------------------------------- |
| `<input-folder>`      | Caminho    | _Obrigatório_     | Pasta contendo imagens multiespectrais RAW/JPG                                         |
| `-o, --output`        | Caminho    | Igual à entrada  | Pasta de saída para imagens processadas                                                     |
| `-n, --project-name`  | String  | Gerado automaticamente | Nome personalizado do projeto                                                                    |
| `--vignette`          | Sinalizador    | Ativado        | Ativar correção de vinheta                                                             |
| `--no-vignette`       | Sinalizador    | -              | Desativar correção de vinheta                                                            |
| `--reflectance`       | Sinalizador    | Ativado        | Ativar calibração de refletância                                                         |
| `--no-reflectance`    | Sinalizador    | -              | Desativar calibração de refletância                                                        |
| `--ppk`               | Sinalizador    | Desativado       | Aplicar correções PPK a partir dos dados do sensor de luz .daq                                      |
| `--format`            | Opção  | TIFF (16 bits)  | Formato de saída: `TIFF (16-bit)`, `TIFF (32-bit, Percent)`, `PNG (8-bit)`, `JPG (8-bit)` |
| `--min-target-size`   | Inteiro | Automático           | Tamanho mínimo do alvo em pixels para detecção do painel de calibração                          |
| `--target-clustering` | Inteiro | Automático           | Limite de agrupamento de alvos (0-100)                                                    |
| `--debayer`           | Opção  | `standard`     | Método de debayer: `standard` ou `texture-aware` (somente Chloros+)                          |
| `--target`, `--targets` | Sinalizador  | Desativado       | Pesquisar alvos de calibração apenas na subpasta “target” ou “targets” (acelera o processamento) |
| `--indices`           | Lista    | Nenhuma           | Índices de vegetação a serem calculados (por exemplo, `--indices NDVI NDRE GNDVI`)                    |
| `--exposure-pin-1`    | String  | Nenhuma           | Bloquear a exposição para o modelo da câmera (Pino 1)                                                 |
| `--exposure-pin-2`    | String  | Nenhuma           | Bloquear exposição para o modelo da câmera (Pino 2)                                                 |
| `--recal-interval`    | Inteiro | Automático           | Intervalo de recalibração em segundos                                                      |
| `--timezone-offset`   | Inteiro | 0              | Deslocamento de fuso horário em horas                                                               |

***

### `login` - Autenticar conta

Faça login com suas credenciais Chloros+ para habilitar o processamento CLI.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemplo:**

```bash
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiais**: Use aspas simples em torno de senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>***

### `logout` - Limpar credenciais

Limpa as credenciais armazenadas e faz o logout da sua conta.

**Sintaxe:**

```bash
chloros-cli logout
```

**Exemplo:**

```bash
chloros-cli logout
```

**Saída:**

```
✓ Logout successful
ℹ Credentials cleared from cache
```

{% hint style="info" %}
**Usuários do SDK**: O Python SDK também oferece um método programático `logout()` para limpar credenciais dentro de scripts Python. Consulte a [documentação do Python SDK](api-python-sdk.md#logout) para obter detalhes.
{% endhint %}

***

### `status` - Verificar status da licença

Exibe o status atual da licença e da autenticação.

**Sintaxe:**

```bash
chloros-cli status
```

**Exemplo:**

```bash
chloros-cli status
```

**Saída:**

```
╔══════════════════════════════════════╗
║     LICENSE & ACCOUNT INFORMATION    ║
╚══════════════════════════════════════╝

📧 Email: user@example.com
📋 Plan: Chloros+ Professional
🔓 API/CLI Access: Enabled
✓ Status: Active
```

***

### `export-status` - Verificar o andamento da exportação

Monitore o andamento da exportação da Thread 4 durante ou após o processamento.

**Sintaxe:**

```bash
chloros-cli export-status
```

**Exemplo:**

```bash
chloros-cli export-status
```

**Caso de uso:** Chame este comando enquanto o processamento estiver em execução para verificar o andamento da exportação.***

### `language` - Gerenciar idioma da interface

Visualize ou altere o idioma da interface CLI.

**Sintaxe:**

```bash
# Show current language
chloros-cli language

# List all available languages
chloros-cli language --list

# Set a specific language
chloros-cli language <language-code>
```

**Exemplos:**

```bash
# View current language
chloros-cli language

# List all 38 supported languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Change to Japanese
chloros-cli language ja
```

#### Idiomas suportados (38 no total)

| Código    | Idioma              | Nome nativo      |
| ------- | --------------------- | ---------------- |
| `en`    | Inglês               | English          |
| `es`    | Espanhol               | Español          |
| `pt`    | Português            | Português        |
| `fr`    | Francês                | Français         |
| `de`    | Alemão                | Deutsch          |
| `it`    | Italiano               | Italiano         |
| `ja`    | Japonês              | 日本語              |
| `ko`    | Coreano                | 한국어              |
| `zh`    | Chinês (Simplificado)  | 简体中文             |
| `zh-TW` | Chinês (Tradicional) | 繁體中文             |
| `ru`    | Russo               | Русский          |
| `nl`    | Holandês                | Nederlands       |
| `ar`    | Árabe                | العربية          |
| `pl`    | Polonês                | Polski           |
| `tr`    | Turco               | Türkçe           |
| `hi`    | Hindi                 | हिंदी            |
| `id`    | Indonésio            | Bahasa Indonesia |
| `vi`    | Vietnamita            | Tiếng Việt       |
| `th`    | Tailandês                  | ไทย              |
| `sv`    | Sueco               | Svenska          |
| `da`    | Dinamarquês                | Dansk            |
| `no`    | Norueguês             | Norsk            |
| `fi`    | Finlandês               | Suomi            |
| `el`    | Grego                 | Ελληνικά         |
| `cs`    | Tcheco                | Čeština          |
| `hu`    | Húngaro             | Magyar           |
| `ro`    | Romeno              | Română           |
| `uk`    | Ucraniano            | Українська       |
| `pt-BR` | Português Brasileiro  | Português Brasileiro |
| `zh-HK` | Cantonês             | 粵語             |
| `ms`    | Malaio                 | Bahasa Melayu    |
| `sk`    | Eslovaco                | Slovenčina       |
| `bg`    | Búlgaro             | Български        |
| `hr`    | Croata              | Hrvatski         |
| `lt`    | Lituano            | Lietuvių         |
| `lv`    | Letão               | Latviešu         |
| `et`    | Estoniano              | Eesti            |
| `sl`    | Esloveno             | Slovenščina      |

{% hint style="success" %}
**Persistência automática**: Sua preferência de idioma é salva em `~/.chloros/cli_language.json` e permanece em todas as sessões.
{% endhint %}

***

### `set-project-folder` - Definir pasta padrão do projeto

Altere o local da pasta padrão do projeto (compartilhada com a GUI em Windows).

**Sintaxe:**

```bash
chloros-cli set-project-folder <folder-path>
```

**Exemplos:**

```bash
# Windows
chloros-cli set-project-folder "C:\Projects\2025"

# Linux
chloros-cli set-project-folder ~/projects/2025
```

***

### `get-project-folder` - Mostrar pasta do projeto

Exibe o local atual da pasta padrão do projeto.

**Sintaxe:**

```bash
chloros-cli get-project-folder
```

**Exemplo:**

```bash
chloros-cli get-project-folder
```

**Saída:**

```

# Windows
ℹ Current project folder: C:\Projects\2025

# Linux
ℹ Current project folder: /home/user/.local/share/chloros/projects
```

***

### `reset-project-folder` - Redefinir para o padrão

Redefine a pasta do projeto para o local padrão.

**Sintaxe:**

```bash
chloros-cli reset-project-folder
```

***

### `selftest` - Executar diagnósticos do sistema

Executa 7 verificações de diagnóstico para conferir a configuração do seu sistema.

**Sintaxe:**

```bash
chloros-cli selftest
```

**Diagnósticos realizados:**

1. Verificação da versão
2. Disponibilidade da porta (5000)
3. Inicialização do backend
4. Teste de conectividade API
5. Informações do sistema e detecção da GPU
6. Verificação dos modelos do denoiser
7. Verificação da disponibilidade do CUDA

{% hint style="info" %}
**Útil para solução de problemas**: Execute o `selftest` após a instalação para verificar se o seu sistema está configurado corretamente, especialmente no Linux/Jetson, onde a configuração da GPU e do CUDA pode precisar de verificação.
{% endhint %}

***

### `update` - Verificar atualizações (somente Linux)

Verifique e instale as atualizações do CLI em sistemas Linux.

**Sintaxe:**

```bash
# Check for updates without installing
chloros-cli update --check

# Check for and install updates
chloros-cli update
```

| Opção    | Descrição                        |
| --------- | ---------------------------------- |
| `--check` | Apenas verificar atualizações, não instalar |

{% hint style="info" %}
Este comando está disponível apenas no Linux. No Windows, as atualizações são fornecidas por meio do instalador.
{% endhint %}

***

## Opções globais

Essas opções se aplicam a todos os comandos:

| Opção            | Tipo    | Padrão       | Descrição                                      |
| ----------------- | ------- | ------------- | ------------------------------------------------ |
| `--backend-exe`   | Caminho    | Detectado automaticamente | Caminho para o executável do backend                       |
| `--port`          | Inteiro | 5000          | Número da porta do backend API                          |
| `--restart`       | Sinalizador    | -             | Forçar reinício do backend (encerra processos existentes) |
| `--version`       | Sinalizador    | -             | Mostrar informações de versão e sair                |
| `--help`          | Sinalizador    | -             | Mostrar informações de ajuda e sair                   |

{% hint style="info" %}
**Detecção automática do backend**: O caminho `--backend-exe` é detectado automaticamente por plataforma:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manual)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

**Exemplo com opções globais:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Survey_001"
```

**Linux:**

```bash
chloros-cli --port 5001 process ~/datasets/survey_001
```

***

## Guia de configurações de processamento

### Processamento paralelo e adaptação dinâmica de computação

O Chloros 1.1.0 inclui [Adaptação dinâmica de computação](processing-architecture/dynamic-compute-adaptation.md) — o mecanismo de processamento **detecta automaticamente seu hardware** e seleciona a estratégia ideal:

| Plataforma | Estratégia | Trabalhadores | Pipeline | Notas |
| --- | --- | --- | --- | --- |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 1 | `tiled_gpu` | Eficiente em memória, serializado |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 3 | `fused_gpu` | Processamento simultâneo por GPU |
| **Desktop com GPU de 8 GB** | `GPU_SINGLE` | 3 | `tiled_gpu` | Bom desempenho de desktop |
| **Desktop com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 3-4 | `fused_gpu` | Desempenho ideal do desktop |
| **Sistema apenas com CPU** | `CPU_PARALLEL` | núcleos - 1 | `cpu_fallback` | Não requer GPU |

{% hint style="success" %}
**Não é necessária configuração manual!** O Chloros detecta automaticamente sua CPU, GPU, RAM e (no Jetson) sensores térmicos, e então configura o pipeline de processamento ideal automaticamente.
{% endhint %}

### Métodos de Debayer

| Método | Sinalizador CLI | Qualidade | Velocidade | Licença |
| --- | --- | --- | --- | --- |
| **Padrão (Rápido, Qualidade Média)** | `--debayer standard` | Boa | Rápido | Grátis / Chloros+ |
| **Sensível à textura (Lento, Qualidade máxima)** | `--debayer texture-aware` | Máxima | Lento | Apenas Chloros+ |

O método padrão de debayer é **Padrão**. O método**Sensível à Textura** utiliza um modelo de redução de ruído baseado em IA/ML para obter resultados da mais alta qualidade, mas requer uma licença Chloros+ e uma GPU NVIDIA.

```bash
# Use Texture Aware debayer (Chloros+ only)
chloros-cli process ~/datasets/field_a --debayer texture-aware
```

### Correção de vinheta

**O que faz:** Corrige a perda de luz nas bordas da imagem (cantos mais escuros comuns em imagens de câmera).

* **Ativado por padrão** - A maioria dos usuários deve manter isso ativado
* Use `--no-vignette` para desativar

{% hint style="success" %}
**Recomendação**: Sempre ative a correção de vinheta para garantir brilho uniforme em todo o quadro.
{% endhint %}

### Calibração de refletância

Converte valores brutos do sensor em porcentagens de refletância padronizadas usando painéis de calibração.

* **Ativado por padrão** - Essencial para análise de vegetação
* Requer painéis de alvo de calibração nas imagens
* Use `--no-reflectance` para desativar

{% hint style="info" %}
**Requisitos**: Certifique-se de que os painéis de calibração estejam devidamente expostos e visíveis em suas imagens para uma conversão precisa da refletância.
{% endhint %}

### Correções PPK

**O que faz:** Aplica correções cinemáticas pós-processadas usando dados de log do DAQ-A-SD para melhorar a precisão do GPS.

* **Desativado por padrão**
* Use `--ppk` para ativar
* Requer arquivos .daq na pasta do projeto provenientes do sensor de luz DAQ-A-SD MAPIR.

### Formatos de saída

<table><thead><tr><th width="197">Formato</th><th width="130.20001220703125">Profundidade de bits</th><th width="116.5999755859375">Tamanho do arquivo</th><th>Ideal para</th></tr></thead><tbody><tr><td><strong>TIFF (16 bits)</strong> ⭐</td><td>Inteiro de 16 bits</td><td>Grande</td><td>Análise GIS, fotogrametria (recomendado)</td></tr><tr><td><strong>TIFF (32 bits, porcentagem)</strong></td><td>Float de 32 bits</td><td>Muito grande</td><td>Análise científica, pesquisa</td></tr><tr><td><strong>PNG (8 bits)</strong></td><td>Inteiro de 8 bits</td><td>Médio</td><td>Inspeção visual, compartilhamento na web</td></tr><tr><td><strong>JPG (8 bits)</strong></td><td>Inteiro de 8 bits</td><td>Pequeno</td><td>Pré-visualização rápida, saída compactada</td></tr></tbody></table>***

## Automação e scripts

### Processamento em lote do PowerShell (Windows)

Processe várias pastas de conjuntos de dados automaticamente no Windows:

```powershell
# process_all_datasets.ps1

$datasets = Get-ChildItem "C:\Datasets\2025" -Directory

foreach ($dataset in $datasets) {
    Write-Host "Processing $($dataset.Name)..." -ForegroundColor Cyan
    
    chloros-cli process $dataset.FullName `
        --vignette `
        --reflectance
    
    if ($LASTEXITCODE -eq 0) {
        Write-Host "✓ $($dataset.Name) complete" -ForegroundColor Green
    } else {
        Write-Host "✗ $($dataset.Name) failed" -ForegroundColor Red
    }
}

Write-Host "All datasets processed!" -ForegroundColor Green
```

### Script em lote Windows (Windows)

Loop simples para processamento em lote no Windows:

```batch
@echo off
echo Starting batch processing...

for /d %%i in (C:\Datasets\2025\*) do (
    echo.
    echo ========================================
    echo Processing: %%i
    echo ========================================
    chloros-cli process "%%i"
    
    if %ERRORLEVEL% EQU 0 (
        echo SUCCESS: %%i processed
    ) else (
        echo ERROR: %%i failed
    )
)

echo.
echo All datasets processed!
pause
```

### Processamento em lote no Bash (Linux)

Processar várias pastas de conjuntos de dados no Linux:

```bash
#!/bin/bash
# process_all_datasets.sh

for dataset in ~/datasets/2026/*/; do
    name=$(basename "$dataset")
    echo "Processing $name..."

    chloros-cli process "$dataset" \
        --vignette \
        --reflectance

    if [ $? -eq 0 ]; then
        echo "✓ $name complete"
    else
        echo "✗ $name failed"
    fi
done

echo "All datasets processed!"
```

### Script de automação Python (multiplataforma)

Automação avançada com tratamento de erros (funciona no Windows e no Linux):

```python
import subprocess
import os
import sys
from pathlib import Path
from datetime import datetime

def process_dataset(input_folder):
    """Process a folder using Chloros CLI"""
    cmd = ['chloros-cli', 'process', str(input_folder)]
    
    # Execute command
    result = subprocess.run(
        cmd, 
        capture_output=True, 
        text=True,
        encoding='utf-8'
    )
    
    return result.returncode == 0, result.stdout, result.stderr

def main():
    """Process all datasets in a directory"""
    # Adjust path for your platform
    # Windows: Path('C:/Datasets/2025')
    # Linux:   Path.home() / 'datasets' / '2025'
    datasets_dir = Path('C:/Datasets/2025')
    log_file = Path('processing_log.txt')
    
    successful = []
    failed = []
    
    # Start processing
    print(f"Starting batch processing: {datetime.now()}")
    print(f"Scanning: {datasets_dir}")
    print("=" * 60)
    
    for dataset_folder in sorted(datasets_dir.iterdir()):
        if not dataset_folder.is_dir():
            continue
        
        print(f"\nProcessing: {dataset_folder.name}")
        
        success, stdout, stderr = process_dataset(dataset_folder)
        
        if success:
            print(f"✓ {dataset_folder.name} - SUCCESS")
            successful.append(dataset_folder.name)
        else:
            print(f"✗ {dataset_folder.name} - FAILED")
            failed.append(dataset_folder.name)
            
            # Log error details
            with open(log_file, 'a', encoding='utf-8') as f:
                f.write(f"\n=== {dataset_folder.name} - {datetime.now()} ===\n")
                f.write(f"STDOUT:\n{stdout}\n")
                f.write(f"STDERR:\n{stderr}\n")
    
    # Print summary
    print("\n" + "=" * 60)
    print(f"SUMMARY - Completed: {datetime.now()}")
    print(f"  Successful: {len(successful)}")
    print(f"  Failed: {len(failed)}")
    
    if failed:
        print(f"\nFailed folders:")
        for folder in failed:
            print(f"  - {folder}")
        print(f"\nCheck {log_file} for error details")
        sys.exit(1)
    else:
        print("\nAll datasets processed successfully!")
        sys.exit(0)

if __name__ == '__main__':
    main()
```

***

## Fluxo de trabalho de processamento

### Fluxo de trabalho padrão

1. **Entrada**: Pasta contendo pares de imagens RAW/JPG
2. **Detecção**: O CLI verifica automaticamente os arquivos de imagem compatíveis
3. **Processamento**: O modo paralelo se adapta aos núcleos da sua CPU (Chloros+)
4. **Saída**: Cria subpastas por modelo de câmera com as imagens processadas

### Exemplo de estrutura de saída

```

MyProject/
├── project.json                             # Project metadata
├── 2025_0203_193056_008.JPG                # Original JPG
├── 2025_0203_193055_007.RAW                # Original RAW
└── Survey3N_RGN/                           # Processed outputs ✓
    ├── 2025_0203_193056_008_Reflectance.tif   # Calibrated reflectance
    ├── 2025_0203_193056_008_Target.tif        # Target detection
    └── ...
```

### Estimativas de tempo de processamento

Tempos de processamento típicos para 100 imagens (12 MP cada):

| Plataforma | Modo | Tempo estimado | Observações |
| --- | --- | --- | --- |
| **Desktop com GPU de 12 GB ou mais** | `GPU_PARALLEL` | 5-10 min | Opção mais rápida |
| **Desktop com GPU de 8 GB** | `GPU_SINGLE` | 10-15 min | Bom desempenho |
| **Jetson Orin NX 16 GB** | `GPU_PARALLEL` | 15-25 min | Computação de ponta |
| **Jetson Nano 8 GB** | `GPU_SINGLE` | 30-60 min | Memória limitada |
| **Somente CPU** | `CPU_PARALLEL` | 20-40 min | Não requer GPU |

{% hint style="info" %}
**Dica de desempenho**: O tempo de processamento varia de acordo com o número de imagens, a resolução, o método de debayer e o hardware. O debayer com reconhecimento de textura leva significativamente mais tempo do que o padrão. Consulte [Adaptação dinâmica de computação](processing-architecture/dynamic-compute-adaptation.md) para obter detalhes.
{% endhint %}

***

## Solução de problemas

### CLI não encontrado

**Erro Windows:**

```
'chloros-cli' is not recognized as an internal or external command
```

**Windows Soluções:**

1. Verifique o local de instalação:

```powershell
dir "C:\Program Files\Chloros\resources\cli\chloros-cli.exe"
```

2. Use o caminho completo se não estiver no PATH:

```powershell
"C:\Program Files\Chloros\resources\cli\chloros-cli.exe" process "C:\Datasets\Field_A"
```

3. Adicione ao PATH manualmente:
   * Abra Propriedades do Sistema → Variáveis de Ambiente
   * Edite a variável PATH
   * Adicione: `C:\Program Files\Chloros\resources\cli`
   * Reinicie o terminal

**Erro Linux:**

```
chloros-cli: command not found
```

**Soluções para Linux:**

1. Verifique a instalação:

```bash
which chloros-cli
dpkg -L chloros-amd64  # or chloros-arm64-jp6
```

2. Recarregue seu shell:

```bash
source ~/.bashrc
```

3. Verifique as permissões:

```bash
sudo chmod +x /usr/bin/chloros-cli
```

***

### Falha ao iniciar o backend**Erro:**

```

Backend failed to start within 30 seconds
```

**Soluções:**

1. Verifique se o backend já está em execução (feche-o primeiro)
2. Verifique se o firewall não está bloqueando (Windows) ou verifique a disponibilidade da porta (Linux: `lsof -i :5000`)
3. Tente uma porta diferente:

```bash
# Windows
chloros-cli --port 5001 process "C:\Datasets\Field_A"

# Linux
chloros-cli --port 5001 process ~/datasets/field_a
```

4. Force a reinicialização do backend:

```bash
# Windows
chloros-cli --restart process "C:\Datasets\Field_A"

# Linux
chloros-cli --restart process ~/datasets/field_a
```

5. No caso de Linux, verifique se o executável do backend existe:

```bash
ls -la /usr/lib/chloros/chloros-backend
```

***

### Problemas de licença/autenticação**Erro:**

```

Chloros+ license required for CLI access
```

**Soluções:**

1. Verifique se você possui uma assinatura ativa do Chloros+
2. Faça login com suas credenciais:

```bash
chloros-cli login user@example.com 'password'
```

3. Verifique o status da licença:

```bash
chloros-cli status
```

4. Entre em contato com o suporte: info@mapir.camera

***

### Nenhuma imagem encontrada**Erro:**

```

No images found in the specified folder
```

**Soluções:**

1. Verifique se a pasta contém formatos compatíveis (.RAW, .TIF, .JPG)
2. Verifique se o caminho da pasta está correto (use aspas para caminhos com espaços)
3. Certifique-se de que você tem permissões de leitura para a pasta
4. Verifique se as extensões dos arquivos estão corretas

***

### Processamento travado ou congelado**Soluções:**

1. Verifique o espaço disponível em disco (garanta que haja espaço suficiente para a saída)
2. Feche outros aplicativos para liberar memória
3. Reduza o número de imagens (processe em lotes)

***

### Porta já em uso**Erro:**

```

Port 5000 is already in use
```

**Soluções:**

**Windows:**

```powershell
chloros-cli --port 5001 process "C:\Datasets\Field_A"
```

**Linux:**

```bash
# Find what's using port 5000
lsof -i :5000

# Use a different port
chloros-cli --port 5001 process ~/datasets/field_a
```

***

## Perguntas frequentes

### P: Preciso de uma licença para o CLI?

**R:**Sim! O CLI requer uma**licença Chloros+** paga.

* ❌ Plano Standard (gratuito): CLI desativado
* ✅ Planos Chloros+ (pagos): CLI totalmente ativado

Inscreva-se em: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

***

### P: Posso usar o CLI em um servidor sem GUI?**R:** Sim! O CLI é executado totalmente sem interface gráfica. Esse é o principal caso de uso no Linux.**Servidor Windows:**
* Servidor Windows 2016 ou posterior
* Visual C++ Redistributable instalado

**Servidor Linux:**
* Ubuntu 20.04+ / Debian 11+ (amd64) ou JetPack 6 (arm64)
* Instalação via pacote `.deb`

**Ambas as plataformas:**
* Mínimo de 8 GB de RAM (recomenda-se 16 GB)
* Ativação única da licença: `chloros-cli login user@example.com 'password'`

***

### P: Onde as imagens processadas são salvas?**R:**Por padrão, as imagens processadas são salvas na**mesma pasta da entrada**, em subpastas de modelos de câmera (por exemplo, `Survey3N_RGN/`).

Use a opção `-o` para especificar uma pasta de saída diferente:

```bash
# Windows
chloros-cli process "C:\Input" -o "D:\Output"

# Linux
chloros-cli process ~/input -o ~/output
```

***

### P: Posso processar várias pastas de uma vez?**R:** Não diretamente em um único comando, mas você pode usar scripts para processar pastas sequencialmente. Consulte a seção [Automação e Scripts](CLI.md#automation--scripting).***

### P: Como faço para salvar a saída do CLI em um arquivo de log?**PowerShell:**

```powershell
chloros-cli process "C:\Datasets\Field_A" | Tee-Object -FilePath "processing.log"
```

**Batch:**

```batch
chloros-cli process "C:\Datasets\Field_A" > processing.log 2>&1
```

**Linux Bash:**

```bash
chloros-cli process ~/datasets/field_a 2>&1 | tee processing.log
```

***

### P: O que acontece se eu pressionar Ctrl+C durante o processamento?**R:** O CLI irá:

1. Interromper o processamento de forma controlada
2. Desligar o backend
3. Sair com o código 130

Imagens parcialmente processadas podem permanecer na pasta de saída.

***

### P: Posso automatizar o processamento do CLI?**R:** Com certeza! O CLI foi projetado para automação. Consulte [Automação e Scripts](CLI.md#automation--scripting) para exemplos em PowerShell (Windows), Batch (Windows), Bash (Linux) e exemplos de Python (multiplataforma).***

### P: Como verifico a versão do CLI?**R:**

```bash
chloros-cli --version
```

**Saída:**

```

Chloros CLI 1.1.0
```

***

## Obtendo ajuda

### Ajuda da linha de comando

Visualize as informações de ajuda diretamente no CLI:

```bash
# General help
chloros-cli --help

# Command-specific help
chloros-cli process --help
chloros-cli login --help
chloros-cli language --help
```

### Canais de suporte

* **E-mail**: info@mapir.camera
* **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preços**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)***

## Exemplos completos

### Exemplo 1: Processamento básico

Processe com as configurações padrão (vinheta, refletância):

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A_2025_01_15"
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a_2025_01_15
```

***

### Exemplo 2: Saída científica de alta qualidade

32 bits de precisão flutuante TIFF:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "TIFF (32-bit, Percent)" ^
  --vignette ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "TIFF (32-bit, Percent)" \
  --vignette \
  --reflectance
```

***

### Exemplo 3: Processamento rápido de visualização

XPROTX de 8 bits 000357XPROTX sem calibração para revisão rápida:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --format "PNG (8-bit)" ^
  --no-vignette ^
  --no-reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --format "PNG (8-bit)" \
  --no-vignette \
  --no-reflectance
```

***

### Exemplo 4: Processamento com correção PPK

Aplicar correções PPK com refletância:

**Windows:**

```powershell
chloros-cli process "C:\Datasets\Field_A" ^
  --ppk ^
  --reflectance
```

**Linux:**

```bash
chloros-cli process ~/datasets/field_a \
  --ppk \
  --reflectance
```

***

### Exemplo 5: Local de saída personalizado

Processe para um local diferente com formato específico:

**Windows:**

```powershell
chloros-cli process "C:\Input\Raw_Images" ^
  -o "D:\Output\Processed" ^
  --format "TIFF (16-bit)"
```

**Linux:**

```bash
chloros-cli process ~/input/raw_images \
  -o ~/output/processed \
  --format "TIFF (16-bit)"
```

***

### Exemplo 6: Fluxo de trabalho de autenticação

Fluxo de autenticação completo (igual em todas as plataformas):

```bash
# Step 1: Login
chloros-cli login user@example.com 'MyP@ssw0rd'

# Step 2: Verify status
chloros-cli status

# Step 3: Process images
# Windows: chloros-cli process "C:\Datasets\Field_A"
# Linux:   chloros-cli process ~/datasets/field_a
chloros-cli process ~/datasets/field_a

# Step 4: Logout (optional, when switching accounts)
chloros-cli logout
```

***

### Exemplo 7: Uso multilíngue

Alterar o idioma da interface (igual em todas as plataformas):

```bash
# List available languages
chloros-cli language --list

# Change to Spanish
chloros-cli language es

# Process with Spanish interface
# Windows: chloros-cli process "C:\Vuelos\Campo_A"
# Linux:   chloros-cli process ~/vuelos/campo_a
chloros-cli process ~/vuelos/campo_a

# Change back to English
chloros-cli language en
```
