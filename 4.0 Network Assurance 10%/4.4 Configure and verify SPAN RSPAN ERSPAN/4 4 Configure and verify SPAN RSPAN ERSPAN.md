# 4.4 Configure and verify SPAN/RSPAN/ERSPAN

# SPAN

<aside>
💡 Puede capturar el trafico local de un switch y enviar el trafico a un puerto para poder analizarlo

</aside>

El destino del trafico que será “copiado” puede ser enviado a un o mas puertos 

**La fuente de los paquetes a analizar puede ser una interface de las siguientes**

- Una o mas interfaces
- Un port-channel
- Una vlan, es decir todo el trafico recibido por el switch asociado a todos los host de dicha vlan, no incluye una SVI)
- Las tramas copiadas a los puertos de destino son copiadas sin información de tags 802.1q y tampoco se copian tramas de control de protocolos de capa 2

**Consideraciones a tener:**

- La mayoría de los switches soporta dos sesiones de SPAN pero los nuevos pueden soportar mas de dos sesiones
- El puerto de origen definido no puede ser reusado en dos sesiones SPAN distintas
- El puerto de destino definido no puede ser reusado en dos sesiones SPAN distintas
- Los puertos pueden ser switched o routed
- Cuando se utiliza SPAN, en el puerto de destino se deshabilita STP

![Untitled](4%204%20Configure%20and%20verify%20SPAN%20RSPAN%20ERSPAN%20f53140b70c154c4fac5d64445820f1fb/Untitled.png)

```jsx
**SW1(config)# monitor session 1 source interface gig 0/1 both**
**SW1(config)# monitor session 1 destination interface gig 0/0**

```

Podemos hacer que el trafico que queremos replicar además del trafico de datos podamos replicar el trafico de control como ser VTP, DTP.

```jsx
**SW1(config)# monitor session 1 source interface gi 0/1**
**SW1(config)# monitor session 1 destination interface Gi 0/0 encapsulation replicate**
```

Si replicamos el trafico de un enlace troncal, este enlace troncal puede estar transmitiendo el trafico de múltiples vlans, **si deseáramos replicar solo el trafico de la vlan que queremos replicar, en el siguiente ejemplo, solo replicaremos el trafico de la vlan 123**

```jsx
**SW1(config)# monitor session 1 source interface gi 0/1**
**SW1(config)# monitor session 1 destination interface Gi 0/0 encapsulation replicate**
**SW1(config)# monitor session 1 filter vlan 123**
```

# RSPAN

<aside>
💡 Puede capturar el trafico de la red de un switch remoto y enviar una copia del trafico a través de L2 hacia un puerto local que este conectado nuestro analizador de trafico

</aside>

Se debe utilizar una vlan especial para el trafico SPAN, una vlan RSPAN

Esa vlan no debe estar asociada en ningún puerto, no puede ser la vlan nativa ni de administración

Se debe crear la vlan en ambos switches

![Untitled](4%204%20Configure%20and%20verify%20SPAN%20RSPAN%20ERSPAN%20f53140b70c154c4fac5d64445820f1fb/Untitled%201.png)

```jsx
Configuracion en ambos switches 
SW1(config)# vlan 999
SW1(config-vlan)# name RSPAN_VLAN
SW1(config-vlan)# remote-span

SW2(config)# vlan 999
SW2(config-vlan)# name RSPAN_VLAN
SW2(config-vlan)# remote-span
```

Se debe configurar la interface la cual se quiere replicar el trafico (origen) y el destino sera la vlan que hemos creado

```jsx
SW1(config)# monitor session 1 source interface gi 0/1
SW1(config)# monitor session 1 destination remote vlan 999
```

En el otro switch se debe configurar como interface de origen la vlan que hemos creado y como destino el sniffer en la que analizaremos el trafico replicado

```jsx
SW2(config)# monitor session 1 source remote vlan 999
SW2(config)# monitor session 1 destination interface gig 0/0
```

**Comandos de verificación**

```jsx
SW1# show monitor session remote
Session 1
--------Type                     : Remote Source
Session
Source Ports            
: Both                 : Gig 0/1
Dest RSPAN VLAN          :999
```

```jsx
SW2# show monitor session 1
Session 1
--------Type                     : Remote Destination
Session
Source RSPAN VLAN        :
999
Destination Ports        :
Gi 0/0
    Encapsulation        :
Native
          Ingress        :
Disabled
```

# ERSPAN

<aside>
💡 Puede captura trafico de red de un dispositivo remoto y enviarlo el trafico a un sistema local a través de un L3 hacia un puerto local que este conectado nuestro analizador de trafico

</aside>

Caracteristica **`presente en los sistemas operativos IOS XE y equipos ASR`**, permite enviar trafico SPAN a swtiches que pertenecen a otro dominio de broadcast utilizando la tecnica de doble encapsulacion (Tuneles GRE) 

![Untitled](4%204%20Configure%20and%20verify%20SPAN%20RSPAN%20ERSPAN%20f53140b70c154c4fac5d64445820f1fb/Untitled%202.png)

**Configuracion de ERSPAN SOURCE**

```jsx
XE-R2# configure terminal
XE-R2(config)# monitor session 1 type erspan-source
**XE-R2(config-mon-erspan-src)# source interface giga 2 both ## Interface que sera analizada y replicada su trafico**
XE-R2(config-mon-erspan-src)# no shutdown
XE-R2(config-mon-erspan-src)# destination 
XE-R2(config-mon-erspan-src-dst)# ip add 1.1.1.1 ## ip address de destino del tunel
XE-R2(config-mon-erspan-src-dst)# origin ip add 2.2.2.2  ## ip add de origen del tunel 
XE-R2(config-mon-erspan-src-dst)# erspan-id 101 ## id del ERSPAN 
XE-R2(config-mon-erspan-src-dst)# mtu 1460   ## Recomendado por que es un tunnel y para que no se aumente bytes en el transcurso y pueda causar fragmentacion
```

**Configuracion de ERSPAN DESTINATION**

```jsx
XE-R1# configure terminal
XE-R1(config)# monitor session 1 type erspan-destination
XE-R1(config-mon-erspan-dst)# **no shutdown**
**XE-R1(config-mon-erspan-dst)# destination inteface giga 2  ## Interface donde esta conectado nuestro sniffer y donde sera enviado el trafico replicado
XE-R1(config-mon-erspan-dst)# source** 
XE-R1(config-mon-erspan-dst-src)# erspan-id 101 ## id del ERSPAN
XE-R1(config-mon-erspan-dst-src)#  ip add 1.1.1.1

```