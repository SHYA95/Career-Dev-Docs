# Factory Method Pattern

## 📱 Overview
The Factory Method pattern defines an interface for creating objects but lets subclasses decide which classes to instantiate. It allows a class to defer instantiation to its subclasses, promoting loose coupling and extensibility.

## 🔑 Key Concepts
- Defines an interface for creating objects
- Lets subclasses alter the type of objects created
- Promotes the "dependency inversion principle"
- Connects parallel class hierarchies

## 💡 Implementation in Swift

```swift
// Product protocol
protocol Vehicle {
    func drive()
}

// Concrete products
class Car: Vehicle {
    func drive() {
        print("Driving a car")
    }
}

class Truck: Vehicle {
    func drive() {
        print("Driving a truck")
    }
}

class Motorcycle: Vehicle {
    func drive() {
        print("Riding a motorcycle")
    }
}

// Creator - abstract factory
protocol VehicleFactory {
    // Factory method
    func createVehicle() -> Vehicle
    
    // Default implementation that uses the factory method
    func deliverVehicle()
}

extension VehicleFactory {
    func deliverVehicle() {
        let vehicle = createVehicle()
        print("Preparing the vehicle...")
        vehicle.drive()
        print("Vehicle delivered!")
    }
}

// Concrete creators
class CarFactory: VehicleFactory {
    func createVehicle() -> Vehicle {
        return Car()
    }
}

class TruckFactory: VehicleFactory {
    func createVehicle() -> Vehicle {
        return Truck()
    }
}

class MotorcycleFactory: VehicleFactory {
    func createVehicle() -> Vehicle {
        return Motorcycle()
    }
}

// Client code
let carFactory = CarFactory()
carFactory.deliverVehicle()
// Output:
// Preparing the vehicle...
// Driving a car
// Vehicle delivered!

let truckFactory = TruckFactory()
truckFactory.deliverVehicle()
// Output:
// Preparing the vehicle...
// Driving a truck
// Vehicle delivered!
```
## ⚠️ Considerations
#### Pros
- Eliminates tight coupling between creator and concrete products
- Single Responsibility Principle: Move product creation code to one place
- Open/Closed Principle: Introduce new types of products without breaking existing code
- Provides hooks for subclasses to extend default behavior

#### Cons
- Can lead to a large number of subclasses, increasing code complexity
- Requires creating a new subclass just to change the type of product
- May be overkill for simple creation scenarios

## 🔄 When to Use
- When you don't know the exact types and dependencies of the objects your code will work with
- When you want to provide users of your library or framework with a way to extend its internal components
- When you want to reuse existing objects instead of rebuilding them each time
- When you need to create different products depending on the context or environment

## 🔗 Additional Resources
