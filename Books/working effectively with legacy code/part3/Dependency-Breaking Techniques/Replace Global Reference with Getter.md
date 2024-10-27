In Swift, global references—such as static properties, singleton instances, or other global variables—can create tightly coupled code, making it hard to isolate dependencies for testing. The "Replace Global Reference with Getter" technique helps address this by replacing global references with a getter method, which can be overridden in a subclass or test setup to provide test-specific behavior.

### Steps to Apply "Replace Global Reference with Getter" in Swift

1. **Identify the Global Reference**: Locate the global reference (such as a singleton or static instance) that you want to replace.
2. **Create a Getter**: Write a getter for the global reference. Make this getter `open` or `internal` to allow it to be overridden by subclasses.
3. **Replace Direct References with the Getter**: Replace all instances of the global reference with calls to the getter.
4. **Override the Getter in a Testing Subclass**: Create a test subclass that overrides the getter to return a mock or test-specific version of the dependency.

### Example: Inventory System with a Singleton

Suppose we have an `Inventory` singleton class that provides access to items by their barcode. We’ll create a `RegisterSale` class that uses this global reference but needs to be decoupled for testing.

#### Step 1: Define the Global Singleton

Here’s the singleton `Inventory` class, which provides access to items based on barcodes. For simplicity, let’s assume `Inventory` has a shared instance and a method to get an item by barcode.

```swift
// Global Singleton Inventory
class Inventory {
    static let shared = Inventory()

    private init() {}  // Private initializer for singleton pattern

    func itemForBarcode(_ code: String) -> Item? {
        // Production code to get an item by barcode
        return Item(barcode: code, name: "Sample Item")
    }
}

// Item model
struct Item {
    let barcode: String
    let name: String
}
```

Here:
- `Inventory.shared` is a global singleton instance.
- `itemForBarcode` provides access to items based on barcodes.

#### Step 2: Create a `RegisterSale` Class That Uses the Singleton

In `RegisterSale`, we create a method to add an item by barcode. Normally, we’d directly use `Inventory.shared` here, but instead, we’ll use a getter to make it more testable.

```swift
class RegisterSale {
    private var items: [Item] = []

    func addItem(byBarcode code: String) {
        // Access the global Inventory instance via the getter
        if let item = getInventory().itemForBarcode(code) {
            items.append(item)
        }
    }

    // Protected getter for Inventory singleton
    func getInventory() -> Inventory {
        return Inventory.shared
    }
}
```

In this setup:
- `getInventory()` returns `Inventory.shared`.
- By using `getInventory()`, we can now override this method in a subclass to replace the global dependency with a test-specific instance.

#### Step 3: Replace Direct References with the Getter

We’ve already replaced the direct use of `Inventory.shared` in `addItem(byBarcode:)` with `getInventory()`. Now, every call to `Inventory` in this class uses `getInventory()` instead, allowing for greater flexibility in testing.

#### Step 4: Create a Test Subclass and Override the Getter

To test `RegisterSale` independently of the real `Inventory` singleton, we create a mock subclass of `Inventory`, called `MockInventory`, that returns a mock item. Then, we create a test subclass of `RegisterSale` that overrides `getInventory()` to return this mock inventory.

```swift
// Mock Inventory class for testing
class MockInventory: Inventory {
    override func itemForBarcode(_ code: String) -> Item? {
        // Return a mock item for testing purposes
        return Item(barcode: code, name: "Mock Item")
    }
}

// Testing subclass for RegisterSale
class TestingRegisterSale: RegisterSale {
    private let mockInventory = MockInventory()

    // Override getter to return mock inventory
    override func getInventory() -> Inventory {
        return mockInventory
    }
}
```

Here:
- `MockInventory` overrides `itemForBarcode` to provide mock data instead of accessing a real database or inventory system.
- `TestingRegisterSale` overrides `getInventory()` to return `MockInventory` instead of the singleton instance, making it easy to test `RegisterSale` without relying on the real `Inventory` class.

#### Step 5: Write Tests Using the Testing Subclass

Now, we can use `TestingRegisterSale` in tests to ensure the `RegisterSale` functionality works as expected without depending on the real `Inventory` singleton.

```swift
import XCTest

class RegisterSaleTests: XCTestCase {
    func testAddItemUsesMockInventory() {
        let registerSale = TestingRegisterSale()
        
        // Add an item using a barcode
        registerSale.addItem(byBarcode: "12345")
        
        // Verify that the item was added using the mock data
        // (In a real test, you'd add assertions to check the item name, count, etc.)
        XCTAssertNotNil(registerSale.addItem(byBarcode: "12345"))
    }
}
```

### Explanation of the Swift Example

1. **Dependency Isolation with Getter**: By using `getInventory()` instead of `Inventory.shared` directly, we isolated the dependency on the singleton and made it easier to substitute.
2. **Override in Testing Subclass**: We created `TestingRegisterSale`, a subclass of `RegisterSale`, that uses `MockInventory` instead of the real `Inventory`, allowing tests to be independent of the singleton.
3. **Flexibility for Testing**: By replacing the global reference with a getter, we achieved flexibility, making it easy to swap out dependencies for testing without changing the original class.

### Summary of Steps to Replace Global Reference with Getter in Swift

1. **Identify the Global Reference**: Find instances where a class directly uses a singleton or static property.
2. **Create a Getter**: Add a method to return the singleton or global instance instead of referencing it directly. Mark it `open` or `internal` if needed.
3. **Replace Direct References with Getter**: Use the getter method in all places where the global reference was used.
4. **Override Getter in Test Subclass**: In a test-specific subclass, override the getter to return a mock or alternative instance.

Using this pattern, you can make global dependencies testable, allowing you to substitute them as needed and improving the modularity and testability of your Swift code.
