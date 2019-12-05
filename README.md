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

Instalaremos unos prerequisitos para el sistema
~~~
$ sudo yum install epel-release yum-utils
$ sudo yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
$ sudo yum-config-manager --enable remi

~~~


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
$ sudo yum install git
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

Nota: si hay problemas al momento de ejecutar este comando, intenta hacer esto.

    - $ sudo yum -y install gcc
    - $ sudo yum install gcc-c++
    - $ sudo npm install -g node-gyp
    - $ cd back-filesystem
    - $ sudo npm install

$ cd back-master && npm install
$ cd front && npm install

Nota: Si hay problemas con el comando anterior, intenta:
	
	- Entra a la carpeta front, 
	- abre con un editor de texto: /octave-online-server-master/front/package.json
	...
	"version": "0.0.0",
  
	"private": true,  <-- Agrega esta linea en el código
  	
	"description": "Front server for Octave Online",
  	...
	
	- $ npm install --no-optional --no-shrinkwrap --no-package-lock



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

A continuación vamos a construir Octave desde código funte:
 
instalamos algunoas dependencias para Octave

~~~

$ sudo yum install -y epel-release yum-utils
$ sudo yum-builddep -y octave
$ sudo yum install -y \
	mercurial \
	libtool \
	gcc-c++ \
	make \
	net-tools \
	traceroute \
	git \
	tar \
	lapack64-devel \
	librsvg2-tools \
	icoutils \
	transfig

~~~

Al construir sin --disable-docs, se requieren los siguientes paquetes adicionales:

~~~
 $ sudo yum install -y texlive-collection-latexrecommended
 $ sudo yum install -y texlive-metapost
~~~

Construye e instala libuv

descarga libuv del siguiente link: https://github.com/libuv/libuv.git

~~~
$ cd libuv 
$ sh autogen.sh 
$ ./configure 
$ make 
$ make install
~~~

Construye e instala json-c con el parche de desinfección UTF-8 personalizado

descarga json-c desde el siguiete link: https://github.com/vote539/json-c.git 
	
~~~
$ cd json-c-master
$ sh autogen.sh
$ ./configure
$ make 
$ sudo make install
~~~

Clonamos el archivo de octave directo de la pagina Mercurial

~~~
$ hg clone --insecure http://www.octave.org/hg/octave
~~~

Copiamos la carpeta oo-changesets que aparece en la carpeta back-octave de la carpeta octave-online-server-master

Entramos a la carpeta Octave, e importamos los archivos que se encuentran en la carpeta que acabamos de copiar
~~~
### 4.2.1 ###
 
 $ cd octave && \
	hg update b9d482dd90f3 && \
	hg import ../oo-changesets/100-2d1fd5fdd1d5.hg.txt && \
	hg import ../oo-changesets/101-bc8cd93feec5.hg.txt && \
	hg import ../oo-changesets/102-30d8ba0fbc32.hg.txt && \
	hg import ../oo-changesets/103-352b599bc533.hg.txt && \
	hg import ../oo-changesets/104-9475120a3110.hg.txt && \
	hg import ../oo-changesets/105-ccbef5c9b050.hg.txt && \
	hg import ../oo-changesets/106-91cb270ffac0.hg.txt && \
	hg import ../oo-changesets/107-80081f9d8ff7.hg.txt
~~~

Ejecutamos los siguientes comandos

~~~
$ cd octave
$ ./bootstrap
$ mkdir build-oo
$ cd build-oo
$ ../configure --disable-readline --disable-docs --disable-atomic-refcount --without-qt
~~~
Esta es la parte mas tardada de la instalación asi que se paciente
~~~

$ make
$ sudo make install
~~~
Instalamos algunos paquetes necesarios

~~~
$ sudo yum install -y units mpfr-devel portaudio-devel sympy patch
~~~

Puede ser necesario seguir el paquete adicional mientras se usa la distribución centos7 (junto con EPEL).
~~~
$ sudo yum install lzip qhull-devel pcre-devel gnuplot texinfo bison byacc flex\
  zlib-devel hdf5-devel fftw-devel glpk-devel libcurl-devel freetype-devel\
  blas-devel lapack-devel gcc-c++ pcre-devel\
  qrupdate-devel suitesparse-devel arpack-devel ncurses-devel readline-devel\
  gperf mesa-libOSMesa-devel fontconfig-devel fltk-devel\
  gl2ps-devel java-1.8.0-openjdk-devel qt-devel qscintilla-devel\
  bzip2-devel atlas-devel libsndfile-devel portaudio-devel GraphicsMagick-c++-devel
~~~


Ejecutamos el siguiente comando: Monkey-patch bug #42352
~~~
$ sudo touch /usr/local/share/octave/4.2.1/etc/macros.texi
~~~

Configuramos Monkey-patch json-c runtime errors PATH y LD_LIBRARY_PATH en CentOS 7
~~~
$ export PATH=$PATH:/usr/local/lib
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
~~~

-> Instale algunos paquetes populares de Octave Forge.
-> Tenga en cuenta que la instalación de sympy también implica la instalación de numpy, que es bastante grande, pero se requiere 	para el paquete simbólico, que es uno de los paquetes más populares en Octave Online.
-> Instale 5 a la vez para que sea más fácil recuperarse de los errores de compilación. Si un paquete no se instala, intente 		construir la imagen nuevamente y podría funcionar la segunda vez.
->La mayoría de los paquetes se cargan automáticamente a través de octaverc (desde la versión 4.2.1), excepto los siguientes 		paquetes que sombrean las funciones de la biblioteca central o son lentos de cargar: tsa, stk, ltfat y nan.
-> Nota: La lista de paquetes se escribe en / usr / local / share / octave / octave_packages

~~~
$ sudo yum install -y \
	units \
	mpfr-devel \
	portaudio-devel \
	sympy \
	patch
	
$ /usr/local/bin/octave -q --eval "\
	pkg install -forge control; \
	pkg install -forge signal; \
	pkg install -forge struct; \
	pkg install -forge image; \
	pkg install -forge symbolic; "
	
$ /usr/local/bin/octave -q --eval "\
	pkg install -forge io; \
	pkg install -forge statistics; \
	pkg install -forge optim; \
	pkg install -forge general; "
	
$ /usr/local/bin/octave -q --eval "\
	pkg install -forge linear-algebra; \
	pkg install -forge geometry; \
	pkg install -forge data-smoothing; \
	pkg install -forge nan; \
	pkg install -forge tsa; "
	
$ /usr/local/bin/octave -q --eval "\
	pkg install -forge financial; \
	pkg install -forge miscellaneous; \
	pkg install -forge interval; \
	pkg install -forge stk; \
	pkg install -forge ltfat; "
	
$ /usr/local/bin/octave -q --eval "\
	pkg install -forge fuzzy-logic-toolkit; \
	pkg install -forge mechanics; \
	pkg install -forge divand; \
	pkg install -forge mapping; "
~~~
Algunos paquetes no se instalan correctamente desde Octave-Forge

Descarga los paquetes desde los siguientes enlaces

~~~
https://bitbucket.org/odepkg/odepkg/get/default.tar.gz

$  hg clone https://bitbucket.org/octave-online/octave-communications
~~~

Ingresa a la carpeta a la carpeta octave-communications y ejecuta el comando 'make install'

Copia los paquetes con extension '.tar.gz' dentro de la carpeta desiganada para ello, y ejecuta los comandos
~~~
$ /usr/local/bin/octave -q --eval "pkg install odepkg.tar.gz; "
$ /usr/local/bin/octave -q --eval "pkg install communications-1.2.1.1.tar.gz; "
~~~

Si bien compilar paquetes de octave después de paquetes adicionales en centos7 puede ayudar a resolver el problema de advertencia.

configure: ADVERTENCIA: Biblioteca Qhull no encontrada. Esto dará como resultado la pérdida de funcionalidad de algunas funciones de geometría. 

## SELinux

Ejecuta
~~~
$ sudo yum install -y selinux-policy-devel policycoreutils-sandbox selinux-policy-sandbox libcgroup-tools
~~~

Asegúrese de que Node.js esté instalado y que las dependencias se descarguen para el proyecto compartido

~~~
$ cd shared && npm install
~~~

Ejecute todos los siguientes comandos make desde el directorio de proyectos.
~~~

$ sudo make install-cgroup
$ sudo make install-selinux-policy
$ sudo make install-selinux-bin
$ sudo make install-site-m

~~~

