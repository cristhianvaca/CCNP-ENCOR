# 3.1.a Troubleshoot static and dynamic 802.1q trunking protocols

## **VTP (Vlan Trunking Protocol)**

Es un protocolo de capa 2.

VTPv3 permite propagar desde la **1 hasta la 4096**.

Si existen varios VTP Servers, estos servidores reciben las actualizaciones de los otros servers, como un cliente lo hace,  Si el dominio es Versión 3 se debe definir al servidor primario, con el siguiente comando

```
**switch#(config) vtp primary**
```

**VTP Comunicación**

**Los anuncios VTP se envían por multidifusión a la dirección MAC 01:00:0C:CC:CC:CC**

**Summary**

Incluye la versión, dominio y numero de revisión de configuración y timestamp

Se envían cada 300 segundos o cuando una vlan es añadida

**Subset**

Envía toda la información para realizar cambios en la base de datos de VLAN

Se envia cuando ocurre algun cambio en alguna VLAN

**Client Requests**

Se envía solo para solicitar mayor información

Ocurre cuando un switch con un numero de revisión menor ingresa al dominio VTP y ve que existe un numero de revisión mayor la cual se almacena localmente.

**Versiones VTP**

Existen tres versiones de VTP, la versión que viene por defecto es la 1, es la mas simple y tanto la versión 1 y 2 están limitadas a propagar desde la VLAN 1 hasta la 1005

**VTP Versión 3**

**Permite enviar información de las vlan desde la 1 - 1005 (rango normal) y 1006 - 4094 (rango extendido)**                                                                                                           Soporta solo 802.1q              

 Permite compartir la información de VLAN RSPAN   

 Sincroniza con MST  

Para poder configurar el rol de server dentro del dominio vtp, la versión 3 introduce el comando ***vtp primary vlan***

### VTP Configuración

```jsx
**Configuracion**
**### Definir la version VTP**
vtp version |1| - |2| - |3|
**### Definir el dominio VTP**
vtp domain <cisco.com>
**### Definir el modo que estara trabajando el switch**
vtp mode |server| |client| |transparent| |none|
**### Definir una contraseña para el dominio VTP (Opcional)**
vtp password <password>

**Verificacion**
**### Comando para verificar VTP**
show vtp status
show vtp status | i version run|Operating|VLANS|Revision
show vtp devices

**Se puede deshabilitar vtp en un interface**
Ingresar a la interface   interface <name-interface>
													no vtp
```

<aside>
💡 Las interfaces de los switches deben estar en modo trunk .

</aside>

> Es importante que cuando se añade un nuevo switch al dominio VTP este este con el numero de revisión 0 , ya que si ingresa un swtich con un numero de versión mayor al actual, ya que puede ser disruptivo en caso el nuevo server no tenga las vlans que ya están configuradas y estas se borren y exista afectación de servicio.
> 

Si una vlan se borra, los puertos que pertenecían a esta vlan se mueven a la vlan 1 

## Dynamic Trunking Protocol

Los puertos trunk dinámicos se establecen mediante el envió de paquetes para la negociación de puertos y si la negociación es exitosa se forma el enlace trunk.

DTP envía sus mensajes de negociación cada 30 segundos

### **Modos para la formación de un enlace trunk**

**Trunk**

Es un modo estatico el cual  define al puerto como modo trunk directamente

Para colocar un puerto como mode trunk se realiza con el comando ***swtichport mode trunk***

**Dynamic Desirable**

El puerto actua como si estuviese en modo acceso , pero envia y escucha los paquetes DTP para poder establecerse como modo trunk

Se configura con el sgte comando ***switchport mode dynamic desirable***

**Dynamic Auto**

El puerto actua como si estuviese en modo acceso, pero escucha los paquetes DTP para la formacion del puerto como modo trunk

Se configura con el sgte comando ***switchport mode dynamic auto***

**Configuración DTP**

```
SW1# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
SW1(config)# interface gi1/0/2
***SW1(config-if)# switchport mode dynamic auto***

SW2# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
SW2(config)# interface gi1/0/1
***SW2(config-if)# switchport mode dynamic desirable

Comandos de verificacion***
SW1(config)# show interfaces trunk
SW1(config)# show interfaces <interface> switchport | i Trunk 

Si se configura de manera estatica el puerto es recomendable
deshabilitar la negociacion del puerto
***Switch(config-if) switchport nonegotiate***
```