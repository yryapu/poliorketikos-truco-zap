# Erros meus, e como eu os encontrei

Ordem cronológica. Nada removido depois de corrigido.

### E1 — escrevi "tabela de 13 resultados" quando F1 lista 15
**Como encontrei:** ao transcrever a tabela de F1 para `REGRAS.md` R8 eu contei as linhas e
deu 15. O "13" tinha vindo de memória minha sobre quantos resultados *distintos* existem, não
de contagem da fonte.
**Por que importa:** é exatamente o tipo de número que soa preciso e não foi verificado.
Corrigido no próprio R8, com a nota visível em vez de apagada.

### E2 — 5 testes do motor quebraram porque `Match::novo` sorteia quem é o "mão"
**Como encontrei:** primeira execução de `cargo test -p truco_core` — 23 passaram, 5 falharam,
todas com `Err(NaoEhSuaVez)`. O motor estava certo; o teste estava errado: eu escrevi os testes
assumindo que o assento 0 começa, mas R13 manda sortear o "mão" da primeira mão, e `novo()`
sorteia.
**Por que importa:** é a classe de erro mais perigosa num benchmark assim — um teste que falha
por motivo errado faz você "consertar" o código que estava correto. A pista de que era o teste
e não o motor foi o código do erro: `NaoEhSuaVez` é uma recusa *do motor funcionando*, não um
resultado de regra errado.
**Correção:** helper `mesa()` que fixa `mao_de_quem = 0` para os testes roteirizados. O sorteio
continua em produção e o fuzz exercita qualquer assento inicial.
