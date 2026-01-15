3079:9:websocket;0:~8:response;1422:6:reason;2:OK,11:status_code;3:200#13:timestamp_end;18:1765289527.6696582^15:timestamp_start;18:1765289527.6674013^8:trailers;0:~7:content;1042:<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KORP Terminal - Log-in</title>
    <link rel="shortcut icon" href="/static/img/icon.png" type="image/png">
    <link rel="stylesheet" href="/static/css/style.css">
</head>

<body>
    <div class="screen">
        <div class="screen-2">
            <form action="" method="post">
                <p class="title">KORP Terminal - User Authentication</p>
                <hr>
                <label>Username:</label>
                <input type="text" name="username" placeholder="...">
                <br>
                <label>Password:</label>
                <input type="password" name="password" placeholder="...">
                <div class="buttons-dialog">
                    <button type="submit" href="javascript:void();">Log-in</button>
                    <button>Close connection</button>
                </div>
            </form>
        </div>
    </div>
</body>

</html>,7:headers;193:42:6:Server,29:Werkzeug/3.1.4 Python/3.12.12,]40:4:Date,29:Tue, 09 Dec 2025 14:12:07 GMT,]44:12:Content-Type,24:text/html; charset=utf-8,]25:14:Content-Length,4:1042,]22:10:Connection,5:close,]]12:http_version;8:HTTP/1.1,}7:request;496:4:path;1:/,9:authority;0:,6:scheme;4:http,6:method;3:GET,4:port;4:1337#4:host;10:172.17.0.2;13:timestamp_end;17:1765289527.659093^15:timestamp_start;18:1765289527.6571612^8:trailers;0:~7:content;0:,7:headers;256:26:4:Host,15:172.17.0.2:1337,]67:6:Accept,54:text/html, text/plain, text/sgml, text/css, */*;q=0.01,]44:15:Accept-Encoding,21:gzip, compress, bzip2,]24:15:Accept-Language,2:en,]75:10:User-Agent,57:Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8,]]12:http_version;8:HTTP/1.0,}6:backup;0:~17:timestamp_created;18:1765289527.6577022^7:comment;0:;8:metadata;0:}6:marked;0:;9:is_replay;0:~11:intercepted;5:false!11:server_conn;454:3:via;0:~19:timestamp_tcp_setup;18:1765289527.6626515^7:address;21:10:172.17.0.2;4:1337#]19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;17:1765289527.661331^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:02793dfe-20b6-4f84-ab02-c58982612bc5;8:sockname;22:10:172.17.0.3;5:60362#]8:peername;21:10:172.17.0.2;4:1337#]}11:client_conn;403:10:proxy_mode;7:regular;8:mitmcert;0:~19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;17:1765289527.650106^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:eb824912-4abe-41c7-ad61-3493e919df49;8:sockname;21:10:172.17.0.3;4:8080#]8:peername;22:10:172.17.0.1;5:40266#]}5:error;0:~2:id;36:eade6cef-6c8b-47ac-ae9e-704a38a0fbe2;4:type;4:http;7:version;2:21#}2284:9:websocket;0:~8:response;418:6:reason;12:UNAUTHORIZED,11:status_code;3:401#13:timestamp_end;18:1765289534.3661015^15:timestamp_start;18:1765289534.3641949^8:trailers;0:~7:content;39:{"message":"Invalid user or password"}
,7:headers;183:42:6:Server,29:Werkzeug/3.1.4 Python/3.12.12,]40:4:Date,29:Tue, 09 Dec 2025 14:12:14 GMT,]36:12:Content-Type,16:application/json,]23:14:Content-Length,2:39,]22:10:Connection,5:close,]]12:http_version;8:HTTP/1.1,}7:request;707:4:path;1:/,9:authority;0:,6:scheme;4:http,6:method;4:POST,4:port;4:1337#4:host;10:172.17.0.2;13:timestamp_end;18:1765289534.0690808^15:timestamp_start;18:1765289534.0644908^8:trailers;0:~7:content;27:username=user&password=toto,7:headers;437:26:4:Host,15:172.17.0.2:1337,]67:6:Accept,54:text/html, text/plain, text/sgml, text/css, */*;q=0.01,]44:15:Accept-Encoding,21:gzip, compress, bzip2,]24:15:Accept-Language,2:en,]20:6:Pragma,8:no-cache,]28:13:Cache-Control,8:no-cache,]75:10:User-Agent,57:Lynx/2.9.0dev.12 libwww-FM/2.14 SSL-MM/1.4.1 GNUTLS/3.7.8,]37:7:Referer,23:http://172.17.0.2:1337/,]53:12:Content-Type,33:application/x-www-form-urlencoded,]23:14:Content-Length,2:27,]]12:http_version;8:HTTP/1.0,}6:backup;0:~17:timestamp_created;18:1765289534.0656574^7:comment;0:;8:metadata;0:}6:marked;0:;9:is_replay;0:~11:intercepted;5:false!11:server_conn;452:3:via;0:~19:timestamp_tcp_setup;18:1765289534.0769682^7:address;21:10:172.17.0.2;4:1337#]19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;15:1765289534.0747^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:17ad6aff-8ecf-4aaf-909d-9833b6395c25;8:sockname;22:10:172.17.0.3;5:60374#]8:peername;21:10:172.17.0.2;4:1337#]}11:client_conn;404:10:proxy_mode;7:regular;8:mitmcert;0:~19:timestamp_tls_setup;0:~13:timestamp_end;0:~15:timestamp_start;18:1765289534.0539029^3:sni;0:~11:tls_version;0:~11:cipher_list;0:]6:cipher;0:~11:alpn_offers;0:]4:alpn;0:~16:certificate_list;0:]3:tls;5:false!5:error;0:~18:transport_protocol;3:tcp;2:id;36:22c1c372-6162-4502-8446-c3089093899f;8:sockname;21:10:172.17.0.3;4:8080#]8:peername;22:10:172.17.0.1;5:40278#]}5:error;0:~2:id;36:51d38b27-d2f7-4513-9f59-2632c3c1d49f;4:type;4:http;7:version;2:21#}