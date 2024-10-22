### Understanding Hidden Dependencies and Dependency Injection

Hidden dependencies are common in object-oriented design, where a class directly creates instances of other classes inside its constructor or methods. This often makes testing difficult because it tightly couples the class to a specific implementation, making it hard to replace that implementation with a test double (such as a mock or stub) during unit tests. 

To solve this, we use **Dependency Injection (DI)**. Dependency Injection is a technique where the dependencies of a class are provided externally rather than being created within the class itself. This makes the class more flexible and easier to test.

One of the ways to implement Dependency Injection is through the **Parameterize Constructor** pattern. Instead of instantiating a dependency directly within the class, the dependency is passed in as a parameter through the constructor. This approach allows us to provide a real object or a mock when testing.

### Problem Example in Swift

Let's consider a Swift example to understand this better. Suppose we have a class `EmailDispatcher` that manages sending emails using a service called `EmailService`.

```swift
class EmailDispatcher {
    private var service: EmailService
    private var status: Int
    
    init() {
        self.service = EmailService()
        self.status = 0
        
        // Connect to the email service
        let clientType = 12
        service.connect()
        
        if service.getStatus() == .available {
            service.register(clientType: clientType)
            service.setParameter(clientType: clientType, options: [.noBounce, .repeatOff])
        } else {
            status = -1
        }
    }
    
    func sendEmail(_ message: String) {
        service.send(message: message)
    }
}
```

### Issues with This Design

1. **Hidden Dependency**: The `EmailService` instance is created inside the `EmailDispatcher` constructor, making it impossible to replace `EmailService` with a mock object for testing.
2. **Magic Number**: The `clientType` is hard-coded as `12`, which lacks context and can make maintenance harder.
3. **Difficult to Test**: Testing this class requires an actual `EmailService` instance, making unit tests less efficient because they depend on the actual behavior of the `EmailService`.

### Refactoring with Parameterize Constructor

To solve the hidden dependency issue, we can refactor the class using a parameterized constructor. The idea is to pass the `EmailService` instance into the `EmailDispatcher` constructor instead of creating it directly.

1. **Extracting the Initialization Logic**: Create a separate `initialize` function to handle the setup.
2. **Parameterize the Constructor**: Add a new initializer that accepts an `EmailService` object.
3. **Default Constructor**: Provide a default constructor that creates the `EmailService` object for existing clients.

Here's how this would look in Swift:

```swift
// EmailService protocol allows using mocks for testing.
protocol EmailService {
    func connect()
    func getStatus() -> Status
    func register(clientType: Int)
    func setParameter(clientType: Int, options: [EmailOption])
    func send(message: String)
}

class RealEmailService: EmailService {
    // Implementation of the actual email service.
}

// Options for configuring the service.
enum EmailOption {
    case noBounce, repeatOff
}

enum Status {
    case available, offline
}

// Refactored EmailDispatcher using Dependency Injection.
class EmailDispatcher {
    private var service: EmailService
    private var status: Int
    
    init(service: EmailService) {
        self.service = service
        self.status = 0
        initialize()
    }
    
    convenience init() {
        self.init(service: RealEmailService())
    }
    
    private func initialize() {
        let clientType = 12
        service.connect()
        
        if service.getStatus() == .available {
            service.register(clientType: clientType)
            service.setParameter(clientType: clientType, options: [.noBounce, .repeatOff])
        } else {
            status = -1
        }
    }
    
    func sendEmail(_ message: String) {
        service.send(message: message)
    }
}
```

### Explanation of Refactoring

1. **Dependency Injection through Parameterized Constructor**: 
   - The `EmailService` instance is now injected through the constructor, allowing flexibility to use a mock `EmailService` during testing.
   - The `convenience init()` method preserves backward compatibility by providing a default `RealEmailService`.

2. **Mocking for Testing**:
   - We can now create a mock `EmailService` to test `EmailDispatcher` without relying on the actual `EmailService`.

3. **Improved Testing**:
   - Since the `EmailDispatcher` class is no longer dependent on the real `EmailService` instance, we can inject a mock implementation for isolated unit tests.

### Example of Unit Test with Mock Service

Here’s how we can write a test using the refactored `EmailDispatcher`:

```swift
class MockEmailService: EmailService {
    var connectCalled = false
    var registeredClientType: Int?
    var sentMessages: [String] = []
    
    func connect() {
        connectCalled = true
    }
    
    func getStatus() -> Status {
        return .available
    }
    
    func register(clientType: Int) {
        registeredClientType = clientType
    }
    
    func setParameter(clientType: Int, options: [EmailOption]) {
        // Mock implementation.
    }
    
    func send(message: String) {
        sentMessages.append(message)
    }
}

class EmailDispatcherTests: XCTestCase {
    func testEmailDispatcherInitialization() {
        let mockService = MockEmailService()
        let dispatcher = EmailDispatcher(service: mockService)
        
        XCTAssertTrue(mockService.connectCalled)
        XCTAssertEqual(mockService.registeredClientType, 12)
    }
    
    func testSendEmail() {
        let mockService = MockEmailService()
        let dispatcher = EmailDispatcher(service: mockService)
        
        dispatcher.sendEmail("Hello, world!")
        
        XCTAssertEqual(mockService.sentMessages, ["Hello, world!"])
    }
}
```

### Summary

By refactoring to use **Dependency Injection through Parameterize Constructor**, we achieve the following benefits:

- **Testability**: We can use a mock `EmailService` for unit testing, making tests faster and more focused.
- **Flexibility**: Different `EmailService` implementations can be used in production, such as using different email providers or adapting to other protocols.
- **Code Maintenance**: It is easier to update or modify the `EmailDispatcher` class without impacting other parts of the codebase.

This technique improves the design by reducing tight coupling and making the class more adaptable to change, a core principle of clean code.
