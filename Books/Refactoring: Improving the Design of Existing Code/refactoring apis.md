
### 1. **Separate Query from Modifier**
   - **Theory**: The *command-query separation* principle suggests that methods should either *perform an action* (a command) or *return data* (a query) but not both. This separation simplifies reasoning about code, making it easier to test, debug, and maintain.
   - **When to Use**: When you find a method that both alters the state of an object and returns some value, separating them makes it easier to manage side effects.
   - **Example**:
     ```swift
     // Before: A function that both queries and modifies
     func getTotalOutstandingAndSendBill() -> Double {
         let total = customer.invoices.reduce(0) { $0 + $1.amount }
         sendBill()
         return total
     }

     // After: Separate the query and the command
     func totalOutstanding() -> Double {
         return customer.invoices.reduce(0) { $0 + $1.amount }
     }

     func sendBill() {
         emailGateway.send(formatBill(for: customer))
     }

     // Usage
     let total = totalOutstanding()
     sendBill()
     ```
     This separation makes `totalOutstanding()` easy to test since it no longer has side effects.

### 2. **Parameterize Function**
   - **Theory**: *Parameterization* involves creating a single function that can handle a variety of scenarios by passing different values through parameters. It reduces duplication by centralizing similar logic.
   - **When to Use**: When you have multiple functions that only differ slightly in behavior due to different literal values.
   - **Example**:
     ```swift
     // Before: Separate functions with similar logic
     func tenPercentRaise(for person: Person) {
         person.salary *= 1.10
     }

     func fivePercentRaise(for person: Person) {
         person.salary *= 1.05
     }

     // After: Parameterized function
     func raise(for person: Person, by percentage: Double) {
         person.salary *= 1 + percentage
     }

     // Usage
     raise(for: person, by: 0.10) // 10% raise
     raise(for: person, by: 0.05) // 5% raise
     ```
     Now, the logic for raising a salary is centralized, making future adjustments easier.

### 3. **Remove Flag Argument**
   - **Theory**: A *flag argument* is a parameter that acts as a signal to control which path a function will take. This introduces unnecessary complexity because the function's behavior varies significantly based on the flag's value.
   - **When to Use**: When a function uses a boolean or enum parameter to determine which of two (or more) significantly different code paths to execute.
   - **Example**:
     ```swift
     // Before: Function with flag argument
     func bookConcert(for customer: Customer, isPremium: Bool) {
         if isPremium {
             // Logic for premium booking
         } else {
             // Logic for regular booking
         }
     }

     // After: Separate functions
     func bookPremiumConcert(for customer: Customer) {
         // Logic for premium booking
     }

     func bookRegularConcert(for customer: Customer) {
         // Logic for regular booking
     }

     // Usage
     bookPremiumConcert(for: customer)
     bookRegularConcert(for: customer)
     ```
     This approach makes the caller’s intent clear and removes the ambiguity introduced by the boolean flag.

### 4. **Preserve Whole Object**
   - **Theory**: This principle involves passing an entire object to a function instead of individual properties or fields. It respects *data encapsulation* by allowing the function to operate directly on the object rather than exposing its internal details.
   - **When to Use**: When a function requires multiple properties of an object, and you find yourself frequently extracting and passing them.
   - **Example**:
     ```swift
     // Before: Passing individual properties
     func isTemperatureWithinRange(low: Double, high: Double) -> Bool {
         return low >= 20 && high <= 30
     }

     // Usage
     let low = room.temperatureRange.low
     let high = room.temperatureRange.high
     isTemperatureWithinRange(low: low, high: high)

     // After: Passing the whole object
     func isTemperatureWithinRange(_ range: TemperatureRange) -> Bool {
         return range.low >= 20 && range.high <= 30
     }

     // Usage
     isTemperatureWithinRange(room.temperatureRange)
     ```
     This approach is more flexible if `TemperatureRange` gains additional properties in the future.

### 5. **Replace Parameter with Query**
   - **Theory**: This refactoring simplifies function calls by allowing the function to *internally determine* the value of a parameter. It moves the responsibility of determining this value from the caller to the function itself.
   - **When to Use**: When the parameter value can be derived from other data already available in the function’s context.
   - **Example**:
     ```swift
     // Before: Passing a parameter that can be derived
     func availableVacation(for employee: Employee, grade: Int) -> Int {
         // calculate vacation based on grade
     }

     // Usage
     let grade = employee.grade
     availableVacation(for: employee, grade: grade)

     // After: Use a query instead
     func availableVacation(for employee: Employee) -> Int {
         let grade = employee.grade
         // calculate vacation based on grade
     }

     // Usage
     availableVacation(for: employee)
     ```
     This reduces duplication and simplifies the caller’s responsibility.

### 6. **Replace Query with Parameter**
   - **Theory**: The inverse of *Replace Parameter with Query*, this refactoring makes a function more flexible by allowing callers to pass in a value instead of having the function query it.
   - **When to Use**: When you want to decouple a function from a global variable or object reference.
   - **Example**:
     ```swift
     // Before: Function depends on a global object
     func targetTemperature() -> Double {
         return thermostat.selectedTemperature
     }

     // After: Pass the value as a parameter
     func targetTemperature(selectedTemperature: Double) -> Double {
         return selectedTemperature
     }

     // Usage
     let temp = thermostat.selectedTemperature
     targetTemperature(selectedTemperature: temp)
     ```
     This makes the function easier to test, as you can pass in different values.

### 7. **Remove Setting Method**
   - **Theory**: This refactoring aligns with the concept of *immutability*. By removing setter methods, you make objects *read-only* after they’re created, which simplifies reasoning about them.
   - **When to Use**: When a field should not change after object creation.
   - **Example**:
     ```swift
     // Before: Class with a setter
     class Person {
         var id: String
         func setId(_ newId: String) {
             self.id = newId
         }
     }

     // After: Remove the setter
     class Person {
         let id: String
         init(id: String) {
             self.id = id
         }
     }

     // Usage
     let person = Person(id: "12345")
     // person.setId("67890") // This is no longer allowed
     ```
     The `id` is now immutable after the `Person` is created, providing better stability.

### 8. **Replace Constructor with Factory Function**
   - **Theory**: A factory function is a regular function that returns a new instance of a class. It offers more flexibility than a constructor.
   - **When to Use**: When you need more control over object creation, such as returning subclasses.
   - **Example**:
     ```swift
     // Before: Using a constructor directly
     let engineer = Employee(name: "Jane", type: "Engineer")

     // After: Use a factory function
     func createEngineer(name: String) -> Employee {
         return Employee(name: name, type: "Engineer")
     }

     // Usage
     let engineer = createEngineer(name: "Jane")
     ```
     This approach makes the creation process more descriptive.

### 9. **Replace Function with Command**
   - **Theory**: This refactoring involves turning a complex function into an object that encapsulates the function's logic. It aligns with the *Command Pattern* and allows for greater flexibility.
   - **When to Use**: When a function is complex or needs to support additional operations like undo.
   - **Example**:
     ```swift
     // Before: A complex function
     func calculateScore(for candidate: Candidate) -> Int {
         // Complex logic here
     }

     // After: Use a command object
     class ScoreCalculator {
         private let candidate: Candidate

         init(candidate: Candidate) {
             self.candidate = candidate
         }

         func execute() -> Int {
             // Complex logic here
         }
     }

     // Usage
     let calculator = ScoreCalculator(candidate: candidate)
     let score = calculator.execute()
     ```
     Using a command object makes it easier to test parts of the complex logic.

### 10. **Replace Command with Function**
   - **Theory**: The inverse of *Replace Function with Command*, this refactoring simplifies a command object back into a regular function when its complexity is no longer needed.
   - **When to Use**: When the command object has become simple and its extra complexity is unnecessary.
   - **Example**:
     ```swift
     // Before: A command object
     class ChargeCalculator {
         func calculateCharge(for usage: Double

) -> Double {
             return usage * rate
         }
     }

     // After: Simplify to a function
     func calculateCharge(for usage: Double) -> Double {
         return usage * rate
     }

     // Usage
     let charge = calculateCharge(for: 100)
     ```
     This refactoring removes the overhead of an unnecessary class, simplifying the code.

These examples illustrate how each refactoring concept can be applied to improve code structure, making it more maintainable and easier to understand.
