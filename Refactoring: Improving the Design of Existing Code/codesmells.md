Let's explore each of the "bad smells" in code, as described by Kent Beck and Martin Fowler. I'll explain each concept clearly and provide examples in Swift when possible to illustrate the concepts.

---

### 1. **Mysterious Name**
   - **Concept**: When a variable, function, or class name is unclear, it can make understanding the code difficult. Clear naming is crucial to maintainable code.
   - **Example in Swift**:
     - **Before**:
       ```swift
       func calc() {
           // What does this do?
       }
       ```
     - **After**:
       ```swift
       func calculateTotalPrice() {
           // The name now indicates its purpose.
       }
       ```

### 2. **Duplicated Code**
   - **Concept**: When the same code exists in multiple places, it’s harder to maintain because changes must be made everywhere the code is duplicated.
   - **Example in Swift**:
     - **Before**:
       ```swift
       func calculateDiscountedPrice() -> Double {
           return price * 0.9
       }

       func calculateVIPDiscountedPrice() -> Double {
           return price * 0.8
       }
       ```
     - **After**:
       ```swift
       func calculateDiscountedPrice(rate: Double) -> Double {
           return price * rate
       }
       ```
       Now, a single function can be used for both regular and VIP discounts.

### 3. **Long Function**
   - **Concept**: Long functions are hard to read and understand. Breaking them into smaller functions can make them easier to manage.
   - **Example in Swift**:
     - **Before**:
       ```swift
       func processOrder() {
           // Validate order
           if !isValidOrder {
               print("Invalid order")
               return
           }
           // Calculate total price
           let total = calculateTotalPrice()
           // Apply discount
           let discountedTotal = total * 0.9
           // Print receipt
           print("Order processed: \(discountedTotal)")
       }
       ```
     - **After**:
       ```swift
       func processOrder() {
           guard isValidOrder else {
               print("Invalid order")
               return
           }
           let total = calculateDiscountedTotal()
           printReceipt(for: total)
       }

       func calculateDiscountedTotal() -> Double {
           let total = calculateTotalPrice()
           return total * 0.9
       }

       func printReceipt(for amount: Double) {
           print("Order processed: \(amount)")
       }
       ```
       The code is now easier to understand with each part having a clear responsibility.

### 4. **Long Parameter List**
   - **Concept**: Functions with too many parameters can be hard to use. Group related parameters into objects.
   - **Example in Swift**:
     - **Before**:
       ```swift
       func createOrder(id: Int, item: String, quantity: Int, price: Double) {
           // Create order logic
       }
       ```
     - **After**:
       ```swift
       struct Order {
           let id: Int
           let item: String
           let quantity: Int
           let price: Double
       }

       func createOrder(order: Order) {
           // Create order logic
       }
       ```
       This simplifies the method signature and makes it easier to pass around related data.

### 5. **Global Data**
   - **Concept**: Global data is accessible from anywhere, making it difficult to track where changes come from. It’s better to encapsulate and restrict access.
   - **Example in Swift**:
     - **Before**:
       ```swift
       var userSettings = ["theme": "dark"]
       ```
     - **After**:
       ```swift
       class UserSettings {
           private var settings = ["theme": "dark"]
           
           func setTheme(_ theme: String) {
               settings["theme"] = theme
           }

           func getTheme() -> String {
               return settings["theme"] ?? "default"
           }
       }
       ```
       Now, changes to `userSettings` can only be made through `UserSettings` methods, making the code safer.

### 6. **Mutable Data**
   - **Concept**: Mutable data can be risky because changes in one place can unexpectedly affect other parts of the program. Use immutable data when possible.
   - **Example in Swift**:
     - **Before**:
       ```swift
       var total = 0
       total += 10
       total += 20
       ```
     - **After**:
       ```swift
       let total = calculateTotal(base: 0, additions: [10, 20])
       
       func calculateTotal(base: Int, additions: [Int]) -> Int {
           return additions.reduce(base, +)
       }
       ```
       The data is now immutable, reducing the risk of unexpected changes.

### 7. **Divergent Change**
   - **Concept**: When one class or module is changed frequently for different reasons, it may be trying to do too much. Splitting responsibilities can make changes easier.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class ReportGenerator {
           func generatePDF() { /* PDF logic */ }
           func generateCSV() { /* CSV logic */ }
       }
       ```
     - **After**:
       ```swift
       class PDFReportGenerator {
           func generate() { /* PDF logic */ }
       }

       class CSVReportGenerator {
           func generate() { /* CSV logic */ }
       }
       ```
       Now, changing the logic for PDF generation doesn’t affect CSV generation.

### 8. **Shotgun Surgery**
   - **Concept**: When making a change requires modifying many different classes, it indicates that logic is too spread out. Group related logic together.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Order {
           func calculateTotal() { /* logic */ }
       }

       class Invoice {
           func calculateTotal() { /* similar logic */ }
       }
       ```
     - **After**:
       ```swift
       class PriceCalculator {
           func calculateTotal(for order: Order) { /* logic */ }
       }
       ```
       Now, changes to the total calculation only need to be made in one place.

### 9. **Feature Envy**
   - **Concept**: A method in one class accesses data from another class excessively. It often indicates that the method belongs in the other class.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Cart {
           var items: [Item]
           
           func calculateTotalPrice() -> Double {
               return items.reduce(0) { $0 + $1.price }
           }
       }
       ```
     - **After**:
       ```swift
       class Item {
           var price: Double
       }

       class Cart {
           var items: [Item]
           
           func calculateTotalPrice() -> Double {
               return items.reduce(0) { $0 + $1.getPrice() }
           }
       }

       extension Item {
           func getPrice() -> Double {
               return price
           }
       }
       ```

### 10. **Data Clumps**
   - **Concept**: Data clumps occur when groups of data frequently appear together in classes or methods, like parameters that are always passed together. This indicates that these data elements likely belong together in their own object or class.
   - **Example in Swift**:
     - **Before**:
       ```swift
       func createAddress(street: String, city: String, zipCode: String) {
           // Use street, city, and zipCode
       }
       ```
     - **After**:
       ```swift
       struct Address {
           let street: String
           let city: String
           let zipCode: String
       }

       func createAddress(address: Address) {
           // Use address.street, address.city, address.zipCode
       }
       ```
       Now, the related fields are encapsulated within an `Address` object, making the method signature simpler and the relationship between the data elements more explicit.

### 11. **Primitive Obsession**
   - **Concept**: This occurs when you use primitive data types (e.g., strings, integers) to represent concepts in your domain instead of creating custom types that better represent them. It can make the code harder to read and maintain.
   - **Example in Swift**:
     - **Before**:
       ```swift
       let price: Double = 100.0 // Represents a monetary value.
       ```
     - **After**:
       ```swift
       struct Money {
           let amount: Double
       }

       let price = Money(amount: 100.0)
       ```
       By creating a `Money` type, you make it clear that the `amount` represents money, and you can add methods specific to money, like currency conversion or formatting.

### 12. **Repeated Switches**
   - **Concept**: When the same `switch` statement or conditional logic appears in multiple places, it’s a sign that the logic should be encapsulated or replaced with polymorphism to reduce duplication.
   - **Example in Swift**:
     - **Before**:
       ```swift
       switch userType {
       case "admin":
           // Admin logic
       case "guest":
           // Guest logic
       default:
           // Default logic
       }

       // Repeated somewhere else
       switch userType {
       case "admin":
           // Different admin logic
       case "guest":
           // Different guest logic
       default:
           // Different default logic
       }
       ```
     - **After**:
       ```swift
       protocol UserType {
           func performAction()
       }

       class Admin: UserType {
           func performAction() {
               // Admin-specific logic
           }
       }

       class Guest: UserType {
           func performAction() {
               // Guest-specific logic
           }
       }

       let user: UserType = Admin() // or Guest()
       user.performAction()
       ```
       This replaces the repetitive `switch` logic with polymorphism, making the code easier to extend and modify.

### 13. **Loops**
   - **Concept**: Traditional loops can make code harder to read, especially when modern functional programming constructs like `map`, `filter`, and `reduce` are available.
   - **Example in Swift**:
     - **Before**:
       ```swift
       var names: [String] = []
       for person in people {
           names.append(person.name)
       }
       ```
     - **After**:
       ```swift
       let names = people.map { $0.name }
       ```
       Using `map` makes the transformation more concise and easier to understand.

### 14. **Lazy Element**
   - **Concept**: A class or function that doesn’t do enough to justify its existence can make the code more complex than necessary. It’s better to inline or remove such elements.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Utility {
           func getGreeting() -> String {
               return "Hello, World!"
           }
       }

       let utility = Utility()
       print(utility.getGreeting())
       ```
     - **After**:
       ```swift
       func getGreeting() -> String {
           return "Hello, World!"
       }

       print(getGreeting())
       ```
       The `Utility` class was unnecessary for such a simple task, so it was removed to simplify the code.

### 15. **Speculative Generality**
   - **Concept**: This occurs when code is written to be more flexible or general than needed, often because developers anticipate future requirements that never materialize. It adds unnecessary complexity.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class User {
           func performAction(actionType: String) {
               // Handle different action types
           }
       }
       ```
       This class is designed to handle any possible action types, even if only one is currently needed.
     - **After**:
       ```swift
       class User {
           func performAction() {
               // Handle the specific action needed now
           }
       }
       ```
       Simplifying the method makes the code easier to understand and maintain until additional actions are genuinely needed.

### 16. **Temporary Field**
   - **Concept**: When a field in a class is only used under certain conditions, it can make the class harder to understand. This often indicates that the field or related behavior belongs in a separate class.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Order {
           var discountCode: String? = nil

           func applyDiscount() {
               if let code = discountCode {
                   // Apply discount logic
               }
           }
       }
       ```
     - **After**:
       ```swift
       class Order {
           private var discount: Discount?

           func applyDiscount() {
               discount?.apply(to: self)
           }
       }

       class Discount {
           func apply(to order: Order) {
               // Apply discount logic
           }
       }
       ```
       Now, the discount logic is encapsulated in its own class, making `Order` simpler.

### 17. **Message Chains**
   - **Concept**: This happens when a method calls another object’s method, which in turn calls another method, and so on. It makes code tightly coupled to the structure of the objects.
   - **Example in Swift**:
     - **Before**:
       ```swift
       let city = user.address?.city
       ```
       This is a simple example, but if it becomes longer and more nested, it can be problematic.
     - **After**:
       ```swift
       extension User {
           func getCity() -> String? {
               return address?.city
           }
       }

       let city = user.getCity()
       ```
       The `getCity` method hides the details of how the city is retrieved.

### 18. **Middle Man**
   - **Concept**: When a class delegates too much of its work to another class, it becomes unnecessary. It’s often better to remove the middle man and have clients interact directly with the relevant class.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Manager {
           let assistant = Assistant()

           func scheduleMeeting() {
               assistant.scheduleMeeting()
           }
       }

       class Assistant {
           func scheduleMeeting() {
               // Logic to schedule a meeting
           }
       }
       ```
     - **After**:
       ```swift
       class Assistant {
           func scheduleMeeting() {
               // Logic to schedule a meeting
           }
       }

       let assistant = Assistant()
       assistant.scheduleMeeting()
       ```
       The `Manager` class was just delegating calls, so it was removed.

### 19. **Large Class**
   - **Concept**: A class that tries to handle too many responsibilities becomes difficult to maintain. It’s often better to split it into smaller, focused classes.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class UserProfile {
           var name: String
           var age: Int
           var address: String
           var phoneNumber: String

           func updateAddress(newAddress: String) { /* logic */ }
           func updatePhoneNumber(newNumber: String) { /* logic */ }
           func updateName(newName: String) { /* logic */ }
       }
       ```
     - **After**:
       ```swift
       class ContactInfo {
           var address: String
           var phoneNumber: String
       }

       class UserProfile {
           var name: String
           var age: Int
           var contactInfo: ContactInfo
       }
       ```
       Now, `UserProfile` is more focused, with contact information encapsulated in its own class.

       Thanks for pointing that out! Let's cover the remaining code smells after **Large Class** with examples in Swift.



### 20. **Alternative Classes with Different Interfaces**
   - **Concept**: This occurs when classes are meant to be interchangeable but have different method names or interfaces. The inconsistency makes it harder to use these classes in a consistent way.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class XMLParser {
           func parseXML(data: Data) {
               // XML parsing logic
           }
       }

       class JSONParser {
           func parseJSON(data: Data) {
               // JSON parsing logic
           }
       }
       ```
     - **After**:
       ```swift
       protocol DataParser {
           func parse(data: Data)
       }

       class XMLParser: DataParser {
           func parse(data: Data) {
               // XML parsing logic
           }
       }

       class JSONParser: DataParser {
           func parse(data: Data) {
               // JSON parsing logic
           }
       }
       ```
       By using a protocol, you ensure that both parsers have a consistent interface, making it easier to swap them out.

### 21. **Data Class**
   - **Concept**: A **Data Class** is a class that only has fields and getters/setters but no meaningful behavior. It often indicates that some behavior should be moved into the class itself.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class User {
           var name: String
           var age: Int
       }
       
       func printUserDetails(user: User) {
           print("Name: \(user.name), Age: \(user.age)")
       }
       ```
     - **After**:
       ```swift
       class User {
           var name: String
           var age: Int

           func printDetails() {
               print("Name: \(name), Age: \(age)")
           }
       }
       ```
       Now, the `printDetails` method is encapsulated within the `User` class, giving it some behavior instead of making it a mere data holder.

### 22. **Refused Bequest**
   - **Concept**: This occurs when a subclass inherits methods or data it doesn’t need. It might be a sign that the subclass doesn’t belong in the same hierarchy.
   - **Example in Swift**:
     - **Before**:
       ```swift
       class Animal {
           func makeSound() {
               // Generic sound logic
           }
       }

       class Fish: Animal {
           // Fish doesn't make sounds but inherits makeSound()
       }
       ```
     - **After**:
       ```swift
       class Fish {
           // Fish doesn't inherit unnecessary methods from Animal
       }

       class Mammal: Animal {
           override func makeSound() {
               // Mammals make sounds
           }
       }
       ```
       By separating `Fish` from `Animal`, you avoid unnecessary inheritance and make the class structure more logical.

### 23. **Comments as a Deodorant**
   - **Concept**: Comments should clarify why certain decisions were made, not what the code is doing. If you find yourself needing comments to explain *what* the code does, it may indicate that the code needs refactoring for clarity.
   - **Example in Swift**:
     - **Before**:
       ```swift
       // Calculate the discount for premium customers
       let discount = price * 0.15
       ```
     - **After**:
       ```swift
       func calculatePremiumDiscount(for price: Double) -> Double {
           return price * 0.15
       }

       let discount = calculatePremiumDiscount(for: price)
       ```
       The method name `calculatePremiumDiscount` makes the logic self-explanatory, reducing the need for comments.

---


These examples illustrate how to identify "bad smells" in code and how to address them through refactoring. By making changes that improve clarity, reduce duplication, and encapsulate complexity, you create a codebase that is easier to maintain, extend, and understand.
