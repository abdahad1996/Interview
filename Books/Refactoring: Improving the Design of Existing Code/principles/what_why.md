### Principles in Refactoring: Detailed Explanation with Before and After Examples in Swift

Refactoring is the process of restructuring the internal structure of software to make it easier to understand and modify without altering its observable behavior. It's an essential part of maintaining a clean codebase and ensuring that software remains adaptable to change over time. Here, we'll explore key concepts related to refactoring, their importance, and illustrate them with detailed before and after examples in Swift.

---

### 1. Defining Refactoring

**Refactoring (Noun):** A change made to the internal structure of software to make it easier to understand and cheaper to modify without changing its observable behavior.

- **Explanation**: Refactoring as a noun refers to the end result of improving the design of the code without altering what the program does. The goal is to make the code more maintainable, clear, and easier to modify in the future.

- **Example - Before and After**:
    - **Before**:
        ```swift
        func calculateDiscount(price: Double, isMember: Bool) -> Double {
            if isMember {
                return price * 0.9
            } else {
                return price
            }
        }
        ```

        This function is simple but has a conditional statement that might grow complex with more conditions in the future.

    - **After**:
        ```swift
        func calculateDiscount(price: Double, discountRate: Double) -> Double {
            return price * discountRate
        }

        func calculateMemberDiscount(price: Double) -> Double {
            return calculateDiscount(price: price, discountRate: 0.9)
        }

        func calculateNonMemberDiscount(price: Double) -> Double {
            return calculateDiscount(price: price, discountRate: 1.0)
        }
        ```

        In this version, we have extracted the calculation logic into a separate function. This makes the code easier to extend if more discount types are added.

---

### 2. The Two Hats

Kent Beck introduced the concept of **The Two Hats** to describe two primary activities during software development: **adding functionality** and **refactoring**.

- **Adding Functionality**: This involves writing new code to add new capabilities or features. It should not involve altering the existing structure of the code.
- **Refactoring**: This focuses on improving the structure of existing code without adding new behavior. The goal is to enhance code readability and maintainability.

- **Example - Switching Hats**:
    - **Before - Adding Functionality Hat**:
        ```swift
        func applyDiscount(price: Double, discountType: String) -> Double {
            if discountType == "member" {
                return price * 0.9
            } else if discountType == "student" {
                return price * 0.85
            } else {
                return price
            }
        }
        ```

        Initially, we are adding functionality to handle different discount types. However, this becomes cumbersome as the conditions grow.

    - **Refactoring Hat - After**:
        ```swift
        protocol DiscountStrategy {
            func applyDiscount(price: Double) -> Double
        }

        class MemberDiscount: DiscountStrategy {
            func applyDiscount(price: Double) -> Double {
                return price * 0.9
            }
        }

        class StudentDiscount: DiscountStrategy {
            func applyDiscount(price: Double) -> Double {
                return price * 0.85
            }
        }

        class NoDiscount: DiscountStrategy {
            func applyDiscount(price: Double) -> Double {
                return price
            }
        }

        func applyDiscount(price: Double, strategy: DiscountStrategy) -> Double {
            return strategy.applyDiscount(price: price)
        }
        ```

        After refactoring, each discount type is encapsulated in its own class, following the Strategy Pattern. This improves the code's scalability and makes it easier to add new discount types.

---

### 3. Why Should We Refactor?

Refactoring offers several key benefits, such as improving software design, making code easier to understand, helping find bugs, and ultimately speeding up development.

#### a. Refactoring Improves the Design of Software

- **Explanation**: Without refactoring, the architecture of software tends to decay over time as quick fixes and patches accumulate. Refactoring helps maintain a clean design and prevent technical debt.

- **Example - Before and After**:
    - **Before**:
        ```swift
        func fetchUserData(userId: String) {
            // Fetch user data from API
            // Parse JSON response
            // Map JSON to User model
            // Save User data to database
        }
        ```

        This function performs multiple tasks, violating the Single Responsibility Principle.

    - **After**:
        ```swift
        func fetchUserData(userId: String) {
            let data = fetchUserDataFromAPI(userId: userId)
            let user = parseUserData(data: data)
            saveUserToDatabase(user: user)
        }

        func fetchUserDataFromAPI(userId: String) -> Data {
            // API call logic
        }

        func parseUserData(data: Data) -> User {
            // JSON parsing logic
        }

        func saveUserToDatabase(user: User) {
            // Save to database logic
        }
        ```

        Now, each function has a clear responsibility, making the code easier to modify and maintain.

#### b. Refactoring Makes Software Easier to Understand

- **Explanation**: Code that is clean and well-structured is easier to read and understand. This reduces the time it takes for developers to comprehend and work with the code.

- **Example - Renaming for Clarity**:
    - **Before**:
        ```swift
        func cvc(_ a: Double, _ b: Double) -> Double {
            return a * b * 0.05
        }
        ```

        The method name `cvc` is not clear, making it difficult for others to understand its purpose.

    - **After**:
        ```swift
        func calculateCommission(amount: Double, rate: Double) -> Double {
            return amount * rate * 0.05
        }
        ```

        With a more descriptive name, the purpose of the function is immediately clear to anyone reading the code.

#### c. Refactoring Helps Find Bugs

- **Explanation**: Refactoring forces developers to deeply understand the code, which often reveals hidden bugs or edge cases that were previously overlooked.

- **Example - Uncovering Bugs**:
    - **Before**:
        ```swift
        func processOrder(quantity: Int, pricePerItem: Double) -> Double {
            let discount = (quantity > 10) ? 0.1 : 0.0
            return Double(quantity) * pricePerItem - discount
        }
        ```

        This code incorrectly applies the discount directly without considering the total amount.

    - **After**:
        ```swift
        func processOrder(quantity: Int, pricePerItem: Double) -> Double {
            let subtotal = Double(quantity) * pricePerItem
            let discount = (quantity > 10) ? subtotal * 0.1 : 0.0
            return subtotal - discount
        }
        ```

        Refactoring helped us identify and correct the mistake in the discount calculation.

#### d. Refactoring Helps Me Program Faster

- **Explanation**: By keeping the codebase clean, refactoring allows new features to be added more quickly, as developers spend less time deciphering complex, tangled code.

- **Example - Before and After**:
    - **Before**:
        ```swift
        func addUser(name: String, email: String, age: Int) {
            // Perform validation
            // Store user in database
            // Send welcome email
        }
        ```

        This function is doing too many things, making it harder to adjust any one aspect of adding a user.

    - **After**:
        ```swift
        func addUser(name: String, email: String, age: Int) {
            guard validateUser(name: name, email: email, age: age) else { return }
            storeUser(name: name, email: email, age: age)
            sendWelcomeEmail(to: email)
        }

        func validateUser(name: String, email: String, age: Int) -> Bool {
            // Validation logic
        }

        func storeUser(name: String, email: String, age: Int) {
            // Store user in database logic
        }

        func sendWelcomeEmail(to email: String) {
            // Send email logic
        }
        ```

        By splitting the responsibilities, we make each function easier to understand and modify, speeding up future changes.

---

### 4. When Should We Refactor?

#### The Rule of Three

- **Explanation**: The Rule of Three is a guideline that suggests refactoring when you notice similar code being written for the third time. It helps avoid code duplication and encourages reuse.

- **Example - Duplicate Logic**:
    - **Before**:
        ```swift
        func sendEmailToAdmin() { print("Sending email to admin") }
        func sendEmailToUser() { print("Sending email to user") }
        func sendEmailToSupport() { print("Sending email to support") }
        ```

    - **After**:
        ```swift
        func sendEmail(to recipient: String) {
            print("Sending email to \(recipient)")
        }
        ```

        After observing the third similar function, refactoring was applied to eliminate duplication.

Let's go through the concepts of refactoring, using examples in Swift to illustrate each one:

### 1. **Preparatory Refactoring**
   - **Concept**: This involves refactoring code to make adding a new feature easier. For example, you might find a function that does almost what you need but has some hard-coded values. Instead of duplicating the function and changing the values, you refactor it to accept parameters, making it more flexible.
   - **Example**:
     Before refactoring, you might have a function like this:
     ```swift
     func greetUser() {
         print("Hello, John!")
     }
     ```
     Suppose you want to greet different users. Instead of copying this function, you can refactor it like this:
     ```swift
     func greetUser(name: String) {
         print("Hello, \(name)!")
     }
     ```
     Now, you can use `greetUser(name: "John")` or `greetUser(name: "Emily")` without duplicating the function.

### 2. **Comprehension Refactoring**
   - **Concept**: This type of refactoring is aimed at making code easier to understand. It involves renaming variables, breaking down complex functions, or reorganizing code for clarity. This can help others (or yourself in the future) understand the code faster.
   - **Example**:
     Before refactoring:
     ```swift
     func calc(t: Double, r: Double, n: Double) -> Double {
         return t * pow(1 + r, n)
     }
     ```
     It might be unclear what `t`, `r`, and `n` represent. After refactoring:
     ```swift
     func calculateCompoundInterest(principal: Double, rate: Double, years: Double) -> Double {
         return principal * pow(1 + rate, years)
     }
     ```
     This makes the function's purpose and parameters much clearer, making it easier for others to understand.

### 3. **Litter-Pickup Refactoring**
   - **Concept**: This involves cleaning up small issues in the code while working on a related task. It could mean simplifying convoluted logic or combining nearly identical functions. It’s about making the code a little better without losing focus on the main task.
   - **Example**:
     Before refactoring:
     ```swift
     func discountForVIP(price: Double) -> Double {
         return price * 0.9
     }

     func discountForRegular(price: Double) -> Double {
         return price * 0.95
     }
     ```
     After refactoring:
     ```swift
     func discount(price: Double, rate: Double) -> Double {
         return price * rate
     }
     ```
     Now, you can use `discount(price: 100, rate: 0.9)` for VIPs or `discount(price: 100, rate: 0.95)` for regular customers, avoiding duplicate logic.

### 4. **Planned vs. Opportunistic Refactoring**
   - **Concept**: Refactoring can be planned (a dedicated time to improve code) or opportunistic (refactoring while adding features or fixing bugs). Opportunistic refactoring is more common and allows for continuous improvement without significant interruptions.
   - **Example**: 
     Suppose you're adding a feature that needs to display user information:
     ```swift
     func displayUserDetails() {
         let name = "John"
         let age = 30
         print("Name: \(name), Age: \(age)")
     }
     ```
     During this process, you notice the values for `name` and `age` could be parameterized for reusability:
     ```swift
     func displayUserDetails(name: String, age: Int) {
         print("Name: \(name), Age: \(age)")
     }
     ```
     This refactoring was opportunistic—it made future changes easier as you added a new feature.

### 5. **Long-Term Refactoring**
   - **Concept**: Sometimes, refactoring requires a larger effort, such as replacing a library or reorganizing a large portion of the codebase. These are often best approached incrementally to avoid breaking the code.
   - **Example**:
     Before refactoring:
     ```swift
     func sendRequest(url: String) {
         // URLSession code for network request
     }

     func sendSecureRequest(url: String) {
         // URLSession code with additional security parameters
     }
     ```
     Suppose you need to switch to a different networking library. Instead of replacing everything at once, you can start by creating an abstraction:
     ```swift
     protocol NetworkService {
         func sendRequest(url: String)
     }

     class URLSessionService: NetworkService {
         func sendRequest(url: String) {
             // URLSession code here
         }
     }

     class SecureURLSessionService: NetworkService {
         func sendRequest(url: String) {
             // URLSession code with security settings here
         }
     }
     ```
     Then, update parts of the code to use the `NetworkService` protocol instead of the concrete implementation. This makes transitioning to a new library easier, as you only need to change the implementation in one place.

### Summary
Refactoring can be a powerful way to make your codebase more maintainable and easier to work with, especially when done incrementally and strategically:
- **Preparatory**: Refactor before adding new features.
- **Comprehension**: Refactor to better understand and clarify the code.
- **Litter-Pickup**: Clean up as you go without losing focus on current tasks.
- **Opportunistic vs. Planned**: Refactor as part of your regular workflow or plan dedicated time if the codebase needs it.
- **Long-Term**: Tackle large changes gradually to keep the code stable.

These examples in Swift demonstrate how simple refactoring steps can lead to better code that is easier to maintain and extend.
