# 4.5 Configure and verify IPSLA 👨🏽‍💻

# Generalidades

**IPSLA (Internet Protocol Service Level Agreement)**

Provee una variedad de herramientas para realizar varias opciones de monitoreo

**`Es una herramienta del IOS que permite monitorear y medir distintos aspectos de la red como ser:`**

- Delay, Jitter, Loss Packets, Connectivity, Website Download Time, Voice Quality y otros

Podemos enviar este tipo de mensajes a un destino en particular y de estos tipos de tráficos podemos monitorear los aspectos de la red

- ICMP, RTP, TCP, UDP, DHCP, HTTP, DNS y FTP

**`Con la información recolectada se puede realizar tshoot, garantizar ciertos niveles de calidad de servicio en el transporte de paquetes, garantizar que nuestros servicios se encuentren siempre disponible.`** 

Ciertos eventos pueden notificarse a un servidor SNMP , por ejemplo que se active una alarma cada vez que un enlace tenga cierto nivel de perdida, cierto nivel de jitter, etc. 

**Casos de uso**

- Probar la calidad de la llamada en nuestra red de telefonía IP y analizar la calidad antes de instalar los teléfonos reales
- Probar cuanto tiempo se tarda en solicitar una IP a un server DHCP
- Probar cuanto tardar un dispositivo en realizar una conexión TCP con un servidor
- Pruebas para determinar si nuestra red se encuentra lista para una cantidad determinada de equipos de video
- Analizar perdidas de paquetes en un enlace

Con la medición que realiza IP SLA se puede tomar una acción al momento de detectar alguna falla. 

# Operación SLA

**Definir que tipo de trafico** vamos a enviar a un cierto destino, para medir ciertas características de la red en base al trafico enviado

- Que tan frecuente enviaremos el trafico?
- Que es lo que queremos recolectar ?

De acuerdo a la operación que realizaremos, `**se necesitara un SLA Responder**`

Se necesita este dispositivo para poder completar **ciertas operaciones SLA que no son entendidas por todos los dispositivos, en esos casos necesitamos un SLA responder, es decir necesitamos un equipo cisco al otro extremo para que pueda responder a nuestra operacion SLA.**

El tipo de operación que podamos configurar en nuestros equipos, depende de nuestra plataforma

- UDP Jitter, UDP Jitter para voip, UDP Echo, HTTP, **ICMP Echo**, ICMP Path Echo, ICMP Path Jitter, etc

Las que si necesitan un SLA responder son:

- UDP Jitter
- UDP Jitter para VOIP

## ICMP Echo

Monitorea el tiempo de respuesta end-to-end entre un dispositivo Cisco y un dispositivo que se encuentre utilizando IPv4 y v6

![Untitled](4%205%20Configure%20and%20verify%20IPSLA%20%F0%9F%91%A8%F0%9F%8F%BD%E2%80%8D%F0%9F%92%BB%209553e25ac5734cab8040d43084e02a58/Untitled.png)

**Configuración de IP SLA**

```jsx
**Switch(config)# ip sla 1  ## Crear el ip sla** 
Switch(config-ip-sla)# icmp-echo 172.16.10.3 source-interface giga 0/1  ## Indicar el destino y la interface origen que enviara la operacion 
Switch(config-ip-sla-echo)# frequency 5  ## Definir la frecuencia con la que se realizara la operacion

Switch(config)# ip sla schedule 1 life forever star-time now ## Indica cuando iniciara la operacion y hasta cuando
```

**Verificación de IP SLA**

```jsx
**Switch# show ip sla configuration
Switch# show ip sla statistics**
```

# IP SLA with Track

Activar o desactivar rutas estáticas en base al resultado de la operación echo

En este ejemplo se tienen dos rutas estáticas por defecto, la cual la primera sale por el R1 y en caso de falla debe salir por R2

```jsx
**R3# show runn | sec route
ip route 0.0.0.0 0.0.0.0 10.10.20.1
ip route 0.0.0.0 0.0.0.0 10.10.10.1 10** 
```

![Untitled](4%205%20Configure%20and%20verify%20IPSLA%20%F0%9F%91%A8%F0%9F%8F%BD%E2%80%8D%F0%9F%92%BB%209553e25ac5734cab8040d43084e02a58/Untitled%201.png)

**Configuración de IP SLA**

```jsx
**R3(config)# ip sla 1** 
**R3(config-ip-sla)# icmp-echo 172.16.10.3 source interface fastether 0/0**
**R3(config-ip-sla-echo)# frequency 5 
R3(config-ip-sla-echo)# end**

**R3(config)# ip sla 1 schedule start-time now life forever**
```

Cuando falle el ping hacia el servidor, con IP SLA vamos a tener registro de esa falla, ahora debemos configurar el track

Track nos sirve para rastrear ciertos eventos y tomar acciones en base a el rastreo realizado

**Configuración de Track**

```jsx
**R3(config)# track 20 ip sla 1 reachability   ## Asociar el track al SLA que hemos configurado**

```

Ahora debemos indicar que acción tomara el track en caso de que IP SLA detecte una falla, en caso de falla eliminara la primera ruta que se tenia hacia Internet por el R1 y se activara la ruta estática por defecto de backup que se tiene configurado

```jsx
ip route 0.0.0.0 0.0.0.0 10.10.20.1 **track 20**  # Asociamos el track a la ruta que se hara el seguimiento
ip route 0.0.0.0 0.0.0.0 10.10.10.1 10 
```