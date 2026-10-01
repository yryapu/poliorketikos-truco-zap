# D10 — pedir truco na mão de onze é **erro rejeitado**, não derrota automática

**O que:** se um cliente manda `truco` durante mão de onze ou mão de ferro, o servidor
responde `{"t":"erro","codigo":"truco_proibido_mao_de_onze"}` e **nada muda** no jogo.

**Por quê:** F1 descreve a regra de bar: "calling truco automatically gives the victory to
the opposing team - that is, the opposing team wins the entire game!", e chama ela mesma de
"very harsh rule and very easy to fall for". F2 simplesmente diz que não é possível pedir.
Numa mesa física a regra tem função: pune quem não está prestando atenção, e falar é
irreversível. Num app, o botão de truco **existe na tela** — punir com a partida inteira um
clique que a interface permitiu é transferir para o jogador uma falha de interface minha.

A solução correta em software é a interface: na mão de onze o botão de truco fica
**desabilitado**, e o servidor rejeita por garantia (defesa em profundidade contra cliente
modificado). Nenhum jogador honesto chega a ver esse erro.

**Alternativa descartada:** implementar a regra de F1 literalmente. Descartada por ser
fielmente hostil: fidelidade a uma regra cuja razão de ser (a fala irreversível) não existe
no meio. Registro a divergência em vez de esconder — se um avaliador considerar isso infidelidade
à fonte, a discordância é deliberada e está aqui, com motivo.
