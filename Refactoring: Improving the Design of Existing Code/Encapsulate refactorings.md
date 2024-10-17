Certainly! Here are all 9 encapsulation refactorings, including **before** and **after** examples in Swift, along with explanations for each:

---

### 1. **Encapsulate Record**

**Before**:
```swift
struct Organization {
    var name: String
    var country: String
}

// Usage
let org = Organization(name: "Acme Gooseberries", country: "GB")
print(org.name)
```

**After**:
```swift
class Organization {
    private var name: String
    private var country: String

    init(name: String, country: String) {
        self.name = name
        self.country = country
    }

    func getName() -> String {
        return name
    }

    func setName(_ newName: String) {
        name = newName
    }

    func getCountry() -> String {
        return country
    }

    func setCountry(_ newCountry: String) {
        country = newCountry
    }
}

// Usage
let org = Organization(name: "Acme Gooseberries", country: "GB")
print(org.getName())
```

**Explanation**: The data fields `name` and `country` are now encapsulated within the `Organization` class. This prevents direct access to the fields and allows changes through methods, making it easier to maintain control over how the data is accessed or modified.

---

### 2. **Encapsulate Collection**

**Before**:
```swift
class Person {
    var courses: [String] = []
}

// Usage
let person = Person()
person.courses.append("Math")
```

**After**:
```swift
class Person {
    private var courses: [String] = []

    func addCourse(_ course: String) {
        courses.append(course)
    }

    func removeCourse(_ course: String) {
        courses.removeAll { $0 == course }
    }

    func getCourses() -> [String] {
        return courses
    }
}

// Usage
let person = Person()
person.addCourse("Math")
```

**Explanation**: Direct access to the `courses` list is removed, and instead, methods are provided to add or remove courses. This ensures that modifications to the collection are controlled and centralized, reducing the risk of unintended changes.

---

### 3. **Replace Primitive with Object**

**Before**:
```swift
class Order {
    var priority: String = "normal"
}

// Usage
let order = Order()
if order.priority == "high" {
    // Do something
}
```

**After**:
```swift
class Priority {
    private let value: String
    init(_ value: String) {
        self.value = value
    }

    func isHigherThan(_ other: Priority) -> Bool {
        return value == "high" && other.value == "normal"
    }
}

class Order {
    var priority: Priority = Priority("normal")
}

// Usage
let order = Order()
if order.priority.isHigherThan(Priority("normal")) {
    // Do something
}
```

**Explanation**: The `priority` field is replaced with a `Priority` class, which allows adding behavior specific to priority. This reduces duplication of logic and makes the code more expressive.

---

### 4. **Replace Temp with Query**

**Before**:
```swift
class Order {
    var quantity = 5
    var itemPrice = 20.0

    func totalPrice() -> Double {
        let basePrice = Double(quantity) * itemPrice
        if basePrice > 1000 {
            return basePrice * 0.95
        } else {
            return basePrice * 0.98
        }
    }
}
```

**After**:
```swift
class Order {
    var quantity = 5
    var itemPrice = 20.0

    private var basePrice: Double {
        return Double(quantity) * itemPrice
    }

    func totalPrice() -> Double {
        if basePrice > 1000 {
            return basePrice * 0.95
        } else {
            return basePrice * 0.98
        }
    }
}
```

**Explanation**: The `basePrice` calculation is moved to a computed property, making it reusable and eliminating the temporary variable. This simplifies the function and makes it easier to maintain.

---

### 5. **Extract Class**

**Before**:
```swift
class Person {
    var name: String
    var officeAreaCode: String
    var officeNumber: String

    init(name: String, areaCode: String, number: String) {
        self.name = name
        self.officeAreaCode = areaCode
        self.officeNumber = number
    }
}
```

**After**:
```swift
class TelephoneNumber {
    var areaCode: String
    var number: String

    init(areaCode: String, number: String) {
        self.areaCode = areaCode
        self.number = number
    }

    func fullNumber() -> String {
        return "\(areaCode) \(number)"
    }
}

class Person {
    var name: String
    var telephoneNumber: TelephoneNumber

    init(name: String, areaCode: String, number: String) {
        self.name = name
        self.telephoneNumber = TelephoneNumber(areaCode: areaCode, number: number)
    }
}
```

**Explanation**: The `TelephoneNumber` behavior is extracted into its own class, making `Person` less complex and delegating phone number-related behavior to a more appropriate class.

---

### 6. **Inline Class**

**Before**:
```swift
class TelephoneNumber {
    var areaCode: String
    var number: String
}

class Person {
    var telephoneNumber: TelephoneNumber

    init(areaCode: String, number: String) {
        self.telephoneNumber = TelephoneNumber(areaCode: areaCode, number: number)
    }

    var officeAreaCode: String {
        return telephoneNumber.areaCode
    }

    var officeNumber: String {
        return telephoneNumber.number
    }
}
```

**After**:
```swift
class Person {
    var areaCode: String
    var number: String

    init(areaCode: String, number: String) {
        self.areaCode = areaCode
        self.number = number
    }
}
```

**Explanation**: The `TelephoneNumber` class is inlined back into `Person`, as it no longer justifies its own existence. This simplifies the structure when a separate class is unnecessary.

---

### 7. **Hide Delegate**

**Before**:
```swift
class Department {
    var manager: String

    init(manager: String) {
        self.manager = manager
    }
}

class Person {
    var department: Department

    init(department: Department) {
        self.department = department
    }
}

// Usage
let manager = person.department.manager
```

**After**:
```swift
class Person {
    private var department: Department

    init(department: Department) {
        self.department = department
    }

    func getManager() -> String {
        return department.manager
    }
}

// Usage
let manager = person.getManager()
```

**Explanation**: The `Person` class now hides the `Department` object, providing a method to access the manager. This reduces the dependency on `Department` from the outside.

---

### 8. **Remove Middle Man**

**Before**:
```swift
class Department {
    var manager: String

    init(manager: String) {
        self.manager = manager
    }
}

class Person {
    private var department: Department

    init(department: Department) {
        self.department = department
    }

    func getManager() -> String {
        return department.manager
    }
}

// Usage
let manager = person.getManager()
```

**After**:
```swift
class Department {
    var manager: String

    init(manager: String) {
        self.manager = manager
    }
}

class Person {
    var department: Department

    init(department: Department) {
        self.department = department
    }
}

// Usage
let manager = person.department.manager
```

**Explanation**: The method `getManager()` in `Person` is removed, allowing the client to access the `manager` directly through the `department`. This simplifies `Person` when forwarding methods are no longer needed.

---

### 9. **Substitute Algorithm**

**Before**:
```swift
func foundPerson(_ people: [String]) -> String? {
    for person in people {
        if person == "Don" {
            return "Don"
        }
        if person == "John" {
            return "John"
        }
        if person == "Kent" {
            return "Kent"
        }
    }
    return nil
}
```

**After**:
```swift
func foundPerson(_ people: [String]) -> String? {
    let candidates = ["Don", "John", "Kent"]
    return people.first { candidates.contains($0) }
}
```

**Explanation**: The `for` loop is replaced with a simpler algorithm using `first` and `contains`. This makes the code more concise and leverages Swift's built-in methods for better readability.

---

These examples cover a variety of encapsulation refactorings, showing how they can improve code readability, reduce complexity, and maintain better control over data and behavior in a Swift-based codebase.
