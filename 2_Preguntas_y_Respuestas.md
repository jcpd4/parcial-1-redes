# PREGUNTAS DE EXAMEN CON RESPUESTA — Prácticas 1 y 2

> **Cómo usarlo:** intenta responder antes de abrir la respuesta (pulsa en "▶ Respuesta"; en VS Code abre la vista previa con `Cmd+Shift+V`).
> Apunta las que falles y repítelas al día siguiente.
> Bloques: **A** test · **B** verdadero/falso · **C** respuesta corta/desarrollo · **D** cálculos (tamaños, fragmentación, subredes, rendimiento) · **E** escenarios sobre la topología del laboratorio · **F** comandos y Wireshark

---

## BLOQUE A — Tipo test

**A1.** ¿Cuántos bits tiene una dirección MAC?
a) 32 b) 48 c) 64 d) 128
<details><summary>▶ Respuesta</summary>

**b) 48 bits** (6 bytes, 12 dígitos hexadecimales). La IP v4 tiene 32.
</details>

**A2.** ¿Qué encapsulación Ethernet utiliza TCP/IP?
a) IEEE 802.3 (RFC 1042) b) Ethernet DIX (RFC 894) c) 802.2 SNAP d) LLC
<details><summary>▶ Respuesta</summary>

**b) Ethernet DIX (RFC 894).** El protocolo LLC usa 802.3.
</details>

**A3.** El MTU de una trama IEEE 802.3 es:
a) 1500 b) 1492 c) 1480 d) 1518
<details><summary>▶ Respuesta</summary>

**b) 1492 bytes.** DIX = 1500. (1480 son los datos IP máximos en Ethernet DIX; 1518 la trama completa con CRC.)
</details>

**A4.** ¿Cuál de estas direcciones MAC es de grupo (multicast)?
a) 00:1A:2B:3C:4D:5E b) 02:00:00:00:00:01 c) 03:00:00:00:00:01 d) FE:00:00:00:00:01
<details><summary>▶ Respuesta</summary>

**c) 03:…** — su primer byte es 0000 001**1**: el bit menos significativo del primer byte está a 1. (00, 02 y FE son pares → unicast.)
</details>

**A5.** La dirección MAC de destino de un ARP Request es:
a) La MAC del destino b) La MAC del router c) FF:FF:FF:FF:FF:FF d) 00:00:00:00:00:00
<details><summary>▶ Respuesta</summary>

**c) FF:FF:FF:FF:FF:FF** (broadcast), porque todavía no se conoce la MAC del destino. El ARP Reply es **unicast**.
</details>

**A6.** ¿En qué RFC está ARP?
a) 791 b) 792 c) 826 d) 894
<details><summary>▶ Respuesta</summary>

**c) RFC 826.** 791 = IPv4, 792 = ICMP, 894 = Ethernet DIX, 1883 = IPv6 (según la práctica).
</details>

**A7.** Tiempo normal de expiración de una entrada ARP en Unix:
a) 2 min b) 20 min c) 2 h d) Nunca expira
<details><summary>▶ Respuesta</summary>

**b) 20 minutos.** Windows: desde 2 min (XP) hasta varias horas (Windows 10).
</details>

**A8.** La dirección 191.20.5.1 es de clase:
a) A b) B c) C d) D
<details><summary>▶ Respuesta</summary>

**b) B** (128–191, empieza por bits 10).
</details>

**A9.** ¿Cuántos equipos se pueden direccionar en una subred /26?
a) 64 b) 62 c) 30 d) 126
<details><summary>▶ Respuesta</summary>

**b) 62** = 2^6 − 2 (se restan la dirección de red y la de broadcast).
</details>

**A10.** Valor habitual del campo HL de la cabecera IP:
a) 4 b) 5 c) 20 d) 60
<details><summary>▶ Respuesta</summary>

**b) 5** → 5 palabras de 32 bits = 20 bytes (sin opciones). HL se mide en palabras de 32 bits, no en bytes.
</details>

**A11.** Longitud máxima de la cabecera IP:
a) 20 B b) 32 B c) 60 B d) 65535 B
<details><summary>▶ Respuesta</summary>

**c) 60 bytes** = (2^4 − 1) × 4 bytes = 15 × 4.
</details>

**A12.** Un paquete IP con el campo Protocolo = 17 transporta:
a) ICMP b) TCP c) UDP d) ARP
<details><summary>▶ Respuesta</summary>

**c) UDP.** 1 = ICMP, 6 = TCP. ARP no va dentro de IP.
</details>

**A13.** Si un fragmento empieza en el byte 2960 del datagrama original, el campo Fragment Offset vale:
a) 2960 b) 370 c) 185 d) 1480
<details><summary>▶ Respuesta</summary>

**b) 370** = 2960 / 8. El offset se expresa en grupos de 8 bytes.
</details>

**A14.** La suma de control de la cabecera IP cubre:
a) Todo el datagrama b) Solo la cabecera IP c) La cabecera IP y los 8 primeros bytes de datos d) La trama Ethernet
<details><summary>▶ Respuesta</summary>

**b) Solo la cabecera IP.** Así el router, que normalmente solo cambia el TTL, la recalcula de forma incremental y rápida. (La SVT de ICMP sí cubre **todo** el mensaje ICMP.)
</details>

**A15.** El tipo ICMP del Echo Request es:
a) 0 b) 3 c) 8 d) 11
<details><summary>▶ Respuesta</summary>

**c) 8.** Echo Reply = 0.
</details>

**A16.** Cuando el TTL llega a 0 en un router se genera el ICMP:
a) 3/1 b) 11/0 c) 11/1 d) 5/1
<details><summary>▶ Respuesta</summary>

**b) 11/0 — Time to Live exceeded in Transit.** 11/1 es Fragment Reassembly Time Exceeded.
</details>

**A17.** Un router necesita fragmentar un paquete con DF = 1. ¿Qué envía?
a) 3/1 b) 3/3 c) 3/4 d) 3/13
<details><summary>▶ Respuesta</summary>

**c) 3/4 — Fragmentation Needed and Don't Fragment was Set.**
</details>

**A18.** El mensaje ICMP que indica que un router filtra el paquete por sus reglas es:
a) 3/1 b) 3/4 c) 3/13 d) 4/0
<details><summary>▶ Respuesta</summary>

**c) 3/13 — Communication Administratively Prohibited.**
</details>

**A19.** El ICMP 3/1 Host Unreachable lo genera un router cuando:
a) El TTL llega a 0 b) No obtiene por ARP la MAC del destino c) No hay proceso en el puerto d) Su cola está llena
<details><summary>▶ Respuesta</summary>

**b)** No consigue la MAC del destino por ARP (máquina apagada o inexistente). c) sería 3/3, d) Source Quench.
</details>

**A20.** El único mecanismo de control de congestión de IP es:
a) Redirect b) Source Quench c) Time Exceeded d) La fragmentación
<details><summary>▶ Respuesta</summary>

**b) Source Quench (tipo 4)**: el router con la cola de entrada llena descarta paquetes y pide al origen que reduzca el flujo.
</details>

**A21.** El ICMP Redirect lo envía:
a) El host destino b) El host origen c) Un router d) El servidor DNS
<details><summary>▶ Respuesta</summary>

**c) Un router**, al emisor del paquete, cuando el siguiente salto es otro router de la misma red y el paquete saldría por la misma interfaz por la que entró.
</details>

**A22.** Un mensaje de error ICMP contiene en su campo de datos:
a) Todo el paquete original b) La cabecera IP + 8 primeros bytes de datos del paquete original c) Solo la IP origen d) Nada
<details><summary>▶ Respuesta</summary>

**b) Cabecera IP + 8 primeros bytes de datos** del paquete que provocó el error.
</details>

**A23.** ¿Cuál de estas situaciones SÍ puede generar un mensaje de error ICMP?
a) Un mensaje de error ICMP b) Un paquete IP a una dirección de broadcast c) Un Echo Request cuyo TTL llega a 0 d) Un fragmento que no es el primero
<details><summary>▶ Respuesta</summary>

**c)** Un Echo Request es una **solicitud** ICMP, no un error: sí puede generar error. Las otras tres no generan error.
</details>

**A24.** ping funciona sobre:
a) TCP b) UDP c) Directamente sobre IP (ICMP) d) ARP
<details><summary>▶ Respuesta</summary>

**c) Directamente sobre IP** (ICMP). Por eso que un equipo responda al ping no garantiza que su TCP/UDP funcione bien.
</details>

**A25.** ¿Cuántos paquetes envía tracert por cada salto?
a) 1 b) 2 c) 3 d) 4
<details><summary>▶ Respuesta</summary>

**c) 3**, para dar una estadística del tiempo en cada salto (por eso salen 3 tiempos por línea).
</details>

**A26.** La opción `-d` de tracert sirve para:
a) Activar DF b) No resolver nombres (no consultar DNS) c) Fijar el TTL d) Cambiar el tamaño
<details><summary>▶ Respuesta</summary>

**b)** Evita el envío de consultas a los servidores DNS.
</details>

**A27.** La opción `-i 5` del ping de Windows:
a) Envía 5 paquetes b) Pone 5 bytes de datos c) Pone TTL = 5 d) Espera 5 s
<details><summary>▶ Respuesta</summary>

**c) TTL = 5.** -n = número de paquetes, -l = bytes de datos, -f = Don't Fragment.
</details>

**A28.** ¿Quién fragmenta un datagrama demasiado grande para Ethernet?
a) Ethernet b) IP c) ICMP d) El switch
<details><summary>▶ Respuesta</summary>

**b) IP.** Ethernet no puede fragmentar: su cabecera no tiene campos para ello.
</details>

**A29.** Con un **switch**, el Wireshark de tu PC captura:
a) Todo el tráfico de la red b) Solo lo que envía/recibe tu PC (y broadcast) c) Nada d) Solo broadcast
<details><summary>▶ Respuesta</summary>

**b).** Con un hub capturarías el tráfico de todos.
</details>

**A30.** Según la práctica, la Ethernet de bus compartido funciona bien hasta unos:
a) 5 equipos b) 25 equipos c) 255 equipos d) 1024 equipos
<details><summary>▶ Respuesta</summary>

**b) Unos 25.** Con más, la red entra en colisión permanente.
</details>

**A31.** Datos mínimo y máximo de una trama Ethernet DIX:
a) 38–1492 b) 46–1500 c) 64–1518 d) 0–1500
<details><summary>▶ Respuesta</summary>

**b) 46–1500.** a) es 802.3; c) es la trama completa.
</details>

**A32.** La puerta de enlace por defecto de los PCs del laboratorio es:
a) 172.20.43.241 b) 172.20.43.240 c) 172.20.43.242 d) 10.4.0.1
<details><summary>▶ Respuesta</summary>

**b) 172.20.43.240 (R2).** Ruta `0.0.0.0 0.0.0.0 172.20.43.240`.
</details>

**A33.** ¿Cuántas redes de clase C existen?
a) 2^8 b) 2^14 c) 2^21 d) 2^24
<details><summary>▶ Respuesta</summary>

**c) 2^21 = 2.097.152** (24 bits de red − 3 fijos "110").
</details>

**A34.** El tercer campo de la trama Ethernet DIX es:
a) Longitud b) Tipo c) DSAP d) CRC
<details><summary>▶ Respuesta</summary>

**b) Tipo** (2 bytes). En 802.3 es **Longitud**.
</details>

**A35.** El segmento del laboratorio que **no** es Ethernet es:
a) 10.4.0.0/24 b) 10.6.0.0/24 c) 10.3.0.0/24 d) 172.18.41.0/28
<details><summary>▶ Respuesta</summary>

**c) 10.3.0.0/24**: línea punto a punto (PaP) entre R1 y R4 a 10 Mbps.
</details>

**A36.** En un datagrama fragmentado en 3 trozos, ¿qué fragmentos llevan MF = 1?
a) Todos b) Solo el último c) El 1º y el 2º d) Solo el primero
<details><summary>▶ Respuesta</summary>

**c) Todos menos el último.**
</details>

**A37.** Los fragmentos de un mismo datagrama tienen igual:
a) Fragment Offset b) Identificación c) Longitud Total d) Suma de control
<details><summary>▶ Respuesta</summary>

**b) Identificación.**
</details>

**A38.** ¿Qué ICMP genera la **máquina destino** cuando no le llega a tiempo algún fragmento?
a) 11/0 b) 11/1 c) 3/4 d) 4
<details><summary>▶ Respuesta</summary>

**b) 11/1 Fragment Reassembly Time Exceeded.** Sirve para detectar congestión por fragmentación.
</details>

**A39.** La característica de IPv6 que destaca la práctica es:
a) Direcciones de 48 bits b) No permite fragmentar en los routers c) No tiene TTL d) Usa ARP
<details><summary>▶ Respuesta</summary>

**b)** Los routers IPv6 no fragmentan.
</details>

**A40.** Si S(%) de los retardos sale 65 %, la red tiene:
a) Buen rendimiento b) Rendimiento bajo c) Rendimiento perfecto d) No se puede saber
<details><summary>▶ Respuesta</summary>

**b) Rendimiento bajo** (> 50 %: carga no homogénea, picos de utilización).
</details>

---

## BLOQUE B — Verdadero o falso (justifica)

**B1.** Las direcciones IP origen y destino de un datagrama cambian en cada router.
<details><summary>▶ Respuesta</summary>

**Falso.** Las IP se mantienen de extremo a extremo; lo que cambia en cada salto son las **direcciones MAC**.
</details>

**B2.** Si el destino está en otra red, el ARP Request pregunta por la IP del destino final.
<details><summary>▶ Respuesta</summary>

**Falso.** Pregunta por la IP de la **puerta de enlace** (el router del siguiente salto).
</details>

**B3.** Ethernet puede fragmentar los datagramas que le pasa IP.
<details><summary>▶ Respuesta</summary>

**Falso.** Su cabecera no tiene información de fragmentación; lo hace IP.
</details>

**B4.** En un ping entre dos PCs de la misma red con las cachés vacías aparecen dos secuencias ARP.
<details><summary>▶ Respuesta</summary>

**Falso.** Aparece **una**: el destino aprende la MAC del origen del propio ARP Request, que lleva la IP/MAC del remitente.
</details>

**B5.** Un mensaje de error ICMP nunca se genera como respuesta a otro mensaje de error ICMP.
<details><summary>▶ Respuesta</summary>

**Verdadero.** Así se evitan cascadas de errores. Sí se puede generar en respuesta a una **solicitud** ICMP.
</details>

**B6.** Si una máquina responde al ping, su capa TCP funciona correctamente.
<details><summary>▶ Respuesta</summary>

**Falso.** ping solo usa IP/ICMP: indica que está activa a **nivel IP**.
</details>

**B7.** El campo Fragment Offset se expresa en bytes.
<details><summary>▶ Respuesta</summary>

**Falso.** En **grupos de 8 bytes** (13 bits). 1480 bytes → 185.
</details>

**B8.** El último fragmento lleva MF = 1.
<details><summary>▶ Respuesta</summary>

**Falso.** El último lleva **MF = 0**; los demás MF = 1.
</details>

**B9.** Cuando un router envía un ICMP Redirect, descarta el paquete original.
<details><summary>▶ Respuesta</summary>

**Falso.** Envía el Redirect al origen **y además reenvía el paquete** al router correcto.
</details>

**B10.** Windows 10 guarda la información del Redirect en la tabla de rutas.
<details><summary>▶ Respuesta</summary>

**Falso.** La guarda con un **temporizador**, no en la tabla de rutas (por eso no aparece en `netstat -rn`, y si expira vuelve a salir el Redirect).
</details>

**B11.** Fragmentar más un mensaje reduce su retardo total en una red moderna.
<details><summary>▶ Respuesta</summary>

**Falso.** Aumenta el retardo: el router gasta el mismo tiempo de CPU por paquete, sea cual sea su tamaño (ejemplo: 2,001 ms frente a 2,1 ms).
</details>

**B12.** Un paquete IP puede atravesar como mucho 255 routers.
<details><summary>▶ Respuesta</summary>

**Verdadero.** El TTL tiene 8 bits (máx. 255) y cada router lo decrementa en 1.
</details>

**B13.** HL = 5 significa que la cabecera mide 5 bytes.
<details><summary>▶ Respuesta</summary>

**Falso.** 5 palabras de 32 bits = **20 bytes**.
</details>

**B14.** El Redirect se genera aunque la IP origen del paquete no pertenezca a la red de la interfaz del router.
<details><summary>▶ Respuesta</summary>

**Falso.** Solo se envía si el origen está en la **misma red IP** que la interfaz del router; en redes intermedias no se genera.
</details>

**B15.** Un paquete IP dirigido a una dirección de broadcast puede generar un Time Exceeded.
<details><summary>▶ Respuesta</summary>

**Falso.** Los paquetes dirigidos a una dirección IP de broadcast no generan errores ICMP (se evitan tormentas de broadcast).
</details>

**B16.** La ruta 0.0.0.0 es la primera que se comprueba en la tabla de rutas.
<details><summary>▶ Respuesta</summary>

**Falso.** Es la **última**: se usa cuando no coincide ninguna otra (ruta por defecto).
</details>

**B17.** La Longitud Total de la cabecera IP incluye cabecera y datos.
<details><summary>▶ Respuesta</summary>

**Verdadero.** Mide todo el datagrama en bytes (máx. 65535).
</details>

**B18.** La SVT de ICMP es una suma de control solo de la cabecera ICMP.
<details><summary>▶ Respuesta</summary>

**Falso.** Cubre **todo el paquete ICMP** (cabecera + datos).
</details>

**B19.** Los routers intermedios reensamblan los fragmentos antes de reenviarlos.
<details><summary>▶ Respuesta</summary>

**Falso.** Solo reensambla el **destino final**. Por eso, en el ping a 10.3.0.2, a tu PC llegan fragmentos del tamaño del PaP.
</details>

**B20.** Un paquete IP con IP origen 0.0.0.0 puede generar errores ICMP.
<details><summary>▶ Respuesta</summary>

**Falso.** Si la IP origen no identifica a una única máquina (0, broadcast o de grupo), no se generan errores.
</details>

**B21.** El conmutador da más privacidad que el concentrador.
<details><summary>▶ Respuesta</summary>

**Verdadero.** Solo entrega cada trama al puerto destino; los demás no pueden capturarla.
</details>

**B22.** El ARP Reply se envía en broadcast.
<details><summary>▶ Respuesta</summary>

**Falso.** Va en **unicast**, a la MAC del que preguntó.
</details>

**B23.** Para el examen, de las 4 subredes /26 de 192.168.100.0/24 se puede usar la 192.168.100.0/26.
<details><summary>▶ Respuesta</summary>

**Verdadero (según la práctica).** La práctica lista las 4 combinaciones (00, 01, 10, 11) como subredes válidas.
</details>

**B24.** Un error ICMP 3/1 siempre llega al origen cuando el host destino está apagado.
<details><summary>▶ Respuesta</summary>

**Falso.** Muchos routers deshabilitan el 3/1 para evitar que un ciberdelincuente provoque congestión → a menudo solo hay "tiempo de espera agotado".
</details>

---

## BLOQUE C — Respuesta corta / desarrollo

**C1.** Explica las diferencias entre hub y switch (funcionamiento, colisiones, seguridad, captura).
<details><summary>▶ Respuesta</summary>

- **Hub**: bus común; repite cada trama por todos los puertos → si dos transmiten a la vez hay **colisión** y hay que retransmitir; con un monitor se captura el tráfico de **todos**; poca privacidad.
- **Switch**: malla de interconexión; envía cada trama **solo al puerto del destinatario** → permite **transmisiones simultáneas** entre pares distintos, evita muchas colisiones, **más seguridad** y mejor aprovechamiento del medio; el monitor solo ve el tráfico **propio** (+ broadcast).
</details>

**C2.** Describe cómo funciona el acceso al medio en la Ethernet original y su problema.
<details><summary>▶ Respuesta</summary>

El equipo escucha el canal; si está libre transmite. Si dos transmiten a la vez, el emisor detecta la **colisión** (señal de interferencia), espera un **tiempo aleatorio** y retransmite (es poco probable que vuelva a colisionar). Va bien con poco tráfico y pocos equipos (< ~25); con muchos, la red entra en **colisión permanente**. Se resolvió con la Ethernet **conmutada**.
</details>

**C3.** ¿Qué es la MTU y de qué depende? ¿Qué ocurre si un datagrama la supera?
<details><summary>▶ Respuesta</summary>

Es el tamaño máximo de datos (el datagrama IP) que transporta una trama del nivel de enlace (1500 en DIX, 1492 en 802.3). Depende del tipo de enlace: **BER (tasa de error)**, **política de acceso al medio**, **velocidad**... Si un datagrama la supera, **IP lo fragmenta** (salvo que DF = 1: entonces se descarta y se envía ICMP 3/4).
</details>

**C4.** ¿Por qué es necesario ARP y por qué existe la caché ARP?
<details><summary>▶ Respuesta</summary>

Para enviar por Ethernet hace falta la **MAC destino**, pero en Internet se indica la **IP destino** → hace falta asociar dinámicamente IP → MAC (ARP, RFC 826). La **caché** evita hacer ARP antes de cada datagrama (más tráfico que impediría transmitir a otros). Las entradas **expiran** para reaprender automáticamente si un equipo cambia de IP o MAC.
</details>

**C5.** Describe la secuencia completa de paquetes de un ping entre dos equipos de redes distintas unidas por un router (cachés vacías).
<details><summary>▶ Respuesta</summary>

Red A: PC1 (MAC1), router (MAC4). Red B: router (MAC5), PC2 (MAC6).
1. MAC1→FF:FF… ARP Request ¿IP router A?
2. MAC4→MAC1 ARP Reply
3. MAC1→MAC4 | PC1→PC2 | Echo Request
4. MAC5→FF:FF… ARP Request ¿IP PC2?
5. MAC6→MAC5 ARP Reply
6. MAC5→MAC6 | PC1→PC2 | Echo Request
7. MAC6→MAC5 | PC2→PC1 | Echo Reply (sin ARP: lo aprendió del Request)
8. MAC4→MAC1 | PC2→PC1 | Echo Reply

Las IP no cambian; las MAC sí en cada segmento.
</details>

**C6.** ¿Cuáles son los parámetros de configuración de un equipo TCP/IP? ¿Qué pasa si falta la puerta de enlace?
<details><summary>▶ Respuesta</summary>

**IP, máscara, puerta de enlace por defecto** y **servidores DNS**. Sin puerta de enlace solo se puede comunicar dentro de **su propia red IP**.
</details>

**C7.** ¿Para qué sirven la dirección de red y la de broadcast? ¿Por qué no se asignan a equipos?
<details><summary>▶ Respuesta</summary>

- **Red** (bits de máquina a 0): identifica la red; la usan los **routers** en las rutas.
- **Broadcast** (bits de máquina a 1): un datagrama dirigido a ella lo procesan **todos** los equipos de esa red.

Están reservadas → equipos por red = 2^h − 2.
</details>

**C8.** Explica los campos Identificación, Flags y Fragment Offset.
<details><summary>▶ Respuesta</summary>

- **Identificación (16 b)**: marca cada datagrama de un equipo (se incrementa en cada envío); se **copia en todos los fragmentos**.
- **Flags (3 b)**: bit 1 **DF** (no fragmentar; si hace falta, se descarta y se envía ICMP 3/4), bit 2 **MF** (1 en todos los fragmentos menos el último).
- **Fragment Offset (13 b)**: posición de los datos del fragmento en el original, **en unidades de 8 bytes**. Hace falta porque los fragmentos pueden llegar desordenados.

Además, se actualiza la **Longitud Total** de cada fragmento.
</details>

**C9.** ¿Por qué la suma de control IP solo cubre la cabecera?
<details><summary>▶ Respuesta</summary>

Porque los routers procesan muchísimos paquetes y en cada uno normalmente solo cambia el **TTL** → la suma se puede recalcular de forma **incremental**, lo que reduce mucho el tiempo de proceso.
</details>

**C10.** ¿Para qué sirve el TTL? ¿Qué pasa al llegar a 0?
<details><summary>▶ Respuesta</summary>

Evita que un datagrama quede atrapado en un **bucle** (por ejemplo, por problemas de encaminamiento). El emisor lo inicializa y cada router lo decrementa en 1; al llegar a **0** el router **elimina** el datagrama y envía **ICMP 11/0** al origen.
</details>

**C11.** Diferencia entre mensajes ICMP de solicitud y de error. Pon ejemplos.
<details><summary>▶ Respuesta</summary>

- **Solicitud / información**: piden o informan de algo que no es un error (Echo Request/Reply, aviso de router, timestamp).
- **Error**: informan de fallos en el envío de paquetes IP (TTL = 0 → 11/0; máquina inaccesible → 3/1; fragmentación necesaria con DF → 3/4…). Llevan la **cabecera IP + 8 bytes** del paquete que los causó, y **nunca se generan por otro error**.
</details>

**C12.** Enumera las situaciones que NO generan error ICMP y di por qué existen estas reglas.
<details><summary>▶ Respuesta</summary>

1) Un mensaje de error ICMP. 2) Un paquete a una dirección IP de broadcast. 3) Un paquete enviado como broadcast de enlace. 4) Un fragmento que no sea el primero (salvo 11/1). 5) Un fragmento fuera de secuencia. 6) Un paquete cuya IP origen no es de una única máquina (0, broadcast, grupo).
Motivo: evitar las **tormentas de broadcast**.
</details>

**C13.** ¿Por qué la fragmentación es perjudicial hoy en día y en el pasado no?
<details><summary>▶ Respuesta</summary>

El tiempo de transmisión apenas cambia, pero el **tiempo de encaminamiento** se multiplica: el router gasta el mismo tiempo de CPU por paquete, sea cual sea su tamaño → más retardo y **congestión**. Antes (redes de ~100 KBps con muchos errores) compensaba, porque si fallaba un trozo solo se reenviaba ese trozo. Hoy las redes son rápidas (cientos de Mbps) y fiables, así que no aporta ventaja. IPv6 no permite fragmentar en los routers.
</details>

**C14.** Explica paso a paso cuándo y cómo se genera un ICMP Redirect.
<details><summary>▶ Respuesta</summary>

1) PC1 envía un paquete a un destino exterior a su puerta de enlace (Router 1). 2) Router 1 consulta su tabla: el siguiente salto es Router 2 y la interfaz de salida es **la misma por la que entró**. 3) Envía un **Redirect (tipo 5)** a PC1 con la IP de Router 2 para que los próximos paquetes a ese destino vayan directamente a Router 2, **y reenvía el paquete** a Router 2. Condición: la IP origen debe pertenecer a la **misma red** que la interfaz del router.
</details>

**C15.** ¿Cómo funciona tracert?
<details><summary>▶ Respuesta</summary>

Envía Echo Request con **TTL = 1**: el primer router lo descarta y responde **11/0** → ese es el salto 1. Luego TTL = 2, 3… hasta recibir un **Echo Reply** del destino. Envía **3 paquetes por salto** (3 tiempos) y llega como máximo a 30 saltos; `-d` evita las consultas DNS.
</details>

**C16.** ¿Para qué es útil el mensaje 11/1?
<details><summary>▶ Respuesta</summary>

Lo envía el **destino** cuando no recibe a tiempo todos los fragmentos para reensamblar. Permite saber si el problema es **congestión** (los fragmentos llegan, pero tarde, y no se pueden reensamblar → el usuario cree que no hay comunicación) o si la información **realmente no llega**.
</details>

**C17.** Explica el funcionamiento de un router con colas y el mensaje Source Quench.
<details><summary>▶ Respuesta</summary>

Los paquetes entran en la **cola de entrada**; el router decide el encaminamiento y los pasa a la **cola de salida**. Si no procesa tan rápido como llegan, la cola crece; cuando se **llena**, descarta los paquetes sobrantes y envía **Source Quench (tipo 4)** al origen para que reduzca el flujo. Es el único control de congestión de IP.
</details>

**C18.** Interpreta esta tabla de rutas:
```
172.20.43.192  255.255.255.192  En vínculo      172.20.43.200
0.0.0.0        0.0.0.0          172.20.43.240   172.20.43.200
```
<details><summary>▶ Respuesta</summary>

- 1ª: la red propia 172.20.43.192/26 es de **entrega directa** ("En vínculo") → ARP busca la MAC del **destinatario**.
- 2ª: **ruta por defecto** (cualquier destino, se comprueba la última) → se envía a la puerta de enlace **172.20.43.240** (R2) → ARP busca la MAC de **R2**.
</details>

**C19.** ¿Por qué en la Cuestión 5 de la P2 (ping sin -f a 10.3.0.2) el tamaño de los fragmentos recibidos no coincide con el MTU que informó el 3/4?
<details><summary>▶ Respuesta</summary>

Porque el **Fragment Offset** se expresa en unidades de **8 bytes**: los datos de cada fragmento (salvo el último) deben ser **múltiplo de 8**. Si MTU − 20 no es múltiplo de 8, se redondea hacia abajo → el paquete IP es algo **menor** que el MTU. Ej.: MTU 576 → 556 → 552 → paquetes de 572.
</details>

**C20.** ¿Por qué la práctica usa `ping -l 472` para medir retardos?
<details><summary>▶ Respuesta</summary>

Para no fragmentar: en el laboratorio hay MTU distintos y se toma uno pequeño de **500 bytes**. 500 − 20 (IP) − 8 (ICMP) = **472** bytes de datos. Trama Ethernet = 514 bytes.
</details>

---

## BLOQUE D — Cálculos

### D.1 Tamaños de paquete

**D1.** `ping -l 300 172.20.43.241`. Tamaño del mensaje ICMP, del datagrama IP y de la trama en Wireshark.
<details><summary>▶ Respuesta</summary>

ICMP = 300 + 8 = **308**; IP = 308 + 20 = **328**; trama = 328 + 14 = **342** (346 con CRC).
</details>

**D2.** ¿Cuál es el mayor valor de `-l` que se puede enviar por Ethernet sin fragmentar?
<details><summary>▶ Respuesta</summary>

1500 − 20 − 8 = **1472**.
</details>

**D3.** ¿Y el mayor `-l` sin fragmentar con MTU 500? ¿Y con MTU 576?
<details><summary>▶ Respuesta</summary>

500 − 28 = **472**; 576 − 28 = **548**.
</details>

**D4.** En Wireshark ves un Echo Request de 1042 bytes de trama. ¿Qué valor de `-l` se usó?
<details><summary>▶ Respuesta</summary>

1042 − 14 − 20 − 8 = **1000**.
</details>

**D5.** Un ping por defecto en Windows. ¿Tamaño de IP y de trama?
<details><summary>▶ Respuesta</summary>

32 B de datos → ICMP 40 → IP **60** → trama **74**. Por defecto envía 4.
</details>

### D.2 Fragmentación (Ethernet, MTU 1500 → 1480 de datos por fragmento)

**D6.** `ping -n 1 -l 3000`. Indica fragmentos, Longitud Total, Offset (bytes y campo) y MF.
<details><summary>▶ Respuesta</summary>

ICMP = 3008 = 1480 + 1480 + 48.

| Frag | Long. Total | Offset B | Campo FO | MF |
|---|---|---|---|---|
| 1 | 1500 | 0 | 0 | 1 |
| 2 | 1500 | 1480 | 185 | 1 |
| 3 | 68 | 2960 | 370 | 0 |
</details>

**D7.** `ping -n 1 -l 4500 172.20.43.242` (Cuestión de la P2).
<details><summary>▶ Respuesta</summary>

ICMP = 4508 = 3·1480 + 68 → **4 paquetes**:

| Frag | Long. Total | Offset B | FO | DF | MF |
|---|---|---|---|---|---|
| 1 | 1500 | 0 | 0 | 0 | 1 |
| 2 | 1500 | 1480 | 185 | 0 | 1 |
| 3 | 1500 | 2960 | 370 | 0 | 1 |
| 4 | 88 | 4440 | 555 | 0 | 0 |
</details>

**D8.** `ping -n 1 -l 1473`. ¿Se fragmenta? ¿Cómo?
<details><summary>▶ Respuesta</summary>

ICMP = 1481 > 1480 → **sí**: fragmento 1 = 1480 datos (IP 1500, MF 1, FO 0); fragmento 2 = **1 byte** de datos (IP **21**, MF 0, FO 185).
</details>

**D9.** `ping -n 1 -l 5000`.
<details><summary>▶ Respuesta</summary>

ICMP = 5008 = 3·1480 (4440) + 568 → 4 fragmentos: IP 1500, 1500, 1500, **588**; FO 0, 185, 370, **555**; MF 1, 1, 1, 0.
</details>

**D10.** Capturas un paquete IP con Longitud Total = 548, MF = 0, Fragment Offset = 185. Interprétalo.
<details><summary>▶ Respuesta</summary>

Es el **último fragmento** (MF = 0 y offset > 0). Lleva 548 − 20 = **528 bytes** que empiezan en el byte 185·8 = **1480** del original. El mensaje original tenía 1480 + 528 = **2008 bytes** de ICMP → es un `ping -l 2000`.
</details>

**D11.** Fragmenta `ping -l 1000` (ICMP 1008) para un enlace con **MTU 576**.
<details><summary>▶ Respuesta</summary>

576 − 20 = 556 → no es múltiplo de 8 → **552** por fragmento. 1008 = 552 + 456.
Frag 1: IP **572**, FO 0, MF 1 · Frag 2: IP **476**, FO 552/8 = **69**, MF 0.
⚠️ El paquete mayor (572) es menor que el MTU (576).
</details>

**D12.** Fragmenta `ping -l 2000` (ICMP 2008) para un enlace con **MTU 1006**.
<details><summary>▶ Respuesta</summary>

1006 − 20 = 986 → múltiplo de 8 inferior: **984** (123·8). 2008 = 984 + 984 + 40.
IP: **1004** (FO 0, MF 1), **1004** (FO 123, MF 1), **60** (FO 246, MF 0).
</details>

**D13.** ¿Cuántos paquetes con MF = 1 genera `ping -l 1600 172.20.43.240` (4 pings por defecto) contando ida y vuelta?
<details><summary>▶ Respuesta</summary>

Cada Echo (1608 B) → 2 fragmentos (1500 con MF = 1 y 148 con MF = 0). 4 Request + 4 Reply = 8 mensajes → **8 paquetes con MF = 1** (y 8 últimos fragmentos con MF = 0).
</details>

**D14.** Calcula el tiempo para enviar 10.000 bits entre dos redes de **10 Mbps** unidas por un router (Tenc = 1 µs/paquete, sin cabeceras): a) en 1 paquete; b) en 10 paquetes de 1000 bits.
<details><summary>▶ Respuesta</summary>

a) Ttr = 10⁴/10⁷ = 1 ms → T = 1 + 0,001 + 1 = **2,001 ms**.
b) Ttr = 10³/10⁷ = 0,1 ms → T = 10·0,1 + 10·0,001 + 10·0,1 = 1 + 0,01 + 1 = **2,01 ms**.
</details>

**D15.** Repite el ejemplo de la práctica: 1000 bits a 1 Mbps, Tenc = 1 µs: 1 paquete vs 100 paquetes de 10 bits.
<details><summary>▶ Respuesta</summary>

1 paquete: 1 + 0,001 + 1 = **2,001 ms**. 100 paquetes: 100·0,01 + 100·0,001 + 100·0,01 = 1 + 0,1 + 1 = **2,1 ms**.
</details>

### D.3 Direccionamiento y subredes

**D16.** Clase y máscara por defecto de: 10.4.0.1 · 172.20.43.240 · 193.145.233.5 · 224.0.0.5 · 130.130.0.0 · 150.22.0.0
<details><summary>▶ Respuesta</summary>

10.4.0.1 → **A** (/8) · 172.20.43.240 → **B** (/16) · 193.145.233.5 → **C** (/24) · 224.0.0.5 → **D** (multicast, sin máscara) · 130.130.0.0 → **B** · 150.22.0.0 → **B**.
</details>

**D17.** Para 172.20.43.200/26 da: máscara, dirección de red, broadcast, rango de equipos y nº de equipos.
<details><summary>▶ Respuesta</summary>

Máscara **255.255.255.192**; .200 cae en el bloque 192–255 → red **172.20.43.192**, broadcast **172.20.43.255**, equipos **.193 – .254**, **62** equipos.
</details>

**D18.** ¿Pertenece 172.20.43.190 a la red de los alumnos?
<details><summary>▶ Respuesta</summary>

**No.** Con /26, .190 está en la subred 172.20.43.128/26 (.129–.190). La red de alumnos empieza en .193.
</details>

**D19.** Red 172.18.41.0/28: máscara, rango, broadcast, nº de equipos.
<details><summary>▶ Respuesta</summary>

255.255.255.**240**; equipos .1 – .14; broadcast **.15**; 2^4 − 2 = **14**.
</details>

**D20.** ¿Están 10.1.10.1 y 10.2.10.3 en la misma red con máscara 255.255.0.0? ¿Y con 255.0.0.0?
<details><summary>▶ Respuesta</summary>

Con /16: **no** (10.1 ≠ 10.2) → habría que ir por la puerta de enlace. Con /8: **sí** (los dos son 10.x).
</details>

**D21.** 130.130.0.0/16 en 7 subredes (P1).
<details><summary>▶ Respuesta</summary>

**3 bits** (2³ = 8) → **/19**, **255.255.224.0**, salto 32 en el 3er octeto, 8190 equipos/subred:
0.0 (0.1–31.254, bc 31.255) · 32.0 (32.1–63.254, bc 63.255) · 64.0 (…95.255) · 96.0 (…127.255) · 128.0 (…159.255) · 160.0 (…191.255) · 192.0 (…223.255) · 224.0 (224.1–255.254, bc 255.255). Todas con prefijo 130.130.
</details>

**D22.** 150.22.0.0/23 en 5 subredes (P1).
<details><summary>▶ Respuesta</summary>

**3 bits** → **/26**, **255.255.255.192**, 62 equipos/subred:
150.22.0.0 (.0.1–.0.62, bc .0.63) · 0.64 (.65–.126, bc .127) · 0.128 (.129–.190, bc .191) · 0.192 (.193–.254, bc .0.255) · 1.0 (.1.1–.1.62, bc .1.63) · 1.64 (.65–.126, bc .127) · 1.128 (.129–.190, bc .191) · 1.192 (.193–.254, bc .1.255).
</details>

**D23.** 192.168.10.0/24 en 5 subredes.
<details><summary>▶ Respuesta</summary>

3 bits → **/27**, **255.255.255.224**, salto 32, **30** equipos:
.0 (1–30, bc 31) · .32 (33–62, 63) · .64 (65–94, 95) · .96 (97–126, 127) · .128 (129–158, 159) · .160 (161–190, 191) · .192 (193–222, 223) · .224 (225–254, 255).
</details>

**D24.** 10.0.0.0/8 en 3 subredes.
<details><summary>▶ Respuesta</summary>

2 bits → **/10**, **255.192.0.0**, salto 64 en el 2º octeto:
10.0.0.0 (10.0.0.1 – 10.63.255.254, bc 10.63.255.255) · 10.64.0.0 (… 10.127.255.255) · 10.128.0.0 (… 10.191.255.255) · 10.192.0.0 (… 10.255.255.255). 2^22 − 2 = 4.194.302 equipos/subred.
</details>

**D25.** 172.16.0.0/16 en 16 subredes. Da la máscara y las dos primeras subredes.
<details><summary>▶ Respuesta</summary>

4 bits → **/20**, **255.255.240.0**, salto 16 en el 3er octeto, 4094 equipos.
172.16.0.0/20: 172.16.0.1 – 172.16.15.254, bc 172.16.15.255.
172.16.16.0/20: 172.16.16.1 – 172.16.31.254, bc 172.16.31.255.
</details>

**D26.** ¿En cuántos bits amplías la máscara para 2, 3, 4, 5, 8, 9 y 17 subredes?
<details><summary>▶ Respuesta</summary>

2 → 1 · 3 → 2 · 4 → 2 · 5 → 3 · 8 → 3 · 9 → 4 · 17 → 5 (el menor n con 2^n ≥ subredes).
</details>

**D27.** Convierte a prefijo: 255.255.255.224 · 255.255.240.0 · 255.255.254.0 · 255.192.0.0
<details><summary>▶ Respuesta</summary>

/27 · /20 · /23 · /10.
</details>

### D.4 Rendimiento

**D28.** Retardos (ms): 20, 22, 18, 20. Calcula media, S y S(%). ¿Buen rendimiento?
<details><summary>▶ Respuesta</summary>

Media = 80/4 = **20**. Σ(20 − rᵢ)² = 0 + 4 + 4 + 0 = 8 → S = √(8/3) = √2,67 = **1,63 ms** → S% = **8,2 %** → **buen rendimiento** (< 50 %).
</details>

**D29.** Retardos (ms): 5, 5, 5, 25.
<details><summary>▶ Respuesta</summary>

Media = 40/4 = **10**. Σ = 25 + 25 + 25 + 225 = 300 → S = √(300/3) = √100 = **10 ms** → S% = **100 %** → **rendimiento bajo** (picos de utilización).
</details>

**D30.** ¿Cuántas muestras se toman en la práctica y por qué se divide entre n − 1?
<details><summary>▶ Respuesta</summary>

Una cada 15 s durante 15 min → **n = 60**. Se divide entre **n − 1** porque es la desviación típica **muestral** (es la fórmula que da la práctica).
</details>

---

## BLOQUE E — Escenarios sobre la topología del laboratorio

**E1.** Desde un PC (172.20.43.200) con la caché ARP vacía: `ping -n 1 172.20.43.242`. Escribe la secuencia de paquetes.
<details><summary>▶ Respuesta</summary>

.242 (R5) está en la misma red → entrega directa:
1. MAC_PC → FF:FF… ARP Request ¿172.20.43.242?
2. MAC_R5 → MAC_PC ARP Reply
3. MAC_PC → MAC_R5 | .200 → .242 | Echo Request
4. MAC_R5 → MAC_PC | .242 → .200 | Echo Reply
</details>

**E2.** Desde el PC, caché vacía: `ping -n 1 10.6.0.5`. ¿Por qué IP pregunta ARP? ¿Qué MAC e IP de destino lleva el Echo Request?
<details><summary>▶ Respuesta</summary>

10.6.0.5 no está en 172.20.43.192/26 → ruta por defecto → ARP pregunta por **172.20.43.240** (R2). Echo Request: **MAC destino = R2**, **IP destino = 10.6.0.5**.
</details>

**E3.** `ping -i 1 -n 1 10.6.0.5`: ¿quién envía el error y cuál?
<details><summary>▶ Respuesta</summary>

**R2 (172.20.43.240)** envía **ICMP 11/0 TTL exceeded in transit**: es el primer router y el TTL pasa de 1 a 0.
</details>

**E4.** `ping -i 2 -n 1 10.3.0.2`: ¿quién envía el error? ¿Qué MAC origen ves en la trama?
<details><summary>▶ Respuesta</summary>

**R1** (IP 10.4.0.2), segundo salto. La trama te llega desde el **router de tu red que la entrega** (p.ej. R2), así que la MAC origen es la de ese router, **no la de R1**. La IP origen sí es la de R1.
</details>

**E5.** En el tracert a 10.3.0.2, el salto 2 aparece como 10.4.0.2 y no como 10.3.0.1. ¿Por qué? ¿Y por qué el salto 3 tarda tanto más?
<details><summary>▶ Respuesta</summary>

R1 responde con la IP de la interfaz por la que entra/sale hacia nosotros (la de la red 10.4.0.0). El salto 3 (R4) atraviesa el **enlace PaP de 10 Mbps**, mucho más lento que los Ethernet de 100 Mbps, y además hay más saltos (ida y vuelta).
</details>

**E6.** `ping -n 1 -l 1200 -f 172.20.43.241`. ¿Habrá ICMP 3/4?
<details><summary>▶ Respuesta</summary>

**No.** IP = 1228 < 1500 y es entrega directa en Ethernet → llega sin fragmentar → Echo Reply normal. (El 3/4 de la práctica sale hacia **10.3.0.2** porque el paquete atraviesa el **PaP**, de MTU menor.)
</details>

**E7.** `ping -n 1 -l 1500 -f 172.20.43.241`. ¿Qué pasa?
<details><summary>▶ Respuesta</summary>

IP = 1528 > 1500 con DF = 1 → **tu propio PC** no puede enviarlo ni fragmentarlo: Windows avisa de que hace falta fragmentar pero DF está activo y **no sale ningún paquete** a la red (no hay router implicado).
</details>

**E8.** `ping -n 1 -l 1200 -f 10.3.0.2`: ¿quién envía el 3/4 y dónde ves el MTU?
<details><summary>▶ Respuesta</summary>

**R1**, que no puede pasar el paquete (1228 B, DF = 1) por el **PaP 10.3.0.0/24**. El MTU del siguiente salto va en la **cabecera ICMP** del 3/4 (Wireshark: "MTU of next hop").
</details>

**E9.** `ping -n 1 -l 1000 10.3.0.2`. ¿Fragmenta tu PC? ¿Llegan fragmentos?
<details><summary>▶ Respuesta</summary>

Tu PC **no** fragmenta (1028 < 1500). R1 fragmenta la petición para el PaP. R4 genera la respuesta y la fragmenta para salir por el PaP; como **los routers no reensamblan**, a tu PC le llegan **varios fragmentos** del Echo Reply, y su tamaño revela el MTU del PaP.
</details>

**E10.** Ejecutas `route add 10.3.0.0 mask 255.255.255.0 172.20.43.241` y luego `ping 10.3.0.1`. ¿Por qué IP pregunta ARP?
<details><summary>▶ Respuesta</summary>

Por **172.20.43.241** (R3): la nueva ruta para 10.3.0.0/24 es más específica que la ruta por defecto.
</details>

**E11.** `route add 172.18.41.0 mask 255.255.255.0 172.20.43.242` y `ping -n 1 172.18.41.4`. ¿Qué pasa? ¿Cuántos paquetes capturas en tu PC?
<details><summary>▶ Respuesta</summary>

Echo Request a **R5** → R5 ve que el siguiente salto es otro router de la misma red (sale por la misma interfaz) → envía un **ICMP Redirect** a tu PC (con la IP del router correcto, p.ej. 172.20.43.240) y **reenvía** el Echo Request a ese router → el Echo Reply te llega por ese router.
En tu PC capturas **3** (Echo Request, Redirect y Echo Reply, más ARP si hace falta); el reenvío R5 → R2 no lo ves por el **switch**. En el segmento hay 4 implicados.
</details>

**E12.** Tras el Redirect, repites el ping. ¿Qué cambia?
<details><summary>▶ Respuesta</summary>

No hay Redirect y el Echo Request va **directamente** al router indicado en el Redirect. Si reaparece, es que expiró el **temporizador** de Windows 10 (no se guarda en la tabla de rutas).
</details>

**E13.** `ping -n 1 172.20.43.250`. ¿Qué capturas y qué concluyes?
<details><summary>▶ Respuesta</summary>

Varios **ARP Request sin respuesta** y **ningún** Echo Request (sin MAC no se puede formar la trama). Tu propio PC informa de "host de destino inaccesible". **.250 no existe o está apagado.**
</details>

**E14.** `ping -n 1 10.6.0.25`. ¿Hay ARP? ¿Hay Echo Reply?
<details><summary>▶ Respuesta</summary>

ARP **solo para la puerta de enlace** y solo si no está en caché (nunca para 10.6.0.25). **No hay Echo Reply**: no existe; el router de 10.6.0.0 fallará en su ARP (que tú no ves) y puede enviar 3/1 o, si lo tiene deshabilitado, nada (tiempo agotado).
</details>

**E15.** `ping 172.20.43.255`. ¿MAC destino? ¿Quién contesta?
<details><summary>▶ Respuesta</summary>

Es el **broadcast** de 172.20.43.192/26 → MAC destino **FF:FF:FF:FF:FF:FF** (sin ARP). Lo reciben todos los equipos del segmento y contestan los que están configurados para responder a Echo en broadcast (normalmente los routers R2, R3 y R5; los PCs Windows no suelen hacerlo).
</details>

**E16.** `ping -n 1 -i 1 10.4.0.255` vs `ping -n 1 -i 1 10.6.0.255`. ¿Cuál genera Time Exceeded?
<details><summary>▶ Respuesta</summary>

Ambos llegan a R2 con TTL 1. **10.4.0.255** es el broadcast de una red **conectada a R2** → R2 lo reconoce como broadcast → **no genera error**. **10.6.0.255** no es una red de R2 → para él es una dirección normal → **11/0**. Regla: no se generan errores ICMP para paquetes dirigidos a una dirección de broadcast.
</details>

**E17.** `tracert -d 10.3.0.10`. Explica el resultado.
<details><summary>▶ Respuesta</summary>

10.3.0.10 **no existe** (el PaP solo tiene .1 y .2). R1 lo envía por el PaP a R4; R4 ve que es de la red 10.3.0.0/24 pero no es suyo y lo devuelve por el PaP a R1… **Bucle de encaminamiento entre R1 y R4**: tracert muestra R1 y R4 alternándose hasta los 30 saltos. El **TTL** evita que el paquete circule indefinidamente.
</details>

**E18.** Ping a 10.4.0.3, 10.6.0.3 y 10.4.0.1: ¿de qué router llega el Echo Reply y por qué?
<details><summary>▶ Respuesta</summary>

10.4.0.3 → **R3**; 10.6.0.3 → **R5**; 10.4.0.1 → **R2**. Cada uno es una interfaz de un router que **también tiene interfaz en tu red**, así que ese router contesta y te entrega la respuesta directamente. (Se comprueba con la MAC origen del Echo Reply y `arp -a`.)
</details>

**E19.** Ruta a 10.6.0.0/24 por R3 y pings a 10.6.0.5 y 10.6.0.6. ¿Puede volver la respuesta por routers distintos? ¿Por qué?
<details><summary>▶ Respuesta</summary>

Sí. La **ida** la decide tu tabla (ambos por R3), pero la **vuelta** la decide la puerta de enlace de cada Linux (Linux2 y Linux3 pueden tener puertas distintas: R3, R4 o R5) → **encaminamiento asimétrico**.
</details>

**E20.** El compañero borra tu entrada ARP y tú le haces ping. ¿Hay ARP? ¿Antes o después del Echo Request?
<details><summary>▶ Respuesta</summary>

Sí, **después**: tú tienes su MAC y envías el Echo Request directamente; él no tiene la tuya y, para responder, primero envía un **ARP Request** (broadcast) por tu IP; respondes con un ARP Reply y entonces llega el Echo Reply.
</details>

**E21.** `ping -n 1 172.18.41.2` devuelve 3/13, pero `ping -n 1 172.18.41.1` no. ¿Qué filtrado hace el router?
<details><summary>▶ Respuesta</summary>

**No filtra todo ICMP**: filtra **según la IP de destino** (reglas que permiten llegar a ciertos equipos, como 172.18.41.1, y prohíben el resto). El 3/13 lo envía el router que filtra (el de acceso a 172.18.41.0/28, R1).
</details>

**E22.** ¿Puedes ver con Wireshark el ARP Request que hace otro alumno? ¿Y su Echo Request a R3?
<details><summary>▶ Respuesta</summary>

El **ARP Request sí** (es broadcast y el switch lo reparte a todos los puertos). Su **Echo Request no** (es unicast, y el switch solo lo envía al puerto de R3).
</details>

---

## BLOQUE F — Comandos y Wireshark

**F1.** Comando para ver MAC, IP, máscara y puerta de enlace en Windows.
<details><summary>▶ Respuesta</summary>

`ipconfig /all`
</details>

**F2.** Ver y borrar la caché ARP.
<details><summary>▶ Respuesta</summary>

`arp -a` (ver) · `arp -d IP` (borrar una entrada).
</details>

**F3.** Ver la tabla de rutas.
<details><summary>▶ Respuesta</summary>

`netstat -rn`
</details>

**F4.** Añade una ruta para que todo lo que vaya a 10.6.0.0/24 pase por R3. Añade otra para que solo el host 10.6.0.3 vaya por R5. Bórralas.
<details><summary>▶ Respuesta</summary>

```
route ADD 10.6.0.0 MASK 255.255.255.0 172.20.43.241
route ADD 10.6.0.3 MASK 255.255.255.255 172.20.43.242
route DELETE 10.6.0.0
route DELETE 10.6.0.3
```
</details>

**F5.** Envía 1 Echo Request de 1000 bytes de datos, con TTL 3 y DF activado, a 10.3.0.2.
<details><summary>▶ Respuesta</summary>

`ping -n 1 -l 1000 -i 3 -f 10.3.0.2`
</details>

**F6.** Restaura la configuración de red inicial del PC del laboratorio.
<details><summary>▶ Respuesta</summary>

`route delete 0.0.0.0` → `ipconfig /release` → `ipconfig /renew` (y `c:\pracredes` para volver a crear la ruta por defecto hacia 172.20.43.240).
</details>

**F7.** Filtros de Wireshark para: broadcast, no IP, enviados por mi IP, UDP, TTL = 1, MF activo, ICMP Time Exceeded.
<details><summary>▶ Respuesta</summary>

`eth.dst == ff:ff:ff:ff:ff:ff` · `!ip` · `ip.src == MI_IP` · `ip.proto == 17` · `ip.ttl == 1` · `ip.flags.mf == 1` · `icmp.type == 11`
</details>

**F8.** ¿Qué hay que cambiar en Wireshark para ver bien la fragmentación?
<details><summary>▶ Respuesta</summary>

`Edit → Preferences → Protocols → IPv4` → **desactivar "Reassemble fragmented IPv4 datagrams"**.
</details>

**F9.** Filtras `frame contains "abcd"`. ¿Qué paquetes aparecen y qué los genera?
<details><summary>▶ Respuesta</summary>

**ICMP Echo Request/Reply** generados por **ping**: el ping de Windows rellena los datos con "abcdefghijklmnopqrstuvwabcdefghi…".
</details>

**F10.** En la captura de la P1, ¿qué paquetes tienen TTL = 1 y TTL = 2?
<details><summary>▶ Respuesta</summary>

Los Echo Request del **tracert -d 172.18.41.4**: TTL 1 = sonda del salto 1 (responde R2 con 11/0), TTL 2 = sonda del salto 2 (llega a R1, que responde con Echo Reply). (Puede haber también algún multicast local con TTL 1.)
</details>
