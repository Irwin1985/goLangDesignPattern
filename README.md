# goLangDesignPattern

- Patrones de Diseño Creacionales

### Singleton

Este patrón es fácil de recordar, como su nombre lo implica, provee una única instancia de un objeto y garantiza
que no hayan duplicados. Al primera llamada a esta instancia crea el objeto y las llamadas susesivas devuelven
la instancia del objeto creado inicialmente en lugar de uno nuevo.

El Go no tenemos clases ni muchos menos la palabra reservada `static` por lo que implementaremos el patrón Singleton
valiendonos del ámbito de los paquetes, primero creamos una estructura que contenga el objeto el cual queremos garantizar que sea un Singleton durante la ejecución del programa:

```Go
package pkg_singleton

import "fmt"

type Singleton struct {
	count int
}

var instance *Singleton

func GetInstance() *Singleton {
	if instance == nil {
		instance = new(Singleton)
	}
	return instance
}

func (s *Singleton) AddOne() int {
	s.count += 1
	return s.count
}

func (s *Singleton) PrintCount() {
	fmt.Printf("%d\n", s.count)
}
```

Para usarlo haríamos lo siguiente:

```Go
package main

import "pkg_singleton"

func main() {
	s1 := singleton.GetInstance()
	s1.AddOne() // suma 1 a count
	s1.PrintCount()

	s1.AddOne()
	s1.AddOne()
	s1.AddOne()
	s1.AddOne()

	s2 := singleton.GetInstance() // supuestamente invocamos otra instancia pero...
	s2.PrintCount()
}
```
