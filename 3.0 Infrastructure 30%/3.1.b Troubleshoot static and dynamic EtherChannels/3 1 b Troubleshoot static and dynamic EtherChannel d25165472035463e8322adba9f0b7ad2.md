# 3.1.b Troubleshoot static and dynamic EtherChannels

Esta definido en el protocolo IEEE 802.3ad

Los enlaces físicos pueden ser agregados en un enlace lógico, el cual a nivel lógico solo se veria como si estuviera un solo enlace conectado y STP lo veria como si estuviese un solo enlace.

El enlace lógico tendrá el bandwith de los enlaces fisicos conectados y podrá realizar el balanceo de trafico entre los miembros del port-channel

Se puede realizar a nivel de Layer 2 (access o trunk) y Layer 3 (routed forwarding)

Soporta hasta 8 enlaces fisicos, el protocolo LACP permite conectar mas pero los tendrá en modo standby y solo 8 estarán en modo activo.

### Ventajas

- Reducción de cambios en la topología en caso se apague o encienda un nuevo enlace, ya que todos serian visto como un solo link para la recirculación de topología STP o enrutamiento.
- Se pueden configurar de manera estática o dinámica.

### Desventajas

- No existe un health Integrity Check, si el medio físico sufre alguna degradación  se vera reflejado en el port-channel y lo cual podría causar perdida de paquetes, esto si se configura el port-channel de manera estática.

**Protocolos de Agregación de Enlaces** 

1. **PAGP**
    
    Es el protocolo propietario de Cisco
    
    **Los mensajes para la negociación son enviadas a la dirección Multicast 0100:0CCC:CCCC**
    
    Modos de Puertos
    
    - **Desirable,** Inicia la negociación del etherchannel.
    - **Auto**, no inicia la negociación, solo recibe los mensajes de negociación y si recibe un **desirable** se forma el etherchannel
    
    ```
    **Configuracion**
    
    ```
    

**PagP Silent Mode - Non- Silent Mode**

**Non- Silent Mode**

Desactiva la interface que forma parte del etherchannel si no se detecta trafico

El timer es de 105 segundos, es el tiempo que le toma detectar si tiene trafico

**Silent Mode**

La interface forme parte del etherchannel aunque no reciba trafico  del dispositivo del otro extremo

Espera 15 segundos , antes de establecer el etherchannel

Esta activado por default

1. **LACP**
    
    Protocolo Estándar que fue desarrollado luego del PAGP
    
    **Los mensajes para la negociación son enviadas a la dirección Multicast 0180:C200:0002.**
    
    Modos de Puertos
    
    - **Active**, Inicia la negociación del etherchannel.
    - **Pasive**, no inicia la negociación, solo recibe los mensajes de negociación y si recibe un active, se forma el etherchannel
    
    ```
    **Configuracion
    SW1(config)# interface range gi1/0/1-2
    SW1(config-if-range)# channel-group 1 mode active
    SW1(config-if-range)# interface port-channel 1
    SW1(config-if)# switchport mode trunk
    
    ##################################################
    
    SW2(config)# interface range gi1/0/1-2
    SW2(config-if-range)# channel-group 1 mode passive
    SW2(config-if-range)# interface port-channel 1
    SW2(config-if)# switchport mode trunk**
    ```
    

**LACP Opciones avanzadas**

LACP presenta algunas opciones avanzadas que PAGP no tiene disponible

- **LACP Fast**
    
    El original LACP estandar envía los paquetes cada 30 segundos y tiene un “intervalo muerto” de 90 segundos, 3 veces mas que el por así decirlo “hello-timer”
    
    LACP fast tiene la mejora de enviar el paquete cada **1 segundo y su respuesta es de 3 segundos**.
    
    Su configuración es en las interfaces que forman parte del port-channel
    
    ```
    **### Configuracion**
    SW1(config)# interface range gi1/0/1-2
    SW1(config-if-range)# lacp rate fast
    **### Verificacion**
    show lacp internal
    **SW1# show lacp internal**
    Flags: S - Device is requesting Slow LACPDUs
    **F - Device is requesting Fast LACPDUs**
    A - Device is in Active mode P - Device is in Passive mode
    Channel group 1
    LACP port Admin Oper Port Port
    Port Flags State Priority Key Key Number State
    Gi1/0/1 **F**A bndl 32768 0x1 0x1 0x102 0x3F
    Gi1/0/2 **F**A bndl 32768 0x1 0x1 0x103 0xF
    ```
    
- **LACP Minimun  Member Interface**
    
    ```
    SW1(config)# interface port-channel 1
    SW1(config-if)# port-channel min-links 2
    ```
    
- **LACP Maximum Member Interface**
    
    Cuando se configura esta característica, el otro puerto miembro del port-channel queda en standby
    
    Solo se puede configurar en un solo switch pero es recomendable para temas de tshoot y buenas practicas configurarlo en ambos switches
    
    El switch define que interface queda como Active de acuerdo a LACP port-priority, la interface con menor  LACP port-priority, es preferida.
    
    ```
    SW1(config)# interface port-channel1
    SW1(config-if)# lacp max-bundle 1
    ```
    
- **LACP System Priority**
    
    Identifica que switch es el maestro para un port-channel, es responsable de la eleccion de que puertos de un port-channel estan en modo activo cuando este sobrepasa la cantidad maxima permitida en el port-channel.
    
    <aside>
    💡 La prioridad por defecto es de 32768
    
    </aside>
    
    El switch que tenga la menor system priority es preferido para tomar este rol.
    
    ```
    **SW1# show lacp sys-id**
    32768, 0062.ec9d.c500
    ###########################
    SW1# configure terminal
    Enter configuration commands, one per line. End with CNTL/Z.
    #############################
    **SW1(config)# lacp system-priority 1
    SW1# show lacp sys-id**
    1, 0062.ec9d.c500
    ```
    
- **LACP Interface Priority**
    
    Habilita al switch master escoger cual de las interfaces miembros del port-channel cuando sobrepasan las cantidad interfaces máximas para un port-channel tienen un rol activo.
    
    El puerto con menor valor es preferido.
    
    ```
    Antes de la configuracion del LACP Priority
    **SW1# show etherchannel summary | b Group**
    Group Port-channel Protocol Ports
    ------+-------------+-----------+-----------------------------------------------
    1 Po1(SU) LACP Gi1/0/1(P) Gi1/0/2(H)
    **Configurar el LACP priority dentro del puerto** 
    SW1(config)# interface gi1/0/2
    SW1(config-if)# lacp port-priority 1
    **Verificacion**
    **SW1# show etherchannel summary | b Group**
    Group Port-channel Protocol Ports
    ------+-------------+-----------+-----------------------------------------------
    1 Po1(SU) LACP Gi1/0/1(H) Gi1/0/2(P)
    ```
    

**Comandos de Verificación**

```
**show etherchannel summary
show interfaces port-channel <port-channel>
show etherchannel port

LACP
show lacp neighbor detail
show lacp counters   ### Ver paquetes que se transmiten/reciben, las interfaces asociadas,
y si existiese algun error.
PAGP
show pagp neighbor 
show pagp counters   ### Ver paquetes que se transmiten/reciben, las interfaces asociadas,
clear lacp counters  ### Limpiar el contador**
```

**Consideraciones para el tshoot de Etherchannel**

Se debe tener en cuenta los siguientes puntos que deben coincidir en las interfaces que forman un etherchannel

- Tipo de Puerto, debe estar configurado ya sea L2 o L3
- Modo de Puerto, En L2 debe ser en ambos extremos trunk o access
- Native VLAN, Las interfaces miembros del port-channel deben tener configurada la misma vlan nativa ***switchport trunk native <vlan-id>***
- Allowed VLAN, Las interfaces miembros L2 deben tener configuradas para permitir las mismas vlans, con el comand ***switchport trunk allowed <vlan-id>***
- Speed, Todos los puertos miembros deben tener la misma velocidad
- Duplex, Todos  los puertos miembros deben tener el mismo duplex
- MTU , Los puertos miembros en L3 deben tener el mismo MTU configurado
- Load Interval, Debe estar configurado el mismo load-interval en todos los puertos miembros del port-channel.
- Storm Control, Debe estar configurado el mismo storm-control en todos los puertos miembros del port-channel.

**Load-Balancing Traffic**

Un hash es calculado y los paquetes son enviados a través del enlace basado en ese hash

El hash tiene las siguientes opciones:

- dst-ip: Destination IP address
- dst-mac: Destination MAC address
- dst-mixed-ip-port: Destination IP address and destination TCP/UDP port
- dst-port: Destination TCP/UDP port
- src-dst-ip: Source and destination IP addresses
- src-dest-ip-only: Source and destination IP addresses only
- src-dst-mac: Source and destination MAC addresses
- src-ip: Source IP address
- src-dst-port: Source and destination TCP/UDP ports only

```
show etherchannel load-balance
```