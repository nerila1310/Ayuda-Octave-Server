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
Para verificar el estatus de Redis escribimos
~~~
$ sudo systemctl status redis
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
  * Crea una cuenta en https://www.mailgun.com/ 
  * entra a Settings-->Security & Users
  * Verifica tu API key
~~~
## Configuraciones

Descarga la infraestructura de octave online server desde https://github.com/octave-online/octave-online-server

Lee config_defaults.hjson para aprender mas acerca de las configuraciones disponibles para octave online server 

Copia config.sample.hjson dentro config.hjson, y completa los datos requeridos

Su propio config.hjson es ignorado por el control de origen

~~~
# Copyright © 2018, Octave Online LLC
#
# This file is part of Octave Online Server.
#
# Octave Online Server is free software: you can redistribute it and/or modify
# it under the terms of the GNU Affero General Public License as published by
# the Free Software Foundation, either version 3 of the License, or (at your
# option) any later version.
#
# Octave Online Server is distributed in the hope that it will be useful, but
# WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY
# or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU Affero General Public
# License for more details.
#
# You should have received a copy of the GNU Affero General Public License
# along with Octave Online Server.  If not, see
# <https://www.gnu.org/licenses/>.

##########
# NOTICE #
##########
#
# For additional information about these settings, refer to
# config_defaults.json
#

# # # # # # # # # # # # #
# Common configurations #
# # # # # # # # # # # # #

redis: {
	hostname: localhost
	port: 6379
	options: {
		auth_pass: xxxxxxxxx
	}
}

mongo: {
	hostname: localhost
	port: 27019
	db: oo
}

mailgun: {
	api_key: xxxxxxxxx
	domain: localhost
}

# # # # # # # # # # # # # # # #
# Back Server configurations  #
# # # # # # # # # # # # # # # #

worker: {
	logDir: /tmp/oo_logs
}

session: {
	implementation: docker
}

git: {
	hostname: localhost
}

# # # # # # # # # # # # # # # #
# Front Server configurations #
# # # # # # # # # # # # # # # #

auth: {
	google: {
		oauth_key: xxxxxxxxx.apps.googleusercontent.com
		oauth_secret: xxxxxxxxx
	}
	easy: {
		### Make up a long random string! ###
		secret: xxxxxxxxx
	}
}

front: {
	protocol: http
	hostname: localhost
	port: 8080
	listen_port: 8080

	# Use "client/app" for debugging
	static_path: client/dist

	cookie: {
		### Make up a long random string! ###
		secret: xxxxxxxxx
	}
}

# # # # # # # # # # # # #
# Client configurations #
# # # # # # # # # # # # #

client: {
	gacode: xxxxxxxxx
}
~~~

Completaremos las configuraciones una por una

### Redis
~~~
redis: {
	hostname: localhost
	port: 6379
	options: {
		auth_pass: xxxxxxxxx
	}
}
~~~

creamos un auth_pass, con el siguiente comando
~~~
$ echo "octave-online-server" | sha256sum
~~~

Colocamos la salida del comando anterior en auth_pass

Ahora configuraremos Redis, abrimos las configuraciones de Redis
~~~
$ sudo nano /etc/redis.conf
~~~
Una vez dentro de las configuraciones buscamos 
~~~
# requirepass foobared
~~~
Descomentamos la instrucción y cambiamos la palabra "foobared" por la salida que nos mostro el comando `$ echo "octave-online-server" | sha256sum`

Después de configurar la contraseña, guarde y cierre el archivo y luego reinicie Redis:

~~~
$ sudo systemctl restart redis
~~~
Para verificar el estatus de Redis escribimos
~~~
$ sudo systemctl status redis
~~~

