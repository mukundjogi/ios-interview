# Object-Oriented & Protocol-Oriented Programming

[← Back to Main](../README.md) | [Previous: Swift Programming](swift-programming.md) | [Next: UIKit Development →](uikit-development.md)

## Table of Contents
- [OOP Concepts in Swift](#oop-concepts-in-swift)
- [Protocol-Oriented Programming](#protocol-oriented-programming-pop)
- [Protocols vs Abstract Classes](#protocols-vs-abstract-classes)
- [Struct vs Class vs Enum](#struct-vs-class-vs-enum)
- [Inheritance & Composition](#inheritance--composition)

---

## OOP Concepts in Swift

Object-Oriented Programming is a paradigm based on objects that contain data and code.

### 1. Encapsulation

Hiding internal implementation details and exposing only necessary interfaces.

```swift
class BankAccount {
    // Private properties (encapsulated)
    private var balance: Double = 0.0
    private var accountNumber: String
    
    // Public interface
    public private(set) var accountHolder: String
    
    init(accountNumber: String, accountHolder: String) {
        self.accountNumber = accountNumber
        self.accountHolder = accountHolder
    }
    
    // Public methods to interact with private data
    func deposit(amount: Double) {
        guard amount > 0 else {
            print("Invalid amount")
            return
        }
        balance += amount
        print("Deposited \(amount). New balance: \(balance)")
    }
    
    func withdraw(amount: Double) -> Bool {
        guard amount > 0 && amount <= balance else {
            print("Invalid withdrawal amount")
            return false
        }
        balance -= amount
        print("Withdrew \(amount). New balance: \(balance)")
        return true
    }
    
    func getBalance() -> Double {
        return balance
    }
    
    // Private helper method
    private func validateTransaction(amount: Double) -> Bool {
        return amount > 0 && amount <= 10000
    }
}

let account = BankAccount(accountNumber: "1234", accountHolder: "John Doe")
account.deposit(amount: 1000)
account.withdraw(amount: 500)
print("Balance: \(account.getBalance())")
// account.balance = 10000 // Error: 'balance' is inaccessible
```

### Access Control Levels

```swift
// open - Most permissive (classes and methods can be subclassed/overridden outside module)
open class OpenClass {
    open func openMethod() { }
}

// public - Accessible from anywhere, but can't be subclassed outside module
public class PublicClass {
    public var publicProperty = "visible"
    public func publicMethod() { }
}

// internal - Default level, accessible within same module
internal class InternalClass {
    internal var property = "internal"
}

// fileprivate - Accessible within same file
fileprivate class FilePrivateClass {
    fileprivate func method() { }
}

// private - Accessible only within enclosing declaration
class SomeClass {
    private var privateVar = "private"
    
    private func privateMethod() {
        print(privateVar) // Accessible within same class
    }
}

// Getters and setters with different access levels
class Person {
    private(set) var age: Int // Public getter, private setter
    public private(set) var name: String // Explicitly public getter, private setter
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func celebrateBirthday() {
        age += 1 // Can modify within class
    }
}

let person = Person(name: "John", age: 25)
print(person.age) // Can read
// person.age = 26 // Error: Cannot assign to property
person.celebrateBirthday() // Modify through method
```

### 2. Inheritance

Creating new classes based on existing classes.

```swift
// Base class
class Vehicle {
    var brand: String
    var year: Int
    var currentSpeed: Double = 0.0
    
    init(brand: String, year: Int) {
        self.brand = brand
        self.year = year
    }
    
    func start() {
        print("\(brand) is starting...")
    }
    
    func stop() {
        currentSpeed = 0.0
        print("\(brand) has stopped")
    }
    
    func accelerate(by speed: Double) {
        currentSpeed += speed
        print("Current speed: \(currentSpeed) km/h")
    }
    
    func description() -> String {
        return "\(year) \(brand)"
    }
}

// Derived class
class Car: Vehicle {
    var numberOfDoors: Int
    var transmission: String
    
    init(brand: String, year: Int, numberOfDoors: Int, transmission: String) {
        self.numberOfDoors = numberOfDoors
        self.transmission = transmission
        super.init(brand: brand, year: year)
    }
    
    // Override method
    override func start() {
        print("Checking seatbelts...")
        super.start() // Call parent implementation
        print("Car is ready to drive")
    }
    
    // New method specific to Car
    func openTrunk() {
        print("Trunk opened")
    }
    
    override func description() -> String {
        return "\(super.description()) - \(numberOfDoors) doors"
    }
}

class ElectricCar: Car {
    var batteryLevel: Double = 100.0
    var range: Double
    
    init(brand: String, year: Int, numberOfDoors: Int, range: Double) {
        self.range = range
        super.init(brand: brand, year: year, numberOfDoors: numberOfDoors, transmission: "Automatic")
    }
    
    override func accelerate(by speed: Double) {
        guard batteryLevel > 10 else {
            print("Battery too low")
            return
        }
        super.accelerate(by: speed)
        batteryLevel -= speed * 0.1
        print("Battery level: \(batteryLevel)%")
    }
    
    func charge() {
        batteryLevel = 100.0
        print("Fully charged")
    }
}

// Usage
let myCar = Car(brand: "Toyota", year: 2020, numberOfDoors: 4, transmission: "Manual")
myCar.start()
myCar.accelerate(by: 50)

let tesla = ElectricCar(brand: "Tesla", year: 2023, numberOfDoors: 4, range: 500)
tesla.start()
tesla.accelerate(by: 60)
tesla.charge()

// Type checking and casting
if myCar is Vehicle {
    print("myCar is a Vehicle")
}

if let electric = tesla as? ElectricCar {
    electric.charge()
}
```

### Final Classes and Methods

```swift
// final - prevents inheritance
final class FinalClass {
    func method() { }
}

// class SubClass: FinalClass { } // Error: Cannot inherit from final class

class BaseClass {
    // final method - cannot be overridden
    final func cannotOverride() {
        print("This method cannot be overridden")
    }
    
    func canOverride() {
        print("This can be overridden")
    }
}

class DerivedClass: BaseClass {
    // override func cannotOverride() { } // Error: Cannot override final method
    
    override func canOverride() {
        print("Overridden")
    }
}
```

### 3. Polymorphism

Same interface, different implementations.

```swift
// Method overloading (compile-time polymorphism)
class Calculator {
    func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
    
    func add(_ a: Double, _ b: Double) -> Double {
        return a + b
    }
    
    func add(_ numbers: [Int]) -> Int {
        return numbers.reduce(0, +)
    }
}

let calc = Calculator()
calc.add(5, 3) // 8
calc.add(5.5, 3.2) // 8.7
calc.add([1, 2, 3, 4, 5]) // 15

// Method overriding (runtime polymorphism)
class Shape {
    func area() -> Double {
        return 0.0
    }
    
    func draw() {
        print("Drawing shape")
    }
}

class Circle: Shape {
    var radius: Double
    
    init(radius: Double) {
        self.radius = radius
    }
    
    override func area() -> Double {
        return Double.pi * radius * radius
    }
    
    override func draw() {
        print("Drawing circle with radius \(radius)")
    }
}

class Rectangle: Shape {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    override func area() -> Double {
        return width * height
    }
    
    override func draw() {
        print("Drawing rectangle \(width) x \(height)")
    }
}

// Polymorphic behavior
let shapes: [Shape] = [
    Circle(radius: 5),
    Rectangle(width: 10, height: 20),
    Circle(radius: 3)
]

for shape in shapes {
    shape.draw() // Calls appropriate overridden method
    print("Area: \(shape.area())")
}
```

### 4. Abstraction

Hiding complex implementation details and showing only essential features.

```swift
// Abstract-like class (using protocol)
protocol PaymentMethod {
    func processPayment(amount: Double) -> Bool
    func refund(amount: Double) -> Bool
}

class CreditCardPayment: PaymentMethod {
    private let cardNumber: String
    private let cvv: String
    
    init(cardNumber: String, cvv: String) {
        self.cardNumber = cardNumber
        self.cvv = cvv
    }
    
    func processPayment(amount: Double) -> Bool {
        print("Processing credit card payment of $\(amount)")
        // Complex credit card processing logic hidden
        return validateCard() && chargeCard(amount: amount)
    }
    
    func refund(amount: Double) -> Bool {
        print("Refunding $\(amount) to credit card")
        return true
    }
    
    private func validateCard() -> Bool {
        return cardNumber.count == 16 && cvv.count == 3
    }
    
    private func chargeCard(amount: Double) -> Bool {
        // Complex logic hidden
        return true
    }
}

class PayPalPayment: PaymentMethod {
    private let email: String
    
    init(email: String) {
        self.email = email
    }
    
    func processPayment(amount: Double) -> Bool {
        print("Processing PayPal payment of $\(amount)")
        return authenticateUser() && transferFunds(amount: amount)
    }
    
    func refund(amount: Double) -> Bool {
        print("Refunding $\(amount) to PayPal account")
        return true
    }
    
    private func authenticateUser() -> Bool {
        return email.contains("@")
    }
    
    private func transferFunds(amount: Double) -> Bool {
        return true
    }
}

// High-level usage - complexity abstracted away
class CheckoutService {
    func checkout(paymentMethod: PaymentMethod, amount: Double) {
        if paymentMethod.processPayment(amount: amount) {
            print("Payment successful")
        } else {
            print("Payment failed")
        }
    }
}

let checkout = CheckoutService()
let creditCard = CreditCardPayment(cardNumber: "1234567890123456", cvv: "123")
checkout.checkout(paymentMethod: creditCard, amount: 99.99)

let paypal = PayPalPayment(email: "user@example.com")
checkout.checkout(paymentMethod: paypal, amount: 49.99)
```

---

## Protocol-Oriented Programming (POP)

Swift emphasizes protocol-oriented programming for more flexible and reusable code.

### Why POP?

**Problems with OOP:**
- Single inheritance limitation
- Tight coupling
- Difficult to test
- Heavy base classes

**Benefits of POP:**
- Multiple protocol conformance
- Value types (struct/enum) can use protocols
- Composition over inheritance
- Better testability
- Protocol extensions provide default implementations

### Basic Protocols

```swift
protocol Drivable {
    var maxSpeed: Double { get }
    func drive()
    func stop()
}

protocol Electric {
    var batteryLevel: Double { get set }
    func charge()
}

// Struct conforming to protocol
struct Tesla: Drivable, Electric {
    let maxSpeed: Double = 250.0
    var batteryLevel: Double = 100.0
    
    func drive() {
        print("Tesla is driving silently")
    }
    
    func stop() {
        print("Tesla stopped")
    }
    
    func charge() {
        batteryLevel = 100.0
        print("Tesla charged")
    }
}

// Class conforming to protocol
class GasCar: Drivable {
    let maxSpeed: Double = 200.0
    var fuelLevel: Double = 50.0
    
    func drive() {
        print("Gas car is driving")
        fuelLevel -= 5
    }
    
    func stop() {
        print("Gas car stopped")
    }
    
    func refuel() {
        fuelLevel = 100.0
    }
}
```

### Protocol Extensions

```swift
protocol Describable {
    var description: String { get }
}

// Protocol extension with default implementation
extension Describable {
    var description: String {
        return "Generic description"
    }
    
    func printDescription() {
        print(description)
    }
}

struct Product: Describable {
    let name: String
    let price: Double
    
    // Can override default implementation
    var description: String {
        return "\(name) - $\(price)"
    }
}

struct SimpleProduct: Describable {
    let name: String
    // Uses default description from protocol extension
}

let product = Product(name: "iPhone", price: 999)
product.printDescription() // "iPhone - $999"

let simple = SimpleProduct(name: "Charger")
simple.printDescription() // "Generic description"
```

### Protocol Inheritance

```swift
protocol Named {
    var name: String { get }
}

protocol Aged {
    var age: Int { get }
}

protocol Person: Named, Aged {
    var email: String { get }
}

struct Student: Person {
    let name: String
    let age: Int
    let email: String
    let studentId: String
}

let student = Student(name: "John", age: 20, email: "john@uni.edu", studentId: "S12345")
```

### Protocol Composition

```swift
protocol Drawable {
    func draw()
}

protocol Transformable {
    func transform()
}

// Function accepting multiple protocols
func render(item: Drawable & Transformable) {
    item.draw()
    item.transform()
}

struct Shape: Drawable, Transformable {
    func draw() {
        print("Drawing shape")
    }
    
    func transform() {
        print("Transforming shape")
    }
}

let shape = Shape()
render(item: shape)
```

### Associated Types

```swift
protocol Container {
    associatedtype Item
    
    var count: Int { get }
    mutating func append(_ item: Item)
    subscript(index: Int) -> Item { get }
}

struct IntStack: Container {
    // typealias Item = Int (inferred)
    
    private var items: [Int] = []
    
    var count: Int {
        return items.count
    }
    
    mutating func append(_ item: Int) {
        items.append(item)
    }
    
    subscript(index: Int) -> Int {
        return items[index]
    }
}

struct StringStack: Container {
    private var items: [String] = []
    
    var count: Int {
        return items.count
    }
    
    mutating func append(_ item: String) {
        items.append(item)
    }
    
    subscript(index: Int) -> String {
        return items[index]
    }
}

// Generic function using protocol with associated type
func printContainer<C: Container>(_ container: C) where C.Item: CustomStringConvertible {
    for i in 0..<container.count {
        print(container[i])
    }
}
```

### POP Example: Network Layer

```swift
// Define protocols
protocol NetworkRequest {
    var url: URL { get }
    var method: String { get }
    var headers: [String: String] { get }
}

protocol NetworkService {
    func execute<T: Decodable>(request: NetworkRequest, completion: @escaping (Result<T, Error>) -> Void)
}

// Default implementation
extension NetworkRequest {
    var headers: [String: String] {
        return ["Content-Type": "application/json"]
    }
}

// Concrete implementations
struct GetUserRequest: NetworkRequest {
    let userId: Int
    
    var url: URL {
        return URL(string: "https://api.example.com/users/\(userId)")!
    }
    
    var method: String {
        return "GET"
    }
}

struct CreateUserRequest: NetworkRequest {
    let user: User
    
    var url: URL {
        return URL(string: "https://api.example.com/users")!
    }
    
    var method: String {
        return "POST"
    }
}

// Network service implementation
class URLSessionNetworkService: NetworkService {
    func execute<T: Decodable>(request: NetworkRequest, completion: @escaping (Result<T, Error>) -> Void) {
        var urlRequest = URLRequest(url: request.url)
        urlRequest.httpMethod = request.method
        request.headers.forEach { urlRequest.setValue($1, forHTTPHeaderField: $0) }
        
        URLSession.shared.dataTask(with: urlRequest) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NetworkError.noData))
                return
            }
            
            do {
                let decoded = try JSONDecoder().decode(T.self, from: data)
                completion(.success(decoded))
            } catch {
                completion(.failure(error))
            }
        }.resume()
    }
}

enum NetworkError: Error {
    case noData
}

// Usage
let service: NetworkService = URLSessionNetworkService()
let request = GetUserRequest(userId: 1)

service.execute(request: request) { (result: Result<User, Error>) in
    switch result {
    case .success(let user):
        print("User: \(user.name)")
    case .failure(let error):
        print("Error: \(error)")
    }
}
```

---

## Protocols vs Abstract Classes

### Abstract Class Pattern in Swift

Swift doesn't have built-in abstract classes, but you can simulate them:

```swift
// Abstract class pattern
class AbstractVehicle {
    // Common implementation
    var brand: String
    var year: Int
    
    init(brand: String, year: Int) {
        self.brand = brand
        self.year = year
    }
    
    // Abstract method - must override
    func start() {
        fatalError("Subclasses must override start()")
    }
    
    // Concrete method
    func description() -> String {
        return "\(year) \(brand)"
    }
}

class Motorcycle: AbstractVehicle {
    override func start() {
        print("Motorcycle starting...")
    }
}

let bike = Motorcycle(brand: "Harley", year: 2023)
bike.start()
```

### Protocol Approach (Preferred in Swift)

```swift
protocol Vehicle {
    var brand: String { get }
    var year: Int { get }
    func start()
}

extension Vehicle {
    func description() -> String {
        return "\(year) \(brand)"
    }
}

struct MotorcycleStruct: Vehicle {
    let brand: String
    let year: Int
    
    func start() {
        print("Motorcycle starting...")
    }
}

class MotorcycleClass: Vehicle {
    let brand: String
    let year: Int
    
    init(brand: String, year: Int) {
        self.brand = brand
        self.year = year
    }
    
    func start() {
        print("Motorcycle starting...")
    }
}
```

### Comparison

| Feature | Abstract Class | Protocol |
|---------|---------------|----------|
| **Type Support** | Classes only | Struct, Class, Enum |
| **Inheritance** | Single | Multiple |
| **Stored Properties** | Yes | No (only computed) |
| **Initializers** | Yes | Yes (with requirements) |
| **Default Implementation** | Yes | Yes (via extensions) |
| **Memory** | Reference type | Depends on conforming type |
| **Swift Preference** | Not idiomatic | Preferred |

---

## Struct vs Class vs Enum

### Struct (Value Type)

```swift
struct Point {
    var x: Int
    var y: Int
    
    // Automatic memberwise initializer
}

var point1 = Point(x: 10, y: 20)
var point2 = point1 // Copy
point2.x = 30

print(point1.x) // 10 (unchanged)
print(point2.x) // 30

// Mutating methods
struct Counter {
    var count = 0
    
    mutating func increment() {
        count += 1
    }
}

var counter = Counter()
counter.increment()
print(counter.count) // 1

// let counter2 = Counter()
// counter2.increment() // Error: Cannot mutate immutable value
```

### Class (Reference Type)

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    // No mutating keyword needed
    func celebrateBirthday() {
        age += 1
    }
    
    deinit {
        print("\(name) deallocated")
    }
}

let person1 = Person(name: "John", age: 25)
let person2 = person1 // Same reference

person2.name = "Jane"

print(person1.name) // "Jane" (both changed)
print(person2.name) // "Jane"

// Reference comparison
if person1 === person2 {
    print("Same instance")
}
```

### Enum

```swift
// Simple enum
enum Direction {
    case north
    case south
    case east
    case west
}

var direction = Direction.north
direction = .south

// Enum with raw values
enum StatusCode: Int {
    case success = 200
    case notFound = 404
    case serverError = 500
}

let code = StatusCode.success
print(code.rawValue) // 200

if let status = StatusCode(rawValue: 404) {
    print("Status: \(status)") // notFound
}

// Enum with associated values
enum Result {
    case success(data: String)
    case failure(error: Error)
}

enum NetworkResponse {
    case success(data: Data, statusCode: Int)
    case failure(Error)
}

let response = NetworkResponse.success(data: Data(), statusCode: 200)

switch response {
case .success(let data, let statusCode):
    print("Success: \(statusCode)")
case .failure(let error):
    print("Error: \(error)")
}

// Enum with methods
enum TrafficLight {
    case red, yellow, green
    
    func duration() -> Int {
        switch self {
        case .red: return 30
        case .yellow: return 5
        case .green: return 25
        }
    }
    
    mutating func next() {
        switch self {
        case .red: self = .green
        case .yellow: self = .red
        case .green: self = .yellow
        }
    }
}

var light = TrafficLight.red
print(light.duration()) // 30
light.next()
print(light) // green

// Recursive enum
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case .number(let value):
        return value
    case .addition(let left, let right):
        return evaluate(left) + evaluate(right)
    case .multiplication(let left, let right):
        return evaluate(left) * evaluate(right)
    }
}

let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))

print(evaluate(product)) // 18
```

### When to Use What?

#### Use Struct When:

```swift
// Modeling simple data
struct User {
    let id: Int
    let name: String
    let email: String
}

// Geometric shapes
struct Size {
    var width: Double
    var height: Double
}

// Value semantics needed
struct Money {
    let amount: Decimal
    let currency: String
}

// Thread safety important
struct AppSettings {
    var theme: String
    var language: String
}
```

#### Use Class When:

```swift
// Representing identity
class Person {
    var name: String
    let dateOfBirth: Date
    
    init(name: String, dateOfBirth: Date) {
        self.name = name
        self.dateOfBirth = dateOfBirth
    }
}

// Need inheritance
class Animal {
    func makeSound() { }
}

class Dog: Animal {
    override func makeSound() {
        print("Woof")
    }
}

// Interfacing with Objective-C
class ViewController: UIViewController {
    // UIKit requires classes
}

// Shared mutable state
class ShoppingCart {
    var items: [Product] = []
    
    func addItem(_ product: Product) {
        items.append(product)
    }
}
```

#### Use Enum When:

```swift
// Fixed set of options
enum PaymentMethod {
    case creditCard
    case debit
    case paypal
    case applePay
}

// State machine
enum ConnectionState {
    case disconnected
    case connecting
    case connected(sessionId: String)
    case error(Error)
}

// Modeling errors
enum ValidationError: Error {
    case invalidEmail
    case passwordTooShort
    case usernameTaken
}

// Pattern matching
enum MediaType {
    case image(url: URL)
    case video(url: URL, duration: TimeInterval)
    case audio(url: URL, title: String)
}
```

### Comparison Table

| Feature | Struct | Class | Enum |
|---------|--------|-------|------|
| **Type** | Value | Reference | Value |
| **Inheritance** | No | Yes | No |
| **Protocols** | Yes | Yes | Yes |
| **Mutating** | Requires `mutating` | No | Requires `mutating` |
| **Deinitializer** | No | Yes | No |
| **Stored Properties** | Yes | Yes | No (associated values instead) |
| **Memory** | Stack (usually) | Heap | Stack (usually) |
| **Thread Safety** | Safe by default | Needs synchronization | Safe by default |
| **Copy Behavior** | Deep copy | Reference copy | Deep copy |

---

## Inheritance & Composition

### Inheritance

Inheritance creates an "is-a" relationship.

```swift
// Traditional inheritance
class Animal {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    func makeSound() {
        print("Some sound")
    }
    
    func sleep() {
        print("\(name) is sleeping")
    }
}

class Dog: Animal {
    var breed: String
    
    init(name: String, age: Int, breed: String) {
        self.breed = breed
        super.init(name: name, age: age)
    }
    
    override func makeSound() {
        print("Woof! Woof!")
    }
    
    func fetch() {
        print("\(name) is fetching")
    }
}

class Cat: Animal {
    var indoor: Bool
    
    init(name: String, age: Int, indoor: Bool) {
        self.indoor = indoor
        super.init(name: name, age: age)
    }
    
    override func makeSound() {
        print("Meow!")
    }
    
    func scratch() {
        print("\(name) is scratching")
    }
}

let dog = Dog(name: "Buddy", age: 3, breed: "Golden Retriever")
dog.makeSound() // "Woof! Woof!"
dog.sleep() // "Buddy is sleeping"
dog.fetch() // "Buddy is fetching"
```

### Problems with Inheritance

```swift
// Problem 1: Inflexible hierarchy
class Bird: Animal {
    override func makeSound() {
        print("Chirp!")
    }
    
    func fly() {
        print("\(name) is flying")
    }
}

// But what about penguins? They can't fly!
class Penguin: Bird {
    override func fly() {
        fatalError("Penguins can't fly!")
    }
}

// Problem 2: Diamond problem (not in Swift, but conceptual)
// Problem 3: Tight coupling
// Problem 4: Difficult to test
```

### Composition

Composition creates a "has-a" relationship and is more flexible.

```swift
// Protocols for capabilities
protocol Flyable {
    func fly()
}

protocol Swimmable {
    func swim()
}

protocol Runnable {
    func run()
}

// Default implementations
extension Flyable {
    func fly() {
        print("Flying in the air")
    }
}

extension Swimmable {
    func swim() {
        print("Swimming in water")
    }
}

extension Runnable {
    func run() {
        print("Running on ground")
    }
}

// Compose behaviors
struct Duck: Flyable, Swimmable, Runnable {
    let name: String
}

struct Penguin2: Swimmable, Runnable {
    let name: String
    // Can't fly - doesn't conform to Flyable
}

struct Eagle: Flyable {
    let name: String
}

let duck = Duck(name: "Donald")
duck.fly()
duck.swim()
duck.run()

let penguin = Penguin2(name: "Pingu")
penguin.swim()
penguin.run()
// penguin.fly() // Error: No fly method
```

### Composition with Dependency Injection

```swift
// Services as dependencies
protocol Logger {
    func log(_ message: String)
}

protocol DataStorage {
    func save(_ data: Data, key: String)
    func load(key: String) -> Data?
}

protocol NetworkClient {
    func fetch(url: URL, completion: @escaping (Result<Data, Error>) -> Void)
}

// Implementations
class ConsoleLogger: Logger {
    func log(_ message: String) {
        print("[LOG] \(message)")
    }
}

class UserDefaultsStorage: DataStorage {
    func save(_ data: Data, key: String) {
        UserDefaults.standard.set(data, forKey: key)
    }
    
    func load(key: String) -> Data? {
        return UserDefaults.standard.data(forKey: key)
    }
}

class URLSessionClient: NetworkClient {
    func fetch(url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
            } else if let data = data {
                completion(.success(data))
            }
        }.resume()
    }
}

// Compose services
class UserService {
    private let logger: Logger
    private let storage: DataStorage
    private let network: NetworkClient
    
    init(logger: Logger, storage: DataStorage, network: NetworkClient) {
        self.logger = logger
        self.storage = storage
        self.network = network
    }
    
    func fetchUser(id: Int) {
        logger.log("Fetching user \(id)")
        
        let url = URL(string: "https://api.example.com/users/\(id)")!
        network.fetch(url: url) { [weak self] result in
            switch result {
            case .success(let data):
                self?.storage.save(data, key: "user_\(id)")
                self?.logger.log("User \(id) saved")
            case .failure(let error):
                self?.logger.log("Error: \(error)")
            }
        }
    }
}

// Usage with dependency injection
let userService = UserService(
    logger: ConsoleLogger(),
    storage: UserDefaultsStorage(),
    network: URLSessionClient()
)

// Easy to test with mock dependencies
class MockLogger: Logger {
    var logs: [String] = []
    func log(_ message: String) {
        logs.append(message)
    }
}

class MockStorage: DataStorage {
    var storage: [String: Data] = [:]
    func save(_ data: Data, key: String) {
        storage[key] = data
    }
    func load(key: String) -> Data? {
        return storage[key]
    }
}

let testService = UserService(
    logger: MockLogger(),
    storage: MockStorage(),
    network: URLSessionClient()
)
```

### Composition vs Inheritance

```swift
// BAD: Deep inheritance hierarchy
class Vehicle2 { }
class LandVehicle: Vehicle2 { }
class Car2: LandVehicle { }
class ElectricCar2: Car2 { }
class TeslaModelS: ElectricCar2 { }

// GOOD: Composition with protocols
protocol Drivable2 {
    func drive()
}

protocol Electric2 {
    var batteryLevel: Double { get set }
    func charge()
}

protocol Autonomous {
    func enableAutopilot()
}

struct Tesla: Drivable2, Electric2, Autonomous {
    var batteryLevel: Double
    
    func drive() {
        print("Driving Tesla")
    }
    
    func charge() {
        batteryLevel = 100
    }
    
    func enableAutopilot() {
        print("Autopilot enabled")
    }
}
```

### Practical Example: App Architecture

```swift
// Using composition for a data manager
protocol NetworkManager {
    func request(endpoint: String, completion: @escaping (Result<Data, Error>) -> Void)
}

protocol CacheManager {
    func save(data: Data, key: String)
    func load(key: String) -> Data?
}

protocol Parser {
    func parse<T: Decodable>(_ data: Data) throws -> T
}

// Concrete implementations
class DefaultNetworkManager: NetworkManager {
    func request(endpoint: String, completion: @escaping (Result<Data, Error>) -> Void) {
        // Implementation
    }
}

class DefaultCacheManager: CacheManager {
    func save(data: Data, key: String) {
        // Implementation
    }
    
    func load(key: String) -> Data? {
        // Implementation
        return nil
    }
}

class JSONParser: Parser {
    func parse<T: Decodable>(_ data: Data) throws -> T {
        return try JSONDecoder().decode(T.self, from: data)
    }
}

// Composed repository
class UserRepository {
    private let network: NetworkManager
    private let cache: CacheManager
    private let parser: Parser
    
    init(network: NetworkManager, cache: CacheManager, parser: Parser) {
        self.network = network
        self.cache = cache
        self.parser = parser
    }
    
    func getUser(id: Int, completion: @escaping (Result<User, Error>) -> Void) {
        let cacheKey = "user_\(id)"
        
        // Try cache first
        if let cachedData = cache.load(key: cacheKey) {
            do {
                let user: User = try parser.parse(cachedData)
                completion(.success(user))
                return
            } catch {
                // Continue to network request
            }
        }
        
        // Fetch from network
        network.request(endpoint: "/users/\(id)") { [weak self] result in
            switch result {
            case .success(let data):
                self?.cache.save(data: data, key: cacheKey)
                do {
                    let user: User = try self!.parser.parse(data)
                    completion(.success(user))
                } catch {
                    completion(.failure(error))
                }
            case .failure(let error):
                completion(.failure(error))
            }
        }
    }
}

// Easy to test and swap implementations
let repository = UserRepository(
    network: DefaultNetworkManager(),
    cache: DefaultCacheManager(),
    parser: JSONParser()
)
```

### Interview Questions

**Q1: Should I use inheritance or composition?**

**Answer:** Prefer composition over inheritance. Composition is more flexible, easier to test, and doesn't create tight coupling. Use inheritance only when there's a clear "is-a" relationship and you need to override behavior.

**Q2: When should I use a struct vs a class?**

**Answer:** Use struct by default for data models and value types. Use class when you need:
- Reference semantics
- Inheritance
- Deinitializer
- Objective-C interoperability

**Q3: What's the difference between OOP and POP?**

**Answer:** OOP focuses on class hierarchies and inheritance. POP focuses on protocols and composition. POP is more flexible, supports value types, allows multiple protocol conformance, and is the preferred approach in Swift.

---

[← Previous: Swift Programming](swift-programming.md) | [Next: UIKit Development →](uikit-development.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: When should you use a struct vs a class in Swift?

**Answer:**

**Use Struct when:**
- Modeling simple data structures (User, Product, Coordinate)
- Value semantics make sense (each instance should be independent)
- No inheritance needed
- Thread safety is important
- Small data that's frequently copied

```swift
struct User {
    let id: Int
    let name: String
    var email: String
}
```

**Use Class when:**
- Modeling identity (Person, ViewController)
- Need inheritance
- Sharing mutable state across app
- Working with Objective-C APIs
- Need deinitializer
- Reference semantics required

```swift
class ViewController: UIViewController {
    // UIKit requires classes
}
```

**Performance Consideration:**
- Structs are generally faster for small data
- Classes better for large, complex objects
- Swift optimizes struct copying (copy-on-write)

### Q2: Explain Protocol-Oriented Programming and why Apple recommends it

**Answer:**

**Protocol-Oriented Programming (POP)** is Apple's recommended approach for Swift development, emphasizing protocols over class inheritance.

**Why POP Over OOP:**

1. **Value types can conform to protocols** (structs, enums)
2. **Multiple protocol conformance** vs single inheritance
3. **Composition over inheritance**
4. **Testability** - easy to create test doubles
5. **No fragile base class problem**

**Example:**

```swift
// Traditional OOP - Limited
class Animal {
    func makeSound() { }
}
class Dog: Animal {
    override func makeSound() { print("Woof") }
}

// POP - Flexible
protocol Flyable {
    func fly()
}
protocol Swimmable {
    func swim()
}

struct Duck: Flyable, Swimmable {
    func fly() { print("Flying") }
    func swim() { print("Swimming") }
}

struct Airplane: Flyable {
    func fly() { print("Flying high") }
}
```

**Protocol Extensions (Default Implementation):**

```swift
protocol Greetable {
    var name: String { get }
    func greet()
}

extension Greetable {
    func greet() {
        print("Hello, \(name)")
    }
}

struct Person: Greetable {
    let name: String
    // greet() provided by protocol extension
}
```

**Benefits:**
- More flexible than inheritance
- Better code reuse
- Easier testing
- Works with value types
- Cleaner dependencies

### Q3: What's the difference between a protocol and an abstract class?

**Answer:**

Swift doesn't have abstract classes, but we can simulate them with protocols.

**Protocols:**
- Can be adopted by struct, class, enum
- Multiple conformance allowed
- No stored properties (only computed)
- Default implementation via extensions
- No initialization logic

```swift
protocol Vehicle {
    var maxSpeed: Double { get }
    func start()
}

extension Vehicle {
    func description() -> String {
        return "Max speed: \(maxSpeed)"
    }
}
```

**Abstract Class Pattern (Not native to Swift):**

```swift
class AbstractVehicle {
    var brand: String
    
    init(brand: String) {
        self.brand = brand
    }
    
    func start() {
        fatalError("Must override")
    }
}

class Car: AbstractVehicle {
    override func start() {
        print("Car starting")
    }
}
```

**Why Protocols are Better in Swift:**
- Work with value types
- Multiple protocol conformance
- More flexible
- Better performance
- True to Swift's design philosophy

### Q4: Explain the SOLID principles with iOS examples

**Answer:**

**S - Single Responsibility Principle**
Each class should have one reason to change.

```swift
// BAD
class UserManager {
    func fetchUser() { }
    func saveUser() { }
    func sendEmail() { }  // Not user management!
}

// GOOD
class UserRepository {
    func fetchUser() { }
    func saveUser() { }
}

class EmailService {
    func sendEmail() { }
}
```

**O - Open/Closed Principle**
Open for extension, closed for modification.

```swift
protocol PaymentMethod {
    func processPayment(amount: Double)
}

class CreditCard: PaymentMethod {
    func processPayment(amount: Double) {
        // Process credit card
    }
}

class PayPal: PaymentMethod {
    func processPayment(amount: Double) {
        // Process PayPal
    }
}

// Can add new payment methods without modifying existing code
```

**L - Liskov Substitution Principle**
Subtypes must be substitutable for base types.

```swift
class Bird {
    func eat() { }
}

class FlyingBird: Bird {
    func fly() { }
}

class Sparrow: FlyingBird {
    // Can fly
}

class Penguin: Bird {
    // Can't fly, doesn't inherit fly()
}
```

**I - Interface Segregation Principle**
Clients shouldn't depend on interfaces they don't use.

```swift
// BAD - Fat interface
protocol Worker {
    func work()
    func eat()
    func sleep()
}

// GOOD - Segregated interfaces
protocol Workable {
    func work()
}

protocol Eatable {
    func eat()
}

class Human: Workable, Eatable {
    func work() { }
    func eat() { }
}

class Robot: Workable {
    func work() { }
}
```

**D - Dependency Inversion Principle**
Depend on abstractions, not concretions.

```swift
// BAD
class UserService {
    let api = NetworkAPI()  // Depends on concrete class
}

// GOOD
protocol NetworkService {
    func request(url: String) -> Data
}

class UserService {
    let network: NetworkService  // Depends on protocol
    
    init(network: NetworkService) {
        self.network = network
    }
}
```

### Q5: What are the main differences between inheritance and composition?

**Answer:**

**Inheritance (is-a relationship):**

```swift
class Animal {
    func eat() { }
}

class Dog: Animal {
    func bark() { }
}
```

**Pros:**
- Code reuse
- Polymorphism
- Clear hierarchy

**Cons:**
- Tight coupling
- Fragile base class
- Limited to single inheritance
- Hard to test

**Composition (has-a relationship):**

```swift
protocol Logger {
    func log(_ message: String)
}

protocol NetworkService {
    func fetch(url: URL)
}

class UserService {
    let logger: Logger
    let network: NetworkService
    
    init(logger: Logger, network: NetworkService) {
        self.logger = logger
        self.network = network
    }
}
```

**Pros:**
- Loose coupling
- Flexible
- Easy to test (inject mocks)
- Multiple "has-a" relationships
- Can change behavior at runtime

**Cons:**
- More boilerplate code
- Can be more complex

**Best Practice:** Prefer composition over inheritance in Swift. Use protocols and dependency injection for flexibility.

### Q6: How do you implement the Delegate pattern in Swift?

**Answer:**

The Delegate pattern allows one object to communicate back to another without creating tight coupling.

**Step 1: Define Protocol**

```swift
protocol UserProfileDelegate: AnyObject {
    func didUpdateProfile(_ profile: UserProfile)
    func didFailWithError(_ error: Error)
}
```

**Step 2: Add Delegate Property (weak)**

```swift
class UserProfileViewController: UIViewController {
    weak var delegate: UserProfileDelegate?
    
    func saveProfile() {
        let profile = UserProfile(name: "John", age: 25)
        delegate?.didUpdateProfile(profile)
        dismiss(animated: true)
    }
}
```

**Step 3: Implement Delegate**

```swift
class HomeViewController: UIViewController, UserProfileDelegate {
    func showProfileEditor() {
        let profileVC = UserProfileViewController()
        profileVC.delegate = self
        present(profileVC, animated: true)
    }
    
    func didUpdateProfile(_ profile: UserProfile) {
        print("Profile updated: \(profile.name)")
        refreshUI()
    }
    
    func didFailWithError(_ error: Error) {
        showError(error)
    }
}
```

**Key Points:**
- Protocol must inherit `AnyObject` (class-only protocol)
- Delegate property must be `weak` to prevent retain cycles
- Delegate is optional (use optional chaining)
- One-to-one communication

**When to Use:**
- Backward communication (child to parent)
- Responding to events
- Customizing behavior
- UIKit patterns (UITableViewDelegate, UITextFieldDelegate)

### Q7: Explain the difference between enum with associated values vs raw values

**Answer:**

**Raw Values** - Same type for all cases:

```swift
enum StatusCode: Int {
    case success = 200
    case notFound = 404
    case serverError = 500
}

let code = StatusCode.success
print(code.rawValue)  // 200

if let status = StatusCode(rawValue: 404) {
    print("Not found")
}
```

**Associated Values** - Different data for each case instance:

```swift
enum NetworkResponse {
    case success(data: Data, statusCode: Int)
    case failure(error: Error, statusCode: Int)
    case loading
}

let response = NetworkResponse.success(data: someData, statusCode: 200)

switch response {
case .success(let data, let code):
    print("Success: \(code)")
case .failure(let error, let code):
    print("Failed: \(error)")
case .loading:
    print("Loading...")
}
```

**Comparison:**

| Feature | Raw Values | Associated Values |
|---------|------------|-------------------|
| Type | Same for all cases | Different per instance |
| Storage | Compile-time constant | Runtime data |
| Initialization | From raw value | With associated data |
| Use Case | Fixed mappings | Flexible data |

**Practical Example - Result Type:**

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}

func fetchUser(id: Int) -> Result<User, NetworkError> {
    if let user = database.find(id) {
        return .success(user)
    } else {
        return .failure(.notFound)
    }
}
```

**Best Practice:** Use raw values for simple mappings, associated values for carrying data.

---

[← Previous: Swift Programming](swift-programming.md) | [Next: UIKit Development →](uikit-development.md)

[Back to Main](../README.md)
