# fatetell-mcp/
> L2 | Public storefront repository for the hosted FateTell MCP server. It is metadata only: no server code, no build, no secrets.

One repository serves several distribution channels at once. The repository root is the plugin root for Claude and Cursor, the Agent Plugins v1 package root for Hermes Agent and the Gemini CLI extension root; the single-plugin Claude marketplace points at it with `"source": "./"`. Each host reads only its own manifest (`.claude-plugin/`, `.cursor-plugin/`, root `plugin.json`, `gemini-extension.json`), so the four never override one another; what they share is the skill under `skills/`.

## Why there is no CLAUDE.md here

The repository root is a Claude plugin root. Claude Code does not load a plugin's `CLAUDE.md` as context, and `claude plugin validate` reports it as a warning, so `--strict` (the gate for this repository) fails. This file, `AGENTS.md`, carries the map instead; it is not a plugin component and no host loads it.

## Members

.claude-plugin/plugin.json: Claude plugin manifest, read by claude.ai, Cowork and Claude Code; `name` is `fatetell`; `version` pins installed users, so bump it on every release; `icon`, `documentationUrl`, `supportUrl`, `privacyPolicyUrl`, `termsOfServiceUrl` feed only the Anthropic directory listing and are allowed only here, never in a marketplace entry
.claude-plugin/marketplace.json: single-plugin Claude marketplace (`claude plugin marketplace add RussGuo/fatetell-mcp`); entry name must equal the manifest name; no `version` and no directory-listing fields in the entry
.mcp.json: Claude plugin connector, `type: http` to `https://mcp.fatetell.com/mcp` (lazy sign-in: anonymous tools work, account tools answer 401); must match the URL submitted as the Claude connector so users see one set of tools; no `oauth` block, the server supports CIMD and DCR
.cursor-plugin/plugin.json: Cursor plugin manifest with `mcpServers` inline, which replaces Cursor's default `mcp.json` discovery so Cursor never reads the Claude `.mcp.json` or the portable root `mcp.json`; points at `https://mcp.fatetell.com/login/mcp` because Cursor (and Grok Bot) only handle a 401 during `initialize`; `logo` is the plated icon
plugin.json: Agent Plugins v1 manifest (closed schema, `$schema` 1.0.0), the entry point Hermes Agent reads on `hermes plugins install RussGuo/fatetell-mcp`; name, version, author, license and keywords mirror `.claude-plugin/plugin.json`, and the description is the same sentence with "for your agent" in place of "in Claude", so bump both versions together; the only host-specific data is `extensions."com.nousresearch.hermes".servers.fatetell.trust: untrusted`, which makes Hermes ask before every call not annotated `readOnlyHint: true` (today only `create_saved_chart`); the Hermes plugin catalog pins a full commit SHA of this repository, so a change here reaches catalog installs only through a SHA-bump PR to `NousResearch/hermes-agent`
mcp.json: Agent Plugins v1 connector, `type: streamable-http` to `https://mcp.fatetell.com/mcp` under server name `fatetell` (must equal the key in `plugin.json` extensions); the portable twin of the Claude `.mcp.json`, kept as a separate file because the two formats spell the transport differently; no headers and no credentials, and the format cannot declare OAuth, so the README has Hermes users add a native `mcp_servers.fatetell` entry on `/login/mcp` with `auth: oauth` (Hermes signs in only on a 401 while connecting) and repeat `trust: untrusted`, since a same-name native entry replaces the package's
gemini-extension.json: Gemini CLI extension manifest, read by `gemini extensions install https://github.com/RussGuo/fatetell-mcp`; `name` `fatetell` (Gemini wants it to match the install directory), `version` and description mirror root `plugin.json`, so bump all version fields together; `mcpServers.fatetell.httpUrl` is `/login/mcp` (`httpUrl` is Streamable HTTP, `url` would mean SSE); no `contextFileName`, because Gemini discovers the skill from `skills/` and loads it on demand, whereas a context file is loaded into every session; the geminicli.com extension gallery lists it by crawling public repositories tagged `gemini-cli-extension` (topic set on this repository), daily, once the manifest validates
glama.json: Glama ownership file (`$schema` https://glama.ai/mcp/schemas/server.json, `maintainers: ["RussGuo"]`); lets the GitHub user RussGuo claim the auto-indexed listing at glama.ai/mcp/servers/RussGuo/fatetell-mcp; after any change, run the Claim flow on Glama again
skills/fatetell-bazi/SKILL.md: the reading skill shared by all four packages (auto-discovered from `skills/`, the Agent Plugins and Gemini CLI fixed location as well; no manifest lists it, since listing replaces discovery); frontmatter only `name` and `description` (Agent Skills standard); copied verbatim from the private server repository's `plugin/` directory, which stays the source
server.json: MCP Registry entry `com.fatetell/mcp` (schema 2025-12-11, published 2026-11-03 as 0.1.0); lists only the default endpoint `/mcp`; `version` is immutable once published, bump before each publish; `repository` points at this storefront repository, and the README states that the service itself is not open source
assets/ (six image files; the Anthropic directory scanner holds for manual review any markdown that spells an image filename, so they are described here without extensions):
- icon, SVG: 1024×1024 vector master of the plated icon: the designer's outline from the brand's Illustrator file (fatetell-logo-0705.pdf, page 2) on a full #F0F0F0 plate (the website's `--paper`, oklch(0.955 0 0)), 12% margin; the source grain is a bitmap, so the SVG carries a radial gradient fitted to its average shading
- icon-1024, PNG: 1024×1024 render of the same plated icon from the Illustrator file with its grain; the Cursor form's Logotype URL
- icon-512, PNG: 512×512 of the same plated icon; Registry `icons`
- icon-plate-512, PNG: byte-identical copy of icon-512, kept so the Cursor `logo` path ("1:1 with background plate") stays valid
- icon-mark-512, PNG: 512×512 transparent ink-drop mark, same framing as the website's logo image, for hosts that draw their own background
- icon-512-white, PNG: the mark on a white plate with 8% margin; the Claude plugin `icon` and the connector listing's custom icon, because the directory card draws its own frame and the grey plate read as a double frame
README.md: the page directories and reviewers read: endpoints, install per client, tool table, sign-in tiers, limits, privacy, support, registry; tool names, titles and the first sentence of each description are copied verbatim from the server repository's generated `docs/tools.md`
LICENSE: MIT, Copyright (c) 2026 FateTell; covers this repository's files only, not the service
.gitignore: keeps `.DS_Store`, `Thumbs.db` and `__MACOSX` out (the Anthropic portal blocks plugins that contain them), plus logs and env files

## Rules

- Production addresses only: `mcp.fatetell.com`, `auth.fatetell.com`, `web.fatetell.com`, `fatetell.com`. No test hosts, internal domains, keys or tokens.
- Tool descriptions, manifests and README carry no promotional copy and no instructions to the model; usage guidance lives in the skill.
- Endpoint per host: Claude plugin, Hermes package and Registry use `/mcp`; Cursor plugin, Gemini CLI extension and the Hermes sign-in entry in the README use `/login/mcp` (Gemini CLI, like Cursor, starts OAuth only on a 401 while connecting); ChatGPT uses `/chatgpt/mcp` (never in a manifest here).
- When the server's tools change, regenerate `docs/tools.md` in the server repository first, then copy the table here; when the skill changes, change it there first, then copy it here.
- Gate before every push: `claude plugin validate --strict .` passes, `hermes plugins validate . --install-deps` passes, every JSON file parses, `server.json` validates against its `$schema`.

[PROTOCOL]: 变更时更新此头部,然后检查 CLAUDE.md(本仓库刻意没有 CLAUDE.md;上级地图是私有服务端仓库的 CLAUDE.md 与 plugin/ 目录)
