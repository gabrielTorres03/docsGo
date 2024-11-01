# Sintaxis

## Formato

Las cuestiones de formato son las más polémicas, pero las que menos consecuencias tienen. La gente puede adaptarse a distintos estilos de formato, pero es mejor que no tengan que hacerlo, y se dedica menos tiempo al tema si todo el mundo se atiene al mismo estilo. El problema es cómo abordar esta utopía sin una larga guía de estilo prescriptiva.

Con Go, adoptamos un enfoque inusual y dejamos que la máquina se ocupe de la mayoría de los problemas de formato. El gofmtprograma (también disponible como go fmt, que opera a nivel de paquete en lugar de a nivel de archivo fuente) lee un programa Go y emite el código fuente en un estilo estándar de sangría y alineación vertical, manteniendo y, si es necesario, reformateando los comentarios. Si desea saber cómo manejar una nueva situación de diseño, ejecute gofmt; si la respuesta no parece correcta, reorganice su programa (o informe un error sobre gofmt), no lo evite.

Por ejemplo, no es necesario dedicar tiempo a alinear los comentarios en los campos de una estructura. Gofmtlo hará por usted. Dada la declaración

```go
tipo T struct { 
    nombre cadena // nombre del objeto 
    valor int // su valor 
}
``

go fmt alineara las columnas:

```go
tipo T struct { 
    nombre cadena // nombre del objeto 
    valor int // su valor 
}
```

Todo el código Go en los paquetes estándar ha sido formateado con gofmt.

Quedan algunos detalles de formato. Muy brevemente:

### Sangría
Usamos tabulaciones para sangrar y gofmtlas emitimos de forma predeterminada. Use espacios solo si es necesario.

### Longitud de línea

Go no tiene límite de longitud de línea. No te preocupes por desbordar una tarjeta perforada. Si una línea parece demasiado larga, envuélvela y sangra con una tabulación adicional.
#### Paréntesis

Go necesita menos paréntesis que C y Java: las estructuras de control ( if, for, switch) no tienen paréntesis en su sintaxis. Además, la jerarquía de precedencia de operadores es más corta y clara, por lo que

```go
x<<8 + y<<16
```
significa lo que implica el espaciado, a diferencia de lo que ocurre en otros idiomas.

## Comentarios

Go ofrece comentarios de bloque al estilo C /* */y comentarios de línea al estilo C++ //. Los comentarios de línea son la norma; los comentarios de bloque aparecen principalmente como comentarios de paquete, pero son útiles dentro de una expresión o para deshabilitar grandes porciones de código.

## Nombres

Los nombres son tan importantes en Go como en cualquier otro lenguaje. Incluso tienen un efecto semántico: la visibilidad de un nombre fuera de un paquete está determinada por si su primer carácter está en mayúscula. Por lo tanto, vale la pena dedicar un poco de tiempo a hablar sobre las convenciones de nombres en los programas Go.

## Nombres de paquetes

Cuando se importa un paquete, el nombre del paquete se convierte en un descriptor de acceso para el contenido.

```go
importar "bytes"
```

El paquete importador puede hablar de bytes.Buffer. Es útil si todos los que usan el paquete pueden usar el mismo nombre para referirse a su contenido, lo que implica que el nombre del paquete debe ser bueno: corto, conciso, evocador. Por convención, los paquetes reciben nombres en minúsculas, de una sola palabra; no debería haber necesidad de guiones bajos o mayúsculas mixtas. Pequeñez por el lado de la brevedad, ya que todos los que usan su paquete escribirán ese nombre. Y no se preocupe por las colisiones a priori . El nombre del paquete es solo el nombre predeterminado para las importaciones; no necesita ser único en todo el código fuente, y en el raro caso de una colisión, el paquete importador puede elegir un nombre diferente para usar localmente. En cualquier caso, la confusión es rara porque el nombre del archivo en la importación determina exactamente qué paquete se está usando.

## Captadores

Go no proporciona soporte automático para métodos getter y setter. No hay nada de malo en proporcionar métodos getter y setter usted mismo, y a menudo es apropiado hacerlo, pero no es idiomático ni necesario ponerlo Geten el nombre del método getter. Si tiene un campo llamado owner(minúscula, no exportado), el método getter debería llamarse Owner(mayúscula, exportado), no GetOwner. El uso de nombres en mayúsculas para la exportación proporciona el gancho para discriminar el campo del método. Una función setter, si es necesaria, probablemente se llamará SetOwner. Ambos nombres se leen bien en la práctica:

```go
propietario := obj.Owner() 
si propietario != usuario { 
    obj.SetOwner(usuario) 
}
```

## Punto y coma

Al igual que C, la gramática formal de Go utiliza punto y coma para finalizar las instrucciones, pero a diferencia de C, esos puntos y coma no aparecen en el código fuente. En su lugar, el analizador léxico utiliza una regla simple para insertar puntos y coma automáticamente a medida que escanea, por lo que el texto de entrada está prácticamente libre de ellos.

## Estructuras de control

Las estructuras de control de Go están relacionadas con las de C, pero difieren en aspectos importantes. NO HAY BUCLES 'do' O BUCLES 'while'

### If 

En Go, el condicional If se ve asi: 

```go
if x > 0 {
    return y
}
```

### For

En Go, el bucle for se ve asi: 
```go
// Like a C for
for init; condition; post { }

// Like a C while
for condition { }

// Like a C for(;;)
for { }
```

### Switch

El método de Go "switch" es más general que el de C. Las expresiones no necesitan ser constantes ni siquiera números enteros, los casos se evalúan de arriba a abajo hasta que se encuentra una coincidencia y, si switchno tiene expresión, se activa true. Por lo tanto, es posible (y idiomático) escribir una cadena if- else- if- else como switch.

```go
func unhex(c byte) byte { 
    switch { 
    caso '0' <= c && c <= '9': 
        devuelve c - '0' 
    caso 'a' <= c && c <= 'f': 
        devuelve c - 'a' + 10 
    caso 'A' <= c && c <= 'F': 
        devuelve c - 'A' + 10 
    } 
    devuelve 0 
}
```

No existe una clasificación automática, pero los casos se pueden presentar en listas separadas por comas.

```go
func shouldEscape(c byte) bool { 
    switch c { 
    case ' ', '?', '&', '=', '#', '+', '%': 
        devuelve verdadero 
    } 
    devuelve falso 
}
```


