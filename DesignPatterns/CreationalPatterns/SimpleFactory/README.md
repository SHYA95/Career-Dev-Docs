# Simple Factory Pattern

## 📱 Overview
The Simple Factory pattern provides an interface for creating objects without specifying their concrete classes. It encapsulates object creation logic in a single place, making the code more maintainable.

## 🔑 Key Concepts
- Centralizes object creation
- Hides instantiation logic
- Reduces duplication of creation code
- Not technically a design pattern, but a programming idiom

## 💡 Implementation in Swift

```swift
// Product protocol
protocol Shape {
    func draw()
}

// Concrete products
class Circle: Shape {
    func draw() {
        print("Drawing a circle")
    }
}

class Rectangle: Shape {
    func draw() {
        print("Drawing a rectangle")
    }
}

class Triangle: Shape {
    func draw() {
        print("Drawing a triangle")
    }
}

// Simple Factory
class ShapeFactory {
    enum ShapeType {
        case circle
        case rectangle
        case triangle
    }
    
    static func createShape(type: ShapeType) -> Shape {
        switch type {
        case .circle:
            return Circle()
        case .rectangle:
            return Rectangle()
        case .triangle:
            return Triangle()
        }
    }
}

// Client code
let circle = ShapeFactory.createShape(type: .circle)
circle.draw() // Output: Drawing a circle

let rectangle = ShapeFactory.createShape(type: .rectangle)
rectangle.draw() // Output: Drawing a rectangle
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
