# Fontes

Todas obtidas em **2026-10-01** (UTC), no início da sessão, antes de qualquer código.

| id | fonte | como foi obtida | quando (UTC) | papel |
|----|-------|-----------------|--------------|-------|
| **F1** | `pagat.com/put/truco_br.html` — "Truco (Brazilian)", de John McLeod, baseado em informação de Jonathan Avila e Gustavo Schafaschek | `curl -sSL` da página inteira + conversão HTML→texto (script Python ad hoc). Cópia local do texto extraído em `fontes/F1-pagat-truco_br.txt`. | 2026-10-01 ~05:30 | **fonte primária em inglês.** Contém a tabela exaustiva de resultados de mão (empates) e a seção "Truco Paulista" com as diferenças em relação ao Mineiro. |
| **F2** | `casin.paginas.ufsc.br/files/2010/10/Regras_Truco_Paulista.txt` — "Regras do Truco Paulista", hospedado na UFSC | `curl -sSL` + `iconv latin1→utf8`. Cópia local em `fontes/F2-regras-ufsc.txt`. | 2026-10-01 ~05:33 | **fonte primária em português.** Mais explícita que F1 em: escada 1→3→6→9→12, quem ganha ao "correr", mão de onze, mão de ferro, naipe como critério só entre manilhas. |
| **F3** | `blog.megajogos.com.br/tudo-sobre-truco-paulista/` (via WebSearch) | busca web `regras truco paulista manilha vira mão de onze empate 12 pontos`; li o resumo retornado pela busca, não a página inteira | 2026-10-01 ~05:32 | **corroboração terciária.** Usada só para checar que F1 e F2 não eram excêntricas no empate e na mão de onze. Não é base de nenhuma regra isolada. |
| **F4** | `pagat.com/put/truco.html` — página-índice do grupo "Put" | WebFetch | 2026-10-01 ~05:29 | só confirmou que Paulista = manilha variável definida pela vira. Redundante com F1. |

## Por que F1 e F2 e não uma só

F1 é a única que dá a **tabela exaustiva dos 13 resultados possíveis de uma mão** com
empates — exatamente a parte onde implementações erram. F2 é a única que afirma
textualmente a regra de **"correr"** (quem corre dá à outra dupla o valor *anterior* ao
pedido) e a de **naipe só desempata entre manilhas**. Nenhuma das duas sozinha fecha o jogo.

## Divergência real entre F1 e F2 (registrada, não escondida)

Quem começa a rodada seguinte a uma rodada **empatada**:

- **F1 (seção Paulista):** "When a trick is tied, the same player who led to the tied trick leads to the next trick."
- **F2:** "No caso da rodada anterior ter empatado, começa por aquele jogador que pôs na mesa a primeira carta que empatou."

F1 lista as três variantes possíveis na seção *Variations* ("Play after a tied trick") e
admite que "different sets of rules disagree". Ou seja: não há verdade única. A escolha
está em [`decisoes/D07-lider-apos-empate.md`](decisoes/D07-lider-apos-empate.md).
