user@motivation:~ $ docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy mitmdump --flow-detail 3 -w4 -w /home/mitmproxy/.mitmproxy/dump.txt


user@motivation:~/Documents/polytech/web/[Very Easy] KORP Terminal $ sh build_docker.sh
Error response from daemon: No such container: web_korp_terminal
[+] Building 2.0s (16/16) FINISHED                           docker:default
 => [internal] load build definition from Dockerfile                   0.0s
 => => transferring dockerfile: 814B                                   0.0s
 => [internal] load metadata for docker.io/library/python:3.12-alpine  1.3s
 => [internal] load .dockerignore                                      0.0s
 => => transferring context: 2B                                        0.0s
 => [ 1/11] FROM docker.io/library/python:3.12-alpine@sha256:f47255bb  0.1s
 => => resolve docker.io/library/python:3.12-alpine@sha256:f47255bb0d  0.1s
 => [internal] load build context                                      0.0s
 => => transferring context: 983B                                      0.0s
 => CACHED [ 2/11] RUN apk update     && apk add --no-cache --update   0.0s
 => CACHED [ 3/11] RUN python -m pip install --upgrade pip             0.0s
 => CACHED [ 4/11] COPY .flag.txt /flag.txt                            0.0s
 => CACHED [ 5/11] RUN mkdir -p /app                                   0.0s
 => CACHED [ 6/11] WORKDIR /app                                        0.0s
 => CACHED [ 7/11] COPY challenge .                                    0.0s
 => CACHED [ 8/11] RUN pip install --no-cache-dir -r requirements.txt  0.0s
 => CACHED [ 9/11] COPY conf/supervisord.conf /etc/supervisord.conf    0.0s
 => CACHED [10/11] COPY --chown=root entrypoint.sh /entrypoint.sh      0.0s
 => CACHED [11/11] RUN chmod +x /entrypoint.sh                         0.0s
 => exporting to image                                                 0.3s
 => => exporting layers                                                0.0s
 => => exporting manifest sha256:1d40639dd0f019377263f88aec1b88324320  0.0s
 => => exporting config sha256:f596e28fa936d72c1a251a5ccde9f20169f42d  0.0s
 => => exporting attestation manifest sha256:a8da331e3c124c37da62d0a1  0.1s
 => => exporting manifest list sha256:99fa03614afbce65589c53dc7ec43b1  0.0s
 => => naming to docker.io/library/web_korp_terminal:latest            0.0s
 => => unpacking to docker.io/library/web_korp_terminal:latest         0.0s
/usr/bin/mysql_install_db: Deprecated program name. It will be removed in a future release, use 'mariadb-install-db' instead
Installing MariaDB/MySQL system tables in '/var/lib/mysql' ...


user@motivation:~/Documents/sqlmap-dev $ http_proxy=http://127.0.0.1:8080 lynx http://172.17.0.3:1337





user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r /home/user/.mit
mproxy/dump.txt
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.12.3#dev}
|_ -| . [,]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 15:55:56 /2025-12-09/

[15:55:56] [INFO] parsing HTTP request from '/home/user/.mitmproxy/dump.txt'
[15:55:56] [CRITICAL] specified file '/home/user/.mitmproxy/dump.txt' does not contain a usable HTTP request (with parameters)

[*] ending @ 15:55:56 /2025-12-09/
