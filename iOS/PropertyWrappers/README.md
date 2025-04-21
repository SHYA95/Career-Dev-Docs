# Property Wrappers in iOS

## 📱 Overview
Property wrappers, introduced in Swift 5.1, provide a layer of separation between code that manages how a property is stored and the code that defines the property.
They help reduce boilerplate code and create reusable property behaviors.

## 🔑 Key Concepts

### Basic Structure
```swift
@propertyWrapper
struct Wrapper<T> {
    private var value: T
    
    var wrappedValue: T {
        get { return value }
        set { value = newValue }
    }
    
    init(wrappedValue initialValue: T) {
        value = initialValue
    }
}
```
## Common Use Cases
. Data Validation: Ensuring values meet specific criteria
. Type Conversion: Automatically converting between types
. Thread Safety: Managing access across different threads
. Persistence: Automatically saving to UserDefaults or other storage
. Observation: Tracking changes to properties

## ⚠️ Limitations and Considerations
. Property wrappers can't be used with let properties
. They can't directly access the enclosing instance (without using @dynamicMemberLookup)
. Composition of multiple property wrappers applies them from the inside out
. Performance overhead for complex property wrappers

## 🔗 Additional Resources
. https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Property-Wrappers

. https://github.com/swiftlang/swift-evolution/blob/main/proposals/0258-property-wrappers.md
