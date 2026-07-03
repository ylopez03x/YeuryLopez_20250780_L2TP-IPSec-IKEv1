# VPN Client-to-Site — L2TP/IPSec IKEv1
**Autor:** Yeury Lopez | **Matrícula:** 20250780 | **Repositorio:** YeuryLopez_20250780_L2TP-IPSec-IKEv1

---

## 1. Objetivo
Configurar una VPN Client-to-Site L2TP sobre IPSec IKEv1. L2TP provee el túnel de capa 2 e IPSec provee el cifrado. El cliente Ubuntu Linux se conecta al router R1 que actúa como servidor L2TP, obteniendo una IP del pool VPN mediante PPP.

---

## 2. Topología

```
[Linux-Client] --- [ISP] --- [R1] --- [SW] --- [Linux-Server]
```

> 📸 **SCREENSHOT:** Insertar captura de la topología completa en EVE-NG

---

## 3. Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP | Rol |
|---|---|---|---|
| ISP | Fa0/0 | 10.7.85.1/30 | WAN → Client |
| ISP | Fa1/0 | 10.7.80.2/30 | WAN → R1 |
| R1 | Fa0/0 | 10.7.80.1/30 | WAN → ISP |
| R1 | Fa1/0 | 192.168.7.1/24 | LAN |
| Linux-Client | ens3 | 10.7.85.2/30 | WAN (cliente remoto) |
| Linux-Client | ppp0 | 172.16.78.1/32 | IP asignada por VPN Pool |
| Linux-Server | ens3 | 192.168.7.2/24 | Host LAN |

---

## 4. Parámetros VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | L2TP over IPSec IKEv1 |
| Protocolo de Tunelización | L2TP (Layer 2 Tunneling Protocol) |
| Autenticación PPP | CHAP / MS-CHAP |
| Autenticación AAA | Local |
| Usuario VPN | vpnuser |
| Contraseña VPN | Yeury0780 |
| Fase 1 — Cifrado | AES-128 |
| Fase 1 — Hash | SHA-1 |
| Fase 2 — Modo | Transport |
| Pre-Shared Key | Yeury0780 |
| Pool VPN | 172.16.78.1 — 172.16.78.10 |
| Cliente | Ubuntu 22 con xl2tpd y strongSwan |

---

## 5. Configuración

### 5.1 Configuración R1 — Servidor L2TP

> 📸 **SCREENSHOT:** Insertar captura del `show running-config` de R1 mostrando aaa, vpdn-group, virtual-template e ip local pool

```
aaa new-model
aaa authentication ppp VPN_AUTH local
aaa authorization network VPN_AUTHOR local
username vpnuser password Yeury0780
vpdn enable
vpdn-group L2TP
 accept-dialin
  protocol l2tp
  virtual-template 1
 no l2tp tunnel authentication
interface Virtual-Template1
 ip unnumbered FastEthernet0/0
 peer default ip address pool VPN-POOL
 ppp authentication chap ms-chap
ip local pool VPN-POOL 172.16.78.1 172.16.78.10
```

### 5.2 Configuración Linux-Client

> 📸 **SCREENSHOT:** Insertar captura de los archivos `/etc/ipsec.conf` y `/etc/xl2tpd/xl2tpd.conf`

```
/etc/ipsec.conf:
conn L2TP-PSK
    authby=secret
    keyexchange=ikev1
    type=transport
    left=10.7.85.2
    leftprotoport=17/1701
    right=10.7.80.1
    rightprotoport=17/1701
    ike=aes128-sha1-modp1024!
    esp=aes128-sha1!
```

---

## 6. Verificación y Funcionamiento

### 6.1 Estado sesión L2TP en R1

Ejecutar en R1:
```
show vpdn session
```
> 📸 **SCREENSHOT:** Insertar captura mostrando `vpnuser` con estado **est** (established)

### 6.2 Estado ISAKMP en R1

Ejecutar en R1:
```
show crypto isakmp sa
```
> 📸 **SCREENSHOT:** Insertar captura mostrando sesión IKEv1 en estado **QM_IDLE ACTIVE**

### 6.3 IP asignada en Linux-Client

Ejecutar en Linux-Client:
```
ip addr show ppp0
```
> 📸 **SCREENSHOT:** Insertar captura mostrando interfaz ppp0 con IP **172.16.78.1** asignada por el pool VPN

### 6.4 Rutas en Linux-Client

Ejecutar en Linux-Client:
```
ip route show
```
> 📸 **SCREENSHOT:** Insertar captura mostrando ruta hacia `192.168.7.0/24` por ppp0

### 6.5 Demostración de conectividad

Ejecutar en Linux-Client:
```
ping -c 4 192.168.7.2
```
> 📸 **SCREENSHOT:** Insertar captura del ping exitoso desde el cliente VPN hacia Linux-Server

---

## 7. Archivos del repositorio

| Archivo | Descripción |
|---|---|
| `YeuryLopez_20250780_Script_P9.txt` | Script de configuración |
| `YeuryLopez_20250780_Informe_P9.pdf` | Documentación técnica en PDF |
| `YeuryLopez_20250780_Links_P9.txt` | Enlace al video |
| `README.md` | Este archivo |
