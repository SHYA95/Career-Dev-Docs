# Abstract Factory Pattern

## 📱 Overview
The Abstract Factory pattern provides an interface for creating families of related or dependent objects without specifying their concrete classes. It ensures that created objects work together consistently.

## 🔑 Key Concepts
- Creates families of related objects
- Enforces consistency among products
- Isolates concrete classes from client code
- Follows both the Dependency Inversion and Open/Closed principles


## 💡 Implementation in Swift

```swift
// Abstract Products
protocol Button {
    func render()
    func onClick()
}

protocol Checkbox {
    func render()
    func toggle()
}

// Concrete Products - iOS Family
class iOSButton: Button {
    func render() {
        print("Rendering an iOS-styled button")
    }
    
    func onClick() {
        print("iOS button clicked")
    }
}

class iOSCheckbox: Checkbox {
    func render() {
        print("Rendering an iOS-styled checkbox")
    }
    
    func toggle() {
        print("iOS checkbox toggled")
    }
}

// Concrete Products - Android Family
class AndroidButton: Button {
    func render() {
        print("Rendering an Android-styled button")
    }
    
    func onClick() {
        print("Android button clicked")
    }
}

class AndroidCheckbox: Checkbox {
    func render() {
        print("Rendering an Android-styled checkbox")
    }
    
    func toggle() {
        print("Android checkbox toggled")
    }
}

// Abstract Factory
protocol UIFactory {
    func createButton() -> Button
    func createCheckbox() -> Checkbox
}

// Concrete Factories
class iOSUIFactory: UIFactory {
    func createButton() -> Button {
        return iOSButton()
    }
    
    func createCheckbox() -> Checkbox {
        return iOSCheckbox()
    }
}

class AndroidUIFactory: UIFactory {
    func createButton() -> Button {
        return AndroidButton()
    }
    
    func createCheckbox() -> Checkbox {
        return AndroidCheckbox()
    }
}

// Client code
class Application {
    private var button: Button
    private var checkbox: Checkbox
    
    init(factory: UIFactory) {
        button = factory.createButton()
        checkbox = factory.createCheckbox()
    }
    
    func createUI() {
        button.render()
        checkbox.render()
    }
    
    func simulateUserInteraction() {
        button.onClick()
        checkbox.toggle()
    }
}

// Usage
// For iOS
let iOSApp = Application(factory: iOSUIFactory())
iOSApp.createUI()
iOSApp.simulateUserInteraction()

// For Android
let androidApp = Application(factory: AndroidUIFactory())
androidApp.createUI()
androidApp.simulateUserInteraction()
```
## ⚠️ Considerations
#### Pros
- Ensures compatibility between created products
- Avoids tight coupling between concrete products and client code
- Single Responsibility Principle: Isolates product creation code
- Open/Closed Principle: Introduce new variants of products without breaking existing code

#### Cons
- Increases complexity of the code due to many interfaces and classes
- Adding new products requires modifying the abstract factory and all its implementations
- Can be overkill for simple creation scenarios

## 🔄 When to Use
- When your code needs to work with various families of related products
- When you want to provide a library of products without exposing implementation details
- When you need to ensure that created products work together consistently
- When you want to swap product families easily at runtime

## 🔗 Additional Resources
