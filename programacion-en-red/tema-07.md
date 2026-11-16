---
title: Tema 7. Protocolo HTTP (aplicación)
parent: Programación en red
nav_order: 50
---

# Tema 7. Protocolo HTTP (aplicación)

HTTP (Hypertext Transfer Protocol) es un protocolo a nivel de aplicación que permite las transferencias de información en la WWW.

- HTTP/0.9 (1991) primera versión muy básica (sólo disponía de GET) y sólo podía transferir datos sin formato a través de internet. 
- HTTP/1.0 (1996, RFC 1945) mejoró el protocolo permitiendo mensajes con formato (MIME), conteniendo metadatos sobre los datos transferidos y otras mejoras, aunque con grandes carencias como pocos códigos de estado, baja calidad de las autentificaciones, problemas con los proxy’s, etc.
- HTTP/1.1 (1999) que añade, entre otras, las siguientes características:
    - soporte de host’s virtuales (diferentes nombres de dominio alojados bajo una misma IP)
    - capacidad de recuperación de transferencias interrumpidas
    - soporte para transferencias fragmentadas (chunked transfer), un sistema de streaming  que permite enviar la información en fragmentos (cada uno precedido de su tamaño en bytes). Esto permite ir enviando información sin saber su tamaño total, lo cual repercute en una respuesta más rápida en las páginas generadas dinámicamente.
    - soporta conexiones persistentes, lo cual permite enviar varias peticiones (request) bajo una misma conexión reduciendo así tiempos de respuesta y ancho de banda al no tener que crear la misma conexión continuamente.

- HTTP 2.0 (2015-) surge la última versión, y que las últimas versiones de los navegadores ya comienzan a soportar, siendo además compatible con HTTP 1.1.

El protocolo HTTP se basa en peticiones (request) del cliente al servidor y respuestas (response) del servidor al cliente. 

## Mensajes HTTP

Los mensajes HTTP pueden ser peticiones (request) del cliente al servidor o respuestas (response) del servidor al cliente. Ámbos mensajes tienen la misma estructura:


    PrimeraLínea
    [ Cabeceras generales, de entidad y/o específicas CRLF ]
    CRLF
    [ Cuerpo del mensaje ] 


donde la `PrimeraLínea` depende de si es una petición o una respuesta, y `CRLF` es la combinación de `CR` (carriage return, retorno de carro, hex 0D ~13) y `LF` (line feed, salto de línea, hex 0A ~10).

Las cabeceras tienen el formato `nombreCabecera:valorCabecera`, y pueden agruparse en:

- generales: se aplican tanto a peticiones como a respuestas, pero no tienen relación con los datos que se puedan transmitir en el cuerpo
- de entidad: contienen información añadida del cuerpo, como su tamaño o tipo MIME
- de petición: contienen información sobre el recurso pedido o sobre el propio cliente
- de respuesta: contienen información sobre la respuesta o sobre el propio servidor (nombres, versiones, etc).

Algunas **cabeceras generales** (aplicables tanto a peticiones como a respuestas) serían:

- Cache-Control – se usa para indicar directivas que deben ser cumplidas por los mecanismos de cacheo (sobreescriben su comportamiento predeterminado). Su uso es unidireccional, por lo que su uso en una petición no implica que es aplique a su respuesta, y viceversa.
- Connection – permite enviar opciones que queremos enviar en esta conexión particular, pero que no debe ser comunicada por los proxies en futuras conexiones. Por ejemplo, la cabecera Connection: close se envía para que, después de genere la respuesta, se cierre la conexión actual (no se considere persistente). Las aplicaciones que no soporten conexiones persistentes, lo tendrán que enviar en cada mensaje.
- Date – indica la fecha y hora a la que se originó el mensaje
- Transfer-Encoding – indica (si existe) qué tipo de transformación ha sido aplicada al cuerpo del mensaje para ser transmitida entre el origen y el destino. 
- Upgrade – lo usa el cliente para indicar qué otros protocolos de comunicación soporta y quiere usar, si el servidor puede hacerlo. El servidor debe usar este campo si genera una respuesta 101 Switching Protocols para indicar a qué protocolo cambia, p.e.: `Upgrade: HTTP/2.0, SHTTP/1.3, IRC/6.9, RTA/x11`
- Via – usado por proxies y gateways para indicar los protocolos intermedios que se deben usar en las peticiones y en las respuestas.
- Warning – se usa para transportar, en lenguaje natural, información adicional acerca del estado  del mensaje. 

Algunas **cabeceras de entidad**  (aplicables tanto a peticiones como a respuestas) serían:

- Allow – especifica los métodos soportados por la URI (recurso) de la petición: `Allow: GET, HEAD, PUT`
- Content-Encoding – informa de la codificación utilizada en el contenido de la cabecera Content-Type. Se usa generalmente para permitir comprimir un recurso sin perder información, p.e. : `Content-Encoding: gzip`
- Content-Language – describe el idioma del contenido, para poder filtrar contenidos en función del perfil del usuario
- Content-Length – indica el tamaño del contenido en bytes 
- Content-Location – describe un URI distinto del URI de la petición, donde se puede encontrar el recurso en el momento actual.
- Content-MD5 – es el MD5 del contenido, que se puede utilizar para chequear su integridad.
- Content-Type – indica el tipo del cuerpo enviado, p.e. `Content-Type: text/html; charset=ISO-8859-4`
- Expires – indica la fecha y hora a partir de la cual la respuesta se considera antigua
- Last-Modified – Indica la última fecha y hora de modificación del recurso. Dependiendo de la naturaleza del recurso, el servidor intentará deducir su valor: si es un archivo, será la última modificación del sistema de archivos, si es un contenido dinámico, será la fecha más moderna de entre las de sus componentes, si es de una BD, será el último timestamp del registro, etc. 


## Peticiones HTTP (Request)

Una petición al servidor tiene la siguiente forma:

    Método URI-recurso VersiónHTTP
    [ Cabeceras (generales, de entidad o de petición) CRLF ]
    CRLF
    [ Cuerpo del mensaje ]

donde:

- `URI-recurso` identifica el recurso sobre el que actuará el método. Se usará un * cuando la petición no se realice sobre un recurso en concreto sino sobre el servidor en sí.
- `Método` (o verbo) especifica la acción que se realizará en el recurso.
    - GET – Únicamente recupera información, sin ningún efecto colateral sobre los datos
    - HEAD – Es exactamente igual a GET, pero únicamente se devuelve la cabecera, no el contenido
    - POST – Envía datos al servidor para su procesamiento (inserción o actualización). Los datos viajan en el cuerpo de la petición.
    - PUT – Reemplaza la representación del recurso por el nuevo contenido. La diferencia con POST es su idempotencia.
    - PATCH – Similar a PUT pero para realizar modificaciones parciales.  No es idempotente.
    - DELETE – Elimina la representación del recurso indicado en la URI
    - OPTIONS – Describe las opciones disponibles para la comunicación (p.e. los métodos permitidos por el servidor)
    - TRACE – Realiza un mensaje de test sobre la ruta existente hasta el recurso. En concreto, se solicita al servidor que incluya en la respuesta todos los datos que recibe en la petición, que puede tener información que han incluído los servidores intermedios.
    - CONNECT – Sirve para establecer una comunicación bidireccional con el servidor indicado en la URI. Se usa por ejemplo con un proxy para realizar un tunel con SSL.
  
    Se consideran **métodos idempotentes** aquellos cuyos efectos son los mismos realizando 1 o más veces una misma petición. En este caso serían GET, HEAD, PUT, DELETE, OPTIONS y TRACE.

Algunas **cabeceras de petición** serían:

- Accept-Charset – indica qué charset’s se aceptan para la respuesta, p.e. `Accept-Charset: utf-8, iso-8859-1;q=0.5`  (q es el peso (la prioridad)  del elemento, 1 si no se indica)
- Accept-Encoding – indica las codificaciones aceptables del contenido. Puede ser `gzip` (LZ77 – Lempel-Ziv coding), `compress` (LZW), `deflate` (algoritmo de compresión), `br` (algoritmo Brotli), `identity` (sin compresión, siempre aceptable), etc.
- Accept-Encoding: `compress`, `gzip` (ambos tienen la misma prioridad, 1.0) `Accept-Encoding: compress;q=0.5, gzip;q=1.0`	(preferiblemente gzip, 0 indicaría inaceptable)
- Accept-Language – indica los lenguajes naturales que se aceptan para la respuesta, p.e. `Accept-Language: da, en-gb;q=0.8, en;q=0.7` indica que acepta preferiblemente el danes, luego el inglés británico y finalmente cualquier otro tipo de inglés.
- Authorization – Generalmente (no necesariamente) después de recibir un `401 Unauthorized` y posiblemente una respuesta `WWW-Authenticate`, con esta cabecera indicamos las credenciales que contienen la información de autentificación, p.e.: `Authorization: Basic YWRtaW46YWJjMTIzLg==` base64(usuario:password) no es un método de cifrado, solo de compresión. Debe ir sobre HTTPS.

![HTTP Authorization](img/basic-auth.svg)
Imagen obtenida de https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Authentication

- Cookie – el cliente envía una cookie al servidor que previamente éste le había enviado en una cabecera `Set-cookie`. El envío se decide en función del servidor de la petición, la URI y la antigüedad de la cookie.
- From – contiene una dirección de correo electrónico a la que poder recurrir si se detecta algún problema, o bien almacenar en algún log.
- Host – especifica el host y el puerto donde se pedirá el recurso, p.e. `GET /pub/WWW/ HTTP/1.1 Host: www.w3.org`
- Max-Forwards – se indica un número máximo de reenvíos para proporcionar un mecanismo de limitación a los métodos TRACE y OPTIONS. Cada proxy o gateway decrementa el número y no lo reenvía al llegar a cero.
- Proxy-Authorization – contendrá las credenciales para autentificarse contra el proxy
- Referer – permite al cliente especificar, por si le viene bien al servidor, el URI desde el cual se ha obtenido la URI de la petición (no se enviará si la URI pedida se obtuvo de un origen que no tiene su propia URI, p.e. un usuario que la ha tecleado)
- User-Agent – contiene información sobre la aplicación (y subproductos) que originaron la petición:

    `User-Agent:Mozilla/5.0 (Windows NT 6.1; Win64; x64; rv:47.0) Gecko/20100101 Firefox/47.0`   
    `User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X x.y; rv:42.0) Gecko/20100101 Firefox/42.0`


## Respuestas HTTP (Response)

Una respuesta del servidor tiene la siguiente forma: 

    VersiónHTTP CódigoDeEstado Descripción
    [ Cabeceras (generales, de entidad o de respuesta) CRLF ]
    CRLF
    [ Cuerpo del mensaje ]

donde el `Código de estado` es el resultado de la respuesta generada a la petición y `Descripción` es una cadena de texto legible para un humano, y que resume dicho código de estado.

Los códigos de estado HTTP se dividen en cinco categorías:
- 1xx	Respuestas informativas. Se recibió la petición.
- 2xx	Éxito. La acción fue recibida, entendida y aceptada
- 3xx	Redirecciones. Es preciso alguna acción para completar la petición.
- 4xx	Errores del cliente. La petición tiene algún error sintáctico, o no se puede realizar.
- 5xx	Errores del servidor. La petición es válida pero el servidor ha fallado al atenderla.

A continuación se muestran algunos códigos de estado con su descripción correspondiente. Tenga en cuenta que la descripción debería modificarse para adecuarse a las circunstancias específicas. Las aplicaciones HTTP no tienen porque gestionar todos los códigos de estado sino los que tenga sentido gestionar en su ámbito. Lo que si deberían controlar sería, al menos, el primer dígito.

    "100" 	Continue
    "101" 	Switching Protocols

    "200" 	OK
    "201" 	Created
    "202" 	Accepted
    "203" 	Non-Authoritative Information
    "204" 	No Content
    "205" 	Reset Content
    "206" 	Partial Content

    "300" 	Multiple Choices
    "301" 	Moved Permanently
    "302" 	Found
    "303" 	See Other
    "304" 	Not Modified
    "305" 	Use Proxy
    "307" 	Temporary Redirect

    "400" 	Bad Request
    "401" 	Unauthorized
    "402" 	Payment Required
    "403" 	Forbidden
    "404" 	Not Found
    "405" 	Method Not Allowed
    "406" 	Not Acceptable
    "407" 	Proxy Authentication Required
    "408" 	Request Time-out
    "409" 	Conflict
    "410" 	Gone
    "411" 	Length Required
    "412" 	Precondition Failed
    "413"	Request Entity Too Large
    "414"	Request-URI Too Large
    "415" 	Unsupported Media Type
    "416" 	Requested range not satisfiable
    "417" 	Expectation Failed

    "500"  	Internal Server Error
    "501"  	Not Implemented
    "502"  	Bad Gateway
    "503"  	Service Unavailable
    "504"  	Gateway Time-out
    "505" 	HTTP Version not supported

Algunas **cabeceras de respuesta** serían:

- Age – indica el tiempo estimado en segundos desde que se generó la respuesta. Si es 0 posiblemente fue traído del servidor y en caso contrario, será el tiempo que lleva en el proxy cache
- Location – Sirve para redireccionar a otra localización distinta de la URI existente en la petición
- Proxy-Authenticate – Debe indicarse en las respuestas 407 (Proxy Authentication Required), y en su valor estará el esquema y parámetros para autentificarse contra el proxy.
- Retry-After – Se suele usar después de un 503 (Service Unavailable) para indicar cuanto tiempo se espera que el recurso no esté disponible. Su valor puede ser una fecha y hora, o bien el número de segundos.
- Set-cookie – se usa para enviar cookies desde el servidor al cliente

    `Set-Cookie:<nombreCookie>=<valorDeLaCookie>;Expires=<fecha>;Max-Age=<nºdeSegundos>;Domain=<dominio>;Path=<ruta>;Secure; HttpOnly`

    donde se pueden enviar distintas directivas configurando el tiempo de vida de la cookie (`Expires` y `Max-Age`), filtrar los dominios a donde enviarla (`Domain`), las rutas que deben existir en el servidor antes de enviarla (`Path`), si enviarla solo si estamos usando HTTPS (`Secure`), o  enviarla de manera que no sean accesibles desde JavaScript en el cliente para evitar XSS (`HttpOnly`)
- Server – Contiene información sobre el software usado por el servidor que gestiona la petición. Se debe usar con precaución, puesto que revelar el software utilizado le hace más vulnerable a sufrir ataques de agujeros de seguridad conocidos para dicho software.
- Vary – indica un conjunto de cabeceras de la petición que se pueden cachear para usar en las respuestas para futuras peticiones.
- WWW-Authenticate – Debe incluirse en las cabeceras de respuestas 401 Unauthorized. Su valor indicará el esquema de autentificación y sus parámetros (`Basic` sobre HTTPS, `Digest` que encripta las credenciales, etc.)
