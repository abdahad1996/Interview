### Problem Overview: The Case of the “Helpful” Language Feature

Language designers often aim to simplify the lives of developers while ensuring code safety and security. Features like `sealed` in C# or `final` in Java prevent classes from being subclassed, limiting how the internal behavior of these classes can be extended. These restrictions are designed to protect the integrity of the class, particularly in scenarios involving sensitive operations or security concerns. While this enhances the reliability of such classes, it can make testing or extending them for specific needs quite difficult.

In many real-world scenarios, we encounter situations where library classes are designed with restrictions that prevent direct instantiation or subclassing. When these classes are part of external libraries or frameworks, we cannot modify them directly, and this limitation can complicate testing or refactoring efforts.

### Theory: Encapsulation, Sealed Classes, and Testing Challenges

Encapsulation is a core concept of object-oriented programming (OOP). It involves keeping certain details of a class hidden from other classes, exposing only what is necessary through a well-defined interface. Encapsulation helps in reducing dependencies and keeps classes easier to manage.

However, when classes are marked as `sealed` (or `final` in other languages), they become non-extendable, which means that you cannot create subclasses of these classes. This restriction is often applied to sensitive classes where allowing subclassing could open up potential security risks or misuse. But it also makes testing challenging, especially when we want to isolate parts of the logic or create mock objects.

For example, let’s assume we have a Swift class `ImageProcessor` that processes images based on some complex logic:

```swift
final class ImageProcessor {
    private let filter: CIFilter
    
    init(filterName: String) {
        guard let filter = CIFilter(name: filterName) else {
            fatalError("Invalid filter name")
        }
        self.filter = filter
    }
    
    func applyFilter(to image: CIImage) -> CIImage? {
        filter.setValue(image, forKey: kCIInputImageKey)
        return filter.outputImage
    }
}
```

Here, `ImageProcessor` is marked as `final`, which means it cannot be subclassed. Additionally, `CIFilter` is part of Apple’s Core Image framework and is not easily mockable or subclassable, posing challenges when trying to test the `applyFilter` method.

### Example: Adapt Parameter Pattern in Swift

When dealing with classes that are `final` or where direct subclassing isn’t possible, we can use the **Adapt Parameter** pattern to make testing easier. The idea is to wrap the class in a custom protocol or interface and then create mock implementations of that protocol.

#### Step 1: Define a Protocol

We define a protocol that matches the behavior of the `CIFilter` class that we need for testing:

```swift
protocol Filter {
    func setInputImage(_ image: CIImage)
    func outputImage() -> CIImage?
}
```

#### Step 2: Create a Wrapper

Next, create a wrapper around `CIFilter` that conforms to the `Filter` protocol:

```swift
class CIFilterWrapper: Filter {
    private let filter: CIFilter
    
    init(filterName: String) {
        guard let filter = CIFilter(name: filterName) else {
            fatalError("Invalid filter name")
        }
        self.filter = filter
    }
    
    func setInputImage(_ image: CIImage) {
        filter.setValue(image, forKey: kCIInputImageKey)
    }
    
    func outputImage() -> CIImage? {
        return filter.outputImage
    }
}
```

#### Step 3: Modify the `ImageProcessor` to Use the Protocol

Now, update `ImageProcessor` to use the `Filter` protocol instead of directly using `CIFilter`:

```swift
final class ImageProcessor {
    private let filter: Filter
    
    init(filter: Filter) {
        self.filter = filter
    }
    
    func applyFilter(to image: CIImage) -> CIImage? {
        filter.setInputImage(image)
        return filter.outputImage()
    }
}
```

#### Step 4: Create a Mock for Testing

With this structure, we can now create a mock implementation of the `Filter` protocol for testing:

```swift
class MockFilter: Filter {
    private var inputImage: CIImage?
    var output: CIImage?
    
    func setInputImage(_ image: CIImage) {
        inputImage = image
    }
    
    func outputImage() -> CIImage? {
        return output
    }
}
```

#### Step 5: Write Tests Using the Mock

We can use `MockFilter` in tests to verify the behavior of `ImageProcessor`:

```swift
import XCTest

class ImageProcessorTests: XCTestCase {
    func testApplyFilter() {
        let mockFilter = MockFilter()
        mockFilter.output = CIImage() // Provide a test output image
        let processor = ImageProcessor(filter: mockFilter)
        
        let inputImage = CIImage()
        let result = processor.applyFilter(to: inputImage)
        
        XCTAssertNotNil(result)
        XCTAssertEqual(result, mockFilter.output)
    }
}
```

### Conclusion

In this example, the **Adapt Parameter** pattern allowed us to abstract away the `CIFilter` class and replace it with a test-friendly protocol, making `ImageProcessor` easier to test. This approach is particularly useful when dealing with third-party libraries or framework classes that cannot be modified or subclassed. It creates a layer of separation that provides more control during testing without sacrificing the encapsulation or safety originally intended by the `final` or `sealed` restrictions.

### Key Takeaways

1. **Encapsulation vs. Testability**: Language features like `final` or `sealed` help protect the internal state of classes but can hinder testing. Balancing these aspects is a common challenge in software design.
2. **Adapt Parameter Pattern**: This pattern involves creating protocols or interfaces and wrapping library classes, making them testable by providing mock implementations.
3. **Improving Design**: Such patterns not only solve immediate testing issues but can also lead to more maintainable and flexible designs by separating concerns and dependencies in the code.
