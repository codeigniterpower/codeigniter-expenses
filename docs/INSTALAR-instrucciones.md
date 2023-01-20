INSTALLACION:
============

## Requisitos

1. servidor o maquina de desarrollo Linux:
    * Debian 7, debian 8, Debian 9, VenenuX 1, Buntu 17.0.4 unicamente estas versiones
    * Php 5.4, 5.6, 7.0, 7.3 unicamente estas versiones soportadas
        * php-mysql
        * php-mcrypt
        * php-xml
    * Mysql 5.0, 5.3, 5.5, 5.7 o Percona 5.5, 5.7 o Mysql 5.0, 5.5, 5.7 unicamente estas versiones
    * git 2.0, 1.9
    * lighttpd 1.4.35 o superior no se emplea nunca apache en el proyecto.
2. red intrna o red de servidores
    * ip 192.168.10.30/dominioejemplo.com en la maquina, se usara en las instrucciones
    * cualquier ip en la de desarrollo
    * acceso a internet libre sin firewall, no es necesario para este sistema
3. este sistema deberia estar desplegado solo en dos posibles lugares:
    * servidor 192.168.10.30 - leer `server-vnz-ruices-ruices-devuan2.md` en el projecto de documentos servidores
    * servidor dominioejemplo.com intranet si se mudase, leer documento respectivo projecto de documentos servidores

## Installacion

Conectarse al servidor de desplilegue o de produccion, aue es actualemtne `192.168.10.30` 
si en en su maquina debe leer el documento de [DESARROLLO.md](DESARROLLO.md) y no este documento.

```
ssh admin@192.168.10.30 -p 19226
```

**NOTA** Las credenciales estan en el projecto de `documentos-servidor` en `server-vnz-ruices-debian9.md`

1. ir al directorio de intranet apps o crearlo en caso no existir
2. crear la base de datos de gastos
3. crear el usuario de gastos
4. crear los permisos de acceso a la base de datos
5. clonar el repositorio git, el servidor debera tener instalado git (ver documento servidores)
6. cargar el script de sql de la base de datos inicial
7. cargar el script de sql de respaldo si hay alguno (depende del caso ojo)

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

Una vez terminado cargar direccion web desde http://192.168.10.30/elsistema/elgasto

## Seguridad

Esto solo aplica a el servidor web lighttpd, y al maquina de produccion, para 
configuraciones mas detalladas DEBE USAR EL DOCUMENTO `server-vnz-ruices-ruices-devuan2.md` 
en el projecto de documentos servidores, o el de intranet.

Todo lo aqui expuesto es a modo explicativo y debe ajustarse segun el caso:

#### Acceso

**ADVERTENCIA: los siguientes directorios/archivos no pueden ser leidos
ni pueden ser accedidos desde internet:**

* assets
    * 99-elgasto.conf
* docs
* appsys
* elgastoweb
    * logs
* README.md
* elgastoweb.geany


#### Base de datos

**IMPORTANTE** la base de datos no puede ser accedida ni permitida para 
ningun otro usuario, si necesitase este debe ser solo lectura

```
sed -i -e 's/skip-external-locking/skip-external-locking\nlocal-infile=0/g' /etc/mysql/my.cnf
sed -i -e 's/skip-external-locking/skip-external-locking\nskip-name-resolve/g' /etc/mysql/my.cnf
```

Las configuracones de produccion son mas amplias, como la memoria para los joins.

#### Firewall

**CUIDADO: configurar correctametne el firewall para la configuracion de el mysql/percona:**

Estos son los comandos, el ultimo es de ejemplo, debe sustituir el dominiopor su ip, puede agregar otro similar para varias ip o dominios.

```
iptables -I INPUT -p tcp -s 0.0.0.0/0 --dport 3306 -j DROP
iptables -I INPUT -p udp -s 0.0.0.0/0 --dport 3306 -j DROP
iptables -I INPUT -p tcp -s 127.0.0.1 --dport 3306 -j ACCEPT
iptables -I INPUT -p tcp -s localhost --dport 3306 -j ACCEPT
iptables -I INPUT -p tcp -s elgasto.com --dport 3306 -j ACCEPT
```

El iptables es sensible al orden, ordenes DROP se ejecutan/van primero que las ACCEPT, cuidado!
**solo use iptables en produccion, OJO!**, el usuario es `elgasto`, y no puede crear esquemas, 
por ende deben existir siempre, al recrear o trabajar en desarrollo, la clave esta definica 
en el passmanager o en el proyecto.

## vease tambien

* [DESARROLLO.md](DESARROLLO.md)
