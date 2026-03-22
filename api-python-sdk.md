# API : Python SDK

O **Chloros Python SDK** fornece acesso programático ao mecanismo de processamento de imagens Chloros, permitindo automação, fluxos de trabalho personalizados e integração perfeita com seus aplicativos Python e pipelines de pesquisa.

### Principais recursos

* 🐍 **Python nativo** - Código API limpo e em Python para processamento de imagens
* 🔧 **Acesso total ao API** - Controle completo sobre o processamento do Chloros
* 🚀 **Automação** - Crie fluxos de trabalho personalizados de processamento em lote
* 🔗 **Integração** - Incorpore o Chloros em aplicativos Python existentes
* 📊 **Pronto para pesquisa** - Perfeito para pipelines de análise científica
* ⚡ **Processamento paralelo** - Escala de acordo com os núcleos da sua CPU (Chloros+)

### Requisitos

| Requisito          | Detalhes                                                             |
| -------------------- | ------------------------------------------------------------------- |
| **Chloros instalado** | Windows: Instalador para desktop; Linux: Pacote `.deb`                  |
| **Licença**          | Chloros+ ([plano pago necessário](https://cloud.mapir.camera/pricing)) |
| **Sistema operacional** | Windows 10/11 (64 bits), Linux x86_64 (amd64), Linux arm64 (NVIDIA Jetson JetPack 6) |
| **Python**           | Python 3.7 ou superior                                                |
| **Memória**           | Mínimo de 8 GB de RAM (recomendado 16 GB)                                  |
| **Internet**         | Necessária para ativação da licença                                     |

{% hint style="warning" %}
**Requisito de licença**: O Python SDK requer uma assinatura paga do Chloros+ para acesso ao API. Os planos padrão (gratuitos) não têm acesso ao API/SDK. Acesse [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing) para fazer o upgrade.
{% endhint %}

## Início rápido

### Instalação

Instale via pip:

```bash
pip install chloros-sdk
```

{% hint style="info" %}
**Configuração inicial**: Antes de usar o SDK, ative sua licença Chloros+ abrindo o Chloros, Chloros (Navegador) ou o Chloros CLI e fazendo login com suas credenciais. Isso só precisa ser feito uma vez. No Linux (sem GUI), use: `chloros-cli login user@example.com 'password'`
{% endhint %}

### Uso básico

Processe uma pasta com apenas algumas linhas:

```python
from chloros_sdk import process_folder

# One-line processing (Windows)
results = process_folder("C:\\DroneImages\\Flight001")

# One-line processing (Linux)
results = process_folder("/home/user/drone_images/flight001")
```

{% hint style="info" %}
**Caminhos multiplataforma**: Os exemplos de código nesta página utilizam caminhos no estilo Windows (por exemplo, `C:\\DroneImages\\Flight001`). No Linux, use caminhos no estilo Linux (por exemplo, `/home/user/drone_images/flight001` ou `~/drone_images/flight001`). O SDK funciona de maneira idêntica em ambas as plataformas.
{% endhint %}

### Controle total

Para fluxos de trabalho avançados:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project
chloros.create_project("MyProject", camera="Survey3N_RGN")

# Import images
chloros.import_images("C:\\DroneImages\\Flight001")  # Windows
# chloros.import_images("/home/user/drone_images/flight001")  # Linux

# Configure settings
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE", "GNDVI"]
)

# Process images
chloros.process(mode="parallel", wait=True)
```

***

## Guia de instalação

### Pré-requisitos

Antes de instalar o SDK, certifique-se de ter:

1. **Chloros instalado** — Windows: Instalador para desktop ([download](download.md)); Linux: Pacote `.deb` ([Instalação do Linux](linux/linux-installation.md))
2. **Python 3.7+** instalado ([python.org](https://www.python.org))
3. **Licença Chloros+ ativa** ([atualização](https://cloud.mapir.camera/pricing))

### Instalar via pip

**Instalação padrão:**

```bash
pip install chloros-sdk
```

**Com suporte para monitoramento de progresso:**

```bash
pip install chloros-sdk[progress]
```

**Instalação de desenvolvimento:**

```bash
pip install chloros-sdk[dev]
```

### Verificar a instalação

Teste se o SDK está instalado corretamente:

```python
import chloros_sdk
print(f"Chloros SDK version: {chloros_sdk.__version__}")
```

***

## Configuração inicial

### Ativação da licença

O SDK usa a mesma licença que o Chloros, o Chloros (Navegador) e o Chloros CLI. Ative uma vez através da GUI ou do CLI:**Windows:**Abra o**Chloros ou o Chloros (Navegador)** e faça login na guia Usuário <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> ou use o CLI.**Linux:** Use o CLI (sem GUI disponível):

```bash
chloros-cli login user@example.com 'your_password'
```

A licença é armazenada em cache localmente e persiste após reinicializações.

{% hint style="success" %}
**Configuração única**: Após fazer login pela GUI ou pelo CLI, o SDK usa automaticamente a licença armazenada em cache. Não é necessária autenticação adicional!
{% endhint %}

{% hint style="info" %}
**Logout**: Os usuários do SDK podem limpar programaticamente as credenciais armazenadas em cache usando o método `logout()`. Consulte o [método logout()](#logout) na Referência do API.
{% endhint %}

### Testar conexão

Verifique se o SDK consegue se conectar ao Chloros:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK (auto-starts backend if needed)
chloros = ChlorosLocal()

# Check status
status = chloros.get_status()
print(f"Backend running: {status['running']}")
```

***

## Referência do API

### Classe ChlorosLocal

Classe principal para processamento de imagens local do Chloros.

#### Construtor

```python
ChlorosLocal(
    api_url="http://localhost:5000",     # Backend URL
    auto_start_backend=True,             # Auto-start backend if not running
    backend_exe=None,                    # Backend path (auto-detected)
    timeout=30,                          # Request timeout (seconds)
    backend_startup_timeout=60           # Backend startup timeout
)
```

**Parâmetros:**

| Parâmetro                 | Tipo | Padrão                   | Descrição                           |
| ------------------------- | ---- | ------------------------- | ------------------------------------- |
| `api_url`                 | str  | `"http://localhost:5000"` | URL do backend local Chloros          |
| `auto_start_backend`      | bool | `True`                    | Iniciar o backend automaticamente, se necessário |
| `backend_exe`             | str  | `None` (detecção automática)      | Caminho para o executável do backend            |
| `timeout`                 | int  | `30`                      | Tempo limite da solicitação em segundos            |
| `backend_startup_timeout` | int  | `60`                      | Tempo limite para inicialização do backend (segundos) |

**Exemplos:**

```python
# Default (auto-start backend, auto-detect path on Windows and Linux)
chloros = ChlorosLocal()

# Connect to running backend
chloros = ChlorosLocal(auto_start_backend=False)

# Custom backend path (Windows)
chloros = ChlorosLocal(backend_exe="C:/Custom/chloros-backend.exe")

# Custom backend path (Linux)
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")

# Custom timeout with longer startup (e.g., for Jetson)
chloros = ChlorosLocal(timeout=60, backend_startup_timeout=120)
```

{% hint style="info" %}
**Detecção automática entre plataformas**: O SDK tenta automaticamente o caminho de backend correto para sua plataforma:
* **Windows**: `C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe`
* **Linux (.deb)**: `/usr/lib/chloros/chloros-backend`
* **Linux (manual)**: `/opt/mapir/chloros/backend/chloros-backend`
{% endhint %}

***

### Métodos

#### `create_project(project_name, camera=None)`

Cria um novo projeto Chloros.

**Parâmetros:**

| Parâmetro      | Tipo | Obrigatório | Descrição                                              |
| -------------- | ---- | -------- | -------------------------------------------------------- |
| `project_name` | str  | Sim      | Nome do projeto                                     |
| `camera`       | str  | Não       | Modelo de câmera (por ex., &quot;Survey3N\_RGN&quot;, &quot;Survey3W\_OCN&quot;) |

**Retorna:** `dict` - Resposta de criação do projeto**Exemplo:**

```python
# Basic project
chloros.create_project("DroneField_A")

# With camera template
chloros.create_project("DroneField_A", camera="Survey3N_RGN")
```

***

#### `import_images(folder_path, recursive=False)`

Importar imagens de uma pasta.

**Parâmetros:**

| Parâmetro     | Tipo     | Obrigatório | Descrição                        |
| ------------- | -------- | -------- | ---------------------------------- |
| `folder_path` | str/Path | Sim      | Caminho para a pasta com imagens         |
| `recursive`   | bool     | Não       | Pesquisar subpastas (padrão: False) |

**Retorna:** `dict` - Resultados da importação com contagem de arquivos**Exemplo:**

```python
# Import from folder
chloros.import_images("C:\\DroneImages\\Flight001")

# Import recursively
chloros.import_images("C:\\DroneImages", recursive=True)
```

***

#### `configure(**settings)`

Configurar as definições de processamento.

**Parâmetros:**

| Parâmetro                 | Tipo | Padrão                 | Descrição                     |
| ------------------------- | ---- | ----------------------- | ------------------------------- |
| `debayer`                 | str  | &quot;Padrão (Rápido, Qualidade Média)&quot; | Método de debayer            |
| `vignette_correction`     | bool | `True`                  | Ativar correção de vinheta      |
| `reflectance_calibration` | bool | `True`                  | Ativar calibração de refletância  |
| `indices`                 | lista | `None`                  | Índices de vegetação a calcular |
| `export_format`           | str  | &quot;TIFF (16 bits)&quot;         | Formato de saída                   |
| `ppk`                     | bool | `False`                 | Ativar correções PPK          |
| `custom_settings`         | dict | `None`                  | Configurações personalizadas avançadas        |

**Formatos de exportação:**

* `"TIFF (16-bit)"` - Recomendado para SIG/fotogrametria
* `"TIFF (32-bit, Percent)"` - Análise científica
* `"PNG (8-bit)"` - Inspeção visual
* `"JPG (8-bit)"` - Saída compactada

**Índices disponíveis:**NDVI, NDRE, GNDVI, OSAVI, CIG, EVI, SAVI, MSAVI, MTVI2 e outros.**Exemplo:**

```python
# Basic configuration
chloros.configure(
    vignette_correction=True,
    reflectance_calibration=True,
    indices=["NDVI", "NDRE"]
)

# Advanced configuration
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=True,
    export_format="TIFF (32-bit, Percent)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI", "CIG"]
)
```

***

#### `process(mode="parallel", wait=True, progress_callback=None)`

Processar as imagens do projeto.

**Parâmetros:**

| Parâmetro           | Tipo     | Padrão      | Descrição                               |
| ------------------- | -------- | ------------ | ----------------------------------------- |
| `mode`              | str      | `"parallel"` | Modo de processamento: &quot;paralelo&quot; ou &quot;serial&quot;   |
| `wait`              | bool     | `True`       | Aguardar conclusão                       |
| `progress_callback` | callable | `None`       | Função de retorno de chamada de progresso (progress, msg) |
| `poll_interval`     | float    | `2.0`        | Intervalo de sondagem para progresso (segundos)   |

**Retorna:** `dict` - Resultados do processamento

{% hint style="warning" %}
**Modo paralelo**: Requer licença Chloros+. Escala automaticamente para os núcleos da sua CPU (até 16 trabalhadores).
{% endhint %}

**Exemplo:**

```python
# Simple processing
results = chloros.process()

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

# Fire-and-forget (non-blocking)
chloros.process(wait=False)
```

***

#### `get_config()`

Obtém a configuração atual do projeto.

**Retorna:** `dict` - Configuração atual do projeto**Exemplo:**

```python
config = chloros.get_config()
print(config['Project Settings'])
```

***

#### `get_status()`

Obtém informações de status do backend, incluindo o progresso do processamento por thread.

**Retorna:** `dict` - Status do backend com a seguinte estrutura:

```python
{
    "running": True,
    "url": "http://localhost:5000",
    "processing": {
        "percent": 75.0,
        "phase": "processing"
    },
    "export": {
        "percent": 50.0,
        "phase": "exporting",
        "active": True
    }
}
```

**Exemplo:**

```python
status = chloros.get_status()
print(f"Running: {status['running']}")
print(f"URL: {status['url']}")
print(f"Processing: {status['processing']['percent']}%")
print(f"Export: {status['export']['percent']}% - Active: {status['export']['active']}")
```

***

#### `shutdown_backend()`

Desligar o backend (se iniciado por SDK).

**Exemplo:**

```python
chloros.shutdown_backend()
```

***

#### `logout()`

Limpa as credenciais armazenadas em cache do sistema local.

**Descrição:**

Desconecta programaticamente removendo as credenciais de autenticação armazenadas em cache. Isso é útil para:
* Alternar entre diferentes contas Chloros+
* Limpar credenciais em ambientes automatizados
* Fins de segurança (por exemplo, remover credenciais antes da desinstalação)

**Retorna:** `dict` - Resultado da operação de logout**Exemplo:**

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Clear cached credentials
result = chloros.logout()
print(f"Logout successful: {result}")

# After logout, login required via GUI/CLI/Browser before next SDK use
```

{% hint style="info" %}
**Reautenticação necessária**: Após chamar `logout()`, você deve fazer login novamente por meio de Chloros, Chloros (Navegador) ou Chloros CLI antes de usar o SDK.
{% endhint %}

***

### Funções de conveniência

#### `process_folder(folder_path, **options)`

Função de conveniência de uma linha para processar uma pasta.

**Parâmetros:**

| Parâmetro                 | Tipo     | Padrão         | Descrição                    |
| ------------------------- | -------- | --------------- | ------------------------------ |
| `folder_path`             | str/Path | Obrigatório        | Caminho para a pasta com imagens     |
| `project_name`            | str      | Gerado automaticamente  | Nome do projeto                   |
| `camera`                  | str      | `None`          | Modelo de câmera                |
| `indices`                 | lista     | `["NDVI"]`      | Índices a serem calculados           |
| `vignette_correction`     | bool     | `True`          | Ativar correção de vinheta     |
| `reflectance_calibration` | bool     | `True`          | Ativar calibração de refletância |
| `export_format`           | str      | &quot;TIFF (16 bits)&quot; | Formato de saída                  |
| `mode`                    | str      | `"parallel"`    | Modo de processamento                |
| `progress_callback`       | callable | `None`          | Callback de progresso              |

**Retorna:** `dict` - Resultados do processamento**Exemplo:**

```python
from chloros_sdk import process_folder

# Simple one-liner
results = process_folder("C:\\DroneImages\\Flight001")

# With custom settings
results = process_folder(
    "C:\\DroneImages\\Flight001",
    project_name="Field_A_Survey",
    camera="Survey3N_RGN",
    indices=["NDVI", "NDRE", "GNDVI"],
    mode="parallel"
)

# With progress monitoring
def show_progress(progress, message):
    print(f"[{progress}%] {message}")

results = process_folder(
    "C:\\DroneImages\\Flight001",
    progress_callback=show_progress
)
```

***

## Suporte a gerenciadores de contexto

O SDK oferece suporte a gerenciadores de contexto para limpeza automática:

```python
from chloros_sdk import ChlorosLocal

# Auto-cleanup when done
with ChlorosLocal() as chloros:
    chloros.create_project("MyProject")
    chloros.import_images("C:\\Images")
    chloros.configure(indices=["NDVI"])
    chloros.process()
# Backend automatically shut down here
```

***

## Exemplos completos

{% hint style="info" %}
**Usuários do Linux**: Todos os exemplos abaixo utilizam caminhos do Windows. Substitua os caminhos do `C:\\...` pelos seus caminhos do Linux (por exemplo, `/home/user/...` ou `~/...`). Todas as funcionalidades do SDK são idênticas em todas as plataformas.
{% endhint %}

### Exemplo 1: Processamento básico

Processe uma pasta com as configurações padrão:

```python
from chloros_sdk import process_folder

# Process with default settings
results = process_folder("C:\\Datasets\\Field_A_2025_01_15")

print(f"Processing complete: {results}")
```

***

### Exemplo 2: Fluxo de trabalho personalizado

Controle total sobre o pipeline de processamento:

```python
from chloros_sdk import ChlorosLocal

# Initialize SDK
chloros = ChlorosLocal()

# Create project with camera template
chloros.create_project("Research_Plot_A", camera="Survey3N_RGN")

# Import images
import_results = chloros.import_images("C:\\Research\\PlotA")
print(f"Imported {len(import_results.get('files', []))} images")

# Configure advanced settings
chloros.configure(
    debayer="Standard (Fast, Medium Quality)",
    vignette_correction=True,
    reflectance_calibration=True,
    ppk=False,
    export_format="TIFF (16-bit)",
    indices=["NDVI", "NDRE", "GNDVI", "OSAVI"]
)

# Process with progress monitoring
def show_progress(progress, message):
    print(f"Progress: {progress}% - {message}")

chloros.process(
    mode="parallel",
    progress_callback=show_progress,
    wait=True
)

print("Processing complete!")
```

***

### Exemplo 3: Processamento em lote de várias pastas

Processar vários conjuntos de dados de voos:

```python
from chloros_sdk import ChlorosLocal
from pathlib import Path

# Initialize SDK once
chloros = ChlorosLocal()

# List of flight folders
flights = [
    "C:\\Datasets\\Flight_001",
    "C:\\Datasets\\Flight_002",
    "C:\\Datasets\\Flight_003"
]

for flight_path in flights:
    flight_name = Path(flight_path).name
    print(f"\n{'='*60}")
    print(f"Processing: {flight_name}")
    print('='*60)
    
    try:
        # Create project
        chloros.create_project(flight_name, camera="Survey3N_RGN")
        
        # Import images
        chloros.import_images(flight_path)
        
        # Configure
        chloros.configure(
            vignette_correction=True,
            reflectance_calibration=True,
            indices=["NDVI", "NDRE", "GNDVI"]
        )
        
        # Process
        chloros.process(mode="parallel", wait=True)
        
        print(f"✓ {flight_name} completed successfully")
    
    except Exception as e:
        print(f"✗ {flight_name} failed: {e}")

print("\n" + "="*60)
print("All flights processed!")
```

***

### Exemplo 4: Integração do pipeline de pesquisa

Integrar o Chloros com a análise de dados:

```python
from chloros_sdk import ChlorosLocal
import pandas as pd
import matplotlib.pyplot as plt

# Initialize Chloros
chloros = ChlorosLocal()

# Field survey data
surveys = [
    {"name": "Plot_A", "folder": "C:\\Research\\PlotA", "biomass": 4500},
    {"name": "Plot_B", "folder": "C:\\Research\\PlotB", "biomass": 3800},
    {"name": "Plot_C", "folder": "C:\\Research\\PlotC", "biomass": 5200}
]

results = []

for survey in surveys:
    # Process with Chloros
    chloros.create_project(survey['name'])
    chloros.import_images(survey['folder'])
    chloros.configure(indices=["NDVI", "NDRE"])
    chloros.process(mode="parallel", wait=True)
    
    # Get results
    config = chloros.get_config()
    
    # Extract NDVI values (example - adjust based on your needs)
    # In real implementation, you would read the processed TIFF files
    
    results.append({
        'plot': survey['name'],
        'biomass': survey['biomass'],
        # Add your NDVI extraction here
    })

# Statistical analysis
df = pd.DataFrame(results)
print("\nResults:")
print(df)

# Create correlation plot
# plt.scatter(df['ndvi'], df['biomass'])
# plt.xlabel('NDVI')
# plt.ylabel('Biomass (kg/ha)')
# plt.title('NDVI vs Biomass Correlation')
# plt.show()
```

***

### Exemplo 5: Monitoramento personalizado do progresso

Acompanhamento avançado do progresso com registro em log:

```python
from chloros_sdk import ChlorosLocal
from datetime import datetime
import logging

# Setup logging
logging.basicConfig(
    filename=f'processing_{datetime.now():%Y%m%d_%H%M%S}.log',
    level=logging.INFO,
    format='%(asctime)s - %(message)s'
)

# Progress callback with logging
def log_progress(progress, message):
    log_msg = f"[{progress}%] {message}"
    logging.info(log_msg)
    print(log_msg)

# Process with logging
chloros = ChlorosLocal()
chloros.create_project("LoggedProcess")
chloros.import_images("C:\\DroneImages")
chloros.configure(indices=["NDVI", "NDRE"])

logging.info("Starting processing...")
chloros.process(
    mode="parallel",
    progress_callback=log_progress,
    wait=True
)
logging.info("Processing complete!")
```

***

### Exemplo 6: Tratamento de erros

Tratamento robusto de erros para uso em produção:

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import (
    ChlorosError,
    ChlorosBackendError,
    ChlorosLicenseError,
    ChlorosProcessingError
)

def process_safely(folder_path):
    """Process with comprehensive error handling"""
    try:
        with ChlorosLocal() as chloros:
            chloros.create_project("SafeProcess")
            chloros.import_images(folder_path)
            chloros.configure(indices=["NDVI"])
            chloros.process()
            
        return True, "Success"
    
    except ChlorosLicenseError as e:
        return False, f"License error: {e}. Upgrade to Chloros+ at cloud.mapir.camera/pricing"
    
    except ChlorosBackendError as e:
        return False, f"Backend error: {e}. Ensure Chloros is installed (Windows installer or Linux .deb package)."
    
    except ChlorosProcessingError as e:
        return False, f"Processing error: {e}"
    
    except FileNotFoundError as e:
        return False, f"Folder not found: {e}"
    
    except ChlorosError as e:
        return False, f"Chloros error: {e}"
    
    except Exception as e:
        return False, f"Unexpected error: {e}"

# Use the safe function
success, message = process_safely("C:\\DroneImages\\Flight001")
if success:
    print(f"✓ {message}")
else:
    print(f"✗ {message}")
```

***

### Exemplo 7: Gerenciamento de contas e logout

Gerencie credenciais programaticamente:

```python
from chloros_sdk import ChlorosLocal

def switch_account():
    """Clear credentials to switch to a different account"""
    try:
        chloros = ChlorosLocal()
        
        # Clear current credentials
        result = chloros.logout()
        print("✓ Credentials cleared successfully")
        print("Please log in with new account via Chloros, Chloros (Browser), or CLI")
        
        return True
    
    except Exception as e:
        print(f"✗ Logout failed: {e}")
        return False

def secure_cleanup():
    """Remove credentials for security purposes"""
    try:
        chloros = ChlorosLocal()
        chloros.logout()
        print("✓ Credentials removed for security")
        
    except Exception as e:
        print(f"Warning: Cleanup error: {e}")

# Switch accounts
if switch_account():
    print("\nRe-authenticate via Chloros GUI/CLI/Browser before next SDK use")

# Or perform secure cleanup
# secure_cleanup()
```

***

### Exemplo 8: Ferramenta de linha de comando

Crie uma ferramenta CLI personalizada com o SDK:

```python
#!/usr/bin/env python
"""
Custom Chloros CLI Tool
Process multiple folders from command line
"""

import sys
import argparse
from pathlib import Path
from chloros_sdk import process_folder

def main():
    parser = argparse.ArgumentParser(description='Custom Chloros Processor')
    parser.add_argument('folders', nargs='+', help='Folders to process')
    parser.add_argument('--indices', nargs='+', default=['NDVI'],
                       help='Indices to calculate (default: NDVI)')
    parser.add_argument('--camera', default=None,
                       help='Camera template')
    parser.add_argument('--format', default='TIFF (16-bit)',
                       help='Export format')
    parser.add_argument('--logout', action='store_true',
                       help='Clear cached credentials before processing')
    
    args = parser.parse_args()
    
    # Handle logout if requested
    if args.logout:
        from chloros_sdk import ChlorosLocal
        chloros = ChlorosLocal()
        chloros.logout()
        print("Credentials cleared. Please re-login via Chloros GUI/CLI/Browser.")
        return 0
    
    successful = []
    failed = []
    
    for folder in args.folders:
        folder_path = Path(folder)
        
        if not folder_path.exists():
            print(f"✗ Skipping {folder}: not found")
            failed.append(folder)
            continue
        
        print(f"\nProcessing: {folder_path.name}...")
        
        try:
            process_folder(
                folder_path,
                camera=args.camera,
                indices=args.indices,
                export_format=args.format
            )
            print(f"✓ {folder_path.name} complete")
            successful.append(folder)
        
        except Exception as e:
            print(f"✗ {folder_path.name} failed: {e}")
            failed.append(folder)
    
    # Summary
    print(f"\n{'='*60}")
    print(f"Summary: {len(successful)} successful, {len(failed)} failed")
    
    return 0 if not failed else 1

if __name__ == '__main__':
    sys.exit(main())
```

**Uso:**

```bash
# Process multiple folders
python my_processor.py "C:\Flight001" "C:\Flight002" --indices NDVI NDRE GNDVI

# Clear cached credentials
python my_processor.py --logout
```

***

## Tratamento de exceções

O SDK fornece classes de exceção específicas para diferentes tipos de erro:

### Hierarquia de exceções

```python
ChlorosError                    # Base exception
├── ChlorosBackendError        # Backend startup/connection issues
├── ChlorosLicenseError        # License validation issues
├── ChlorosConnectionError     # Network/connection failures
├── ChlorosProcessingError     # Image processing failures
├── ChlorosAuthenticationError # Authentication failures
└── ChlorosConfigurationError  # Configuration errors
```

### Exemplos de exceções

```python
from chloros_sdk import ChlorosLocal
from chloros_sdk.exceptions import *

try:
    chloros = ChlorosLocal()
    chloros.process()

except ChlorosLicenseError:
    print("Chloros+ license required. Upgrade at cloud.mapir.camera/pricing")

except ChlorosBackendError:
    print("Backend failed to start. Ensure Chloros is installed (Windows installer or Linux .deb package).")

except ChlorosProcessingError as e:
    print(f"Processing failed: {e}")

except ChlorosError as e:
    print(f"General Chloros error: {e}")
```

***

## Tópicos avançados

### Configuração personalizada do backend

Use um local ou configuração personalizada do backend:

```python
chloros = ChlorosLocal(
    backend_exe="C:\\Custom\\chloros-backend.exe",
    api_url="http://localhost:5001",  # Custom port
    timeout=60,                        # Longer timeout
    backend_startup_timeout=120        # 2 minutes startup
)
```

### Processamento sem bloqueio

Inicie o processamento e continue com outras tarefas:

```python
# Start processing (non-blocking)
chloros.process(wait=False)

# Do other work here...
print("Processing started in background...")

# Check status later
import time
while True:
    status = chloros.get_config()
    if status.get('processing_complete'):
        break
    time.sleep(5)

print("Processing complete!")
```

### Gerenciamento de memória

Para conjuntos de dados grandes, processe em lotes:

```python
from pathlib import Path

base_folder = Path("C:\\LargeDataset")
batch_size = 100

# Get all image files
images = list(base_folder.glob("*.RAW"))

# Process in batches
for i in range(0, len(images), batch_size):
    batch = images[i:i+batch_size]
    batch_folder = base_folder / f"batch_{i//batch_size}"
    
    # Create batch folder and move images
    # ... (implementation details)
    
    # Process batch
    process_folder(batch_folder)
```

***

## Solução de problemas

### Backend não inicia

**Problema:** SDK não consegue iniciar o backend**Soluções:**

1. Verifique se o Chloros está instalado:

```python
import os
import platform

# Auto-detect backend path
if platform.system() == "Windows":
    backend_path = r"C:\Program Files\MAPIR\Chloros\resources\backend\chloros-backend.exe"
else:
    backend_path = "/usr/lib/chloros/chloros-backend"

print(f"Backend exists: {os.path.exists(backend_path)}")
```

2. Verifique o firewall (Windows) ou a disponibilidade da porta (Linux: `lsof -i :5000`)
3. Tente o caminho manual do backend:

```python
# Windows
chloros = ChlorosLocal(backend_exe="C:\\Path\\To\\chloros-backend.exe")

# Linux
chloros = ChlorosLocal(backend_exe="/opt/mapir/chloros/backend/chloros-backend")
```

***

### Licença não detectada**Problema:** SDK avisa sobre a falta de licença**Soluções:**

1. Abra o Chloros, o Chloros (navegador) ou o Chloros CLI e faça login.
2. Verifique se a licença está armazenada em cache:

```python
from pathlib import Path
import os
import platform

# Check cache location
if platform.system() == "Windows":
    cache_path = Path(os.getenv('APPDATA')) / 'Chloros' / 'cache'
else:
    cache_path = Path.home() / '.cache' / 'chloros'

print(f"Cache exists: {cache_path.exists()}")
```

3. Se estiver enfrentando problemas com credenciais, limpe as credenciais armazenadas em cache e faça login novamente:

```python
from chloros_sdk import ChlorosLocal

# Clear cached credentials
chloros = ChlorosLocal()
chloros.logout()

# Then login again via Chloros, Chloros (Browser), or Chloros CLI
```

4. Entre em contato com o suporte: info@mapir.camera

***

### Erros de importação**Problema:** `ModuleNotFoundError: No module named 'chloros_sdk'`**Soluções:**

```bash
# Verify installation
pip show chloros-sdk

# Reinstall if needed
pip uninstall chloros-sdk
pip install chloros-sdk

# Check Python environment
python -c "import sys; print(sys.path)"
```

***

### Tempo limite de processamento**Problema:** Tempo limite de processamento expirado**Soluções:**

1. Aumente o tempo limite:

```python
chloros = ChlorosLocal(timeout=120)  # 2 minutes
```

2. Processe lotes menores
3. Verifique o espaço disponível em disco
4. Monitore os recursos do sistema

***

### Porta já em uso**Problema:** Porta 5000 do backend ocupada**Soluções:**

```python
# Use different port
chloros = ChlorosLocal(api_url="http://localhost:5001")
```

Ou localize e feche o processo em conflito:

```powershell
# Windows PowerShell
Get-NetTCPConnection -LocalPort 5000
```

```bash
# Linux
lsof -i :5000
kill $(lsof -t -i :5000)
```

***

## Dicas de desempenho

### Otimize a velocidade de processamento

1. **Use o modo paralelo** (requer Chloros+)

```python
chloros.process(mode="parallel")  # Up to 16 workers
```

2. **Reduza a resolução de saída** (se aceitável)

```python
chloros.configure(export_format="PNG (8-bit)")  # Faster than TIFF
```

3. **Desative índices desnecessários**

```python
# Only calculate needed indices
chloros.configure(indices=["NDVI"])  # Not all indices
```

4. **Processe em SSD** (não em HDD)***

### Otimização de memória

Para conjuntos de dados grandes:

```python
# Process in batches instead of all at once
# See "Memory Management" in Advanced Topics
```

***

### Processamento em segundo plano

Libere Python para outras tarefas:

```python
chloros.process(wait=False)  # Non-blocking

# Continue with other work
# ...
```

***

## Exemplos de integração

### Integração com Django

```python
# views.py
from django.http import JsonResponse
from chloros_sdk import process_folder

def process_images_view(request):
    if request.method == 'POST':
        folder_path = request.POST.get('folder_path')
        
        try:
            results = process_folder(folder_path)
            return JsonResponse({'success': True, 'results': results})
        except Exception as e:
            return JsonResponse({'success': False, 'error': str(e)})
```

### Flask API

```python
# app.py
from flask import Flask, request, jsonify
from chloros_sdk import process_folder

app = Flask(__name__)

@app.route('/api/process', methods=['POST'])
def process():
    data = request.get_json()
    folder_path = data.get('folder_path')
    
    try:
        results = process_folder(folder_path)
        return jsonify({'success': True, 'results': results})
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 500

if __name__ == '__main__':
    app.run()
```

### Jupyter Notebook

```python
# notebook.ipynb
from chloros_sdk import ChlorosLocal
import matplotlib.pyplot as plt

# Initialize
chloros = ChlorosLocal()

# Process
chloros.create_project("JupyterTest")
chloros.import_images("C:\\Data")
chloros.configure(indices=["NDVI"])

# Progress in notebook
from IPython.display import clear_output

def notebook_progress(progress, message):
    clear_output(wait=True)
    print(f"Progress: {progress}%")
    print(message)

chloros.process(progress_callback=notebook_progress)

# Visualize results
# ... (your visualization code)
```

***

## Perguntas frequentes

### P: O SDK requer conexão com a internet?

**R:** Apenas para a ativação inicial da licença. Após fazer login via Chloros, Chloros (Navegador) ou Chloros CLI, a licença é armazenada em cache localmente e funciona offline por 30 dias.***

### P: Posso usar o SDK em um servidor sem GUI?**R:** Sim! O SDK funciona sem interface gráfica tanto em servidores Windows quanto em servidores Linux.**Linux (recomendado para modo headless):**
* Instale através do pacote `.deb`
* Ative a licença: `chloros-cli login user@example.com 'password'`

**Servidor Windows:**
* Servidor Windows 2016 ou posterior
* Chloros instalado (uma única vez)
* Licença ativada via CLI ou em qualquer máquina

***

### P: Qual é a diferença entre Desktop, CLI e SDK?

| Recurso         | GUI do Desktop | Linha de comando do CLI | Python SDK  |
| --------------- | ----------- | ---------------- | ----------- |
| **Interface**   | Apontar e clicar | Comando          | Python API  |
| **Ideal para**    | Trabalho visual | Scripts        | Integração |
| **Automação**  | Limitada     | Boa             | Excelente   |
| **Flexibilidade** | Básica       | Boa             | Máxima     |
| **Licença**     | Chloros+    | Chloros+         | Chloros+    |***

### P: Posso distribuir aplicativos criados com o SDK?**R:** O código do SDK pode ser integrado aos seus aplicativos, mas:

* Os usuários finais precisam ter o Chloros instalado
* Os usuários finais precisam de licenças ativas do Chloros+
* A distribuição comercial requer licenciamento OEM

Entre em contato com o info@mapir.camera para consultas sobre OEM.

***

### P: Como atualizo o SDK?

```bash
pip install --upgrade chloros-sdk
```

***

### P: Onde as imagens processadas são salvas?

Por padrão, no Caminho do Projeto:

```

Project_Path/
└── MyProject/
    └── Survey3N_RGN/          # Processed outputs
```

***

### P: Posso processar imagens a partir de scripts do Python executados em horários programados?**R:** Sim! Use o agendador do seu sistema operacional com os scripts do Python:

```python
# scheduled_processing.py
from chloros_sdk import process_folder

# Process today's flights
results = process_folder("/data/flights/today")  # Linux
# results = process_folder("C:\\Flights\\Today")  # Windows
```

**Windows:** Programe via Agendador de Tarefas para execução diária.**Linux:** Programe via cron:

```cron
# Run at 2 AM daily
0 2 * ** /usr/bin/python3 /home/user/scheduled_processing.py >> /var/log/chloros.log 2>&1
```

***

### P: O SDK suporta async/await?**R:** A versão atual é síncrona. Para comportamento assíncrono, use o `wait=False` ou execute em thread separado:

```python
import threading

def process_thread():
    chloros.process()

thread = threading.Thread(target=process_thread)
thread.start()

# Continue with other work...
```

***

### P: Como faço para alternar entre diferentes contas Chloros+?**R:** Use o método `logout()` para limpar as credenciais armazenadas em cache e, em seguida, faça login novamente com a nova conta:

```python
from chloros_sdk import ChlorosLocal

# Clear current credentials
chloros = ChlorosLocal()
chloros.logout()

# Re-login via Chloros, Chloros (Browser), or Chloros CLI with new account
```

Após o logout, autentique-se com a nova conta via GUI, navegador ou CLI antes de usar o SDK novamente.

***

## Obter Ajuda

### Documentação

* **Referência do API**: Esta página

### Canais de suporte

* **E-mail**: info@mapir.camera
* **Site**: [https://www.mapir.camera/community/contact](https://www.mapir.camera/community/contact)
* **Preços**: [https://cloud.mapir.camera/pricing](https://cloud.mapir.camera/pricing)

### Código de exemplo

Todos os exemplos listados aqui foram testados e estão prontos para produção. Copie-os e adapte-os para o seu caso de uso.

***

## Licença**Software proprietário** - Copyright (c) 2025 MAPIR Inc.

O SDK requer uma assinatura ativa do Chloros+. É proibido o uso, a distribuição ou a modificação não autorizados.
