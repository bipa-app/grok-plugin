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

Suggested testing instructions:

> Install the plugin and sign in with the Bipa reviewer account on the first tool call (OAuth 2.1; grant all four permissions). Ask "What's my Bipa balance?": the agent calls `bipa_balance`. Ask "Show my last 5 transactions": it calls `bipa_transactions`. Ask "Send R$ 1 to <reviewer Pix key> as a test": the agent shows the recipient and amount, asks for confirmation, then calls `bipa_pix_pay_key`. The request returns `awaiting_approval` and stays pending until it is approved in the Bipa app; no money moves without that approval. Ask "Did that Pix go through?": it calls `bipa_payment_status`. A read-only demo with fictional data, no sign-in needed, is at https://mcp.bipa.app/demo.

Reviewer credentials go only in the form, never in this repository.

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
