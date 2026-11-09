---
name: magic-bot-onboard
description: Set up a seller's Magic Bot Cursor Plugin with an account-level key through Plugins -> Configure, without requesting credentials in chat.
---

# Connect a seller to Magic Bot

## What to do

If the seller does not have an account, point them to
`https://bots.beansmile.ai/register` first.

1. Sign in at <https://bots.beansmile.ai/console>.
2. Open the **API keys** section.
3. Leave key access on **All installed bots (account-level)**.
4. Click **Generate API key** and copy the key once while it is shown.
5. Open **Cursor → Plugins → Configure** and paste the key into
   `BEANKIT_API_KEY`.

After Configure is complete, you may call `account_list_bots` and
`account_list_installations` to show what is available and what is already
installed. Summarize the result in plain language.

## Credential safety

- Never ask the seller to paste an API key, password, console token, or any
  other secret into chat.
- Never ask for a password or console token at all.
- Enter or ask for the key only in Cursor's Plugins → Configure flow. If a
  secret-request UI exists, use it only for Configure values; never invent
  credentials.
- Do not print headers, configuration file paths, environment variable values,
  or raw secrets.
- An account-level key is required for account-wide tools. A bot-bound key
  should be replaced with an account-level key in the console, not extended or
  edited by you.

## Boundaries

- Loading the plugin does not install any paid bot, create an Application, or
  charge the seller. Say this plainly if the seller asks.
- Do not call `account_install_bot` during onboarding unless the seller has
  already explicitly asked to install a named bot.
- Do not create API keys, top up the wallet, start checkout, or call any payment
  operation.
- Keep the product name **Magic Bot** and, in Chinese, **魔偶**. In Chinese, always say
  **bot**; do not use other Chinese terms for the assistant.
- If a tool fails, translate it into plain language. Missing or invalid
  credentials mean: “Please create an account-level key in the Magic Bot
  console and enter it in Plugins → Configure.”

## Plain-language greeting after setup

English: “You're connected. Tell me which bot you want to use, or ask me to
check your balance or usage.”

中文：“已经连接好了。告诉我想用哪个 bot，或者让我查余额和使用记录。”

Do not narrate tool names or protocol details while greeting the seller.

---

# 中文说明

## 设置步骤

如果卖家还没有账户，请先让对方打开
`https://bots.beansmile.ai/register`。

1. 登录 <https://bots.beansmile.ai/console>。
2. 打开 **API keys / API 密钥** 区域。
3. 将密钥访问范围保留为 **账户密钥（全部已安装 bot）**。
4. 点击 **生成 API Key / Generate API key**，并在显示时立即复制一次。
5. 打开 **Cursor → Plugins → Configure**，把密钥粘贴到
   `BEANKIT_API_KEY`。

完成 Configure 后，可以调用 `account_list_bots` 和
`account_list_installations`，再用简单语言说明有哪些 bot 可用、哪些已经
开通。

## 凭据安全

- 绝不要让卖家把 API 密钥、密码、控制台令牌或其他密钥粘贴到聊天里。
- 绝不索要密码或控制台令牌。
- 只在 Cursor 的 Plugins → Configure 流程中输入或请求密钥。如果出现密钥
  输入界面，只用于填写 Configure 的值，不要替卖家发明凭据。
- 不要输出请求头、配置文件路径、环境变量值或原始密钥。
- 账户级工具需要账户级密钥。bot 级密钥应替换为在控制台新建的账户级密钥；
  不要手动扩展或修改它的权限。

## 边界

- 仅加载插件不会开通任何付费 bot、不会创建 Application，也不会产生扣费。
  如果卖家问起，要明确说明。
- 除非卖家已经明确要求开通某个指定的 bot，否则不要在引导过程中调用
  `account_install_bot`。
- 不要创建 API 密钥、不要充值、不要发起支付或调用任何支付操作。
- 保持产品名 **Magic Bot**，中文使用**魔偶**。中文里统一说 **bot**，不要换用其他称呼。
- 工具失败时用简单语言解释。凭据缺失或无效时说明：“请在 Magic Bot 控制台
  创建一个账户级密钥，并在 Plugins → Configure 里填写。”
