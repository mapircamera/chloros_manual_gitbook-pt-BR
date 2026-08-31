# Rede e Sincronização de Tempo do DAQ-E

> A configuração física da rede para o sensor — cabeamento, PoE, atribuição de IP e as próprias configurações de rede do dispositivo — está descrita no **[manual do usuário do DAQ](https://mapir.gitbook.io/daq/daq-e/network-setup)**. Esta página aborda o lado do Chloros: conexão, sincronização de tempo e o que fazer quando a descoberta não encontrar nenhum dispositivo.

O DAQ-E é o membro da família DAQ que utiliza Ethernet: alimentado por PoE, detectado via mDNS (serviço `_daq-e._tcp`) e endereçável por um nome de host derivado de seu ID de sensor — `daq-e-<6 hex>.local`, por exemplo, `daq-e-def330.local`. Esta página aborda como ele transfere dados na rede e como participa da sincronização de tempo PTP.

## Modos de transporte

| Modo | Ponto final | Consumidores | Observações |
| --- | --- | --- | --- |
| **Multicast** (padrão) | UDP `239.10.10.10:5002` | Qualquer número na mesma LAN recebe o mesmo fluxo | Cada datagrama é validado por CRC-16/CCITT |
| **Raw** | Porta TCP `5000` | Exatamente um cliente (exclusivo) | Compatível por byte com o DAQ-U |

O Chloros utiliza multicast por padrão, o que permite que a GUI, o CLI e o SDK monitorem um sensor ao mesmo tempo.

## Requisitos de rede

* **Mesmo domínio de broadcast.** A máquina que executa o Chloros deve estar no mesmo segmento de rede L2 que o sensor — a descoberta por mDNS não atravessa roteadores.
* **Solicitação do firewall do Windows: aceite-a.** Na primeira vez que o Chloros vincula os soquetes de multicast, o Windows Defender solicita permissão uma vez. Permitir isso abrange os dados do DAQ-E (UDP 5002), mDNS (UDP 5353) e PTP (UDP 319/320). No Linux, isso ocorre silenciosamente.
* **Alimentação PoE, sem LED de status.** O DAQ-E não possui LED próprio — verifique a alimentação por meio do indicador de link/PoE no switch ou na porta do injetor e aguarde alguns segundos após a inicialização para que ele inicialize e se conecte à rede.

## Conexão

**GUI:** Guia “Sensores de luz” → “Conectar sensor” → Tipo de dispositivo “DAQ-E (Ethernet)”. A descoberta ocorre apenas enquanto a caixa de diálogo de conexão estiver na tela (navegação mDNS mais uma varredura ARP no Windows), repetindo a cada 15 segundos; o botão Atualizar realiza uma nova varredura imediatamente. Os sensores descobertos aparecem na lista suspensa; o primeiro sensor detectado é selecionado automaticamente.

<!-- SCREENSHOT-NEEDED: DAQ connect dialog with Device Type set to "DAQ-E (Ethernet)" and at least one discovered sensor listed in the Hostname/IP dropdown (e.g. daq-e-xxxxxx.local), Connect button enabled. -->

**CLI** (backend em execução):

```bash
chloros-cli daq pool-connect --eth                              # auto-discover on the LAN
chloros-cli daq pool-connect --eth-host daq-e-def330.local      # explicit host — the reliable form
chloros-cli daq pool-connect --eth-host 192.168.1.57            # a plain IP works too
```

### Hosts com várias placas de rede e a primeira conexão após a inicialização

Em hosts com mais de uma interface de rede ativa, o **primeiro** `pool-connect --eth` após a inicialização pode ficar vazio mesmo quando o sensor está em bom estado — a varredura de descoberta pode deixar de detectar a interface na qual o sensor está localizado enquanto o cache ARP ainda estiver inativo. A solução confiável é pular a descoberta e passar o endereço explicitamente:

```bash
chloros-cli daq pool-connect --eth-host daq-e-def330.local
```

`--eth-host` aceita o nome de host mDNS ou o IP, sempre direciona para o sensor correto e é a forma recomendada para scripts e instalações sem interface gráfica. Na GUI, use o botão “Atualizar” da caixa de diálogo de conexão e aguarde um ciclo de nova varredura.

## Configurações do dispositivo e firmware

O próprio sensor armazena as configurações de rede — IP estático versus DHCP + endereçamento local de link, nome do dispositivo, transmissão automática na inicialização e senha OTA. Essas configurações do dispositivo não são expostas como comandos no CLI fornecido; elas são gerenciadas por meio da GUI do Chloros, onde são exibidas, ou com o suporte do MAPIR.

**As atualizações de firmware estão integradas à interface gráfica.**Quando um DAQ-E conectado executa um firmware mais antigo do que a imagem incluída na sua versão do Chloros, a linha do sensor exibe um ícone âmbar**Atualização disponível**, e a janela modal de configurações (engrenagem) oferece um<version>

botão</version> “Atualizar para<version>

”. A atualização é transferida pela rede em cerca de 30 segundos; o sensor reinicia e se reconecta automaticamente, e uma transferência interrompida mantém o firmware atual intacto.

<!-- SCREENSHOT-NEEDED: DAQ-E per-sensor settings modal showing the DAQ-E-only rows: Hostname/IP, Firmware row with the "Update to <ver>" button (or "Up to date"), and the PTP Sync row with a live state value. -->

## Sincronização de tempo PTP

O firmware v1.2.0+ do DAQ-E participa do IEEE 1588 PTPv2 como um relógio comum (apenas escravo). **O backend do host Chloros é o grandmaster PTP** — todos os DAQ-E e todas as câmeras LATTICE na LAN funcionam como escravos dele no domínio 0, mantendo todos os carimbos de data/hora dos dispositivos dentro de uma tolerância de ~1 ms. Esse relógio compartilhado é o que permite que as leituras do DAQ tenham carimbos de data/hora compatíveis com as exposições da câmera (consulte [Gravação e o formato .daq](recording.md)).

Verifique a sincronização a partir do CLI:

| Comando | Exibe |
| --- | --- |
| `chloros-cli time-sync status` | Estado do grandmaster do host, prioridades BMCA, identidade do relógio |
| `chloros-cli time-sync peers` | Todos os escravos detectados (sensores DAQ-E + câmeras LATTICE) |
| `chloros-cli time-sync cameras` | Integridade do PTP por câmera (`PtpStatus`, `PtpOffsetFromMaster`, `PtpMeanPathDelay`) |
| `chloros-cli time-sync restart` | Reiniciar o processo grandmaster |

Na interface gráfica (GUI), a janela modal de configurações do DAQ-E exibe uma linha **PTP Sync** em tempo real com o estado atual do PTP do sensor.

Detalhes para consumidores que exigem alinhamento rigoroso:

* Cada datagrama transmitido contém um campo de sinalizadores; **o bit 2 é definido em quadros cujo carimbo de data/hora está sincronizado com o PTP**. Pipelines que exigem alinhamento rigoroso entre câmera e DAQ devem ser acionados com base nesse bit.
* Antes de uma captura sincronizada, confirme se o sensor aparece no `chloros-cli time-sync peers`. (As ferramentas internas de hardware direto do MAPIR também podem condicionar a gravação ao bloqueio PTP com um sinalizador do `--wait-ptp` que aguarda até 15 s para que o sensor atinja o estado SLAVE; essa funcionalidade não faz parte do CLI fornecido.)
* Enquanto o PTP estiver ativamente em modo escravo, o sensor rejeita envios manuais de clock (“O PTP está fornecendo clock”). Isso é intencional — confie no PTP.

## Notas sobre o Linux

* **O PTP precisa do `libcap2-bin` no momento da instalação.** O script de pós-instalação `.deb` concede permissão ao `cap_net_bind_service=+ep` no `/usr/lib/chloros/chloros-backend` para que ele possa vincular as portas PTP 319/320 sem privilégios de root. Se o `libcap2-bin` estiver ausente, essa etapa será ignorada e o PTP não conseguirá iniciar. Solução:

  ```bash
  sudo apt install libcap2-bin
  sudo apt reinstall chloros
  ```

* **Jetson / Raspberry Pi sem interface gráfica:** na primeira instalação, a unidade do systemd `chloros-backend.service` é gerada, mas não está habilitada. Para manter o PTP sempre ativo (e a disponibilidade do DAQ) sem a interface gráfica:

  ```bash
  sudo systemctl enable --now chloros-backend.service
  ```

  Sem ela, o PTP funciona apenas enquanto a GUI Chloros estiver aberta.

## Solução de problemas: “Nenhum dispositivo DAQ-E encontrado”

| Verificação | Detalhe |
| --- | --- |
| Alimentação | Nenhum LED aceso no sensor — verifique os indicadores de PoE e de link da porta do switch/injetor; aguarde alguns segundos após a inicialização |
| Domínio de difusão | Host e sensor no mesmo segmento L2; o mDNS não está roteando |
| Firewall do Windows | Aceite a solicitação do Defender na primeira execução (UDP 5002, 5353, 319/320) |
| Host com várias placas de rede | A primeira descoberta após a inicialização pode não detectar o sensor — conecte-se com `--eth-host <ip-or-hostname>` |
| Nova varredura na GUI | A descoberta só ocorre enquanto a caixa de diálogo de conexão estiver aberta; use “Atualizar” |</version>
