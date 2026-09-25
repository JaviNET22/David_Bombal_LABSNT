# OSPF Multi Area

## Topología

![Topología](images/Pasted%20image%2020260925102200.png)

---

## Descripción del laboratorio

Configurar OSPF en múltiples áreas según se muestra en el diagrama y siguiendo estas indicaciones:
1) Utilizar el **process ID** 1 de OSPF
2) R1 - habilitar OSPF mediante el comando **network** con coincidencia exacta de IP
3) R2 - habilitar OSPF mediante el comando **network** basado en la **subnet mask**
4) R3 - habilitar OSPF mediante el comando de interfaz
5) R4 - habilitar OSPF en todas las interfaces con un único comando **network**

Material proporcionado por **David Bombal**.

---

## Configuración

La única dificultad real de este laboratorio consiste en configurar cada interfaz del router en el área correspondiente. Si una interfaz no queda asignada al área correcta, OSPF no puede funcionar.

Por ello, presto especial atención a los **Area Border Routers** (ABR), que son los routers encargados de conectar dos áreas entre sí. Un **Internal Router** no conecta áreas, mientras que un ABR sí lo hace, normalmente uniendo un área con la **Backbone Area**.

```cmd
R1(config)#router ospf 1
R1(config-router)#network 10.1.1.0 0.0.0.255 area 1
R1(config-router)#network 1.1.1.1 0.0.0.0 area 1
!
R2(config)#router ospf 1
R2(config-router)#network 10.1.2.0 0.0.0.255 area 0
R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
R2(config-router)#network 10.1.2.0 0.0.0.255 area 0
!
R3(config)#int g0/1
R3(config-if)#ip ospf 1 area 0
R3(config-if)#int l0
R3(config-if)#ip ospf 1 area 0
R3(config-if)#int g0/0
R3(config-if)#ip ospf 1 area 2
!
R4(config)#router ospf 1
R4(config-router)#network 0.0.0.0 255.255.255.255 area 2
```

---

## Problemas encontrados
En este punto me encuentro con una pequeña confusión, ya que algunos comandos generan errores en los routers porque la configuración no se aplica correctamente. Para solucionarlo, reconfiguro los comandos tal y como se muestra a continuación.

```cmd
#Comandos que me estaban causando el problema
R1(config-router)#network 1.1.1.1 255.255.255.255 area 1
R2(config-router)#network 2.2.2.2 255.255.255.255 area 0
!
# Comandos para arreglarlo
R1(config-router)#network 1.1.1.1 0.0.0.0 area 1
R2(config-router)#network 2.2.2.2 0.0.0.0 area 0
```

Esta configuración incorrecta me genera un problema que investigo sin llegar a encontrar una explicación completa. Dejo aquí el mensaje de log que recibo, por si resulta de utilidad. El error indica un **area ID mismatch**, a pesar de que ambas interfaces están configuradas con la misma área (área 1), que es la que corresponde. Por este motivo, no logro identificar la causa exacta del problema en ese momento.

```cmd
%OSPF-4-ERRRCV: Received invalid packet: mismatch area ID, from backbone area must be virtual-link but not found from 10.1.1.1, GigabitEthernet0/0/0
```

Otro problema que encontré fue que al utilizar un comando con la red `0.0.0.0` y la **wildcard mask** `255.255.255.255`, estoy anunciando todas las interfaces del router de una sola vez. Esto provoca que los comandos `network` configurados previamente queden sobrescritos y dejen de aplicarse correctamente. Para solucionarlo, elimino los comandos afectados y los vuelvo a configurar de forma individual.

---

## Verificación

Por último, realizo algunas pruebas de conectividad desde Router1 para comprobar que la comunicación funciona correctamente entre todas las áreas.

![Pruebas de conectividad desde Router1](images/Pasted%20image%2020260925102013.png)
