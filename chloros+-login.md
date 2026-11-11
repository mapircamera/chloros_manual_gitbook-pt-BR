# Login no Chloros+

## Login pela interface gráfica (GUI)

O menu lateral do usuário <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite que você faça login na sua conta do Chloros+ e desbloqueie recursos adicionais.

**Você só precisa fazer login uma vez por máquina.** A GUI, a CLI e o SDK do Python compartilham a mesma sessão em cache — fazer login pela GUI da área de trabalho também ativa a CLI e o SDK naquele computador (e vice-versa via `chloros-cli login`).

Ao fazer login, os detalhes da sua conta serão exibidos:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the logged-in user account panel in Chloros 1.3.0 — plan name display and the registered-device list UI may have changed; must show plan name, expiration, and device list. -->

## Níveis de plano

| Plano | `plan_id` | Tipo |
| --- | --- | --- |
| Iron | `0` | Gratuito |
| Copper | `1` | Pago (Chloros

+) |
| Bronze | `2` | Pago (Chloros

+) |
| Silver | `3` | Pago (Chloros

+) |
| Ouro | `4` | Pago (Chloros

+) |

Consulte [planos e preços](https://cloud.mapir.camera/pricing) para saber o que cada plano pago inclui.

### O acesso à CLI

/SDK

requer um plano pago

O acesso à CLI

e ao Python

SDK

requer **qualquer plano pago do Chloros

+ (Cobre ou superior)**. Isso é aplicado**no lado do servidor** — toda solicitação de CLI

/SDK

deve conter tanto uma sessão ativa quanto um plano pago:

| Status HTTP

| `error_code` | Significado | Solução |
| --- | --- | --- | --- |
| `401` | `AUTH_REQUIRED` | Não está conectado nesta máquina | `chloros-cli login <email> <password>` |
| `403` | `PLAN_UPGRADE_REQUIRED` | Está conectado, mas o nível do plano é muito baixo (nível Iron gratuito) | Faça upgrade para qualquer plano pago do Chloros

+ |

`chloros-cli status` continua acessível no plano gratuito, portanto, você sempre pode ver seu plano atual e o motivo pelo qual o acesso foi negado.

### Limites de hardware conectado por plano

Cada plano estabelece um limite para o número de câmeras LATTICE e sensores de luz DAQ que podem ser conectados ao vivo simultaneamente:

| Plano | Câmeras LATTICE | Sensores de luz DAQ |
| --- | --- | --- |
| Iron (gratuito / sem login) | 4 | 2 |
| Copper / Bronze | 6 | 3 |
| Silver | 10 | 6 |
| Gold | 20 | 12 |

## Login na CLI



Faça login com suas credenciais do Chloros

+ para habilitar o processamento via CLI

. No Linux

(sem GUI), essa é a única maneira de ativar sua licença.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Usuários do SDK**: O Python

SDK

também oferece um método programático `logout()` para limpar as credenciais armazenadas em cache. Consulte a [Referência do SDK

](reference/sdk-reference.md) para obter detalhes.
{% endhint %}

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiais**: Use aspas simples ao redor de senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>
<!-- SCREENSHOT-UPDATE: re-shoot the CLI login output — the banner now prints "Chloros CLI 1.3.0"; capture a successful login with the current output format. -->

### Armazenamento de credenciais

As credenciais e a configuração armazenadas em cache ficam na pasta `.chloros` do diretório pessoal do usuário em **todas as plataformas**:

| Plataforma | Caminho do cache de credenciais |
| --- | --- |
| **Windows** | `%USERPROFILE%\.chloros\` |
| **Linux** | `~/.chloros/` |

A instalação de uma versão mais recente sobre uma já existente mantém suas credenciais de login, no Windows

(o instalador) e no Linux

(o `.deb`, incluindo o `chloros-cli update`). Desinstalar o Chloros

— ou remover o pacote do Linux

— limpa o login armazenado em cache; portanto, você precisará fazer login novamente após uma nova instalação.

### Vencimento do plano e período de carência offline

O vencimento do plano na interface gráfica (GUI) indica quando sua licença se tornará inválida. Para assinaturas mensais recorrentes, a expiração ocorre no final do mês; para assinaturas anuais, ocorre um ano após o início da assinatura.

O Chloros

valida sua licença online, mas o trabalho offline é permitido dentro de um período de carência:

* As validações bem-sucedidas no servidor ficam armazenadas em cache por **5 minutos**, portanto, o uso normal gera pouquíssimas chamadas de licença.
* Um cache de licença assinado e vinculado à máquina cobre períodos offline mais longos: **30 dias para planos mensais**e**até a data de vencimento da sua assinatura (no máximo 365 dias) para planos anuais**.
* Quando o período de carência expira, o plano passa para o nível gratuito “Iron” até que o computador consiga se conectar ao servidor de licenças; o acesso é retomado na próxima verificação bem-sucedida.

### Limite de dispositivos

Cada plano Chloros

+ oferece um número diferente de dispositivos registrados. Cada dispositivo no qual você faz login com uma conta Chloros

+ conta para o seu número de dispositivos registrados. Você pode renomear e remover um dispositivo na página da sua conta MAPIR

Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">SILVER</th><th align="center">GOLD</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>

O número exato de dispositivos permitidos na sua conta é exibido na página da sua conta do MAPIR Cloud. Ao sair de um dispositivo, a vaga correspondente é liberada de forma confiável, e um dispositivo já registrado sempre pode fazer login novamente, mesmo quando a conta estiver no limite de dispositivos.
