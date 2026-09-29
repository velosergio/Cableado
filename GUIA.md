# Guía de estudio: infraestructura de conectividad para un coworking multi-tenant

Esta guía sirve para entender el proyecto a fondo, reproducirlo en Packet Tracer y prepararse para la sustentación. Cada módulo explica un concepto de la asignatura, muestra cómo se aplicó en el proyecto y termina con preguntas de repaso, que tienen la respuesta oculta para que te autoevalúes.

**Cómo usarla:** lee los módulos en orden. En el módulo 7 construyes la red desde cero y en el módulo 9 practicas cómo resolver fallas. Al final encontrarás preguntas tipo sustentación y un glosario.

| Módulo | Tema | Tiempo sugerido |
|---|---|---|
| 1 | Subsistemas del cableado estructurado | 30 min |
| 2 | Medios de transmisión: UTP Cat 6 y fibra OS2 | 30 min |
| 3 | Espacios, canalizaciones y separación eléctrica | 30 min |
| 4 | Administración y etiquetado TIA-606-C | 20 min |
| 5 | Cálculos del diseño físico | 45 min |
| 6 | Diseño lógico: VLAN, troncales, router-on-a-stick, DHCP, NAT y ACL | 60 min |
| 7 | Laboratorio guiado en Packet Tracer | 90 min |
| 8 | Verificación y pruebas | 30 min |
| 9 | Solución de problemas | 30 min |
| 10 | Ejercicios de ampliación | 60 min |
| 11 | Preparación para la sustentación | 30 min |

---

## 0. Mapa del proyecto

```text
                       Internet simulado
                  SRV-INTERNET (8.8.8.8)
                           │
                        [ ISP ] 2911
                           │ 200.10.10.0/30 (cable cruzado)
                     [ R-BORDE ] 2911  ← router-on-a-stick + DHCP + NAT + ACL
                           │ Gi0/1 ── troncal 802.1Q ── Gi1/0/24
 PISO 1 ───────── [ SW-CORE-P1 ] 3650-24PS (rack 1A, administración)
                    │  │  │  │ Gi1/1/1 (SFP)
                    │  │  │  │    ║  backbone de fibra OS2 (troncal 802.1Q)
                    │  │  │  │    ║
 PISO 2 ────────────┼──┼──┼──┼─ [ SW-P2 ] 3650-24PS (gabinete 2A)
                    │  │  │  │         │    │    │
                 ADMIN OF101 OF102 AP-P1  OF201 OF202 AP-P2
                 V10   V20   V30   V60    V40   V50   V60
```

El proyecto tiene dos partes que se complementan:

| Parte | Pregunta que responde | Normas y herramientas |
|---|---|---|
| **Diseño físico** | ¿Por dónde va cada cable, cuánto mide, cómo se etiqueta y cuánto material se necesita? | ANSI/TIA-568, 569, 606, RETIE |
| **Diseño lógico** | ¿Cómo se aíslan las empresas si todas comparten los mismos cables y switches? | VLAN, 802.1Q, ACL, Packet Tracer |

La idea central que debes poder explicar en una frase: **una sola infraestructura física de cableado, varias redes lógicas privadas**.

---

## Módulo 1. Subsistemas del cableado estructurado

El cableado estructurado es un sistema **genérico, normalizado y documentado**: no depende de la aplicación (datos, voz, video o PoE) y se organiza en subsistemas.

| Subsistema (TIA-568) | Qué es | En el proyecto |
|---|---|---|
| Instalaciones de entrada (EF) | Punto donde entra el proveedor al edificio | El ISP entrega su enlace en el rack 1A |
| Cuarto de equipos (ER) | Aloja los equipos principales del edificio | Rack 1A en la oficina de administración (R-BORDE, SW-CORE-P1) |
| Cuarto de telecomunicaciones (TR) | Distribuidor de cada piso | 1A (comparte rack con el ER) y 2A (gabinete de pared, piso 2) |
| Backbone | Une el ER con los TR | Fibra OS2 de 6 hilos entre 1A y 2A |
| Cableado horizontal | Va del TR a cada salida del piso | UTP Cat 6 en estrella, 32 salidas |
| Área de trabajo | Salida (TO), patch cord y equipo del usuario | Faceplates dobles y patch cords de 3 m |

**Topología en estrella:** cada salida tiene su propio cable hasta el patch panel del piso. No hay empalmes, derivaciones ni cadenas de switches bajo los escritorios.

**Límites de distancia del canal horizontal:**

```text
[switch]─patch cord─[patch panel]════ enlace permanente ≤ 90 m ════[TO]─patch cord─[PC]
          └──────────── canal completo ≤ 100 m (incluye ≤ 10 m de cordones) ───────────┘
```

**¿Por qué en el piso 1 no hay backbone?** Porque el cuarto de equipos y el distribuidor del piso 1 son el mismo rack. El backbone solo se necesita entre espacios distintos.

<details><summary><b>Repaso del módulo 1</b></summary>

1. **¿Qué diferencia hay entre un enlace permanente y un canal?**
   El enlace permanente va del patch panel a la salida y mide como máximo 90 m. El canal incluye además los patch cords de ambos extremos y mide como máximo 100 m.
2. **¿Por qué 1A es a la vez cuarto de equipos y cuarto de telecomunicaciones?**
   Porque en el piso 1 los equipos principales y el distribuidor del piso comparten el mismo rack.
3. **¿Qué subsistema conecta 1A con 2A y con qué medio?**
   El backbone, con fibra óptica monomodo OS2 de 6 hilos.

</details>

---

## Módulo 2. Medios de transmisión

### UTP categoría 6 (ANSI/TIA-568.2-D)

- 4 pares trenzados, 100 Ω, especificado hasta **250 MHz**.
- Soporta 1000BASE-T a 100 m. Con 10GBASE-T el alcance se reduce a unos 55 m (para 100 m se necesita Cat 6A).
- Soporta **PoE**, que en el proyecto alimenta los puntos de acceso desde el switch 3650-24PS.
- Se eligió porque cubre gigabit en toda la red horizontal, que en este edificio no supera los 28 m.

### Fibra óptica monomodo OS2

- Núcleo de 9 µm (9/125). Transmite un solo modo de luz, por lo que tiene muy baja dispersión y gran alcance.
- Transceptor **GLC-LH-SMD** (1000BASE-LX/LH, 1310 nm): gigabit hasta 10 km sobre monomodo.
- Conector **LC dúplex**: un hilo transmite y otro recibe.

**¿Por qué fibra, si el backbone mide solo unos 15 m?** Con UTP bastaría por distancia, pero la fibra:

1. Es inmune a la interferencia electromagnética. Viaja por el ducto vertical, cerca de la acometida eléctrica.
2. Permite subir el ancho de banda sin reemplazar el cable (se cambian los SFP, no la fibra).
3. Tiene 4 hilos de reserva para un segundo enlace o para un tercer piso.

En el proyecto, la fibra primero estuvo "proyectada" y luego se **instaló** en la simulación. Para eso hubo que cambiar el switch 3560 por el 3650: en Packet Tracer, los puertos gigabit del 3560 son solo de cobre, mientras que el 3650 tiene ranuras SFP.

<details><summary><b>Repaso del módulo 2</b></summary>

1. **¿Qué significan las siglas LX en 1000BASE-LX?**
   *Long wavelength*: láser de onda larga (1310 nm), apto para fibra monomodo.
2. **¿Cuántos hilos usa el enlace entre pisos y cuántos quedan de reserva?**
   Usa 2 (transmisión y recepción) y quedan 4 de reserva.
3. **¿Por qué el cambio al switch 3650 también beneficia a los puntos de acceso?**
   Porque el 3650-24PS entrega PoE, así que los AP no necesitan un tomacorriente propio.

</details>

---

## Módulo 3. Espacios, canalizaciones y separación eléctrica

### Canalizaciones (ANSI/TIA-569-D)

Regla clave: **la ocupación máxima es el 40 % del área interna** cuando van tres o más cables.

```text
área de un cable Cat 6 (Ø 6,2 mm) = π × (3,1 mm)² ≈ 30,2 mm²
área mínima de la canaleta = (n.º de cables × 30,2 mm²) / 0,40
```

| Tramo | Cables | Canaleta elegida | Ocupación |
|---|---|---|---|
| Salida del rack 1A | 18 | 60×40 mm | 22,6 % |
| Troncal del pasillo, piso 1 | 14 | 60×40 mm | 17,6 % |
| Derivación a cada oficina | 6 | 40×25 mm | 18,1 % |
| Bajante a cada faceplate | 2 | 20×12 mm | 25,2 % |
| Backbone de fibra | 1 | Tubería EMT de 1" | 5,1 % |

### Separación eléctrica (RETIE y valores de referencia de BICSI)

- Los cables de datos y los eléctricos **no comparten canalización**.
- Separación mínima de referencia en recorridos paralelos: **127 mm** con circuitos de menos de 2 kVA en canalización no metálica, **305 mm** con circuitos de 2 a 5 kVA, **1,2 m** de motores y transformadores y **120 mm** de luminarias fluorescentes.
- Los cruces entre datos y electricidad se hacen a **90°**.
- Los racks y el ODF se conectan a la puesta a tierra del edificio.

<details><summary><b>Repaso del módulo 3</b></summary>

1. **¿Cabe el troncal de 14 cables en una canaleta de 40×25 mm?**
   No. Se necesitan 14 × 30,2 / 0,4 ≈ 1057 mm², y la canaleta de 40×25 mm solo tiene unos 1000 mm². Por eso se usó una de 60×40 mm.
2. **¿Por qué se cruzan los cables de datos y los eléctricos a 90°?**
   Porque así se minimiza el acoplamiento electromagnético entre ambos.
3. **¿Qué pasa si se supera el 40 % de ocupación?**
   Los cables se aplastan y se calientan, se degrada su desempeño y no queda espacio para crecer.

</details>

---

## Módulo 4. Administración y etiquetado (ANSI/TIA-606-C)

Se usó un esquema **de clase 2**, adecuado para un edificio con varios cuartos de telecomunicaciones.

| Elemento | Formato | Ejemplo | Lectura del ejemplo |
|---|---|---|---|
| Espacio | `fs` | `2A` | Piso 2, espacio A |
| Patch panel | `fs.a` | `1A.A` | Panel A del espacio 1A |
| Enlace horizontal | `fs.an` | `1A.A05` | Puerto 05 del panel A, en 1A |
| Backbone | `fs1/fs2-n` | `1A/2A-1` | Cable 1 entre 1A y 2A |
| Hilo de fibra | `fs1/fs2-n.h` | `1A/2A-1.03` | Hilo 3 del cable 1 |

**La regla de oro del proyecto:** la salida `1A.A05` llega al **puerto 5 del patch panel**, que se conecta al **puerto Gi1/0/5 del switch**, que pertenece a la **VLAN 20**. Con solo leer la etiqueta de la pared sabes a qué puerto del switch y a qué VLAN corresponde.

Los nombres de los equipos siguen la misma lógica: `OF101-PC1`, `SW-P2`, y descripciones de puerto como `P1-OF101-EMPRESA-A`.

<details><summary><b>Repaso del módulo 4</b></summary>

1. **¿A qué puerto del switch y a qué VLAN corresponde la salida `2A.A12`?**
   Al puerto Gi1/0/12 de SW-P2, en la VLAN 50 (Empresa D, OF202).
2. **¿Dónde se coloca la etiqueta de un enlace horizontal?**
   En ambos extremos del cable, en el puerto del patch panel y en el faceplate.

</details>

---

## Módulo 5. Cálculos del diseño físico

### 5.1 Puntos de red

| Espacio | Fórmula | Puntos |
|---|---|---|
| Oficina | 4 puestos + 2 de reserva | 6 × 4 oficinas = 24 |
| Administración | PC + recepción + impresora + 1 de reserva | 4 |
| Punto de acceso | 1 + 1 de reserva | 2 × 2 = 4 |
| **Total** | | **32** (21 en uso y 11 de reserva) |

### 5.2 Cable UTP: método de la distancia promedio

```text
Lprom = (Lmáx + Lmín) / 2
Lcalc = Lprom × 1,10 + 2,5 m        (10 % de imprevistos + holgura de terminación)
cables por bobina = ⌊305 / Lcalc⌋   (se redondea hacia abajo: un enlace no se puede empalmar)
bobinas = ⌈puntos / cables por bobina⌉
```

**Ejemplo resuelto para el piso 1** (18 puntos, Lmín = 5,4 m, Lmáx = 28 m):

```text
Lprom = (28 + 5,4) / 2   = 16,70 m
Lcalc = 16,70 × 1,1 + 2,5 = 20,87 m
cables por bobina = ⌊305 / 20,87⌋ = ⌊14,61⌋ = 14
bobinas = ⌈18 / 14⌉ = ⌈1,29⌉ = 2
```

El piso 2 (14 puntos, Lmín = 15,6 m) da Lcalc = 26,48 m, 11 cables por bobina y 2 bobinas. **En total se necesitan 4 bobinas (1220 m)** para un consumo estimado de 746 m. La diferencia se debe a que los sobrantes de cada bobina no se pueden unir.

<details><summary><b>Ejercicio 5.A</b>: si el piso 1 tuviera 30 puntos con las mismas distancias, ¿cuántas bobinas necesitaría?</summary>

Con Lcalc = 20,87 m se obtienen 14 cables por bobina, y ⌈30 / 14⌉ = ⌈2,14⌉ = **3 bobinas**.

</details>

<details><summary><b>Ejercicio 5.B</b>: una ruta mide Lmín = 10 m y Lmáx = 85 m. ¿Cumple la norma? ¿Qué cambiarías?</summary>

La salida más lejana (85 m) cumple el límite de 90 m del enlace permanente, pero queda muy justa: después de agregar las holguras de terminación, cualquier desvío en la obra puede superarlo. Además, Lcalc = 47,5 × 1,1 + 2,5 = 54,75 m, lo que da solo 5 cables por bobina. Lo recomendable sería **agregar un cuarto de telecomunicaciones más cercano** para acortar las rutas.

</details>

---

## Módulo 6. Diseño lógico

### 6.1 VLAN

Una VLAN divide un switch en varios switches lógicos: los puertos de una VLAN no ven las tramas de otra. Cada empresa tiene la suya:

| VLAN | Uso | Red |
|---|---|---|
| 10 | Administración | 192.168.10.0/24 |
| 20 / 30 / 40 / 50 | Empresas A, B, C y D | 192.168.20–50.0/24 |
| 60 | WiFi de invitados | 192.168.60.0/24 |
| 99 | Gestión de los switches | 192.168.99.0/24 |

Los puertos sin uso (Gi1/0/17–19 y 22–23 en SW-CORE-P1) están en `shutdown`: es una buena práctica de seguridad.

### 6.2 Troncales 802.1Q

Un enlace troncal transporta varias VLAN por el mismo cable, marcando cada trama con una etiqueta (*tag*) de 4 bytes que indica su VLAN. En el proyecto hay dos troncales: Gi1/0/24 (hacia el router) y Gi1/1/1 (la fibra entre pisos). Por eso los equipos del piso 2 obtienen su dirección por DHCP desde un router que está en el piso 1.

### 6.3 Router-on-a-stick

Las VLAN aíslan en capa 2, pero para salir a Internet cada VLAN necesita una puerta de enlace. R-BORDE usa **una sola interfaz física (Gi0/1) dividida en subinterfaces**, una por VLAN:

```text
interface GigabitEthernet0/1.20
 description VLAN20-EMPRESA-A
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 ip nat inside
 ip access-group AISLAR-VLAN20 in
```

> **Idea clave para la sustentación:** en cuanto el router tiene una subinterfaz en cada VLAN, **puede enrutar entre ellas**. Si no hay ACL, la Empresa A llegaría a la Empresa B pasando por el router. Por eso **las VLAN por sí solas no bastan**: se necesitan las ACL. Este fue uno de los ajustes frente al Momento 1.

### 6.4 DHCP

El router entrega direcciones a cada VLAN. Las direcciones .1 a .10 se reservan para las puertas de enlace y los equipos fijos:

```text
ip dhcp excluded-address 192.168.20.1 192.168.20.10
ip dhcp pool VLAN20-EMPRESA-A
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8
```

La VLAN 99 no tiene pool: los switches usan direcciones estáticas (.2 y .3).

### 6.5 NAT con sobrecarga (PAT)

Todas las redes privadas salen a Internet con **una sola IP pública** (200.10.10.2). El router distingue cada conversación por el número de puerto:

```text
access-list 1 permit 192.168.0.0 0.0.255.255
ip nat inside source list 1 interface GigabitEthernet0/0 overload
ip route 0.0.0.0 0.0.0.0 200.10.10.1
```

Con `show ip nat translations` se ve cómo cada IP privada se traduce a 200.10.10.2 con un puerto distinto.

### 6.6 Listas de control de acceso: la pieza clave del aislamiento

Cada red de empresa y la de invitados tiene una ACL extendida aplicada **de entrada (`in`) en su subinterfaz**, es decir, lo más cerca posible del origen del tráfico:

```text
ip access-list extended AISLAR-VLAN20
 permit ip   192.168.20.0 0.0.0.255 192.168.20.0 0.0.0.255              ! 1
 permit icmp 192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 echo-reply   ! 2
 permit tcp  192.168.20.0 0.0.0.255 192.168.10.0 0.0.0.255 established  ! 3
 deny   ip   any 192.168.0.0 0.0.255.255                                ! 4
 permit ip   any any                                                    ! 5
```

| Línea | Qué hace | Por qué |
|---|---|---|
| 1 | Permite el tráfico dentro de la propia red | Por ejemplo, hacia su puerta de enlace |
| 2 | Permite **responder** a los ping de la administración | La administración puede diagnosticar, pero la empresa no puede iniciar un ping hacia ella |
| 3 | Permite las respuestas TCP de sesiones que abrió la administración | `established` solo acepta segmentos con ACK o RST, es decir, respuestas |
| 4 | Bloquea todo lo demás hacia cualquier red privada 192.168.x.x | Esto es el **aislamiento entre inquilinos** (y hacia la gestión) |
| 5 | Permite el resto | Es la salida a Internet |

**Orden:** una ACL se evalúa de arriba abajo y se detiene en la primera coincidencia. Además, al final hay un `deny any` implícito, por eso la línea 5 es necesaria.

**¿Por qué la administración sí llega a todos?** Porque la subinterfaz de la VLAN 10 no tiene ACL, y las ACL de las empresas dejan pasar las respuestas (líneas 2 y 3).

<details><summary><b>Repaso del módulo 6</b></summary>

1. **Si se borra la línea 4 de AISLAR-VLAN20, ¿qué pasa?**
   La Empresa A podría comunicarse con todas las demás redes privadas, porque la línea 5 lo permitiría. Se perdería el aislamiento.
2. **¿Por qué la ACL se aplica `in` en la subinterfaz y no `out`?**
   Para filtrar el tráfico apenas entra al router desde su origen. Así, una sola ACL por VLAN controla todo lo que esa red envía.
3. **¿Puede la Empresa C hacer ping a 192.168.99.2?**
   No. La línea 4 bloquea 192.168.0.0/16, que incluye la red de gestión.
4. **¿Cuál es la debilidad de `established`?**
   Solo revisa las banderas TCP (ACK o RST), no el estado real de la conexión. Un atacante podría fabricar segmentos con ACK activado. Un firewall con estado (*stateful*) o las ACL reflexivas son más seguros.
5. **¿Por qué la VLAN 60 (invitados) tiene la misma ACL que las empresas?**
   Porque los invitados solo deben tener acceso a Internet, nunca a las redes internas.

</details>

---

## Módulo 7. Laboratorio guiado en Packet Tracer

Objetivo: reconstruir la red desde cero. Los comandos son los de la configuración real del proyecto.

### Paso 1. Dispositivos y cableado

| Enlace | Cable | Puertos |
|---|---|---|
| SRV-INTERNET ↔ ISP | Directo | Fa0 ↔ Gi0/1 |
| ISP ↔ R-BORDE | **Cruzado** | Gi0/0 ↔ Gi0/0 |
| R-BORDE ↔ SW-CORE-P1 | Directo | Gi0/1 ↔ Gi1/0/24 |
| SW-CORE-P1 ↔ SW-P2 | **Fibra** | Gi1/1/1 ↔ Gi1/1/1 |
| Switches ↔ PC y AP | Directo | Según la tabla de puertos |

Antes de conectar la fibra, instala el módulo SFP **GLC-LH-SMD** en Gi1/1/1 de ambos switches (con el equipo apagado).

### Paso 2. VLAN en ambos switches

```text
vlan 10
 name ADMIN
vlan 20
 name EMPRESA-A-OF101
vlan 30
 name EMPRESA-B-OF102
vlan 40
 name EMPRESA-C-OF201
vlan 50
 name EMPRESA-D-OF202
vlan 60
 name WIFI-INVITADOS
vlan 99
 name GESTION
```

Como no se usa VTP, las VLAN se crean **en los dos switches**.

### Paso 3. Puertos de acceso y troncales (SW-CORE-P1)

```text
interface range GigabitEthernet1/0/5 - 10
 description P1-OF101-EMPRESA-A
 switchport mode access
 switchport access vlan 20
interface range GigabitEthernet1/0/17 - 19 , GigabitEthernet1/0/22 - 23
 shutdown
interface GigabitEthernet1/0/24
 description TRONCAL-A-R-BORDE
 switchport mode trunk
interface GigabitEthernet1/1/1
 description BACKBONE-FIBRA-A-SW-P2
 switchport mode trunk
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
ip default-gateway 192.168.99.1
```

Repite la misma lógica para las VLAN 10, 30 y 60, y en SW-P2 para las VLAN 40, 50 y 60 (su SVI de gestión es 192.168.99.3).

### Paso 4. Router R-BORDE

```text
interface GigabitEthernet0/0
 description ENLACE-ISP
 ip address 200.10.10.2 255.255.255.252
 ip nat outside
 no shutdown
interface GigabitEthernet0/1
 no shutdown
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 ip nat inside
! ... repite para 20, 30, 40, 50, 60 y 99 (módulo 6.3)
```

Luego agrega los pools DHCP (6.4), el NAT y la ruta por defecto (6.5) y las ACL (6.6). Guarda todo con:

```text
R-BORDE# copy running-config startup-config
```

> Si al abrir el archivo un equipo muestra el diálogo de configuración inicial, es porque la configuración no se guardó en la NVRAM. Guárdala siempre en cada equipo.

---

## Módulo 8. Verificación y pruebas

| Qué verificar | Comando | Resultado esperado |
|---|---|---|
| VLAN y puertos | `show vlan brief` | Cada puerto en su VLAN |
| Troncales | `show interfaces trunk` | Gi1/0/24 y Gi1/1/1 en estado *trunking* |
| Subinterfaces | `show ip interface brief` | Todas las Gi0/1.x en *up/up* |
| DHCP | `show ip dhcp binding` | Una entrada por cada PC y laptop |
| ACL | `show access-lists` | Contadores de coincidencias en la línea `deny` después de probar |
| NAT | `show ip nat translations` | IP privadas traducidas a 200.10.10.2 |
| IP del PC | `ipconfig` | IP de su VLAN y gateway .1 |

### Matriz de pruebas

| # | Desde → hacia | Esperado |
|---|---|---|
| P1 | Cada PC: `ipconfig` | IP por DHCP |
| P2 | OF101-PC1 → 8.8.8.8 | Responde |
| P3 | OF101-PC1 → 192.168.30.11 | `Destination host unreachable` |
| P4 | OF101-PC1 → 192.168.40.11 | `Destination host unreachable` |
| P5 | INV-LAPTOP-P1 → 192.168.20.11 | `Destination host unreachable` |
| P6 | INV-LAPTOP-P2 → 8.8.8.8 | Responde |
| P7 | ADMIN-PC → 192.168.30.11 | Responde |
| P8 | ADMIN-PC → 192.168.99.2 | Responde |
| P9 | OF201-PC1: `ipconfig` | 192.168.40.x (el DHCP llega por la fibra) |

> **Dato útil:** es normal que el **primer** ping a un destino nuevo pierda un paquete mientras se resuelve ARP. Repite la prueba antes de concluir que algo falla.

Las salidas reales de estas pruebas están en `img/figura-03` a `img/figura-08`.

---

## Módulo 9. Solución de problemas

| Síntoma | Causa probable | Cómo confirmarlo | Solución |
|---|---|---|---|
| El PC recibe una IP 169.254.x.x | No le llega el DHCP | `show vlan brief` o `show interfaces trunk` | Revisar la VLAN del puerto, la troncal y el pool |
| El piso 2 no obtiene IP | La troncal de fibra está caída o falta el SFP | `show interfaces trunk` en SW-P2 | Instalar el SFP y poner `switchport mode trunk` en ambos extremos |
| Una empresa llega a otra | Falta la ACL o está aplicada en el sentido incorrecto | `show running-config` en la subinterfaz | `ip access-group AISLAR-VLANxx in` |
| Nadie sale a Internet | Falta `ip nat inside/outside` o la ruta por defecto | `show ip nat translations` y `show ip route` | Corregir NAT y ruta |
| Una empresa no sale a Internet | La línea `deny` está por debajo de `permit any` o falta `permit ip any any` | `show access-lists` | Reordenar la ACL |
| La consola pide el diálogo inicial | La configuración no se guardó | — | Responder `no` y guardar con `copy run start` |
| `Translating "xyz"...` y la consola se congela | Se escribió un comando inválido y el equipo intenta resolverlo por DNS | — | `no ip domain-lookup` (ya está en R-BORDE) |

---

## Módulo 10. Ejercicios de ampliación

<details><summary><b>10.1 Llega una Empresa E a la nueva oficina OF203 (piso 2). Intégrala usando los puertos de reserva.</b></summary>

1. **Físico:** usa las salidas `2A.A01` a `2A.A04`, que ya están cableadas en el panel y cuyos puertos Gi1/0/1–4 de SW-P2 están libres.
2. **VLAN 70** en los dos switches: `vlan 70` y `name EMPRESA-E-OF203`.
3. **SW-P2:** `interface range Gi1/0/1 - 4`, `switchport mode access`, `switchport access vlan 70`.
4. **R-BORDE:** crear la subinterfaz `Gi0/1.70` con `encapsulation dot1Q 70`, la IP 192.168.70.1/24 e `ip nat inside`.
5. **DHCP:** excluir .1 a .10 y crear el pool `VLAN70-EMPRESA-E`.
6. **ACL `AISLAR-VLAN70`:** copiar la estructura de las demás y aplicarla con `ip access-group AISLAR-VLAN70 in`.
7. **Probar:** que salga a Internet y que no llegue a las demás empresas.

Las troncales ya permiten todas las VLAN (1–1005), así que no hay que tocarlas. **Esto demuestra el valor del diseño: un nuevo inquilino no requiere cableado nuevo.**

</details>

<details><summary><b>10.2 Endurecer la seguridad de la red</b></summary>

- Cambiar la VLAN nativa de las troncales (hoy es la 1) por una VLAN sin uso, y ejecutar `switchport nonegotiate`.
- Configurar `enable secret` y usuarios locales, y habilitar **SSH** en las líneas VTY. Hoy `line vty` tiene `login` sin contraseña, lo que bloquea el acceso remoto en lugar de protegerlo.
- Activar `switchport port-security` en los puertos de las oficinas.
- Activar `spanning-tree portfast` y `bpduguard enable` en los puertos de acceso.
- Asignar los puertos apagados a una VLAN "de estacionamiento" sin uso.

</details>

<details><summary><b>10.3 Eliminar el punto único de falla</b></summary>

R-BORDE concentra el enrutamiento, el DHCP y el NAT. Algunas opciones:

- **Enrutamiento entre VLAN en el 3650**, que es multicapa: crear SVI por VLAN, `ip routing` y ACL en las SVI. El router quedaría solo para el NAT.
- Agregar un segundo router con **HSRP** (FHRP) y un segundo enlace ISP.
- Pasar el DHCP a un servidor dedicado con `ip helper-address`.

</details>

<details><summary><b>10.4 Recalcular para un tercer piso</b></summary>

Supón un piso 3 igual al piso 2. Usa 2 de los 4 hilos de fibra de reserva para un backbone 1A → 3A (o 2A → 3A), calcula las bobinas con el método del módulo 5 y actualiza el esquema de etiquetado (`3A.Axx`, `1A/3A-1`).

</details>

---

## Módulo 11. Preparación para la sustentación

**Preguntas probables y la idea clave de cada respuesta:**

1. **¿Cuál es el problema y a quién afecta?**
   La red plana e improvisada en un coworking compartido. Afecta al administrador (servicio poco confiable) y a las empresas (sin privacidad).
2. **¿Por qué VLAN y no redes físicas separadas?**
   Porque con una sola infraestructura se obtiene el mismo aislamiento lógico, a menor costo y con crecimiento más simple.
3. **¿Las VLAN bastan para aislar?**
   No. Si el router enruta entre ellas, se necesitan las ACL (módulo 6.6).
4. **¿Por qué fibra en un edificio tan pequeño?**
   Por la inmunidad a la interferencia electromagnética, la capacidad de crecimiento y los hilos de reserva (módulo 2).
5. **¿Cómo calcularon el cable?**
   Con el método de la distancia promedio. Dio 4 bobinas y un enlace máximo de 28 m frente al límite de 90 m (módulo 5).
6. **¿Cómo se lee una etiqueta?**
   `1A.A05` significa piso 1, espacio A, panel A, puerto 05, que corresponde a Gi1/0/5 y a la VLAN 20 (módulo 4).
7. **¿Qué cambió respecto al Momento 1 y por qué?**
   La fibra se instaló, el 3560 se cambió por el 3650, se agregaron ACL y VLAN de invitados y de gestión, el etiquetado pasó a TIA-606-C y la configuración de los equipos entró en el alcance.
8. **¿Cómo validaron el diseño?**
   Con una matriz de 9 pruebas en Packet Tracer y sus salidas de consola.
9. **¿Qué limitaciones tiene?**
   Las medidas son supuestas, el router es un punto único de falla y el cableado no está certificado en sitio.
10. **¿Qué metodología siguieron?**
    El diseño descendente (top-down) de Oppenheimer: requerimientos, diseño lógico, diseño físico y pruebas.
11. **¿Por qué UTP Cat 6 en el horizontal y no Cat 5e o Cat 6A?**
    Cat 6 llega a 250 MHz y soporta 1000BASE-T a 100 m, de sobra para un enlace máximo de 28 m. Cat 6A haría falta para 10GBASE-T a 100 m, que este diseño no exige, y además Cat 6 soporta PoE para los puntos de acceso (módulo 2).
12. **¿Por qué la topología es en estrella y qué límite de distancia aplica?**
    Cada salida tiene su propio cable hasta el patch panel del piso: no hay empalmes ni cadenas de switches. El enlace permanente mide como máximo 90 m y el canal completo, con los patch cords, como máximo 100 m (módulo 1).
13. **¿Qué es router-on-a-stick y por qué una sola interfaz física?**
    R-BORDE divide Gi0/1 en subinterfaces, una por VLAN, con `encapsulation dot1Q`. Un solo enlace troncal da puerta de enlace a todas las redes sin dedicar un puerto físico a cada empresa (módulo 6.3).
14. **¿Cómo salen varias redes privadas a Internet con una sola IP pública?**
    Con NAT por sobrecarga (PAT): la ACL 1 permite 192.168.0.0/16 y `overload` traduce todo a 200.10.10.2, distinguiendo cada sesión por el número de puerto (módulo 6.5).
15. **¿Por qué la ACL se aplica de entrada (`in`) y no de salida?**
    Para filtrar el tráfico apenas entra al router desde su origen. Así, una sola ACL por VLAN controla todo lo que esa red envía (módulo 6.6).
16. **¿Qué puede y qué no puede hacer un invitado de la VLAN 60?**
    Puede salir a Internet. No puede llegar a las empresas, a la administración ni a la gestión, porque su subinterfaz lleva la misma ACL de aislamiento (módulo 6.6).
17. **¿Por qué el piso 1 no tiene backbone?**
    Porque el cuarto de equipos y el distribuidor del piso 1 son el mismo rack (1A). El backbone solo une espacios distintos: la fibra OS2 entre 1A y 2A (módulos 1 y 2).
18. **¿Cómo se protege el cableado de datos frente a la red eléctrica?**
    Datos y electricidad no comparten canalización, los recorridos paralelos respetan la separación de referencia (RETIE y BICSI) y los cruces se hacen a 90°. La ocupación de las canaletas se mantiene por debajo del 40 % (módulo 3).
19. **¿Por qué se reemplazó el switch 3560 por el 3650?**
    En Packet Tracer los puertos gigabit del 3560 son solo de cobre. El 3650 tiene ranuras SFP para el GLC-LH-SMD de la fibra y, al ser 3650-24PS, entrega PoE a los puntos de acceso (módulo 2).
20. **Si llega un inquilino nuevo, ¿hay que tender cableado nuevo?**
    No. Las salidas de reserva ya están cableadas y las troncales aceptan las VLAN 1–1005. Basta crear la VLAN, la subinterfaz, el pool DHCP y la ACL (módulo 10.1).

**Demostración en vivo recomendada (3 minutos):**

1. En OF101-PC1: `ping 8.8.8.8` responde.
2. En OF101-PC1: `ping 192.168.30.11` falla (aislamiento).
3. En ADMIN-PC: `ping 192.168.30.11` responde (acceso de la administración).
4. En R-BORDE: `show access-lists` muestra los contadores de coincidencias subiendo.

---

## Glosario

| Término | Definición |
|---|---|
| **ACL** | Lista de control de acceso: reglas que permiten o bloquean tráfico según origen, destino y protocolo |
| **Backbone** | Cableado troncal que une los cuartos de telecomunicaciones |
| **Canal** | Enlace completo de extremo a extremo, incluidos los patch cords (máximo 100 m) |
| **DHCP** | Protocolo que asigna direcciones IP automáticamente |
| **Enlace permanente** | Tramo fijo entre el patch panel y la salida (máximo 90 m) |
| **ER / TR** | Cuarto de equipos / cuarto de telecomunicaciones |
| **Faceplate / TO** | Placa de pared con las salidas de telecomunicaciones |
| **NAT / PAT** | Traducción de direcciones privadas a una IP pública; PAT distingue las conexiones por puerto |
| **ODF** | Distribuidor de fibra óptica donde se terminan los hilos |
| **OS2** | Fibra monomodo de interior y exterior para largas distancias |
| **PoE** | Alimentación eléctrica sobre el cable de red |
| **Router-on-a-stick** | Un router que enruta entre VLAN usando subinterfaces en un solo puerto troncal |
| **SFP** | Transceptor modular que convierte la señal eléctrica en óptica |
| **SVI** | Interfaz virtual de VLAN en un switch (por ejemplo, `interface Vlan99`) |
| **Troncal 802.1Q** | Enlace que transporta varias VLAN etiquetando cada trama |
| **VLAN** | Red lógica independiente dentro de un mismo switch físico |

## Referencias para estudiar

- BICSI. (2014). *Telecommunications distribution methods manual* (13.ª ed.).
- Oppenheimer, P. (2011). *Top-down network design* (3.ª ed.). Cisco Press.
- Telecommunications Industry Association. (2015a). *Generic telecommunications cabling for customer premises* (ANSI/TIA-568.0-D).
- Telecommunications Industry Association. (2015b). *Telecommunications pathways and spaces* (ANSI/TIA-569-D).
- Telecommunications Industry Association. (2017). *Administration standard for telecommunications infrastructure* (ANSI/TIA-606-C).
- Telecommunications Industry Association. (2018). *Balanced twisted-pair telecommunications cabling and components standard* (ANSI/TIA-568.2-D).
- Ministerio de Minas y Energía. (2013). *Reglamento técnico de instalaciones eléctricas (RETIE)*.
- Cisco Systems. (s.f.). *Cisco Packet Tracer* [Software]. https://www.netacad.com/cisco-packet-tracer
