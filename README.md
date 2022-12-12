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

## Deploy VPC Infrastructure

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

## Create Web Tier VPC Instance

In this section we will create and configure a VPC load balancer for the web application tier. For more information on configuration of load Balancers (listeners, back-end pools, etc.) see [Using Load Balancers for VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-nlb-vs-elb)

### Create the Load Balancer

On the *VPC Infrastructure* menu, select "Load balancers" and then "New load balancer".

Create a `public` load balancer `LB1` on `subnet1`. Configuring the load balancer involves creating a pool, pool members (application servers), and a listener that points to our application servers.

**Load Balancer = LB1**

Select:
- Enter instance name (the UI does not allow upper case for resource names).
- Resource Group = `VPC1`
- Type = `Public`
- Select `subnet1` (10.10.11.0/24)
- Back-end pool:
   - Add a Back-end pool `pool1` for `http` protocol using a `round-robin` method and health checks every `20 seconds`.
   ![Create Pool1](images/backendpool.png)
   - Attach `AppServ1` and `AppServ2`
   ![Create Pool Member](images/attachinstance.png)
- Front-end listener
   - Add a public front-end `http` listener for our web application using port `80` and assign it to back-end pool `pool1`
   ![Create Listener](images/listener.png)

![Create Load Balancer](images/lb1.png)

**Note**: Load Balancer health checks will fail until the application is installed in section [Install and Configure Application Software](WebApp.md).

## Prepare to Load Application Software

Because custom images are not supported (Bring-Your-Own-Image), we will enable access to the internet for each VPC instance so we can download the required application software. Since the VSIs are isolated from the internet, a floating IPs will be used to temporarily gain access. Once the application software has been installed, internet access will be disabled.

**Add Public IP to each Data and Application servers**

On the *VPC Infrastructure* menu, select "Virtual server instances" and then select one server to drill down to the details. Under "Network interfaces" click the pencil icon to edit it on eth0 and select "Reserve a new floating IP" at Floating IP address.

**AppServ1**

![AppServ1 FIP](images/appserv1fip.png)

**AppServ2**

![AppServ2 FIP](images/appserv2fip.png)

**MySQL1**

![MySQL1 FIP](images/mysql1fip.png)

**MySQL2**

![MySQL2 FIP](images/mysql2fip.png)

## Next Step

At this point the VPC infrastructure components are ready for the next step which is to deploy the application software to the VSIs and test the Load Balancer. Please go to [Install and Configure Application Software](WebApp.md) for the next steps.

## Remove Floating IPs

Once the environment is up and running, you can remove the floating IPs to remove public access on the VSIs.

On the *VPC Infrastructure* menu, select "Virtual server instances" and then select one server to drill down to the details. Under "Network interfaces" select the `minus` sign on `eth0` to remove a Floating IP.

Optionally, once the Floating IPs have been removed, you can also release the Floating IPs if there is no longer a need for them.

On the *VPC Infrastructure* menu, select "Virtual server instances" and then select one server to drill down to the details. Under "Network interfaces" select the `minus` sign on `eth0` to remove a Floating IP.























### Instalación de la aplicación web

Deploy the application once the VPC infrastructure has been deployed.

[Install Application Layer](WebApp.md)

## Error Scenarios

Application layer failures are included during the deployment and test of the software stack. No infrastructure failures were introduced.

## Documentation Provided

Useful links for VPC documentation.

[Getting started with IBM Cloud Virtual Private Cloud](https://cloud.ibm.com/docs/vpc)

[Assigning role-based access to VPC resources](https://cloud.ibm.com/docs/resources?topic=resources-rgs#rgs)

[IBM Cloud CLI for VPC Reference](https://cloud.ibm.com/docs/vpc?topic=vpc-infrastructure-cli-plugin-vpc-reference)

[VPC API](https://cloud.ibm.com/apidocs/vpc)



Referencias

Based on [Solution Tutorials - Highly Available & Scalable Web App](https://cloud.ibm.com/docs/tutorials?topic=solution-tutorials-highly-available-and-scalable-web-application#use-virtual-servers-to-build-highly-available-and-scalable-web-app)
