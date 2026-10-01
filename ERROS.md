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

### E4 — eu escrevi uma asserção de teste que contradizia a R9 que eu mesmo havia escrito
**Como encontrei:** a primeira execução do Playwright falhou no teste de truco. A asserção era
"depois de a mão chegar a 6, nenhum dos dois pode pedir". A R9 que eu escrevi em `REGRAS.md`,
citando F2, diz o contrário: "O pedido de 6, 9 e 12 só pode ser feito pela dupla que **não**
fez o último pedido". O último pedido (6) foi do respondente, então o pedinte original **pode**
pedir nove.
**Por que importa:** o motor estava certo; o teste estava errado *na direção perigosa* — se eu
tivesse "consertado" o código para passar o teste, teria quebrado uma regra correta para
satisfazer uma asserção inventada. A única coisa que me salvou foi reler a R9 antes de mexer
no motor.
**Correção:** o teste agora exige "pedir nove" no pedinte original e botão ausente no
respondente, o que o transforma num teste de R9 em vez de um teste de nada.

### E5 — a entrega de webhook do E2E não alcançava o receptor: `http://e2e:9099` não resolve
**Como encontrei:** nos logs do servidor, `entrega ... falhou (tentativa 1): http: error
sending request for url (http://e2e:9099/hook)`, três vezes por evento. O teste do webhook
falhou por timeout e eu só soube o motivo olhando o log do *servidor*, não o do teste.
**O erro:** eu apontei o webhook para o nome do serviço do Compose (`e2e`). Um container criado
por `docker compose run` é anexado à rede, mas se chama `<projeto>-e2e-run-<hash>`, e o alias
de serviço não é garantido para containers avulsos.
**Correção:** o teste descobre o próprio IP na rede do Compose (`os.networkInterfaces()`) e
registra o webhook nesse IP. Não depende de DNS nenhum.
**Lição:** quando um teste de integração falha por timeout, o log útil costuma estar no outro
lado da conexão.

### E6 — `jogarAteOFim` devolvia "quem viu o fim primeiro" e eu chamei de "vencedor"
**Como encontrei:** o teste de ranking falhou esperando o emblema "pé-de-meia" num jogador que
não tinha vitória. A tela de fim aparece para os **dois** jogadores — vencedor e perdedor — e
meu laço devolvia a primeira página em que ela ficou visível.
**Correção:** função `quemVenceu()` separada, que lê o texto e procura "Vitória". A confusão
estava no nome da variável, que é onde esse tipo de bug mora.

### E7 — testes E2E compartilhavam fila e se emparelhavam entre si
**Como encontrei:** falhas intermitentes com `abrirMesa` estourando 30s. A fila do servidor é
indexada por `(modo, aposta)`, e vários testes usavam a mesma aposta — então o jogador do teste
N era emparelhado com um resto do teste N-1 e o par do teste N ficava esperando para sempre.
**Correção:** cada spec usa uma aposta única (7, 13, 17, 23, 29, 31, 37, 41). Fila isolada por
construção, sem precisar de limpeza entre testes.
**Efeito colateral bom:** para fazer isso eu precisei trocar o `<select>` de 5 apostas fixas por
um campo numérico — e aí percebi que o `<select>` **violava o enunciado**, que diz "aposta
quanto quiser". Eu tinha estreitado o requisito sem notar. Ver E9.

### E8 — o mão rotacionava na direção errada, e o meu teste petrificava o erro
**Como encontrei:** não encontrei — a revisão adversarial que eu pedi a um subagente encontrou,
e foi o achado mais valioso dela. Crédito registrado.
**O erro:** `nova_mao` fazia `+1`. F2, que é a fonte que `REGRAS.md` R13 cita, manda o mão andar
para a esquerda, isto é, **contra** a ordem de jogo (`-1`). F1 manda `+1`. Eu implementei F1 e
documentei F2, sem marcar divergência — exatamente o pecado que o resto do `REGRAS.md` evita.
**O agravante, que é a parte que eu quero lembrar:** o teste `r13_o_mao_rotaciona_um_assento`
afirmava `(primeiro + i) % 4`. Ele passava porque estava copiando a implementação. Um teste que
copia a implementação é pior que nenhum teste: ele produz confiança falsa e congela o bug.
Reescrevi o teste a partir da frase da fonte, e aí ele falhou — como devia ter falhado desde o
começo. Decisão em [D16](decisoes/D16-rotacao-do-mao.md).
**Padrão que anoto:** escrever o teste olhando o código em vez de olhar a fonte. Os outros
testes de regra eu escrevi a partir de F1/F2; este eu escrevi a partir do `engine.rs`.

### E9 — o campo de aposta era um `<select>` de 5 valores, e o enunciado diz "quanto quiser"
**Como encontrei:** de lado, enquanto consertava E7. Precisei de apostas arbitrárias para
isolar as filas dos testes, fui trocar o `<select>` e só então li de novo o requisito:
"aposta **quanto quiser** na partida". Cinco presets não são "quanto quiser".
**Por que eu não vi antes:** eu tratei "aposta" como um controle de interface a projetar, e não
como uma frase do enunciado a cumprir. O `<select>` até parecia melhor UX — e era mais fácil de
validar. Conveniência minha disfarçada de decisão de produto.
**Correção:** campo numérico com atalhos (amistoso / 50 / 100 / 500 / tudo), validado no
cliente para dar erro rápido e validado no servidor contra o saldo lido do **banco**.

### E10 — duas regras de F1 que eu nunca implementei nem especifiquei
**Como encontrei:** a mesma revisão adversarial (categoria "regra-ausente").
1. **Ver a mão do parceiro ao responder um pedido.** F1 (Paulista) é explícito: "Immediately
   after truco is called, the opponents can look at each other's hands... Similarly, if the
   opposing team decides to call 6 later on, the Truco-calling team can also look". Eu tinha
   lido essa seção — ela está no mesmo bullet list de onde tirei três outras regras — e passei
   por cima. Agora é R17.
2. **Ilegal aumentar quando aceitar já venceria a partida.** F1: "It is illegal to raise a truco
   if just accepting would give you enough points to win the game", com exemplo numérico
   (7 × 5, A não pode pedir 9 depois do 6 de B). Eu li essa regra como "penalidade", que é a
   seção onde ela reaparece, e penalidade eu tinha decidido não implementar — e assim perdi a
   regra junto com a penalidade. Agora é R18, implementada como recusa do comando (mesma
   filosofia de [D10](decisoes/D10-truco-na-mao-de-onze.md)) e com o botão escondido na interface.
**Padrão:** as duas regras estavam em parágrafos que eu já tinha lido e aproveitado. Reler não
é o mesmo que reler procurando o que falta.

### E11 — moldura dupla: pus uma caixa de carta atrás de um glifo que já desenha uma carta
**Como encontrei:** tirei prints das telas e olhei. No print a carta aparecia dentro de outra
carta — um retângulo creme maior em volta do desenho do glifo.
**O erro:** eu tratei o code point como se fosse um ícone a ser emoldurado. Ele não é: o glifo
U+1F0A1 **é** a carta inteira, com borda, naipe e valor.
**A correção errada que tentei primeiro:** tirar a caixa e deixar só o glifo. Resultado pior — o
glifo do Noto Sans Symbols 2 é **só contorno**, a face é transparente, então as cartas viraram
fantasmas de contorno sobre a mesa escura. O print mostrou isso na hora.
**A correção certa:** medir. `e2e/medir.mjs` renderiza o glifo em canvas e devolve o retângulo
de tinta: avanço 0.675em, desenho 0.575em × 0.765em, folga 0.05em de cada lado. O corpo creme
agora é um `::before` encaixado nessas medidas exatas.
**Lição:** eu estava prestes a ajustar números no olho até "ficar bom". Medir custou um script
de 30 linhas e acabou com a questão.

### E12 — botões invisíveis: texto creme sobre painel creme
**Como encontrei:** print do lobby. "Registrar endereço" simplesmente não estava lá — ou melhor,
estava, com `color: var(--giz)` (creme) sobre a superfície de papel (creme).
**Por que aconteceu:** criei a classe `.liso` pensando no fundo escuro e depois inventei a
superfície de papel sem revisar quem herdava o quê. Classe utilitária com cor fixa e duas
superfícies de fundo diferentes é uma armadilha que eu mesmo armei.
**Correção:** `.papel .liso` com cor própria. O certo a longo prazo seria a cor do botão vir de
uma variável que a superfície redefine; para o tamanho deste front, a regra de escopo basta.

### E13 — no celular, o painel de avisos cobria o botão de procurar mesa
**Como encontrei:** **não foi pelo print.** O print mostrava a tela inteira e parecia bem. Quem
achou foi o Playwright, tentando clicar: `<section class="papel recibo">… intercepts pointer
events`.
**O erro:** declarei `grid-area: avisos` nos painéis **fora** da media query, mas
`grid-template-areas` só existe no layout de duas colunas. Sem as áreas nomeadas, os painéis
caíram empilhados no mesmo lugar.
**Lição que importa mais que o bug:** print prova aparência, não prova *uso*. Um elemento
invisível por cima de um botão é exatamente o tipo de defeito que uma imagem não mostra e um
clique mostra na primeira tentativa.

### E14 — as cartas do leque interceptavam o clique do botão de truco
**Como encontrei:** a suíte E2E. Dois testes de truco falharam, um deles levando 3 minutos — o
tempo do teste inteiro — porque o Playwright ficou repetindo o clique.
**A causa, que eu não teria adivinhado:** as cartas da mão usam `transform` para abrir em leque.
`transform` cria **contexto de empilhamento**, e com isso elas passam a pintar na camada de
posicionados, **acima** do conteúdo de fluxo normal que vem depois delas no DOM — inclusive os
botões de ação. A carta da ponta cobria o botão e engolia o clique.
**Correção:** `position: relative; z-index: 5` na barra de ações, e mais respiro embaixo do leque.
**Por que registro:** eu teria jurado que "elemento posterior no DOM fica por cima". Fica — até
alguém aplicar `transform`. É a classe de bug que só um clique de verdade encontra.

### E15 — mudei a copy e quebrei um teste que conferia a copy antiga
**Como encontrei:** suíte E2E. O teste procurava `'sua vez'` em minúscula; a copy nova diz
"Sua vez. Escolha uma carta."
**Por que registro, já que é trivial:** porque a tentação era "consertar" a copy de volta para o
teste passar. O teste é que estava acoplado demais — conferia caixa alta de uma frase de
interface, que é exatamente a coisa que deve poder mudar. Agora ele usa `/sua vez/i`, e a frase
melhor fica. **Teste não é dono da copy.**

### E16 — ranking em que o jogador não se acha
**Como encontrei:** o teste de ranking falhou — procurava o apelido do vencedor na lista e não
achava. A primeira reação foi "o teste está errado". Estava, mas não só.
**O que a mensagem de erro mostrou:** o `Received string` do Playwright trazia os 50 primeiros
colocados, todos com `1V / 0D`, e o jogador recém-vitorioso não estava entre eles. Com o banco
cheio de convidados das execuções anteriores, o top 50 ficou saturado de empates.
**O problema real, que o teste só tropeçou:** `/api/ranking` devolvia os N primeiros e nada mais.
Um ranking em que você não se encontra não é um ranking — é uma lista de outras pessoas. Isso
não aparece com o banco vazio, que é como eu testei a vida inteira.
**Correção:** `/api/ranking` passa a devolver também a **sua** linha, com a posição real
calculada sobre a tabela inteira (`ROW_NUMBER()` numa subconsulta), esteja você em 1º ou em 300º.
A tela fixa essa linha no pé da lousa. A lista visível caiu de 50 para 25, porque agora ela não
precisa mais tentar conter todo mundo.
**Lição:** "o teste está errado" e "o produto está errado" não são excludentes. Aqui os dois
estavam, e corrigir só o teste teria apagado a evidência do defeito.
