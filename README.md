#Bot de Telegram de La Liga Española de Fútbol

[![Build Status](https://travis-ci.org/manuelalonsobraojos/proyectoIV.svg?branch=master)](https://travis-ci.org/manuelalonsobraojos/proyectoIV)

###Introducción

El proyecto escogido para realizar a lo largo del curso en la asignatura de infraestructura virtual, es un bot de telegram.  
En concreto el bot será de la liga española de futbol, este bot mostrará información como puden ser los resultados de la jornada, la clasificación, el pichichi, etc.., todo ello dependiendo del comando que se use.

El bot será desarrollado en Python y será alojado junto con una base de datos en un servidor en la nube tales como **c9.io ó Amazon web services**. Se utilizarán dos servidores de base de datos, en uno guardaremos las IDs de los usuarios de telegram, y en otro servidor de base de datos será guardada la información que se le mostrará a los usuarios, una vez que la información haya sido obtenido mediante la tecnica de **Web Scraping** utilizando la libreria para python **BeautifulSoup**. 


###Testeo e Integración continua

Como herramientas de construcción tengo un archivo Makefile y un requirements.txt. Con este archivo makefile podremos realizar la instalación de dependencias necescesarias para la ejecución de bot, ejecutar los test y ejecutar el bot. Todo ello con las siguientes ordenes respectivamente:  
- make install
- make test
- make ejecutar

Para la realización de los test he utilizado la libreria **unittest**, para su ejecución se ha incluido un objetivo *make test* dentro del archivo Makefile.

Como sistema de integración continua se utiliza Travis-CI, para ello he incluido un archivo .travis.yml en el repositorio, donde se configura la conexión con travis-ci y se le indica como instalar las dependencias necesarias para la ejecución del bot y como ejecutar los test.

###Despliegue en Heroku

Para realizar el despliegue en Heroku, lo he configurado para cada que cada vez que haga push a el repositorio de github donde se encuentra el bot directamente se despliegue en heroku.

Para el despliegue haremos uso de un fichero **Procfile**, este fichero deberá de encontrarse en el directorio raíz y en el se declararán los comandos que deben ser ejecutados para arrancar el bot. El fichero quedaría asi:  
```
worker: cd bot_LaLiga && python bot_LaLiga.py 
```

Además del archivo Procfile es necesario un archivo con nombre **runtime.txt** en el que le especificaremos la versión de python que se está utilizando, en este caso la versión *python-2.7.12*.

Una vez tenemos esto vamos a la página de Heroku y hacemos clic en nueva aplicación y le ponemos el nombre que deseemos.  
Una vez creada la aplicación en Heroku, nos vamos a la configuración de despliegue y vinculamos la aplicación con el repositorio de github donde se encuentra el bot. Habilitaremos el despligue automático para que cada vez que hagamos un push a github se despliegue automáticamente, además de esto habilitaremos la opción de que solo se despligue si pasa los test de travis.

![img](https://github.com/manuelalonsobraojos/IV-Ejercicios/blob/master/Ejercicios-tema3/capturas/Captura.PNG)

Con respecto al servicio de base de datos, con respectos a los anteriores hitos la he pasado PostgreSQL, ya que es uno de los servicios de base de datos que nos ofrece Heroku gratuitamente. Para crear las tablas que necesitemos deberemos acceder a dicha base de datos con el comando ```heroku logs --tail --ps postgres --app <nombre de la app>```. 

Con esto ya tendremos configurado nuestro despliegue atomático y podremos probarlo desde telegram buscando el bot por el nombre de **@La_Liga_bot**. Se puede ver que está desplegado en el Paas Heroku desde el siguiente [Enlace](https://botlaliga.herokuapp.com/) aunque mostrará un error ya que no se trata de una web. 


###Subiendo proyecto a docker

Primero instalaremos Docker mediante el comando ```wget -qO- https://get.docker.com/ | sh```.

**Creación de contenedor**

Para subir el proyecto a docker primero deberemos de crear y configurar el contenedor de docker. Para ello crearemos en nuestro repositorio un archivo Dockerfile, que contendrá todas las tareas de construcción del contenedor. En el archivo dockerfile se hará referencia a un script docker_run que instalará todo lo necesario para preparar el contenedor y llamará al archivo Makefile que instalará todos los requisitos que necesita el bot.  
  

**Subiendolo a DockerHub**

Para subir el contenerdor a DockerHub primero deberemos de [registrarnos](https://hub.docker.com/). Una vez nos hemos registrado crearemos un repositorio, en mi caso el repositorio será *manuelalonsobraojos/proyectoiv*. Le asignaremos a dicho repositorio un *tag*, que será el ID de la imagen que vamos a subir, para ello utilizaremos el comando ```docker tag 7e0247a93381 manuelalonsobraojos/proyectoiv```, una vez asignado el tag lo subimos ejecutando el comando ```docker push manuelalonsobraojos/proyectoiv```.

Para instalar el contenedor tan solo deberemos de ejecutar el comando ```docker pull manuelalonsobraojos/proyectoiv``` y el contenedor comenzará a instalarse automáticamente.  
Una vez instalado el contenedor lo arrancamos ejecutando el comando ```sudo docker run -i -t manuelalonsobraojos/proyectoiv  /bin/bash```.  
Una vez ha arrancado el contenedor deberemos ejecutar la siguiente orden para ejecutar el bot:  
```
cd /home/proyectoIV && make ejecutar 
```
El docker de mi proyecto se puede encontrar en el siguiente [enlace](https://hub.docker.com/r/manuelalonsobraojos/proyectoiv/).


###Despliegue con Vagrant y Ansible

Para crear máquinas virtuales en Azure, empezaremos con descargar e instalar **Vagrant**, lo podemos hacer desde su [página oficial](https://www.vagrantup.com/downloads.html). Una vez hemos instalado vagrant, instalaremos **Ansible**, para ello ejecutaremos el siguiente comando:  
``` 
apt-get install ansible
```
Una vez aquí instalaremos los plugins de Azure, para ello deberemos de ejecutar el siguiente comando:
``` 
vagrant plugin install vagrant-azure
``` 
Para realizar el despliegue en Azure en necesario que dispongamos de una suscripción, para la asignatura el profesor nos ha proporcionado una suscripción. Si no disponemos de una se puede conseguir mediante pago.

Una vez tenemos la suscripción deberemos de crear los certificados de servidor. Lo haremos ejecutando las siguientes ordenes:
``` 
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout azure.pem -out azure.pem
openssl x509 -inform pem -in azure.pem -outform der -out azure.cer
chmod 600 azure.pem
``` 
Una vez creados los certificados, nos loguearemos en nuestra cuenta de Azure, en este caso lo haremos en el [portal antiguo](https://manage.windowsazure.com), una vez logueados accederemos al apartado de Configuración, Certificados y Administración, y cargaremos el archivo *.cer*.  

Una vez hecho todo esto ya podemos realizar el archivo Vagrantfile. Primero ejecutaremos la siguiente orden:
``` 
vagrant init
```
Una vez hecho desarrollaremos los archivos [vagranfile](https://github.com/manuelalonsobraojos/proyectoIV/blob/master/Vagrantfile) y [configuración_ansible](https://github.com/manuelalonsobraojos/proyectoIV/blob/master/configuracion_ansible.yml) (llamado desde el **vagrantfile**), que se encargaran del despliegue.

Una vez está todo listo procederemos al despligue ejecutando la siguiente orden:
```
vagrant up --provider=azure
```

###Fabric

**Fabric** es una herramienta con la que podremos administrar una máquina remota una vez haya sido creada y aprovisionada con todo lo necesario. Para la utilización de dicha herramienta deberemos de ejecutar la orden de la siguiente forma:
```
fab -p CONTRASEÑA -H usuario@host ORDEN
```
Este comando buscará en la ubicación en la que nos encontremos actualmente un fichero fabfile que contenga la orden especificada, dicha orden conllevará unas acciones que la máquina llevará a cabo.  
Las ordenes que se podrán llevar en mi archivo [fabfile]() son:

- descargar: descarga el proyecto de el repositorio de github.
- iniciar: ejecuta el bot.
- detener: detiene el bot.
- borrar: borra el proyecto.
- testear: lanza los test del proyecto.
- instalar: instala todos los componentes necesarios para la ejecución del bot.

La ejecución del bot se hace mediante un supervisor que ejecutará el bot sin la necesidad de tener abierta la MV, para ell hemos creado un archivo de configuración que podemos ver [aquí](https://github.com/manuelalonsobraojos/proyectoIV/blob/master/laliga.conf)






