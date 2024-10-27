The **Primitivize Parameter** technique in Swift, especially in the context of legacy code, is a way to add functionality to a complex class without directly modifying its dependencies or internal structure, which may be tightly coupled to other parts of the system. This technique allows you to work with simpler representations of the data, bypassing the complexities of the class itself. 

Let’s break down how this technique works, why it’s useful in Swift for legacy code, and walk through a practical example.

### The Problem

Imagine a legacy Swift class with a complex dependency structure, for example, a `SongSequence` class representing sequences of musical events. Each `SongSequence` object contains an array of `MusicalEvent` objects, each of which has its own dependencies (perhaps it interacts with audio frameworks, databases, or external libraries). Writing tests or adding functionality directly to `SongSequence` becomes cumbersome because initializing or modifying these objects requires handling all these dependencies.

The **Primitivize Parameter** technique helps by:
1. **Isolating the Logic** in a free function that operates on simple data types (like `Int`, `String`, or arrays).
2. **Delegating the Logic** in the original class by converting complex data to this primitive form.
3. **Enabling Testing** by making it possible to test the free function independently, without requiring the setup of the original class and its dependencies.

### Steps of Primitivize Parameter in Swift

#### Step 1: Identify the Functionality to Add

Suppose we want to add a `hasGapForPattern()` function to `SongSequence` to determine if a pattern (another `SongSequence`) can fit into any gaps within the main sequence. Since `SongSequence` has complex dependencies, we can’t directly work on it or test it easily.

#### Step 2: Create a Free Function

Instead of adding the functionality directly to `SongSequence`, create a standalone function that operates on **primitive types**. This function will simulate the logic using simplified data, such as arrays of integers representing durations.

Here’s an example of the free function:

```swift
// Free function that checks if there's a gap in baseSequence for pattern
func sequenceHasGapFor(_ baseSequence: [Int], _ pattern: [Int]) -> Bool {
    // Iterating over baseSequence to find a suitable gap for the pattern
    for i in 0..<(baseSequence.count - pattern.count + 1) {
        var gapFound = true
        for j in 0..<pattern.count {
            // Check if there is enough space (0s) for the pattern to fit
            if baseSequence[i + j] != 0 || pattern[j] > baseSequence[i + j] {
                gapFound = false
                break
            }
        }
        if gapFound { return true }
    }
    return false
}
```

In this code:
- `baseSequence` and `pattern` are arrays of integers, where each integer represents the duration of a musical event. A zero indicates a gap.
- `sequenceHasGapFor` checks if `pattern` can fit into `baseSequence` by finding a section of consecutive zeros of the same length as `pattern`.

#### Step 3: Add a Delegate Method in the Class

Now we add a method in `SongSequence` that calls this free function by **converting the complex data into a simpler, primitive form**. We assume each `MusicalEvent` has a `duration` property.

```swift
class SongSequence {
    var events: [MusicalEvent]
    
    init(events: [MusicalEvent]) {
        self.events = events
    }
    
    // Method to check if there is a gap for the pattern sequence
    func hasGapForPattern(_ patternSequence: SongSequence) -> Bool {
        // Convert events to primitive durations
        let baseDurations = getDurationsCopy()
        let patternDurations = patternSequence.getDurationsCopy()
        
        // Delegate the logic to the free function
        return sequenceHasGapFor(baseDurations, patternDurations)
    }
    
    // Helper method to convert complex events to primitive durations
    private func getDurationsCopy() -> [Int] {
        return events.map { $0.duration }
    }
}
```

Here:
- `hasGapForPattern()` prepares the data by converting `events` (complex `MusicalEvent` objects) to an array of durations.
- The `getDurationsCopy()` method extracts each `MusicalEvent`’s `duration` as an integer array.
- **Delegation**: The function `hasGapForPattern()` then delegates the actual gap-checking work to `sequenceHasGapFor()` by passing it the arrays of integers.

This design allows us to test `sequenceHasGapFor()` independently, making it easier to verify without requiring `SongSequence` or `MusicalEvent` setup.

#### Step 4: Write Tests for the Free Function

Since `sequenceHasGapFor` operates on primitive types, it’s easy to test without involving `SongSequence` or `MusicalEvent` instances. We can test it using simple arrays:

```swift
func testSequenceHasGapFor() {
    let baseSequence = [1, 0, 0, 1, 0]  // Represents a base sequence with gaps
    let pattern = [0, 0]  // Pattern to fit into the base sequence
    
    // Check if the function finds a gap for the pattern
    let result = sequenceHasGapFor(baseSequence, pattern)
    assert(result == true, "Expected to find a gap for the pattern in the base sequence.")
}
```

This test is quick and isolated:
- **Direct Testing**: Testing the `sequenceHasGapFor` function independently of `SongSequence` and `MusicalEvent`.
- **No Dependencies**: No need to set up a complex object graph of `SongSequence` or initialize dependencies for `MusicalEvent`.

### Pros and Cons of Primitivize Parameter in Swift for Legacy Code

#### Advantages

1. **Reduced Coupling**: By moving the logic into a free function, the `SongSequence` class remains decoupled from the new functionality, making it easier to add, test, and modify without impacting other parts of the system.
2. **Isolated Testing**: You can test the core logic in `sequenceHasGapFor` independently, bypassing complex dependencies that would otherwise make testing difficult.
3. **Quick Implementation**: Primitivize Parameter is an effective quick solution for adding functionality when refactoring the entire class is impractical.

#### Disadvantages

1. **Exposes Internal Representation**: The `getDurationsCopy()` method exposes internal details of `SongSequence` (e.g., durations), which might not be ideal for maintaining encapsulation.
2. **Data Duplication**: The internal state (durations) is duplicated as an array of integers, which could lead to inconsistencies or increased memory usage in a larger system.
3. **Temporary Solution**: This technique doesn’t fix the underlying dependency issues. It’s often a **stopgap solution** until the legacy code can be properly refactored.

### Alternative Next Steps: Refactoring for Better Design

After using Primitivize Parameter, you may consider further refactoring:
- **Sprout Class**: Create a `GapChecker` class to encapsulate the gap-checking logic, which could be a more sustainable design. 
- **Dependency Injection**: Refactor `SongSequence` to inject dependencies or use protocols, making it more testable and modular in the long term.
- **Facade Pattern**: If the class relies on several external systems, a facade can hide the complex details, making it easier to handle dependencies when writing tests.

### Summary

The **Primitivize Parameter** technique in Swift allows you to add and test functionality in legacy code by working with simplified data representations and delegating functionality to free functions. Although it can leave code in a less-than-ideal state, it’s often a practical way to implement and verify new features without undertaking a large refactoring effort.
