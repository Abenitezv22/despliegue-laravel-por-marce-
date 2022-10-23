# Despliegue Laravel 1 - Máquina virtual

## 1: Crear Máquina virtual del servidor

Lo primero es hacer el montaje de una máquina virtual con sistema operativo de la familia Linux, lista para luego ser convertida en cualquier tipo de servidor.

### Crear la imagen

Instalar [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/server) en una máquina virtual.

- 1 core, 1 Giga de RAM debería ser suficiente.
- Activar el servidor de SSH
- No instalar ningún snap
- Recomiendo el nombre de usuario ```server``` y el password ```server```
- Al terminar la instalación añadir nuestra clave pública a las authorized keys del servidor para poder acceder con SSH sin contraseña para el resto de pasos
  
### Actualizar la máquina

- Para conectarnos a la máquina
  - Necesitaremos su dirección IP (aparecerá en el arranque)
  - El usuario es el que especificamos en la instalación (```server``` si seguiste mi consejo)
  - Podemos utilizar una terminal de nuestro ordenador anfitrión (el portátil físico) para conectar con el comando ```ssh server@192.168.XXX.XXX``` (la ip será la de tu máquina)
  - No olvides ni el usuario, ni la contraseña ni la ip, las utilizarás constantemente.
- Obtener las versiones más recientes de los paquetes en los repositorios con ```sudo apt update```
- Descargar las versiones nuevas de los paquetes actualizables con ```sudo apt upgrade```
  
> ES BUENA IDEA CREAR UN SNAPSHOT AQUÍ POR SI ALGO VA MAL

## 2: Instalar el software necesario

En ésta sección convertimos nuestra máquina virtual en un servidor LEMP (Linux, NGINX, MariaDB, PHP) que pueda ser usado para servir nuestras aplicaciones web.

Vamos a istalar los requisitos necesarios de producción de Laravel 9 (en caso de duda o para referencia futura, la sección "[Deployment](https://laravel.com/docs/9.x/deployment)" de la documentación oficial mantiene listados actualizados)
> Para facilitar las cosas, aquí adjunto los **nombres de los paquetes**, todos se instalan con ```sudo apt install nombre_del_paquete```
> > Nada nos inpide instalar TODOS los paquetes con un solo comando apt install, simplemente escribiendo todos los nombres separados por espacios.

### NGINX

Servidor Web/Proxy Inverso NGINX

- Paquete base: ```nginx```

### PHP

Lenguaje de programación PHP en versión 8.0 o superior.

- El paquete base de Ubuntu se llama ```php8.1-fpm``` es importante el sufijo fpm para que no instale Apache2 (vamos a usar NGINX como servidor y proxy inverso)

#### Extensiones

Instalar todas las extensiones de PHP necesarias para que
Laravel funcione.

- BCMath: ```php-bcmath```
- Ctype: Incluido en ```php8.1-fpm```
- cURL: ```php-curl```
- DOM: Incluido en ```php-xml```
- Fileinfo: Incluido en ```php8.1-fpm```
- JSON: ```php-json```
- Mbstring: ```php-mbstring```
- OpenSSL:  Incluido en ```php8.1-fpm```
- PCRE:  Incluido en ```php8.1-fpm```
- PDO:  Incluido en ```php8.1-fpm```
- Tokenizer: Incluido en ```php8.1-fpm```
- XML: ```php-xml```
- ZIP (para dusk): ```php-zip```

### MariaDB

Base de datos MariaDB

- Servidor: ```mariadb-server```
- Cliente (para entrar por consola): ```mariadb-client```
- Extensión MySQL (compatible con MariaDB): ```php-mysql```

### Gestores de Dependencias

- Composer ```composer```
- NPM y NodeJS
  - Necesitamos una versión más moderna de la incluida en Ubuntu 22.04
  - Ejecutamos el comando ```curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -``` para añadir el repositorio.
  - El paquete se llama ```nodejs```
  - NPM ```npm``` (si tenemos node >= 14 npm viene incluido)

### Comprobación de la instalación

Podemos probar que se han instalado los elementos principales de software con éxito escribiendo en la consola:

- ```nginx -v```
- ```php -v```
- ```mariadb --version```
- ```composer --verson```
- ```npm -v```

> ES BUEN MOMENTO PARA GUARDAR OTRO SNAPSHOT

## Clonar el repositorio en la máquina virtual

> En esta sección usaremos git, pero como alternativa también podemos subir el código usando un cliente de FTP como FileZilla

- git viene pre-instalado en la máquina
- Si el repositorio que queremos clonar es público solo tenemos que usar el comando git clone
- Navegamos hasta la carpeta ```/var/www/``` en el servidor usando el comando ```cd```
- Creamos la carpeta ```ecommerce-laravel.local``` como administrador con el comando ```mkdir```
- Cambiamos el propietario de la carpeta con el comando ```chown``` para poder escribir dentro sin ser root.
  - ```sudo chown $USER:$USER ecommerce-laravel.local```
- Navegamos dentro de la carpeta con el comando ```cd```
- Clonamos nuestro repositorio dentro de la carpeta actual con ```git clone https://github.com/cmo7/ecommerce-laravel-2022 .``` (usar nuestra propia URL del repositorio)
- Instalamos depenencias
  - ```composer install``` o si falla por el archivo lock, ```composer update```
  - ```npm install```
- Construimos los assets de las páginas de login, logout, etc.
  - ```npm run build```

## Configurar el entorno

Configuramos todo el software que hemos instalado en el servidor para funcionar correctamente con nuestra aplicación.

- Necesitamos un usuario y una base de datos:
- Entramos en el cliente de consola de MariaDB ```sudo mariadb```
- Creamos una base de datos nueva ```MariaDB> create database laravel;```
- Creamos un usuario y le damos todos los privilegios sobre la base de datos que acabamos de crear: ```MariaDB> grant all privileges on laravel.* to laravel@'%' identified by 'laravel';```
- Actualizamos los permisos ```MariaDB> flush privileges;```
- Salimos con ```exit```

Necesitamos un .env de producción configurado para servir la página web a usuarios finales y no para desarrollo, con las credenciales de nuestra base de datos:

``` .env
APP_NAME=ecommerce  
APP_ENV=production  
APP_KEY=  
APP_DEBUG=false  
APP_URL=http://ecommerce-laravel.local  
  
LOG_CHANNEL=stack  
LOG_DEPRECATIONS_CHANNEL=null  
LOG_LEVEL=debug  
  
DB_CONNECTION=mysql  
DB_HOST=127.0.0.1  
DB_PORT=3306  
DB_DATABASE=laravel  
DB_USERNAME=laravel  
DB_PASSWORD=laravel  
  
BROADCAST_DRIVER=log  
CACHE_DRIVER=file  
FILESYSTEM_DISK=local  
QUEUE_CONNECTION=sync  
SESSION_DRIVER=file  
SESSION_LIFETIME=120  
  
MEMCACHED_HOST=127.0.0.1  
  
REDIS_HOST=127.0.0.1  
REDIS_PASSWORD=null  
REDIS_PORT=6379  
  
MAIL_MAILER=smtp  
MAIL_HOST=mailhog  
MAIL_PORT=1025  
MAIL_USERNAME=null  
MAIL_PASSWORD=null  
MAIL_ENCRYPTION=null  
MAIL_FROM_ADDRESS="hello@example.com"  
MAIL_FROM_NAME="${APP_NAME}"  
  
AWS_ACCESS_KEY_ID=  
AWS_SECRET_ACCESS_KEY=  
AWS_DEFAULT_REGION=us-east-1  
AWS_BUCKET=  
AWS_USE_PATH_STYLE_ENDPOINT=false  
  
PUSHER_APP_ID=  
PUSHER_APP_KEY=  
PUSHER_APP_SECRET=  
PUSHER_HOST=  
PUSHER_PORT=443  
PUSHER_SCHEME=https  
PUSHER_APP_CLUSTER=mt1  
  
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"  
VITE_PUSHER_HOST="${PUSHER_HOST}"  
VITE_PUSHER_PORT="${PUSHER_PORT}"
```

- Además necesitamos configurar la key: ```php artisan key:generate```
- Para terminar realizamos las migraciones en la base de datos con ```php artisan migrate```

## Configurar NGINX y el Host Virtual

- Como se trata de una máquina en red local y sin ip pública ni dominio necesitamos crear una nueva entrada en el archivo ```hosts``` de la máquina anfitriona (nuestro portátil)
  - El archivo hosts se encuentra en distintos lugares según nuestro sistema operativo:
    - Windows: ```c:/windows/system32/drivers/etc/hosts```
    - MacOS / Linux / Otros Unix: ```/etc/hosts```
  - La entrada debería ser una linea tal que:
    - ```192.168.XXX.XXX              ecommerce-laravel.local```
  - Utilizar el nombre que hemos añadido (```ecommerce-laravel.local```) debería llevarnos a la página de bienvenida de NGINX.
- Los ficheros de configuración de sitios de NGINX (similares a los de Apache) se encuentran en ```/etc/nginx/conf.d/```
- Creamos ```ecommerce-laravel.local.conf``` en esa carpeta.
  - Podemos usar ```vim``` o ```nano``` como administrador (```sudo vim /etc/nginx/conf.d/ecommerce-laravel.local.conf```) y copiar el contenido del fichero de configuración.
  - Podemos crear el fichero con cualquier otro programa, subirlo al servidor y copiarlo a la carpeta de destino como administrador ```sudo cp ecommerce-laravel.local.conf /etc/ngnix/conf.d/```
- El contenido incial para el fichero de configuración para Laravel es:

``` nginx
server {
    listen 80;
    listen [::]:80;
    server_name ecommerce-laravel.local;
    root /var/www/ecommerce-laravel.local/public;
 
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
 
    index index.php;
 
    charset utf-8;
 
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }
 
    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }
 
    error_page 404 /index.php;
 
    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }
 
    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Permisos

Nos interesa poder:

- Servir la web con NGINX
- Poder usar Artisan con el usuario de SSH

Para ello vamos a modificar los permisos de los directorios de la aplicación segun esta estrategia:

- El directorio de nuestro proyecto es ```/var/www/ecommerce-laravel.local``` que actualmente es propiedad del usuario ```server``` pero para que todo funcione bien vamos a hacer que el grupo propietario sea  ```www-data```
  - ```sudo chown -R server:www-data /var/www/ecommerce-laravel.local```
Para que el servidor pueda funcionar necesitamos establecer los permisos correctos a los archivos y directorios:
- Todos los archivos con permisos 664 (tarda)

- ```sudo find /var/www/ecommerce-laravel.local/ -type f -exec chmod 664 {} \;```

- Todos los directorios con permisos 775 (tarda)
- ```sudo find /var/www/ecommerce-laravel.local/ -type d -exec chmod 775 {} \;```
- Cambiamos el propietario de estos directorios para que sean de NGINX y pueda escribir en ellos libremente
- ```sudo chown -R www-data:www-data /var/www/ecommerce-laravel.local/storage/```
- ```sudo chown -R www-data:www-data /var/www/ecommerce-laravel.local/bootstrap/cache/```

## Comprobamos que funciona correctamente

Reiniciamos NGINX con la nueva configuración:  ```sudo systemctl restart nginx```
[http://ecommerce-laravel.local](http://ecommerce-laravel.local)

## Tareas

- Escribe un comando apt install con todos los paquetes necesarios.
- Escribe un documento con las ordenes que se necesitan para hacer todo el trabajo de configuración del servidor, una por una, de forma que puedan ir siendo pegadas en la consola por orden y funcione.
- Localizar en que partes de la guía es importante cambiar ips o nombres de dominio por otros para poder desplegar otros servidores.
- Desplegar un segundo proyecto de Laravel en la misma máquina virtual.
