<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas: Práctica 1**

**Tema:** Protocolos de interconexión

**Subtema:** Servicios IPv4 (direccionamiento VLSM y enrutamiento estático)

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## 1. Introducción

El manual que aquí comienza se desarrolla sobre un único escenario corporativo que irá creciendo práctica tras práctica. La empresa ficticia _Comercializadora Sierra Norte, S.A. de C.V._ opera una matriz en la ciudad de Puebla y sucursales en Cholula y Atlixco, y hacia el final del manual incorporará una oficina remota en Teziutlán conectada a través de un proveedor de servicios.

Esta primera práctica levanta los cimientos: se define el plan de direccionamiento IPv4 completo mediante máscaras de longitud variable y se habilita la comunicación entre las dos primeras sedes con rutas configuradas manualmente. Todo lo construido aquí se conserva y se amplía en las prácticas siguientes, por lo que se recomienda guardar el archivo resultante con un nombre que permita identificarlo sin ambigüedad.

## 2. Objetivos

**General**

Diseñar e implementar el esquema de direccionamiento IPv4 de la red corporativa aplicando VLSM, y establecer conectividad extremo a extremo entre dos sedes mediante enrutamiento estático.

**Específicos**

- Subdividir el bloque 192.168.100.0/24 en subredes de tamaño desigual según la demanda de hosts de cada sede.
- Configurar parámetros básicos de seguridad y administración en dispositivos Cisco IOS.
- Asignar direcciones IPv4 a interfaces LAN y WAN, incluyendo la señalización de reloj en enlaces seriales.
- Construir rutas estáticas directamente conectadas y recursivas, y diferenciar su comportamiento.
- Validar la conectividad mediante herramientas de diagnóstico y lectura de la tabla de enrutamiento.

## 3. Marco teórico

El agotamiento de direcciones IPv4 públicas obligó a abandonar el reparto por clases y adoptar esquemas donde la frontera entre la porción de red y la de host puede colocarse en cualquier bit. VLSM lleva esa idea un paso más allá: dentro de un mismo bloque asignado se emplean prefijos distintos, de modo que un enlace punto a punto entre dos routers consuma únicamente cuatro direcciones y no un segmento completo desperdiciado. El procedimiento consiste en ordenar los requerimientos de mayor a menor y asignar secuencialmente, lo que evita solapamientos y deja los espacios libres agrupados al final del bloque.

El enrutamiento estático, por su parte, consiste en que el administrador indique explícitamente al router por dónde alcanzar cada destino remoto. Carece de la adaptabilidad de los protocolos dinámicos, pero no consume ancho de banda en mensajes de actualización ni ciclos de procesador en cálculos de ruta, y su comportamiento resulta completamente predecible. Por ello sigue siendo la opción adecuada en topologías pequeñas, en enlaces de salida únicos y como respaldo flotante de rutas aprendidas dinámicamente. Una ruta estática puede expresarse señalando la interfaz de salida, la dirección del siguiente salto, o ambas; la primera forma resuelve el reenvío en una sola consulta, mientras que la segunda obliga al router a una búsqueda adicional para determinar cómo alcanzar ese siguiente salto.

## 4. Material y equipo

|Recurso|Especificación|
|---|---|
|Software|Cisco Packet Tracer 8.2 o superior|
|Routers|2 × ISR 4331 con módulo NIM-2T|
|Switches|2 × Catalyst 2960|
|Equipos finales|3 × PC genérica, 1 × Server|
|Cableado|Cobre directo para LAN, serial DCE/DTE para WAN|

## 5. Topología

![[practica1_topologia.png]]

Las interfaces Gi0/0/0, Se0/1/1 de R1 y Se0/1/0 de R2 permanecen sin configurar en esta práctica; se habilitarán en la Práctica 2.

## 6. Plan de direccionamiento

### 6.1 Cálculo VLSM sobre 192.168.100.0/24

Los requerimientos ordenados de mayor a menor son: 60 hosts para la matriz, 30 para Cholula, 12 para Atlixco y tres enlaces punto a punto de 2 direcciones utilizables cada uno.

|Segmento|Hosts requeridos|Dirección de red|Prefijo|Máscara|Rango utilizable|Broadcast|
|---|---|---|---|---|---|---|
|LAN Puebla|60|192.168.100.0|/26|255.255.255.192|.1 – .62|192.168.100.63|
|LAN Cholula|30|192.168.100.64|/27|255.255.255.224|.65 – .94|192.168.100.95|
|LAN Atlixco|12|192.168.100.96|/28|255.255.255.240|.97 – .110|192.168.100.111|
|WAN R1–R2|2|192.168.100.112|/30|255.255.255.252|.113 – .114|192.168.100.115|
|WAN R2–R3|2|192.168.100.116|/30|255.255.255.252|.117 – .118|192.168.100.119|
|WAN R1–R3|2|192.168.100.120|/30|255.255.255.252|.121 – .122|192.168.100.123|
|Reserva|—|192.168.100.124|/30|255.255.255.252|.125 – .126|192.168.100.127|

Las subredes de Atlixco y los enlaces hacia R3 se calculan desde ahora para que el plan no deba rehacerse al incorporar la tercera sede.

### 6.2 Asignación de interfaces

|Dispositivo|Interfaz|Dirección IPv4|Máscara|Gateway|
|---|---|---|---|---|
|R1|Gi0/0/1|192.168.100.1|255.255.255.192|—|
|R1|Se0/1/0 (DCE)|192.168.100.113|255.255.255.252|—|
|R2|Gi0/0/1|192.168.100.65|255.255.255.224|—|
|R2|Se0/1/1|192.168.100.114|255.255.255.252|—|
|SRV-PUE|NIC|192.168.100.10|255.255.255.192|192.168.100.1|
|PC-PUE1|NIC|192.168.100.11|255.255.255.192|192.168.100.1|
|PC-CHO1|NIC|192.168.100.70|255.255.255.224|192.168.100.65|
|PC-CHO2|NIC|192.168.100.71|255.255.255.224|192.168.100.65|

## 7. Desarrollo

### Paso 1. Construcción de la topología física

Se colocan en el área de trabajo dos routers ISR 4331 y dos switches 2960. Antes de cablear los enlaces seriales es necesario apagar cada router, insertar el módulo NIM-2T en la ranura disponible y volver a encenderlo, ya que Packet Tracer no permite instalar módulos con el equipo energizado.

El cableado se realiza de la siguiente manera: cable de cobre directo entre Gi0/0/1 de cada router y un puerto FastEthernet de su switch correspondiente; cable de cobre directo entre cada equipo final y su switch; cable serial DCE desde Se0/1/0 de R1 hacia Se0/1/1 de R2. El extremo donde se conecta el cable DCE determina qué router deberá proporcionar la señal de reloj.

### Paso 2. Configuración básica de los routers

En R1, desde el modo de configuración global:

```
enable
configure terminal
hostname R1
no ip domain-lookup
enable secret Cl4v3.Pr1v
line console 0
 password Cl4v3.C0n
 login
 logging synchronous
 exec-timeout 10 0
 exit
line vty 0 4
 password Cl4v3.VTY
 login
 transport input ssh
 exit
service password-encryption
banner motd #Acceso restringido - Sierra Norte SA de CV#
exit
```

El procedimiento se repite en R2 cambiando únicamente el nombre de host. El comando `no ip domain-lookup` evita que el router intente resolver como nombre de dominio cualquier palabra mal escrita en el modo EXEC, comportamiento que de otro modo bloquea la terminal durante varios segundos.

### Paso 3. Direccionamiento de interfaces en R1

```
configure terminal
interface GigabitEthernet0/0/1
 description LAN-Puebla
 ip address 192.168.100.1 255.255.255.192
 no shutdown
 exit
interface Serial0/1/0
 description WAN-hacia-R2
 ip address 192.168.100.113 255.255.255.252
 clock rate 128000
 no shutdown
 exit
end
```

La instrucción `clock rate` se aplica exclusivamente en el extremo DCE. Si se omite, la interfaz permanecerá administrativamente activa pero con el protocolo de línea caído.

### Paso 4. Direccionamiento de interfaces en R2

```
configure terminal
interface GigabitEthernet0/0/1
 description LAN-Cholula
 ip address 192.168.100.65 255.255.255.224
 no shutdown
 exit
interface Serial0/1/1
 description WAN-hacia-R1
 ip address 192.168.100.114 255.255.255.252
 no shutdown
 exit
end
```

### Paso 5. Verificación intermedia

Antes de agregar rutas conviene comprobar que las interfaces negociaron correctamente:

```
show ip interface brief
show controllers Serial0/1/0
```

Las cuatro interfaces configuradas deben reportar estado _up_ y protocolo _up_. El segundo comando confirma en R1 la presencia del cable DCE y la tasa de reloj aplicada. En este punto cada router alcanza únicamente sus redes directamente conectadas: una prueba de conectividad desde PC-PUE1 hacia PC-CHO1 debe fallar.

### Paso 6. Configuración de rutas estáticas

En R1 se declara la ruta hacia la LAN de Cholula empleando la dirección del siguiente salto:

```
configure terminal
ip route 192.168.100.64 255.255.255.224 192.168.100.114
exit
```

En R2 se declara la ruta hacia la LAN de Puebla, esta vez combinando interfaz de salida y siguiente salto:

```
configure terminal
ip route 192.168.100.0 255.255.255.192 Serial0/1/1 192.168.100.113
exit
```

Ambas formas producen conectividad equivalente, pero la segunda queda registrada en la tabla como ruta completamente especificada y evita la resolución recursiva.

### Paso 7. Verificación final

```
show ip route
show ip route static
ping 192.168.100.65
traceroute 192.168.100.70
```

Desde PC-PUE1 se ejecuta `ping 192.168.100.70` y desde PC-CHO1 se ejecuta `ping 192.168.100.11`. Ambas pruebas deben responder. Conviene además emplear el modo de simulación de Packet Tracer filtrando únicamente ICMP, para observar el recorrido de la unidad de datos a través del enlace serial.

### Paso 8. Resguardo de la configuración

```
copy running-config startup-config
```

El archivo del proyecto se guarda como `Practica1_SierraNorte.pkt`. Este archivo constituye el punto de partida obligatorio de la Práctica 2.

## 8. Resultados esperados

|Prueba|Origen|Destino|Resultado|
|---|---|---|---|
|Conectividad LAN local|PC-PUE1|SRV-PUE|Exitosa|
|Conectividad hacia gateway|PC-CHO1|192.168.100.65|Exitosa|
|Conectividad WAN|R1|192.168.100.114|Exitosa|
|Conectividad extremo a extremo|PC-PUE1|PC-CHO2|Exitosa|
|Conectividad a sede no implementada|PC-PUE1|192.168.100.97|Fallida|

El último renglón no constituye un error: la sede de Atlixco se incorpora hasta la práctica siguiente.

## 9. Cuestionario

1. Justifique por qué el prefijo /26 resulta suficiente para la LAN de Puebla y qué ocurriría si la plantilla creciera a 70 equipos.
2. Explique la diferencia observable en la salida de `show ip route` entre la ruta configurada en R1 y la configurada en R2.
3. Describa el efecto de omitir el comando `clock rate` en el extremo DCE y qué salida del comando `show interfaces` lo evidencia.
4. Calcule cuántas direcciones del bloque 192.168.100.0/24 quedaron sin asignar tras aplicar el plan VLSM propuesto.
5. Argumente en qué circunstancias el enrutamiento estático dejaría de ser viable para esta topología.

## 10. Evidencias solicitadas

- Captura de la topología completa con todos los enlaces en estado operativo.
- Captura de `show ip interface brief` en ambos routers.
- Captura de `show ip route` en ambos routers.
- Captura de una prueba de conectividad exitosa entre equipos de distinta sede.
- Archivo `Practica1_SierraNorte.pkt`.

## 11. Conclusiones

_Espacio reservado para la redacción individual de cada integrante del equipo._

## 12. Referencias

- Cisco Systems. _CCNA: Switching, Routing, and Wireless Essentials_. Cisco Networking Academy.
- Fuller, V. y Li, T. (2006). _Classless Inter-domain Routing (CIDR): The Internet Address Assignment and Aggregation Plan_. RFC 4632. IETF.
- Rekhter, Y. et al. (1996). _Address Allocation for Private Internets_. RFC 1918. IETF.