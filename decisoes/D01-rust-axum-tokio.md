# D01 — axum + tokio no servidor

**O que:** `axum` 0.8 sobre `tokio`, com `axum::extract::ws` para o WebSocket e
`tower-http` para arquivos estáticos e tracing.

**Por quê:** o requisito de "melhores bibliotecas para este cenário" aponta para uma coisa
concreta: HTTP + WebSocket **no mesmo processo e na mesma porta**, com estado compartilhado
e tipado. `axum` faz exatamente isso — o upgrade de WS é um extractor, não um servidor
separado — e é o front-end HTTP do ecossistema `hyper`/`tower`, o que significa middleware
(trace, limites de corpo) sem cola escrita à mão. `tokio` é a única escolha séria de runtime
async hoje para este perfil de carga.

**Alternativa descartada:** `actix-web`. É rápido e maduro, mas o modelo de atores dele
coloca uma segunda abstração de concorrência em cima do async/await que eu não preciso: a
minha concorrência é "um `Mutex` sobre um `HashMap` de mesas", não um sistema de atores.
Também descartei `tungstenite` cru + `hyper` à mão: seria escrever o upgrade e o roteamento
que o `axum` já me dá, sem comprar nada.
