# Magic Bot Cursor Plugin — Design

## Goal

Ship a usable Cursor Plugin under `plugins/magic-bot/` that gives sellers a
Magic Bot / 魔偶 account-level MCP connection. The plugin can:

1. List the bots enabled in this deployment.
2. Install or activate a bot only when the seller explicitly asks for it.
3. Read the seller's wallet balance, reserved amount, and available amount.
4. Read a paginated usage ledger.

The package is the distribution source in this private repository. Publishing
to a public marketplace is out of scope for this change.

## Identity and layout

- Plugin name: `magic-bot`
- Display name: `Magic Bot`
- Public host: `https://bots.beansmile.ai`
- MCP endpoint: `https://bots.beansmile.ai/mcp`
- Layout:
  - `plugins/magic-bot/.cursor-plugin/plugin.json`
  - `plugins/magic-bot/mcp.json`
  - `plugins/magic-bot/skills/magic-bot-onboard/SKILL.md`
  - `plugins/magic-bot/skills/magic-bot-account/SKILL.md`
  - `plugins/magic-bot/README.md`
  - `plugins/magic-bot/LICENSE`
  - `plugins/magic-bot/assets/logo.png`

The logo is Jayce's official 2026-11-08 hex pick, copied byte-for-byte from
`brand-assets/magic-bot-logo/jayce-pick-2026-11-08/icon-hex.png`: a muted
sage-green hexagon with the bean robot on black. It is not a redesigned mark.

The package deliberately does not reuse the Shopify GEO plugin name, server
id, or branding.

## Authentication decision

Use an **account-level API secret** (`ApiSecret.application_id is None`) minted
in the Magic Bot console through `POST /secrets`. Cursor stores it through the
plugin's Configure flow as `BEANKIT_API_KEY`; `mcp.json` sends it as a Bearer
header. Existing secret verification in `SecretAuth` remains the only verifier.

Why this option:

- It follows the existing `bk_live_…` secret mechanism; there is no new auth
  path and no way to pass a password or console JWT through chat.
- It lets one configured account connection read account data and intentionally
  install any enabled bot, while still resolving a concrete Application only
  when a bot-specific tool runs.
- It avoids asking a seller to install a bot before they can even list the
  catalog or inspect their wallet.
- It keeps tenant isolation at the authenticated `user_id` from the secret.

Rejected alternatives:

- **Skills that ask for a console JWT or call REST with curl.** That would put
  long-lived console credentials in chat and duplicate authorization outside
  the MCP layer.
- **Reusing a Shopify GEO bot-bound key.** A bot-bound key must not gain
  account-wide list, install, wallet, or ledger access.
- **A new OAuth flow.** Out of scope and unnecessary for this private-plugin
  release.
- **Password-based setup.** Passwords must never be pasted into chat or stored
  by a plugin.

## Authorization model

New platform account scopes:

- `account:read` — list bots, list installations, read wallet, read ledger.
- `account:install` — create or activate an installation.

`PLATFORM_ACCOUNT_SCOPES` includes both scopes together with the existing
knowledge scopes. Account-level keys derive their live effective scopes from
this platform set plus the union of installed bot scopes, so a newly minted
account-level key works without manual scope editing. Bot-bound keys keep their
stored scopes unchanged; no account scope is silently added to them.

Every new account tool requires:

1. An authenticated principal with a `user_id`.
2. `bound_application_id is None` — the request must use an account-level key.
3. The tool-specific scope. `account_install_bot` requires
   `account:install`; the read tools require `account:read`.

A bot-bound key receives a clear denial explaining that these tools require an
account-level key. If an account principal somehow lacks the required effective
scope, the tool returns a clear missing-scope error telling the seller to mint a
new account-level key in the Magic Bot console. The implementation never
broadens scopes stored on bot-bound secrets and never bypasses `SecretAuth`.

## Tool contracts

All tools return JSON-serializable objects and redact credentials.

### `account_list_bots`

Returns enabled registry entries: `bot_type`, `display_name`, `version`,
`scopes`, and `enabled`. It reads the plugin registry and `BotRegistry`; it does
not create applications, secrets, ledger rows, or billing records.

### `account_list_installations`

Returns only Applications owned by the authenticated `user_id`: `id`, `slug`,
`name`, `bot_type`, `active`, `secret_configured`, and `created_at`. It never
returns secret values, hashes, verification tokens, or webhook secrets.

### `account_get_balance`

Returns `{balance, frozen, available}` from the authenticated user's wallet.

### `account_get_usage`

Returns ledger rows scoped to the authenticated `user_id`, aligned with
`GET /me/ledger`/`/me/usage`: `items`, `total`, `limit`, `offset`, and
`has_more`. `limit` defaults to 20 and is hard-capped at 100 (minimum 1);
`offset` must be non-negative. Optional `bot_type` and `application_slug`
filters are additive. The join to Application is only for displaying a slug and
the user filter always remains in place.

Ledger `note` values are free text. During MCP response shaping only, notes are
scrubbed with a small helper: case-insensitive public-upstream terms
(`kie.ai`, a standalone `kie`, `中转站`, `credits_consumed`) become
`[redacted]`, and payment identifiers written as `session=<id>`, `event=<id>`,
or `charge=<id>` keep their human-readable prefix but replace the identifier
with `[redacted]` (for example `session=[redacted]`). The database row is never
modified.

### `account_install_bot`

Takes `bot_type`, optional `slug`, and optional `name`. It validates that the
bot is configured and enabled. The two modes are deliberately different:

- **Default mode (`slug` omitted):** the tool locks the authenticated `User`
  row with `SELECT ... FOR UPDATE` inside the install transaction (plus an
  in-process per-user lock for SQLite), then re-queries the user's Applications
  for that `bot_type`. If one exists it returns the earliest row, preferring
  an active row, and reactivates it if needed. Only if none exists does it
  create one Application. This provides true per-user serialization and
  idempotency without a `UNIQUE(user_id, bot_type)` constraint.
- **Explicit-slug mode:** an unused valid slug is a multi-install roadmap path
  and creates another Application of the same `bot_type` for that user. If the
  slug already belongs to this user and bot type, that exact row is returned
  idempotently. A slug owned by someone else or by another bot type returns
  `slug_taken`; the code never falls back to an unrelated installation.

The tool does not freeze, settle, top up, charge, create secrets, or call
Stripe. There is no `force_new` option in v1.

This split makes all query tools read-only. Only the explicit install tool can
create an Application, and it does so without a billing side effect.

## Safety and isolation

- All queries filter by the authenticated user ID.
- Any second user's records are invisible, including through filters.
- Ledger pagination is enforced inside the MCP tool, independent of client
  input, with the same 1–100 bound as the REST route.
- Tool payloads contain no raw API secrets, password hashes, email-verification
  tokens, webhook secrets, or payment-provider secrets.
- Ledger notes are sanitized on output so forbidden public-upstream terms and
  Stripe-ish session/event/charge identifiers do not reach the plugin surface.
- The plugin exposes no admin routes, checkout tools, auto top-up, or Stripe
  operations.

## Manifest fields and official sources

The Cursor Plugin manifest is `.cursor-plugin/plugin.json`; `name` is the only
required field. The documented optional fields are `description`, `version`,
`author` (`name` required, `email` optional), `homepage`, `repository`,
`license`, `keywords`, `logo`, `mcpServers`, and `variables`, per
<https://cursor.com/docs/reference/plugins>. The `variables` value is a
top-level JSON Schema object (`{ "type": "object", "properties": { ... } }`)
and every `${VAR}` placeholder in `mcp.json` is declared there. `mcpServers`
may be a path, object, or array, and root `mcp.json` is the default discovery
location.

The manifest deliberately does not include `category` (documented for
marketplace entries, not the Cursor Plugin manifest table) or `author.url`
(documentation lists only `author.name` and `author.email`). It retains
`displayName`, which appears in Cursor's `cursor/plugin-template` even though
it is not in the prose optional-field table, for template compatibility. The
`logo` value is `assets/logo.png`, a relative path to an existing file.

The MCP server uses the HTTP `url` + `headers` shape shown in Cursor Plugin
examples. The separate Agent Plugins MCP schema at
<https://agent-plugins.org/schemas/1.0.0/mcp.schema.json> describes stdio
servers; this HTTP transport is not that stdio schema.

## Plugin variables and seller guidance

`plugin.json` declares an optional `BEANKIT_API_KEY` string variable, so
installation can finish before Configure. `mcp.json` references
`${BEANKIT_API_KEY}`. The README and onboard skill give exact steps: sign in at
<https://bots.beansmile.ai/console>, open **API keys**, leave access on
**All installed bots (account-level)** / **账户密钥（全部已安装 bot）**, click
**Generate API key** / **生成 API Key**, copy once, then paste into
`BEANKIT_API_KEY` through Cursor → Plugins → Configure. Skills explicitly say
never to ask for a password, console JWT, or API key in plain chat.

The onboard skill also states that installing the plugin does not open paid
bots or create charges by itself. The account skill requires an explicit
install / 开通 request before calling `account_install_bot`, and requires plain
language summaries rather than raw scope or JSON dumps.

## Verification plan

- Targeted pytest coverage for account tools, bot-bound denial, default
  idempotency/reactivation, explicit-slug multi-install, concurrent default
  installs, read-only behavior, ledger pagination/isolation, note and credential
  redaction, and missing-scope denial.
- Package validation for the manifest, MCP config, logo file, and skill YAML
  frontmatter, plus a scan for accidental private-key material in the plugin
  tree.
