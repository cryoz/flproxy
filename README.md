# flproxy
## Flibusta proxy in docker

Based on TorPrivoxy concept: \
https://github.com/avpnusr/torprivoxy \
with tor-relay-scanner: \
https://github.com/ValdikSS/tor-relay-scanner

## Usage

```
docker compose up -d
```
### Reverse proxy config (nginx)

```
upstream flproxy {
    server 127.0.0.1:8120;
}

server {
    listen 80;
    server_name flibrary.example.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name flibrary.example.com;
    ssl_certificate /etc/nginx/certs/fullchain;
    ssl_certificate_key /etc/nginx/certs/key;
    client_header_timeout 60;
    client_body_timeout 60;
    send_timeout 60;
    keepalive_timeout 60;

    location / {
        proxy_pass http://flproxy;
        proxy_buffering off;
    }
}
```
### Reverse proxy config (Caddy)
```
# if required to run proxy without ssl
http://flibrary.example.com {
    reverse_proxy 127.0.0.1:8120
}

# default SSL with LetsEncrypt certs acquired automagically
flibrary.example.com {
    reverse_proxy 127.0.0.1:8120
}
```
