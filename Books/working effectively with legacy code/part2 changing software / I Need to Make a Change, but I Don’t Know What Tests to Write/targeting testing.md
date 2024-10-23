Let's break down the theoretical concepts behind targeted testing, the importance of this approach when refactoring code, and how to ensure code quality through comprehensive testing. The examples will be in Swift to keep things consistent with your request.

### Targeted Testing: What Is It?

**Targeted testing** focuses on specific sections of code that are most likely to change or that need verification to ensure correct behavior. The idea is to:
- Understand the existing code behavior.
- Write tests that validate this behavior.
- Refactor the code with confidence that the tests will catch any unexpected changes.

This method is particularly important when working with legacy code or when you need to refactor a complex method into smaller, more manageable pieces. It ensures that:
1. **Behavior remains consistent** after refactoring.
2. **Critical branches** in the code are covered by tests.
3. **Type changes or conversions** do not introduce unexpected bugs.

### Why Is Targeted Testing Important During Refactoring?

When refactoring code, especially if it involves moving logic into new methods or classes, there is a risk of:
- **Breaking existing functionality**.
- **Misinterpreting the original logic**, resulting in errors.
- **Introducing issues** related to data types or the interaction between new and old code.

By writing targeted tests, you minimize these risks. The tests act as a safety net, allowing you to verify that the behavior has not unintentionally changed. 

### The Process of Targeted Testing

1. **Characterize the Code with Tests**:
   - Before making changes, write tests to understand how the existing code behaves.
   - Focus on the critical branches or paths that the code can take.
   - This helps to establish a baseline for the functionality you need to preserve during the refactoring.

2. **Refactor with Confidence**:
   - Extract methods or move logic into new classes.
   - Use the tests to verify that the functionality remains the same after refactoring.
   - Any changes to behavior should be intentional, and your tests will help highlight unintentional changes.

3. **Handle Type Changes Carefully**:
   - When refactoring, sometimes you may change how data is stored or passed (e.g., from `Double` to `Int`).
   - Write tests that confirm the conversions are handled correctly to avoid issues like truncation of decimals.

### Refactoring Scenario

Imagine a class `FuelShare` that handles fuel cost calculations based on a lease type. The method `addReading` determines how to adjust the cost based on the number of gallons and the lease type. For example, it checks if the lease is monthly and adjusts the cost accordingly.

We want to refactor this method by moving the logic of calculating the fuel value to a new method in a different class, `ZonedHawthorneLease`.

Here's the **original method**:

```swift
class FuelShare {
    var cost: Double = 0
    var corpBase: Double = 12.0
    var lease: ZonedHawthorneLease

    init(lease: ZonedHawthorneLease) {
        self.lease = lease
    }

    func addReading(gallons: Int, readingDate: Date) {
        if lease.isMonthly {
            if gallons < Lease.CORP_MIN {
                cost += corpBase
            } else {
                cost += 1.2 * priceForGallons(gallons: gallons)
            }
        }
        lease.postReading(readingDate: readingDate, gallons: gallons)
    }

    private func priceForGallons(gallons: Int) -> Double {
        return Double(gallons) * 1.5 // Example price calculation
    }
}
```

### Refactoring Plan

We want to move the `if`-statement logic into a new method in `ZonedHawthorneLease`. After refactoring, the code might look like this:

```swift
class FuelShare {
    var cost: Double = 0
    var lease: ZonedHawthorneLease

    init(lease: ZonedHawthorneLease) {
        self.lease = lease
    }

    func addReading(gallons: Int, readingDate: Date) {
        cost += lease.computeValue(gallons: gallons, totalPrice: priceForGallons(gallons: gallons))
        lease.postReading(readingDate: readingDate, gallons: gallons)
    }

    private func priceForGallons(gallons: Int) -> Double {
        return Double(gallons) * 1.5
    }
}

class ZonedHawthorneLease {
    var isMonthly: Bool
    var corpBase: Double = 12.0

    init(isMonthly: Bool) {
        self.isMonthly = isMonthly
    }

    func computeValue(gallons: Int, totalPrice: Double) -> Double {
        var cost: Double = 0
        if isMonthly {
            if gallons < Lease.CORP_MIN {
                cost += corpBase
            } else {
                cost += 1.2 * totalPrice
            }
        }
        return cost
    }

    func postReading(readingDate: Date, gallons: Int) {
        // Logic for posting the reading
    }
}
```

### Writing Tests to Support Refactoring

#### 1. Characterization Tests

Characterization tests help to capture the current behavior before you make changes. These tests ensure that the logic you intend to refactor is covered.

Example:
```swift
func testValueForGallonsBelowCorpMin() {
    let lease = ZonedHawthorneLease(isMonthly: true)
    let share = FuelShare(lease: lease)
    share.addReading(gallons: 5, readingDate: Date()) // 5 is below Lease.CORP_MIN
    XCTAssertEqual(share.cost, 12.0) // Ensures the cost was calculated using corpBase
}
```

This test verifies that if the gallons are below `Lease.CORP_MIN`, the `corpBase` value is used, maintaining the behavior.

#### 2. Edge Case Tests

These tests validate behavior at the edges of conditions, like when the number of gallons is exactly equal to `Lease.CORP_MIN` or slightly above.

Example:
```swift
func testValueForGallonsEqualToCorpMin() {
    let lease = ZonedHawthorneLease(isMonthly: true)
    let share = FuelShare(lease: lease)
    share.addReading(gallons: Lease.CORP_MIN, readingDate: Date())
    XCTAssertEqual(share.cost, 12.0) // This should trigger the corpBase addition.
}

func testValueForGallonsAboveCorpMin() {
    let lease = ZonedHawthorneLease(isMonthly: true)
    let share = FuelShare(lease: lease)
    share.addReading(gallons: Lease.CORP_MIN + 1, readingDate: Date())
    XCTAssertEqual(share.cost, 16.2) // Assuming priceForGallons returns 10.0
}
```

#### 3. Tests for Refactoring Changes

After refactoring, we ensure that the `computeValue` method behaves correctly. This involves writing tests that confirm it calculates the cost properly and is used correctly in `addReading`.

Example:
```swift
func testComputeValueInLease() {
    let lease = ZonedHawthorneLease(isMonthly: true)
    let computedCost = lease.computeValue(gallons: Lease.CORP_MIN + 1, totalPrice: 10)
    XCTAssertEqual(computedCost, 12.0 * 1.2) // Confirms the multiplication logic
}
```

#### 4. Type Handling and Conversions

In some cases, the refactoring may involve changing data types (e.g., using `Double` instead of `Int` for monetary values). If there’s a risk of losing precision, you should test that behavior with input values that could trigger such issues.

Example:
```swift
func testDoubleToIntConversionIssues() {
    let lease = ZonedHawthorneLease(isMonthly: true)
    let share = FuelShare(lease: lease)
    share.addReading(gallons: 1, readingDate: Date()) // Small number of gallons
    XCTAssertEqual(share.cost, 12.0) // Ensure there is no truncation error.
}
```

### Summary

**Targeted testing** is a strategic approach to ensure that changes made during refactoring maintain the original functionality of your code. It involves:
- Writing tests before refactoring to understand the current behavior.
- Testing both "happy path" (expected) and edge cases.
- Ensuring that any data type changes do not introduce unintended bugs.
- Using tests as a safety net to validate that the refactoring maintains the intended logic and connections.

By following these steps and using tests as a safety net, you can refactor with confidence, knowing that any deviations from expected behavior will be caught early in the process.
