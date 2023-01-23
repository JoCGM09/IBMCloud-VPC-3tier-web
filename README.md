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
