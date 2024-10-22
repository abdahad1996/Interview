### Problem Overview: The Case of the Undetectable Side Effect

Testing functionality in software development is often straightforward when methods have clear inputs and outputs. However, complications arise when classes interact heavily with other classes and methods that don't directly return values or communicate their results. These situations can make it challenging to understand and test what is happening internally in the class.

Such issues are common in applications that interact with GUIs or other frameworks where the flow of information involves actions and callbacks rather than direct method results. This is known as **side effects**, where a method's behavior affects the state of the system in ways not visible through the method's return values.

For example, imagine a class handling user interactions in a UI, such as responding to button clicks or displaying additional information. The methods in these classes might trigger visual changes or updates in other UI components without returning any direct results, making it difficult to verify their behavior in a test environment.

### Theory: Command/Query Separation and Refactoring for Testability

A key design principle to address this problem is **Command/Query Separation (CQS)**. This principle, introduced by Bertrand Meyer, suggests that methods should either:

- Be **commands** that perform an action or modify the state but do not return a value.
- Be **queries** that return information but do not modify the state.

This separation helps to clarify the role of methods, making it easier to reason about their behavior. For testing, CQS helps because queries can be verified directly through their return values, while commands require verifying their effects on the system state or environment.

To test side-effect-heavy classes, we often need to **refactor** them to separate their responsibilities. This process may involve:

1. **Extracting Methods**: Breaking down complex methods into smaller, more manageable pieces.
2. **Introducing Protocols and Mocking**: Using protocols to abstract dependencies, allowing us to create mock objects for testing.
3. **Isolating Side Effects**: Moving UI-related or framework-dependent code into specific methods, so other logic can be tested independently.

### Example in Swift: Refactoring for Testability

Let's walk through an example similar to the Java scenario but in Swift. We'll refactor a class that handles user interactions in a GUI to make it testable.

#### Step 1: Original Problematic Class

Consider a Swift class that displays account details and handles user interactions, such as opening a detail view:

```swift
import UIKit

class AccountDetailViewController: UIViewController {
    private let displayLabel = UILabel()
    private var detailView: DetailViewController?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Setup displayLabel and other UI components
    }
    
    @objc func buttonTapped() {
        detailView = DetailViewController()
        detailView?.descriptionText = getDetailText() + " " + getProjectionText()
        detailView?.presentDetailView()
        
        if let symbol = detailView?.getAccountSymbol() {
            displayLabel.text = "\(symbol):"
        }
    }
    
    private func getDetailText() -> String {
        return "Detail"
    }
    
    private func getProjectionText() -> String {
        return "Projection"
    }
}
```

In this example, `buttonTapped` triggers actions like creating a new view, displaying it, and updating a label with the result from the detail view. This method has multiple responsibilities, mixing UI logic with business logic, making it difficult to test:

- It directly creates a `DetailViewController` instance.
- It updates the UI (`displayLabel`) based on results from `detailView`.

Testing this class would require simulating button taps and verifying the UI state, which can be cumbersome and unreliable.

#### Step 2: Refactoring for Testability

To improve testability, we can extract the business logic and separate it from the UI-related tasks. Let's break down the responsibilities and introduce protocols where needed.

1. **Extract the Detail Retrieval Logic**: Create a protocol and a new class for managing the detail data.

```swift
protocol DetailProvider {
    func getAccountSymbol() -> String
    func getDescription() -> String
}

class DetailDataProvider: DetailProvider {
    func getAccountSymbol() -> String {
        return "SYM"
    }
    
    func getDescription() -> String {
        return "Detail Projection"
    }
}
```

2. **Refactor the View Controller**: Use `DetailProvider` instead of directly interacting with `DetailViewController`.

```swift
class AccountDetailViewController: UIViewController {
    private let displayLabel = UILabel()
    private var detailProvider: DetailProvider
    
    init(detailProvider: DetailProvider) {
        self.detailProvider = detailProvider
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    @objc func buttonTapped() {
        let description = detailProvider.getDescription()
        setDetailDescription(description)
        
        let symbol = detailProvider.getAccountSymbol()
        setDisplayText("\(symbol):")
    }
    
    private func setDetailDescription(_ description: String) {
        // Logic to display the description, potentially in another view.
    }
    
    private func setDisplayText(_ text: String) {
        displayLabel.text = text
    }
}
```

3. **Create a Mock for Testing**: Use a mock implementation of `DetailProvider` to test `AccountDetailViewController`.

```swift
class MockDetailProvider: DetailProvider {
    var mockSymbol = "MOCK_SYM"
    var mockDescription = "Mock Description"
    
    func getAccountSymbol() -> String {
        return mockSymbol
    }
    
    func getDescription() -> String {
        return mockDescription
    }
}
```

4. **Write Tests for `AccountDetailViewController`**:

```swift
import XCTest

class AccountDetailViewControllerTests: XCTestCase {
    func testButtonTappedUpdatesLabel() {
        let mockProvider = MockDetailProvider()
        let controller = AccountDetailViewController(detailProvider: mockProvider)
        
        controller.buttonTapped()
        
        XCTAssertEqual(controller.displayLabel.text, "MOCK_SYM:")
    }
}
```

#### Step 3: Benefits of the Refactoring

- **Isolated Business Logic**: By separating out the `DetailProvider` logic, the `AccountDetailViewController` no longer directly handles the data fetching, making it easier to substitute the real logic with a mock for testing.
- **Decoupled UI Updates**: Methods like `setDisplayText` handle the UI updates, keeping the actual data-fetching logic independent.
- **Improved Testability**: The `buttonTapped` method can now be tested by injecting different `DetailProvider` implementations, making tests easier and more reliable.

### Conclusion

Refactoring code to make it more testable involves identifying and isolating responsibilities, particularly when dealing with side effects and UI interactions. In the example, we applied the **Command/Query Separation** principle and used **Extract Method** refactoring to separate concerns between UI logic and business logic. By doing so, we created a more testable design without sacrificing the original functionality of the class.

This approach is useful not only for making tests easier to write but also for creating a cleaner and more maintainable codebase. It allows for flexibility in testing and makes future modifications to the logic easier, as changes in the business logic do not directly impact the UI code and vice versa.
