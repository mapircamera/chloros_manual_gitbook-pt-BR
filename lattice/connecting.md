# Conectando câmeras

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>A guia “Câmeras” antes de conectar qualquer dispositivo</p></figcaption></figure>O

Chloros
detecta automaticamente as câmeras LATTICE na rede — pela guia “Câmeras” da interface gráfica, pelo `chloros-cli lattice` ou pelo Python
SDK
. A string do modelo da câmera determina tudo o que vem a seguir: o Chloros
identifica o perfil do sensor, o layout das bandas e a calibração de fábrica a partir dos arquivos `DeviceUserID` + `DeviceSerialNumber` da câmera, portanto, **não há nada a ser configurado por câmera**.

Antes de conectar, certifique-se de que a rede do host esteja configurada — endereçamento link-local, quadros jumbo e, para matrizes, as configurações do buffer de recepção da placa de rede. Essa é a configuração do lado do hardware e está descrita no manual da LATTICE: [**Configuração de Rede**](https://mapir.gitbook.io/lattice-camera/setup/network-setup).

## Conectando-se pela GUI

Abra a aba **Câmeras** na barra lateral do Chloros
(as abas de hardware aparecem assim que o backend terminar de iniciar) ou use o menu principal → **Conectar à Câmera**. Ambas abrem a caixa de diálogo**Conectar Câmera(s)**.

### A caixa de diálogo **Conectar câmera(s)**A caixa de diálogo verifica a rede assim que é aberta (“Verificando a rede...”) e lista todas as câmeras encontradas. Cada linha mostra o**modelo**da câmera (por exemplo, `LATT-M3M-L41-F550`), o**número de série**e o**endereço IP**.

* **Clique em uma linha para selecioná-la**(destaque em verde). Você pode selecionar**várias câmeras** e conectá-las de uma só vez — o Chloros
as conecta em sequência.
* As linhas com o ícone **“Conectado”** já estão conectadas e não podem ser selecionadas novamente.
* As linhas com o ícone **“Em matriz”** pertencem a uma matriz de câmeras atualmente conectada. Desconecte primeiro o conjunto para usar essa câmera de forma independente.
* **Conectar** — conecta a(s) câmera(s) selecionada(s); o botão exibe uma contagem, por exemplo, “Conectar (3)”, quando mais de uma está selecionada.
* **Reexecutar varredura** — executa a varredura novamente.
* **Fechar** — fecha a caixa de diálogo.
* Se a varredura terminar sem resultados, a caixa de diálogo exibirá **“Nenhuma câmera encontrada na rede”** — consulte [Solução de problemas](connecting.md#troubleshooting) abaixo.

<figure><img src="../.gitbook/assets/image (38).png" alt=""><figcaption><p>A caixa de diálogo “Conectar câmera(s)” — mostrada aqui sem câmeras na rede</p></figcaption></figure>

### Primeira conexão: download do pacote de calibração

Na **primeira vez** que uma determinada câmera é conectada a uma máquina, o Chloros
obtém o pacote de calibração de fábrica da câmera (\~3,8 MB) diretamente da própria câmera via GigE. Enquanto isso ocorre, a caixa de diálogo exibe um painel verde com a mensagem **“Baixando dados de calibração da câmera”**e uma barra de progresso por porta serial — espere cerca de**70 segundos** por câmera. Com mais de seis câmeras, as linhas de progresso rolam dentro do painel. O pacote é armazenado em cache no host; portanto, conexões posteriores da mesma câmera pulam o download por completo (e nunca exibem o painel).

### Analisar sistema

O botão **Analisar Sistema** da caixa de diálogo verifica o host e a rede (o rótulo exibe “Analisando...” enquanto o processo está em andamento) e gera um relatório de diagnóstico:

* **Host** — núcleos da CPU e RAM; nome e memória da GPU, ou “GPU: Nenhuma detectada”.
* **Interfaces de rede** — nome de cada placa de rede, velocidade do link, MTU (com a tag “jumbo” quando ativa), estado de ativação/desativação e se está conectada a um barramento USB.
* **Câmeras**— número de série, modelo, IP e**em qual placa de rede cada câmera está conectada**.
* **Desempenho** — fps atual versus ideal por câmera para o formato de pixel, com uma linha verde “Potencial: melhoria de N× possível” quando o ideal excede o atual.
* **Avisos e recomendações numeradas** — ou “O sistema parece estar em boas condições para o número atual de câmeras.” quando não há nada a ser corrigido.

Execute-o sempre que a descoberta ou a transmissão apresentarem comportamento inesperado — ele identifica a maioria dos problemas relacionados às placas de rede (MTU incorreto, câmera na interface errada, limites do adaptador USB) sem sair da caixa de diálogo.

### Conectando um conjunto

Para conectar duas ou mais câmeras como um **conjunto sincronizado**, use o assistente de conexão de conjunto (**Conectar Conjunto de Câmeras**): ele orienta na seleção de mestre/escravo (preenchida automaticamente por um teste de cabo de sincronização — qualquer câmera conectada por cabo pode ser mestre, e a câmera RGB
`-FRGB` é pré-selecionada quando conectada por cabo), a escolha do modo de exibição (blocos separados vs. combinados) e uma tela de configurações do conjunto com uma projeção em tempo real dos fps alcançáveis e da largura de banda do cabo antes de confirmar. O assistente e os fluxos de trabalho do conjunto são abordados na seção sobre conjuntos de múltiplas câmeras deste manual; o equivalente na CLI
é o “Fluxo de Trabalho de Primeira Conexão da Câmera LATTICE” na [Referência da CLI
](../reference/cli-reference.md).

## Conexão pela CLI
e pelo SDK
O acesso à

CLI
e ao SDK
requer um plano pago do Chloros
+ e que o usuário esteja conectado; isso é imposto pelo servidor (`401 AUTH_REQUIRED` quando não estiver conectado, `403 PLAN_UPGRADE_REQUIRED` no plano gratuito).

```bash
# List cameras on the network (vendor, model, serial, IP, MAC)
chloros-cli lattice info

# Single-camera smoke test: capture one frame (saves every applicable export type)
chloros-cli lattice capture -o output/

# Connect a synchronized array — same smart-prep flow as the GUI
chloros-cli lattice array-connect --serials 213800234,214000533
```

```python
import chloros_sdk

# Persistent live-camera session through the backend
with chloros_sdk.connect_camera("213800234") as cam:
    ...

# Array session (smart-prep: network probe, tier auto-pick, PTP, AE seeding, trigger config)
with chloros_sdk.connect_array(["213800234", "214000533"]) as array:
    ...
```

Assinaturas completas, opções e fluxos de trabalho de captura: [Referência da CLI
](../reference/cli-reference.md) § `chloros-cli lattice`, [Referência do SDK
](../reference/sdk-reference.md) § `connect_camera()` / `connect_array()`.

## Como a calibração é resolvida na conexão

Cada câmera LATTICE possui seu pacote de calibração de fábrica **na própria câmera**, e o Chloros
também verifica a nuvem da MAPIR
quando a câmera se conecta:

| Situação   | O que o Chloros
usa                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Online**| A**calibração mais recente publicada para aquele número de série** — a cópia na nuvem prevalece sobre a cópia na própria câmera. Uma câmera que tenha sido recalibrada ou atualizada pela MAPIR
, portanto, se atualiza automaticamente; não é necessária nenhuma ação do usuário. |
| **Offline**| O**pacote da câmera**, tal como está. Fluxos de trabalho totalmente offline continuam funcionando; eles simplesmente não incorporam calibrações mais recentes até que a câmera fique online uma vez (ou seja reflashada de fábrica).                                                  |

No momento da captura, os coeficientes efetivamente aplicados são **fixados nos metadados XMP de cada imagem**. Uma atualização posterior da calibração nunca altera silenciosamente as imagens que você já capturou — o reprocessamento de uma captura antiga utiliza os coeficientes gravados em seu XMP, e não os mais recentes disponíveis atualmente.

## Solução de problemas

* **&quot;Nenhuma câmera encontrada na rede&quot;**— verifique a configuração de link local em [Configuração de Rede](https://mapir.gitbook.io/lattice-camera/setup/network-setup): NIC do host estática `169.254.x.x/16`, câmeras no mesmo link, sem DHCP/gateway previsto. Em seguida, use**Analisar Sistema**na caixa de diálogo de conexão para verificar em qual placa de rede cada câmera está (ou não está) visível.**Refaça a varredura** após qualquer alteração no cabeamento ou na placa de rede.
* **Um equipamento que antes funcionava não consegue se conectar** (painéis de matriz com erros `FRAMES WILL DROP` / `Reduce ROI to enable`) — uma atualização do driver da placa de rede redefiniu silenciosamente as configurações do anel de recepção. Reaplique-as ou execute o comando `chloros-cli lattice network --fix` em um terminal com privilégios elevados; consulte [Configuração de Rede](https://mapir.gitbook.io/lattice-camera/setup/network-setup).
* **Uma câmera exibe “In Array”** — ela pertence a uma sessão de matriz conectada. Desconecte a matriz para usar a câmera de forma independente.
