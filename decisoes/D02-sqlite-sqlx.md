# D02 — SQLite via `sqlx`, no mesmo container

**O que:** `sqlx` com feature `sqlite`, arquivo `truco.db` em volume, WAL ligado, migrations
em `migrations/`, queries checadas em tempo de compilação onde dá.

**Por quê:** o estado durável deste app é pequeno e de baixa contenção — jogadores, saldo,
histórico de partidas, webhooks. SQLite dá transação ACID (crítico para saldo: debitar aposta
e creditar prêmio não pode ficar pela metade) com **zero** serviço adicional. "Mantenha
simples. Nada de camada que não compre capacidade" — um Postgres aqui compra durabilidade que
eu já tenho e cobra um container, uma rede, um healthcheck e um backup.

**Alternativa descartada:** PostgreSQL. Descartado porque o único benefício real no horizonte
(muitos escritores concorrentes, réplicas) não existe numa v1 de um processo. Também descartei
**estado só em memória**: inaceitável, perderia saldo e ranking a cada restart, e saldo que
some não é saldo.

**Nota honesta:** `sqlx` com `query!` exige banco em tempo de compilação ou `.sqlx` offline.
Para não acoplar o build a isso, uso a API dinâmica (`sqlx::query_as` com `bind`) — perco a
checagem estática e pago com testes. Decisão consciente de trade-off, não descuido.
