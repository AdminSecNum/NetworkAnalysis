# Requirements

8-16 GB
8 Vcpu
350 GB

# Install Malcolm

```
apt install python3 python3-dotenv python3-yaml python3-requests docker-compose
```

```
wget https://github.com/cisagov/Malcolm/archive/refs/tags/v24.02.1.tar.gz
```

```
tar -xvf v24.02.1.tar.gz -C /srv/
```

```
cd /srv/Malcolm-24.02.1/scripts && python3 install.py
```

Configure as u want.

```
reboot now
```

With a user account in docker group :

```
cd /srv/Malcolm-24.02.1/scripts && ./start
```

Wait for this output :

```
Started Malcolm
Malcolm services can be accessed at https://x.x.x.x/
```

# Install Hedgehog Linux
