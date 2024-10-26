Encapsulating global references in Swift involves wrapping global variables or functions in a class or struct. This makes the code more modular, testable, and maintainable by creating a clear separation between how data is stored and how it's used throughout the application. I’ll walk through this concept in detail with a step-by-step example using Swift.

### What Are Global References?

Global references are variables or functions that are defined at a global scope, accessible from anywhere in the code. While convenient, they come with some downsides:
- **Tight Coupling**: Any part of the code that uses these global variables is directly coupled to them.
- **Difficult to Test**: Replacing or modifying global state in tests is hard, making it challenging to isolate components during testing.
- **Reduced Maintainability**: As the app grows, understanding which parts of the code rely on global variables can become challenging.

### Strategy: Encapsulate Global References

The strategy here is to wrap the global variable or function in a class or struct, providing a single access point to it. This way, you can easily replace or mock this access point during testing and refactor it without impacting the rest of your code.

### Step-by-Step Example in Swift

Let’s assume you have an application with a global configuration that manages settings like whether a feature is enabled and the app’s version:

```swift
// Initial global reference
var globalConfig = Configuration()

class Configuration {
    var version: String = "1.0"
    var isFeatureEnabled: Bool = false
}

func printAppVersion() {
    print("App Version: \(globalConfig.version)")
}

func enableFeature() {
    globalConfig.isFeatureEnabled = true
}
```

Here, `globalConfig` is a global instance of the `Configuration` class. It’s accessed directly in functions like `printAppVersion()` and `enableFeature()`. This makes it difficult to replace `globalConfig` with a different configuration during testing, as it is tightly coupled to those functions.

### Refactoring Using Encapsulation

1. **Create a `ConfigManager` class**: Wrap the global reference in a class. This class will manage access to the `Configuration`.

   ```swift
   class ConfigManager {
       var configuration: Configuration

       init(configuration: Configuration) {
           self.configuration = configuration
       }
   }
   ```

   Now, instead of accessing `Configuration` directly through `globalConfig`, you’ll access it through an instance of `ConfigManager`.

2. **Replace the global variable**: Replace `globalConfig` with a global instance of `ConfigManager`.

   ```swift
   var configManager = ConfigManager(configuration: Configuration())
   ```

   This global instance will now serve as the access point for all configuration settings.

3. **Update methods to use `configManager`**: Modify any methods that previously accessed `globalConfig` directly to now use `configManager`:

   ```swift
   func printAppVersion() {
       print("App Version: \(configManager.configuration.version)")
   }

   func enableFeature() {
       configManager.configuration.isFeatureEnabled = true
   }
   ```

   Now, instead of directly using `globalConfig`, these methods use the `configuration` property of `configManager`.

4. **Improve Encapsulation with Methods**: Make `ConfigManager` more robust by adding methods that wrap configuration operations:

   ```swift
   class ConfigManager {
       var configuration: Configuration

       init(configuration: Configuration) {
           self.configuration = configuration
       }

       func getAppVersion() -> String {
           return configuration.version
       }

       func enableFeature() {
           configuration.isFeatureEnabled = true
       }

       func isFeatureEnabled() -> Bool {
           return configuration.isFeatureEnabled
       }
   }
   ```

   Now, `ConfigManager` hides the details of how the `Configuration` is managed. Functions like `printAppVersion()` can be further simplified:

   ```swift
   func printAppVersion() {
       print("App Version: \(configManager.getAppVersion())")
   }
   ```

### Testing the Encapsulated Code

One of the main benefits of this refactoring is that it makes testing easier. You can now replace `configManager` with a mock version for testing:

```swift
class MockConfigManager: ConfigManager {
    override init(configuration: Configuration) {
        super.init(configuration: configuration)
        self.configuration.version = "TestVersion"
        self.configuration.isFeatureEnabled = false
    }

    override func getAppVersion() -> String {
        return "MockVersion"
    }

    override func enableFeature() {
        // No-op or custom behavior for testing
    }

    override func isFeatureEnabled() -> Bool {
        return false
    }
}

// In a test, you can replace configManager with an instance of MockConfigManager:
let mockConfigManager = MockConfigManager(configuration: Configuration())
configManager = mockConfigManager

printAppVersion()  // Prints: App Version: MockVersion
```

This mock version can be used to simulate different configurations during tests without affecting the global state of the app.

### Benefits of Encapsulating Global References

1. **Improved Testability**: By using `ConfigManager`, you can replace `configManager` with a mock version in tests, allowing you to control how the `Configuration` behaves.

2. **Single Responsibility Principle (SRP)**: The `ConfigManager` is now responsible for managing configuration access, making the `Configuration` class itself simpler.

3. **Reduced Coupling**: Other parts of the code no longer rely directly on the `globalConfig`. Instead, they depend on the interface provided by `ConfigManager`, making the code more modular and easier to refactor.

4. **Better Encapsulation**: The details of how the `Configuration` is managed are hidden inside `ConfigManager`, providing a clean API for accessing and modifying configuration data.

### Extending the Refactoring

Once the global reference is encapsulated, you can further enhance the `ConfigManager` class. For example, if you need to save the configuration to a file, you can add methods for loading and saving:

```swift
class ConfigManager {
    var configuration: Configuration

    init(configuration: Configuration) {
        self.configuration = configuration
    }

    func getAppVersion() -> String {
        return configuration.version
    }

    func enableFeature() {
        configuration.isFeatureEnabled = true
    }

    func isFeatureEnabled() -> Bool {
        return configuration.isFeatureEnabled
    }

    func saveConfiguration() {
        // Save the configuration to disk or user defaults
    }

    func loadConfiguration() {
        // Load the configuration from disk or user defaults
    }
}
```

With this design, any logic related to managing the `Configuration` (like saving or loading) is encapsulated within `ConfigManager`, making the overall design more robust and easier to maintain.

### Summary of the Process

1. **Wrap global variables in a class**: This makes a single point of access.
2. **Replace direct references**: Update functions and classes to access data through this new class.
3. **Enhance encapsulation**: Add methods to simplify access and modify the encapsulated data.
4. **Use mocks or fakes for testing**: Replace the global instance with a mock during tests for better control.

By following this approach, you get a cleaner, more maintainable codebase with improved testability, and you avoid the pitfalls of direct global references.
