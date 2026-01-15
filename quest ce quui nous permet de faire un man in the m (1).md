quest ce quui nous permet de faire un man in the middle si facielement?


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



user@motivation:~/.mitmproxy $ docker network inspect bridge
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
            "245b900a0e9671fe06493473c7ded8205135e1e5f7605d70322a64bef9942716": {
                "Name": "crazy_wing",
                "EndpointID": "cdd42bfb2f0c0551de20691e7a507f67f9246800cb88d0eb84ac13bcb512c2dd",
                "MacAddress": "fe:e6:19:c4:03:5f",
                "IPv4Address": "172.17.0.4/16",
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





Pour lancer le mitm :


docker run --rm -it \
  -v ~/.mitmproxy:/home/mitmproxy/.mitmproxy \
  -p 8080:8080 \
  mitmproxy/mitmproxy


puis faire la requete puis x

1337 -> 8080 -> 1337


user@motivation:~ $ http_proxy=http://127.0.0.1:8080 lynx http://172.17.0.2:1337


user@motivation:~ $ docker network inspect bridge | grep IPv4
        "EnableIPv4": true,
                "IPv4Address": "172.17.0.2/16",
                "IPv4Address": "172.17.0.3/16",

en regardant en detail, la bonne est 172.17.0.2


user@motivation:~ $ docker run --rm -it -v ~/.mitmproxy:/home/mitmproxy/.mit
mproxy -p 8080:8080 mitmproxy/mitmproxy mitmdump -w /home/mitmproxy/.mitmpro
xy/dump.txt

avec le dump cest mieux



user@motivation:~/.mitmproxy $ ls
command_history        mitmproxy-ca-cert.p12  mitmproxy-ca.pem
dump.txt               mitmproxy-ca-cert.pem  mitmproxy-dhparam.pem
mitmproxy-ca-cert.cer  mitmproxy-ca.p12
user@motivation:~/.mitmproxy $ cat dump.txt
user@motivation:~/.mitmproxy $ cat dump.txt
2290:9:websocket;0:~8:response;417:6:reason;12:UNAUTHORIZED,11:status_code;3:401#13:timestamp_end;17:1765193695.567535^15:timestamp_start;18:1765193695.5651147^8:trailers;0:~7:content;39:{"message":"Invalid user or password"}
,7:headers;183:42:6:Server,29:Werkzeug/3.1.4 Python/3.12.12,]40:4:Date,29:Mon, 08 Dec 2025 11:34:55 GMT,]36:12:Content-Type,16:application/json,]23:14:Content-Length,2:39,]22:10:Connection,5:close,]]12:http_version;8:HTTP/1.1,}7:request;711:4:path;1:/,9:authority;0:,6:scheme;4:http,6:method;4:POST,4:port;4:1337#4:host;10:172.17.0.2;13:timestamp_end;18:1765193695.2682946^15:timestamp_start;18:1765193695.2628496^8:trailers;0:~7:content;31:username=user&password=password,7:headers;437:26:4:Host,15:172.17.0.2:1337,]67:6:Accept,54:text/html, text/plain, text/sgml, text/css, */*;q=0.01,]44:15:Accept-Encoding,21:gzip, compress, bzip2,]24:15:Accept-Language,2:en,]20:6:Pragma,8:no-cache,]28:13:Cache-Control,8:no-cache,]75:10:User-Agent,57:Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8,]37:7:Referer,23:http://172.17.0.2:1337/,]53:12:Content-Type,33:application/x-www-form-urlencoded,]23:14:Content-Length,2:31,]]12:http_version;8:HTTP/1.0,}6:backup;0:~17:timestamp_created;18:1765193695.2644253^7:comment;0:;8:metadata;0:}6:marked;0:;9:is_replay;0:~11:intercepted;5:false!11:server_conn;455:3:via;0:~19:timestamp_tcp_setup;18:1765193695.2769241^7:address;21:10:172.17.0.2;4:1337#]19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;18:1765193695.2741663^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:94e078fc-f074-407b-856b-473e89de0755;8:sockname;22:10:172.17.0.3;5:40784#]8:peername;21:10:172.17.0.2;4:1337#]}11:client_conn;404:10:proxy_mode;7:regular;8:mitmcert;0:~19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;18:1765193695.2462704^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:78857d7c-6f69-45d8-92a9-99edc17e7954;8:sockname;21:10:172.17.0.3;4:8080#]8:peername;22:10:172.17.0.1;5:42322#]}5:error;0:~2:id;36:04280597-58f4-460f-b3fa-3c503638862d;4:type;4:http;7:version;2:21#}