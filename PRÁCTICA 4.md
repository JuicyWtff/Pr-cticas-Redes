<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas: Práctica 4**

**Tema:** Protocolos de interconexión

**Subtema:** Servicios IPv6 (OSPFv3 y asignación dinámica con DHCPv6)

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## 1. Introducción

La Práctica 3 dejó la red operando en pila doble, pero con una asimetría evidente: mientras el lado IPv4 converge automáticamente y asigna parámetros mediante DHCP, el lado IPv6 depende de rutas escritas a mano y de una autoconfiguración que no entrega información de servidores de nombres. Esta práctica cierra esa brecha.

Se sustituyen las rutas estáticas IPv6 por OSPFv3 y se incorporan las dos modalidades de DHCPv6 que contempla el estándar, contrastando sus diferencias mediante los indicadores que viajan en los anuncios de router. Al concluir, ambas pilas ofrecerán un nivel de servicio equivalente, condición necesaria para abordar la práctica final.

## 2. Objetivos

**General**

Implementar enrutamiento dinámico OSPFv3 y asignación dinámica de parámetros DHCPv6 en sus modalidades sin estado y con estado, completando la paridad funcional entre ambas pilas de protocolo.

**Específicos**

- Configurar OSPFv3 de área única y explicar por qué requiere un identificador de router en formato de 32 bits.
- Analizar la relación de vecindad IPv6 y verificar que se construya sobre direcciones de enlace local.
- Distinguir los indicadores de configuración administrada y de otra configuración presentes en los anuncios de router.
- Implementar DHCPv6 sin estado como complemento de la autoconfiguración.
- Implementar DHCPv6 con estado y verificar el registro de asignaciones en el servidor.
- Comparar el comportamiento de ambas pilas ante la pérdida de un enlace.

## 3. Marco teórico

OSPFv3 conserva el mecanismo de estado de enlace y el algoritmo de cálculo de su antecesor, pero se reescribió como un protocolo independiente de la versión de IP transportada. Las diferencias operativas más relevantes son tres. Primera, la relación entre vecinos se establece empleando direcciones de enlace local como origen y destino, lo que vuelve innecesario que los enlaces de tránsito posean direccionamiento global. Segunda, la selección de las interfaces participantes dejó de depender del comando de red y pasó a realizarse directamente en cada interfaz, lo que elimina el manejo de máscaras comodín. Tercera, y pese a operar sobre direcciones de 128 bits, el identificador de router conserva el formato de 32 bits heredado de IPv4, por lo que debe asignarse de forma explícita cuando el equipo carece de direcciones IPv4 configuradas.

En cuanto a la asignación de parámetros, IPv6 ofrece un espectro de alternativas que IPv4 no tenía. La autoconfiguración pura cubre prefijo y puerta de enlace, pero nada más. Cuando se requiere entregar información adicional, como la dirección del servidor de nombres o el nombre de dominio, se recurre a DHCPv6 sin estado: el router continúa anunciando el prefijo y el equipo sigue construyendo su propia dirección, pero el anuncio activa un indicador que instruye al equipo a solicitar los parámetros faltantes a un servidor. El servidor no lleva registro alguno de las direcciones, de ahí la denominación.

La modalidad con estado invierte el esquema. El anuncio de router señala que el equipo no debe autoconfigurarse, y la totalidad de los parámetros, dirección incluida, se solicita al servidor, que mantiene una tabla de asignaciones y concesiones con su correspondiente tiempo de vigencia. Esta variante resulta apropiada cuando la organización necesita control administrativo sobre qué dirección recibe cada equipo, por ejemplo para efectos de auditoría o de aplicación de políticas.

## 4. Material y equipo

|Recurso|Especificación|
|---|---|
|Software|Cisco Packet Tracer 8.2 o superior|
|Archivo base|`Practica3_SierraNorte.pkt`|
|Routers|4 × ISR 4331 con módulo NIM-2T|
|Switches|4 × Catalyst 2960|
|Equipos finales|5 × PC genérica, 2 × Server|

## 5. Topología

Se conserva sin modificaciones la topología de las prácticas anteriores. El cambio es exclusivamente lógico.

|Segmento|Modalidad de asignación|
|---|---|
|LAN Puebla|DHCPv6 con estado|
|LAN Cholula|DHCPv6 sin estado|
|LAN Atlixco|Autoconfiguración pura, sin cambios|

La asignación diferenciada por sede permite observar los tres comportamientos dentro de un mismo escenario.

## 6. Parámetros de configuración

### 6.1 Identificadores OSPFv3

|Router|Router ID|Proceso|Área|
|---|---|---|---|
|R1|1.1.1.1|20|0|
|R2|2.2.2.2|20|0|
|R3|3.3.3.3|20|0|

Se emplea deliberadamente un número de proceso distinto al de OSPFv2 para evidenciar que ambos procesos son independientes y mantienen bases de datos separadas, aunque coincidan los identificadores de router.

### 6.2 Ámbitos DHCPv6

|Ámbito|Sede|Modalidad|Prefijo asignable|Parámetros entregados|
|---|---|---|---|---|
|PUEBLA-STFUL|Puebla|Con estado|2001:DB8:ACAD:10::/64|Dirección, DNS, dominio|
|CHOLULA-STLESS|Cholula|Sin estado|—|DNS, dominio|

El servidor de nombres se declara como 2001:DB8:ACAD:10::20 y el dominio como `sierranorte.mx`. Esta dirección corresponde al equipo SRV-PUE de la LAN de Puebla, al cual debe asignarse manualmente la dirección `2001:DB8:ACAD:10::20/64` con puerta de enlace `fe80::1` antes de continuar, ya que un servidor no debe depender de asignación dinámica.

## 7. Desarrollo

### Paso 1. Preparación

Se abre `Practica3_SierraNorte.pkt` y se guarda como `Practica4_SierraNorte.pkt`.

### Paso 2. Retiro de las rutas estáticas IPv6

En R1:

```
configure terminal
no ipv6 route 2001:DB8:ACAD:20::/64 2001:DB8:ACAD:A1::2
no ipv6 route 2001:DB8:ACAD:30::/64 Serial0/1/1 fe80::3
no ipv6 route 2001:DB8:ACAD:A2::/64 Serial0/1/0
exit
```

En R2:

```
configure terminal
no ipv6 route 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:A1::1
no ipv6 route 2001:DB8:ACAD:30::/64 2001:DB8:ACAD:A2::2
no ipv6 route 2001:DB8:ACAD:A3::/64 2001:DB8:ACAD:A1::1
exit
```

En R3:

```
configure terminal
no ipv6 route 2001:DB8:ACAD:10::/64 2001:DB8:ACAD:A3::1
no ipv6 route 2001:DB8:ACAD:20::/64 2001:DB8:ACAD:A2::1
no ipv6 route 2001:DB8:ACAD:A1::/64 2001:DB8:ACAD:A3::1
exit
```

Antes de continuar conviene confirmar con `show ipv6 route` que únicamente permanecen entradas de redes directamente conectadas y de direcciones locales, y comprobar que la comunicación IPv6 entre sedes ha dejado de funcionar. Este estado intermedio sirve como línea base para apreciar el efecto del protocolo dinámico.

### Paso 3. Habilitación de OSPFv3 en R1

```
configure terminal
ipv6 router ospf 20
 router-id 1.1.1.1
 passive-interface GigabitEthernet0/0/1
 exit
interface GigabitEthernet0/0/1
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/0
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/1
 ipv6 ospf 20 area 0
 exit
end
```

### Paso 4. Habilitación de OSPFv3 en R2 y R3

En R2:

```
configure terminal
ipv6 router ospf 20
 router-id 2.2.2.2
 passive-interface GigabitEthernet0/0/1
 exit
interface GigabitEthernet0/0/1
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/1
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/0
 ipv6 ospf 20 area 0
 exit
end
```

En R3:

```
configure terminal
ipv6 router ospf 20
 router-id 3.3.3.3
 passive-interface GigabitEthernet0/0/1
 exit
interface GigabitEthernet0/0/1
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/1
 ipv6 ospf 20 area 0
 exit
interface Serial0/1/0
 ipv6 ospf 20 area 0
 exit
end
```

Se observará que la declaración de interfaces resulta notablemente más directa que en OSPFv2, al no requerir el cálculo de máscaras comodín ni la correspondencia entre sentencias de red e interfaces.

### Paso 5. Verificación del protocolo

```
show ipv6 ospf neighbor
show ipv6 ospf interface brief
show ipv6 route ospf
show ipv6 protocols
```

En la salida del primer comando debe advertirse que los vecinos se identifican mediante su identificador de router en formato de 32 bits, mientras que la dirección empleada en el intercambio corresponde al ámbito de enlace local. Las rutas aprendidas aparecen en la tabla marcadas con el código `O`.

Se comprueba la conectividad extremo a extremo con `ping` desde un equipo de Puebla hacia uno de Atlixco.

### Paso 6. DHCPv6 sin estado en Cholula

En R2:

```
configure terminal
ipv6 dhcp pool CHOLULA-STLESS
 dns-server 2001:DB8:ACAD:10::20
 domain-name sierranorte.mx
 exit
interface GigabitEthernet0/0/1
 ipv6 dhcp server CHOLULA-STLESS
 ipv6 nd other-config-flag
 exit
end
```

El indicador de otra configuración comunica a los equipos del segmento que el prefijo sigue siendo válido para autoconfiguración, pero que existen parámetros adicionales disponibles mediante solicitud explícita. En los equipos finales se selecciona _Auto Config_ en el apartado IPv6; la dirección se forma localmente y los datos del servidor de nombres se obtienen del router.

### Paso 7. DHCPv6 con estado en Puebla

En R1:

```
configure terminal
ipv6 dhcp pool PUEBLA-STFUL
 address prefix 2001:DB8:ACAD:10::/64
 dns-server 2001:DB8:ACAD:10::20
 domain-name sierranorte.mx
 exit
interface GigabitEthernet0/0/1
 ipv6 dhcp server PUEBLA-STFUL
 ipv6 nd managed-config-flag
 ipv6 nd prefix 2001:DB8:ACAD:10::/64 no-autoconfig
 exit
end
```

El indicador de configuración administrada instruye a los equipos a obtener la totalidad de sus parámetros del servidor. La sentencia adicional sobre el prefijo evita que el equipo construya además una dirección por cuenta propia, situación que de otro modo produciría dos direcciones globales simultáneas y dificultaría la interpretación de las pruebas.

En los equipos de la LAN de Puebla se selecciona la opción _DHCP_ dentro de la sección IPv6, no _Auto Config_.

### Paso 8. Verificación de la asignación dinámica

```
show ipv6 dhcp pool
show ipv6 dhcp binding
show ipv6 interface GigabitEthernet0/0/1
```

El registro de asignaciones únicamente contendrá entradas en R1, dado que el ámbito sin estado no lleva control de direcciones. El tercer comando permite confirmar, en la sección de anuncios de router, el estado de los indicadores configurados en cada caso.

Para documentar el intercambio se emplea el modo de simulación filtrando DHCPv6, y se registra la secuencia de cuatro mensajes correspondiente a la modalidad con estado, contrastándola con el intercambio reducido de la modalidad sin estado.

### Paso 9. Comparación del comportamiento de ambas pilas

Se ejecutan simultáneamente dos pruebas de conectividad prolongadas desde un equipo de Cholula: una hacia la dirección IPv4 de un equipo de Puebla y otra hacia su dirección IPv6. Con ambas en curso se desactiva la interfaz Se0/1/1 de R2 y se registra, para cada pila, el número de paquetes perdidos hasta la recuperación por la trayectoria alterna.

```
configure terminal
interface Serial0/1/1
 shutdown
 exit
end
```

Transcurrido el registro se restablece la interfaz con `no shutdown` y se documenta nuevamente el comportamiento. Los resultados suelen ser comparables, lo que confirma que ambos procesos operan de forma autónoma pero con la misma lógica de convergencia.

### Paso 10. Resguardo

Se guarda la configuración en los tres routers internos y el proyecto como `Practica4_SierraNorte.pkt`.

## 8. Resultados esperados

|Prueba|Resultado|
|---|---|
|`show ipv6 ospf neighbor`|Dos vecinos en estado FULL por router, identificados con direcciones de enlace local|
|`show ipv6 route ospf`|Rutas marcadas con código O hacia los segmentos remotos|
|Equipo en Cholula|Dirección formada localmente, servidor de nombres recibido del router|
|Equipo en Puebla|Dirección y parámetros entregados íntegramente por el servidor|
|`show ipv6 dhcp binding` en R1|Una entrada por equipo configurado con DHCP|
|`show ipv6 dhcp binding` en R2|Vacío|
|Conectividad IPv4|Sin alteraciones respecto a la Práctica 2|

## 9. Cuestionario

1. Explique por qué OSPFv3 requiere un identificador de router de 32 bits pese a operar sobre direccionamiento de 128 bits, y qué sucede si no se declara en un router sin direcciones IPv4.
2. Justifique por qué el registro de asignaciones de R2 permanece vacío aun cuando los equipos de Cholula sí recibieron parámetros desde ese router.
3. Describa el efecto combinado de los indicadores de configuración administrada y de otra configuración, incluyendo el caso en que ambos se encuentren activos.
4. Analice qué ocurriría si se omitiera la sentencia `ipv6 nd prefix ... no-autoconfig` en la LAN de Puebla.
5. Compare el número de comandos necesarios para habilitar OSPFv2 y OSPFv3 en un router de tres interfaces, e identifique la causa de la diferencia.
6. Contraste los tiempos de reconvergencia obtenidos en el Paso 9 para ambas pilas y proponga una explicación de cualquier discrepancia observada.

## 10. Evidencias solicitadas

- Captura de `show ipv6 ospf neighbor` en los tres routers.
- Captura de `show ipv6 route ospf` en R1.
- Capturas de la configuración IPv6 obtenida en un equipo de cada sede, señalando la modalidad empleada.
- Captura de `show ipv6 dhcp binding` en R1 y en R2.
- Capturas del modo de simulación con los intercambios DHCPv6 de ambas modalidades.
- Registro comparativo de la prueba de convergencia.
- Archivo `Practica4_SierraNorte.pkt`.

## 11. Conclusiones

_Espacio reservado para la redacción individual de cada integrante del equipo._

## 12. Referencias

- Coltun, R., Ferguson, D., Moy, J. y Lindem, A. (2008). _OSPF for IPv6_. RFC 5340. IETF.
- Mrugalski, T. et al. (2018). _Dynamic Host Configuration Protocol for IPv6 (DHCPv6)_. RFC 8415. IETF.
- Narten, T. et al. (2007). _Neighbor Discovery for IP version 6 (IPv6)_. RFC 4861. IETF.
- Cisco Systems. _CCNA: Enterprise Networking, Security, and Automation_. Cisco Networking Academy.