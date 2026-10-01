# D06 — **não** usar WASM no cliente

**O que:** o front é HTML + CSS + JavaScript puro, um arquivo cada, sem build step, sem
bundler, sem npm.

**Por quê não WASM** (investiguei antes de decidir, como pedido):

1. **O cliente não tem lógica de regra para rodar.** O servidor é autoritativo: ele valida a
   jogada, resolve a rodada e manda o novo estado. Compartilhar o motor Rust via
   `wasm-bindgen` só faria sentido se o cliente precisasse *decidir* algo sozinho — previsão
   otimista, replay offline, bot local. Nada disso está na v1.
2. **Não há cálculo pesado.** O trabalho do cliente é desenhar ≤ 11 caracteres Unicode e
   reagir a mensagens JSON de poucas centenas de bytes. Isso é trabalho de DOM, onde WASM
   é *mais lento* que JS porque toda chamada atravessa a fronteira.
3. **Custo real e imediato:** `wasm-pack`/`trunk` no build, um artefato de centenas de KB a
   baixar antes da primeira carta, e um segundo alvo de compilação para manter. O requisito
   explícito é "cadastro rápido — jogando em menos de um minuto"; WASM empurra na direção
   oposta.
4. **Compartilhar tipos não é motivo suficiente.** O único tipo compartilhado interessante é
   a carta, e ela já é *um caractere* ([D04](D04-cartas-unicode-no-protocolo.md)). Não há
   duplicação de lógica para eliminar.

**Quando eu mudaria de ideia** (critério registrado, para ser honesto sobre a decisão):
se a v2 quiser bot local, animação de física de cartas, ou validação otimista da jogada
antes do round-trip — aí o motor no cliente passa a comprar capacidade, e `wasm-bindgen`
sobre o `truco_core` **já existente e já testado** é o caminho, porque o crate de regras é
`no_std`-friendly e sem I/O justamente para manter essa porta aberta.

**Alternativa descartada:** além do WASM, descartei React/Vue + bundler. Para uma tela de
mesa e uma de lobby, um framework cobra um `node_modules` e um pipeline de build para
economizar umas 100 linhas de JS. Mau negócio.
