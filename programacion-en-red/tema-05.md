---
title: Tema 5. Protocolos TCP (transporte)
parent: Programación en red
nav_order: 30
---

# Tema 5. Protocolos TCP (transporte)

**TCP** (*Transmission Control Protocol*, definido en el [RFC 793](https://www.rfc-editor.org/rfc/rfc793), actualizado por el [RFC 9293](https://www.rfc-editor.org/rfc/rfc9293)) trabaja a nivel de transporte y proporciona un servicio **confiable** de entrega de paquetes y **orientado a la conexión**. En concreto:

- Garantiza la entrega de los paquetes (datagramas IP).
- Asegura que los datos enviados por las aplicaciones se dividan en paquetes y se vuelvan a unir en el destino en el orden correcto.
- Asegura la integridad de los datos mediante *checksum*.

Como desventajas, es un protocolo más lento y solo admite comunicación punto a punto.

Lo utilizan multitud de protocolos, como FTP, Telnet, HTTP, etc.

# Los puertos

Un **puerto** es un número de dos bytes (16 bits) que permite a un mismo host establecer simultáneamente varias conexiones con otras máquinas. Entre todas las aplicaciones que se ejecutan en un host, el puerto identifica de forma inequívoca con cuál nos estamos comunicando.

Es decir, cuando un paquete llega a una dirección IP (host), se dirige a la aplicación que esté unida (escuchando) en ese puerto. Una buena analogía es un número de teléfono y una extensión.

Solo un proceso puede estar escuchando (unido) a una combinación de IP y puerto con el mismo protocolo. Por tanto, en una misma IP (host o interfaz de red), UDP y TCP pueden usar el mismo número de puerto.

## Categorías de puertos

La IANA (*Internet Assigned Numbers Authority*) estableció tres categorías:

| Categoría | Rango | Descripción |
|---|---|---|
| **Bien conocidos** (*well-known*) | 0 – 1023 | Reservados a protocolos estándar. |
| **Registrados** | 1024 (`0x0400`) – 49151 (`0xBFFF`) | Los utilizan aplicaciones concretas, por ejemplo MySQL (3306). |
| **Privados o dinámicos** (temporales) | 49152 (`0xC000`) – 65535 (`0xFFFF`) | Se asignan dinámicamente cuando una aplicación cliente se conecta a un servidor. |

Ejemplos de puertos bien conocidos:

- Echo: 7
- FTP: 20 (datos) y 21 (control)
- Telnet: 23
- SMTP: 25
- HTTP: 80
- POP3: 110

Pueden consultarse en `/etc/services` (Linux) y en `C:\Windows\System32\drivers\etc\services` (Windows).

## Estados de un puerto

Por lo general, un puerto puede estar en uno de tres estados:

- **Abierto**: hay una aplicación escuchando y acepta conexiones (que, obviamente, podrían ser rechazadas).
- **Cerrado**: no se acepta la conexión, normalmente porque no hay ninguna aplicación escuchando.
- **Bloqueado o filtrado**: no se contesta. Simula que el host está apagado, y es lo que suelen hacer los firewalls.

# Sockets

Un **socket** es cada uno de los extremos de una comunicación a través de una red. Para que haya comunicación existirá un socket en cada lado de la línea de comunicación (en cada host). Cada socket tiene una dirección, que es la combinación de una dirección IP y un puerto.

Existen varios tipos de sockets:

- **Datagram sockets** (sockets no orientados a conexión): usan el protocolo UDP.
- **Stream sockets** (sockets orientados a conexión): usan el protocolo TCP.
- **Raw sockets**: trabajan a nivel IP. Se utilizan en routers y otros dispositivos de red.

Como son de tipos distintos, pueden coexistir dos sockets de distinto tipo en el mismo puerto.

## Sockets en cliente y en servidor

Un **servidor** es un proceso que, al arrancar, crea un socket que se pone a escuchar las posibles conexiones de los clientes para ofrecerles un servicio.

**Servidor TCP**: puede atender a varios clientes simultáneamente creando un proceso o hilo para cada uno y estableciendo una conexión TCP entre ese proceso/hilo y el cliente. Para cada conexión se crea un socket dedicado en el servidor, formando un circuito virtual o sesión TCP con el socket del cliente, es decir, un flujo (*stream*) de bytes full dúplex.

El servidor puede tener varios sockets TCP con la misma IP y puerto local, cada uno en su proceso/hilo atendiendo a su cliente. El sistema operativo los trata como sockets distintos porque no coincide el par de direcciones local/remota: aunque las direcciones locales sean iguales, cambian las remotas. Lo que identifica a un socket TCP es el par de direcciones en su conjunto.

**Servidor UDP**: un socket UDP no puede establecer una conexión, por lo que no crea un proceso/hilo por cliente. El mismo proceso gestiona secuencialmente los paquetes que llegan de todos los clientes a través del mismo socket. Por eso los sockets UDP se identifican únicamente por su dirección local, aunque cada mensaje que llega incluye su propia dirección de origen.

El puerto UDP funciona como una única cola de mensajes que recibe todos los datagramas destinados al programa. Por ello puede recibir mensajes de varios emisores a la vez y admite tanto la comunicación punto a punto como de un punto a varios.

Los puertos TCP son distintos de los UDP y solo admiten comunicación punto a punto, ya que las conexiones TCP se identifican mediante dos pares IP:puerto (los de los dos hosts conectados).