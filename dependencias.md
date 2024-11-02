# Gestion de dependencias

Cuando su código utiliza paquetes externos, esos paquetes (distribuidos como módulos) se convierten en dependencias. Con el tiempo, es posible que deba actualizarlos o reemplazarlos. Go ofrece herramientas de administración de dependencias que lo ayudan a mantener seguras sus aplicaciones Go a medida que incorpora dependencias externas.

En este tema se describe cómo realizar tareas para administrar las dependencias que se toman en el código. La mayoría de estas tareas se pueden realizar con herramientas de Go. En este tema también se describe cómo realizar algunas otras tareas relacionadas con las dependencias que pueden resultar útiles.

## Flujo de trabajo para el uso y la gestion de dependencias

Puede obtener y usar paquetes útiles con las herramientas de Go. En pkg.go.dev , puede buscar paquetes que le resulten útiles y luego usar el gocomando para importar esos paquetes a su propio código para llamar a sus funciones.

A continuación, se enumeran los pasos de administración de dependencias más comunes. Para obtener más información sobre cada uno de ellos, consulte las secciones de este tema.

1. Localice paquetes utiles en pkg.go.dev

2. Importa los paquetes que quieras en tu codigo.

3. Agregue su código a un módulo para realizar un seguimiento de dependencias (si aún no está en un módulo). Consulte Habilitar el seguimiento de dependencias

4. Agrega paquetes externos como dependencias para que puedas administrarlos.

5. Actualice o degrade las versiones de dependencia según sea necesario a lo largo del tiempo.

## Gestion de dependencias como modulos

En Go, las dependencias se administran como módulos que contienen los paquetes que se importan. Este proceso es compatible con:

* Un sistema descentralizado para publicar módulos y recuperar su código. Los desarrolladores ponen sus módulos a disposición de otros desarrolladores para que los utilicen desde su propio repositorio y los publiquen con un número de versión.

* Un motor de búsqueda de paquetes y un navegador de documentación (pkg.go.dev) en el que puede encontrar módulos. Consulte Localización e importación de paquetes útiles .

* Convención de numeración de versiones de módulos que le ayudará a comprender las garantías de estabilidad y compatibilidad con versiones anteriores de un módulo. Consulte Numeración de versiones de módulos .

* Herramientas de Go que facilitan la gestión de dependencias, incluida la obtención del código fuente de un módulo, la actualización, etc. Consulte las secciones de este tema para obtener más información.

## Localizacion e importacion de paquetes utiles

Puedes buscar en pkg.go.dev para encontrar paquetes con funciones que puedan resultarte útiles.

Cuando haya encontrado un paquete que desee utilizar en su código, busque la ruta del paquete en la parte superior de la página y haga clic en el botón Copiar ruta para copiar la ruta en el portapapeles. En su propio código, pegue la ruta en una declaración de importación, como en el siguiente ejemplo:

```go
import "rsc.io/quote"
```

Una vez que el código importe el paquete, habilite el seguimiento de dependencias y obtenga el código del paquete para compilarlo. Para obtener más información, consulte Habilitar el seguimiento de dependencias en el código y Agregar una dependencia .

## Habilitar el seguimiento de dependencias en su codigo

ara realizar un seguimiento y administrar las dependencias que agrega, comience colocando su código en su propio módulo. Esto crea un archivo go.mod en la raíz de su árbol de código fuente. Las dependencias que agregue se incluirán en ese archivo.

Para agregar su código a su propio módulo, utilice el go mod comando init. Por ejemplo, desde la línea de comandos, cambie al directorio raíz de su código y luego ejecute el comando como en el siguiente ejemplo:

```go
$ go mod init ejemplo/mimodulo
```

El argumento go mod init es la ruta del modulo. Si es posible, la ruta del módulo debe ser la ubicación del repositorio de su código fuente.

Si al principio no sabe la ubicación final del repositorio del módulo, utilice un sustituto seguro. Puede ser el nombre de un dominio que posea u otro nombre que controle (como el nombre de su empresa), junto con una ruta que siga el nombre del módulo o el directorio de origen. Para obtener más información, consulte Asignar un nombre a un módulo .

A medida que utiliza las herramientas Go para administrar dependencias, las herramientas actualizan el archivo go.mod para mantener una lista actualizada de sus dependencias.

Cuando agrega dependencias, las herramientas de Go también crean un archivo go.sum que contiene las sumas de comprobación de los módulos de los que depende. Go utiliza esto para verificar la integridad de los archivos de módulos descargados, especialmente para otros desarrolladores que trabajan en su proyecto.

Incluya los archivos go.mod y go.sum en su repositorio con su código.

## Nombrar un modulo 

Cuando se ejecuta go mod initpara crear un módulo para realizar un seguimiento de las dependencias, se especifica una ruta de módulo que sirve como nombre del módulo. La ruta del módulo se convierte en el prefijo de la ruta de importación para los paquetes del módulo. Asegúrese de especificar una ruta de módulo que no entre en conflicto con la ruta de módulo de otros módulos.

Como mínimo, la ruta de un módulo solo debe indicar algo sobre su origen, como el nombre de una empresa, un autor o un propietario. Pero la ruta también puede ser más descriptiva sobre lo que es o hace el módulo.

La ruta del módulo normalmente tiene el siguiente formato:
```go
<prefix>/<descriptive-text>
```

### Prefijos de ruta de modulos reservados

Go garantiza que las siguientes cadenas no se utilizarán en los nombres de los paquetes.

* test– Puede usarlo testcomo prefijo de ruta de módulo para un módulo cuyo código esté diseñado para probar localmente funciones en otro módulo.

Utilice el testprefijo de ruta para los módulos que se crean como parte de una prueba. Por ejemplo, la prueba en sí podría ejecutarse go mod init testy luego configurar ese módulo de alguna manera particular para realizar la prueba con una herramienta de análisis de código fuente de Go.

* example– Se utiliza como prefijo de ruta de módulo en alguna documentación de Go, como en los tutoriales en los que se crea un módulo solo para rastrear dependencias.

Tenga en cuenta que la documentación de Go también se utiliza example.compara ilustrar cuándo el ejemplo podría ser un módulo publicado.

## Agregar una dependencia

Una vez que importe paquetes desde un módulo publicado, puede agregar ese módulo para administrarlo como una dependencia usando el go get comando .

El comando hace lo siguiente:

* Si es necesario, agrega requiredirectivas a su archivo go.mod para los módulos necesarios para crear paquetes nombrados en la línea de comandos. Una directiva require rastrea la versión mínima de un módulo del que depende su módulo.

* Si es necesario, descarga el código fuente del módulo para que puedas compilar los paquetes que dependen de él. Puede descargar módulos desde un proxy de módulo como proxy.golang.org o directamente desde repositorios de control de versiones. El código fuente se almacena en caché localmente.

A continuación se describen algunos ejemplos.

* Para agregar todas las dependencias de un paquete en su módulo, ejecute un comando como el siguiente ("." se refiere al paquete en el directorio actual):

```go
$ go get .
```

* Para agregar una dependencia específica, especifique su ruta de módulo como argumento al comando.

```go
$ go get example.com/theirmodule
```

El comando también autentica cada módulo que descarga. Esto garantiza que no haya cambiado desde que se publicó el módulo. Si el módulo ha cambiado desde que se publicó (por ejemplo, el desarrollador cambió el contenido de la confirmación), las herramientas de Go mostrarán un error de seguridad. Esta comprobación de autenticación lo protege de los módulos que podrían haber sido manipulados.

## Sincronizar las dependencias del codigo

Puede asegurarse de administrar las dependencias de todos los paquetes importados de su código y, al mismo tiempo, eliminar las dependencias de los paquetes que ya no importa.

Esto puede ser útil cuando ha estado realizando cambios en su código y dependencias, posiblemente creando una colección de dependencias administradas y módulos descargados que ya no coinciden con la colección específicamente requerida por los paquetes importados en su código.

Para mantener ordenado el conjunto de dependencias administradas, utilice el go mod tidycomando. Con el conjunto de paquetes importados en su código, este comando edita su archivo go.mod para agregar módulos que son necesarios pero faltan. También elimina módulos no utilizados que no proporcionan ningún paquete relevante.

El comando no tiene argumentos excepto un indicador, -v, que imprime información sobre los módulos eliminados.

```go
$ go mod tidy
```

## Eliminar una dependencia

Cuando su código ya no utiliza ningún paquete en un módulo, puede dejar de rastrear el módulo como una dependencia.

Para dejar de rastrear todos los módulos no utilizados, ejecute el comando go mod tidy. Este comando también puede agregar dependencias faltantes necesarias para crear paquetes en su módulo.

```go
$ go mod tidy
```

Para eliminar una dependencia específica, utilice el comando go get, especificando la ruta del módulo y agregando @none, como en el siguiente ejemplo:

```go
$ go get example.com/theirmodule@none
```

El comando go get tambien degradara o eliminara otras dependencias que dependen del modulo eliminado.


