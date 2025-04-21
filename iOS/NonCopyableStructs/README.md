# Non-Copyable Structs in Swift

## 📱 Overview
Non-copyable structs (also known as "move-only types") were introduced in Swift 5.9 as part of the ownership model improvements. They allow developers to create types that can be moved but not implicitly copied, providing better control over resources and memory management.

## 🔑 Key Concepts

### Basic Structure
```swift
@noncopyable struct ResourceHandle {
    var resource: UnsafeMutableRawPointer
    
    init(acquiring resource: UnsafeMutableRawPointer) {
        self.resource = resource
    }
    
    deinit {
        // Clean up the resource when the handle is destroyed
        freeResource(resource)
    }
}
```
## Core Characteristics
- Marked with the @noncopyable attribute
- Can be moved but not copied
- Ownership is transferred when passed to functions or assigned to variables
- Enables RAII (Resource Acquisition Is Initialization) pattern in Swift
- Compiler enforces single ownership semantics
  
 ## 💡 Use Cases
### Resource Management 
```swift
@noncopyable struct FileHandle {
    private var descriptor: Int32
    
    init?(path: String, flags: Int32) {
        let fd = open(path, flags)
        guard fd >= 0 else { return nil }
        self.descriptor = fd
    }
    
    deinit {
        close(descriptor)
    }
    
    func read(into buffer: UnsafeMutableRawPointer, maxLength: Int) -> Int {
        return Darwin.read(descriptor, buffer, maxLength)
    }
}

// Usage
func processFile() {
    guard var handle = FileHandle(path: "/path/to/file", flags: O_RDONLY) else {
        print("Failed to open file")
        return
    }
    
    // handle is moved (not copied) to the function
    processFileHandle(handle: &handle)
    
    // handle is no longer valid here after being moved
}

func processFileHandle(handle: consuming FileHandle) {
    // Work with the file handle
    // File is automatically closed when handle goes out of scope
}
```


### Unique Ownership of Resources

```swift
@noncopyable struct UniqueBuffer {
    private var pointer: UnsafeMutableBufferPointer<UInt8>
    
    init(capacity: Int) {
        pointer = UnsafeMutableBufferPointer.allocate(capacity: capacity)
    }
    
    deinit {
        pointer.deallocate()
    }
    
    mutating func fill(with value: UInt8) {
        pointer.initialize(repeating: value)
    }
}

// Usage
func processData() {
    var buffer = UniqueBuffer(capacity: 1024)
    buffer.fill(with: 0)
    
    // Transfer ownership
    consumeBuffer(&buffer)
    
    // Cannot use buffer here - it has been moved
}

func consumeBuffer(_ buffer: consuming UniqueBuffer) {
    // Work with buffer
    // Memory is automatically deallocated when function returns
}
```

## ⚠️ Limitations and Considerations
#### Consuming Parameters
- Functions that take ownership of non-copyable types use the consuming parameter modifier
- When passing a non-copyable value to a consuming parameter, use & to indicate transfer of ownership
  
#### Borrowing
- Use borrowing parameter modifier to temporarily access a non-copyable value without taking ownership
- The original value remains valid after the function call
  
```swift
  func readFromBuffer(_ buffer: borrowing UniqueBuffer) -> UInt8 {
    // Read from buffer without taking ownership
    return buffer.pointer[0]
}
```
### Compiler Enforcement
- The compiler tracks ownership of non-copyable values
- Prevents use of moved values
- Ensures resources are properly released

### Compatibility
- Non-copyable types cannot conform to Copyable protocol
- Cannot be used with APIs that require copyable types
- Cannot be stored in arrays or dictionaries (without special handling)
  
## 🔄 Comparison with Other Patterns
#### vs. Reference Types (Classes)
- Both provide unique ownership semantics
- Non-copyable structs avoid reference counting overhead
- Non-copyable structs have deterministic destruction at compile-time known points
  
#### vs. Regular Structs
- Regular structs are implicitly copyable
- Non-copyable structs enforce move semantics
- Non-copyable structs prevent accidental duplication of resources

## 🔗 Additional Resources


