# D04 — a carta no WebSocket é o caractere Unicode, e o servidor é a única autoridade

**O que:** no protocolo WS a carta é a string de **um** code point do bloco
*Playing Cards* (U+1F0A0–U+1F0DF): `"🂡"`, `"🂱"`, `"🃁"`, `"🃑"`. Nada de `{"rank":"A","suit":"S"}`.

Mapeamento (verificado em código, teste `unicode_roundtrip`):

| naipe | base | A | 2..7 | J | Q | K |
|-------|------|---|------|---|---|---|
| ♠ Espadas | U+1F0A0 | 1F0A1 | +2..+7 | 1F0AB | 1F0AD | 1F0AE |
| ♥ Copas   | U+1F0B0 | 1F0B1 | +2..+7 | 1F0BB | 1F0BD | 1F0BE |
| ♦ Ouros   | U+1F0C0 | 1F0C1 | +2..+7 | 1F0CB | 1F0CD | 1F0CE |
| ♣ Paus    | U+1F0D0 | 1F0D1 | +2..+7 | 1F0DB | 1F0DD | 1F0DE |

O offset `+C` (U+1F0xC, "Knight"/Cavaleiro) **não existe** no truco e nunca trafega.
A carta encoberta trafega como `"🂠"` (U+1F0A0, *Playing Card Back*) — que é justamente o
caractere Unicode certo para "carta de costas", não um símbolo inventado.

**Por quê:** foi mandado pelo enunciado, e é uma boa ideia por conta própria: o alfabeto do
protocolo tem exatamente 40 símbolos válidos, então um parser estrito rejeita qualquer coisa
fora do baralho sem schema extra, e o log de uma partida é legível por humano.

**Alternativa descartada:** objeto `{rank, suit}`. Mais fácil de debugar num editor que não
renderiza emoji, mas exige validar dois campos e os cruzamentos impossíveis. Também
descartei mandar o Unicode **e** o par estruturado: seriam duas fontes de verdade para a
mesma coisa, e uma delas acabaria divergindo.

**Consequência de segurança que isso NÃO resolve:** o cliente poderia mandar uma carta que
não tem. O servidor valida que a carta está na mão daquele jogador; o Unicode é só
transporte. O estado de jogo nunca sai do servidor completo — cada jogador recebe a própria
mão e nunca a dos outros ([D06](D06-wasm-nao.md) depende disso).
