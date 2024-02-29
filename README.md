# NetworkAnalysis

# Squid 

Download opensquid-ssl

```
apt install squid-openssl
```

Genreate Certificat

```
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout /etc/squid/myCA.pem -out /etc/squid/myCA.pem
```

V2

```
openssl req -new -newkey rsa:2048 -days 999 -nodes -x509 -keyout /etc/squid/bump.key -out /etc/squid/bump.crt
openssl x509 -in bump.crt -outform DER -out bump.der
openssl dhparam -outform PEM -out /etc/squid/bump_dhparam.pem 2048
```


Generate ssl db

```
/usr/lib/squid/security_file_certgen -c -s /var/spool/squid/ssl_db -M 4MB
```

Remove Comment in conf file 

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.original
grep -v '^#' /etc/squid/squid.conf | uniq | sort > /etc/squid/squid.conf
```

add option to `/etc/squid/squid.conf`

```
sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/spool/squid/ssl_db -M 4MB
sslcrtd_children 5
ssl_bump server-first all
sslproxy_cert_error allow all
```
and add 

```
http_port 3128 ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=4MB cert=/etc/squid/myCA.pem
```

V2
```
http_port 3128 tcpkeepalive=60,30,3 ssl-bump generate-host-certificates=on dynamic_cert_mem_cache_size=20MB cert=/etc/squid/bump.crt key=/etc/squid/bump.key cipher=HIGH:MEDIUM:!LOW:!RC4:!SEED:!IDEA:!3DES:!MD5:!EXP:!PSK:!DSS options=NO_TLSv1,NO_SSLv3,NO_SSLv2,SINGLE_DH_USE,SINGLE_ECDH_USE tls-dh=prime256v1:/etc/squid/bump_dhparam.pem
```

# Suricata

```
apt install suricata
```

Update Rules :

```
suricata-update -o /etc/suricata/rules
```

Create Dummy Interface for replay PCAP in `/etc/network/interface`

```
# Create interface
/etc/systemd/network/10-dummy0.netdev

[NetDev]
Name=dummy0
Kind=dummy
# Create network file
/etc/systemd/network/0-dummy0.network

[Match]
Name=dummy0

[Network]
Address=169.254.0.1/24
```

Configure yaml `nano /etc/suricata/suricata.yaml`

```
# Configure Interface
af-packet:
  - interface: eth0

# Configure Live rule reload
detect-engine:
  - rule-reload: true
```

Test configuration file :

```
suricata -T -c /etc/suricata/suricata.yaml -v
```

Test suricata :

```
curl http://testmynids.org/uid/index.html
grep 2100498 /var/log/suricata/fast.log
```

# Arkime

# PolarProxy

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


## Install Suricata
