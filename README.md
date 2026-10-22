<!-- Placeholder to fill before publishing: <SECURITY_EMAIL> (proposed security@fatetell.com). -->

# FateTell MCP

FateTell computes; your AI interprets.

FateTell MCP is a hosted remote MCP server (Streamable HTTP) that returns deterministic BaZi (Four Pillars) charts, Chinese almanac data and the charts saved in a FateTell account to any MCP client. The server calls no language model: the AI you already use reads and explains the results. There is nothing to download or run.

This repository holds the public metadata for that server: a plugin for Claude and Claude Code, a plugin for Cursor, the MCP Registry entry, and the `fatetell-bazi` skill on how to read FateTell results. The server itself is not in this repository.

## Endpoints

| Endpoint | Behavior | Use it for |
|---|---|---|
| `https://mcp.fatetell.com/mcp` | Default. Connecting needs no account; chart casting and the almanac work anonymously. A tool that needs an account answers with an HTTP 401 challenge, and the client then opens FateTell's sign-in page. | Claude (web, desktop, mobile), Claude Code, Codex, VS Code, and most other clients |
| `https://mcp.fatetell.com/login/mcp` | Sign in on connect. The first request already answers 401, so sign-in happens while the client is connecting. Every tool on this endpoint is used signed in; there is no anonymous use. | Clients that start sign-in only while connecting: Cursor, Grok Bot, Manus |
| `https://mcp.fatetell.com/chatgpt/mcp` | ChatGPT's own endpoint; the tools are identical, sign-in follows ChatGPT's app conventions. | ChatGPT only |

The tools, their inputs and their results are the same on every endpoint. Sign-in uses OAuth 2.1; there is no key or token to configure in advance.

## Install

### Claude (web, desktop, mobile)

Settings → Connectors → Add → **Add custom connector**. Name `FateTell`, URL:

```
https://mcp.fatetell.com/mcp
```

Under Authentication choose **Sign in when needed** (Claude detects "No sign-in" by default because the free tools work without an account; with that choice it never offers sign-in for account features). Leave the OAuth client on Claude's published identity and enter no key. Connectors follow your Claude account, so the desktop and mobile apps pick it up. Once the FateTell plugin is listed in the Claude directory, installing it from there adds the same connector together with the skill.

### Claude Code

This repository is also a Claude Code plugin marketplace. The plugin adds the connector (`/mcp`) and the `fatetell-bazi` skill:

```sh
claude plugin marketplace add RussGuo/fatetell-mcp
claude plugin install fatetell@fatetell
```

Connector only, without the skill:

```sh
claude mcp add --transport http fatetell https://mcp.fatetell.com/mcp
```

Add `--scope user` before the name to use it in every project. To sign in ahead of time, run `claude mcp login fatetell`; otherwise the first account feature prompts you. `/mcp` shows the connection status.

### Codex

```sh
codex mcp add fatetell --url https://mcp.fatetell.com/mcp
```

Codex opens FateTell's sign-in page right away. If the browser does not open, run `codex mcp login fatetell`. In the ChatGPT desktop app, Codex has its own page: Settings → Plugins → MCPs → Add, with the same URL.

### ChatGPT

On chatgpt.com (or the desktop app): Plugins → Add → **Add custom MCP server**. Name `FateTell`, keep Connection on Server URL and use ChatGPT's endpoint:

```
https://mcp.fatetell.com/chatgpt/mcp
```

Keep Authentication on OAuth, leave the advanced OAuth settings alone, tick the confirmation and choose **Create as a plugin**. ChatGPT opens FateTell's sign-in page; choose **Allow**. Use this endpoint only in ChatGPT.

### Cursor

Install FateTell from the Cursor Marketplace once it is listed; the plugin connects to the sign-in endpoint. To add it by hand, open this link and choose **Install**:

[Add to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=fatetell&config=eyJ1cmwiOiJodHRwczovL21jcC5mYXRldGVsbC5jb20vbG9naW4vbWNwIn0=)

or add this entry to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "fatetell": { "url": "https://mcp.fatetell.com/login/mcp" }
  }
}
```

Cursor asks you to sign in to FateTell while it connects. Cursor handles a sign-in challenge only at that stage, which is why it uses `/login/mcp` rather than the default endpoint.

### VS Code

```sh
code --add-mcp '{"name":"fatetell","url":"https://mcp.fatetell.com/mcp"}'
```

The first time a chat uses a feature that needs your account, follow the prompt to sign in.

### Grok Bot

Grok Bot is Cursor's cloud agent. Add FateTell as a remote MCP connector with the sign-in endpoint, or ask Grok Bot to add it for you:

```
https://mcp.fatetell.com/login/mcp
```

### Manus

On manus.im: Plugins → Create → **Custom MCP**. Server Name `FateTell`, Transport Type HTTP, Server URL:

```
https://mcp.fatetell.com/login/mcp
```

Leave the headers empty and choose **Save**; Manus starts sign-in while connecting.

### Other MCP clients

Add a remote MCP server named `fatetell` over Streamable HTTP (often called `http`) with the default endpoint, OAuth for authentication and no key or header:

```
https://mcp.fatetell.com/mcp
```

If the client connects but never opens a sign-in page when an account feature is used, it is one of the clients that only signs in while connecting: use `https://mcp.fatetell.com/login/mcp` instead.

An agent that can run commands or edit its own MCP settings can also set itself up from [install.md](https://mcp.fatetell.com/install.md):

> Read https://mcp.fatetell.com/install.md and follow it to connect me to FateTell MCP.

## Tools

Nine tools, all deterministic. Access tiers are explained under [Sign-in and permissions](#sign-in-and-permissions).

| Tool | Title | Access | Scope | What it does |
|---|---|---|---|---|
| `cast_bazi_chart` | Cast BaZi chart | Anonymous | — | Compute a BaZi chart from birth data: pillars with Ten Gods, hidden stems, Na Yin, Day Master, all luck pillars, palaces and today's pillars; southern-hemisphere births use cross-region rules. |
| `get_almanac` | Get almanac | Anonymous | — | Return the Chinese almanac for one to 31 consecutive days: Gregorian and lunar dates, solar terms, and the year, month and day pillars. |
| `get_fatetell_account` | Get FateTell account | Signed in | `account:read` | Return the connected FateTell account: masked email, sign-in method, interface language, time zone, membership status, the permissions granted to this client, and this account's remaining quota for each tool. |
| `list_saved_charts` | List saved charts | Signed in | `chart:read` | List the charts saved in the connected FateTell account: id, display name, gender, whether it is the user's own chart, hemisphere, chart rule version and whether the chart has been computed. |
| `get_saved_chart` | Get saved chart | Signed in | `chart:read` | Return one saved chart in full: birth date, time and time zone (no birthplace), pillars with Ten Gods, hidden stems and Na Yin, all luck pillars (current one marked), palaces, the stored Day Master strength and favorable elements, and today's pillars. |
| `get_bazi_judgment` | Get BaZi judgment | Member | `chart:read` | Return FateTell's Day Master verdict (strong, weak, follow-strong, follow-weak) and ranked favorable, unfavorable and neutral elements under rule set v1.0, for birth data or a saved chart_id. |
| `get_daily_fortune` | Get daily fortune | Member | `fortune:read` | Return a saved chart's daily fortune for one date: energy level (high / medium / low), the day's stems-and-branches relations with the natal chart, the day's do/avoid items, the day pillar, this chart's year and month pillars and month clash. |
| `get_chart_calendar` | Get chart calendar | Member | `fortune:read` | Return a saved chart's view of upcoming dates: for each day (up to 31) the year, month and day pillars adjusted for this chart's hemisphere, relations with the natal pillars, month clash and solar terms; or for each month (up to 12) the month pillar and its natal relations. |
| `create_saved_chart` | Create saved chart | Member | `chart:write` | Save a new chart to the connected FateTell account from name, gender, Gregorian birth date, optional birth time and birthplace; time zone and hemisphere come from the birthplace as in the app. |

`create_saved_chart` is the only tool that writes; it adds a chart and cannot edit or delete existing ones. Every other tool is read-only. Every result carries its rule version, and the same input always gives the same result.

## Sign-in and permissions

Three access tiers; each includes everything in the tiers before it.

| Tier | What you need | Tools |
|---|---|---|
| Anonymous | Nothing (default endpoint only) | `cast_bazi_chart`, `get_almanac` |
| Signed in | A FateTell account | `get_fatetell_account`, `list_saved_charts`, `get_saved_chart` |
| Member | An active FateTell membership | `get_bazi_judgment`, `get_daily_fortune`, `get_chart_calendar`, `create_saved_chart` |

- **OAuth 2.1**, authorization code with PKCE (S256). The authorization server is `https://auth.fatetell.com`; you sign in and approve on FateTell's own pages at `https://web.fatetell.com`. Your AI never sees your password.
- **Client registration**: Client ID Metadata Documents and Dynamic Client Registration are both supported, so no client ID needs to be configured.
- **Scopes**: `account:read` (account and membership status), `chart:read` (saved charts and FateTell's judgment), `fortune:read` (daily fortune and chart calendar), `chart:write` (create new charts; listed separately on the consent page), `offline_access` (stay connected without signing in again). ChatGPT additionally requests `openid` and `email`.
- **Membership is checked on every call**, not fixed at sign-in. The same membership works in the FateTell app, on the website and here; what you can do here matches what you can do in the app.
- **Disconnecting**: FateTell website → System Settings → **Connected apps** → Disconnect. The AI loses access within a minute.

## Usage limits

Every tool has a usage cap. The caps exist only to prevent abuse; they are not an allowance, they are not linked to any usage count in the FateTell app, and reaching one returns only the cap and its reset time.

- Signed in: each tool has a daily cap per account, plus a combined daily cap. Days run on UTC.
- Anonymous on the default endpoint: clients that do not identify anonymous users share one public cap that resets hourly. When it runs out, signing in moves you onto your own account's count.
- Asking the same thing again within 24 hours does not count. One almanac lookup of 1 to 31 days counts once. Checking your account does not count.
- Saved charts are also limited by the same total as in the app.

Once signed in, `get_fatetell_account` reports each tool's cap, current use and reset time.

## Data and privacy

- **No language model on the server.** FateTell MCP computes and reads data. It never sees your conversation with your AI.
- **Anonymous use is not stored.** Inputs and results of `cast_bazi_chart` and `get_almanac` are not written to any database and are not linked to anyone. To avoid counting repeated questions, a result is held in server memory for up to 24 hours, keyed by a hash of the request, then discarded.
- **Logs** record which tool ran, how long it took, the outcome, any error code and a keyed hash of the input. They never record birth dates, birth times, longitudes, place names or chart names.
- **Signed-in reads** return only what you approved on the consent page. Saved charts are returned without the birthplace as written and without coordinates.
- **Saving a chart** is the only feature that takes a place name: FateTell uses it to find the time zone and hemisphere, exactly as when you create a chart in the app. FateTell MCP itself does not keep or log it. Saving a chart for someone else requires their knowledge and consent.
- **What this repository's plugins send**: the plugins contain no code. They connect only to `mcp.fatetell.com`; sign-in goes through `auth.fatetell.com` and `web.fatetell.com`. Nothing is sent anywhere else.

Once your AI has a result, how it stores it (for example in your chat history) is governed by that app's own policy.

BaZi results are for reference and are not a substitute for medical, legal or financial advice. How your AI interprets FateTell's data is the AI's own reading.

- Privacy Policy: https://fatetell.com/privacyPolicy
- Terms of Service: https://fatetell.com/terms

## Support and security

- Support: `contact@fatetell.com`
- Security reports: `<SECURITY_EMAIL>`. Please report vulnerabilities privately rather than in a public issue.

## Registry

MCP Registry name: `com.fatetell/mcp`. The entry is [`server.json`](server.json) and lists only the default endpoint, `https://mcp.fatetell.com/mcp`.

## Repository contents

```
.claude-plugin/plugin.json        Claude plugin manifest (Claude, Cowork, Claude Code)
.claude-plugin/marketplace.json   single-plugin marketplace: this repository
.mcp.json                         Claude plugin connector: https://mcp.fatetell.com/mcp
.cursor-plugin/plugin.json        Cursor plugin manifest, connector inline: https://mcp.fatetell.com/login/mcp
skills/fatetell-bazi/SKILL.md     reading skill, shared by both plugins
server.json                       MCP Registry entry
assets/icon.svg                   icon, vector, on a paper-colored plate
assets/icon-1024.png              icon, 1024×1024, on a paper-colored plate
assets/icon-512.png               icon, 512×512, on a paper-colored plate
assets/icon-plate-512.png         same as icon-512.png (Cursor logo path)
assets/icon-mark-512.png          mark only, 512×512, transparent
```

Validate the Claude plugin with `claude plugin validate --strict .`.

## License

The files in this repository are released under the [MIT License](LICENSE). The license covers this repository only; the FateTell service and its calculation engine are not open source.

## 简体中文摘要

FateTell 负责算对,你的 AI 负责解读。FateTell MCP 是托管的远程 MCP 服务器,返回确定性的八字排盘、历法与 FateTell 账户里保存的命盘;服务端不调用任何大模型,不需要下载或安装。

- **两个地址**:`https://mcp.fatetell.com/mcp` 是默认地址,连接不需要账户,排盘与历法匿名可用,调用需要账户的工具时才登录,适合 Claude、Claude Code、Codex、VS Code 等;`https://mcp.fatetell.com/login/mcp` 是「连接即登录」地址,第一次请求就要求登录,给只在连接阶段处理登录的客户端:Cursor、Grok Bot、Manus。ChatGPT 用它自己的 `https://mcp.fatetell.com/chatgpt/mcp`。
- **三档权限**:匿名(排盘、历法)、登录(账户、已存命盘)、会员(判定、日运、档案视角的历法、新建命盘)。OAuth 2.1 + PKCE,登录只在 FateTell 自己的页面完成。
- **使用上限**只为防滥用,不是额度,与 App 内次数不相通;24 小时内重复的请求不重复计数。
- **隐私**:匿名调用不落库;日志不记出生日期、时间、地名与命盘名;只有新建命盘会收出生地名,由后端解析时区与半球,本服务不存、不记。
- 详细条款见[隐私政策](https://fatetell.com/privacyPolicy)与[服务条款](https://fatetell.com/terms)。
