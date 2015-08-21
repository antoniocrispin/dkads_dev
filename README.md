Setup
=====

# Ubuntu

TERMINAL 1
instalar las librerias y preparar el entorno virtual
:~$ sudo apt-get install postgresql-9.4
:~$ sudo apt-get install postgresql-server-dev-9.4
:~$ sudo apt-get install git
:~$ sudo apt-get install python-pip python-setuptools python-dev build-essential python-virtualenv


editar el archivo pg_hba.conf
:~$ sudo nano /etc/postgresql/9.4/main/pg_hba.conf 
ir al final del editor abierto y cambiar los metodos a trust como se indica, descomentar la linea asignada a local si lo estuviese

................................................................................
# TYPE  DATABASE        USER            ADDRESS                 METHOD
# "local" is for Unix domain socket connections only
local   all             all                                     trust  -- cambiar
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust  -- cambiar
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
#local   replication     postgres                                peer
#host    replication     postgres        127.0.0.1/32            md5
#host    replication     postgres        ::1/128                 md5
................................................................................

guardar el archivo y luego reiniciar postgres
:~$ sudo service postgresql restart

crear la bd de la app "basedb"
:~$ sudo -u postgres -- sh -c 'cd; dropdb basedb; createdb basedb'


TERMINAL 2
crear el usuario "development" y darle privilegios a la bd "basedb" creada 
:~$ sudo su postgres
:~$ cd
:~$ psql
postgres=# CREATE USER development WITH PASSWORD '12345678';
CREATE ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE "basedb" to development;
GRANT


TERMINAL 1
crear el entorno virtual y activarlo
:~$ virtualenv --distribute nombre_entorno
:~$ source nombre_entorno/bin/activate  

clonar la app "base_dev"
:~$ cd path_to_save_app
:~$ git clone https://user_name@bitbucket.org/user/base_dev.git
:~$ cd base_dev
:~/base_dev$ pip install -r requirements/main.txt

crear el esquema, ejecutar los scripts que estan el la app base_dev
:~/base_dev$ psql -f scripts/SchemaPg/1_Tables.sql basedb development
:~/base_dev$ psql -f scripts/SchemaPg/2_Constrains.sql basedb development
:~/base_dev$ psql -f scripts/SchemaPg/3_Data.sql basedb development


TERMINAL 2
verificar el esquema y el nuevo usuario creado
postgres=# \list    -- me lista las bds o usar \l
postgres=# \connect basedb -- o usar \c basedb
You are now connected to database "basedb" as user "postgres".
basedb=# \dt       -- me lista las tablas creadas
.........................................................
                 List of relations
 Schema |     Name     | Type  |    Owner    
--------+--------------+-------+-------------
 public | art          | table | development
 public | rsn          | table | development
 public | sy_role      | table | development
 public | sy_user      | table | development
 public | sy_user_role | table | development
 public | whs          | table | development
 public | whs_art      | table | development
 public | whs_tran     | table | development
(8 rows)
.........................................................
basedb=# select * from sy_user;

antes de levantar la aplicacion, cambiar las credenciales en 
\base_dev\utils\env.py , verificar db, user, passwd, host y port
y cambiar http_port si se requiere para un puerto especifico para levantar la app

TERMINAL 1
ejecutar la app "base_dev"
:~/base_dev$ python app.py


2014-04-27
Update Momoko a la version 1.1.2 para manejar las transacciones
================================================================
:~$ source nombre_entorno/bin/activate  
:~$ pip install --upgrade momoko

2014-05-24
Install pwgen para generador de passwords
================================================================
:~$ sudo apt-get install pwgen


Deployment
==========
#!/bin/bash -e

sudo -u postgres -- sh -c 'cd; dropdb basedb; createdb basedb'

##ifdef DESARROLLO
psql -f schema.sql basedb development
psql -f nueva_unidad.sql basedb development

##else
psql -f schema.sql basedb reynaldodevel
psql -f nueva_unidad.sql basedb reynaldodevel

##endif

bash cook_some_qr.sh

AUTH LOGIN
==========
reynaldomic@gmail.com
svidania


notas
(1)
si no permite hacer drop a la bd por los conecctiones activas, ejecutar el
siguiente query:
    SELECT pg_terminate_backend(pg_stat_activity.pid)
    FROM pg_stat_activity
    WHERE pg_stat_activity.datname = 'TARGET_DB'
      AND pid <> pg_backend_pid();
luego hacer el drop
url: http://stackoverflow.com/questions/5408156/how-to-drop-a-postgresql-database-if-there-are-active-connections-to-it

(2)
verificar si python es de 32 bits, en cmd tipear python
el resultado de la consola muestra, ejm:
    Python 2.7.1 (r271:86832, Nov 27 2010, 18:30:46) [MSC v.1500 32 bit (Intel)] on win32 ....
si es 32 bits como el muestra el resultado superior descargar postgresql 9.4 (Win x86-32) en
http://www.enterprisedb.com/products-services-training/pgdownload#windows
esto es para evitar conflitos con el psycopg2 que es un driver de postgresql en python
url: http://stackoverflow.com/questions/12765800/django-error-for-psycopg2-importerror-dll-load-failed/23322291#23322291


(3)
fix instalacion de postgres-9.4 en ubuntu 14.04
- http://stackoverflow.com/questions/27652545/how-do-i-install-update-to-postgres-9-4
- http://www.rogerpence.com/2015/01/02/install-postgres-9-4-on-ubuntu/
    editar o crear el archivo "pgdg.list" en la siguiente ruta: "/etc/apt/sources.list.d/"
    (http://askubuntu.com/questions/21556/how-to-create-an-empty-file-from-command-line) con el comando "touch" en linea de comandos, ejm: touch newfile.list
    
    adicionar la siguiente linea: deb http://apt.postgresql.org/pub/repos/apt/ trusty-pgdg main
    (http://askubuntu.com/questions/197564/how-do-i-add-a-line-to-my-etc-apt-sources-list) editar el file creado con el comando "gedit", ejm: sudo gedit /etc/apt/newfile.list
    
    ejecutar los siguientes comandos en la terminal:
    ..$ wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
    ..$ sudo apt-get update
    ..$ sudo apt-get install postgresql-9.4
    ..$ sudo apt-get install pgadmin3


(4)
instalacion de paquetes con la extension .deb
- http://askubuntu.com/questions/335752/how-to-install-deb-file-in-ubuntu-using-terminal
ejemplo:
    ..$ sudo dpkg -i google-chrome-stable_current_i386.deb    

(5)
extracion de paquetes con la extension .bz2
- http://askubuntu.com/questions/25961/how-do-i-install-a-tar-gz-or-tar-bz2-file
ejemplo:
    ..$ tar xvzf PACKAGENAME.tar.gz
    ..$ tar xvjf PACKAGENAME.tar.bz2


(6)
WINDOWS

************* windows
descargar el exe en http://www.postgresql.org/download/windows/
installar postgresql la version 9.4 de 32 bits si se tiene python 2.7 en 32 bits (2)
luego agregar la variable de entorno la ruta C:\Program Files (x86)\PostgreSQL\9.4\bin
install python 2.7, agregar variable de entorno en path : C:\Python27;C:\Python27\Scripts
cambiar de disco en cmd, ejemplo a disco F
..>F:
comandos en cmd, los archivos ez_setup.py y get-pip.py se encuentran en "...\base_dev\scripts"
.. >python ez_setup.py
.. >python get-pip.py
.. >pip install virtualenv
crear el entorno virtual, seleccionar una ubicacio especifica
.. >virtualenv virtualtest 
para activar el entorno virtual creado ir a la ruta
.. >cd virtualtest 
.. >cd Scripts
.. >activate
.. >cd ..
.. >cd ..
*************

editar el archivo pg_hba.conf
ir al final del editor abierto y cambiar los metodos a trust como se indica, descomentar la linea asignada a local si lo estuviese

************* windows
ir a la ruta "C:\Program Files (x86)\PostgreSQL\9.4\data" y abrirlo con editor de texto
*************

************* windows
reiniciar el servicio, tecla windows + r, tipear services.msc, buscar el servicio postgresql-9.4 y reiniciarlo
*************

************* windows
abrir la consola cmd
conectarse al psql
..>psql -h localhost -p 5432 -U postgres
ingresar el passwd para el usuario postgres
postgres=# DROP DATABASE IF EXISTS basedb; -- opcional (1)
postgres=# CREATE DATABASE basedb;
postgres=# CREATE USER development WITH PASSWORD '12345678';
postgres=# GRANT ALL PRIVILEGES ON DATABASE "basedb" to development;
*************

************* windows
ir a la ruta path_to_save_virtualenv donde se guardara los entornos virtuales
debe ser diferente a la ruta donde esta la app
..>cd path_to_save_virtualenv
..>virtualenv --distribute base_env
activar el entorno virtual
..>cd base_env
..>cd Scripts
..>activate
se activa el entorno mostrando: (base_env) ...>
ir a la ruta donde esta el app, ejm: "F:\Proyectos\base_dev"
..>cd path_to_save_app
installar las dependencias del proyecto
..>pip install -r requirements\main.txt
crear el esquema, ejecutar los scripts que estan el la app base_dev
..>psql -f scripts\schema.sql basedb development
..>psql -f scripts\nueva_unidad.sql basedb development
crear un distribuidor con un paquete de 50 uuids que vence en 3 meses
..>psql -f scripts\newDistPackage50uuids.sql basedb development
*************

ejecutar la app "base_dev"

************* windows
(base_env) ..>python app.py
*************

************* windows
activar el entorno virtual en windows para levantar la app
ir a la ruta path_to_save_virtualenv donde se guardara los entornos virtuales
...>F:
F:\>cd VirtualEnv\base_env
...>cd Scripts
...>activate
se activa el entorno mostrando: (base_env) ...>
ir a la ruta donde esta el app, ejm: "F:\Proyectos\base_dev" y ejecutar la app
...>cd F:\Proyectos\base_dev
...>python app.py
*************






