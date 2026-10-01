# Subagentes: o que custou, o que rendeu, e se valeu

Decisão prévia: [`decisoes/D14`](decisoes/D14-subagentes.md). Resumo: **um** subagente, usado
*depois* do motor estar escrito, com tarefa estreita e **sem** permissão de editar.

## Medição

| | |
|---|---|
| subagentes usados | 1 |
| tokens consumidos pelo subagente | **133.443** |
| chamadas de ferramenta dele | 15 |
| duração | 435 s (7m15s) de parede, em paralelo com o meu trabalho |
| custo meu de orquestração | escrever o prompt (≈40 linhas) + ler e verificar o relatório |
| arquivos que ele editou | **0** (por desenho) |

## O que ele achou, e o que era verdade

Ele reportou 15 achados. Eu conferi **cada um** contra as fontes antes de agir — e isso é parte
do custo, não um extra.

| achado | o que era | agi? |
|---|---|---|
| rotação do mão segue F1 enquanto a spec cita F2, e o teste petrifica o código | **verdade, e o mais valioso** | sim — [D16](decisoes/D16-rotacao-do-mao.md), [E8](ERROS.md) |
| F1 permite ver a mão do parceiro ao responder um pedido; não implementado nem especificado | **verdade** | sim — R17 |
| F1 proíbe aumentar quando aceitar já venceria; não implementado nem especificado | **verdade** | sim — R18 |
| `aplicar` não valida o número do assento; nas fases de resposta `pode_agir` olha só a paridade | **verdade** (furo de autorização, sem pânico) | sim — `Erro::AssentoInexistente` |
| 7 regras citadas na spec e **sem teste** (mão de ferro empatada, resposta por qualquer membro em 2x2, rodada toda encoberta, três empates pelo `Match`, R10/R11 em 2x2, o par `4<5`, invariante do delta de placar) | **verdade, uma por uma** | sim — 9 testes novos |
| 3 achados que ele mesmo classificou como **falso positivo que descartou** (R14/R15 deliberados, vira tirada antes de distribuir, auditoria de `unwrap`) | corretos como falsos positivos | nada a fazer |
| `Match::novo` panica com 3 jogadores em vez de devolver `Err` | verdade, e aceitável num crate puro | não — documentado |

**Zero achados fabricados.** Ele declarou explicitamente "categoria C: nada encontrado" e
"não achei nenhuma divergência em `resultado_da_mao`" em vez de inventar algo para parecer útil —
que era exatamente o comportamento que eu pedi no prompt e o risco que eu temia.

## Valeu?

**Sim, e por uma razão específica:** o enunciado diz "regra de jogo errada invalida o resto", e
o achado principal é uma regra errada que **os meus 28 testes não pegavam, porque um deles
estava copiando a implementação em vez da fonte**. Nenhuma quantidade de reler o meu próprio
código teria encontrado isso — o viés que produziu o bug é o mesmo que produziu o teste.

O que fez valer não foi "mais um modelo", foi a **assimetria da tarefa**: ele recebeu as fontes
e a spec e teve que confrontá-las com o código, sem ter escrito o código. Se eu tivesse pedido
"revise o motor", ele teria lido o motor e concordado comigo.

Três escolhas de desenho que eu atribuo o resultado a:

1. **Depois, não durante.** Paralelizar escrita exigiria um contrato entre as partes que ainda
   não existia; teria produzido integração, não velocidade.
2. **Sem permissão de editar.** Ele reporta, eu verifico e decido. Um agente que conserta o que
   acha que achou introduz erro novo com a confiança do achado antigo.
3. **Formato de saída obrigando "nada encontrado"** e uma categoria explícita de falso positivo.
   Sem isso, a pressão para parecer útil produz ruído.

## E o que eu faria diferente

Teria rodado **dois**: um sobre o motor (este) e outro sobre o `truco_server` — que foi
justamente o que ele declarou não ter conseguido verificar, por não estar no escopo que eu dei.
A camada WebSocket/lobby ficou sem revisão independente, e está nos riscos conhecidos.
