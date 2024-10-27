Link substitution is a technique often used for testing, particularly in situations where dependency injection isn’t available or practical. In Swift, this can be achieved with protocols, allowing us to substitute implementations dynamically at runtime. Here’s how you can implement a form of link substitution in Swift by using protocols and dependency injection to create alternative definitions for testing.

### Step-by-Step Explanation with Example

Let’s say you have a bank account system with a `BankAccount` class that handles deposits and withdrawals. We’ll simulate link substitution by creating a protocol that the class conforms to, allowing us to swap out implementations easily.

#### 1. Define the Protocol
We start by defining a protocol with the methods we want to substitute.

```swift
protocol AccountActions {
    func deposit(amount: Int)
    func withdraw(amount: Int)
}
```

#### 2. Create the Production Class
Now, create the actual implementation of `BankAccount` that we’ll use in production.

```swift
class BankAccount: AccountActions {
    private var balance: Int = 0

    func deposit(amount: Int) {
        balance += amount
        print("Deposited \(amount). New balance is \(balance)")
    }

    func withdraw(amount: Int) {
        if amount <= balance {
            balance -= amount
            print("Withdrew \(amount). New balance is \(balance)")
        } else {
            print("Insufficient funds")
        }
    }
}
```

This `BankAccount` class conforms to the `AccountActions` protocol, and you could use this in your main application. 

#### 3. Create a Mock Class for Testing
Next, create a mock version of the class to use in tests. This mock class will keep track of calls to `deposit` and `withdraw` for later verification.

```swift
class MockBankAccount: AccountActions {
    var depositCalls: [(amount: Int)] = []
    var withdrawCalls: [(amount: Int)] = []

    func deposit(amount: Int) {
        depositCalls.append((amount: amount))
    }

    func withdraw(amount: Int) {
        withdrawCalls.append((amount: amount))
    }
}
```

In this mock, we aren’t modifying any real balance but are instead recording the arguments passed to each method call. This allows you to inspect whether the functions were called correctly in your tests.

#### 4. Use Dependency Injection for Link Substitution
To achieve substitution, we can inject an instance of either `BankAccount` or `MockBankAccount` into a class that uses it.

```swift
class BankService {
    private var account: AccountActions

    init(account: AccountActions) {
        self.account = account
    }

    func makeDeposit(amount: Int) {
        account.deposit(amount: amount)
    }

    func makeWithdrawal(amount: Int) {
        account.withdraw(amount: amount)
    }
}
```

#### 5. Run Tests with Mock Implementation
Now, in your tests, you can inject a `MockBankAccount` into `BankService` to simulate interactions.

```swift
func testBankService() {
    let mockAccount = MockBankAccount()
    let bankService = BankService(account: mockAccount)

    // Perform operations
    bankService.makeDeposit(amount: 100)
    bankService.makeWithdrawal(amount: 50)

    // Assertions
    assert(mockAccount.depositCalls.count == 1, "Deposit should have been called once")
    assert(mockAccount.depositCalls.first?.amount == 100, "Deposit amount should be 100")
    assert(mockAccount.withdrawCalls.count == 1, "Withdraw should have been called once")
    assert(mockAccount.withdrawCalls.first?.amount == 50, "Withdraw amount should be 50")
}
```

By examining `mockAccount.depositCalls` and `mockAccount.withdrawCalls`, we can verify that the `BankService` called these methods with the correct arguments and in the correct sequence.

### Summary of Steps for Link Substitution in Swift

1. Define a protocol (`AccountActions`) that lists the methods you want to substitute.
2. Create the production class (`BankAccount`) that implements the protocol.
3. Create a mock class (`MockBankAccount`) implementing the same protocol, designed for testing.
4. Use dependency injection to inject either the production or mock implementation into other classes (`BankService`).
5. Run your tests using the mock class and verify behavior.

This setup mirrors the concept of link substitution, allowing you to swap out the `BankAccount` class for a test version (`MockBankAccount`) at runtime, without modifying the `BankService` class itself.
