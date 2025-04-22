# Singleton Pattern

## 📱 Overview
The Singleton pattern ensures a class has only one instance and provides a global point of access to it.
This is useful when exactly one object is needed to coordinate actions across the system.

## 🔑 Key Concepts
- Ensures a class has just a single instance
- Provides a global access point to that instance
- Lazy initialization (typically)
- Thread safety considerations


## 💡 Implementation in Swift

### Basic Implementation

```swift
class Singleton {
    // The static property that holds the singleton instance
    static let shared = Singleton()
    
    // Private initializer to prevent creating new instances
    private init() {
        // Initialization code
        print("Singleton instance initialized")
    }
    
    // Business methods
    func doSomething() {
        print("Singleton is doing something")
    }
}

// Usage
func testSingleton() {
    // Access the singleton instance
    let instance = Singleton.shared
    
    // Use the singleton
    instance.doSomething()
    
    // Trying to create another instance would fail:
    // let anotherInstance = Singleton() // Error: 'Singleton' initializer is inaccessible
    
    // Both variables reference the same instance
    let anotherReference = Singleton.shared
    print("Are they the same instance? \(instance === anotherReference)") // true
}
```

### Modern Swift Implementation with Property Wrapper
```swift
@propertyWrapper
struct Singleton<T> {
    private static var instance: T?
    private static var creator: (() -> T)?
    
    var wrappedValue: T {
        mutating get {
            if Singleton.instance == nil {
                Singleton.instance = Singleton.creator?()
            }
            return Singleton.instance!
        }
    }
    
    init(_ creator: @escaping () -> T) {
        Singleton.creator = creator
    }
}

// Usage with property wrapper
class NetworkManager {
    private init() {
        print("NetworkManager initialized")
    }
    
    func fetchData() {
        print("Fetching data...")
    }
}

class AppDependencies {
    @Singleton({ NetworkManager() })
    var networkManager: NetworkManager
}

// Client code
let dependencies = AppDependencies()
dependencies.networkManager.fetchData()
```
## ⚠️ Considerations
#### Pros
- Ensures a class has just a single instance
- Provides a global access point to that instance
- The singleton object is initialized only when it's requested for the first time
- Avoids polluting the global namespace

#### Cons
- Violates the Single Responsibility Principle (manages its own creation and lifecycle)
- Can make unit testing difficult (hard to mock, shared state between tests)
- Can hide dependencies instead of making them explicit
- Requires special treatment in a multithreaded environment

## 🔄 When to Use
- When exactly one instance of a class is needed throughout your application
- When you need stricter control over global variables
- For shared resources like a database connection or a file manager
- For coordination across the system (e.g., a logging service)

## 🛠️ Real-World Examples in iOS
- UIApplication.shared - The singleton instance representing your app
- UserDefaults.standard - The standard user defaults database
- FileManager.default - The default file manager
- URLSession.shared - The shared session for basic URL requests
- NotificationCenter.default - The default notification center

## 📝 Implementation Tips
- Make the constructor private to prevent direct instantiation
- Consider thread safety for lazy initialization
- Be cautious about subclassing singletons
- In Swift, use a static constant for thread-safe initialization
- Consider dependency injection as an alternative

## 🔗 Additional Resources
