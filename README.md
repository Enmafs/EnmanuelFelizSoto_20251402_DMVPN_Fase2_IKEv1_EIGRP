# 🔐 Lab 07 — DMVPN Fase 2 + IKEv1 + EIGRP
**Estudiante:** Enmanuel Feliz Soto | **Matrícula:** 2025-1402  
**Institución:** Instituto Tecnológico de Las Américas (ITLA)  
**Curso:** Seguridad en Redes | **Sección:** 2-1C  
**Docente:** Jonathan Esteban Rondón Corniel

---

## 📋 Descripción

DMVPN Fase 2 permite túneles Spoke-to-Spoke dinámicos sin pasar por el Hub. NHRP resuelve el mapping NBMA entre spokes. EIGRP distribuye las rutas de las LANs dentro del overlay (Tunnel0 mGRE).

| Campo | Valor |
|-------|-------|
| **Tipo de VPN** | DMVPN Hub-and-Spoke |
| **Protocolo** | IKEv1 + IPSec ESP-AES256-SHA256 (mode transport) + mGRE |
| **Mecanismo** | NHRP + mGRE + IPSec transport mode — Túneles Spoke-to-Spoke directos |
| **Routing** | EIGRP AS 100 sobre Tunnel0 mGRE |
| **Pre-shared Key** | `Cisco123` |

---

## 🗺️ Topología

> 📸 **[INSERTAR CAPTURA DE TOPOLOGÍA AQUÍ]**
<!-- Coloca aquí el screenshot de PNetLab con la topología del Lab 07 -->

**Entorno:** PNetLab — Cisco IOL  
**Peers:** HUB ISP (14.2.10.1) | SPOKE1 R1-S1 (14.2.10.2) | SPOKE2 R3 (14.2.10.3)

### Tabla de Direccionamiento

| Rol | Router | Interfaz WAN | IP WAN | IP Tunnel | LAN |
|-----|--------|-------------|--------|-----------|-----|
| HUB | ISP | e0/0 | 20.25.1.1/30 | 14.2.10.1/24 | — |
| SPOKE1 | R1-S1 | e0/0 | 20.25.1.2/30 | 14.2.10.2/24 | — |
| SPOKE2 | R3 | e0/0 | 20.25.2.6/30 | 14.2.10.3/24 | 30.30.30.0/24 |

### ISP (Hub)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.1.1/30 | Link to R1-S1 (Spoke1) y transit hacia R3 |
| Tunnel0 | 14.2.10.1/24 | mGRE Hub DMVPN |

### R1-S1 (Spoke1 + Transit para R3)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.1.2/30 | Link to ISP Hub |
| Ethernet0/2 | 20.25.2.5/30 | Link to R3 (transit) |
| Tunnel0 | 14.2.10.2/24 | GRE Spoke1 DMVPN |

### R3 (Spoke2)

| Interfaz | IP | Descripción |
|---------|-----|-------------|
| Ethernet0/0 | 20.25.2.6/30 | Link to R1-S1 (transit hacia ISP) |
| Ethernet0/1 | 30.30.30.1/24 | LAN local |
| Tunnel0 | 14.2.10.3/24 | GRE Spoke2 DMVPN |

### Dirección Túnel

| Endpoint | IP Tunnel | NBMA (física) |
|----------|-----------|---------------|
| ISP (Hub) | 14.2.10.1 | 20.25.1.1 |
| R1-S1 (Spoke1) | 14.2.10.2 | 20.25.1.2 |
| R3 (Spoke2) | 14.2.10.3 | 20.25.2.6 |

> **Nota:** R3 llega al Hub ISP pasando por R1-S1 como router de tránsito. El tunnel source de R3 es `e0/0` (20.25.2.6); NHRP registra esa IP como NBMA ante el Hub.

---

## ⚙️ Configuración

El script completo de configuración se encuentra en:  
📄 [`EnmanuelFelizSoto_2025-1402_Lab07_P3.txt`](./EnmanuelFelizSoto_2025-1402_Lab07_P3.txt)

### Parámetros IKE/IPSec

| Parámetro | Valor |
|-----------|-------|
| Encryption | AES-256 |
| Hash/Integrity | SHA-256 |
| DH Group | 14 (2048-bit) |
| SA Lifetime (IKE) | 86400 s (24h) |
| Auth Method | Pre-Shared Key |
| IPSec Mode | Transport |
| Transform-set | esp-aes 256 esp-sha256-hmac |

### Parámetros NHRP

| Parámetro | Valor |
|-----------|-------|
| Network-ID | 100 |
| Authentication | NHRP2024 |
| Holdtime | 300 s |
| NHS (Hub) | 14.2.10.1 |
| Tunnel mode | gre multipoint |

---

## ▶️ Procedimiento de Ejecución

### 1. Cargar configuración en PNetLab

```
# Aplicar configuración en cada dispositivo en este orden:
# 1. ISP (Hub) → 2. R1-S1 (Spoke1) → 3. R3 (Spoke2)
```

### 2. Verificar la VPN

```
show dmvpn
```
```
show dmvpn detail
```
```
show ip nhrp
```
```
show ip eigrp neighbors
```
```
show crypto isakmp sa
```
```
show crypto ipsec sa
```

### 3. Prueba de conectividad

```
ping 14.2.10.1 source Tunnel0
```
```
ping 14.2.10.3 source Tunnel0
```
```
ping 30.30.30.1
```

---

## 📸 Capturas de Verificación

> 📸 **[INSERTAR CAPTURA: show dmvpn]**
<!-- Captura mostrando spokes registrados con estado UP -->

> 📸 **[INSERTAR CAPTURA: show crypto isakmp sa]**
<!-- Captura mostrando estado QM_IDLE -->

> 📸 **[INSERTAR CAPTURA: show crypto ipsec sa]**
<!-- Captura mostrando pkts encaps/decaps incrementando -->

> 📸 **[INSERTAR CAPTURA: ping exitoso]**
<!-- Ping desde R1-S1 hacia 30.30.30.1 (LAN de R3) -->

---

## 🔍 Análisis y Comparativa

### Ventajas de este tipo de VPN
- Ver documentación técnica en el informe PDF

### Diferencias con otros labs
- Ver tabla comparativa en el README principal

---

## 📎 Recursos

| Recurso | Enlace |
|---------|--------|
| Repositorio Principal | [Enmafs/NetSec](https://github.com/Enmafs/NetSec) |
| Script de configuración | [`EnmanuelFelizSoto_2025-1402_DMVPN_Fase2_IKEv1_EIGRP_P3.txt`](./EnmanuelFelizSoto_2025-1402_DMVPN_Fase2_IKEv1_EIGRP_P3.txt) |
| Video demostración | 🎬 [Aquí](https://youtu.be/iVLqKyAXDYM) |

---

> ⚠️ *Laboratorio realizado en entorno controlado (PNetLab). Fines exclusivamente académicos.*
