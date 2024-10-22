The concepts from the section "**How Do I Add a Feature?**" focus on using a methodical approach to add new features to legacy codebases, particularly using **Test-Driven Development (TDD)**. This approach is valuable in both new and legacy code because it ensures that new functionality is well-tested before integrating it into the system. Below, I’ll break down each part of the TDD process with a corresponding Swift example to make it clearer.

### What is Test-Driven Development (TDD)?

**Test-Driven Development (TDD)** is a software development practice where you write tests before writing the actual code. It follows a simple loop:

1. **Write a failing test case**: Write a test that specifies the behavior you want.
2. **Get it to compile**: Add enough structure to your code to make the test compile.
3. **Make it pass**: Implement the minimal amount of code necessary to pass the test.
4. **Remove duplication**: Refactor the code, improving its structure while ensuring the test still passes.
5. **Repeat**: Continue this cycle for each new piece of functionality.

### Example in Swift

Let’s walk through an example where we need to add a method to calculate the first and second statistical moments (mean and variance) for a data set. Assume we have a class `InstrumentCalculator` that will perform these calculations.

#### Step 0: Setup the Class

We start with an `InstrumentCalculator` class that will hold a list of numbers (elements) and calculate moments:

```swift
import Foundation

class InstrumentCalculator {
    private var elements: [Double] = []

    func addElement(_ element: Double) {
        elements.append(element)
    }
}
```

#### Step 1: Write a Failing Test Case

We want to add a method called `firstMomentAbout` that calculates the first statistical moment about a given point. We start by writing a test case for this method:

```swift
import XCTest

class InstrumentCalculatorTests: XCTestCase {
    func testFirstMomentAbout() {
        let calculator = InstrumentCalculator()
        calculator.addElement(1.0)
        calculator.addElement(2.0)
        let result = calculator.firstMomentAbout(point: 2.0)
        XCTAssertEqual(result, -0.5, accuracy: 0.0001)
    }
}
```

This test checks if the `firstMomentAbout` method calculates the moment correctly. However, it won’t compile yet because the `firstMomentAbout` method does not exist.

#### Step 2: Get It to Compile

Now we add a stub for the `firstMomentAbout` method to make the test compile:

```swift
class InstrumentCalculator {
    private var elements: [Double] = []

    func addElement(_ element: Double) {
        elements.append(element)
    }

    func firstMomentAbout(point: Double) -> Double {
        return Double.nan // Returning NaN as a placeholder
    }
}
```

This allows the code to compile, but the test will fail since it returns `Double.nan`.

#### Step 3: Make It Pass

Next, we implement the method to make the test pass:

```swift
func firstMomentAbout(point: Double) -> Double {
    guard !elements.isEmpty else { return Double.nan }
    let numerator = elements.reduce(0.0) { $0 + ($1 - point) }
    return numerator / Double(elements.count)
}
```

This implementation calculates the first moment by taking the sum of differences between each element and the point, then dividing by the number of elements. Now, the test should pass.

#### Step 4: Remove Duplication

Suppose now we want to add another method, `secondMomentAbout`, which calculates the variance. We follow a similar process:

1. Write a test for `secondMomentAbout`:

```swift
func testSecondMomentAbout() {
    let calculator = InstrumentCalculator()
    calculator.addElement(1.0)
    calculator.addElement(2.0)
    let result = calculator.secondMomentAbout(point: 2.0)
    XCTAssertEqual(result, 0.5, accuracy: 0.0001)
}
```

2. Add a stub for `secondMomentAbout`:

```swift
func secondMomentAbout(point: Double) -> Double {
    return Double.nan
}
```

3. Implement `secondMomentAbout`:

```swift
func secondMomentAbout(point: Double) -> Double {
    guard !elements.isEmpty else { return Double.nan }
    let numerator = elements.reduce(0.0) { $0 + pow($1 - point, 2.0) }
    return numerator / Double(elements.count)
}
```

Now, both tests pass, but we notice some duplication in `firstMomentAbout` and `secondMomentAbout`.

#### Step 5: Remove Duplication

We can extract the shared logic into a private method:

```swift
private func nthMomentAbout(point: Double, n: Double) -> Double {
    guard !elements.isEmpty else { return Double.nan }
    let numerator = elements.reduce(0.0) { $0 + pow($1 - point, n) }
    return numerator / Double(elements.count)
}

func firstMomentAbout(point: Double) -> Double {
    return nthMomentAbout(point: point, n: 1.0)
}

func secondMomentAbout(point: Double) -> Double {
    return nthMomentAbout(point: point, n: 2.0)
}
```

This new method `nthMomentAbout` takes an exponent `n` to calculate any statistical moment. Now, the code is cleaner, and we’ve removed duplication.

### Benefits of TDD in Legacy Code

1. **Safety**: Writing tests before adding new features ensures that you don’t break existing functionality.
2. **Refactoring Confidence**: With tests in place, you can refactor code to improve its structure, knowing that any mistakes will be caught by your tests.
3. **Understanding**: Writing tests helps you understand the behavior of the code, especially when working with legacy systems that lack documentation.
4. **Incremental Improvements**: TDD allows you to improve the code incrementally. You can start with new functionality, then gradually refactor the existing code as you understand it better.

### Extending TDD for Legacy Code

For legacy code that is not well tested, you might need to take additional steps before you can even start using TDD:

0. **Get the class under test**: Use techniques like sprouting, wrapping, or breaking dependencies to make the class testable.
1. **Write a failing test case**: Define the behavior you want to add or change.
2. **Get it to compile**: Add enough structure to the class to get the test to compile.
3. **Make it pass**: Implement the logic to pass the test.
4. **Refactor**: Remove duplication and improve code structure while keeping the test green.
5. **Repeat**: Continue this process for each new feature or change.

### Summary

The process of adding a new feature using TDD involves small, incremental steps: writing tests first, implementing just enough code to make those tests pass, and then refactoring to keep the code clean. This approach is especially powerful in legacy codebases, where tests can serve as a safety net that allows you to refactor and improve the code over time. By taking this approach, you ensure that new features are well-tested, the code becomes easier to understand, and future changes can be made with greater confidence.

The concept of **Programming by Difference** is a technique where, instead of modifying an existing class directly to introduce a new feature, you create a subclass that adds or overrides functionality. This method allows you to add new behavior quickly without changing the existing class. Later, you can refactor the code into a more appropriate structure if needed. This approach is particularly useful in Object-Oriented Programming (OOP) when dealing with legacy code.

Here, I'll break down the theory and illustrate the concepts with **Swift** examples:

### Overview of Programming by Difference

Programming by Difference is a technique where you extend or change the behavior of an existing class by creating a subclass, rather than modifying the class directly. This allows for new functionality without altering the original code, which can be especially useful when working with legacy systems that are difficult to change or test.

The idea behind **Programming by Difference** is to use inheritance to introduce new behavior without directly modifying the existing class. This way, we maintain the original functionality while extending it for new use cases. This technique can be useful when:

- We have tests for the existing class and want to ensure that modifications don't break it.
- We want to isolate new functionality before deciding how to integrate it with the existing design.

 ## When to Use:
When you need to add new behavior but don't want to change the existing class.
When the existing class is difficult to test, and you want to isolate new functionality for easier testing.

### Example Scenario

Let’s say we have a base class `MessageForwarder` in Swift that manages forwarding email messages. The class has a method `getFromAddress()` that retrieves the "from" address from a message:

```swift
import Foundation

class MessageForwarder {
    private let defaultFrom: String = "default@domain.com"

    func getFromAddress(for message: String) -> String {
        // Assume `message` contains the "from" address.
        let fromAddress = extractFromAddress(from: message)
        return fromAddress ?? defaultFrom
    }

    private func extractFromAddress(from message: String) -> String? {
        // Simulate extracting an address (e.g., "user@example.com").
        return message.contains("@") ? message : nil
    }

    func forwardMessage(_ message: String) {
        let fromAddress = getFromAddress(for: message)
        print("Forwarding message from \(fromAddress)")
    }
}
```

Here, the `MessageForwarder` class has a method `getFromAddress()` that extracts the "from" address from a message. If the "from" address is not available, it uses a default address. The `forwardMessage()` method uses `getFromAddress()` to determine which address to use when forwarding messages.

### Step 1: Adding a New Feature Using Inheritance

Now, let’s say we have a new requirement: we need to support "anonymous" mailing lists where the "from" address is always set to a generic address, such as `anon-members@domain.com`. We could create a subclass of `MessageForwarder` called `AnonymousMessageForwarder`:

```swift
class AnonymousMessageForwarder: MessageForwarder {
    private let anonymousFrom: String = "anon-members@domain.com"

    override func getFromAddress(for message: String) -> String {
        return anonymousFrom
    }
}
```

In this subclass, we override the `getFromAddress()` method to always return the anonymous address, regardless of the content of the message. This allows us to quickly support anonymous forwarding without changing the existing `MessageForwarder` class.

### Step 2: Using the New Class in Tests

We can write a test case for `AnonymousMessageForwarder` to verify its behavior:

```swift
func testAnonymousMessageForwarding() {
    let forwarder = AnonymousMessageForwarder()
    forwarder.forwardMessage("user@example.com")
    // Output: "Forwarding message from anon-members@domain.com"
}
```

This test checks that `AnonymousMessageForwarder` uses the anonymous address as expected.

### Step 3: Refactoring to Integrate the Feature

While using a subclass to add the feature is quick, it might not be the best long-term solution. If we find that this behavior should be configurable rather than hardcoded into a subclass, we can refactor the code to use a configuration object:

```swift
class MessageForwarder {
    private let defaultFrom: String = "default@domain.com"
    private var isAnonymous: Bool = false

    init(isAnonymous: Bool = false) {
        self.isAnonymous = isAnonymous
    }

    func getFromAddress(for message: String) -> String {
        if isAnonymous {
            return "anon-members@domain.com"
        } else {
            return extractFromAddress(from: message) ?? defaultFrom
        }
    }

    private func extractFromAddress(from message: String) -> String? {
        return message.contains("@") ? message : nil
    }

    func forwardMessage(_ message: String) {
        let fromAddress = getFromAddress(for: message)
        print("Forwarding message from \(fromAddress)")
    }
}
```

Now, the `MessageForwarder` class can be initialized with a parameter that specifies whether it should behave anonymously. This eliminates the need for a separate subclass and makes the behavior configurable:

```swift
let forwarder = MessageForwarder(isAnonymous: true)
forwarder.forwardMessage("user@example.com")
// Output: "Forwarding message from anon-members@domain.com"
```

### Benefits of Programming by Difference

- **Quickly Add Features**: Using inheritance allows us to introduce new behavior without modifying the existing class.
- **Maintain Existing Functionality**: The original class remains unchanged, preserving its tested functionality.
- **Safe Refactoring**: Tests help ensure that changes don't break existing behavior, allowing us to refactor later if the design needs to be improved.

### Potential Pitfalls: Liskov Substitution Principle (LSP)

One issue that can arise when using inheritance is violating the **Liskov Substitution Principle (LSP)**. LSP states that subclasses should be substitutable for their base classes without altering the correctness of the program. Violations occur when the behavior of a subclass deviates too much from the base class, leading to unexpected results.

For example, if someone uses `AnonymousMessageForwarder` but expects the behavior of `MessageForwarder`, they might be surprised to find that the "from" address is always set to `anon-members@domain.com`.

To avoid LSP violations:

1. **Avoid Overriding Concrete Methods**: Instead, use abstract methods if a subclass should provide different behavior.
2. **Use Composition Instead of Inheritance**: Sometimes, it’s better to use a strategy pattern where the new behavior is passed into the class as a dependency.

### Step 4: Refactoring Towards Composition

We could further improve our design by using a strategy pattern where `getFromAddress` behavior is provided by a separate component, rather than using inheritance:

```swift
protocol FromAddressProvider {
    func getFromAddress(for message: String) -> String
}

class DefaultFromAddressProvider: FromAddressProvider {
    private let defaultFrom: String

    init(defaultFrom: String = "default@domain.com") {
        self.defaultFrom = defaultFrom
    }

    func getFromAddress(for message: String) -> String {
        return message.contains("@") ? message : defaultFrom
    }
}

class AnonymousFromAddressProvider: FromAddressProvider {
    func getFromAddress(for message: String) -> String {
        return "anon-members@domain.com"
    }
}

class MessageForwarder {
    private let fromAddressProvider: FromAddressProvider

    init(fromAddressProvider: FromAddressProvider) {
        self.fromAddressProvider = fromAddressProvider
    }

    func forwardMessage(_ message: String) {
        let fromAddress = fromAddressProvider.getFromAddress(for: message)
        print("Forwarding message from \(fromAddress)")
    }
}
```

Now, `MessageForwarder` relies on a `FromAddressProvider` for determining the "from" address. This allows for more flexibility without using inheritance:

```swift
let anonymousProvider = AnonymousFromAddressProvider()
let forwarder = MessageForwarder(fromAddressProvider: anonymousProvider)
forwarder.forwardMessage("user@example.com")
// Output: "Forwarding message from anon-members@domain.com"
```

### Summary

**Programming by Difference** allows for quick feature addition using inheritance, making it easy to extend behavior without changing existing code. However, it can lead to complex inheritance hierarchies and Liskov Substitution Principle violations if overused. Tests are essential in this process, allowing for safe refactoring toward better designs, such as using composition over inheritance when appropriate. By starting with inheritance and moving toward composition, you can strike a balance between rapid feature development and maintainable code.
