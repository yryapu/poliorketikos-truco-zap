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
**Resultado: ❌ errei — e errei sobre o motivo, não só sobre o resultado.**
`truco_core` compilou de primeira. `truco_server` (≈1200 linhas, axum 0.8 + sqlx + reqwest)
deu **5 erros**, e nenhum deles foi o que eu previ: não houve um único problema de assinatura
do axum. Os 5 eram a mesma coisa — eu referenciei em `main.rs` um campo (`Mesa::bolo`) e um
método (`Mesa::reagendar`) que decidi criar enquanto escrevia o `main`, e esqueci de voltar
para declará-los no `hub.rs`.

Isto é mais interessante que a previsão em si: eu apostei contra a minha capacidade de
**lembrar APIs de biblioteca**, quando deveria ter apostado contra a minha capacidade de
**manter dois arquivos coerentes escrevendo de cima para baixo**. A previsão calibrou o risco
no lugar errado. Anoto como padrão: o erro raramente está na parte que dá medo.

### P5 — `p = 0.8` — o E2E Playwright falha na primeira execução por *timing*, não por lógica
Registrada antes de rodar o E2E.
**Previ:** 0.8 de que a primeira falha seja espera/ordem de eventos (dois navegadores entrando
na mesma mesa), e não regra de jogo errada — porque as regras já estarão cobertas por teste de
unidade.
**Resultado: ❌ errei, e de um jeito instrutivo.** A primeira execução deu 12 passando, 5
falhando, 1 intermitente. A causa dominante foi **lógica, não timing**:

| falha | causa real | é timing? |
|---|---|---|
| truco não sobe para 3 | asserção minha que contradizia a R9 que eu mesmo escrevi (E4) | não |
| ranking / emblema | confundi "quem viu o fim primeiro" com "quem venceu" (E6) | não |
| webhook não entregue | `http://e2e:9099` não resolve para container de `compose run` (E5) | não |
| 2x2 não termina | driver lento + limite de 120s pequeno para 2x2 | **sim** |
| unicode (intermitente) | contenção de host + fila compartilhada entre testes (E7) | parcialmente |

Eu previ 0.8 em "timing, não lógica" com o raciocínio de que "as regras já estarão cobertas por
teste de unidade". O raciocínio estava certo e **irrelevante**: as regras estavam cobertas, e o
que falhou foi a lógica *dos testes*, não a do produto. Eu tratei "lógica" como "lógica do
jogo" e esqueci que o teste também tem lógica — e que a lógica do teste é a menos revisada do
projeto, porque não existe teste do teste.

### P6 — `p = 0.5` — o subagente revisor acha ≥1 divergência real no motor
Registrada antes de invocar o subagente.
**Previ:** 0.5. Se os testes de unidade cobrem a tabela de F1, o que sobra para ele achar é
regra *não* coberta por teste: mão de onze, ordem de pedido de truco, ou encoberta na 1ª rodada.
**Resultado: ✅ acertei, e subestimei.** Ele achou **uma** divergência real de regra (a direção
da rotação do mão, R13/E8) **mais** duas regras de F1 que eu nunca implementei nem especifiquei
(R17 e R18) **mais** um furo de autorização no motor, **mais** 7 regras citadas e sem teste —
todas confirmadas por mim contra as fontes. Zero achados fabricados; ele declarou
"categoria C: nada encontrado" em vez de inventar.

E o palpite dentro da previsão — "o que sobra para ele achar é regra não coberta por teste:
mão de onze, ordem de pedido de truco, ou encoberta na 1ª rodada" — estava errado nos três
itens. O que ele achou foi o que eu nem tinha considerado possível: um teste que afirmava a
implementação em vez da fonte. Medição completa em [MULTIAGENTE.md](MULTIAGENTE.md).

### P7 — `p = 0.6` — depois das correções, a suíte E2E passa inteira
Registrada **depois** de disparar a segunda execução e **antes** de ver qualquer resultado dela.
**Previ:** 0.6. Não mais alto porque esta máquina está com 90+ containers de pé e eu já vi uma
query trivial de SQLite levar 10 s aqui; a partida 2x2 entre quatro navegadores é o teste mais
sensível a isso. Se falhar, aposto que é ela, por tempo, e não por asserção.
**Resultado: ✅ acertei.** 18 de 18, em 3,9 min, **zero intermitentes** (a primeira execução
levara 17,8 min com 5 falhas e 1 intermitente). O teste que eu apontei como mais sensível, o
2x2, caiu de 2,3 min estourando o limite para 1,1 min passando — o ganho veio do driver de
partida em um único `page.evaluate` por passo em vez de quatro `isVisible()`.

Vale separar duas coisas que eu misturei na previsão: eu acertei o **resultado** e o raciocínio
sobre **onde** estava o risco estava certo, mas a causa que eu temia (contenção de host) não
apareceu porque eu removi a sensibilidade a ela em vez de torcer. Previsão resolvida por ação,
não por sorte — o que é o único jeito honesto de resolver uma a 0.6.

### P8 — `p = 0.3` — existe ainda ≥1 regra de F1/F2 que eu não implementei e não sei
Registrada agora, sem jeito de verificar nesta sessão.
**Previ:** 0.3 **depois** da revisão adversarial ter achado duas (R17, R18). Antes dela eu teria
dito 0.1, e teria errado. Corrijo para cima porque descobri que meu modo de ler as fontes perde
regras que estão em parágrafos que eu já aproveitei — e a revisão cobriu o motor, não as fontes
inteiras linha por linha.
**Resultado: não verificável nesta sessão.** Fica registrado como risco conhecido, não como
acerto nem como erro. Registrar uma previsão que eu não posso resolver é melhor que não
registrar: ela diz onde eu acho que o trabalho está incompleto.
