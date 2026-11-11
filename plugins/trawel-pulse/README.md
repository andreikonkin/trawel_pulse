# Trawel Pulse

Your travel account in Claude. Tell Claude about your trips, or paste ticket emails or a bank statement, and it keeps them in your Trawel Pulse account. From them it counts the days in each country as that country counts them, your tax residence and how long more you may stay, your progress to citizenship by residence and where you've been, and draws them right in the chat. It also answers travel questions: a country's safety level from several governments' advisories, visa rules for your passport, emergency numbers, natural disasters, recent changes and the cheapest flights.

## What's in the plugin

- **The Trawel Pulse MCP server**, `https://trawelpulse.com/mcp` (`.mcp.json`): a remote server over HTTPS. Nothing runs on your machine.
- **A skill**, `skills/trawel-pulse/SKILL.md`: how to enter trips as moves and answer from the account.
- The same server for ChatGPT and Codex (`plugin.json`, `mcp.json`, [Agent Plugins](https://agent-plugins.org)) and an icon.

## Sign-in and data

The first time a tool runs, you sign in with Telegram on trawelpulse.com (OAuth) and allow Claude in. The server keeps what you enter in your account: moves, citizenships, tax residence, visas and goals. Tool calls go to trawelpulse.com only; the plugin stores nothing on your machine and sends nothing anywhere else. You can turn assistants off at any time in the account's settings on trawelpulse.com.

- [Privacy policy](https://trawelpulse.com/en/privacy)
- [Terms of use](https://trawelpulse.com/en/terms)
- [Support](https://trawelpulse.com/en/support), support@trawelpulse.com

The counts are a guide, not tax or legal advice.
