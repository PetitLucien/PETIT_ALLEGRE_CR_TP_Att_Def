user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r /home/user/.mitmproxy/dump.txt
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.9.12.3#dev}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:16:03 /2025-12-09/

[14:16:03] [INFO] parsing HTTP request from '/home/user/.mitmproxy/dump.txt'
[14:16:03] [CRITICAL] specified file '/home/user/.mitmproxy/dump.txt' does not contain a usable HTTP request (with parameters)

[*] ending @ 14:16:03 /2025-12-09/

user@motivation:~/Documents/sqlmap-dev $

docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy -p 8080:8080 mitmproxy/mitmproxy mitmdump --flow-detail 3 --showhost -v -w /home/mitmproxy/.mitmproxy/dump.txt


////////////////////////////////////////////////////////////////////////////////



user@motivation:~/Documents/sqlmap-dev $ cat > request.txt << 'EOF'
POST / HTTP/1.0
Host: 172.17.0.2:1337
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=user&password=toto
EOF
user@motivation:~/Documents/sqlmap-dev $ python sqlmap.py -r request.txt --proxy=http://127.0.0.1:8080/


////////////////////////////////////////////////////////////////////////////////


