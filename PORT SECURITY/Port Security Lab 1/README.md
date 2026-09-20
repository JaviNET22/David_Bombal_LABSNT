# Port Security Lab 1

## Topología

![Topología](images/Pasted%20image%2020260920212917.png)

Material proporcionado por **David Bombal**.

---

## Ejercicio 1 — Port Security básico en G1/0/1

Objetivo: activar Port Security con un único comando y observar su comportamiento por defecto.

```cmd
Switch(config)#int g1/0/1
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
```

Después de aplicar la configuración, hago ping desde PC1 hacia el servidor. El primer ping se completa sin ningún problema. Al entrar en el switch y ejecutar el comando `show port security interface` sobre la interfaz, compruebo que el switch ha aprendido la MAC address de PC1 de forma automática. Observo también que el aging time es cero y que, por defecto, el número máximo de MAC addresses permitidas es uno.

![Estado de Port Security tras el ping de PC1](images/Pasted%20image%2020260920202608.png)
![Detalle de la MAC address aprendida](images/Pasted%20image%2020260920202659.png)

A continuación, hago ping desde PC2 hacia el mismo servidor. En este caso, el puerto pasa al estado **err-disabled**, ya que el modo de violación por defecto es **shutdown**, y esto provoca que el puerto deje de funcionar. Al ejecutar de nuevo `show port security interface`, confirmo que el máximo sigue siendo uno, que la última MAC address detectada corresponde a PC2 y que el puerto se encuentra bloqueado.

![Puerto en estado err-disabled tras el ping de PC2](images/Pasted%20image%2020260920202836.png)
![Detalle del bloqueo del puerto](images/Pasted%20image%2020260920202855.png)

Al revisar la running config, compruebo que ninguna MAC address ha quedado guardada. Sin embargo, con el comando `show port security address` sí puedo ver la MAC address aprendida de forma dinámica. Esto ocurre porque, para que una MAC address aprendida dinámicamente pase a formar parte de la running config, y pueda guardarse posteriormente mediante `write memory` o `copy running-config startup-config`, es necesario activar la propiedad **sticky**. Esta propiedad es la que permite convertir las MAC addresses aprendidas de forma dinámica en entradas persistentes dentro de la running config.

![Comparación entre running config y MAC address aprendida](images/Pasted%20image%2020260920203413.png)

Tras guardar la configuración y reiniciar el switch, compruebo que Port Security continúa activo en la interfaz, ya que la activación del comando sí ha quedado guardada. Sin embargo, la MAC address no se conserva, puesto que nunca llegó a formar parte de la running config y, por tanto, no pudo incluirse en la startup config.

Como consecuencia, si tras el reinicio PC2 es el primero en enviar tráfico hacia el servidor, el switch asocia su MAC address al puerto, y PC1 queda entonces sin poder enviar tráfico por esa misma interfaz. Considero que este comportamiento puede resultar tanto una ventaja como una desventaja: es una desventaja si se pretende que un dispositivo concreto conserve el acceso tras un reinicio, pero es una ventaja si, tras reiniciar el switch, se desea conectar un nuevo dispositivo sin restricciones.

![Comportamiento tras el reinicio del switch](images/Pasted%20image%2020260920205019.png)

---

## Ejercicio 2 — Port Security con sticky en G1/0/3

Objetivo: activar Port Security en G1/0/3 y añadir la MAC address a la running config de forma automática mediante sticky.

```cmd
Switch(config)#int g1/0/3
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
Switch(config-if)#switchport port-security mac-address sticky
```

Tras aplicar estos comandos con la propiedad **sticky**, compruebo que la MAC address aprendida de forma dinámica se convierte en una secure MAC address y queda registrada directamente en la running config, tal y como se observa en las siguientes capturas.

![Configuración de sticky en G1/0/3](images/Pasted%20image%2020260920205859.png)
![MAC address registrada en la running config](images/Pasted%20image%2020260920210022.png)

Como era de esperar, al hacer ping hacia el servidor desde PC4, el puerto pasa al estado **err-disabled**. El switch registra además el siguiente mensaje en la consola:

```
%LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet1/0/3, changed state to down

%PM-4-ERR_DISABLE: psecure-violation error detected on Gig1/0/3, putting Gig1/0/3 in err-disable state.

%PORT_SECURITY-2-PSECURE_VIOLATION: Security violation occurred, caused by MAC address 00C0.4444.4444 on port GigabitEthernet1/0/3.
```

![Log de violación de Port Security](images/Pasted%20image%2020260920210221.png)

Finalmente, reinicio el switch sin guardar la configuración, tal y como se indica en el enunciado del ejercicio. Tras el reinicio, compruebo que no se conserva ninguna configuración: ni el modo access, ni Port Security en la interfaz. En consecuencia, ambos PCs conectados a esta interfaz pueden enviar tráfico sin ninguna restricción.

![Interfaz sin configuración tras el reinicio](images/Pasted%20image%2020260920210721.png)
![Confirmación de acceso sin restricciones](images/Pasted%20image%2020260920211013.png)

---

## Ejercicio 3 — MAC address manual y modo restrict en G1/0/4

Objetivo: activar Port Security en G1/0/4 especificando manualmente la MAC address de PC5, descartando el resto del tráfico y generando mensajes de log ante cualquier violación.

```cmd
Switch(config)#int g1/0/4
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
Switch(config-if)#switchport port-security mac-address 00C0.5555.5555
Switch(config-if)#switchport port-security violation restrict
```

Tras aplicar esta configuración, hago ping desde un PC cuya MAC address no coincide con la configurada de forma manual. Como consecuencia, aparece un mensaje de log en la terminal del switch. Al ejecutar `show port security interface` sobre la interfaz, compruebo que el contador total de violaciones ha aumentado y que queda registrada la última MAC address que ha intentado enviar tráfico por el puerto.

![Violación de Port Security en modo restrict](images/Pasted%20image%2020260920212026.png)

---

## Ejercicio 4 — Aumento del número máximo de dispositivos en G1/0/1

Objetivo: aumentar a dos el número de dispositivos permitidos en G1/0/1 y comprobar que ambos PCs reciben dirección IP mediante DHCP.

```cmd
Switch(config-if)#switchport port-security maximum 2
```

Tras aplicar este comando en la interfaz correspondiente, compruebo que el número máximo de MAC addresses permitidas ha aumentado a dos. Al hacer ping desde PC2, el tráfico se transmite correctamente, lo que confirma que ambos dispositivos pueden operar simultáneamente en la misma interfaz sin generar ninguna violación de Port Security.

![Límite máximo de MAC addresses actualizado](images/Pasted%20image%2020260920212636.png)
![Ping exitoso desde PC2](images/Pasted%20image%2020260920212743.png)
