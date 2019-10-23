## Tabla de contenidos
0. [Introducción](#intro)

1. [SNMP](#snmp)
    + [Configuración](#snmpconf)
    + [Comprobación](#test)
2. [Conclusión](#conclusion)

3. [Fuentes](#fuentes)

<div id='intro'/>

## 0. Introducción 
En este documento veremos la parte del alumno Marouane Boukhriss realizada para el trabajo grupal "Monitorización de servicios en red".
En los siguientes apartados veremos como configurar un servidor snmp bajo el sistema operativo de CentOS 7 con pruebas de veracidad.
La parte que he realizado en esta práctica es configurar un servidor snmp con pruebas remotas y locales para la verificación del trabajo, todos los archivos usados logs se encuentran en la carpeta "Logs".

<div id='snmp'/>

## 1. SNMP
SNMP es el protocolo simple de administración  de red usado para administrar la red y su diagnostico. Este protocolo trabajo con 
+ TRAPS
    + Son mensajes que lanza un dispositivo final en forma de eventos o cambios sucedidos.
+ POLLING
    + La función del polling es lanzar consultas remotas.

Existen conceptos básicos que es necesario conocer para comprender como funciona este protocolo, las definiciónes necesarias de conocer son MIB y OID.
MIB es la base de información de administración que recopila información para la administración de un elemento en la red.
OID es el identificador de objeto siendo de está forma unico para cada elemento, cada identificador apunta a características de el dispositivo.
La relacción existente entre MIB y OID es el organización jerárquica en arbol sobre un identificador, por ejemplo "sysDescr" es .1.3.6.1.2.1.1.1. 

Este protocolo dispone de tres versiones
+ SNMPV1
    + Puesto a que la SNMPV1 esta obsoleta no hablaremos de ella.
+ SNMPV2C
    + Incluye mejoras de estructuración de MIB, transporte de paquetes etc..
+ SNMPV3
    + Es la versión segura de SNMP.


<div id='snmpconf'/>

#### 1.2 Configuración SNMP
Deberemos de contar con multiples paquetes necesarios para configurar SNMP bajo la versión 2-3 y realizar la comprobación. Todos los cambios realizados en snmpd serán en el fichero de configuración.
```
$ /etc/snmp/snmpd.conf
```
Y cada modificación para aplicarse debemos de reiniciar el servicio mediante el comando:
```
$ systemctl restart snmpd
```

Los paquetes necesarios son:
```
$ yum install net-snmp net-snmp-utils
```
Como medida de seguridad vamos a realizar una copia del documento de configuración de snmp original:
```
$ mv /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.orig
$ cp /etc/snmp/snmpd.conf.orig /etc/snmp/snmpd.conf
```
Ahora si procederemos a configurar comunidades y grupos para asignar permisos y crear las dichas views para poder realizar las consultas (MIB).

```
#       sec.name        source          community
com2sec ConfigUser      default         maruser
com2sec AllUser         default         marconf

#                      sec.model       sec.name
group   ConfigGroup     v2c             ConfigUser
group   AllGroup        v2c             AllUser

#                       incl/excl       subtree
view    SystemView      included        .1.3.6.1.2.1.1
view    SystemView      included        .1.3.6.1.2.1.25.1.1
view    AllView         included        .1

#                       context model   level   prefix  read            write   notify
access  ConfigGroup     ""      any     noauth  exact   SystemView      none    none
access  AllGroup        ""      any     noauth  exact   AllView         none    none
```
Bien procederemos a explicar la configuración, en primer lugar disponemos de dos comunidades.
+ maruser
    + Esta comunidad esta configurada para obtener todos los OID del sistema (AllView, .1),está pensada  para ser monitorizada remotamente (Dicha función se ha comprobado en clase mediante PANDORA FMS localmente por mi mismo.)
+ marconf
    + Esta comunidad está configurada para verificar la configuración de forma local.

La configuración restante realizada es por permisos y acceso al grupo (Comunidades).
Configuraremos snmpv3 para añadir autentificación mediante un usuario con encriptación MD5 para ello usaremos el siguiente comando

```
net-snmp-create-v3-user
```

Cabe destacar que no hay que modificar ningún tipo de configuración dentro snmpd.conf, debido a que snmpv3 añade autentificación no podemos espeficiar usar la versión 3 mediante el fichero de configuración pero si se puede limitar solo a v2c 

<div id='test'/>

#### 1.3 Comprobación
Para realizar la comprobación deberemos de usar snmpwalk (SNMP BROWSER), este comando permitira consultar MIB
```
snmpwalk -v 2c -c maruser -O e 10.10.2.189
snmpwalk -v 2c -c marconf -O e 127.0.0.1
```
Para la versión 3 debemos de especificar el usuario y clave creados anteriormente. La salida de esté comando es identica a la versión 2c, como hemos comentado solo influye en la autentificación.
```
snmpwalk -v3 -a MD5 -A Admin234/ -x DES -X Admin234/ -l authPriv -u snmpadmin 192.168.1.138
```

La salidas de dichos comandos se muestrán en el apartado de Logs, a continuación veremos una corta representación de la salida de maruser

```
SNMPv2-MIB::sysDescr.0 = STRING: Linux localhost.localdomain 3.10.0-1062.el7.x86_64 #1 SMP Wed Aug 7 18:08:02 UTC 2019 x86_64
SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (16640) 0:02:46.40
SNMPv2-MIB::sysContact.0 = STRING: Root <root@localhost> (configure /etc/snmp/snmp.local.conf)
SNMPv2-MIB::sysName.0 = STRING: localhost.localdomain
SNMPv2-MIB::sysLocation.0 = STRING: Unknown (edit /etc/snmp/snmpd.conf)
SNMPv2-MIB::sysORLastChange.0 = Timeticks: (8) 0:00:00.08
SNMPv2-MIB::sysORID.1 = OID: SNMP-MPD-MIB::snmpMPDCompliance
SNMPv2-MIB::sysORID.2 = OID: SNMP-USER-BASED-SM-MIB::usmMIBCompliance 
```

<div id='conclusion'/>

## 2. Conclusión
Hemos visto y explicado todo lo necesario para poder realizar una configuración de un servidor SNMPV2C bajo CentOS 7.

<div id='fuentes'/>

## 3. Fuentes 
+ https://www.servernoobs.com/enabling-snmp-on-centos-rhel/
+ https://www.liquidweb.com/kb/how-to-install-and-configure-snmp-on-centos/
+ https://support.managed.com/kb/a2390/how-to-install-snmp-and-configure-the-community-string-for-centos.aspx

> Trabajo realizado por Marouane Boukhriss



