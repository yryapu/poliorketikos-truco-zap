# D16 — o "mão" rotaciona **contra** a ordem de jogo (segue F2)

**O que:** `nova_mao` faz `mao_de_quem = (mao_de_quem + jogadores - 1) % jogadores`.

**A divergência:** a ordem de jogo é anti-horária — "o próximo a jogar é o que está a direita
do que jogou" (F2), que no meu modelo de assentos é `+1`.

- **F2:** "Nas demais mãos, o jogador que começa a primeira rodada é sempre **o da esquerda** ao
  que começou a mão anterior." ⇒ `-1`.
- **F1:** "the turn to deal passes **to the right** after each hand", e o mão é o jogador à
  direita do dealer ⇒ `+1`.

As fontes se contradizem de verdade. Não é ambiguidade de leitura: F2 manda o mão andar contra
a direção do jogo, F1 manda andar junto.

**Por quê F2:** mesmo critério de [D07](D07-lider-apos-empate.md) — entre duas fontes que se
contradizem sobre a variante paulista, ganha a mais específica sobre ela. Também é o critério
que eu já tinha escrito, e trocar de critério por conveniência num segundo caso seria escolher
o resultado e chamar de método.

**Alternativa descartada:** F1 (`+1`), que era o que o código fazia antes. Descartada pelo
critério acima.

**Efeito material: pequeno, e digo isso de propósito.** Em 4 mãos de uma mesa 2x2 todo mundo é
mão uma vez nas duas variantes; em 1x1 as duas coincidem. O que muda é a *ordem* em que isso
acontece, logo quem é "pé" (último a jogar, posição vantajosa) em cada mão específica.

**Como isto apareceu:** o código fazia `+1` enquanto `REGRAS.md` R13 citava **só F2** — e R13
dizia "rotaciona um assento", sem direção, o que esconde a divergência em vez de registrá-la.
Pior: o teste `r13_o_mao_rotaciona_um_assento_por_mao` afirmava `+1`, isto é, testava o que o
código fazia, não o que a fonte manda. Um teste assim é pior que nenhum: ele dá confiança
falsa. Achado pela revisão adversarial — ver [ERROS.md](../ERROS.md) E8.
