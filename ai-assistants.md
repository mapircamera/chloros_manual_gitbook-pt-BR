# Como usar o Chloros com assistentes de IA

Este manual foi elaborado para dois públicos: pessoas e os assistentes de IA com os quais as pessoas trabalham cada vez mais. Cada página apresenta valores exatos, configurações padrão e comandos que podem ser copiados e colados, para que um assistente (Claude, ChatGPT, Copilot, um agente de programação, etc.) possa criar uma automação Chloros funcional logo na primeira tentativa.

Versão do Chloros: **

1.2.0**. Plataformas do CLI/SDK: Windows 10/11 x64 e Linux (x86_64 / Jetson aarch64).

## O que entregar ao seu assistente

| Recurso | URL | Para que serve |
| --- | --- | --- |
| **llms.txt** | `https://mapir.gitbook.io/chloros/llms.txt` | Índice legível por máquina de todas as páginas deste manual. |
| **Referência do CLI** | `https://mapir.gitbook.io/chloros/reference/cli-reference` | A superfície de comando completa do `chloros-cli`: todos os comandos, sinalizadores, valores padrão, códigos de saída e regras de pasta de saída. Escrito para uso por LLM. |
| **Referência do SDK** | `https://mapir.gitbook.io/chloros/reference/sdk-reference` | O `chloros_sdk` Python API completo: classes, assinaturas, exceções e exemplos práticos. Escrito para uso por LLMs. |
| **Qualquer página como Markdown bruto** | acrescente `.md` à página URL | por exemplo `https://mapir.gitbook.io/chloros/reference/sdk-reference.md` retorna a página como Markdown bruto — ideal para colar em uma janela de contexto ou recuperar de um agente. |

Links no manual: [Referência CLI](reference/cli-reference.md) · [Referência de SDK](reference/sdk-reference.md).

{% hint style="info" %}
As duas páginas de referência são autônomas: um assistente que tenha lido uma delas não precisa do restante do manual para escrever um script correto.
{% endhint %}

## Receitas de prompt

Copie, preencha o `<placeholders>` e cole em seu assistente.

### 1. Processar uma pasta de voos no NDVI

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

### 3. Conectar um array LATTICE e capturar

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
A criação de scripts para o DAQ a partir da linha de comando sempre passa pela família `daq pool-*` (`pool-connect`, `pool-list`, `pool-latest`, `pool-stream`, `pool-record`, `pool-set-cap`, `pool-disconnect`). Outros subcomandos do tipo `daq` que seu assistente possa inventar não estão disponíveis nas versões fornecidas e geram um erro ao serem executados.
{% endhint %}

## Por que os scripts escritos por IA funcionam bem com o Chloros

Cada um desses é um comportamento real e comprovado do Chloros 1.2.0 — eles eliminam os modos clássicos de falha da automação escrita por máquinas:

* **Sem complicações de configuração.**Os auxiliares de conexão inteligente do SDK (`connect_camera`, `connect_array`, `connect_daq_sensor`) e os pontos de entrada de processamento (`ChlorosLocal`, `process_folder`)**inicializam automaticamente o backend local**. Um script gerado não precisa que a GUI esteja aberta nem de um servidor iniciado manualmente — basta que o pacote desktop/CLI esteja instalado.
* **Todo o pipeline é uma única chamada.** O `chloros_sdk.process_folder("path", indices=["NDVI"])` executa importação → calibração → refletância → exportação do índice de ponta a ponta. Menos áreas de interface, menos pontos em que um script gerado pode dar errado.
* **Execuções sem saída realizam autodiagnóstico.** Após o `process()`, o resumo da execução é anexado ao resultado, e todas as dicas de processamento (por exemplo, *por que* uma execução não produziu saída) também é reemitida como um Python `UserWarning` — assim, mesmo um script que nunca inspecione o dicionário de resultados exibe o diagnóstico.
* **O CLI falha de forma evidente.**Uma execução do `chloros-cli process` que solicitou produtos, mas não gerou nenhum, exibe o código `Processing finished but wrote no image products.` e**encerra com código de saída diferente de zero**; assim, scripts de shell e CI detectam isso com uma simples verificação do código de saída. Execuções bem-sucedidas relatam o código `Image products written: N`.

Uma assimetria que um assistente deve conhecer: o `process()` do SDK deliberadamente **não** gera exceção em uma execução com zero produto — em vez disso, ele reporta por meio do resumo/dicas. Se um pipeline Python precisar ser interrompido em uma execução vazia, verifique o resumo (a receita 2 faz isso).

## Advertências

* **É necessário fazer login no Chloros+.**O CLI e o SDK exigem um plano**pago** Chloros+, aplicada no lado do servidor: as solicitações falham com o código de erro `401 AUTH_REQUIRED` quando não houver login e com o código `403 PLAN_UPGRADE_REQUIRED` no plano gratuito. Execute o `chloros-cli login` uma vez por máquina antes de executar os scripts gerados. Consulte [Chloros+ Login](chloros+-login.md).
* **Os comandos de captura controlam o hardware real.** Os comandos `lattice` / `daq` / `project` e os objetos de sessão SDK conectam, transmitem e acionam câmeras e sensores físicos. Revise um script gerado antes de sua primeira execução e execute-o com o hardware sob supervisão.
* **Faça uma verificação aleatória dos resultados.** Verifique as pastas do produto e alguns valores de pixels antes de publicar os resultados. Em particular, os arquivos TIFF de refletância são dimensionados por fonte — leia a tag XMP do `Chloros:PixelScale` (LATTICE: 32768 = 1,0 de refletância; Survey3: 65535) em vez de supor um divisor. Ambas as páginas de referência documentam isso na seção “Lendo pixels de refletância”.
* **Pequenas armadilhas que prejudicam o código gerado:**`pool-record` grava no sistema de arquivos do**host de backend** (padrão `~/Documents/DAQ Live View/`); em máquinas com várias interfaces de rede, prefira o `daq pool-connect --eth-host <ip-or-hostname>` em vez da descoberta automática; e use o `http://127.0.0.1:5000` (nunca o `localhost`) em qualquer lugar onde aparecer um backend URL.
