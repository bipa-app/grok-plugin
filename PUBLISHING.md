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
| Brief description | Pay by Pix, pay boletos and check your Bipa balances, transaction history, Bitcoin and USDT portfolio and prices, right in the chat. Payment requests wait for your approval in the Bipa app; preauthorized trusted-contact Pix is the exception. |
| Testing instructions | See below |

Suggested testing instructions (installed plugin with fictional reviewer OAuth):

Deployment gate: use these instructions only after the companion CLI change is deployed and a fresh login is verified in the target Grok/Grok Bot client. Do not attest that in-app testing passed based only on local tests.

1. Install the plugin from https://github.com/bipa-app/grok-plugin. Keep `https://mcp.bipa.app/mcp` and OAuth enabled. Connect Bipa; under **Reviewer demo account**, enter the private reviewer username and password supplied with the submission, then select **Connect fictional account**. Do not choose **Continue with my Bipa account**. No mobile app, SMS, email verification, real account or extra setup is required. Keep the browser used for OAuth available for simulated payment approvals.
2. Ask **"What's my Bipa balance?"** — `bipa_balance` shows fictional BRL, BTC and USDT balances (R$ 2,450.25 available, R$ 5,000 savings, 0.015 BTC and 250 USDT). Ask **"Show my last 5 transactions"** — `bipa_transactions` returns fictional history.
3. Ask **"Send R$ 1 to demo@example.invalid as a simulated test"**. The agent shows the fictional recipient and amount and asks for confirmation before calling `bipa_pix_pay_key` with `amount_cents: 100`. The request returns `awaiting_approval`, an `outflow_request_id` and a reviewer approval URL.
4. Before approving, ask **"Did that Pix go through?"**. `bipa_payment_status` with the returned `pix_request_id` shows `pending` / `not_started`. Polling cannot approve it.
5. Open the approval URL in the **same browser used for OAuth**, refresh the requests and click **Approve simulated Pix**. The agent cannot approve through MCP. Status becomes `approved` / `succeeded`, explicitly a simulation; fictional available BRL falls by R$ 1 and history gains the payment. No real money moves.
6. Request another simulated Pix and click **Reject simulated Pix**. Status becomes `rejected` / `not_started`; balance stays unchanged. Retry an unchanged request with the same UUIDv7 idempotency key: it must return the same request without another debit.
7. Ask **"Preview this fictional boleto: 00000000000000000000000000000000000000000000000"**. `bipa_bank_slip_preview` shows a fixed R$ 1 fictional bill. Confirm payment in chat; `bipa_pay_bank_slip` must omit `amount_cents` and return `awaiting_user_approval` with an `approval_id`. Pass that ID as `operation_approval_id` to `bipa_payment_status`, approve or reject in the browser, then check again. No settlement occurs.
8. Ask to list the fictional trusted contacts. Contact ID 1 models an existing authorization up to R$ 100. After explicitly confirming a simulated R$ 1 payment, `bipa_pix_pay_trusted_contact` can return `sent` without a new browser approval. This is the preauthorized-contact exception; it is still fictional and cannot reach real funds.

Reviewer account fields: **Login URL:** `https://mcp.bipa.app/oauth`; **Tenant:** not applicable; **Username/password:** the private credentials supplied through the submission's secure access field. The login URL is informational: initiate OAuth from the installed plugin. Never put credentials in the public repository, plugin skill or chat.

The connection requests `financial:read`, `account:read`, `payments:request` and `payments:execute_trusted`. These credentials select only an isolated fictional backend. The published plugin and real customers retain the normal payment-capable `/mcp` connection. Normal payment requests require app approval; previously authorized trusted-contact payments are the explicit exception.

Reviewer access tokens last one hour and refresh automatically for up to 24 hours. Refresh preserves simulated state. Reconnect with the same credentials after expiry or a server restart to start a new session. Only the fictional Pix key `demo@example.invalid`, trusted contact ID 1 and the 47-zero boleto are accepted; Pix amounts are R$ 0.01–100. The demo does not cover scheduling, other payment entry methods, mobile biometrics, real settlement or live prices.

The separate public https://mcp.bipa.app/demo/grok remains a development sandbox. Do not replace the installed MCP URL or switch off OAuth for review: that would skip the sign-in path under review. OpenAI and Claude remain read-only at `/readonly/mcp`, with their separate `/demo/mcp` sandbox.

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
     "description": "Pay by Pix, pay boletos and check your Bipa balances, transaction history, Bitcoin and USDT portfolio and prices, right in the chat. Payment requests wait for your approval in the Bipa app; preauthorized trusted-contact Pix is the exception.",
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
