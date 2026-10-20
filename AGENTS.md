# fatetell-mcp/
> L2 | Public storefront repository for the hosted FateTell MCP server. It is metadata only: no server code, no build, no secrets.

One repository serves four distribution channels at once. The repository root is the plugin root for both Claude and Cursor, and the single-plugin Claude marketplace points at it with `"source": "./"`.

## Why there is no CLAUDE.md here

The repository root is a Claude plugin root. Claude Code does not load a plugin's `CLAUDE.md` as context, and `claude plugin validate` reports it as a warning, so `--strict` (the gate for this repository) fails. This file, `AGENTS.md`, carries the map instead; it is not a plugin component and no host loads it.

## Members

.claude-plugin/plugin.json: Claude plugin manifest, read by claude.ai, Cowork and Claude Code; `name` is `fatetell`; `version` pins installed users, so bump it on every release; `icon`, `documentationUrl`, `supportUrl`, `privacyPolicyUrl`, `termsOfServiceUrl` feed only the Anthropic directory listing and are allowed only here, never in a marketplace entry
.claude-plugin/marketplace.json: single-plugin Claude marketplace (`claude plugin marketplace add RussGuo/fatetell-mcp`); entry name must equal the manifest name; no `version` and no directory-listing fields in the entry
.mcp.json: Claude plugin connector, `type: http` to `https://mcp.fatetell.com/mcp` (lazy sign-in: anonymous tools work, account tools answer 401); must match the URL submitted as the Claude connector so users see one set of tools; no `oauth` block, the server supports CIMD and DCR
.cursor-plugin/plugin.json: Cursor plugin manifest with `mcpServers` inline, which replaces Cursor's default `mcp.json` discovery so Cursor never reads the Claude `.mcp.json`; points at `https://mcp.fatetell.com/login/mcp` because Cursor (and Grok Bot) only handle a 401 during `initialize`; `logo` is the plated icon
skills/fatetell-bazi/SKILL.md: the reading skill shared by both plugins (auto-discovered from `skills/`; neither manifest lists it, since listing replaces discovery); frontmatter only `name` and `description` (Agent Skills standard); copied verbatim from the private server repository's `plugin/` directory, which stays the source
server.json: MCP Registry entry `com.fatetell/mcp` (schema 2025-12-11); lists only the default endpoint `/mcp`; `version` is immutable once published, bump before each publish; no `repository` field because the server source is private
assets/icon-512.png: 512×512 transparent ink-drop mark, generated from the brand logo with Pillow; Claude `icon` and Registry `icons`
assets/icon-plate-512.png: 512×512 mark centred on a #f3f2ef paper plate; Cursor `logo` and the Cursor form's Logotype URL ("1:1 with background plate")
README.md: the page directories and reviewers read: endpoints, install per client, tool table, sign-in tiers, limits, privacy, support, registry; tool names, titles and the first sentence of each description are copied verbatim from the server repository's generated `docs/tools.md`
LICENSE: MIT, Copyright (c) 2026 FateTell; covers this repository's files only, not the service
.gitignore: keeps `.DS_Store`, `Thumbs.db` and `__MACOSX` out (the Anthropic portal blocks plugins that contain them), plus logs and env files

## Rules

- Production addresses only: `mcp.fatetell.com`, `auth.fatetell.com`, `web.fatetell.com`, `fatetell.com`. No test hosts, internal domains, keys or tokens.
- Tool descriptions, manifests and README carry no promotional copy and no instructions to the model; usage guidance lives in the skill.
- Endpoint per host: Claude plugin and Registry use `/mcp`; Cursor plugin uses `/login/mcp`; ChatGPT uses `/chatgpt/mcp` (never in a manifest here).
- When the server's tools change, regenerate `docs/tools.md` in the server repository first, then copy the table here; when the skill changes, change it there first, then copy it here.
- Gate before every push: `claude plugin validate --strict .` passes, every JSON file parses, `server.json` validates against its `$schema`.

[PROTOCOL]: 变更时更新此头部,然后检查 CLAUDE.md(本仓库刻意没有 CLAUDE.md;上级地图是私有服务端仓库的 CLAUDE.md 与 plugin/ 目录)
