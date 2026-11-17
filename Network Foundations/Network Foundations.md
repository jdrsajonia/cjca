
## Firewalls

- Stateless: filtran paquetes según una regla
- Stateful: recuerdan conexiones
- NGWF: Next gen; controladores con software que separan los datos de su procesamiento (les quita el cerebro a los equipos intermediarios para controlarlos por software)

	![[routerswitch]]

- Firewall de aplicación: se encuentran en la capa 7 como el **Firewall de Windows**

## NAT

Esconde varias IP privadas detrás de una pública.
La más famosas es la PAT y funciona como en el siguiente dibujo

![[pat|800]]]

Cuando un host en una LAN hace una conexión a internet, esta última ve únicamente la dirección pública para poderse routear hacia afuera, pero... hay muchas IPs en la LAN, como diferenciar quien hizo la petición al volver?
La IP privada origen abre un socket en un puerto, como 55555 con la IP pública con 44444 por ejemplo. Entonces la IP pública sale a internet haciendo conexiones desde ese puerto 44444, y todo lo que llegue por ahí, le llegará a la IP privada en 55555. Es un mapeo.


## User-Agent

```HTTP
GET /index.html HTTP/1.1
Host: www.ejemplo.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36
Accept: text/html
Connection: close
```

Encabezado HTTP que sirve para que la app/web consultada sepa quién/qué navegador le está haciendo peticiones.
En este caso el User-Agent es Mozilla/5.0, pero también puede ser curl/8.1, o personalizado.

La aplicación puede filtrar las Request según el User-Agent.

Es fácil de manipular con:
```bash
curl -A "<user-agent>" http://<ip_target>:port/
```


## FTP

File transfer protocolo: sirve para transferir archivos y se sitúa en el puerto 21.
Pero realmente, FTP abre dos canales de comunicación
1. El puerto 21 para comandos y respuestas como USER, PASS, PASV, LIST, RETR
2. El puerto 20 para datos de archivos transferidos. ESTA LA INICIA EL SERVIDOR FTP


### Modo ACTIVO (default)
El puerto 20 para datos transferidos es cuando FTP está en modo ACTIVO, que es cuando FTP maneja ese canal manualmente. Pero esto tiene un problema:
- Cuando estamos detrás de un NAT, si el servidor FTP abre un canal hacia nuestra IP pública sin saber a qué puerto apuntar para comunicarnos con nuestro host privado, los datos no van a llegar.

![[ftptransmision.png]]
### Modo PASIVO 
Se activa con PASV. Al activarlo, el servidor nos devuelve una tupla de valores tipo:
(209,51,188,20,99,7), donde los primeros cuatro valores corresponden a la IP del servidor FTP, cada valor en un byte, formando 209.51.188.20.
Y los dos últimos valores son el puerto de DATOS partido en dos bytes, 99 y 7. Para pasarlo a Decimal se aplica la fórmula *p1x256 + p0* donde p1=99 y p0=7, entonces el puerto de datos sería 25351. Entonces, para conectarse por el canal de datos, se hace con netcat:

```bash
nc 209.51.188.20 25351
```

y para bajar algún archivo como ejemplo, se hace con RETR <nombre del archivo>






