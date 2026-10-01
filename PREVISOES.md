# Previsões

Registradas **antes** de cada verificação, com probabilidade. O resultado é preenchido depois,
no mesmo arquivo, sem reescrever a previsão.

Formato: `P<n>` · claim · `p` · resultado.

---

### P1 — `p = 0.75` — Fontes vão divergir em pelo menos um ponto material
Registrada antes de ler F2 (já tinha F1).
**Previ:** que ao cruzar duas fontes eu acharia ≥1 divergência que muda comportamento
observável do jogo.
**Resultado: ✅ acertei.** Duas divergências materiais: quem puxa depois de empate
(R14/[D07](decisoes/D07-lider-apos-empate.md)) e o que acontece ao pedir truco com 11
(R15/[D10](decisoes/D10-truco-na-mao-de-onze.md)). Bônus não previsto: o tamanho do baralho
"limpo" (27 em F1 vs 24 em F2).

### P2 — `p = 0.9` — U+1F0A1/B1/C1/D1 são exatamente ♠♥♦♣ do Ás, e o exemplo do enunciado confirma
Registrada antes de rodar qualquer código.
**Previ:** que `🂡 🂱 🃁 🃑` do enunciado são Ás de espadas/copas/ouros/paus nessa ordem, e que o
offset do Valete é `+0xB` e o da Dama `+0xD`, com `+0xC` sendo o Cavaleiro (inexistente no truco).
**Resultado: ✅ acertei.** Verificado por teste `unicode_roundtrip` em `truco_core` (ida e volta
nas 40 cartas) e por inspeção dos nomes Unicode. A existência do Cavaleiro em `+0xC` é
exatamente o tipo de armadilha que teria dado um baralho de 44 cartas se eu tivesse assumido
ranks contíguos.

### P3 — `p = 0.6` — o motor passa a tabela de 15 linhas de F1 na primeira tentativa
Registrada depois de escrever `resolve_hand` e **antes** de rodar `cargo test`.
**Previ:** 0.6 de passar de primeira, porque a regra "vence quem ganhou a primeira rodada
não-empatada" é curta mas a linha `B,A,empate → B` é contraintuitiva (o time que perdeu duas
comparações consecutivas... não; o que ganhou a *primeira*). Risco principal: eu confundir
"primeira rodada" com "rodada anterior".
**Resultado: ✅ acertei, e por sorte menor do que parece.** `r8_tabela_de_empates_de_f1`
passou de primeira, e o teste extra `r8_espaco_completo_decide_no_momento_mais_cedo_possivel`
(que varre {A,B,T}³ e exige que a 3ª rodada nunca mude um resultado já decidido na 2ª) também.
O que eu temia — confundir "primeira rodada" com "rodada anterior" — não aconteceu porque
escrevi a resolução sobre a *lista* de rodadas (`rodadas.iter().flatten().next()`), não sobre
estado incremental. Vale registrar: o acerto veio da forma do código, não de eu ter sido
cuidadoso na hora.

### P4 — `p = 0.35` — `cargo build` compila de primeira
Registrada antes do primeiro build.
**Previ:** 0.35. Baixa de propósito: axum 0.8 mudou assinaturas de handler e `WebSocketUpgrade`
em relação a 0.6/0.7, e escrevo sem consultar os docs de cada assinatura.
**Resultado parcial: `truco_core` compilou de primeira.** Mas a previsão era sobre o build todo,
e o risco que eu citei (assinaturas do axum 0.8) está no `truco_server`, que ainda não compilou
quando escrevi isto. Resolvido abaixo, junto com o resultado do servidor — não vou declarar
acerto com metade do build.

### P5 — `p = 0.8` — o E2E Playwright falha na primeira execução por *timing*, não por lógica
Registrada antes de rodar o E2E.
**Previ:** 0.8 de que a primeira falha seja espera/ordem de eventos (dois navegadores entrando
na mesma mesa), e não regra de jogo errada — porque as regras já estarão cobertas por teste de
unidade.
**Resultado:** ver `RESULTADO-P5` abaixo.

### P6 — `p = 0.5` — o subagente revisor acha ≥1 divergência real no motor
Registrada antes de invocar o subagente.
**Previ:** 0.5. Se os testes de unidade cobrem a tabela de F1, o que sobra para ele achar é
regra *não* coberta por teste: mão de onze, ordem de pedido de truco, ou encoberta na 1ª rodada.
**Resultado:** ver `RESULTADO-P6` abaixo.
