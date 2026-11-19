<div align="center">

# Benemérita Universidad Autónoma de Puebla

## Facultad de Ciencias de la Computación

**Materia:** Intercomunicación y Seguridad en Redes

**Manual de prácticas: Práctica 2**

**Tema:** Protocolos de interconexión

**Subtema:** Servicios IPv4 (enrutamiento dinámico OSPFv2, DHCP y traducción de direcciones)

---

**Integrantes del equipo**

Cholula Olvera Alejandro

Tapia Gálvez Samuel Fernando

Rodríguez Bravo Hugo Javier

---

Otoño 2026

</div> <div style="page-break-after: always;"></div>

## 1. Introducción

Con la base construida en la práctica anterior, la red corporativa de _Comercializadora Sierra Norte_ incorpora ahora su tercera sede y sustituye las rutas declaradas manualmente por un protocolo capaz de recalcular trayectorias ante una falla. Se agregan además dos servicios imprescindibles en cualquier despliegue real: la asignación automática de parámetros a los equipos finales y la traducción de direcciones privadas hacia el espacio público, que permite a la organización salir a Internet con un solo identificador global.

Al término de esta práctica el lado IPv4 de la red queda completo. Las prácticas 3 y 4 replicarán esta misma funcionalidad sobre IPv6 y la práctica 5 resolverá la coexistencia entre ambos mundos.

## 2. Objetivos

**General**

Sustituir el enrutamiento estático por OSPFv2 en una topología triangular con redundancia, e incorporar los servicios de asignación dinámica de direcciones y traducción de direcciones de red.

**Específicos**

- Integrar un tercer router y un enlace hacia un proveedor de servicios simulado.
- Configurar OSPFv2 de área única, ajustando identificadores de router y costos de interfaz.
- Declarar interfaces pasivas y propagar una ruta predeterminada dentro del dominio de enrutamiento.
- Implementar ámbitos DHCPv4 con exclusiones y parámetros opcionales.
- Configurar sobrecarga de direcciones (PAT) y verificar la tabla de traducciones.
- Comprobar la convergencia del protocolo ante la caída deliberada de un enlace.

## 3. Marco teórico

OSPF pertenece a la familia de protocolos de estado de enlace. A diferencia de los esquemas de vector distancia, donde cada router confía en el resumen que le entrega su vecino, aquí todos los participantes de un área construyen una base de datos idéntica que describe la topología completa y, sobre ella, ejecutan de manera independiente el algoritmo de Dijkstra para obtener el árbol de rutas más cortas desde su propia posición. La consecuencia práctica es una convergencia rápida y la ausencia de bucles de enrutamiento, a cambio de un mayor consumo de memoria y procesamiento.

La métrica que emplea se denomina costo y se deriva de forma inversamente proporcional al ancho de banda de la interfaz. Dado que las referencias originales quedaron rebasadas por las velocidades actuales, en enlaces de alta capacidad el cálculo tiende a igualar costos distintos, por lo que se acostumbra ajustar el ancho de banda de referencia o fijar el costo manualmente.

La traducción de direcciones responde a un problema distinto. Las organizaciones direccionan su red interna con los bloques reservados para uso privado, que carecen de validez en el enrutamiento global. Al cruzar la frontera hacia el proveedor, el router reescribe la dirección de origen de cada paquete por una dirección pública y registra la correspondencia en una tabla. Cuando varias conversaciones comparten la misma dirección pública, la distinción se consigue incorporando el número de puerto de origen a la traducción, mecanismo conocido como sobrecarga o PAT. El efecto secundario relevante es que las sesiones solo pueden iniciarse desde el interior, lo que proporciona un grado incidental de ocultamiento sin constituir por sí mismo una medida de seguridad.

## 4. Material y equipo

|Recurso|Especificación|
|---|---|
|Software|Cisco Packet Tracer 8.2 o superior|
|Archivo base|`Practica1_SierraNorte.pkt`|
|Routers|4 × ISR 4331 con módulo NIM-2T|
|Switches|4 × Catalyst 2960|
|Equipos finales|5 × PC genérica, 2 × Server|

## 5. Topología

![[practica2_topologia.png]]

Nota sobre el diagrama: R1 se conecta a R3 por Se0/1/1 de R1 contra Se0/1/0 de R3, y R2 se conecta a R3 por Se0/1/1 de R2 contra Se0/1/1 de R3. El enlace hacia el proveedor parte de Gi0/0/0 de R1.

## 6. Plan de direccionamiento

### 6.1 Direcciones privadas

Se conserva íntegramente el plan VLSM de la Práctica 1 y se activan las subredes previstas para Atlixco.

|Dispositivo|Interfaz|Dirección IPv4|Máscara|
|---|---|---|---|
|R1|Gi0/0/1|192.168.100.1|255.255.255.192|
|R1|Se0/1/0 (DCE)|192.168.100.113|255.255.255.252|
|R1|Se0/1/1 (DCE)|192.168.100.121|255.255.255.252|
|R1|Gi0/0/0|209.165.200.225|255.255.255.252|
|R2|Gi0/0/1|192.168.100.65|255.255.255.224|
|R2|Se0/1/1|192.168.100.114|255.255.255.252|
|R2|Se0/1/0 (DCE)|192.168.100.117|255.255.255.252|
|R3|Gi0/0/1|192.168.100.97|255.255.255.240|
|R3|Se0/1/0|192.168.100.122|255.255.255.252|
|R3|Se0/1/1|192.168.100.118|255.255.255.252|

### 6.2 Direcciones públicas

|Dispositivo|Interfaz|Dirección IPv4|Máscara|
|---|---|---|---|
|ISP|Gi0/0/0|209.165.200.226|255.255.255.252|
|ISP|Gi0/0/1|209.165.201.1|255.255.255.224|
|SRV-WEB|NIC|209.165.201.10|255.255.255.224|

### 6.3 Identificadores OSPF

|Router|Router ID|Área|
|---|---|---|
|R1|1.1.1.1|0|
|R2|2.2.2.2|0|
|R3|3.3.3.3|0|

## 7. Desarrollo

### Paso 1. Preparación del escenario

Se abre el archivo `Practica1_SierraNorte.pkt` y se guarda de inmediato con el nombre `Practica2_SierraNorte.pkt` para preservar intacta la evidencia anterior. Se incorporan el router R3 con su módulo NIM-2T, el switch S3, dos equipos finales para la LAN de Atlixco, el router ISP y el servidor SRV-WEB.

El cableado adicional comprende: serial DCE desde Se0/1/1 de R1 hacia Se0/1/0 de R3; serial DCE desde Se0/1/0 de R2 hacia Se0/1/1 de R3; cobre directo desde Gi0/0/0 de R1 hacia Gi0/0/0 de ISP; cobre directo desde Gi0/0/1 de ISP hacia SRV-WEB.

### Paso 2. Eliminación de las rutas estáticas previas

Las rutas declaradas en la práctica anterior deben retirarse para que OSPF sea la única fuente de información de enrutamiento interno.

En R1:

```
configure terminal
no ip route 192.168.100.64 255.255.255.224 192.168.100.114
exit
```

En R2:

```
configure terminal
no ip route 192.168.100.0 255.255.255.192 Serial0/1/1 192.168.100.113
exit
```

### Paso 3. Configuración de R3 y de las interfaces nuevas

```
enable
configure terminal
hostname R3
no ip domain-lookup
enable secret Cl4v3.Pr1v
interface GigabitEthernet0/0/1
 description LAN-Atlixco
 ip address 192.168.100.97 255.255.255.240
 no shutdown
 exit
interface Serial0/1/0
 description WAN-hacia-R1
 ip address 192.168.100.122 255.255.255.252
 no shutdown
 exit
interface Serial0/1/1
 description WAN-hacia-R2
 ip address 192.168.100.118 255.255.255.252
 no shutdown
 exit
end
```

En R1 se agregan las dos interfaces restantes:

```
configure terminal
interface Serial0/1/1
 ip address 192.168.100.121 255.255.255.252
 clock rate 128000
 no shutdown
 exit
interface GigabitEthernet0/0/0
 description Enlace-ISP
 ip address 209.165.200.225 255.255.255.252
 no shutdown
 exit
end
```

En R2:

```
configure terminal
interface Serial0/1/0
 ip address 192.168.100.117 255.255.255.252
 clock rate 128000
 no shutdown
 exit
end
```

### Paso 4. Configuración del proveedor de servicios

```
enable
configure terminal
hostname ISP
no ip domain-lookup
interface GigabitEthernet0/0/0
 ip address 209.165.200.226 255.255.255.252
 no shutdown
 exit
interface GigabitEthernet0/0/1
 ip address 209.165.201.1 255.255.255.224
 no shutdown
 exit
end
```

El router ISP no recibe ninguna ruta hacia el espacio privado de la empresa. Esta omisión es deliberada: demuestra que el tráfico interno únicamente alcanzará el servidor externo si la traducción de direcciones opera correctamente.

### Paso 5. Habilitación de OSPFv2

En R1:

```
configure terminal
router ospf 10
 router-id 1.1.1.1
 network 192.168.100.0 0.0.0.63 area 0
 network 192.168.100.112 0.0.0.3 area 0
 network 192.168.100.120 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0/1
 auto-cost reference-bandwidth 1000
 exit
```

En R2:

```
configure terminal
router ospf 10
 router-id 2.2.2.2
 network 192.168.100.64 0.0.0.31 area 0
 network 192.168.100.112 0.0.0.3 area 0
 network 192.168.100.116 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0/1
 auto-cost reference-bandwidth 1000
 exit
```

En R3:

```
configure terminal
router ospf 10
 router-id 3.3.3.3
 network 192.168.100.96 0.0.0.15 area 0
 network 192.168.100.116 0.0.0.3 area 0
 network 192.168.100.120 0.0.0.3 area 0
 passive-interface GigabitEthernet0/0/1
 auto-cost reference-bandwidth 1000
 exit
```

Las máscaras que acompañan al comando `network` son máscaras comodín, es decir, el complemento bit a bit de la máscara de subred. La declaración de interfaz pasiva impide que el router emita paquetes de saludo hacia la LAN, donde no existe ningún vecino que los procese, sin dejar de anunciar esa red al resto del dominio. El ajuste del ancho de banda de referencia debe aplicarse de manera uniforme en todos los routers del área; de lo contrario los costos resultan inconsistentes.

### Paso 6. Ruta predeterminada y su propagación

Únicamente R1 conoce la salida hacia el exterior, por lo que allí se declara la ruta predeterminada y se ordena su distribución al resto del área:

```
configure terminal
ip route 0.0.0.0 0.0.0.0 GigabitEthernet0/0/0 209.165.200.226
router ospf 10
 default-information originate
 exit
end
```

Transcurridos unos segundos, R2 y R3 deben mostrar en su tabla de enrutamiento una entrada marcada con el código `O*E2`.

### Paso 7. Servicio DHCPv4

En R2 se configura el ámbito para la LAN de Cholula:

```
configure terminal
ip dhcp excluded-address 192.168.100.65 192.168.100.69
ip dhcp pool CHOLULA
 network 192.168.100.64 255.255.255.224
 default-router 192.168.100.65
 dns-server 209.165.201.10
 domain-name sierranorte.mx
 lease 3
 exit
end
```

En R3, para la LAN de Atlixco:

```
configure terminal
ip dhcp excluded-address 192.168.100.97 192.168.100.100
ip dhcp pool ATLIXCO
 network 192.168.100.96 255.255.255.240
 default-router 192.168.100.97
 dns-server 209.165.201.10
 domain-name sierranorte.mx
 lease 3
 exit
end
```

Las direcciones excluidas corresponden a la puerta de enlace y a un pequeño margen reservado para equipos que requieran direccionamiento fijo. En los equipos finales de ambas sedes se selecciona la opción DHCP en la configuración de la interfaz de red; la asignación debe reflejarse en segundos.

### Paso 8. Traducción de direcciones con sobrecarga

En R1:

```
configure terminal
access-list 10 permit 192.168.100.0 0.0.0.127
ip nat inside source list 10 interface GigabitEthernet0/0/0 overload
interface GigabitEthernet0/0/1
 ip nat inside
 exit
interface Serial0/1/0
 ip nat inside
 exit
interface Serial0/1/1
 ip nat inside
 exit
interface GigabitEthernet0/0/0
 ip nat outside
 exit
end
```

La lista de acceso emplea la máscara comodín 0.0.0.127 porque abarca de una sola vez las tres LAN internas, que ocupan en conjunto la primera mitad del bloque /24. Los enlaces seriales se marcan como interfaces internas para que el tráfico originado en Cholula y Atlixco también resulte traducido al atravesar R1.

### Paso 9. Verificación

```
show ip ospf neighbor
show ip ospf interface brief
show ip route ospf
show ip dhcp binding
show ip nat translations
show ip nat statistics
```

El primer comando debe listar dos adyacencias en cada router, todas en estado FULL. Desde un equipo de Atlixco se ejecuta `ping 209.165.201.10`; la respuesta confirma simultáneamente el funcionamiento de OSPF, de la ruta predeterminada propagada y de la traducción. Inmediatamente después, `show ip nat translations` en R1 debe exhibir la correspondencia entre la dirección privada de origen y la dirección pública con su puerto asignado.

### Paso 10. Prueba de convergencia

Se ejecuta un ping extendido desde un equipo de Cholula hacia la LAN de Puebla y, mientras se encuentra en curso, se desactiva administrativamente la interfaz Se0/1/1 de R2:

```
configure terminal
interface Serial0/1/1
 shutdown
 exit
end
```

Se registra el número de paquetes perdidos antes de que la comunicación se restablezca por la trayectoria alterna vía R3. Posteriormente se reactiva la interfaz con `no shutdown` y se documenta el tiempo de reconvergencia. Esta prueba evidencia la ventaja del enrutamiento dinámico frente al esquema estático de la Práctica 1, donde la caída del enlace habría dejado incomunicadas ambas sedes.

### Paso 11. Resguardo

Se guarda la configuración en los cuatro routers y el proyecto como `Practica2_SierraNorte.pkt`.

## 8. Resultados esperados

|Prueba|Resultado|
|---|---|
|Adyacencias OSPF en estado FULL|2 por router|
|Tabla de enrutamiento de R3|Rutas O hacia las LAN de Puebla y Cholula, más entrada O*E2|
|Asignación DHCP|Direcciones dentro del rango no excluido en Cholula y Atlixco|
|Ping desde Atlixco hacia 209.165.201.10|Exitoso|
|Tabla de traducciones en R1|Entradas con protocolo, puerto interno y puerto global|
|Ping desde ISP hacia 192.168.100.11|Fallido|
|Reconvergencia tras caída de enlace|Comunicación restablecida por trayectoria alterna|

## 9. Cuestionario

1. Explique por qué el último renglón de la tabla anterior arroja un resultado fallido y qué implicación de seguridad tiene.
2. Determine el costo acumulado que R3 calcula hacia la LAN de Puebla por cada una de las dos trayectorias disponibles, y justifique cuál selecciona.
3. Describa qué ocurriría si el comando `auto-cost reference-bandwidth` se aplicara solo en R1.
4. Analice el efecto de omitir `default-information originate` tras haber declarado la ruta predeterminada en R1.
5. Compare el contenido de `show ip nat translations` antes y después de generar tráfico desde dos equipos distintos hacia el mismo destino.

## 10. Evidencias solicitadas

- Captura de `show ip ospf neighbor` en los tres routers internos.
- Captura de `show ip route` en R3.
- Captura de la configuración IP obtenida por DHCP en un equipo de cada sede.
- Captura de `show ip nat translations` con al menos tres entradas activas.
- Registro de la prueba de convergencia con el conteo de paquetes perdidos.
- Archivo `Practica2_SierraNorte.pkt`.

## 11. Conclusiones

_Espacio reservado para la redacción individual de cada integrante del equipo._

## 12. Referencias

- Moy, J. (1998). _OSPF Version 2_. RFC 2328. IETF.
- Droms, R. (1997). _Dynamic Host Configuration Protocol_. RFC 2131. IETF.
- Srisuresh, P. y Egevang, K. (2001). _Traditional IP Network Address Translator (Traditional NAT)_. RFC 3022. IETF.
- Cisco Systems. _CCNA: Enterprise Networking, Security, and Automation_. Cisco Networking Academy.