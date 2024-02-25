# NetworkAnalysis

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
auto dummy0
iface dummy0 inet static
    address 169.254.0.1/24
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

# Arkime

# PolarProxy

# Analys Encrypt HTTPS traffic

## Create a Ngninx Proxy

```
server {
    listen 443;
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
