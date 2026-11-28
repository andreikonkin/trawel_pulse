# Trawel Pulse MCP server reference

Public integration and reviewer documentation. Maintained by **Andrei Konkin**.
Last checked against the server implementation: **2026-11-28**.

Trawel Pulse manages a user's travel account: border crossings, citizenships,
tax residence, visas and citizenship goals. It calculates country day counts
and progress from that history, renders account views, and supplies destination
safety, entry-requirement and flight-search information.

- Production MCP endpoint: **https://trawelpulse.com/mcp** (and **https://trawelpulse.com/v1/mcp**, the latest set to any client; see [Versions](#versions))
- Transport: **Streamable HTTP over HTTPS**, JSON-RPC 2.0
- Authentication: **OAuth Authorization Code with PKCE S256**
- Complete input schemas and annotations: [mcp-tools.json](mcp-tools.json)
- User connection guide: https://trawelpulse.com/en/bot
- [Privacy](https://trawelpulse.com/en/privacy) · [Terms](https://trawelpulse.com/en/terms) · [Support](https://trawelpulse.com/en/support)
- Developer, integration and reviewer contact: **support@trawelpulse.com**

The JSON file is a snapshot exported from `tools/list` at `/v1/mcp` on a local instance of
the production server code, using a synthetic account. It contains public tool
definitions, not account data or credentials. The authenticated live server's
`tools/list` response is authoritative for the deployed version. Output shapes
are described below; the current tools do not advertise formal `outputSchema`
contracts. Accept omitted optional fields and empty collections represented as
`[]` or `null`.

## Contents

- [Versions](#versions)
- [Connection and authentication](#connection-and-authentication)
- [Permissions and side effects](#permissions-and-side-effects)
- [Common parameters and records](#common-parameters-and-records)
- [Tool reference](#tool-reference)
- [Requests and example results](#requests-and-example-results)
- [MCP App UI resources](#mcp-app-ui-resources)
- [Errors, limits and retries](#errors-limits-and-retries)
- [Data handling and service boundaries](#data-handling-and-service-boundaries)
- [Reviewer journeys](#reviewer-journeys)

## Versions

One server, two addresses, each with its own set of tools:

| Address | Set of tools | For |
| --- | --- | --- |
| `https://trawelpulse.com/v1/mcp` | The latest of 1.x (now 1.0.2, 32 tools), to any client. Later 1.x releases update it in place. | The Claude Code plugin from 1.0.2; anyone who adds it by hand. |
| `https://trawelpulse.com/mcp` | Pinned to 1.0.1 (27 tools), as submitted to the app directories still reviewing it; **OpenAI's clients (ChatGPT) get the latest set here**. | The website and the Telegram bot, ChatGPT's app (its address for good), Codex, the directories' submissions. It moves to the latest set for everyone once those reviews are through. |

- `/v1/mcp` only adds to 1.0.1: no tool is removed or renamed, no input is
  removed or changes type, and no input becomes newly required. A client written
  for 1.0.1 works unchanged at `/v1/mcp`.
- At `/mcp`, the server tells OpenAI's clients by their OAuth client: one that
  sends people back to (or is described at) https on `chatgpt.com` or `openai.com`
  gets the latest set. A sign-in only goes back to a client's own addresses, so
  this can't be claimed by another client. A personal link (`/mcp/<key>`) has no
  client to tell and gets the pinned set.
- Both addresses use the same account, sign-in and tokens (see
  [Tokens](#tokens-scope-and-disconnecting)); switching address needs no new sign-in.
- A breaking change would get a new address (`/v2/mcp`); 1.x stays at `/v1/mcp`.

The rest of this reference describes `/v1/mcp`. Where 1.0.1 differs, it says so.

## Connection and authentication

A free Trawel Pulse account is required for the OAuth-protected MCP endpoint,
including tools that return public destination information. No paid subscription
is required. End users normally authenticate with Google or Telegram on trawelpulse.com,
then approve the OAuth client. The assistant does not receive Google or Telegram credentials.
Dedicated reviewer sign-in instructions are supplied privately on request.

### Endpoints

| Purpose | URL |
| --- | --- |
| MCP endpoint and resource | `https://trawelpulse.com/mcp`, or `https://trawelpulse.com/v1/mcp` for the latest set to any client |
| Protected resource metadata | `https://trawelpulse.com/.well-known/oauth-protected-resource/mcp` (for `/v1/mcp`: `…/oauth-protected-resource/v1/mcp`) |
| Authorization server metadata | `https://trawelpulse.com/.well-known/oauth-authorization-server` |
| Authorization | `https://trawelpulse.com/oauth/authorize` |
| Token exchange and refresh | `https://trawelpulse.com/oauth/token` |
| Dynamic client registration | `https://trawelpulse.com/oauth/register` |

An unauthenticated request to `/mcp` (or `/v1/mcp`) returns HTTP `401` with a
`WWW-Authenticate: Bearer` challenge identifying that address's protected
resource metadata, which names that address as the resource.
Use discovery instead of assuming undocumented authentication endpoints.

1. Discover resource and authorization-server metadata.
2. Register a public client using Dynamic Client Registration (RFC 7591), or use
   an HTTPS Client ID Metadata Document (CIMD) containing the matching `client_id`
   and registered `redirect_uris`.
3. Start authorization with `response_type=code`, `client_id`, the registered
   `redirect_uri`, a fresh `state`, `code_challenge`,
   `code_challenge_method=S256`, and `scope=account`. If specifying `resource`,
   use the address you connect to: `https://trawelpulse.com/mcp` (or `/v1/mcp`).
4. The user signs in and approves or declines. Validate the returned `state`
   and authorization-server issuer as appropriate for the client implementation.
5. Exchange the code at `/oauth/token` using form encoding and
   `grant_type=authorization_code`, `client_id`, `code`, `redirect_uri` and
   `code_verifier`. The redirect URI must exactly match the authorized URI.
6. Send `Authorization: Bearer <access_token>` on MCP requests.

Dynamic registration accepts `client_name` and one to ten `redirect_uris` and
returns a `client_id`. These are **public clients**, with
`token_endpoint_auth_method=none`; the service does not issue a client secret.
Do not invent a client secret if a platform asks for one; contact the developer
if that platform cannot use public-client PKCE registration.

Registered callbacks use HTTPS, a supported application's custom scheme, or
HTTP for loopback hosts only. Redirect fragments are not accepted. Clients must
use their own actual callback URI, not one copied from another integration.

### Tokens, scope and disconnecting

- Authorization codes expire after **10 minutes** and are consumed on exchange.
- Access tokens last **1 hour**. The response contains `expires_in`,
  `token_type: "Bearer"`, `refresh_token` and `scope: "account"`.
- Refresh tokens last **90 days** and **rotate on each successful refresh**.
  Use `grant_type=refresh_token`, `client_id` and the current `refresh_token`.
  Save the newly returned access and refresh tokens together; do not keep using
  the previous refresh token. Refresh does not require another interactive login
  while the grant remains valid.
- Tokens are bound to the MCP resource. `/v1/mcp` and `/mcp` are one resource, so
  a token is valid at both. Any other explicitly supplied `resource` is rejected
  with `invalid_target`.
- `account` is the **single current OAuth scope**. It authorizes both read and
  write account tools; separate server-enforced read-only grants do not currently
  exist. A client can limit the tools it enables, but that does not narrow the
  bearer token's server-side authorization.
- Users can disconnect assistants in [account settings](https://trawelpulse.com/en/me).
  This revokes the account's MCP links and OAuth grants for **all assistants**.
  The MCP interface does not expose a tool for deleting the account or changing
  sharing permissions. There is no advertised OAuth token revocation endpoint.

Never include access tokens, refresh tokens, reviewer passwords or private MCP
link keys in public files, prompts, analytics or issue reports. The older
personal-link connection method also exists, but its URL contains a credential;
use the OAuth endpoint above for platform onboarding and public documentation.

### Transport and protocol versions

Use an MCP SDK that supports Streamable HTTP and OAuth discovery. The server
uses stateless HTTP requests with JSON responses; do not depend on a persistent
`Mcp-Session-Id` or a long-lived SSE connection.

Set `Content-Type: application/json` and accept both `application/json` and
`text/event-stream`. SDK integration tests cover protocol versions `2025-03-26`,
`2025-06-18`, `2025-11-25` and `2026-07-28`. For older versions, perform the
`initialize` / `notifications/initialized` handshake and send the negotiated
`MCP-Protocol-Version` on subsequent requests. For `2026-07-28`, use an SDK that
implements `server/discover`, request metadata and matching `Mcp-*` headers;
forwarding only the JSON body can fail protocol validation.

## Permissions and side effects

The connector exposes **32 tools** at `/v1/mcp`: 20 reads and 12 writes (1.0.1 at
`/mcp`: 27 tools, 17 reads and 10 writes). Each tool's exact annotations are
included in [mcp-tools.json](mcp-tools.json).

| Classification | Tools | Current annotations / behavior |
| --- | --- | --- |
| Read, account data | `get_profile`, `list_moves`, `get_days`, `list_people`, `show_goals`, `show_tax_residence`, `show_days`, `show_visited_countries`, `show_calendar` | `readOnlyHint=true`, `destructiveHint=false`, `openWorldHint=false` |
| Read, citizenship programmes (no account data) | `list_citizenship_programs`, `get_citizenship_program`, `calculate_citizenship` | `readOnlyHint=true`, `destructiveHint=false`, `openWorldHint=false` |
| Read, destination/external information | `country_safety`, `countries_safety`, `visa_requirements`, `emergency_numbers`, `natural_disasters`, `recent_changes`, `find_city`, `cheapest_flights` | `readOnlyHint=true`, `destructiveHint=false`, `openWorldHint=true` |
| Write, additive | `add_moves`, `add_visa`, `add_goal`, `show_card` | `readOnlyHint=false`, `destructiveHint=false`, `openWorldHint=false`; persists new account records (`show_card`: a public link to a card, or the one already there) |
| Write, replacement/deletion | `update_profile`, `update_move`, `delete_move`, `update_visa`, `remove_visa`, `update_goal`, `remove_goal`, `set_travel_limit` | `readOnlyHint=false`, `destructiveHint=true`, `openWorldHint=false`; overwrites or removes stored data |

All tools currently advertise `idempotentHint=false`. Add operations can create
duplicates if repeated. Reads do not change the user's travel records, but
requests and tool calls can produce operational logs and usage metadata.

The host platform must apply its own read/write approval policy before calling
a tool. A bearer token authorizes access; it is not a separate confirmation of
each requested change. The server checks account and sharing authorization,
validation and ownership, but does not implement an additional per-action
confirmation-token workflow. For reviewer classification, flag replacements
and deletions as consequential account changes; applying a platform's stricter
sensitive-write policy must not be bypassed by the additive tools' annotations.

`show_card` makes (or reuses) the account's public link to a card, as the
website's Share button does: anyone with the link sees the card, which shows
countries, days and a goal's progress, never dates, notes or names. It is a link
to a picture, not to the account; the user turns it off in the account. It has
no `person` input: it acts on the signed-in user's own account only, and a call
with `person` is rejected as an unexpected argument.

Tools cannot book tickets, make purchases, initiate payments or transfers, send
email, grant account access, or delete the user's Trawel Pulse account.

## Common parameters and records

Input schemas are closed objects (`additionalProperties=false`). Names and
required fields below match the JSON snapshot. Send JSON numbers for integer
fields and JSON booleans for boolean fields; omit unused optional parameters.

| Parameter | Meaning |
| --- | --- |
| `person` | Optional string on account tools. Omit for the signed-in user's own account. Otherwise use an account ID as a string, name or username returned by `list_people`. Matching a name is case-insensitive; IDs avoid ambiguity. |
| `lang` | Optional string on relevant tools. `ru` selects Russian; otherwise English is used. |
| `year` | Optional integer on relevant tools. `list_moves` filters crossings by departure year; `get_days` filters the `counts.years` list while retaining other calculation fields. `show_calendar` and `show_card` default to the current UTC year. |
| Countries | Prefer ISO 3166-1 alpha-2 codes, e.g. `CY`, `GB`, `JP`. Country names are resolved where documented. A visa can also use `SCHENGEN`. IATA city codes are a separate namespace, e.g. `LCA`, `LON`. |
| Dates | Calendar dates in `YYYY-MM-DD`; day counts are calculated as of the server's current UTC date. |
| `index` | Zero-based position in the current profile's visa or goal list. Read the profile immediately before updating/removing an indexed item; deletion shifts later indices. |
| `id` | Integer move ID returned by `add_moves` or `list_moves`; it is not a list index. |

### Shared accounts

`list_people` returns the accounts that another owner has opened to the signed-in
user, with `id`, `name`, `username`, `sections`, `edit`, and an optional `since`.
Sections are `moves`, `days`, `tax`, `visas`, `goals`, or `all`. Reads are filtered
to permitted sections. Writes require `edit=true` and the appropriate section;
unauthorized fields or accounts are rejected. A `person` value does not grant
access by itself. Sharing is configured on the website, not through MCP.
Clearly identify whose account a result or change concerns.

### Move input

A move is **one border crossing**, not an entire round trip. A return journey
normally requires two moves.

| Field | Type | Required | Meaning |
| --- | --- | --- | --- |
| `departed` | string | Yes | Departure date. |
| `from` | string | Yes | Country left. |
| `to` | string | Yes | Country entered; must differ from `from`. |
| `arrived` | string | No | Arrival date; defaults to `departed`. Same day or up to two days later. |
| `from_city`, `to_city` | string | No | Display labels, up to 60 characters each. |
| `note` | string | No | User-provided note, up to 100 characters. |

`update_move` requires `id` plus the required move fields and replaces the whole
record. Omitted optional city/note fields become empty; it is not a partial patch.
Between crossings, the account is treated as staying in the last arrival country.
Future moves contribute planned counts. Missing or inconsistent travel history
can produce `gaps`; ask the user to resolve gaps rather than inventing a crossing.

### Visa input

`country` and `until` are required strings. `country` identifies a country or
`SCHENGEN`; `until` is the expiry date. Optional `kind` is a display label up to
60 characters. `update_visa` additionally requires `index` and replaces that
visa; omitted `kind` becomes empty.

### Citizenship goal input

| Field | Type | Meaning |
| --- | --- | --- |
| `preset` | string, required | A citizenship programme's ID, as `list_citizenship_programs` lists them, or `custom`. At `/mcp` (1.0.1): the identifiers its `add_goal` schema description lists. |
| `dates` | object, optional | `/v1/mcp` only. Qualifying dates as `YYYY-MM-DD`, keyed by the programme's `date_inputs` (`marriage_since`, `spouse_citizen_since`, `cohabitation_since`, `permanent_since`). A date the programme doesn't take is rejected; without a required one, the goal's application forecast stays pending. |
| `since` | string, optional | Residence start date; omitted means the first move is used as the starting point. |
| `country` | string, optional | Required for `preset=custom`. |
| `days`, `years` | integers, optional | Required for a custom goal: target days and the assessment window in years. |
| `last_year_max` | integer, optional | Custom goal's maximum absence in the final year; zero means no limit. |
| `year_absent_max` | integer, optional | Custom goal's maximum absence in any assessment year; zero means no limit. |
| `stretch_max` | integer, optional | Custom goal's maximum continuous absence; zero means no limit. |
| `last_year_mode` | string, optional | Cyprus goal: `deduct` (default) or `ignore` for allowed absences in the final year. |
| `prev_mode` | string, optional | Cyprus goal: `over` (default) or `full` for excess absences in earlier years. |

`update_goal` additionally requires `index` and replaces the selected goal.
Fields omitted from the replacement do not preserve the old goal's values.
Input goal absence field names differ from output profile names:
`last_year_max` → `last_year`, `year_absent_max` → `year_absent`,
`stretch_max` → `stretch`.

## Tool reference

`?` marks optional parameters. The common record definitions above apply.

### Account reads and views

| Tool | Input | Result payload |
| --- | --- | --- |
| `get_profile` | `person?` | `name`, `profile`, `account_url`; shared reads also include `shared` and a filtered profile. Profile includes `citizenships`, `tax_residence`, `tax_rule`, ties (`tied`, `uk`, `us`, `il`), `visas`, `goals`, and `travel_limits` (`country`, `year`, `year_away?`, `trip_away?`). |
| `list_moves` | `person?`, `year?` | `moves` in chronological order, each with its ID, stored fields and `stay_days`; also `gaps`. Future crossings have zero `stay_days`. |
| `get_days` | `person?`, `year?` | `counts`, `visited`, `account_url`. Counts include the calculation date, yearly country totals, Schengen, supported residence tests, goal progress, gaps, and `travel_planning`: for each travel limit, the days away so far (`away`) and planned (`planned`), what is `left`, and the overruns of the year (`over`) and of single trips. |
| `list_people` | `{}` | `user`, `people`, `account_url`; lists existing shares without creating access. |
| `show_goals` | `person?`, `lang?`, `year?` | Account view payload with `view="goals"`, profile, goals, country names and links. |
| `show_tax_residence` | `person?`, `lang?`, `year?` | `view="residence"`, yearly counts, supported residence tests, remaining tax-residence allowances and visa/stay estimates. |
| `show_days` | `person?`, `lang?`, `year?` | `view="days"`, yearly counts and history start. `year` selects the initial view; the payload can include all years. |
| `show_visited_countries` | `person?`, `lang?`, `year?` | `view="visited"`, visited country codes, accumulated stay days, totals and names for the map. |
| `show_calendar` | `person?`, `lang?`, `year?`, `layout?` | `view="calendar"`, selected year, available years, `today`, date `runs`, country totals, display `order`, names, links, `person` and `layout`. `layout` is `year` (default: the year at a glance, a row a month) or `months` (each month's days in a grid). The view switches years by calling `show_calendar` itself. At `/mcp` (1.0.1) there is no `layout`. |

### Citizenship programmes

These read no account data and change nothing.

| Tool | Input | Result payload |
| --- | --- | --- |
| `list_citizenship_programs` | `{}` | `programs` (each with its ID as `preset`, `country`, `name_en`, `name_ru`, `birth_status` and filter summaries), `examples`, `rules_date`. |
| `get_citizenship_program` | `preset` | The programme: `preset`, `country`, names, its conditions (`requirements`), the dates it takes (`date_inputs`), `after_citizenship`, renunciation rules, official `sources` and the date they were checked (`rules_date`); plus `birth_citizenship` for children born in the country to foreign parents. |
| `calculate_citizenship` | `preset`, `since?`, `dates?` | The website's calculator (/citizenship) estimate: `preset`, `country`, `since`, `today`, `rules`, `progress` (credited days and about when one may apply), `program`, `birth_citizenship`. Without actual trips; `since` within the past 30 years, a year ago by default. Missing required dates leave the estimate pending. |

An estimate counts time and days only; language, exams, status and other
conditions are checked separately. To follow a programme by the user's actual
trips, save it as a goal (`add_goal`) and read `get_days` or `show_goals`.

`get_days.counts.years[].countries[]` uses `days` for country-specific tax
counting, `stay` for time present, and `planned` for the projected tax count from
future travel. These are different measures. Goal and residence results depend
on the recorded history and profile; a day-count threshold alone is not a legal
eligibility decision.

### Account writes

| Tool | Input | Result / side effect |
| --- | --- | --- |
| `update_profile` | `person?`, `citizenships?` (string array), `tax_residence?`, `tax_rule?` | Saves specified profile fields; returns `profile`, `account_url`. Omitted fields remain. A nonempty citizenship array replaces the list; an empty array does not clear it. Use `tax_residence="none"` to clear residence; `tax_rule="cy60"` for Cyprus's 60-day rule or `"usual"` for the standard rule. Other profile tie fields cannot be changed through this tool. |
| `add_moves` | `moves` (array of Move input), `person?` | Persists valid entries individually; returns integer IDs in `added`, error strings in `failed`, and `account_url`. **Partial success is possible.** |
| `update_move` | `id`, Move input, `person?` | Replaces one owned/authorized crossing; returns the normalized Move record. |
| `delete_move` | `id`, `person?` | Removes one crossing; returns `{"deleted":true}` on success. No undo tool exists; re-entry creates a new record. |
| `add_visa` | Visa input, `person?` | Appends one visa; returns the resulting `profile`, `account_url`. |
| `update_visa` | `index`, Visa input, `person?` | Replaces one visa; returns the resulting `profile`, `account_url`. |
| `remove_visa` | `index`, `person?` | Removes one visa; returns the resulting `profile`, `account_url`. |
| `add_goal` | Goal input, `person?` | Appends one goal; returns the resulting `profile`, `account_url`. |
| `update_goal` | `index`, Goal input, `person?` | Replaces one goal; returns the resulting `profile`, `account_url`. |
| `remove_goal` | `index`, `person?` | Removes one goal; returns the resulting `profile`, `account_url`. |
| `set_travel_limit` | `country`, `year`, `year_away?`, `trip_away?`, `person?` | Sets the user's own limits of days abroad for one country and year: the most in the year (`year_away`, 0–366) and in one trip (`trip_away`, 0–3660). Both are replaced at once: an omitted limit is cleared, zero is a real limit, and with both omitted the setting is removed. Returns `profile`, `account_url`. These limits never change tax, visa or citizenship counts. |
| `show_card` | `kind`, `year?`, `goal?`, `lang?` | A card to share: `kind` is `year` (a year's travels), `all` (all the years'), `goal` (the goal at index `goal`) or `calendar` (a year's calendar). Makes or reuses the card's public link. Returns `view="card"`, `kind`, `lang`, `url` (the card's page), `image` (a post's picture), `story` (a story's), `year` or `goal`, `account_url`. Own account only. |

After changing crossings, call `get_days` to retrieve recalculated results.
Never treat an error, a partial import or an uncertain HTTP outcome as a completed
change to all requested records.

### Destination information and flight search

All of these tools are reads and advertise `openWorldHint=true`. They do not
modify travel records, apply for visas or create flight reservations.

| Tool | Input | Result payload |
| --- | --- | --- |
| `country_safety` | `country` (string), `lang?` | Country detail: overall advisory level 1–4, contributing source assessments, alerts, hazards, advice, facts and available summaries. A city/resort can resolve to its country. |
| `countries_safety` | `min_level?` (integer, 1–4), `lang?` | Array of compact country objects: `code`, `name`, `name_ru`, `level`, `levels`, ordered by safety level with higher-risk countries first. |
| `visa_requirements` | `passport?`, `destination?` (strings) | Requirements keyed by destination country code. With a destination: `passport` and that country's requirement. An omitted passport uses the profile's first citizenship; if none is available, the call fails. |
| `emergency_numbers` | `country` (string), `citizenship?` | Emergency-number data and available embassy/consulate information. Citizenship defaults to the profile's first citizenship when available. |
| `natural_disasters` | `{}` | Available active GDACS orange/red hazards, including affected countries. |
| `recent_changes` | `country?`, `days?` (integer, default 7), `level_only?` (boolean) | Recent advisory-change records, up to 50; optional country and level-change filters. |
| `find_city` | `query` (string), `lang?` | Array of airport/city matches with IATA codes and display information. |
| `cheapest_flights` | `to`, `from` (strings); `city?`, `date_from?`, `date_to?`, `direct_only?` (boolean), `currency?`, `lang?` | `cheapest` offer and `cheapest_days` data. No reservation or price guarantee is returned. |

For `cheapest_flights`, `to` is a country code/name; `from` is a comma-separated
list of IATA origin city codes (e.g. `LCA,PFO`) or a country code for its major
origins. Optional `city` is a destination IATA city code. Resolve city names with
`find_city`; do not assume a country code is an IATA code. Supply both date fields
for a bounded window of at most a month. Without both, the normal upcoming window
is used. Currency values are `eur`, `usd` or `rub`; the default is EUR for English
and RUB for Russian. `direct_only=true` requests nonstop options.
Prices are based on recent Aviasales search data and may differ at booking time.
Flight data or the cheapest-days portion can be unavailable or empty; report
that limitation instead of constructing an offer.

## Requests and example results

These examples use the `2025-11-25` request format following normal MCP
initialization. IDs and account data are synthetic. Use an SDK for discovery,
initialization, authentication and newer protocol header requirements.

### Discover tools

```json
{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}
```

The result contains `tools`, with `name`, `description`, `inputSchema`,
`annotations`, and UI `_meta` on view tools. See the [full snapshot](mcp-tools.json).

### Read a profile

```json
{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"get_profile","arguments":{}}}
```

Successful tools return both JSON in text content and `structuredContent`.
For example, `result.structuredContent` for a synthetic profile is:

```json
{
  "account_url": "https://trawelpulse.com/en/me",
  "name": "Example traveller",
  "profile": {
    "citizenships": ["CY"],
    "tax_residence": "CY",
    "tax_rule": "",
    "tied": null,
    "uk": {"resident_before": false, "family": false, "accommodation": false, "work": false},
    "us": {"green_card": false, "exempt": false, "closer_connection": false},
    "il": {"home": false, "family": false, "work": false},
    "visas": null,
    "goals": null
  }
}
```

Clients without UI support can use the JSON text. Empty profile lists can be
`null`; do not assume that every field is a populated array.

### Add a round trip

This is a **write** example. Execute only when the user requests and authorizes
these particular dates and destinations, using the host's required approval flow.

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "add_moves",
    "arguments": {
      "moves": [
        {"departed": "2026-01-10", "from": "CY", "to": "GB"},
        {"departed": "2026-01-20", "from": "GB", "to": "CY"}
      ]
    }
  }
}
```

Successful `result.structuredContent` for these two valid entries:

```json
{"account_url":"https://trawelpulse.com/en/me?tab=moves","added":[1,2],"failed":[]}
```

`added` contains IDs, not a guarantee that every submitted item succeeded.
If `failed` is nonempty, report the partial import and resolve those errors.
Use `list_moves` to confirm saved entries before retrying after a timeout.

### Tool-level failure

For a syntactically valid `delete_move` request with a nonexistent ID, the
HTTP/RPC request can succeed while the tool fails. The response is:

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "result": {
    "content": [{"type":"text","text":"no such move"}],
    "isError": true
  }
}
```

Always check `result.isError` as well as HTTP status and a JSON-RPC `error`.

## MCP App UI resources

| Tool(s) | Resource |
| --- | --- |
| `show_goals`, `show_tax_residence`, `show_days`, `show_visited_countries` | `ui://trawelpulse/account.html` |
| `show_calendar` | `ui://trawelpulse/calendar.html` |
| `show_card` | `ui://trawelpulse/card.html` (`/v1/mcp` only) |

Use `resources/list` and `resources/read` for resources. Each has MIME type
`text/html;profile=mcp-app`. View tools declare `_meta.ui.resourceUri` and also
`openai/outputTemplate` for compatible hosts. Tool results supply view data in
`structuredContent`; the host supplies it to the iframe through the MCP Apps
bridge. Do not navigate to `ui://` in a normal browser.

The HTML/CSS/JavaScript and map geometry are bundled. The account's and the
calendar's resources declare empty `connectDomains` and `resourceDomains`; they
require no external scripts, fonts or network requests. The card's resource
declares `resourceDomains: ["https://trawelpulse.com"]` (and the same in
`openai/widgetCSP`) for the card's picture only, and no `connectDomains`. The
calendar view switches years by calling `show_calendar` through the host
(`tools/call`, or `window.openai.callTool` in ChatGPT). A host without MCP Apps support should
present the tool's text/structured results. UI display modes and support depend
on the host; compatibility with every host is not implied by this reference.

## Errors, limits and retries

| Layer | Behavior and examples | Client response |
| --- | --- | --- |
| Authentication | HTTP `401` for a missing, expired or revoked access token; bearer discovery challenge | Refresh once with the latest refresh token or request interactive reconnection. |
| OAuth | JSON `error` / `error_description`, e.g. `invalid_client`, `invalid_grant`, `invalid_target`, `invalid_request`, `invalid_redirect_uri`, `unsupported_grant_type` | Fix the indicated client/configuration problem. An invalid refresh grant requires reconnection. |
| Protocol/schema | JSON-RPC errors for invalid requests, unknown tools, missing required arguments or unexpected properties; newer header/body mismatches can produce `-32020` | Follow the negotiated protocol and current input schema; do not retry the same invalid request unchanged. |
| Tool execution | `result.isError=true`, with text such as `bad dates`, `bad countries`, `no such move`, `no visa at that index`, `no goal at that index`, `too many`, or sharing-denial details | Explain the failure and request corrected data or access. |
| Partial batch | `add_moves` can return both nonempty `added` and `failed` without `isError=true` | Report each failed entry; do not repeat already successful entries. |
| Network/upstream | Timeout, HTTP error, unavailable or empty external data | For reads, use bounded retries with backoff. For writes, verify state before retrying; there are no idempotency keys. |

Current validation limits:

- At most **200 moves per `add_moves` call** and **3,000 stored moves per account**.
- Departure dates from **1950**, up to one year ahead of the server's current
  date; arrival must be between departure and two calendar days later.
- At most **5 citizenships**, **20 visas** and **10 goals** per profile.
- Custom goals: **1–15 years**, target days between 1 and `years * 366`, absence
  limits up to 366 days for yearly/final-year limits and 3,660 for a continuous
  absence. Zero absence limits mean no limit.
- `show_calendar.year` is between **1950 and the next calendar year**;
  `layout` is `year` or `months`.
- Travel limits: `year` from **1950 to five years ahead**, `year_away` **0–366**,
  `trip_away` **0–3,660**, at most **100** country/year settings per profile.
- `show_card`: `year` from **1900 to the next calendar year** (`all` ignores it);
  `goal` an index of the profile's goals.
- `recent_changes` returns at most **50** records.

There is no published per-minute MCP request quota or response-time SLA.
Do not interpret that as unlimited capacity. Avoid concurrent bulk imports,
cache suitable read results, and honor any HTTP `429` / `Retry-After` response
from infrastructure or providers. Contact support for sustained high-volume use.

## Data handling and service boundaries

- Account tools access recorded crossings, profile citizenships, residence/tie
  settings, visas and goals, plus accounts explicitly shared with the user.
  They do not have direct inbox or bank-account access: users may paste travel
  details for their assistant to interpret before making explicit tool calls.
- Destination data is aggregated from external sources; flight search uses
  external travel data. Dates, routes and other necessary query parameters can
  be sent to the relevant provider. The UI's lack of external requests does not
  mean the backend never contacts external services.
- Server storage keeps account records. OAuth codes and tokens are stored as
  hashes. Tool activity logging includes tool names and selected outcome/error
  details, not a full transcript of every argument and response. Operational MCP
  logs and usage metadata also record request/client information; do not put
  unrelated personal data or credentials in tool arguments or client metadata.
- Account activity logs are retained for 90 days. Account deletion is managed on
  the website, with a 28-day restoration period; MCP access is blocked while an
  account is marked for deletion. Refer to the [privacy policy](https://trawelpulse.com/en/privacy)
  for the published data-handling terms.
- A card's public link (`show_card`, or Share on the website) shows countries,
  days and a goal's progress to anyone with it; never dates, notes or names. The
  picture is drawn by trawelpulse.com. The owner turns the link off in the account.
- Advisory data, entry requirements and tax/immigration rules can change.
  Calculations are informational estimates from supported rules and the supplied
  history, not professional advice or an official residence/citizenship decision.
  Flight search does not create a booking, charge a user or guarantee a fare.

## Reviewer journeys

Request a dedicated demo account and its current sign-in instructions through
**support@trawelpulse.com** or the submission portal's private credential channel.
No reviewer password, access token or private key is published in this repository.
Do not use a personal production account for destructive tests.

Suggested tests after completing OAuth:

1. Discover all tools and read the synthetic account with `get_profile`,
   `list_moves` and `get_days`.
2. Add an explicitly approved two-crossing trip. Verify `added` and `failed`,
   retrieve it with `list_moves`, and recalculate with `get_days`.
3. Update one saved crossing, then remove it with the host's required approval.
   Check that another account's inaccessible IDs cannot be changed.
4. Read the current profile, add/update/remove a demo visa or goal by its current
   index, and verify the profile after each operation.
5. Exercise each `show_*` tool, checking both text fallback and UI where the host
   supports MCP Apps. Read all three UI resources. In the calendar, switch the
   year and the layout.
6. With owner-prepared shares, use `list_people` and `person` to verify a read-only
   share, a writable share, a restricted section and an account with no share.
7. Query a destination's safety/entry rules and a flight route. Handle missing
   upstream data without claiming an unavailable result or reservation.
8. Test an invalid date, an unknown move ID and a batch containing a valid and an
   invalid crossing. Check protocol errors, tool errors and partial success.
9. Refresh the OAuth token using the rotated refresh token, then disconnect in
   account settings and verify that previous access is rejected.
10. List the citizenship programmes, read one, and estimate it with and without
    the dates it takes; save it as a goal with its dates and read the progress.
11. Set a travel limit for a country and year, read `get_days.counts.travel_planning`,
    then remove it by calling `set_travel_limit` with both limits omitted.
12. Make a card with `show_card`, open its `url` and `image` without signing in,
    then turn the link off in the account and check the page is gone.

Actual review credentials, client-specific registration details and any access
restrictions will be provided privately. Successful tests on one assistant do
not constitute approval or a compatibility guarantee for another platform.
