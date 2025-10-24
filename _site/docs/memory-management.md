# Memory Management & Performance

[← Back to Main](../README.md) | [Previous: Dependency Management](dependency-management.md) | [Next: Testing →](testing.md)

## Table of Contents
- [ARC in Depth](#arc-in-depth)
- [Strong, Weak & Unowned References](#strong-weak--unowned-references)
- [Retain Cycles & Memory Leaks](#retain-cycles--memory-leaks)
- [Instruments for Debugging](#instruments-for-debugging--profiling)
- [Performance Optimization](#optimizing-scrolling-performance)
- [Best Practices](#best-practices-for-high-performance)

---

## ARC in Depth

Automatic Reference Counting manages memory automatically.

### How ARC Works

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) initialized")
    }
    deinit {
        print("\(name) deinitialized")
    }
}

var person1: Person? = Person(name: "John") // RC = 1
var person2 = person1 // RC = 2
var person3 = person1 // RC = 3

person1 = nil // RC = 2
person2 = nil // RC = 1
person3 = nil // RC = 0 → deinit called
```

---

## Strong, Weak & Unowned References

### Strong (Default)

```swift
class Owner {
    var property: Property?
}

let owner = Owner()
owner.property = Property() // Strong reference
```

### Weak

```swift
class ViewController: UIViewController {
    weak var delegate: SomeDelegate? // Weak reference
}
```

### Unowned

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
}

class CreditCard {
    let number: String
    unowned let owner: Customer // Unowned reference
    
    init(number: String, owner: Customer) {
        self.number = number
        self.owner = owner
    }
}
```

---

## Retain Cycles & Memory Leaks

### Common Retain Cycle

```swift
// BAD: Retain cycle
class ViewController: UIViewController {
    var closure: (() -> Void)?
    
    func setup() {
        closure = {
            self.view.backgroundColor = .red // Captures self strongly
        }
    }
}

// GOOD: Break retain cycle
class ViewController: UIViewController {
    var closure: (() -> Void)?
    
    func setup() {
        closure = { [weak self] in
            self?.view.backgroundColor = .red
        }
    }
}
```

### Capture Lists

```swift
// Weak self
{ [weak self] in
    self?.doSomething()
}

// Unowned self
{ [unowned self] in
    self.doSomething()
}

// Multiple captures
{ [weak self, weak other] in
    self?.doSomething()
    other?.doSomething()
}
```

---

## Instruments for Debugging & Profiling

### Memory Leaks Detection

1. Product → Profile (Cmd+I)
2. Select "Leaks" template
3. Record and use app
4. View leak traces

### Allocations

Monitor memory allocation:
- Track object creation
- Identify memory growth
- Find allocation patterns

---

## Optimizing Scrolling Performance

### TableView/CollectionView

```swift
// Reuse cells
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
    // Configure cell
    return cell
}

// Estimate heights
func tableView(_ tableView: UITableView, estimatedHeightForRowAt indexPath: IndexPath) -> CGFloat {
    return 100
}

// Cache heights
var heightCache: [IndexPath: CGFloat] = [:]
```

---

## Best Practices for High Performance

1. **Use value types** (structs) when possible
2. **Lazy loading** for expensive operations
3. **Image optimization** and caching
4. **Background processing** for heavy tasks
5. **Avoid premature optimization**
6. **Profile before optimizing**
7. **Use Instruments** regularly
8. **Monitor memory** usage

### Interview Questions

**Q: When to use weak vs unowned?**

**A:** Use `weak` when reference can become nil. Use `unowned` when reference should never be nil after initialization. Weak is safer.

**Q: How to detect memory leaks?**

**A:** 
- Use Instruments Leaks tool
- Check deinit is called
- Use Memory Graph Debugger
- Monitor memory usage

---

[← Previous: Dependency Management](dependency-management.md) | [Next: Testing →](testing.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: Explain ARC (Automatic Reference Counting) in detail

**Answer:**

ARC automatically manages memory by tracking references to objects.

**How it Works:**

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) initialized")
    }
    deinit {
        print("\(name) deallocated")
    }
}

var person1: Person? = Person(name: "John")  // RC = 1
var person2 = person1  // RC = 2
var person3 = person1  // RC = 3

person1 = nil  // RC = 2
person2 = nil  // RC = 1
person3 = nil  // RC = 0 → deinit called
```

**Rules:**
- Every strong reference increases count
- Setting to nil decreases count
- When count reaches 0, object is deallocated
- Only applies to class instances (reference types)
- Structs and enums are value types (no ARC)

**Memory Management:**
- Happens at compile time
- No runtime overhead (unlike garbage collection)
- Deterministic (immediate deallocation)

### Q2: When to use weak vs unowned references?

**Answer:**

**weak:**
- Optional reference
- Automatically becomes nil when object is deallocated
- Safe, no crashes
- Slight performance overhead (optional checking)

```swift
class ViewController {
    weak var delegate: MyDelegate?  // Can become nil
}
```

**unowned:**
- Non-optional reference
- Doesn't become nil
- Crashes if accessed after deallocation
- Slightly better performance
- Use only when you're certain reference will never be nil

```swift
class CreditCard {
    unowned let owner: Customer  // Owner always exists
}
```

**When to Use:**

| Scenario | Use |
|----------|-----|
| Delegate pattern | weak |
| Parent-child (child holds parent) | weak |
| Object always exists together | unowned |
| Not sure | weak (safer) |

**Example:**
```swift
class Parent {
    var child: Child?
}

class Child {
    weak var parent: Parent?  // weak (parent can be deallocated)
}

class A {
    var b: B?
}

class B {
    unowned let a: A  // unowned (B can't exist without A)
    init(a: A) {
        self.a = a
    }
}
```

### Q3: What's a retain cycle and how do you fix it?

**Answer:**

**Retain Cycle:** Two objects hold strong references to each other, preventing deallocation.

**Problem:**
```swift
class Person {
    var apartment: Apartment?
}

class Apartment {
    var tenant: Person?  // Strong reference
}

var john: Person? = Person()
var unit4A: Apartment? = Apartment()

john?.apartment = unit4A
unit4A?.tenant = john

john = nil
unit4A = nil
// Neither deallocated - MEMORY LEAK!
```

**Solution 1: Weak Reference**
```swift
class Apartment {
    weak var tenant: Person?  // Weak reference
}

// Now both will be deallocated
```

**Solution 2: Unowned Reference**
```swift
class Person {
    var card: CreditCard?
}

class CreditCard {
    unowned let owner: Person  // Unowned reference
    init(owner: Person) {
        self.owner = owner
    }
}
```

**Common in Closures:**
```swift
// Problem
class ViewController {
    var name = "View"
    var closure: (() -> Void)?
    
    func setup() {
        closure = {
            print(self.name)  // Captures self strongly
        }
    }
}

// Solution
closure = { [weak self] in
    guard let self = self else { return }
    print(self.name)
}
```

**Detection:**
- Use Xcode Memory Graph Debugger
- Check deinit is called
- Use Instruments (Leaks tool)

### Q4: How do you debug memory leaks in iOS?

**Answer:**

**1. Memory Graph Debugger (Xcode):**
- Run app
- Debug → View Memory Graph
- Look for unexpected object retention
- Click object to see reference cycle

**2. Instruments - Leaks Tool:**
- Product → Profile (Cmd + I)
- Select "Leaks" template
- Record and use app
- Leaks appear as red bars
- View call stack for each leak

**3. Instruments - Allocations:**
- Track memory growth
- Find objects not being deallocated
- Compare snapshots
- See allocation stack traces

**4. deinit Verification:**
```swift
class MyClass {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) initialized")
    }
    
    deinit {
        print("\(name) deallocated")  // Should be called
    }
}
```

**5. Memory Warnings:**
```swift
override func didReceiveMemoryWarning() {
    super.didReceiveMemoryWarning()
    print("Memory warning!")
    // Clear caches
}
```

### Q5: What causes the most common memory leaks in iOS?

**Answer:**

**1. Delegate Retain Cycles:**
```swift
// BAD
protocol MyDelegate { }
class DataSource {
    var delegate: MyDelegate?  // Strong reference
}

// GOOD
protocol MyDelegate: AnyObject { }
class DataSource {
    weak var delegate: MyDelegate?  // Weak reference
}
```

**2. Closure Retain Cycles:**
```swift
// BAD
class ViewController {
    var closure: (() -> Void)?
    
    func setup() {
        closure = {
            self.updateUI()  // Captures self strongly
        }
    }
}

// GOOD
closure = { [weak self] in
    self?.updateUI()
}
```

**3. Timer Retain Cycles:**
```swift
// BAD
class ViewController {
    var timer: Timer?
    
    func startTimer() {
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.update()  // Captures self
        }
    }
}

// GOOD
timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
    self?.update()
}

deinit {
    timer?.invalidate()  // Must invalidate!
}
```

**4. Notification Observers:**
```swift
// BAD
NotificationCenter.default.addObserver(self, selector: #selector(handle), name: .someNotification, object: nil)
// Forgot to remove

// GOOD
deinit {
    NotificationCenter.default.removeObserver(self)
}
```

**5. Parent-Child Cycles:**
```swift
class Parent {
    var child: Child?
}

class Child {
    weak var parent: Parent?  // Must be weak!
}
```

### Q6: How do you optimize app performance?

**Answer:**

**1. Profile First:**
- Use Instruments before optimizing
- Identify actual bottlenecks
- Don't guess

**2. Image Optimization:**
```swift
// Downscale images
func downsample(imageAt url: URL, to size: CGSize) -> UIImage? {
    let options: [CFString: Any] = [
        kCGImageSourceCreateThumbnailFromImageAlways: true,
        kCGImageSourceThumbnailMaxPixelSize: max(size.width, size.height),
        kCGImageSourceShouldCacheImmediately: true
    ]
    
    guard let imageSource = CGImageSourceCreateWithURL(url as CFURL, nil),
          let image = CGImageSourceCreateThumbnailAtIndex(imageSource, 0, options as CFDictionary) else {
        return nil
    }
    
    return UIImage(cgImage: image)
}
```

**3. Lazy Loading:**
```swift
lazy var expensiveObject: HeavyObject = {
    return HeavyObject()  // Created only when accessed
}()
```

**4. Background Processing:**
```swift
DispatchQueue.global(qos: .utility).async {
    let processed = self.processData()
    
    DispatchQueue.main.async {
        self.updateUI(with: processed)
    }
}
```

**5. Reduce View Hierarchy:**
- Flatten view structure
- Use stack views
- Minimize transparent views

**6. Reuse Cells:**
```swift
let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
```

### Q7: What tools do you use to measure app performance?

**Answer:**

**1. Instruments:**

**Time Profiler:**
- CPU usage per method
- Find performance bottlenecks
- See call stack traces

**Allocations:**
- Memory usage over time
- Object creation/destruction
- Find memory leaks

**Leaks:**
- Detect retain cycles
- Find leaked objects

**Network:**
- Track API calls
- Response times
- Data usage

**2. Xcode Debug Gauges:**
- Real-time CPU usage
- Memory usage
- Network activity
- FPS (frames per second)

**3. MetricKit (iOS 13+):**
```swift
import MetricKit

class MetricsManager: NSObject, MXMetricManagerSubscriber {
    override init() {
        super.init()
        MXMetricManager.shared.add(self)
    }
    
    func didReceive(_ payloads: [MXMetricPayload]) {
        for payload in payloads {
            // CPU metrics
            let cpuMetrics = payload.cpuMetrics
            
            // Memory metrics
            let memoryMetrics = payload.memoryMetrics
            
            // Network metrics
            let networkMetrics = payload.networkTransferMetrics
        }
    }
}
```

**4. Custom Metrics:**
```swift
class PerformanceMonitor {
    func measureExecutionTime(_ block: () -> Void) {
        let start = CFAbsoluteTimeGetCurrent()
        block()
        let elapsed = CFAbsoluteTimeGetCurrent() - start
        print("Execution time: \(elapsed)s")
    }
}
```

**Best Practice:**
- Profile on real devices
- Test on oldest supported device
- Monitor production metrics
- Set performance budgets

---

[← Previous: Dependency Management](dependency-management.md) | [Next: Testing →](testing.md)

[Back to Main](../README.md)
