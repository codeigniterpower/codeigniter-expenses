# Como comenzar
===========================

Este documento le indicara instrucciones de como comenzar desarrollar y que usar en este proyecto

* [Como comenzar a trabajar](#como-comenzar-a-trabajar)
 * [1 Requisitos para trabajar](#1-requisitos-para-trabajar)
 * [2 Configurar tu entorno](#2-configurar-tu-entorno)
 * [3 clonar las fuentes](#3-clonar-las-fuentes)
 * [4 Cargar en Geany y ver en web](#4-cargar-en-geany-y-ver-en-web)
 * [5 Inicializar la base de datos](#5-inicializar-la-base-de-datos)
* [Estructura de desarrollo](#estructura-de-desarrollo)
 * [Modelo de datos y base de datos](#modelo-de-datos-y-base-de-datos)
 * [Codigo y fuentes](#codigo-y-fuentes)
 * [Querys SQL](#querys-sql)
 * [codigo PHP](#codigo-php)
 * [Como trabajar git](#como-trabajar-git)
* [Logica aplicacion web](#logica-aplicacion-web)
 * [Inicio sesion y modelo usuario](#inicio-sesion-y-modelo-usuario)
 * [empleo de Grocery Crud y sybase](#empleo-de-grocery-crud-y-sybase)
 * [Envio de correos](#envio-de-correos)
 * [Trabajar local webserver desde home](#trabajar-local-webserver-desde-home)


## Como comenzar a trabajar
---------------------------

***Crear un directorio `Devel` en home, cambiarse a este y alli 
clonar el repo, iniciar y arrancar el editor Geany.***

Todo esto se explica en detalle a continuacion por partes

Diseño y tecnicos de la base de datos leer [../elgastodb-README.md](../elgastodb/elgastodb-README.md)

### 1 Requisitos para trabajar

* sistema linux soportado: Debian: 7, 8, 9; Buntu 17, 21; VenenuX 7, 8, 9; Devuan 1, 2, 5
* crear un usuario general y usar este como el usuario principal en su pc de desarrollo
* git (manejador de repositorio y proyecto) `apt-get install git git-core giggle`
* mysql (manejador y servidor DB que hara de pivote) `apt-get install mysql-client mysql-server` (no hacer si tiene percona)
* odbc, myodbc, freetds (coneccion DB mysql, ODBC para sybase y mssql) `apt-get install tdsodbc`
* geany (editor para manejo php asi como ver el preview) `apt-get install geany geany-plugin-webhelper`
* lighttpd (webserver localmente para trabajar el webview) `apt-get install lighttpd`
* php (interprete) en debian/buntu `apt-get install php-cgi php-mysql php-odbc php-gd php-mcrypt php-curl`
* curl (invocar urls) `apt-get install curl`

Se recomienda usar mysql-workbench con `apt-get install mysql-workbench` para carga y trabajo con data sql.

* **NOTA 1**: mysqlworkbench solo esta en debian 7,8 y 9, buntu 17. En debian 10/11 usar repos viejos
* **NOTA 2**: php5 solo esta en debian 7,8 por ende usar repos venenux, devuan usar repos trinity
* **NOTA 3**: se asume que en su mysql su usuario root tiene clave root, sino coloque la clave
* **NOTA 4**: debe tener su maquina en linux usando el documento de https://proyectos.tijerazo.net/soporte/manuales-maquinas-linux

### 2 Configurar su entorno

**IMPORTANTE** ejecute cada bloque de comandos separado por una linea en blanco, 
**cada linea en blanco es y separa otro lote de comandos a ejecutar** al mismo tiempo!


configurar el servidio web y php de su maquina, ejecutar los **siguientes comandos como root** asi:

```
mkdir /etc/skel/Devel
for i in $(ls /home);do mkdir /home/$i/Devel;done
for i in $(ls /home);do chown -R $i:$i /home/$i/Devel;done
for i in $(ls /home);do chmod 777 /home/$i/Devel;done
for i in $(ls /home);do find /home/$i/Devel/ -type f -exec chmod 664 {} ";";done
for i in $(ls /home);do find /home/$i/Devel/ -type d -exec chmod 775 {} ";";done
for i in $(ls /home);do touch /home/$i/Devel/.keep;done
for i in $(ls /home);do chmod 555 /home/$i/Devel/.keep;done

sed -s -i -r "s|.*UserDir.*_.*|\tUserDir Devel|g" /etc/apache2/mods-available/userdir.conf
sed -s -i -r "s|.*<Directory.*home.*|\t<Directory /home/*/Devel>|g" /etc/apache2/mods-available/userdir.conf
sed -s -i -r "s|.*php_admin_value engine.*|\t#php_admin_value engine On|g" /etc/apache2/mods-available/php*.conf
/usr/sbin/a2enmod userdir usertrack lua md dir
/usr/sbin/service apache2 restart

sed -s -i -r 's|.*userdir.path.*|userdir.path         = "Devel"|g' /etc/lighttpd/conf-enabled/10-userdir.conf
/usr/sbin/lighty-enable-mod fastcgi-php cgi dir-listing fastcgi userdir usertrack
/usr/sbin/service lighttpd restart
```

configura el usuario git y directorio de desarrollo, ejecutar **estos comandos como usuario general** asi:

```
git config --global status.submoduleSummary true
git config --global diff.submodule log
git config --global fetch.recurseSubmodules on-demand
git config --global http.sslVerify false
```

configurar su usuario y clave de intranet, tenga cuidado cambiar "apellido_nombre" por el suyo como general asi:

```
git config --global user.email apellido_nombre@tijerazo.net
git config --global user.name apellido_nombre
```

### 3 clonar las fuentes y cargar la base de datos

Se usa git para tener las fuentes y se arranca el IDE geany para codificar, 
ejecute como usuario `general` de su pc, debe clonar las fuentes en Devel de home:

``` 
mkdir -p ~/Devel
cd Devel
git -c http.sslVerify=false clone --recursive http://proyectos.tijerazo.net/elsistema/elgasto
```

aqui pedira clave y usuario intranet, despues sigua con estos comandos como usuario normal:

```
cd elgasto
git pull
git submodule init || true
git submodule update --rebase || true
git submodule foreach git checkout master || true
git submodule foreach git pull || true
```

**IMPORTANTE** Asumiendo que `apellido_nombre` (su usuario intranet) tiene acceso al repo.

### 4 Inicializar la base de datos

Ejecutar sea como usuario `general` o como usuario root estos comandos:

```
mysql -h 127.0.0.1 -u root -proot -v -e "CREATE DATABASE IF NOT EXISTS elgastodb COLLATE 'utf8_general_ci';"

cd ~/Devel/elgasto/elgastodb

mysql -h 127.0.0.1 -u root -proot elgastodb < elgastodb.sql || true
```

Si lo desea puede cargar el archivo de produccion de ejemplo asi:

```
mysql -h 127.0.0.1 -u root -proot elgastodb < elgastodb-20221014000000.sql
```

**NOTA IMPORTANTE** esto es asumiendo que su base de datos esta configurada como se indico, 
si no es asi debe ejecutar los pasos documentados en el proyecto https://proyectos.tijerazo.net/soporte/manuales-maquinas-linux


### 5 Cargar en Geany y ver en web

* abrir el geany
    * ir a menu->herramientas->admincomplementos
    * activar webhelper(ayudante web), treebrowser(visor de arbol) y addons(añadidos extras)
    * activar vc y gitchangebar para poder trabajar con el repo git
    * aceptar y probar el visor web (que se recarga solo) abajo en la ultima pestaña de las de abajo
* en el menu proyectos abrir, cargar el archivo `Devel/elgasto/elgasto.geany` y cargara el proyecto
    * en la listado seleccionar el proyecto o el directorio `~/Devel/elgasto`
* depsues abajo ubicar en las pestañas la vista web que es un mininavegador
    * cargar abajo en la ultima pestaña de webpreview la ruta http://127.0.0.1/Devel/ y visitar elgasto

**NOTA IMPORTANTE** esto es asumiendo que su usuario se llama `general` y 
si no es asi debe ejecutar los pasos documentados en el proyecto https://proyectos.tijerazo.net/soporte/manuales-maquinas-linux

# Estructura de desarrollo
===========================

El sistema central tiene una interfaz web, por ahora construida con `PHP/codeigniter`, 
en futuro con `GAMBAS`, tiene una tabla de usuarios la cual en futuro sera solo 
una tabla fake de acceso, ya que la autenticacion se hara contra intranet.

El sistema automaticamente genera un menu basado en la cantidad de "controladores" 
por directorios (cada uno de ellos se asume ser un modulo del sistema). Es decir 
**el menu es construido automatico basado en los controladores que existen**.

## Codigo y fuentes

El directorio [elgastoweb](elgastoweb) contiene el codigo fuente del sistema, 
se trabajara SQL con percona y PHP con framework codeigniter2 y se maneja con GIT, 
abajo se describe cada uno y como comenzar de ultimo.

En el directorio [elgastodb](elgastodb) esta el archivo `elgastodb.sql` cargar 
esto en el servidor localhost de la maquina instalado en "localhost" y especificar o 
corregir la conexcion en el archivo `elgastoweb/config/database.php` del grupo correspondiente "elgastodb".
En el mismo archivo esta ya el string DNS especificado de OASIS, certificar y corregir.

### medias, Javascripts y CSS

Estos archivos van en el directorio `assets`, y en un futuro migrados a `elgastofiles` 
en donde estaran tanto las cargas (uploads) asi como los assets futuros.

## Modelo de datos y base de datos

El directorio [elgastodb](elgastodb) contiene el modelo, imagenes y scripts SQL, 
se usa una DB central que actualiza la tablas de usuarios y modulos, y 
se conecta a sybase para obtener los datos de reportes. (Esto en un futuro)

* base de datos MySQL/MariaDB, Sybase. Se emplea MySQL solo para pintar los reportes en tablas al vuelo.
* modelado de datos en mysqlworkbench formato script STANDARD usuario no especificado
* formato tablas en pares cabecera/detalle como maximo la tabla detalle incluye el nombre cabecera separado por `_`
* formato columnas es `<tip>_<nombre>` donde tipo puede ser cod, mon, ind, des y nobmre autodescriptivo
* llave primaria es `cod_<nombretabla>` en caso detalle la separacion de nombretabla por `_` se omite
* no hay llaves foraneas, integracion de los datos viene data por la aplicacion, puesto se maneja otras db
* no hay llaves foraneas, permitiendo la manipulacion de los datos para modularizacion y flexibilidad
* todos los campos con comentario no mayor a 40 caracteres, sqlite no admite COMMENT y Sybase lo hace por separado.

Para iniciar una conexcion en un php dentro del framework sera asi en un controlador, vista o modelo:

``` php
	$dbmy = $this->load->database('oasisdb', TRUE);
	$driverconected = $dbmy->initialize();
	if($driverconected != TRUE)
		return FALSE;
	$queryprovprod = $dbmy->query("SELECT * FROM tabla");
	$arreglo_reporte = $queryprovprod->result_array();
```

**IMPORTANTE** este codigo y las consultas deben realizarse 
en archivos php en el directorio `elgastoweb/model` pero 
el framework permite que dicho codigo se ejecute en cualquier lado.

### Querys SQL

* No usar `TOP X` ni algun otro SQL, si usase, encapsular dentro de procedimientos almacenados (sybase/mssql)
* `COALESCE` = `IFNULL` ya que actua distinto, se debe usar  `COALESCE` que verifica null
* `GROUPO_CONCAT` es solo mysql realiza referencia cruzada, pero no funciona en columnas de igual nombre

### Codigo PHP

Se emplea Codeigniter 2 y no 3, se describe mas abajo como iniciar el codigo, 
el empleo de el codeigniter 2 es porque es uan aplicacion heredada, 
se describe como funciona aqui:

* **elgastoweb/controllers** cada archivo representa una llamada web y determina que se mostrara
* **elgastoweb/views** aqui se puede separar la presentacion de los datos desde el controller
* **elgastoweb/libraries** toma los datos y los amasa, moldea y manipula para usarse al momento o temporal
* **elgastoweb/models** toma los datos y los amasa, modea y prepara para ser presentados o guardados

Para establecer una equivalentea "elgastoweb" es lo mismo que el directorio "applications" de 
el codeigniter, y el directorio "appsys" el lo mismo que el directorio "system" de codeigniter.

### Modulos y Menu automatico

Los **Modulos** seran sub directorios dentro del directorio de controladores, 
cada sub directorio sera un modulo del sistema, y dentro cada clase controller 
sera una llamada web url, ademas de los que ya esten en el directorio `elalmacenwebweb/controllers` 
que tambien seran una llamada web url.

El **Menu** sera automaticamente construido a partir de los subdirectorios y controladores, 
hay dos niveles de menu, el menu principal que es todo lo de primer nivel (directorios y los index) 
y el menu de cada modulo, que se construye pasando el nombre del subdirectorio (solo los controlers).

En el directorio `elgastoweb/controllers`, para todo archivo que tenga en el nombre "index" 
sera incluido en el menu principal, adicional a todo subdirectorio, el resto de archivos, asi 
como los archivos despues de dicho primer nivel no seran incluidos para generar el menu principal.
Para el sub menu, segun el nombre el modulo (subdirectorio) de `elgastoweb/controllers`, 
se buscara todo archivo controller y sera incluido en la generacion de el submenu, y este se 
muestra debajo del menu principal.

## Como trabajar con git

El repositorio principal "elgasto" contine adentro el de codeingiter, de esta forma si se actualiza, 
si tiene contenido nuevo, hay que primero traerlo al principal, 
y despues actualizar la referencia de esta marca, entonces el repositorio principal tendra los cambios marcados.

**POR ENDE**: los commits dentro de un submodulo son independientes del git principal

1. primero debe **"coordinar" el repo git**, esto es mantenerlos actualizados con fetch y pull
2. segundo haga **"coordinar" estos submodulos tambien**, por si cada uno tiene alguna actualizacion
3. despues debe **check y pull en los submodulos antes** de hacer commit y push en el principal
4. ya entonces **con todo al dia, puede editar archivos** para trabajar en el desarrollo
5. entonces terminado de editar realize **adicion al repo de estos cambios**
6. despues **haga commit de estos cambios para registrarlos** en el historial del repo
7. y **push hacia el repo remoto para que otros** asi puedan tambien coordinar los cambios

``` bash
git fetch && git pull

git submodule init && git submodule update --rebase

git submodule foreach git checkout master && git submodule foreach git pull

editor archivo.nuevo # (o abres el geany aqui y trabajas)

git add <archivo.nuevo> # agregas este archivo nuevo o mejor en el geany le das commit y hace todo

git commit -a -m 'actualizado el repo adicionado <archivo.nuevo> modificaciones'

git push
```

En la sucesion de comandos se trajo todo trabajo realizado en los submodulos 
y actualiza "su marca" en el principal, despues que tiene todo a lo ultimo coordinado 
se edita un archivo nuevo y se acomete

**IMPORTANTE** Geany debe tener los plugins addons y filetree cargados y activados 
en caso contrario debe leer y hacer lo que esta en el documento de https://proyectos.tijerazo.net/soporte/manuales-maquinas-linux


# Logica aplicacion web
---------------------------

## Inicio sesion y modelo usuario

Este proyecto emplea un esquema de migracion hibrida, se emplea una db base mas no central, donde se hace 
pivote de usuario, acceso y acciones, despues se conecta a otras db remotas pór odbc para presentar datos.

En cada controlador solo se debe usar el "checku" y este se encargara de sacar o no de sesion 
a el usuario si no ha iniciado sesion, esto es todo, no hay que realizar mayores verificaciones.

En la tabla `usuario` se lista usuario y clave, pero su acceso se define realmente por `yan_usuario_modulo` 
que define a donde puede ir, cada entrada de modulo es un directorio de controlador que puede invocar, 
cada controlador es una presentacion de datos especifica, por ende si no esta listada en la tabla de 
los modulos no puede ser visitada, adicional si no esta en la de relacion de usuario-modulo tampoco.

**ESTA LOGICA SERA CAMBIADA A UN QUE EMPLEARA ACCESO IMAP Y CHECK DE MODULOS**

### Core YA_Controler

Ete es una clase que se implementara en un futuro.

Inicializa objetos de verificacion de sesion, modelo y libreria de usuario, y lo hace disponible.
Todos los controladores de lso modulos deben heredar de este, para poder facilmente ovidarse de 
verificaciones de sesion o de usuario.

* checku: revisa la sesion actual, empleando la libreria que a su vez emplea el modelo, si invalido, redirige login.
     * @access	public
     * @return	void

* render: pinta igual que el CI view, solo que este antepone el header y pone despeus el footer
     * @access	public
     * @param	array/string  Si string, el nombre de la vista a cargar, si array cada vista se carga en secuencia
     * @param	array     $data a pasar a las vistas tal comolo hace CI
     * @return	void

Ninguna de las funciones de este retorna valores, porque este controlador altera el flujo segun las credenciales.

**IMPORTANTE** la clase login debe hacer dos verificaciones una contra la intranet donde esta la clave, 
y otra contra la db local donde esta el usuario, si el usuario esta en intranet entra, pero si no esta en 
la db no puede ver nada, aunque entre. Esto es para facilitar el cambio de claves y separar permisos 
de manera descentralizada, mientras que la clave y acceso lo define intranet desde gerencia y nomina.

### Libreria Login

Se encarga de el transporte de informacion y datos entre la representacion de los datos y el manejo de acceso.
La libreia usa el modelo yan_usuario para verificar las credenciales,y toda operacion de acceso de datos.

* userlogin: validation user credentials, instancia el modelo yan_usuario y verifica las credenciales en db
     * @access	public
     * @param	string    usuaername o ficha
     * @param	string    userclave
     * @return	bool      TRUE o -1 si credenciales validas

* userlogout: destruye la sesion e invalida los datos en la db
     * @access	public
     * @param	string    usuaername o ficha
     * @return	void

* usercheck: check session user, if user are null check if there a currentl loged in user in true
     * @access	public
     * @param	string    usuaername o ficha, si no se provee detecta el actual
     * @return	integer   0/FALSE si el usuario no es ya valido

* userpass actualiza la clave en la tabla particular de esta base de datos. TAMBIEN LA ACTUALIZA OTRAS APPS
    * @access  public
    * @param  string Username
    * @param  string older passowrd
    * @param  string newer password
    * @return integer 0 si no se pudo o usuario invalido

### Modelo yan_usuario

Interactua con al libreria login para abstraer un usuario de la base de datos y maneja la informacion con la libreia.
La libreria usa el modelo para obtener datos y verificarlos asi como cambiarlos, el inicio de sesion es 
mediante la verificacion de los datos usando este modelo.

* getusuario: verifica si el usuario es valido con la clave provista en md5
     * @access	public
     * @param	string  username
     * @param	string  userclave
     * @param	string  credential(md6 de usuario y clave juntos)
     * @return	boolean TRUE si datos son validos

* udpusuario: actualiza data del usuario segiun filtro, si campos desconocidos en filtro falla
     * @access	public
     * @param	array  datauser campos de la tabla con sus valores
     * @param	string  filter  parte "where" del query con los filtros
     * @return	boolean TRUE si actualizacion exitosa

* getuserinfo: retorna informacion de un usuario especifico, solo campos visibles no sensibles
     * @access	public
     * @param	string  username
     * @return	array('cuandos'=>integer,...) arreglo de un arreglo conlos datos del usuario, incluyendo clave


# Modulos

IMPORTANTE: (WIP) la tabla de modulos se actualiza sola en cada request.

En los directorios de vistas, modelo y controladores hay subdirectorios, cada uno de estos 
representara un modulo y cada uno abordara una funcionalidad especifica de la logica de la app.

### empleo de Grocery Crud y sybase

Grocery Crud solo sirve en mysql, para poder pintar reportes desde otras db, 
se emplea esa db base y tablas dinamicas:

1. se realiza el query que ofrece el reporte, 
2. estos datos se guardan en una tabla creada al vuelo
3. el nombre de la tabla tiene como sufijo el nombre del usuario y un numero aleatorio
4. se indica al GC que pinte el reporte desde esta tabla
5. no se coloca uan isntruccion que borre la tabla al final, esta se coloca al principio

Obviamente esto significa dos cosas:

* bajo rendimiento, pero no importa ya que estos reportes son solo accedidos por pocos.
* deja siempre una o dos tablas de usuario basura creadas..


### envio de correos

Cada modulo al presentar debe poder enviar por correo la descarga de CSV, lo correcto para no preocupar 
por asincronia es:

1. verificar si el host de correo responde
2. preparar el correo
3. enviar el correo

Este ultimo paso no es seguridad que se envio, pero si el primero no e da se debe sacar un mensaje 
diciendo que no hay internet rapido en estos momentos.

