INSTALLATION:
============

## Requirements

1. server or pc with only Linux:
    * Debian 7, debian 8, Debian 9, VenenuX 1, Buntu 17.0.4 only those systems
    * Php 5.4, 5.6, 7.0, 7.3 only those varians
        * php-mysql
        * php-mcrypt
        * php-xml
    * Percona 5.5, 5.7 o Mysql 5.0, 5.5, 5.7 only
    * git 2.0, 1.9
    * lighttpd 1.4.35 dont use apache2 or nginx
2. network with internet:
    * ip 192.168.10.30/domainexample.com address, those were used in the instructions
    * internet access without firewall, this project dont need any stupid protection

## Installation

Connect to the server production, othercase use a root terminal on your computer where will be deployed:

```
ssh admin@192.168.10.30 -p 19226
```

**NOTA** credentials of server will be at `documentos-servidor` project, at `server-vnz-ruices-debian9.md` document


```
cd /home/ && mkdir -p /home/intranet/intranetapps/elsistema

mysql -h 127.0.0.1 -u root -p -v -e "CREATE DATABASE IF NOT EXISTS elgastodb COLLATE 'utf8_general_ci';"

mysql -h 127.0.0.1 -u root -p -v -e "CREATE USER 'elgasto'@'%' IDENTIFIED BY 'elgasto.clave'"

mysql -h 127.0.0.1 -u root -p -v -e "GRANT ALL PRIVILEGES ON elgastodb.* TO 'elgasto'@'%' IDENTIFIED BY 'elgasto.clave' ;"

git -c http.sslVerify=false clone https://proyectos.tijerazo.net/elsistema/elgasto && cd elgasto

mysql -h 127.0.0.1 -u elgasto -p elgastodb < /home/intranet/intranetapps/elsistema/elgasto/elgastodb/elgastodb.sql

mysql -h 127.0.0.1 -u elgasto -p elgastodb < /home/intranet/intranetapps/elsistema/elgasto/elgastodb/elgastodb-20221014144500.log

ln -s ../../../home/intranet/intranetapps/elsistema/elgasto/assets/99-elgasto.conf /etc/lighttpd/conf-available/99-elgasto.conf

/usr/sbin/lighty-enable-mod elgasto

/usr/sbin/service lighttpd restart
```

then visit http://192.168.10.30/elsistema/elgasto

#### Database

**IMPORTANTE** la base de datos no puede ser accedida ni permitida para 
ningun otro usuario, si necesitase este debe ser solo lectura

```
sed -i -e 's/skip-external-locking/skip-external-locking\nlocal-infile=0/g' /etc/mysql/my.cnf
sed -i -e 's/skip-external-locking/skip-external-locking\nskip-name-resolve/g' /etc/mysql/my.cnf
```

Las configuracones de produccion son mas amplias, como la memoria para los joins.

#### Firewall

**CUIDADO: we must recommend percona server only

```
iptables -I INPUT -p tcp -s 0.0.0.0/0 --dport 3306 -j DROP
iptables -I INPUT -p udp -s 0.0.0.0/0 --dport 3306 -j DROP
iptables -I INPUT -p tcp -s 127.0.0.1 --dport 3306 -j ACCEPT
iptables -I INPUT -p tcp -s localhost --dport 3306 -j ACCEPT
iptables -I INPUT -p tcp -s elgasto.com --dport 3306 -j ACCEPT
```

Please note that iptables are so sensible to right order, "drop" rules must be before "allow" rules.

## See also

* [DESARROLLO.md](DESARROLLO.md)
