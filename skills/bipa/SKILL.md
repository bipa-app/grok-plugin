---
name: bipa
description: Pague por Pix e boleto e consulte saldos, extrato, carteira, limites, chaves e preços da Bipa pelo conector MCP da Bipa. Use quando a pessoa quiser mandar um Pix, pagar um QR Code, Copia e Cola ou boleto, ver o saldo ou o extrato, acompanhar um pagamento ou falar da carteira de Bitcoin e USDT na Bipa.
---

# Bipa: pagamentos e consultas

Use os dados retornados pelo conector para atender ao pedido. Responda em português do Brasil, salvo preferência diferente da pessoa. Consulte só o necessário: uma pergunta sobre saldo não exige buscar os dados pessoais da conta.

## Conexão e permissões

- Servidor: `https://mcp.bipa.app/mcp`, transporte Streamable HTTP, autenticação OAuth 2.1 gerenciada pelo aplicativo que hospeda o conector.
- Na tela de consentimento, a pessoa escolhe o que concede: `financial:read` (saldos, extrato, limites, carteira), `account:read` (dados da conta, chaves, contatos e favorecidos), `payments:request` (pedir Pix e boletos, que ficam pendentes até ela aprovar no app) e `payments:execute_trusted` (Pix para contatos de confiança já cadastrados, sem nova aprovação). Use apenas as ferramentas disponíveis na conexão; `bipa_capabilities` mostra o que a sessão pode fazer.
- Todas as ferramentas, inclusive as cotações, exigem a conta conectada. Sem conexão, o aplicativo pede que a pessoa conecte a Bipa.
- Se a conexão pedir autenticação, oriente a pessoa a reconectar a Bipa dentro do próprio aplicativo. Nunca peça PIN, senha, código ou token na conversa: o PIN só é digitado no app da Bipa.
- Se a pessoa não tiver conta, ela pode abrir uma pela opção "Criar conta na Bipa", na tela de login do conector, ou em `https://bipa.app/cadastro`. Nunca colete CPF, documentos ou selfie na conversa.

## Regras para pagamentos

1. **Só pague quando a pessoa pedir.** Não procure chaves Pix nem favorecidos sem intenção de pagamento.
2. **Confirme antes de enviar.** Mostre destinatário, valor e data, e espere a confirmação explícita. Para QR Code ou Copia e Cola, use `bipa_pix_brcode_preview` antes, para mostrar o nome e o valor.
3. **`agent_message` é obrigatório.** É o texto que a pessoa lê no app ao aprovar. Escreva o motivo real, por exemplo: "Você pediu para pagar R$ 20 ao João pelo almoço".
4. **Valores em centavos.** `amount_cents` é inteiro e maior que zero: R$ 50 = `5000`.
5. **Um `idempotency_key` por pagamento.** Gere um UUIDv7 novo para cada pagamento distinto e guarde-o junto com os IDs retornados. Se a resposta se perder, consulte a operação existente antes de tentar de novo; nunca reaproveite a chave com destinatário, valor ou data diferentes.
6. **Pedido não é liquidação.** Um Pix comum volta como `awaiting_approval` com um `outflow_request_id`: diga à pessoa que aprove no app da Bipa e acompanhe com `bipa_payment_status` ou `bipa_pix_payment_status`. Boletos e DDA voltam com `approval_id`, que também pode ser consultado em `bipa_payment_status`. `approved` e `scheduled` não provam que o dinheiro saiu.
7. **Contatos de confiança são exceção.** `bipa_pix_pay_trusted_contact` usa uma delegação que a pessoa já criou no app e pode executar sem nova aprovação. Confirme o contato e o valor com ainda mais cuidado. O pagamento ainda pode falhar ou cair em revisão manual.

## Escolha da ferramenta

| Pedido | Ferramenta | Observações |
| --- | --- | --- |
| Pix para uma chave (CPF, CNPJ, e-mail, telefone, aleatória) | `bipa_pix_pay_key` | Já resolve o destinatário; não consulte a chave antes. `bipa_pay` aceita `schedule` para agendar ou repetir. |
| Pix para alguém que a pessoa já pagou | `bipa_pix_recipient_suggestions` → `bipa_pix_pay_recipient` | Confirme o favorecido certo ("É o João do Nubank?"). |
| Pix para contato de confiança | `bipa_pix_trusted_contacts` → `bipa_pix_pay_trusted_contact` | Executa sem nova aprovação (regra 7). |
| Pix para uma tag Bipa (`$tag`) | `bipa_pix_tag_preview` → `bipa_pix_pay_tag` | |
| QR Code ou Copia e Cola | `bipa_pix_brcode_preview` → `bipa_pix_pay_brcode` | Em código de valor fixo, omita `amount_cents`. `bipa_pix_brcode_decode` só lê o código, sem consultar o servidor. |
| Cobrar alguém por Pix | `bipa_pix_brcode_encode` | Cada chamada cria um código novo; reaproveite a resposta em vez de chamar de novo. |
| Boleto pela linha digitável ou código de barras | `bipa_bank_slip_preview` → `bipa_pay_bank_slip` | Boletos de valor livre exigem `amount_cents` entre o mínimo e o máximo. |
| Contas no DDA | `bipa_dda_list` → `bipa_dda_pay` | Uma lista vazia não prova que o DDA está ativo; leia `subscription_state`. |
| Status de um pagamento | `bipa_payment_status`, `bipa_pix_payment_status` | Use o ID retornado pelo pedido, não o ID do extrato. |
| Saldo e cofrinho | `bipa_balance` | Separe disponível, total e cofrinho; mantenha cada moeda identificada. |
| Extrato | `bipa_transactions`, `bipa_transaction_detail`, `bipa_history`, `bipa_timeline` | `amount_cents` é sempre positivo: use `direction` para saber se entrou ou saiu e `status` antes de somar. Siga a paginação antes de resumir um período. |
| Limites de Pix | `bipa_limits` | Consultar um limite não autoriza transferência. |
| Chaves Pix e de depósito | `bipa_pix_keys`, `bipa_deposit` | Reproduza a chave exata, com o tipo. |
| Dados da conta | `bipa_account`, `bipa_whoami` | Preserve os documentos mascarados. |
| Carteira de BTC ou USDT | `bipa_portfolio` | `asset` (`btc`, `usdt`) e `period` (`day`, `week`, `month`, `year`, `all`). |
| Cotações | `bipa_tickers`, `bipa_btc_prices`, `bipa_usdt_prices` | `ask` é compra e `bid` é venda; preserve a moeda de cada par. |

A maioria das ferramentas de dados tem uma versão `_widget`. Em aplicativos que mostram cards, passe o resultado da ferramenta de dados, sem alterar, para o widget correspondente. Em aplicativos só de texto, responda direto. Widgets nunca buscam dados novos nem enviam pagamentos.

## Fora do escopo

- Comprar ou vender cripto: o conector não negocia ativos. Oriente a pessoa a usar o app da Bipa.
- Contas em outros bancos: o conector só enxerga a conta Bipa conectada.
- Criar contatos de confiança, chaves Pix ou alterar limites: isso só é feito no app.
