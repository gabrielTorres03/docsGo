#Espacios de trabajo multimodulo

En este apartado presentamos los conceptos básicos de los espacios de trabajo de varios módulos en Go. Con los espacios de trabajo de varios módulos, puedes indicarle al comando Go que estás escribiendo código en varios módulos al mismo tiempo y compilar y ejecutar código fácilmente en esos módulos.

Se crearán dos módulos en un espacio de trabajo compartido de varios módulos, realizará cambios en esos módulos y verá los resultados de esos cambios en una compilación.

Para comenzar, crea un módulo para el código que escribirás.

Abra un símbolo del sistema y cambie a su directorio de inicio.

En Linux o Mac:
```go
$ cd
```
En Windows:
```go
C:\> cd %HOMEPATH%
```

El resto del tutorial mostrará un $ como símbolo del sistema. Los comandos que utilice también funcionarán en Windows.

Desde el símbolo del sistema, cree un directorio para su código llamado espacio de trabajo.
```go
$ mkdir workspace
$ cd workspace
```

##Inicializar el módulo

Nuestro ejemplo creará un nuevo módulo hello que dependerá del módulo golang.org/x/example.

Crea el módulo hola:
```go
$ mkdir hello
$ cd hello
$ go mod init example.com/hello
go: creating new go.mod: module example.com/hello
```

Agregue una dependencia al paquete golang.org/x/example/hello/reverse usando go get.
```go
$ go get golang.org/x/example/hello/reverse
```

Crea hello.go en el directorio hello con el siguiente contenido:
```go
package main

import (
    "fmt"

    "golang.org/x/example/hello/reverse"
)

func main() {
    fmt.Println(reverse.String("Hello"))
}
```

Ahora, ejecute el programa hello:
```go
$ go run .
olleH
```

##Crear el espacio de trabajo

En este paso, crearemos un go.workarchivo para especificar un espacio de trabajo con el módulo.

###Inicializar el espacio de trabajo

En el workspacedirectorio, ejecute:
```go
$ go work init ./hello
```

El comando go work init indica a go que se debe crear un archivo go.work para un espacio de trabajo que contenga los módulos en el ./hello directorio.

El gocomando produce un archivo go.work que se parece a esto:
```go
go 1.18

use ./hello
```

El archivo go.work tiene una sintaxis similar a go.mod.

La directiva go le indica a Go con qué versión de Go se debe interpretar el archivo. Es similar a la directiva del go.mod archivo.

La directiva use le dice a Go que el módulo en el hello directorio debe ser el módulo principal al realizar una compilación.

Entonces en cualquier subdirectorio del modulo workspace estará activo.

###Ejecute el programa en el directorio del espacio de trabajo
 
En el workspacedirectorio, ejecute:
```go
$ go run ./hello
olleH
```

El comando Go incluye todos los módulos del espacio de trabajo como módulos principales. Esto nos permite hacer referencia a un paquete del módulo, incluso fuera del módulo. Si se ejecuta el go runcomando fuera del módulo o del espacio de trabajo, se produciría un error porque el comando go no sabría qué módulos utilizar.

A continuación, agregaremos una copia local del golang.org/x/example/hellomódulo al espacio de trabajo. Ese módulo se almacena en un subdirectorio del go.googlesource.com/examplerepositorio de Git. Luego, agregaremos una nueva función al paquete reverse que podamos usar en lugar de String.

##Descargar y modificar el golang.org/x/example/hellomódulo
En este paso, descargaremos una copia del repositorio Git que contiene el golang.org/x/example/hellomódulo, lo agregaremos al espacio de trabajo y luego le agregaremos una nueva función que usaremos desde el programa hello.

	1. Clonar el repositorio

	Desde el directorio del espacio de trabajo, ejecute el gitcomando para clonar el repositorio:
	```go
	$ git clone https://go.googlesource.com/example
	Cloning into 'example'...
	remote: Total 165 (delta 27), reused 165 (delta 27)
	Receiving objects: 100% (165/165), 434.18 KiB | 1022.00 KiB/s, done.
	Resolving deltas: 100% (27/27), done.
	```

	2. Agregar el módulo al espacio de trabajo
	
	El repositorio de Git se acaba de descargar en ./example. El código fuente del golang.org/x/example/hellomódulo está en ./example/hello. Añádelo al espacio de trabajo:
```go
$ go work use ./example/hello
```

El go work comando use agrega un nuevo módulo al archivo go.work. Ahora tendrá el siguiente aspecto:
```go
go 1.18

use (
    ./hello
    ./example/hello
)
```

El espacio de trabajo ahora incluye tanto el example.com/hellomódulo como el golang.org/x/example/hellomódulo, que proporciona el golang.org/x/example/hello/reversepaquete.

Esto nos permitirá usar el nuevo código que escribiremos en nuestra copia del paquete reverse en lugar de la versión del paquete en el caché del módulo que descargamos con el comando get.

	3. Añade la nueva función.

	Agregaremos una nueva función para revertir un número al golang.org/x/example/hello/reversepaquete.

	Cree un nuevo archivo llamado int.goen el workspace/example/hello/reversedirectorio que contenga el siguiente contenido:
```go
package reverse

import "strconv"

// Int returns the decimal reversal of the integer i.
func Int(i int) int {
    i, _ = strconv.Atoi(String(strconv.Itoa(i)))
    return i
}
```

	4. Modifique el programa hola para utilizar la función.

	Modificar el contenido de workspace/hello/hello.gopara incluir lo siguiente:
```go
package main

import (
    "fmt"

    "golang.org/x/example/hello/reverse"
)

func main() {
    fmt.Println(reverse.String("Hello"), reverse.Int(24601))
}
```

###Ejecutar el código en el espacio de trabajo

Desde el directorio del espacio de trabajo, ejecute
```go
$ go run ./hello
olleH 10642
```

El comando Go encuentra el example.com/hellomódulo especificado en la línea de comando en el hellodirectorio especificado por el go.work archivo y, de manera similar, resuelve la golang.org/x/example/hello/reverseimportación utilizando el go.workarchivo.

go.work Se puede utilizar en lugar de agregar replace directivas para trabajar en múltiples módulos.

Dado que los dos módulos están en el mismo espacio de trabajo, es fácil realizar un cambio en un módulo y usarlo en otro.

###Paso futuro

Ahora bien, para publicar correctamente estos módulos, tendríamos que hacer una publicación del golang.org/x/example/hello módulo, por ejemplo en v0.1.0. Esto se hace normalmente etiquetando una confirmación en el repositorio de control de versiones del módulo. Consulta la documentación del flujo de trabajo de publicación del módulo para obtener más detalles. Una vez que se realiza la publicación, podemos aumentar el requisito del golang.org/x/example/hellomódulo en hello/go.mod:
```go
cd hello
go get golang.org/x/example/hello@v0.1.0
```

De esta manera, el comando go puede resolver correctamente los módulos fuera del espacio de trabajo.