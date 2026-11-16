---
title: Tema 8. Servicions web (API REST)
parent: Programación en red
nav_order: 60
---

# Tema 8. Servicions web

## Introducción

Los servicios web son componentes distribuidos de una aplicación, accesibles externamente, generalmente usados para acceder desde aplicaciones escritas en distintos lenguajes y ejecutadas desde distintas plataformas. Esta independencia se consigue gracias al uso de estándares como los formatos XML y JSON, y protocolos como HTTP.

Son alternativa a otras arquitecturas de computación distribuida como son CORBA (Common Object Request Broker Architecture), RMI (Java Remote Method Invocation) o DCOM (Distributed Component Object Model). Éstas arquitecturas son más eficaces al no estar basadas en texto como los servicios web, y más preparadas para la realización de operaciones comunes como las transacciones. Pero la independencia de la plataforma y la flexibilidad de los servicios web suplen estas carencias.

Existen distintos tipos de arquitecturas para crear un API que nos permita comunicar clientes y servicios a través de la web: <u>SOAP,REST,gRPC,GraphQL</u>, etc.
- GraphQL fue creado originalmente por Facebook en 2015 y es un lenguaje de consultas en donde se define un modelo y con una consulta se obtiene exactamente la información requerida
- gRPC (gRPC Remote Procedure Calls) fue desarrollado originalmente por Google. Basado en el clásico RPC de los años 80, utiliza HTTP/2 como transporte y (opcionalmente) Protocol Buffers como lenguaje de descripción del interfaz y serialización de los datos transportados.
- SOAP son los servicios web tradicionales (~ 1998), y se basan en la exposición de sus interfaces en documentos publicos en un formato XML denominado WSDL (Lenguaje de descripción de servicios web, Web Services Description Language). En dicho documento se incluye por ejemplo la localización del servicio web y el interfaz de sus posibles operaciones (los métodos que podemos invocar, así como sus parámetros, tipos de datos, etc). Esta información sirve para saber cómo utilizarlo, pero no qué es lo que hace.

## REST (Representational State Transfer)

Roy Fielding (uno de los autores de la especificación HTTP) presentó en el año 2000 un estilo de arquitectura para sistemas distribuidos a través del protocolo HTTP denominado REST.  
Define el siguiente conjunto de principios para diseñar APIs:
- Separación de cliente y servidor: sus implementaciones se realizan independientemente y uno no conoce los detalles del otro. Simplemente  los distintos clientes realizan peticiones al servidor, y reciben una respuesta en un formato que ambos conocen (XML, JSON, … )
- Sin estado (stateless): cada petición es independiente y lleva toda la información necesaria, por lo que no es necesario mantener sesiones (el servidor no almacenará información, toda estará en el cliente).
- Cacheable: Las respuestas del servidor llevarán información acerca de si sus datos son cacheables o no, lo cual evitaría repetir conexiones entre cliente y servidor para recuperar un mismo recurso.
- Interfaz uniforme: existirá una interfaz genérica para administrar las interacciones entre el cliente y el servidor. Para ello existirá una manera única de **identificar un recurso (a través de una URI)** y una serie de **operaciones básicas aplicables (métodos o verbos HTTP**). Mediante cabeceras HTTP como Accept o Content-Type se puede especificar la **representación** que nos interesa para transportar el recurso.
- Sistema de capas: el servidor se implementará con una arquitectura en capas donde cada componente está construido sobre su capa anterior, mejorando la escalabilidad y la seguridad.

Cuando un API concreto consigue implementar dichos principios REST se dice que es RESTful.

Es una nueva manera de comunicación basado en modificar las operaciones de las cabeceras HTTP, tratando los recursos como URIs y especificando el método o verbo adecuado.  
Los verbos que se suelen utilizar son los asociados a las operaciones básicas CRUD.  
De la ruta especificada en la URL (endpoint) obtenemos el recurso sobre el que actúa el verbo.


| Método | Acción | Descripción | Ejemplo de URI (endpoint) |
|-|-|-|-|
| GET    | Read    | Obtener uno o varios recursos | `/usuarios` o `/usuarios/{id}` | 
| POST   | Create  | Crear un nuevo recurso | `/usuarios` |
| PUT    | Update  | Reemplazar completamente un recurso | `/usuarios/{id}`|
| PATCH  | Update  | Actualizar parcialmente un recurso | `/usuarios/{id}` |
| DELETE | Delete  | Eliminar un recurso | `/usuarios/{id}` |


Si con el verbo y la URI no es suficiente para realizar la operación, puesto que hay que hacer una inserción o actualización (caso de POST/PUT/PATCH), entonces debemos acceder a los datos que vienen en el cuerpo de la petición. Dichos datos pueden venir en alguno de los siguientes formatos:

| Formato | Descripción | Cabecera | Ejemplo |
|-|-|-|-|
| JSON    | El más utilizado actualmente | `Content-Type: application/json` | `{ "nombre": "Pepe", "edad": 54 }`
| Form Data| Formato clásico de formularios  | `Content-Type: application/x-www-form-urlencoded` | `nombre=Pepe&edad=54`
| XML    | Usado en otros APIs (p.e. SOAP)  | `Content-Type: application/xml` | `<usuario><nombre>Pepe</nombre> <edad>54</edad> </usuario>` |
| Multipart Form Data  | Usado para enviar archivos | `Content-Type: multipart/form-data` |
| Otros | YAML, CSV, TXT, etc.  | `Content-Type: text/plain` `Content-Type: text/csv`, etc. | Datos con separadores, sin estructura, etc. |


### Códigos de respuesta y contenido opcional

Nuestro servicio web REST devolverá siempre un código de respuesta válido, resultado de la acción solicitada. Recordamos de HTTP que dichos códigos pueden pertenecer a una de estas cinco categorías:

- 1xx	Respuestas informativas
- 2xx	Éxito.
- 3xx	Redirecciones
- 4xx	Errores del cliente
- 5xx	Errores del servidor

Algunos de los más utilizados en las respuestas REST son los siguientes:  

| Código | Descripción corta | Significado detallado |
|-|-|-|
| 200 | OK                        | Operación correcta                                          |
| 201 | Created                   | Recurso creado (POST/INSERT)                                |
| 204 | No Content                | Operación correcta que no devuelve ningún contenido (DELETE)|
| 301 | Moved Permanently         | El recurso se ha movido a la URI indicada                   |
| 304 | Not Modified              | El recurso no ha cambiado                                   |
| 400 | Bad request               | Error en la petición del cliente                            |
| 401 | Unauthorized              | No está *autentificado* para acceder al recurso             |
| 403 | Forbidden                 | No está *autorizado* a acceder al recurso                   |
| 404 | Not Found                 | El recurso especificado no existe                           |
| 405 | Method Not Allowed        | Método no permitido                                         |
| 422 | Unprocessable Entity      | Faltan parámetros o son incorrectos (error de validación)   |
| 429 | Too Many Requests         | Usuario ha superado el número límite de peticiones          |
| 500 | Internal Server Error     | Error en el servidor                              |
| 503 | Service Unavailable       | Servidor temporalmente fuera de servicio                    |

Además del código de respuesta, el servicio podrá devolver contenido al cliente si tiene sentido dependiendo de la petición (los recursos pedidos (GET), el código del recurso insertado (POST), etc.) y suelen estar codificados en formato JSON, aunque de nuevo se podrían utilizar otros formatos estructurados similares como YAML o XML.  

Es responsabilidad del cliente analizar adecuadamente este contenido para su interpretación en la aplicación cliente (mostrar un mensaje, poblar una lista, operar con los datos o sacar resúmenes/conclusiones, etc.)

Resumiendo, la información intercambiada entre el cliente y el servidor en un API REST es la siguiente:

| Petición del cliente (Request) | Respuesta del servidor (Response) |
|-|-|
| •  Método HTTP (GET,POST,PUT,PATCH,DELETE)<br> •  URI del recurso<br> • Cabeceras (opcionales)<br> •  Cuerpo (con POST,PUT,PATCH)  | •  Código de estado<br> •  Cabeceras (opcionales)<br> •  Cuerpo (con GET, POST)  |

### Cabeceras HTTP más utilizadas

- Gestión del contenido
  
    | Tipo de cabecera | Cabecera | Descripción |
    |-|-|-|
    | Petición | **Accept-Language**: es-ES | Indica el idioma esperado en la respuesta |
    | Petición | **Accept**: application/json | Indica el tipo MIME en el que espera los datos |
    | Petición | **Content-Type**: application/json | Indica el formato en el que envía el clientelos datos en el cuerpo (PUT/POST) |
    | Respuesta | **Content-Type**: application/json | Indica el formato en el que se devuelven los datos en el cuerpo (GET/POST) |
    | Respuesta | **Last-Modified**: Wed, 21 Feb 2026 10:00:00 GMT | Indica la fecha de última modificación del contenido generado |
    | Respuesta | **Content-Length**: 348 | Indica el tamaño en bytes del contenido generado |

  
- Autorización

    | Tipo de cabecera | Cabecera | Descripción |
    |-|-|-|
    | Respuesta | **WWW-Authenticate**: Basic realm="api" | La envía el servidor junto con un 401 para indicar cómo autentificarse para acceder al recurso (Basic) |
    | Respuesta | **WWW-Authenticate**: Bearer realm="api" | La envía el servidor junto con un 401 para indicar cómo autentificarse para acceder al recurso  (Bearer token) . El token se envía en el cuerpo de la respuesta. |
    | Petición | **Authorization**: Basic base64(usuario:password)  | Indica codificadas (no cifradas) las credenciales para autentificarse (sobre HTTPS) |
    | Petición | **Authorization**: Bearer token | Indica el token obtenido anteriormente por el servidor, para autentificarse |


    Existen muchos sistemas de autentificación y tipos de token (Basic, Bearer, Digest, OAuth2, JWT, ... )

    Un flujo típico con login (Bearer / JWT) sería el siguiente:

   1. Cliente envía credenciales
        ```json
        POST /login HTTP/1.1
        Content-Type: application/json
        {
        "username": "carlos",
        "password": "123456"
        }
        ```
   2. Servidor valida y genera token
   
      2.1. En el body
        ```json
        HTTP/1.1 200 OK
        Content-Type: application/json
        {
        "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "token_type": "Bearer",
        "expires_in": 3600
        }
        ```
      2.2. En una cookie
        ```
        HTTP/1.1 200 OK
        Set-Cookie: access_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...; 
            HttpOnly; 
            Secure; 
            SameSite=Strict; 
            Path=/; 
            Max-Age=3600
        ```

        2.3. Si no puede validar correctamente las credenciales, devolverá un 401 Unauthorized. No confundir con un 403 Forbidden, que significa que el usuario está autentificado pero no tiene permisos.

        ```json
        HTTP/1.1 401 Unauthorized
        Content-Type: application/json
        WWW-Authenticate: Bearer realm="api"
        {
            "error": "Credenciales incorrectas",
            "message": "Usuario y/o contraseña incorrectos"
        }
        ```

    3. Posteriormente el cliente envíará el token en una cabecera, dependiendo de cómo llegó:

       3.1. Si llegó  en el body
        ```json
        Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
        ```
       3.2. Si llegó en una cookie
        ```json
        GET /users HTTP/1.1
        Host: api.midominio.com
        Cookie: access_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
        ```

- Cookies

    | Tipo de cabecera | Cabecera | Descripción |
    |-|-|-|
    | Respuesta | **Set-Cookie**: datosCookie | El servidor envía una cookie al cliente  |
    | Petición | **Cookie**: datosCookie | El cliente envía la cookie almacenada previamente enviada por el servidor |


### Endpoints - URIs con Apache

En el caso de trabajar con el servidor web Apache, para que los endpoints funcionen con ese tipo de URI separadas por barras (evitando pasos por GET tradicionales con clave=valor separados por &'s), hay que modificar el archivo .htaccess (si trabajamos con Apache):

```php
RewriteEngine On
RewriteBase /clientes/

# Redirige todas las peticiones a index.php
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```
- RewriteEngine On: Activa el motor de reescritura de URLs en Apache.
- RewriteBase /: Define la base para las reglas de reescritura, importante si el servicio no está en la raíz del dominio.
- RewriteCond %{REQUEST_FILENAME}: Estas condiciones aseguran que la solicitud no sea para un archivo existente (!-f) o para un directorio existente (!-d).
- RewriteRule ^(.*)$ index.php [QSA,L]: Esta es la regla de reescritura. Captura cualquier solicitud que no sea un archivo o directorio existente y la redirige a index.php. Los parámetros de la URL serán pasados a index.php como parte de la consulta (QSA es para mantener parámetros de la URL existentes).


### Implementación PHP de un servicio web REST

Para implementar un servicio web REST es necesario recoger, al menos, los dos siguientes datos:
```php
$_SERVER['REQUEST_METHOD'] // es el verbo indicado por el cliente
$_SERVER['PATH_INFO'] // es la ruta que indica el recurso sobre el que actuaremos
```

Con las siguientes líneas de código podremos hacer una primera recogida de datos:

```php
$verbo=$_SERVER['REQUEST_METHOD'];
$pathInfo=isset($_SERVER['PATH_INFO'])?trim($_SERVER['PATH_INFO'],'/'):'';  
$rutas=$pathInfo==''?[]:explode('/',$pathInfo);
```
Con estas líneas obtenemos el verbo (p.e. `GET`) y un array `$rutas` con tantos elementos como *tokens* existentes en la URI, p.e. 'usuarios' y '5' si el URI es `/usuarios/5`. Así podremos programar un API REST básico a base de `if's / switch`.  
Posteriormente exploraremos soluciones más avanzadas programando un enrutador y un controlador.

En el caso de necesitar obtener los datos del cuerpo de la petición, lo haremos de la siguiente manera:
```php
$stringDatosCabecera=file_get_contents('php://input',true);
```
En función del formato del envío, los parsearemos con la librería adecuada.
- En el caso de envíos en form-urlencode, el utilizado normalmente en los formularios sencillos, haremos:  
```php
parse_str($stringDatosCabecera, $datos);
```
La llamada a parse_str asume la sintaxis tipo string de consulta de una URL (var1=valor1&var2=valor2&…), crea dichas entradas en el array asociativo pasado como segundo parámetro ($datos).

- En el caso de envíos en JSON, utilizaremos la siguiente función para pasar un string JSON  a un array:  
```php
$datos=json_decode($stringDatosCabecera);
```

- En el caso de otros formatos (XML, CSV, etc.) usaremos las librerías/funciones adecuadas.

Y finalmente, una vez realizada la operación solicitada en la petición, la respuesta de nuestro servicio web PHP utilizará:
```php
http_response_code($codigo);
```
y en el caso de devolver un contenido (GET/POST), será directamente la salida del script (generado habitualmente con `echo`).
Utilizaremos la siguiente función para pasar de array a string JSON:
```php
echo json_encode(['nombre'=>'Pepe','edad'=>54]);
```


### Probar el servicio web

- Desde un navegador probando por GET y por POST (desde un formulario HTML). Con la extensión adecuada (u opciones del propio navegador, p.e. Desarrollador Web del Firefox) existen más posibilidades.
- Utilizando utilidades genéricas que nos permiten probar las funcionalidades de un API REST. Existen diversas soluciones (p.e. Postman, Insomnia), varias extensiones del navegador Chrome (p.e. Advanced REST client), así como extensiones para entornos de programación (p.e. Thunder Client for Visual Studio Code).
- Utilizando desde la línea de comandos el comando **cURL**
- Escribiendo nuestro propio cliente realizando las llamadas HTTP precisas indicando el método y la ruta adecuada, y parseando el resultado devuelto en función del formato utilizado.

### cURL ###

Aplicación de línea de comandos que soporta multitud de protocolos. En este caso nos interesa que soporte HTTP, por lo que será una herramienta muy útil para hacer peticiones personalizadas y poder testear nuestro servicio API REST.

La c que precede a URL puede ser de Client URL, cURL (en inglés pronunciado como "ver"), o incluso que todo el nombre sea un ACL recursivo (Curl URL Request Library) ...

Los parámetros más importantes son los siguientes: (`curl --help all` para ver todo)

- -X o --request: Especifica el método HTTP a usar (GET, POST, PUT, DELETE, etc.).
    - `curl -X POST http://example.com`
- -H o --header: Añade una cabecera HTTP a la solicitud.
    - `curl -H "Content-Type: application/json" http://example.com`
- -d o --data: Envía datos en el cuerpo de la solicitud (útil para POST, PUT, etc.).
    - `curl -X POST -d "nombre=Juan&edad=30" http://example.com`
- -F o --form: Envía datos como un formulario multipart/form-data.
    - `curl -F "nombre=Juan" -F "foto=@imagen.jpg" http://example.com`
- -o o --output: Guarda la respuesta en un archivo.
    - `curl -o respuesta.txt http://example.com`
- -i o --include: Incluye las cabeceras HTTP en la salida.
    - `curl -i http://example.com`
- -u o --user: Especifica credenciales de autenticación básica.
    - `curl -u usuario:contraseña http://example.com`
- -L o --location: Sigue redirecciones (útil si la URL cambia).
    - `curl -L http://example.com`

Y combinando parámetros podemos conseguir:

- POST con JSON y autentificación:
  - `curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer <token>" -d '{"nombre":"Ignacio","edad":21}' http://example.com/api/usuarios`
- Subir archivo:
  - `curl -X POST -F "archivo=@documento.pdf" http://example.com/subir`
- Autentificación básica y seguir redirecciones (Basic Auth, codificada no cifrada, siempre bajo HTTPS): 
  - `curl -u usuario:contraseña -L https://example.com`


### Comparativa entre REST y SOAP

|  |REST|SOAP|
|--------------|----|----|
|**Significado**|Representational State Transfer|Simple Object Access Protocol|
|**Diseño**|Estilo de arquitectura con guías y recomendaciones|Protocolo estándar con reglas bien definidas|
|**Estilo de uso**|Orientado a datos: `/clientes`|Orientado a funciones: `getClientes()`|
|**Complejidad**|Sencillo|Algo más complejo|
|**Rendimiento**|Muy ligero|Pesado, requiere más ancho de banda|
|**Formato de los datos**|JSON, XML, YAML, CSV, RSS, etc.|XML|
|**Sin estado**|Sí|Sí, pero fácil de mantener el estado|
|**Protocolos**|Sólo HTTP|HTTP, SMTP, UDP, etc.|
|**Métodos usados**|GET, POST, PUT, PATCH, DELETE|Suele usar POST|
|**Seguridad**|No (sí con HTTPS / SSL)|Sí|
|**Cacheable**|Sí|No|
|**Usos**|APIs públicas, redes sociales, móviles|Apps empresariales, pasarelas de pago, telecomunicaciones|


### Implementación PHP de un enrutador API REST

Un enrutador API REST gestiona las rutas (**endpoints**) de una aplicación dirigiéndolas al controlador adecuado en función del URI y el método/verbo especificados. De esta manera actúa como intermediario entre el cliente y la lógica de negocio.

 El siguiente código nos permite crear un enrutador API REST sencillo.

 ```php
 <?php
// Router.php

class Router {
    private $rutas = [];
    public function get($path, $callback) { $this->rutas['GET'][$path] = $callback; }
    public function post($path, $callback) { $this->rutas['POST'][$path] = $callback; }
    public function put($path, $callback) { $this->rutas['PUT'][$path] = $callback; }
    public function patch($path, $callback) { $this->rutas['PATCH'][$path] = $callback; }
    public function delete($path, $callback) { $this->rutas['DELETE'][$path] = $callback; }
    
    public function dispatch($method, $path) {
        foreach ($this->rutas[$method]??[] as $route => $callback)
            if (preg_match("@^$route$@", $path, $params)) {
                $respuesta=call_user_func($callback, $params);
                $statusCode=$respuesta['status']??200;
                http_response_code($statusCode);
                echo $respuesta['body']??'';
                return;
            }
        http_response_code(404);
        echo json_encode(['message'=>'Not Found']);
    }
}
```


```php
<?php
// index.php

require 'Router.php';
require 'controladores/ClienteControlador.php';
require 'BD.php';

header('Content-Type: application/json');
$router = new Router();
$controlador=new ClienteControlador();

$router->get('/clientes', function () use ($controlador,$bd) {
    return $controlador->getAll($bd);
});

$router->get('/clientes/(\d+)', function ($params) use ($controlador,$bd) {
    $codCliente=$params[1];
    return $controlador->get($bd,$codCliente);
});

$router->delete('/clientes/(\d+)', function ($params) use ($controlador,$bd) {
    $codCliente=$params[1];
    return $controlador->delete($bd,$codCliente);
});

$router->post('/clientes', function () use ($controlador,$bd) {
    $datos=file_get_contents('php://input');
    switch($_SERVER['CONTENT_TYPE']??$_SERVER['HTTP_CONTENT_TYPE']??'') {
        case 'application/x-www-form-urlencoded':
            parse_str($datos, $cliente);
            $cliente=(object)$cliente;
            break;
        case 'application/json': 
            $cliente=json_decode($datos);
            break;
        default:
            // No se especifica el formato de los datos del body
            // Podríamos asumir uno de los dos anteriores 
            // p.e. curl envía por defecto form-urlencoded con -d
            return;
    }
    return $controlador->post($bd,$cliente);
});

$router->dispatch($_SERVER['REQUEST_METHOD'], $_SERVER['PATH_INFO']??'/');
```


```php
<?php
// controladores/ClienteControlador.php

class ClienteControlador {

    public function getAll($bd) {
        return ['body'=>json_encode($bd->getClientes())];
    }

    public function get($bd,$codCliente) {
        if($cliente=$bd->getCliente($codCliente))
            return ['body'=>json_encode($cliente)];
        return ['status'=>404,'body'=>json_encode(['message'=>'Cliente Not Found'])];
    }

    public function post($bd,$c) {
        //TODO comprobar que $c tiene los campos adecuados
        $codCliente=$bd->touchCliente($c->nombre,$c->apellidos,$c->nif,$c->codProvincia);
        if($codCliente<0)
            return ['status'=>422,'body'=>json_encode(['message'=>'Datos incorrectos'])];
        return ['status'=>201,'body'=>json_encode(['id'=>$codCliente])];
    }

    public function put($bd,$c) {
        //TODO comprobar que $c tiene los campos adecuados
        $codCliente=$bd->touchCliente($c->nombre,$c->apellidos,$c->nif,$c->codProvincia,$c->codCliente);
        if($codCliente<0)
            return ['status'=>422,'body'=>json_encode(['message'=>'Datos incorrectos'])];
        return ['status'=>204, 'body'=>json_encode(['message'=>"Cliente $codCliente actualizado"])];
    }

    public function delete($bd,$codCliente) {
        if($bd->eliminarCliente($codCliente))
            return ['status'=>204];
        return ['status'=>404,'body'=>json_encode(['message'=>'Cliente Not Found'])];
    }
}
```


```sql
-- phpMyAdmin SQL Dump
-- version 5.2.1
-- https://www.phpmyadmin.net/
--
-- Servidor: 127.0.0.1
-- Tiempo de generación: 12-03-2026 a las 19:51:34
-- Versión del servidor: 10.4.32-MariaDB
-- Versión de PHP: 8.2.12

SET SQL_MODE = "NO_AUTO_VALUE_ON_ZERO";
START TRANSACTION;
SET time_zone = "+00:00";

--
-- Base de datos: `clientes`
--
-- --------------------------------------------------------
--
-- Estructura de tabla para la tabla `clientes`
--

CREATE TABLE `clientes` (
  `codCliente` int(11) NOT NULL,
  `nombre` varchar(255) NOT NULL,
  `apellidos` varchar(50) NOT NULL,
  `nif` char(9) NOT NULL,
  `codProvincia` int(11) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish_ci;


INSERT INTO `clientes` (`codCliente`, `nombre`, `apellidos`, `nif`, `codProvincia`) VALUES
(1, 'Pepe', 'López', '12345678Z', 1),
(2, 'María', 'Vila', '87654321X', 4);

--
-- Estructura de tabla para la tabla `provincias`
--

CREATE TABLE `provincias` (
  `codProvincia` int(11) NOT NULL,
  `nombre` varchar(50) NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_spanish_ci;


INSERT INTO `provincias` (`codProvincia`, `nombre`) VALUES
(1, 'A Coruña'),
(2, 'Lugo'),
(3, 'Ourense'),
(4, 'Pontevedra');

--
-- Indices de la tabla `clientes`
--
ALTER TABLE `clientes`
  ADD PRIMARY KEY (`codCliente`),
  ADD KEY `codProvincia` (`codProvincia`);

--
-- Indices de la tabla `provincias`
--
ALTER TABLE `provincias`
  ADD PRIMARY KEY (`codProvincia`);

--
-- AUTO_INCREMENT de la tabla `clientes`
--
ALTER TABLE `clientes`
  MODIFY `codCliente` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=7;

--
-- AUTO_INCREMENT de la tabla `provincias`
--
ALTER TABLE `provincias`
  MODIFY `codProvincia` int(11) NOT NULL AUTO_INCREMENT, AUTO_INCREMENT=5;

--
-- Filtros para la tabla `clientes`
--
ALTER TABLE `clientes`
  ADD CONSTRAINT `clientes_ibfk_1` FOREIGN KEY (`codProvincia`) REFERENCES `provincias` (`codProvincia`);
COMMIT;


```


```php
<?php
// BD.php
class BD extends PDO {
    private const SERVIDOR_BD = 'localhost';
    private const USUARIO_BD = 'root';
    private const PASSWORD_BD = '';
    private const BD='clientes';
    private const DSN='mysql:host='.self::SERVIDOR_BD.';dbname='.self::BD.';charset=utf8mb4';
    
    public function __construct() {
        parent::__construct(self::DSN,self::USUARIO_BD,self::PASSWORD_BD);
    }

    function getClientes($codCliente=''):array {
        if($codCliente=='')
            $WHERE='';
        else
            if(!Validar::esEntero($codCliente))
                return [];
            else
                $WHERE="WHERE codCliente=$codCliente";
        $sql="SELECT codCliente,clientes.nombre AS nombre,apellidos,
        nif,clientes.codProvincia AS codProvincia,provincias.nombre AS nombreProvincia
        FROM clientes LEFT JOIN provincias ON clientes.codProvincia=provincias.codProvincia
        $WHERE ORDER BY apellidos, clientes.nombre";
        if($cursor=$this->query($sql))
            return $cursor->fetchAll(PDO::FETCH_OBJ);
        return [];
    }

    function getCliente($codCliente):object|null {
        $clientes=$this->getClientes($codCliente);
        if(empty($clientes)) 
            return null;
        return $clientes[0];
    }

    function touchCliente($nombre,$apellidos,$nif,$codProvincia,$codCliente=''):int {
        $nombre=strip_tags($nombre);
        $apellidos=strip_tags($apellidos);
        if(!Validar::esNIFvalido($nif)) return -1;
        if(!Validar::esEntero($codProvincia,true)) return -1;
        if($codProvincia=='') $codProvincia=null;

        $params=[$nombre,$apellidos,$nif,$codProvincia];
        if($esEdicion=($codCliente!='')) {
            if(!Validar::esEntero($codCliente)) return -1;
            $params[]=$codCliente;
            $sql='UPDATE clientes SET nombre=?,apellidos=?,nif=?,codProvincia=? WHERE codCliente=?';
        }
        else
            $sql='INSERT INTO Clientes (nombre, apellidos, nif, codProvincia) VALUES (?,?,?,?)';
        $stmt=$this->prepare($sql);
        try {
            $stmt->execute($params);
            if(!$esEdicion) $codCliente=$this->lastInsertId();
        }
        catch (PDOException $ex) { return -1; }
        return $codCliente;
    }

    function eliminarCliente($codCliente):bool {
        if(!Validar::esEntero($codCliente)) return false;
        $stmt=$this->prepare('DELETE FROM clientes WHERE codCliente=?');
        $stmt->execute([$codCliente]);
        return $stmt->rowCount()>0;
    }

    function getProvincias():array {
        $sql='SELECT * FROM provincias ORDER BY nombre';
        return $this->query($sql)->fetchAll(PDO::FETCH_OBJ);
    }
}

try { $bd = new BD(); }
catch (PDOException $ex) { exit('No se ha podido conectar con la BD');  }

class Validar {
    
    static function esEntero($num,$permitirVacio=false):bool {
        if($permitirVacio) if($num=='') return true;
        return preg_match('/^\d+$/',$num);
    }

    static function esNIFvalido($nif):bool {
        //TODO no incluye NIFs válidos que empiecen por letra
        $letras = 'TRWAGMYFPDXBNJZSQVHLCKE';
        $erNIF = "/^(\d{8})([$letras])$/i";
        if(!preg_match($erNIF,$nif,$grupos)) return false;
        $dni=$grupos[1]; $letraNIF=$grupos[2];
        return ($letras[($dni%23)]==strtoupper($nif[8]));
    }
}
```

### Implementación Java de un cliente API REST - GET
A continuación programaremos, utilizando la librería java.net.http (Java 11+), el código mínimo para conectar con un endpoint y realizar las operaciones básicas de un API REST.
 ```java
   public static void main(String[] args) {
        String uri="http://localhost/clientes/index.php/clientes";
        try {
            HttpClient client = HttpClient.newHttpClient();
            HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(uri))
                    .header("Accept", "application/json")
                    .GET()
                    .build();
            HttpResponse<String> response = client.send(request,HttpResponse.BodyHandlers.ofString());
            int statusCode = response.statusCode();
            switch (statusCode) {
                case 200:
                    procesarClientes(response.body()); break;
                case 400:
                    System.out.println("Error en la petición"); break;
               case 500:
                    System.out.println("Error del servidor"); break;
                default:
                    System.out.println("Error inesperado: " + statusCode);
                    System.out.println("Respuesta del servidor:");
                    System.out.println(response.body());
            }
        } catch (Exception e) { System.out.println("Problemas"); }
    }

    private static void procesarClientes(String json) {
        JSONArray arrayClientes = new JSONArray(json);
        ArrayList<Cliente> clientes = new ArrayList<>();
        for (int i = 0; i < arrayClientes.length(); i++) {
            JSONObject c = arrayClientes.getJSONObject(i);
            String nombre = c.getString("nombre");
            String apellidos = c.getString("apellidos");
            String nif = c.getString("nif");
            int codProvincia = c.getInt("codProvincia");
            
            // procesar los datos del cliente
            clientes.add(new Cliente(nombre, apellidos, nif, codProvincia));
        }
        clientes.forEach(System.out::println);
    }
```

### Implementación Java de un cliente API REST - POST

 ```java
     String uri="http://localhost/clientes/index.php/clientes";
        HttpClient client = HttpClient.newHttpClient();

        JSONObject clienteJson = new JSONObject();
        clienteJson.put("nombre", "Ana");
        clienteJson.put("apellidos", "López");
        clienteJson.put("nif", "12345678Z");
        clienteJson.put("codProvincia", 2);

        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(uri))
            .header("Accept", "application/json")
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(clienteJson.toString()))
            .build();

        try {
            HttpResponse<String> response = client.send(
                request,
                HttpResponse.BodyHandlers.ofString()
            );
            switch (response.statusCode()) {
                case 201:
                    JSONObject respuestaJson = new JSONObject(response.body());
                    int idGenerado = respuestaJson.getInt("id");
                    System.out.println("ID generado: " + idGenerado);
                    break;

                case ........
            }
        } catch (Exception e) { System.out.println("Problemas"); }
    }
```


### Probar el cliente API REST con cURL

- curl http://localhost/clientes/index.php/clientes
- curl http://localhost/clientes/index.php/clientes/2
- curl -X DELETE http://localhost/clientes/index.php/clientes/5
- curl -X POST http://localhost/clientes/index.php/clientes
     -d "nombre=Juan&apellidos=C&nif=12345678z&codProvincia=3"
- curl -X POST localhost/clientes/index.php/clientes
     -H "Content-Type:application/json"
     -d "{\"nombre\":\"Ignacio\",\"apellidos\":\"L\",\"nif\":\"12345678Z\",\"codProvincia\":4}"



### Probar desde el navegador

Desde el navegador, sin instalar ninguna extensión, podemos probar obviamente GET (tecleando la URI) y POST haciendo un sencillo formulario como el siguiente:

```php
<?php
require 'BD.php';
$provincias=$bd->getProvincias();
?>
<!DOCTYPE html>
<body>
    <h1>Añadir cliente</h1>
    <form method='POST' action='index.php/clientes'>
        <input name="nombre" placeholder="Nombre"><br>
        <input name="apellidos" placeholder="Apellidos"><br>
        <input name="nif" placeholder="NIF"><br>
        <select name="codProvincia">
            <option></option>
            <?php foreach($provincias as $p):?>
            <option value="<?=$p->codProvincia?>"><?=$p->nombre?></option>
            <?php endforeach?>
        </select>
        <br><input type='submit' value='Añadir'>
    </form>
</body>
```