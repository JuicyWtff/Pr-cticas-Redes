<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas — Índice general**

**Tema:** Protocolos de interconexión

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## Propósito del manual

Las cinco prácticas que integran este manual no son ejercicios independientes. Todas se desarrollan sobre un mismo escenario corporativo que crece en cada sesión, de modo que el archivo resultante de una práctica constituye el punto de partida obligatorio de la siguiente. Al concluir la quinta, el equipo habrá construido una red empresarial completa con pila doble, servicios de asignación dinámica en ambas versiones del protocolo, traducción de direcciones hacia el exterior y un mecanismo de transición operativo.

## Escenario

_Comercializadora Sierra Norte, S.A. de C.V._ es una empresa con matriz en la ciudad de Puebla y sucursales en Cholula y Atlixco, interconectadas mediante enlaces dedicados. La matriz contrata con un proveedor el acceso hacia el exterior. Durante el desarrollo del manual la organización incorpora una cuarta oficina en Teziutlán, alcanzable únicamente a través de la infraestructura del proveedor.

## Secuencia de prácticas

|#|Subtema|Contenido|Archivo generado|
|---|---|---|---|
|1|Servicios IPv4|Diseño VLSM, configuración básica de dispositivos, enrutamiento estático entre dos sedes|`Practica1_SierraNorte.pkt`|
|2|Servicios IPv4|Tercera sede, OSPFv2 con redundancia, DHCPv4, PAT hacia el proveedor|`Practica2_SierraNorte.pkt`|
|3|Servicios IPv6|Pila doble, direccionamiento global y de enlace local, EUI-64, SLAAC, rutas estáticas IPv6|`Practica3_SierraNorte.pkt`|
|4|Servicios IPv6|OSPFv3, DHCPv6 sin estado y con estado, comparación de convergencia entre pilas|`Practica4_SierraNorte.pkt`|
|5|Cambio de IPv4 a IPv6|Cuarta sede sobre proveedor IPv4, túnel manual, redistribución, ACL IPv6, verificación integral|`Practica5_SierraNorte.pkt`|

## Dependencias entre prácticas

```
 P1  ──>  P2  ──>  P3  ──>  P4  ──>  P5
 VLSM    OSPFv2   IPv6     OSPFv3   Tunel
 estat.  DHCP     SLAAC    DHCPv6   ACLv6
         PAT      rutas             integra
                  estat.            cion
```

Cada práctica retira o sustituye configuraciones de la anterior únicamente cuando el avance temático lo justifica: la Práctica 2 elimina las rutas estáticas IPv4 al introducir OSPFv2, y la Práctica 4 hace lo propio con las rutas estáticas IPv6. El resto de la configuración se acumula.

## Plan de direccionamiento consolidado

### IPv4 — bloque 192.168.100.0/24 con VLSM

|Segmento|Red|Prefijo|
|---|---|---|
|LAN Puebla|192.168.100.0|/26|
|LAN Cholula|192.168.100.64|/27|
|LAN Atlixco|192.168.100.96|/28|
|WAN R1–R2|192.168.100.112|/30|
|WAN R2–R3|192.168.100.116|/30|
|WAN R1–R3|192.168.100.120|/30|
|Reserva|192.168.100.124|/30|

### IPv4 — espacio público

|Segmento|Red|Prefijo|
|---|---|---|
|Enlace R1–ISP|209.165.200.224|/30|
|Enlace ISP–R4|209.165.200.228|/30|
|LAN del proveedor|209.165.201.0|/27|

### IPv6 — bloque 2001:DB8:ACAD::/48

|Segmento|Prefijo|
|---|---|
|LAN Puebla|2001:DB8:ACAD:10::/64|
|LAN Cholula|2001:DB8:ACAD:20::/64|
|LAN Atlixco|2001:DB8:ACAD:30::/64|
|LAN Teziutlán|2001:DB8:ACAD:40::/64|
|WAN R1–R2|2001:DB8:ACAD:A1::/64|
|WAN R2–R3|2001:DB8:ACAD:A2::/64|
|WAN R1–R3|2001:DB8:ACAD:A3::/64|
|Túnel R1–R4|2001:DB8:ACAD:FF::/64|

### Convención de direcciones de enlace local

|Router|Dirección|
|---|---|
|R1|fe80::1|
|R2|fe80::2|
|R3|fe80::3|
|R4|fe80::4|

## Inventario de equipo final

|Dispositivo|Modelo|Módulo adicional|
|---|---|---|
|R1, R2, R3, R4, ISP|ISR 4331|NIM-2T|
|S1, S2, S3, S4|Catalyst 2960|—|
|SRV-PUE|Server genérico|—|
|SRV-WEB|Server genérico|—|
|Equipos de usuario|PC genérica|—|

## Criterios de entrega

Cada práctica se entrega con su archivo de simulación, las capturas señaladas en el apartado de evidencias y las conclusiones redactadas de forma individual por cada integrante. Las respuestas al cuestionario se incorporan al final del documento correspondiente.