# Funciones y Datos

## Múltiples valores de retorno 

Una de las características inusuales de Go es que las funciones y los métodos pueden devolver múltiples valores. Esta forma se puede utilizar para mejorar un par de expresiones idiomáticas torpes en los programas de C: devoluciones de errores en banda como -1for EOF y modificando un argumento pasado por address.

En C, un error de escritura se indica mediante un recuento negativo y el código de error se oculta en una ubicación volátil. En Go, Write puede devolver un recuento y un error: “Sí, escribiste algunos bytes, pero no todos, porque llenaste el dispositivo”. La firma del metodo Write en los archivos del paquete os es:

```go
func (file *File) Write(b []byte) (n int, err error)
``

Como dice la documentación, devuelve la cantidad de bytes escritos y un valor distinto de nulo error cuando n != len(b). Este es un estilo común; consulte la sección sobre manejo de errores para obtener más ejemplos.

Un enfoque similar evita la necesidad de pasar un puntero a un valor de retorno para simular un parámetro de referencia. Aquí hay una función simple para tomar un número de una posición en una porción de bytes, devolviendo el número y la siguiente posición.

```go
func nextInt(b []byte, i int) (int, int) {
    for ; i < len(b) && !isDigit(b[i]); i++ {
    }
    x := 0
    for ; i < len(b) && isDigit(b[i]); i++ {
        x = x*10 + int(b[i]) - '0'
    }
    return x, i
}
```

## Datos

### Asignacion con new

Go tiene dos primitivas de asignación, las funciones integradas newy make. Hacen cosas diferentes y se aplican a diferentes tipos, lo que puede resultar confuso, pero las reglas son simples. Hablemos newprimero de . Es una función integrada que asigna memoria, pero a diferencia de sus homónimas en otros lenguajes, no inicializa la memoria, solo la pone a cero . Es decir, new(T)asigna almacenamiento puesto a cero para un nuevo elemento de tipo Ty devuelve su dirección, un valor de tipo *T. En la terminología de Go, devuelve un puntero a un valor cero recién asignado de tipo T.

Dado que la memoria devuelta por newse pone a cero, es útil organizar al diseñar las estructuras de datos que el valor cero de cada tipo se pueda utilizar sin una inicialización adicional. Esto significa que un usuario de la estructura de datos puede crear una con newy ponerse a trabajar de inmediato. Por ejemplo, la documentación de bytes.Bufferindica que "el valor cero de a Bufferes un búfer vacío listo para usar". De manera similar, sync.Mutexno tiene un constructor o Initmétodo explícito. En cambio, el valor cero de a sync.Mutex se define como un mutex desbloqueado.

La propiedad "el valor cero es útil" funciona de manera transitiva. Considere esta declaración de tipo.

```go
tipo SyncedBuffer struct { 
    bloqueo sincronización.Mutex 
    buffer bytes.Buffer 
}
```

### Constructores

A veces, el valor cero no es suficiente y es necesario un constructor inicializador, como en este ejemplo derivado del paquete os.

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := new(File)
    f.fd = fd
    f.name = name
    f.dirinfo = nil
    f.nepipe = 0
    return f
}
```

Hay mucho código repetitivo. Podemos simplificarlo usando un literal compuesto , que es una expresión que crea una nueva instancia cada vez que se evalúa.

```go
func NewFile(fd int, name string) *File {
    if fd < 0 {
        return nil
    }
    f := File{fd, name, nil, 0}
    return &f
}
```
Tenga en cuenta que, a diferencia de C, es perfectamente aceptable devolver la dirección de una variable local; el almacenamiento asociado con la variable sobrevive después de que la función retorna. De hecho, tomar la dirección de un literal compuesto asigna una instancia nueva cada vez que se evalúa, por lo que podemos combinar estas dos últimas líneas.

```go
    return &File{fd, name, nil, 0}
```








