# Como escribir código Go

## Introducción

Este documento demuestra el desarrollo de un paquete Go simple dentro de un módulo y presenta la herramienta go , la forma estándar de buscar, crear e instalar módulos, paquetes y comandos Go.

## Organización del código

Los programas Go se organizan en paquetes. Un paquete es una colección de archivos fuente en el mismo directorio que se compilan juntos. Las funciones, tipos, variables y constantes definidas en un archivo fuente son visibles para todos los demás archivos fuente dentro del mismo paquete.

Un repositorio contiene uno o más módulos. Un módulo es una colección de paquetes Go relacionados que se publican juntos. Un repositorio Go normalmente contiene solo un módulo, ubicado en la raíz del repositorio. Un archivo nombrado go.mod allí declara la ruta del módulo : el prefijo de la ruta de importación para todos los paquetes dentro del módulo. El módulo contiene los paquetes en el directorio que contiene su archivo go.mod, así como los subdirectorios de ese directorio, hasta el siguiente subdirectorio que contiene otro archivo go.mod (si lo hay).

Tenga en cuenta que no es necesario publicar el código en un repositorio remoto antes de compilarlo. Se puede definir un módulo localmente sin pertenecer a un repositorio. Sin embargo, es una buena costumbre organizar el código como si fuera a publicarlo algún día.

La ruta de cada módulo no solo sirve como prefijo de ruta de importación para sus paquetes, sino que también indica dónde go debe buscar el comando para descargarlo. Por ejemplo, para descargar el módulo golang.org/x/tools, el comando go consultaría el repositorio indicado por https://golang.org/x/tools(descrito con más detalle aquí ).

Una ruta de importación es una cadena que se utiliza para importar un paquete. La ruta de importación de un paquete es la ruta de su módulo unida a su subdirectorio dentro del módulo. Por ejemplo, el módulo github.com/google/go-cmpcontiene un paquete en el directorio cmp/. La ruta de importación de ese paquete es github.com/google/go-cmp/cmp. Los paquetes de la biblioteca estándar no tienen un prefijo de ruta de módulo.

## Primer programa

Para compilar y ejecutar un programa simple, primero elija una ruta de módulo (usaremos example/user/hello) y cree un go.modarchivo que la declare:

```go
$ mkdir hello # Alternativamente, clónelo si ya existe en el control de versiones.
$ cd hola
$ go mod init ejemplo/usuario/hola
ir: crear nuevo módulo go.mod: ejemplo/usuario/hola
$ gato go.mod
módulo ejemplo/usuario/hola

ir 1.16
$
```

La primera declaración en un archivo fuente de Go debe ser . Los comandos ejecutables siempre deben utilizar . package namepackage main

A continuación, crea un archivo llamado hello.godentro de ese directorio que contenga el siguiente código Go:

```go
paquete principal

importar "fmt"

función principal() {
    fmt.Println("Hola, mundo.")
}
```

Ahora puedes construir e instalar ese programa con la goherramienta:

```go
$ go install ejemplo/usuario/hola
$
```

Este comando crea el comando hello y produce un binario ejecutable. Luego instala ese binario como $HOME/go/bin/hello(o, en Windows, %USERPROFILE%\go\bin\hello.exe).

El directorio de instalación está controlado por las variables de entorno GOPATH y . Si se establece , los archivos binarios se instalan en ese directorio. Si se establece , los archivos binarios se instalan en el subdirectorio del primer directorio de la lista. De lo contrario, los archivos binarios se instalan en el subdirectorio del directorio predeterminado ( o ). GOBIN GOBINGOPATHbinGOPATHbinGOPATH$HOME/go%USERPROFILE%\go

Puede utilizar el go envcomando para establecer de forma portátil el valor predeterminado de una variable de entorno para comandos go futuros:

```go
$ go env -w GOBIN=/en algún otro lugar/otro/bin
$
```

Para anular la configuración de una variable previamente establecida por go env -w, utilice go env -u:

```go
$ go env -u GOBIN
$
```

Los comandos como go install se aplican dentro del contexto del módulo que contiene el directorio de trabajo actual. Si el directorio de trabajo no está dentro del example/user/hellomódulo, go install puede fallar.

Para mayor comodidad, los comandos go aceptan rutas relativas al directorio de trabajo y, de manera predeterminada, el paquete que se encuentra en el directorio de trabajo actual si no se proporciona otra ruta. Por lo tanto, en nuestro directorio de trabajo, los siguientes comandos son todos equivalentes:

```go
$ go install ejemplo/usuario/hola
```

```go
$go instalar .
```

```go
$ ir a instalar
```

A continuación, ejecutemos el programa para asegurarnos de que funciona. Para mayor comodidad, agregaremos el directorio de instalación a nuestro directorio PATH para que la ejecución de los archivos binarios sea más sencilla:

```go
# Los usuarios de Windows deben consultar /wiki/SettingGOPATH
# para configurar %PATH%.
$ export PATH=$PATH:$(dirname $(go list -f '{{.Target}}' .)) 
$ hola
Hola Mundo.
$
```

Si está utilizando un sistema de control de código fuente, este sería un buen momento para inicializar un repositorio, agregar los archivos y confirmar el primer cambio. Nuevamente, este paso es opcional: no necesita utilizar el control de código fuente para escribir código Go.

```go
$ git init
Se inicializó un repositorio Git vacío en /home/user/hello/.git/
$ git add go.mod hello.go 
$ git commit -m "confirmación inicial"
[master (root-commit) 0b4507d] confirmación inicial
 1 archivo modificado, 7 inserciones(+)
 modo de creación 100644 go.mod hello.go
$
```

El comando go ubica el repositorio que contiene una ruta de módulo determinada solicitando una URL HTTPS correspondiente y leyendo los metadatos integrados en la respuesta HTML (consulte go help importpath). Muchos servicios de alojamiento ya proporcionan esos metadatos para repositorios que contienen código Go, por lo que la forma más sencilla de hacer que su módulo esté disponible para que otros lo utilicen suele ser hacer que la ruta del módulo coincida con la URL del repositorio.

## Importando paquetes desde tu modulo

Escribamos un paquete morestrings y utilicémoslo desde el programa hello. Primero, creamos un directorio para el paquete llamado $HOME/hello/morestrings, y luego un archivo llamado reverse.goen ese directorio con el siguiente contenido:

```go
// El paquete morestrings implementa funciones adicionales para manipular UTF-8
// cadenas codificadas, más allá de lo proporcionado en el paquete "strings" estándar.
paquete morestrings

// ReverseRunes devuelve su cadena de argumentos invertida de izquierda a derecha.
func ReverseRunes(s cadena) cadena {
    r := []runa(s)
    para i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
        r[i], r[j] = r[j], r[i]
    }
    devolver cadena(r)
}
```

Debido a que nuestra función ReverseRunes comienza con una letra mayúscula, se exporta y puede usarse en otros paquetes que importen nuestro paquete morestrings. 

Probemos que el paquete se compila con go build:

```go
$ cd $HOME/hola/morestrings
$ ve a construir
$
```

Esto no generará un archivo de salida, sino que guardará el paquete compilado en la memoria caché de compilación local.

Después de confirmar que el paquete morestrings se compila, vamos a usarlo desde el programa hello. Para ello, modifique el original $HOME/hello/hello.gopara utilizar el paquete morestrings:

```go
paquete principal

importar (
    "fmt"

    "ejemplo/usuario/hola/morestrings"
)

función principal() {
    fmt.Println(morestrings.ReverseRunes("!oG,olleH"))
}
```
Instalar el programa hello: 

```go
$ go install ejemplo/usuario/hola
```

Al ejecutar la nueva version del programa, deberia er un mensaje nuevo e invertido: 

```go
$ hola
Hola, ¡Vamos!
```

## Importacion de paquetes desde modulos remotos

Una ruta de importación puede describir cómo obtener el código fuente del paquete mediante un sistema de control de revisión como Git o Mercurial. La goherramienta utiliza esta propiedad para obtener automáticamente los paquetes de repositorios remotos. Por ejemplo, para utilizar github.com/google/go-cmp/cmpen su programa:

```go
paquete principal

importar (
    "fmt"

    "ejemplo/usuario/hola/morestrings"
    "github.com/google/go-cmp/cmp"
)

función principal() {
    fmt.Println(morestrings.ReverseRunes("!oG,olleH"))
    fmt.Println(cmp.Diff("Hola mundo", "Hola Go"))
}
```

Ahora que tiene una dependencia de un módulo externo, debe descargar ese módulo y registrar su versión en su archivo go.mod. El comando go mod tidy agrega los requisitos de módulo faltantes para los paquetes importados y elimina los requisitos de los módulos que ya no se usan.

```go
$go mod ordenado
ir: encontrar módulo para el paquete github.com/google/go-cmp/cmp
ir: encontré github.com/google/go-cmp/cmp en github.com/google/go-cmp v0.5.4
$ go install ejemplo/usuario/hola
$ hola
Hola, ¡Vamos!
  cadena(
- "Hola Mundo",
+ "Hola Go",
  )
$ gato go.mod
módulo ejemplo/usuario/hola

ir 1.16

Requiere github.com/google/go-cmp v0.5.4
$
```

Las dependencias de los módulos se descargan automáticamente en el pkg/mod subdirectorio del directorio indicado por la variable de entorno GOPATH. Los contenidos descargados para una versión determinada de un módulo se comparten entre todos los demás módulos que requiere esa versión, por lo que el comando go marca esos archivos y directorios como de solo lectura. Para eliminar todos los módulos descargados, puede pasar el indicador -modcache a go clean:

```go
$ go clean -modcache
$
```


