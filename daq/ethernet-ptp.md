# Rede e sincronização de tempo do DAQ-E

> A configuração física da rede para o sensor — cabeamento, PoE, atribuição de endereço IP e as configurações de rede do próprio dispositivo — está descrita no **[manual do usuário do DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página aborda o lado do Chloros: conexão, sincronização de tempo e o que fazer quando a descoberta não encontrar nenhum dispositivo.

O DAQ-E é o membro da família DAQ que utiliza Ethernet: alimentado por PoE, detectado via mDNS (serviço `_daq-e._tcp`) e endereçável por um nome de host derivado de seu ID de sensor — `daq-e-<6 hex>.local`, por exemplo, `daq-e-def330.local`. Esta página aborda como ele transfere dados na rede e como participa da sincronização de tempo PTP.

## Modos de transporte

| Modo | Ponto final | Consumidores | Observações |
| --- | --- | --- | --- |
| **Multicast** (padrão) | UDP `239.10.10.10:5002` | Qualquer número na mesma LAN recebe o mesmo fluxo | Cada datagrama é validado por CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Exatamente um cliente (exclusivo) | Compatível com DAQ-U em nível de byte |

O Chloros usa multicast por padrão, o que permite que a GUI, a CLI e o SDK monitorem um sensor por vez.

## Requisitos de rede

* **Mesmo domínio de broadcast.** A máquina que executa o Chloros deve estar no mesmo segmento de rede L2 que o sensor — a descoberta por mDNS não atravessa roteadores.
* **Firewall.** No Windows, o instalador adiciona a regra de firewall **Chloros Backend (UDP-In)**para o backend, em todos os perfis de rede, que abrange dados do DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320), além de uma regra**“Chloros Backend (TCP-In)”**, limitada à sub-rede local, para a conexão que um DAQ-E estabelece de volta ao computador durante uma atualização de firmware. No Linux, o `.deb` abre as portas UDP no `ufw` ou no `firewalld` quando esse firewall está ativo no momento da instalação. Um firewall de terceiros, ou um ativado posteriormente, precisa ter essas mesmas portas permitidas.
* **Alimentação PoE, sem LED de status.** O DAQ-E não possui LED próprio — verifique a alimentação por meio do indicador de link/PoE no switch ou na porta do injetor e aguarde alguns segundos após a inicialização para que ele inicialize e se conecte à rede.

## Conexão

**GUI:** Guia “Sensores de luz” → “Conectar sensor” → Tipo de dispositivo “DAQ-E (Ethernet)”. A descoberta ocorre apenas enquanto a caixa de diálogo de conexão estiver na tela (navegação mDNS mais uma varredura ARP no Windows), repetindo-se a cada 15 segundos; o botão “Atualizar” realiza uma nova varredura imediatamente. Os sensores detectados aparecem na lista suspensa; o primeiro sensor detectado é selecionado automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend em execução):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

Um DAQ-E que já esteja no pool é reconhecido tanto pelo nome do host quanto pelo endereço IP: conectá-lo novamente com outro endereço retorna a entrada existente no pool, em vez de abrir a unidade pela segunda vez.

### Hosts com várias placas de rede e a primeira conexão após a inicialização

Em hosts com mais de uma interface de rede ativa, o **primeiro** `pool-connect --eth` após a inicialização pode ficar vazio mesmo quando o sensor está em bom estado — a varredura de descoberta pode não detectar a interface na qual o sensor está conectado enquanto o cache ARP ainda estiver vazio. A solução confiável é pular a descoberta e passar o endereço explicitamente:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aceita o nome de host mDNS ou o IP, sempre direciona para o sensor correto e é a forma recomendada para scripts e instalações sem interface gráfica. Na GUI, use o botão “Atualizar” da caixa de diálogo de conexão e aguarde um ciclo de varredura.

## Configurações do dispositivo e firmware

O próprio sensor mantém sua configuração de rede: DHCP (a configuração de fábrica, que recorre a um endereço local de link 169.254.x.x quando nenhum servidor DHCP responde) ou um endereço estático. Ele também mantém a configuração de iniciar a transmissão ao ser ligado. Altere essas configurações na interface gráfica (nas configurações de engrenagem do sensor, **Rede**:**Alterar…**,**Transmitir ao ligar**,**Reiniciar sensor**; consulte [A guia DAQ](gui.md)), ou pela CLI e pelo SDK com `pool-set-network`, `pool-set-auto-stream` e `pool-reboot` (`DAQSensorSession.set_network()`, `set_auto_stream()`, `reboot()`; consulte a [Referência da CLI](../reference/cli-reference.md#daq-e-device-settings)). O sensor reinicia para aplicar uma alteração na rede, e o Chloros o reconecta. Um endereço estático deve estar na rede do computador que executa o Chloros: o Chloros rejeita aquele que não conseguiu acessar após a reinicialização. O ID do sensor e seu nome de host `daq-e-<id>.local` nunca mudam. Novidade na versão 1.3.0.

**As atualizações de firmware estão integradas à interface gráfica (GUI).** Quando um DAQ-E conectado executa um firmware mais antigo do que a imagem incluída na sua versão do Chloros, a linha do sensor exibe um ícone âmbar com a mensagem **Atualização disponível**, e a janela modal de configurações (ícone de engrenagem) oferece um <version>botão</version>

“Atualizar para<version>”. A atualização é transferida pela rede em cerca de 30 segundos; o sensor reinicia e se reconecta automaticamente, e uma transferência interrompida mantém o firmware atual intacto. Na CLI ou no SDK, e no Linux — que não possui GUI —, execute o comando `chloros-cli daq pool-update-firmware --sensor-id <id>` (`DAQSensorSession.update_firmware()`).

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronização de tempo PTP

O firmware v1.2.0+ do DAQ-E participa do IEEE 1588 PTPv2 como um relógio comum (apenas escravo). **O backend do host Chloros é o grandmaster PTP** — todos os DAQ-E e todas as câmeras LATTICE na LAN funcionam como escravos dele no domínio 0, mantendo todos os carimbos de data/hora dos dispositivos dentro de uma tolerância de ~1 ms. Esse relógio compartilhado é o que permite que as leituras do DAQ tenham carimbos de data/hora compatíveis com as exposições da câmera (consulte [Gravação e o formato .daq](recording.md)).

Verifique a sincronização pela CLI:

| Comando | Exibe |
| --- | --- |
| `chloros-cli time-sync status` | Estado do grandmaster do host, prioridades BMCA, identidade do relógio e `health`: `OK` ou `SENDS_FAILING` / `DEGRADED` / `NO_SOCKETS` / `PEERS_SILENT` com um `health_detail` explicando por que seus pacotes não estão chegando à rede |
| `chloros-cli time-sync peers` | Todos os escravos detectados (sensores DAQ-E + câmeras LATTICE) |
| `chloros-cli time-sync cameras` | Integridade do PTP por câmera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar o processo do grandmaster |

Na interface gráfica (GUI), a janela modal de configurações do DAQ-E exibe uma linha **Sincronização PTP** em tempo real com o estado atual do PTP do sensor.**Um DAQ-E com o PTP desativado permanece desativado.** A unidade mantém sua configuração de ativação/desativação do PTP em sua própria memória; portanto, uma unidade cujo PTP foi desativado permanece desativada após uma reconexão ou reinicialização, e seus carimbos de data/hora não são sincronizados via PTP. O `chloros-cli daq pool-list` sinaliza tal unidade (`!! PTP is switched OFF on this unit …`) e exibe o comando que corrige o problema:

```bash
chloros-cli daq pool-enable-ptp --sensor-id daq-e-def330
``` O

Chloros nunca reativa uma unidade por conta própria, pois o desligamento pode ser intencional.

Detalhes para usuários que exigem alinhamento rigoroso:

* Cada datagrama transmitido contém um campo de sinalizadores; **o bit 2 é definido em quadros cujo carimbo de data/hora está sincronizado com o PTP**. Pipelines que exigem alinhamento rigoroso entre câmera e DAQ devem ser controlados por esse bit.
* Antes de uma captura sincronizada, confirme se o sensor aparece no `chloros-cli time-sync peers` e se o `chloros-cli daq pool-list` não sinaliza que seu PTP está desligado. (As ferramentas internas de hardware direto do MAPIR também podem condicionar a gravação ao bloqueio PTP por meio de um sinalizador em `--wait-ptp` que aguarda até 15 s para que o sensor atinja o estado SLAVE; essas ferramentas não fazem parte da CLI fornecida.)
* Enquanto o PTP estiver ativamente no modo escravo, o sensor recusa envios manuais de clock (“O PTP está fornecendo clock”). Isso é intencional — confie no PTP.

## Notas sobre o Linux

* **Portas PTP sem privilégios de root.** O PTP utiliza as portas UDP 319/320. O `.deb` grava o `/etc/sysctl.d/60-chloros-ptp.conf` (`net.ipv4.ip_unprivileged_port_start = 319`), de modo que o backend se vincula a elas, rodando como seu usuário. A partir da versão 1.3.0, o pacote não define nenhuma capacidade de arquivo (`setcap`) no backend e não precisa de `libcap2-bin`; não adicione uma capacidade por conta própria, pois o backend descompactado não iniciará. Se o PTP não conseguir vincular a porta 319, consulte [Instalação no Linux](../linux/linux-installation.md#ptp-will-not-start--cannot-bind-port-319).

* **Jetson sem monitor / host Linux:** o pacote instala os serviços do backend, mas não habilita nenhum deles. Para manter o PTP sempre ativo (e a disponibilidade do DAQ) sem que ninguém execute um comando, habilite o serviço como seu usuário:

  ```bash
  sudo systemctl enable --now chloros-backend@$USER.service    # $USER expands to your user name
  ```

  Sem isso, o PTP só funciona enquanto algo tiver iniciado o back-end — um comando `chloros-cli` ou um script do SDK. Consulte [Instalação no Linux](../linux/linux-installation.md#always-on-ptp-for-headless-hosts) para obter informações sobre o usuário root `chloros-backend.service` e como alternar a partir dele.

## Solução de problemas: “Nenhum dispositivo DAQ-E encontrado”

| Verificação | Detalhes |
| --- | --- |
| Alimentação | Nenhum LED aceso no sensor — verifique os indicadores de PoE e de link da porta do switch/injetor; aguarde alguns segundos após a inicialização |
| Domínio de broadcast | Host e sensor no mesmo segmento L2; o mDNS não está roteando |
| Firewall | Windows: a regra **Chloros Backend (UDP-In)** do instalador permite a entrada do fluxo do DAQ-E (UDP 5002), mDNS (5353) e PTP (319/320); um firewall de terceiros precisa da mesma configuração. Linux: o `.deb` abre essas portas no `ufw` / `firewalld` somente se o firewall estiver ativo no momento da instalação — permita-as manualmente caso ele seja ativado posteriormente |
| Host com várias placas de rede | A descoberta pesquisa o mDNS com um fallback de ARP; o `--eth-host <ip-or-hostname>` ignora a descoberta e se conecta diretamente |
| Nova varredura pela GUI | A descoberta só é executada enquanto a caixa de diálogo de conexão estiver aberta; use “Atualizar” |
