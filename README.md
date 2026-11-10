# Trawel Pulse for ChatGPT and Claude

[Trawel Pulse](https://trawelpulse.com) in your AI assistant:

- **Your travel account.** Add trips from tickets, booking emails or a bank statement and see:
  - the days by country, as each country counts them;
  - your tax residence and how long more you may stay;
  - your progress to citizenship;
  - your visas.

  The account is the same as on trawelpulse.com, and the charts are drawn right in the chat.
- **Travel questions.** A country's safety level, visa rules for your passport, emergency numbers, natural disasters, recent changes and the cheapest flights.

Every assistant connects to one address:

```
https://trawelpulse.com/mcp
```

You sign in with Telegram in the assistant's own window, and nothing else is needed.

## Connect

| Assistant | How |
| --- | --- |
| **Claude** (web, desktop, mobile) | Settings → Connectors → Add custom connector → paste the address → Connect |
| **ChatGPT** | Plugins → + → Add custom MCP server → paste the address, OAuth → Create as a plugin |
| **Claude Code** | `/plugin marketplace add andreikonkin/trawel_pulse`, then `/plugin install trawel-pulse@trawel-pulse` |
| **Codex** | `codex plugin marketplace add andreikonkin/trawel_pulse` |

Then just ask:

- "Add my flights of 2025" (paste the ticket emails);
- "How many more days may I stay in the UK this year?";
- "When can I apply for Cypriot citizenship?"

The assistant sees and changes your moves, visas and goals, but it can't delete your account. You can turn assistants off at any time in the account's settings on [trawelpulse.com](https://trawelpulse.com/en/me).

## Подключение (по-русски)

Адрес один для всех ассистентов: `https://trawelpulse.com/mcp`. Вход через Telegram прямо в окне ассистента.

- **Claude:** Настройки → Коннекторы → Добавить свой коннектор → вставьте адрес.
- **ChatGPT:** Plugins → + → Add custom MCP server → вставьте адрес, вход OAuth → Create as a plugin.
- **Claude Code:** `/plugin marketplace add andreikonkin/trawel_pulse`, затем `/plugin install trawel-pulse@trawel-pulse`.

Отключить ассистентов можно в настройках кабинета на сайте.

## What's here

`plugins/trawel-pulse/` is the plugin. It comes in two formats:

- `plugin.json` and `mcp.json` ([Agent Plugins](https://agent-plugins.org)) for ChatGPT and Codex;
- `.claude-plugin/plugin.json` and `.mcp.json` for Claude Code.

It holds the server's address, a skill on how to use it, and the icon. The marketplaces are `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json`.
