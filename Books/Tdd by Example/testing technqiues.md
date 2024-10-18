Let's delve deeper into the testing concepts, providing more detailed explanations and examples in Swift, aligned closely with the original text.

### 1. **Child Test**

**Concept:**  
When a test case becomes too large or complex and fails, the approach is to write a smaller test that focuses on the problematic part of the larger test. This allows you to identify and fix issues in a more controlled manner. Once the smaller test passes, you can reintroduce the larger test.

The red/green/refactor cycle is critical in TDD. If a test stays red (failing) for too long, it can be demotivating. By breaking down tests, you maintain this rhythm, ensuring progress and reducing frustration.

**Example in Swift:**

Suppose you have a function that calculates a total price including taxes and discounts:

```swift
func calculateTotalPrice(items: [Item], tax: Double, discount: Double) -> Double {
    let subtotal = items.reduce(0) { $0 + $1.price }
    let discounted = subtotal * (1 - discount)
    return discounted * (1 + tax)
}
```

You write a test for the entire function, but it fails:

```swift
func testCalculateTotalPrice() {
    let items = [Item(price: 100), Item(price: 200)]
    let result = calculateTotalPrice(items: items, tax: 0.1, discount: 0.2)
    XCTAssertEqual(result, 264) // Fails!
}
```

Instead of debugging the whole calculation, you can break down the test:

1. Write a smaller test to verify the subtotal:

```swift
func testCalculateSubtotal() {
    let items = [Item(price: 100), Item(price: 200)]
    let subtotal = items.reduce(0) { $0 + $1.price }
    XCTAssertEqual(subtotal, 300) // Passes
}
```

2. Write a smaller test for applying the discount:

```swift
func testApplyDiscount() {
    let subtotal = 300.0
    let discount = 0.2
    let discounted = subtotal * (1 - discount)
    XCTAssertEqual(discounted, 240) // Passes
}
```

3. Reintroduce the larger test:

```swift
func testCalculateTotalPrice() {
    let items = [Item(price: 100), Item(price: 200)]
    let result = calculateTotalPrice(items: items, tax: 0.1, discount: 0.2)
    XCTAssertEqual(result, 264) // Now it passes after fixing logic!
}
```

By breaking down the complex test into smaller pieces, you isolate problems and fix them without losing sight of the overall functionality.

---

### 2. **Mock Object**

**Concept:**  
Mock objects simulate complex dependencies like databases or APIs, allowing tests to run independently of these external systems. This makes tests faster, more predictable, and less dependent on the state of external resources.

The classic example involves testing an object that uses a database. Instead of using a real database, a mock is created that returns constant data.

**Example in Swift:**

Imagine you have a `UserService` that fetches user details from an API:

```swift
protocol NetworkServiceProtocol {
    func fetchData(for userId: String) -> Data
}

class MockNetworkService: NetworkServiceProtocol {
    func fetchData(for userId: String) -> Data {
        // Simulate a JSON response
        return Data("{\"name\": \"John Doe\", \"age\": 30}".utf8)
    }
}

func testFetchUser() {
    let mockService = MockNetworkService()
    let userService = UserService(networkService: mockService)
    let user = userService.fetchUser(userId: "123")
    
    XCTAssertEqual(user.name, "John Doe")
    XCTAssertEqual(user.age, 30)
}
```

In this example, `MockNetworkService` provides a fake response instead of making a real network call. This allows you to test the `UserService` logic without relying on an actual API.

---

### 3. **Self Shunt**

**Concept:**  
Self shunt is a testing technique where the test case itself acts as the collaborator for the object under test. This allows you to verify the communication between objects directly within the test, making the tests easier to read and understand.

**Example in Swift:**

Suppose you have a `DataLoader` class that sends updates to a delegate:

```swift
protocol DataLoaderDelegate: AnyObject {
    func dataLoader(_ loader: DataLoader, didUpdateProgress progress: Float)
}

class DataLoader {
    weak var delegate: DataLoaderDelegate?
    
    func loadData() {
        // Simulate data loading progress
        delegate?.dataLoader(self, didUpdateProgress: 0.5)
        delegate?.dataLoader(self, didUpdateProgress: 1.0)
    }
}
```

To test this, the test class can act as the delegate:

```swift
class DataLoaderTests: XCTestCase, DataLoaderDelegate {
    var progressUpdates: [Float] = []
    
    func dataLoader(_ loader: DataLoader, didUpdateProgress progress: Float) {
        progressUpdates.append(progress)
    }
    
    func testProgressUpdates() {
        let loader = DataLoader()
        loader.delegate = self
        
        loader.loadData()
        
        XCTAssertEqual(progressUpdates, [0.5, 1.0])
    }
}
```

By using `DataLoaderTests` as the delegate, you can verify the interaction between `DataLoader` and its delegate without needing a separate mock object.

---

### 4. **Log String**

**Concept:**  
Log strings are useful when you need to verify the sequence of operations. Instead of setting multiple flags or counters, you can append messages to a string to track the order of method calls.

**Example in Swift:**

Suppose you have a `Task` class that has `prepare`, `execute`, and `cleanup` methods:

```swift
class Task {
    var log: String = ""
    
    func prepare() {
        log += "prepare "
    }
    
    func execute() {
        log += "execute "
    }
    
    func cleanup() {
        log += "cleanup "
    }
}
```

You want to ensure these methods are called in the correct order:

```swift
func testTaskExecutionOrder() {
    let task = Task()
    task.prepare()
    task.execute()
    task.cleanup()
    
    XCTAssertEqual(task.log, "prepare execute cleanup ")
}
```

Here, the `log` string helps confirm that `prepare`, `execute`, and `cleanup` are called in the right sequence.

---

### 5. **Crash Test Dummy**

**Concept:**  
A crash test dummy simulates error conditions that are difficult to reproduce naturally. For example, you might need to simulate a file system error to test how your code handles it.

**Example in Swift:**

Imagine you need to test how your app handles a situation where the file system is full:

```swift
class FullFileWriter: FileWriter {
    override func write(_ data: Data, to file: String) throws {
        throw FileError.diskFull
    }
}

func testFileWriteFailure() {
    let fileWriter = FullFileWriter()
    
    XCTAssertThrowsError(try fileWriter.write(Data(), to: "test.txt")) { error in
        XCTAssertEqual(error as? FileError, FileError.diskFull)
    }
}
```

`FullFileWriter` simulates a disk full error by throwing `FileError.diskFull`, allowing you to verify that your error-handling logic works correctly without actually filling up the disk.

---

### 6. **Broken Test**

**Concept:**  
Leaving a test broken at the end of a coding session helps maintain focus when you resume work. It provides a clear direction for where to start and quickly gets you back into the rhythm of development.

**Example in Swift:**

Before ending your work session, you might write a test that doesn't pass:

```swift
func testNewFeature() {
    let result = featureX()
    XCTAssertEqual(result, "Expected Result") // This test is intentionally failing.
}
```

When you return, this failing test will remind you what you were working on, making it easier to get back into the flow.

---

### 7. **Clean Check-in**

**Concept:**  
When working in a team, always ensure that all tests pass before checking in your code. This prevents introducing errors into the shared codebase and ensures that everyone starts from a stable point.

**Example in Swift:**

Use Xcode or the command line to run all tests before committing:

```bash
xcodebuild test -scheme MyApp
```

Make sure all tests pass before pushing the changes to the shared repository, ensuring that your team members won't encounter issues because of your code.

---

These detailed explanations and examples in Swift align closely with the original text, emphasizing the testing strategies and their practical application. Each example illustrates how to maintain the principles of TDD and testing best practices.
