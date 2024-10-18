The concept of **seams** is about finding points in a program where you can change behavior without directly editing the code at that location. Seams are particularly useful when working with legacy code or systems with tight dependencies, as they provide ways to make code testable. Let’s break down the theory of seams and provide examples in Swift.

### Understanding Seams

#### 1. **What is a Seam?**
   - A **seam** is a place in the code where you can alter the behavior of a program without directly editing the code at that point.
   - It allows you to inject test-specific behavior, making it possible to test parts of the code that would otherwise be difficult or impossible to test due to dependencies on external systems or complex logic.

#### 2. **Types of Seams**:
   - **Object Seams**: Common in object-oriented languages, this type of seam allows you to change behavior by subclassing or using protocols. It is the most natural type of seam in Swift.
   - **Preprocessing Seams**: These exist in languages like C/C++ with preprocessors that can include or exclude code before compilation (e.g., using `#ifdef`). Swift doesn't have preprocessing in the same way, but it has conditional compilation using `#if` for different environments.
   - **Link Seams**: Available in languages where you can change linked libraries or modules. Swift’s modular system can achieve similar results using dependency injection or swapping modules at runtime.

### Example of Seams in Swift

Let’s explore the concept of seams through a concrete example in Swift, focusing on **object seams**.

### Example Scenario: NetworkManager with Dependencies

Imagine we have a class `NetworkManager` that interacts with an external API to fetch data:

```swift
class NetworkManager {
    func fetchData(completion: @escaping (Result<String, Error>) -> Void) {
        // Simulating a network call
        let url = URL(string: "https://example.com/api")!
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
            } else if let data = data, let result = String(data: data, encoding: .utf8) {
                completion(.success(result))
            } else {
                completion(.failure(NSError(domain: "Unknown Error", code: -1, userInfo: nil)))
            }
        }
        task.resume()
    }
}
```

### 1. **Object Seams using Protocols**:

To make `NetworkManager` testable, we can use a **protocol** as a seam. By doing this, we can swap out the real `NetworkManager` with a mock implementation during testing.

```swift
protocol NetworkService {
    func fetchData(completion: @escaping (Result<String, Error>) -> Void)
}

class NetworkManager: NetworkService {
    func fetchData(completion: @escaping (Result<String, Error>) -> Void) {
        // Real network request implementation
    }
}

class MockNetworkManager: NetworkService {
    var result: Result<String, Error>?
    
    func fetchData(completion: @escaping (Result<String, Error>) -> Void) {
        if let result = result {
            completion(result)
        } else {
            completion(.failure(NSError(domain: "Mock error", code: -1, userInfo: nil)))
        }
    }
}
```

Now, in our tests, we can use `MockNetworkManager` instead of the real `NetworkManager`, allowing us to simulate various network conditions without making actual network requests.

```swift
import XCTest

class NetworkManagerTests: XCTestCase {
    func testFetchDataSuccess() {
        let mockService = MockNetworkManager()
        mockService.result = .success("Mock Data")
        let expectation = self.expectation(description: "fetchData")

        mockService.fetchData { result in
            switch result {
            case .success(let data):
                XCTAssertEqual(data, "Mock Data")
            case .failure:
                XCTFail("Expected success but got failure")
            }
            expectation.fulfill()
        }

        wait(for: [expectation], timeout: 1.0)
    }
}
```

- **Seam Explanation**: 
  - The protocol `NetworkService` acts as a **seam**. It allows us to swap out the behavior of `fetchData` for testing purposes.
  - The enabling point is when we inject the `MockNetworkManager` instead of the real `NetworkManager` during testing.
  - This approach isolates the test from the real network, making it faster and more reliable.

### 2. **Conditional Compilation as a Preprocessing Seam**:

While Swift doesn’t have preprocessing like C/C++, it does support **conditional compilation** using `#if` to enable different code paths for testing or debugging.

```swift
class Logger {
    func log(message: String) {
        #if DEBUG
        print("DEBUG: \(message)")
        #else
        // In production, logs might be sent to a remote server or ignored.
        #endif
    }
}
```

In this example, the `Logger` class only prints logs during the `DEBUG` build configuration. This allows you to change behavior for testing without editing the production code directly.

- **Seam Explanation**:
  - The `#if DEBUG` directive acts as a **preprocessing seam**, enabling different logging behavior based on the build configuration.
  - This allows you to log additional information during testing or debugging, while keeping production code clean.

### 3. **Link Seam through Dependency Injection**:

In Swift, **dependency injection** is a common way to achieve a **link seam**. For example, if `NetworkManager` depends on `URLSession`, we can inject a custom `URLSession` during testing.

```swift
class NetworkManager {
    private let session: URLSession

    init(session: URLSession = .shared) {
        self.session = session
    }

    func fetchData(completion: @escaping (Result<String, Error>) -> Void) {
        let url = URL(string: "https://example.com/api")!
        let task = session.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
            } else if let data = data, let result = String(data: data, encoding: .utf8) {
                completion(.success(result))
            } else {
                completion(.failure(NSError(domain: "Unknown Error", code: -1, userInfo: nil)))
            }
        }
        task.resume()
    }
}
```

- **Testing with a Custom `URLSession`**:

We can create a custom `URLSession` using `URLProtocol` to mock network responses:

```swift
class MockURLProtocol: URLProtocol {
    static var mockResponse: (Data?, URLResponse?, Error?)?

    override class func canInit(with request: URLRequest) -> Bool {
        return true
    }

    override class func canonicalRequest(for request: URLRequest) -> URLRequest {
        return request
    }

    override func startLoading() {
        if let (data, response, error) = MockURLProtocol.mockResponse {
            if let data = data {
                client?.urlProtocol(self, didLoad: data)
            }
            if let response = response {
                client?.urlProtocol(self, didReceive: response, cacheStoragePolicy: .notAllowed)
            }
            if let error = error {
                client?.urlProtocol(self, didFailWithError: error)
            }
        }
        client?.urlProtocolDidFinishLoading(self)
    }

    override func stopLoading() {}
}

// Test with MockURLProtocol
class NetworkManagerTests: XCTestCase {
    func testFetchDataWithMockSession() {
        let config = URLSessionConfiguration.ephemeral
        config.protocolClasses = [MockURLProtocol.self]
        let session = URLSession(configuration: config)
        let networkManager = NetworkManager(session: session)

        MockURLProtocol.mockResponse = (Data("Mock Data".utf8), nil, nil)

        let expectation = self.expectation(description: "fetchData")

        networkManager.fetchData { result in
            switch result {
            case .success(let data):
                XCTAssertEqual(data, "Mock Data")
            case .failure:
                XCTFail("Expected success but got failure")
            }
            expectation.fulfill()
        }

        wait(for: [expectation], timeout: 1.0)
    }
}
```

- **Seam Explanation**:
  - The `URLSession` parameter in `NetworkManager` allows us to inject a custom session during testing.
  - The enabling point is when we inject a session configured with `MockURLProtocol`, which simulates network responses.
  - This makes it possible to test network-related logic without actual HTTP requests.

### Summary:
Seams provide powerful ways to make code more testable. By using **object seams** through protocols, **preprocessing seams** with conditional compilation, and **link seams** using dependency injection, we can isolate dependencies and test behavior more effectively. In Swift, protocols and dependency injection are particularly useful for creating seams, allowing us to replace or modify behavior for testing without altering production code. This makes testing faster, more reliable, and allows for safer refactoring of legacy code.
