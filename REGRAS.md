# Regras do Truco Paulista implementadas, com a origem de cada regra

Cada regra abaixo cita a fonte (`F1`, `F2` — ver [`fontes/`](fontes/)). Onde as fontes
divergem, está marcado **[DIVERGÊNCIA]** com link para a decisão.

Esta é a especificação de referência de `truco_core` em
https://github.com/yryapu/truco-zap — se o código discordar daqui, o código está errado.

---

## R1. Baralho — 40 cartas ("baralho sujo")

Baralho francês de 52 menos 8, 9, 10 e curingas → 40 cartas.
Ranks: `4 5 6 7 Q J K A 2 3`. Naipes: ♦ ♠ ♥ ♣.

> F2: "No baralho sujo (ou cheio) são 40 cartas, retirando-se apenas os 8, 9, 10 e curingas."
> F1: "The 'dirty deck' of 40 cards is used".

Implementei **só** o baralho sujo. O limpo é variante opcional e não compra capacidade
nenhuma na v1 ([D08](decisoes/D08-baralho-sujo.md)).

## R2. Ordem básica de valor (sem manilha)

Crescente: `4 < 5 < 6 < 7 < Q < J < K < A < 2 < 3`

> F2: "4 < 5 < 6 < 7 < Q < J < K < Ás < 2 < 3"
> F1: "The other cards rank in the normal Truco order 3-2-A-K-J-Q-7-6-5-4"

Atenção ao que quase todo mundo erra: **Q vale menos que J**.
> F1: "as in many games of Portuguese ancestry, the Queens rank lower than the Jacks."

## R3. Vira e manilha (manilha variável / "manilha nova")

No começo de cada mão vira-se uma carta do baralho (a **vira**). As manilhas da mão são
as quatro cartas do rank **imediatamente acima** da vira na ordem **cíclica**
`[4]-3-2-A-K-J-Q-7-6-5-4-[3]` — isto é, se a vira é 3, as manilhas são os 4.

> F1: "The manilhas for this hand are the four cards of the rank immediately above the vira
> in the cyclic order [4]-3-2-A-K-J-Q-7-6-5-4-[3]. For example if the vira is a Five, the
> manilhas are the Sixes, if the vira is a Seven the manilhas are the Queens (remember that
> Queens normally rank below Jacks in this game), and if the vira is a Three (the highest
> rank), the manilhas are the Fours (the lowest)."
> F2: "A carta imediatamente superior a carta virada determina a manilha da mão. Se a carta
> virada for um 3, as manilhas são as cartas com 4."

Exemplo de controle (F1): vira = J ⇒ ordem `K(manilhas) > 3 > 2 > A > J > Q > 7 > 6 > 5 > 4`.
Nota que o rank da manilha **sai** da ordem comum. Este exemplo é um teste no código.

## R4. Ordem de naipe entre manilhas

`♣ Paus (zap) > ♥ Copas (copeta/escopeta) > ♠ Espadas (espadilha) > ♦ Ouros (pica-fumo)`

> F1: "The four manilhas rank according to their suits in descending order
> Clubs > Hearts > Spades > Diamonds."
> F2: "6 de Ouro < 6 de Espadas < 6 de Copas < 6 de Paus"

**O naipe só desempata entre manilhas.** Duas cartas comuns de mesmo rank empatam.
> F2: "O naipe da carta só é usado como critério de desempate nas rodadas em que há mais de
> uma manilha na mesa. No caso de empate onde as cartas mais altas da mesa não são manilhas,
> não há desempate e a rodada é considerada empatada."

Consequência: **não existe empate entre manilhas**. Toda manilha tem força única.

## R5. Estrutura da mão

3 cartas por jogador, 3 rodadas, jogo em sentido **anti-horário** (o próximo é o da direita).

> F2: "cada jogador recebe 3 cartas viradas do baralho. Cada mão é composta por 3 rodadas."
> F1 (Paulista): "The deal is one card at a time, in counter-clockwise order... There is no
> opportunity to pass or burn cards - everyone plays with the three cards they are dealt."

Sem passar/queimar cartas: isso é Mineiro, não Paulista (F1 explicita a diferença).

## R6. Quem vence a rodada

Vence quem pôs a carta de maior força. Se as duas maiores forem iguais e de times
opostos, a rodada **empata**. Se as duas maiores iguais forem do **mesmo** time, o time
vence (não empata).

> F1: "In the unusual case where two partners play equal highest cards to a trick while
> their opponents play lower (or face down) cards, the trick is not tied, but is won by the
> team that played the highest cards."

## R7. Carta virada para baixo (encoberta)

Na 2ª e 3ª rodada o jogador pode jogar a carta **de costas**. Ela nunca vence a rodada e é
desconsiderada na comparação. Proibido na 1ª rodada.

> F1: "In the second and third tricks there is the option to play one's card face down
> (encoberta)... A card played face down can never win a trick... Cards cannot be played
> face down in the first trick."
> F2: "Nas segunda e terceira rodada de uma mão há a opção do jogador na sua vez jogar a
> carta virada de costas."

## R8. Quem vence a mão — a tabela de empates

> F2: "Vence a mão a dupla que ganhar 2 rodadas da mão ou vencer uma e empatar outra."
> F2: "A dupla que vence a primeira rodada de uma mão tem a vantagem em caso de empate nas
> outras duas rodadas. Em caso de empate na primeira rodada, vence a mão a dupla que vencer
> primeiro uma das outras rodadas. Se todas as 3 rodadas terminarem empatadas, ela é
> finalizada sem dupla vencedora."

F1 dá a tabela exaustiva (13 linhas). Transcrita literalmente — é o teste de ouro do motor:

| R1 | R2 | R3 | resultado |
|----|----|----|-----------|
| A | A | — | **A** |
| A | empate | — | **A** |
| A | B | A | **A** |
| A | B | empate | **A** |
| A | B | B | **B** |
| empate | A | — | **A** |
| empate | empate | A | **A** |
| empate | empate | empate | **empatada (ninguém pontua)** |
| empate | empate | B | **B** |
| empate | B | — | **B** |
| B | A | A | **A** |
| B | A | empate | **B** |
| B | A | B | **B** |
| B | empate | — | **B** |
| B | B | — | **B** |

(F1 lista 15 linhas; o cabeçalho "13" acima era meu — corrigido, ver [ERROS.md](ERROS.md).)

Algoritmo equivalente, implementado: *o primeiro time a chegar a 2 rodadas ganhas vence;
se depois da 3ª rodada ninguém tem 2, vence o time que ganhou a primeira rodada não
empatada; se nenhuma rodada teve vencedor, a mão empata e ninguém pontua.*

## R9. Truco — a escada 1 → 3 → 6 → 9 → 12

Pedido de truco leva a mão de 1 para 3. Retrucos levam 3→6, 6→9, 9→12. Nada de pular etapa,
nada de voltar atrás. Valores possíveis de uma mão: **1, 3, 6, 9, 12** e nada mais.

> F2: "A ordem de pedido de truco, 6, 9, e 12 é sempre crescente e deve ser respeitada...
> Não existem outros valores de pontuação ganha por uma mão além destas: 1, 3, 6, 9 ou 12."

Respostas: **correr**, **aceitar**, **aumentar**.

- **Correr:** quem pediu ganha **o valor anterior ao pedido** (1 se o pedido era truco, 3 se
  era 6, 6 se era 9, 9 se era 12) e a mão acaba sem mais rodadas.
  > F2: "a dupla desiste da mão, a que pediu truco ganha imediatamente 1 ponto"
  > F2 (retrucos): "a dupla que pediu truco ganha imediatamente o valor atual da mão (3, 6 ou 9)"
- **Aceitar:** a mão passa a valer o valor pedido e o jogo continua.
- **Aumentar:** só pela dupla que **não** fez o último pedido. No pedido de 12 não há aumento.
  > F2: "O pedido de 6, 9 e 12 só pode ser feito pela dupla que não fez o último pedido de
  > truco ou retruco na mão." / "No pedido de 12 não existe a opção de retrucar."

No Paulista, o truco aposta na **mão** inteira (não na rodada corrente, que é o Mineiro):
> F1 (Paulista): "A call of truco is a bet on winning the hand according to the usual
> criteria: winning two tricks or the first trick won in case of ties - not just a bet on who
> will win the current trick."

Isso simplifica muito o motor: o truco só mexe no **valor** da mão, nunca em quem a vence.

## R10. Mão de onze

Quando uma dupla chega a **11** pontos: a mão já começa valendo 3 e essa dupla decide, antes
da primeira carta, se **joga** (vale 3) ou **recusa** (a outra dupla ganha 1 ponto na hora).
Pode ver as cartas do parceiro para decidir. **Sem truco** nessa mão.

> F2: "Se a mão for aceita, ela já começa valendo 3 pontos... Se a mão for recusada, a dupla
> rival que não está com 11 pontos recebe imediatamente 1 ponto a mais... Na mão de onze não
> é possível pedir 6, ou seja, ela sempre vale 3 se for aceita ou 1 se for recusada."
> F1: "the members of the team with 11 points can look at each other's hands... and must then
> decide whether to give up for 1 point or play for 3 points."

## R11. Mão de ferro — as duas duplas com 11

Ninguém decide nada, ninguém vê cartas, sem truco. Quem vencer a mão vence a **partida**.

> F2: "Joga-se a mão valendo 1 ponto, sem as duplas verem as cartas entre si e sem
> possibilidade de pedido de truco. Vence a partida a dupla que vencer a mão de ferro."
> F1: "players must not look at their cards... the winners of the deal win the game."

F2 diz que existem as variantes **aberta** (vê a própria carta) e **fechada** (joga às cegas).
Escolha em [D09](decisoes/D09-mao-de-ferro-aberta.md).
Se a mão de ferro empatar, joga-se outra (F1: "If the hand is tied, another iron hand is played").

## R12. Fim de partida — 12 pontos

> F2: "A partida termina quando a primeira dupla chega ou passa de 12 pontos. Isso pode
> acontecer mesmo que ela não passe pela mão de onze. Por exemplo, num placar de 9x9, uma
> dupla vence uma mão trucada valendo 3 pontos, ela vence a partida com 12x9."

## R13. **[DIVERGÊNCIA]** Rotação do "mão" (quem puxa a primeira rodada)

Primeira mão: jogador sorteado. Mãos seguintes: rotaciona um assento **contra** a ordem de jogo.

> F2: "Nas demais mãos, o jogador que começa a primeira rodada é sempre **o da esquerda** ao que
> começou a mão anterior." — e a ordem de jogo é anti-horária, "o próximo a jogar é o que está a
> **direita** do que jogou". Logo o mão anda contra a ordem de jogo.
> F1: "the turn to deal passes **to the right** after each hand" — logo o mão anda *junto* com a
> ordem de jogo. Contradiz F2.

Sigo F2. Decisão e critério: [D16](decisoes/D16-rotacao-do-mao.md). Esta divergência estava
**escondida** na primeira versão deste arquivo, que dizia só "rotaciona um assento" sem direção
enquanto o código seguia F1 — ver [ERROS.md](ERROS.md) E8.

## R14. **[DIVERGÊNCIA]** Quem puxa depois de uma rodada empatada

F1 (Paulista) diz: quem **liderou** a rodada empatada. F2 diz: quem pôs a **primeira carta
que empatou**. F1 lista as três variantes em *Variations* e diz que as regras discordam.
Decisão e justificativa: [D07](decisoes/D07-lider-apos-empate.md).

## R15. Pedir truco com 11 na mesa

F1 (Paulista) tem uma regra dura: "When one or both of the teams has 11 points, calling truco
automatically gives the victory to the opposing team - that is, the opposing team wins the
entire game". F2 simplesmente diz que não é possível.
Decisão: [D10](decisoes/D10-truco-na-mao-de-onze.md).

## R17. Ver a mão do parceiro para responder a um pedido

A dupla que precisa **responder** a um truco (ou a um 6, 9, 12) pode trocar as cartas de olhada
antes de correr, aceitar ou aumentar.

> F1 (Paulista): "Immediately after truco is called, the opponents can look at each other's hands
> by passing all cards face down to each other and discuss whether to accept the call. Similarly,
> if the opposing team decides to call 6 later on (or immediately), the Truco-calling team can
> also look at each other's hands before deciding if they accept the raise to 6 or decline."

Quem decide é quem vê — a olhada existe para a decisão. Em 1x1 isto é vacuamente verdadeiro.
Regra que eu tinha **perdido** na primeira versão, apesar de ter lido o parágrafo: ERROS.md E10.

## R18. Ilegal aumentar quando aceitar já venceria a partida

> F1: "It is illegal to raise a truco if just accepting would give you enough points to win the
> game. For example, suppose team A has a score of 7 and team B has 5. Team A says 'truco' and
> team B says 'vale 6'. It is now illegal (as well as stupid) for team A to say 'vale 9', because
> 6 points are already sufficient to win the game for them."

Implementada como **recusa do comando** (`aumento_desnecessario`), e o botão de aumentar não
aparece na interface nesse caso — mesma filosofia de [D10](decisoes/D10-truco-na-mao-de-onze.md):
não puno um clique que a interface permitiu. F1 trata o caso como penalidade de 1 tento; eu não
implemento penalidades, e perder a regra junto com a penalidade foi o erro E10.

## R16. 1x1

Nenhuma das fontes descreve truco paulista a dois: ambas descrevem o jogo de 4 em duplas
fixas. O 1x1 é extrapolação minha, registrada como tal: **mesmas regras, cada jogador é um
time de um, mesmo baralho de 40, mesma mão de onze (olhar a própria mão) e mesma mão de ferro.**
Isso não vem de fonte. Ver [D11](decisoes/D11-mesa-1x1.md).
