<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas: Práctica 5**

**Tema:** Protocolos de interconexión

**Subtema:** Cambio de IPv4 a IPv6 (mecanismos de transición e integración final)

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## 1. Introducción

_Comercializadora Sierra Norte_ abre una oficina en Teziutlán. El inmueble queda fuera del alcance de los enlaces dedicados que unen a las tres sedes existentes, por lo que la única conectividad disponible es un circuito contratado con el mismo proveedor que ya da salida a la matriz. El problema surge de inmediato: la nueva oficina requiere participar en la red IPv6 corporativa, pero el proveedor opera exclusivamente sobre IPv4 y no ofrece tránsito nativo de nueva generación.

Esta situación, lejos de ser un artificio didáctico, describe la condición habitual durante la migración de cualquier organización. La práctica final del manual resuelve el escenario mediante un túnel configurado manualmente, integra la sede remota al dominio de enrutamiento existente y cierra con una verificación completa de todo lo construido a lo largo de las cinco prácticas.

## 2. Objetivos

**General**

Interconectar dos islas IPv6 separadas por una infraestructura de tránsito exclusivamente IPv4 mediante encapsulamiento manual, e integrar la sede resultante a la red corporativa verificando la operación conjunta de ambas pilas.

**Específicos**

- Clasificar los mecanismos de transición en las categorías de pila doble, encapsulamiento y traducción.
- Configurar una interfaz de túnel con modo de encapsulamiento IPv6 sobre IPv4.
- Identificar los requisitos de accesibilidad entre los extremos del túnel.
- Propagar las redes alcanzadas por el túnel dentro del dominio OSPFv3 mediante redistribución.
- Analizar el efecto del encapsulamiento sobre la unidad máxima de transmisión.
- Aplicar una lista de control de acceso IPv6 sobre el tráfico que cruza el túnel.
- Realizar la verificación integral de la red corporativa en sus dos pilas de protocolo.

## 3. Marco teórico

La sustitución de IPv4 por IPv6 nunca se planteó como un evento instantáneo. Al no existir compatibilidad directa entre ambos formatos de encabezado, un nodo que solo habla una versión resulta incapaz de comunicarse con uno que solo habla la otra, lo que obligó a diseñar mecanismos de convivencia. Estos se agrupan en tres categorías.

La pila doble consiste en que cada dispositivo ejecute ambos protocolos simultáneamente, eligiendo cuál emplear según la disponibilidad del destino. Es la estrategia preferida cuando se controla toda la trayectoria, y es precisamente la que se implementó en las prácticas 3 y 4. Su limitación aparece cuando algún tramo intermedio escapa al control administrativo, como ocurre con la red de un proveedor.

El encapsulamiento resuelve justamente ese caso. Las unidades de datos IPv6 se transportan como carga útil dentro de unidades IPv4, de modo que la infraestructura intermedia las reenvía sin necesidad de comprender su contenido. Existen variantes automáticas, que derivan la dirección del extremo remoto a partir de la propia dirección IPv6, y variantes manuales, donde el administrador declara explícitamente origen y destino de cada túnel. Las primeras escalan mejor pero resultan opacas al diagnóstico; las segundas exigen configuración punto por punto a cambio de un comportamiento completamente determinista. Dado que la organización requiere un único enlace hacia una sola sede remota, la variante manual es la adecuada.

La traducción, finalmente, permite que un nodo que solo habla una versión se comunique con uno que solo habla la otra, mediante un dispositivo intermedio que reescribe los encabezados. El mecanismo vigente combina la traducción con una manipulación de las respuestas del servicio de nombres, de modo que un cliente que solo dispone de IPv6 reciba una dirección sintética para un destino que únicamente existe en IPv4. Esta familia de mecanismos no se encuentra disponible en el simulador empleado, por lo que se aborda únicamente en el plano conceptual.

Un aspecto que conviene anticipar es el impacto del encapsulamiento sobre el tamaño de las unidades de datos. El encabezado IPv4 que se antepone consume veinte octetos adicionales, de manera que la unidad máxima de transmisión efectiva del túnel resulta menor que la del enlace físico subyacente. Si un nodo de origen emite unidades que, una vez encapsuladas, exceden ese límite, el resultado es una pérdida silenciosa de tráfico que suele manifestarse como un comportamiento errático: las pruebas de conectividad responden, pero las transferencias de volumen fallan.

## 4. Material y equipo

|Recurso|Especificación|
|---|---|
|Software|Cisco Packet Tracer 8.2 o superior|
|Archivo base|`Practica4_SierraNorte.pkt`|
|Routers|5 × ISR 4331 con módulo NIM-2T|
|Switches|5 × Catalyst 2960|
|Equipos finales|7 × PC genérica, 2 × Server|

## 5. Topología

![[practica5_topologia.png]]

El túnel atraviesa lógicamente al router ISP, el cual nunca procesa tráfico IPv6: desde su perspectiva únicamente observa unidades IPv4 intercambiadas entre dos direcciones públicas.

## 6. Plan de direccionamiento adicional

|Dispositivo|Interfaz|Dirección|Observación|
|---|---|---|---|
|ISP|Se0/1/0 (DCE)|209.165.200.229/30|Enlace hacia la sede remota|
|R4|Se0/1/0|209.165.200.230/30|Extremo físico del túnel|
|R4|Gi0/0/1|2001:DB8:ACAD:40::1/64|LAN Teziutlán, sin IPv4|
|R4|Gi0/0/1|fe80::4|Enlace local|
|R1|Tunnel0|2001:DB8:ACAD:FF::1/64|Extremo lógico en matriz|
|R4|Tunnel0|2001:DB8:ACAD:FF::2/64|Extremo lógico en sede remota|

La LAN de Teziutlán se despliega intencionalmente sin direccionamiento IPv4, de modo que su conectividad dependa por completo del mecanismo de transición y cualquier falla resulte evidente.

## 7. Desarrollo

### Paso 1. Preparación

Se abre `Practica4_SierraNorte.pkt` y se guarda como `Practica5_SierraNorte.pkt`. Se incorporan el router R4 con su módulo NIM-2T, el switch S4 y dos equipos finales. Se instala también un módulo NIM-2T en el router ISP.

El cableado adicional consiste en un enlace serial DCE desde Se0/1/0 de ISP hacia Se0/1/0 de R4, y cables de cobre directo entre R4, S4 y los equipos de la nueva sede.

### Paso 2. Configuración del enlace hacia el proveedor

En ISP:

```
configure terminal
interface Serial0/1/0
 description Enlace-Teziutlan
 ip address 209.165.200.229 255.255.255.252
 clock rate 128000
 no shutdown
 exit
end
```

En R4:

```
enable
configure terminal
hostname R4
no ip domain-lookup
enable secret Cl4v3.Pr1v
ipv6 unicast-routing
interface Serial0/1/0
 description Enlace-ISP
 ip address 209.165.200.230 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0/1
 description LAN-Teziutlan
 ipv6 address 2001:DB8:ACAD:40::1/64
 ipv6 address fe80::4 link-local
 no shutdown
 exit
ip route 0.0.0.0 0.0.0.0 209.165.200.229
end
```

La ruta predeterminada IPv4 en R4 no sirve para transportar tráfico de usuarios: su única función es permitir que el router alcance la dirección pública del extremo opuesto del túnel. Sin ella, la interfaz de túnel permanecería operativa en apariencia pero ningún paquete encapsulado llegaría a destino.

### Paso 3. Verificación previa de accesibilidad

Antes de configurar el túnel debe comprobarse que ambos extremos se alcanzan mutuamente sobre IPv4:

```
ping 209.165.200.225
```

Esta prueba, ejecutada desde R4, debe responder. En sentido inverso, desde R1 se ejecuta `ping 209.165.200.230`. Si cualquiera de las dos falla, el túnel no funcionará y el diagnóstico debe resolverse en este punto, nunca después.

### Paso 4. Configuración del túnel en R1

```
configure terminal
interface Tunnel0
 description Tunel-hacia-Teziutlan
 ipv6 address 2001:DB8:ACAD:FF::1/64
 ipv6 address fe80::1 link-local
 tunnel source GigabitEthernet0/0/0
 tunnel destination 209.165.200.230
 tunnel mode ipv6ip
 no shutdown
 exit
end
```

El origen puede declararse mediante el nombre de la interfaz o mediante su dirección. La primera forma resulta preferible porque el túnel sobrevive a un cambio de direccionamiento en el extremo local.

### Paso 5. Configuración del túnel en R4

```
configure terminal
interface Tunnel0
 description Tunel-hacia-Matriz
 ipv6 address 2001:DB8:ACAD:FF::2/64
 ipv6 address fe80::4 link-local
 tunnel source Serial0/1/0
 tunnel destination 209.165.200.225
 tunnel mode ipv6ip
 no shutdown
 exit
end
```

Los valores de origen y destino deben encontrarse cruzados con exactitud respecto a los declarados en R1. Un error en este punto produce una interfaz de túnel en estado activo que nunca transporta tráfico, situación particularmente difícil de diagnosticar para quien no revisa la configuración de ambos extremos.

### Paso 6. Verificación del túnel

```
show interfaces Tunnel0
show ipv6 interface brief
ping 2001:DB8:ACAD:FF::1
```

La prueba de conectividad se ejecuta desde R4 hacia el extremo opuesto del túnel. Una respuesta afirmativa confirma que el encapsulamiento opera correctamente. Resulta instructivo ejecutar simultáneamente el modo de simulación y observar cómo la unidad de datos sale de R4 con encabezado IPv4 y es reenviada por el router ISP sin que este tenga conocimiento alguno de su contenido.

### Paso 7. Enrutamiento a través del túnel

En R4 se declara una ruta predeterminada IPv6 que dirija todo el tráfico hacia la matriz:

```
configure terminal
ipv6 route ::/0 2001:DB8:ACAD:FF::1
exit
```

En R1 se declara la ruta hacia la LAN remota y se ordena su inyección en el dominio OSPFv3, de modo que R2 y R3 también la conozcan:

```
configure terminal
ipv6 route 2001:DB8:ACAD:40::/64 2001:DB8:ACAD:FF::2
ipv6 router ospf 20
 redistribute static
 exit
end
```

Sin la redistribución, únicamente la matriz alcanzaría Teziutlán, mientras que Cholula y Atlixco quedarían incomunicadas con la nueva sede. Tras unos segundos, `show ipv6 route` en R3 debe exhibir la red remota marcada como externa al dominio.

### Paso 8. Configuración de los equipos de la sede remota

En los equipos de la LAN de Teziutlán se selecciona _Auto Config_ en el apartado IPv6 y se deja sin configurar el apartado IPv4. La dirección debe formarse a partir del prefijo 2001:DB8:ACAD:40::/64 anunciado por R4.

Desde uno de ellos se ejecuta una prueba de conectividad hacia un equipo de la LAN de Atlixco. El resultado exitoso implica que el tráfico recorrió la LAN remota, se encapsuló en R4, cruzó la red IPv4 del proveedor, se desencapsuló en R1 y continuó por el dominio OSPFv3 hasta R3.

### Paso 9. Análisis de la unidad máxima de transmisión

Se consulta el valor vigente en la interfaz de túnel:

```
show interfaces Tunnel0 | include MTU
```

Se contrasta con el valor de la interfaz física subyacente y se calcula la diferencia. A continuación se ejecuta desde un equipo de Teziutlán una prueba con unidades de datos de tamaño incrementado hacia un destino de la matriz, con el fin de localizar el umbral a partir del cual la comunicación deja de completarse. El resultado se documenta junto con una explicación del fenómeno.

### Paso 10. Control de acceso sobre el tráfico del túnel

La dirección corporativa establece que la sede remota únicamente requiere acceso al servicio web y al servicio de nombres de la matriz, no a la totalidad de la red interna. En R1 se aplica la política correspondiente:

```
configure terminal
ipv6 access-list FILTRO-TEZIUTLAN
 permit tcp 2001:DB8:ACAD:40::/64 host 2001:DB8:ACAD:10::20 eq www
 permit udp 2001:DB8:ACAD:40::/64 host 2001:DB8:ACAD:10::20 eq domain
 permit icmp 2001:DB8:ACAD:40::/64 2001:DB8:ACAD:10::/64
 deny ipv6 2001:DB8:ACAD:40::/64 2001:DB8:ACAD:20::/64
 deny ipv6 2001:DB8:ACAD:40::/64 2001:DB8:ACAD:30::/64
 permit ipv6 any any
 exit
interface Tunnel0
 ipv6 traffic-filter FILTRO-TEZIUTLAN in
 exit
end
```

Se verifica que la conectividad desde Teziutlán hacia Puebla continúe funcionando mientras que las pruebas hacia Cholula y Atlixco sean descartadas, y se consulta el conteo de coincidencias:

```
show ipv6 access-list
```

Debe notarse que la sentencia final resulta indispensable: a diferencia de lo que podría suponerse, la denegación implícita al término de toda lista afectaría también a los mensajes de descubrimiento de vecinos, degradando el funcionamiento del propio protocolo.

### Paso 11. Verificación integral de la red corporativa

Se ejecuta la batería completa de pruebas que valida el trabajo de las cinco prácticas:

```
show ip interface brief
show ipv6 interface brief
show ip route
show ipv6 route
show ip ospf neighbor
show ipv6 ospf neighbor
show ip nat translations
show ip dhcp binding
show ipv6 dhcp binding
show interfaces Tunnel0
show ipv6 access-list
```

### Paso 12. Resguardo final

Se guarda la configuración en los cinco routers y el proyecto como `Practica5_SierraNorte.pkt`. Este archivo constituye la entrega final del manual y debe contener la totalidad de las funcionalidades desarrolladas.

## 8. Matriz de verificación final

|#|Origen|Destino|Protocolo|Mecanismo involucrado|Resultado|
|---|---|---|---|---|---|
|1|PC Puebla|PC Cholula|IPv4|OSPFv2|Exitoso|
|2|PC Cholula|SRV-WEB (209.165.201.10)|IPv4|OSPFv2 + ruta predeterminada + PAT|Exitoso|
|3|PC Atlixco|Obtención de dirección|IPv4|DHCPv4|Exitoso|
|4|PC Puebla|Obtención de dirección|IPv6|DHCPv6 con estado|Exitoso|
|5|PC Cholula|Obtención de parámetros|IPv6|DHCPv6 sin estado|Exitoso|
|6|PC Puebla|PC Atlixco|IPv6|OSPFv3|Exitoso|
|7|R4|2001:DB8:ACAD:FF::1|IPv6|Túnel manual|Exitoso|
|8|PC Teziutlán|PC Puebla|IPv6|Túnel + OSPFv3 + ACL|Exitoso|
|9|PC Teziutlán|PC Cholula|IPv6|ACL aplicada|Descartado|
|10|ISP|PC Puebla|IPv6|—|Fallido|
|11|ISP|192.168.100.11|IPv4|—|Fallido|

Los tres últimos renglones corresponden a comportamientos deseados, no a defectos de la implementación.

## 9. Cuestionario

1. Explique por qué el router ISP transporta correctamente el tráfico de la sede remota sin tener habilitado el reenvío IPv6.
2. Describa qué ocurriría si la ruta predeterminada IPv4 de R4 se eliminara una vez que el túnel ya se encuentra operativo.
3. Justifique la necesidad de redistribuir la ruta estática en el proceso OSPFv3 y determine qué sedes quedarían aisladas sin ella.
4. A partir del valor obtenido en el Paso 9, calcule el sobrecosto porcentual que impone el encapsulamiento sobre una unidad de datos de 1500 octetos.
5. Compare el mecanismo implementado con una solución basada en traducción, señalando en qué escenarios cada uno resulta preferible.
6. Analice qué sucedería si la sentencia `permit ipv6 any any` se retirara de la lista de control de acceso.
7. Proponga, con base en la topología final, una estrategia de migración que permita prescindir del túnel en el mediano plazo.

## 10. Evidencias solicitadas

- Captura de la topología final completa con los cinco routers.
- Captura de `show interfaces Tunnel0` en ambos extremos.
- Captura del modo de simulación mostrando el encapsulamiento en el router ISP.
- Captura de `show ipv6 route` en R3 con la red remota marcada como externa.
- Registro del análisis de unidad máxima de transmisión.
- Captura de `show ipv6 access-list` con conteo de coincidencias.
- Matriz de verificación final completada con resultados observados.
- Archivo `Practica5_SierraNorte.pkt`.

## 11. Conclusiones

_Espacio reservado para la redacción individual de cada integrante del equipo._

## 12. Referencias

- Nordmark, E. y Gilligan, R. (2005). _Basic Transition Mechanisms for IPv6 Hosts and Routers_. RFC 4213. IETF.
- Bao, C. et al. (2010). _IPv6 Addressing of IPv4/IPv6 Translators_. RFC 6052. IETF.
- Bagnulo, M., Matthews, P. y van Beijnum, I. (2011). _Stateful NAT64: Network Address and Protocol Translation from IPv6 Clients to IPv4 Servers_. RFC 6146. IETF.
- Carpenter, B. y Moore, K. (2001). _Connection of IPv6 Domains via IPv4 Clouds_. RFC 3056. IETF.
- Cisco Systems. _CCNA: Enterprise Networking, Security, and Automation_. Cisco Networking Academy.