# D13 — teste de front com Playwright, rodando **dentro do Docker**

**O que:** suíte Playwright (`@playwright/test`) num container
`mcr.microsoft.com/playwright`, apontada para o servidor em outro container na mesma rede
Compose. Um `docker compose --profile test run e2e` roda tudo.

**Por quê Playwright:** o que precisa ser provado aqui é que **duas sessões de navegador
independentes jogam uma partida inteira por WebSocket** — isto é, dois contextos de browser
isolados, cookies separados, esperas por evento em vez de `sleep`. Playwright faz isso
nativamente (`browser.newContext()` por jogador, auto-waiting, `expect(...).toHaveText` com
retry). É a ferramenta que testa exatamente o requisito "comunicação em tempo real", em vez
de testar o DOM sozinho.

**Por que dentro do Docker e não na máquina:** o `node` do host está quebrado
(`Library not loaded: libllhttp.9.3.dylib` — Homebrew com dylib faltando). Descobri isso no
primeiro comando da sessão. Em vez de consertar o ambiente de alguém, rodo o teste onde ele é
reprodutível para qualquer avaliador. Isso também atende "rode e teste localmente, em Docker,
de forma segura".

**Alternativas descartadas:**
- **Selenium/WebDriver:** precisa de grid ou driver separado, não tem auto-wait decente, e
  multi-contexto isolado é desconfortável. Mais peça móvel para menos garantia.
- **Só testes Rust (`#[tokio::test]` com cliente WS):** eu *também* tenho esses, e eles provam
  o protocolo. Mas não provam que a tela funciona — e o enunciado pede o front testado.
- **Testar o front com jsdom/vitest:** não executa WebSocket real nem layout. Provaria menos
  e eu teria que fingir que provou mais.
