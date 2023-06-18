# 4.1 Diagnose network problems using tools such as debugs, conditional debugs, trace route, ping, SNMP, and syslog

# **Debug**

Debug es una herramienta del Cisco IOS que causa que el equipo genere mensajes syslogs de acuerdo a los eventos de red.

Los mensajes debug son generados sin importar el nivel que se tenga habilitado para los syslogs. 

Si estamos conectados de forma remota podemos habilitar los mensajes debugs con el comando **`terminal monitor`**

Si estamos conectados por consola, esta habilitado por defecto, podemos deshabilitarlo con el comando **`no logging console`**

```jsx
**R1# debug adjacency
R1# show debug
R1# undebug all    --> Deshabilitar el debug
R1# no debug all   -> Deshabilitar el debug**
```

Cisco recomienda tener cuidado al momento de utilizar el comando debug

- Solo se debe usar bajo la dirección de un TAC cuando se esta realizando tareas de tshoot
- **`Habilitar el debug podría ser disruptivo cuando se tiene alto procesamiento en el equipo, ya que se puede sobrecargar de mensajes.`**
- Se recomienda verificar antes de hacer un debug, la carga de procesamiento del CPU con el comando **`show process cpu`**

# Conditional Debug

Se puede especificar una o varias condiciones para realizar debugs, y solo serán mostrados los mensajes que estén dentro de la condición establecida para el debug 

```jsx
**R1# debup ip packet
R1# debug condition interface g0/1**

```

El comando **`undebug all`**, deshabilita los debugs pero no elimina las condiciones debug que se han declarado

<aside>
💡 Para eliminar un **conditional debug** se realiza con el comando **`undebug condition all`** o no **`debug condition all`**

</aside>

# Syslog (System Loggin)

Es un sistema de envió de mensajes generados para notificarnos de ciertos eventos que ocurren en nuestro dispositivo

Por defecto solamente son mostrados en la conexión por consola

Para activar que nos **`muestre los logs generados cuando estamos conectados vía remota`**

```jsx
**Router# terminal monitor
Para desactivarlo es con el comando
Router# terminal no monitor**
```

**Cuando estamos recibiendo los logs generados, es probable que luego no podamos escribir comando porque se estaría sobrescribiendo a nuestro prompt, para evitar esto podríamos hacer lo siguiente:**

```jsx
**Dentro de la configuracion line console**

**R1(config)#** line console
**R1(config-line)# logging synchronous**

Dentro de la configuracion line vty
**R1(config)#** line vty 0 4
**R1(config-line)# logging synchronous**
```

Si es que no queremos ver los logs que se generan cuando estamos conectados por consola, podemos utilizar el siguiente comando:

```jsx
**R1(config)# no logging console**
```

Estos logs son almacenados en la memoria del dispositivo, podemos definir la cantidad de memoria asignada a estos comandos que se vayan generando, **`si no colocamos ningun valor por defecto tiene un valor de 4096 bytes`**

```jsx
**R1(config)# logging buffered <VALOR>**
```

Lo ideal es tener un servidor **`syslog remoto`**:

```jsx
**R1(config)# logging <IP-HOST-SERVIDOR>**
```

## Estructura de los mensajes Syslog

- **Timestamp**
    - **`Es la fecha y hora`** en la que ocurrió el evento
    - Podemos deshabilitarlo si es que así quisiéramos y en vez de eso habilitar con un numero de secuencia:
        
        ```jsx
        **R1(config)# no service timestamps
        R1(config)# service sequence-numbers** 
        ```
        
- **Facility**
    - Por ejemplo **%LINEPROTO**
    - Hace referencia al **`proceso, protocolo o el modulo`** que ha creado el mensaje syslog.
- **Severity**
    - Hace referencia al nivel de **`severidad del mensaje syslog`**
        
        
        | Nivel | Severity | Description |
        | --- | --- | --- |
        | 0 | Emergencies | El sistema no se puede utilizar |
        | 1 | Alerts | Se necesita una accion inmediata |
        | 2 | Critical | Condiciones criticas |
        | 3 | Errors | Condiciones de error |
        | 4 | Warning | Condiciones de advertencia |
        | 5 | Notifications | Condiciones normales pero significativas |
        | 6 | Informational | Mensajes Informativos |
        | 7 | Debbuging | Mensajes de debug |
    - **Every Awesone Cisco Engineer wants notification information y debugging (Acrónimo para memorizar)**
- **MNEMONIC**
    - **UPDOWN**
    - Es un texto que describe al mensaje
- **Descriptive Message**
    - Descripción mas detallada del mensaje syslog

# Ping

Consiste en que envía paquetes ICMP para verificar si tenemos conectividad a algún dispositivo

```jsx
**R1# ping <IP o NOMBRE DE DOMINIO>**
```

Cuando el ping es exitoso es representado con un signo de admiracion

```jsx
**R1# ping 10.1.12.2**
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.12.2, timeout is 2 seconds:
**!!!!!**
**Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/1 ms**
```

Cuando el ping no es exitoso es representado con un signo punto 

```jsx
**R1# ping 10.1.12.2**
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.1.12.2, timeout is 2 seconds:
**.....**
Success rate is 0 percent (0/5)
```

Podemos ejecutar un ping extendido para ver si es que se pierden paquetes **`icmp`**en el camino 

```jsx
**R1# ping 10.1.12.2 repeat 100**
Type escape sequence to abort.
Sending 100, 100-byte ICMP Echos to 10.1.12.2, timeout is 2 seconds:
**.....................!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Success rate is 79 percent (79/100), round-trip min/avg/max = 1/1/1 ms**
```

Se puede generar con múltiples opciones

```jsx
**R1# ping 22.22.22.22 source loopback 101 size 1500 repeat 10**
Type escape sequence to abort.
Sending 10, 1500-byte ICMP Echos to 22.22.22.22, timeout is 2 seconds:
Packet sent with a source address of 11.11.11.11
**!!!!!!!!!!
Success rate is 100 percent (10/10), round-trip min/avg/max = 1/1/1 ms**
```

# Traceroute

Es utilizado para determinar en donde el trafico se pierde , es decir indica la cantidad de saltos que tiene que atravesar desde el origen hacia el destino 

```jsx
**R1# traceroute 22.22.22.22**
Type escape sequence to abort.
Tracing the route to 22.22.22.22
VRF info: (vrf in name/id, vrf out name/id)
  **1 10.1.12.2 0 msec *  1 msec**

```

Existen puede existir varias razones para que un traceroute pueda ser no exitoso y se mostraría de la siguiente forma

```jsx
**R1# traceroute 22.22.22.23**
Type escape sequence to abort.
Tracing the route to 22.22.22.23
VRF info: (vrf in name/id, vrf out name/id)
  **1  *  *  *
  2  *  *  *
	3  *  *  *
  4  *  *  *
  5  *  *  *
  6  *  *  *
  7  *  *  *
  8  *  *  *
  9  *  *  *
 10  *  *  *
 11  *  *  *
 12  *  *  *
 13  *  *  *
 14  *  *  *
 15  *  *  *
 16  *  *  *
 17  *  *  *
 18  *  *  ***
```

Algunos símbolos que podrían aparecer al momento de realizar un tshoot con el comando traceroute

| Símbolo | Descripción |
| --- | --- |
| ! | Exitoso |
| . | Timeout |
| N | Network inalcanzable |
| H | Host Inalcanzable |
| P | Protocolo Inalcanzable |
| A | Admin Denegado |
| Q | Congestión |
| ? | Paquete desconocido |

# SNMP

SNNP envía traps no solicitados al NMS (Network Management System)

**`Los traps pueden enviar informacion sobre el estado de los enlaces si ocurre algun evento, estado de las fuentes de los equipos, etc.`** 

Los eventos están definidos en el MIB (Management Information Base)

| Versión | Nivel | Autenticación | Encriptación | Resultado |
| --- | --- | --- | --- | --- |
| SNMPv1 | noAuthNoPriv | Community
string | No | Utiliza a una comunidad para la autenticacion |
| SNMPv2c | noAuthNoPriv | Community
string | No | Utiliza a una comunidad para la autenticacion |
| SNMPv3 | noAuthNoPriv | Username | No | Utiliza un usuario para la autenticacion |
| SNMPv3 | authNoPriv | Message Digest 5
(MD5) or Secure
Hash Algorithm
(SHA) | No | HMAC-MD5 or
HMAC-SHA |
| SNMPv3 | authPriv
(requires the
cryptographic
software
image) | MD5 or SHA | (DES) o (AES) | HMAC-MD5 or
HMAC-SHA  User-based
Security Model (USM)                                                                DES 56-bit encryption in addition to authentication
based on the CBC-DES
(DES-56) standard.                                                                 3DES 168-bit encryption
AES 128-bit, 192-bit
256-bit encryption |

SNMPv3 utiliza usuarios y SHA O MD5 para la autenticación, lo cual lo hace muy seguro comparado con sus versiones predecesoras

**`SNMPv1 y v2 utilizan una comunidad para poder autenticar a los dispositivos, la cual puede ser RO(read-only) o RW(read-write)`**

**`RO = recolectar informacion de los equipos`** 

**`RW = permite realizar configuraciones hacia los equipos`**

Un buena practica para limitar el acceso SNMP en los equipos es utilizar listas de acceso, ya que sin estos existe un riesgo potencial de ser atacados por usuarios no autorizados. 

**Por defecto viene habilitada la versión 1** 

## Operación SNMP

| Operación | Descripción |
| --- | --- |
| get-request | Recupera un valor de una variable en especifico |
| get-next-request | Recupera un valor de una variable dentro de una tabla |
| get-bulk-request | Recupera grandes cantidades de datos, como múltiples filas dentro de una tabla |
| get-reponse | Responde a un get-request |
| set-request | Almacena un valor en una variable especifica |
| trap | Envía mensajes no solicitados hacia el SNMP Manager cuando ocurre un evento.  |

![Untitled](4%201%20Diagnose%20network%20problems%20using%20tools%20such%20as%20%20e68bdcd0b45e4025a38cdba8c8cb46d4/Untitled.png)

**Las consideraciones que se deben tener al momento de configurar SNMP**

- Definir que host enviaran traps y el NMS asignado
- Crear una access-list para restringir el acceso
- Definir una comunidad RO
- Definir una comunidad RW
- Definir la locación SNMP
- Definir el contacto SNMP

```jsx
**R4(config)# access-list 99 permit 192.168.14.100 0.0.0.0
R4(config)# snmp-server community READONLY ro 99
R4(config)# snmp-server community READONLY rw 99**
```

Habilitación de los traps SNMP

```jsx
**snmp-server trap-source Loopback0
snmp-server enable traps snmp authentication linkdown linkup coldstart warmstart
snmp-server enable traps ds1
snmp-server enable traps call-home message-send-fail server-fail
snmp-server enable traps tty
snmp-server enable traps eigrp
snmp-server enable traps ospf state-change
snmp-server enable traps ospf errors
snmp-server enable traps ospf retransmit
snmp-server enable traps ospf lsa
snmp-server enable traps ospf cisco-specific state-change nssa-trans-change
snmp-server enable traps ospf cisco-specific state-change shamlink interface-old
snmp-server enable traps ospf cisco-specific state-change shamlink neighbor
snmp-server enable traps ospf cisco-specific errors
snmp-server enable traps ospf cisco-specific retransmit
snmp-server enable traps ospf cisco-specific lsa
snmp-server enable traps license
snmp-server enable traps ike policy add
snmp-server enable traps ike policy delete
snmp-server enable traps ike tunnel start
snmp-server enable traps ike tunnel stop
snmp-server enable traps ipsec cryptomap add
snmp-server enable traps ipsec cryptomap delete
snmp-server enable traps ipsec cryptomap attach
snmp-server enable traps ipsec cryptomap detach
snmp-server enable traps ipsec tunnel start
snmp-server enable traps ipsec tunnel stop
snmp-server enable traps ipsec too-many-sas
snmp-server enable traps atm subif
snmp-server enable traps bfd
snmp-server enable traps bgp
snmp-server enable traps bgp cbgp2
snmp-server enable traps config-copy
snmp-server enable traps config
snmp-server enable traps config-ctid
snmp-server enable traps dhcp
snmp-server enable traps event-manager
snmp-server enable traps hsrp
snmp-server enable traps ipmulticast
snmp-server enable traps isis
snmp-server enable traps msdp
snmp-server enable traps pim neighbor-change rp-mapping-change invalid-pim-message
snmp-server enable traps ipsla
snmp-server enable traps bridge newroot topologychange
snmp-server enable traps stpx inconsistency root-inconsistency loop-inconsistency
snmp-server enable traps syslog
snmp-server enable traps ether-oam
snmp-server enable traps ethernet cfm cc mep-up mep-down cross-connect loop config
snmp-server enable traps ethernet cfm crosscheck mep-missing mep-unknown service-up
snmp-server enable traps memory bufferpeak
snmp-server enable traps entity-state
snmp-server enable traps fru-ctrl
snmp-server enable traps entity
snmp-server enable traps cpu threshold
snmp-server enable traps rep
snmp-server enable traps vtp
snmp-server enable traps vlancreate
snmp-server enable traps vlandelete
snmp-server enable traps sonet
snmp-server enable traps cef resource-failure peer-state-change peer-fib-state-change inconsistency
snmp-server enable traps resource-policy
snmp-server enable traps flash insertion
snmp-server enable traps flash removal
snmp-server enable traps rsvp
snmp-server enable traps aaa_server
snmp-server enable traps ethernet evc status create delete
snmp-server enable traps mvpn
snmp-server enable traps mpls rfc ldp
snmp-server enable traps mpls ldp
snmp-server enable traps mpls traffic-eng
snmp-server enable traps mpls fast-reroute protected
snmp-server enable traps pw vc
snmp-server enable traps l2tun session
snmp-server enable traps l2tun pseudowire status
snmp-server enable traps alarms informational
snmp-server enable traps bulkstat collection transfer
snmp-server enable traps rf
snmp-server enable traps ethernet cfm alarm
snmp-server enable traps transceiver all
snmp-server enable traps mpls vpn
snmp-server enable traps mpls rfc vpn**
```

**Configuracion del SNMP host** 

```jsx
**snmp-server host 200.87.101.180 version 2c Ac3bu7ol0l 
snmp-server host 200.87.101.181 version 2c Ac3bu7ol0l 
snmp-server host 10.57.236.4 version 2c W4eN5o 
snmp-server host 10.57.236.6 version 2c W4eN5o**
```

**Definir las comunidades SNMP y asociarlas a la lista de acceso 30**

```jsx
**snmp-server community Ac3bu7ol0l RO 30
snmp-server community 3nt3ln3t RO 30
snmp-server community Ac374m1n0f3n0 RW 30
snmp-server community c0trmr03c0 RO 30
snmp-server community W4eN5o RO 30**
```

**Definir que interfaz será la que enviara los traps**

```jsx
**snmp-server trap-source Loopback0**
```