# Trawel Pulse for AI assistants

Developed and published by Andrei Konkin. Trawel Pulse is the product name.

[Trawel Pulse](https://trawelpulse.com) makes your travel account and destination
tools available to AI assistants through a remote MCP server.
Connect a client that supports Streamable HTTP and OAuth with PKCE to use:

- **Your travel account.** Add trips from tickets, booking emails or a bank statement and see:
  - the days by country, as each country counts them;
  - your tax residence and how long more you may stay, and your own limits of days abroad;
  - your progress to citizenship by residence, and what any of 66 programmes in 44 countries asks and when you could apply;
  - your visas;
  - cards of your year, a goal or a calendar to share.

  The account is the same as on trawelpulse.com. Clients with MCP Apps support can show the calendar, map and charts in the chat; other clients can use the text and structured results.
- **Travel questions.** A country's safety level, visa rules for your passport, emergency numbers, natural disasters, recent changes and the cheapest flights.

Compatible MCP clients connect to one address:

```
https://trawelpulse.com/mcp
```

You sign in with Google or Telegram on trawelpulse.com through the client's OAuth flow, then approve access to your travel account.

The package version does not change the server URL: every client uses `/mcp`. While other app directories review version 1.0.1, OpenAI OAuth clients receive the latest tool set and other clients retain the reviewed set. ChatGPT tool availability follows OpenAI's MCP scans. See [versions](docs/mcp.md#versions).

## MCP integration documentation

For platform integrators and reviewers: [MCP server reference](docs/mcp.md),
including OAuth, all tool inputs and results, permissions, errors and limits.
Exact input schemas and annotations: [tool schema snapshot](docs/mcp-tools.json).

## Connect

| Assistant | How |
| --- | --- |
| **Claude** (web, desktop, mobile) | Settings → Connectors → Add custom connector → paste the address → Connect |
| **ChatGPT** | [Open Trawel Pulse](https://chatgpt.com/plugins/plugin_asdk_app_6af3c168f6ec81919e98d0d0c32a22ba) and connect your account through OAuth |
| **Claude Code** | `/plugin marketplace add andreikonkin/trawel_pulse`, then `/plugin install trawel-pulse@trawel-pulse` |
| **Codex** | `codex plugin marketplace add andreikonkin/trawel_pulse` |
| **Other MCP clients** | Add the endpoint above as a remote Streamable HTTP server and use OAuth with PKCE. See the [integration reference](docs/mcp.md) for discovery and client-registration requirements. |

Tested with ChatGPT and Claude. Integration preparation for
Gemini and Meta Muse is in progress; their OAuth flows, tool execution and UI
rendering still need platform-specific validation. Public directory availability
depends on each platform's review and publication process.

Then just ask:

- "Add my flights of 2025" (paste the ticket emails);
- "How many more days may I stay in the UK this year?";
- "When can I apply for Cypriot citizenship?"

The assistant sees and changes your moves, visas, goals and limits, and makes a public link to a card when you ask (you can turn it off in the account), but it can't delete your account. You can turn assistants off at any time in the account's settings on [trawelpulse.com](https://trawelpulse.com/en/me).

[Privacy policy](https://trawelpulse.com/en/privacy) · [Terms](https://trawelpulse.com/en/terms) · [Support](https://trawelpulse.com/en/support) · support@trawelpulse.com

## Подключение (по-русски)

Адрес MCP-сервера: `https://trawelpulse.com/mcp`. Подходит клиентам с поддержкой Streamable HTTP и OAuth с PKCE. Вход — через Google или Telegram на сайте, затем разрешение доступа ассистенту.

Все подключения используют один адрес `/mcp`, независимо от версии пакета. Пока другие каталоги проверяют 1.0.1, клиенты OpenAI с OAuth получают последний набор инструментов, остальные — набор на проверке. Доступность новых инструментов ChatGPT зависит от сканирования MCP в OpenAI.

- **Claude:** Настройки → Коннекторы → Добавить свой коннектор → вставьте адрес.
- **ChatGPT:** [откройте плагин Trawel Pulse](https://chatgpt.com/plugins/plugin_asdk_app_6af3c168f6ec81919e98d0d0c32a22ba) и подключите кабинет через OAuth.
- **Claude Code:** `/plugin marketplace add andreikonkin/trawel_pulse`, затем `/plugin install trawel-pulse@trawel-pulse`.
- **Другие MCP-клиенты:** добавьте адрес как удалённый сервер Streamable HTTP и выберите OAuth. Требования к подключению — в [справочнике MCP](docs/mcp.md).

Подключение проверено с ChatGPT и Claude. Интеграции Gemini и Meta Muse готовятся; работу OAuth, инструментов и виджетов ещё предстоит проверить на этих платформах. Календарь, карта и графики внутри чата доступны в клиентах с поддержкой MCP Apps.

Отключить ассистентов можно в настройках кабинета на сайте.

## What's here

`plugins/trawel-pulse/` is the plugin. It comes in two formats:

- `plugin.json` and `mcp.json` ([Agent Plugins](https://agent-plugins.org)) for ChatGPT and Codex;
- `.claude-plugin/plugin.json` and `.mcp.json` for Claude Code.

It holds the server's address, a skill on how to use it, and the icon. The marketplaces are `.claude-plugin/marketplace.json` and `.agents/plugins/marketplace.json`.
