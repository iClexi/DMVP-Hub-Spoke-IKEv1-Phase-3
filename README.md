# DMVPN Hub and Spoke Fase 3 con IKEv2 y EIGRP

![Estado](https://img.shields.io/badge/Estado-Completado-brightgreen)
![VPN](https://img.shields.io/badge/VPN-DMVPN-blue)
![Fase](https://img.shields.io/badge/Fase-3-purple)
![IKE](https://img.shields.io/badge/IKE-IKEv2-orange)
![IPsec](https://img.shields.io/badge/Seguridad-IPsec-red)
![Routing](https://img.shields.io/badge/Enrutamiento-EIGRP-yellow)
![NHRP](https://img.shields.io/badge/NHRP-Redirect%20%2B%20Shortcut-informational)
![GNS3](https://img.shields.io/badge/Laboratorio-GNS3-lightgrey)

**Nombre:** Michael Robles  
**Matrícula:** 20250845  
**Asignatura:** Seguridad de Redes  
**Práctica:** VPN Hub and Spoke punto a multipunto DMVPN Fase 3 con IKEv2 y enrutamiento dinámico  
**Repositorio:** `https://github.com/iClexi/DMVPN-Hub-Spoke-IKEv2-Phase-3`  
**Video demostrativo:** https://youtu.be/p5YHZzax3G0

## Índice

1. [Descripción del proyecto](#descripción-del-proyecto)
2. [Objetivo del laboratorio](#objetivo-del-laboratorio)
3. [Topología](#topología)
4. [Direccionamiento IP](#direccionamiento-ip)
5. [Parámetros de la VPN](#parámetros-de-la-vpn)
6. [Explicación de DMVPN Fase 3](#explicación-de-dmvpn-fase-3)
7. [Configuraciones](#configuraciones)
8. [Verificación](#verificación)
9. [Evidencias explicadas](#evidencias-explicadas)
10. [Estructura del repositorio](#estructura-del-repositorio)
11. [Entregables](#entregables)
12. [Conclusión](#conclusión)

---

## Descripción del proyecto

Este repositorio documenta la implementación de una VPN **DMVPN Hub and Spoke Fase 3** usando **IKEv2**, **IPsec**, **GRE multipoint**, **NHRP** y **EIGRP** como protocolo de enrutamiento dinámico.

La práctica conecta tres sedes: un router central **R1-HUB** y dos sucursales remotas llamadas **R2-SPOKE1** y **R3-SPOKE2**. Entre ellos se encuentra un router **ISP**, que representa la red pública o Internet. El ISP solo proporciona conectividad WAN; no conoce las redes LAN privadas. Las redes internas viajan dentro del túnel DMVPN protegido por IPsec.

---

## Objetivo del laboratorio

El objetivo principal es configurar una VPN **punto a multipunto** donde varias sucursales puedan comunicarse de forma segura y dinámica sin crear un túnel punto a punto manual para cada par de routers.

Con esta práctica se demuestra:

- Registro dinámico de los spokes contra el HUB usando NHRP.
- Protección del túnel usando IKEv2 e IPsec.
- Anuncio automático de redes mediante EIGRP.
- Comunicación entre sedes remotas usando una nube DMVPN.
- Funcionamiento de DMVPN Fase 3 mediante redirección y shortcuts.

---

## Topología

![Topología DMVPN Fase 3](images/01_topologia.png)

La topología está formada por los siguientes dispositivos:

| Dispositivo | Función |
|---|---|
| R1-HUB | Router central, servidor NHRP y punto de control de la nube DMVPN. |
| R2-SPOKE1 | Sucursal remota 1. Se registra con el HUB y participa en la nube DMVPN. |
| R3-SPOKE2 | Sucursal remota 2. Se registra con el HUB y participa en la nube DMVPN. |
| ISP | Red pública simulada en GNS3. Solo transporta tráfico WAN. |
| SW-HUB | Switch de acceso para la LAN del HUB. |
| SW-SPOKE-1 | Switch de acceso para la LAN del SPOKE1. |
| SW-SPOKE-2 | Switch de acceso para la LAN del SPOKE2. |
| PC-HUB | Host de prueba ubicado en la LAN del HUB. |
| PC-SPOKE-1 | Host de prueba ubicado en la LAN del SPOKE1. |
| PC-SPOKE-2 | Host de prueba ubicado en la LAN del SPOKE2. |

La idea de la topología es simular una empresa con una sede principal y dos sucursales. El ISP funciona como medio de transporte, mientras que DMVPN permite construir una red privada virtual sobre esa red pública.

---

## Direccionamiento IP

### Red WAN pública simulada

| Enlace | Dispositivo | Interfaz | Dirección IP | Vecino |
|---|---|---|---|---|
| ISP - R1 | ISP | G0/0 | 20.25.8.1/30 | R1-HUB: 20.25.8.2 |
| R1 - ISP | R1-HUB | G0/0 | 20.25.8.2/30 | ISP: 20.25.8.1 |
| ISP - R2 | ISP | G0/1 | 20.25.8.5/30 | R2-SPOKE1: 20.25.8.6 |
| R2 - ISP | R2-SPOKE1 | G0/0 | 20.25.8.6/30 | ISP: 20.25.8.5 |
| ISP - R3 | ISP | G0/2 | 20.25.8.9/30 | R3-SPOKE2: 20.25.8.10 |
| R3 - ISP | R3-SPOKE2 | G0/0 | 20.25.8.10/30 | ISP: 20.25.8.9 |

### Red de túnel DMVPN

| Router | Interfaz | Dirección IP |
|---|---|---|
| R1-HUB | Tunnel0 | 172.16.45.1/24 |
| R2-SPOKE1 | Tunnel0 | 172.16.45.2/24 |
| R3-SPOKE2 | Tunnel0 | 172.16.45.3/24 |

### Redes LAN internas

| Sede | Red | Gateway | Host de prueba |
|---|---|---|---|
| HUB | 192.168.45.0/24 | 192.168.45.1 | PC-HUB: 192.168.45.10 |
| SPOKE1 | 192.168.84.0/24 | 192.168.84.1 | PC-SPOKE-1: 192.168.84.10 |
| SPOKE2 | 192.168.8.0/24 | 192.168.8.1 | PC-SPOKE-2: 192.168.8.10 |

---

## Parámetros de la VPN

| Parámetro | Valor |
|---|---|
| Tipo de VPN | DMVPN Hub and Spoke multipunto |
| Fase | Fase 3 |
| Protocolo de túnel | GRE multipoint |
| Protocolo de control | NHRP |
| Seguridad | IPsec |
| Negociación | IKEv2 |
| Cifrado IKEv2 | AES-CBC-256 |
| Integridad IKEv2 | SHA-256 |
| Grupo Diffie-Hellman | Grupo 14 |
| Autenticación | Pre-Shared Key |
| PSK | ITLA20250845 |
| Transform-set IPsec | ESP-AES 256 + ESP-SHA256-HMAC |
| Modo IPsec | Transport |
| NHRP Network ID | 45 |
| NHRP Authentication | DMVPN45 |
| Tunnel Key | 45 |
| MTU | 1400 |
| TCP MSS | 1360 |
| Enrutamiento dinámico | EIGRP AS 45 |

---

## Explicación de DMVPN Fase 3

DMVPN permite crear una nube VPN multipunto. En vez de crear túneles separados entre cada router, se crea una sola interfaz de túnel multipunto. El HUB actúa como punto central de registro, y los spokes se registran contra él usando NHRP.

En **Fase 3**, el HUB puede funcionar como camino inicial para el tráfico entre spokes. Cuando un spoke intenta llegar a otro spoke, el tráfico puede pasar primero por el HUB. Si el HUB detecta que el origen y el destino pertenecen a la misma nube DMVPN, envía una redirección NHRP para indicarle al spoke de origen que puede crear un camino directo hacia el spoke destino.

La diferencia clave es esta:

- **Fase 1:** el tráfico entre spokes pasa por el HUB.
- **Fase 2:** el spoke aprende rutas con el next-hop real del otro spoke.
- **Fase 3:** el tráfico puede iniciar por el HUB, pero el HUB redirige al spoke para crear un shortcut directo.

En esta práctica, R1-HUB se encarga del registro y la redirección, mientras que R2-SPOKE1 y R3-SPOKE2 tienen la capacidad de crear shortcuts. IKEv2 negocia la seguridad, IPsec cifra el tráfico y EIGRP anuncia las redes LAN dinámicamente.

---

## Configuraciones

Las configuraciones completas se encuentran en la carpeta [`configs`](configs/):

| Dispositivo | Archivo |
|---|---|
| R1-HUB | [`configs/R1-HUB.cfg`](configs/R1-HUB.cfg) |
| R2-SPOKE1 | [`configs/R2-SPOKE1.cfg`](configs/R2-SPOKE1.cfg) |
| R3-SPOKE2 | [`configs/R3-SPOKE2.cfg`](configs/R3-SPOKE2.cfg) |
| ISP | [`configs/ISP.cfg`](configs/ISP.cfg) |
| Switches | [`configs/SWITCHES.cfg`](configs/SWITCHES.cfg) |

---

## Verificación

Para comprobar el funcionamiento de la VPN se utilizaron estos comandos:

```cisco
show ip interface brief
show crypto ikev2 sa
show crypto ipsec sa
show dmvpn
show ip nhrp
show ip eigrp neighbors
show ip route eigrp
traceroute 192.168.8.1 source 192.168.84.1
```

La verificación confirma que:

1. Las interfaces físicas y Tunnel0 están activas.
2. IKEv2 negocia correctamente y muestra estado `READY`.
3. DMVPN registra los spokes contra el HUB.
4. NHRP resuelve las direcciones de túnel y las direcciones NBMA.
5. EIGRP aprende rutas dinámicamente entre las sedes.
6. El traceroute desde SPOKE1 hacia SPOKE2 demuestra comunicación entre LANs remotas.

---

## Evidencias explicadas

Esta sección documenta cada captura usada como prueba del laboratorio. Cada imagen valida una parte específica de la VPN DMVPN Fase 3 con IKEv2 y EIGRP.

### 1. Topología del laboratorio

![Topología](images/01_topologia.png)

**Explicación:**  
La imagen muestra la topología completa implementada en GNS3. Se observa un diseño **Hub and Spoke** compuesto por R1-HUB, R2-SPOKE1, R3-SPOKE2 y un router ISP. También aparecen los switches de acceso y las PCs de prueba de cada sede.

**Qué demuestra:**  
Demuestra que la práctica fue montada con una sede central y dos sucursales remotas. El ISP representa la red pública, mientras que las LAN privadas quedan detrás de cada router. Esta estructura permite validar el objetivo de DMVPN: conectar múltiples sedes usando una nube multipunto segura.

**Relación con Fase 3:**  
R1-HUB funciona como servidor NHRP. R2 y R3 se registran contra el HUB y, cuando sea necesario, pueden crear shortcuts para comunicarse directamente entre ellos.

---

### 2. Interfaces activas en R1-HUB

![R1 show ip interface brief](images/02_r1_show_ip_interface_brief.png)

**Explicación:**  
La captura corresponde al comando `show ip interface brief` ejecutado en R1-HUB. Se muestran la interfaz WAN, la interfaz LAN y la interfaz virtual Tunnel0.

| Interfaz | IP | Estado | Función |
|---|---|---|---|
| GigabitEthernet0/0 | 20.25.8.2 | up/up | Enlace WAN hacia el ISP. |
| GigabitEthernet0/1 | 192.168.45.1 | up/up | Gateway de la LAN del HUB. |
| Tunnel0 | 172.16.45.1 | up/up | Interfaz de la nube DMVPN. |

**Qué demuestra:**  
Confirma que el HUB tiene sus interfaces principales operativas. El estado `up/up` indica que la capa física y el protocolo de línea están funcionando correctamente.

**Por qué importa:**  
La dirección 20.25.8.2 es la dirección NBMA del HUB usada por los spokes para encontrarlo por la red pública. La dirección 172.16.45.1 identifica al HUB dentro del túnel DMVPN.

---

### 3. Interfaces activas en R3-SPOKE2

![R3 show ip interface brief](images/03_r3_show_ip_interface_brief.png)

**Explicación:**  
La captura muestra el estado de interfaces en R3-SPOKE2. Se valida que R3 tiene conectividad WAN, conectividad LAN y Tunnel0 activo.

| Interfaz | IP | Estado | Función |
|---|---|---|---|
| GigabitEthernet0/0 | 20.25.8.10 | up/up | Enlace WAN hacia el ISP. |
| GigabitEthernet0/1 | 192.168.8.1 | up/up | Gateway de la LAN del SPOKE2. |
| Tunnel0 | 172.16.45.3 | up/up | Interfaz DMVPN del SPOKE2. |

**Qué demuestra:**  
Confirma que R3 está integrado correctamente en la topología. Su WAN permite llegar al HUB por medio del ISP, su LAN conecta la red del SPOKE2 y Tunnel0 confirma su participación en la nube DMVPN.

---

### 4. IKEv2 activo en R3-SPOKE2

![R3 show crypto ikev2 sa](images/04_r3_show_crypto_ikev2_sa.png)

**Explicación:**  
La captura muestra el comando `show crypto ikev2 sa` en R3-SPOKE2. El estado de la sesión IKEv2 aparece como `READY`, lo que indica que la negociación de seguridad entre R3 y R1-HUB fue exitosa.

| Campo | Valor observado |
|---|---|
| Local | 20.25.8.10/500 |
| Remote | 20.25.8.2/500 |
| Estado | READY |
| Cifrado | AES-CBC 256 |
| PRF | SHA256 |
| Hash | SHA256 |
| Grupo DH | 14 |
| Autenticación | PSK |

**Qué demuestra:**  
Valida que IKEv2 está funcionando correctamente. R3 logró autenticarse con el HUB usando clave precompartida y ambos routers negociaron los parámetros criptográficos.

**Por qué importa:**  
IKEv2 es la negociación de seguridad. Si esta sesión no está en estado `READY`, IPsec no puede proteger correctamente el tráfico del túnel.

---

### 5. DMVPN en R1-HUB con dos peers registrados

![R1 show dmvpn](images/05_r1_show_dmvpn.png)

**Explicación:**  
Esta captura muestra el comando `show dmvpn` ejecutado en R1-HUB. El router aparece como tipo Hub y muestra dos peers NHRP registrados.

| Peer NBMA | Peer Tunnel | Estado | Atributo |
|---|---|---|---|
| 20.25.8.6 | 172.16.45.2 | UP | D |
| 20.25.8.10 | 172.16.45.3 | UP | D |

**Qué demuestra:**  
Confirma que R2 y R3 se registraron correctamente contra R1-HUB. El atributo `D` significa que las entradas son dinámicas, lo cual es correcto en el HUB porque los spokes se registran mediante NHRP.

**Por qué importa:**  
Esta evidencia demuestra que el HUB cumple su función central: recibir y mantener los registros NHRP de los spokes. Sin este registro, no habría una nube DMVPN funcional.

---

### 6. NHRP en R2-SPOKE1 hacia el HUB

![R2 show ip nhrp](images/06_r2_show_ip_nhrp.png)

**Explicación:**  
La captura corresponde al comando `show ip nhrp` ejecutado en R2-SPOKE1. Se observa que R2 tiene una entrada NHRP hacia el HUB.

| Campo | Valor observado |
|---|---|
| IP de túnel del HUB | 172.16.45.1 |
| NBMA del HUB | 20.25.8.2 |
| Tipo | static |
| Interfaz | Tunnel0 |

**Qué demuestra:**  
Confirma que R2 conoce la relación entre la IP de túnel del HUB y su IP WAN. Esta información es necesaria para que el spoke pueda encontrar al HUB inicialmente y registrarse en la nube DMVPN.

**Relación con Fase 3:**  
En Fase 3, R2 primero se apoya en el HUB para el registro y descubrimiento. Luego puede aceptar redirecciones del HUB para crear shortcuts hacia otros spokes.

---

### 7. Traceroute de SPOKE1 hacia SPOKE2

![R2 traceroute](images/07_r2_traceroute_spoke_to_spoke.png)

**Explicación:**  
La captura muestra un `traceroute` ejecutado desde R2-SPOKE1 hacia la LAN de R3-SPOKE2, usando como origen la LAN de R2.

| Origen | Destino | Propósito |
|---|---|---|
| 192.168.84.1 | 192.168.8.1 | Validar comunicación entre SPOKE1 y SPOKE2. |

| Salto | IP | Interpretación |
|---|---|---|
| 1 | 172.16.45.1 | R1-HUB dentro del túnel DMVPN. |
| 2 | 172.16.45.3 | R3-SPOKE2 dentro del túnel DMVPN. |

**Qué demuestra:**  
Confirma que R2 puede llegar a la red de R3 usando la nube DMVPN. También demuestra que el enrutamiento dinámico permite alcanzar redes LAN remotas que no están conectadas directamente.

**Relación con Fase 3:**  
En Fase 3 es normal que el tráfico pueda iniciar pasando por el HUB. Luego, mediante NHRP redirect y shortcut, los spokes pueden optimizar el camino cuando se genera la entrada dinámica correspondiente.

---

### 8. DMVPN en R2-SPOKE1

![R2 show dmvpn](images/08_r2_show_dmvpn.png)

**Explicación:**  
Esta imagen muestra el comando `show dmvpn` en R2-SPOKE1. R2 aparece como tipo Spoke y muestra como peer principal al HUB.

| Peer NBMA | Peer Tunnel | Estado | Atributo |
|---|---|---|---|
| 20.25.8.2 | 172.16.45.1 | UP | S |

**Qué demuestra:**  
Confirma que R2-SPOKE1 está conectado correctamente al HUB dentro de la nube DMVPN. El atributo `S` indica que la entrada es estática, porque el spoke necesita conocer manualmente al HUB para iniciar el registro NHRP.

**Por qué importa:**  
Esta evidencia valida la base del diseño Hub and Spoke: los spokes primero conocen al HUB, se registran con él y desde ahí pueden descubrir caminos hacia otros routers dentro de la nube.

---

### Resumen de validación por evidencias

| Evidencia | Qué valida |
|---|---|
| Topología | Diseño Hub and Spoke con ISP, switches y PCs. |
| R1 show ip interface brief | Interfaces WAN, LAN y Tunnel0 del HUB activas. |
| R3 show ip interface brief | Interfaces WAN, LAN y Tunnel0 del SPOKE2 activas. |
| R3 show crypto ikev2 sa | Negociación IKEv2 exitosa en estado READY. |
| R1 show dmvpn | R2 y R3 registrados dinámicamente en el HUB. |
| R2 show ip nhrp | Mapeo NHRP del SPOKE1 hacia el HUB. |
| R2 traceroute | Comunicación entre LAN del SPOKE1 y LAN del SPOKE2. |
| R2 show dmvpn | SPOKE1 conectado al HUB en la nube DMVPN. |

Con estas evidencias se comprueba que la VPN está funcionando correctamente: las interfaces están activas, IKEv2 negocia la seguridad, IPsec protege el túnel, DMVPN registra los peers, NHRP resuelve las direcciones necesarias y EIGRP permite comunicación entre redes LAN remotas.

---

## Estructura del repositorio

```text
DMVPN-Hub-Spoke-IKEv2-Phase-3/
├── README.md
├── configs/
│   ├── R1-HUB.cfg
│   ├── R2-SPOKE1.cfg
│   ├── R3-SPOKE2.cfg
│   ├── ISP.cfg
│   ├── SWITCHES.cfg
├── docs/
│   ├── Documentacion Tecnica Profesional.docx
│   └── Documentacion Tecnica Profesional.pdf
├── images/
│   ├── 01_topologia.png
│   ├── 02_r1_show_ip_interface_brief.png
│   ├── 03_r3_show_ip_interface_brief.png
│   ├── 04_r3_show_crypto_ikev2_sa.png
│   ├── 05_r1_show_dmvpn.png
│   ├── 06_r2_show_ip_nhrp.png
│   ├── 07_r2_traceroute_spoke_to_spoke.png
│   └── 08_r2_show_dmvpn.png
```

---

## Conclusión

La práctica demuestra una VPN DMVPN Fase 3 funcional con IKEv2, IPsec y EIGRP. El HUB centraliza el registro NHRP y los spokes pueden comunicarse con otras sedes de forma dinámica. Las evidencias muestran interfaces activas, negociación IKEv2 en estado `READY`, peers DMVPN registrados, resolución NHRP y comunicación entre LANs remotas.
