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

########################### imagen de la arquitectura ##########################################


## Componentes funcionales de la VPC

- VPC
- Subnets
- IPs privadas
- Instancias de servidores virtuales (VSI)
- Múltiples interfaces de red en la VSI
- Balanceador de carga a nivel de aplicación
- IPv4 flotante
- Public Gateway

### Sistemas operativos utilizados

- Servidor web y de aplicación: Ubuntu
- Servidor de datos: Ubuntu

#### Hardware seleccionado

- Servidor web de aplicación: bx2-4x16
- Servidor de datos: bx2-4x16

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

Seleccione Infraestructura VPC desde el menú en la esquina superior izquierda.

![VPC Infrastructure](images/hamburgermenu.png)

## Deploy VPC Infrastructure

## Create an SSH Key

An SSH key is required when creating a VPC instance. From the VPC Infrastructure menu select "SSH keys" and then "Create +".  Enter name, select region (e.g. Dallas) and add the contents of your RSA public key. Hit the "Add SSH key" button to save.

![SSH Key](images/sshkey.png)

## Create a VPC

Create a VPC named `wp-vpc`.

On the *VPC Infrastructure* menu, select "VPCs" under "Network.". Then select "Create +". Fill out the form and hit the "Create virtual private cloud" button.

Use `subnet0` for subnet and **attach** a Public Gateway. The public gateway will be needed to access the application software from the public repositories.

![Create VPC](images/VPC.png)

## Create Address Prefixes

For more information on address prefixes, please refer to [Understanding IP address ranges, address prefixes, regions, and subnets](https://cloud.ibm.com/docs/vpc?topic=vpc-vpc-addressing-plan-design).

Create address prefixes for `10.10.11.0/24` and `10.10.12.0/24`.

On the *VPC Infrastructure* menu, select "VPCs" under "Network.". Select VPC `wp-vpc` to get the details, then select "Address prefixes" and "Create".

![Address Prefixes](images/addressprefixes.png)

## Create Two VPC Subnets

Create two VPC Subnets for ipv4-cidr-blocks `10.10.11.0/24` and `10.10.12.0/24`.  

The __*application*__ tier will be `subnet1` and the __*data*__ tier will be `subnet2`.

On the *VPC Infrastructure* menu, select "Subnets" and then click "Create +". Create each subnet and attach the VPC's public gateway.

**Subnet1**
![Create subnet1](images/subnet1.png)

**Subnet2**
![Create subnet2](images/subnet2.png)

### VPC Subnets

The initial status of a newly created subnet is set to __*pending*__.  You must wait until the subnet status is available before assigning any resources to it.

To check the subnet status, refresh the Subnets list.  Keep checking until the status is set to available.

![Check Subnet Status](images/subnets.png)

### Optionally, Remove Original Subnet0

Creating a VPC requires a subnet (`subnet0`) that uses a default address prefix.  Since we will not be using the default address prefixes nor the initial subnet, those can be deleted. To delete `subnet0`, on the *VPC Infrastructure* menu, select "Subnets" and then the three periods (`...`) to select delete.

![Delete Original Subnet](images/subnet0_del.png)

## VPC Instance Profiles and Images

Before continuing we must select an instance profile and image for our VPC instances.  
- The profile describes the instance size in terms of CPUs and memory.
- The image is the operating system that will be loaded into the instance.

We will use the `b-4x16` balanced profile for all our instances, which is 4 CPUs and 16G of memory.  For OS image, the `ubuntu-18.04-amd64` which is Ubuntu Linux (18.04 LTS Bionic Beaver Minimal Install).

## Security Groups and Access Control Lists

For purposes of this use case, we will create two security groups for application and data servers. For more information on security groups, please refer to [Security in your IBM Cloud VPC](https://cloud.ibm.com/docs/vpc?topic=vpc-using-security-groups).

In our scenario we will configure the security groups to enable the required ports and protocols.

On the *VPC Infrastructure* menu, select "Security groups" and then "Create +".

**Application Security Group - app-sg**

- Add an inbound rule to allow all tcp access on port 22 for SSH access to the VSIs.
- Add an inbound rule to allow all tcp access on port 80 for HTTP access to the web application.
- Add an outbound rule to allow all access.

![Application Security Group](images/appsg.png)

**Data Security Group - data-sg**

- Add an inbound rule to allow all tcp access on port 22 for SSH access to the VSIs.
- Add an inbound rule to allow all tcp access on port 3306 for MySQL (default port for MySQL).
- Add an outbound rule to allow all access.

![Data Security Group](images/datasg.png)

## Create Data Tier VPC Instances - Subnet2

Now we have all the required information, let's create two Ubuntu 18.04 VSIs in `subnet2` for the MySQL backend.

On the *VPC Infrastructure* menu, select "Virtual server instances" under "Compute" and then "Create +".

Select:
- Enter instance name (the UI does not allow upper case for resource names).
- Select Ubuntu 18.04 image and balanced 4x16 profile.
- SSH Key = `vpc-key`
- Change `eth0` Network Interface to pick `subnet2` and `data_sg`.

**Instance = MySQL1**

![Create MySQL1](images/mysql1.png)

**Instance = MySQL2**

![Create MySQL2](images/mysql2.png)

## Create Web and Application tier VPC instances - Subnet1

Next, create two Ubuntu VSIs in `subnet1` for the application tier.

In this case we will create a second ethernet interface to connect to resources in `subnet2` where MySQL servers will be located.

Select:
- Enter instance name (the UI does not allow upper case for resource names).
- Select Ubuntu 18.04 image and balanced 4x16 profile.
- SSH Key = `vpc-key`
- Change `eth0` Network Interface to pick `subnet1` and `app_sg`.
- Add `eth1` Network Interface to pick `subnet2` and `data_sg`.

**Instance = AppServ1**

![Create AppServ1](images/appserv1.png)

**Instance = AppServ2**

![Create AppServ2](images/appserv2.png)

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
