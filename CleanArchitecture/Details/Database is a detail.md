# Clean Architecture: Treating the Database as a Detail in Swift

## Overview
In Clean Architecture, the database is considered a **detail**, not a core architectural element. While the **data model** is central to the architecture, the **database** is simply a mechanism for storing and retrieving data. Separating the two helps maintain flexibility, testability, and maintainability.

## Key Concepts

- **Data Model vs. Database**: The **data model** defines the structure and relationships of entities like `User`, `Order`, or `Product`. The **database** is a storage mechanism (e.g., SQLite, CoreData, JSON files). By abstracting the database, the core application logic remains unchanged even if the storage mechanism changes.

- **Decoupling Business Logic from Data Access**: Business logic should not be tied to specific storage mechanisms (e.g., SQL queries or CoreData). This can be achieved using protocols or interfaces that define operations for interacting with data.

    ```swift
    struct User {
        let id: UUID
        let name: String
        let email: String
    }

    protocol UserRepository {
        func getUser(by id: UUID) -> User?
        func saveUser(_ user: User)
    }
    ```

- **Concrete Implementations as Details**: Implementations of `UserRepository` for different storage mechanisms should be hidden behind the protocol, allowing for easy replacement.

    ```swift
    class CoreDataUserRepository: UserRepository {
        // Implementation using CoreData
    }

    class InMemoryUserRepository: UserRepository {
        // Implementation for testing
    }
    ```

- **Dependency Injection**: Injecting dependencies allows choosing different repository implementations at runtime, improving flexibility.

    ```swift
    class UserService {
        private let repository: UserRepository

        init(repository: UserRepository) {
            self.repository = repository
        }

        func getUserName(by id: UUID) -> String? {
            return repository.getUser(by: id)?.name
        }
    }
    ```

- **Example Usage**: Switch between repository implementations based on the environment:

    ```swift
    let repository: UserRepository = environment == .production ? CoreDataUserRepository() : InMemoryUserRepository()
    let userService = UserService(repository: repository)
    ```

## Benefits

- **Maintainability**: Isolating storage details means that the underlying database can change without affecting the rest of the system.
- **Testability**: Using mock or in-memory repositories makes unit testing faster and simpler.
- **Flexibility**: It’s easy to switch between different storage mechanisms (e.g., local databases, cloud storage) without rewriting the business logic.
- **Clean Code**: This approach adheres to the **Dependency Inversion Principle** by depending on abstractions rather than concrete classes.

## Example Test

Here’s a test example using an in-memory repository:

```swift
func testUserService() {
    let repository = InMemoryUserRepository()
    let userService = UserService(repository: repository)

    userService.registerNewUser(name: "Alice", email: "alice@example.com")
    let userName = userService.getUserName(by: UUID())
    XCTAssertEqual(userName, "Alice")
}
