# Chloros+ Login

## Chloros e Chloros (Navegador) Login

O menu lateral do usuário <img src=".gitbook/assets/icon_user.JPG" alt="" data-size="line"> permite que você faça login na sua conta Chloros+ e desbloqueie recursos adicionais.

Quando estiver conectado, os detalhes da sua conta serão exibidos:

<figure><img src=".gitbook/assets/user_account.JPG" alt="" width="375"><figcaption></figcaption></figure>## CLI Login

Faça login com suas credenciais Chloros+ para habilitar o processamento CLI.

**Sintaxe:**

```bash
chloros-cli login <email> <password>
```

**Exemplo:**

```powershell
chloros-cli login user@example.com 'MyP@ssw0rd123'
```

{% hint style=&quot;warning&quot; %}
**Caracteres especiais**: use aspas simples em torno de senhas que contenham caracteres como `$`, `!` ou espaços.
{% endhint %}

**Saída:**

<figure><img src=".gitbook/assets/cli login_w.JPG" alt=""><figcaption></figcaption></figure>### Expiração do plano

A expiração do plano na GUI mostra quando sua licença se tornará inválida. Para assinaturas mensais recorrentes, a expiração ocorre no final do mês. Para assinaturas anuais, ocorre um ano após o início da assinatura. A verificação da licença requer uma conexão mensal com a Internet para verificação, com um período de carência de 30 dias.

### Limite de dispositivos

Cada plano Chloros+ oferece um número diferente de dispositivos registrados. Cada dispositivo em que você fizer login com uma conta Chloros+ será contabilizado no número de dispositivos registrados. Você pode renomear e remover um dispositivo na página da sua conta MAPIR Cloud.

<table><thead><tr><th width="168.5999755859375" align="right">Plano Chloros</th><th align="center">COPPER</th><th align="center">BRONZE</th><th align="center">PRATA</th><th align="center">OURO</th></tr></thead><tbody><tr><td align="right">Dispositivos compatíveis</td><td align="center">2</td><td align="center">2</td><td align="center">5</td><td align="center">10</td></tr></tbody></table>
