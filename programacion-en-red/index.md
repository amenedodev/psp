---
title: Programación en red
nav_order: 20
has_children: true
---

# Protocolos y modelo OSI

Un **protocolo** es un conjunto de normas que han de cumplir las partes involucradas en la comunicación para poder entenderse. Para reducir la complejidad en el diseño de aplicaciones, se dividen en pequeñas partes o **capas**.

La ISO (*International Organization for Standardization*) definió una arquitectura de red teórica de 7 capas: el modelo **OSI** (*Open Systems Interconnection*).

| Capa OSI | Descripción | Modelo TCP/IP |
|---|---|---|
| **Aplicación** | Nivel más cercano al usuario, usado por las aplicaciones para acceder a las funciones de red de alto nivel. | **Aplicación**: HTTP, FTP, Telnet, DHCP, SMTP, SNMP, etc. |
| **Presentación** | Controla la semántica y la sintaxis (que los datos tengan el formato correcto). Realiza conversiones, (de)codificaciones y (des)compresiones, según sea envío o recepción. | *(incluida en **Aplicación**)* |
| **Sesión** | Presta servicios de valor añadido: implementa sesiones para coordinar el diálogo entre aplicaciones, agrupa datos de distintas aplicaciones dirigidas al mismo destino, etc. | *(incluida en **Aplicación**)* |
| **Transporte** | Gestiona el flujo de datos, controlando errores, ordenamiento y duplicados. Puede ofrecer servicio orientado a conexión o no. | **Transporte**: TCP, UDP |
| **Red** | Redirige los datos hacia su destino, eligiendo la mejor ruta posible. | **Red**: IP, ARP, ICMP |
| **Enlace** | Controla el flujo de datos entre nodos contiguos físicamente interconectados. | **Interfaces de red**: Ethernet, drivers de tarjeta |
| **Física** | Transforma los datos en señales físicas y viceversa, según las características eléctricas y funcionales de la línea utilizada. | *(incluida en **Interfaces de red**)* |

## Java y la pila de protocolos

El lenguaje Java proporciona numerosas abstracciones para permitir la comunicación entre dos ordenadores a través de las redes existentes, sin necesidad de conocer sus detalles técnicos.

Las clases que ofrece el lenguaje permiten trabajar a distintos niveles de la pila de protocolos utilizada en una red real, como es **TCP/IP** y **HTTP**