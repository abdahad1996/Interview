Let’s break down the **Organizing Data** refactorings, including each of the methods mentioned: **Split Variable**, **Rename Field**, **Replace Derived Variable with Query**, and **Change Reference to Value**, with explanations, before and after examples, all in Swift.

### 1. **Split Variable**

**Motivation**:
A variable that is used for more than one purpose within a method can cause confusion and bugs. If a variable has multiple responsibilities, it should be split into separate variables, each representing a single responsibility. This helps clarify the code and makes it more maintainable.

**Before**:
```swift
func calculateAreaAndPerimeter(height: Double, width: Double) {
    var temp = 2 * (height + width)
    print("Perimeter: \(temp)")
    temp = height * width
    print("Area: \(temp)")
}
```

**After**:
```swift
func calculateAreaAndPerimeter(height: Double, width: Double) {
    let perimeter = 2 * (height + width)
    print("Perimeter: \(perimeter)")
    let area = height * width
    print("Area: \(area)")
}
```

**Explanation**:
In the original code, `temp` is used to calculate both the perimeter and the area, which can cause confusion. By splitting `temp` into two distinct variables, `perimeter` and `area`, the code becomes more readable and each variable has a clear purpose.

---

### 2. **Rename Field**

**Motivation**:
Field names are essential for clarity, especially when they are widely used across the program. Renaming fields can make the data structures easier to understand and reduce confusion.

**Before**:
```swift
struct Organization {
    var name: String
    var country: String
}

let organization = Organization(name: "Acme", country: "US")
```

**After**:
```swift
struct Organization {
    var title: String
    var country: String
}

let organization = Organization(title: "Acme", country: "US")
```

**Explanation**:
The field `name` is renamed to `title` because it better represents the concept within the context of the `Organization`. Renaming the field makes the intention clearer to other developers reading the code.

---

### 3. **Replace Derived Variable with Query**

**Motivation**:
Derived variables hold values that can be computed from other data. Using derived variables can lead to inconsistencies if they are not updated correctly. Instead, it's better to use a computed property or a method to calculate the value whenever it is needed.

**Before**:
```swift
class Order {
    private var _discountedTotal: Double = 0.0
    var discount: Double = 0.0

    var baseTotal: Double = 100.0

    func applyDiscount() {
        _discountedTotal = baseTotal - discount
    }

    func getDiscountedTotal() -> Double {
        return _discountedTotal
    }
}
```

**After**:
```swift
class Order {
    var discount: Double = 0.0
    var baseTotal: Double = 100.0

    var discountedTotal: Double {
        return baseTotal - discount
    }
}
```

**Explanation**:
Instead of storing `_discountedTotal` as a variable that needs to be updated whenever `discount` changes, we replace it with a computed property `discountedTotal`. This ensures that `discountedTotal` is always accurate and reduces the chance of inconsistency.

---

### 4. **Change Reference to Value**

**Motivation**:
Treating an object as a value means that changes to the object do not affect other instances that share the same reference. Value objects are particularly useful when immutability is desired, making the code easier to reason about.

**Before**:
```swift
class Person {
    var telephoneNumber: TelephoneNumber

    init(areaCode: String, number: String) {
        self.telephoneNumber = TelephoneNumber(areaCode: areaCode, number: number)
    }

    var officeAreaCode: String {
        get { return telephoneNumber.areaCode }
        set { telephoneNumber.areaCode = newValue }
    }

    var officeNumber: String {
        get { return telephoneNumber.number }
        set { telephoneNumber.number = newValue }
    }
}

class TelephoneNumber {
    var areaCode: String
    var number: String

    init(areaCode: String, number: String) {
        self.areaCode = areaCode
        self.number = number
    }
}
```

**After**:
```swift
class Person {
    private var telephoneNumber: TelephoneNumber

    init(areaCode: String, number: String) {
        self.telephoneNumber = TelephoneNumber(areaCode: areaCode, number: number)
    }

    var officeAreaCode: String {
        get { return telephoneNumber.areaCode }
        set { telephoneNumber = TelephoneNumber(areaCode: newValue, number: telephoneNumber.number) }
    }

    var officeNumber: String {
        get { return telephoneNumber.number }
        set { telephoneNumber = TelephoneNumber(areaCode: telephoneNumber.areaCode, number: newValue) }
    }
}

struct TelephoneNumber {
    let areaCode: String
    let number: String
}
```

**Explanation**:
In the before example, `TelephoneNumber` is a reference type (`class`), meaning changes to `areaCode` or `number` could affect other instances that share the same reference. By making `TelephoneNumber` a `struct` (value type) and removing setters, we make it immutable. Now, whenever the `areaCode` or `number` changes, a new instance is created, ensuring immutability and making the `Person` class easier to reason about.

---
### 5. **Change Value to Reference**
Motivation: Sometimes, an object should be shared among multiple instances to ensure consistency. For example, if multiple objects are referencing the same "Company" object, a change in the company's information (like an address) should be reflected everywhere that object is referenced. This is typical for objects that are large or have identity and need to be shared rather than copied. Changing a value object to a reference can help achieve this shared behavior.

Before:

swift
Copy code
struct Address {
    var street: String
    var city: String
    var postalCode: String
}

class Customer {
    var name: String
    var address: Address

    init(name: String, address: Address) {
        self.name = name
        self.address = address
    }
}

let address1 = Address(street: "123 Main St", city: "Springfield", postalCode: "12345")
let customer1 = Customer(name: "Alice", address: address1)
let customer2 = Customer(name: "Bob", address: address1)

// Changing the address of customer1 doesn't affect customer2, as they have copies of the address.
customer1.address.street = "456 Elm St"
print(customer2.address.street) // Outputs "123 Main St"
After:

swift
Copy code
class Address {
    var street: String
    var city: String
    var postalCode: String

    init(street: String, city: String, postalCode: String) {
        self.street = street
        self.city = city
        self.postalCode = postalCode
    }
}

class Customer {
    var name: String
    var address: Address

    init(name: String, address: Address) {
        self.name = name
        self.address = address
    }
}

let sharedAddress = Address(street: "123 Main St", city: "Springfield", postalCode: "12345")
let customer1 = Customer(name: "Alice", address: sharedAddress)
let customer2 = Customer(name: "Bob", address: sharedAddress)

// Changing the address of customer1 now affects customer2 as well, since they share the same reference.
customer1.address.street = "456 Elm St"
print(customer2.address.street) // Outputs "456 Elm St"
Explanation: In the "before" example, Address is a struct, making it a value type. This means each Customer object has its own copy of the Address. Changing one does not affect the other.

In the "after" example, Address is converted to a class, making it a reference type. Now, both customer1 and customer2 share the same Address instance. When the street property is updated through customer1, the change is reflected when accessing customer2's address because they both reference the same Address object.

This change makes sense when an object's identity is important, and you want changes in one place to be reflected across all references. It is especially useful for shared data that should be kept consistent across different parts of a system.

By understanding and applying these refactorings, you can make your codebase cleaner, more maintainable, and easier to understand. Each of these techniques helps to organize data and improve code clarity, making your software easier to develop and modify in the long term.
