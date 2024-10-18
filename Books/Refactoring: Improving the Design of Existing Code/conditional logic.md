Sure! Let's break down each of these refactoring concepts and illustrate them with Swift examples for better understanding:

### 1. **Decompose Conditional**
**Motivation**: Simplifies complex conditional logic by breaking it down into functions. This makes the code easier to understand and maintain.

**Swift Example:**

```swift
// Original Code
func calculateCharge(for date: Date) -> Double {
    if !date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd) {
        return quantity * plan.summerRate
    } else {
        return quantity * plan.regularRate + plan.regularServiceCharge
    }
}

// Refactored Code
func calculateCharge(for date: Date) -> Double {
    if isSummer(date) {
        return summerCharge()
    } else {
        return regularCharge()
    }
}

func isSummer(_ date: Date) -> Bool {
    return !date.isBefore(plan.summerStart) && !date.isAfter(plan.summerEnd)
}

func summerCharge() -> Double {
    return quantity * plan.summerRate
}

func regularCharge() -> Double {
    return quantity * plan.regularRate + plan.regularServiceCharge
}
```
By decomposing the condition and the logic, each piece of code has a clearer purpose.

---

### 2. **Consolidate Conditional Expression**
**Motivation**: When multiple conditionals lead to the same result, you can combine them into a single conditional expression.

**Swift Example:**

```swift
// Original Code
func disabilityAmount(employee: Employee) -> Double {
    if employee.seniority < 2 {
        return 0
    }
    if employee.monthsDisabled > 12 {
        return 0
    }
    if employee.isPartTime {
        return 0
    }
    // Calculate actual disability amount
    return calculateDisability()
}

// Refactored Code
func disabilityAmount(employee: Employee) -> Double {
    if isNotEligibleForDisability(employee) {
        return 0
    }
    // Calculate actual disability amount
    return calculateDisability()
}

func isNotEligibleForDisability(_ employee: Employee) -> Bool {
    return employee.seniority < 2 || employee.monthsDisabled > 12 || employee.isPartTime
}
```
Combining conditions improves readability and highlights that the conditions represent a single idea.

---

### 3. **Replace Nested Conditional with Guard Clauses**
**Motivation**: Use guard statements to handle edge cases and exit early, making the main logic more readable.

**Swift Example:**

```swift
// Original Code
func getPayAmount() -> Double {
    var result: Double
    if isDead {
        result = deadAmount()
    } else {
        if isSeparated {
            result = separatedAmount()
        } else {
            if isRetired {
                result = retiredAmount()
            } else {
                result = normalPayAmount()
            }
        }
    }
    return result
}

// Refactored Code with Guard Clauses
func getPayAmount() -> Double {
    if isDead { return deadAmount() }
    if isSeparated { return separatedAmount() }
    if isRetired { return retiredAmount() }
    return normalPayAmount()
}
```
By using guard clauses or early returns, the code becomes easier to follow.

---

### 4. **Replace Conditional with Polymorphism**
**Motivation**: Replace complex conditionals with polymorphic behavior, leveraging subclassing or protocols to handle variations.

**Swift Example:**

```swift
// Original Code
func birdSound(bird: Bird) -> String {
    switch bird.type {
    case .sparrow:
        return "Chirp"
    case .parrot:
        return bird.canSpeak ? "Hello!" : "Squawk"
    case .owl:
        return "Hoot"
    default:
        return "Unknown"
    }
}

// Refactored Code with Polymorphism
protocol Bird {
    func sound() -> String
}

class Sparrow: Bird {
    func sound() -> String {
        return "Chirp"
    }
}

class Parrot: Bird {
    var canSpeak: Bool
    init(canSpeak: Bool) {
        self.canSpeak = canSpeak
    }

    func sound() -> String {
        return canSpeak ? "Hello!" : "Squawk"
    }
}

class Owl: Bird {
    func sound() -> String {
        return "Hoot"
    }
}

// Usage
let birds: [Bird] = [Sparrow(), Parrot(canSpeak: true), Owl()]
for bird in birds {
    print(bird.sound())
}
```
Polymorphism allows each bird to handle its sound logic, making the code more scalable and easier to extend.

---

### 5. **Introduce Special Case (Null Object Pattern)**
**Motivation**: Introduce a special-case object to handle scenarios that typically involve checking for `nil` or other special cases.

**Swift Example:**

```swift
// Original Code
func getCustomerName(customer: Customer?) -> String {
    if let customer = customer {
        return customer.name
    } else {
        return "Guest"
    }
}

// Refactored Code with Null Object
protocol Customer {
    var name: String { get }
}

class RealCustomer: Customer {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class NullCustomer: Customer {
    var name: String { return "Guest" }
}

func getCustomerName(customer: Customer) -> String {
    return customer.name
}

// Usage
let realCustomer = RealCustomer(name: "Alice")
let nullCustomer = NullCustomer()
print(getCustomerName(customer: realCustomer)) // Outputs: Alice
print(getCustomerName(customer: nullCustomer)) // Outputs: Guest
```
Using the `NullCustomer`, we remove the need to check for `nil`, simplifying the logic.

---

### 6. **Introduce Assertion**
**Motivation**: Add assertions to make assumptions explicit, improving code reliability and clarity.

**Swift Example:**

```swift
// Original Code
func applyDiscount(_ discount: Double) -> Double {
    return price - (price * discount)
}

// Refactored Code with Assertion
func applyDiscount(_ discount: Double) -> Double {
    assert(discount >= 0 && discount <= 1, "Discount should be between 0 and 1")
    return price - (price * discount)
}
```
With the assertion, we explicitly state that the discount must be between 0 and 1, catching errors early in development.

---

Each of these refactoring techniques helps in writing cleaner, more maintainable code by breaking down complex logic, reducing duplication, and leveraging object-oriented principles.
