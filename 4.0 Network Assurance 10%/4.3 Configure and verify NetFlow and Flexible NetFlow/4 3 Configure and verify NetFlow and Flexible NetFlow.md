# 4.3 Configure and verify NetFlow and Flexible NetFlow

Realiza un análisis del trafico que circula en los dispositivos

**Beneficios de Netflow**

- Estadísticas del trafico que circula en nuestros dispositivos
- Analizar el trafico de aplicaciones y medir el impacto en la red
- Analizar el trafico de enlaces WAN y tomar medidas para evitar la saturación de estos enlaces
- Analizar cuellos de botella
- Validación de QoS
- Detectar anomalías en el trafico de la red

Permite que podamos entender, quien, que, cuando y como fluye el trafico de la red

**`Se considera un flujo a los paquetes que tienen los mismos atributos`**

- IP origen
- IP destino
- Puerto de Origen
- Puerto de Destino
- L3 Protocolo
- ToS
- Interfaz lógica de entrada

**`Netflow nos proporciona estadisticas sobre los flujos`**

Netflow consume recursos adicionales de memoria y procesamiento, la memoria depende de cada dispositivo

**Netflow tiene dos componentes:**

- **Netflow Data Capture**
    
    Captura las estadísticas del trafico
    
- **Netflow Data Export**
    
    Exportar las estadísticas recolectadas a un Netflow colector, como DNA Center, PRTG, Prime
    

Los datos son exportados al Collector de acuerdo a dos criterios 

- **Si el flujo se encuentra inactivo 15 segundos**
- **Si el flujo se encuentra activo por 30 minutos**

**Comandos de configuración**

```jsx
R1# configure terminal
Enter configuration commands, one per line. End with CNTL/Z.

**R1(config)# ip flow-export version 9
R1(config)# ip flow-export destination 192.168.14.100 9999

R1(config)# interface Ethernet0/1
R1(config-if)# ip flow ingress
R1(config-if)# ip flow egress**

Se puede ver los flujos que estan transmitiendo mayor cantidad
de trafico en nuestra interface
**R1(config)# ip flow top-talkers
R1(config-flow-top-talkers)# top <VALOR>
R1(config-flow-top-talkers)# sort-by <bytes|packets>**

```

**Comandos de verificación**

```jsx
**R1# show ip flow interface
R1# show ip flow export
R1# show ip cache flow
R1# show ip flow-top-talkers**
```

# Flexible Netflow

Fue creado para poder analizar trafico de una forma mas especifica que netflow

Se puede enviar hacia varios collectors

Es una herramienta mas poderosa y escalable que netflow 

Componentes de Flexible Netflow

| Nombre del Componente | Descripción |
| --- | --- |
| Flow Records | Combinación de campos clave y no clave.                Hay predefinidos y
registros definidos por el usuario |
| Flow Monitors | Aplicado a la interface en la cual se monitoreara el trafico |
| Flow Exporters | Exportar el dato de netflow monitor cache hacia el netflow collector |
| Flow Samplers | Muestrea datos parciales de NetFlow en lugar de analizar todos los datos de NetFlow |

En términos de consumo de CPU y memoria es menor comparado con netflow

Se puede crear caches individuales para cada tipo de flujo

**Pasos para crear un flow record**

1. Definir el nombre de record
2. Añadir una descripción (Buena practica)
3. Definir los criterios para los campos principales
4. Definir los criterios que serán recolectados

```jsx
## Definir el nombre del record
**R4(config)# flow record CUSTOM1** 
## Definir la descripcion
**R4(config-flow-record)# description Custom Flow Record for IPv4 Traffic** 
 ## Los criterios a analizar
**R4(config-flow-record)# match ipv4 destination address**
## Criterios qeu seran recolectados
**R4(config-flow-record)# collect counter bytes  
R4(config-flow-record)# collect counter packets 
R4(config-flow-record)# exit

show flow record <NOMBRE-RECORD>**
```

**Pasos para configurar el flow exporter**

1. Definir el nombre del flow exporter
2. Definir la descripción del flow exporter
3. Especificar la ip de destino del flow exporter
4. Especificar la versión de netflow
5. Especificar el puerto UDP

```jsx
**R4(config)# flow exporter CUSTOM1
R4(config-flow-exporter)# description EXPORT-TO-NETFLOW-COLLECTOR
R4(config-flow-exporter)# destination 192.168.14.100
R4(config-flow-exporter)# export-protocol netflow-v9
R4(config-flow-exporter)# transport UDP 9999
R4(config-flow-exporter)# exit
R4(config)# exit**
```

**Pasos para configurar el flow monitor**

1. Definir el nombre del flow monitor
2. Definir una descripcion al flow monitor
3. Especificar el flow record que se utilizara
4. Especificar el cache timeout para las conexiones activas
5. Asignar el exporter al monitor

```jsx
**R4(config)# flow monitor CUSTOM1
R4(config-flow-monitor)# description Uses Custom Flow Record CUSTOM1 for IPv4$
R4(config-flow-monitor)# record ?
CUSTOM1 Custom Flow Record for IPv4 Traffic
R4(config-flow-monitor)# record CUSTOM1
R4(config-flow-monitor)# cache active timeout 60
R4(config-flow-monitor)# end

### Comandos de verificacion
R4# show run flow monitor CUSTOM1
R4# show flow monitor CUSTOM1**
```

Luego se debe asignar a la interface en la que estará pasando el trafico que deseamos analizar/recolectar

```jsx
**R4(config)# interface ethernet0/1
R4(config-if)# ip flow monitor ?
CUSTOM1 Uses Custom Flow Record CUSTOM1 for IPv4 Traffic
R4(config-if)# ip flow monitor CUSTOM1 input
R4(config-if)# interface ethernet0/2
R4(config-if)# ip flow monitor CUSTOM1 input
R4(config-if)# end**

```