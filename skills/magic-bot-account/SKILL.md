---
name: magic-bot-account
description: Use Magic Bot account tools to list bots and installations, read wallet and usage, and install a bot only on an explicit seller request.
---

# Magic Bot account tools

## When to use each tool

- `account_list_bots` — use when the seller asks what bots are available or
  which bot fits a task. This is a read-only catalog call.
- `account_list_installations` — use when the seller asks what is already
  installed or active on their account. This is read-only.
- `account_get_balance` — use when the seller asks about wallet balance,
  reserved amount, or available amount. This is read-only.
- `account_get_usage` — use when the seller asks for recent or paginated usage
  history. This is read-only. The default page is 20 rows; request no more than
  100 rows at a time, and use `offset` for later pages.
- `account_install_bot` — call **only** after the seller explicitly asks to
  install, activate, or 开通 a specific bot. Listing a bot or asking whether it
  is available is not an install request.

## Install rules

- If the seller is ambiguous, ask one short confirmation naming the bot before
  installing.
- **Default install (omit `slug`):** this mode is idempotent for the same
  account and bot type. If an installation already exists, report that it is
  already present and reuse that earliest row, reactivating it if needed. Never
  create a duplicate in this mode.
- **Multi-install (explicit `slug`):** only use this when the seller explicitly
  asks for another installation and supplies an unused slug. A second
  Application of the same bot type is allowed. If that exact slug already
  belongs to this seller's same bot type, reuse it idempotently. If the slug is
  taken by anyone else or by another bot type, report that the slug is taken
  and ask for another.
- Never claim an install request charges the wallet. The tool creates or
  reactivates the installation without a payment or top-up.
- If the bot type is unknown or disabled, call `account_list_bots` again and
  offer the available choices.

## Read-only guarantees

`account_list_bots`, `account_list_installations`, `account_get_balance`, and
`account_get_usage` never create Applications, API keys, charges, top-ups, or
ledger movements. Do not describe them as doing so.

## Talking to the seller

- Summarize results in plain language. Do not dump raw JSON, internal IDs, or
  the full scope list unless the seller specifically needs a detail.
- Never reveal or repeat the API key, password hashes, verification tokens,
  webhook secrets, or payment secrets.
- Use the seller's language. In Chinese, keep the product name **魔偶** and say **bot** consistently.
- If a tool says an account-level key is required, ask the seller to create an
  account-level key in the Magic Bot console and update Plugins → Configure.
- If a tool says a scope is missing, ask the seller to create a new
  account-level key so it receives the current platform permissions; do not ask
  them to paste secrets into chat.
- Do not create API keys, top up the wallet, start checkout, or call payment
  tools. Those actions are outside this plugin.

## Useful response patterns

- Balance: “Your wallet has X available, with Y reserved out of a Z balance.”
- Installations: “You currently have these bots installed: …”
- Usage: “Here are the latest entries. There are more; would you like the next
  page?”
- Existing installation: “That bot is already active on your account. You can
  use it now.”

---

# 中文说明

## 各工具的用途

- `account_list_bots` — 卖家询问有哪些 bot 可用或该选哪个 bot 时使用。只读。
- `account_list_installations` — 卖家询问账户已开通或激活了哪些 bot 时使用。
  只读。
- `account_get_balance` — 卖家询问钱包余额、冻结额度或可用额度时使用。只读。
- `account_get_usage` — 卖家询问最近或分页的使用记录时使用。只读。默认每页
  20 条，最多 100 条；后续页面用 `offset`。
- `account_install_bot` — 只有卖家明确要求安装、激活或开通某个具体 bot 时才
  调用。只是查看或询问某个 bot 是否可用，不算开通请求。

## 开通规则

- 如果卖家表达不明确，先用一句话确认要开通的 bot。
- **默认开通（不传 `slug`）：** 同一账户、同一 bot 的默认开通是幂等的。
  如果已有安装，说明它已存在并复用最早的一条记录，必要时重新激活；此模式
  绝不重复创建。
- **多安装（显式传 `slug`）：** 仅在卖家明确要求再安装一个，并提供未使用
  slug 时使用。允许为同一 bot 创建第二个 Application。如果该 slug 已属于
  当前卖家的同一 bot，则幂等复用；如果 slug 被其他用户或另一种 bot 占用，
  说明 slug 已被占用并询问新的 slug。
- 不要声称开通请求会扣钱包费用。该工具只创建或重新激活安装，不发起支付或
  充值。
- 如果 bot 不存在或已停用，再次调用 `account_list_bots`，并给出可用选项。

## 只读保证

`account_list_bots`、`account_list_installations`、`account_get_balance` 和
`account_get_usage` 不会创建 Application、API 密钥、扣费、充值或账本记录。
不要把它们描述成会执行这些操作。

## 和卖家沟通

- 用简单语言总结结果。不要直接倾倒原始 JSON、内部 ID 或完整权限列表，除非
  卖家确实需要某个细节。
- 绝不泄露或重复 API 密钥、密码哈希、验证令牌、webhook 密钥或支付密钥。
- 使用卖家的语言。中文里保持产品名**魔偶**，统一说 **bot**。
- 如果工具提示需要账户级密钥，请卖家在 Magic Bot 控制台创建账户级密钥，并
  更新 Plugins → Configure。
- 如果工具提示缺少权限，请卖家新建一个账户级密钥，让新密钥获得当前的平台
  权限；不要让卖家把密钥粘贴到聊天里。
- 不要创建 API 密钥、不要充值、不要发起支付或调用支付工具。这些操作不属于
  本插件。

## 常用回复示例

- 余额：“你的钱包目前可用 X，冻结 Y，总余额 Z。”
- 已开通：“你的账户目前开通了这些 bot：……”
- 使用记录：“这是最近的记录。后面还有更多，需要我继续查看下一页吗？”
- 已经开通：“这个 bot 已经在你的账户上激活，可以直接使用。”
