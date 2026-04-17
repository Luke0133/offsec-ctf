# Offsecfcil (OFFSEC - Hash) Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/offsecfcil](https://tryhackme.com/room/offsecfcil)
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/16
## Sumário

> Link do writeup no github: 

- [Ferramentas Utilizadas](#ferramentas%20utilizadas)
- [Resolução do CTF](#Resolução%20do%20CTF)
	1. [Hash 1-1](#Hash%201-1)
	2. [Hash 1-2](#Hash%201-2)
	3. [Hash 1-3](#Hash%201-3)
	4. [Hash 1-4](#Hash%201-4)
	5. [Hash 1-5](#Hash%201-5)
- [Conclusão](#Conclusão)
- [Referências](#Referências)

## Ferramentas Utilizadas

Para este CTF, foram utilizadas as seguintes ferramentas:
- [hashID](https://psypanda.github.io/hashID/)[^hashid]: ferramenta de identificação de algoritmos de hash. Ela é capaz de identificar um único hash ou um arquivo contendo vários hashes únicos, e pode indicar o modo para usar em ferramentas como `hashcat`[^hashcat] e `JohnTheRipper`[^john], ferramentas de quebra de hash.
- [hashcat](https://hashcat.net/)[^hashcat]: ferramenta de recuperação de senhas, usada para atacar algoritmos de hash. Ela pode ser usada em diversos modos, como força bruta (podendo ser auxiliado por uma máscara), lista de dicionário (como a lista de senhas vazada da RockYou[^rockyou]), dentre outros. 
## Resolução do CTF

O CTF offsecfcil (ou OFFSEC - Hash), disponível no TryHackMe, é um desafio de dificuldade fácil que requer a identificação e quebra de vários hashes, feito pela OFFSEC para a SATECH/UFSC. No caso, esse desafio não envolve invadir uma máquina e obter flags, então, assim como no [Crack The Hash](https://tryhackme.com/room/crackthehash), é apenas uma prática de quebra de hashes. Para isso, utilizei das ferramentas `hashID`[^hashid] e `hashcat`[^hashcat]. Por essas razões, irei indicar cada quebra de hash como uma `<FLAG>`.

O fluxo da solução foi essencialmente o mesmo para todos: identificação de hash com o `hashID`[^hashid] ou Hash Analyzer[^hashanalyzer] e, posteriormente, a sua quebra com `hashcat`[^hashcat]. O comando para o `hashID` foi o seguinte:

```bash
hashid -m '<hash>'
```

em que `-m` retorna o modo que deve ser usado pelo `hashcat`[^hashcat] para cada hash identificado. Ao obter uma lista contendo os possíveis algoritmos de hash, eu conseguia usar o seguinte comando no `hashcat`[^hashcat] (na maioria dos casos, pois em outros foi necessário usar a força bruta):

```bash
hashcat -a 0 -m <hash_type> '<hash>' /usr/share/wordlists/rockyou.txt
```

em que `-a` indica o modo de ataque (em todos os casos fiz ataques com dicionário, que é o argumento `0`), `-m` indica o tipo do hash, ou seja, o algoritmo de hash que deve ser usado para a quebra (fornecido pelo `hashID`[^hashid]) e a wordlist escolhida foi a `rockyou.txt`[^rockyou], um conjunto de senhas que vazaram do site RockYou de mais de 32 milhões de contas, ou seja, esse arquivo tem muitas senhas e, portanto, é bem útil.

No total foram 5 hashes, e a solução do CTF foi dividida conforme o número de hashes a serem quebrados:
1. [Hash 1-1](#Hash%201-1)
2. [Hash 1-2](#Hash%201-2)
3. [Hash 1-3](#Hash%201-3)
4. [Hash 1-4](#Hash%201-4)
5. [Hash 1-5](#Hash%201-5)
### Hash 1-1

**Hash Fornecido:** `482c811da5d5b4bc6d497ffa98491e38`
Comecei com a identificação do hash:

```bash
$ hashid -m '482c811da5d5b4bc6d497ffa98491e38'
Analyzing '482c811da5d5b4bc6d497ffa98491e38'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
#[...]
```

Ao testar com `MD5`, obtive o resultado `<FLAG_1-1>`:

```bash
$ hashcat -a 0 -m 0 '482c811da5d5b4bc6d497ffa98491e38' /usr/share/wordlists/rockyou.txt
#[...]
482c811da5d5b4bc6d497ffa98491e38:<FLAG_1-1>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 482c811da5d5b4bc6d497ffa98491e38
#[...]
```

### Hash 1-2

**Hash Fornecido:** `861c4f67e887dec85292d36ab05cd7a1a7275228`
Comecei com a identificação do hash:

```bash
$ hashid -m '861c4f67e887dec85292d36ab05cd7a1a7275228'
Analyzing '861c4f67e887dec85292d36ab05cd7a1a7275228'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160 
#[...]
```

Ao testar com `SHA1`, obtive o resultado `<FLAG_1-2>`:

```bash
$ hashcat -a 0 -m 100 '861c4f67e887dec85292d36ab05cd7a1a7275228' /usr/share/wordlists/rockyou.txt
#[...]
861c4f67e887dec85292d36ab05cd7a1a7275228:<FLAG_1-2>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 100 (SHA1)
Hash.Target......: 861c4f67e887dec85292d36ab05cd7a1a7275228
#[...]
```

### Hash 1-3

**Hash Fornecido:** `4149c5cc4c378444d116d65ad5ba4099`
Comecei com a identificação do hash:

```bash
$ hashid -m '4149c5cc4c378444d116d65ad5ba4099'
Analyzing '4149c5cc4c378444d116d65ad5ba4099'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
#[...]
```

Ao testar com `MD5` no `hashcat`[^hashcat], o programa retornou que houve exaustão:

```bash
$ hashcat -a 0 -m 0 '4149c5cc4c378444d116d65ad5ba4099' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 0 (MD5)
Hash.Target......: 4149c5cc4c378444d116d65ad5ba4099
#[...]
```

A dica do CTF dizia que a senha continha apenas letras e números. Sabendo então, pelo formulário de resposta do TryHackMe, que a senha continha apenas 6 caracteres executei o seguinte comando no hashcat[^hashcat]:

```bash
hashcat -m 0 -a 3 -1 ?l?u?d '4149c5cc4c378444d116d65ad5ba4099' ?1?1?1?1?1?1
```

A diferença desse comando é que o modo de ataque é por força bruta (`-a 3`) e, configurando um conjunto de caracteres customizados com `-1 ?l?u?d` (atribui ao `custom-charset1` a união dos conjuntos de caracteres `l`, que testa letras minúsculas, `u`, que testa letras maiúsculas, e `d`, que testa os dígitos de 0 a 9) e a máscara para da força bruta (`?1?1?1?1?1?1`, que irá usar para cada um dos 6 caracteres o conjunto de caracteres que defini), testei novamente com `MD5`, mas deu exaustão novamente:  

```bash
$ hashcat -m 0 -a 3 -1 ?l?u?d '4149c5cc4c378444d116d65ad5ba4099' ?1?1?1?1?1?1
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 0 (MD5)
Hash.Target......: 4149c5cc4c378444d116d65ad5ba4099
#[...]
```

Tentei então com `MD4` e encontrei a `<FLAG_1-3>`, que, realmente, não era uma palavra que estava em `rockyou.txt`[^rockyou]:

```bash
$ hashcat -m 0 -a 3 -1 ?l?u?d '4149c5cc4c378444d116d65ad5ba4099' ?1?1?1?1?1?1
#[...]
4149c5cc4c378444d116d65ad5ba4099:<FLAG_1-3>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 900 (MD4)
Hash.Target......: 4149c5cc4c378444d116d65ad5ba4099
#[...]
```

### Hash 1-4

**Hash Fornecido:** `cdeb746ec095149627348b61d4140fc58b745875`
**Salt Fornecido:** `satech`
Comecei com a identificação do hash:

```bash
$ hashid -m 'cdeb746ec095149627348b61d4140fc58b745875'
Analyzing 'cdeb746ec095149627348b61d4140fc58b745875'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160 
[+] Tiger-160 
```

Decidi testar com `SHA-1`, porém era necessário escolher o modo certo que aceitasse o salt fornecido, de modo que o hash de entrada seria `cdeb746ec095149627348b61d4140fc58b745875:satech`. Com a dica do CTF, encontrei o resultado `<FLAG_1-4>` com o modo `HMAC-SHA1 (key = $salt)`:

```bash
$ hashcat -a 0 -m 160 'cdeb746ec095149627348b61d4140fc58b745875:satech' /usr/share/wordlists/rockyou.txt
#[...]
cdeb746ec095149627348b61d4140fc58b745875:satech:<FLAG_1-4>

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 160 (HMAC-SHA1 (key = $salt))
Hash.Target......: cdeb746ec095149627348b61d4140fc58b745875:satech
#[...]
```

### Hash 1-5

**Hash Fornecido:** `362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912`
Comecei com a identificação do hash:

```bash
$ hashid -m '362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912'
Analyzing '362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912'
[+] Snefru-256 
[+] SHA-256 [Hashcat Mode: 1400]
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 [Hashcat Mode: 6900]
#[...]
```

Ao testar com `SHA-256` no `hashcat`[^hashcat], o programa retornou que houve exaustão:

```bash
$ hashcat -a 0 -m 1400 '362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: 362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963...852912
#[...]
```

A dica do CTF dizia que a senha continha apenas letras minúsculas e números. Sabendo então, pelo formulário de resposta do TryHackMe, que a senha continha apenas 6 caracteres executei o seguinte comando no hashcat[^hashcat]:

```bash
hashcat -m 1400 -a 3 -1 ?l?d '362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912' ?1?1?1?1?1?1
```

Assim como para o [Hash 1-3](#Hash%201-3), esse comando está no modo de ataque por força bruta (`-a 3`), mas configurei o conjunto de caracteres customizados apenas com `-1 ?l?d` (apenas letras minúsculas, `l`, e números, `d`). Com a máscara para da força bruta (`?1?1?1?1?1?1`, que irá usar para cada um dos 6 caracteres o conjunto de caracteres que defini), testei novamente com `SHA-256`, e encontrei a `<FLAG_1-5>`:

```bash
$ hashcat -m 1400 -a 3 -1 ?l?d '362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912' ?1?1?1?1?1?1
#[...]
362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963ee2abdf852912:<FLAG_1-5>

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: 362fda2183b7ac73400a83f6ab2c359451e48adf6c3d46a2963...852912
#[...]
```

E finalizei o CTF.
## Conclusão

O CTF offecfcil, apesar de não envolver enumeração e exploração, também permitiu, assim como o [Crack The Hash](https://tryhackme.com/room/crackthehash), a prática de quebra e identificação de hashes. Com alguns hashes mais complexos de se quebrar que outros, exigindo o uso de força bruta, por não estarem presentes na `rockyou.txt`, consegui aprender melhor o funcionamento das ferramentas `hashid` e `hashcat`.
## Referências

[^hashid]: Hashid: [https://psypanda.github.io/hashID/](https://psypanda.github.io/hashID/)
[^hashcat]: Hahscat: [https://hashcat.net/](https://hashcat.net/)
[^rockyou]: RockYou:
	- Sobre Rockyou e o vazamento de suas senhas: [https://en.wikipedia.org/wiki/RockYou](https://en.wikipedia.org/wiki/RockYou)
	- Wordlist `rockyou.txt`: [https://weakpass.com/wordlists/rockyou.txt](https://weakpass.com/wordlists/rockyou.txt)
[^jhon]: JhonTheRipper: https://www.openwall.com/john/