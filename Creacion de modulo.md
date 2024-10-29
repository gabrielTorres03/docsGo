# Creacion de modulo Go

Esta es la primera parte de un tutorial que presenta algunas características fundamentales del lenguaje Go. A continuacion se procedera a la creacion de dos modulos que serviran como ejemplo para la demostracion.  El primero es una biblioteca que se pretende que sea importada por otras bibliotecas o aplicaciones. El segundo es una aplicación de llamada que utilizará el primero.

## Iniciar un módulo que otros puedan utilizar

Comience por crear un módulo Go. En un módulo, se recopilan uno o más paquetes relacionados para un conjunto discreto y útil de funciones. Por ejemplo, puede crear un módulo con paquetes que tengan funciones para realizar análisis financieros, de modo que otros usuarios que escriban aplicaciones financieras puedan utilizar su trabajo. Para obtener más información sobre el desarrollo de módulos, consulte Desarrollo y publicación de módulos .

El código Go se agrupa en paquetes, y los paquetes se agrupan en módulos. El módulo especifica las dependencias necesarias para ejecutar el código, incluida la versión de Go y el conjunto de otros módulos que requiere.

A medida que agrega o mejora funciones en su módulo, publica nuevas versiones del módulo. Los desarrolladores que escriben código que llama a funciones en su módulo pueden importar los paquetes actualizados del módulo y probar con la nueva versión antes de ponerlo en uso en producción.

	1. Abra un simbolo de sistema y vaya a su directorio de inicio
	```go
	cd
	```

	En windows: 
	```go
	cd %RUTA DE INICIO%
	```
	
	2. Crea un "greetings" para el codigo fuente de tu modulo Go.
	```go
	Saludos mkdir
	saludos en cd
	```

	3. Inicie su modulo utilizando el comando de Go, go mod init. Ejecute el comando go mod init y asigne la ruta de su módulo (aquí, utilice ) example.com/greetings. Si publica un módulo, debe ser una ruta desde la cual las herramientas de Go puedan descargarlo. Ese sería el repositorio de su código.
	```go
	$ go mod init ejemplo.com/saludos
	```

	El comando go mod init crea un archivo go.mod para realizar un seguimiento de las dependencias de su código. Hasta ahora, el archivo solo incluye el nombre de su módulo y la versión de Go que admite su código. Pero a medida que agrega dependencias, el archivo go.mod enumerará las versiones de las que depende su código. Esto permite que las compilaciones sean reproducibles y le brinda control directo sobre qué versiones de módulo usar.

	4. En su editor de texto, cree un archivo en el que escribir su codigo y llamelo greetings.go.

	5. Pegue el siguiente codigo en su archivo greetings.go y guarde el archivo. 
	
	```go
	paquete de saludos
	
	importar "fmt"

// Hola devuelve un saludo para la persona nombrada.
func Hola(nombre cadena) cadena {
    // Devuelve un saludo que incorpora el nombre en un mensaje.
    mensaje := fmt.Sprintf("Hola, %v. ¡Bienvenido!", nombre)
    mensaje de retorno
}
```

Este es el primer código de tu módulo. Devuelve un saludo a cualquier usuario que lo solicite. Escribirás el código que llama a esta función en el siguiente paso.

En este código, usted:
* Declara un paquete greetings para recopilar funciones relacionadas.
* Implementar una function Hello para devolver el saludo.

