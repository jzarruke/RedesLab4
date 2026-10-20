# LABORATORIO 4 — OSPF
## Sección 6 · Grupo 1 (X = 6, Y = 1)
### Actividad 6.2 · Topología 1 — IPv4 (Guía 3)

---

## 1. TOPOLOGÍA

### Mapa de la topología

```
   [PC-1]                                                          [PC-2]
192.168.30.16                                                  192.168.40.16
      |                                                               |
  [Switch0]                                                       [Switch1]
      | 192.168.30.0/24                               192.168.40.0/24 |
      |                                                               |
 G0/1 [R2] G0/0 ---- 192.168.10.0/24 ---- G0/0 [R1] G0/1 ---- 192.168.20.0/24 ---- G0/0 [R3] G0/1
      |                                                                                |
  Loopback2: 172.16.0.0/16                                               Loopback3: 172.17.0.0/16
```

Es la misma topología de RIPv1: solo cambia el protocolo de enrutamiento.

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red |
|---|----------|---------------|-----|
| 1 | R1 **G0/0** → R2 **G0/0** | Copper Cross-Over | 192.168.10.0/24 |
| 2 | R1 **G0/1** → R3 **G0/0** | Copper Cross-Over | 192.168.20.0/24 |
| 3 | R2 **G0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 192.168.30.0/24 |
| 4 | Switch0 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 192.168.30.0/24 |
| 5 | R3 **G0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 192.168.40.0/24 |
| 6 | Switch1 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 192.168.40.0/24 |
| 7 | R2 **Loopback2** → (virtual) | — | 172.16.0.0/16 |
| 8 | R3 **Loopback3** → (virtual) | — | 172.17.0.0/16 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

---

## 2. TABLA DE DIRECCIONAMIENTO (X=6, Y=1)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|--------------|---------|---------|
| R1 | G0/0 | 192.168.10.1 | 255.255.255.0 | N/A |
| R1 | G0/1 | 192.168.20.1 | 255.255.255.0 | N/A |
| R2 | G0/0 | 192.168.10.2 | 255.255.255.0 | N/A |
| R2 | G0/1 | 192.168.30.1 | 255.255.255.0 | N/A |
| R2 | Loopback2 | 172.16.6.1 | 255.255.0.0 | N/A |
| R3 | G0/0 | 192.168.20.2 | 255.255.255.0 | N/A |
| R3 | G0/1 | 192.168.40.1 | 255.255.255.0 | N/A |
| R3 | Loopback3 | 172.17.6.1 | 255.255.0.0 | N/A |
| PC-1 | NIC | 192.168.30.16 | 255.255.255.0 | 192.168.30.1 |
| PC-2 | NIC | 192.168.40.16 | 255.255.255.0 | 192.168.40.1 |

> El cálculo de cada dirección está en `Lab4_RIPv1_Seccion6_Grupo1.md`.

---

## 3. CONCEPTO OSPF

OSPF es un protocolo de **estado de enlace (link-state)** que trabaja dentro de un mismo sistema autónomo (IGP). Cada router arma una base de datos con el estado de todos los enlaces y calcula las mejores rutas con el algoritmo SPF de Dijkstra. La métrica es el **costo**, basado en el ancho de banda (10^8 / ancho de banda en bps).

| Aspecto | Detalle |
|---------|---------|
| Tipo | Link-state (IGP) |
| Métrica | Costo (10^8 / ancho de banda); GigabitEthernet y FastEthernet = 1 |
| Actualizaciones | Solo ante cambios de topología (LSA); Hellos cada 10 s |
| Classless | Sí — usa wildcard mask |
| Transporte | IP protocolo 89 |
| Multicast | 224.0.0.5 (todos los routers OSPF), 224.0.0.6 (DR/BDR) |
| Rutas en tabla | Marcadas con **O** |
| Distancia administrativa | 110 |

**Wildcard mask** = 255.255.255.255 − máscara de subred:
- /24 → `255.255.255.0` → wildcard **`0.0.0.255`**
- /16 → `255.255.0.0` → wildcard **`0.0.255.255`**

---

## 4. CONFIGURACIÓN COMPLETA OSPF

> Si vienen de RIPv1 en los mismos routers, primero borrar RIP con `no router rip`. Si no, las rutas O (AD 110) reemplazan a las R (AD 120), pero RIP sigue enviando tráfico y ensucia la captura.

### PASO 1 — Configuración de R1

```
enable
configure terminal

hostname R1
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0

end
write memory
```

> R1 no tiene loopback. Su Router-ID será **192.168.20.1** (la IP más alta de una interfaz activa).

---

### PASO 2 — Configuración de R2

```
enable
configure terminal

hostname R2
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback2
 ip address 172.16.6.1 255.255.0.0

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.255.255 area 0

end
write memory
```

> Su Router-ID será **172.16.6.1**: si hay loopbacks, OSPF toma la IP de loopback más alta.

---

### PASO 3 — Configuración de R3

```
enable
configure terminal

hostname R3
no ip domain-lookup

interface GigabitEthernet0/0
 ip address 192.168.20.2 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown

interface Loopback3
 ip address 172.17.6.1 255.255.0.0

router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 172.17.0.0 0.0.255.255 area 0

end
write memory
```

> Su Router-ID será **172.17.6.1**.

---

### PASO 4 — Configuración de PCs

**PC-1:**
```
IP Address  : 192.168.30.16
Subnet Mask : 255.255.255.0
Gateway     : 192.168.30.1
```

**PC-2:**
```
IP Address  : 192.168.40.16
Subnet Mask : 255.255.255.0
Gateway     : 192.168.40.1
```

---

## 5. VALIDACIÓN

### PASO 5a — Conectividad local antes de OSPF

Igual que en RIPv1: pings de cada PC a su gateway y entre routers vecinos. Sin OSPF, `show ip route` solo muestra redes C y L y PC-1 no llega a PC-2.

### PASO 5b — Vecinos OSPF (esperar unos 10 a 40 segundos)

```
R1# show ip ospf neighbor
Neighbor ID     Pri   State      Dead Time   Address         Interface
172.16.6.1        1   FULL/BDR   00:00:35    192.168.10.2    GigabitEthernet0/0
172.17.6.1        1   FULL/BDR   00:00:35    192.168.20.2    GigabitEthernet0/1
```

```
R2# show ip ospf neighbor
Neighbor ID     Pri   State      Dead Time   Address         Interface
192.168.20.1      1   FULL/DR    00:00:35    192.168.10.1    GigabitEthernet0/0
```

```
R3# show ip ospf neighbor
Neighbor ID     Pri   State      Dead Time   Address         Interface
192.168.20.1      1   FULL/DR    00:00:35    192.168.20.1    GigabitEthernet0/0
```

> Lo obligatorio es el estado **FULL**. El rol DR/BDR depende del orden de arranque: si los routers arrancan al tiempo, gana el Router-ID más alto (R1, 192.168.20.1). Si uno arrancó antes, puede quedar al revés, y eso no es un error. Si se ve INIT, 2WAY o EXSTART, hay un problema (ver sección 6).

### PASO 5c — Tabla de enrutamiento

> OSPF anuncia las interfaces loopback como rutas de host **/32**, no con la máscara /16 configurada. Por eso en los vecinos aparece `172.16.6.1/32`, no `172.16.0.0/16`.

```
R1# show ip route
     172.16.0.0/32 is subnetted, 1 subnets
O       172.16.6.1 [110/2] via 192.168.10.2, GigabitEthernet0/0
     172.17.0.0/32 is subnetted, 1 subnets
O       172.17.6.1 [110/2] via 192.168.20.2, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
O    192.168.30.0/24 [110/2] via 192.168.10.2, GigabitEthernet0/0
O    192.168.40.0/24 [110/2] via 192.168.20.2, GigabitEthernet0/1
```

```
R2# show ip route
C    172.16.0.0/16 is directly connected, Loopback2
     172.17.0.0/32 is subnetted, 1 subnets
O       172.17.6.1 [110/3] via 192.168.10.1, GigabitEthernet0/0
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
O    192.168.20.0/24 [110/2] via 192.168.10.1, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
O    192.168.40.0/24 [110/3] via 192.168.10.1, GigabitEthernet0/0
```

```
R3# show ip route
     172.16.0.0/32 is subnetted, 1 subnets
O       172.16.6.1 [110/3] via 192.168.20.1, GigabitEthernet0/0
C    172.17.0.0/16 is directly connected, Loopback3
O    192.168.10.0/24 [110/2] via 192.168.20.1, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
O    192.168.30.0/24 [110/3] via 192.168.20.1, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

> Se omitieron las líneas L para abreviar. Cálculo del costo: cada enlace Gigabit cuesta 1 y la loopback cuesta 1. Por ejemplo, de R2 a 192.168.40.0 = G0/0 de R2 (1) + G0/1 de R1 (1) + G0/1 de R3 (1) = 3.

### PASO 5d — Prueba final PC a PC

```
PC-1> ping 192.168.40.16   ! PC-2
PC-2> ping 192.168.30.16   ! PC-1
PC-1> ping 172.17.6.1      ! Loopback de R3
PC-2> ping 172.16.6.1      ! Loopback de R2
```

### PASO 5e — Diagnóstico adicional

```
R1# show ip protocols
R1# show ip ospf
R1# show ip ospf interface brief
R1# show ip ospf database
R1# show ip route ospf
R1# show running-config | section router ospf
```

### PASO 5f — Captura de Wireshark

- Filtro para OSPF: `ospf` o `ip.proto == 89`
- Filtro para el ping: `icmp`
- En el informe: mostrar el ICMP entre PC-1 y PC-2 y paquetes OSPF Hello (destino 224.0.0.5, área 0, Router-ID, DR/BDR) y, si se capturan desde el arranque, DBD/LSR/LSU/LSAck.

---

## 6. ERRORES COMUNES OSPF

### Error 1: No aparecen vecinos o quedan en INIT
**Causas:** interfaz apagada, IPs en subredes distintas o un `network` que no cubre la interfaz.
```
R1# show ip interface brief
R1# show ip ospf interface brief
```

### Error 2: Vecino atascado en EXSTART
**Causa:** Router-ID duplicado. Se puede asignar a mano:
```
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
R1# clear ip ospf process
```
(Usar 2.2.2.2 en R2 y 3.3.3.3 en R3; `clear ip ospf process` pide confirmar con `yes`).

### Error 3: Una red no aparece con O
**Causa:** falta el `network` correspondiente. Por ejemplo, si no aparece la loopback de R2:
```
R2(config)# router ospf 1
R2(config-router)# network 172.16.0.0 0.0.255.255 area 0
```

### Error 4: La loopback aparece como /32 y no como /16
No es un error: OSPF anuncia las loopbacks como host /32. Solo si se quisiera ver la /16 se usaría `ip ospf network point-to-point` en la loopback; para este laboratorio no hace falta.

---

## 7. TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | R2 G0/0 | `192.168.10.2` | Siempre |
| R1 | R3 G0/0 | `192.168.20.2` | Siempre |
| R2 | R1 G0/0 | `192.168.10.1` | Siempre |
| R3 | R1 G0/1 | `192.168.20.1` | Siempre |
| PC-1 | Gateway R2 | `192.168.30.1` | Siempre |
| PC-2 | Gateway R3 | `192.168.40.1` | Siempre |

### Conectividad remota (con OSPF en estado FULL)
| Desde | Hacia | IP destino | Resultado esperado |
|-------|-------|-----------|-------------------|
| R1 | PC-1 | `192.168.30.16` | Con OSPF |
| R1 | PC-2 | `192.168.40.16` | Con OSPF |
| R1 | Loopback R2 | `172.16.6.1` | Con OSPF |
| R1 | Loopback R3 | `172.17.6.1` | Con OSPF |
| R2 | PC-2 | `192.168.40.16` | Con OSPF |
| R3 | PC-1 | `192.168.30.16` | Con OSPF |
| **PC-1** | **PC-2** | **`192.168.40.16`** | **Objetivo final** |
| **PC-2** | **PC-1** | **`192.168.30.16`** | **Objetivo final** |

---

## 8. CHECKLIST OSPF

- [ ] Topología construida y mapa de la topología hecho
- [ ] Hostnames R1, R2, R3
- [ ] R1: `router ospf 1` con `network` para 192.168.10.0 y 192.168.20.0
- [ ] R2: `router ospf 1` con `network` para 192.168.10.0, 192.168.30.0 y 172.16.0.0
- [ ] R3: `router ospf 1` con `network` para 192.168.20.0, 192.168.40.0 y 172.17.0.0
- [ ] PC-1: 192.168.30.16/24, gateway 192.168.30.1
- [ ] PC-2: 192.168.40.16/24, gateway 192.168.40.1
- [ ] `show ip ospf neighbor` en R1: 2 vecinos en **FULL**
- [ ] `show ip route` en R1: 4 rutas O (dos /24 y dos /32 de loopback)
- [ ] PC-1 hace ping a PC-2 y PC-2 a PC-1
- [ ] Captura de Wireshark: ICMP exitoso + paquetes OSPF (protocolo IP 89)

---

## 9. CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Para routers con interfaces FastEthernet (Cisco 1841, 2811). Las IPs y los comandos OSPF son los mismos. El costo de FastEthernet también es 1, así que las métricas no cambian.

### R1 — FastEthernet
```
enable
configure terminal
hostname R1
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
end
write memory
```

### R2 — FastEthernet
```
enable
configure terminal
hostname R2
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.10.2 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
interface Loopback2
 ip address 172.16.6.1 255.255.0.0
router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.255.255 area 0
end
write memory
```

### R3 — FastEthernet
```
enable
configure terminal
hostname R3
no ip domain-lookup
interface FastEthernet0/0
 ip address 192.168.20.2 255.255.255.0
 no shutdown
interface FastEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown
interface Loopback3
 ip address 172.17.6.1 255.255.0.0
router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 172.17.0.0 0.0.255.255 area 0
end
write memory
```

---

## 10. BORRAR TODA LA CONFIGURACIÓN

```
enable
write erase
reload
```
> En `Proceed with reload? [confirm]` presionar **Enter**.

Para borrar solo OSPF:
```
enable
configure terminal
no router ospf 1
end
write memory
```
