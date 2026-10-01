# poliorketikos-truco-zap — caderno de laboratório

Pesquisa, fontes, decisões e erros do projeto **Truco Paulista** web app.
Repositório operacional (código): https://github.com/yryapu/truco-zap

Sufixo escolhido: `zap` — nome da manilha de paus, a carta mais alta do truco.

## Por que esta separação existe (minha leitura, antes de criar)

Acho que a separação existe por três razões, e concordo com duas e meia:

1. **Separa justificativa de artefato.** O repo de pesquisa é um caderno falsificável:
   fontes, previsões registradas *antes* de verificar, e refutações. O valor dele não
   depende de o código funcionar. O repo operacional tem o oposto: só vale se roda.
   Misturar os dois piora os dois — o revisor não distingue mais o que é afirmação
   carregando peso do que é prosa.
2. **Torna a ordem auditável.** Com dois repos e commits datados, dá para *provar* que a
   pesquisa precedeu o código. Num repo só, isso é indistinguível de racionalização
   retroativa escrita no fim.
3. **Protege o histórico do código.** Churn de prosa não polui o log de um repo onde
   `git log` precisa ser legível.

Onde discordo: **ADRs (decisões de desenho) idealmente moram junto ao código que elas
governam**, porque longe dele elas apodrecem silenciosamente. O enunciado manda as
decisões para cá, então elas ficam aqui — mas deixei em `truco-zap/DECISOES.md` um
índice-ponteiro para cá, para que quem lê o código ache o porquê.

## Índice

- [`fontes/`](fontes/) — cada fonte, como foi obtida e quando
- [`REGRAS.md`](REGRAS.md) — as regras do truco paulista, com a origem de cada regra
- [`decisoes/`](decisoes/) — cada decisão de desenho, com a alternativa descartada
- [`PREVISOES.md`](PREVISOES.md) — o que previ antes de verificar, com probabilidade, e se acertei
- [`ERROS.md`](ERROS.md) — todo erro meu, e como eu o encontrei
- [`MULTIAGENTE.md`](MULTIAGENTE.md) — o subagente que usei: custo medido, o que rendeu, se valeu
