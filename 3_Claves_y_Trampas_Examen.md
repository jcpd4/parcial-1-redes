# LO IMPRESCINDIBLE Y LAS TRAMPAS — Prácticas 1 y 2

> Repásalo justo antes del examen. Si dominas esta hoja, tienes la base para todo lo demás.

---

## 🔟 Las 10 ideas que más puntos dan

1. **Las IP no cambian de extremo a extremo; las MAC cambian en cada salto.** Las MAC solo sirven para llegar al siguiente salto dentro de cada segmento.
2. **Si el destino está en otra red, ARP pregunta por la puerta de enlace**, no por el destino. Desde tu PC solo ves el ARP de **tu** segmento.
3. **Con una sola secuencia ARP basta** en un ping dentro de la misma red: el destino aprende la MAC del origen gracias al propio ARP Request.
4. **ping -l D → ICMP = D + 8, IP = D + 28, trama = D + 42.** Máximo sin fragmentar en Ethernet: **-l 1472**.
5. **Fragmentar en Ethernet: trozos de 1480 B; el Offset va en grupos de 8 B (1480 → 185); MF = 1 en todos menos el último; la Identificación es la misma; solo el primer fragmento lleva la cabecera ICMP.**
6. **Los datos de cada fragmento (salvo el último) son múltiplo de 8** → con un MTU "raro" el paquete más grande es **menor** que el MTU.
7. **Tipos ICMP: 0 Reply · 3 Unreachable · 4 Source Quench · 5 Redirect · 8 Request · 11 Time Exceeded.** Códigos: 3/1 host, 3/3 puerto, 3/4 DF, 3/13 filtrado · 11/0 TTL, 11/1 reensamblado.
8. **Los errores ICMP llevan la cabecera IP + 8 bytes del paquete culpable** y **nunca** se generan por otro error, por un broadcast (IP o MAC), por fragmentos que no sean el primero, por fragmentos fuera de secuencia ni por orígenes que no sean una única máquina.
9. **Redirect**: lo envía un **router** cuando reenviaría el paquete **por la misma interfaz** por la que le llegó, a otro router de la misma red; **el paquete sí se reenvía**; solo si el origen está en la **misma red** que la interfaz del router.
10. **Subredes**: amplía n bits con 2^n ≥ subredes; equipos = 2^h − 2; red = bits de máquina a 0, broadcast = bits de máquina a 1. Se listan **todas** las 2^n subredes.

---

## 🗺️ La topología en 30 segundos

| | |
|---|---|
| Red de alumnos | **172.20.43.192/26** · máscara 255.255.255.192 · equipos .193–.254 · broadcast **.255** |
| Puerta de enlace por defecto | **172.20.43.240 = R2** |
| Otros routers en tu red | **.241 = R3** · **.242 = R5** · Linux1 = .243 |
| R1 | 172.18.41.4 (Internet, NAT) · 10.4.0.2 · 10.3.0.1 |
| R2 | 10.4.0.1 · 172.20.43.240 |
| R3 | 10.4.0.3 · 10.6.0.2 · 172.20.43.241 |
| R4 | 10.3.0.2 · 10.6.0.1 |
| R5 | 10.6.0.3 · 172.20.43.242 |
| 10.6.0.0/24 | R3, R4, R5, Linux2 (10.6.0.5), Linux3 (10.6.0.6) |
| **10.3.0.0/24** | **PaP a 10 Mbps entre R1 y R4: el único NO Ethernet → MTU menor → aquí salen el 3/4 y la fragmentación por el camino** |
| Ruta a 10.3.0.2 (tracert) | 172.20.43.240 (R2) → 10.4.0.2 (R1) → 10.3.0.2 (R4) |
| DNS de la UA | 193.145.233.5 y 193.145.233.6 |
| Todo son **switches** | Solo capturas tu tráfico + broadcast |

---

## 🔢 Números que tienes que saber de memoria

| Dato | Valor |
|---|---|
| MAC | 48 bits / 6 bytes / 12 hex · 3 B fabricante + 3 B adaptador |
| IPv4 | 32 bits |
| Broadcast MAC | FF:FF:FF:FF:FF:FF |
| MAC de grupo | bit menos significativo del **1er byte** = 1 (1er byte impar) |
| Cabecera Ethernet DIX | 14 B (6 + 6 + 2) + CRC 4 |
| Datos DIX / 802.3 | **46–1500** / **38–1492** |
| Cabecera IP | 20 B (HL = 5); máx. 60 B (HL = 15) |
| Cabecera ICMP | **8 B** (tipo 1 + código 1 + SVT 2 + info variable 4) |
| Longitud Total IP | 16 bits → máx. 65535 B |
| Fragment Offset | 13 bits, unidades de **8 bytes** |
| TTL | 8 bits → máx. 255 saltos |
| Flags | bit 0 reservado · bit 1 **DF** · bit 2 **MF** |
| Protocolo | 1 ICMP · 6 TCP · 17 UDP |
| ARP cache | Unix 20 min · Windows de 2 min (XP) a horas (W10) |
| Ethernet compartida | funciona bien con menos de ~25 equipos |
| Velocidades | 10 Mbps → 100 (Fast) → 1 Gbps (Gigabit) → 10 Gbps |
| tracert | 3 sondas por salto, máximo 30 saltos |
| ping (Windows) | 4 paquetes y 32 B por defecto |
| Medición de rendimiento | MTU 500 → **-l 472** · cada 15 s durante 15 min → **n = 60** · **S% > 50 → rendimiento bajo** |
| RFCs | 791 IPv4 · 792 ICMP · 826 ARP · 894 Ethernet DIX · 1042 802.3 · 1883 IPv6 |

### Clases
| Clase | Bits iniciales | 1er octeto | Máscara | Redes | Equipos |
|---|---|---|---|---|---|
| A | 0 | 0–127 | /8 | 2^7 | 2^24 − 2 |
| B | 10 | 128–191 | /16 | 2^14 | 2^16 − 2 |
| C | 110 | 192–223 | /24 | 2^21 = 2.097.152 | 254 |
| D | 1110 | 224–239 | — | multicast | — |
| E | 11110 | 240–247 (según la figura) | — | futuras ampliaciones | — |

### Máscaras (octeto)
`/25 = 128 · /26 = 192 · /27 = 224 · /28 = 240 · /29 = 248 · /30 = 252`
Salto = 256 − valor del octeto (192 → 64; 224 → 32; 240 → 16).

---

## ⚠️ TRAMPAS TÍPICAS (las que hacen fallar a la gente)

### Ethernet / MAC / hub-switch
- ⚠️ **MTU ≠ tamaño de trama.** MTU (1500) = datos de la trama = datagrama IP completo. La trama en Wireshark es de 1514 (1518 con CRC).
- ⚠️ **TCP/IP usa DIX (RFC 894)**, no 802.3. El 3er campo de DIX es **Tipo**; en 802.3, **Longitud**.
- ⚠️ **Ethernet NO fragmenta.** Lo hace IP.
- ⚠️ Para saber si una MAC es de grupo, mira el **bit menos significativo del PRIMER byte** (no el último, ni el más significativo). 33:33:… → 0x33 es impar → grupo.
- ⚠️ El broadcast FF:FF:FF:FF:FF:FF también tiene el bit de grupo a 1.
- ⚠️ El **switch** da **más** seguridad y permite transmisiones simultáneas; el **hub** repite todo y causa colisiones.

### ARP
- ⚠️ **Request = broadcast; Reply = unicast.**
- ⚠️ ARP **no** lleva cabecera IP → cuenta como paquete **no IP**.
- ⚠️ Si la entrada está en caché → **no** hay ARP. Para forzarlo: `arp -d`.
- ⚠️ Si el compañero borró tu entrada, el ARP aparece **después** del Echo Request (lo hace él para responderte).
- ⚠️ Destino en tu red pero inexistente (.250) → ARP sin respuesta y **no sale ningún Echo Request**.
- ⚠️ Ping a la IP de broadcast → **no hay ARP**: la MAC destino es directamente FF:FF:FF:FF:FF:FF.
- ⚠️ Nota 2: solo cuentan los ARP con destino FF:FF:FF:FF:FF:FF (hay ARP unicast "anti-spoofing" que debes ignorar).

### IP / direccionamiento
- ⚠️ **HL está en palabras de 32 bits**, no en bytes (5 → 20 B).
- ⚠️ **Longitud Total = cabecera + datos**, en bytes.
- ⚠️ **La suma de control IP solo cubre la cabecera**; la SVT de ICMP cubre **todo** el mensaje ICMP.
- ⚠️ Equipos por red = **2^h − 2** (no 2^h).
- ⚠️ En subredes, la práctica cuenta la subred de "todo ceros" y la de "todo unos" (192.168.100.0/26 y .192/26 se listan).
- ⚠️ Cuidado con /23: 150.22.0.0/23 abarca **dos** valores del 3er octeto (0 y 1).
- ⚠️ 10.0.0.0 es de **clase A** aunque en la práctica se use con /16 o /24 (es "ampliación de máscara").
- ⚠️ La ruta **0.0.0.0/0** es la **última** que se comprueba; una ruta /32 o /24 añadida con `route` tiene preferencia.

### ICMP
- ⚠️ **ping NO usa TCP ni UDP.** Responder al ping ≠ que TCP/UDP funcionen.
- ⚠️ El Echo Request **sí** puede generar errores (es una solicitud); un error **nunca** genera otro error.
- ⚠️ **3/1** lo genera el **router** al fallar el ARP del destino, y muchos routers lo desactivan → a menudo solo ves "tiempo agotado".
- ⚠️ **3/3** lo genera el **host destino** (puerto sin proceso), no un router.
- ⚠️ **11/0** lo genera un **router**; **11/1** lo genera el **destino** (reensamblado).
- ⚠️ **Redirect**: el router **no** descarta el paquete, lo reenvía; Windows 10 **no** lo guarda en la tabla de rutas (usa un temporizador).
- ⚠️ El Redirect **no** se genera en redes intermedias (el origen tiene que estar en la red de esa interfaz del router).
- ⚠️ **Source Quench** es el **único** mecanismo de control de congestión de IP.
- ⚠️ En un error ICMP que viene de un router lejano (R1), la **IP origen** es la de R1 pero la **MAC origen** es la del router de tu segmento que te lo entrega.

### Fragmentación
- ⚠️ El Offset del 2º fragmento en Ethernet vale **185 en el campo** (1480 bytes). Si te preguntan "valor del campo", divide entre 8.
- ⚠️ **Solo el 1er fragmento lleva la cabecera ICMP (8 B)**: datos del ping en el 1er fragmento = 1472.
- ⚠️ El último fragmento: **MF = 0 pero offset > 0**. Un paquete no fragmentado: MF = 0 **y** offset = 0.
- ⚠️ `ping -l 1200 -f` a una máquina de **tu** red **no** da error (1228 < 1500). El 3/4 solo sale cuando el camino pasa por un enlace de MTU menor (el PaP hacia 10.3.0.2).
- ⚠️ `-f` con un paquete mayor que **tu propio** MTU (p.ej. `-l 1500`): el error lo da tu PC y **no sale nada** a la red.
- ⚠️ **Los routers no reensamblan** → la respuesta de 10.3.0.2 te llega fragmentada.
- ⚠️ Más fragmentos = **más** retardo (2,001 ms vs 2,1 ms), por el tiempo de encaminamiento por paquete.
- ⚠️ IPv6: los routers **no** fragmentan.

### tracert / TTL
- ⚠️ Cada línea de tracert tiene **3 tiempos** porque envía 3 sondas por salto.
- ⚠️ El último salto **no** responde con Time Exceeded sino con **Echo Reply**.
- ⚠️ Con `ping -i 1` el error lo da **R2** (el primer router); con `-i 2` lo da el **segundo** router (R1 si vas hacia 10.3.0.x o 10.4.0.2).
- ⚠️ tracert a una IP inexistente del PaP (10.3.0.10) → **bucle R1 ↔ R4** hasta que se agota el TTL.
- ⚠️ `ping -i 1` a **10.4.0.255** no da error (es el broadcast de una red conectada a R2); a **10.6.0.255** sí (11/0 de R2).

### Rendimiento
- ⚠️ La desviación típica divide entre **n − 1**, no entre n.
- ⚠️ `-l` = MTU − 28 (no MTU − 20 ni MTU − 14).
- ⚠️ El umbral es **S% > 50 %** → rendimiento bajo.

---

## 🧠 Plantillas de razonamiento (para cualquier pregunta de "¿qué paquetes aparecen?")

**Paso 1 — ¿El destino está en mi red?** Aplica la máscara (/26 → .192–.255).
- Sí → ARP por el **destino**. MAC destino = destino.
- No → mira la tabla de rutas (¿hay una ruta añadida con route? si no, la por defecto) → ARP por el **router** elegido. MAC destino = router; IP destino = destino final.

**Paso 2 — ¿Está la MAC en la caché?** Sí → no hay ARP. No → Request (broadcast) + Reply (unicast).

**Paso 3 — ¿Cabe el paquete?** IP = D + 28. Si > 1500 → fragmenta tu PC (1480 por trozo). Si cabe pero pasa por el PaP → fragmenta R1 (o, con DF = 1, R1 envía un 3/4).

**Paso 4 — ¿TTL suficiente?** Cada router resta 1; si llega a 0 → ese router envía 11/0 (salvo que el destino sea una dirección de broadcast).

**Paso 5 — ¿Existe el destino?** No → en tu red: ARP sin respuesta. En otra red: tiempo agotado (o 3/1).

**Paso 6 — ¿Por dónde vuelve?** Lo decide el otro extremo (su puerta de enlace) y los routers → puede ser **asimétrico**. Identifícalo por la **MAC origen** de la respuesta + `arp -a`.

**Paso 7 — ¿Qué veo en mi Wireshark?** Solo lo que entra o sale de mi PC y los broadcast (switch).

---

## 📝 Fórmulas de bolsillo

```
ICMP = D + 8        IP = D + 28        Trama Ethernet = D + 42 (+4 CRC)
Datos por fragmento = ⌊(MTU − 20)/8⌋·8          (Ethernet: 1480)
Campo FO = offset_bytes / 8                       (1480→185, 2960→370, 4440→555)
Nº subredes = 2^n          Equipos/subred = 2^h − 2         Salto = 256 − octeto de máscara
T_tr = bits / velocidad    T_total = N·Ttr1 + N·Tenc + N·Ttr2
Media = Σr/n    S = √(Σ(media − r)²/(n − 1))    S% = S/media·100   (>50 % → malo)
```

---

## ✅ Checklist final (márcalas cuando las sepas hacer sin mirar)

- [ ] Dibujar la topología con todas las IPs de los routers
- [ ] Escribir la secuencia ARP + ICMP de un ping en la misma red y en otra red (con MACs e IPs)
- [ ] Explicar por qué solo hay una secuencia ARP
- [ ] Decir si una MAC es unicast o multicast
- [ ] Dar la clase y la máscara por defecto de cualquier IP
- [ ] Dividir una red en N subredes (máscara, rangos, broadcast)
- [ ] Recitar los campos de la cabecera IP con su tamaño y función
- [ ] Calcular los tamaños de un ping (ICMP, IP, trama)
- [ ] Fragmentar cualquier ping (Long. Total, Offset en bytes y en el campo, MF, DF)
- [ ] Explicar por qué el fragmento no coincide con el MTU (múltiplo de 8)
- [ ] Recitar los tipos ICMP (0, 3, 4, 5, 8, 11) y los códigos 3/1, 3/3, 3/4, 3/13, 11/0, 11/1
- [ ] Las 6 situaciones que no generan error ICMP y su motivo (tormentas de broadcast)
- [ ] Explicar el Redirect paso a paso y su condición
- [ ] Explicar tracert y su resultado a 10.3.0.2
- [ ] Explicar el bucle de tracert a 10.3.0.10
- [ ] Explicar por qué 10.4.0.255 no da error y 10.6.0.255 sí
- [ ] Explicar Source Quench y 11/1
- [ ] Explicar por qué fragmentar es malo (con el ejemplo de 2,001 vs 2,1 ms)
- [ ] Calcular media, S y S% y decidir si el rendimiento es bueno
- [ ] Escribir los comandos: ipconfig /all, arp -a/-d, netstat -rn, route add/delete, ping -n -l -i -f, tracert -d
- [ ] Escribir los filtros de Wireshark de las cuestiones 4.1
