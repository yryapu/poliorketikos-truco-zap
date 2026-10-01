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

### E3 — SSRF por DNS rebinding: eu validava o DNS e deixava o cliente HTTP resolver de novo
**Como encontrei:** não encontrei — o review de segurança automático que roda sobre os commits
empurrados achou, e apontou TOCTOU em `webhooks.rs`. Dou o crédito porque esconder a origem de
um achado é o tipo de desonestidade que este caderno existe para não cometer.
**O erro:** `url_permitida()` resolvia o host, checava que nenhum IP era interno, devolvia
`Ok(())` — e então `reqwest` resolvia o host **de novo** na hora do connect. Um resolvedor
hostil devolve um IP público para a checagem e `169.254.169.254` para a conexão. A guarda
parecia funcionar (e os testes passavam) porque os testes só exercitavam hosts que resolvem
consistentemente.
**Por que eu não vi:** escrevi a guarda pensando em "a URL é permitida?", que é uma pergunta
sobre a string. A pergunta certa é "para qual endereço esta conexão vai?", que é sobre o socket.
O tipo de retorno (`Result<(), String>`) carregava esse erro de enquadramento: ele descartava
justamente o dado que precisava ser usado.
**Correção:** `endereco_permitido()` devolve `Option<(host, SocketAddr)>`, e a entrega monta um
cliente com `.resolve(host, addr)` para **fixar** o endereço validado. SNI e header Host ficam
no domínio original, então TLS continua verificado contra o nome certo. Teste novo:
`devolve_o_endereco_validado_para_ser_fixado_na_conexao`.
**Lição que anoto:** uma guarda que devolve `bool` ou `()` quase sempre tem um TOCTOU escondido.
Guarda boa devolve a coisa validada.
