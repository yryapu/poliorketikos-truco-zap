# D15 — robôs completam a mesa depois de 8 segundos de fila

**O que:** se um jogador fica 8 segundos numa fila sem que ninguém mais entre, a mesa abre
com robôs nos assentos vazios. O robô joga uma política mínima (menor carta que ganha do que
está na mesa; aceita/corre por força de mão; pede truco com 25% de chance quando pode).

**A tensão, dita antes da justificativa:** o enunciado diz "o que a v1 precisa ter, **e nada
além disso**", e robô não está na lista. Então isto é, formalmente, escopo a mais.

**Por que mesmo assim:** a lista *contém* "o jogador entra e começa a jogar em menos de um
minuto". Com a casa vazia — que é o estado de toda demo e de toda avaliação — essa frase é
simplesmente falsa sem alguém do outro lado. Sem robô, o requisito não é atendido: ele é
atendido *condicionalmente a haver outro humano online*, o que é uma condição que o produto
não controla. Preferi entregar o requisito e declarar o custo a entregar uma tela de espera e
chamá-la de conformidade.

Custo real: ~60 linhas em `hub.rs::acao_do_robo`, nenhuma estrutura nova, nenhuma dependência.
O robô é um assento sem `jogador_id` — o motor não sabe que ele existe.

**Efeito colateral que precisou de decisão própria:** robô não tem conta, então não aposta. Se
o prêmio fosse "2× a aposta" como num 1x1 humano-contra-humano, ganhar de um robô imprimiria
moeda. Resolvi definindo o prêmio como **o bolo** (soma do que humanos apostaram), dividido
entre os humanos vencedores. Consequência: ganhar de robô devolve a própria aposta e não lucra.
Isso é menos satisfatório e é o certo — fecha a farmagem de moedas.

**Alternativa descartada:** caixa de seleção "quero robôs". Descartada porque transfere ao
jogador uma decisão que ele não tem informação para tomar (ele não sabe quantas pessoas estão
na fila) e porque deixa o caminho ruim como padrão. Também descartei um robô melhor: um robô
que joga bem é um projeto, e aqui ele é parceiro de treino, não adversário.

**Risco aceito:** humano desconectado no meio da partida passa a ser jogado pela mesma
política, senão a mesa travaria para os outros três. Ele volta a controlar o assento se
reconectar. É pior que pausar, e muito melhor que travar.
