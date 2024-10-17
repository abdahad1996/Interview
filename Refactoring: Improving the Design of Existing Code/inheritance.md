
### 1. **Pull Up Method**

   - **Theory**: *Pull Up Method* involves moving a method from subclasses to a common superclass when those methods have identical or nearly identical logic. This helps to reduce duplication and centralizes shared behavior.
   - **Example**:
     ```swift
     class Employee {
         func annualCost() -> Double {
             fatalError("Must be overridden")
         }
     }

     class Salesman: Employee {
         override func annualCost() -> Double {
             return monthlyCost * 12
         }
     }

     class Engineer: Employee {
         override func annualCost() -> Double {
             return monthlyCost * 12
         }
     }

     // Pull up the method to the superclass
     class Employee {
         func annualCost() -> Double {
             return monthlyCost * 12
         }
     }

     class Salesman: Employee {}
     class Engineer: Employee {}
     ```
     By pulling up the `annualCost()` method, both subclasses now use the same implementation from the `Employee` class.

### 2. **Pull Up Field**

   - **Theory**: *Pull Up Field* is used when a field or property is duplicated across multiple subclasses. Moving it to a common superclass reduces redundancy and keeps the data centralized.
   - **Example**:
     ```swift
     class Salesman: Employee {
         var name: String
     }

     class Engineer: Employee {
         var name: String
     }

     // Pull up the field to the superclass
     class Employee {
         var name: String
     }

     class Salesman: Employee {}
     class Engineer: Employee {}
     ```
     The `name` property now resides in `Employee`, eliminating the duplication in `Salesman` and `Engineer`.

### 3. **Pull Up Constructor Body**

   - **Theory**: *Pull Up Constructor Body* involves moving shared constructor code from subclasses to the constructor of a common superclass. This simplifies constructors and centralizes shared initialization logic.
   - **Example**:
     ```swift
     class Employee {
         var name: String

         init(name: String) {
             self.name = name
         }
     }

     class Salesman: Employee {
         var monthlyCost: Double

         init(name: String, monthlyCost: Double) {
             self.monthlyCost = monthlyCost
             super.init(name: name)
         }
     }

     class Engineer: Employee {
         var monthlyCost: Double

         init(name: String, monthlyCost: Double) {
             self.monthlyCost = monthlyCost
             super.init(name: name)
         }
     }

     // Pull up the common constructor body to the superclass
     class Employee {
         var name: String
         var monthlyCost: Double

         init(name: String, monthlyCost: Double) {
             self.name = name
             self.monthlyCost = monthlyCost
         }
     }

     class Salesman: Employee {}
     class Engineer: Employee {}
     ```
     Here, the `monthlyCost` property and its initialization are moved up to `Employee`, simplifying the constructors of the subclasses.

### 4. **Push Down Method**

   - **Theory**: *Push Down Method* is the inverse of Pull Up Method. It moves a method from a superclass into one or more of its subclasses if the method is only relevant to those subclasses.
   - **Example**:
     ```swift
     class Employee {
         func quota() -> Double {
             fatalError("Must be implemented by subclass")
         }
     }

     class Salesman: Employee {
         override func quota() -> Double {
             return 100.0
         }
     }

     class Engineer: Employee {}

     // Push down the `quota()` method to the subclass
     class Employee {}

     class Salesman: Employee {
         func quota() -> Double {
             return 100.0
         }
     }

     class Engineer: Employee {}
     ```
     The `quota()` method was only relevant for `Salesman`, so it was moved down from `Employee`.

### 5. **Push Down Field**

   - **Theory**: *Push Down Field* is used when a field in a superclass is only relevant to some of its subclasses. It involves moving that field to those subclasses.
   - **Example**:
     ```swift
     class Employee {
         var salesQuota: Double = 0.0
     }

     class Salesman: Employee {}
     class Engineer: Employee {}

     // Push down the `salesQuota` field to `Salesman`
     class Employee {}

     class Salesman: Employee {
         var salesQuota: Double = 0.0
     }

     class Engineer: Employee {}
     ```
     The `salesQuota` field is only relevant to `Salesman`, so it is moved down to that subclass.

### 6. **Replace Type Code with Subclasses**

   - **Theory**: This refactoring replaces a type code (often a string or enum) with subclasses. It allows using polymorphism to simplify conditional logic based on that type.
   - **Example**:
     ```swift
     class Employee {
         let type: String

         init(type: String) {
             self.type = type
         }

         func bonus() -> Double {
             switch type {
             case "engineer":
                 return 1000
             case "salesman":
                 return 1500
             default:
                 return 0
             }
         }
     }

     // Replace Type Code with Subclasses
     class Employee {
         func bonus() -> Double {
             fatalError("Must be implemented by subclass")
         }
     }

     class Engineer: Employee {
         override func bonus() -> Double {
             return 1000
         }
     }

     class Salesman: Employee {
         override func bonus() -> Double {
             return 1500
         }
     }
     ```
     Now, `Engineer` and `Salesman` classes use polymorphism for the `bonus()` calculation instead of type checking.

### 7. **Replace Subclass with Delegate**

   - **Theory**: This refactoring moves behavior from a subclass into a delegate object. It helps when you want to change from using inheritance to composition.
   - **Example**:
     ```swift
     class Booking {
         var show: Show

         init(show: Show) {
             self.show = show
         }

         func hasTalkback() -> Bool {
             return show.isTalkbackAvailable && !isPeakDay()
         }
     }

     class PremiumBooking: Booking {
         override func hasTalkback() -> Bool {
             return show.isTalkbackAvailable
         }
     }

     // Replace Subclass with Delegate
     class Booking {
         var show: Show
         private var premiumDelegate: PremiumDelegate?

         init(show: Show, isPremium: Bool) {
             self.show = show
             if isPremium {
                 self.premiumDelegate = PremiumDelegate(show: show)
             }
         }

         func hasTalkback() -> Bool {
             return premiumDelegate?.hasTalkback() ?? (show.isTalkbackAvailable && !isPeakDay())
         }
     }

     class PremiumDelegate {
         var show: Show

         init(show: Show) {
             self.show = show
         }

         func hasTalkback() -> Bool {
             return show.isTalkbackAvailable
         }
     }
     ```
     The `PremiumBooking` subclass is replaced by a `PremiumDelegate` that `Booking` delegates to when needed.

### 8. **Extract Superclass**

   - **Theory**: *Extract Superclass* involves creating a new superclass to hold common behavior and properties from two or more existing classes. It simplifies code by centralizing shared elements.
   - **Example**:
     ```swift
     class Department {
         var name: String
         var budget: Double

         init(name: String, budget: Double) {
             self.name = name
             self.budget = budget
         }

         func annualCost() -> Double {
             return budget * 12
         }
     }

     class Employee {
         var name: String
         var salary: Double

         init(name: String, salary: Double) {
             self.name = name
             self.salary = salary
         }

         func annualCost() -> Double {
             return salary * 12
         }
     }

     // Extract common elements into a superclass
     class Party {
         var name: String

         init(name: String) {
             self.name = name
         }

         func annualCost() -> Double {
             fatalError("Must be implemented by subclass")
         }
     }

     class Department: Party {
         var budget: Double

         init(name: String, budget: Double) {
             self.budget = budget
             super.init(name: name)
         }

         override func annualCost() -> Double {
             return budget * 12
         }
     }

     class Employee: Party {
         var salary: Double

         init(name: String, salary: Double) {
             self.salary = salary
             super.init(name: name)
         }

         override func annualCost() -> Double {
             return salary * 12
         }
     }
     ```
     A new `Party` superclass is created to share the `name` property and `annualCost()` method signature.

These refactoring techniques help manage and improve code that uses inheritance, making it cleaner, more maintainable, and easier to extend.
