# NetworkAnalysis



# MitmProxy

```
apt install mitmproxy
```

```
mitmproxy
```

Set client proxy to : `<IP>:8080` and open 

http://mitm.it/

# Arkime

# Analys Encrypt HTTPS traffic

## Create a Ngninx Proxy

```
server {
    listen 443 ssl;
    server_name example.com;
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;

    location / {
        mirror /suricata_mirror;
        proxy_pass https://backend_server;
    }

    location = /suricata_mirror {
        internal;
        proxy_pass http://127.0.0.1:Suricata_port;
    }
}
```
