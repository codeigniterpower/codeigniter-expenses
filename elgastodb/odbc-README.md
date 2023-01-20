[elyanerodb-README.md](elyanerodb-README.md)

# ODBC EN general

*"Open Database Connectivity"* `ODBC`, es una especificacion abierta para llmadas SQL desde otros lenguajes de programacion.
y **en php es solo una capa de manejo de las llamadas** por lo que no espere resultados especiales.
**Esto significa que es mejor hacer la mayor parte en codigo php que con trucos sql**.

El proyecto se conecta a sistemas Sybase y SQLserver los cuales no cumplen estandares algunos

**NOTA IMPORTANTE** aqui solo se explica relacionado a UNIX/Linux para **otros OS es innecesaria la explicacion si son populares**.

## ODBC installacion y configuracion

Este proyecto solo trabaja con entornos Devuan o Debian.

`apt-get install unixodbc tdsodbc`

El DSN es el nombre de la conexcion, y se guarda en archivos "odbc.ini" 
En linux hay dos alcances de un DSN, por sistema o por usuario:

* si configura el DSN en `/etc/odbc.ini` lo vera todo el mundo.
* si configure el DSN en `~/.odbc.ini` lo vera solo el usuario.

**NOTA:** el usuario del webserver es `www-data` y este no tiene un home por seguridad, por lo que no existe.

**IMPORTANTE**: las claves son guardadas en texto plano y pasadas sin encriptar, SQLserver soporta SSL pero el ODBC no lo soporta.

### El modulo ODBC inst

Para trabajar con ODBC se requiere el "driver" modulo ODBC de **FreeTDS**, este al instalar 
se registra en `/etc/odbcinst.ini` para poder usarse como librerias de conectividad, quedando asi aproximadamente:

``` ini
  
[FreeTDS]
Description	= FreeTDS Driver for Linux using guindosers DBs
Driver		= /usr/lib/odbc/libtdsodbc.so
Setup		= /usr/lib/odbc/libtdsS.so
UsageCount	= 1
```

**IMPORTANTE** las rutas de "Driver" dependen de la version de linux Debian/Devuan si soporta multiarch.

### Configuracion DSN ODBC

Se usa la libreria de acceso `FreeTDS` registrada en el `/etc/odbcinst.ini` para la definicion del DSN, 
en el archivo `/etc/odbc.ini` con los parametros de conexcion a la base de datos 

Se muestra coneciones a db local en MySQL/MAriaDB, a SQLserver y/o Sybase:

``` ini
[dblocal]
Driver		= MySQL
Database	= mysql
Server		= localhost
Port		= 2638
ReadOnly	= No

[oasiscnx]
Driver		= FreeTDS
Database	= sybasedemo
Server		= 10.10.200.10
Port		= 2638
ReadOnly	= No
TDS_Version	= 5.0

[saintcnx]
Driver			= FreeTDS
Database		= saintnonmina
Server			= 192.168.1.100
Port			= 1723
ReadOnly		= Yes
TDS_Version		= 8.0
```

### Probando en ODBC configurado

El paquete UnixODBC tiene un programa llamado `isql` el cual se puede usar para probar, debe 
usarse desde el contexto del usuario, es decir si se configura en un "home" se debe hacer la 
llamada del programa desde el usuario y no desde otro usuario o administrador.

**IMPORTANTE** Las credenciales se deben pasar, no toma las del DSN configurado.

``` bash

% isql saintcnx sa saclave
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>select * from users
```

**IMPORTANTE**: las claves son guardadas en texto plano y pasadas sin encriptar, SQLserver soporta SSL pero el ODBC no lo soporta.

## Usar el DSN en Codeigniter

Debe definirse en el archivo `config/database.php` del directorio de aplicacion, aqui solo se muestran las 
primeras 5 lineas de una conexcion, en el archivo esta en las primeras lineas la eplicacion de como se usa
sin mayores problemas.. 

Cuando se configura el DNS el usuario y clave no sirve guardarlo, siempre debe pasarse en el string de conexcion.


``` php
$db['saintdb']['hostname'] = 'dsn=saintcnx;uid=sa;pwd=saclave';
$db['saintdb']['username'] = 'sa';
$db['saintdb']['password'] = 'saclave';
$db['saintdb']['database'] = 'XX99';
$db['saintdb']['dbdriver'] = 'odbc';

```

**ADVERTENCIA** el modulo ODBC de codeigniter esta malo, se mejora usando los cambios realizados en codeigniterpower 
desde el proyecto https://gitlab.com/codeigniterpower/codeigniterpower/tree/master/appsys/database/drivers/odbc hay 
varios commits en dicho proyecto.

