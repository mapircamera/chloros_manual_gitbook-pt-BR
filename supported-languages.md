# Idiomas suportados

O Chloros oferece suporte completo à interface em **38 idiomas em todo o mundo**, tornando-o acessível a usuários em todo o globo. É possível alternar entre os idiomas instantaneamente tanto na interface gráfica do desktop quanto no CLI.

O Chloros oferece suporte aos seguintes idiomas:

| # | Idioma | Nome nativo | Código do CLI |
|---|----------|-------------|----------|
| 1 | 🇺🇸 Inglês | English | `en` |
| 2 | 🇪🇸 Espanhol | Español | `es` |
| 3 | 🇵🇹 Português | Português | `pt` |
| 4 | 🇫🇷 Francês | Français | `fr` |
| 5 | 🇩🇪 Alemão | Deutsch | `de` |
| 6 | 🇮🇹 Italiano | Italiano | `it` |
| 7 | 🇯🇵 Japonês | 日本語 | `ja` |
| 8 | 🇰🇷 Coreano | 한국어 | `ko` |
| 9 | 🇨🇳 Chinês (simplificado) | 简体中文 | `zh` |
| 10 | 🇹🇼 Chinês (tradicional) | 繁體中文 | `zh-TW` |
| 11 | 🇷🇺 Russo | Русский | `ru` |
| 12 | 🇳🇱 Holandês | Nederlands | `nl` |
| 13 | 🇸🇦 Árabe | العربية | `ar` |
| 14 | 🇵🇱 Polonês | Polski | `pl` |
| 15 | 🇹🇷 Turco | Türkçe | `tr` |
| 16 | 🇮🇳 Hindi | हिंदी | `hi` |
| 17 | 🇮🇩 Indonésio | Bahasa Indonesia | `id` |
| 18 | 🇻🇳 Vietnamita | Tiếng Việt | `vi` |
| 19 | 🇹🇭 Tailandês | ไทย | `th` |
| 20 | 🇸🇪 Sueco | Svenska | `sv` |
| 21 | 🇩🇰 Dinamarquês | Dansk | `da` |
| 22 | 🇳🇴 Norueguês | Norsk | `no` |
| 23 | 🇫🇮 Finlandês | Suomi | `fi` |
| 24 | 🇬🇷 Grego | Ελληνικά | `el` |
| 25 | 🇨🇿 Tcheco | Čeština | `cs` |
| 26 | 🇭🇺 Húngaro | Magyar | `hu` |
| 27 | 🇷🇴 Romeno | Română | `ro` |
| 28 | 🇺🇦 Ucraniano | Українська | `uk` |
| 29 | 🇧🇷 Português do Brasil | Português Brasileiro | `pt-BR` |
| 30 | 🇭🇰 Cantonês | 粵語 | `zh-HK` |
| 31 | 🇲🇾 Malaio | Bahasa Melayu | `ms` |
| 32 | 🇸🇰 Eslovaco | Slovenčina | `sk` |
| 33 | 🇧🇬 Búlgaro | Български | `bg` |
| 34 | 🇭🇷 Croata | Hrvatski | `hr` |
| 35 | 🇱🇹 Lituano | Lietuvių | `lt` |
| 36 | 🇱🇻 Letão | Latviešu | `lv` |
| 37 | 🇪🇪 Estoniano | Eesti | `et` |
| 38 | 🇸🇮 Esloveno | Slovenščina | `sl` |

## Como alterar o idioma

### No Chloros Desktop

1. Abra as configurações do aplicativo
2. Acesse o menu de seleção de idioma
3. Escolha o idioma de sua preferência na lista
4. A interface será atualizada instantaneamente

### No Chloros CLI

Use o comando `language` para visualizar ou alterar o idioma da interface do CLI:

```bash
# View current language
chloros-cli language

# Change to Spanish
chloros-cli language es

# Change to Chinese (Simplified)
chloros-cli language zh

# Change to Brazilian Portuguese
chloros-cli language pt-BR

# List all available languages
chloros-cli language --list
```

Para mais detalhes, consulte a [documentação do CLI](CLI.md).

## Cobertura

Todos os 38 idiomas são totalmente suportados em:

* **Chloros Desktop** — Tradução completa da interface gráfica (GUI)
* **Chloros CLI** — Interface de linha de comando e mensagens de saída

O Python SDK API e sua [documentação de referência](reference/sdk-reference.md) são fornecidos em inglês.

O suporte a idiomas garante que usuários em todo o mundo possam trabalhar com eficiência em seu idioma nativo, sem barreiras.
