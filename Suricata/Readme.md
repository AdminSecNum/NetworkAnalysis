# Suricata

```
apt install suricata
```

Update Rules :

```
suricata-update -o /etc/suricata/rules
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
