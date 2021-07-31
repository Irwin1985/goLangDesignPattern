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

### Builder

Usa este patrón cuando necesites crear objetos que comparten características similares, también se les suele llamar *"Familias"*, por ejemplo: *Coche, Camión, Autobus, Motocicleta* son algunos objetos que tienen **Ruedas** y **Asientos**, por lo tanto si en tu aplicación necesitas crear objetos de este tipo entonces te conviene mejor usar un builder para Vehiculos y que cada uno de ellos implemente la interfáz común.

Para crear un patrón builder se necesitan 3 cosas:

- **Producto:** un tipo abstracto del cual derivan todos los productos finales. Para nuestro ejemplo sería el Vehiculo y los tipos concretos serían de este tipo.
- **BuildProcess:** esta sería una interfaz que contiene los métodos de ensamblado para producir el producto final.
- **Director:** esta sería la línea de ensamblaje. Es decir, parte primero con el Producto en bruto y luego le va dando forma llamando a todos y cada uno de los métodos de ensamblado contenidos en el **BuildProcess.**

Veamos un ejemplo:

```Go
package builder

// Como lo dijimos antes, todo parte de un tipo abstracto. En nuestro caso sería Vehiculo ya que:
// Un Coche es un tipo de Vehículo
// Un Camión es un tipo de Vehículo
// Un Autobus es un tipo de Vehículo
// Una Motocicleta es un tipo de Vehículo
// ¿Ya lo vas pillando no?
type VehicleProduct struct {
    Wheels    int 	// Todos tienen ruedas verdad?
    Seats     int	// Todos tienen asiento verdad?
    Structure string	// Esto es solo el tipo o nombre del vehiculo final.
}

// Esta es la interfaz mencionada anteriormente. Todas las estructuras que deseen formar parte de esta cadena de producción necesitan implementar esta interfaz.
type VehicleBuildProcess interface {
    SetWheels() VehicleBuildProcess
    SetSeats() VehicleBuildProcess
    SetStructure() VehicleBuildProcess
    GetVehicle() VehicleProduct
}

// ¿Recuerdas que hablamos de un director? pues esta estructura solo toma la interfaz VehicleBuildProcess
// para poder invocar cada uno de sus métodos de ensamblado.
type VehicleDirector struct {
    product VehicleBuildProcess
}

// Este método es opcional porque bien pudimos haber creado la propiedad "product" de tipo public.
func (f *VehicleDirector) SetBuilder(b VehicleBuildProcess) {
    f.product = b
}

// Construct dirige la línea de ensamblaje.
func (f *VehicleDirector) Construct() {
    f.product.SetSeats()	// Primero creamos los asientos
    f.product.SetWheels()	// Luego creamos las ruedas
    f.product.SetStructure()	// Finalmente le damos un nombre
}

// CarBuilder es la estructura que implementa la interfaz VehicleBuildProcess. Esta es una estructura concreta
type CarBuilder struct {
    v VehicleProduct
}

func (c *CarBuilder) SetWheels() VehicleBuildProcess {
    c.v.Wheels = 4
    return c
}
func (c *CarBuilder) SetSeats() VehicleBuildProcess {
    c.v.Seats = 5
    return c
}
func (c *CarBuilder) SetStructure() VehicleBuildProcess {
    c.v.Structure = "Car"
    return c
}
func (c *CarBuilder) GetVehicle() VehicleProduct {
    return c.v
}

// BikeBuilder es otra estructura que quiere formar parte de la linea de ensamblaje.
// es un tipo de vehículo pero lleva 1 asiento y 2 ruedas.
type BikeBuilder struct {
    v VehicleProduct
}

func (b *BikeBuilder) SetWheels() VehicleBuildProcess {
    b.v.Wheels = 2
    return b
}
func (b *BikeBuilder) SetSeats() VehicleBuildProcess {
    b.v.Seats = 1
    return b
}
func (b *BikeBuilder) SetStructure() VehicleBuildProcess {
    b.v.Structure = "Bike"
    return b
}
func (b *BikeBuilder) GetVehicle() VehicleProduct {
    return b.v
}

// TruckBuilder quiere entrar también en la línea de ensamblado
// al ser un camión tiene 10 ruedas, 3 asientos y un bolquete para la carga.
type TruckBuilder struct {
    v VehicleProduct
}

func (t *TruckBuilder) SetWheels() VehicleBuildProcess {
    t.v.Wheels = 10
    return t
}
func (t *TruckBuilder) SetSeats() VehicleBuildProcess {
    t.v.Seats = 3
    return t
}
func (t *TruckBuilder) SetStructure() VehicleBuildProcess {
    t.v.Structure = "Truck"
    return t
}
func (t *TruckBuilder) GetVehicle() VehicleProduct {
    return t.v
}

```

Vamos a crear unos cuantos vehículos...

```Go
// Creamos un único director
vehicleDirector := builder.VehicleDirector{}

// Crearemos un Coche de 5 asientos
carBuilder := &builder.CarBuilder{}
vehicleDirector.SetBuilder(carBuilder)
vehicleDirector.Construct()

// Obtenemos la instancia de nuestro Coche
car := carBuilder.GetVehicle()
fmt.Printf("Wheels: %d, Seats: %d, Structure: %s\n", car.Wheels, car.Seats, car.Structure)

// Ahora creemos una Motocicleta
bikeBuilder := &builder.BikeBuilder{}
vehicleDirector.SetBuilder(bikeBuilder)
vehicleDirector.Construct()

// Obtenemos nuestra motocicleta
bike := bikeBuilder.GetVehicle()
fmt.Printf("Wheels: %d, Seats: %d, Structure: %s\n", bike.Wheels, bike.Seats, bike.Structure)

// que tal si creamos un camión?
truckBuilder := &builder.TruckBuilder{}
vehicleDirector.SetBuilder(truckBuilder)
vehicleDirector.Construct()

// venga, dame mi camión...!
truck := truckBuilder.GetVehicle()
fmt.Printf("Wheels: %d, Seats: %d, Structure: %s\n", truck.Wheels, truck.Seats, truck.Structure)
```

Todo lo anterior imprime:

```
Wheels: 4, Seats: 5, Structure: Car        
Wheels: 2, Seats: 1, Structure: Bike       
Wheels: 10, Seats: 3, Structure: Truck  
```
