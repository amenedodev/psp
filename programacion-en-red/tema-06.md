---
title: Tema 6. Sockets
parent: Programación en red
nav_order: 40
---

# Tema 6. Sockets

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