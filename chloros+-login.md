# Login no Chloros+

## Login no Chloros e no Chloros (navegador)

O <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite que você faça login na sua conta Chloros+ e desbloqueie recursos adicionais.

Ao fazer login, os detalhes da sua conta serão exibidos:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## Login no CLI

Faça login com suas credenciais do Chloros+ para habilitar o processamento do CLI. No Linux (sem GUI), esta é a única maneira de ativar sua licença.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

{% hint style="info" %}
**Usuários do SDK**: O Python SDK também oferece um método programático `logout()` para limpar credenciais armazenadas em cache. Consulte a [documentação do Python SDK](api-python-sdk.md#logout) para obter detalhes.
{% endhint %}

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style="warning" %}
**Caracteres especiais**: Use aspas simples em torno de senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Armazenamento de credenciais

As credenciais em cache são armazenadas em um local específico da plataforma:

| Plataforma | Caminho do cache de credenciais |
| --- | --- |
| **Windows** | `%APPDATA%\Chloros\cache\` |
| **Linux** | `~/.cache/chloros/` |

### Vencimento do plano

O vencimento do plano na GUI indica quando sua licença se tornará inválida. Para assinaturas mensais recorrentes, o vencimento ocorre no final do mês. Para assinaturas anuais, ocorre um ano após o início da assinatura. A verificação da licença requer uma conexão mensal à Internet para confirmação, com um período de carência de 30 dias.

### Limite de dispositivos

Cada plano Chloros+ oferece um número diferente de dispositivos registrados. Cada dispositivo no qual você fizer login com uma conta Chloros+ será contabilizado no seu número de dispositivos registrados. Você pode renomear e remover um dispositivo na página da sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros+</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PRATA</th><th align="center">OURO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
