Let's dive deeper into each concept of refactoring, providing detailed explanations along with before and after examples in Swift. These examples will illustrate how the refactoring improves readability, maintainability, and sometimes performance.

### 1. **Extract Function**
   - **Concept**: This involves moving a block of code into a separate function, giving it a descriptive name. It’s useful for breaking down large methods, making code easier to read and understand.
   - **Why?**: It allows each function to focus on a single responsibility, adhering to the Single Responsibility Principle.
   
   - **Before**:
     ```swift
     func calculateTotalPrice(items: [Item]) -> Double {
         var total = 0.0
         for item in items {
             total += item.price * Double(item.quantity)
         }
         return total
     }
     ```
     The `calculateTotalPrice` function is doing two things—iterating through items and calculating their price. This can be broken down for better readability.

   - **After**:
     ```swift
     func calculateTotalPrice(items: [Item]) -> Double {
         var total = 0.0
         for item in items {
             total += calculateItemPrice(item: item)
         }
         return total
     }

     func calculateItemPrice(item: Item) -> Double {
         return item.price * Double(item.quantity)
     }
     ```
     Now, `calculateTotalPrice` is focused on summing up the total, and `calculateItemPrice` handles the price calculation for an individual item, making the code cleaner.

### 2. **Inline Function**
   - **Concept**: This is the reverse of Extract Function. If a function is too small or is only used once, you might inline it to simplify the code.
   - **Why?**: It reduces unnecessary indirection and can make the flow of logic easier to follow.
   
   - **Before**:
     ```swift
     func calculateServiceCharge(amount: Double) -> Double {
         return amount * 0.1
     }

     let charge = calculateServiceCharge(amount: 100.0)
     ```
     Here, `calculateServiceCharge` is very simple and could be inlined.

   - **After**:
     ```swift
     let charge = 100.0 * 0.1
     ```
     Inlining this function makes the code more straightforward since the function only contained a simple calculation.

### 3. **Move Function**
   - **Concept**: This involves moving a function from one class to another if it logically belongs in the other class.
   - **Why?**: It helps adhere to the principle of encapsulation and keeps classes focused on their respective responsibilities.
   
   - **Before**:
     ```swift
     class Order {
         func calculateDiscount() -> Double {
             return 5.0
         }
     }

     class Customer {
         var orders: [Order] = []
     }
     ```
     Here, the discount calculation is in `Order`, but it might make more sense in `Customer` since discounts might depend on customer status.

   - **After**:
     ```swift
     class Order {
         // No discount logic here.
     }

     class Customer {
         var orders: [Order] = []

         func calculateDiscount() -> Double {
             return 5.0
         }
     }
     ```
     Now, `Customer` has control over the discount logic, making it easier to customize discounts based on customer-specific criteria.

### 4. **Replace Conditional with Polymorphism**
   - **Concept**: Replace conditionals with polymorphism by creating subclasses or conforming types, each handling a specific behavior.
   - **Why?**: This approach reduces complex `if` or `switch` statements, making the code more flexible and easier to extend.
   
   - **Before**:
     ```swift
     func calculateShippingCost(order: Order) -> Double {
         if order.type == "Express" {
             return 20.0
         } else if order.type == "Standard" {
             return 10.0
         } else {
             return 0.0
         }
     }
     ```
     This `if-else` chain grows complex as more shipping types are added.

   - **After**:
     ```swift
     protocol Shipping {
         func cost() -> Double
     }

     class ExpressShipping: Shipping {
         func cost() -> Double { return 20.0 }
     }

     class StandardShipping: Shipping {
         func cost() -> Double { return 10.0 }
     }

     class FreeShipping: Shipping {
         func cost() -> Double { return 0.0 }
     }

     // Usage:
     let shipping: Shipping = ExpressShipping()
     let cost = shipping.cost()
     ```
     By using polymorphism, adding new shipping types now only requires creating a new class conforming to `Shipping`, without modifying existing logic.

### 5. **Replace Temp with Query**
   - **Concept**: Replace temporary variables with functions that calculate the value directly. This ensures consistency and reduces the chances of using outdated values.
   - **Why?**: It makes code easier to maintain and reduces bugs caused by improper updates to temporary variables.
   
   - **Before**:
     ```swift
     let basePrice = item.quantity * item.price
     let discount = basePrice * 0.1
     let finalPrice = basePrice - discount
     ```
     Here, `basePrice` is a temporary variable that could be replaced with a computed property.

   - **After**:
     ```swift
     var finalPrice: Double {
         return basePrice - discount
     }

     var basePrice: Double {
         return item.quantity * item.price
     }

     var discount: Double {
         return basePrice * 0.1
     }
     ```
     Now, the calculation logic is encapsulated in computed properties, ensuring that the `basePrice` is always calculated consistently.

### 6. **Split Phase**
   - **Concept**: Separate the computation of data from how it is formatted or presented.
   - **Why?**: This makes the code easier to change, especially if the logic or formatting needs to be updated independently.
   
   - **Before**:
     ```swift
     func printInvoice(invoice: Invoice) -> String {
         var result = "Invoice for \(invoice.customer)\n"
         var totalAmount = 0.0
         for item in invoice.items {
             result += "\(item.name): \(item.price * Double(item.quantity))\n"
             totalAmount += item.price * Double(item.quantity)
         }
         result += "Total: \(totalAmount)"
         return result
     }
     ```
     Here, calculation and formatting are mixed together.

   - **After**:
     ```swift
     func calculateInvoiceData(invoice: Invoice) -> (items: [(name: String, amount: Double)], totalAmount: Double) {
         var totalAmount = 0.0
         var items = [(String, Double)]()
         for item in invoice.items {
             let amount = item.price * Double(item.quantity)
             items.append((item.name, amount))
             totalAmount += amount
         }
         return (items, totalAmount)
     }

     func renderInvoiceText(data: (items: [(name: String, amount: Double)], totalAmount: Double)) -> String {
         var result = "Invoice Details:\n"
         for item in data.items {
             result += "\(item.0): \(item.1)\n"
         }
         result += "Total: \(data.totalAmount)"
         return result
     }

     let data = calculateInvoiceData(invoice: invoice)
     let result = renderInvoiceText(data: data)
     ```
     Now, the logic for calculating the invoice data is separated from the logic for rendering it, making it easier to modify either independently.

### 7. **Creating Polymorphic Calculator**
   - **Concept**: Use polymorphism to handle variations in behavior, such as different rules for calculating values.
   - **Why?**: It makes the code more scalable and easier to maintain when new variations are added.
   
   - **Before**:
     ```swift
     func calculateAmount(for performance: Performance) -> Double {
         switch performance.type {
         case .comedy:
             return 300 + Double(performance.audience) * 0.5
         case .tragedy:
             return 400 + (performance.audience > 30 ? Double(performance.audience - 30) * 1.5 : 0)
         default:
             return 0
         }
     }
     ```
     Here, the logic is tied up in a `switch` statement that could grow as more performance types are added.

   - **After**:
     ```swift
     class PerformanceCalculator {
         let performance: Performance

         init(performance: Performance) {
             self.performance = performance
         }

         func amount() -> Double {
             return 0.0
         }
     }

     class ComedyCalculator: PerformanceCalculator {
         override func amount() -> Double {
             return 300 + Double(performance.audience) * 0.5
         }
     }

     class TragedyCalculator: PerformanceCalculator {
         override func amount() -> Double {
             return 400 + (performance.audience > 30 ? Double(performance.audience - 30) * 1.5 : 0)
         }
     }

     func createCalculator(for performance: Performance) -> PerformanceCalculator {
         switch performance.type {
         case .comedy: return ComedyCalculator(performance: performance)
         case .tragedy: return TragedyCalculator(performance: performance)
         default: return PerformanceCalculator(performance: performance)
         }
     }

     let calculator = createCalculator(for: performance)
     let amount = calculator.amount()
     ```
     Here,

 adding new performance types is easier—just create a new calculator class, and no existing logic needs to be modified.

### Summary
Refactoring aims to create code that is easier to read, understand, and extend. Each of these techniques—from breaking down functions to using polymorphism—helps maintain a clean codebase, which ultimately leads to fewer bugs and easier enhancements. The process is iterative, involving making small changes, testing often, and gradually improving the design.
