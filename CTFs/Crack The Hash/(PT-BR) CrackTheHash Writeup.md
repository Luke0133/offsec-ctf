# Crack The Hash Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/crackthehash](https://tryhackme.com/room/crackthehash)
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/11`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Crack%20The%20Hash/(PT-BR)%20CrackTheHash%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Crack%20The%20Hash/(PT-BR)%20CrackTheHash%20Writeup.md)

- [Ferramentas Utilizadas](#ferramentas%20utilizadas)
- [Resolução do CTF](#Resolução%20do%20CTF)
	1. [Hash 1-1](#Hash%201-1)
	2. [Hash 1-2](#Hash%201-2)
	3. [Hash 1-3](#Hash%201-3)
	4. [Hash 1-4](#Hash%201-4)
	5. [Hash 1-5](#Hash%201-5)
	6. [Hash 2-1](#Hash%202-1)
	7. [Hash 2-2](#Hash%202-2)
	8. [Hash 2-3](#Hash%202-3)
	9. [Hash 2-4](#Hash%202-4)
- [Conclusão](#Conclusão)
- [Referências](#Referências)

## Ferramentas Utilizadas

Para este CTF, foram utilizadas as seguintes ferramentas:
- [hashID](https://psypanda.github.io/hashID/)[^hashid]: ferramenta de identificação de algoritmos de hash. Ela é capaz de identificar um único hash ou um arquivo contendo vários hashes únicos, e pode indicar o modo para usar em ferramentas como `hashcat`[^hashcat] e `JohnTheRipper`[^john], ferramentas de quebra de hash.
- [hashcat](https://hashcat.net/)[^hashcat]: ferramenta de recuperação de senhas, usada para atacar algoritmos de hash. Ela pode ser usada em diversos modos, como força bruta (podendo ser auxiliado por uma máscara), lista de dicionário (como a lista de senhas vazada da RockYou[^rockyou]), dentre outros. 
Bem como recursos como:
- [Crackstation](https://crackstation.net/)[^crackstation]: site de quebra de hashes por meio de lookup tables, tabelas que fazem uma relação entre hahses e suas respectivas senhas, podendo assim encontrar hashes de forma rápida, contanto que estejam na tabela. Pode ser útil caso o `hashcat`[^hashcat] não consiga quebrar a hash por exaustão.


## Resolução do CTF

O CTF Crack The Hash, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a identificação e quebra de vários hashes. No caso, esse desafio não envolve invadir uma máquina e obter flags, é apenas uma prática de quebra de hashes. Para isso, utilizei das ferramentas `hashID`[^hashid] e `hashcat`[^hashcat], e, em um caso, precisei de usar da Crackstation[^crackstation]. Por essas razões, irei indicar cada quebra de hash como uma `<FLAG>`.

O fluxo da solução foi essencialmente o mesmo para todos: identificação de hash com o `hashID`[^hashid] e, posteriormente, a sua quebra com `hashcat`[^hashcat] ou Crackstation[^crackstation]. O comando para o `hashID` foi o seguinte:

```bash
hashid -m '<hash>'
```

em que `-m` retorna o modo que deve ser usado pelo `hashcat`[^hashcat] para cada hash identificado. Ao obter uma lista contendo os possíveis algoritmos de hash, eu conseguia usar o seguinte comando no `hashcat`[^hashcat] :

```bash
hashcat -a 0 -m <hash_type> '<hash>' /usr/share/wordlists/rockyou.txt
```

em que `-a` indica o modo de ataque (em todos os casos fiz ataques com dicionário, que é o argumento `0`), `-m` indica o tipo do hash, ou seja, o algoritmo de hash que deve ser usado para a quebra (fornecido pelo `hashID`[^hashid]) e a wordlist escolhida foi a `rockyou.txt`[^rockyou], um conjunto de senhas que vazaram do site RockYou de mais de 32 milhões de contas, ou seja, esse arquivo tem muitas senhas e, portanto, é bem útil.

No total foram 9 hashes, divididas em dois grupos, de modo em que o segundo grupo teria uma dificuldade superior ao primeiro, por envolverem hahses mais complexos que ferramentas online poderiam ter mais dificuldade de identificar e quebrar (no caso, já comecei com `hashid`[^hashid] e `hashcat`[^hashcat], então não tive problemas com esse aumento de dificuldade). A solução do CTF, foi dividida conforme o número de hashes a serem quebrados:
1. [Hash 1-1](#Hash%201-1)
2. [Hash 1-2](#Hash%201-2)
3. [Hash 1-3](#Hash%201-3)
4. [Hash 1-4](#Hash%201-4)
5. [Hash 1-5](#Hash%201-5)
6. [Hash 2-1](#Hash%202-1)
7. [Hash 2-2](#Hash%202-2)
8. [Hash 2-3](#Hash%202-3)
9. [Hash 2-4](#Hash%202-4)
### Hash 1-1

**Hash Fornecido:** `48bb6e862e54f2a795ffc4e541caed4d`
Comecei com a identificação do hash:

```bash
$ hashid -m '48bb6e862e54f2a795ffc4e541caed4d'
Analyzing '48bb6e862e54f2a795ffc4e541caed4d'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
#[...]
```

Ao testar com `MD5`, obtive o resultado `<FLAG_1-1>`:

```bash
$ hashcat -a 0 -m 0 '48bb6e862e54f2a795ffc4e541caed4d' /usr/share/wordlists/rockyou.txt
#[...]
48bb6e862e54f2a795ffc4e541caed4d:<FLAG_1-1>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 48bb6e862e54f2a795ffc4e541caed4d
#[...]
```

### Hash 1-2

**Hash Fornecido:** `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`
Comecei com a identificação do hash:

```bash
$ hashid -m 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97'
Analyzing 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160
#[...]
```

Ao testar com `SHA1`, obtive o resultado `<FLAG_1-2>`:

```bash
$ hashcat -a 0 -m 100 'CBFDAC6008F9CAB4083784CBD1874F76618D2A97' /usr/share/wordlists/rockyou.txt
#[...]
cbfdac6008f9cab4083784cbd1874f76618d2a97:<FLAG_1-2>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 100 (SHA1)
Hash.Target......: cbfdac6008f9cab4083784cbd1874f76618d2a97
#[...]
```

### Hash 1-3

**Hash Fornecido:** `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032`
Comecei com a identificação do hash:

```bash
$ hashid -m '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032'
Analyzing '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032'
[+] Snefru-256 
[+] SHA-256 [Hashcat Mode: 1400]
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 [Hashcat Mode: 6900]
#[...]
```

Ao testar com `SHA-256`, obtive o resultado `<FLAG_1-3>`:

```bash
$ hashcat -a 0 -m 1400 '1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032' /usr/share/wordlists/rockyou.txt
#[...]
1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc946683d7b336b63032:<FLAG_1-3>       
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA-256)
Hash.Target......: 1c8bfe8f801d79745c4631d09fff36c82aa37fc4cce4fc94668...b63032
#[...]
```

### Hash 1-4

**Hash Fornecido:** `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`
Comecei com a identificação do hash:

```bash
$ hashid -m '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom'
Analyzing '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom'
[+] Blowfish(OpenBSD) [Hashcat Mode: 3200]
[+] Woltlab Burning Board 4.x 
[+] bcrypt [Hashcat Mode: 3200]
```

Decidi testar `blowfish`/`bycript` (ambos com o mesmo modo no hashcat), porém a previsão para quebrar o hash era de mais de um dia, por ser um algoritmo de hashing intencionalmente lento[^bcypt]. Para melhorar a eficiência da quebra desse hash, então, escolhi restringir o tamaho de senhas a serem procuradas, dado que, na página do TryHackMe, era possível ver que a resposta deveria ter 4 caracteres. Criando uma cópia de rockyou.txt contendo apenas as senhas de 4 caracteres:

```bash
awk 'length($0) == 4' rockyou.txt  > len4rockyou.txt  
```

obtive o resultado `<FLAG_1-4>`:

```bash
$ hashcat -a 0 -m 3200 '$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom' len4rockyou.txt
#[...]
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom:<FLAG_1-4>

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
Hash.Target......: $2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX...8wsRom
#[...]
```

### Hash 1-5

**Hash Fornecido:** `279412f945939ba78ce0758d3fd83daa`
Comecei com a identificação do hash:

```bash
$ hashid -m '279412f945939ba78ce0758d3fd83daa'
Analyzing '279412f945939ba78ce0758d3fd83daa'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
[+] RIPEMD-128 
#[...]
```

Ao testar com `MD5` e, depois, com `MD4` no `hashcat`[^hashcat], deparei-me com a exaustão em ambos os casos:

```bash
$ hashcat -a 0 -m 0 '279412f945939ba78ce0758d3fd83daa' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 0 (MD5)
Hash.Target......: 279412f945939ba78ce0758d3fd83daa
#[...]

$ hashcat -a 0 -m 900 '279412f945939ba78ce0758d3fd83daa' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 900 (MD4)
Hash.Target......: 279412f945939ba78ce0758d3fd83daa
#[...]
```

Sem querer desistir desses tipos de hash, testei na Crackstation[^crackstation] e tive sucesso, encontrando a `<FLAG_1-5>` rapidamente e, realmente, era do tipo `MD4`:

![cth_crackstation.png](cth_crackstation.png)

### Hash 2-1

**Hash Fornecido:** `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`
Comecei com a identificação do hash:

```bash
$ hashid -m 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85'
Analyzing 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85'
[+] Snefru-256 
[+] SHA-256 [Hashcat Mode: 1400]
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 [Hashcat Mode: 6900]
#[...]
```

Ao testar com `SHA-256`, obtive o resultado `<FLAG_2-1>`:

```bash
$ hashcat -a 0 -m 1400 'F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85' /usr/share/wordlists/rockyou.txt
#[...]
f09edcb1fcefc6dfb23dc3505a882655ff77375ed8aa2d1c13f640fccc2d0c85:<FLAG_2-1>

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: f09edcb1fcefc6dfb23dc3505a882655ff77375ed8aa2d1c13f...2d0c85
#[...]
```

### Hash 2-2

**Hash Fornecido:** `1DFECA0C002AE40B8619ECF94819CC1B`
Comecei com a identificação do hash:

```bash
$ hashid -m '1DFECA0C002AE40B8619ECF94819CC1B'
Analyzing '1DFECA0C002AE40B8619ECF94819CC1B'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
#[...]
[+] Snefru-128 
[+] NTLM [Hashcat Mode: 1000]
[+] Domain Cached Credentials [Hashcat Mode: 1100]
#[...]
```

Ao testar com `MD5` e, depois, com `MD4` no `hashcat`[^hashcat], deparei-me com a exaustão em ambos os casos:

```bash
$ hashcat -a 0 -m 0 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 0 (MD5)
Hash.Target......: 1dfeca0c002ae40b8619ecf94819cc1b
#[...]

$ hashcat -a 0 -m 900 '1DFECA0C002AE40B8619ECF94819CC1B' /usr/share/wordlists/rockyou.txt
#[...]
Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 900 (MD4)
Hash.Target......: 1dfeca0c002ae40b8619ecf94819cc1b
#[...]
```

Para não perder tempo testando diferentes tipos de algoritmos de hashing, decidi testar na Crackstation[^crackstation] novamente, e tive sucesso, encontrando a `<FLAG_2-2>` rapidamente:

![cth_crackstation2.png](cth_crackstation2.png)

O tipo do hash era `NTLM`, iria ter demorado testar vários algoritmos até ter chegado neste. De fato, ao testar com o `hashcat`[^hashcat], com o modo `NTLM`, também encontrei a `<FLAG_2-2>`:

```bash
$ hashcat -a 0 -m 1000 '1dfeca0c002ae40b8619ecf94819cc1b' /usr/share/wordlists/rockyou.txt
#[...]
1dfeca0c002ae40b8619ecf94819cc1b:<FLAG_2-2>             

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1000 (NTLM)
Hash.Target......: 1dfeca0c002ae40b8619ecf94819cc1b
#[...]
```

### Hash 2-3

**Hash Fornecido:** `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`
**Salt Fornecido:** `aReallyHardSalt`
Comecei com a identificação do hash:

```bash
$ hashid -m '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.'
Analyzing '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.'
[+] SHA-512 Crypt [Hashcat Mode: 1800]
#[...]
```

Como apenas tinha uma opção, testei com `SHA-512 Crypt`, o qual já contém o salt embutido nele, ou seja, não precisei de configurar mais nada na entrada do `hashcat`[^hashcat]. O resultado obtido foi `<FLAG_2-3>`:

```bash
$ hashcat -a 0 -m 1800 '$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.' /usr/share/wordlists/rockyou.txt
#[...]
$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.:<FLAG_2-3>
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1800 (sha512crypt $6$, SHA512 (Unix))
Hash.Target......: $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPM...ZAs02.
#[...]
```

### Hash 2-4

**Hash Fornecido:** ``e5d8870e5bdd26602cab8dbe07a942c8669e56d6``
**Salt Fornecido:** `tryhackme`
Comecei com a identificação do hash:

```bash
$ hashid -m 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6'
Analyzing 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6'
[+] SHA-1 [Hashcat Mode: 100]
[+] Double SHA-1 [Hashcat Mode: 4500]
[+] RIPEMD-160 [Hashcat Mode: 6000]
[+] Haval-160 
#[...]
```

Decidi testar com `SHA-1`, porém era necessário escolher o modo certo que aceitasse o salt fornecido, de modo que o hash de entrada seria `e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme`. Após testar alguns casos, encontrei o resultado `<FLAG_2-4>` com o modo `HMAC-SHA1` (key = $salt):

```bash
$ hashcat -a 0 -m 160 'e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme' /usr/share/wordlists/rockyou.txt
#[...]
e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme:<FLAG_2-4>

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 160 (HMAC-SHA1 (key = $salt))
Hash.Target......: e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme
#[...]
```

E finalizei o CTF.
## Conclusão

O CTF Crack The Hash, apesar de não envolver enumeração e exploração, permitiu a prática de quebra e identificação de hashes. Com alguns hashes mais complexos de se quebrar que outros, exigindo o uso de redução de dicionário e recursos externos, consegui aprender melhor o funcionamento das ferramentas `hashid` e `hashcat`.
## Referências

[^hashid]: Hashid: [https://psypanda.github.io/hashID/](https://psypanda.github.io/hashID/)
[^hashcat]: Hahscat: [https://hashcat.net/](https://hashcat.net/)
[^rockyou]: RockYou:
	- Sobre Rockyou e o vazamento de suas senhas: [https://en.wikipedia.org/wiki/RockYou](https://en.wikipedia.org/wiki/RockYou)
	- Wordlist `rockyou.txt`: [https://weakpass.com/wordlists/rockyou.txt](https://weakpass.com/wordlists/rockyou.txt)
[^jhon]: JhonTheRipper: https://www.openwall.com/john/
[^crackstation]: Crackstation: [https://crackstation.net/](https://crackstation.net/)
[^bcypt]: Sobre hashing bcrypt: [https://medium.com/@adavicenko/an-insight-into-bcrypt-slow-hashing-d11380b18391](https://medium.com/@adavicenko/an-insight-into-bcrypt-slow-hashing-d11380b18391)
