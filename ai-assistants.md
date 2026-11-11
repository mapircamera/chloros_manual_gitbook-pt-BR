# Como usar o Chloros com assistentes de IA

Este manual foi elaborado para dois públicos: pessoas e os assistentes de IA com os quais as pessoas trabalham cada vez mais. Cada página apresenta valores exatos, configurações padrão e comandos que podem ser copiados e colados, para que um assistente (Claude, ChatGPT, Copilot, um agente de programação, etc.) possa criar uma automação funcional no Chloros logo na primeira tentativa.

Versão do Chloros: **

1.3.0**. Plataformas CLI/SDK: Windows 10/11 x64 e Linux (x86_64 / Jetson aarch64).

## O que fornecer ao seu assistente

| Recurso | URL | Para que serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legível por máquina de todas as páginas deste manual. |
| **Referência da CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A superfície de comandos completa do `chloros-cli`: todos os comandos, sinalizadores, valores padrão, códigos de saída e regras de pastas de saída. Escrita para uso por LLMs. |
| **Referência do SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | A API completa do `chloros_sdk` em Python: classes, assinaturas, exceções e exemplos práticos. Escrito para uso por LLM. |
| **Qualquer página como Markdown bruto** | acrescente `.md` ao URL da página | por exemplo, `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` retorna a página como Markdown bruto — ideal para colar em uma janela de contexto ou obter de um agente. |

Links no manual: [Referência da CLI](reference/cli-reference.md) · [Referência do SDK](reference/sdk-reference.md).

{% hint style="info" %}
As duas páginas de referência são autônomas: um assistente que tenha lido uma delas não precisa do restante do manual para escrever um script correto.
{% endhint %}

## Deixe seu assistente executar o Chloros diretamente: `chloros-mcp`

Tudo o que foi descrito acima faz com que seu assistente *escreva* um script ou um comando. Com o `chloros-mcp`, um assistente que utiliza o [Model Context Protocol](https://modelcontextprotocol.io) (MCP), como o Claude Desktop, o Claude Code ou o Cursor, controla o próprio Chloros. Você faz a solicitação em linguagem natural e ele aciona as ferramentas do servidor:

* **Processar imagens:** criar um projeto, importar uma pasta de capturas, escolher configurações, processar e gerar um relatório com os resultados exportados.
* **Operar hardware:** conectar câmeras LATTICE, matrizes e sensores de luz DAQ, visualizar, definir exposição e cor, capturar, gravar e atualizar o firmware.

```text
You:  Process the flight in D:\Flights\2026-10-17 to reflectance and NDVI.

You:  Show me what the array sees, then capture reflectance every 10 seconds
      for 5 minutes.
```

As ferramentas do servidor abrangem o que o [Python SDK](api-python-sdk.md) faz. Elas são instaladas junto com o SDK, portanto, geralmente não há nada mais a instalar: no Windows, o instalador do Chloros coloca o `chloros-mcp.exe` na pasta `Scripts` do seu Python, e, no Linux, o `.deb` coloca o `chloros-mcp` em `/usr/bin`.

**Código do Claude:**

```bash
claude mcp add chloros -- chloros-mcp
```

**Claude Desktop:**adicione isso a `claude_desktop_config.json` (**Configurações > Desenvolvedor > Editar Configuração**abre o arquivo) e, em seguida, reinicie o Claude Desktop. O**cursor** recebe a mesma entrada em `~/.cursor/mcp.json`.

```json
{
  "mcpServers": {
    "chloros": {"command": "chloros-mcp"}
  }
}
```

Se o cliente informar que o servidor não conseguiu iniciar, provavelmente o `chloros-mcp` não está no `PATH` que ele vê: forneça o caminho completo (`where chloros-mcp` no Windows, `which chloros-mcp` no Linux).

É necessário ter o Chloros instalado no mesmo computador e estar conectado nele, seja no aplicativo ou via `chloros-cli login`, em qualquer plano pago do Chloros+ (Copper ou superior). O servidor não pode fazer o login por você.

{% hint style="info" %}
Configuração do cliente para cada aplicativo, opções, todas as ferramentas e como uma execução se comporta: [Servidor MCP (Assistentes de IA)](reference/sdk-reference.md#mcp-server-ai-assistants) na Referência do SDK.
{% endhint %}

## Receitas de prompts

Copie, preencha o `<placeholders>` e cole no seu assistente.

### 1. Processar uma pasta de voo para NDVI

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md.
Then write a script for <Windows PowerShell | bash> that:
1. logs in with `chloros-cli login <email> '<password>'` (only needed once per machine),
2. processes the folder <path/to/flight_001> with reflectance and the NDVI index,
3. prints where each output product landed, using the reference's
   "Where the outputs land" folder rules.
```

### 2. Monitorar em lote um diretório de capturas

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (sections
"Quickstart" and "Post-Run Summary & Hints"). Write a Python script that
watches <path/to/captures> for new flight subfolders and runs
chloros_sdk.process_folder() with indices=["NDVI"] on each new one.
After each run, print every hint from result["summary"]["hints"] and treat
a run with zero image products as a failure for that folder.
```

### 3. Conectar uma matriz LATTICE e capturar

```

Read https://mapir.gitbook.io/chloros/reference/sdk-reference.md (section
"connect_array"). Write a Python script that connects my LATTICE cameras
with serials <213800234, 214000533, ...> as one synchronized array, captures
a reflectance image set into <output/> every 10 seconds for one hour, and
disconnects cleanly when done (use the context-manager form).
```

### 4. Registrar espectros do sensor de luz DAQ

```

Read https://mapir.gitbook.io/chloros/reference/cli-reference.md (section
"chloros-cli daq" — use only the pool-* commands). Write a script that:
1. connects my DAQ-E sensor with `chloros-cli daq pool-connect --eth-host <daq-e-xxxxxx.local>`,
2. lists the pool with `pool-list` to get the sensor id,
3. records a 10-minute calibrated .daq file named "<field-A>" with `pool-record`,
4. disconnects with `pool-disconnect`.
```

{% hint style="warning" %}
A criação de scripts para o DAQ a partir da linha de comando sempre passa pela família `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Outros subcomandos do tipo `daq` que seu assistente possa inventar não estão disponíveis nas versões lançadas e geram um erro ao serem executados.
{% endhint %}

## Por que os scripts escritos por IA funcionam bem com o Chloros

Cada um desses é um comportamento real e comprovado do Chloros 1.3.0 — eles eliminam os modos clássicos de falha da automação escrita por máquinas:

* **Sem complicações de configuração.**Os auxiliares de conexão inteligente do SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e os pontos de entrada de processamento (`ChlorosLocal`, `process_folder`)**iniciam automaticamente o backend local**. Um script gerado não precisa que a GUI esteja aberta nem de um servidor iniciado manualmente — basta que o pacote para desktop ou CLI esteja instalado.
* **Todo o pipeline é executado em uma única chamada.** O `chloros_sdk.process_folder("path", indices=["NDVI"])` executa importação → calibração → refletância → exportação do índice de ponta a ponta. Menos etapas envolvidas, menos pontos em que um script gerado pode apresentar falhas.
* **Execuções sem saída realizam autodiagnóstico.** Após o `process()`, o resumo da execução é anexado ao resultado, e todas as dicas de processamento (por exemplo, *por que* uma execução não produziu saída) também é reemitida como um `UserWarning` em Python — assim, mesmo um script que nunca inspeciona o dicionário de resultados exibe o diagnóstico.
* **A CLI falha de forma evidente.**Uma execução `chloros-cli process` que solicitou produtos, mas não gravou nenhum, exibe `Processing finished but wrote no image products.` e**encerra com código de saída diferente de zero**, de modo que scripts de shell e CI detectam isso com uma simples verificação do código de saída. Execuções bem-sucedidas relatam o código `Image products written: N`, além do `(plus M clamp masks)` quando máscaras foram gravadas. Da mesma forma, os comandos de captura listam todos os arquivos que não conseguiram gravar e saem com o código `1`.
* **Caminhos relativos significam o que um script espera.** Com um backend local, um caminho de saída relativo (`-o out/` na CLI, `output_dir="out/"` no SDK) é resolvido em relação à pasta a partir da qual o comando ou script é executado.

Uma assimetria que um assistente deve conhecer: o `process()` do SDK deliberadamente **não** gera exceção em uma execução sem produto — em vez disso, ele relata o problema por meio do resumo/dicas. Se um pipeline em Python precisar ser interrompido em uma execução vazia, verifique o resumo (a receita 2 faz isso).

## Considerações

* **É necessário fazer login no Chloros+.**A CLI e o SDK exigem um plano**pago** do Chloros+, com verificação do lado do servidor: as solicitações falham com o código de erro `401 AUTH_REQUIRED` quando não há login e com o código `403 PLAN_UPGRADE_REQUIRED` no plano gratuito. Execute o comando `chloros-cli login` uma vez por máquina antes de executar os scripts gerados. Consulte [Login no Chloros+](chloros+-login.md).
* **Os comandos de captura controlam hardware real.** `lattice` / `daq` / `project`, os objetos de sessão do SDK e as ferramentas do `chloros-mcp` se conectam, transmitem e acionam câmeras e sensores físicos. Revise um script gerado antes de sua primeira execução e execute-o com alguém supervisionando o hardware.
* **Faça uma verificação aleatória das saídas.** Verifique as pastas do produto e alguns valores de pixels antes de publicar os resultados. Em particular, os arquivos TIFF de refletância são dimensionados por fonte — leia a tag XMP do `Chloros:PixelScale` (os arquivos LATTICE a contêm: 32768 = 1,0 de refletância; os arquivos Survey3 não a possuem e usam 65535 = 1,0) em vez de supor um divisor. Ambas as páginas de referência documentam isso na seção “Lendo pixels de refletância”.
* **Pequenas armadilhas que prejudicam o código gerado:**o `pool-record` grava no sistema de arquivos do**host de backend** (padrão: `~/Documents/DAQ Live View/`); em máquinas com várias interfaces de rede, prefira o `daq pool-connect --eth-host <ip-or-hostname>` em vez da descoberta automática; use o `http://127.0.0.1:5000` (nunca o `localhost`) sempre que aparecer um URL de backend; um script que percorre a árvore de saída de um projeto deve ignorar as pastas `clamp_masks/` (seus arquivos `*_clampmask.tif` são máscaras, não imagens — consulte [Formatos de imagem de saída](output-image-formats.md#clamp-masks)); e no Windows, com caminhos longos desativados, mantenha as pastas do projeto curtas — uma pasta de projeto com mais de cerca de 130 caracteres é rejeitada antes do início do processamento.
