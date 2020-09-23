# Curso Autenticación de Usuarios con Passport

## Introducción

### Qué es la autenticación y la autorización

_La **autenticación** es la acción de verificar la identidad de un usuario, es decir, verificar si el usuario existe y si en efecto es él. En nuestra app vamos a implentar autenticación usando usuario y contraseña para posteriormente generar un token de autorización._

_La **autorización** es la acción de otorgar permisos de manera limitada a nuestros recursos. A veces queremos otorgar permisos de solo lectura y escritura y a veces queremos otorgar permisos administrativos, esto lo hacemos menejando tokes que le otorgaremos a nuestro servidor._

**Autenticación:** ¿Quién soy yo?
**Autorización:** ¿Que puedo hacer yo?

### Stack de seguridad moderno

_Anteriormente las compañías se comunicaban mediante una intranet, una intranet, a diferencia de internet, es una red privada que funciona dentro de las compañías. En esta red habían protocolos, pero estos se quedaron cortos cuando vino la revolución mobile. Tecnologías como HTML5 empezaron a necesitar cosas que estos protocolos no se la daban. Esto llevó a la necesidad de crear un nuevo stack de autenticación que se compone, generalmente, de tres protocolos:_

- **JSON Web Tokens:** Son un standar de la industria, abierto, que nos permite comunicarnos entre dos clientes de una manera más segura.
- **OAunth 2.0:** Es un standar que permite implementar autorización (NO CONFUNDIR AUTORIZACIÓN CON AUTENTICACIÓN)
- **OpenID Connect:** Es una capa de autenticación que funciona por encima del OAuth 2.0

### Introducción a las sesiones

_Cuando visitamos un sitio web se crea una **petición http**, este protocolo no tiene estado, es decir, diferentes peticiones http nunca comparten información entre si. La manera de compartir información entre dos peticiones http es mediante el uso de una sesión._

_Cuando visitmos un sitio por primera vez se crea una sesión, por más que no estemos autenticados, a medida que vamos haciendo una búsqueda o haciendo lo que hagamos en ese sitio se guardan nuestras prefrerencias en la sesión. Luego esta sesión genera un id que es almacenado en una **cookie**, la cookie es una archivo que se almacena en nuestro navegador para que cuando cerremos el navegador la cookie permanezca con el id de la sesión, así la próxima vez que volvamos el id de la sesión que está guardado en la cookie, se relaciona con la sesión que estaba abierta antes y así carga nuestras preferencias de lo que hicimos la última vez que entramos a ese sitio._

## Conocer que son los JSON Web Tokens

### Anatomía de un JWT

_Un JWT es un standard que nos permite generar demandas entre dos clientes de manera segura. Un JWT consta de **tres partes** generalmente devididas por un punto:_

- **Header:** El header tiene dos atributos **tipo** debe tener el valor de `JWT` y el elgoritmo de encriptación de la firma **alg**, este puede ser asíncrono a síncrono. Los algorítmos asíncronos utilizan dos llaves de encriptación, llave pública - llave privada, donde la llave privada se utiliza para desencriptar y la llave pública para encriptar. En los algorítmos síncronos se utiliza un método de encriptación que encripta y desencripta.

- **Payload:** Es donde guardamos toda la información de nuestro usuario, incluso, todos los scopes de autorización. El payload se compone de algo llamado **Claims**, estos generalmente son representados por tres letras para mantener el JWT muy pequeño. Hay diferentes tipos de claims, están los **Registered Claims** que tienen una definición propia y deben respetarse, los **Public Claims** son una lista que pueden usar y están definidos y los **Private Claims** son los que uno mosmo define para la aplicación

- **Signture:** Se compone del header codificado + el payload codificado, a todo esto se le aplica el algoritmo de encriptación usando un secret, en el caso del algoritmo hs-256 debemos usar un string de 256 bits de longitud.

Un JWT se ve más o menos así:

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ5b2VsIiwiaWF0IjoxNjAwMzA2ODg4fQ.VjsXE1gIuMIyMqd4bIoihcZkkOt2MeHaeDxyKsq1H4o`

### Autenticación tradicional vs JWT

_En la **autenticación tradicional** cuando sucede un proceso de autenticación se crea una sesión, el id de esta sesión es guardado en una cookie que es enviada al nevegador (las cookies se llaman así por las galletas de la fortuna que tienen un mensaje). A partir de aquí todos los request en adelante tienen la cookie que tiene almacenado el id de la sesión y este es usado para verificar la sesión previamente activa. Un **problema** que tiene este es métodos es que en la **SPA**, como el navegador no refrezca, no puede saber si hubo cambios o no en la sesión. Otro problema es que las **rest API no deben tener estado**, pero al generar sesión estas pasan a tener estado por tanto estamos violando ese principio. Otro problema es que es **difícil de escalar** ya que la sesión, que solo existe en una máquina, no fluye mediante los otros clientes. El **control de acceso** siempre requiere que vayamos a base de datos. Cada usuario que genere una sesión generará **más memoria**._

_En la **autenticación con JWT** al secuder el proceso de autenticación se firma un token, a partir de ahí el token es enviado al cliente y este debe ser almacenado en memoria o en una cookie, todos los request de ahí en adelante llevan este token. Una de las grandes **ventajas** es que las SPA ya no requieren del backend para saber si el usuario está autenticado. La otra ventaja es que el backend puede **recibir múltiples request** de múltiples clientes y lo único que le interesa es saber si el token está bien firmado. Finalmente es el **cliente** el que **sabe que permisos tiene** y no tiene que ir hasta base de datos para saber que permisos tiene._

### Firmar y verificar nuestro JWT

_Para **firmar** un JWT se hace uso de una librería llamado nodeJWT, esta tiene un método llamado **sign**, este método recibe como primer attr el **payload** del JWT. Como segundo attr debe recibir el **secret** con que va a ser firmado. Finalmente un tercer argumento que pueden ser **opciones** extra para nuestro firmado del JWT._

_Para **verificar** usamos la misma librería pero con el método **verify**, en el primer attr recibimos el **token** que queremos verificar. Como segundo argumento recibimos el **secret**. Como tercer attr de manera opcional recibimos un callback que nos devuelve el JWT decodificado, podemos omitir este último attr y recibirlo de manera síncrona._

### Server-side vs Client-side sessions

#### ¿Que es una sesión?

_En terminos generales una sesion es una manera de preservar un estado deseado._

#### ¿Que es una sesión del lado del servidor?

_La sesión en el lado del servidor suele ser una pieza de información que se **guarda** **en memoria** o **en** una **base de datos** y esta permite hacerle seguimiento a la información de autenticación, con el fin de identificar al usuario y determinar cuál es el estado de autenticación. Mantener la sesión de esta manera en el lado del servidor es lo que se considera **“stateful”**, es decir que maneja un estado._

#### ¿Qué es una sesión del lado del cliente?

_Las **SPA (Single-page apps)** requieren una manera de saber si el usuario esta autenticado o no. Pero esto no se puede hacer de una manera tradicional porque suelen ser muy **desacopladas con el backend** y no suelen refrescar la página como lo hacen las aplicaciones renderizadas en el servidor._

_**JWT (JSON Web Token)** es un mecanismo de autenticación sin estado, lo que conocemos como **“stateless”**. Lo que significa que **no hay** una **sesión** que exista **del lado del servidor**._

La manera como se comporta la sesión del lado del cliente es:

1. Cuando el usuario hace **“login”** agregamos una bandera para indicar que lo esta.
2. En cualquier punto de la aplicación **verificamos** la **expiración** del token.
3. Si el **token expira**, **cambiamos la bandera** para indicar que el usuario no está logueado.
4. Se suele chequear cuando la ruta cambia.
5. Si el token expiró lo **redireccionamos** a la ruta de **“login”** y **actualizamos** el **estado** como **“logout”**.
6. Se **actualiza** la **UI** para mostrar que el usuario ha cerrado la sesión.

### Buenas prácticas con JWT

_En los últimos años se ha **criticado** fuertemente el uso de JSON Web Tokens como buena practica de seguridad. La realidad es que muchas compañías hoy en día los usan sin ningún problema siguiendo unas buenas practicas de seguridad, que aseguran su uso sin ningún inconveniente._

A continuación se listan unos consejos a tener en cuenta:

- **Evitar almacenar información sensible:** Debido a que los JSON Web tokens son decodificables es posible visualizar la información del payload, por lo que ningún tipo de información sensible debe ser expuesto como contraseñas, keys, etc. Tampoco se debería agregar ninguna información confidencial del usuario.
- **Mantener su peso lo más liviano posible:** Suele tenerse la tentación de guardar toda la información del perfil en el payload del JWT, pero esto no debería hacerse ya que necesitamos que el JWT sea lo más pequeño posible debido a que al enviarse con todos los request estamos consumiendo parte del ancho de banda.
- **Establecer un tiempo de expiración corto:** Debido a que los tokens pueden ser robados si no se toman las medidas correctas de almacenamiento seguro, es muy importante que estos tengan unas expiración corta, el tiempo recomendado es desde 15 minutos hasta un maximo de 2 horas.
- **Tratar los JWT como tokens opacos:** Aunque los tokens se pueden decodificar, deben tratarse como tokens opacos, es decir como si no tuviesen ningún valor legible. Lo mejor, es siempre enviar el token del lado del servidor y hacer las verificaciones allí.
- **¿Donde guardar los tokens?:** Cuando estamos trabajando con SPA (Single Page apps) debemos evitar almacenar los tokens en Local Storage o Session Storage. Estos deben ser almacenados en memoria o en una Cookie, pero solo de manera segura y con el flag httpOnly, esto quiere decir que la cookie debe venir del lado del servidor con el token almacenado.
- **Silent authenticacion vs Refresh tokens:** Debido a que es riesgoso almacenar tokens del lado del cliente, no se deberian usar Refresh Tokens cuando se trabaja solo con una SPA. Lo que se debe implementar es Silent Authentication, para ello se debe seguir el siguiente flujo:
  1. La SPA obtiene un access token al hacer login o mediante cualquier flujo de OAuth.
  2. Cuando el token expira el API retornara un error 401.
  3. En este momento se debe detectar el error y hacer un request para obtener de nuevo un access token.
  4. Si nuestro backend server tiene una sesión valida (Se puede usar una cookie) entonces respondemos con un nuevo access token.

- [ ] Hay que tener en cuenta que para implementar **Silent authentication** y **Refresh tokens**, se require tener un tipo de sesión valida del lado del servidor por lo que en una SPA es posible que sea necesario una especie de backend-proxy, ya que la sesión no debería convivir en el lado del API server. 

- [ ] En el paso 2, si se esta usando alguna librería para manejo de estado como **redux**, se puede implementar un **middleware** que detecte este error y proceda con el paso 3.

## Cómo funcionan las cookies

### ¿Que son las cookies y cómo implementar el manejo de sesión?

_Una **cookie** es un archivo creado por un sitio web que tiene pequeños pedazos de datos almacenados en el. Su propósito principal es identificar al usuario mediante el almacenamiento de su historial._

Tenemos diferentes tipos de cookies:

- **cookies de sesión o session coookies:** tienen un corto tiempo de vida ya que estas son removidas cuando se cierra el tab o el navegador.

- **cookies persistentes o persistences cokies:** se usan generalmente para guardar información del interés del usuario.

- **cookies seguras o security cookies:** almacenan datos de manera cifrada para que terceros no puedan robar la información, estas suelen usarse en conexiones **https**.

Hay unas leyes sobre cookies

1. Simpre se debe avisar al usuario que se está haciendo uso de cookies
2. Es necesario el consentimiento del usuario para implementar el manejo de cookies

Si las cookies son necesarias para la autenticación del usuario o para algún problema de seguridad esas leyes no aplican en estos casos.

### Cookies vs SessionStorage vs LocalStorage

**LocalStorage:** tiene un almacenamiento máximo de `5mb`, otra cosa es que la info almacenada en el localStorage no se va con cada request que hacemos al servidor, esto nos ayuda a reducir la información entre cliente y servidor. La info almacenada en LocalStorage persiste aunque cerremos la ventana de nuestro navegador. Cuando volvamos a entrar en cualquier momento la sesión seguirá allí.

**SessionStorage:** tiene un comportamiento muy similar al localStorage, la diferencia es que la información está disponible por tab o por window, esto quiere decir que cuando cerremos un tab o un window nuestra sesión desaparecerá.

**Cookies:** solo almacenan `4kb`, lo interante de estas es que si se les puede establecer un tiempo de expiración. Para local o session esto lo deberíamos hacer programáticamente. Una de las desventajas que tienen las cookies es que por cada petición que se haga el server, ya sea de imagenes, HTML, etc, las cookies van a ir juntas a la petición, esto ocaciona un gran consumo de datos cada vez que se hacen las peticiones. Una de sus ventajas es que las cookies se pueden hacer seguras mediante un flag llamado **httpOnly**, esto permite que la info de la cookie solo sea accedida y modificada en el servidor.

**¿En que momento usar uno u otro?**

- **Información no sensible:** Podemos almacenarla en **local o session storage**.

- **Información medianamente sensible:** Como nombre de usuario, algunos términos que puedan identificar al usuario, lo más recomendable es usar el **session storage**.

- **Información muy sensible:** Como contraseñas o JWT debemos guardarlo en **cookies** pero siempre teniendo en cuenta al flag httpOnly.


