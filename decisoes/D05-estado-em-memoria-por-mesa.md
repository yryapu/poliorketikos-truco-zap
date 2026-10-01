# D05 — mesa viva em memória, resultado no banco

**O que:** `Arc<AppState>` com `tokio::sync::Mutex<HashMap<TableId, Table>>`. A `Table`
guarda o `Match` (motor de regras puro) e os canais `broadcast` por jogador. Ao fim da
partida: uma transação SQLite grava placar, mexe no saldo e enfileira webhooks.

**Por quê:** uma mão de truco é efêmera e dura segundos; persistir cada carta jogada
compraria recuperação de crash que ninguém pediu, ao custo de uma escrita por jogada. O que
**precisa** sobreviver a um crash é saldo e histórico, e isso vai pro banco.

**Alternativa descartada:** um ator/task por mesa com mensagens (`mpsc`). Mais elegante na
teoria e é o que eu faria com 10 mil mesas; com um `Mutex` eu leio o fluxo todo numa função
e o lock é mantido por microssegundos (o motor é CPU puro, sem await dentro do lock).
Descartei também event sourcing: camada que não compra capacidade nenhuma aqui.

**Risco conhecido e aceito:** se o processo cair no meio de uma partida, a aposta travada
dos jogadores fica travada. Mitigação implementada: a aposta é debitada **no início** e, em
`startup`, qualquer partida que ficou `in_progress` é cancelada com estorno das apostas.
