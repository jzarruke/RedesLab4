# LABORATORIO 4 — RIPng
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Topología 3 — IPv6

---

## 1. TOPOLOGÍA

```
                        [R0]
                    G0/0 /  \ G0/1
                        /    \
          2001:ABCD:1435:A1::/64   2002:ABCD:1435:B1::/64
                      /                  \
                  [R1]                    [R2]
              G0/0  G0/1             G0/0   G0/1
                       |                      |
                   [Switch0]             [Switch1]
                       |                      |
                    [PC-0]                 [PC-1]
          2002:ABCD:1435:C1::/64   2002:ABCD:1435:D1::/64
```

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R0 **GE0/0** → R1 **GE0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R0 **GE0/1** → R2 **GE0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R1 **GE0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | Switch0 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R2 **GE0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | Switch1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

---

## 2. TABLA DE DIRECCIONAMIENTO FINAL (X=4, Y=10)

| Dispositivo | Interfaz | Dirección IPv6 / Prefijo | Gateway |
|-------------|----------|--------------------------|---------|
| R0 | G0/0 | **2001:ABCD:1435:A1::10/64** | N/A |
| R0 | G0/1 | **2002:ABCD:1435:B1::10/64** | N/A |
| R1 | G0/0 | **2001:ABCD:1435:A1::11/64** | N/A |
| R1 | G0/1 | **2002:ABCD:1435:C1::1/64** | N/A |
| R2 | G0/0 | **2002:ABCD:1435:B1::11/64** | N/A |
| R2 | G0/1 | **2002:ABCD:1435:D1::1/64** | N/A |
| PC-0 | NIC | **2002:ABCD:1435:C1::104/64** | 2002:ABCD:1435:C1::1 |
| PC-1 | NIC | **2002:ABCD:1435:D1::104/64** | 2002:ABCD:1435:D1::1 |

### Cálculo de valores

| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| ::Y | Y = 10 | **::10** → R0 en ambas interfaces |
| ::Y+1 | Y+1 = 11 | **::11** → R1 G0/0 y R2 G0/0 |
| ::YX (concat) | "10"+"4" = "104" | **::104** → PCs |

---

## 3. DIFERENCIAS CLAVE: RIPv1 vs RIPng

| Aspecto | RIPv1 (IPv4) | RIPng (IPv6) |
|---------|-------------|--------------|
| Protocolo IP | IPv4 | IPv6 |
| Configuración | Global (`router rip`) | Por interfaz (`ipv6 rip enable`) |
| Comando global | `network [red]` | `ipv6 unicast-routing` (obligatorio) |
| Multicast updates | 255.255.255.255 | FF02::9 |
| Puerto | UDP 520 | UDP 521 |
| Classful/Classless | Classful | Classless (usa prefijos IPv6) |
| Máscara en rutas | No (classful) | Sí (longitud de prefijo) |

> ⚠️ **Obligatorio antes de todo:** `ipv6 unicast-routing` en cada router. Sin esto, el router no enruta paquetes IPv6.

---

## 4. CONFIGURACIÓN COMPLETA RIPng

### PASO 1 — Configuración de R0

```
enable
configure terminal

hostname R0

ipv6 unicast-routing

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::10/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::10/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

---

### PASO 2 — Configuración de R1

```
enable
configure terminal

hostname R1

ipv6 unicast-routing

interface GigabitEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::11/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

---

### PASO 3 — Configuración de R2

```
enable
configure terminal

hostname R2

ipv6 unicast-routing

interface GigabitEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::11/64
 ipv6 rip RIP1 enable
 no shutdown

interface GigabitEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

---

### PASO 4 — Configuración de PCs (IPv6)

> En Packet Tracer: click en el PC → **Desktop** → **IP Configuration** → sección **IPv6 Configuration** → seleccionar **Static**

**PC-0:**
```
IPv6 Address        : 2002:ABCD:1435:C1::104
Prefix Length       : 64
IPv6 Default Gateway: 2002:ABCD:1435:C1::1
```

**PC-1:**
```
IPv6 Address        : 2002:ABCD:1435:D1::104
Prefix Length       : 64
IPv6 Default Gateway: 2002:ABCD:1435:D1::1
```

> ⚠️ Asegúrate de configurar el gateway en el campo **IPv6 Default Gateway**, no en el campo IPv4.

---

## 5. VALIDACIÓN

### PASO 5a — Verificar interfaces IPv6

```
R0# show ipv6 interface brief
```
**Salida esperada en R0:**
```
GigabitEthernet0/0  [up/up]
    FE80::...                         ← link-local (auto)
    2001:ABCD:1435:A1::10             ← dirección configurada
GigabitEthernet0/1  [up/up]
    FE80::...
    2002:ABCD:1435:B1::10
```

---

### PASO 5b — Verificar tabla de enrutamiento IPv6 (esperar ~30 seg)

```
R0# show ipv6 route
```
**Salida esperada en R0:**
```
IPv6 Routing Table

C   2001:ABCD:1435:A1::/64 [0/0]
     via GigabitEthernet0/0, directly connected
C   2002:ABCD:1435:B1::/64 [0/0]
     via GigabitEthernet0/1, directly connected
R   2002:ABCD:1435:C1::/64 [120/2]
     via FE80::..., GigabitEthernet0/0
R   2002:ABCD:1435:D1::/64 [120/2]
     via FE80::..., GigabitEthernet0/1
```
> Las rutas RIPng aparecen con **R** igual que RIPv1, pero ahora son prefijos IPv6.

```
R1# show ipv6 route
R2# show ipv6 route
```

---

### PASO 5c — Verificar proceso RIPng

```
R0# show ipv6 rip
```
**Salida esperada:**
```
RIP process "RIP1", port 521, multicast-group FF02::9, pid 263
      Administrative distance is 120. Maximum paths is 16
      Updates every 30 seconds, expire after 180
      ...
```

```
R0# show ipv6 rip database
```

---

### PASO 5d — Prueba de conectividad

```
R0# ping 2001:ABCD:1435:A1::11    ! → R1 G0/0
R0# ping 2002:ABCD:1435:B1::11    ! → R2 G0/0

R0# ping 2002:ABCD:1435:C1::1     ! → R1 G0/1
R0# ping 2002:ABCD:1435:D1::1     ! → R2 G0/1

ping 2002:ABCD:1435:D1::104       ! → PC-1
ping 2002:ABCD:1435:C1::1         ! → gateway R1

ping 2002:ABCD:1435:C1::104       ! → PC-0
```

---

## 6. ERRORES COMUNES RIPng

### Error 1: `% IPv6 routing not enabled`
**Síntoma:** Al escribir `ipv6 rip RIP1 enable` en la interfaz aparece este error
**Causa:** No se ejecutó `ipv6 unicast-routing` antes
```
R0(config)# ipv6 unicast-routing
```

---

### Error 2: Interfaces sin dirección IPv6 o en estado down
**Síntoma:** `show ipv6 interface brief` muestra `[down/down]` o sin dirección
```
R0# show ipv6 interface brief
R0(config)# interface GigabitEthernet0/0
R0(config-if)# ipv6 address 2001:ABCD:1435:A1::10/64
R0(config-if)# no shutdown
```

---

### Error 3: Rutas RIPng no aparecen en tabla
**Síntoma:** `show ipv6 route` solo muestra rutas `C` (connected)
**Causa:** Se olvidó `ipv6 rip RIP1 enable` en alguna interfaz, o el nombre del proceso es diferente en cada router
```

R0# show ipv6 rip
R1# show ipv6 rip
R2# show ipv6 rip
```

---

### Error 4: PC no hace ping a su gateway IPv6
**Síntoma:** Ping desde PC-0 a 2002:ABCD:1435:C1::1 falla
**Causas posibles:**
1. Gateway configurado en campo IPv4 en vez de IPv6
2. Prefijo incorrecto (no /64)
3. Dirección mal escrita (revisar los bloques ABCD:1435:C1)

```
R1# show ipv6 interface GigabitEthernet0/1
```

---

### Error 5: Confundir direcciones de red entre topologías
> ⚠️ Las redes de Topología 3 son **completamente distintas** a las de Topología 1 y 2. Son prefijos IPv6, no IPv4. Asegúrate de estar trabajando en el archivo .pkt correcto.

---


---

## 🔍 TABLA DE PINGS — REFERENCIA RÁPIDA (IPv6)

### Conectividad directa (siempre debe funcionar)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/0 | `2001:ABCD:1435:A1::11` | ✅ Siempre |
| R0 | R2 G0/0 | `2002:ABCD:1435:B1::11` | ✅ Siempre |
| R1 | R0 G0/0 | `2001:ABCD:1435:A1::10` | ✅ Siempre |
| R2 | R0 G0/1 | `2002:ABCD:1435:B1::10` | ✅ Siempre |
| PC-0 | R1 gateway | `2002:ABCD:1435:C1::1` | ✅ Siempre |
| PC-1 | R2 gateway | `2002:ABCD:1435:D1::1` | ✅ Siempre |

### Conectividad remota (solo después de RIPng convergido)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/1 | `2002:ABCD:1435:C1::1` | ✅ Con RIPng |
| R0 | R2 G0/1 | `2002:ABCD:1435:D1::1` | ✅ Con RIPng |
| R1 | R2 G0/0 | `2002:ABCD:1435:B1::11` | ✅ Con RIPng |
| R1 | R2 G0/1 | `2002:ABCD:1435:D1::1` | ✅ Con RIPng |
| R2 | R1 G0/1 | `2002:ABCD:1435:C1::1` | ✅ Con RIPng |
| **PC-0** | **PC-1** | **`2002:ABCD:1435:D1::104`** | **✅ OBJETIVO FINAL** |
| **PC-1** | **PC-0** | **`2002:ABCD:1435:C1::104`** | **✅ OBJETIVO FINAL** |

## 7. CHECKLIST RIPng

- [ ] Topología construida en Packet Tracer (misma estructura que Topo 1 pero nueva)
- [ ] `ipv6 unicast-routing` activado en R0, R1 y R2
- [ ] R0: IPv6 en G0/0 y G0/1, `ipv6 rip RIP1 enable` en ambas, proceso `ipv6 router rip RIP1`
- [ ] R1: IPv6 en G0/0 y G0/1, `ipv6 rip RIP1 enable` en ambas, proceso `ipv6 router rip RIP1`
- [ ] R2: IPv6 en G0/0 y G0/1, `ipv6 rip RIP1 enable` en ambas, proceso `ipv6 router rip RIP1`
- [ ] PC-0: IPv6 2002:ABCD:1435:C1::104/64, gateway 2002:ABCD:1435:C1::1
- [ ] PC-1: IPv6 2002:ABCD:1435:D1::104/64, gateway 2002:ABCD:1435:D1::1
- [ ] `show ipv6 interface brief` en todos los routers muestra `[up/up]`
- [ ] `show ipv6 route` en R0 muestra 2 rutas `C` + 2 rutas `R`
- [ ] PC-0 hace ping exitoso a PC-1
- [ ] Captura Wireshark: paquetes ICMPv6 exitosos + paquetes RIPng (UDP 521) visibles

---

## CONFIGURACIÓN ALTERNATIVA — FastEthernet

> Usa esta sección si los routers del laboratorio físico tienen interfaces **FastEthernet** (Fa0/0, Fa0/1). Modelos comunes: **Cisco 1841**, **2811**, **2621XM**.
> Las direcciones IPv6 y la lógica RIPng son exactamente las mismas — solo cambia el nombre de la interfaz.

### Conexiones con FastEthernet

| # | Conexión | Tipo de cable | Red IPv6 |
|---|----------|---------------|----------|
| 1 | R0 **Fa0/0** → R1 **Fa0/0** | Copper Cross-Over | 2001:ABCD:1435:A1::/64 |
| 2 | R0 **Fa0/1** → R2 **Fa0/0** | Copper Cross-Over | 2002:ABCD:1435:B1::/64 |
| 3 | R1 **Fa0/1** → Switch0 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 4 | Switch0 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:C1::/64 |
| 5 | R2 **Fa0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |
| 6 | Switch1 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 2002:ABCD:1435:D1::/64 |

### R0 — FastEthernet

```
enable
configure terminal

hostname R0

ipv6 unicast-routing

interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::10/64
 ipv6 rip RIP1 enable
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:B1::10/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

### R1 — FastEthernet

```
enable
configure terminal

hostname R1

ipv6 unicast-routing

interface FastEthernet0/0
 ipv6 address 2001:ABCD:1435:A1::11/64
 ipv6 rip RIP1 enable
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:C1::1/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

### R2 — FastEthernet

```
enable
configure terminal

hostname R2

ipv6 unicast-routing

interface FastEthernet0/0
 ipv6 address 2002:ABCD:1435:B1::11/64
 ipv6 rip RIP1 enable
 no shutdown

interface FastEthernet0/1
 ipv6 address 2002:ABCD:1435:D1::1/64
 ipv6 rip RIP1 enable
 no shutdown

ipv6 router rip RIP1

end
write memory
```

> Los comandos de verificación (`show ipv6 route`, `show ipv6 rip`, etc.) funcionan igual. La salida mostrará `FastEthernet0/0` en lugar de `GigabitEthernet0/0`.

---

## BORRAR TODA LA CONFIGURACIÓN

Para empezar de cero en cualquier router (borra todo y reinicia):

```
enable
write erase
reload
```

> Cuando pregunte `Proceed with reload? [confirm]` presiona **Enter**. El router arranca sin ninguna configuración.

Para borrar solo RIPng y las direcciones IPv6 sin resetear el router completo:

```
enable
configure terminal
no ipv6 router rip RIP1
interface GigabitEthernet0/0
 no ipv6 address
 no ipv6 rip RIP1 enable
interface GigabitEthernet0/1
 no ipv6 address
 no ipv6 rip RIP1 enable
no ipv6 unicast-routing
end
write memory
```
