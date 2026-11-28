# Trawel Pulse

Developed and published by Andrei Konkin. Trawel Pulse is the product name.

Your travel account in ChatGPT, Codex, Claude and other compatible assistants. Tell the assistant about your trips, or paste ticket emails or a bank statement, and it keeps them in your Trawel Pulse account. From them it counts days by country, tax residence, how long more you may stay and progress towards citizenship, with calendars and maps in supported clients. It also answers travel questions: safety advisories, visa rules, emergency numbers, natural disasters, recent changes and cheap flights.

[Open in ChatGPT](https://chatgpt.com/plugins/plugin_asdk_app_6af3c168f6ec81919e98d0d0c32a22ba) · [Connection guide](https://trawelpulse.com/en/bot#ai)

The latest tool set also provides the citizenship catalog (44 countries, 66 programs), sourced conditions and public estimates, personal country/year travel limits, and shareable cards. Tool availability depends on the client and its approved tool set; new ChatGPT tools become available after OpenAI's MCP checks.

## What's in the plugin

- **The Trawel Pulse MCP server**, `https://trawelpulse.com/mcp`: the same HTTPS address in `mcp.json` and `.mcp.json`, independent of the package version. No local MCP server is required.
- **A skill**, `skills/trawel-pulse/SKILL.md`: how to enter trips as moves and answer from the account.
- The same server for ChatGPT and Codex (`plugin.json`, `mcp.json`, [Agent Plugins](https://agent-plugins.org)) and an icon.

## Sign-in and data

When connecting, sign in with Google or Telegram on trawelpulse.com (OAuth) and allow the assistant access. The server keeps the account's moves, citizenships, tax residence, visas, goals and personal travel limits. MCP tool calls go to trawelpulse.com; account data is stored on the server. You can turn assistants off at any time in the account's settings on trawelpulse.com.

- [Privacy policy](https://trawelpulse.com/en/privacy)
- [Terms of use](https://trawelpulse.com/en/terms)
- [Support](https://trawelpulse.com/en/support), support@trawelpulse.com

The counts are a guide, not tax or legal advice.
