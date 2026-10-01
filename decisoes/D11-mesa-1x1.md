# D11 — 1x1 é extrapolação declarada, não regra de fonte

**O que:** mesa de 2 jogadores, cada um é um time de um. Mesmas 40 cartas, mesma vira, mesma
escada de truco, mesmo 12 pontos. Na mão de onze o jogador olha a própria mão (não há parceiro
para consultar). Na mão de ferro, idem D09.

**Por quê:** nenhuma das fontes descreve truco paulista a dois — F1 e F2 descrevem um jogo de
quatro em duplas fixas. O enunciado pede 1x1. Então marco isto explicitamente como
**extrapolação minha**, para que não seja confundido com regra citada. A extrapolação é
conservadora: não inventei nada, só colapsei "dupla" em "time de tamanho 1". Nada no motor
assume 2 jogadores por time — o `Match` é indexado por assento e por `team = seat % 2`.

**Alternativa descartada:** 1x1 com 2 cartas por jogador, ou mão de onze sem decisão no 1x1.
Descartadas porque mudariam a matemática do jogo e eu não tenho fonte para nenhuma das duas.
