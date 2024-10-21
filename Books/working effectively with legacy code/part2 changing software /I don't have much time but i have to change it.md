Certainly! Let’s dive deeper into the theories behind these methods for handling changes in legacy code, with detailed explanations and examples in Swift. The goal is to illustrate how to maintain code quality even when time is limited and changes are urgent. Here are the explanations:

---

### **1. The Dilemma: “I Don’t Have Much Time and I Have to Change It”**

#### **Theory**:
- When working with legacy code, it's common to encounter a situation where a feature or bug fix needs to be implemented quickly, but the existing code lacks tests. Making changes without tests can introduce risks because any change might break the existing functionality.
- The challenge is deciding whether to invest time in writing tests and refactoring before making the change or just diving in and making a quick fix.
- It’s a trade-off: **Pay now** (by writing tests and improving code structure) or **Pay more later** (by dealing with potential bugs and more complex maintenance).

#### **Example in Swift**:
Imagine you have an old class, `TransactionService`, that handles bank transactions. It doesn’t have any tests, but you need to add a feature that charges a small service fee for every transaction above a certain amount:

```swift
class TransactionService {
    func processTransaction(amount: Double) -> Double {
        // Original logic to process a transaction
        return amount * 0.98 // Deducts a 2% transaction fee
    }
}
```

Now, you’re asked to add a rule: transactions above $1000 should have an extra fee of $10.

- **Option 1: Quick Change (No Tests)**:
    You could add the logic directly into `processTransaction`, but you’d be making changes without any safety net:

    ```swift
    func processTransaction(amount: Double) -> Double {
        var finalAmount = amount * 0.98
        if amount > 1000 {
            finalAmount -= 10 // Extra fee for large transactions
        }
        return finalAmount
    }
    ```

    This might seem like a quick fix, but if there’s a bug, you won’t have tests to help you identify it, and debugging could become time-consuming.

- **Option 2: Write Tests First**:
    Even if it takes a bit longer, you write a test for the current functionality and then make the change:

    ```swift
    class TransactionServiceTests: XCTestCase {
        func testProcessTransaction_withStandardAmount() {
            let service = TransactionService()
            let result = service.processTransaction(amount: 500)
            XCTAssertEqual(result, 500 * 0.98)
        }

        func testProcessTransaction_withLargeAmount() {
            let service = TransactionService()
            let result = service.processTransaction(amount: 1500)
            XCTAssertEqual(result, 1500 * 0.98 - 10)
        }
    }
    ```

    Now, when you add the new feature, you have tests that ensure your changes don’t break existing functionality.

- **Benefit**: Writing tests might take an extra 30 minutes, but it could save hours of debugging time later if a bug is introduced.

---

Let's explain the **Sprout Method** theory with a detailed example in Swift. This method is particularly useful when you want to add a new feature to a system, but the existing method is complex or lacks tests. Instead of adding new logic directly into the existing method, you create a separate method for the new functionality, which allows you to keep the existing code clean and makes testing easier.

### **Sprout Method: Explanation**

- **Problem**: You need to add new functionality to an existing method that is complex and untested. Adding the new code directly into this method could make it messier and more difficult to maintain.
- **Solution**: Create a new method that handles the new functionality (the "sprouted" method), and call it from the original method. This allows you to write tests for the new functionality separately.

### **Swift Example**

Suppose we have a class `TransactionService` that processes a list of transactions. Currently, the class has a method `postTransactions` that posts each transaction and adds them to a bundle. However, before posting, we need to ensure that no duplicate transactions are added.

#### **Original Code (Before Sprout Method)**

```swift
class TransactionService {
    var transactionBundle = TransactionBundle()

    func postTransactions(transactions: [Transaction]) {
        for transaction in transactions {
            transaction.post()
        }
        transactionBundle.add(transactions)
    }
}

class Transaction {
    func post() {
        // Logic to post a transaction
    }
}

class TransactionBundle {
    private var transactions: [Transaction] = []

    func add(_ transactions: [Transaction]) {
        self.transactions.append(contentsOf: transactions)
    }
    
    func hasTransaction(_ transaction: Transaction) -> Bool {
        return transactions.contains(where: { $0 === transaction })
    }
}
```

In this code, the `postTransactions` method directly posts each transaction and then adds them to the `transactionBundle`. Now, we need to ensure that duplicate transactions are not added before posting them.

#### **Applying the Sprout Method**

Instead of modifying the `postTransactions` method directly, we use the Sprout Method to create a new method called `uniqueTransactions` that filters out duplicates. Then, we call this new method from `postTransactions`.

1. **Create the New Method**:
    - Write a new method `uniqueTransactions` that filters out duplicates.
    - This method takes a list of transactions and returns only the ones that are not already in the bundle.

```swift
class TransactionService {
    var transactionBundle = TransactionBundle()

    func uniqueTransactions(from transactions: [Transaction]) -> [Transaction] {
        return transactions.filter { !transactionBundle.hasTransaction($0) }
    }
}
```

2. **Modify the Original Method**:
    - Now, modify `postTransactions` to use the `uniqueTransactions` method.

```swift
class TransactionService {
    var transactionBundle = TransactionBundle()

    func postTransactions(transactions: [Transaction]) {
        let transactionsToPost = uniqueTransactions(from: transactions)
        for transaction in transactionsToPost {
            transaction.post()
        }
        transactionBundle.add(transactionsToPost)
    }

    func uniqueTransactions(from transactions: [Transaction]) -> [Transaction] {
        return transactions.filter { !transactionBundle.hasTransaction($0) }
    }
}
```

3. **Write Tests for the New Method**:
    - You can now write unit tests for `uniqueTransactions` separately without testing the entire `postTransactions` method.

```swift
import XCTest

class TransactionServiceTests: XCTestCase {
    func testUniqueTransactions() {
        let service = TransactionService()
        let transaction1 = Transaction()
        let transaction2 = Transaction()

        service.transactionBundle.add([transaction1])

        let transactions = [transaction1, transaction2]
        let uniqueTransactions = service.uniqueTransactions(from: transactions)
        
        // transaction1 is already in the bundle, so only transaction2 should be returned.
        XCTAssertEqual(uniqueTransactions.count, 1)
        XCTAssertTrue(uniqueTransactions.contains(transaction2))
        XCTAssertFalse(uniqueTransactions.contains(transaction1))
    }
}
```

#### **Benefits of Using Sprout Method in This Example**:
- **Isolation of New Logic**: The new method `uniqueTransactions` is isolated from the existing method, which makes it easier to test.
- **Easier Testing**: Instead of testing the entire flow of `postTransactions`, you can focus on testing the logic that removes duplicate transactions.
- **Cleaner Code**: `postTransactions` is still focused on posting and adding transactions, while the duplicate-checking logic is separated into `uniqueTransactions`.

#### **What if We Added the Logic Directly?**

If we added the duplicate-checking logic directly into `postTransactions`, it might look like this:

```swift
func postTransactions(transactions: [Transaction]) {
    var transactionsToPost: [Transaction] = []
    for transaction in transactions {
        if !transactionBundle.hasTransaction(transaction) {
            transaction.post()
            transactionsToPost.append(transaction)
        }
    }
    transactionBundle.add(transactionsToPost)
}
```

- **Downside**: Now, `postTransactions` is doing more than one thing—it posts transactions *and* filters out duplicates. This makes the method more complex and harder to test.
- **Potential for Further Complexity**: If more logic needs to be added later (e.g., logging or additional validation), it would further bloat `postTransactions`.

By using the Sprout Method, we kept `postTransactions` simpler and delegated the responsibility of filtering duplicates to a separate, testable method.

#### **Conclusion**:

The Sprout Method is particularly useful when dealing with legacy code where you need to add new functionality but want to keep the original logic intact. It’s a way to break down responsibilities and make the code more maintainable and testable without diving into a complete refactor. This approach is especially beneficial in time-sensitive situations where fully refactoring the original class isn’t feasible.

By using the Sprout Method, you:
- Avoid cluttering the original method.
- Write tests for new functionality in isolation.
- Make the codebase easier to understand and maintain over time.

### **Sprout Class: Explanation with Swift Example**

The **Sprout Class** technique is used when you need to add or change functionality in a class that has so many dependencies or complex setup that it is hard to get it under test. Instead of directly modifying the class, you create a new class to handle the new behavior, making your changes easier to test and keeping your existing class from becoming more tangled.

### **When to Use Sprout Class**

- **High Dependency**: When your existing class has many dependencies that make it difficult to instantiate and test.
- **Complex Setup**: When setting up the existing class in a test environment would be time-consuming.
- **Single Responsibility**: When you want to introduce new behavior that can be considered a separate responsibility.

### **Swift Example**

Let's walk through an example in Swift to illustrate how to use the Sprout Class technique.

Suppose we have an existing `ReportGenerator` class that generates reports in HTML format. It queries data and generates an HTML report from it. The class is large, with lots of dependencies, making it difficult to test directly.

#### **Original Code (Before Sprout Class)**

```swift
class ReportGenerator {
    var database: Database
    var dateRange: DateRange

    init(database: Database, dateRange: DateRange) {
        self.database = database
        self.dateRange = dateRange
    }

    func generate() -> String {
        let results = database.queryResults(from: dateRange.start, to: dateRange.end)
        var pageText = "<html><head><title>Monthly Report</title></head><body><table>"
        
        if !results.isEmpty {
            for result in results {
                pageText += "<tr><td>\(result.department)</td><td>\(result.manager)</td><td>\(result.profit)</td><td>\(result.expense)</td></tr>"
            }
        } else {
            pageText += "No results for this period"
        }
        
        pageText += "</table></body></html>"
        return pageText
    }
}
```

- This `ReportGenerator` class generates an HTML report using data fetched from a `Database` object.
- It’s challenging to test because it involves a `Database` dependency and requires setting up a `DateRange`.

#### **New Requirement**

We need to add a header row to the HTML table, like this:

```html
<tr><td>Department</td><td>Manager</td><td>Profit</td><td>Expenses</td></tr>
```

#### **Applying the Sprout Class**

Instead of modifying the `generate` method directly, we can use a new class called `ReportTableHeaderProducer` to generate the header. This allows us to focus on testing the new functionality without worrying about the complex setup of the `ReportGenerator`.

1. **Create a New Class for the Header**:

```swift
class ReportTableHeaderProducer {
    func makeHeader() -> String {
        return "<tr><td>Department</td><td>Manager</td><td>Profit</td><td>Expenses</td></tr>"
    }
}
```

2. **Modify the Original Class to Use the New Class**:

Now, modify `ReportGenerator` to use the new `ReportTableHeaderProducer` class.

```swift
class ReportGenerator {
    var database: Database
    var dateRange: DateRange
    var headerProducer: ReportTableHeaderProducer

    init(database: Database, dateRange: DateRange, headerProducer: ReportTableHeaderProducer) {
        self.database = database
        self.dateRange = dateRange
        self.headerProducer = headerProducer
    }

    func generate() -> String {
        let results = database.queryResults(from: dateRange.start, to: dateRange.end)
        var pageText = "<html><head><title>Monthly Report</title></head><body><table>"
        
        // Use the new header producer class to add the header row.
        pageText += headerProducer.makeHeader()
        
        if !results.isEmpty {
            for result in results {
                pageText += "<tr><td>\(result.department)</td><td>\(result.manager)</td><td>\(result.profit)</td><td>\(result.expense)</td></tr>"
            }
        } else {
            pageText += "No results for this period"
        }
        
        pageText += "</table></body></html>"
        return pageText
    }
}
```

3. **Write Tests for the New Class**:

Now that we’ve separated the header generation into its own class, we can write unit tests for `ReportTableHeaderProducer`.

```swift
import XCTest

class ReportTableHeaderProducerTests: XCTestCase {
    func testMakeHeader() {
        let producer = ReportTableHeaderProducer()
        let expectedHeader = "<tr><td>Department</td><td>Manager</td><td>Profit</td><td>Expenses</td></tr>"
        
        XCTAssertEqual(producer.makeHeader(), expectedHeader)
    }
}
```

#### **Advantages of Using Sprout Class in This Example**:
- **Testability**: We can test `ReportTableHeaderProducer` independently without needing to set up `ReportGenerator` or `Database`.
- **Separation of Concerns**: The header generation is separated into its own class, making it easier to maintain.
- **Incremental Refactoring**: We can gradually refactor `ReportGenerator` without needing to tackle all of its complexity at once.

#### **What if We Modified the Original Class Directly?**

If we added the header generation directly into `generate`, it might look like this:

```swift
func generate() -> String {
    let results = database.queryResults(from: dateRange.start, to: dateRange.end)
    var pageText = "<html><head><title>Monthly Report</title></head><body><table>"
    
    // Add header directly here
    pageText += "<tr><td>Department</td><td>Manager</td><td>Profit</td><td>Expenses</td></tr>"
    
    if !results.isEmpty {
        for result in results {
            pageText += "<tr><td>\(result.department)</td><td>\(result.manager)</td><td>\(result.profit)</td><td>\(result.expense)</td></tr>"
        }
    } else {
        pageText += "No results for this period"
    }
    
    pageText += "</table></body></html>"
    return pageText
}
```

- **Downside**: Now, the `generate` method is doing even more, making it harder to understand and test.
- **Reduced Flexibility**: If we need to change the header logic later, we must modify `generate` directly, which might affect existing functionality.

By using the **Sprout Class** approach, we keep the logic for generating the header separate, making future changes easier and keeping the `ReportGenerator` class simpler.

The Sprout Class technique allows you to introduce new functionality into a system without making the original class more complex. It’s particularly helpful when dealing with legacy code or classes that have many dependencies, allowing you to write tests for the new functionality independently. Over time, this approach can help improve the overall design of your codebase without requiring a large upfront refactoring effort.

Let's delve deeper into the **Wrap Method** technique, explaining each theoretical point in detail with corresponding Swift examples, ensuring we cover the full range of concepts.

---

### **Theoretical Explanation: Wrap Method**

#### **What is the Wrap Method?**

The **Wrap Method** is a refactoring technique that allows you to add new behavior to an existing method without modifying the original method's core logic. This is useful when you need to add functionality, such as logging, validation, or any pre/post-processing, to an existing method without tangling it with the primary behavior. The idea is to "wrap" the original functionality in a new method or structure that adds the extra logic.

#### **Why Use the Wrap Method?**

1. **Avoiding Temporal Coupling**:
   - Temporal coupling occurs when we add new functionality to a method just because it needs to happen at the same time. For example, if a `processData()` method also logs, sends notifications, and updates some metrics, these responsibilities are temporally coupled.
   - **Wrap Method** helps keep these responsibilities separate, making the code easier to understand and maintain.

2. **Decoupling Behaviors**:
   - If the additional behavior is separate from the core functionality, it’s often better to keep them decoupled. This makes the code more modular and easier to modify in the future.

3. **Enhancing Testability**:
   - By wrapping new behavior around existing methods, you can write tests for the new functionality independently, without affecting the existing logic.

4. **Preserving Method Signatures**:
   - When you use the Wrap Method, you can preserve the original method's signature and behavior. Existing clients that use the method don’t have to change their code when you add new functionality.

#### **How Does Wrap Method Work?**

The Wrap Method can be implemented in two main ways:

1. **Renaming the Original Method** and creating a new method with the same name that calls the renamed method.
2. **Adding a New Method** that combines the new behavior with the existing one.

---

### **Detailed Examples in Swift**

Let's use a Swift example to illustrate each concept and approach.

#### **Scenario: Processing Employee Payroll**

We have an `Employee` class that processes payroll based on timecards. Our new requirement is to log every time payroll is processed.

#### **Original Method (Before Wrap Method)**

```swift
class Employee {
    var timecards: [Timecard]
    var payRate: Double
    var payPeriod: DateRange
    var payDispatcher: PayDispatcher

    init(timecards: [Timecard], payRate: Double, payPeriod: DateRange, payDispatcher: PayDispatcher) {
        self.timecards = timecards
        self.payRate = payRate
        self.payPeriod = payPeriod
        self.payDispatcher = payDispatcher
    }

    // Original method for processing payment
    func processPayroll() {
        var totalAmount = 0.0
        for card in timecards where payPeriod.contains(card.date) {
            totalAmount += card.hours * payRate
        }
        payDispatcher.pay(employee: self, amount: totalAmount)
    }
}
```

In this example, the `processPayroll` method calculates the payroll and uses `PayDispatcher` to make the payment.

#### **Approach 1: Renaming the Original Method**

Here, we rename the original method and create a new method that wraps around it.

```swift
class Employee {
    // Other properties and initializers remain the same...

    // Step 1: Rename the original method and make it private
    private func dispatchPayroll() {
        var totalAmount = 0.0
        for card in timecards where payPeriod.contains(card.date) {
            totalAmount += card.hours * payRate
        }
        payDispatcher.pay(employee: self, amount: totalAmount)
    }

    // Step 2: Create a new method with the same name as the original
    func processPayroll() {
        logPayroll()
        dispatchPayroll()
    }

    // Step 3: Add the new behavior (logging)
    private func logPayroll() {
        print("Logging payroll for employee: \(self)")
    }
}
```

- **What We Did**: 
  - The `processPayroll` method now logs the payroll before calling `dispatchPayroll` (which contains the original logic).
  - This way, the logging behavior is separated from the payroll processing logic.

- **Advantages**:
  - The new logging behavior is separate from the core payroll processing.
  - Tests can be written for `logPayroll` independently.
  - Clients calling `processPayroll` don’t need to change their code.

#### **Approach 2: Adding a New Method**

Another way is to create a new method that combines the existing behavior with the new one, offering an alternative way to call it.

```swift
class Employee {
    // Other properties and initializers remain the same...

    func processPayroll() {
        dispatchPayroll()
    }

    // Step 1: Create a new method that wraps the existing one
    func processLoggedPayroll() {
        logPayroll()
        processPayroll()
    }

    private func dispatchPayroll() {
        var totalAmount = 0.0
        for card in timecards where payPeriod.contains(card.date) {
            totalAmount += card.hours * payRate
        }
        payDispatcher.pay(employee: self, amount: totalAmount)
    }

    private func logPayroll() {
        print("Logging payroll for employee: \(self)")
    }
}
```

- **What We Did**:
  - `processLoggedPayroll` is a new method that logs payroll before calling `processPayroll`.
  - This method can be used when logging is required, while `processPayroll` can still be used without logging.

- **Advantages**:
  - The new functionality is optional, giving more flexibility.
  - `processLoggedPayroll` can be tested separately from `processPayroll`.

#### **Comparison: Wrap Method vs. Sprout Method**

| Feature                     | Wrap Method                                | Sprout Method                           |
|-----------------------------|---------------------------------------------|-----------------------------------------|
| **Focus**                   | Adds new behavior around existing code.    | Adds new functionality as a new method. |
| **Impact on Original Code** | Minimal changes to the original method.    | Requires adding a call to the new method. |
| **Use Case**                | When the behavior is related but separate. | When the behavior can be isolated as a new function. |
| **Example**                 | Logging a method call.                     | Extracting validation logic into a new method. |

---

### **Pros and Cons of Wrap Method**

#### **Advantages**:
1. **Preserves Original Method**: The original logic remains unchanged, making it safer when dealing with complex logic.
2. **Improves Code Modularity**: By separating the added behavior, you make the code easier to understand and maintain.
3. **Enhances Testability**: Allows you to test the new behavior separately.
4. **Maintains Client Code**: Clients using the original method don’t need to be updated when new behavior is added.

#### **Disadvantages**:
1. **Naming Challenges**: Renaming the original method can lead to less intuitive names (e.g., `dispatchPayroll`).
2. **Code Indirection**: It can introduce additional layers of method calls, making the code harder to trace if overused.
3. **Initial Complexity**: It might seem like overengineering for small changes, especially when the new behavior is trivial.

---

### **Conclusion**

The **Wrap Method** is a valuable technique for adding new behavior to existing methods without making intrusive changes to the core logic. It’s especially useful when adding functionality that must run before or after the existing logic, such as logging, metrics tracking, or notification triggers. By keeping responsibilities separate, the Wrap Method makes the codebase more maintainable and testable, ensuring that each part of the code does one thing well. Using Swift, we demonstrated how this technique can be applied to keep the code clean while addressing evolving requirements.
