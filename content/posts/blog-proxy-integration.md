+++
title =  "Integration of my blog and proxy service"
tags = ["trojan", "blog","caddy"]
date = "2020-07-02"
+++


### Background
* The VPC running this blog is used as only a proxy node when I bought this server. I have tried 3 types of proxy tools: shadowsocks, v2ray and trojan, and now I run trojan only on my vpc for both security and proxy speed. Now I also running my own blog on this VPC for multi-use of this machine.

### VPC structure
![Route structure](/images/vpc.jpg)
* Trojan listen on port 443 for any https request or trojan stream, if it is a normal HTTPS request, it will proxy the stream to caddy and provide corresponding service, such as blog access.
* Caddy will listen on both port 80 and 8088, all of the stream to port 80 will be redirect to port 443, and port 8088 is used internal for trojan https proxy.

### About this blog
* My blog is a hugo based simple static site, after I pushed my code to Github, I will run the remote ssh command to pull the source code and rebuild the blog.
```bash
ssh root@zjsxsjc1001.xyz "cd /var/www/HugoBlog; git pull origin master; hugo"
```

### Caddy setting
* My http server is Caddy, a lightweight solution for simple proxy and static resource distribute. 
* Caddy is easy to configuration, for example my Caddyfile is as follow:
```nginx
:8088 {
    root /var/www/HugoBlog/public
    gzip
    browse
    #proxy /xxx 127.0.0.1:7777 {   //This is used for v2ray/trojan co-exist
    #    without /xxx
    #    websocket
    #    header_upstream -Origin
    #}
}

:80 {
    redir https://zjsxsjc1001.xyz
}
import sites/*

```
* In order to rejecting requests from outside to port 8088 , just run two iptables command:
```bash
iptables -I INPUT -p TCP --dport 8088 -j DROP
iptables -I INPUT -s 127.0.0.1 -p TCP --dport 8088 -j ACCEPT
```


### Trojan setting
* Trojan is a new proxy tool for us to avoid active probe, in the meantime, it provides nice performance for us to explore the free network.
* The simple setting is as follows, it will listen on port 443 and provide actual proxy service.

```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",
    "remote_port": 8088,
    "password": [
        "xxxxx"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "********.crt",
        "key": "********.key",
        "key_password": "",
        "cipher": "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
        "cipher_tls13": "TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
        "prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "alpn_port_override": {
            "h2": 81
        },
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "prefer_ipv4": false,
        "no_delay": true,
        "keep_alive": true,
        "reuse_port": false,
        "fast_open": false,
        "fast_open_qlen": 20
    }
}

```