# Publishing the Bipa plugin for Grok

Two routes exist, and they lead to different listings.

## 1. Grok Bot and Cursor marketplace (web form)

Submit at [x.ai/bot/marketplace/submit-a-plugin](https://x.ai/bot/marketplace/submit-a-plugin). Submitting means agreeing to the publisher terms at cursor.com/marketplace-publisher-terms. Test the plugin locally and in-app before ticking the confirmation box.

| Field | Value |
| --- | --- |
| Organization name | Bipa |
| Organization handle | `bipa` |
| Website URL | https://bipa.app |
| Logotype URL | https://raw.githubusercontent.com/bipa-app/grok-plugin/main/assets/bipa-logo.png (1:1 PNG, 1024×1024, black background plate) |
| Listing destination | Grok Bot and Cursor (and Grok, once it opens) |
| MCP server URL or public GitHub repository | https://github.com/bipa-app/grok-plugin |
| Brief description | Pay by Pix, pay boletos and check your Bipa balances, transaction history, Bitcoin and USDT portfolio and prices, right in the chat. Payment requests wait for your approval in the Bipa app. |
| Testing instructions | See below |

Suggested testing instructions (Grok payment demo):

1. Install the plugin from https://github.com/bipa-app/grok-plugin. Open https://mcp.bipa.app/demo/grok in your browser and click **Create demo session**. No credentials, Bipa account, OAuth consent or mobile app are required for this fictional account. Keep this browser session open.
2. Copy the session-specific MCP URL displayed on the page. In your review installation, replace the Bipa URL in `.mcp.json` and `mcp.json` with this URL, choose **no authentication**, and reconnect. This is a reviewer override: the published plugin continues to connect to `https://mcp.bipa.app/mcp` with OAuth 2.1 and payment permissions. If the review client cannot override the installed connection, add the session URL as a separate custom MCP connector and disable the production Bipa connection during the demo.
3. Ask **"What's my Bipa balance?"** — `bipa_balance` returns fictional BRL, BTC and USDT balances. Ask **"Show my last 5 transactions"** — `bipa_transactions` returns five fictional transactions.
4. Ask **"Send R$ 1 to demo@example.invalid as a simulated test"**. The agent shows the fictional recipient and amount and asks for your confirmation before calling `bipa_pix_pay_key` with `amount_cents: 100`. The response includes an `outflow_request_id`, a reviewer approval URL, and `status: awaiting_approval`.
5. Before approving, ask **"Did that Pix go through?"** — `bipa_payment_status`, with `pix_request_id` set to that returned ID, must show `approval_state: pending` and `execution_state: not_started`. Repeated status checks do not approve it.
6. Open the returned approval URL in the **same browser that created the session**, refresh the requests, and click **Approve simulated Pix**. The agent cannot approve through MCP. Ask about the status again: `bipa_payment_status` now shows `approved` / `succeeded`, explicitly a simulation. The fictional available BRL balance decreases by R$ 1 and the simulated Pix appears in the transaction history. No real money moves.
7. For the rejection case, request another simulated Pix and click **Reject simulated Pix** in the browser. Status becomes `rejected` / `not_started`; the fictional balance stays unchanged. Repeating an identical request with the same UUIDv7 idempotency key must return the same request, without another debit.

Only the fictional key `demo@example.invalid` is accepted, with amounts from R$ 0.01 to R$ 100. Sessions are isolated, expire after one hour, and reset on server restart; create a new session and reconnect if necessary. This demo covers Pix initiation, human approval/rejection, status and history. It does not simulate trusted-contact execution, boletos, scheduling, real OAuth, mobile biometrics or bank settlement.

Reviewer account fields: **Login URL:** `https://mcp.bipa.app/demo/grok`; **Tenant / username / password:** not required — select no authentication on the separate demo connection. Use the steps above as the access instructions. Do not invent credentials or describe this as an OAuth test account.

The OpenAI/Claude demo remains read-only at https://mcp.bipa.app/demo (MCP: `https://mcp.bipa.app/demo/mcp`). Do not use it for the Grok payment test. Keep these review instructions separate from OpenAI/Claude submissions.

## 2. Grok Build marketplace (pull request)

The catalog lives in [xai-org/plugin-marketplace](https://github.com/xai-org/plugin-marketplace). A PR adds one entry to `.grok-plugin/marketplace.json` that points at this repository, pinned to a commit.

1. Get the SHA of the release commit on `main`:

   ```
   git ls-remote https://github.com/bipa-app/grok-plugin.git HEAD
   ```

2. Fork `xai-org/plugin-marketplace`, branch from `main`, and append this entry to the `plugins` array in `.grok-plugin/marketplace.json`, with the full 40-character SHA from step 1:

   ```json
   {
     "name": "bipa",
     "description": "Pay by Pix, pay boletos and check your Bipa balances, transaction history, Bitcoin and USDT portfolio and prices, right in the chat. Payment requests wait for your approval in the Bipa app.",
     "category": "productivity",
     "source": {
       "source": "url",
       "url": "https://github.com/bipa-app/grok-plugin.git",
       "sha": "0000000000000000000000000000000000000000"
     },
     "homepage": "https://agents.bipa.app/docs/grok",
     "keywords": ["bipa", "bipa pix", "bipa app", "bipa boleto", "bipa bitcoin"],
     "domains": ["bipa.app", "agents.bipa.app", "mcp.bipa.app"]
   }
   ```

3. Regenerate the component index and validate. CI runs exactly these:

   ```
   python3 scripts/generate-plugin-index.py
   python3 scripts/validate-catalog.py
   python3 scripts/generate-plugin-index.py --check
   ```

4. Open the PR, fill in the template, and wait for CI and code-owner review.

To ship an update later, push to `main` here, then open a PR that bumps the `sha` and regenerates the index. Do not open a second entry.
