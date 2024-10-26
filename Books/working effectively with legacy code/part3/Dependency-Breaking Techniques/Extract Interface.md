### Extract Interface: Detailed Theory and Swift Example

**Extract Interface** is a technique in object-oriented programming used to break dependencies by creating a new interface (or protocol in Swift) that captures a subset of methods from a class. This process decouples a class from its concrete dependencies, making it easier to test and extend.

### Why Extract Interface?

In many systems, classes have dependencies on other concrete classes, which can make testing and extending the system difficult. By introducing an interface:
1. **Decoupling**: Dependencies are decoupled, allowing us to work with any object that adheres to the expected interface.
2. **Testability**: By using a fake or mock implementation of the interface, we can isolate the class under test and avoid complex setup.
3. **Flexibility**: The class can work with different implementations of the same interface, allowing for more flexible and reusable code.

This technique is widely supported in statically typed languages like Java and Swift, where the compiler enforces correct usage of the interface, making it less prone to runtime errors.

### The Extract Interface Technique

1. **Identify Dependencies**: Find the methods in a class that need to be extracted into an interface.
2. **Define the Interface**: Create a new protocol containing these methods.
3. **Conform the Original Class to the Interface**: Update the original class to implement the protocol, which now ensures the class provides the expected behavior.
4. **Use the Interface Instead of the Concrete Class**: Update the dependent classes to use the interface (protocol) instead of the concrete class.

### Example in Swift

Let's look at an example to see how Extract Interface works in Swift. Here, we have a `PaymentProcessor` class that relies on `TransactionLogger` to log transactions. We'll extract an interface (protocol) from `TransactionLogger` to decouple it from `PaymentProcessor` and make testing easier.

#### Initial Setup

Consider this initial setup where `PaymentProcessor` depends on `TransactionLogger`.

```swift
class PaymentProcessor {
    private let logger: TransactionLogger

    init(logger: TransactionLogger) {
        self.logger = logger
    }

    func processPayment(amount: Double) {
        // Payment processing logic
        logger.logTransaction("Processed payment of \(amount)")
    }
}

class TransactionLogger {
    func logTransaction(_ message: String) {
        print("Logging transaction: \(message)")
    }
}
```

In this setup:
- `PaymentProcessor` is tightly coupled to `TransactionLogger`.
- Testing `PaymentProcessor` would require dealing with `TransactionLogger` directly, making it harder to isolate and verify behavior.

#### Step 1: Extract a Protocol (Interface)

We’ll extract a `Logger` protocol from `TransactionLogger`. This protocol will contain the `logTransaction` method that `PaymentProcessor` requires.

```swift
protocol Logger {
    func logTransaction(_ message: String)
}
```

#### Step 2: Make `TransactionLogger` Conform to `Logger`

Now, we modify `TransactionLogger` to conform to `Logger`, enabling it to fulfill the requirements of any instance that requires a `Logger`.

```swift
class TransactionLogger: Logger {
    func logTransaction(_ message: String) {
        print("Logging transaction: \(message)")
    }
}
```

#### Step 3: Update `PaymentProcessor` to Depend on `Logger`

Now, we update `PaymentProcessor` to depend on the `Logger` protocol instead of the concrete `TransactionLogger` class. This change makes `PaymentProcessor` more flexible, allowing it to work with any type that conforms to `Logger`.

```swift
class PaymentProcessor {
    private let logger: Logger

    init(logger: Logger) {
        self.logger = logger
    }

    func processPayment(amount: Double) {
        // Payment processing logic
        logger.logTransaction("Processed payment of \(amount)")
    }
}
```

With this change:
- `PaymentProcessor` no longer depends on a specific logger class but on a `Logger` protocol. This change improves flexibility and makes the class easier to test with various logger implementations.

#### Step 4: Create a Fake Logger for Testing

To test `PaymentProcessor`, we can create a fake logger that conforms to `Logger`. This fake logger will store log messages instead of printing them, allowing us to verify that `PaymentProcessor` logs messages correctly.

```swift
class FakeLogger: Logger {
    var loggedMessages = [String]()

    func logTransaction(_ message: String) {
        loggedMessages.append(message) // Store messages instead of printing
    }
}
```

#### Step 5: Write the Test

We can now test `PaymentProcessor` using `FakeLogger`, which allows us to verify its behavior without relying on actual console output.

```swift
import XCTest

class PaymentProcessorTests: XCTestCase {
    func testProcessPaymentLogsTransaction() {
        // Arrange
        let fakeLogger = FakeLogger()
        let processor = PaymentProcessor(logger: fakeLogger)
        
        // Act
        processor.processPayment(amount: 100.0)
        
        // Assert
        XCTAssertEqual(fakeLogger.loggedMessages, ["Processed payment of 100.0"])
    }
}
```

### Summary of Extract Interface

1. **Define the Protocol**: We created a `Logger` protocol to represent logging functionality.
2. **Update the Dependent Class**: `PaymentProcessor` was updated to depend on `Logger` instead of `TransactionLogger`.
3. **Create a Fake Implementation for Testing**: `FakeLogger` enabled us to test `PaymentProcessor` without using a real logger.
4. **Test the Class**: The test verifies that `PaymentProcessor` correctly logs transactions without requiring the real `TransactionLogger`.

### Benefits of Extract Interface in Swift

- **Improved Testability**: By injecting a `Logger` protocol, we can test `PaymentProcessor` without relying on actual log output.
- **Greater Flexibility**: `PaymentProcessor` can now work with any type that conforms to `Logger`, including new implementations or mock objects.
- **Cleaner Code**: Extracting an interface makes dependencies more explicit, improving code readability and modularity. 

The Extract Interface pattern, adapted as protocol extraction in Swift, simplifies dependency management and promotes clean, testable, and flexible code.
