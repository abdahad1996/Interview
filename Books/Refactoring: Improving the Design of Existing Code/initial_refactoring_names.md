### A First Set of Refactorings Explained with Examples

This set of refactorings is fundamental when cleaning up or improving code. Let's go through each concept in detail, explaining what it means, when it's useful, and providing Swift examples for clarity.

---

### 1. **Extract Function**

**Definition**: Extracting a function means taking a piece of code from a larger function and moving it into its own, named function. This improves readability, reusability, and simplifies the original function.

**Motivation**: If a section of code is complex or has a specific role, extracting it into a function with a descriptive name makes it easier to understand. For example, a `calculateOutstanding` function is easier to read than having the calculations inline.

**Swift Example**:

**Before**:
```swift
func printOwing(invoice: Invoice) {
    print("**** Customer Owes ****")
    var outstanding = 0.0
    for order in invoice.orders {
        outstanding += order.amount
    }
    print("Name: \(invoice.customer)")
    print("Amount: \(outstanding)")
}
```

**After**:
```swift
func printOwing(invoice: Invoice) {
    printBanner()
    let outstanding = calculateOutstanding(for: invoice)
    printDetails(invoice: invoice, outstanding: outstanding)
}

func printBanner() {
    print("**** Customer Owes ****")
}

func calculateOutstanding(for invoice: Invoice) -> Double {
    return invoice.orders.reduce(0) { $0 + $1.amount }
}

func printDetails(invoice: Invoice, outstanding: Double) {
    print("Name: \(invoice.customer)")
    print("Amount: \(outstanding)")
}
```

---

### 2. **Inline Function**

**Definition**: Inlining a function means taking the body of a function and replacing its calls with that body directly. This is the opposite of Extract Function.

**Motivation**: Use this when a function's implementation is simple, or the function name doesn't add clarity. This reduces unnecessary indirection in the code.

**Swift Example**:

**Before**:
```swift
func calculateTax(for amount: Double) -> Double {
    return amount * 0.1
}

let tax = calculateTax(for: 100)
```

**After**:
```swift
let tax = 100 * 0.1
```

---

### 3. **Extract Variable**

**Definition**: Extracting a variable involves assigning a complex or repeated expression to a named variable.

**Motivation**: It helps by giving a name to an expression, making the code easier to understand and maintain.

**Swift Example**:

**Before**:
```swift
let finalPrice = price * (1 - discount) + shippingCost
```

**After**:
```swift
let discountedPrice = price * (1 - discount)
let finalPrice = discountedPrice + shippingCost
```

---

### 4. **Inline Variable**

**Definition**: Replacing a variable with its actual expression, removing the variable.

**Motivation**: Use this when a variable’s name doesn’t provide additional clarity over its expression, or when it’s only used once.

**Swift Example**:

**Before**:
```swift
let discountedPrice = price * (1 - discount)
let finalPrice = discountedPrice + shippingCost
```

**After**:
```swift
let finalPrice = price * (1 - discount) + shippingCost
```

---

### 5. **Change Function Declaration**

**Definition**: Modifying a function’s name or parameters to better reflect its role.

**Motivation**: This is useful when the function name or its parameters no longer match what the function does, or when you need to adjust how it interacts with other parts of the code.

**Swift Example**:

**Before**:
```swift
func calcOutstanding() -> Double { ... }
```

**After**:
```swift
func calculateTotalOutstanding() -> Double { ... }
```

---

### 6. **Rename Variable**

**Definition**: Changing a variable’s name to make its purpose clearer.

**Motivation**: Code readability can be significantly improved with better-named variables.

**Swift Example**:

**Before**:
```swift
let a = price * 0.1
```

**After**:
```swift
let tax = price * 0.1
```

---

### 7. **Introduce Parameter Object**

**Definition**: Replacing multiple parameters that logically belong together with a single parameter object.

**Motivation**: It simplifies method signatures and makes the relationship between parameters clearer.

**Swift Example**:

**Before**:
```swift
func calculatePrice(startDate: Date, endDate: Date, baseRate: Double) -> Double { ... }
```

**After**:
```swift
struct DateRange {
    let start: Date
    let end: Date
}

func calculatePrice(dateRange: DateRange, baseRate: Double) -> Double { ... }
```

---

### 8. **Combine Functions into Class**

**Definition**: Grouping functions and the data they operate on into a class.

**Motivation**: This is useful when functions share a lot of data and can be logically grouped together to improve organization.

**Swift Example**:

**Before**:
```swift
func calculateDiscount(price: Double) -> Double { ... }
func applyTax(price: Double) -> Double { ... }
```

**After**:
```swift
class PriceCalculator {
    func calculateDiscount(for price: Double) -> Double { ... }
    func applyTax(to price: Double) -> Double { ... }
}
```

---

### 9. **Combine Functions into Transform**

**Definition**: Using a transformation function to calculate and store derived data in a single place.

**Motivation**: This keeps calculations centralized, reducing duplication and inconsistency.

**Swift Example**:

**Before**:
```swift
func baseCharge(quantity: Int) -> Double { ... }
func totalCharge(quantity: Int) -> Double { ... }
```

**After**:
```swift
struct EnrichedReading {
    let quantity: Int
    let baseCharge: Double
    let totalCharge: Double
}

func enrichReading(quantity: Int) -> EnrichedReading {
    let baseCharge = calculateBaseCharge(quantity)
    let totalCharge = baseCharge + calculateTax(baseCharge)
    return EnrichedReading(quantity: quantity, baseCharge: baseCharge, totalCharge: totalCharge)
}
```

---

### 10. **Split Phase**

**Definition**: Breaking a process into distinct stages where each stage deals with a different part of the problem.

**Motivation**: This separation simplifies each phase and makes it easier to modify one part without affecting the other.

**Swift Example**:

**Before**:
```swift
func processOrder(order: Order) -> Double {
    let orderData = parseOrder(order)
    let price = calculatePrice(orderData)
    return price
}
```

**After**:
```swift
func parseOrder(order: Order) -> OrderData { ... }
func calculatePrice(orderData: OrderData) -> Double { ... }

func processOrder(order: Order) -> Double {
    let orderData = parseOrder(order)
    return calculatePrice(orderData)
}
```

---

These refactorings are the building blocks for maintaining a clean, understandable, and maintainable codebase. They help by keeping methods focused, improving naming clarity, and organizing code into logical structures.
