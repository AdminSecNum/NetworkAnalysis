# Squid 

Download opensquid-ssl

```
apt install squid-openssl
```

Genreate Certificat

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
grep -v '^#' /etc/squid/squid.conf.original | uniq | sort > /etc/squid/squid.conf
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
