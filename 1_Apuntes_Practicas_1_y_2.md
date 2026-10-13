# APUNTES — Redes de Computadores · Prácticas 1 y 2

> **Práctica 1:** Introducción a las redes (Ethernet, MAC, ARP, IP, direccionamiento y subredes)
> **Práctica 2:** Protocolo ICMP (ping, fragmentación, Destination Unreachable, Redirect, Time Exceeded, Source Quench, rutas estáticas, rendimiento)
>
> Leyenda: 📌 = sale casi seguro en el examen · ⚠️ = trampa típica · ➕ = ampliación que no viene literal en el PDF pero ayuda a entenderlo (o a responder preguntas de razonamiento)

---

## ÍNDICE

0. [La topología del laboratorio (apréndetela de memoria)](#0-la-topología-del-laboratorio)
1. [Arquitectura TCP/IP y encapsulación](#1-arquitectura-tcpip-y-encapsulación)
2. [Hub vs Switch](#2-hub-concentrador-vs-switch-conmutador)
3. [Ethernet: historia, CSMA/CD, formatos de trama y MTU](#3-ethernet)
4. [Direcciones MAC](#4-direcciones-mac)
5. [ARP](#5-arp-address-resolution-protocol)
6. [IP: direcciones, máscara, clases, subredes](#6-ip-direccionamiento)
7. [Configuración TCP/IP de un equipo y envío dentro/fuera de la red](#7-configuración-de-un-equipo-y-envío-de-paquetes-dentro-y-fuera-de-la-red)
8. [Formato del datagrama IPv4 (campo a campo)](#8-formato-del-datagrama-ipv4)
9. [ICMP: generalidades, solicitud vs error, reglas](#9-icmp-generalidades)
10. [Echo / Echo Reply y el comando ping](#10-echo-request--echo-reply-y-ping)
11. [Fragmentación IP (con todos los cálculos)](#11-fragmentación-ip)
12. [Destination Unreachable (tipo 3)](#12-destination-unreachable-tipo-3)
13. [Tabla de rutas, route y Redirect (tipo 5)](#13-tabla-de-rutas-comando-route-y-redirect-tipo-5)
14. [Time Exceeded (tipo 11) y tracert](#14-time-exceeded-tipo-11-y-tracert)
15. [Source Quench (tipo 4)](#15-source-quench-tipo-4)
16. [Evaluación del rendimiento: retardo medio y desviación típica](#16-evaluación-experimental-del-rendimiento)
17. [Comandos de Windows y filtros de Wireshark](#17-comandos-y-filtros-de-wireshark)
18. [Cuestiones de la Práctica 1 resueltas y razonadas](#18-cuestiones-de-la-práctica-1-resueltas)
19. [Cuestiones de la Práctica 2 resueltas y razonadas](#19-cuestiones-de-la-práctica-2-resueltas)

---

## 0. La topología del laboratorio

📌 Casi todas las preguntas de "¿qué router contesta?", "¿qué MAC aparece?" o "¿por dónde va el paquete?" dependen de este esquema. **Apréndetelo.**

```
 Internet ── Ethernet 172.18.41.0/28 ──[172.18.41.4] R1 [10.3.0.1]═══ PaP 10.3.0.0/24 (10 Mbps) ═══[10.3.0.2] R4
 (Red EPS)      (100 Mbps)              (NAT)  [10.4.0.2]                                              [10.6.0.1]
                                                   │                                                       │
              ┌──────── Ethernet 10.4.0.0/24 ──────┴─────┐          ┌──────────── Ethernet 10.6.0.0/24 ─────┴───┬──────────┬──────────┐
              │                                          │          │                          │                │          │
         [10.4.0.1]                                 [10.4.0.3]  [10.6.0.2]                [10.6.0.3]        Linux2     Linux3
             R2                                          └── R3 ────┘                         R5           10.6.0.5   10.6.0.6
       [172.20.43.240]                               [172.20.43.241]                    [172.20.43.242]
              │                                             │                                  │
 ═════════════╧═════════ Ethernet 172.20.43.192/26 (switch 100 Mbps/1 Gbps) — "Red aula L24" ══╧══════════════════
        │            │            │      ...       │                    │
  172.20.43.198    .199         .200             .231            Linux1 172.20.43.243
                     (PCs de los alumnos)
```
R3 tiene **tres** interfaces (10.4.0.3, 10.6.0.2 y 172.20.43.241); R5 tiene dos (10.6.0.3 y 172.20.43.242).

### Tabla resumen de interfaces

| Router | Interfaces (IP) | Redes a las que está conectado |
|---|---|---|
| **R1** | 172.18.41.4 · 10.4.0.2 · 10.3.0.1 | Internet (172.18.41.0/28, con NAT), 10.4.0.0/24, PaP 10.3.0.0/24 |
| **R2** | 10.4.0.1 · **172.20.43.240** | 10.4.0.0/24, red alumnos — **es la puerta de enlace por defecto** de los PCs |
| **R3** | 10.4.0.3 · 10.6.0.2 · **172.20.43.241** | 10.4.0.0/24, 10.6.0.0/24, red alumnos |
| **R4** | 10.3.0.2 · 10.6.0.1 | PaP 10.3.0.0/24, 10.6.0.0/24 |
| **R5** | 10.6.0.3 · **172.20.43.242** | 10.6.0.0/24, red alumnos |

| Segmento | Tipo | Notas |
|---|---|---|
| 172.20.43.192/26 | Ethernet conmutada (switch) | Red de los alumnos. Máscara 255.255.255.192. Hosts .193–.254. Broadcast **.255**. Aquí están R2, R3, R5, Linux1 y los PCs |
| 10.4.0.0/24 | Ethernet conmutada | R1, R2, R3 |
| 10.6.0.0/24 | Ethernet conmutada | R3, R4, R5, Linux2 (.5), Linux3 (.6) |
| 10.3.0.0/24 | **Punto a punto (PaP), 10 Mbps** | Solo R1 (.1) y R4 (.2). **Es el único segmento NO Ethernet** — tiene un MTU distinto (más pequeño) |
| 172.18.41.0/28 | Ethernet 100 Mbps | Salida a Internet a través de R1 (NAT). Hosts .1–.14, broadcast .15 |

- Los routers se conectan a los segmentos Ethernet a **100 Mbps**; los switches admiten 100 Mbps y 1 Gbps.
- Hay 3 Linux para experimentos: Linux1 (172.20.43.243), Linux2 (10.6.0.5), Linux3 (10.6.0.6).
- **Toda la infraestructura del laboratorio es Ethernet con conmutadores (switches)** → con Wireshark en tu PC solo verás **tus** paquetes (los que envías/recibes) + los de **difusión** (broadcast) y multidifusión.
- Ruta de salida por defecto de los PCs: `0.0.0.0  0.0.0.0  172.20.43.240` (R2). Si no aparece se crea con `c:\pracredes`.
- Servidores DNS de la UA: **193.145.233.5** y **193.145.233.6**.

📌 **Segmento de red** en esta asignatura = interpretación de **nivel de red**: conjunto de dispositivos que comparten la **misma dirección de red IP**.

📌 Ruta conocida (viene en el enunciado de la P2, tracert a 10.3.0.2): `PC → 172.20.43.240 (R2) → 10.4.0.2 (R1) → 10.3.0.2 (R4)`.

---

## 1. Arquitectura TCP/IP y encapsulación

| Nivel TCP/IP | Protocolos de estas prácticas | Dirección que usa | Unidad |
|---|---|---|---|
| Aplicación | ping, tracert, DNS, WWW, FTP... | nombres (www.ua.es) | mensaje |
| Transporte | TCP, UDP (**ping NO los usa**) | puertos | segmento/datagrama |
| Red | **IP**, **ICMP** (va dentro de IP), (ARP hace de puente con enlace) | **IP (32 bits)** | datagrama / paquete IP |
| Enlace | **Ethernet** (DIX / 802.3), PaP | **MAC (48 bits)** | trama / paquete Ethernet |
| Físico | cable par trenzado UTP con RJ‑45, coaxial... | — | bits |

Encapsulación de un ping (lo que ves en Wireshark):

```
| Cab. Ethernet 14B | Cab. IP 20B | Cab. ICMP 8B | DATOS (-l bytes) | (CRC 4B, Wireshark no lo muestra)
                    |<----------------- datagrama IP (≤ MTU = 1500) ------------->|
|<------------------------- trama Ethernet (≤ 1514 en Wireshark, 1518 con CRC) -------------->|
```

- ARP va **directamente sobre Ethernet** (no lleva cabecera IP). Por eso ARP es un paquete "no IP".
- ICMP va **dentro de IP** (campo Protocolo = 1).

---

## 2. Hub (concentrador) vs Switch (conmutador)

| | **Hub / concentrador** | **Switch / conmutador** |
|---|---|---|
| Funcionamiento interno | **Bus común**: repite todo lo que recibe por TODOS los puertos | **Malla de interconexión**: envía cada trama solo al puerto del destinatario |
| Colisiones | Sí, cuando dos equipos transmiten a la vez → hay que retransmitir → lentitud | Se evitan muchas: permite **transmisiones simultáneas entre pares distintos** |
| Monitor de red (Wireshark) | Captura los paquetes de **todos** los PCs de la red | Captura solo los paquetes que **envía o recibe tu PC** (+ broadcast) |
| Seguridad / privacidad | Baja | **Mayor** (la info no es accesible físicamente a los demás) |
| Aprovechamiento del medio | Peor | **Mejor** |

- Conexión con **cable par trenzado (UTP)** y conectores **RJ‑45** (parecidos a los RJ‑11 de teléfono).
- ⚠️ En el laboratorio **todo son switches**: si te preguntan por qué no ves el ping de tu compañero a otro equipo → porque es un switch.

---

## 3. Ethernet

### 3.1 Historia y acceso al medio
- Red de área local de **difusión** basada en un **bus común**.
- La original (introducida por **Novell**, según el PDF) era un **bus de cable coaxial** con derivadores en **T**, **10 Mbps**, y con el problema clásico de las **colisiones**.
- Funcionamiento (➕ es lo que se llama **CSMA/CD**):
  1. El equipo que quiere transmitir **escucha el canal** un tiempo.
  2. Si está libre, **transmite**.
  3. Si dos lo hacen a la vez → **colisión**: el emisor detecta una señal de interferencia (superposición).
  4. Espera un **tiempo aleatorio** y retransmite (la aleatoriedad hace poco probable volver a colisionar).
- Funciona bien con poco tráfico y pocos equipos (**menos de unos 25**). Si hay demasiados, la red entra en **estado de colisión permanente**.
- Solución: **Ethernet conmutada** (switch) → permitió subir velocidades:

| Versión | Velocidad |
|---|---|
| Ethernet original | 10 Mbps |
| Fast Ethernet conmutada | 100 Mbps |
| Gigabit Ethernet | 1000 Mbps = 1 Gbps |
| Últimas versiones | 10 Gbps |

### 3.2 Los dos formatos de trama 📌

```
Ethernet DIX (RFC 894)  ← LA QUE USA TCP/IP
+-----------+-----------+------+---------------------+-----+
| Dir.Dest. | Dir.Fuente| Tipo |    Datagrama IP     | CRC |
|    6      |    6      |  2   |     46 – 1500       |  4  |
+-----------+-----------+------+---------------------+-----+

IEEE 802.2/802.3 (RFC 1042)  ← la que usa LLC
+-----------+-----------+------+------+------+------+----------+------+--------------+-----+
| Dir.Dest. | Dir.Fuente| Long.| DSAP | SSAP | Cntl | org code | Tipo | Datagrama IP | CRC |
|    6      |    6      |  2   |  1   |  1   |  1   |    3     |  2   |  38 – 1492   |  4  |
+-----------+-----------+------+------+------+------+----------+------+--------------+-----+
            |  802.3 MAC        |     802.2 LLC      |   802.2 SNAP  |
```

| | Ethernet DIX | IEEE 802.3 |
|---|---|---|
| RFC | **894** | 1042 |
| Lo usa | **TCP/IP** | LLC |
| 3er campo | **Tipo** (2 B) — ➕ 0x0800 = IPv4, 0x0806 = ARP | **Longitud** (2 B) |
| Datos (MTU) | **46 – 1500** | 38 – 1492 |
| Cabecera | **14 B** (6+6+2) + CRC 4 | 6+6+2+3 (LLC) +5 (SNAP) = 22 B + CRC 4 |

### 3.3 MTU (Maximum Transfer Unit) 📌
- **MTU** = tamaño máximo de **datos** (es decir, del datagrama IP completo) que transporta una trama del nivel de enlace.
  - **DIX: 1500 bytes**; **802.3: 1492 bytes**.
- Si un datagrama IP > MTU → **IP lo fragmenta**. ⚠️ **Ethernet NO puede fragmentar** (su cabecera no tiene campos para ello): es la **capa de red** la que debe entregar paquetes del tamaño adecuado.
- El MTU depende del tipo de enlace: **tasa de error media (BER)**, **política de acceso al medio**, **velocidad de transmisión**, etc. (por eso el PaP del laboratorio tiene otro MTU).

---

## 4. Direcciones MAC

- También llamada: **dirección física, dirección MAC (Media Access Control), dirección Ethernet o dirección del adaptador de red**.
- **48 bits = 6 bytes**, en hexadecimal: 12 dígitos hex agrupados de 2 en 2 separados por `:` → `00:00:3F:34:5A:BB`.
- Estructura:
  - **3 primeros bytes** (izquierda): fijados para cada **fabricante** (➕ se llama OUI).
  - **3 últimos bytes**: los asigna el **fabricante** a cada adaptador.
  - → Cada MAC es **única e irrepetible** (necesario para que convivan adaptadores de distintos fabricantes).
- Cada adaptador procesa tramas dirigidas a: **su propia MAC** y a la **MAC de broadcast** `FF:FF:FF:FF:FF:FF` (48 bits a 1). (Y a las de grupo a las que pertenezca.)
- **Broadcast**: la atienden **todos los equipos del mismo segmento**. La usa mucho **ARP**.
- **Multidifusión (dirección de grupo)** 📌: el **bit menos significativo del primer byte = 1** → `xxxxxxx1:XX:XX:XX:XX:XX`.

**Cómo comprobar si una MAC es de grupo:** mira el **primer byte** y si es **impar** → grupo/multicast.

| MAC | 1er byte en binario | ¿Grupo? |
|---|---|---|
| 01:00:0C:CC:CC:CC | 0000 000**1** | ✅ multicast |
| 01:80:C2:00:00:00 | 0000 000**1** | ✅ multicast |
| 33:33:01:23:14:31 | 0011 001**1** | ✅ multicast |
| FF:FF:FF:FF:FF:FF | 1111 111**1** | ✅ (broadcast, caso especial de grupo) |
| 00:00:3F:34:5A:BB | 0000 000**0** | ❌ unicast |
| 02:1A:... | 0000 001**0** | ❌ unicast (el 2 es par) |
| 0D:... | 0000 110**1** | ✅ (D = 13, impar) |

⚠️ Truco: el primer byte es impar ⇔ es de grupo. Impares hex: 1,3,5,7,9,B,D,F en el **segundo dígito** del primer byte.

---

## 5. ARP (Address Resolution Protocol)

### 5.1 Para qué sirve 📌
- Ethernet solo permite comunicar dentro del **mismo segmento de difusión** y necesita la **MAC destino**.
- IP usa **direcciones IP** (globales). Un equipo en Ethernet conectado a Internet tiene **IP + MAC**.
- **ARP** asocia **dinámicamente IP (nivel de red) → MAC (nivel de enlace)**. Estándar: **RFC 826**.

### 5.2 Funcionamiento (notación del PDF)
Red: `10.1.10.1 (MAC1)` quiere enviar a `10.1.10.2 (MAC2)`. Solo conoce su MAC y la IP destino.

```
1) ARP Request (pregunta, a TODOS):
   MAC1 → FF:FF:FF:FF:FF:FF   ARP Request ¿10.1.10.2?
2) ARP Reply (respuesta, solo al que pregunta → UNICAST):
   MAC2 → MAC1               ARP Reply 10.1.10.2 ⇔ MAC2
3) Ya puede enviar el datagrama:
   MAC1 → MAC2  | 10.1.10.1 → 10.1.10.2 | datos
```

⚠️ El **Request es broadcast**; el **Reply es unicast** (dirigido a MAC1).

📌 **El ARP Request lleva la MAC/IP del remitente**: el destino (10.1.10.2) la guarda en **su** caché ARP. Por eso, **cuando contesta, no necesita hacer ARP** → solo aparece **UNA secuencia ARP** en un ping entre dos equipos de la misma red.

### 5.3 Caché ARP 📌
- Hacer ARP antes de cada datagrama aumentaría el tráfico → se guarda la asociación IP⇔MAC en la **memoria caché ARP**.
- Al iniciar el sistema está **vacía**. Cada entrada dura un tiempo; si no se usa, **se borra**. Así, si un equipo cambia su IP o MAC, se vuelve a aprender automáticamente.
- Tiempos de expiración: **20 min en Unix**; **desde 2 min (Windows XP) hasta varias horas (Windows 10)**.
- Comandos: `arp -a` (ver tabla), `arp -d IP` (borrar entrada).
- Si la IP está en la caché → **no aparecen paquetes ARP** al hacer ping. Si no está → sí aparecen.

### 5.4 ARP solo resuelve en TU segmento 📌⚠️
- Si el destino está **en otra red**, ARP pregunta por la **MAC de la puerta de enlace**, **no** por la IP del destino final.
- Desde tu PC **solo ves la secuencia ARP de tu segmento físico**; las de los routers en otros segmentos no las ves.

### 5.5 Nota 2 del PDF (ARP Spoofing)
Hay un mecanismo contra la suplantación ARP que envía ARP con **MAC destino distinta de FF:FF:FF:FF:FF:FF** (unicast). En la práctica **solo se consideran los ARP con destino FF:FF:FF:FF:FF:FF y sus respuestas**.

---

## 6. IP: direccionamiento

### 6.1 Objetivo de IP
- Encaminar paquetes a través de una red formada por la interconexión de **distintos segmentos físicos** (Internet).
- Proporciona un **sistema de direccionamiento global**.
- Estudiamos **IPv4** (la mayoría de equipos de usuario). **IPv6** está en los troncales (**backbone**) y será el futuro.

### 6.2 Dirección IPv4
- **32 bits**, únicos, en 4 grupos de 8 bits separados por puntos: `x.x.x.x` (x de 0 a 255).
- Dos partes: **parte de red** (igual en todos los equipos de la misma red; son siempre los **primeros bits**) y **parte de máquina** (distinta para cada equipo).
- El **router** interconecta redes; tiene **un adaptador por segmento**, cada uno con su **MAC y su IP**.

### 6.3 Máscara de red 📌
- Valor de 32 bits: los **n primeros bits a 1** (parte de red) y el resto a **0** (parte de máquina).
- Ej.: `11111111.11111111.00000000.00000000` = **255.255.0.0** = **/16**.
- **Prefijo**: `/24` = 24 bits a 1 = 255.255.255.0.

### 6.4 Direcciones reservadas en cada red 📌
| | Bits de máquina | Ejemplo (120.12.0.0/16) | Uso |
|---|---|---|---|
| **Dirección de red** | **todos a 0** | 120.12.0.0 | La usan los **routers** para las rutas |
| **Dirección de broadcast** | **todos a 1** | 120.12.255.255 | Todos los equipos de esa red procesan el paquete (análogo al broadcast MAC) |

→ **Nº de máquinas asignables = 2^h − 2** (h = bits de máquina).

### 6.5 Clases de direcciones IP 📌

| Clase | Primeros bits | Rango | Máscara por defecto | Bits red / máquina | Nº redes | Máquinas/red |
|---|---|---|---|---|---|---|
| **A** | `0` | 0.0.0.0 – 127.255.255.255 | 255.0.0.0 (/8) | 7 / 24 | 2^7 = 128 | 2^24 − 2 = 16.777.214 |
| **B** | `10` | 128.0.0.0 – 191.255.255.255 | 255.255.0.0 (/16) | 14 / 16 | 2^14 = 16.384 | 2^16 − 2 = 65.534 |
| **C** | `110` | 192.0.0.0 – 223.255.255.255 | 255.255.255.0 (/24) | 21 / 8 | 2^21 = **2.097.152** | 2^8 − 2 = **254** |
| **D** | `1110` | 224.0.0.0 – 239.255.255.255 | — | 28 bits multicast | — | **Multicast** |
| **E** | `11110` | 240.0.0.0 – 247.255.255.255 (según la figura) | — | 27 bits | — | **Futuras ampliaciones** |

⚠️ La figura del PDF pone la clase E hasta **247.255.255.255** (porque empieza por 11110). Si en el examen te preguntan "según la práctica", usa ese valor.

**Cómo saber la clase rápido (primer octeto):** 0–127 A · 128–191 B · 192–223 C · 224–239 D · 240+ E.

### 6.6 Subredes (ampliación de la máscara) 📌📌
- Partir una red en varias cuando no pueden estar todas las máquinas en el mismo segmento físico.
- Se **amplía la máscara en n bits** con **2^n ≥ nº de subredes pedidas**.
- Ej. de la figura 6: 10.0.0.0 es clase A (/8), pero se usa /16 → subredes 10.1.0.0 y 10.2.0.0.

#### Método paso a paso
1. Bits a ampliar: menor n tal que **2^n ≥ subredes**.
2. Nuevo prefijo = prefijo + n → nueva máscara.
3. **Salto (tamaño de bloque)** = 256 − (valor del octeto "interesante" de la máscara). O bien = 2^(bits de máquina que quedan en ese octeto).
4. Para cada subred: **red** (bits de máquina a 0), **primera IP** = red + 1, **broadcast** = siguiente red − 1, **última IP** = broadcast − 1.
5. ⚠️ En esta asignatura se listan **todas** las 2^n subredes (incluida la "toda 0" y la "toda 1"), aunque sobren (el PDF dice "una de las cuales no se emplearía").

#### Ejemplo del PDF: 192.168.100.0/24 en 3 subredes
- 2^1 = 2 < 3 → 2^**2** = 4 ≥ 3 → ampliar **2 bits** → **/26** → máscara **255.255.255.192** (`11000000` = 192). Salto = 64.

| Subred | Rango de equipos | Broadcast |
|---|---|---|
| 192.168.100.**0**/26 (`00`000000) | .1 – .62 | .63 |
| 192.168.100.**64**/26 (`01`000000) | .65 – .126 | .127 |
| 192.168.100.**128**/26 (`10`000000) | .129 – .190 | .191 |
| 192.168.100.**192**/26 (`11`000000) | .193 – .254 | .255 |

Cada una: 2^6 − 2 = **62 equipos**. Fíjate que **la red de los alumnos 172.20.43.192/26 es exactamente la cuarta subred de este tipo**.

#### Tabla de máscaras que debes saber de memoria
| Prefijo | Máscara | | Prefijo | Máscara |
|---|---|---|---|---|
| /8 | 255.0.0.0 | | /24 | 255.255.255.0 |
| /16 | 255.255.0.0 | | /25 | 255.255.255.128 |
| /17 | 255.255.128.0 | | /26 | 255.255.255.192 |
| /18 | 255.255.192.0 | | /27 | 255.255.255.224 |
| /19 | 255.255.224.0 | | /28 | 255.255.255.240 |
| /20 | 255.255.240.0 | | /29 | 255.255.255.248 |
| /21 | 255.255.248.0 | | /30 | 255.255.255.252 |
| /22 | 255.255.252.0 | | /32 | 255.255.255.255 (un solo host) |
| /23 | 255.255.254.0 | | | |

Valores de un octeto de máscara: 128, 192, 224, 240, 248, 252, 254, 255.

---

## 7. Configuración de un equipo y envío de paquetes dentro y fuera de la red

### 7.1 Parámetros de configuración TCP/IP 📌
1. **Dirección IP**.
2. **Máscara de red** (puede estar ampliada si es una subred).
3. **Puerta de enlace por defecto (P.E.D.) / default gateway (D.G.)**: IP de un **router** de tu red que da acceso a otras redes. **Sin ella solo puedes comunicarte dentro de tu propia red IP.**
4. **Servidores DNS**: traducen nombres (www.ua.es) a IP. Están en una **estructura jerárquica** (si uno no conoce un nombre, pregunta a otro). En la UA: 193.145.233.5 y .6.

### 7.2 ¿El destino está en mi red? (decisión del emisor) 📌
El emisor aplica **su máscara** a la IP destino y compara la parte de red con la suya:
- **Iguales** → destino en **mi red** → ARP pregunta por la **IP del destino**.
- **Distintas** → destino en **otra red** → envío a la **puerta de enlace** → ARP pregunta por la **IP de la puerta de enlace**.

### 7.3 Caso 1: misma red (ping -n 1 10.1.10.2 desde 10.1.10.1, máscara 255.255.0.0)
```
MAC1 → FF:FF:FF:FF:FF:FF   ARP Request ¿10.1.10.2?
MAC2 → MAC1               ARP Reply 10.1.10.2 ⇔ MAC2
MAC1 → MAC2  | 10.1.10.1 → 10.1.10.2 | ICMP Echo Request | datos
MAC2 → MAC1  | 10.1.10.2 → 10.1.10.1 | ICMP Echo Reply   | datos   ← sin ARP: aprendió MAC1 del Request
```
Total: **4 paquetes** (2 ARP + 2 ICMP).

### 7.4 Caso 2: otra red (ping -n 1 10.2.10.3 desde 10.1.10.1) 📌📌
Red A 10.1.0.0/16: PC1 10.1.10.1 (MAC1), router 10.1.10.4 (MAC4).
Red B 10.2.0.0/16: router 10.2.x.x (MAC5), PC 10.2.10.3 (MAC6).

```
--- Segmento A ---
MAC1 → FF:FF:FF:FF:FF:FF   ARP Request ¿10.1.10.4?          ← pregunta por el ROUTER, no por 10.2.10.3
MAC4 → MAC1               ARP Reply 10.1.10.4 ⇔ MAC4
MAC1 → MAC4  | 10.1.10.1 → 10.2.10.3 | ICMP Echo Request    ← IP destino = destino final; MAC destino = router
--- Segmento B ---
MAC5 → FF:FF:FF:FF:FF:FF   ARP Request ¿10.2.10.3?
MAC6 → MAC5               ARP Reply 10.2.10.3 ⇔ MAC6
MAC5 → MAC6  | 10.1.10.1 → 10.2.10.3 | ICMP Echo Request    ← IPs SIN CAMBIAR, MACs cambian
MAC6 → MAC5  | 10.2.10.3 → 10.1.10.1 | ICMP Echo Reply      ← sin ARP (aprendió MAC5 del Request)
--- Segmento A ---
MAC4 → MAC1  | 10.2.10.3 → 10.1.10.1 | ICMP Echo Reply
```

> 📌 **LA IDEA MÁS IMPORTANTE DE LA P1:** las **direcciones IP origen y destino NO cambian** a lo largo del camino (el datagrama debe llegar al destino y conservar el remitente). Las **direcciones MAC cambian en cada salto**: sirven para llegar al **siguiente salto** dentro de cada segmento.

---

## 8. Formato del datagrama IPv4

```
 0        4        8               16    19                         31
+--------+--------+---------------+------------------------------------+
| Ver(4) | HL(4)  |    TOS (8)    |        Longitud Total (16)         |
+--------+--------+---------------+------+-----------------------------+
|        Identificación (16)              | Flag(3)| Fragment Offset(13)|
+-----------------+----------------------+------------------------------+
|     TTL (8)     |    Protocolo (8)     |  Suma de control cabecera(16)|
+-----------------+----------------------+------------------------------+
|                     Dirección IP Fuente (32)                         |
+----------------------------------------------------------------------+
|                     Dirección IP Destino (32)                        |
+----------------------------------------------------------------------+
|              Opciones (si existen) — múltiplo de 32 bits              |
+----------------------------------------------------------------------+
|                               Datos                                  |
```
Bit más significativo = 0 (izquierda), menos significativo = 31. Se transmite del 0 al 31.

| Campo | Bits | Lo que tienes que saber 📌 |
|---|---|---|
| **Ver** | 4 | Versión (4 para IPv4) |
| **HL** (Header Length) | 4 | Nº de **palabras de 32 bits** de la cabecera. Normalmente **5** (= 20 bytes, sin opciones). Máx: (2^4 − 1)·4 B = 15·4 = **60 bytes** |
| **TOS** | 8 | Tipo de servicio (Calidad de Servicio, QoS). **No es objeto de la asignatura** |
| **Longitud Total** | 16 | Tamaño en **bytes** de todo el datagrama (**cabecera + datos**). Máx **65535** bytes. Con HL se sabe dónde empiezan/terminan los datos |
| **Identificación** | 16 | Marca **cada datagrama** de una máquina; normalmente se **incrementa** con cada envío. En fragmentación se **copia** a todos los fragmentos |
| **Flag** | 3 | Bit 0: reservado · **Bit 1: DF (Don't Fragment)** · **Bit 2: MF (More Fragments)** |
| **Fragment Offset** | 13 | Posición del fragmento en el datagrama original, **en unidades de 8 bytes** |
| **TTL** | 8 | Tiempo de vida. Lo pone el emisor y **cada router lo decrementa en 1**. Si llega a **0** → se descarta y se envía **ICMP** al origen. Evita bucles. Máx 255 routers |
| **Protocolo** | 8 | Qué lleva dentro: **1 = ICMP**, **6 = TCP**, **17 = UDP** (➕ valores) |
| **Suma de control** | 16 | **Solo la cabecera IP**. Suma sin acarreo de palabras de 16 bits. Como el router normalmente solo cambia el TTL, se puede recalcular **de forma incremental** (rápido) |
| **IP fuente / destino** | 32 + 32 | Remitente y destinatario |
| **Opciones** | variable | Rara vez se usan en Internet |

- **DF = 1**: IP **no puede fragmentar**; si el paquete es demasiado grande, se **elimina** y se envía **ICMP error** al origen (3/4, ver P2).
- **MF = 1** en **todos los fragmentos menos el último**.

---

## 9. ICMP: generalidades

### 9.1 Qué es 📌
- **ICMP (Internet Control Message Protocol)**: informa del **estado y de situaciones de error** de la **capa de red**: encaminamiento, congestión, fragmentación...
- Se apoya en **IP**: los mensajes ICMP viajan **dentro de datagramas IP** (Protocolo = 1). RFC **792**.

```
|<------------------------------- datagrama IP ------------------------------->|
| Cabecera IP (20 B) |              Mensaje ICMP                               |
                     | tipo(8b) | código(8b) | SVT(16b) | inf. variable(32b) | DATOS |
                     |<----------------- cabecera ICMP: 8 bytes ---------->|
```

| Campo ICMP | Tamaño | Función |
|---|---|---|
| **Tipo** | 8 bits | Clase de mensaje |
| **Código** | 8 bits | Concreta el tipo (motivo) |
| **SVT** (secuencia de verificación de trama) | 16 bits | **Suma de control de TODO el mensaje ICMP** ⚠️ (la de IP solo cubre la cabecera) |
| **Información variable** | 32 bits | Depende del tipo (➕ en Echo: Identificador + Nº de secuencia; en Redirect: IP del nuevo router; en 3/4: MTU del siguiente salto) |

### 9.2 Tipos ICMP 📌 (memorízalos)
| Tipo | Mensaje |
|---|---|
| **0** | **Echo Reply** |
| **3** | **Destino inaccesible** (Destination Unreachable) |
| **4** | **Source Quench** (cadencia de envío demasiado elevada) |
| **5** | **Redirección** (Redirect) |
| **8** | **Echo Request** |
| 9 | Aviso de Router |
| 10 | Solicitud de Router |
| **11** | **Tiempo sobrepasado** (Time Exceeded, TTL) |
| 12 | Problema de parámetros o de forma |
| 13 | Petición Timestamp |
| 14 | Respuesta Timestamp |

Truco: **8 = pregunta, 0 = respuesta** ("8 va, 0 vuelve").

### 9.3 Mensajes de solicitud vs mensajes de error 📌
- **Solicitud / información**: piden o informan de algo que **no es un error** (Echo, Echo Reply, aviso de router...). Los datos llevan info de la solicitud.
- **Error**: informan de errores al enviar paquetes IP (TTL = 0, máquina inaccesible, fragmentación necesaria con DF...).
  - 📌 Un mensaje de error ICMP lleva en sus datos **la cabecera IP + los 8 primeros bytes de datos del paquete que lo provocó** → así se sabe qué paquete causó el error.
  - 📌 **Un error ICMP nunca se genera como consecuencia de otro error ICMP** (evita cascadas infinitas).

### 9.4 Situaciones que **NO** generan error ICMP 📌📌
1. Un **mensaje de error ICMP** (pero sí puede generarse error en respuesta a una **solicitud** ICMP, p.ej. a un Echo Request).
2. Un paquete IP destinado a una **dirección IP de broadcast**.
3. Un paquete IP enviado como **broadcast de la capa de enlace** (MAC FF:FF:...).
4. Un **fragmento que no sea el primero** (excepto el *Fragment Reassembly Time Exceeded*, que se genera cuando falta algún fragmento).
5. Un **fragmento recibido fuera de secuencia**.
6. Un paquete cuya **IP origen no identifica a una única máquina**: origen **0**, **broadcast** o **de grupo** (multicast).

**Motivo**: prevenir las **tormentas de broadcast (broadcast storms)**.

---

## 10. Echo Request / Echo Reply y ping

- **ping** usa **Echo Request (tipo 8)** y espera **Echo Reply (tipo 0)**. Mide tiempos de **ida y vuelta** (RTT), paquetes perdidos, mín/máx/medio → tasa de error y velocidad real.
- ⚠️ **ping NO usa TCP ni UDP**: va directamente sobre **IP** (ICMP). Que responda significa que la máquina está **activa a nivel IP**, pero **no garantiza** que su TCP o UDP funcionen.
- Formato: `| Cab. IP 20 B | ICMP 8 B | datos (tamaño según -l) |`

### Opciones de ping (Windows) 📌
| Opción | Significado |
|---|---|
| `-n x` | Envía x Echo Request (por defecto **4**) |
| `-l y` | **y bytes de datos** en el Echo Request (➕ por defecto 32) |
| `-i z` | **TTL = z** |
| `-f` | Activa el bit **Don't Fragment** |

### Cálculo de tamaños 📌
Con `ping -l D`:
- Mensaje ICMP = **D + 8**
- Datagrama IP (Longitud Total) = **D + 8 + 20 = D + 28**
- Trama Ethernet en Wireshark = **D + 28 + 14 = D + 42** (sin CRC; con CRC +4)

| Comando | ICMP | IP | Trama (Wireshark) |
|---|---|---|---|
| ping (por defecto, 32) | 40 | 60 | 74 |
| ping -l 200 | 208 | 228 | 242 |
| ping -l 500 | 508 | 528 | 542 |
| ping -l 1000 | 1008 | 1028 | 1042 |
| ping -l 1472 | 1480 | **1500** (máx sin fragmentar en Ethernet) | 1514 |
| ping -l 1473 o más | — | se fragmenta | — |

Ejemplo del PDF: `ping -l 200 172.20.43.241` → **4 paquetes IP** hacia R3, cada uno con 200 B de datos + 20 (IP) + 8 (ICMP).

➕ El relleno de datos que mete el ping de Windows es `abcdefghijklmnopqrstuvwabcdefghi...` → por eso en la P1 se buscan paquetes con la cadena "abcd".

---

## 11. Fragmentación IP

### 11.1 Ideas clave 📌
- **Ethernet no fragmenta** → lo hace **IP** (nivel de red).
- IP necesita el **MTU** del enlace: datos máximos por paquete IP = **MTU − 20** → en Ethernet **1480 bytes**.
- Los 3 campos implicados: **Identificación** (igual en todos los fragmentos), **MF** (1 en todos menos el último), **Fragment Offset** (posición en el original).
- Los paquetes pueden **llegar desordenados** → el offset dice cómo reordenarlos.
- 📌⚠️ **El Fragment Offset se expresa en grupos de 8 bytes**: 1480 bytes → campo = 1480/8 = **185** (0x0B9). → Por eso **los datos de cada fragmento (salvo el último) deben ser múltiplo de 8**.
- ➕ El **reensamblado solo lo hace el destino final**, nunca los routers intermedios.

### 11.2 Fórmulas
- Datos máximos por fragmento = **⌊(MTU − 20) / 8⌋ · 8**
- Nº de fragmentos = ⌈ (D + 8) / datos_por_fragmento ⌉
- Offset (bytes) del fragmento k (k = 0,1,2…) = k · datos_por_fragmento; campo = offset/8.

### 11.3 Ejemplo del PDF: `ping -n 1 -l 2000 172.20.43.241` (Ethernet, MTU 1500)
- ICMP = 8 + 2000 = **2008** B. IP sin fragmentar sería 2028 > 1500 → fragmentar en trozos de 1480.

| Fragmento | Cab. IP | Contenido | Datos IP | Long. Total | MF | Offset (bytes) | Campo FO | Trama |
|---|---|---|---|---|---|---|---|---|
| 1º | 20 | cab. ICMP 8 + 1472 datos | 1480 | **1500** | **1** | 0 | 0 | 1514 |
| 2º | 20 | 528 datos (sin cab. ICMP) | 528 | **548** | **0** | 1480 | **185** | 562 |

Ambos con la **misma Identificación** (ej. del PDF: 12154). ⚠️ **Solo el primer fragmento lleva la cabecera ICMP.**

### 11.4 Ejemplo: `ping -n 1 -l 4500 172.20.43.242` (Cuestión 2 P2)
ICMP = 4508 B → 4508 = 1480 + 1480 + 1480 + 68.

| Frag. | Datos IP | Long. Total | Offset (B) | Campo FO | DF | MF |
|---|---|---|---|---|---|---|
| 1 | 1480 | 1500 | 0 | 0 | 0 | 1 |
| 2 | 1480 | 1500 | 1480 | 185 | 0 | 1 |
| 3 | 1480 | 1500 | 2960 | 370 | 0 | 1 |
| 4 | 68 | 88 | 4440 | 555 | 0 | 0 |

→ **4 paquetes IP**, misma Identificación.

### 11.5 Ejemplo: `ping -l 1600` (P1, captura 4.1)
ICMP = 1608 → 1480 + 128 → IP de **1500** (MF=1, off 0) y **148** (MF=0, off 1480 → 185). Por defecto envía 4 pings → 8 fragmentos de ida + 8 de vuelta.

### 11.6 Ejemplo con un MTU que no deja múltiplo de 8 (➕ clave para la Cuestión 5 P2)
Supón un enlace con MTU = **576** y un Echo con `-l 1000` (ICMP = 1008):
- MTU − 20 = 556 → **no es múltiplo de 8** → datos por fragmento = ⌊556/8⌋·8 = **552**.
- Fragmentos: 552 + 456 → paquetes IP de **572** y **476** bytes; offsets 0 y 552 (campo **69**).
- ⚠️ El paquete más grande (572) **no coincide con el MTU (576)**. Esa es exactamente la "razón de que no coincidan" que pregunta la práctica: **el Fragment Offset solo puede expresar múltiplos de 8 bytes**.
(576 es un valor de ejemplo: en el examen usa el MTU que te den o el que viste en tu captura.)

### 11.7 Por qué la fragmentación es mala 📌
- Enviar un paquete grande o varios trozos apenas cambia el **tiempo de transmisión** (solo unas cabeceras más), pero **sí el tiempo de encaminamiento**: el router gasta **el mismo tiempo de CPU por paquete independientemente de su tamaño**.
- Antes (redes lentas, ~100 KBps, muchos errores) fragmentar compensaba: si un trozo fallaba, se reenviaba solo ese trozo.
- Hoy (cientos de Mbps, pocos errores) no aporta ventaja y provoca **congestión** en los routers.

**Ejemplo numérico del PDF** (2 redes de 1 Mbps unidas por un router, Tenc = 1 µs/paquete, 1000 bits, sin cabeceras):
- **1 paquete de 1000 bits**: T = Ttr1 + Tenc + Ttr2 = 1000/10^6 + 0,001 ms + 1 ms = 1 + 0,001 + 1 = **2,001 ms**
- **100 paquetes de 10 bits**: T = 100·0,01 + 100·0,001 + 100·0,01 = 1 + 0,1 + 1 = **2,1 ms**

(Ttr de 10 bits a 1 Mbps = 10/10^6 s = 0,01 ms.) → Más fragmentos = más retardo.

- **IPv6 no permite fragmentar en los routers.**
- Referencia: *Fragmentation Considered Harmful* (Kent y Mogul, SIGCOMM '87).

---

## 12. Destination Unreachable (tipo 3)

Se genera cuando un paquete IP **no consigue alcanzar su destino**. Códigos que entran 📌:

| Tipo/Código | Nombre | Cuándo | Quién lo envía |
|---|---|---|---|
| **3/1** | **Host Unreachable** | El router que debe entregar el paquete **no obtiene por ARP la MAC del destino** (máquina apagada/inexistente) | El **último router** (el conectado a la red del destino). ⚠️ Muchos routers lo **deshabilitan** para evitar que un ciberdelincuente provoque congestión |
| **3/3** | **Port Unreachable** | Llega al destino un paquete de transporte a un **puerto sin proceso** que lo atienda | El **host destino** |
| **3/4** | **Fragmentation Needed and Don't Fragment was Set** | Un router necesita fragmentar pero el paquete tiene **DF = 1** | El **router** que no puede pasar el paquete por el siguiente enlace. ➕ Incluye en la cabecera ICMP el **MTU del siguiente salto** |
| **3/13** | **Communication Administratively Prohibited** | El paquete no puede encaminarse por **reglas de filtrado** del router | El **router que filtra** |

---

## 13. Tabla de rutas, comando route y Redirect (tipo 5)

### 13.1 Tabla de rutas 📌
Cualquier equipo TCP/IP (PC o router) tiene una tabla que dice **cómo enviar los paquetes por sus interfaces**. Se ve con `netstat -rn` (➕ o `route print`).

```
Dirección de red   Máscara de red     Puerta de enlace   Interfaz
172.20.43.192      255.255.255.192    En vínculo         172.20.43.x
0.0.0.0            0.0.0.0            172.20.43.240      172.20.43.x
```

- **172.20.43.192 / En vínculo**: mi propia red. "En vínculo" = entrega **directa**: ARP busca la MAC **del destinatario**.
- **0.0.0.0 / 0.0.0.0**: **ruta por defecto**, coincide con cualquier destino. Es la **última que se comprueba**. Indica la **puerta de enlace por defecto** (172.20.43.240 = R2): ARP busca la MAC **de la puerta de enlace**.
- ➕ Si varias rutas coinciden, gana la más **específica** (máscara más larga): /32 > /24 > /0.

### 13.2 Comando route 📌
```
route ADD  destino  MASK  máscara  puerta_de_enlace
route DELETE destino
```
Ejemplos del PDF:
```
route ADD 10.3.0.0 MASK 255.255.255.0 172.20.43.241     ← toda la red 10.3.0.0/24 por R3
route ADD 10.6.0.2 MASK 255.255.255.255 172.20.43.242   ← solo el host 10.6.0.2 por R5 (máscara /32)
route DELETE 10.3.0.0
route DELETE 10.6.0.2
```
- Tras añadir la ruta por .241, un ping a 10.3.0.1 hace que **ARP busque la MAC de 172.20.43.241**.
- ⚠️ Las rutas añadidas **permanecen** hasta que las borres.
- La tabla se modifica con **mensajes ICMP Redirect** o con **route**.

### 13.3 Mensaje Redirect (tipo 5) 📌📌
Lo envía un **router** al **emisor** de un paquete cuando ese paquete **debería haberse enviado a otro router**.

```
          PC1
     (1)↓   ↑(3) Redirect ICMP
 ───────────────────────────────────── (misma red)
   Router 1  ──(2) datagrama──►  Router 2 ──► destino final
```
1. PC1 envía un paquete a un destino de fuera → lo manda a **Router 1** (su puerta de enlace).
2. Router 1 consulta su tabla: el siguiente salto es **Router 2**, y la **interfaz de salida es la misma por la que entró** el paquete.
3. Router 1 envía un **ICMP Redirect** a PC1 diciéndole que mande los próximos paquetes a ese destino **directamente a Router 2**. **Y además reenvía el paquete original a Router 2** (el paquete no se pierde).

📌 **Condición**: solo se envía si la **IP origen** del paquete (PC1) pertenece a la **misma red IP que la interfaz del router** que lo reenvía. En redes intermedias (donde no está la máquina origen) **no se genera**.

➕ Datos que transporta el Redirect: en el campo de información variable la **IP del router al que hay que ir** (gateway), y en los datos la **cabecera IP + 8 primeros bytes** del paquete original (es un mensaje de error). Códigos ➕: 0 = para red, **1 = para host**, 2 y 3 = con TOS.

⚠️ Windows 10 guarda la información del Redirect con un **temporizador**, **no en la tabla de rutas** (si vuelve a salir el Redirect es que ha expirado).

---

## 14. Time Exceeded (tipo 11) y tracert

### 14.1 Código 0: Time to Live exceeded in Transit (11/0) 📌
- Cada router que recibe un paquete **no dirigido a él** **decrementa el TTL en 1**.
  - Si el resultado ≠ 0 → lo encamina.
  - Si el resultado = 0 → **lo elimina** y envía **11/0** al **remitente**.
- TTL es de 8 bits → **máximo 255 routers**. Evita paquetes circulando indefinidamente (bucles).

### 14.2 tracert 📌📌
- Envía **Echo Request con TTL = 1** → el 1er router lo descarta y envía 11/0 → tracert muestra ese router como **salto 1**.
- Luego TTL = 2 → responde el 2º router... y así hasta recibir un **Echo Reply del destino**.
- Envía **3 paquetes por salto** (por eso salen 3 tiempos: para estadística del tiempo medio). Máximo **30 saltos**.
- `-d`: **no** resuelve nombres (evita consultas DNS).
- ➕ tracert de Windows usa ICMP Echo; traceroute de Linux usa UDP por defecto.

Ejemplo del PDF:
```
tracert -d 10.3.0.2
  1     1 ms     1 ms     1 ms  172.20.43.240   ← R2
  2    28 ms    19 ms    20 ms  10.4.0.2        ← R1
  3    80 ms    45 ms    58 ms  10.3.0.2        ← R4 (destino)
```
→ Ruta: **R2 → R1 → R4** (cruza el PaP de 10 Mbps: por eso sube tanto el tiempo).

➕ La IP que aparece en cada salto es la de la **interfaz por la que el router envía el ICMP** (normalmente la que mira hacia ti): R1 aparece como 10.4.0.2, no como 10.3.0.1.

### 14.3 Código 1: Fragment Reassembly Time Exceeded (11/1) 📌
- Lo envía la **máquina destino** cuando, al **reensamblar** un paquete fragmentado, **algún fragmento no llega a tiempo**.
- Útil para detectar **congestión debida a la fragmentación**: con congestión, los paquetes independientes solo se retrasan, pero los fragmentos con mucho retardo **no se pueden reensamblar** y el usuario cree que **no hay comunicación**. Viendo 11/1 se distingue **congestión** de "la info realmente no llega".

---

## 15. Source Quench (tipo 4)

- Router = procesador de paquetes con **colas de entrada y de salida**.
- Recoge en la cola de entrada → decide el encaminamiento → mete en la cola de salida.
- Si no procesa tan rápido como le llegan, la cola de entrada crece; si **se llena**, **elimina** los paquetes en exceso y envía **ICMP Source Quench (tipo 4)** al **origen** para que **reduzca el flujo**.
- 📌 Es **el único mecanismo de control de congestión de IP**.

---

## 16. Evaluación experimental del rendimiento

- Parámetros: velocidad de transmisión, tasa de error, **retardo**...
- Se mide el **retardo (RTT)** con ping cada **15 s durante 15 min → n = 60 muestras**, hacia **10.3.0.2** y **10.4.0.2**.
- Tamaño: el mayor que no se fragmente. Hay varios MTU en el laboratorio → se elige **MTU = 500 bytes**.
  - 📌 `longitud (-l) = MTU − cabIP − cabICMP = 500 − 20 − 8 = 472` → `ping -n 1 -l 472 dirección_ip`
  - Trama Ethernet = 500 + 14 = 514 bytes.

### Fórmulas 📌
```
Retardo medio:     R̄ = (Σ retardo_i) / n

Desviación típica: S = √[ Σ (R̄ − retardo_i)² / (n − 1) ]

En porcentaje:     S(%) = S / R̄ · 100
```
- **S(%) > 50 → rendimiento bajo** (la carga no está distribuida de forma homogénea, hay **picos de utilización**).
- Cuanto mayor la desviación, peor el rendimiento.

### Ejemplo resuelto
Retardos (ms): 10, 12, 8, 30, 10 (n = 5)
- R̄ = 70/5 = **14 ms**
- (14−10)² = 16; (14−12)² = 4; (14−8)² = 36; (14−30)² = 256; (14−10)² = 16 → Σ = **328**
- S = √(328/4) = √82 = **9,06 ms**
- S(%) = 9,06/14 · 100 = **64,7 %** → > 50 % → **rendimiento bajo**.

➕ Se espera más retardo hacia **10.3.0.2** (más saltos y atraviesa el **PaP de 10 Mbps**) que hacia 10.4.0.2.

---

## 17. Comandos y filtros de Wireshark

### 17.1 Comandos de Windows (cmd como administrador)
| Comando | Para qué |
|---|---|
| `ipconfig /all` | Ver MAC, IP, máscara, puerta de enlace, DNS (el único interfaz físico real es el Ethernet) |
| `ipconfig /release` / `/renew` | Liberar / renovar la configuración (DHCP) |
| `arp -a` | Ver la caché ARP |
| `arp -d IP` | Borrar una entrada ARP |
| `netstat -rn` | Ver la tabla de rutas |
| `route ADD dest MASK masc gw` / `route DELETE dest` | Añadir / borrar rutas |
| `ping -n -l -i -f IP` | Ver sección 10 |
| `tracert -d IP` | Ruta hasta el destino sin DNS |
| `c:\pracredes` | Script del laboratorio que configura la ruta por defecto (0.0.0.0 → 172.20.43.240) |

Restaurar la configuración inicial: `route delete 0.0.0.0` → `ipconfig /release` → `ipconfig /renew`.

### 17.2 Filtros de visualización de Wireshark (➕ sintaxis para responder las cuestiones 4.1)
| Qué quieres ver | Filtro |
|---|---|
| Tramas de difusión (broadcast) | `eth.dst == ff:ff:ff:ff:ff:ff` |
| Paquetes ARP | `arp` |
| Paquetes IP | `ip` |
| Paquetes NO IP | `!ip` (o `not ip`) |
| Enviados por mi equipo | `ip.src == MI_IP` |
| Recibidos por mi equipo | `ip.dst == MI_IP` |
| Intercambiados con mi equipo | `ip.addr == MI_IP` |
| Que contengan "abcd" | `frame contains "abcd"` (o `icmp contains "abcd"`) |
| Protocolo IP = 17 (UDP) | `ip.proto == 17` |
| TTL = 1 / TTL = 2 | `ip.ttl == 1` / `ip.ttl == 2` |
| Bit MF activo | `ip.flags.mf == 1` |
| Cualquier fragmento (incluido el último) | `ip.flags.mf == 1 || ip.frag_offset > 0` |
| Bit DF activo | `ip.flags.df == 1` |
| Solo ICMP | `icmp` |
| Echo Request / Reply / Time Exceeded | `icmp.type == 8` / `icmp.type == 0` / `icmp.type == 11` |

Porcentaje = paquetes "Mostrados" (Displayed) / paquetes totales (Captured) × 100 → la barra de estado de Wireshark lo muestra.

📌 Para ver la fragmentación hay que **desactivar** en `Edit → Preferences → Protocols → IPv4` la opción **"Reassemble fragmented IPv4 datagrams"**.

---

## 18. Cuestiones de la Práctica 1 resueltas

> Lo que depende de tu captura concreta (porcentajes, MACs) lo explico como **método + resultado esperado**.

### 4.1 Análisis de la captura (ping 172.20.43.240, ping -l 1600 172.20.43.240, tracert -d 172.18.41.4)

**C1. % de tramas de difusión** → `eth.dst == ff:ff:ff:ff:ff:ff`. Serán sobre todo **ARP Request** y algún broadcast de Windows (NetBIOS, DHCP...). % = difusión / total · 100.

**C2. % de paquetes IP** → `ip`.

**C3. % de paquetes no IP** → `!ip` (ARP, y otros como LLDP/STP si los hay). ⚠️ C2 + C3 = 100 %.

**C4/C5. % enviados / recibidos por mi equipo** → `ip.src == MI_IP` / `ip.dst == MI_IP` (la IP con `ipconfig`).

**C6. Paquetes IP con "abcd" intercambiados con mi IP** → `ip.addr == MI_IP && frame contains "abcd"`.

**C7. ¿Qué protocolo/aplicación?** → Son **ICMP Echo Request y Echo Reply** generados por **ping**: el ping de Windows rellena los datos con `abcdefghijklmnopqrstuvwabcdefghi...`. (El Echo Reply devuelve los mismos datos, por eso también aparecen.)

**C8. Protocol = 17** → `ip.proto == 17` → son **UDP** (DNS, NetBIOS, SSDP, etc.).

**C9. TTL = 1** → `ip.ttl == 1` → los **3 Echo Request del primer salto del tracert** (➕ y quizá algún multicast local como IGMP/LLMNR, que se envía con TTL 1).

**C10. TTL = 2** → `ip.ttl == 2` → los **3 Echo Request del segundo salto del tracert**. El tracert a 172.18.41.4 tiene **2 saltos**: R2 (172.20.43.240) responde 11/0 a los de TTL 1, y **R1** (172.18.41.4 es interfaz suya) responde Echo Reply a los de TTL 2.

**C11. % fragmentados (MF activo)** → `ip.flags.mf == 1`. Vienen del `ping -l 1600` (1608 B de ICMP > 1480): 4 Echo Request → 4 fragmentos con MF=1 + 4 Echo Reply → 4 con MF=1 = **8 paquetes con MF=1**. (Los últimos fragmentos tienen MF=0 aunque también son fragmentos: para contarlos usa `ip.frag_offset > 0`.)

### 4.2 ARP (red 172.20.43.192/26)

**C1. ping -n 1 172.20.43.241 (R3, mismo segmento):**
```
MAC_PC → FF:FF:FF:FF:FF:FF   ARP Request ¿172.20.43.241?
MAC_R3 → MAC_PC              ARP Reply 172.20.43.241 ⇔ MAC_R3
MAC_PC → MAC_R3  | PC → 172.20.43.241 | Echo Request
MAC_R3 → MAC_PC  | 172.20.43.241 → PC | Echo Reply
```
`arp -a` muestra la nueva entrada; un segundo ping **no genera ARP** (está en caché). Para que vuelva a salir: esperar a que expire o `arp -d 172.20.43.241`.

**C2.1 ping al compañero:**
- **Una sola secuencia ARP** (Request de mi PC + Reply del compañero).
- Mi tabla gana la entrada del compañero; **la del compañero también gana la mía** (la aprende del ARP Request, que lleva mi IP/MAC).
- Repitiendo el ping: **no hay ARP**. Si borro la entrada con `arp -d`, vuelve a haber ARP.

**C2.2 El compañero borra MI entrada; yo vuelvo a hacer ping:**
- **Sí** se capturan paquetes ARP, pero **después del Echo Request**:
```
MAC_yo   → MAC_comp           | Echo Request       ← yo tengo su MAC en caché: no pregunto
MAC_comp → FF:FF:FF:FF:FF:FF  ARP Request ¿MI_IP?   ← él no sabe mi MAC para responder
MAC_yo   → MAC_comp           ARP Reply MI_IP ⇔ MAC_yo
MAC_comp → MAC_yo             | Echo Reply
```
- **Por qué**: yo ya conozco su MAC, así que envío directamente. Él necesita **mi MAC para enviar el Echo Reply** y la había borrado, así que pregunta **antes de responder**. (La caché ARP se rellena con mensajes ARP, no con los Echo recibidos.)

**C3. arp -d 172.20.43.240 ; ping -n 1 10.4.0.2:**
- 10.4.0.2 **no está en mi red** (172.20.43.192/26) → se envía a la puerta de enlace → el ARP Request pregunta por **172.20.43.240** (R2), **no** por 10.4.0.2.
- Echo Request: MAC origen = mi PC, **MAC destino = R2**; IP origen = mi IP, **IP destino = 10.4.0.2**.
- Solo veo el ARP de mi segmento (el que haga R2 en 10.4.0.0 no lo veo).

**C4. ¿De qué router (R2, R3, R5) viene el Echo Reply?** Método: mira la **MAC origen** del Echo Reply y búscala en `arp -a` (172.20.43.240 = R2, .241 = R3, .242 = R5). Los Echo Request **siempre salen por R2** (ruta por defecto); la **vuelta** la decide la tabla de rutas de cada router → el encaminamiento puede ser **asimétrico**. Razonamiento:

| Destino | Quién es | Vuelta esperada |
|---|---|---|
| 10.4.0.1 | Interfaz de **R2** | Contesta R2 directamente → **R2** (seguro) |
| 10.4.0.3 | Interfaz de **R3** | R3 está conectado directamente a 172.20.43.192/26 → responde por su interfaz .241 → **R3** (seguro) |
| 10.6.0.3 | Interfaz de **R5** | R5 está conectado a nuestra red → **R5** (seguro) |
| 10.4.0.2 | R1 | Depende de la ruta de R1 hacia 172.20.43.192/26 (vía R2 o R3) |
| 10.3.0.2 | R4 | Depende de R4 (vía R1→R2/R3 o vía 10.6.0.0→R3/R5) |
| 10.6.0.5 / 10.6.0.6 | Linux2 / Linux3 | Depende de la **puerta de enlace por defecto** de cada Linux (R3, R4 o R5) → pueden volver por routers distintos |

(Las tres primeras son deducibles por topología; las demás tienes que verlas en la captura. Un router que recibe un ping a una de sus interfaces responde él mismo y, si tiene interfaz en tu red, te lo entrega directamente.)

**C5. ping -n 1 172.20.43.250:**
- 172.20.43.250 está **dentro de mi red** (.193–.254) → ARP pregunta directamente por .250.
- Se ven **varios ARP Request sin respuesta** y **ningún Echo Request** (no se puede construir la trama sin MAC destino) y por tanto ningún Echo Reply. Windows muestra algo como "Host de destino inaccesible" (lo genera **tu propio PC**).
- Conclusión: el equipo **172.20.43.250 no existe o está apagado** (no responde a ARP).

**C6. ping -n 1 10.6.0.25:**
- ¿ARP? Solo si la MAC de la puerta de enlace (R2) **no está en caché**; si está, **no aparecen**. Nunca preguntaré por 10.6.0.25 (no está en mi red).
- ¿Echo Reply? **No**: 10.6.0.25 no existe en la topología. El router conectado a 10.6.0.0/24 hará ARP en ese segmento sin respuesta (no lo vemos). Podría llegar un **ICMP 3/1 Host Unreachable** de ese router, pero muchos routers lo deshabilitan → normalmente "Tiempo de espera agotado".

**C7. Broadcast de 172.20.43.192/26 = 172.20.43.255** (`ping 172.20.43.255`):
- **MAC destino = FF:FF:FF:FF:FF:FF** (no hay ARP: la IP de broadcast se mapea directamente al broadcast MAC).
- Lo reciben todos los equipos del segmento; contestan los que están configurados para responder a Echo dirigidos a broadcast (normalmente los **routers** R2, R3, R5; los PCs Windows no suelen contestar). Responder a pings de broadcast es configurable (➕ para evitar ataques de amplificación).

### 4.3 Direccionamiento IP

**a) 130.130.0.0/16 en 7 subredes** → 2^3 = 8 ≥ 7 → **ampliar 3 bits** → **/19** → máscara **255.255.224.0**. Salto = 32 en el 3er octeto. Cada subred: 2^13 − 2 = **8190** equipos.

| Subred | Rango de equipos | Broadcast |
|---|---|---|
| 130.130.0.0/19 | 130.130.0.1 – 130.130.31.254 | 130.130.31.255 |
| 130.130.32.0/19 | 130.130.32.1 – 130.130.63.254 | 130.130.63.255 |
| 130.130.64.0/19 | 130.130.64.1 – 130.130.95.254 | 130.130.95.255 |
| 130.130.96.0/19 | 130.130.96.1 – 130.130.127.254 | 130.130.127.255 |
| 130.130.128.0/19 | 130.130.128.1 – 130.130.159.254 | 130.130.159.255 |
| 130.130.160.0/19 | 130.130.160.1 – 130.130.191.254 | 130.130.191.255 |
| 130.130.192.0/19 | 130.130.192.1 – 130.130.223.254 | 130.130.223.255 |
| 130.130.224.0/19 | 130.130.224.1 – 130.130.255.254 | 130.130.255.255 |

(Se necesitan 7; la 8ª sobra.)

**b) 150.22.0.0/23 en 5 subredes** → 2^3 = 8 ≥ 5 → **ampliar 3 bits** → **/26** → máscara **255.255.255.192**. La red /23 abarca 150.22.0.0 – 150.22.1.255. Salto 64. Cada subred: 2^6 − 2 = **62** equipos.

| Subred | Rango de equipos | Broadcast |
|---|---|---|
| 150.22.0.0/26 | 150.22.0.1 – 150.22.0.62 | 150.22.0.63 |
| 150.22.0.64/26 | 150.22.0.65 – 150.22.0.126 | 150.22.0.127 |
| 150.22.0.128/26 | 150.22.0.129 – 150.22.0.190 | 150.22.0.191 |
| 150.22.0.192/26 | 150.22.0.193 – 150.22.0.254 | 150.22.0.255 |
| 150.22.1.0/26 | 150.22.1.1 – 150.22.1.62 | 150.22.1.63 |
| 150.22.1.64/26 | 150.22.1.65 – 150.22.1.126 | 150.22.1.127 |
| 150.22.1.128/26 | 150.22.1.129 – 150.22.1.190 | 150.22.1.191 |
| 150.22.1.192/26 | 150.22.1.193 – 150.22.1.254 | 150.22.1.255 |

⚠️ Trampa: 150.22.0.255 y 150.22.1.0 **sí** son direcciones válidas en la red /23 original (pero en las subredes /26, .0.255 es un broadcast y .1.0 una dirección de subred).

---

## 19. Cuestiones de la Práctica 2 resueltas

**C1. ping -n 1 -l 500 172.20.43.241**
- a) Tipos ICMP: **Echo Request (8/0)** y **Echo Reply (0/0)** (más ARP si R3 no estaba en caché).
- b) Request: MAC origen = mi PC, MAC destino = **R3** (172.20.43.241 está en mi red: entrega directa); IP origen = mi IP, IP destino = 172.20.43.241. Reply: al revés.
- c) Tamaños: datos ICMP 500 + cab. ICMP 8 = 508; + cab. IP 20 → **Longitud Total 528**; + cab. Ethernet 14 → **trama de 542 bytes**. No hay fragmentación (528 < 1500).

**C2. Fragmentación**
- `ping -n 1 -l 2000 172.20.43.241` → 2 fragmentos: IP de 1500 (MF=1, off 0) y 548 (MF=0, off 1480 → campo 185), misma ID (ver 11.3). **MTU de mi PC = 1500** y **MTU de 172.20.43.241 = 1500** (sus fragmentos de respuesta también son de 1500): ambos en Ethernet.
- `ping -n 1 -l 1000 10.3.0.2` → mi PC **no fragmenta**: 1 paquete IP de **1028 B** (MF=0, offset 0). Pero el camino atraviesa el **PaP 10.3.0.0/24**, de MTU menor: **R1 fragmenta** la petición para el PaP. La respuesta de R4 (10.3.0.2) **sale ya fragmentada** por el PaP y **ningún router reensambla**, así que **a mi PC llegan varios fragmentos**: el tamaño del mayor revela el **MTU del destino** (ajustado a múltiplo de 8 + 20).
- `ping -n 1 -l 4500 172.20.43.242` → **4 paquetes IP**: Long. Total 1500, 1500, 1500, 88; offsets 0/1480/2960/4440 bytes (campo 0/185/370/555); **DF = 0** en todos; **MF = 1,1,1,0**. Coincide con lo capturado (ver 11.4).

**C3. Rutas estáticas**
- `route add 10.6.0.3 mask 255.255.255.255 172.20.43.242` + `ping -n 1 10.6.0.3`:
  - a) El Echo Request va a la **MAC de R5** (.242) → **sí**, coincide con la ruta añadida (una ruta /32 es más específica que la por defecto).
  - b) El Echo Reply llega **por R5**: 10.6.0.3 es interfaz de R5, que tiene interfaz en nuestra red y responde directamente.
  - Borrar: `route delete 10.6.0.3`.
- `route add 10.6.0.0 mask 255.255.255.0 172.20.43.241` + `ping -n 1 10.6.0.5`:
  - b) El Echo Request va a la **MAC de R3** → coincide.
  - c) El Reply llega por el router que sea **puerta de enlace de Linux2** (míralo con la MAC origen).
- `ping -n 1 10.6.0.6`: la ida **también va por R3** (la ruta cubre toda la 10.6.0.0/24), pero la **vuelta puede llegar por otro router** porque **Linux3 puede tener una puerta de enlace distinta** a Linux2 → **encaminamiento asimétrico**: la ida la decide tu tabla de rutas, la vuelta la tabla del otro extremo y los routers intermedios.
  - Borrar: `route delete 10.6.0.0`.

**C4. Redirect** (`route add 172.18.41.0 mask 255.255.255.0 172.20.43.242` + `ping -n 1 172.18.41.4`)
```
1) PC  → R5  (MAC_PC → MAC_R5)   | PC → 172.18.41.4 | Echo Request
2) R5  → PC  (MAC_R5 → MAC_PC)   | R5 (172.20.43.242) → PC | ICMP Redirect (tipo 5): "usa 172.20.43.240"
3) R5  → R2  (MAC_R5 → MAC_R2)   | PC → 172.18.41.4 | Echo Request (reenviado, IPs iguales)
   ... R2 → R1 (172.18.41.4 es de R1) ...
4) R2  → PC  (MAC_R2 → MAC_PC)   | 172.18.41.4 → PC | Echo Reply
```
- a) Involucrados en nuestro segmento: **4** (más ARP si hace falta). Con el switch, **en tu PC capturas 3** (el 3 va de R5 a R2 y no pasa por tu puerto).
- c) Lo envía **R5** (172.20.43.242): el siguiente salto hacia 172.18.41.0 es otro router de la **misma red** por la **misma interfaz** por la que entró, y el origen (tu PC) está en esa red.
- d) Lleva la **IP del router al que debes enviar** (gateway, aquí presumiblemente 172.20.43.240 = R2) + la **cabecera IP y 8 primeros bytes** del Echo Request original.
- Al repetir el ping: **sin Redirect**, el Echo Request va **directo** al router indicado. (Si reaparece es que expiró el temporizador de Windows.)
- Borrar: `route delete 172.18.41.0`.

**C5. Destination Unreachable**
- `ping -n 1 -l 1200 -f 10.3.0.2` → IP de 1228 B con **DF = 1**. PC → R2 (MAC de R2) → R1. R1 no puede pasarlo por el **PaP** (MTU < 1228) y no puede fragmentar → **lo descarta y envía ICMP 3/4**.
  - a) Echo Request: MAC PC → MAC R2, IP PC → 10.3.0.2. 3/4: IP origen = **R1**, IP destino = PC; MAC origen = el router de nuestra red que lo entrega (p.ej. R2), MAC destino = PC.
  - b) Lo envía **R1** (el router que no puede reenviar el paquete por el siguiente enlace).
  - c) El MTU del PaP se lee en la **cabecera ICMP** del 3/4 (campo "MTU of next hop" en Wireshark).
- Sin `-f` (`ping -n 1 -l 1200 10.3.0.2`, tras limpiar con `route delete 0.0.0.0`, `ipconfig /release`, `/renew`, `c:\pracredes`): ahora R1 fragmenta y el Echo Reply llega fragmentado.
  - a) **No**: el mayor fragmento recibido es **algo menor** que el MTU informado.
  - b) **Razón**: el **Fragment Offset se expresa en unidades de 8 bytes**, así que los datos de cada fragmento (salvo el último) tienen que ser **múltiplo de 8**: datos = ⌊(MTU−20)/8⌋·8 (ver ejemplo 11.6).
  - ➕ La limpieza inicial sirve para borrar la información que el PC haya memorizado sobre ese destino (rutas/MTU aprendido), para que el experimento salga desde cero.
- `ping -n 1 172.18.41.2` → se recibe **ICMP 3/13 (Communication Administratively Prohibited)** enviado por el router que filtra (el que da acceso a 172.18.41.0/28, **R1**; identifícalo por la IP origen del ICMP).
- `ping -n 1 172.18.41.1` → si ahora **no** llega 3/13 (y sí Echo Reply), el router **no filtra todo ICMP**: filtra **según la dirección de destino** (una regla que solo deja pasar el tráfico a ciertos equipos, p.ej. la salida 172.18.41.1, y bloquea el resto).

**C6. Time Exceeded**
- `ping -i 1 -n 1 10.3.0.2`:
  - a) Lo envía **R2** (172.20.43.240): primer router, TTL 1 → 0.
  - b) Lo causó **nuestro Echo Request** (se ve dentro del 11/0: cabecera IP + 8 bytes).
  - c) **2 paquetes ICMP**: Echo Request + Time Exceeded.
- `ping -i 2 -n 1 10.3.0.2`:
  - a) Lo envía **R1** (segundo salto; IP 10.4.0.2): R2 lo baja a 1 y lo reenvía, R1 lo baja a 0.
  - b) El mismo Echo Request.
  - c) De nuevo **2 paquetes ICMP** en tu captura.
  - d) Diferencia: el error viene **de un router más lejano** (R1 en vez de R2): IP origen distinta, tarda más, y en la trama la **MAC origen es la del router de tu red que lo entrega**, no la de R1.
- tracert: `10.3.0.2` → R2, R1, R4 (dado en el PDF). Para `10.6.0.2`, `10.6.0.3`, `10.4.0.3` apunta los saltos de tu captura; si el destino es una interfaz de un router con pata en tu red (10.6.0.3 de R5, 10.4.0.3 de R3), puede llegar el Echo Reply directamente de él.
- `tracert -d 10.3.0.10` → **bucle de encaminamiento**: 10.3.0.10 **no existe** en el PaP (solo .1 y .2). R1 lo manda por el PaP a R4; R4 ve que pertenece a 10.3.0.0/24 (red conectada) pero no es suyo y, al ser un enlace punto a punto, lo devuelve a R1… El paquete rebota entre **R1 y R4** hasta que el TTL llega a 0 → tracert muestra **R1 y R4 alternándose** hasta los 30 saltos. (Si algún router lo descartara, verías `*` de tiempo agotado.) El TTL es justamente lo que impide que circule para siempre.

**C7. Errores ICMP con direcciones de difusión** (`ping -n 1 -i 1 10.4.0.255` y `ping -n 1 -i 1 10.6.0.255`)
- Ambos llegan primero a **R2** con TTL = 1.
- **10.4.0.255**: es el **broadcast de una red directamente conectada a R2** (10.4.0.0/24) → R2 lo reconoce como **dirección de difusión** → por la regla, **NO genera Time Exceeded** (como mucho R2 contesta él mismo, o no se recibe nada).
- **10.6.0.255**: R2 **no** está conectado a 10.6.0.0/24 → para R2 es una dirección normal → TTL llega a 0 → envía **ICMP 11/0**.
- Respuesta: el destino que **NO** genera error es **10.4.0.255**, porque no se generan errores ICMP para paquetes dirigidos a una **dirección IP de broadcast** (evita tormentas de broadcast).

**C8. Rendimiento** → `ping -n 1 -l 472 10.3.0.2` y `... 10.4.0.2` cada 15 s durante 15 min (60 muestras) → gráfica retardo–tiempo, R̄, S y S(%) (ver sección 16). Si S(%) > 50 % → rendimiento bajo.
