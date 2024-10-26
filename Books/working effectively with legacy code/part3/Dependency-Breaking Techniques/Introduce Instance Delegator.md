 The *Instance Delegator* is a refactoring technique used to make code that relies on static methods more testable and flexible. Static methods, while convenient for utility functions or Singleton designs, can be hard to work with in testing scenarios because they do not allow substituting behaviors or injecting dependencies. This can lead to tightly coupled code that is hard to modify or extend. The Instance Delegator introduces an alternative by wrapping static methods in instance methods, allowing flexibility and supporting dependency injection.

### Theory of Instance Delegator

#### Static Methods and Their Limitations

In many programming languages, including Java and Swift, static methods are often used for utility classes or Singleton patterns. A utility class in Java, such as `Math`, provides trigonometric and other mathematical operations as static methods because these functions don’t require any instance data or specific state to perform their operations.

However, static methods are tightly coupled to their class, making them rigid. They can’t be overridden in subclasses, and their behavior cannot be easily changed. This inflexibility becomes problematic in testing because static methods are harder to mock or substitute in tests, making the code challenging to test in isolation.

#### The Purpose of Instance Delegator

Instance Delegator is a technique to introduce a “seam,” or point of flexibility, by creating instance methods that delegate calls to static methods. By wrapping static methods in instance methods, you can inject dependencies, mock behaviors, or otherwise change the behavior without altering the class design itself.

#### Steps to Introduce an Instance Delegator

1. **Identify a Static Method to Delegate:** Find a static method that you wish to use in a more flexible way or that is difficult to test because it’s tightly coupled to its class.
   
2. **Create an Instance Method that Delegates to the Static Method:** Add an instance method to the class that calls the static method internally. This instance method will act as a delegator for the static method.

3. **Replace Static Method Calls with the Delegated Instance Method:** Use dependency injection or another approach to replace direct calls to the static method. Pass an instance of the class to code that previously relied on static calls.

4. **Refactor Over Time:** Gradually refactor until all calls go through the instance method instead of the static method. When complete, consider moving the method body from the static to the instance method and removing the static method.

### Example in Swift

Consider a class in Swift that has static methods, such as a banking service:

```swift
class BankingServices {
    static func updateAccountBalance(userID: Int, amount: Double) {
        // Logic to update account balance
        print("Updated balance for user \(userID) by \(amount)")
    }
}
```

This class uses a static method to update an account balance. To make this easier to test and provide more flexibility, we can create an instance method that delegates to the static method.

1. **Create an Instance Method to Delegate to the Static Method:**

   Add a new instance method, `updateBalance`, that delegates the call to the static method:

   ```swift
   class BankingServices {
       static func updateAccountBalance(userID: Int, amount: Double) {
           // Logic to update account balance
           print("Updated balance for user \(userID) by \(amount)")
       }
       
       func updateBalance(userID: Int, amount: Double) {
           BankingServices.updateAccountBalance(userID: userID, amount: amount)
       }
   }
   ```

2. **Use the Instance Method in Client Code:**

   Let’s assume you have a class that uses the static method directly, which you now want to refactor:

   ```swift
   class AccountManager {
       func processTransaction(userID: Int, amount: Double) {
           // Old usage of static method
           BankingServices.updateAccountBalance(userID: userID, amount: amount)
       }
   }
   ```

   Refactor this to use the new instance method:

   ```swift
   class AccountManager {
       let bankingServices: BankingServices
       
       init(bankingServices: BankingServices) {
           self.bankingServices = bankingServices
       }
       
       func processTransaction(userID: Int, amount: Double) {
           // Use the instance method instead of the static method
           bankingServices.updateBalance(userID: userID, amount: amount)
       }
   }
   ```

3. **Testing with Dependency Injection:**

   Now that we’ve introduced an instance method, it’s easier to substitute `BankingServices` with a mock in tests:

   ```swift
   class MockBankingServices: BankingServices {
       override func updateBalance(userID: Int, amount: Double) {
           // Provide mock behavior for testing
           print("Mocked balance update for user \(userID) by \(amount)")
       }
   }

   // Test Example
   let mockServices = MockBankingServices()
   let accountManager = AccountManager(bankingServices: mockServices)
   accountManager.processTransaction(userID: 123, amount: 50.0)
   ```

   With this setup, you have effectively broken the dependency on the static method, making the `AccountManager` class more testable.

### Benefits of Using Instance Delegator

- **Improved Testability:** Dependency injection allows us to substitute `BankingServices` with a mock, facilitating isolated tests.
- **Increased Flexibility:** The instance method allows modifying or overriding behavior without changing the static implementation.
- **Refactoring Path:** Over time, you may migrate all static calls to instance methods, removing the static method if it’s no longer necessary.

### When to Use and Not Use Instance Delegator

- **Use It When:** You need flexibility for testing, or the static method’s behavior may need to change based on context.
- **Avoid It When:** The static method represents an atomic, unchanging utility (like `Math.sin`) that doesn’t benefit from instance methods or flexible dependency injection.

Instance Delegator is a powerful way to refactor static methods for improved flexibility and testability, especially in complex, dependency-driven projects.
