---
title: Variables de entorno, secretos y AWS Parameter Store
tags:
  - variables-de-entorno
  - secretos
  - aws
  - parameter-store
---

## Variable de entorno

Una variable de entorno es una forma de asociar un nombre con un valor para que una aplicación pueda utilizarlo sin tener que escribir ese valor directamente en el código.

### Ejemplo: configuración dentro del código

Si escribimos el valor directamente en el código, tendríamos algo como esto:

```typescript
const databaseHost = "IP-DEV:5432";
```

El problema es que, si queremos ejecutar la aplicación en otro ambiente con sus propia IP, como producción, tendríamos que cambiar ese valor en el código. Esto puede complicar el mantenimiento y provocar errores al desplegar la aplicación.

### Ejemplo: utilizando una variable de entorno

En cambio, podemos utilizar una variable de entorno:

```ts
const databaseHost = process.env.DB_HOST;
```

Y definir su valor fuera del código, por ejemplo, en un archivo `.env`:

```txt
# .env.dev
DB_HOST=IP-DEV:5432
```

De esta manera, el código obtiene el valor a través de la variable `DB_HOST`, sin necesidad de conocer de antemano cuál es su contenido.

### ¿Para qué sirven las variables de entorno?

La principal ventaja es que permiten separar la configuración del código. Así, la misma aplicación puede ejecutarse en diferentes ambientes utilizando distintos valores, sin necesidad de modificar su implementación.

Por ejemplo, en desarrollo podemos utilizar una base de datos local:

```txt
# .env
DB_HOST=localhost:5432
```

Mientras que en producción podemos utilizar la dirección de una base de datos alojada en un servidor:

```txt
# .env
DB_HOST=IP-PROD:5432
```

La aplicación sigue utilizando `process.env.DB_HOST`; lo único que cambia es el valor configurado en cada ambiente.

Esto facilita el mantenimiento, evita modificaciones innecesarias en el código y permite administrar la configuración de forma más flexible.

> **Importante**: las variables de entorno no se cifran ni se protegen automáticamente. Si utilizamos un archivo `.env`, debemos evitar subirlo a un repositorio público y restringir quién puede acceder a él.
>
> Además, `process.env` es una característica de Node.js. En otros lenguajes o entornos de ejecución, la forma de consultar las variables puede ser diferente.

---

## Secretos

Un secreto es un dato sensible que debe protegerse porque, si una persona no autorizada obtiene acceso a él, podría utilizarlo para acceder a sistemas, consultar información privada o realizar operaciones sin permiso.

Imagina que una aplicación necesita conectarse a una base de datos que requiere una contraseña. Si escribimos esa contraseña directamente en el código, cualquier persona que tenga acceso a ese código podría llegar a conocerla y utilizarla.

```typescript
const databasePassword = "MiPassword123";
```

Algunos ejemplos de secretos son:

- Contraseñas de bases de datos.
- Claves privadas y claves de API.
- Tokens de acceso.
- Credenciales de servicios en la nube, como AWS.

Por eso, los secretos deben almacenarse y compartirse mediante mecanismos que permitan controlar su acceso, evitando exponerlos en el código fuente, los repositorios o los registros de la aplicación.

### ¿Por qué es importante gestionar los secretos correctamente?

La exposición de un secreto puede tener consecuencias como:

- Acceso no autorizado: alguien podría utilizar las credenciales para entrar en un sistema.
- Robo o pérdida de información: una persona podría consultar, modificar o eliminar datos.
- Costos inesperados: una clave de AWS expuesta podría utilizarse para crear recursos y generar cargos.
- Compromiso de otros sistemas: si se reutilizan las mismas credenciales en varios servicios, el problema podría extenderse.

No basta con ocultar los secretos. También debemos limitar los permisos de cada credencial, controlar quién puede utilizarla y cambiarla si existe la sospecha de que ha sido expuesta.

### ¿Qué relación tienen los secretos con las variables de entorno?

Una variable de entorno es un mecanismo que permite proporcionar un valor a una aplicación; un secreto es un tipo de información que necesita protección.

Una variable de entorno puede contener un secreto, pero no todas las variables de entorno contienen información sensible.

Por ejemplo, podemos guardar una contraseña en una variable de entorno:

```txt
# .env
DB_PASSWORD="DATABASE-PASSWORD"
```

Y acceder a ella desde una aplicación Node.js:

```node
const  = process.env.DB_PASSWORD;
```

Esto permite mantener la contraseña fuera del código fuente, pero no garantiza que esté protegida. Si el archivo `.env` se publica, los permisos del sistema son incorrectos o el valor aparece en los registros, el secreto podría quedar expuesto.

Por eso, cuando trabajamos con aplicaciones desplegadas en la nube, podemos utilizar servicios especializados para almacenar secretos, cifrar los valores sensibles y controlar qué aplicaciones o usuarios pueden acceder a ellos.

---

## AWS (Amazon Web Services)

AWS es una plataforma de servicios en la nube que permite alojar aplicaciones, utilizar bases de datos, almacenar información y administrar infraestructura sin tener que mantener físicamente todos los servidores.

AWS también ofrece servicios para administrar permisos, cifrar información, gestionar secretos y configuraciones de aplicaciones.

### AWS Systems Manager Parameter Store (SSM)

Es un servicio que permite administrar y operar recursos de AWS. Una de sus funcionalidades es Parameter Store, que permite almacenar parámetros de configuración y secretos de forma centralizada.

Parameter Store permite guardar parámetros para que las aplicaciones autorizadas puedan consultarlos cuando los necesiten, sin tener que incluir sus valores directamente en el código fuente.

**Podemos crear parámetros de la siguiente forma**

_Estos datos son de ejemplo, no significa que deban tener la misma estructura._

```txt
/mi-app/dev/DB_HOST
/mi-app/prod/DB_HOST
/mi-organización/mi-app/prod/DB_PASSWORD
```

Esta estructura permite organizar las configuraciones por aplicación y ambiente.

- `/mi-app/dev/DB_HOST`: dirección de la base de datos de desarrollo.
- `/mi-app/prod/DB_HOST`: dirección de la base de datos de producción.
- `/mi-organización/mi-app/prod/DB_PASSWORD`: si queremos que la contraseña de la base de datos de producción este asociada a una organización o primer nivel, luego a un servicio y por ultimo a un ambiente;

#### ¿Cómo se protegen los valores sensibles?

Parameter Store permite utilizar distintos tipos de parámetros:

- `String`: para valores de texto que no necesitan cifrado mediante Parameter Store.
- `StringList`: para listas de cadenas de texto.
- `SecureString`: para valores sensibles que deben almacenarse cifrados.

Los parámetros `SecureString` utilizan AWS KMS para cifrar sus valores. Además, podemos configurar permisos mediante IAM para controlar quién puede consultar los parámetros y quién puede descifrarlos.

El cifrado no sustituye al control de acceso: debemos configurar correctamente ambos mecanismos.

#### Ventajas de gestionar los secretos con Parameter Store

**Separación del código**

Evitamos incluir contraseñas y claves directamente en el código fuente o en el repositorio.

**Centralización**

Podemos administrar las configuraciones desde un servicio común, en lugar de distribuirlas manualmente entre diferentes archivos y aplicaciones.

**Separación de ambientes**

Desarrollo, pruebas y producción pueden tener sus propias configuraciones y credenciales, reduciendo el riesgo de mezclar datos o utilizar credenciales incorrectas.

**Control de acceso**

Mediante IAM, podemos limitar qué usuarios, roles y aplicaciones tienen permiso para consultar determinados parámetros.

**Cifrado de valores sensibles**

Podemos almacenar contraseñas y otros valores sensibles como `SecureString`, utilizando AWS KMS para su cifrado.

**Mantenimiento más sencillo**

Cambiar una configuración no requiere necesariamente modificar el código de la aplicación. Podemos actualizar el parámetro y hacer que la aplicación obtenga el nuevo valor según su implementación y su estrategia de lectura o caché.

Nota: estas ventajas dependen de configurar correctamente los permisos, el cifrado y el acceso a los parámetros. Utilizar AWS no garantiza automáticamente que una aplicación sea segura.
