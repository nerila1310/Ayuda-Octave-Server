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


### Mailgun
~~~
  * Crea una cuenta en https://www.mailgun.com/ 
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
	implementation: SELinux
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

### Mongo

~~~
mongo: {
	hostname: localhost
	port: 27019
	db: oo
}
~~~

Creamos la base de datos con el nombre "oo"
~~~
>mongo
>use oo
~~~

### Mailgun
~~~
mailgun: {
	api_key: xxxxxxxxx
	domain: localhost
}

~~~
Entra a Settings-->Security & Users
Verifica tu API key
Copiamos la clave Private_key en api_key



## Instalación de dependencias y construcción

En cada uno de los cinco directorios que contienen proyectos Node.js, ingrese y ejecute npm install:

~~~
$ cd shared && npm install
$ cd back-filesystem && npm install

Nota: si hay problemas al momento de ejecutar el comando, intenta
    - Remover la carpeta node-gyp: rm -R /home/userName/.node-gyp
    - Instalar manualmente la librería: node install -g node-gy

$ cd back-master && npm install
$ cd front && npm install
$ cd client && npm install
~~~
Enlace el proyecto compartido a todos los demás
Esto permite que todos los proyectos usen el código en el directorio compartido:

~~~
$ cd shared && npm link  
$ cd back-filesystem && npm link @oo/shared
$ cd back-master && npm link @oo/shared
$ cd front && npm link @oo/shared
$ cd client && npm link @oo/shared
~~~
Se necesita instalar las dependencias de Bower (del lado del cliente) para el proyecto del cliente:

~~~
$ cd client && npm run bower install
~~~

Finalmente, compile los proyectos de cliente y servidor frontal (el servidor posterior se ejecuta sin necesidad de ser construido):

 Instalando GruntJS
~~~
$ sudo npm install -g grunt-cli
~~~

~~~
$ cd client && npm run grunt
$ cd front && npm run grunt
~~~

## Back Server

A continuación vamos a construir Octave desde código funte siguiendo 
     
Vaya al directorio que contiene el código fuente del paquete y escriba './configure' para configurar el paquete para su sistema.
~~~
$ ./configure
~~~

Ejecutar 'configurar' puede llevar un tiempo. Mientras se ejecuta, imprime algunos mensajes que indican qué características está buscando. Luego compila el paquete. Opcionalmente, escriba 'make check' para ejecutar las autocomprobaciones que vienen con el paquete.

~~~
$ make
$ hacer cheque
~~~


Escriba 'make install' para instalar los programas y cualquier archivo de datos y documentación. Al instalar en un prefijo propiedad de root, se recomienda que el paquete se configure y cree como un usuario normal, y solo la fase 'make install' se ejecute con privilegios de root.
~~~
$ make install
~~~
Puede ser necesario seguir el paquete adicional mientras se usa la distribución centos7 (junto con EPEL).
~~~
$ yum instalar qhull-devel hdf5-devel fftw-devel fftw fftw-libs fftw-libs-long glpk-devel suitesparse-devel arpack-devel qt-devel ibXcursor-devel libXi-devel libXinerama-devel libXrandr-devel libXvilla-gl2 qccint -devel gl-manpages libXdamage-devel libXext-devel libXfixes-devel libXxf86vm-devel libdrm-devel libxshmfence-devel mesa-libGL-devel mesa-libGL-devel mesa-libGLU-devel
~~~

Si bien compilar paquetes de octava después de paquetes adicionales en centos7 puede ayudar a resolver el problema de advertencia.

configure: ADVERTENCIA: Biblioteca Qhull no encontrada. Esto dará como resultado la pérdida de funcionalidad de algunas funciones de geometría. 






