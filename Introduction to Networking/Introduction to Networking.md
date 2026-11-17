Recordando Firewalls [[Network Foundations#Firewalls]]:

**ACL**: Access Control List
- Lo usan los firewalls para saber a quién dejar pasar según sea una configuración de una regla.

Una red en general, necesita IDS (Intrusion Detection System) y un IPS (Intrusion Prevention System) junto a la documentación de la topología de la red.

## FQDN
Significa **Fully Quialified Domain Name**. Es a lo que normalmente nos referimos por "dominio".

ej: 
```bash
www.hackthebox.com
```

## URL
Significa **Uniform Resource Location**.

ej:
```bash
https://subdoimain.domain.tld/path?parameters
```


## Topología ideal de una red 
![[topology|800]]


DMZ: Zona desmilitarizada -> Separa en otra subred junto a un firewall a los recursos públicos de la red general.

## RFC 1918 
Define los rangos de IP privadas:
10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16


## VPNs

- Site-to-site: conecta 2 redes directamente por medio de routers y firewalls
![[site2sitevpn|800]]

## Remote Access VPN
Un solo dispositivo por medio de un cliente VPN como OpenVPN, nord, UrbanVPN, etc. maneja la conexión tunelizandola:

![[remoteaccesvpn|800]]

Una VPN funciona encapsulando un paquete IP dentro de otro paquete IP.
- header1: [[IPVPN private source -> IPVPN pública destino][...][payload]
- header2: [[IP private source -> IP pública destino][TCP][header1][...]


## Split VPN
Permite que soolo ciertas direcciones de host sean ruteadas a la VPN según la tabla de direcciones de red de la VPN:

```zsh
┌──(silksnix㉿jdroz)-[/mnt/c/Users/jdroz]
└─$ ip route
default via 172.17.128.1 dev eth0 proto kernel
10.10.8.0/22 via 10.10.16.1 dev tun0
10.10.16.0/23 dev tun0 proto kernel scope link src 10.10.17.133
10.10.110.0/24 via 10.10.16.1 dev tun0
10.13.37.0/24 via 10.10.16.1 dev tun0
10.13.38.0/24 via 10.10.16.1 dev tun0
10.129.0.0/16 via 10.10.16.1 dev tun0
172.17.128.0/20 dev eth0 proto kernel scope link src 172.17.140.205
```

Se puede obtener a quién pertenece cierta dirección de host:

```zsh
┌──(silksnix㉿jdroz)-[/mnt/c/Users/jdroz]
└─$ ip route get 10.13.38.2
10.13.38.2 via 10.10.16.1 dev tun0 src 10.10.17.133 uid 1000
    cache
```




## Proxy
Usualmente operan en capa 7 del modelo OSI.
Es un servicio o dispositivo **MEDIADOR** en la comunicación.

Si se puede inspeccionar el contenido, es un PROXY, si no, simplemente es un Gateway. Por lo tanto los firewalls a nivel de aplicación son Proxys de capa 7.

**Gateway**: dispositivo que reenvía trafico sin entender el contenido de la app

**Forward proxy:** servicio o dispositivo que está del lado del cliente y hace peticiones a su nombre. Filtra solicitudes salientes por asi decirlo.
- PC -> PROXY -> Internet

**Reverse Proxy**: filtra solicitudes entrantes. Ej: cloudflare, nginx, waf, entre otros...

**(Non) Transparent Proxys:** 
- transparentes: no se sabe que existen
- no transparentes: se sabe que existen dentro del sistema

## TCP/IP

![[ositcppdu.png]]

Enviando: Se lee de arriba hacia abajo
Recibiendo: Se lee de abajo hacia arriba

{data APP}
{TCP|data APP}
{IP|TCP|data APP}
{TCP|IP|TCP|data APP}

- IP: asegura el destino
- TCP: asgura la transferencia de datos

## IPv4
Una dirección IP tendrá una parte de Red y otra de Host, según su mascara de red.
Ejemplo:

IP: 192.168.0.1/24 -> (24 bits de mascara de red)
Mask: 255.255.255.0

En binario se puede ver que la Mask es una mascara de bits que hace un AND con la IP, y con esto se sabe cuál es la porción de host:

- 11111111.11111111.11111111.00000000

Por lo tanto, la red 192.168.0.1/24 tendría  2^8 - 2 direcciones de host
La razón del -2 es porque:

 1. IP_0 = 192.168.0.0 (dirección de red)
 2. IP_255 = 192.168.0.255 (dirección de Broadcast)

Recordar que para pasar de binario a decimal rápidamente se puede así:
![[binary.png]]


Sufijo CIDR: /<Número de bits de red>
- ej: /24
- 192.168.0.0/24

## Subnetting

Para subnetear una red, solo lo podemos hacer en potencias de 2.
Si necesitamos 3 subredes, entonces necesitamos crear 4 subredes en total.

Digamos que tenemos las siguientes specs:
- IP: 192.168.0.0/24
- Se necesitan 3 subredes: oficina, gerencia y TI

Necesitamos la mascara de red que determine las 4 subredes. Esta se obtiene con Log_2(Número de subredes en potencia de 2), en este caso: Log_2(4) = 2. 
Entonces nuestras nuevas subredes de la red padre tendrán la mascara de subred /24 + 2 = /26 quedando las siguientes subredes:

- Subred 1: 192.168.0.0/26
- Subred 2: 192.168.0.64/26
- Subred 3: 192.168.0.128/26
- Subred 2: 192.168.0.192/26

Los incrementos en cada red se obtiene con: 2^(32 - CIDR). En este caso: 2^(32-26) = 2^6 = 64.
Entonces cada subred tendrá 64 - 2 hosts.

![[subredes|800]]



## Seguridad y Segmentación
En una red, conviene segmentarla por subredes para ponerlas en diferentes LANS o VLANS. Esto permite que el trafico salga a la capa de red (L3) y permite controlarlo por medio de reglas ACL o Firewalls.
Al segmentar una red, disminuye su dominio de Broadcast y ya no se puede hacer **MAC Spoofing**, **MAC Flooding**ni **MAC address filtering**
![[lans.png]]


## MAC
La dirección MAC está grabada Burned-in Address en una interfaz de red física.
Existen 3 tipos de MACs

1. Ethernet
2. WLAN
3. Bluetooth

En una misma LAN, los hosts se buscan por IPs pero deben saber a qué MAC le corresponde esa IP para mandar el paquete por el switch directamente.
### ARP
Address Resolution Protocol es un mensaje en broadcast de capa 2 que busca averiguar quien tiene cierta IP, mandando un ARP Request en broadcast y el destinatario le responde en Unicast con ARP Reply. }En otras palabras, resuelve una dirección IP a una dirección MAC.

Cada router, Switch y PC tiene su ARP table que mapea las MACs a sus correspondientes IPs. En el caso del Switch, este recuerda a qué puerto tiene cierta MAC.

Existen vectores de ataque en MAC:
- MAC SPOOFING
- MAC FLOODING
- MAC ADDRESS FILTERING


# Protocolo vs implementación vs servicio

En el uso coloquial decimos "el protocolo SSH está escuchando en el 22" como si el protocolo fuera el programa. Pero lo que escucha es el proceso (`sshd`), que es una implementación del protocolo. El protocolo solo dicta cómo deben hablar; no habla por sí mismo.

## Los tres niveles

| Concepto | Qué es | Ejemplo |
|---|---|---|
| **Protocolo** | Conjunto de reglas (una especificación, un RFC). No es programa, no se ejecuta | SSH (RFC 4251-4254) |
| **Implementación** | Un programa que cumple esas reglas | OpenSSH, Dropbear, PuTTY |
| **Servicio / proceso** | Una instancia corriendo de esa implementación, escuchando en un puerto | `sshd` en el puerto 22 |

Analogía: el protocolo son las reglas del ajedrez, la implementación es un programa de ajedrez concreto, el servicio es una partida en curso.

## Por qué importa en ofensiva

- Las vulnerabilidades casi nunca están en el protocolo, están en la **implementación**. Un CVE de OpenSSH 8.5 es un bug de *ese programa*, no de SSH como tal.
- Por eso en enumeración no basta "hay SSH en el 22": necesitas la **implementación + versión exacta** (`nmap -sV` → "OpenSSH 8.9p1 Ubuntu"). Esa cadena es la que buscas en las bases de CVE.
- Un fallo de **protocolo** (en las reglas mismas) es más raro y más grave, porque afecta a todas las implementaciones. Ejemplo: las debilidades de diseño de WEP.


## VPN
Permite acceso remoto a intranets remotas.
- Usualmente usaba el puerto TCP/1723 "PPTP: Point To Point Tunneling Protocol" pero desde el 2012 ya no por su metodo de autenticación crackeable: MSCHAPv2
- O en UDP/50 para IKEv1 e IKEv2

Por lo tanto, los protocolos que se usan para VPN son estos:
- IP (Internet protocol): Puertos IP; 50(ESP)-51(AH). Routea paquetes entre el VPN cliente y el VPN server
- IKE (Internet Key Exchange): Puerto UDP/500. Mantiene una conexión segura entre el VPN cliente y servidor por medio del algoritmo Diffie-Hellman
- ESP (EncapsulatingSecurity Payload): Puertos IP; 50. NAT-T: UDP/4500. Encripta el trafico entre el server y client vpn

