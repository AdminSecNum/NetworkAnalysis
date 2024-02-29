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
openssl x509 -in /etc/squid/bump.crt -outform DER -out /etc/squid/bump.der
openssl dhparam -outform PEM -out /etc/squid/bump_dhparam.pem 2048
```


Generate ssl db

```
/usr/lib/squid/security_file_certgen -c -s /var/spool/squid/ssl_db -M 20MB
```

Remove Comment in conf file 

```
mv /etc/squid/squid.conf /etc/squid/squid.conf.original
grep -v '^#' /etc/squid/squid.conf | uniq | sort > /etc/squid/squid.conf
```

add option to `/etc/squid/squid.conf`

```
sslcrtd_program /usr/lib/squid/security_file_certgen -s /var/spool/squid/ssl_db -M 20MB
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

## Allow Website

```
acl whitelist dstdomain .google.com
http_access allow whitelist
```

or in a file

```
acl whitelist dstdomain "/etc/squid/sites.whitelist.txt"
```

## Apply Configuration

```
squid -k reconfigure
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
# PolarProxy

https://www.netresec.com/?page=Blog&month=2020-01&post=Sniffing-Decrypted-TLS-Traffic-with-Security-Onion

Prepare Requirement

```
adduser --system --shell /bin/bash proxyuser
mkdir /var/log/PolarProxy
chown proxyuser:root /var/log/PolarProxy/
```

Create Dummy Interface


Add a service in `/etc/systemd/system/dummy.service`

```
[Unit]
Description=Create dummy interface

[Service]
Type=simple
ExecStart=/bin/sh -c 'ip link add decrypted type dummy && ip link set decrypted arp off up'
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```
Start Service

```
systemctl enable PolarProxy.service
systemctl start PolarProxy.service 
```

Download and install PolarProxyService

```
mkdir /srv/PolarProxy && cd /srv/PolarProxy
curl https://www.netresec.com/?download=PolarProxy | tar -xzf -
cp PolarProxy.service /etc/systemd/system/PolarProxy.service
chown -R proxyuser:root /srv/PolarProxy/
```

Edit /etc/systemd/system/PolarProxy.service 
Modfify the WorkingDir & ExecStart
add After=dummy.service
and add "--pcapoverip 57012" at the end of the ExecStart command. 

Start Service

```
systemctl enable PolarProxy.service
systemctl start PolarProxy.service 
```

if bug change user to root and 

```
systemctl daemon-reload
```

Install TCPReplay

```
apt install tcpreplay
```

Launch the replay with 

```
nc localhost 57012 | tcpreplay -i decrypted -t -
```

Create TCPReplay Service

in `/etc/systemd/system/tcpreplay.service`

```
[Unit]
Description=Tcpreplay of decrypted traffic from PolarProxy
After=PolarProxy.service

[Service]
Type=simple
ExecStart=/bin/sh -c 'nc localhost 57012 | tcpreplay -i decrypted -t -'
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
```

Start Service

```
systemctl enable tcpreplay.service
systemctl start tcpreplay.service
```

Configure firewall

```
ufw allow 22/TCP
ufw allow in 10443/tcp
ufw allow in 10080/tcp
ufw enable
ufw reload 
```

Connfigure proxy
https://www.netresec.com/?page=PolarProxy

You can Add to the top of `/etc/ufw/before.rules`
<!> Debug 
```
*nat
:PREROUTING ACCEPT [0:0]
-A PREROUTING -i enp0s3 -p tcp --dport 443 -j REDIRECT --to 10443
COMMIT
```


Configure the client

Get the polarcert and install it :

```
http://<IP>:10443/
```

add the proxy to client `<IP>:10443`

Configure suricata to check interface `decrypted`

and test it :

```
curl --insecure --connect-to www.testmynids.org:443:127.0.0.1:10443 https://www.testmynids.org/uid/index.html
```

Check in `/var/log/suricata/fast.log`

`[**] [1:2100498:7] GPL ATTACK_RESPONSE id check returned root [**] [Classification: Potentially Bad Traffic] [Priority: 2]`


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


## Install Suricata
