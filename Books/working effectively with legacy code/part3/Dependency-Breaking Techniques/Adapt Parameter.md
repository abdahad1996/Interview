The *Adapt Parameter* technique helps decouple code from complex or tightly-coupled parameter types by introducing an abstraction, making testing easier and allowing for more flexible code design. This is particularly helpful when dealing with parameters tied to specific frameworks or libraries. Here, I'll illustrate this with a Swift example, similar to the Java one provided, to show how you can use this concept effectively in Swift.

### Problem Scenario
Suppose we have a method that interacts with a specific API parameter type. For instance, consider a class that processes HTTP requests using a parameter that closely resembles `URLRequest`:

```swift
import Foundation

class RequestProcessor {
    func process(request: URLRequest) {
        if let token = request.value(forHTTPHeaderField: "Authorization") {
            // Do some processing with the token
            print("Token: \(token)")
        }
        // Other logic that depends on URLRequest...
    }
}
```

In this example:
- The `process` method depends directly on `URLRequest`, which ties our method to a specific type.
- Testing this might require setting up a full `URLRequest`, even if we only need a small part of its functionality, like an HTTP header.

### Solution: Adapt Parameter
To make this method easier to test and less dependent on `URLRequest`, we can use the *Adapt Parameter* technique by creating a new protocol that abstracts the behavior we need.

### Step-by-Step Implementation

1. **Create a new protocol that represents the behavior we need**:

    ```swift
    protocol ParameterSource {
        func value(forKey key: String) -> String?
    }
    ```

    - This protocol only has a single method, `value(forKey:)`, which is all we need for our processing logic.

2. **Create a production implementer of the protocol**:

    ```swift
    class URLRequestParameterSource: ParameterSource {
        private let request: URLRequest
        
        init(request: URLRequest) {
            self.request = request
        }
        
        func value(forKey key: String) -> String? {
            return request.value(forHTTPHeaderField: key)
        }
    }
    ```

    - This class wraps `URLRequest` and conforms to the `ParameterSource` protocol, allowing us to extract the required header value.

3. **Create a fake implementer for testing**:

    ```swift
    class FakeParameterSource: ParameterSource {
        private var values: [String: String]
        
        init(values: [String: String] = [:]) {
            self.values = values
        }
        
        func value(forKey key: String) -> String? {
            return values[key]
        }
        
        func setValue(_ value: String, forKey key: String) {
            values[key] = value
        }
    }
    ```

    - This fake class allows us to simulate various scenarios by providing controlled values for our tests.

4. **Modify the method to use the protocol instead of `URLRequest`**:

    ```swift
    class RequestProcessor {
        func process(parameterSource: ParameterSource) {
            if let token = parameterSource.value(forKey: "Authorization") {
                // Do some processing with the token
                print("Token: \(token)")
            }
            // Other logic that now depends on the abstracted ParameterSource...
        }
    }
    ```

5. **Write a test using the fake implementer**:

    ```swift
    import XCTest

    class RequestProcessorTests: XCTestCase {
        func testProcessWithValidToken() {
            // Arrange
            let fakeSource = FakeParameterSource()
            fakeSource.setValue("Bearer test_token", forKey: "Authorization")
            let processor = RequestProcessor()
            
            // Act
            processor.process(parameterSource: fakeSource)
            
            // Assert
            // Use assertions to verify behavior if needed.
            // In this example, we could verify printed output or use a spy/mock pattern.
        }
    }
    ```

    - Using the `FakeParameterSource`, we can now test the `process` method without needing a real `URLRequest`.

6. **Use the production implementer in actual code**:

    ```swift
    let request = URLRequest(url: URL(string: "https://example.com")!)
    let source = URLRequestParameterSource(request: request)
    let processor = RequestProcessor()
    processor.process(parameterSource: source)
    ```

    - In production, we use `URLRequestParameterSource` to wrap `URLRequest` and pass it to the `process` method.

### Explanation of Benefits

- **Decoupling**: `RequestProcessor` no longer depends directly on `URLRequest`, making it easier to test and modify.
- **Testability**: We can use `FakeParameterSource` to simulate different parameter values, making tests simpler and more focused on the behavior of the `process` method.
- **Flexibility**: If we later need to support a different request mechanism or data source, we only need to implement the `ParameterSource` protocol, without modifying `RequestProcessor`.

### Summary
The *Adapt Parameter* technique allows us to introduce a simpler, custom interface (`ParameterSource`) in place of a complex or tightly coupled parameter type (`URLRequest`). By doing this, we gain the flexibility to create fake or mock versions for testing, leading to cleaner, more modular code. It’s especially useful in scenarios where using or testing the original parameter type would be cumbersome or overly dependent on specific implementation details.
