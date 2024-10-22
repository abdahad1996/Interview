The passage discusses a common challenge in software development: managing codebases that are heavily dependent on external APIs or third-party libraries. The main problem is that such code can become difficult to maintain, test, and extend because the core logic of the application is buried under a pile of API calls. Let’s break down the problem, the suggested solutions, and provide a parallel example in Swift for better understanding.

### Problem Summary
When building software, developers often need to decide whether to:
- Build custom functionality.
- Buy or integrate third-party solutions.
- Borrow or use open-source libraries.

Using third-party APIs can speed up development. For example, when integrating with a third-party email service, you might use a library instead of implementing the entire protocol yourself. But over time, this reliance on external APIs can create tightly coupled code that becomes hard to change or extend. Here’s why:
- **Dependency on External Libraries:** The code is tightly integrated with third-party APIs, making it hard to understand or refactor without affecting functionality.
- **Testing Difficulties:** Since most of the logic involves API calls, writing unit tests becomes challenging because it requires simulating the behavior of these APIs.
- **Lack of Modularity:** If all logic is embedded directly into methods that call the API, it is hard to reuse or adapt parts of the code without rewriting the entire structure.

The passage suggests two ways to refactor such code to make it more maintainable:
1. **Skin and Wrap the API:** Create wrappers around the API, making it easier to mock or replace in tests.
2. **Responsibility-Based Extraction:** Break down the logic into separate responsibilities, which makes the core logic easier to test independently of the API.

### Example in Swift: A Mailing List Application

Imagine building a Swift application that sends email notifications. To do this, you might use a third-party library like `Alamofire` for making network requests to an email service API.

#### Problematic Design: Tight Coupling
Here is a basic, tightly coupled version where all the logic is embedded directly in the method that sends the email:

```swift
import Foundation
import Alamofire

struct EmailService {
    let smtpHost: String
    let smtpUser: String
    let smtpPassword: String
    
    func sendEmail(to recipients: [String], subject: String, body: String) {
        let parameters: [String: Any] = [
            "host": smtpHost,
            "user": smtpUser,
            "password": smtpPassword,
            "recipients": recipients,
            "subject": subject,
            "body": body
        ]
        
        // Directly using Alamofire for network request
        AF.request("https://api.emailservice.com/send", method: .post, parameters: parameters).response { response in
            switch response.result {
            case .success:
                print("Email sent successfully")
            case .failure(let error):
                print("Failed to send email: \(error)")
            }
        }
    }
}
```

In this code:
- **Tight Coupling:** The `sendEmail` method directly uses `Alamofire` to make an API request.
- **Difficult to Test:** It’s hard to write a unit test for `sendEmail` without actually making a network request, which is undesirable during testing.
- **Lack of Abstraction:** If you want to change the email provider, you must modify the logic directly in this method.

#### Solution 1: Skin and Wrap the API

By wrapping the `Alamofire` logic in a separate class or protocol, you can decouple the API dependency:

```swift
protocol EmailAPI {
    func sendEmail(parameters: [String: Any], completion: @escaping (Result<Void, Error>) -> Void)
}

class AlamofireEmailAPI: EmailAPI {
    func sendEmail(parameters: [String: Any], completion: @escaping (Result<Void, Error>) -> Void) {
        AF.request("https://api.emailservice.com/send", method: .post, parameters: parameters).response { response in
            if let error = response.error {
                completion(.failure(error))
            } else {
                completion(.success(()))
            }
        }
    }
}

struct EmailService {
    private let api: EmailAPI
    
    init(api: EmailAPI) {
        self.api = api
    }
    
    func sendEmail(to recipients: [String], subject: String, body: String) {
        let parameters: [String: Any] = [
            "recipients": recipients,
            "subject": subject,
            "body": body
        ]
        
        api.sendEmail(parameters: parameters) { result in
            switch result {
            case .success:
                print("Email sent successfully")
            case .failure(let error):
                print("Failed to send email: \(error)")
            }
        }
    }
}
```

In this refactored version:
- **Decoupling**: `EmailService` depends on the `EmailAPI` protocol instead of `Alamofire` directly.
- **Easier Testing**: You can create a mock class conforming to `EmailAPI` for unit testing without making actual network requests.
- **Flexibility**: If you decide to switch from `Alamofire` to another library, only the `AlamofireEmailAPI` needs to change, not the `EmailService`.

#### Solution 2: Responsibility-Based Extraction

Instead of embedding everything in one class, separate the responsibilities into multiple components:

1. **EmailSender**: Responsible for constructing the email and invoking the API.
2. **EmailMessageBuilder**: Constructs the message details (recipients, subject, body).
3. **Scheduler**: Manages periodic checks to send emails (if needed).

```swift
struct EmailMessage {
    let recipients: [String]
    let subject: String
    let body: String
}

protocol EmailMessageBuilder {
    func buildMessage(to recipients: [String], subject: String, body: String) -> EmailMessage
}

class DefaultEmailMessageBuilder: EmailMessageBuilder {
    func buildMessage(to recipients: [String], subject: String, body: String) -> EmailMessage {
        return EmailMessage(recipients: recipients, subject: subject, body: body)
    }
}

class EmailSender {
    private let api: EmailAPI
    
    init(api: EmailAPI) {
        self.api = api
    }
    
    func send(email: EmailMessage) {
        let parameters: [String: Any] = [
            "recipients": email.recipients,
            "subject": email.subject,
            "body": email.body
        ]
        
        api.sendEmail(parameters: parameters) { result in
            switch result {
            case .success:
                print("Email sent successfully")
            case .failure(let error):
                print("Failed to send email: \(error)")
            }
        }
    }
}
```

Now, **EmailSender** is focused only on sending, while **EmailMessageBuilder** takes care of constructing messages. This design allows testing each part independently and adapting the implementation without affecting the rest of the system.

### Conclusion
The problem with API-heavy applications is that they can become difficult to test and change. By using strategies like **Skin and Wrap the API** or **Responsibility-Based Extraction**, you can decouple the business logic from third-party libraries. This makes the code easier to understand, test, and maintain, allowing for more flexibility as the application evolves. The Swift examples demonstrate how these concepts can be applied to a mailing list application, offering a cleaner and more testable design.
