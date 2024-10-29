# Introduccion a Go

Go es un lenguaje de programacion COMPILADO Y TIPADO ESTATICO. El lenguaje a menudo se conoce como Golang debido al nombre de la web oficial, golang.org, pero el nombre propio es go.

## Paquetes

Las aplicaciones de Go están organizadas en paquetes. Un paquete es una colección de archivos fuente ubicados en el mismo directorio. Todos los archivos de origen en un directorio deben compartir el mismo nombre de paquete.

Lo normal es que el nombre del paquete sea el último directorio en la ruta de importación. Por ejemplo, el paquete math/rand importaría todas las funciones, tipos, variables, constantes del paquete rand, cuyos nombres comienzan con una letra mayúscula.

El estilo de nomenclatura recomendado en Go es que los identificadores se nombran usando camelCase, excepto aquellos destinados a ser accesibles entre paquetes, que deberían ser PascalCase.

Un ejemplo de definición de un paquete en código sería:

```go
package prove
```

## Variables

Go usa tipos estáticos, lo que significa que todas las variables deben tener un tipo definido en cuando la aplicación se compile.
Las variables se pueden definir especificando explícitamente un tipo:

```go
var explicit init
```

Tambien puede usar un inicializador y el compilador asignara el tipo de variable para que coincida con el tipo del inicializador.

```go
implicit := 10
```

Una vez declaradas, a las variables se les pueden asignar valores usando el operador =, pero el tipo de una variable nunca puede cambiar.

```go
number := 1 // Se le asigna un valor inicial de 1
number = 2  // Se edita a valor 2

number = true // Esto provoca un error porque el tipo de valor que se intenta guardar es booleano no un int
```

## Constantes

Las constantes contienen datos al igual que las variables, pero su valor no se puede cambiar durante la ejecución del programa.

Las constantes se definen mediante la palabra clave const y pueden ser números, caracteres, cadenas o valores booleanos:

```go
const Name = "FuenRob"
const Age = 29
```

## Funciones

Las funciones en Go aceptan cero o más parámetros. Los parámetros deben escribirse explícitamente, no hay inferencia de tipo.

Los valores se devuelven de las funciones utilizando la palabra clave return.

Una función se invoca especificando el nombre de la función y pasando argumentos para cada uno de los parámetros de la función.

```go
package wellcome

// Función pública
func Hello (name string) string {
    return hi(name)
}

// Función privada
func hi (name string) string {
    return "hi " + name
}
```

## Comentarios

Hay dos formas de poner comentarios en Go:
comentarios de una sola línea que están precedidos por // y bloques de comentarios de varias líneas que están envueltos con /* y */.



