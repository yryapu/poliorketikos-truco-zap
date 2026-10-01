# D17 — direção visual: boteco, não cassino

**O que:** mesa de bar sob uma lâmpada incandescente, toalha xadrez vermelha escura, carta em
papel creme, e o placar contado em **sementes de olho-de-cabra** — doze por dupla.
Tipografia: `Alfa Slab One` (letra de placa pintada à mão) **só** no logotipo e no grito de
truco; `Archivo` em todo o resto; `Noto Sans Symbols 2` nas cartas.

**O que estava errado antes:** feltro verde. Feltro verde é **cassino** — pôquer, blackjack,
Las Vegas. Truco paulista não é jogado em cassino; é jogado em mesa de boteco, de padaria, de
sítio, no fim de um churrasco. Eu tinha pegado o *default de "jogo de cartas"* em vez de olhar
para o assunto. É o mesmo erro de desenho que leva todo app de finanças a ser azul.

**De onde veio a paleta:** de um objeto concreto, não de um gerador. Uma mesa de boteco depois
das onze — o marrom-escuro fora do foco da lâmpada (`#140c09`), a toalha xadrez desbotada de
lavagem (`#7d1f1a`), o filamento incandescente (`#ffc35c`), o papel da carta (`#f7f1e3`).

**A ousadia gasta num lugar só:** o placar. Dois números viraram duas cuias de 12 sementes.
E isso não é enfeite — é a forma **tradicional** de contar tentos, e está na fonte que eu mesmo
citei:

> F1: "It is traditional to score using ormosia seeds (tentos): at least 22 seeds are needed,
> and at the start of the game they are stored in a convenient container, such as a dish or jar."

A semente de olho-de-cabra (*Ormosia arborea*) é escarlate com um olho preto — é por isso que
ela tem esse nome, e é por isso que ela funciona desenhada: "quanto falta pra eu ganhar" vira
físico e instantâneo, em vez de exigir aritmética.

**Estrutura como informação, não como decoração:** no lobby, superfície de **papel** = onde você
age; superfície de **lousa de giz** = o que você só lê. Isso substituiu cinco cartões cinzas
idênticos — o kit de SaaS, que é o default de interface gerada.

**Alternativas descartadas:**
- **Feltro verde** (o que estava). Descartado por ser o assunto errado.
- **Desenhar a carta em CSS** (retângulo, índice no canto, pips) em vez de usar o glifo. Ficaria
  mais bonito e mais controlável, e eu descartei por coerência: o enunciado faz do code point a
  representação da carta, e usá-lo também na tela mantém uma única noção de "carta" do protocolo
  ao pixel. Se um dia a legibilidade das cartas de figura virar problema, esta é a primeira coisa
  a reabrir.
- **Toalha xadrez em vermelho vivo.** Descartada porque gritava mais alto que as cartas. Ficou
  escura e com o quadriculado largo — material, não padronagem.

**Medir em vez de chutar:** o glifo do bloco Playing Cards é **só contorno** — a face da carta é
transparente. Medi o glifo em canvas (`e2e/medir.mjs`): avanço `0.675em`, desenho
`0.575em × 0.765em`, folga de `0.05em` de cada lado. O corpo creme da carta é um `::before`
encaixado nessas medidas. Chutar isso foi o erro E11.
