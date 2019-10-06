# Ayuda-Octave-Server
Ayuda con la instalacion de paquetes y programas para correr octave en linea desde tu computadora
Toda la información requerida se encuentra en el siguiente link, véase : https://github.com/octave-online/octave-online-server


## Prerequisitos

Utilizaremos Centos 7 tal como se recomienda. 


#### Instalación de Node.js 6.x LTS en CentOS 7:

Utilizaremos los comandos que vienen en las instrucciones de octave-online-server

~~~
$ curl --silent --location https://rpm.nodesource.com/setup_6.x | sudo bash -
$ sudo yum makecache
$ sudo yum install nodejs
~~~

### Instalación de Redis 

Utilizaremos los comandos que vienen en las instrucciones de octave-online-server

~~~
$ sudo yum install redis

$ systemctl enable redis
$ systemctl start redis

$ sudo nano /etc/redis.conf

# Busca "notify-keyspace-events"
# Cambia el valor a: "Ex"
~~~

### Instalación de Git Server

Para instalar Git, debe tener las siguientes bibliotecas de las que depende Git: curl, zlib, openssl, expat y libiconv. 

~~~
$ sudo yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel perl-devel asciidoc xmlto
$ yum install git
~~~

### Instalación MongoDB
Crea un archivo /etc/yum.repos.d/mongodb-org-4.2.repo para poder instalar MongoDB directamente usando yum:

~~~
$ sudo nano /etc/yum.repos.d/mongodb.repo
~~~

agrega las siguientes lineas

~~~
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
~~~
Instala MongoDB
~~~
$ sudo yum install -y mongodb-org
~~~
Habilitar mongo 
~~~
$ sudo systemctl start mongod.service
$ mongo
~~~


### Creación de cuenta Mailgun
~~~
  *Crea una cuenta en https://www.mailgun.com/ 
  *entra a Settings-->Security & Users
  *Verifica tu API key
~~~
## Configuraciones

