<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas: Práctica 3**

**Tema:** Protocolos de interconexión

**Subtema:** Servicios IPv6  (direccionamiento, SLAAC y enrutamiento estático)

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## 1. Introducción

La red de _Comercializadora Sierra Norte_ funciona correctamente sobre IPv4, pero la organización proyecta incorporar servicios que exigen direccionamiento de nueva generación. En lugar de sustituir la infraestructura existente, se opta por la estrategia de pila doble: cada router operará simultáneamente ambas versiones del protocolo, de modo que los procesos previos continúen intactos mientras se habilita la nueva conectividad.

Esta práctica aborda el direccionamiento IPv6 en sus distintos ámbitos, la autoconfiguración de los equipos finales y el enrutamiento manual entre las tres sedes. El enrutamiento dinámico sobre IPv6 corresponde a la Práctica 4.

## 2. Objetivos

**General**

Habilitar pila doble en la red corporativa, asignando direccionamiento IPv6 global y de enlace local, y establecer conectividad entre las tres sedes mediante rutas estáticas IPv6 sin alterar la operación IPv4 existente.

**Específicos**

- Distinguir los ámbitos de dirección unicast global, de enlace local y multicast de nodo solicitado.
- Aplicar el método EUI-64 y contrastarlo con la asignación manual del identificador de interfaz.
- Configurar direcciones de enlace local reconocibles para simplificar el diagnóstico.
- Habilitar el reenvío de unidades de datos IPv6 y comprender su carácter no predeterminado.
- Implementar autoconfiguración sin estado en los equipos finales y analizar los mensajes de descubrimiento de vecinos.
- Construir rutas estáticas IPv6 en sus tres variantes y verificar la conectividad resultante.

## 3. Marco teórico

El salto de 32 a 128 bits en la longitud de la dirección es el cambio más visible de IPv6, pero no el más determinante. La reestructuración del encabezado, la eliminación de la fragmentación en routers intermedios, la supresión de la suma de verificación de capa de red y la sustitución de los mensajes de difusión por multidifusión selectiva modifican de fondo el comportamiento del protocolo.

Un aspecto que suele generar confusión es la coexistencia de varias direcciones en una misma interfaz. Toda interfaz habilitada posee obligatoriamente una dirección de enlace local, perteneciente al prefijo fe80::/10, cuyo alcance se limita al segmento físico y que nunca es reenviada por un router. Sobre ella se apoyan el descubrimiento de vecinos, la detección de direcciones duplicadas y la propia relación entre routers vecinos en los protocolos de enrutamiento. De manera adicional se configura una dirección unicast global, equivalente funcional de una dirección pública IPv4, enrutable a través de la Internet.

El identificador de interfaz, es decir los 64 bits menos significativos, puede derivarse de la dirección física mediante el procedimiento EUI-64, que inserta el patrón fffe en el centro de la dirección MAC e invierte el séptimo bit del primer octeto. El resultado es único pero expone el identificador del fabricante y permite rastrear un equipo entre distintas redes, motivo por el cual los sistemas operativos actuales prefieren identificadores generados de forma aleatoria.

La autoconfiguración sin estado permite que un equipo obtenga prefijo, longitud y puerta de enlace a partir de los anuncios de router que se emiten periódicamente en el segmento, sin necesidad de un servidor que lleve registro de las asignaciones. El equipo completa por sí mismo el identificador de interfaz y verifica, mediante la detección de direcciones duplicadas, que nadie más la esté utilizando.

## 4. Material y equipo

|Recurso|Especificación|
|---|---|
|Software|Cisco Packet Tracer 8.2 o superior|
|Archivo base|`Practica2_SierraNorte.pkt`|
|Routers|4 × ISR 4331 con módulo NIM-2T|
|Switches|4 × Catalyst 2960|
|Equipos finales|5 × PC genérica, 2 × Server|

## 5. Topología

La topología física no cambia respecto a la Práctica 2. Lo que se incorpora es una segunda pila de protocolo sobre las mismas interfaces.

![[practica3_topologia.png]]

El enlace hacia el proveedor permanece exclusivamente en IPv4. Esta restricción es intencional y constituye el punto de partida del problema que resuelve la Práctica 5.

## 6. Plan de direccionamiento IPv6

El bloque asignado a la organización es 2001:DB8:ACAD::/48, lo que permite 65 536 subredes de prefijo /64. Se reserva el cuarto grupo hexadecimal para identificar el segmento.

|Segmento|Prefijo|Función|
|---|---|---|
|LAN Puebla|2001:DB8:ACAD:10::/64|Red de usuarios de la matriz|
|LAN Cholula|2001:DB8:ACAD:20::/64|Red de usuarios de sucursal|
|LAN Atlixco|2001:DB8:ACAD:30::/64|Red de usuarios de sucursal|
|WAN R1–R2|2001:DB8:ACAD:A1::/64|Enlace punto a punto|
|WAN R2–R3|2001:DB8:ACAD:A2::/64|Enlace punto a punto|
|WAN R1–R3|2001:DB8:ACAD:A3::/64|Enlace punto a punto|
|LAN Teziutlán|2001:DB8:ACAD:40::/64|Reservado para la Práctica 5|
|Túnel|2001:DB8:ACAD:FF::/64|Reservado para la Práctica 5|

### 6.1 Asignación por interfaz

|Dispositivo|Interfaz|Dirección unicast global|Enlace local|
|---|---|---|---|
|R1|Gi0/0/1|2001:DB8:ACAD:10::1/64|fe80::1|
|R1|Se0/1/0|2001:DB8:ACAD:A1::1/64|fe80::1|
|R1|Se0/1/1|2001:DB8:ACAD:A3::1/64|fe80::1|
|R2|Gi0/0/1|2001:DB8:ACAD:20::1/64|fe80::2|
|R2|Se0/1/1|2001:DB8:ACAD:A1::2/64|fe80::2|
|R2|Se0/1/0|2001:DB8:ACAD:A2::1/64|fe80::2|
|R3|Gi0/0/1|2001:DB8:ACAD:30::1/64|fe80::3|
|R3|Se0/1/1|2001:DB8:ACAD:A2::2/64|fe80::3|
|R3|Se0/1/0|2001:DB8:ACAD:A3::2/64|fe80::3|

La repetición de la misma dirección de enlace local en todas las interfaces de un router es válida porque su unicidad solo se exige dentro de cada segmento. Esta convención facilita enormemente la lectura de las tablas de vecinos y de enrutamiento.

## 7. Desarrollo

### Paso 1. Preparación

Se abre `Practica2_SierraNorte.pkt` y se guarda como `Practica3_SierraNorte.pkt`. No se elimina ninguna configuración previa.

### Paso 2. Habilitación del reenvío IPv6

El procesamiento de unidades de datos IPv6 se encuentra desactivado de fábrica en IOS. En cada uno de los tres routers internos:

```
configure terminal
ipv6 unicast-routing
exit
```

Omitir este comando produce un escenario engañoso: las interfaces responden a pruebas de conectividad dirigidas a ellas mismas, pero el router descarta silenciosamente todo tráfico que deba atravesarlo y no emite anuncios de router, por lo que los equipos finales nunca completan su autoconfiguración.

### Paso 3. Direccionamiento de R1

```
configure terminal
interface GigabitEthernet0/0/1
 ipv6 address 2001:DB8:ACAD:10::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
interface Serial0/1/0
 ipv6 address 2001:DB8:ACAD:A1::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
interface Serial0/1/1
 ipv6 address 2001:DB8:ACAD:A3::1/64
 ipv6 address fe80::1 link-local
 no shutdown
 exit
end
```

### Paso 4. Direccionamiento de R2

```
configure terminal
interface GigabitEthernet0/0/1
 ipv6 address 2001:DB8:ACAD:20::1/64
 ipv6 address fe80::2 link-local
 exit
interface Serial0/1/1
 ipv6 address 2001:DB8:ACAD:A1::2/64
 ipv6 address fe80::2 link-local
 exit
interface Serial0/1/0
 ipv6 address 2001:DB8:ACAD:A2::1/64
 ipv6 address fe80::2 link-local
 exit
end
```

### Paso 5. Direccionamiento de R3 con el método EUI-64

Para contrastar ambos procedimientos, la interfaz LAN de Atlixco se direcciona de forma manual y los enlaces seriales emplean derivación automática del identificador:

```
configure terminal
interface GigabitEthernet0/0/1
 ipv6 address 2001:DB8:ACAD:30::1/64
 ipv6 address fe80::3 link-local
 exit
interface Serial0/1/1
 ipv6 address 2001:DB8:ACAD:A2::/64 eui-64
 ipv6 address fe80::3 link-local
 exit
interface Serial0/1/0
 ipv6 address 2001:DB8:ACAD:A3::/64 eui-64
 ipv6 address fe80::3 link-local
 exit
end
```

Se ejecuta `show ipv6 interface brief` y se registran las direcciones resultantes en los enlaces seriales. Dado que las interfaces seriales carecen de dirección física propia, IOS toma prestada la de una interfaz Ethernet del mismo equipo, por lo que ambos enlaces exhibirán identificadores idénticos sobre prefijos distintos. Este comportamiento, correcto pero poco intuitivo, es uno de los argumentos a favor del direccionamiento manual en enlaces punto a punto.

Una vez documentado el resultado, se sustituyen ambas direcciones por las del plan original:

```
configure terminal
interface Serial0/1/1
 no ipv6 address
 ipv6 address 2001:DB8:ACAD:A2::2/64
 ipv6 address fe80::3 link-local
 exit
interface Serial0/1/0
 no ipv6 address
 ipv6 address 2001:DB8:ACAD:A3::2/64
 ipv6 address fe80::3 link-local
 exit
end
```

### Paso 6. Verificación de vecindad

```
show ipv6 interface brief
show ipv6 interface GigabitEthernet0/0/1
show ipv6 neighbors
```

El segundo comando revela la lista de grupos de multidifusión a los que la interfaz se ha unido: el grupo de todos los nodos ff02::1, el de todos los routers ff02::2 y el grupo de nodo solicitado ff02::1:ff00:1, derivado de los últimos 24 bits de la dirección unicast. Este último es el que permite que la resolución de direcciones en IPv6 interrumpa únicamente a los equipos cuyo identificador coincide parcialmente, en lugar de molestar a todo el segmento como hacía ARP.

Desde R1 se comprueba la adyacencia con su vecino directo:

```
ping 2001:DB8:ACAD:A1::2
```

### Paso 7. Autoconfiguración de los equipos finales

En la configuración de red de cada equipo se selecciona la opción _Auto Config_ dentro de la sección IPv6. Los equipos de la LAN de Puebla, Cholula y Atlixco deben obtener automáticamente una dirección con el prefijo de su segmento y registrar como puerta de enlace predeterminada la dirección de enlace local del router correspondiente, no su dirección global.

Para analizar el intercambio se activa el modo de simulación con filtro exclusivo para ICMPv6, se desactiva y reactiva la interfaz del equipo, y se observa la secuencia: solicitud de router emitida hacia ff02::2, anuncio de router respondido hacia ff02::1, y solicitud de vecino dirigida al grupo de nodo solicitado para verificar que la dirección formada no esté duplicada.

### Paso 8. Enrutamiento estático IPv6

En R1 se emplean las tres variantes para efectos comparativos:

```
configure terminal
ipv6 route 2001:DB8:ACAD:20::/64 2001:DB8:ACAD:A1::2
ipv6 route 2001:DB8:ACAD:30::/64 Serial0/1/1 fe80::3
ipv6 route 2001:DB8:ACAD:A2::/64 Serial0/1/0
exit
```

En R2:

```
configure terminal
ipv6 route 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:A1::1
ipv6 route 2001:DB8:ACAD:30::/64 2001:DB8:ACAD:A2::2
ipv6 route 2001:DB8:ACAD:A3::/64 2001:DB8:ACAD:A1::1
exit
```

En R3:

```
configure terminal
ipv6 route 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:A3::1
ipv6 route 2001:DB8:ACAD:20::/64 2001:DB8:ACAD:A2::1
ipv6 route 2001:DB8:ACAD:A1::/64 2001:DB8:ACAD:A3::1
exit
```

La segunda ruta declarada en R1 ilustra un punto fundamental: cuando el siguiente salto se expresa mediante una dirección de enlace local, resulta obligatorio especificar también la interfaz de salida, puesto que la misma dirección fe80::3 podría existir en varios segmentos conectados al router y la información por sí sola sería ambigua.

### Paso 9. Verificación final

```
show ipv6 route
show ipv6 route static
show ipv6 protocols
```

Desde un equipo de la LAN de Puebla se ejecuta `ping` hacia la dirección global de un equipo de Atlixco y se repite la prueba con `tracert`. Se verifica adicionalmente que la conectividad IPv4 establecida en las prácticas anteriores continúe operando sin degradación, incluyendo el acceso al servidor externo a través de la traducción de direcciones.

### Paso 10. Resguardo

Se guarda la configuración en los tres routers internos y el proyecto como `Practica3_SierraNorte.pkt`.

## 8. Resultados esperados

|Prueba|Resultado|
|---|---|
|`show ipv6 interface brief`|Dos direcciones por interfaz: enlace local y unicast global|
|Autoconfiguración en equipos finales|Prefijo correcto y puerta de enlace fe80:: del router local|
|Tabla de vecinos|Entradas en estado REACH o STALE tras generar tráfico|
|Ping IPv6 entre sedes distintas|Exitoso|
|Ping IPv4 hacia 209.165.201.10|Exitoso, sin cambios respecto a la Práctica 2|
|Ping IPv6 hacia el proveedor|Fallido|

## 9. Cuestionario

1. Explique por qué la puerta de enlace predeterminada que anuncian los routers corresponde a una dirección de enlace local y no a la dirección unicast global.
2. Reconstruya manualmente, a partir de la dirección física de la interfaz, el identificador EUI-64 obtenido en el Paso 5 y detalle la transformación aplicada al séptimo bit.
3. Determine la dirección de multidifusión de nodo solicitado correspondiente a 2001:DB8:ACAD:20::1 y justifique el cálculo.
4. Argumente por qué la ruta que utiliza fe80::3 como siguiente salto exige declarar la interfaz de salida.
5. Describa qué funcionalidad se pierde si se omite `ipv6 unicast-routing` y por qué el síntoma puede confundirse con un problema de direccionamiento.
6. Compare el número de rutas estáticas requeridas en esta práctica con las que serían necesarias si la red creciera a seis sedes.

## 10. Evidencias solicitadas

- Captura de `show ipv6 interface brief` en los tres routers.
- Captura de `show ipv6 interface GigabitEthernet0/0/1` mostrando los grupos de multidifusión.
- Captura del registro obtenido por autoconfiguración en un equipo de cada sede.
- Capturas del modo de simulación con los mensajes de solicitud y anuncio de router.
- Captura de `show ipv6 route` en R2.
- Archivo `Practica3_SierraNorte.pkt`.

## 11. Conclusiones

_Espacio reservado para la redacción individual de cada integrante del equipo._

## 12. Referencias

- Deering, S. y Hinden, R. (2017). _Internet Protocol, Version 6 (IPv6) Specification_. RFC 8200. IETF.
- Thomson, S., Narten, T. y Jinmei, T. (2007). _IPv6 Stateless Address Autoconfiguration_. RFC 4862. IETF.
- Narten, T. et al. (2007). _Neighbor Discovery for IP version 6 (IPv6)_. RFC 4861. IETF.
- Hinden, R. y Deering, S. (2006). _IP Version 6 Addressing Architecture_. RFC 4291. IETF.