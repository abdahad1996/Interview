### Problem Overview: Dependencies on Libraries

Using third-party libraries can significantly speed up development by providing ready-made solutions for common problems. However, this reliance on libraries can become a problem if the library is tightly integrated into the application code. This makes it difficult to replace, modify, or even test parts of your application without being dependent on the library. The primary challenges arise when:

1. **Vendor Lock-in**: If a library vendor changes terms, raises costs, or discontinues support, the application might be stuck with the library because replacing it would require a substantial rewrite.

2. **Testing Difficulties**: Libraries may include classes or methods that are difficult to mock or substitute during testing. This can be due to:
   - Classes being final or methods being non-overridable (sealed/final).
   - Singleton patterns that enforce only one instance.
   - Concrete types without interfaces, making dependency injection hard.

3. **Over-reliance on Implementation Details**: Directly calling library functions throughout the code makes it tightly coupled to that library. Even if a better or cheaper alternative arises, swapping it out can be very challenging due to this tight coupling.

### Theory: Managing Dependencies with Wrappers

One of the best practices to reduce over-reliance on libraries is to create **wrappers** around the library’s functionality. Wrappers are thin layers that abstract the third-party code, providing a seam between your application and the library. This practice provides several advantages:

1. **Ease of Replacement**: Since your application interacts with the wrapper rather than the library directly, changing the underlying library becomes easier.

2. **Simplified Testing**: Wrappers can provide interfaces that are easy to mock or replace during testing, allowing you to test your code without relying on the actual library behavior.

3. **Maintainable Code**: Wrappers can simplify the interface that your application uses, hiding complex or unnecessary library details, making the codebase easier to understand and maintain.

### Example in Swift: Wrapping a Library

Let’s consider a scenario in Swift where you might be using a third-party library for network requests, such as `Alamofire` (a popular networking library). Here’s an example of how direct usage of `Alamofire` could become problematic and how a wrapper can help.

#### Direct Use of `Alamofire` (Problematic)

```swift
import Alamofire

class UserService {
    func fetchUserDetails(userId: String, completion: @escaping (User?, Error?) -> Void) {
        let url = "https://api.example.com/users/\(userId)"
        AF.request(url).responseDecodable(of: User.self) { response in
            switch response.result {
            case .success(let user):
                completion(user, nil)
            case .failure(let error):
                completion(nil, error)
            }
        }
    }
}
```

In this example:
- The `UserService` class directly uses `Alamofire` to make a network request.
- It tightly couples the `UserService` class to the `Alamofire` API.
- Testing this code is difficult because mocking `AF.request` calls is not straightforward without special testing libraries.

#### Issues with Direct Dependency
1. **Vendor Lock-in**: If `Alamofire` becomes obsolete or a better library is found, replacing `AF.request` throughout the codebase would be tedious.
2. **Testing Difficulty**: It’s hard to test `fetchUserDetails` without making actual network calls, as it directly uses `Alamofire`.

#### Solution: Using a Wrapper

We can introduce a wrapper to abstract the networking details:

```swift
// Define a protocol that represents the network request behavior.
protocol NetworkService {
    func request<T: Decodable>(_ url: String, completion: @escaping (Result<T, Error>) -> Void)
}

// Implement the protocol using Alamofire.
class AlamofireNetworkService: NetworkService {
    func request<T: Decodable>(_ url: String, completion: @escaping (Result<T, Error>) -> Void) {
        AF.request(url).responseDecodable(of: T.self) { response in
            completion(response.result)
        }
    }
}

// Update UserService to depend on the protocol instead of Alamofire directly.
class UserService {
    private let networkService: NetworkService

    init(networkService: NetworkService) {
        self.networkService = networkService
    }

    func fetchUserDetails(userId: String, completion: @escaping (User?, Error?) -> Void) {
        let url = "https://api.example.com/users/\(userId)"
        networkService.request(url) { (result: Result<User, Error>) in
            switch result {
            case .success(let user):
                completion(user, nil)
            case .failure(let error):
                completion(nil, error)
            }
        }
    }
}
```

#### Benefits of Using a Wrapper
1. **Ease of Replacement**: If you later decide to switch from `Alamofire` to `URLSession` or another library, you only need to change the implementation of `NetworkService`, not the `UserService`.
   
2. **Simplified Testing**: You can easily mock `NetworkService` for testing:

```swift
class MockNetworkService: NetworkService {
    var shouldReturnError = false

    func request<T>(_ url: String, completion: @escaping (Result<T, Error>) -> Void) where T : Decodable {
        if shouldReturnError {
            completion(.failure(MockError.someError))
        } else {
            let mockUser = User(id: "1", name: "Test User")
            completion(.success(mockUser as! T))
        }
    }
}

class UserServiceTests: XCTestCase {
    func testFetchUserDetailsSuccess() {
        let mockNetworkService = MockNetworkService()
        let userService = UserService(networkService: mockNetworkService)

        userService.fetchUserDetails(userId: "1") { user, error in
            XCTAssertNotNil(user)
            XCTAssertEqual(user?.name, "Test User")
            XCTAssertNil(error)
        }
    }

    func testFetchUserDetailsFailure() {
        let mockNetworkService = MockNetworkService()
        mockNetworkService.shouldReturnError = true
        let userService = UserService(networkService: mockNetworkService)

        userService.fetchUserDetails(userId: "1") { user, error in
            XCTAssertNil(user)
            XCTAssertNotNil(error)
        }
    }
}
```

In the tests above:
- `MockNetworkService` simulates both successful and failed network responses without relying on real network conditions.
- This allows for **unit testing** of `UserService` without needing the `Alamofire` library or an actual network connection.

#### Summary

- **Problem**: Directly using libraries like `Alamofire` throughout your code can make it hard to switch libraries or isolate functionality for testing.
- **Theory**: By creating wrappers around library functionalities and depending on protocols or abstractions, you can decouple your code from specific libraries.
- **Solution**: Use dependency injection to provide a `NetworkService` protocol and its concrete implementation, allowing easy replacement and mocking.
- **Benefits**: This approach simplifies testing, reduces vendor lock-in, and improves the maintainability of the codebase.

By following this pattern, you can keep your code more flexible, easier to test, and better prepared for changes in the future, avoiding the pitfalls of heavy dependency on third-party libraries.
