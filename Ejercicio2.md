# EJERCICIO #2

## Productor-Consumidor con GoRoutines y Canales.

ste ejercicio implementa el patrón productor-consumidor usando goroutines y channels. El productor genera números aleatorios y el consumidor calcula su cuadrado.

```go
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

// Función que genera números aleatorios y los envía a un canal
func producer(ch chan int, wg *sync.WaitGroup, id int) {
	defer wg.Done()
	for i := 0; i < 5; i++ {
		num := rand.Intn(100)
		fmt.Printf("Productor %d produjo: %d\n", id, num)
		ch <- num
		time.Sleep(time.Millisecond * time.Duration(rand.Intn(500)))
	}
}

// Función que consume números y calcula su cuadrado
func consumer(ch chan int, wg *sync.WaitGroup, id int) {
	defer wg.Done()
	for num := range ch {
		fmt.Printf("Consumidor %d recibió: %d y calculó su cuadrado: %d\n", id, num, num*num)
		time.Sleep(time.Millisecond * time.Duration(rand.Intn(500)))
	}
}

func main() {
	rand.Seed(time.Now().UnixNano())

	ch := make(chan int, 5)
	var wg sync.WaitGroup

	// Iniciar productores
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go producer(ch, &wg, i+1)
	}

	// Iniciar consumidores
	for i := 0; i < 2; i++ {
		wg.Add(1)
		go consumer(ch, &wg, i+1)
	}

	// Espera a que los productores terminen y cierra el canal
	go func() {
		wg.Wait()
		close(ch)
	}()

	// Espera a que los consumidores terminen
	wg.Wait()
	fmt.Println("Todos los productores y consumidores han terminado.")
}
```
1. El producer genera un número aleatorio y lo envía a través del canal.
2. El consumer recibe el número y calcula su cuadrado.
3. El canal se cierra después de que todos los productores han terminado.
