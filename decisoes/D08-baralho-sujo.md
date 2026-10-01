# D08 — só o baralho sujo (40 cartas)

**O que:** 40 cartas, sem opção de baralho limpo.

**Por quê:** é o baralho que F1 e F2 dão como o do truco paulista. Baralho limpo é variante
e, pior, as fontes **divergem no tamanho dele** (F1 diz 27 cartas; F2 diz 24) — implementar
uma variante onde eu não sei nem quantas cartas são é fabricar bug.

**Alternativa descartada:** oferecer os dois como opção de mesa. Descartada por "nada de
camada que não compre capacidade": dobraria a matriz de teste do motor para servir um gosto
que ninguém pediu na v1.
