# FateTell MCP

[![FateTell MCP connector on Glama](https://glama.ai/mcp/connectors/com.fatetell/mcp/badges/score.svg)](https://glama.ai/mcp/connectors/com.fatetell/mcp)
[![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.fatetell%2Fmcp%26version%3Dlatest&query=%24.servers%5B0%5D.server.version&label=MCP%20Registry&prefix=v&color=4c1)](https://registry.modelcontextprotocol.io/v0/servers?search=com.fatetell/mcp&version=latest)

[English](README.md) · 简体中文

FateTell 负责算对,你的 AI 负责解读。

FateTell MCP 是托管的远程八字 MCP 服务(Streamable HTTP),不用下载,不用本地安装。在 Claude、ChatGPT、Cursor 等支持 MCP 的 AI 里填一个地址,就能排八字命盘、查黄历与节气;登录后还能读你在 FateTell 存的档案。服务端不调用任何大模型:FateTell 只返回按固定规则算出的数据,解读交给你自己的 AI。

```
https://mcp.fatetell.com/mcp
```

## 一览

| | |
|---|---|
| 形态 | 托管的远程 MCP 服务器,Streamable HTTP。不用在本地安装或运行任何程序。 |
| 地址 | `https://mcp.fatetell.com/mcp`(另有两个地址给登录方式不同的 AI,见[三个地址](#三个地址)) |
| 支持的 AI | Claude(网页、桌面、手机)、ChatGPT、Cursor、Claude Code、Codex、VS Code、Gemini CLI、Grok Bot、Manus、Hermes Agent,以及其他支持 Streamable HTTP 的 MCP 客户端 |
| 认证 | 排盘与历法不用登录;账户功能用 OAuth 2.1 登录。不需要 API Key。 |
| 工具 | 9 个,全部是确定性计算:2 个不登录可用,3 个登录可用,4 个会员可用([列表](#工具)) |
| 输入 | 出生日期(公历,或农历含闰月)、出生时间、IANA 时区、性别、南北半球;只有要求真太阳时才需要经度 |
| 输出 | 结构化数据:四柱、十神、藏干、纳音、大运、宫位、节气;每条结果都带规则版本号 |
| 口径 | 晚上 11 点以后出生,日柱按第二天算;年柱在立春换年,不在春节;默认不做真太阳时修正,要求时才做;南半球出生调整月柱。是否修正真太阳时、是否按南半球调整,结果里都会写明。 |
| 命盘卡 | 支持 MCP Apps 的 AI 会在对话里直接显示命盘卡;其他 AI 用 Skill 里附带的静态 HTML 模板 |
| 大模型 | 服务端不调用 |
| 费用 | FateTell 不为 MCP 的使用收费。会员功能需要 FateTell 会员,与 App、网页版的会员通用。 |
| 上限 | 每项功能有次数上限,只为防止滥用([次数上限](#次数上限)) |
| 文档 | [fatetell.com/zh-CN/mcp/docs](https://fatetell.com/zh-CN/mcp/docs)(八种语言)· [常见问题](https://fatetell.com/zh-CN/mcp/faq) |

这个仓库放的是这个服务的公开元数据:Claude 与 Claude Code 插件、Cursor 插件、给 Hermes Agent 的 Agent Plugins 包、Gemini CLI 扩展、MCP Registry 条目,以及教 AI 怎样读 FateTell 结果的 `fatetell-bazi` Skill。服务端代码不在这个仓库里。

## 三个地址

| 地址 | 行为 | 给谁用 |
|---|---|---|
| `https://mcp.fatetell.com/mcp` | 通用地址。连接不需要账户,排盘与历法不登录就能用;用到需要账户的功能时,AI 才打开 FateTell 的登录页。 | Claude、Claude Code、Codex、VS Code 等大多数 AI |
| `https://mcp.fatetell.com/login/mcp` | 连接即登录。添加时就要求登录,这个地址上的功能都在登录后使用。 | 只在添加时处理登录的 AI:Cursor、Grok Bot、Manus、Gemini CLI |
| `https://mcp.fatetell.com/chatgpt/mcp` | ChatGPT 专用地址。功能完全相同,登录方式按 ChatGPT 的约定。 | 只给 ChatGPT |

三个地址上的工具、入参与结果完全相同。登录用 OAuth 2.1,不需要提前配置任何密钥。

## 接入

每个 AI 的完整图文步骤见[连接你的 AI](https://fatetell.com/zh-CN/mcp/docs/connect)。下面是最短的做法。

**Claude(网页、桌面、手机)**:设置 → 连接器 → Add → 添加自定义连接器。名称填 `FateTell`,地址填通用地址;认证选「需要时登录」(Sign in when needed)。

```
https://mcp.fatetell.com/mcp
```

**ChatGPT**:Plugins → Add → Add custom MCP server。名称填 `FateTell`,认证保持 OAuth,地址填专用地址:

```
https://mcp.fatetell.com/chatgpt/mcp
```

**Claude Code**(连接器加 Skill):

```sh
claude plugin marketplace add RussGuo/fatetell-bazi-mcp
claude plugin install fatetell@fatetell
```

只要连接器:

```sh
claude mcp add --transport http fatetell https://mcp.fatetell.com/mcp
```

**Codex**:

```sh
codex mcp add fatetell --url https://mcp.fatetell.com/mcp
```

**Cursor**:在 `~/.cursor/mcp.json` 里加一项,Cursor 会在连接时请你登录 FateTell:

```json
{
  "mcpServers": {
    "fatetell": { "url": "https://mcp.fatetell.com/login/mcp" }
  }
}
```

**VS Code**:

```sh
code --add-mcp '{"name":"fatetell","url":"https://mcp.fatetell.com/mcp"}'
```

**Gemini CLI**(在终端里运行):

```sh
gemini extensions install https://github.com/RussGuo/fatetell-bazi-mcp
```

**其他 AI**:添加一个远程 MCP 服务器,传输方式选 Streamable HTTP(常写作 `http`),填通用地址,认证选 OAuth,不填密钥。如果连上后用到账户功能却不弹登录页,改用 `https://mcp.fatetell.com/login/mcp`。Grok Bot、Manus、Hermes Agent 的步骤见[英文 README](README.md#install)。

**只能启动本地(stdio)服务器的 AI**:可以通过第三方开源桥接工具 [`mcp-remote`](https://github.com/geelen/mcp-remote) 连到同一个地址:

```sh
npx -y mcp-remote https://mcp.fatetell.com/mcp
```

```json
{
  "mcpServers": {
    "fatetell": { "command": "npx", "args": ["-y", "mcp-remote", "https://mcp.fatetell.com/mcp"] }
  }
}
```

桥接工具只转发请求,所有计算仍在 FateTell 的服务器上完成。这种接法验证过不登录可用的功能;要用账户功能,建议换用能直接连接远程服务器的 AI。

能执行命令或改自己 MCP 设置的 AI,也可以让它自己连:

> 读一下 https://mcp.fatetell.com/install.md ,按里面的步骤帮我连接 FateTell MCP。

## 工具

9 个工具,全部是确定性计算:同样的输入永远得到同样的结果,每条结果都带规则版本号。

| 工具 | 功能 | 从哪一档开始 | 做什么 |
|---|---|---|---|
| `cast_bazi_chart` | FateTell 排盘 | 不登录 | 从出生时间排出完整命盘:四柱、十神、藏干、纳音、日主、全部大运、宫位与今日干支;南半球出生按跨地域规则排 |
| `get_almanac` | FateTell 历法 | 不登录 | 一次查 1 到 31 天的公历与农历日期、节气,以及年、月、日的干支 |
| `get_fatetell_account` | 账户 | 登录 | 当前连接的 FateTell 账户:会员状态、已授予这个 AI 的权限、各项功能今天的剩余次数 |
| `list_saved_charts` | 档案与命盘 | 登录 | 列出账户里保存的档案 |
| `get_saved_chart` | 档案与命盘 | 登录 | 读出一份档案的完整命盘,与 App 一致;不返回出生地 |
| `get_bazi_judgment` | FateTell 独有算法 | 会员 | 按 FateTell 的规则判断日主强弱(强、弱、从强、从弱)与喜忌五行 |
| `get_daily_fortune` | 个性化日运 | 会员 | 某份档案某一天的能量高低(高、中、低)、当天干支与命局的冲合、宜忌 |
| `get_chart_calendar` | 个性化历法 | 会员 | 以某份档案为视角,看未来每天(最多 31 天)或每月(最多 12 个月)的干支与命局的关系 |
| `create_saved_chart` | 档案管理 | 会员 | 在对话里把一份新档案存进你的账户 |

`create_saved_chart` 是唯一会写入的工具:它只新增档案,不能修改或删除已有的。其余工具都是只读。

## 登录与权限

三档权益层层包含:登录可用的,会员都有;不登录可用的,登录后都有。

- **登录在 FateTell 自己的页面完成。** OAuth 2.1 授权码加 PKCE;你的 AI 看不到你的密码。
- **会员身份每次使用时实时查询。** 会员与 FateTell App、网页版通用;你在 App 里能做的,在这里也能做。
- **随时可以断开。** FateTell 网页版 → 系统设置 → 已连接的应用 → 断开,一分钟内生效。

细节见[登录与权限](https://fatetell.com/zh-CN/mcp/docs/sign-in)。

## 次数上限

每项功能都有次数上限,只为防止滥用。它不收费,不是额度,也不与 App 内的次数相通;达到上限时,AI 只会告诉你上限是多少、什么时候恢复。

- 已登录:按账户计算,每项功能每天有各自的上限,另有一个每天的合计上限。
- 不登录:不能区分匿名用户的 AI 共用一个公共上限,按小时恢复;用满后登录,就按你自己的账户计算。
- 同样的问题 24 小时内重复问,不计次数。一次查 1 到 31 天的历法,算一次。

细节见[次数上限](https://fatetell.com/zh-CN/mcp/docs/limits)。

## 数据与隐私

- **服务端不调用任何大模型。** FateTell MCP 只做计算与数据读取,看不到你和 AI 的对话。
- **不登录时的使用不写入数据库。** 临时排的盘、查过的历法,不落库,也不与任何人关联。
- **日志不记录出生信息。** 不记出生日期、出生时间、经度、地名与档案名。
- **AI 只能读到你允许的内容。** 读出的档案不含出生地与坐标。
- **只有保存档案会收出生地名。** 用来确定时区与半球,与在 App 里建档一致;FateTell MCP 本身不保存、不记录它。

命理结果仅供参考,不能代替医疗、法律或财务方面的专业意见。你的 AI 怎样解读,是那个 AI 自己的判断。

- [数据与隐私](https://fatetell.com/zh-CN/mcp/docs/privacy)
- [隐私政策](https://fatetell.com/privacyPolicy#mcp) · [服务条款](https://fatetell.com/terms#mcp)

## 常见问题

**有没有不用本地安装的八字 MCP?**
有,FateTell MCP 就是。它是托管的远程服务:在 Claude、ChatGPT(用专用地址)、Cursor 等 AI 里把 `https://mcp.fatetell.com/mcp` 加为远程 MCP 服务器即可,你的电脑上不需要运行任何程序。

**它和用 `npx` 启动的本地八字 MCP 有什么不同?**
本地的 MCP 是装在你电脑上、和 AI 客户端一起运行的程序,需要 Node 或 Python 环境,也需要桌面客户端。FateTell MCP 运行在 FateTell 的服务器上,通过一个地址访问,所以网页版和手机上那些不能启动本地程序的 AI 也能用;登录后还能读你在 FateTell 存的档案,这是本地程序做不到的。计算引擎不开源;它遵循的口径写在每一条结果里。

**需要 FateTell 账户或 API Key 吗?**
排盘与历法不需要账户,任何时候都不需要 API Key。读已保存的档案需要登录 FateTell;FateTell 独有算法、个性化日运、个性化历法与档案管理需要 FateTell 会员。

**FateTell 会替我解读命盘吗?**
不会。FateTell 只返回算出的数据,解读由你的 AI 来写;这个仓库里的 `fatetell-bazi` Skill 告诉 AI 怎样读这些数据。

**为什么这里排的盘和别处不一样?**
常见原因是真太阳时(这里默认不修正,与 FateTell App 一致)、南半球出生的月柱调整,或没有给出生时间(按中午 12 点排,时柱只作参考)。详见[常见问题](https://fatetell.com/zh-CN/mcp/faq)。

## 支持

- 支持与安全问题:`contact@fatetell.com`。安全问题请在标题注明「Security」,不要发公开 issue。

## 许可

这个仓库里的文件按 [MIT License](LICENSE) 发布。许可只覆盖这个仓库;FateTell 的服务与计算引擎不开源。
