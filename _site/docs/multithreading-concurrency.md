# Multithreading & Concurrency

[← Back to Main](../README.md) | [Previous: Networking](networking.md) | [Next: Architecture & Design Patterns →](architecture-design-patterns.md)

## Table of Contents
- [GCD (Grand Central Dispatch)](#gcd-grand-central-dispatch)
- [OperationQueue](#operationqueue)
- [Async/Await in Swift](#asyncawait-in-swift)
- [Concurrency Best Practices](#concurrency-best-practices)
- [Avoiding Race Conditions & Deadlocks](#avoiding-race-conditions--deadlocks)

---

## GCD (Grand Central Dispatch)

GCD is Apple's low-level API for managing concurrent operations.

### Dispatch Queues

```swift
import Foundation

// Main Queue (UI updates)
DispatchQueue.main.async {
    // Update UI
    print("Running on main thread")
}

// Global Queue (background work)
DispatchQueue.global().async {
    // Perform background task
    print("Running on background thread")
}

// Global Queue with QoS (Quality of Service)
DispatchQueue.global(qos: .userInitiated).async {
    // High priority background task
}

DispatchQueue.global(qos: .utility).async {
    // Medium priority background task
}

DispatchQueue.global(qos: .background).async {
    // Low priority background task
}
```

### Quality of Service Levels

```swift
// User Interactive - Highest priority (UI updates, animations)
DispatchQueue.global(qos: .userInteractive).async {
    // Immediate user interaction
}

// User Initiated - High priority (user-requested tasks)
DispatchQueue.global(qos: .userInitiated).async {
    // User requested, needs quick response
}

// Default - Standard priority
DispatchQueue.global(qos: .default).async {
    // Default work
}

// Utility - Low priority (long-running tasks)
DispatchQueue.global(qos: .utility).async {
    // Downloads, imports
}

// Background - Lowest priority (maintenance)
DispatchQueue.global(qos: .background).async {
    // Indexing, cleanup
}
```

### Custom Serial Queue

```swift
class DataManager {
    private let serialQueue = DispatchQueue(label: "com.app.dataManager")
    private var data: [String] = []
    
    func addItem(_ item: String) {
        serialQueue.async {
            self.data.append(item)
            print("Added: \(item)")
        }
    }
    
    func getItems(completion: @escaping ([String]) -> Void) {
        serialQueue.async {
            completion(self.data)
        }
    }
}
```

### Custom Concurrent Queue

```swift
class ImageProcessor {
    private let concurrentQueue = DispatchQueue(
        label: "com.app.imageProcessor",
        attributes: .concurrent
    )
    
    func processImages(_ images: [UIImage], completion: @escaping ([UIImage]) -> Void) {
        var processedImages: [UIImage] = []
        let group = DispatchGroup()
        
        for image in images {
            group.enter()
            concurrentQueue.async {
                let processed = self.applyFilter(to: image)
                processedImages.append(processed)
                group.leave()
            }
        }
        
        group.notify(queue: .main) {
            completion(processedImages)
        }
    }
    
    private func applyFilter(to image: UIImage) -> UIImage {
        // Apply filter
        return image
    }
}
```

### Dispatch Groups

```swift
func downloadMultipleFiles() {
    let group = DispatchGroup()
    let urls = [
        URL(string: "https://example.com/file1.jpg")!,
        URL(string: "https://example.com/file2.jpg")!,
        URL(string: "https://example.com/file3.jpg")!
    ]
    
    for url in urls {
        group.enter()
        
        URLSession.shared.dataTask(with: url) { data, response, error in
            defer { group.leave() }
            
            if let data = data {
                print("Downloaded \(data.count) bytes from \(url)")
            }
        }.resume()
    }
    
    group.notify(queue: .main) {
        print("All downloads completed")
    }
    
    // Wait synchronously (blocks current thread)
    // group.wait()
}
```

### Dispatch Barriers

```swift
class ThreadSafeArray<T> {
    private var array: [T] = []
    private let concurrentQueue = DispatchQueue(
        label: "com.app.threadSafeArray",
        attributes: .concurrent
    )
    
    func append(_ element: T) {
        concurrentQueue.async(flags: .barrier) {
            self.array.append(element)
        }
    }
    
    func get(at index: Int) -> T? {
        var result: T?
        
        concurrentQueue.sync {
            guard index < array.count else { return }
            result = array[index]
        }
        
        return result
    }
    
    func getAll() -> [T] {
        var result: [T] = []
        
        concurrentQueue.sync {
            result = array
        }
        
        return result
    }
}
```

### Dispatch Semaphore

```swift
class LimitedConcurrencyManager {
    private let semaphore: DispatchSemaphore
    
    init(maxConcurrency: Int) {
        semaphore = DispatchSemaphore(value: maxConcurrency)
    }
    
    func performTask(_ task: @escaping () -> Void) {
        DispatchQueue.global().async {
            self.semaphore.wait()
            defer { self.semaphore.signal() }
            
            task()
        }
    }
}

// Usage: Limit to 3 concurrent network requests
let manager = LimitedConcurrencyManager(maxConcurrency: 3)

for i in 1...10 {
    manager.performTask {
        print("Task \(i) started")
        Thread.sleep(forTimeInterval: 2)
        print("Task \(i) completed")
    }
}
```

### Dispatch Work Item

```swift
class SearchViewModel {
    private var searchWorkItem: DispatchWorkItem?
    
    func search(query: String) {
        // Cancel previous search
        searchWorkItem?.cancel()
        
        let workItem = DispatchWorkItem { [weak self] in
            // Perform search
            self?.performSearch(query: query)
        }
        
        searchWorkItem = workItem
        
        // Debounce: Execute after delay
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5, execute: workItem)
    }
    
    private func performSearch(query: String) {
        guard !query.isEmpty else { return }
        print("Searching for: \(query)")
    }
}
```

---

## OperationQueue

Operation and OperationQueue provide object-oriented approach to concurrency.

### Basic Operation

```swift
class DownloadOperation: Operation {
    let url: URL
    var data: Data?
    
    init(url: URL) {
        self.url = url
    }
    
    override func main() {
        guard !isCancelled else { return }
        
        let semaphore = DispatchSemaphore(value: 0)
        
        URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
            defer { semaphore.signal() }
            
            guard let self = self, !self.isCancelled else { return }
            
            if let data = data {
                self.data = data
            }
        }.resume()
        
        semaphore.wait()
    }
}

// Usage
let operation = DownloadOperation(url: URL(string: "https://example.com/file.jpg")!)
operation.completionBlock = {
    if let data = operation.data {
        print("Downloaded \(data.count) bytes")
    }
}

let queue = OperationQueue()
queue.addOperation(operation)
```

### BlockOperation

```swift
let operation1 = BlockOperation {
    print("Operation 1 executing")
    Thread.sleep(forTimeInterval: 1)
}

let operation2 = BlockOperation {
    print("Operation 2 executing")
    Thread.sleep(forTimeInterval: 1)
}

let operation3 = BlockOperation {
    print("Operation 3 executing")
}

// Dependencies
operation2.addDependency(operation1) // operation2 waits for operation1
operation3.addDependency(operation2) // operation3 waits for operation2

let queue = OperationQueue()
queue.addOperations([operation1, operation2, operation3], waitUntilFinished: false)
```

### Asynchronous Operation

```swift
class AsyncOperation: Operation {
    enum State: String {
        case ready, executing, finished
        
        fileprivate var keyPath: String {
            return "is\(rawValue.capitalized)"
        }
    }
    
    var state = State.ready {
        willSet {
            willChangeValue(forKey: newValue.keyPath)
            willChangeValue(forKey: state.keyPath)
        }
        didSet {
            didChangeValue(forKey: oldValue.keyPath)
            didChangeValue(forKey: state.keyPath)
        }
    }
    
    override var isReady: Bool {
        return super.isReady && state == .ready
    }
    
    override var isExecuting: Bool {
        return state == .executing
    }
    
    override var isFinished: Bool {
        return state == .finished
    }
    
    override var isAsynchronous: Bool {
        return true
    }
    
    override func start() {
        if isCancelled {
            state = .finished
            return
        }
        
        main()
        state = .executing
    }
    
    override func cancel() {
        super.cancel()
        state = .finished
    }
}

class DataFetchOperation: AsyncOperation {
    let url: URL
    var result: Data?
    
    init(url: URL) {
        self.url = url
    }
    
    override func main() {
        URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
            guard let self = self else { return }
            
            self.result = data
            self.state = .finished
        }.resume()
    }
}
```

### OperationQueue Configuration

```swift
class TaskManager {
    private let operationQueue: OperationQueue
    
    init() {
        operationQueue = OperationQueue()
        operationQueue.maxConcurrentOperationCount = 3
        operationQueue.qualityOfService = .userInitiated
        operationQueue.name = "com.app.taskQueue"
    }
    
    func addTask(_ task: @escaping () -> Void) {
        operationQueue.addOperation(task)
    }
    
    func cancelAllTasks() {
        operationQueue.cancelAllOperations()
    }
    
    func waitForAllTasks() {
        operationQueue.waitUntilAllOperationsAreFinished()
    }
}
```

---

## Async/Await in Swift

Modern concurrency with async/await (Swift 5.5+, iOS 13+).

### Basic Async Functions

```swift
func fetchData() async throws -> Data {
    let url = URL(string: "https://api.example.com/data")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

func fetchUser(id: Int) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// Usage
Task {
    do {
        let user = try await fetchUser(id: 1)
        print("User: \(user.name)")
    } catch {
        print("Error: \(error)")
    }
}
```

### Sequential vs Concurrent

```swift
// Sequential (one after another)
func fetchDataSequentially() async throws {
    let user = try await fetchUser(id: 1)
    let posts = try await fetchPosts(for: user.id)
    let comments = try await fetchComments(for: posts[0].id)
    
    print("Done")
}

// Concurrent (parallel execution)
func fetchDataConcurrently() async throws {
    async let user = fetchUser(id: 1)
    async let posts = fetchPosts(for: 1)
    async let profile = fetchProfile(for: 1)
    
    let (userData, postsData, profileData) = try await (user, posts, profile)
    print("All data fetched")
}

func fetchPosts(for userId: Int) async throws -> [Post] {
    // Fetch posts
    return []
}

func fetchProfile(for userId: Int) async throws -> Profile {
    // Fetch profile
    return Profile(bio: "")
}

struct Post: Codable {
    let id: Int
    let title: String
}

struct Profile: Codable {
    let bio: String
}
```

### Task Groups

```swift
func fetchMultipleUsers(ids: [Int]) async throws -> [User] {
    try await withThrowingTaskGroup(of: User.self) { group in
        for id in ids {
            group.addTask {
                try await fetchUser(id: id)
            }
        }
        
        var users: [User] = []
        for try await user in group {
            users.append(user)
        }
        
        return users
    }
}

// Non-throwing version
func downloadImages(urls: [URL]) async -> [UIImage] {
    await withTaskGroup(of: UIImage?.self) { group in
        for url in urls {
            group.addTask {
                await downloadImage(from: url)
            }
        }
        
        var images: [UIImage] = []
        for await image in group {
            if let image = image {
                images.append(image)
            }
        }
        
        return images
    }
}

func downloadImage(from url: URL) async -> UIImage? {
    // Download image
    return nil
}
```

### Actors

Actors protect shared mutable state.

```swift
actor Counter {
    private var value = 0
    
    func increment() {
        value += 1
    }
    
    func getValue() -> Int {
        return value
    }
}

// Usage
Task {
    let counter = Counter()
    
    await counter.increment()
    await counter.increment()
    
    let value = await counter.getValue()
    print("Counter: \(value)")
}

// Thread-safe cache
actor ImageCache {
    private var images: [URL: UIImage] = [:]
    
    func image(for url: URL) -> UIImage? {
        return images[url]
    }
    
    func cache(_ image: UIImage, for url: URL) {
        images[url] = image
    }
    
    func clear() {
        images.removeAll()
    }
}
```

### MainActor

Ensure code runs on main thread.

```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    
    func loadUsers() async {
        isLoading = true
        defer { isLoading = false }
        
        do {
            let users = try await fetchMultipleUsers(ids: [1, 2, 3])
            self.users = users // Automatically on main thread
        } catch {
            print("Error: \(error)")
        }
    }
}

// Specific function on main thread
class DataService {
    @MainActor
    func updateUI(with data: Data) {
        // Guaranteed to run on main thread
    }
    
    func fetchData() async {
        let data = Data()
        
        // Call main actor function
        await updateUI(with: data)
    }
}
```

---

## Concurrency Best Practices

### Avoid Blocking Main Thread

```swift
// BAD: Blocks UI
func loadDataBad() {
    let url = URL(string: "https://api.example.com/data")!
    let data = try? Data(contentsOf: url) // Blocks main thread
    updateUI(with: data)
}

// GOOD: Async execution
func loadDataGood() {
    Task {
        let url = URL(string: "https://api.example.com/data")!
        let (data, _) = try await URLSession.shared.data(from: url)
        
        await MainActor.run {
            updateUI(with: data)
        }
    }
}

func updateUI(with data: Data?) {
    // Update UI
}
```

### Cancellation

```swift
class TaskViewModel {
    private var currentTask: Task<Void, Never>?
    
    func loadData() {
        // Cancel previous task
        currentTask?.cancel()
        
        currentTask = Task {
            for i in 1...10 {
                // Check for cancellation
                guard !Task.isCancelled else {
                    print("Task cancelled")
                    return
                }
                
                try? await Task.sleep(nanoseconds: 1_000_000_000)
                print("Processing \(i)")
            }
        }
    }
    
    func cancelLoading() {
        currentTask?.cancel()
    }
}
```

### Error Handling

```swift
enum DataError: Error {
    case networkError
    case decodingError
    case cancelled
}

class DataLoader {
    func loadData() async throws -> [User] {
        guard !Task.isCancelled else {
            throw DataError.cancelled
        }
        
        do {
            let url = URL(string: "https://api.example.com/users")!
            let (data, _) = try await URLSession.shared.data(from: url)
            
            return try JSONDecoder().decode([User].self, from: data)
        } catch is DecodingError {
            throw DataError.decodingError
        } catch {
            throw DataError.networkError
        }
    }
}
```

---

## Avoiding Race Conditions & Deadlocks

### Race Conditions

```swift
// BAD: Race condition
class UnsafeCounter {
    private var count = 0
    
    func increment() {
        DispatchQueue.global().async {
            self.count += 1 // Multiple threads can modify simultaneously
        }
    }
}

// GOOD: Thread-safe with serial queue
class SafeCounter {
    private var count = 0
    private let queue = DispatchQueue(label: "com.app.counter")
    
    func increment() {
        queue.async {
            self.count += 1
        }
    }
    
    func getCount() -> Int {
        return queue.sync {
            return count
        }
    }
}

// GOOD: Thread-safe with actor
actor ActorCounter {
    private var count = 0
    
    func increment() {
        count += 1
    }
    
    func getCount() -> Int {
        return count
    }
}
```

### Avoiding Deadlocks

```swift
// BAD: Deadlock
class DeadlockExample {
    private let queue = DispatchQueue(label: "com.app.queue")
    
    func causeDeadlock() {
        queue.sync {
            // Trying to sync on same queue from within
            queue.sync { // DEADLOCK!
                print("This will never execute")
            }
        }
    }
}

// GOOD: Avoid nested sync
class NoDeadlock {
    private let queue = DispatchQueue(label: "com.app.queue")
    
    func safeExecution() {
        queue.async {
            self.performTask()
        }
    }
    
    private func performTask() {
        // Don't call queue.sync here
        print("Safe execution")
    }
}
```

### Thread-Safe Singleton

```swift
class ThreadSafeSingleton {
    static let shared = ThreadSafeSingleton() // Thread-safe in Swift
    
    private let queue = DispatchQueue(label: "com.app.singleton", attributes: .concurrent)
    private var _data: [String] = []
    
    private init() { }
    
    func append(_ item: String) {
        queue.async(flags: .barrier) {
            self._data.append(item)
        }
    }
    
    func getData() -> [String] {
        return queue.sync {
            return _data
        }
    }
}
```

### Interview Questions

**Q1: What's the difference between sync and async?**

**Answer:** 
- **sync**: Blocks current thread until task completes. Can cause deadlocks if misused.
- **async**: Returns immediately, task executes on target queue. Doesn't block current thread.

**Q2: When to use GCD vs OperationQueue?**

**Answer:**
- **GCD**: Lightweight, simple tasks, better performance
- **OperationQueue**: Complex dependencies, cancellation, KVO support, object-oriented

**Q3: How do actors prevent data races?**

**Answer:** Actors ensure that only one task can access mutable state at a time, eliminating data races through compile-time checking and runtime serialization.

---

[← Previous: Networking](networking.md) | [Next: Architecture & Design Patterns →](architecture-design-patterns.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between sync and async in GCD?

**Answer:**

**sync** (Synchronous):
- Blocks current thread until task completes
- Returns after task finishes
- Can cause deadlocks if misused
- Use for quick operations

```swift
let queue = DispatchQueue(label: "com.app.queue")

queue.sync {
    print("Task 1")
}
print("After task 1")  // Executes after task 1 completes
```

**async** (Asynchronous):
- Returns immediately
- Task executes on target queue
- Doesn't block current thread
- Preferred for most operations

```swift
queue.async {
    print("Task 1")
}
print("After task 1")  // May execute before task 1
```

**Dangerous Pattern (Deadlock):**
```swift
let queue = DispatchQueue(label: "com.app.queue")

queue.sync {
    queue.sync {  // DEADLOCK! Same queue
        print("Never executes")
    }
}
```

**Best Practice:** Use async by default, sync only when you need to wait for result.

### Q2: Explain Quality of Service (QoS) levels in GCD

**Answer:**

QoS determines task priority and system resource allocation.

**1. User Interactive** (Highest Priority):
- UI updates, animations
- User is actively waiting
- Completes quickly

```swift
DispatchQueue.global(qos: .userInteractive).async {
    // Update UI animation
}
```

**2. User Initiated:**
- User-requested tasks
- Immediate results expected
- Example: Opening document, loading email

```swift
DispatchQueue.global(qos: .userInitiated).async {
    // Load data user requested
}
```

**3. Default:**
- Standard priority
- No specific QoS assigned

**4. Utility:**
- Long-running tasks
- User aware but not waiting
- Example: Downloads, imports

```swift
DispatchQueue.global(qos: .utility).async {
    // Download large file
}
```

**5. Background** (Lowest Priority):
- Maintenance tasks
- User not aware
- Example: Cleanup, sync, indexing

```swift
DispatchQueue.global(qos: .background).async {
    // Sync data, cleanup cache
}
```

**Priority Inversion:**
System may boost priority of lower QoS tasks if higher priority tasks depend on them.

### Q3: How does async/await differ from completion handlers?

**Answer:**

**Completion Handlers (Traditional):**

```swift
// Nested callbacks (callback hell)
func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            completion(.failure(error))
            return
        }
        completion(.success(data!))
    }.resume()
}

// Usage
fetchData { result in
    switch result {
    case .success(let data):
        self.parseData(data) { result in
            // More nesting
        }
    case .failure(let error):
        print(error)
    }
}
```

**Async/Await (Modern):**

```swift
func fetchData() async throws -> Data {
    let (data, _) = try await URLSession.shared.data(from: url)
    return data
}

// Usage - Sequential, readable code
Task {
    do {
        let data = try await fetchData()
        let parsed = try await parseData(data)
        let processed = try await processData(parsed)
        print("Done")
    } catch {
        print(error)
    }
}
```

**Benefits of Async/Await:**
- Linear, readable code
- Natural error handling
- No callback hell
- Compiler-checked
- Automatic cancellation support
- Better debugging

**When to Use:**
- iOS 15+ projects: Use async/await
- Supporting older iOS: Use completion handlers
- Libraries: Provide both APIs

### Q4: What are actors and how do they prevent data races?

**Answer:**

**Actors** protect shared mutable state from concurrent access.

**Problem (Without Actors):**
```swift
class Counter {
    var count = 0
    
    func increment() {
        count += 1  // Data race! Multiple threads can access simultaneously
    }
}
```

**Solution (With Actors):**
```swift
actor Counter {
    private var count = 0
    
    func increment() {
        count += 1  // Safe! Actor ensures one task at a time
    }
    
    func getCount() -> Int {
        return count
    }
}

// Usage
Task {
    let counter = Counter()
    
    await counter.increment()  // Suspends until actor is available
    await counter.increment()
    
    let value = await counter.getCount()
    print(value)  // 2
}
```

**How Actors Work:**
- Serialize access to mutable state
- Only one task can access actor at a time
- Other tasks wait (suspend)
- Compiler enforces with await
- No locks or semaphores needed

**MainActor:**
```swift
@MainActor
class ViewModel: ObservableObject {
    @Published var data: [String] = []  // Always accessed on main thread
    
    func updateData() {
        data.append("New item")  // Guaranteed on main thread
    }
}
```

**Benefits:**
- Prevents data races at compile time
- No manual locks needed
- Cleaner code
- Better performance than locks

### Q5: When should you use GCD vs OperationQueue?**

**Answer:**

**Use GCD (Grand Central Dispatch) when:**
- Simple tasks
- Fire and forget operations
- Lower overhead
- Better performance
- Don't need cancellation
- Don't need dependencies

```swift
DispatchQueue.global().async {
    // Simple background task
    let result = heavyComputation()
    
    DispatchQueue.main.async {
        self.updateUI(result)
    }
}
```

**Use OperationQueue when:**
- Need to cancel operations
- Complex dependencies between tasks
- Need KVO on operation state
- Want to limit concurrent operations
- Object-oriented approach preferred

```swift
let queue = OperationQueue()
queue.maxConcurrentOperationCount = 3

let operation1 = BlockOperation {
    // Task 1
}

let operation2 = BlockOperation {
    // Task 2
}

operation2.addDependency(operation1)  // operation2 runs after operation1

queue.addOperations([operation1, operation2], waitUntilFinished: false)

// Cancel
operation1.cancel()
```

**Comparison:**

| Feature | GCD | OperationQueue |
|---------|-----|----------------|
| Type | C-based API | Object-oriented |
| Overhead | Lower | Higher |
| Cancellation | Manual | Built-in |
| Dependencies | Manual (DispatchGroup) | Built-in |
| KVO | No | Yes |
| Priority | QoS | Priority property |

**Best Practice:** Use GCD for simple tasks, OperationQueue for complex workflows.

### Q6: How do you avoid race conditions in Swift?**

**Answer:**

**Race Condition:** Multiple threads accessing shared data simultaneously.

**1. Use Serial Queue:**
```swift
class ThreadSafeCounter {
    private var count = 0
    private let queue = DispatchQueue(label: "com.app.counter")
    
    func increment() {
        queue.async {
            self.count += 1  // Serialized access
        }
    }
    
    func getCount() -> Int {
        return queue.sync {
            return count
        }
    }
}
```

**2. Use Actors (Swift 5.5+):**
```swift
actor ThreadSafeCounter {
    private var count = 0
    
    func increment() {
        count += 1  // Compiler-enforced safety
    }
    
    func getCount() -> Int {
        return count
    }
}
```

**3. Use Dispatch Barriers:**
```swift
class ThreadSafeArray {
    private var array: [Int] = []
    private let queue = DispatchQueue(label: "com.app.array", attributes: .concurrent)
    
    func append(_ value: Int) {
        queue.async(flags: .barrier) {
            self.array.append(value)  // Exclusive access
        }
    }
    
    func getAll() -> [Int] {
        return queue.sync {
            return array  // Concurrent reads allowed
        }
    }
}
```

**4. Use Locks (Last Resort):**
```swift
class LockedCounter {
    private var count = 0
    private let lock = NSLock()
    
    func increment() {
        lock.lock()
        defer { lock.unlock() }
        count += 1
    }
}
```

**Best Practice:** Prefer actors (Swift 5.5+) or serial queues over manual locks.

### Q7: What's a deadlock and how do you prevent it?

**Answer:**

**Deadlock:** Two or more threads waiting for each other indefinitely.

**Common Causes:**

**1. Nested sync on same queue:**
```swift
// DEADLOCK
let queue = DispatchQueue(label: "com.app.queue")

queue.sync {
    queue.sync {  // Waiting for same queue
        print("Never executes")
    }
}
```

**2. Circular dependency:**
```swift
// DEADLOCK
let queueA = DispatchQueue(label: "queueA")
let queueB = DispatchQueue(label: "queueB")

queueA.sync {
    queueB.sync {
        queueA.sync {  // Circular wait
            print("Deadlock")
        }
    }
}
```

**Prevention:**

**1. Use async instead of sync:**
```swift
queue.async {
    // Won't block
}
```

**2. Avoid nested sync calls:**
```swift
// BAD
queue.sync {
    queue.sync { }  // Don't do this
}

// GOOD
queue.async {
    self.processData()
}
```

**3. Use serial queues for related operations:**
```swift
let serialQueue = DispatchQueue(label: "com.app.serial")

serialQueue.async {
    // Operation 1
}

serialQueue.async {
    // Operation 2 (runs after operation 1)
}
```

**4. Use DispatchGroup for synchronization:**
```swift
let group = DispatchGroup()

queue1.async(group: group) {
    // Task 1
}

queue2.async(group: group) {
    // Task 2
}

group.notify(queue: .main) {
    print("Both completed")
}
```

**Best Practice:** Minimize use of sync, use async with proper completion handlers.

---

[← Previous: Networking](networking.md) | [Next: Architecture & Design Patterns →](architecture-design-patterns.md)

[Back to Main](../README.md)
