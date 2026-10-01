# D12 — webhooks: fila no banco, entrega em background, assinatura HMAC

**O que:** o jogador registra uma URL + recebe um `secret`. Eventos `partida.comecou`,
`partida.terminou`, `partida.resultado` entram numa tabela `webhook_delivery` dentro da
**mesma transação** que grava o resultado. Uma task em background entrega com `reqwest`,
assina o corpo com `HMAC-SHA256` no header `X-Truco-Signature`, e tenta 3 vezes com backoff.

**Por quê:** entregar webhook em linha com o fim da partida acopla a latência do jogo à
latência de um servidor de terceiro — e se ele estiver fora, o jogo trava ou o evento se
perde. Fila na mesma transação dá a garantia que importa: **se o resultado foi gravado, o
evento vai ser entregue** (at-least-once). HMAC porque sem assinatura o receptor não tem como
saber que o POST veio de mim, e webhook sem autenticação é um convite.

**Alternativa descartada:** entregar direto, in-line, sem fila. Descartada pelo acoplamento
acima. Descartei também Redis/fila externa: o "broker" aqui é uma tabela com `SELECT ... WHERE
status='pending'` — comprar um Redis para isso é exatamente a camada que não compra capacidade.

**Risco aceito:** at-least-once, não exactly-once. O receptor precisa ser idempotente pelo
`evento_id`, que vai no corpo. Documentado.

**Segurança — SSRF:** webhook é "o servidor faz request para URL que o usuário escolheu", o
que é SSRF por construção. Mitigação: só `https://` (e `http://` liberado apenas quando
`TRUCO_WEBHOOK_ALLOW_HTTP=1`, para teste local), e recusa de hosts que resolvem para
loopback/link-local/privado quando `TRUCO_WEBHOOK_ALLOW_PRIVATE` não está ligado.
