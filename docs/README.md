# Documentación práctica NGINX

Práctica completa de despliegue de servidor web NGINX utilizando contenedores Docker. Esta práctica incluye la configuración de un virtual host, gestión de contenedores y despliegue de un sitio web estático.

## Instalación servidor web NGINX

Para instalar el servidor nginx en nuestra máquina Debian actualizaremos los repositorios e instalaremos el paquete nginx.

Una vez instalado el paquete nginx, comprobaremos que está funcionando correctamente.

![Instalación nginx](./images/actualizar-instalar-nginx.png)

## Creación de las carpetas del sitio web

Igual que ocurre en Apache, todos los archivos que formarán parte de un sitio web que servirá nginx se organizarán en carpetas. Estas carpetas, típicamente están dentro de /var/www.

Vamos a crear la carpeta de nuestro sitio web o dominio, y dentro de esa carpeta clonaremos un repositorio:

![Crear carpeta y clonar repositorio](./images/clonar-git.png)

Una vez hecho esto, haremos que el propietario de la carpeta y todo su contenido sea "www-data", y le daremos los permisos adecuados para que no nos de un error al entrar en el sitio web:

![Propietario y permisos](./images/dar-permisos.png)

Ahora comprobaremos que el servidor está funcionando y sirviendo páginas correctamente accediendo desde el cliente mediante la IP de nuestra máquina:

![Instalación nginx](./images/comprobar-servidor.png)

## Configuración de servidor web NGINX

En Nginx hay dos rutas importantes. La primera de ellas es "sites-available", que contiene los archivos de configuración de los hosts virtuales o bloques disponibles en el servidor. Es decir, cada uno de los sitios webs que alberga el servidor. La otra es "sites-enabled", que contiene los archivos de configuración de los sitios habilitados, es decir, los que funcionan en ese momento.

Dentro de "sites-available" hay un archivo de configuración por defecto, que es la página que se muestra si accedemos al servidor sin indicar ningún sitio web o cuando el sitio web no es encontrado en el servidor (debido a una mala configuración por ejemplo). Esta es la página que nos ha aparecido en el apartado anterior.

Para que Nginx presente el contenido de nuestra web, es necesario crear un bloque de servidor con las directivas correctas. En vez de modificar el archivo de configuración predeterminado directamente, crearemos uno nuevo con el siguiente código.

![Archivo de configuración](./images/contenido-site-available.png)

Aquí la directiva root debe ir seguida de la ruta absoluta dónde se encuentre el archivo "index.html" de nuestra página web, que se encuentra entre todos los que habéis descomprimido.

Y crearemos un archivo simbólico entre este archivo y el de sitios que están habilitados, para que se dé de alta automáticamente.

![Dar de alta automáticamente](./images/dar-de-alta.png)

Para aplicar la configuración tendremos que reiniciar el servidor nginx:

![Reiniciar servidor](./images/reiniciamos-servidor.png)

### Comprobaciones

Si no poseemos un servidor DNS que traduzca los nombres a IPs, debemos hacerlo de forma manual. Se puede realizar de dos maneras:
• Editando el archivo hosts de nuestra máquina anfitriona
• Usando un servicio como nip.io o xip.io que hacen la traducción automáticamente.

En mi caso lo voy a hacer de la primera forma, vamos a editar el archivo "/etc/hosts" de nuestra máquina anfitriona para que asocie la IP de la máquina virtual, a nuestro "server_name". Este archivo, en Linux, está en /etc/hosts y en Windows: C:\Windows\System32\drivers\etc\hosts. Y deberemos añadirle la siguiente línea al archivo:

![Editar archivo hosts](./images/archivo-hosts.png)

Cada uno pondrá la IP de su máquina virtual.

![Comprobar el nombre](./images/alejandro.test.png)

Comprobaremos que las peticiones se están registrando correctamente en los archivos de logs, tanto las correctas como las erróneas:

![Comprobar logs](./images/access-log.png)

Cada solicitud a su servidor web se registra en este archivo de registro, a menos que Nginx esté configurado para hacer algo diferente.

![Comprobar logs](./images/error-log.png)

Cualquier error de Nginx se asentará en este registro.

## Instalación de Docker

Primero vamos a comprobar que tenemos instalado Docker en el sistema:

**docker --version**

Si no lo tenemos instalado, lo podremos instalar de la siguiente manera:

![Instalar Docker](./images/instalar-docker.png)

## Creación de la estructura de carpetas del sitio web

En la máquina anfitriona vamos a crear la estructura de carpetas que contendrá los archivos del sitio web y la configuración de Nginx y dentro de la carpeta "html" vamos a clonar un repositorio:

![Estructura carpetas](./images/crear-carpetas-clonar.png)

## Configuración de servidor web NGINX con Docker

Creamos un archivo de configuración de Nginx en la máquina anfitriona con el siguiente contenido:

**nano ~/nginx/example.test/conf/nginx.conf**

![Archivo configuración Nginx](./images/contenido-nginx-conf.png)

Ahora crearemos un contenedor Docker que ejecute Nginx. Este contenedor montará los archivos del sitio web desde la máquina anfitriona. Además comprobaremos que el contenedor se está ejecutando y veremos sus logs.

![Crear contenedor](./images/crear-contenedor-ps-logs.png)

## Comprobación del funcionamiento

Para comprobar que el servidor está funcionando y sirviendo páginas correctamente, accederemos desde el cliente:

![Comprobación](./images/comprobacion-correcto-funcionamiento.png)

Si queremos usar un nombre de dominio, podemos utilizar uno de los métodos vistos anteriormente.

En mi caso voy a volver a utilizar el mismo método.

Vamos a editar el archivo "/etc/hosts" de nuestra máquina anfitriona para que asocie la IP local 127.0.0.1 (o la IP de la máquina host si accedéis desde otro equipo) a nuestro nombre de dominio. Este archivo, en Linux, está en "/etc/hosts" y en Windows: C:\Windows\System32\drivers\etc\hosts. Y deberemos añadirle la línea:

![Editar archivo hosts](./images/archivo-hosts.png)

Podemos ver los logs del contenedor usando Docker:

![Comprobar logs](./images/docker-logs-f.png)

También podemos acceder a los logs dentro del contenedor:

**docker exec nginx-example cat /var/log/nginx/access.log**

Cada solicitud a su servidor web se registra en este archivo de registro, a menos que Nginx esté configurado para hacer algo diferente.

**docker exec nginx-example cat /var/log/nginx/error.log**

Cualquier error de Nginx se asentará en este registro.

## Gestión del contenedor

Podemos detener el contenedor o reiniciarlo:

![Detener/Reiniciar contenedor](./images/parar-reiniciar-contenedor.png)

Si necesitamos cambiar la configuración de Nginx, editamos el archivo "~/nginx/example.test/conf/nginx.conf" en la máquina anfitriona y luego reiniciamos el contenedor.

Si no necesitamos el contenedor y queremos borrarlo:

![Eliminar contenedor](./images/eliminar-contenedor.png)

## Usando docker-compose

Si queremos una configuración más robusta y fácil de reproducir, podemos crear un archivo "docker-compose.yml" con el siguiente contenido:

**nano ~/nginx/example.test/docker-compose.yml**

![Docker compose yml](./images/contenido-docker-yml.png)

Para ejecutar el contenedor con docker-compose:

![Ejecutar contenedor](./images/crear-contenedor-docker-compose.png)

Para ver los logs:

![Ver los logs](./images/logs-docker-compose.png)

Para detener los contenedores:

![Detener contenedores](./images/detener-docker-compose.png)