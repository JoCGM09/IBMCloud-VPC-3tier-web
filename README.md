# IBMCloud-VPC-3tier-web
## Objetivo

Construir una arquitectura [3-tier](https://www.ibm.com/cloud/learn/three-tier-architecture) de una aplicación web, que separa las capas de aplicación, web y de datos colocándolas en subredes separadas. Se elaborará el despliegue de un [WordPress](https://wordpress.com) montado sobre un [stack LAMP](https://en.wikipedia.org/wiki/LAMP) hosteado en [IBM Cloud Virtual Private Cloud](https://cloud.ibm.com/docs/vpc) (VPC).

Features:
1. Aplicación
  - Una aplicación con balanceo de carga - WordPress
  - Uso de múltiples bases de datos - HyperDB
  - Replicación de los datos - MySQL
2. Infraestructura
  - Aisalmiento de la red pública - VPC
  - Las capas de aplicación y de datos están desplegadas en diferentes subredes
  - Se presentan grupos de seguridad separados

## PASO 1: Despliegue de la arquitectura VPC

<p align="center"><img width="800" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra2.png"></p>


## Componentes funcionales de la VPC

- VPC
- Subnets
- IPs privadas
- Instancias de servidores virtuales (VSI)
- Múltiples interfaces de red en la VSI
- Balanceador de carga a nivel de aplicación
- IPv4 flotantes (solo para configuración)
- Public Gateway (solo para configuración)

### Sistemas operativos utilizados

- Servidor web y de aplicación: Ubuntu
- Servidor de datos: Ubuntu

#### Hardware seleccionado

- Servidor web de aplicación: bx2-4x8
- Servidor de datos: cx2-4x32

## Prerequisitos

1. Tener acceso a una llave pública SSH en [SSH Keys](https://cloud.ibm.com/docs/vpc?topic=vpc-ssh-keys)
2. Como buena práctica, crear un grupo de recursos, en este caso llamado `VPC1` como se describe en la [gestión de grupos de recursos](https://cloud.ibm.com/docs/resources?topic=resources-rgs#rgs)
3. Una vez creado el grupo de recursos, contar con los permisos y accesos necesarios descritos en [Managing user permissions for VPC resources](https://cloud.ibm.com/docs/vpc?topic=vpc-managing-user-permissions-for-vpc-resources)

### Despliegue de la arquitectura

Los pasos a seguir se desciben a continuación, para referencias, visitar [About VPC](https://cloud.ibm.com/docs/vpc).

1. Crear una llave SSH que será usada en una instancia virtual (VSI)
2. Crear la VPC.
3. Crear los prefijos de direcciones (CIDR) para la VPC
4. Crear las subredes
5. Elegir una imagen por defecto para crear una VSI
6. Crear y configurar el balanceador de carga
7. Crear IPs flotantes y asignarlas a una VSI
8. Agregar reglas al grupo de seguridad

## Login a IBM Cloud

Ingresa en tu navegador https://cloud.ibm.com/login e inicia sesión. 

## Grupo de recursos, regiones y zonas de disponibilidad

Los recursos serán asignados en este caso al grupo de recursos **VPC1**. Además serán alojadas en la región Dallas (*us-south*) y en la zona de disponibilidad *Dallas 1*. Para mayor información ingresar a [Creating a VPC in a different region](https://cloud.ibm.com/docs/vpc?topic=vpc-creating-a-vpc-in-a-different-region).

## Desplegar la infraestructura VPC

Seleccione Infraestructura VPC desde el menú en la esquina superior izquierda.

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra1.png"></p>

## Crear una llave SSH

Al crear una instancia de VPC se necesita una clave SSH. En el menú Infraestructura VPC, seleccione "SSH Keys" y "Crear +".  Introduzca el nombre, seleccione la región (por ejemplo, Dallas) y añada el contenido de su clave pública RSA (el contenido se ubica en el la carpeta .ssh donde fue generada la llave). Pulsa el botón "Añadir clave SSH" para guardar.

## Crea una VPC

Crea la VPC llamada `web-app-vpc`.

En el menú *Infraestructura VPC*, seleccione "VPCs" en "Red". A continuación, seleccione "Crear +". Ingrese la ubicación de la VPC, nombre, grupo de recursos.
**Deseleccionar** la casilla de `crear por defecto un prefijo para cada zona` y pulse el botón "Crear nube privada virtual".

<p align="center"><img width="600" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra3.png"></p>

## Crear los prefijos de direcciones (CIDR)

Para más información sobre prefijos de direcciones ingrese a [Understanding IP address ranges, address prefixes, regions, and subnets](https://cloud.ibm.com/docs/vpc?topic=vpc-vpc-addressing-plan-design).

Se crearán los prefijos de direcciones para `10.10.11.0/24` y `10.10.12.0/24`. En el menú *Infraestructura VPC*, selecciona "VPCs" en "Red". Seleccione la VPC `webb-app-vpc` para obtener los detalles, luego seleccione "Prefijos de dirección" y "Crear", agregar la dirección mencionada y luego repetir el proceso para la segunda.

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra4.png"></p>

## Crear dos subredes en la VPC

Se crearán dos subredes para los CIDR `10.10.11.0/24` y `10.10.12.0/24`. La capa de __*aplicación*__ usará la `subnet1` y la capa de __*datos*__ usará la `subnet2`.

En el menú *Infraestructura VPC*, seleccione "Subredes" y, a continuación, haga clic en "Crear +". Cree cada subred y adjunte la puerta de enlace pública de la VPC.

**Subnet1**
<p align="center"><img width="600" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra5.png"></p>

**Subnet2**
<p align="center"><img width="600" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra6.png"></p>

### Subredes VPC

El estado inicial de una subred recién creada es __*pendiente*__.  Debe esperar hasta que el estado de la subred esté disponible antes de asignarle recursos. Para comprobar el estado de la subred, actualice la lista de subredes.  Siga comprobando hasta que el estado sea disponible.

## Security Groups y Access Control Lists

Para los propósitos de este caso de uso, crearemos dos grupos de seguridad para los servidores de aplicaciones y de datos. Para obtener más información sobre los grupos de seguridad, consulte [Security in your IBM Cloud VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-using-security-groups).

En nuestro escenario configuraremos los grupos de seguridad para habilitar los puertos y protocolos requeridos.
En el menú *Infraestructura VPC*, selecciona "Grupos de seguridad" y luego "Crear +".

**App Security Group - app-sg**

- Añada una regla de entrada para permitir todos los accesos tcp en el puerto 22 para el acceso SSH a las VSI.
- Añada una regla de entrada para permitir todos los accesos tcp en el puerto 80 para el acceso HTTP a la aplicación web.
- Añada una regla de salida para permitir todos los accesos.


<p align="center"><img width="600" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra7.png"></p>


**Data Security Group - data-sg**

- Añada una regla de entrada para permitir todos los accesos tcp en el puerto 22 para el acceso SSH a las VSI.
- Añada una regla de entrada para permitir todos los accesos tcp en el puerto 3306 para MySQL (puerto por defecto para MySQL).
- Añada una regla de salida para permitir todos los accesos.


<p align="center"><img width="600" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra8.png"></p>


## Crear las VSI de la capa de datos - Subnet2

Ahora que tenemos toda la información necesaria, vamos a crear dos VSIs en `subnet2` para el backend MySQL. En el menú *Infraestructura VPC*, selecciona "Instancias de servidor virtual" en "Cómputo" y luego "Crear +".

Seleccione:
- Región = Dallas
- Zona = Dallas 2
- Introduzca el nombre de la instancia (la interfaz de usuario no permite mayúsculas para los nombres de recursos).
- Seleccione la imagen Ubuntu 18.04 y el perfil memory mx2-4x32.
- Clave SSH = `añadir la clave ssh creada`.
- VPC = web-app-vpc.
- Cambie `eth0` Network Interface para elegir `subnet2` y `data_sg`.

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra9.png"></p>

Repetir el proceso para la segunda instancia.


## Crear las VSI de la capa de aplicación - Subnet1

A continuación, cree dos VSIs Ubuntu en `subnet1` para el nivel de aplicación.

En este caso crearemos una segunda interfaz ethernet para conectarnos a los recursos en `subnet2` donde estarán ubicados los servidores MySQL.

Seleccionar:
- Región = Dallas
- Zona = Dallas 2
- Introduzca el nombre de la instancia (la interfaz de usuario no permite mayúsculas para los nombres de recursos).
- Seleccione la imagen Ubuntu 18.04 y el perfil compute c2-4x8.
- Clave SSH = `añadir la clave ssh creada`.
- VPC = web-app-vpc.
- Cambie la interfaz de red `eth0` para elegir `subnet1` y `app_sg`.
- Añade `eth1` Network Interface para elegir `subnet2` y `data_sg`.


<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra10.png"></p>

Repetir el proceso para la segunda instancia.

### Crear el balanceador de carga de aplicación

Para más información sobre la configuración de un balanceador de carga (listeners, back-end pools, etc.) ver [Using Load Balancers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-nlb-vs-elb)

En el menú *infraestructura VPC*, selecciona "Balanceadores de carga" y luego "Nuevo balanceador de carga".

Cree un balanceador de carga `público` `lb-web-app` en `subred1`. Configurar el balanceador de carga implica crear un pool, miembros del pool (servidores de aplicación), y un listener que apunte a nuestros servidores de aplicación.

**Load Balancer = lb-web-app**

Seleccionar:
- Introduzca el nombre de la instancia (la interfaz de usuario no permite el uso de mayúsculas en los nombres de recursos).
- Tipo = `Público`.
- Seleccione `subred1` (10.10.11.0/24)
- Grupo back-end:
   - Añade un pool de back-end `pool1` para el protocolo `http` utilizando un método `round-robin` y comprobaciones de estado cada `20 segundos`.
   - Adjuntar `AppServ1` y `AppServ2`.

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra11.png"></p>

- Receptor front-end:
   - Añade un listener público `http` para nuestra aplicación web usando el puerto `80` y asígnalo al pool de back-end `pool1`.

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra12.png"></p>

**Notas**: Mantener el grupo de seguridad por defecto de la VPC. Las comprobaciones de estado (health checks) del balanceador de carga fallarán hasta que la aplicación sea instalada más adelante

## Preparar el balanceador de carga de aplicación

Dado que no se admiten imágenes personalizadas (Bring-Your-Own-Image), habilitaremos el acceso a Internet para cada instancia de VPC para poder descargar el software de aplicación necesario. Dado que las VSI están aisladas de Internet, se utilizará una IP flotante para obtener acceso temporalmente. Una vez instalado el software de aplicación, se desactivará el acceso a Internet.

**Añadir IP Pública a cada servidor de Datos y Aplicación**

En el menú *Infraestructura VPC*, selecciona "Instancias de servidor virtual" y luego selecciona un servidor para ver los detalles. En "Interfaces de red" haga clic en el icono del lápiz para editarlo en eth0 y seleccione "Reservar una nueva IP flotante" en Dirección IP flotante.


**Web-server**

<p align="center"><img width="700" src="https://github.com/JoCGM09/IBMCloud-VPC-3tier-web/blob/master/Capturas%20VPC/infra13.png"></p>

Repetir el mismo proceso para web-server-2 únicamente en la interfaz eth0. Luego repetir el proceso con db-server y sb-server-2.


## PASO 2: Instalación de las aplicaciones web

## Instalar MySQL

Comenzamos desplegando el entorno de la base de datos de la aplicación Wordpress, Instsalamos MySQL en cada servidor de base de datos y creamos la base de datos de Wordpress.

Nos conectamos a la instancia de base de datos 1 por SSH usando la dirección IP flotante y la llave privada que colocamos a la instancia.

```
ssh -i "su llave privada" root@"ip pública"
```
### Actualizar el entorno Linux 

Actualizar el entorno usando los comandos: 

```
apt-get update
apt-get upgrade
```

### Instalamos los paquetes de MySQL

Instalamos MySQL. Ingresa a [MySQL](https://www.mysql.com/)para mayor información.
```
apt-get -y install mysql-server php-mysql
```
**Nota**: Si la instalación responde con un `groot` error, sigue las siguientes instrucciones: [Error: groot must be grub root device on ubuntu](https://developer.ibm.com/answers/questions/462237/error-groot-must-be-grub-root-device-on-ubuntu/)

### Agrega seguridad a tus aplicaciones de MySQL

El paquete del servidor MySQL viene con un script llamado `mysql_secure_installation` que puede realizar varias operaciones relacionadas con la seguridad. Ejecute el siguiente comando para establecer la contraseña de la base de datos y permitir que el usuario `root` se conecte desde un servidor remoto. Para obtener más información, consulte [Improve MySQL Installation Security](https://dev.mysql.com/doc/refman/5.7/en/mysql-secure-installation.html)

primero ejecute el comando mysql

```
mysql
```
luego una vez se encuentre en la interfaz de mysql, use el siguiente comando para establecer su nueva contraseña para el usuario root, en este caso será mysqlpass.

```
mysql> ALTER USER 'root'@'localchost' IDENTIFIED WITH mysql_native_password by 'mysqlpass';
```
luego salga de la interfaz con el comando exit

```
mysql> exit
Bye
```
Ahora corra el comando `mysql_secure_installation` e ingrese la nueva contraseña de acceso, luego responda las siguientes alternativas para una configuración segura sugerida, puede ser considerada bajo su criterio.

```
mysql_secure_installation
```
Respuestas para esta instalación:

- Responda `n` cuando se le solicite configurar *VALIDATE PASSWORD PLUGIN*.
- Responda `n` cuando se le solicite cambiar la contraseña actual.
- Responda `y` para eliminar usuarios anónimos.
- Responda `n` para no permitir el inicio de sesión de raíz de forma remota.
- Responda `y` para eliminar la base de datos de prueba.
- Responda `y` para recargar las tablas de privilegios.


### Crear base de datos de aplicaciones

Ahora crearemos una base de datos para la aplicación `WordPress`.

**Iniciar sesión en MySQL**
```
mysql -u root -p
```
**Crear la base de datos `wordpress`**
```
mysql> CREATE DATABASE wordpress;
```
Resultado
```
Query OK, 1 row affected (0.01 sec)
```

**Garantizar privilegios del usuario `root`**
Revisar [GRANT Syntax](https://dev.mysql.com/doc/refman/8.0/en/grant.html).
```
mysql> CREATE USER wordpress@localhost IDENTIFIED BY 'root';
```
Resultado
```
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
```
mysql> GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER ON wordpress.* TO wordpress@localhost;
```
Resultado
```
Query OK, 0 rows affected, 1 warning (0.00 sec)
```
**Nota importante** Si se utiliza en un entorno de producción, se recomienda utilizar un usuario con privilegios limitados y no utilizar root. Es recomendable no utilizar contraseñas simples o fácilmente adivinables para proteger la seguridad de la base de datos. Por tanto, es recomendable revisar los permisos necesarios para el usuario y asegurar que no se está otorgando un acceso excesivo.

**Recargar las grant tables**
```
FLUSH PRIVILEGES;
```
Resultado
```
Query OK, 0 rows affected (0.00 sec)
```
**Listar bases de datos disponibles**
```
show databases;
```
Resultado
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| wordpress          |
+--------------------+
5 rows in set (0.00 sec)
```
**Cerrar MySQL**
```
exit
```
Resultado
```
Bye
```

**Nota**: En cualquier momento del proceso de instalación de WordPress, puede repetir la creación de la base de datos de WordPress eliminándola (utilice `DROP DATABASE wordpress;`) y repitiendo los pasos anteriores para volver a crearla.

### Hacer el servidor MySQL visible a través de la red

Edita `/etc/mysql/my.cnf` y agrega estas líneas:
```
[mysqld]
bind-address    = 0.0.0.0
```
**Reinicia MySQL**
```
systemctl restart mysql
```
**Validar que MYSQL esté listo**

Confirme que MySQL está escuchando en todas las interfaces ejecutando el siguiente comando.
```
apt install net-tools
netstat --listen --numeric-ports | grep 3306
```
Resultado:
```
tcp        0      0 localhost:3306          0.0.0.0:*               LISTEN
tcp        0      0 localhost:33060         0.0.0.0:*               LISTEN
```
**Salir del servidor `MySQL1`**
Cerrar sesión de la VSI usando el comando `exit`. 

## Instalar Nginx - web-server 1

Antes de instalar WordPress, se debe configurar un servidor web y un tiempo de ejecución de PHP. Aquí usaremos dos sesiones de terminal SSH para conectarnos a cada servidor de aplicaciones por separado.

Conéctese al servidor `web-server 1` usando SSH y la dirección IP flotante reservada.
```
ssh -i "su llave privada" root@"ip pública"
```
### Actualizar el entorno Linux 

Actualizar el entorno usando los comandos: 

```
apt-get update
apt-get upgrade
```
### Instalar NGINX.

Instalar [Nginx](https://www.nginx.com/).
```
apt-get -y install nginx
```
**Nota**: Si la instalación encuentra un error `groot`, por favor siga las instrucciones en el siguiente enlace: [Error: groot must be grub root device on ubuntu](https://developer.ibm.com/answers/questions/462237/error-groot-must-be-grub-root-device-on-ubuntu/)

En este punto Nginx ha sido instalado y está funcionando. Puedes validar su instalación introduciendo la IP flotante de `web-server 1` en un navegador web (deberías obtener el siguiente mensaje: "¡Bienvenido a nginx!").

## Instalar Nginx - web-server 2

Conéctate al servidor `web-server 2` usando SSH y la dirección IP flotante reservada.
```
ssh -i "su llave privada" root@"ip pública"
```
Repite la instalación de Nginx para `AppServ2` siguiendo los pasos para `AppServ1`.

### Pruebe el balanceador de carga.

Una vez completada la instalación de Nginx, las comprobaciones de salud del balanceador de carga deberían pasar y los servidores de aplicaciones deberían ser accesibles a través del balanceador de carga. Utilice la dirección pública (nombre de host) del equilibrador de carga para acceder a la página de Nginx (debería recibir el siguiente mensaje: "¡Bienvenido a nginx!").

En nuestro caso --> `http://8d3374f1.lb.appdomain.cloud/`

Nota: Las comprobaciones de estado del equilibrador de carga ejecutan un GET HTTP en el puerto 80 y esperan que se devuelva un 200 HTTP. También puede probar el retorno HTTP ejecutando el siguiente comando cURL en cualquiera de los servidores: `curl -v http://<private_ip>:80/` donde <private_ip> es la IP del host de destino.

Por ejemplo,
```
curl -v http://10.10.11.6:80/
```
Resultado

```
Connected to 10.10.11.6 (10.10.11.6) port 80 (#0)
> GET / HTTP/1.1
> Host: 10.10.11.6
> User-Agent: curl/7.58.0
> Accept: */*
>
< HTTP/1.1 200 OK
< Server: nginx/1.14.0 (Ubuntu)
< Date: Wed, 10 Apr 2019 04:46:41 GMT
< Content-Type: text/html
< Content-Length: 612
< Last-Modified: Wed, 10 Apr 2019 04:27:45 GMT
< Connection: keep-alive
< ETag: "5cad70c1-264"
< Accept-Ranges: bytes
```

## Instalar PHP - web-server 1

En esta actividad instalaremos [PHP-FPM](https://php-fpm.org/) y el [plugin MySQL](https://www.php.net/manual/en/set.mysqlinfo.php). (PHP-FPM es necesario para el servidor web Nginx).

PHP se iniciará automáticamente después de la instalación.
```
apt-get -y install php-fpm php-mysql
```

**Configurar PHP**

La instalación de Nginx incluye un archivo de configuración por defecto. Necesitamos modificar este archivo para habilitar PHP pero primero vamos a crear una copia de seguridad de este archivo en caso de que necesitemos restaurar la configuración por defecto.
```
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default_original
```

Edita el archivo de configuración `/etc/nginx/sites-available/default` y sigue estos pasos:
1. Elimina/Comenta `default_server` del puerto de escucha.
```
listen 80; # default_server;
listen [::]:80; # default_server;
```
2. Añade `index.php` al principio de la directiva `index`.
```
index index.php index.html index.htm index.nginx-debian.html;
```
- Descomenta las líneas de `location ~ \.php$` y actualiza la versión de PHP a la que acabas de instalar. En este caso, `7.2`.
```
location ~ \.php$ {
        include snippets/fastcgi-php.conf;
#
# # Con php-fpm (u otros sockets unix):
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
# Con php-cgi (u otros sockets tcp):
# fastcgi_pass 127.0.0.1:9000;
}
```
- Descomentar líneas para `location ~ /\.ht`
```
location ~ /\.ht {
        deny all;
}
```
Notas sobre el contenido del archivo de configuración de Nginx:
- `listen` - Define en qué puerto escuchará Nginx. En este caso, escuchará en el puerto 80, el puerto por defecto para HTTP.
- `root` - Define la raíz del documento donde se almacenan los archivos servidos por el sitio web.
- `index` - Configura Nginx para que dé prioridad a los archivos index.php cuando se solicite un archivo index, si están disponibles.
- `location /` - El primer bloque de ubicación incluye una directiva try_files, que comprueba la existencia de archivos que coincidan con una petición URI. Si Nginx no puede encontrar el archivo apropiado, devolverá un error 404.
- `location ~ \.php$` - Este bloque de ubicación maneja el procesamiento real de PHP apuntando a Nginx al archivo de configuración fastcgi-php.conf y al archivo php7.2-fpm.sock, que declara qué socket está asociado con php-fpm.
`location ~ /\.ht` - El último bloque de ubicación se ocupa de los archivos .htaccess, que Nginx no procesa. Añadiendo la directiva deny all, si cualquier archivo .htaccess se encuentra en la raíz del documento, no será servido a los visitantes.

Compruebe si hay errores de sintaxis en su nuevo archivo de configuración escribiendo:
```
sudo nginx -t
```
**Crear un archivo PHP para probar la configuración**

El archivo de configuración por defecto de Nginx define `root` como `/var/www/html/`. Cree el archivo `/var/www/html/info.php` en esta ubicación con las siguientes entradas:
```
<?php
phpinfo();
```
**Prueba PHP**

Reiniciar Nginx
```
systemctl reload nginx
```
Carga `info.php` en tu navegador usando la dirección IP pública de `web-server 1`.
```
http://"ip pública del servidor"/info.php
```
Deberías obtener de la información de `AppServ1` generada por PHP.

**nota**: Si recibes un error 502 Bad Gateway, lo recomendable es revisar el log del nginx, se recomienda usar el siguiente script: 

``
tail -f /var/log/nginx/error.log
``
Posiblemente el ejecutar este comando observes el siguiente problema: 

``
/var/run/php/php7.2-fpm.sock failed (2: No such file in directory)
``
Esto significa que la versión de php-fpm no es la adecuada para el archivo de configuración del archivo `/etc/nginx/sites-available/default`

Para conocer la versión correcta de php, ejecutar `cd /var/run/php` y revisar internamente la versión del archivo .lock, finalmente modificar esa versión en el archivo de configuración.
Luego al realizar la modificación, reiniciar nginx `systemctl reload nginx`

## Instalar PHP - web-server 2

Conéctese al servidor `web-server 2` usando SSH y la dirección IP flotante reservada.
```
ssh -i "su llave privada" root@"ip pública"
```
Repite la instalación de PHP para `web-server 2` siguiendo los pasos para `web-server 1`.

## Pruebe el balanceador de carga.

Ahora puede probar su Load Balancer y validar que se está llamando a ambos servidores de aplicaciones.

En nuestro caso --> `http://8d3374f1.lb.appdomain.cloud/info.php`

Al volver a cargar el navegador web varias veces, la información de PHP cambiará de `web-server 1` a `web-server 2`.

## Instalación de WordPress

Hasta este punto, hemos realizado las mismas actividades tanto en `web-server 1` como en `web-server 2`. Desde este punto en adelante, todas las actividades se ejecutarán solo en `web-server 1` a menos que se indique lo contrario.

### Detenga PHP y Nginx en ambos servidores.

Apagaremos ambos servidores para los siguientes pasos. 

```
systemctl stop php7.2-fpm
systemctl stop nginx
```
### Instalar WordPress - web-server 1

**Añadir extensiones PHP adicionales**

Hasta ahora, sólo hemos necesitado un conjunto mínimo de extensiones para conseguir que PHP se comunique con MySQL. WordPress y muchos de sus plugins utilizan extensiones PHP adicionales. Instálalas.
```
apt install php-curl php-gd php-intl php-mbstring php-soap php-xml php-xmlrpc php-zip
```
**Recuperar los archivos de instalación de Wordpress**

Cambia a la carpeta `/tmp`.
```
cd /tmp
```
Obtener el último archivo tar
```
curl -O https://wordpress.org/latest.tar.gz
```
Resultado
```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 10.1M  100 10.1M    0     0  14.6M      0 --:--:-- --:--:-- --:--:-- 14.6M
```
Descomprimir archivo
```
tar xzvf latest.tar.gz
```
**Preparar los archivos de Wordpress para la instalación**

Utilice el archivo de configuración por defecto.
```
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```
Crear una carpeta de actualización.
```
mkdir /tmp/wordpress/wp-content/upgrade
```
Usaremos `/var/www/wordpress` como directorio raíz de nuestra instalación de WordPress.
```
mkdir /var/www/wordpress
```
Transferir archivos a la ubicación `/var/www/wordpress
```
rsync -av -P /tmp/wordpress/. /var/www/wordpress
```
Establece los permisos de acceso a los archivos.
```
chown -R www-data:www-data /var/www/wordpress
find /var/www/wordpress -type d -exec chmod g+s {};
chmod g+w /var/www/wordpress/wp-content
chmod -R g+w /var/www/wordpress/wp-content/themes
chmod -R g+w /var/www/wordpress/wp-content/plugins
```
### Configurar WordPress

Edite el fichero `/var/www/wordpress/wp-config.php` y configure la información de la BD para `database1` (use el puerto 3306 en DB_HOST).
```
define('DB_NAME', 'wordpress');
define('DB_USER', 'root');
define('DB_PASSWORD', 'mysqlpass');
define('DB_HOST', '10.10.12.7:3306');
```
Llama al siguiente servicio web para obtener las claves de seguridad de WordPress. Utiliza el resultado para actualizar las entradas correspondientes en el archivo `/var/www/wordpress/wp-config.php`.
```
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```
Resultado - **NO COPIE ESTAS - Utilice las claves generadas en el comando anterior para su configuración**.
```
define('AUTH_KEY', 'cH$fF-|t$fMHlX#kf,{a+w|Z=efedUGv{2&V#@e `0-@S_(I`jV4EV=tKQsk]Dbe');
define('SECURE_AUTH_KEY', 'LocO1 dNUYQaK::E+Dve|At1)h|U||E>nH4tyA+Qq@X{Ed3t[DTqpj#x_^*#0&MA');
define('LOGGED_IN_KEY', 'XE,|u,JEDMbasQ~ chXzV;Q!L-Pj0tHf}Xt^PY&Y`}TgUuRlI[#:?+}z*iAk5x,U');
define('NONCE_KEY', 'jq(6T?%T~*~nv?+%vcPBPYPoLu,vHBF>|D!^73QsL`-nTvP(qZY1WEAf<T=t-l;)');
define('AUTH_SALT', '?C?lO.K,/46|(FQby=;/J#NX[uAg{b#FT&&e>>s6$iA{m@X9@!68016to&>-+Eyt');
define('SECURE_AUTH_SALT', 'HSu#sJcP`3BOu33d*tL>sxLCJ`RSe_M2O%qjWw Yq#30_m/Et-Ns3Zh-iPbID&LG');
define('LOGGED_IN_SALT', '~0$^]Z;,/X?F[+c6=4(xGcG+@YI}<9&1~?B&GlTFg+({#|b:|v1|QhHRmWlk(2G5');
define('NONCE_SALT', ' wu^]-@w>oq){N%DPZM)P=]w|Vp scl9kzhRebDqHheJMSfhm)vG/DAl)o|fOTgg');
```
**NOTA**: Deben utilizarse las mismas claves de seguridad WP en `web-server 1` y `web-server 2` (o en todos los servidores App de un cluster).

### Configurar Nginx para WordPress

Necesitamos modificar el archivo de configuración de Nginx para WordPress. (ver [WordPress rules](https://codex.wordpress.org/Nginx)).

Antes de la actualización, hagamos una copia de seguridad de la configuración actual del archivo para PHP.
```
cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default_PHP
```
Edita el archivo de configuración de Nginx `/etc/nginx/sites-available/default` y añade las siguientes ubicaciones dentro del bloque principal `server`:
```
        location = /favicon.ico {
                 log_not_found off;
                 access_log off;
                 expires max;
         }

        location = /robots.txt {
                allow all;
                log_not_found off;
                access_log off;
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
                caduca máx;
                log_not_found off;
        }
```
Dentro del bloque `location /` existente, necesitamos ajustar la lista try_files para que en lugar de devolver un error 404 por defecto, se pase el control al archivo index.php con los argumentos de la petición. Esto debería ser algo como esto
```
        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #try_files $uri $uri/ =404;

                # This is cool because no php is touched for static content.
                # include the "$is_args$args" so non-default permalinks doesn't break when using query string
                try_files $uri $uri/ /index.php$is_args$args;
        }
```
Por último, actualiza `root` para que apunte a la ubicación de WordPress.
```
        #root /var/www/html;
        root /var/www/wordpress;
```
La configuración ya está completa. Guarde el archivo Nginx y compruebe si hay errores con este comando:
```
sudo nginx -t
```
**web-server 2** Saltar a [Configurar HyperDB](WebApp.md#configure-hyperdb)

**Iniciar el servidor web y el runtime PHP.**
```
systemctl start php7.2-fpm 
systemctl start nginx
```

7.2 o la versión que se esté usando. 

### Completar la instalación de WordPress a través de la Web UI

**Notas:**
1. Se puede acceder al archivo `readme` de WordPress utilizando el siguiente enlace `http://<floatingip>/readme.html`.
2. Las comprobaciones de estado del equilibrador de carga fallarán hasta que se complete la instalación de WordPress.

Continúe con la instalación de WordPress como se indica a continuación:
- En su navegador web, acceda a la instalación de Wordpress utilizando `web-server 1` IP flotante `http://"ip pública del servidor"`.
- Esto le llevará a `http://"ip pública del servidor"/wp-admin/install.php`.
- Seleccione "Instalar WordPress" y rellene los campos necesarios para completar la instalación.
```
Site Title = Basic 3-Tier Web App
User Name = wpadmin
Password = mywppass
Your Email = <your email>
```
**Actualizar URLs de WordPress**

1. Acceda a la página web del administrador de Wordpress `http://"ip pública del servidor"/wp-admin` utilizando el ID y la contraseña de administrador.
2. Seleccione `Settings` en el panel de control para acceder a la vista `General Settings`.
3. 3. Establezca la `Wordpress address (URL)` y la `Site Address (URL)` en la dirección pública del balanceador de carga (**no** incluya una barra al final). En nuestro caso las entradas serán:
```
Dirección de WordPress (URL) = http://8d3374f1.lb.appdomain.cloud
Dirección del sitio (URL) = http://8d3374f1.lb.appdomain.cloud
```
4. Guarda los cambios.
5. Detener PHP y Nginx
```
systemctl stop php7.2-fpm
systemctl stop nginx
```
6. Edite el archivo `/var/www/wordpress/wp-config.php` y añada las dos entradas siguientes al final del archivo:
```
/* Override WP Admin console settings for Site and Home URLs */
define( 'WP_SITEURL', 'http://8d3374f1.lb.appdomain.cloud' );
define( 'WP_HOME', 'http://8d3374f1.lb.appdomain.cloud' );
```
7. Iniciar PHP y Nginx
```
systemctl start php7.2-fpm
systemctl start nginx
```
La instalación de WordPress se ha completado y la comprobación del estado del balanceador de carga será correcta para `AppServ1`. En una ventana del navegador, introduzca la dirección pública del equilibrador de carga para validarla.

**Nota**: Si es necesario, puede restablecer la instalación de la interfaz web de WordPress eliminando y volviendo a crear la base de datos `wordpress` tal y como se indica en la sección de configuración del servidor `database 1`. A continuación, repita los pasos anteriores.

### Añadir WordPress Incorrect Datetime Bug Fix

HyperDB introduce un problema de fecha/hora en MySQL y necesitamos instalar el plugin de WordPress `Incorrect Datetime Bug Fix` para solucionar este problema.
1. Acceda a la página web del administrador de Wordpress `http://8d3374f1.lb.appdomain.cloud/wp-admin` utilizando el ID y la contraseña de administrador.
2. Seleccione `Plugins` en el panel de control y luego `Add New` para llegar a la página de plugins de WordPress.
3. Busque "Incorrect Datetime" para encontrar el plugin.
4. Seleccione `install now` y `activate`.

### Pruebe las funciones de WordPress
1. Acceda a la página web del administrador de Wordpress `http://http://8d3374f1.lb.appdomain.cloud/wp-admin` utilizando el ID y la contraseña de administrador.
2. En el dashboard, añada un nuevo post: `Posts` > `Add New` e introduzca los datos que desee. Publica tu post.
3. 3. Confirme que su entrada aparece en la página web de WordPress. Utilice otra ventana del navegador e introduzca la dirección pública del balanceador de carga. **Nota**: Si utiliza la IP flotante `AppServ1` (`http://169.61.244.83`) será redirigido a la dirección del Balanceador de Carga.
4. En la página web de WordPress, introduzca un comentario/respuesta a la entrada que acaba de publicar.
5. 5. De vuelta al panel de control del administrador de WordPress, seleccione `Comentarios` para mostrar el nuevo comentario en espera de aprobación. Apruébelo.
6. En el panel de control, seleccione `Apariencia` > `Temas` > seleccione el tema `Twenty Seventeen` > `Vista previa en vivo`.
7. `Activar y publicar` el nuevo tema.
8. Valide la actualización del tema en otra ventana del navegador.

## Configuración de HA

En esta sección utilizaremos [Replicación de bases de datos MySQL](https://dev.mysql.com/doc/refman/5.7/en/replication.html) y el plugin [HyperDB](https://wordpress.org/plugins/hyperdb/) para permitir la escalabilidad de la solución.

## Replicación de bases de datos MySQL.

La replicación de bases de datos MySQL permite que los datos de un servidor de bases de datos MySQL (base de datos fuente) sean copiados automáticamente a uno o más servidores de bases de datos MySQL (base de datos réplica). La replicación se utiliza generalmente para difundir el acceso de lectura en varios servidores para la escalabilidad. Además, dado que los datos se replican en varias bases de datos, éstas pueden funcionar como copias de seguridad o conmutación por error cuando la base de datos de origen no está disponible.

La replicación es asíncrona por defecto; las bases de datos réplica no necesitan estar conectadas permanentemente para recibir actualizaciones de la base de datos origen. Dependiendo de la configuración, puede replicar todas las bases de datos, bases de datos seleccionadas o incluso tablas seleccionadas dentro de una base de datos.

En nuestro escenario, configuraremos `MySQL2` como **base de datos réplica** de `MySQL1` como **base de datos fuente** para permitir la escalabilidad de la solución.

### Configurar MySQL1 - Maestro

Conéctese al servidor MySQL1 usando la dirección ip flotante reservada.
```
$ ssh root@169.61.244.78
```

Edita el fichero de configuración de MySQL `/etc/mysql/my.cnf` y añade lo siguiente al grupo `[mysqld]` para configurarlo como base de datos origen.
```
server-id = 1
log_bin = /var/log/mysql/mysql-bin.log
binlog_do_db = wordpress
```
- server-id` - Con la replicación MySQL source-replica cada instancia debe tener su propio server-id único, use 1 para la base de datos fuente.
- `log_bin` - Nombre y ubicación del archivo de registro binario. Este registro es utilizado por las bases de datos réplica para replicar los cambios registrados en la base de datos.
- `binlog_do_db` - Base de datos a replicar. En nuestro caso, `wordpress`.

Reiniciar MySQL.
```
systemctl restart mysql
```







## PASO 3: Remover las IP flotanes

Una vez que el entorno esté en funcionamiento, puede eliminar las IP flotantes para eliminar el acceso público en las VSI.

En el menú *Infraestructura VPC*, selecciona "Instancia de servidor virtual" y luego selecciona un servidor para ver los detalles. En "Interfaz de red" seleccione el signo `menos` en `eth0` para eliminar una IP flotante.

Opcionalmente, una vez que las IPs Flotantes han sido eliminadas, también puedes liberar las IPs Flotantes si ya no hay necesidad de ellas.

En el menú *VPC Infrastructure*, selecciona "Virtual server instances" y luego selecciona un servidor para ver los detalles. En "Network interfaces" selecciona el signo `minus` en `eth0` para eliminar una IP Flotante.


## Referencias

- [Getting started with IBM Cloud Virtual Private Cloud](https://cloud.ibm.com/docs/vpc)

- [Assigning role-based access to VPC resources](https://cloud.ibm.com/docs/resources?topic=resources-rgs#rgs)

- [IBM Cloud CLI for VPC Reference](https://cloud.ibm.com/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference)

- [VPC API](https://cloud.ibm.com/apidocs/vpc)

- [Solution Tutorials - Highly Available & Scalable Web App](https://cloud.ibm.com/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
