# Magic Bot Cursor Plugin

Magic Bot / 魔偶 connects Cursor to your Magic Bot account through the hosted MCP
endpoint at `https://bots.beansmile.ai/mcp`.

This repository is the **public** Cursor Plugin package for Magic Bot / 魔偶.
Install from the Cursor Marketplace once listed, or load this repo as a local / GitHub plugin.
Submit or refresh the listing at https://cursor.com/marketplace/publish

## What the plugin can do

- List the bots available in this Magic Bot deployment.
- List the bots already installed on your account.
- Read your wallet balance, reserved amount, and available amount.
- Read your paginated usage ledger.
- Install or activate a bot only when you explicitly ask for it.

The plugin does not create API keys, top up a wallet, start checkout, or expose
admin tools. Installing the plugin does not open a paid bot, create an
Application, or charge your account by itself.

## Requirements

- A Magic Bot account. Create one at
  <https://bots.beansmile.ai/register>.
- An **account-level** API key from the Magic Bot console at
  <https://bots.beansmile.ai/console>.
- Cursor with Cursor Plugin support.

A bot-bound key cannot use account-wide tools such as the bot catalog, wallet,
ledger, or install. Create an account-level key instead.

## Create the account-level API key

1. Sign in at <https://bots.beansmile.ai/console>.
2. Open the **API keys** section.
3. Leave key access on **All installed bots (account-level)**.
4. Click **Generate API key** and copy the key once while it is shown.
5. Open **Cursor → Plugins → Configure** and paste the key into
   `BEANKIT_API_KEY`.

Never paste the key into chat.

## Local load

For local development, load the `plugins/magic-bot/` directory as a local Cursor
Plugin. The directory containing `.cursor-plugin/plugin.json` is the plugin
root. Cursor discovers `mcp.json` and the skills from that root.

After loading, open **Cursor → Plugins → Configure** and set
`BEANKIT_API_KEY` to your account-level API key. The key is optional at install
time so loading the package can succeed before Configure; the tools remain
unavailable until a valid key is configured.

Never paste an API key, password, or console token into chat. Enter the key only
in Cursor's Configure form. If a secret-request UI appears, use it only for the
Configure value; never invent or ask Cursor to create credentials.

## Example prompts

- “Which Magic Bot bots are available?”
- “What is my Magic Bot balance?”
- “Show my recent Magic Bot usage.”
- “Install the BrandLab Translation bot on my Magic Bot account.” (Only this
  explicit kind of request should create an installation.)
- “Install another BrandLab Translation bot with slug brandlab-secondary.”
  (An explicit unused slug creates a separate installation.)

## Package layout

```text
plugins/magic-bot/
  .cursor-plugin/plugin.json
  mcp.json
  README.md
  LICENSE
  assets/logo.png
  skills/
    magic-bot-onboard/SKILL.md
    magic-bot-account/SKILL.md
```

The server id is `magic-bot` and the MCP URL is
`https://bots.beansmile.ai/mcp`. The Authorization header is populated from
`${BEANKIT_API_KEY}`.

The plugin mark is Jayce's official 2026-11-08 hex pick: a muted sage-green
hexagon with the bean robot on black. The binary source is copied from
`brand-assets/magic-bot-logo/jayce-pick-2026-11-08/icon-hex.png`; the plugin
uses it directly as `assets/logo.png`.

---

# Magic Bot Cursor 插件（中文）

Magic Bot / 魔偶通过 `https://bots.beansmile.ai/mcp` 把 Cursor 连接到你的
Magic Bot 账户。

本目录是私有 `beankit` 仓库中的分发源，是一个完整的单插件包，不是公开市场
上架包。市场上架是单独的发布步骤，本次不包含。

## 插件能做什么

- 查看当前部署中可用的 bot。
- 查看账户已经开通的 bot。
- 查询钱包余额、冻结额度和可用额度。
- 查看分页的使用记录。
- 只有在你明确要求开通某个 bot 时，才开通或激活该 bot。

插件不会创建 API 密钥、不会自动充值、不会发起支付，也不会暴露管理员工具。
仅安装插件不会自动开通付费 bot、创建 Application 或产生扣费。

## 使用前准备

- 一个 Magic Bot 账户：<https://bots.beansmile.ai/register>。
- 一个在 Magic Bot 控制台创建的**账户级** API 密钥：
  <https://bots.beansmile.ai/console>。
- 支持 Cursor Plugin 的 Cursor 客户端。

bot 级密钥不能使用账户级的 bot 列表、钱包、使用记录或开通工具。请创建账户级
密钥。

## 创建账户级 API 密钥

1. 登录 <https://bots.beansmile.ai/console>。
2. 打开 **API keys / API 密钥** 区域。
3. 将密钥访问范围保留为 **账户密钥（全部已安装 bot）**。
4. 点击 **生成 API Key / Generate API key**，并在显示时立即复制一次。
5. 打开 **Cursor → Plugins → Configure**，把密钥粘贴到
   `BEANKIT_API_KEY`。

不要把密钥粘贴到聊天里。

## 本地加载

本地开发时，把 `plugins/magic-bot/` 目录作为本地 Cursor 插件加载。包含
`.cursor-plugin/plugin.json` 的目录就是插件根目录，Cursor 会从该目录发现
`mcp.json` 和技能。

加载后打开 **Cursor → Plugins → Configure**，把 `BEANKIT_API_KEY` 设置为你的
账户级 API 密钥。安装时该变量可以是空的，因此加载插件本身可以成功；完成
Configure 之前工具不可用。

不要把 API 密钥、密码或控制台令牌粘贴到聊天里。只通过 Cursor 的 Configure
表单输入密钥。如果出现密钥输入界面，只用于填写 Configure 的值，不要让助手
替你创建或猜测凭据。

## 示例指令

- “现在有哪些 Magic Bot bot 可用？”
- “查一下我的 Magic Bot 余额。”
- “看看我最近的使用记录。”
- “在我的 Magic Bot 账户上开通 BrandLab Translation bot。”（只有这类明确
  请求才应创建安装。）
- “再开通一个 BrandLab Translation bot，slug 用 brandlab-secondary。”（显式
  传入未使用的新 slug 会创建独立安装。）

## 安全边界

- 所有查询只读，不会创建 Application、密钥或账本记录。
- 开通工具只在明确请求时调用。默认不带 slug 时，同一账户、同一 bot 会复用
  已有安装并在需要时重新激活，不会重复创建；显式传入新的 slug 时，可以再创建
  同一 bot 的另一个安装。
- 不自动充值，不发起支付。
- 插件和工具不会返回原始密钥、密码哈希、验证令牌或支付密钥。


## One-click MCP (while Marketplace review is pending)

1. Create an account and an **account-level** API key at https://bots.beansmile.ai/console
2. Export the key in your shell: `export BEANKIT_API_KEY=bk_live_…`
3. Open this install link (opens Cursor):
   https://cursor.com/install-mcp?name=magic-bot&config=eyJ1cmwiOiJodHRwczovL2JvdHMuYmVhbnNtaWxlLmFpL21jcCIsImhlYWRlcnMiOnsiQXV0aG9yaXphdGlvbiI6IkJlYXJlciAke2VudjpCRUFOS0lUX0FQSV9LRVl9In19

Prefer the full plugin (skills + MCP) from this GitHub repository via Cursor → Plugins once Marketplace lists it, or Add from GitHub.
