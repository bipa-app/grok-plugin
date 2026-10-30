# Bipa

Pay and check your money at Bipa from Grok Bot, Grok Build and Cursor. Send Pix to a key, a saved recipient, a Bipa tag or a QR code, pay boletos and DDA bills, and see your balance in reais, Bitcoin and USDT, your transaction history, your portfolio results and Bipa's prices.

## Try asking

- "What's my Bipa balance?"
- "Send R$ 35 to maria@email.com for lunch"
- "Pay this boleto: 23793.38128 ..."
- "Did my Pix to João go through?"
- "How did my Bitcoin portfolio do this month?"

## You stay in control

- **You choose the permissions.** When you connect, you sign in with Bipa (OAuth 2.1) and pick what the agent can do: read financial data (`financial:read`), read account details and keys (`account:read`), request payments (`payments:request`) and pay trusted contacts (`payments:execute_trusted`).
- **Payment requests wait for you.** Pix and boleto requests stay pending until you approve them with biometrics in the Bipa app.
- **Trusted contacts are the one exception.** If you granted `payments:execute_trusted`, the agent can pay contacts you have already marked as trusted in the app, within the limits you set there, without a new approval. The agent can't create trusted contacts, Pix keys or change limits.
- **No trading.** The plugin can't buy or sell crypto.

Reviewers can use the installed plugin's normal OAuth connection with the private fictional-account credentials after the reviewer gateway is deployed. Follow [PUBLISHING.md](PUBLISHING.md); keep `/mcp` and OAuth unchanged. The [public Grok demo](https://mcp.bipa.app/demo/grok) remains a separate development sandbox. OpenAI and Claude use the read-only connection.

## What's inside

- **A connection to Bipa's MCP server**, `https://mcp.bipa.app/mcp`, over streamable HTTP. It is declared in `.mcp.json` and mirrored in `mcp.json` for Cursor-style loaders.
- **The `bipa` skill**, in Portuguese, which tells the agent which Bipa tool answers each request, how to confirm payments before sending them and how to follow a payment until it settles.
- **Two manifests with the same metadata:** `.grok-plugin/plugin.json` for Grok Build and `.cursor-plugin/plugin.json` for Grok Bot and Cursor.

The plugin runs no local code, hooks or scripts. Its only network endpoint is `https://mcp.bipa.app/mcp`, and the only credential is the OAuth token your app gets when you sign in with Bipa.

## Privacy and support

- Privacy policy: <https://suporte.bipa.app/hc/pt-br/articles/36305196684315>
- Terms of use: <https://suporte.bipa.app/hc/pt-br/articles/55072595716507-Termos-de-uso-aplicativo-Bipa>
- Help center: <https://suporte.bipa.app/hc/pt-br>
- Setup guide: <https://agents.bipa.app/docs/grok>
- Security model: <https://agents.bipa.app/docs/seguranca>

---

## Em português

Pague e consulte o seu dinheiro na Bipa pelo Grok Bot, Grok Build e Cursor. Mande Pix para uma chave, um favorecido, uma tag Bipa ou um QR Code, pague boletos e contas do DDA e veja seu saldo em reais, Bitcoin e USDT, o extrato, o resultado da sua carteira e as cotações da Bipa.

Experimente pedir:

- "Qual é o meu saldo na Bipa?"
- "Manda R$ 35 para maria@email.com pelo almoço"
- "Paga esse boleto: 23793.38128 ..."
- "O Pix pro João caiu?"

Ao conectar, você entra com a sua conta Bipa e escolhe o que o agente pode fazer: consultar dados financeiros, consultar dados da conta e chaves, pedir pagamentos e pagar contatos de confiança. Pedidos de Pix e boleto ficam pendentes até você aprovar com biometria no app da Bipa. A exceção são os contatos de confiança que você já cadastrou no app: com essa permissão, o agente paga sem nova aprovação, dentro dos limites que você definiu. O agente não compra nem vende cripto, não cria contatos de confiança nem chaves Pix e não altera limites.

O plugin conecta apenas ao servidor da Bipa (`https://mcp.bipa.app/mcp`) e não executa código local, hooks nem scripts.

Para revisão, após a implantação do fluxo, use as credenciais privadas da conta fictícia na janela OAuth do plugin instalado. Mantenha `/mcp` e a autenticação. Veja as [instruções de revisão](PUBLISHING.md). A [demonstração pública](https://mcp.bipa.app/demo/grok) continua disponível para desenvolvimento. OpenAI e Claude permanecem somente leitura.

## License

MIT. See [LICENSE](LICENSE).
