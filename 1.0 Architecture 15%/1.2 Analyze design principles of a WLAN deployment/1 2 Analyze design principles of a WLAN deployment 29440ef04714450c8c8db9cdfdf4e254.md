# 1.2 Analyze design principles of a WLAN deployment

## 1.2.a Wireless deployment models (centralized, distributed, controller-less, controller based, cloud, remote branch)

**Controller-less**

Se tiene las características de un WLC integradas en un dispositivo de red, ya no seria necesario adquirir un WLC de manera física.

Lo podemos encontrar dentro de un AP o un Switch

**WLC en un AP**

- Cisco Mobility Express, utiliza el software AirOS
- Cisco Embedded Wireless Controller, utiliza IOS XE.
- Deben ser instalados dentro de la capa de acceso
- Flexconnect
- Se puede administrar 100 AP con 2000 clientes.

**Cisco Embedded Wireless Controller**

- Cisco esta promocionando esta solución.
- Soporta hasta 200 AP con 4000 clientes.
- Puede estar conectado al sw de acceso o distribucion.

**Cloud Deployment Mode**

Los AP son administrados directamente desde la Nube.

La solución Meraki es una de las principales de este despliegue.

Los AP meraki son instalados de manera local pero el Controlador Meraki esta en la nube y no tiene limites d AP que pueden administrar.

Los WLC 9800 se pueden instalar en la nube, tiene capacidad de 6000 AP con 64000 clientes.

**Branch Deployment Mode**

Obtener una solución inalámbrica en las oficinas remotas

Un WLC que se encuentre implementado en la oficina remota, si la oficina es grande

Si la oficina no es grande, se puede optar por  **Cisco Embedded Wireless Controller (Solucion donde se integra el WLC dentro de un SW o AP)**

Si la oficina es grande podemos implementar Flexconnect, permite que implementos AP en las oficinas remotas y los AP serán administrados por un WLC en la oficina Central.

Flexconnect envía el trafico de la oficina remota hacia el WLC de la oficina principal, atravesando la WAN y saturando el trafico de este, por lo que no podría ser viable.

Según Cisco, podríamos implementar Flexconnet en la oficina remota. 

- 1 switch de acceso  o switch en stack
- Menos de 50 APs
- Latencia menor a 100ms hacia el WLC

## 1.2.b Location services in a WLAN design

Normalmente cuando desplegamos una red inalámbrica, el enfoque principal es la cobertura de la señal y que abarque el espacio deseado, nunca se toma en cuenta la ubicación que puedan tener los dispositivos, en algunos caso esta información puede ser relevante y la red nos la puede proveer.

**Rastrea dispositivos que se encuentren dentro de nuestra red inalámbrica.**

- Encontrar activos.
- Reducir perdidas o sustracción de activos.
- Mejorar satisfacción de clientes al encontrar sus activos críticos.
- Encontrar Rogue Devices.
- Genera notificaciones cuando se añaden o quitan dispositivos.
- Estrategias de marketing.

Podriamos tener el **foot traffic,** poder rastrear el flujo del dispositivo y de acuerdo a eso, poder ofrecer algún producto, o alguna estrategia de marketing. 

Poder tener los **wifi asset tags**, que se colocan en nuestros activos , y en caso estos salgan de nuestra red inalambrica poder alertarnos y poder rastrearlos.

Tecnicas

Cell of Origin, nos indica la celda donde donde esta el dispositivo.

Distance - Based (Lateration) Techniques.

1. Time of Arrival (ToA)
    
    Se puede encontrar, según la medida de tiempo que le toma a una señal en ser transmitida desde un dispositivo hacia el AP.
    
    Se utiliza la formula de V=D/T. 
    
    Sabríamos la lejanía del dispositivo, pero no la dirección en la que se encuentra, dado que las ondas que propaga el AP es omnidireccional.
    
2. Time Difference of Arrival (TDoA)

Angle - Base (Angulation) Techniques.

**Location Patterning.**

Realiza un muestreo del comportamiento de la señal en un ambiente determinado, con un dispositivo móvil nos desplazamos dentro de la cobertura de red wireless, los AP van a emitir su señal

La señal se comporta de manera unica en cada punto de la red inalambrica

Muestras de señal que se van tomando en diferentes ubicaciones, en cada punto tenemos datos respecto a cada AP.

Software [Ekahau.com](http://Ekahau.com) , es un software popular en el mundo inalambrico.

**RF Fingerprinting**

Utilizado por cisco.

Combina la técnica de Lateration Technique y Location Patterning

Mide los valores de potencia de la señal emitida por un dispositivo inalámbrico (dBm) 

Mientras mas negativo, mayor perdida de señal o mas atenuación.

Un inconveniente de Lateration Technique es que no toma en cuenta la interferencia que pueda existir en el área donde se analiza el valor de RSSI

Para solucionar este inconveniente se combina con Location Patterning

Se puede detectar rogue devices e interferencias que puedan estar afectando a la red, a traves de la potencia que emiten estos dispositivos.

**Plataformas que pueden ser integradas**

- Cisco Primer Infraestructure
- DNA Center
- MSE(Mobility Service Engine)
- Meraki