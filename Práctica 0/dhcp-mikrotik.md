### DHCP
Para nuestra práctica 0, vamos a hacer uso de un servidor DHCP que distribuya de forma automática direcciones IP del pool que especifiquemos. 

Para ello, en Mikrotik vamos a acceder a:

```
ip dhcp-server
```
Y una vez dentro vamos a ejecutar el comando
```
setup
```
A continuación, Mikrotik nos pedirá ciertos datos:

``` dhcp server interface: ```
* Seleccionar en qué interfaz vamos a configurar nuestro servidor DHCP

```dhcp address space:```
* Especificar la red para la que se va a usar el DHCP
  * En nuestro caso será: 192.168.0.40/29
  
```gateway for dhcp network:```
* La puerta de enlace a lo cual los clientes pedirán sus direcciones
  * En nuestro caso será: 192.168.0.41/29

```addresses to give out:```
* Primera y última dirección del pool ó direcciones específicas que van a entregarse mediante DHCP
  * En nuestro caso se darán todas las disponibles para nuestra pequeña red. 

```dns servers:```
* Servidor DNS para la red. 
  * Normalmente es el mismo que el gateway, pero no tiene porqué ser así siempre

```lease time:```
  * Cuánto tiempo se va a mantener esa IP. Puede ser desde unos pocos minutos hasta una cantidad indefinida.

Una vez configurados estos parámetros, nuestro servidor DHCP ya estará funcionando y sólo será necesario que los clientes soliciten una dirección IP.
