# LABORATORIO 4 — Enrutamiento Dinámico IPv4
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Guía completa: RIPv1 → OSPFv2 → BGP

---

## 1. INTERPRETACIÓN DE LA TOPOLOGÍA 1

### Diagrama lógico (Topología 1 — RIPv1 y OSPF)

```
                        [R0]
                    G0/0 /  \ G0/1
                        /    \
               192.168.10.0   192.168.20.0
                      /            \
                  [R1]              [R2]
              G0/0  G0/1        G0/0  G0/1
             /         \      /         \
   192.168.10.0   192.168.30.0  192.168.20.0  192.168.40.0
                       |                        |
                   [Switch0]               [Switch1]
                       |                        |
                    [PC-0]                   [PC-1]
               192.168.30.104           192.168.40.104
```

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red |
|---|----------|---------------|-----|
| 1 | R0 **GE0/0** → R1 **GE0/0** | Copper Cross-Over | 192.168.10.0/24 |
| 2 | R0 **GE0/1** → R2 **GE0/0** | Copper Cross-Over | 192.168.20.0/24 |
| 3 | R1 **GE0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 192.168.30.0/24 |
| 4 | Switch0 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 192.168.30.0/24 |
| 5 | R2 **GE0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 192.168.40.0/24 |
| 6 | Switch1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 192.168.40.0/24 |
| 7 | R1 **Loopback2** → (virtual) | — | 172.16.0.0/16 |
| 8 | R2 **Loopback3** → (virtual) | — | 172.17.0.0/16 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

**Equipos requeridos en Packet Tracer:**
- 3 × Router Cisco 2911
- 2 × Switch Cisco 2960-24TT
- 2 × PC (o Laptop)
- Cables: **Crossover** entre routers (o usar Auto-MDI-X), **Straight-through** de router a switch y de switch a PC

---

## 2. TABLA DE DIRECCIONAMIENTO — VALORES FINALES (X=4, Y=10)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|--------------|---------|---------|
| R0 | G0/0 | **192.168.10.10** | 255.255.255.0 | N/A |
| R0 | G0/1 | **192.168.20.10** | 255.255.255.0 | N/A |
| R1 | G0/0 | **192.168.10.11** | 255.255.255.0 | N/A |
| R1 | G0/1 | **192.168.30.1** | 255.255.255.0 | N/A |
| R1 | Loopback2 | **172.16.4.10** | 255.255.0.0 | N/A |
| R2 | G0/0 | **192.168.20.11** | 255.255.255.0 | N/A |
| R2 | G0/1 | **192.168.40.1** | 255.255.255.0 | N/A |
| R2 | Loopback3 | **172.17.4.10** | 255.255.0.0 | N/A |
| PC-0 | NIC | **192.168.30.104** | 255.255.255.0 | 192.168.30.1 |
| PC-1 | NIC | **192.168.40.104** | 255.255.255.0 | 192.168.40.1 |

### Cálculo paso a paso
| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| 192.168.10.**Y** | Y = 10 | 192.168.10.**10** |
| 192.168.10.**Y+1** | Y+1 = 11 | 192.168.10.**11** |
| 192.168.20.**Y** | Y = 10 | 192.168.20.**10** |
| 192.168.20.**Y+1** | Y+1 = 11 | 192.168.20.**11** |
| 172.16.**X.Y** | X=4, Y=10 | 172.16.**4.10** |
| 172.17.**X.Y** | X=4, Y=10 | 172.17.**4.10** |
| 192.168.30.**YX** | Y=10, X=4 → concat "104" | 192.168.30.**104** |
| 192.168.40.**YX** | Y=10, X=4 → concat "104" | 192.168.40.**104** |

---

## 3. CONFIGURACIÓN COMPLETA — FASE 1: RIPv1

> ⚠️ RIPv1 es **classful**: los comandos `network` usan la dirección de red de clase (sin subnet mask). Esto significa que R2 y R3 anuncian sus loopbacks como redes clase B (172.16.0.0 y 172.17.0.0).

---

### PASO 1 — Construir la topología en Packet Tracer

1. Abrir Packet Tracer → New
2. Agregar: **3 routers Cisco 2911**, **2 switches Cisco 2960-24TT**, **1 PC**, **1 Laptop (como PC-1)**
3. Conexiones (usar **Copper Cross-Over** entre routers, **Copper Straight-Through** para router→switch y switch→PC):

| Desde | Puerto | Hacia | Puerto |
|-------|--------|-------|--------|
| R0 | G0/0 | R1 | G0/0 |
| R0 | G0/1 | R2 | G0/0 |
| R1 | G0/1 | Switch0 | cualquier FastEthernet |
| R2 | G0/1 | Switch1 | cualquier FastEthernet |
| Switch0 | cualquier FastEthernet | PC-0 | NIC |
| Switch1 | cualquier FastEthernet | PC-1 | NIC |

---

### PASO 2 — Configuración de R0

```
enable
configure terminal

hostname R0

interface GigabitEthernet0/0
 ip address 192.168.10.10 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.20.10 255.255.255.0
 no shutdown

router rip
 version 1
 network 192.168.10.0
 network 192.168.20.0

end
write memory
```

---

### PASO 3 — Configuración de R1

```
enable
configure terminal

hostname R1

interface GigabitEthernet0/0
 ip address 192.168.10.11 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown

interface Loopback2
 ip address 172.16.4.10 255.255.0.0
 no shutdown

router rip
 version 1
 network 192.168.10.0
 network 192.168.30.0
 network 172.16.0.0

end
write memory
```

---

### PASO 4 — Configuración de R2

```
enable
configure terminal

hostname R2

interface GigabitEthernet0/0
 ip address 192.168.20.11 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.40.1 255.255.255.0
 no shutdown

interface Loopback3
 ip address 172.17.4.10 255.255.0.0
 no shutdown

router rip
 version 1
 network 192.168.20.0
 network 192.168.40.0
 network 172.17.0.0

end
write memory
```

---

### PASO 5 — Configuración de PCs

#### PC-0
```
IPv4 Address        : 192.168.30.104
Subnet Mask         : 255.255.255.0
Default Gateway     : 192.168.30.1
```
*(En Packet Tracer: click en PC-0 → Desktop → IP Configuration → ingresar valores)*

#### PC-1
```
IPv4 Address        : 192.168.40.104
Subnet Mask         : 255.255.255.0
Default Gateway     : 192.168.40.1
```
*(En Packet Tracer: click en PC-1/Laptop → Desktop → IP Configuration → ingresar valores)*

---

## 4. VALIDACIÓN

### PASO 6a — Verificar conectividad local ANTES de RIP
> Primero confirmar que cada dispositivo llega a su vecino directo (sin enrutamiento dinámico aún activo):

```
ping 192.168.10.11    ! → debe llegar a R1 G0/0
ping 192.168.20.11    ! → debe llegar a R2 G0/0

ping 192.168.10.10    ! → debe llegar a R0 G0/0
ping 192.168.30.104   ! → debe llegar a PC-0 (si ya tiene IP)

ping 192.168.20.10    ! → debe llegar a R0 G0/1
ping 192.168.40.104   ! → debe llegar a PC-1 (si ya tiene IP)

ping 192.168.30.1     ! → debe llegar a R1 G0/1 (gateway)

ping 192.168.40.1     ! → debe llegar a R2 G0/1 (gateway)
```

### PASO 6b — Verificar tabla de enrutamiento RIP (esperar ~30 segundos)

#### En R0 — debería ver rutas R hacia redes 30, 40, 172.16, 172.17:
```
R0# show ip route
```
**Salida esperada:**
```
Codes: C - connected, S - static, R - RIP ...

     172.16.0.0/16 is variably subnetted
R       172.16.0.0/16 [120/1] via 192.168.10.11, GigabitEthernet0/0
R       172.17.0.0/16 [120/1] via 192.168.20.11, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
R    192.168.30.0/24 [120/1] via 192.168.10.11, GigabitEthernet0/0
R    192.168.40.0/24 [120/1] via 192.168.20.11, GigabitEthernet0/1
```

#### En R1 — debería ver rutas R hacia redes 20, 40, 172.17:
```
R1# show ip route
```
**Salida esperada:**
```
R    172.17.0.0/16 [120/2] via 192.168.10.10, GigabitEthernet0/0
C    172.16.0.0/16 is directly connected, Loopback2
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
R    192.168.20.0/24 [120/1] via 192.168.10.10, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
R    192.168.40.0/24 [120/2] via 192.168.10.10, GigabitEthernet0/0
```

#### En R2 — debería ver rutas R hacia redes 10, 30, 172.16:
```
R2# show ip route
```
**Salida esperada:**
```
R    172.16.0.0/16 [120/2] via 192.168.20.10, GigabitEthernet0/0
C    172.17.0.0/16 is directly connected, Loopback3
R    192.168.10.0/24 [120/1] via 192.168.20.10, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
R    192.168.30.0/24 [120/2] via 192.168.20.10, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

### PASO 6c — Verificar configuración RIP
```
R0# show running-config
R1# show running-config
R2# show running-config
```
Buscar sección `router rip` con `version 1` y los `network` correspondientes.

### PASO 6d — Prueba final PC a PC (¡el objetivo principal!)
```
ping 192.168.40.104    ! → PC-1 (debe funcionar con RIP activo)
ping 172.16.4.10       ! → Loopback R1
ping 172.17.4.10       ! → Loopback R2

ping 192.168.30.104    ! → PC-0
```

---

## 5. ERRORES COMUNES Y SOLUCIONES

### Error 1: Interfaces en estado down/down
**Síntoma:** `show ip interface brief` muestra la interfaz como `down/down`
**Causa:** Falta el comando `no shutdown` o cable mal conectado
```
R0# configure terminal
R0(config)# interface GigabitEthernet0/0
R0(config-if)# no shutdown
```
**En Packet Tracer:** Verificar que el cable esté conectado (luz verde en los extremos)

---

### Error 2: RIP no anuncia redes / tabla de rutas vacía
**Síntoma:** `show ip route` no muestra rutas con `R`
**Causa:** Comando `network` mal escrito, o interfaces sin `no shutdown`
```
R1# show running-config | section router rip
```
**Solución:** Volver a entrar al proceso RIP y re-agregar las redes
```
R0# configure terminal
R0(config)# router rip
R0(config-router)# network 192.168.10.0
R0(config-router)# network 192.168.20.0
```

---

### Error 3: Loopback no aparece en tabla de rutas de vecinos
**Síntoma:** 172.16.0.0 ó 172.17.0.0 no aparecen en R0 o R2
**Causa:** Se olvidó agregar `network 172.16.0.0` o `network 172.17.0.0`
```
R1(config)# router rip
R1(config-router)# network 172.16.0.0

R2(config)# router rip
R2(config-router)# network 172.17.0.0
```

---

### Error 4: PC no hace ping a su gateway
**Síntoma:** Ping de PC-0 a 192.168.30.1 falla
**Causas posibles:**
1. IP de la PC mal configurada (verificar en Desktop → IP Configuration)
2. Interfaz G0/1 de R1 apagada (`no shutdown`)
3. PC en VLAN equivocada en el switch (raro en PT básico)
```
R1# show ip interface brief
```

---

### Error 5: PC-0 no llega a PC-1 aunque los routers sí se ven entre sí
**Síntoma:** Routers hacen ping entre ellos, pero PCs no se alcanzan
**Causa:** Gateway mal configurado en las PCs
```
PC-0: gateway debe ser 192.168.30.1  (no 192.168.30.104 ni ningún otro)
PC-1: gateway debe ser 192.168.40.1  (no 192.168.40.104 ni ningún otro)
```

---

### Error 6: RIPv1 y las subredes de loopback
**⚠️ Importante:** RIPv1 es classful. La interfaz Loopback2 tiene IP 172.16.4.10 con máscara /16, pero RIPv1 la anuncia como la red de clase B completa: **172.16.0.0/16**. Esto es correcto y esperado. No confundir con un error.

---

### Comandos de debug (usar con cuidado en PT):
```
R1# debug ip rip

R1# undebug all
```

---

## 6. SIGUIENTES PASOS

- **OSPFv2** → ver archivo `Lab4_OSPFv2_Seccion4_Grupo10.md`
- **BGP** → ver archivo `Lab4_BGP_Seccion4_Grupo10.md` (pendiente)

---


---


---

## 🔍 TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa (antes y después de RIP)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/0 | `192.168.10.11` | ✅ Siempre |
| R0 | R2 G0/0 | `192.168.20.11` | ✅ Siempre |
| R1 | R0 G0/0 | `192.168.10.10` | ✅ Siempre |
| R2 | R0 G0/1 | `192.168.20.10` | ✅ Siempre |
| PC-0 | R1 gateway | `192.168.30.1` | ✅ Siempre |
| PC-1 | R2 gateway | `192.168.40.1` | ✅ Siempre |

### Conectividad remota (solo después de RIP convergido)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | PC-0 | `192.168.30.104` | ✅ Con RIP |
| R0 | PC-1 | `192.168.40.104` | ✅ Con RIP |
| R0 | Loopback R1 | `172.16.4.10` | ✅ Con RIP |
| R0 | Loopback R2 | `172.17.4.10` | ✅ Con RIP |
| R1 | PC-1 | `192.168.40.104` | ✅ Con RIP |
| R1 | Loopback R2 | `172.17.4.10` | ✅ Con RIP |
| R2 | PC-0 | `192.168.30.104` | ✅ Con RIP |
| R2 | Loopback R1 | `172.16.4.10` | ✅ Con RIP |
| **PC-0** | **PC-1** | **`192.168.40.104`** | **✅ OBJETIVO FINAL** |
| **PC-1** | **PC-0** | **`192.168.30.104`** | **✅ OBJETIVO FINAL** |

## RESUMEN RÁPIDO — CHECKLIST RIPv1

- [ ] Topología construida en Packet Tracer (3 routers, 2 switches, 2 PCs)
- [ ] Cables conectados correctamente (crossover R↔R, straight R↔SW↔PC)
- [ ] R0 configurado: hostname, IPs en G0/0 y G0/1, `no shutdown`, RIPv1
- [ ] R1 configurado: hostname, IPs en G0/0, G0/1 y Loopback2, `no shutdown`, RIPv1
- [ ] R2 configurado: hostname, IPs en G0/0, G0/1 y Loopback3, `no shutdown`, RIPv1
- [ ] PC-0: IP 192.168.30.104, máscara /24, gateway 192.168.30.1
- [ ] PC-1: IP 192.168.40.104, máscara /24, gateway 192.168.40.1
- [ ] `show ip route` en R0 muestra 6 redes (2 C + 4 R)
- [ ] `show ip route` en R1 muestra 6 redes (3 C + 3 R)
- [ ] `show ip route` en R2 muestra 6 redes (3 C + 3 R)
- [ ] PC-0 hace ping exitoso a PC-1 (192.168.40.104)
- [ ] PC-1 hace ping exitoso a PC-0 (192.168.30.104)
- [ ] C