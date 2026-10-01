# D14 — uso de subagentes: um, adversarial, sobre o motor de regras

**O que:** usei **um** subagente, depois do motor estar escrito, com uma tarefa estreita:
receber a tabela de 15 resultados de F1 e a especificação R1–R16, ler `truco_core` e
reportar **divergências**, sem autorização para editar. Não usei subagente para escrever
código.

**Por quê assim:** o risco dominante do enunciado é declarado nele mesmo — "regra de jogo
errada invalida o resto". O ganho marginal de paralelizar *escrita* num projeto deste
tamanho é pequeno (o gargalo é decisão, não digitação) e o custo é alto: um subagente
escrevendo o motor enquanto eu escrevo o servidor produz duas noções do mesmo tipo. Já um
leitor independente do motor ataca o risco dominante sem tocar no código.

**Alternativa descartada:** fan-out de 4–5 agentes (motor / HTTP / front / testes / infra).
Descartada porque o contrato entre essas partes não existia ainda no começo — paralelizar
antes do contrato estar escrito gera integração, não velocidade. Também descartei não usar
nenhum: deixaria a parte mais crítica sem revisão independente.

**Medição:** ver seção `multiagente` em `resultado.json` do repo operacional, preenchida com
o resultado real (o que o agente achou, se era verdade, e se valeu).
