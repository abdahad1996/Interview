### Understanding the Issue: Challenges with Procedural Code

The main challenge discussed in the chapter is how to make safe changes to legacy code that’s written in a procedural style, especially when it lacks the structure provided by object-oriented (OO) programming. Let’s break down the key aspects of the issue, why it’s important, and how we can address it, with a detailed example using Swift.

### What’s Procedural Code?

Procedural code is based on procedures or functions that perform operations directly, without encapsulating related data and behavior into objects. For example, languages like C and older versions of BASIC follow this paradigm. These languages work well for many problems, but they have limitations when it comes to testing and modifying complex systems because:

- **Tight Coupling**: Procedural functions often directly depend on other functions or global variables.
- **Global State**: Functions in procedural programs may rely on or modify global state, making it difficult to isolate behavior for testing.
- **No Built-In Seams**: Seams are places in your code where you can change behavior without altering the code itself, such as through polymorphism or dependency injection. Procedural code often lacks these seams, making testing and refactoring harder.

### The Need for Safe Changes

When you need to modify a procedural codebase—like adding new features or fixing bugs—it’s crucial to ensure that changes don’t break existing functionality. But testing procedural code can be challenging because:

1. **Dependency on External Systems**: For example, a function might directly call a database or external service, making it difficult to test the logic without affecting the external systems.
2. **Lack of Modularity**: Without encapsulation, changes in one function might unintentionally affect another.

Thus, we must find ways to introduce testing capabilities and refactor the code safely before making significant changes.

### Strategies for Refactoring Procedural Code

The chapter discusses several strategies, including **link seams**, **preprocessing seams**, and transitioning to OO design. Let’s explore each of these and see how they can be applied in a Swift context.

#### Strategy 1: Link Seams

A **link seam** is a technique where you substitute different implementations of functions or methods at link time (when the program is being compiled). In languages like C, you can use this to replace a function with a test version during testing.

For example, in C:

```c
// Original function definition
void send_notification(const char* message) {
    // Sends message to a third-party system.
    sendToExternalSystem(message);
}

// Test version
void send_notification(const char* message) {
    // In tests, this does nothing or logs the message locally.
    printf("Mock notification: %s\n", message);
}
```

During testing, you link the program with the test version of `send_notification`, allowing you to test the rest of the code without triggering the real external system.

**In Swift**, we don’t use link-time substitutions, but we achieve similar results with **protocols** and **dependency injection**. This allows us to inject a mock implementation instead of the real one during testing.

#### Strategy 2: Using Protocols and Dependency Injection in Swift

1. **Extract a Protocol**: Define a protocol for the behavior that you want to replace during testing.

    ```swift
    protocol Notifier {
        func send(data: String)
    }
    ```

2. **Create Real and Mock Implementations**:

    ```swift
    class RealNotifier: Notifier {
        func send(data: String) {
            print("Sending real data: \(data)")
            // Actual implementation to send data to a server.
        }
    }

    class MockNotifier: Notifier {
        var receivedData: String?
        
        func send(data: String) {
            receivedData = data
            print("Mock notifier received data: \(data)")
        }
    }
    ```

3. **Inject the Dependency**: Modify the function to accept a `Notifier` instance, making it easier to replace the real notifier with a mock version during testing.

    ```swift
    func processData(input: String, notifier: Notifier) {
        let processedData = "Processed \(input)"
        notifier.send(data: processedData)
    }
    ```

4. **Write Tests Using the Mock**:

    ```swift
    func testProcessData() {
        let mockNotifier = MockNotifier()
        processData(input: "Test Input", notifier: mockNotifier)
        
        assert(mockNotifier.receivedData == "Processed Test Input")
        print("Test passed!")
    }
    ```

By using a protocol and dependency injection, we decouple `processData` from its dependency on a specific `Notifier` implementation, allowing us to test its behavior without interacting with an external system.

### Strategy 3: Preprocessor Seams (C Example)

In C, **preprocessor seams** use the preprocessor to conditionally compile code, allowing you to change behavior based on whether the code is being tested.

```c
#ifdef TESTING
#define send_notification(message) // No-op during testing
#else
#define send_notification(message) real_send_notification(message)
#endif
```

In this example, the `send_notification` function is defined differently when `TESTING` is set, allowing us to bypass the real notification behavior during tests.

**In Swift**, a similar approach would involve using conditional compilation with `#if DEBUG`.

```swift
#if DEBUG
let notifier: Notifier = MockNotifier()
#else
let notifier: Notifier = RealNotifier()
#endif
```

### Strategy 4: Transitioning to Object-Oriented Design

Object-oriented design provides more options for breaking up dependencies and testing. Swift is inherently object-oriented, so applying OO principles can help make legacy procedural code more modular and easier to maintain.

1. **Encapsulate Functions in a Class**: Move related functions into a class, allowing you to manage dependencies through instance variables and methods.

    ```swift
    class DataManager {
        private let notifier: Notifier

        init(notifier: Notifier) {
            self.notifier = notifier
        }

        func processData(input: String) {
            let processedData = "Processed \(input)"
            notifier.send(data: processedData)
        }
    }
    ```

2. **Inject Dependencies**: This makes it easier to use different `Notifier` implementations during testing and production.

    ```swift
    let dataManager = DataManager(notifier: RealNotifier())
    dataManager.processData(input: "Sample Input")
    ```

3. **Write Tests Using a Mock Notifier**:

    ```swift
    func testDataManager() {
        let mockNotifier = MockNotifier()
        let dataManager = DataManager(notifier: mockNotifier)
        dataManager.processData(input: "Test Input")
        
        assert(mockNotifier.receivedData == "Processed Test Input")
        print("Test passed!")
    }
    ```

### Summary of Strategies

- **Link Seams**: Replace functions with test versions at compile time. In Swift, use protocols for similar flexibility.
- **Preprocessor Seams**: Use conditional compilation to include or exclude test-specific code. In Swift, use `#if DEBUG`.
- **Object-Oriented Refactoring**: Encapsulate functions into classes, introduce protocols, and use dependency injection to create testable units.

### Benefits of These Techniques

- **Decoupling**: Makes your functions less dependent on specific implementations, reducing the risk of changes breaking other parts of the system.
- **Testability**: Allows you to test individual functions or classes without relying on the entire environment being set up (e.g., external systems or databases).
- **Modularity**: Moving toward an object-oriented design breaks up large, monolithic functions into smaller, more manageable pieces.

### Conclusion

Refactoring a procedural codebase can be challenging, especially when trying to make safe changes without introducing bugs. Techniques like using link seams, preprocessors, or transitioning to object-oriented design can help isolate changes and make code more testable. In Swift, the use of protocols, dependency injection, and object-oriented principles allows you to safely modify and extend your code, ensuring that changes remain isolated and verifiable through testing. This gradual shift can lead to a more maintainable and flexible codebase over time.
