# EJERCICIO #1 

## Encontrar el camino mas corto en un laberinto

Este ejercicio implementa un algoritmo de búsqueda de caminos para encontrar la ruta más corta entre el inicio y el final de un laberinto representado por una matriz de enteros. Usa el algoritmo BFS (Breadth-First Search) para asegurar la mínima distancia.

```go
package main

import (
	"fmt"
)

// Estructura para representar coordenadas en el laberinto
type Point struct {
	x, y int
}

// Define direcciones de movimiento: arriba, abajo, izquierda y derecha
var directions = []Point{
	{-1, 0}, {1, 0}, {0, -1}, {0, 1},
}

// Función para verificar si un punto es válido
func isValid(maze [][]int, visited [][]bool, x, y int) bool {
	return x >= 0 && y >= 0 && x < len(maze) && y < len(maze[0]) && maze[x][y] == 0 && !visited[x][y]
}

// Encuentra la distancia mínima en el laberinto usando BFS
func shortestPath(maze [][]int, start, end Point) int {
	rows, cols := len(maze), len(maze[0])
	visited := make([][]bool, rows)
	for i := range visited {
		visited[i] = make([]bool, cols)
	}

	queue := []Point{start}
	visited[start.x][start.y] = true
	distance := 0

	for len(queue) > 0 {
		size := len(queue)
		for i := 0; i < size; i++ {
			current := queue[0]
			queue = queue[1:]

			if current == end {
				return distance
			}

			for _, dir := range directions {
				newX, newY := current.x+dir.x, current.y+dir.y
				if isValid(maze, visited, newX, newY) {
					visited[newX][newY] = true
					queue = append(queue, Point{newX, newY})
				}
			}
		}
		distance++
	}

	return -1 // No se encontró un camino
}

func main() {
	// 0: camino libre, 1: obstáculo
	maze := [][]int{
		{0, 1, 0, 0, 0},
		{0, 1, 0, 1, 0},
		{0, 0, 0, 1, 0},
		{0, 1, 1, 1, 0},
		{0, 0, 0, 0, 0},
	}
	start := Point{0, 0}
	end := Point{4, 4}

	result := shortestPath(maze, start, end)
	if result != -1 {
		fmt.Printf("La distancia más corta es: %d\n", result)
	} else {
		fmt.Println("No hay camino disponible")
	}
}

```
### Explicacion

1. Point es una estructura que representa coordenadas.
2. directions define los movimientos permitidos.
3. shortestPath implementa BFS para explorar el laberinto.
4. El programa imprime la distancia más corta o un mensaje si no hay un camino disponible.
