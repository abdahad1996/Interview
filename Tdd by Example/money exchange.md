Let's walk through the entire theory of **Test-Driven Development (TDD)** using the detailed multi-currency example step by step, including every concept and example in the text provided.

### 1. **The Money Example**

The example starts with the need to generate a multi-currency report. The report must add up values in different currencies and convert them based on exchange rates. Initially, the report looks like this:

```
Instrument    Shares    Price
IBM           1000      25
GE            400       100
Total:                  75000
```

After adding currencies and specifying exchange rates, the report is revised to:

```
Instrument    Shares    Price    Currency
IBM           1000      25000    USD
Novartis      400       40000    CHF
Total:                  75000    USD
```

An exchange rate is added between CHF and USD:
```
From   To    Rate
CHF    USD   1.5
```

The task is to make sure that these values can be added together, multiplied, and converted according to the exchange rate.

### 2. **Defining the Behavior**

The behavior we want the system to produce can be summarized as follows:
- We need to **add amounts in different currencies** and convert the result, using exchange rates.
- We need to **multiply an amount by a number** (e.g., multiplying price per share by the number of shares).

We start by listing the tests we need to implement:
- `$5 + 10 CHF = $10` if the exchange rate is 2:1.
- `$5 * 2 = $10`.

### 3. **Starting with Tests, Not Objects**

TDD doesn't start by writing objects or code. It starts with **writing tests**. We aim to think small: instead of starting with the complex addition of currencies, we begin with something simple, like multiplication.

### 4. **First Test: Multiplication**

Here’s the first test:
```java
public void testMultiplication() {
    Dollar five = new Dollar(5);
    five.times(2);
    assertEquals(10, five.amount);
}
```

This test fails initially because:
1. We don’t have a `Dollar` class.
2. There’s no `times` method.
3. The `amount` field doesn’t exist.

Now, let's **make the test compile** by creating the `Dollar` class and the necessary methods and fields:
```java
class Dollar {
    int amount;
    Dollar(int amount) {
        this.amount = amount;
    }
    void times(int multiplier) {
        this.amount *= multiplier;
    }
}
```

At this stage, we have implemented enough to make the test pass. The class `Dollar` has an `amount` field and a `times` method that multiplies the amount by a given multiplier.

### 5. **Making Tests Pass, then Refactoring**

After running the test, the test passes, but the implementation is not perfect. For example, we hardcoded values and implemented logic directly in the class. Now we refactor:
```java
class Dollar {
    int amount;
    Dollar(int amount) {
        this.amount = amount;
    }
    Dollar times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```

By returning a new `Dollar` object instead of modifying the existing one, we avoid side effects where the object’s state changes unexpectedly.

### 6. **Adding New Tests: Equality**

Next, we add the test for **equality**:
```java
public void testEquality() {
    assertTrue(new Dollar(5).equals(new Dollar(5)));
    assertFalse(new Dollar(5).equals(new Dollar(6)));
}
```

To pass this test, we implement the `equals` method in the `Dollar` class:
```java
@Override
public boolean equals(Object object) {
    Dollar dollar = (Dollar) object;
    return amount == dollar.amount;
}
```

This allows us to compare `Dollar` objects correctly and ensures that two `Dollar` instances with the same value are considered equal.

### 7. **Refactoring with a Superclass: Money**

After adding more functionality, we notice duplication between different currencies. For example, if we introduce `Franc`, we would have similar logic repeated. To avoid this, we generalize with a `Money` superclass:
```java
class Money {
    protected int amount;
    protected String currency;
}
```

Both `Dollar` and `Franc` extend the `Money` class:
```java
class Dollar extends Money {
    Dollar(int amount) {
        this.amount = amount;
        this.currency = "USD";
    }
}

class Franc extends Money {
    Franc(int amount) {
        this.amount = amount;
        this.currency = "CHF";
    }
}
```

By introducing `Money`, we can centralize common functionality like `amount`, `times`, and `equals` in one place, reducing duplication.

### 8. **Handling Mixed Currencies with Bank**

Next, we introduce a way to handle **mixed currencies**. The goal is to add amounts in different currencies, such as USD and CHF, and convert them based on exchange rates. For this, we create a `Bank` class responsible for currency conversion:
```java
Bank bank = new Bank();
bank.addRate("CHF", "USD", 2);
Money result = bank.reduce(Money.franc(10), "USD");
assertEquals(Money.dollar(5), result);
```

The `Bank` class has an `addRate` method that stores exchange rates and a `reduce` method that converts amounts to a target currency:
```java
class Bank {
    private Hashtable<Pair, Integer> rates = new Hashtable<>();
    void addRate(String from, String to, int rate) {
        rates.put(new Pair(from, to), rate);
    }

    Money reduce(Expression source, String to) {
        return source.reduce(this, to);
    }
}
```

The **reduce** method uses exchange rates to convert `Franc` to `Dollar` based on the stored rates.

### 9. **Sum and Expression**

To support adding different amounts of money, we introduce the `Expression` interface, which represents any money-related operation (such as adding or multiplying money). A `Sum` class implements this interface:
```java
interface Expression {
    Money reduce(Bank bank, String to);
}

class Sum implements Expression {
    Money augend;
    Money addend;
    Sum(Money augend, Money addend) {
        this.augend = augend;
        this.addend = addend;
    }

    public Money reduce(Bank bank, String to) {
        int amount = augend.amount + addend.amount;
        return new Money(amount, to);
    }
}
```

The `Sum` class allows us to represent the sum of two amounts of money. The `reduce` method converts the sum into a target currency using the `Bank`.

Now, we can add two amounts of money and reduce them to a single currency:
```java
Expression sum = new Sum(fiveBucks, tenFrancs);
Money result = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), result);
```
You're right that I need to provide examples from the text starting from **Avoiding Duplication**. Let's continue from there, filling in the examples as needed.

---

### 8. **Avoiding Duplication**

A major part of **Test-Driven Development (TDD)** is **eliminating duplication**. Duplication often occurs when the same logic, data, or code is repeated in multiple places, which increases the risk of errors and makes the code harder to maintain. TDD encourages removing duplication after passing tests, ensuring the code is simple and efficient.

#### Example: Duplication between Test and Code

After getting the multiplication test to pass, we notice **duplication** between the test data and the code:
```java
Dollar five = new Dollar(5);
five.times(2);
```
This test works, but we realize that we manually did the multiplication in our heads when asserting the result. To eliminate this duplication, we generalize the code:
```java
void times(int multiplier) {
    amount *= multiplier;
}
```
This improvement removes hardcoding and duplication between the calculation and the test, relying on variables instead.

#### Example: Duplication in `Dollar` and `Franc`

Initially, we created two separate classes, `Dollar` and `Franc`. The `Dollar` class looked like this:
```java
class Dollar {
    int amount;
    Dollar(int amount) {
        this.amount = amount;
    }
    Dollar times(int multiplier) {
        return new Dollar(amount * multiplier);
    }
}
```

Later, we implemented the same functionality for `Franc`:
```java
class Franc {
    int amount;
    Franc(int amount) {
        this.amount = amount;
    }
    Franc times(int multiplier) {
        return new Franc(amount * multiplier);
    }
}
```

Here we see **duplication**: both classes have the same `amount` field, constructor, and `times` method, which violates the **Don’t Repeat Yourself (DRY)** principle.

#### Refactoring to Remove Duplication

To remove this duplication, we create a **common superclass** `Money`:
```java
class Money {
    protected int amount;
    protected String currency;

    Money(int amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    Money times(int multiplier) {
        return new Money(amount * multiplier, currency);
    }
}
```

Now, `Dollar` and `Franc` can both extend `Money`, removing the duplication:
```java
class Dollar extends Money {
    Dollar(int amount) {
        super(amount, "USD");
    }
}

class Franc extends Money {
    Franc(int amount) {
        super(amount, "CHF");
    }
}
```

By extracting the shared behavior into the superclass `Money`, we’ve eliminated the repeated code in `Dollar` and `Franc`. This is a common TDD practice: after writing a failing test and getting it to pass, refactor the code to remove duplication and make it cleaner.

---

### 9. **Handling Mixed Currencies with Bank**

After refactoring `Dollar` and `Franc` into a common `Money` superclass, we need to address the issue of **currency conversion**. We introduce the `Bank` class to handle exchange rates and conversion.

#### Example: Handling Conversion with Bank

Here’s the scenario: we want to add 10 CHF to 5 USD. To do this, we first need to convert CHF to USD based on an exchange rate. The `Bank` class manages this conversion.

We write the following test:
```java
Bank bank = new Bank();
bank.addRate("CHF", "USD", 2);
Money result = bank.reduce(Money.franc(10), "USD");
assertEquals(Money.dollar(5), result);
```

In this test, we are saying that 10 CHF should convert to 5 USD, given an exchange rate of 2:1. To make this test pass, we implement the `reduce` method in the `Bank` class, which looks up the exchange rate and performs the conversion:
```java
class Bank {
    private Hashtable<Pair, Integer> rates = new Hashtable<>();

    void addRate(String from, String to, int rate) {
        rates.put(new Pair(from, to), rate);
    }

    Money reduce(Expression source, String to) {
        return source.reduce(this, to);
    }

    int rate(String from, String to) {
        if (from.equals(to)) return 1;
        return rates.get(new Pair(from, to));
    }
}
```

By centralizing the currency conversion logic in the `Bank` class, we avoid duplicating this logic in other parts of the code. This way, any future currency conversions will go through the `Bank`, ensuring consistency and maintainability.

---

### 10. **Sum and Expression**

To handle **mixed currency arithmetic**, we introduce the concept of **expressions**. An expression can be the sum of two amounts of money or a multiplication of money. This generalization allows us to handle more complex operations.

#### Example: Adding Mixed Currencies

When we need to add two different currencies, such as 5 USD and 10 CHF, we represent this operation using the `Sum` class, which implements the `Expression` interface:
```java
Expression sum = new Sum(fiveBucks, tenFrancs);
Money result = bank.reduce(sum, "USD");
assertEquals(Money.dollar(10), result);
```

Here’s how the `Sum` class looks:
```java
class Sum implements Expression {
    Money augend;
    Money addend;

    Sum(Money augend, Money addend) {
        this.augend = augend;
        this.addend = addend;
    }

    public Money reduce(Bank bank, String to) {
        int amount = augend.reduce(bank, to).amount + addend.reduce(bank, to).amount;
        return new Money(amount, to);
    }
}
```

The `Sum` class represents the addition of two money objects, and the `reduce` method is responsible for converting both amounts to the target currency before adding them.

---

### 11. **Polymorphism and Flexibility**

Through the use of interfaces and polymorphism, TDD helps create a flexible design that can easily handle new operations. Both `Money` and `Sum` implement the `Expression` interface, allowing the system to treat them uniformly.

#### Example: Expression Interface

By introducing the `Expression` interface, we ensure that both `Money` and `Sum` can be handled in the same way:
```java
interface Expression {
    Money reduce(Bank bank, String to);
}
```

The `reduce` method is used to convert expressions (whether sums or individual money amounts) to a specific currency. This abstraction makes the code more flexible and easy to extend.

#### Example: Generalizing Money and Sum

The `Money` class now implements `Expression`, and the `plus` method returns an `Expression`:
```java
class Money implements Expression {
    public Expression plus(Expression addend) {
        return new Sum(this, addend);
    }

    public Money reduce(Bank bank, String to) {
        return this;
    }
}
```

This way, whether you’re adding two `Money` objects or a `Money` object and a `Sum`, the system treats both operations consistently using the `Expression` interface.

---

### 12. **Testing Edge Cases**

Throughout the process, we write tests for **edge cases** to ensure the system behaves correctly even under unusual conditions. For example, we test whether `Money` objects from different currencies are considered equal:
```java
assertFalse(new Franc(5).equals(new Dollar(5)));
```

This test ensures that the system distinguishes between different currencies and does not consider `5 Franc` equal to `5 Dollars`.

---

### 13. **Refactoring After Tests Pass**

Once tests pass, we continuously **refactor** the code to ensure it remains clean, readable, and maintainable. For example, after getting currency conversion and addition to work, we refactor the code to remove any hardcoded values or duplicated logic.

In the case of exchange rates, we initially had hardcoded exchange rates like this:
```java
int rate = (currency.equals("CHF") && to.equals("USD")) ? 2 : 1;
```

After refactoring, the exchange rates are stored in a `Hashtable` inside the `Bank` class, making the system more flexible and easier to extend.

---

### Conclusion: The Complete Theory of TDD in the Multi-Currency Example

This multi-currency example demonstrates how **Test-Driven Development (TDD)** can be applied to solve complex problems by breaking them down into small, manageable steps. Starting from simple tests like multiplication and equality, we gradually build up the system by adding new functionality, handling mixed currencies, and refactoring to eliminate duplication.

Through this process, we:
- **Start small**: We begin with simple tests and small pieces of functionality, gradually adding complexity.
- **Red/Green/Refactor**: We write tests (red), make them pass (green), and then refactor the code to remove duplication and improve design.
- **Eliminate duplication**: Duplication is constantly identified and removed, making the code simpler and more maintainable.
- **Use polymorphism and abstraction**: By introducing abstractions like `Expression`, we create a flexible system that can handle various operations (e.g., adding, multiplying, converting currencies).
- **Test edge cases**: Testing edge cases ensures that the system behaves correctly even under less common conditions.

In summary, TDD helps create clean, flexible, and well-tested code that can handle complex scenarios like multi-currency arithmetic. The process ensures that the system is built incrementally, with each piece of functionality thoroughly tested before moving on to the next.
### Conclusion: The Theory of TDD in Action

Through this example, the following key points of **Test-Driven Development** emerge:
1. **Start small and build incrementally**: Begin with the simplest possible test (e.g., multiplication), and gradually add more complex behavior.
2. **Red/Green/Refactor cycle**: Write a test, make it pass (green), then refactor the code to improve its design.
3. **Let tests guide the design**: The process of writing tests first encourages you to think about how the code will be used, leading to better-designed and more maintainable software.
4. **Introduce abstraction when needed**: By introducing abstractions like `Expression`, TDD helps you create flexible code that can be extended without requiring major changes.
5. **Always test edge cases**: Testing edge cases early ensures that the system behaves correctly even under unusual conditions.


. **Continuous refactoring**: Once tests pass, always refactor to keep the code clean, avoid duplication, and improve its structure.

This example demonstrates how TDD can help you develop complex features (like multi-currency money systems) by breaking the problem into small, manageable steps, always ensuring that the code works correctly before moving forward.
