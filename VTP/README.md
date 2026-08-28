# Laboratorio VTP

Este documento explica cómo se configuró VTP (VLAN Trunking Protocol) en tres switches, y cómo se comprobó que el protocolo funciona de forma correcta.

## Topología

![Topología](images/Pasted%20image%2020260828144545.png)

## Objetivo del laboratorio

Se debe configurar VTP siguiendo estas condiciones:

1. Los usuarios deben poder crear VLANs en S1 y en S2, pero no en S3.
2. S2 no debe sincronizar su base de datos de VLANs con los demás switches.
3. El dominio VTP debe llamarse `ccna`.

Al final del laboratorio se debe comprobar lo siguiente:

3. Las VLANs creadas en S1 se replican en S3, pero no en S2.
4. Es posible crear VLANs en S1 y en S2, pero no en S3.

## Configuración de los switches

### Switch 2

```cmd
s2(config)#vtp domain RED
s2(config)#vtp mode transparent
```

![Configuración VTP en S2](images/Pasted%20image%2020260828145025.png)

### Switch 1

```cmd
S1(config)#vtp domain RED
S1(config)#vtp mode server
```

![Configuración VTP en S1](images/Pasted%20image%2020260828145208.png)

### Switch 3

```cmd
S3(config)#vtp domain RED
S3(config)#vtp mode client
```

![Configuración VTP en S3](images/Pasted%20image%2020260828145417.png)

## Configuración de los puertos trunk

**Switch 1**

```cmd
S1(config)#int g1/0/1
S1(config-if)#sw mode trunk
S1(config-if)#switchport trunk allowed vlan 1,10
```

**Switch 2**

```cmd
s2(config)#int range g1/0/1-2
s2(config-if-range)#sw mode trunk
s2(config-if-range)#switchport trunk allowed vlan 1,10
```

**Switch 3**

```cmd
S3(config)#int g1/0/1
S3(config-if)#sw mode trunk
S3(config-if)#switchport trunk allowed vlan 1,10
```

## Pruebas realizadas

### Prueba 1: creación de una VLAN en el servidor VTP

Se añade la VLAN 10 en el switch que actúa como servidor VTP (S1).

```cmd
S1(config)#vlan 10
S1(config-vlan)#name RED10
```

En el switch 2, que está configurado en modo transparente, el número de revisión no aumenta y no se crea ninguna VLAN nueva. Esto confirma que un switch en modo transparente no procesa la información de VTP que recibe.

![Número de revisión sin cambios en S2](images/Pasted%20image%2020260828145706.png)

En el switch 3, configurado en modo cliente, el número de revisión sí aumenta y se crea la VLAN 10, llamada RED10. Esto confirma que un switch en modo cliente acepta y aplica los cambios enviados por el servidor.

![VLAN creada en S3](images/Pasted%20image%2020260828150910.png)

En el switch 2 se confirma que no se ha incrementado el número de revisión y que la VLAN RED10 no se ha añadido.

![Confirmación en S2](images/Pasted%20image%2020260828151029.png)

### Prueba 2: creación de una VLAN en el switch transparente

Se crea una nueva VLAN en el switch 2, que está configurado en modo transparente.

![Nueva VLAN creada en S2](images/Pasted%20image%2020260828151128.png)

Se comprueba que ni el switch 1 ni el switch 3 reciben esta VLAN. Por lo tanto, el número de revisión de ambos switches no cambia. Esto confirma que un switch en modo transparente no distribuye su información de VLANs al resto de la red.

![S1 sin cambios](images/Pasted%20image%2020260828151321.png)
![S3 sin cambios](images/Pasted%20image%2020260828151340.png)

### Prueba 3: intento de creación de una VLAN en el switch cliente

Se comprueba que el switch 3, al estar en modo cliente, no permite crear VLANs nuevas.

![Error al crear VLAN en S3](images/Pasted%20image%2020260828151443.png)

## Comportamiento

Los resultados obtenidos confirman el comportamiento esperado de VTP:

- El servidor (S1) puede crear VLANs y estas se distribuyen a los switches en modo cliente.
- El switch en modo transparente (S2) no recibe ni distribuye información de VLANs, aunque sí puede crear VLANs propias que solo le afectan a él.
- El switch en modo cliente (S3) recibe las VLANs del servidor, pero no puede crear VLANs por sí mismo.
