### The Case of the Irritating Global Dependency: Theory with Swift Example

#### Overview
Global dependencies in software design, such as singletons, can make testing difficult because they create tight coupling between classes and shared resources. This coupling makes it challenging to isolate classes for unit testing. Here, we explore how to identify and manage these global dependencies using various strategies, with practical examples in Swift.

### Understanding Global Dependencies and Singletons

#### What is a Global Dependency?
A global dependency occurs when a class depends on a globally accessible object or a singleton. This creates challenges in testing, as these global instances often maintain state across different parts of the application, making it hard to isolate and control their behavior during tests.

#### The Singleton Pattern
The Singleton design pattern ensures that a class has only one instance and provides a global point of access to it. It’s commonly used for managing shared resources like configuration settings, database connections, or service classes.

**Example: Singleton in Swift**

```swift
class PermitRepository {
    static let shared = PermitRepository()
    
    private init() {} // Private constructor prevents other instances
    
    func findAssociatedPermit(notice: PermitNotice) -> Permit? {
        // Simulate fetching a permit from a database
        return Permit(notice: notice)
    }
}
```

In this example:
- `PermitRepository` has a static property `shared` which makes it a singleton.
- The `private init()` ensures no other instance of `PermitRepository` can be created outside this class.
- This Singleton pattern is often used for shared services like a database or network manager.

### The Problem with Testing Singletons
When you have a class that directly uses this singleton, like this:

```swift
class Facility {
    private var basePermit: Permit?
    
    init(facilityCode: Int, owner: String, notice: PermitNotice) throws {
        // Directly accesses the Singleton
        if let associatedPermit = PermitRepository.shared.findAssociatedPermit(notice: notice),
           associatedPermit.isValid && !notice.isValid {
            basePermit = associatedPermit
        } else if !notice.isValid {
            let permit = Permit(notice: notice)
            permit.validate()
            basePermit = permit
        } else {
            throw PermitViolation()
        }
    }
}
```

In the above code:
- `Facility` directly depends on `PermitRepository.shared`, making it hard to replace with a mock or test double.
- This coupling means that tests on `Facility` are indirectly tied to the behavior of `PermitRepository`, leading to unpredictable tests if the state of `PermitRepository` changes.

**Example of a Problematic Test:**

```swift
func testFacilityInitialization() throws {
    let notice = PermitNotice(id: 1, description: "Test Notice")
    let facility = try Facility(facilityCode: 100, owner: "John Doe", notice: notice)
    // Test facility's state or behavior here
}
```

This test might seem straightforward, but it has a hidden dependency on `PermitRepository.shared`. If `findAssociatedPermit` interacts with a database or shared state, the test becomes slow, unreliable, and difficult to configure correctly.

### Strategies to Deal with Global Dependencies in Testing

#### Strategy 1: Dependency Injection
Dependency injection involves passing the dependencies that a class needs directly to it, instead of letting the class create or access them directly.

**Example: Using Dependency Injection in Swift**

1. Define a Protocol for `PermitRepository`:

```swift
protocol PermitRepositoryProtocol {
    func findAssociatedPermit(notice: PermitNotice) -> Permit?
}
```

2. Make `PermitRepository` Conform to the Protocol:

```swift
class PermitRepository: PermitRepositoryProtocol {
    static let shared = PermitRepository()
    
    private init() {}
    
    func findAssociatedPermit(notice: PermitNotice) -> Permit? {
        return Permit(notice: notice)
    }
}
```

3. Update `Facility` to Use the Protocol:

```swift
class Facility {
    private var basePermit: Permit?
    private let repository: PermitRepositoryProtocol
    
    init(facilityCode: Int, owner: String, notice: PermitNotice, repository: PermitRepositoryProtocol) throws {
        self.repository = repository
        
        if let associatedPermit = repository.findAssociatedPermit(notice: notice),
           associatedPermit.isValid && !notice.isValid {
            basePermit = associatedPermit
        } else if !notice.isValid {
            let permit = Permit(notice: notice)
            permit.validate()
            basePermit = permit
        } else {
            throw PermitViolation()
        }
    }
}
```

4. Write a Test with a Mock Repository:

```swift
class MockPermitRepository: PermitRepositoryProtocol {
    func findAssociatedPermit(notice: PermitNotice) -> Permit? {
        // Return a mock permit for testing
        return Permit(notice: notice)
    }
}

func testFacilityInitialization() throws {
    let notice = PermitNotice(id: 1, description: "Test Notice")
    let mockRepository = MockPermitRepository()
    let facility = try Facility(facilityCode: 100, owner: "John Doe", notice: notice, repository: mockRepository)
    
    // Assert or verify behavior here
}
```

With dependency injection:
- `Facility` no longer depends on the global `PermitRepository.shared`.
- The `MockPermitRepository` can simulate different scenarios, making tests faster and more reliable.

#### Strategy 2: Introduce a Static Setter for Testing
Another approach is to modify the singleton to allow replacing its instance, making it easier to use different implementations during tests.

**Example: Static Setter in Swift**

```swift
class PermitRepository: PermitRepositoryProtocol {
    private static var sharedInstance = PermitRepository()
    
    static var shared: PermitRepositoryProtocol {
        get { return sharedInstance }
        set { sharedInstance = newValue as! PermitRepository }
    }
    
    private init() {}
    
    func findAssociatedPermit(notice: PermitNotice) -> Permit? {
        return Permit(notice: notice)
    }
}
```

Now, during testing, you can replace the shared instance:

```swift
func testFacilityInitialization() throws {
    let mockRepository = MockPermitRepository()
    PermitRepository.shared = mockRepository // Replace the singleton with a mock for testing
    
    let notice = PermitNotice(id: 1, description: "Test Notice")
    let facility = try Facility(facilityCode: 100, owner: "John Doe", notice: notice)
    
    // Test behavior here
}
```

**Caution**: This approach requires careful management to ensure that the singleton instance is reset between tests to prevent unintended state sharing.

#### Strategy 3: Subclassing for Testing
You can create a subclass of the singleton specifically for testing, where you override the behavior needed for testing.

**Example: Subclassing for Testing**

```swift
class TestingPermitRepository: PermitRepository {
    override func findAssociatedPermit(notice: PermitNotice) -> Permit? {
        // Return a predictable value for testing
        return Permit(notice: notice)
    }
}
```

In the test, you can use this subclass:

```swift
func testFacilityInitialization() throws {
    let repository = TestingPermitRepository()
    PermitRepository.shared = repository // Use the testing subclass
    
    let notice = PermitNotice(id: 1, description: "Test Notice")
    let facility = try Facility(facilityCode: 100, owner: "John Doe", notice: notice)
    
    // Test behavior here
}
```

This approach allows for specialized testing behavior but still requires managing the global state properly.

### Conclusion
Global dependencies like singletons can create challenges in testing by making classes tightly coupled to specific implementations. By using strategies like dependency injection, static setters, or subclassing, you can reduce this coupling and make your classes more testable.

- **Dependency Injection**: Preferred for flexibility and isolation, allowing you to replace dependencies easily during tests.
- **Static Setter**: Useful when refactoring for dependency injection is not feasible, but it requires careful state management.
- **Subclassing**: Can be helpful when only small changes are needed for testing, but still involves managing global states.

The key to handling these global dependencies is to strike a balance between maintaining the intended design (e.g., a single instance in production) while ensuring that tests can be isolated and controlled effectively. By choosing the right approach, you can make your code more maintainable, modular, and test-friendly.
