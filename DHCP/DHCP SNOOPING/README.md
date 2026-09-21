# DHCP Snooping

## Descripción del lab

Este laboratorio tiene como propósito mostrar el funcionamiento de la característica **DHCP Snooping** en un switch Cisco. El escenario presenta una red con dos clientes, un **Enterprise DHCP Server** legítimo y un **Rogue DHCP Server** no autorizado. El objetivo es demostrar cómo, mediante DHCP Snooping, un switch puede identificar el puerto confiable (**trust**) hacia el servidor DHCP legítimo y bloquear las respuestas del servidor DHCP no autorizado, evitando así que los clientes reciban configuraciones de red maliciosas.

## Topología

![Topología](images/Pasted%20image%2020260921121142.png)

Material proporcionado por **David Bombal**

## Comprobación del problema

Antes de aplicar cualquier configuración, comprobé el estado de los clientes para confirmar si estaban recibiendo la dirección IP del servidor correcto o del servidor no autorizado.

Al revisar la configuración IP de PC1 y PC2, observé que ambos equipos habían recibido una dirección dentro de la red 10.1.100.0/24, que corresponde al Rogue DHCP Server y no a la red legítima (10.1.1.0/24).

![Configuración IP de los clientes](images/Pasted%20image%2020260921121357.png)

Esto representa un problema grave, ya que el Rogue DHCP Server se configuró a sí mismo como **default gateway**. Como resultado, todo el tráfico de los clientes pasaría primero por este servidor antes de ser reenviado al gateway legítimo, permitiendo así un posible ataque de tipo **man-in-the-middle**.

Después, revisé la configuración del servicio **DHCP** en ambos servidores para confirmar el origen del problema.

En el Rogue DHCP Server, confirmé que el pool asignaba direcciones de la red 10.1.100.0/24 y establecía como default gateway 10.1.100.254 (el propio servidor):

![Configuración del Rogue DHCP Server](images/Pasted%20image%2020260921121436.png)

En el Enterprise DHCP Server, en cambio, verifiqué que el pool asignaba direcciones de la red legítima 10.1.1.0/24, con el default gateway correcto (10.1.1.254):

![Configuración del Enterprise DHCP Server](images/Pasted%20image%2020260921121422.png)

## Configuración de DHCP Snooping

1. Confiar únicamente en el Enterprise DHCP Server.
2. Bloquear el Rogue DHCP Server.

Para resolver el problema, configuré DHCP Snooping en el switch S1 y marqué como **trust** únicamente el puerto conectado al Enterprise DHCP Server (Fa0/2):

```cmd
S1(config)#ip dhcp snooping
S1(config)#ip dhcp snooping vlan 1
!
S1(config)#int f0/2
S1(config-if)#ip dhcp snooping trust
```

De este modo, el switch acepta mensajes de servidor DHCP (como **DHCPOFFER** y **DHCPACK**) únicamente a través del puerto Fa0/2. Cualquier mensaje de este tipo recibido por otro puerto, como el conectado al Rogue DHCP Server, es descartado automáticamente.

## Verificación

En el siguiente gif se puede observar cómo los mensajes del servidor legítimo, enviados mediante **broadcast**, son reenviados con normalidad al cliente que solicita la dirección IP. Los mensajes del Rogue DHCP Server, en cambio, son denegados por el switch gracias a la configuración de DHCP Snooping aplicada:

![Verificación mediante debug](images/Untitled%20Project.gif)

Finalmente, revisé la tabla de **bindings** de DHCP Snooping en el switch, donde confirmé que ambos clientes (PC1 y PC2) obtuvieron una dirección IP de la red legítima (10.1.1.0/24) a través del Enterprise DHCP Server:

![Tabla de bindings de DHCP Snooping](images/Pasted%20image%2020260921124625.png)

Con esta comprobación, confirmé que la configuración de DHCP Snooping cumplió su objetivo: el switch únicamente confía en el Enterprise DHCP Server y bloquea cualquier respuesta proveniente del Rogue DHCP Server.
