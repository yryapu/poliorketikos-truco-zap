# D03 — sessão: token opaco em cookie HttpOnly, não JWT

**O que:** no cadastro/login, gero 32 bytes de `OsRng`, mando ao cliente em base64url num
cookie `HttpOnly; SameSite=Strict; Path=/`, e guardo no banco apenas o **SHA-256** do token.
Toda rota autenticada resolve `token → player_id` por esse hash. O WebSocket autentica pelo
mesmo cookie no handshake.

**Por quê:** "dados de cada jogador isolados dos outros" é uma propriedade de autorização, e
o jeito mais curto de garanti-la é o servidor nunca confiar em nada que o cliente diga sobre
quem ele é. Token opaco é **revogável na hora** (deleta a linha) e um vazamento do banco não
entrega sessões vivas (só hashes). `HttpOnly` tira o token do alcance de XSS.

**Alternativa descartada:** JWT. Para este app o JWT só adiciona custo: revogação exige uma
denylist (ou seja, o banco que eu queria evitar), o payload não precisa ser auto-contido
porque é um único processo, e a superfície de erro de JWT (`alg: none`, confusão
HMAC/RSA, expiração mal checada) é grande para zero ganho. Também descartei
**senha enviada em cada request**: óbvio não.

Senhas: `argon2` (Argon2id, parâmetros default da crate). Alternativa descartada: `bcrypt`
(limite de 72 bytes e sem resistência a GPU comparável) e qualquer SHA puro (inaceitável).
