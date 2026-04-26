# LABORATORIO 4 — BGP
## Sección 4 · Grupo 10 (X = 4, Y = 10)
### Topología 2

---

## 1. TOPOLOGÍA

```
         ASN 100                   ASN 200                   ASN 300
                                                          
  [PC-0]──[Switch2]──[R0]──────────[R1]──────────[R2]──[Switch1]──[PC-2]
            192.168.10.0/24   192.168.30.0/24  192.168.40.0/24  192.168.50.0/24
                                     |
                                 [Switch0]
                                     |
                                  [PC-1]
                               192.168.60.0/24
```

### Tabla de conexiones físicas

| # | Conexión | Tipo de cable | Red | ASN |
|---|----------|---------------|-----|-----|
| 1 | R0 **GE0/0** → Switch2 **Fa0/1** | Copper Straight-Through | 192.168.10.0/24 | 100 |
| 2 | Switch2 **Fa0/2** → PC-0 **Fa0** | Copper Straight-Through | 192.168.10.0/24 | 100 |
| 3 | R0 **GE0/1** → R1 **GE0/0** | Copper Cross-Over | 192.168.30.0/24 | 100 ↔ 200 |
| 4 | R1 **GE0/1** → R2 **GE0/0** | Copper Cross-Over | 192.168.40.0/24 | 200 ↔ 300 |
| 5 | R1 **GE0/2** → Switch0 **Fa0/1** | Copper Straight-Through | 192.168.60.0/24 | 200 |
| 6 | Switch0 **Fa0/2** → PC-1 **Fa0** | Copper Straight-Through | 192.168.60.0/24 | 200 |
| 7 | R2 **GE0/1** → Switch1 **Fa0/1** | Copper Straight-Through | 192.168.50.0/24 | 300 |
| 8 | Switch1 **Fa0/2** → PC-2 **Fa0** | Copper Straight-Through | 192.168.50.0/24 | 300 |

> Los puertos del switch (Fa0/1, Fa0/2) pueden ser cualquier FastEthernet libre; los números son de ejemplo.

**Equipos requeridos:**
- 3 × Router Cisco 2911 (R1 necesita 3 interfaces GigabitEthernet → ya las trae el 2911)
- 3 × Switch Cisco 2960-24TT
- 3 × PC

---

## 2. TABLA DE DIRECCIONAMIENTO FINAL (X=4, Y=10)

| Dispositivo | Interfaz | Dirección IP | Máscara | Gateway | ASN |
|-------------|----------|--------------|---------|---------|-----|
| R0 | G0/0 | 192.168.10.1 | 255.255.255.0 | N/A | 100 |
| R0 | G0/1 | 192.168.30.**5** | 255.255.255.0 | N/A | 100 |
| R1 | G0/0 | 192.168.30.**104** | 255.255.255.0 | N/A | 200 |
| R1 | G0/1 | 192.168.40.**104** | 255.255.255.0 | N/A | 200 |
| R1 | G0/2 | 192.168.60.1 | 255.255.255.0 | N/A | 200 |
| R2 | G0/0 | 192.168.40.**5** | 255.255.255.0 | N/A | 300 |
| R2 | G0/1 | 192.168.50.1 | 255.255.255.0 | N/A | 300 |
| PC-0 | NIC | 192.168.10.**14** | 255.255.255.0 | 192.168.10.1 | — |
| PC-1 | NIC | 192.168.60.**14** | 255.255.255.0 | 192.168.60.1 | — |
| PC-2 | NIC | 192.168.50.**14** | 255.255.255.0 | 192.168.50.1 | — |

### Cálculo de valores

| Fórmula | Sustitución | Resultado |
|---------|-------------|-----------|
| X+1 | 4+1 | **5** → R0 G0/1 y R2 G0/0 |
| X+Y | 4+10 | **14** → PCs |
| YX (concat) | "10"+"4" | **104** → R1 G0/0 y R1 G0/1 |

---

## 3. CONCEPTO BGP

BGP (Border Gateway Protocol) es el único protocolo EGP (Exterior Gateway Protocol) actualmente en uso. A diferencia de RIP y OSPF que funcionan **dentro** de un sistema autónomo (AS), BGP enruta **entre** sistemas autónomos distintos — es el protocolo que sostiene el Internet.

| Aspecto | RIP/OSPF (IGP) | BGP (EGP) |
|---------|---------------|-----------|
| Ámbito | Dentro de un AS | Entre AS distintos |
| Métrica | Saltos / Costo | Políticas (atributos) |
| Transporte | UDP/multicast | TCP puerto 179 |
| Vecinos | Auto-descubiertos | Configurados manualmente |
| Uso típico | Red interna | Internet / ISPs |

En esta topología hay **3 Sistemas Autónomos:**
- **ASN 100** → R0 + PC-0 (red de una organización)
- **ASN 200** → R1 + PC-1 (red del proveedor de tránsito)
- **ASN 300** → R2 + PC-2 (red de otra organización)

---

## 4. CONFIGURACIÓN COMPLETA BGP

### PASO 1 — Configuración de R0 (ASN 100)

```
enable
configure terminal

hostname R0

interface GigabitEthernet0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.30.5 255.255.255.0
 no shutdown

router bgp 100
 neighbor 192.168.30.104 remote-as 200
 network 192.168.10.0 mask 255.255.255.0

end
write memory
```

---

### PASO 2 — Configuración de R1 (ASN 200)

```
enable
configure terminal

hostname R1

interface GigabitEthernet0/0
 ip address 192.168.30.104 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.40.104 255.255.255.0
 no shutdown

interface GigabitEthernet0/2
 ip address 192.168.60.1 255.255.255.0
 no shutdown

router bgp 200
 neighbor 192.168.30.5 remote-as 100
 neighbor 192.168.40.5 remote-as 300
 network 192.168.60.0 mask 255.255.255.0

end
write memory
```

---

### PASO 3 — Configuración de R2 (ASN 300)

```
enable
configure terminal

hostname R2

interface GigabitEthernet0/0
 ip address 192.168.40.5 255.255.255.0
 no shutdown

interface GigabitEthernet0/1
 ip address 192.168.50.1 255.255.255.0
 no shutdown

router bgp 300
 neighbor 192.168.40.104 remote-as 200
 network 192.168.50.0 mask 255.255.255.0

end
write memory
```

---

### PASO 4 — Configuración de PCs

**PC-0:**
```
IP Address  : 192.168.10.104
Subnet Mask : 255.255.255.0
Gateway     : 192.168.10.1
```

**PC-1:**
```
IP Address  : 192.168.60.104
Subnet Mask : 255.255.255.0
Gateway     : 192.168.60.1
```

**PC-2:**
```
IP Address  : 192.168.50.104
Subnet Mask : 255.255.255.0
Gateway     : 192.168.50.1
```

---

## 5. VALIDACIÓN

### PASO 5a — Verificar sesión BGP (esperar ~30-60 segundos)

```
R0# show ip bgp summary
```
**Salida esperada en R0:**
```
BGP router identifier 192.168.30.5, local AS number 100
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.30.104   4   200      10      10        3    0    0 00:01:30        1
```
> El estado debe mostrar un **número** en la última columna (prefijos recibidos), no "Active" ni "Idle". Si dice "Active" es que la sesión TCP no se ha establecido.

```
R1# show ip bgp summary
```
**Salida esperada en R1:**
```
Neighbor        V    AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
192.168.30.5    4   100      10      10        3    0    0 00:01:30        1
192.168.40.5    4   300      10      10        3    0    0 00:01:30        1
```

---

### PASO 5b — Verificar tabla BGP

```
R0# show ip bgp
```
**Salida esperada en R0:**
```
Network          Next Hop         Metric LocPrf Weight Path
*> 192.168.10.0  0.0.0.0               0         32768 i
*> 192.168.50.0  192.168.30.104         0             0 200 300 i
*> 192.168.60.0  192.168.30.104         0             0 200 i
```
> El `*>` indica la mejor ruta. El `Path` muestra los AS que atraviesa (AS_PATH).

```
R1# show ip bgp
R2# show ip bgp
```

---

### PASO 5c — Verificar tabla de enrutamiento

```
R0# show ip route
```

---

### PASO 5d — Prueba final PC a PC

```
ping 192.168.60.104    ! → PC-1
ping 192.168.50.104    ! → PC-2

ping 192.168.10.104    ! → PC-0
ping 192.168.50.104    ! → PC-2

ping 192.168.10.104    ! → PC-0
ping 192.168.60.104    ! → PC-1
```

---

### PASO 5e — Comandos adicionales de diagnóstico

```
R0# show ip bgp neighbors

R0# show ip route bgp

R0# show ip bgp 192.168.50.0

R0# show running-config | section router bgp
```

---

## 6. ERRORES COMUNES BGP

### Error 1: Sesión en estado "Active" (no establece conexión TCP)
**Síntoma:** `show ip bgp summary` muestra "Active" en la columna State
**Causas:**
- IP del vecino mal escrita en `neighbor` 
- Las interfaces entre routers no tienen conectividad (verificar ping R0↔R1 antes de BGP)
- ASN incorrecto en `remote-as`

```
R0# ping 192.168.30.104    ! debe funcionar antes de configurar BGP

R0# show running-config | section router bgp
```

---

### Error 2: Sesión establecida pero no recibe prefijos (PfxRcd = 0)
**Síntoma:** La sesión está "Up" pero `PfxRcd = 0`
**Causa:** Falta el comando `network` o la red no existe en la tabla de rutas del router

```
R0# show ip route 192.168.10.0
```

---

### Error 3: Rutas BGP no aparecen en tabla de enrutamiento
**Síntoma:** `show ip bgp` muestra rutas pero `show ip route` no las tiene
**Causa:** La ruta no tiene el mejor path seleccionado (falta el `>` en `show ip bgp`)

```
R0# show ip bgp
```

---

### Error 4: R1 no propaga rutas entre R0 y R2 (problema iBGP/eBGP)
**Síntoma:** R0 y R2 no pueden comunicarse aunque ambos hablan con R1
**Causa:** BGP por defecto no redistribuye rutas eBGP hacia otros vecinos eBGP sin configuración adicional
**Solución en este lab:** R1 debe anunciar explícitamente las rutas que aprende, o configurar redistribución. En la configuración de este lab, R1 solo anuncia su propia red (192.168.60.0). Para que R0 vea la red de R2 y viceversa, R1 debe actuar como tránsito.

```
R1(config)# router bgp 200
R1(config-router)# network 192.168.10.0 mask 255.255.255.0
R1(config-router)# network 192.168.50.0 mask 255.255.255.0
```

---


---

## 🔍 TABLA DE PINGS — REFERENCIA RÁPIDA

### Conectividad directa (verificar ANTES de configurar BGP)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | R1 G0/0 | `192.168.30.104` | ✅ Siempre |
| R1 | R0 G0/1 | `192.168.30.5` | ✅ Siempre |
| R1 | R2 G0/0 | `192.168.40.5` | ✅ Siempre |
| R2 | R1 G0/1 | `192.168.40.104` | ✅ Siempre |
| PC-0 | R0 gateway | `192.168.10.1` | ✅ Siempre |
| PC-1 | R1 gateway | `192.168.60.1` | ✅ Siempre |
| PC-2 | R2 gateway | `192.168.50.1` | ✅ Siempre |

### Conectividad remota (solo después de sesión BGP establecida)
| Desde | Hacia | IP destino | ¿Debe funcionar? |
|-------|-------|-----------|-----------------|
| R0 | Red PC-1 | `192.168.60.1` | ✅ Con BGP |
| R0 | Red PC-2 | `192.168.50.1` | ✅ Con BGP |
| R2 | Red PC-0 | `192.168.10.1` | ✅ Con BGP |
| R2 | Red PC-1 | `192.168.60.1` | ✅ Con BGP |
| **PC-0** | **PC-1** | **`192.168.60.104`** | **✅ OBJETIVO FINAL** |
| **PC-0** | **PC-2** | **`192.168.50.104`** | **✅ OBJETIVO FINAL** |
| **PC-1** | **PC-2** | **`192.168.50.104`** | **✅ OBJETIVO FINAL** |

## 7. CHECKLIST BGP

- [ ] Topología construida: 3 routers, 3 switches, 3 PCs
- [ ] R0: interfaces UP, `router bgp 100`, neighbor R1 (192.168.30.104), network 192.168.10.0
- [ ] R1: interfaces UP, `router bgp 200`, neighbors R0 y R2, network 192.168.60.0
- [ ] R2: interfaces UP, `router bgp 300`, neighbor R1, network 192.168.50.0
- [ ] PC-0, PC-1, PC-2 con IPs y gateways correctos
- [ ] Ping R0 ↔ R1 funciona **antes** de configurar BGP
- [ ] Ping R1 ↔ R2 funciona **antes** de configurar BGP
- [ ] `show ip bgp summary` en R1 muestra 2 vecinos con estado numérico (no "Active")
- [ ] `show ip bgp` muestra rutas con `*>` 
- [ ] `show ip route` muestra rutas con `B`
- [ ] PC-0 hace ping exitoso a PC-1 y PC-2
- [ ] Captura Wireshark: paquetes ICMP exitosos + paquetes BGP (TCP puerto 179)

---

## BORRAR TODA LA CONFIGURACIÓN

Para empezar de cero en cualquier router (borra todo y reinicia):

```
enable
write erase
reload
```

> Cuando pregunte `Proceed with reload? [confirm]` presiona **Enter**. El router arranca sin ninguna configuración.

Para borrar solo el proceso BGP sin resetear el router completo (reemplaza el número de ASN según el router):

```
enable
configure terminal
no router bgp 100
end
write memory
```
