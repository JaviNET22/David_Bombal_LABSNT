# ACL Lab 2

Material proporcionado por **David Bombal**.

## Topología

![Topología](images/Pasted%20image%2020260904224440.png)

## Descripción del laboratorio

Se deben configurar listas de control de acceso (ACL) siguiendo estas indicaciones:

**Tráfico interno**
- Usar la lista de acceso número 100.
- El PC 1 interno, situado en la subred 10.1.2.0/24, solo puede acceder a los servidores HTTP 1 y 2 de la subred 10.1.1.0/24 mediante HTTP y HTTPS (esto debe lograrse usando únicamente dos líneas en la ACL).
- Ningún otro PC o servidor de la subred 10.1.2.0/24 puede acceder a la subred 10.1.1.0/24 (esta línea debe añadirse de forma explícita; normalmente se haría con la palabra `log` para registrar el tráfico, pero Packet Tracer no admite esta opción).
- Los equipos de la subred 10.1.2.0/24 pueden acceder a cualquier otra red.
- La lista de acceso debe aplicarse en el punto más eficiente del Router1.

**Tráfico externo**
- Usar la lista de acceso número 101.
- Cualquier equipo externo puede acceder a los servidores HTTP internos mediante HTTP o HTTPS.
- Ningún equipo externo puede acceder a la subred de usuarios 10.1.2.0/24 (esta línea también debe añadirse de forma explícita, con la misma salvedad respecto al registro).
- La lista de acceso debe aplicarse en el punto más eficiente del Router1.

**Verificación**
- Comprobar que el PC1 interno puede acceder a los servidores HTTP internos 1 y 2, pero no a los servidores 3 y 4.
- Comprobar que el PC2 interno no puede acceder a los servidores HTTP internos.
- Comprobar que tanto el PC1 como el PC2 internos pueden navegar a cisco.com y facebook.com.
- Comprobar que el PC1 externo puede acceder a ambos servidores internos mediante HTTP y HTTPS, pero no puede hacer ping a los PCs internos.

**Pistas**
1. Pensar en cómo funciona el binario.
2. Pensar en el tráfico DNS.
3. Pensar en el tráfico de retorno de los servidores de Internet.

## Configuración

Esta es la configuración que he aplicado.

### ACL interna (100)

```cmd
Router1(config)#access-list 100 permit tcp host 10.1.2.101 10.1.1.100 0.0.0.1 eq 80
Router1(config)#access-list 100 permit tcp host 10.1.2.101 10.1.1.100 0.0.0.1 eq 443
Router1(config)#access-list 100 deny ip 10.1.2.0 0.0.0.255 10.1.1.0 0.0.0.255
Router1(config)#access-list 100 permit ip 10.1.2.0 0.0.0.255 any
!
Router1(config)#int g0/0/0
Router1(config-if)#ip access-group 100 in
```

### ACL externa (101)

```cmd
Router1(config)#access-list 101 permit tcp 8.8.8.0 0.0.0.255 10.1.1.0 0.0.0.255 eq 80
Router1(config)#access-list 101 permit tcp 8.8.8.0 0.0.0.255 10.1.1.0 0.0.0.255 eq 443
Router1(config)#access-list 101 permit udp host 8.8.8.8 eq 53 any
Router1(config)#access-list 101 permit tcp host 8.8.8.8 eq 53 any
Router1(config)#access-list 101 permit tcp 8.8.8.0 0.0.0.255 10.1.2.0 0.0.0.255 established
Router1(config)#access-list 101 deny ip 8.8.8.0 0.0.0.255 10.1.2.0 0.0.0.255
!
Router1(config)#int g0/0/1
Router1(config-if)#ip access-group 101 in
```

## Verificación

### PC1 interno

Compruebo que desde el PC1 interno se puede acceder a los servidores 1 y 2 mediante HTTP y HTTPS, y que no se puede acceder ni hacer ping a los demás servidores. No incluyo todas las capturas realizadas, ya que serían demasiadas.

![Verificación PC1 - captura 1](images/Pasted%20image%2020260904233023.png)

![Verificación PC1 - captura 2](images/Pasted%20image%2020260904233101.png)

![Verificación PC1 - captura 3](images/Pasted%20image%2020260904233124.png)

![Verificación PC1 - captura 4](images/Pasted%20image%2020260904233149.png)

![Verificación PC1 - captura 5](images/Pasted%20image%2020260904233200.png)

### PC2 interno

Compruebo que el PC2 no puede acceder mediante HTTP ni HTTPS a los servidores de la red 10.1.1.0/24, tal como se ha definido en la ACL, y que tampoco puede hacer ping. No incluyo todas las capturas realizadas para no extender demasiado el documento.

![Verificación PC2 - captura 1](images/Pasted%20image%2020260904233645.png)

![Verificación PC2 - captura 2](images/Pasted%20image%2020260904233657.png)

![Verificación PC2 - captura 3](images/Pasted%20image%2020260904233716.png)

### Conexión a Cisco y Facebook

Compruebo que tanto el PC1 como el PC2 pueden conectarse correctamente a la web de Cisco y a la web de Facebook.

![Conexión a Cisco y Facebook](images/Pasted%20image%2020260905000751.png)

### PC1 externo

Compruebo que el PC1, situado en la red externa, puede comunicarse con los servidores HTTP mediante HTTP y HTTPS, y que no puede hacer ping a los equipos de la red 10.1.2.0/24.

![Verificación PC1 externo - captura 1](images/Pasted%20image%2020260905001946.png)

![Verificación PC1 externo - captura 2](images/Pasted%20image%2020260905002259.png)

![Verificación PC1 externo - captura 3](images/Pasted%20image%2020260905002329.png)

![Verificación PC1 externo - captura 4](images/Pasted%20image%2020260905002102.png)
