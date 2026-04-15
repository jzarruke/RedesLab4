# LABORATORIO 4 — OSPFv2
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Topología 1 — misma que RIPv1

---

## 1. TOPOLOGÍA Y DIRECCIONAMIENTO

> OSPFv2 usa exactamente la **misma topología e IPs** que RIPv1. Solo cambia el protocolo de enrutamiento.

```
                        [R0]
                    G0/0 /  \ G0/1
                        /    \
               192.168.10.0   192.168.20.0
                      /            \
                  [R1]              [R2]
              G0/0  G0/1        G0/0  G0/1
                       |                |
                   [Switch0]       [Switch1]
                       |                |
                    [PC-0]           [PC-1]
               192.168.30.104   192.168.40.104
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

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway |
|-------------|----------|--------------|---------|---------|
| R0 | G0/0 | 192.168.10.10 | 255.255.255.0 | N/A |
| R0 | G0/1 | 192.168.20.10 | 255.255.255.0 | N/A |
| R1 | G0/0 | 192.168.10.11 | 255.255.255.0 | N/A |
| R1 | G0/1 | 192.168.30.1 | 255.255.255.0 | N/A |
| R1 | Loopback2 | 172.16.4.10 | 255.255.0.0 | N/A |
| R2 | G0/0 | 192.168.20.11 | 255.255.255.0 | N/A |
| R2 | G0/1 | 192.168.40.1 | 255.255.255.0 | N/A |
| R2 | Loopback3 | 172.17.4.10 | 255.255.0.0 | N/A |
| PC-0 | NIC | 192.168.30.104 | 255.255.255.0 | 192.168.30.1 |
| PC-1 | NIC | 192.168.40.104 | 255.255.255.0 | 192.168.40.1 |

---

## 2. DIFERENCIAS CLAVE: RIPv1 vs OSPFv2

| Aspecto | RIPv1 | OSPFv2 |
|---------|-------|--------|
| Tipo | Vector distancia | Link-state |
| Métrica | Saltos (máx 15) | Costo (ancho de banda) |
| Actualizaciones | Tabla completa cada 30 seg | Solo ante cambios de topología |
| Convergencia | Lenta (~minutos) | Rápida (~segundos) |
| Classful/Classless | Classful (sin máscara) | Classless (con wildcard mask) |
| Sintaxis `network` | `network [red-clase]` | `network [ip] [wildcard] area [n]` |
| Rutas en tabla | Marcadas con **R** | Marcadas con **O** |
| Distancia administrativa | 120 | 110 |

**Wildcard mask** = inverso de la máscara de subred:
- Máscara /24 → `255.255.255.0` → wildcard **`0.0.0.255`**
- Máscara /16 → `255.255.0.0`   → wildcard **`0.0.255.255`**

---

## 3. CONFIGURACIÓN COMPLETA OSPFv2

### PASO 1 — Eliminar RIPv1 en los 3 routers

```
R0# configure terminal
R0(config)# no router rip
R0(config)# end

R1# configure terminal
R1(config)# no router rip
R1(config)# end

R2# configure terminal
R2(config)# no router rip
R2(config)# end
```

Confirmar que RIP desapareció:
```
R0# show ip route
```

---

### PASO 2 — Configuración R0

```
enable
configure terminal

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0

end
write memory
```

> R0 no tiene loopback → su Router-ID será **192.168.20.10** (la IP activa más alta), asignado automáticamente.

---

### PASO 3 — Configuración R1

```
enable
configure terminal

router ospf 1
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 172.16.0.0 0.0.255.255 area 0

end
write memory
```

> R1 tiene Loopback2 (172.16.4.10) → su Router-ID será **172.16.4.10** automáticamente (OSPF prefiere la IP de loopback más alta).

---

### PASO 4 — Configuración R2

```
enable
configure terminal

router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
 network 172.17.0.0 0.0.255.255 area 0

end
write memory
```

> R2 tiene Loopback3 (172.17.4.10) → su Router-ID será **172.17.4.10** automáticamente.

---

## 4. VALIDACIÓN

### PASO 5a — Verificar vecinos OSPF (esperar ~10 segundos)

```
R0# show ip ospf neighbor
```
**Salida esperada en R0:**
```
Neighbor ID     Pri   State     Dead Time   Address         Interface
172.16.4.10       1   FULL/DR   00:00:35    192.168.10.11   GigabitEthernet0/0
172.17.4.10       1   FULL/DR   00:00:35    192.168.20.11   GigabitEthernet0/1
```

```
R1# show ip ospf neighbor
```
**Salida esperada en R1:**
```
Neighbor ID     Pri   State      Dead Time   Address         Interface
192.168.20.10     1   FULL/BDR   00:00:35    192.168.10.10   GigabitEthernet0/0
```

```
R2# show ip ospf neighbor
```
**Salida esperada en R2:**
```
Neighbor ID     Pri   State      Dead Time   Address         Interface
192.168.20.10     1   FULL/BDR   00:00:35    192.168.20.10   GigabitEthernet0/0
```

> ⚠️ El estado **FULL** es obligatorio. Si ves INIT, 2WAY o EXSTART hay un problema (ver sección de errores).

---

### PASO 5b — Verificar tabla de enrutamiento

```
R0# show ip route
```
**Salida esperada en R0:**
```
Codes: C - connected, O - OSPF ...

O    172.16.0.0/16 [110/2] via 192.168.10.11, GigabitEthernet0/0
O    172.17.0.0/16 [110/2] via 192.168.20.11, GigabitEthernet0/1
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/1
O    192.168.30.0/24 [110/2] via 192.168.10.11, GigabitEthernet0/0
O    192.168.40.0/24 [110/2] via 192.168.20.11, GigabitEthernet0/1
```

```
R1# show ip route
```
**Salida esperada en R1:**
```
O    172.17.0.0/16 [110/3] via 192.168.10.10, GigabitEthernet0/0
C    172.16.0.0/16 is directly connected, Loopback2
C    192.168.10.0/24 is directly connected, GigabitEthernet0/0
O    192.168.20.0/24 [110/2] via 192.168.10.10, GigabitEthernet0/0
C    192.168.30.0/24 is directly connected, GigabitEthernet0/1
O    192.168.40.0/24 [110/3] via 192.168.10.10, GigabitEthernet0/0
```

```
R2# show ip route
```
**Salida esperada en R2:**
```
O    172.16.0.0/16 [110/3] via 192.168.20.10, GigabitEthernet0/0
C    172.17.0.0/16 is directly connected, Loopback3
O    192.168.10.0/24 [110/2] via 192.168.20.10, GigabitEthernet0/0
C    192.168.20.0/24 is directly connected, GigabitEthernet0/0
O    192.168.30.0/24 [110/3] via 192.168.20.10, GigabitEthernet0/0
C    192.168.40.0/24 is directly connected, GigabitEthernet0/1
```

---

### PASO 5c — Prueba final PC a PC

```
ping 192.168.40.104    ! → PC-1
ping 172.16.4.10       ! → Loopback R1
ping 172.17.4.10       ! → Loopback R2

ping 192.168.30.104    ! → PC-0
```

---

### PASO 5d — Comandos adicionales de diagnóstico

```
R0# show ip ospf interface

R0# show ip ospf database

R0# show ip route ospf

R0# show running-config | section router ospf

R0# show ip ospf
```

---

## 5. ERRORES COMUNES OSPFv2

### Error 1: Vecinos no aparecen / estado INIT
**Síntoma:** `show ip ospf neighbor` no muestra vecinos o muestra INIT
**Causas:**
- Interfaces apagadas
- IPs no están en la misma subred
- El comando `network` no cubre la interfaz correcta

```
R0# show ip interface brief
```

---

### Error 2: Vecino en estado EXSTART (nunca llega a FULL)
**Síntoma:** `show ip ospf neighbor` muestra EXSTART eternamente
**Causa más común en Packet Tracer:** Router-ID duplicado entre routers

```
R0# show ip ospf | include Router ID
R1# show ip ospf | include Router ID
R2# show ip ospf | include Router ID
```

Si hay duplicados, asignar manualmente:
```
R0(config)# router ospf 1
R0(config-router)# router-id 0.0.0.1
R0# clear ip ospf process   → responder "yes"

R1(config)# router ospf 1
R1(config-router)# router-id 0.0.0.2
R1# clear ip ospf process   → responder "yes"

R2(config)# router ospf 1
R2(config-router)# router-id 0.0.0.3
R2# clear ip ospf process   → responder "yes"
```

---

### Error 3: Una red no aparece con "O" en la tabla
**Síntoma:** Por ejemplo, 172.16.0.0 no aparece en R0 o R2
**Causa:** El comando `network` en OSPF no cubre esa interfaz

```
R1(config)# router ospf 1
R1(config-router)# network 172.16.0.0 0.0.255.255 area 0
```

---

### Error 4: RIP y OSPF corriendo al mismo tiempo
**Síntoma:** La tabla mezcla rutas `R` y `O`
**Causa:** No se eliminó RIP antes de configurar OSPF

```
R0# show running-config | section router rip
R0(config)# no router rip
```

---


---

## 🔍 TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa (siempre debe funcionar)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/0 | `192.168.10.11` | ✅ Siempre |
| R0 | R2 G0/0 | `192.168.20.11` | ✅ Siempre |
| R1 | R0 G0/0 | `192.168.10.10` | ✅ Siempre |
| R2 | R0 G0/1 | `192.168.20.10` | ✅ Siempre |
| PC-0 | R1 gateway | `192.168.30.1` | ✅ Siempre |
| PC-1 | R2 gateway | `192.168.40.1` | ✅ Siempre |

### Conectividad remota (solo después de OSPF en estado FULL)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | PC-0 | `192.168.30.104` | ✅ Con OSPF |
| R0 | PC-1 | `192.168.40.104` | ✅ Con OSPF |
| R0 | Loopback R1 | `172.16.4.10` | ✅ Con OSPF |
| R0 | Loopback R2 | `172.17.4.10` | ✅ Con OSPF |
| R1 | PC-1 | `192.168.40.104` | ✅ Con OSPF |
| R1 | Loopback R2 | `172.17.4.10` | ✅ Con OSPF |
| R2 | PC-0 | `192.168.30.104` | ✅ Con OSPF |
| R2 | Loopback R1 | `172.16.4.10` | ✅ Con OSPF |
| **PC-0** | **PC-1** | **`192.168.40.104`** | **✅ OBJETIVO FINAL** |
| **PC-1** | **PC-0** | **`192.168.30.104`** | **✅ OBJETIVO FINAL** |

## 6. CHECKLIST OSPFv2

- [ ] RIPv1 eliminado en R0, R1 y R2 (`no router rip`)
- [ ] `show ip route` muestra solo rutas `C` antes de configurar OSPF
- [ ] R0: `router ospf 1` con `network` para 192.168.10.0 y 192.168.20.0
- [ ] R1: `router ospf 1` con `network` para 192.168.10.0, 192.168.30.0 y 172.16.0.0
- [ ] R2: `router ospf 1` con `network` para 192.168.20.0, 192.168.40.0 y 172.17.0.0
- [ ] `show ip ospf neighbor` en R0 → 2 vecinos en estado **FULL**
- [ ] `show ip ospf neighbor` en R1 → 1 vecino en estado **FULL**
- [ ] `show ip ospf neighbor` en R2 → 1 vecino en estado **FULL**
- [ ] `show ip route` en R0 muestra 4 rutas con `O`
- [ ] `show ip route` en R1 muestra 3 rutas con `O`
- [ ] `show ip route` en R2 muestra 3 rutas con `O`
- [ ] PC-0 hace ping exitoso a PC-1 (192.168.40.104)
- [ ] PC-1 hace ping exitoso a PC-0 (192.168.30.104)
- [ ] Captura Wireshark: paquetes ICMP exitosos + paquetes OSPF Hello visibles
