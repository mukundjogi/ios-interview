# Swift Programming Language

[← Back to Main](../README.md) | [Previous: iOS Basics](ios-basics.md) | [Next: OOP and POP →](oop-and-pop.md)

## Table of Contents
- [Basics of Swift](#basics-of-swift)
- [Optionals in Swift](#optionals-in-swift)
- [Closures & Higher Order Functions](#closures--higher-order-functions)
- [Error Handling in Swift](#error-handling-in-swift)
- [Extensions & Protocols](#extensions--protocols)
- [Generics in Swift](#generics-in-swift)
- [Value Type vs Reference Type](#value-type-vs-reference-type)
- [ARC (Automatic Reference Counting)](#arc-automatic-reference-counting)

---

## Basics of Swift

### Variables and Constants

```swift
// Constants - value cannot be changed
let name = "John"
let age = 25
let pi = 3.14159

// Variables - value can be changed
var score = 0
score = 100

var greeting = "Hello"
greeting = "Hi"

// Type annotation (explicit type)
let country: String = "USA"
var count: Int = 0
var price: Double = 99.99

// Type inference (Swift infers the type)
let city = "New York" // Inferred as String
var items = 10 // Inferred as Int
```

**Best Practice:** Use `let` by default, only use `var` when you need to mutate the value.

### Data Types

#### Basic Types

```swift
// Integer
let smallNumber: Int8 = 127 // -128 to 127
let regularNumber: Int = 1000 // Platform-dependent (Int32 or Int64)
let bigNumber: Int64 = 9_223_372_036_854_775_807

let unsignedNumber: UInt = 100 // Only positive values

// Floating Point
let float: Float = 3.14 // 32-bit
let double: Double = 3.14159265359 // 64-bit (preferred)

// Boolean
let isActive: Bool = true
let isHidden: Bool = false

// String
let message: String = "Hello, Swift!"
let multiline = """
This is a
multiline string
in Swift
"""

// Character
let letter: Character = "A"
let emoji: Character = "😊"
```

#### String Operations

```swift
// String concatenation
let firstName = "John"
let lastName = "Doe"
let fullName = firstName + " " + lastName

// String interpolation (preferred)
let age = 25
let intro = "My name is \(fullName) and I'm \(age) years old"

// String properties and methods
let text = "Hello, World!"
print(text.count) // 13
print(text.uppercased()) // "HELLO, WORLD!"
print(text.lowercased()) // "hello, world!"
print(text.isEmpty) // false

// Checking prefix/suffix
print(text.hasPrefix("Hello")) // true
print(text.hasSuffix("!")) // true

// String iteration
for char in text {
    print(char)
}

// Substring
let startIndex = text.index(text.startIndex, offsetBy: 7)
let substring = text[startIndex...] // "World!"

// String contains
if text.contains("World") {
    print("Found World")
}
```

#### Collections

**Arrays**

```swift
// Array declaration
var fruits = ["Apple", "Banana", "Cherry"]
var numbers: [Int] = [1, 2, 3, 4, 5]
var emptyArray: [String] = []
var anotherEmpty = [Int]()

// Accessing elements
let first = fruits[0] // "Apple"
let last = fruits[fruits.count - 1] // "Cherry"

// Safe access
if let secondFruit = fruits.indices.contains(1) ? fruits[1] : nil {
    print(secondFruit)
}

// Array operations
fruits.append("Date")
fruits.insert("Avocado", at: 0)
fruits.remove(at: 2)
fruits.removeLast()

// Array properties
print(fruits.count) // Number of elements
print(fruits.isEmpty) // false
print(fruits.first) // Optional("Apple")
print(fruits.last) // Optional("Date")

// Iteration
for fruit in fruits {
    print(fruit)
}

for (index, fruit) in fruits.enumerated() {
    print("\(index): \(fruit)")
}

// Array methods
let uppercasedFruits = fruits.map { $0.uppercased() }
let longFruits = fruits.filter { $0.count > 5 }
let allStartWithA = fruits.allSatisfy { $0.hasPrefix("A") }
let someStartWithA = fruits.contains { $0.hasPrefix("A") }

// Sorting
let sortedFruits = fruits.sorted()
let sortedByLength = fruits.sorted { $0.count < $1.count }
```

**Dictionaries**

```swift
// Dictionary declaration
var person = ["name": "John", "city": "New York"]
var scores: [String: Int] = ["Alice": 95, "Bob": 87]
var emptyDict: [String: String] = [:]
var anotherEmptyDict = [Int: String]()

// Accessing values
let name = person["name"] // Optional("John")
let age = person["age"] // nil

// Safe access with default value
let city = person["city", default: "Unknown"]

// Updating values
person["name"] = "Jane"
person["age"] = "30"
person.updateValue("London", forKey: "city")

// Removing values
person["city"] = nil
person.removeValue(forKey: "age")

// Dictionary properties
print(person.count)
print(person.isEmpty)
print(person.keys) // ["name"]
print(person.values) // ["Jane"]

// Iteration
for (key, value) in person {
    print("\(key): \(value)")
}

for key in person.keys {
    print(key)
}
```

**Sets**

```swift
// Set declaration
var uniqueNumbers: Set<Int> = [1, 2, 3, 4, 5]
var colors: Set = ["Red", "Green", "Blue"]

// Sets automatically remove duplicates
var numbers: Set = [1, 2, 2, 3, 3, 3] // Results in {1, 2, 3}

// Set operations
uniqueNumbers.insert(6)
uniqueNumbers.remove(3)
uniqueNumbers.contains(2) // true

// Set operations
let setA: Set = [1, 2, 3, 4]
let setB: Set = [3, 4, 5, 6]

let union = setA.union(setB) // {1, 2, 3, 4, 5, 6}
let intersection = setA.intersection(setB) // {3, 4}
let difference = setA.subtracting(setB) // {1, 2}
let symmetric = setA.symmetricDifference(setB) // {1, 2, 5, 6}

// Subset checks
let subset: Set = [1, 2]
subset.isSubset(of: setA) // true
setA.isSuperset(of: subset) // true
```

### Control Flow

#### Conditionals

```swift
// If-else
let temperature = 25

if temperature > 30 {
    print("It's hot")
} else if temperature > 20 {
    print("It's warm")
} else {
    print("It's cold")
}

// Ternary operator
let message = temperature > 25 ? "Warm" : "Cool"

// Guard statement (early exit)
func greet(name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    print("Hello, \(name)")
}

// Switch statement
let fruit = "Apple"

switch fruit {
case "Apple":
    print("It's an apple")
case "Banana":
    print("It's a banana")
case "Cherry", "Strawberry":
    print("It's a berry")
default:
    print("Unknown fruit")
}

// Switch with ranges
let score = 85

switch score {
case 90...100:
    print("A")
case 80..<90:
    print("B")
case 70..<80:
    print("C")
default:
    print("F")
}

// Switch with tuples
let point = (0, 0)

switch point {
case (0, 0):
    print("Origin")
case (_, 0):
    print("On x-axis")
case (0, _):
    print("On y-axis")
case (-2...2, -2...2):
    print("Inside the box")
default:
    print("Outside the box")
}

// Switch with value binding
let anotherPoint = (2, 0)

switch anotherPoint {
case (let x, 0):
    print("On x-axis at \(x)")
case (0, let y):
    print("On y-axis at \(y)")
case let (x, y):
    print("At (\(x), \(y))")
}
```

#### Loops

```swift
// For-in loop
for i in 1...5 {
    print(i) // 1, 2, 3, 4, 5
}

for i in 1..<5 {
    print(i) // 1, 2, 3, 4
}

// Stride
for i in stride(from: 0, to: 10, by: 2) {
    print(i) // 0, 2, 4, 6, 8
}

for i in stride(from: 10, through: 0, by: -2) {
    print(i) // 10, 8, 6, 4, 2, 0
}

// While loop
var count = 0
while count < 5 {
    print(count)
    count += 1
}

// Repeat-while loop
var number = 0
repeat {
    print(number)
    number += 1
} while number < 5

// Loop control
for i in 1...10 {
    if i == 3 {
        continue // Skip this iteration
    }
    if i == 7 {
        break // Exit the loop
    }
    print(i)
}
```

### Functions

```swift
// Basic function
func greet() {
    print("Hello!")
}
greet()

// Function with parameters
func greet(name: String) {
    print("Hello, \(name)!")
}
greet(name: "John")

// Function with return value
func add(a: Int, b: Int) -> Int {
    return a + b
}
let sum = add(a: 5, b: 3)

// Multiple return values using tuple
func minMax(array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    
    return (currentMin, currentMax)
}

let bounds = minMax(array: [1, 5, 3, 9, 2])
print("Min: \(bounds.min), Max: \(bounds.max)")

// Optional return type
func findIndex(of string: String, in array: [String]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == string {
            return index
        }
    }
    return nil
}

// Default parameter values
func greet(name: String, greeting: String = "Hello") {
    print("\(greeting), \(name)!")
}
greet(name: "John") // Hello, John!
greet(name: "Jane", greeting: "Hi") // Hi, Jane!

// Variadic parameters
func average(numbers: Double...) -> Double {
    var total = 0.0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
average(numbers: 1, 2, 3, 4, 5) // 3.0

// Inout parameters (modifies original value)
func swapValues(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

var x = 5
var y = 10
swapValues(&x, &y)
print("x: \(x), y: \(y)") // x: 10, y: 5

// Function types
func addIntegers(_ a: Int, _ b: Int) -> Int {
    return a + b
}

var mathFunction: (Int, Int) -> Int = addIntegers
print(mathFunction(2, 3)) // 5

// Functions as parameters
func printResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result: \(mathFunction(a, b))")
}
printResult(addIntegers, 3, 5) // Result: 8
```

---

## Optionals in Swift

Optionals handle the absence of a value. An optional can contain either a value or `nil`.

### Why Optionals?

```swift
// Without optionals (in other languages)
// String name = null; // Can cause null pointer exceptions

// With optionals (Swift)
var name: String? = nil // Explicitly states that name can be nil
name = "John"
```

### Optional Declaration

```swift
// Optional types
var optionalString: String? = "Hello"
var optionalInt: Int? = 42
var optionalBool: Bool? = nil

// Optional from function
func findUser(id: Int) -> String? {
    if id == 1 {
        return "John"
    }
    return nil
}

let user = findUser(id: 1) // Optional("John")
let unknown = findUser(id: 99) // nil
```

### Unwrapping Optionals

#### 1. Forced Unwrapping (!)

```swift
var name: String? = "John"

// Forced unwrapping - use only when sure value exists
let unwrapped = name! // "John"

// Dangerous - will crash if nil
var noName: String? = nil
// let crash = noName! // Runtime error: Fatal error: Unexpectedly found nil
```

**Warning:** Only use forced unwrapping when you're absolutely certain the optional contains a value.

#### 2. Optional Binding (if let / guard let)

```swift
var name: String? = "John"

// if let
if let unwrappedName = name {
    print("Name is \(unwrappedName)")
} else {
    print("Name is nil")
}

// Multiple optional binding
var firstName: String? = "John"
var lastName: String? = "Doe"

if let first = firstName, let last = lastName {
    print("Full name: \(first) \(last)")
}

// guard let (early exit)
func greet(name: String?) {
    guard let name = name else {
        print("No name provided")
        return
    }
    
    // name is unwrapped and available here
    print("Hello, \(name)")
}

// guard with multiple conditions
func processUser(name: String?, age: Int?) {
    guard let name = name,
          let age = age,
          age >= 18 else {
        print("Invalid user data")
        return
    }
    
    print("\(name) is \(age) years old")
}
```

#### 3. Nil Coalescing Operator (??)

```swift
var optionalName: String? = nil

// Provide default value if nil
let name = optionalName ?? "Guest"
print(name) // "Guest"

optionalName = "John"
let actualName = optionalName ?? "Guest"
print(actualName) // "John"

// Chaining
let first: String? = nil
let second: String? = nil
let third: String? = "Third"

let result = first ?? second ?? third ?? "Default"
print(result) // "Third"
```

#### 4. Optional Chaining (?.)

```swift
class Person {
    var residence: Residence?
}

class Residence {
    var address: Address?
}

class Address {
    var street: String = "123 Main St"
}

let person = Person()

// Optional chaining - returns nil if any step is nil
let street = person.residence?.address?.street
print(street) // nil

// Set values
person.residence = Residence()
person.residence?.address = Address()

let actualStreet = person.residence?.address?.street
print(actualStreet) // Optional("123 Main St")

// Calling methods through optional chaining
class Counter {
    var count = 0
    func increment() {
        count += 1
    }
}

var counter: Counter? = Counter()
counter?.increment() // Method is called
counter?.increment()

print(counter?.count) // Optional(2)
```

#### 5. Implicitly Unwrapped Optionals (!)

```swift
// Used when optional will always have a value after initial setup
var assumedString: String! = "Hello"

// Can be used like a regular value
let implicit: String = assumedString // No unwrapping needed

// Still optional underneath
if assumedString != nil {
    print(assumedString) // Can be used without unwrapping
}

// Common use case: IBOutlets
class ViewController: UIViewController {
    @IBOutlet weak var nameLabel: UILabel! // Set by storyboard
    
    override func viewDidLoad() {
        super.viewDidLoad()
        nameLabel.text = "John" // No unwrapping needed
    }
}
```

### Optional Map and FlatMap

```swift
// map - transform optional value
let optionalNumber: Int? = 5
let doubled = optionalNumber.map { $0 * 2 }
print(doubled) // Optional(10)

let nilNumber: Int? = nil
let doubledNil = nilNumber.map { $0 * 2 }
print(doubledNil) // nil

// flatMap - transform and flatten nested optionals
func toInt(_ string: String) -> Int? {
    return Int(string)
}

let optionalString: String? = "123"
let number = optionalString.flatMap(toInt)
print(number) // Optional(123)

// Prevents Optional(Optional(value))
let nestedOptional = optionalString.map(toInt) // Optional(Optional(123))
let flatOptional = optionalString.flatMap(toInt) // Optional(123)
```

### Practical Example

```swift
struct User {
    let id: Int
    let name: String
    let email: String?
    let phoneNumber: String?
}

class UserManager {
    private var users: [Int: User] = [:]
    
    func getUser(id: Int) -> User? {
        return users[id]
    }
    
    func getUserEmail(id: Int) -> String {
        // Multiple unwrapping techniques
        
        // Option 1: Optional chaining with nil coalescing
        return getUser(id: id)?.email ?? "no-email@example.com"
        
        // Option 2: Guard statement
        guard let user = getUser(id: id),
              let email = user.email else {
            return "no-email@example.com"
        }
        return email
        
        // Option 3: If let
        if let user = getUser(id: id), let email = user.email {
            return email
        }
        return "no-email@example.com"
    }
    
    func displayContactInfo(userId: Int) {
        guard let user = getUser(id: userId) else {
            print("User not found")
            return
        }
        
        print("Name: \(user.name)")
        
        // Handle optional email
        if let email = user.email {
            print("Email: \(email)")
        } else {
            print("Email: Not provided")
        }
        
        // Handle optional phone with nil coalescing
        let phone = user.phoneNumber ?? "Not provided"
        print("Phone: \(phone)")
    }
}
```

---

## Closures & Higher Order Functions

Closures are self-contained blocks of functionality that can be passed around and used in your code.

### Closure Syntax

```swift
// Basic closure
let greeting = {
    print("Hello, World!")
}
greeting()

// Closure with parameters
let greetPerson = { (name: String) in
    print("Hello, \(name)!")
}
greetPerson("John")

// Closure with return type
let add = { (a: Int, b: Int) -> Int in
    return a + b
}
let sum = add(5, 3) // 8

// Single expression closures can omit return
let multiply = { (a: Int, b: Int) -> Int in
    a * b
}
```

### Closures as Function Parameters

```swift
func performOperation(_ a: Int, _ b: Int, operation: (Int, Int) -> Int) -> Int {
    return operation(a, b)
}

// Using with named function
func subtract(_ a: Int, _ b: Int) -> Int {
    return a - b
}
let result1 = performOperation(10, 5, operation: subtract)

// Using with closure
let result2 = performOperation(10, 5, operation: { (a: Int, b: Int) -> Int in
    return a * b
})

// Trailing closure syntax
let result3 = performOperation(10, 5) { (a: Int, b: Int) -> Int in
    return a + b
}

// Shorthand argument names
let result4 = performOperation(10, 5) { $0 + $1 }

// Operator as closure
let result5 = performOperation(10, 5, operation: +)
```

### Capturing Values

```swift
func makeIncrementer(incrementAmount: Int) -> () -> Int {
    var total = 0
    
    let incrementer: () -> Int = {
        total += incrementAmount
        return total
    }
    
    return incrementer
}

let incrementByTwo = makeIncrementer(incrementAmount: 2)
print(incrementByTwo()) // 2
print(incrementByTwo()) // 4
print(incrementByTwo()) // 6

let incrementByFive = makeIncrementer(incrementAmount: 5)
print(incrementByFive()) // 5
print(incrementByFive()) // 10
```

### Escaping Closures

```swift
class NetworkManager {
    var completionHandlers: [() -> Void] = []
    
    // @escaping - closure is called after function returns
    func fetchData(completion: @escaping (String) -> Void) {
        DispatchQueue.global().async {
            // Simulate network call
            sleep(2)
            let data = "Response data"
            
            DispatchQueue.main.async {
                completion(data)
            }
        }
    }
    
    func storeCompletion(_ handler: @escaping () -> Void) {
        completionHandlers.append(handler)
        // Closure escapes because it's stored and called later
    }
}

// Usage
let manager = NetworkManager()
manager.fetchData { data in
    print("Received: \(data)")
}
```

### Autoclosures

```swift
// @autoclosure - automatically creates closure from expression
func logIfTrue(_ condition: @autoclosure () -> Bool) {
    if condition() {
        print("Condition is true")
    }
}

// Can be called without braces
logIfTrue(2 > 1)

// Useful for delayed evaluation
func debugLog(_ message: @autoclosure () -> String) {
    #if DEBUG
    print(message())
    #endif
}

// Expensive operation only runs in debug
debugLog("Current time: \(Date())")
```

### Higher Order Functions

Functions that take other functions as parameters or return functions.

#### Map

```swift
// Transform each element
let numbers = [1, 2, 3, 4, 5]
let doubled = numbers.map { $0 * 2 }
print(doubled) // [2, 4, 6, 8, 10]

let names = ["john", "jane", "bob"]
let uppercased = names.map { $0.uppercased() }
print(uppercased) // ["JOHN", "JANE", "BOB"]

// Map with custom type
struct Person {
    let name: String
    let age: Int
}

let people = [
    Person(name: "John", age: 25),
    Person(name: "Jane", age: 30)
]

let ages = people.map { $0.age }
print(ages) // [25, 30]
```

#### Filter

```swift
// Keep elements matching condition
let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
let evenNumbers = numbers.filter { $0 % 2 == 0 }
print(evenNumbers) // [2, 4, 6, 8, 10]

let names = ["Alice", "Bob", "Charlie", "David"]
let shortNames = names.filter { $0.count <= 4 }
print(shortNames) // ["Bob"]

// Combining filter and map
let adults = people
    .filter { $0.age >= 18 }
    .map { $0.name }
```

#### Reduce

```swift
// Combine all elements into single value
let numbers = [1, 2, 3, 4, 5]
let sum = numbers.reduce(0) { $0 + $1 }
print(sum) // 15

// Shorthand
let product = numbers.reduce(1, *)
print(product) // 120

// Complex reduce
let words = ["Hello", "World", "Swift"]
let sentence = words.reduce("") { result, word in
    result.isEmpty ? word : "\(result) \(word)"
}
print(sentence) // "Hello World Swift"

// Reduce into dictionary
let names = ["Alice", "Bob", "Charlie", "Anna"]
let groupedByFirstLetter = names.reduce(into: [Character: [String]]()) { result, name in
    let firstLetter = name.first!
    result[firstLetter, default: []].append(name)
}
print(groupedByFirstLetter) // ["A": ["Alice", "Anna"], "B": ["Bob"], "C": ["Charlie"]]
```

#### CompactMap

```swift
// Map and remove nil values
let strings = ["1", "2", "three", "4", "5"]
let numbers = strings.compactMap { Int($0) }
print(numbers) // [1, 2, 4, 5]

let optionals: [Int?] = [1, nil, 3, nil, 5]
let unwrapped = optionals.compactMap { $0 }
print(unwrapped) // [1, 3, 5]
```

#### FlatMap

```swift
// Flatten nested arrays
let nestedArrays = [[1, 2, 3], [4, 5], [6, 7, 8]]
let flattened = nestedArrays.flatMap { $0 }
print(flattened) // [1, 2, 3, 4, 5, 6, 7, 8]

// FlatMap with transformation
let numbers = [1, 2, 3]
let repeated = numbers.flatMap { Array(repeating: $0, count: $0) }
print(repeated) // [1, 2, 2, 3, 3, 3]
```

#### ForEach

```swift
let numbers = [1, 2, 3, 4, 5]
numbers.forEach { print($0) }

// Note: forEach cannot use break or continue
// Use regular for-in loop if you need that
```

#### Sorted

```swift
let numbers = [3, 1, 4, 1, 5, 9, 2, 6]
let ascending = numbers.sorted()
print(ascending) // [1, 1, 2, 3, 4, 5, 6, 9]

let descending = numbers.sorted(by: >)
print(descending) // [9, 6, 5, 4, 3, 2, 1, 1]

// Custom sorting
struct Person {
    let name: String
    let age: Int
}

let people = [
    Person(name: "John", age: 25),
    Person(name: "Jane", age: 30),
    Person(name: "Bob", age: 20)
]

let sortedByAge = people.sorted { $0.age < $1.age }
let sortedByName = people.sorted { $0.name < $1.name }
```

#### Contains

```swift
let numbers = [1, 2, 3, 4, 5]
let hasThree = numbers.contains(3) // true
let hasNegative = numbers.contains { $0 < 0 } // false

let names = ["Alice", "Bob", "Charlie"]
let hasShortName = names.contains { $0.count <= 3 } // true
```

#### AllSatisfy

```swift
let numbers = [2, 4, 6, 8]
let allEven = numbers.allSatisfy { $0 % 2 == 0 } // true

let ages = [18, 21, 25, 30]
let allAdults = ages.allSatisfy { $0 >= 18 } // true
```

### Practical Example: Data Processing

```swift
struct Transaction {
    let id: String
    let amount: Double
    let category: String
    let date: Date
}

class TransactionProcessor {
    let transactions: [Transaction]
    
    init(transactions: [Transaction]) {
        self.transactions = transactions
    }
    
    // Total amount by category
    func totalByCategory(_ category: String) -> Double {
        return transactions
            .filter { $0.category == category }
            .map { $0.amount }
            .reduce(0, +)
    }
    
    // Get all unique categories
    func uniqueCategories() -> [String] {
        return Array(Set(transactions.map { $0.category }))
            .sorted()
    }
    
    // Find large transactions (>$1000)
    func largeTransactions() -> [Transaction] {
        return transactions
            .filter { $0.amount > 1000 }
            .sorted { $0.amount > $1.amount }
    }
    
    // Group transactions by category
    func groupByCategory() -> [String: [Transaction]] {
        return transactions.reduce(into: [:]) { result, transaction in
            result[transaction.category, default: []].append(transaction)
        }
    }
    
    // Average transaction amount
    func averageAmount() -> Double {
        guard !transactions.isEmpty else { return 0 }
        let total = transactions.map { $0.amount }.reduce(0, +)
        return total / Double(transactions.count)
    }
    
    // Transactions above average
    func aboveAverageTransactions() -> [Transaction] {
        let avg = averageAmount()
        return transactions.filter { $0.amount > avg }
    }
}
```

---

## Error Handling in Swift

Swift provides first-class support for throwing, catching, propagating, and manipulating recoverable errors.

### Defining Errors

```swift
// Error enumeration
enum NetworkError: Error {
    case badURL
    case noInternetConnection
    case timeout
    case serverError(code: Int)
    case decodingError
}

enum ValidationError: Error {
    case emptyField(fieldName: String)
    case invalidEmail
    case passwordTooShort
    case passwordMismatch
}

// Error with custom description
enum FileError: Error, LocalizedError {
    case fileNotFound
    case permissionDenied
    case corrupted
    
    var errorDescription: String? {
        switch self {
        case .fileNotFound:
            return "The file could not be found"
        case .permissionDenied:
            return "Permission denied to access file"
        case .corrupted:
            return "The file is corrupted"
        }
    }
}
```

### Throwing Errors

```swift
func validateEmail(_ email: String) throws -> Bool {
    if email.isEmpty {
        throw ValidationError.emptyField(fieldName: "Email")
    }
    
    if !email.contains("@") {
        throw ValidationError.invalidEmail
    }
    
    return true
}

func fetchData(from urlString: String) throws -> Data {
    guard let url = URL(string: urlString) else {
        throw NetworkError.badURL
    }
    
    // Simulate network call
    guard isConnectedToInternet() else {
        throw NetworkError.noInternetConnection
    }
    
    // Fetch data...
    return Data()
}

func isConnectedToInternet() -> Bool {
    // Check connection
    return true
}
```

### Handling Errors

#### do-catch

```swift
func processEmail(_ email: String) {
    do {
        try validateEmail(email)
        print("Email is valid")
    } catch ValidationError.emptyField(let field) {
        print("Error: \(field) is empty")
    } catch ValidationError.invalidEmail {
        print("Error: Invalid email format")
    } catch {
        print("Unexpected error: \(error)")
    }
}

// Multiple catch patterns
func handleNetworkRequest() {
    do {
        let data = try fetchData(from: "https://api.example.com")
        print("Data received: \(data)")
    } catch NetworkError.badURL {
        print("Invalid URL")
    } catch NetworkError.noInternetConnection {
        print("No internet connection")
    } catch NetworkError.serverError(let code) {
        print("Server error with code: \(code)")
    } catch {
        print("Unknown error: \(error)")
    }
}
```

#### try?

```swift
// Converts error to optional
let email = "test@example.com"
let isValid = try? validateEmail(email) // Optional(true)

let invalidEmail = ""
let isInvalid = try? validateEmail(invalidEmail) // nil

// Useful for optional chaining
if let data = try? fetchData(from: "https://api.example.com") {
    print("Got data: \(data)")
} else {
    print("Failed to fetch data")
}
```

#### try!

```swift
// Force try - crashes if error is thrown
// Only use when you're absolutely sure no error will occur
let data = try! fetchData(from: "https://api.example.com")

// Common use case: loading bundled resources
let path = Bundle.main.path(forResource: "config", ofType: "json")!
let fileData = try! Data(contentsOf: URL(fileURLWithPath: path))
```

### Propagating Errors

```swift
// Function that throws
func loadUserData(userId: Int) throws -> User {
    let data = try fetchData(from: "https://api.example.com/users/\(userId)")
    let user = try JSONDecoder().decode(User.self, from: data)
    return user
}

// Caller handles the error
func displayUser(userId: Int) {
    do {
        let user = try loadUserData(userId: userId)
        print("User: \(user.name)")
    } catch {
        print("Failed to load user: \(error)")
    }
}

struct User: Codable {
    let id: Int
    let name: String
}
```

### Rethrowing Functions

```swift
// Function that rethrows errors from closure
func performOperation<T>(_ operation: () throws -> T) rethrows -> T {
    print("Starting operation...")
    let result = try operation()
    print("Operation completed")
    return result
}

// Usage
func riskyOperation() throws -> String {
    // Might throw error
    return "Success"
}

do {
    let result = try performOperation {
        try riskyOperation()
    }
    print(result)
} catch {
    print("Error: \(error)")
}
```

### Defer Statement

```swift
func processFile(filename: String) throws {
    let file = openFile(filename)
    
    // defer - executed when leaving scope, regardless of how
    defer {
        closeFile(file)
        print("File closed")
    }
    
    // Work with file
    try readFromFile(file)
    try writeToFile(file, data: "Hello")
    
    // File will be closed even if error is thrown
}

// Multiple defer statements (executed in reverse order)
func example() {
    defer { print("First defer") }
    defer { print("Second defer") }
    defer { print("Third defer") }
    print("Function body")
}
// Output:
// Function body
// Third defer
// Second defer
// First defer

func openFile(_ name: String) -> String { return name }
func closeFile(_ file: String) { }
func readFromFile(_ file: String) throws { }
func writeToFile(_ file: String, data: String) throws { }
```

### Result Type

```swift
// Result type for explicit success/failure
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}

func fetchUser(id: Int, completion: (Result<User, NetworkError>) -> Void) {
    // Simulate async operation
    DispatchQueue.global().async {
        // Success case
        if id > 0 {
            let user = User(id: id, name: "John")
            completion(.success(user))
        } else {
            // Failure case
            completion(.failure(.badURL))
        }
    }
}

// Usage
fetchUser(id: 1) { result in
    switch result {
    case .success(let user):
        print("User: \(user.name)")
    case .failure(let error):
        print("Error: \(error)")
    }
}

// Converting Result to throws
func loadUser(id: Int) throws -> User {
    var userResult: Result<User, NetworkError>!
    
    fetchUser(id: id) { result in
        userResult = result
    }
    
    // Convert Result to throw
    return try userResult.get()
}
```

### Practical Example: Form Validation

```swift
struct RegistrationForm {
    let username: String
    let email: String
    let password: String
    let confirmPassword: String
}

enum FormValidationError: Error, LocalizedError {
    case emptyUsername
    case usernameTooShort
    case invalidEmail
    case passwordTooShort
    case passwordMismatch
    case weakPassword
    
    var errorDescription: String? {
        switch self {
        case .emptyUsername:
            return "Username cannot be empty"
        case .usernameTooShort:
            return "Username must be at least 3 characters"
        case .invalidEmail:
            return "Please enter a valid email address"
        case .passwordTooShort:
            return "Password must be at least 8 characters"
        case .passwordMismatch:
            return "Passwords do not match"
        case .weakPassword:
            return "Password must contain letters and numbers"
        }
    }
}

class FormValidator {
    func validate(_ form: RegistrationForm) throws {
        try validateUsername(form.username)
        try validateEmail(form.email)
        try validatePassword(form.password, confirmation: form.confirmPassword)
    }
    
    private func validateUsername(_ username: String) throws {
        guard !username.isEmpty else {
            throw FormValidationError.emptyUsername
        }
        
        guard username.count >= 3 else {
            throw FormValidationError.usernameTooShort
        }
    }
    
    private func validateEmail(_ email: String) throws {
        let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let emailPredicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        
        guard emailPredicate.evaluate(with: email) else {
            throw FormValidationError.invalidEmail
        }
    }
    
    private func validatePassword(_ password: String, confirmation: String) throws {
        guard password.count >= 8 else {
            throw FormValidationError.passwordTooShort
        }
        
        guard password == confirmation else {
            throw FormValidationError.passwordMismatch
        }
        
        let hasLetter = password.rangeOfCharacter(from: .letters) != nil
        let hasNumber = password.rangeOfCharacter(from: .decimalDigits) != nil
        
        guard hasLetter && hasNumber else {
            throw FormValidationError.weakPassword
        }
    }
}

// Usage
let validator = FormValidator()
let form = RegistrationForm(
    username: "john",
    email: "john@example.com",
    password: "password123",
    confirmPassword: "password123"
)

do {
    try validator.validate(form)
    print("Form is valid!")
} catch let error as FormValidationError {
    print("Validation error: \(error.errorDescription ?? "")")
} catch {
    print("Unexpected error: \(error)")
}
```

---

## Extensions & Protocols

### Extensions

Extensions add new functionality to existing types without modifying their source code.

```swift
// Extending built-in types
extension String {
    var isEmail: Bool {
        let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        let emailPredicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
        return emailPredicate.evaluate(with: self)
    }
    
    func truncate(to length: Int, trailing: String = "...") -> String {
        if self.count > length {
            let endIndex = self.index(self.startIndex, offsetBy: length)
            return String(self[..<endIndex]) + trailing
        }
        return self
    }
}

// Usage
let email = "test@example.com"
print(email.isEmail) // true

let longText = "This is a very long text"
print(longText.truncate(to: 10)) // "This is a ..."

// Extending Int
extension Int {
    var squared: Int {
        return self * self
    }
    
    func times(_ closure: () -> Void) {
        for _ in 0..<self {
            closure()
        }
    }
}

print(5.squared) // 25

3.times {
    print("Hello")
}

// Extending Array
extension Array where Element: Numeric {
    func sum() -> Element {
        return self.reduce(0, +)
    }
}

let numbers = [1, 2, 3, 4, 5]
print(numbers.sum()) // 15

// Extending custom types
struct Person {
    let firstName: String
    let lastName: String
}

extension Person {
    var fullName: String {
        return "\(firstName) \(lastName)"
    }
    
    init(fullName: String) {
        let components = fullName.components(separatedBy: " ")
        self.firstName = components.first ?? ""
        self.lastName = components.last ?? ""
    }
}

let person = Person(fullName: "John Doe")
print(person.fullName) // "John Doe"
```

### Protocols

Protocols define a blueprint of methods, properties, and other requirements.

```swift
// Basic protocol
protocol Identifiable {
    var id: String { get }
}

protocol Nameable {
    var name: String { get set }
}

// Protocol with methods
protocol Drawable {
    func draw()
}

protocol Resizable {
    mutating func resize(by percentage: Double)
}

// Implementing protocols
struct User: Identifiable, Nameable {
    let id: String
    var name: String
}

class Rectangle: Drawable, Resizable {
    var width: Double
    var height: Double
    
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    func draw() {
        print("Drawing rectangle: \(width) x \(height)")
    }
    
    func resize(by percentage: Double) {
        width *= (1 + percentage / 100)
        height *= (1 + percentage / 100)
    }
}

// Protocol inheritance
protocol Animal {
    var name: String { get }
    func makeSound()
}

protocol Pet: Animal {
    var owner: String { get set }
}

struct Dog: Pet {
    let name: String
    var owner: String
    
    func makeSound() {
        print("Woof!")
    }
}

// Protocol composition
func identify(_ item: Identifiable & Nameable) {
    print("ID: \(item.id), Name: \(item.name)")
}

// Protocol with associated types
protocol Container {
    associatedtype Item
    var count: Int { get }
    mutating func append(_ item: Item)
    subscript(index: Int) -> Item { get }
}

struct IntContainer: Container {
    typealias Item = Int
    
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

// Protocol extensions
protocol Greetable {
    var name: String { get }
    func greet()
}

extension Greetable {
    func greet() {
        print("Hello, \(name)!")
    }
    
    func formalGreet() {
        print("Good day, \(name).")
    }
}

struct Person2: Greetable {
    let name: String
    // greet() is provided by protocol extension
}

let person2 = Person2(name: "John")
person2.greet() // "Hello, John!"
person2.formalGreet() // "Good day, John."
```

### Protocol-Oriented Programming Example

```swift
// Define protocols
protocol Printable {
    func formatted() -> String
}

protocol Saveable {
    func save() throws
}

protocol Loadable {
    static func load(from id: String) throws -> Self
}

// Protocol extensions with default implementation
extension Saveable {
    func saveToUserDefaults(key: String, value: String) {
        UserDefaults.standard.set(value, forKey: key)
    }
}

// Model conforming to protocols
struct Article: Printable, Saveable, Loadable {
    let id: String
    let title: String
    let content: String
    let author: String
    
    func formatted() -> String {
        return """
        Title: \(title)
        Author: \(author)
        
        \(content)
        """
    }
    
    func save() throws {
        saveToUserDefaults(key: "article_\(id)", value: formatted())
    }
    
    static func load(from id: String) throws -> Article {
        // Load from storage
        return Article(id: id, title: "Sample", content: "Content", author: "Author")
    }
}

// Generic function using protocols
func display<T: Printable>(_ item: T) {
    print(item.formatted())
}

func persistAndDisplay<T: Printable & Saveable>(_ item: T) {
    print(item.formatted())
    try? item.save()
}
```

---

## Generics in Swift

Generics enable you to write flexible, reusable functions and types that can work with any type.

### Generic Functions

```swift
// Without generics
func swapInts(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

func swapStrings(_ a: inout String, _ b: inout String) {
    let temp = a
    a = b
    b = temp
}

// With generics (one function for all types)
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var int1 = 5
var int2 = 10
swap(&int1, &int2)
print("int1: \(int1), int2: \(int2)") // int1: 10, int2: 5

var str1 = "Hello"
var str2 = "World"
swap(&str1, &str2)
print("str1: \(str1), str2: \(str2)") // str1: World, str2: Hello
```

### Generic Types

```swift
// Generic stack
struct Stack<Element> {
    private var items: [Element] = []
    
    var isEmpty: Bool {
        return items.isEmpty
    }
    
    var count: Int {
        return items.count
    }
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        return items.popLast()
    }
    
    func peek() -> Element? {
        return items.last
    }
}

// Usage
var intStack = Stack<Int>()
intStack.push(1)
intStack.push(2)
intStack.push(3)
print(intStack.pop()) // Optional(3)

var stringStack = Stack<String>()
stringStack.push("A")
stringStack.push("B")
print(stringStack.peek()) // Optional("B")

// Generic pair
struct Pair<T, U> {
    let first: T
    let second: U
}

let coordinate = Pair(first: 10, second: 20)
let nameAge = Pair(first: "John", second: 25)
```

### Type Constraints

```swift
// Constraint: T must conform to Equatable
func findIndex<T: Equatable>(of valueToFind: T, in array: [T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}

let numbers = [1, 2, 3, 4, 5]
if let index = findIndex(of: 3, in: numbers) {
    print("Found at index \(index)") // Found at index 2
}

// Multiple constraints
func compare<T: Comparable & CustomStringConvertible>(_ a: T, _ b: T) {
    if a > b {
        print("\(a.description) is greater")
    } else {
        print("\(b.description) is greater")
    }
}

// Where clauses
func allEqual<T: Equatable>(_ items: [T]) -> Bool where T: Comparable {
    guard let first = items.first else { return true }
    return items.allSatisfy { $0 == first }
}
```

### Associated Types

```swift
protocol Queue {
    associatedtype Element
    mutating func enqueue(_ item: Element)
    mutating func dequeue() -> Element?
}

struct IntQueue: Queue {
    typealias Element = Int
    
    private var items: [Int] = []
    
    mutating func enqueue(_ item: Int) {
        items.append(item)
    }
    
    mutating func dequeue() -> Int? {
        return items.isEmpty ? nil : items.removeFirst()
    }
}

struct GenericQueue<T>: Queue {
    private var items: [T] = []
    
    mutating func enqueue(_ item: T) {
        items.append(item)
    }
    
    mutating func dequeue() -> T? {
        return items.isEmpty ? nil : items.removeFirst()
    }
}
```

### Generic Subscripts

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Element == Int {
        var result: [Item] = []
        for index in indices {
            result.append(self[index])
        }
        return result
    }
}
```

### Practical Generic Example

```swift
// Generic Result wrapper
enum NetworkResult<T> {
    case success(T)
    case failure(Error)
    
    func map<U>(_ transform: (T) -> U) -> NetworkResult<U> {
        switch self {
        case .success(let value):
            return .success(transform(value))
        case .failure(let error):
            return .failure(error)
        }
    }
}

// Generic Repository
protocol Repository {
    associatedtype Model
    func fetch(id: String) async throws -> Model
    func save(_ model: Model) async throws
    func delete(id: String) async throws
}

class UserRepository: Repository {
    typealias Model = User
    
    func fetch(id: String) async throws -> User {
        // Fetch user from API
        return User(id: 1, name: "John")
    }
    
    func save(_ model: User) async throws {
        // Save user
    }
    
    func delete(id: String) async throws {
        // Delete user
    }
}

// Generic Caching
class Cache<Key: Hashable, Value> {
    private var storage: [Key: Value] = [:]
    
    func set(_ value: Value, forKey key: Key) {
        storage[key] = value
    }
    
    func get(_ key: Key) -> Value? {
        return storage[key]
    }
    
    func remove(_ key: Key) {
        storage.removeValue(forKey: key)
    }
    
    func clear() {
        storage.removeAll()
    }
}

// Usage
let imageCache = Cache<String, UIImage>()
if let image = UIImage(named: "logo") {
    imageCache.set(image, forKey: "logo")
}

let cachedImage = imageCache.get("logo")
```

---

## Value Type vs Reference Type

### Value Types

Value types are copied when assigned or passed to functions.

```swift
// Struct (Value Type)
struct Point {
    var x: Int
    var y: Int
}

var point1 = Point(x: 10, y: 20)
var point2 = point1 // Copy created

point2.x = 30

print(point1.x) // 10 (unchanged)
print(point2.x) // 30

// Enum (Value Type)
enum Direction {
    case north, south, east, west
}

var direction1 = Direction.north
var direction2 = direction1 // Copy created

direction2 = .south

print(direction1) // north (unchanged)
print(direction2) // south
```

### Reference Types

Reference types share a single copy of data.

```swift
// Class (Reference Type)
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
}

let person1 = Person(name: "John", age: 25)
let person2 = person1 // Same reference

person2.name = "Jane"

print(person1.name) // "Jane" (changed)
print(person2.name) // "Jane"

// Checking reference equality
if person1 === person2 {
    print("Same instance")
}

if person1 !== person2 {
    print("Different instances")
}
```

### Comparison

| Aspect | Value Type (Struct, Enum) | Reference Type (Class) |
|--------|---------------------------|------------------------|
| **Memory** | Stack (usually) | Heap |
| **Assignment** | Creates copy | Shares reference |
| **Mutation** | Requires `mutating` | Direct mutation |
| **Thread Safety** | Thread-safe by default | Requires synchronization |
| **Performance** | Faster for small data | Better for large data |
| **Inheritance** | No | Yes |
| **Deinitializer** | No | Yes (`deinit`) |
| **Identity** | Value equality | Reference equality (`===`) |

### When to Use Which?

**Use Struct (Value Type) when:**
- Data represents a simple value (Point, Size, Range)
- You want independent copies
- Data will be used in multithreaded environment
- Data is relatively small

**Use Class (Reference Type) when:**
- Data represents an identity (Person, Account)
- You need inheritance
- You need shared mutable state
- Working with objective-C APIs

### Copy-on-Write

Swift uses copy-on-write optimization for collections to get benefits of both.

```swift
var array1 = [1, 2, 3, 4, 5]
var array2 = array1 // No copy yet, shared storage

// Copy only happens when array2 is modified
array2.append(6) // Now a copy is made

print(array1) // [1, 2, 3, 4, 5]
print(array2) // [1, 2, 3, 4, 5, 6]

// Custom copy-on-write
struct MyArray<Element> {
    private var storage: Storage
    
    init() {
        storage = Storage()
    }
    
    var count: Int {
        return storage.items.count
    }
    
    subscript(index: Int) -> Element {
        get {
            return storage.items[index]
        }
        set {
            if !isKnownUniquelyReferenced(&storage) {
                storage = Storage(items: storage.items)
            }
            storage.items[index] = newValue
        }
    }
    
    private class Storage {
        var items: [Element] = []
        
        init() { }
        
        init(items: [Element]) {
            self.items = items
        }
    }
}
```

---

## ARC (Automatic Reference Counting)

ARC automatically manages memory by keeping track of references to class instances.

### How ARC Works

```swift
class Person {
    let name: String
    
    init(name: String) {
        self.name = name
        print("\(name) is initialized")
    }
    
    deinit {
        print("\(name) is deinitialized")
    }
}

var person1: Person? = Person(name: "John") // Reference count = 1
var person2 = person1 // Reference count = 2
var person3 = person1 // Reference count = 3

person1 = nil // Reference count = 2
person2 = nil // Reference count = 1
person3 = nil // Reference count = 0 → deinit called
```

### Strong References

```swift
class Apartment {
    let number: Int
    var tenant: Person?
    
    init(number: Int) {
        self.number = number
    }
    
    deinit {
        print("Apartment #\(number) is deinitialized")
    }
}

var john: Person? = Person(name: "John")
var unit4A: Apartment? = Apartment(number: 4)

john?.residence = unit4A
unit4A?.tenant = john

// Both instances kept alive by strong references
```

### Weak References

Weak references don't increase reference count and automatically become nil when the instance is deallocated.

```swift
class Person3 {
    let name: String
    var apartment: Apartment2?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is deinitialized")
    }
}

class Apartment2 {
    let number: Int
    weak var tenant: Person3? // weak reference
    
    init(number: Int) {
        self.number = number
    }
    
    deinit {
        print("Apartment #\(number) is deinitialized")
    }
}

var john2: Person3? = Person3(name: "John")
var unit4B: Apartment2? = Apartment2(number: 4)

john2?.apartment = unit4B
unit4B?.tenant = john2

john2 = nil // John is deinitialized
// tenant automatically becomes nil
```

### Unowned References

Unowned references don't increase reference count but assume the reference always has a value.

```swift
class Customer {
    let name: String
    var card: CreditCard?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is deinitialized")
    }
}

class CreditCard {
    let number: String
    unowned let owner: Customer // unowned reference
    
    init(number: String, owner: Customer) {
        self.number = number
        self.owner = owner
    }
    
    deinit {
        print("Card #\(number) is deinitialized")
    }
}

var john3: Customer? = Customer(name: "John")
john3?.card = CreditCard(number: "1234", owner: john3!)

john3 = nil // Both customer and card are deinitialized
```

### Retain Cycles

```swift
// Retain cycle problem
class Person4 {
    let name: String
    var apartment: Apartment3?
    
    init(name: String) {
        self.name = name
    }
    
    deinit {
        print("\(name) is deinitialized")
    }
}

class Apartment3 {
    let number: Int
    var tenant: Person4? // Strong reference
    
    init(number: Int) {
        self.number = number
    }
    
    deinit {
        print("Apartment #\(number) is deinitialized")
    }
}

var person: Person4? = Person4(name: "John")
var apartment: Apartment3? = Apartment3(number: 4)

person?.apartment = apartment
apartment?.tenant = person

person = nil
apartment = nil
// Neither deinit is called - memory leak!
```

### Closures and Capture Lists

```swift
class ViewController {
    var name = "ViewController"
    var closure: (() -> Void)?
    
    func setupClosure() {
        // Strong reference cycle
        closure = {
            print(self.name) // Captures self strongly
        }
    }
    
    func setupClosureCorrectly() {
        // Weak reference
        closure = { [weak self] in
            print(self?.name ?? "nil")
        }
    }
    
    func setupClosureWithUnowned() {
        // Unowned reference
        closure = { [unowned self] in
            print(self.name)
        }
    }
    
    deinit {
        print("\(name) is deinitialized")
    }
}

// Multiple captures
class DataManager {
    var data: [String] = []
    
    func processData(completion: @escaping () -> Void) {
        DispatchQueue.global().async { [weak self] in
            guard let self = self else { return }
            // Use self safely
            self.data.append("New data")
            completion()
        }
    }
}

// Capture specific values
class Counter {
    var count = 0
    
    func makeIncrementer(amount: Int) -> () -> Void {
        return { [weak self] in
            self?.count += amount
        }
    }
}
```

### Practical Example: Delegation Pattern

```swift
// Wrong way - retain cycle
protocol DataSourceDelegate {
    func didReceiveData(_ data: String)
}

class DataSource {
    var delegate: DataSourceDelegate? // Strong reference - WRONG
    
    func fetchData() {
        delegate?.didReceiveData("Some data")
    }
}

// Correct way
protocol DataSourceDelegate2: AnyObject {
    func didReceiveData(_ data: String)
}

class DataSource2 {
    weak var delegate: DataSourceDelegate2? // Weak reference - CORRECT
    
    func fetchData() {
        delegate?.didReceiveData("Some data")
    }
}

class ViewController2: DataSourceDelegate2 {
    let dataSource = DataSource2()
    
    init() {
        dataSource.delegate = self
    }
    
    func didReceiveData(_ data: String) {
        print("Received: \(data)")
    }
    
    deinit {
        print("ViewController deinitialized")
    }
}
```

### Interview Questions

**Q1: What's the difference between weak and unowned?**

**Answer:**
- **weak**: Optional reference that becomes nil when deallocated. Use when reference can become nil during lifetime.
- **unowned**: Non-optional reference that doesn't become nil. Use when reference should never be nil after initialization. Crashes if accessed after deallocation.

**Q2: When do retain cycles occur?**

**Answer:** Retain cycles occur when two or more objects hold strong references to each other, preventing ARC from deallocating them. Common scenarios:
- Parent-child relationships
- Delegate patterns without weak
- Closures capturing self

**Q3: How do you detect memory leaks?**

**Answer:**
- Use Instruments' Leaks tool
- Check deinit is called
- Use Memory Graph Debugger in Xcode
- Monitor memory usage in Debug Navigator

**Q4: Explain the difference between value types and reference types with examples**

**Answer:**

**Value Types (Struct, Enum):**
- Copied when assigned or passed to functions
- Each instance has its own copy of data
- Stored on stack (usually faster)
- Thread-safe by default
- No reference counting overhead

```swift
struct Point {
    var x: Int
    var y: Int
}

var point1 = Point(x: 10, y: 20)
var point2 = point1  // Copy created
point2.x = 30

print(point1.x)  // 10 (unchanged)
print(point2.x)  // 30
```

**Reference Types (Class):**
- Shared reference when assigned
- Multiple variables can reference same instance
- Stored on heap
- Requires ARC for memory management
- Can have inheritance

```swift
class Person {
    var name: String
    init(name: String) { self.name = name }
}

let person1 = Person(name: "John")
let person2 = person1  // Same reference
person2.name = "Jane"

print(person1.name)  // "Jane" (both changed)
```

**When to use:**
- Use struct for data models, simple values
- Use class for shared state, identity, inheritance

**Q5: What are optionals and why does Swift have them?**

**Answer:**

Optionals handle the absence of a value safely, preventing null pointer crashes.

**Why Swift has Optionals:**
- Type safety: Compiler enforces checking for nil
- Prevents crashes from accessing nil values
- Makes intent explicit (value may or may not exist)
- Forces developers to handle missing values

**Unwrapping Methods:**

```swift
var name: String? = "John"

// 1. If let (safe)
if let unwrappedName = name {
    print(unwrappedName)
}

// 2. Guard let (early exit)
guard let unwrappedName = name else { return }
print(unwrappedName)

// 3. Nil coalescing (default value)
let displayName = name ?? "Guest"

// 4. Optional chaining
let uppercased = name?.uppercased()

// 5. Force unwrap (unsafe - only if 100% sure)
let forcedName = name!  // Crashes if nil
```

**Best Practice:** Prefer optional binding (if let/guard let) over force unwrapping.

**Q6: Explain closures and capture lists**

**Answer:**

**Closures** are self-contained blocks of functionality that can capture and store references to variables from surrounding context.

**Basic Closure:**
```swift
let greet = { (name: String) -> String in
    return "Hello, \(name)"
}
```

**Capture Lists** prevent retain cycles:

**Problem (Retain Cycle):**
```swift
class ViewController {
    var name = "View"
    var closure: (() -> Void)?
    
    func setup() {
        closure = {
            print(self.name)  // Strong reference to self
        }
    }
}
```

**Solution 1 - Weak:**
```swift
closure = { [weak self] in
    guard let self = self else { return }
    print(self.name)
}
```

**Solution 2 - Unowned:**
```swift
closure = { [unowned self] in
    print(self.name)
}
```

**Multiple Captures:**
```swift
closure = { [weak self, weak delegate] in
    self?.doSomething()
    delegate?.notify()
}
```

**When to use:**
- Use `weak` when reference can become nil
- Use `unowned` when reference should never be nil (but be careful!)

**Q7: What is the difference between map, flatMap, and compactMap?**

**Answer:**

**Map** - Transform each element:
```swift
let numbers = [1, 2, 3, 4]
let doubled = numbers.map { $0 * 2 }
// [2, 4, 6, 8]
```

**CompactMap** - Transform and remove nils:
```swift
let strings = ["1", "2", "three", "4"]
let numbers = strings.compactMap { Int($0) }
// [1, 2, 4] - "three" removed
```

**FlatMap** - Transform and flatten nested collections:
```swift
let nested = [[1, 2], [3, 4], [5, 6]]
let flattened = nested.flatMap { $0 }
// [1, 2, 3, 4, 5, 6]
```

**Practical Example:**
```swift
struct User {
    let id: Int
    let name: String
}

let users = [
    User(id: 1, name: "John"),
    User(id: 2, name: "Jane")
]

// map - transform
let names = users.map { $0.name }
// ["John", "Jane"]

// compactMap - transform with optionals
let ids = ["1", "2", "abc"].compactMap { Int($0) }
// [1, 2]

// flatMap - flatten nested
let departments = [
    [User(id: 1, name: "John")],
    [User(id: 2, name: "Jane")]
]
let allUsers = departments.flatMap { $0 }
// [User(id:1), User(id:2)]
```

**Q8: Explain generics in Swift and when to use them**

**Answer:**

**Generics** allow you to write flexible, reusable code that works with any type while maintaining type safety.

**Without Generics (Code Duplication):**
```swift
func swapInts(_ a: inout Int, _ b: inout Int) {
    let temp = a
    a = b
    b = temp
}

func swapStrings(_ a: inout String, _ b: inout String) {
    let temp = a
    a = b
    b = temp
}
```

**With Generics (One Function):**
```swift
func swap<T>(_ a: inout T, _ b: inout T) {
    let temp = a
    a = b
    b = temp
}

var x = 5, y = 10
swap(&x, &y)  // Works with Int

var str1 = "Hello", str2 = "World"
swap(&str1, &str2)  // Works with String
```

**Generic Types:**
```swift
struct Stack<Element> {
    private var items: [Element] = []
    
    mutating func push(_ item: Element) {
        items.append(item)
    }
    
    mutating func pop() -> Element? {
        return items.popLast()
    }
}

var intStack = Stack<Int>()
intStack.push(5)

var stringStack = Stack<String>()
stringStack.push("Hello")
```

**Type Constraints:**
```swift
func findIndex<T: Equatable>(of value: T, in array: [T]) -> Int? {
    for (index, element) in array.enumerated() {
        if element == value {
            return index
        }
    }
    return nil
}
```

**When to Use:**
- Collections (Array, Dictionary, Set)
- Algorithms that work with multiple types
- Avoiding code duplication
- Maintaining type safety

**Benefits:**
- Code reuse
- Type safety
- Better performance (no runtime type checking)
- Cleaner, more maintainable code

---

[← Previous: iOS Basics](ios-basics.md) | [Next: OOP and POP →](oop-and-pop.md)

[Back to Main](../README.md)

