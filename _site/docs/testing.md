# Testing in iOS

[← Back to Main](../README.md) | [Previous: Memory Management](memory-management.md) | [Next: Advanced Topics →](advanced-topics.md)

## Table of Contents
- [Unit Testing with XCTest](#unit-testing-with-xctest)
- [UI Testing](#ui-testing)
- [Test Doubles](#test-doubles-mock-stub-fake)
- [TDD & BDD](#tdd--bdd-in-ios)

---

## Unit Testing with XCTest

### Basic Test

```swift
import XCTest
@testable import MyApp

class CalculatorTests: XCTestCase {
    var calculator: Calculator!
    
    override func setUp() {
        super.setUp()
        calculator = Calculator()
    }
    
    override func tearDown() {
        calculator = nil
        super.tearDown()
    }
    
    func testAddition() {
        let result = calculator.add(2, 3)
        XCTAssertEqual(result, 5)
    }
    
    func testDivisionByZero() {
        XCTAssertThrowsError(try calculator.divide(10, by: 0))
    }
}

class Calculator {
    func add(_ a: Int, _ b: Int) -> Int {
        return a + b
    }
    
    func divide(_ a: Int, by b: Int) throws -> Int {
        guard b != 0 else { throw CalculatorError.divisionByZero }
        return a / b
    }
    
    enum CalculatorError: Error {
        case divisionByZero
    }
}
```

### Async Testing

```swift
func testAsyncFetch() async throws {
    let service = UserService()
    let users = try await service.fetchUsers()
    
    XCTAssertFalse(users.isEmpty)
}

func testAsyncWithExpectation() {
    let expectation = XCTestExpectation(description: "Fetch users")
    
    userService.fetchUsers { result in
        switch result {
        case .success(let users):
            XCTAssertFalse(users.isEmpty)
        case .failure:
            XCTFail("Should not fail")
        }
        expectation.fulfill()
    }
    
    wait(for: [expectation], timeout: 5.0)
}
```

---

## UI Testing

### Basic UI Test

```swift
import XCTest

class MyAppUITests: XCTestCase {
    var app: XCUIApplication!
    
    override func setUp() {
        super.setUp()
        continueAfterFailure = false
        app = XCUIApplication()
        app.launch()
    }
    
    func testLoginFlow() {
        let emailField = app.textFields["emailTextField"]
        emailField.tap()
        emailField.typeText("test@example.com")
        
        let passwordField = app.secureTextFields["passwordTextField"]
        passwordField.tap()
        passwordField.typeText("password123")
        
        app.buttons["loginButton"].tap()
        
        XCTAssertTrue(app.staticTexts["Welcome"].exists)
    }
}
```

---

## Test Doubles (Mock, Stub, Fake)

### Mock

```swift
class MockNetworkService: NetworkServiceProtocol {
    var fetchDataCalled = false
    var fetchDataResult: Result<Data, Error>?
    
    func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
        fetchDataCalled = true
        if let result = fetchDataResult {
            completion(result)
        }
    }
}

// Test
func testUserRepository() {
    let mockService = MockNetworkService()
    mockService.fetchDataResult = .success(Data())
    
    let repository = UserRepository(networkService: mockService)
    repository.getUsers { _ in }
    
    XCTAssertTrue(mockService.fetchDataCalled)
}
```

### Stub

```swift
class StubUserService: UserServiceProtocol {
    func fetchUser(id: Int) -> User {
        return User(id: id, name: "Test User", email: "test@example.com")
    }
}
```

---

## TDD & BDD in iOS

### TDD (Test-Driven Development)

1. Write failing test
2. Write minimal code to pass
3. Refactor

```swift
// 1. Write test (fails)
func testValidateEmail() {
    let validator = EmailValidator()
    XCTAssertTrue(validator.isValid("test@example.com"))
}

// 2. Implement
class EmailValidator {
    func isValid(_ email: String) -> Bool {
        return email.contains("@")
    }
}

// 3. Refactor if needed
```

### BDD (Behavior-Driven Development)

```swift
// Using Quick & Nimble
import Quick
import Nimble

class UserSpec: QuickSpec {
    override func spec() {
        describe("User") {
            context("when created") {
                it("has a name") {
                    let user = User(name: "John")
                    expect(user.name).to(equal("John"))
                }
            }
        }
    }
}
```

---

[← Previous: Memory Management](memory-management.md) | [Next: Advanced Topics →](advanced-topics.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between unit tests, integration tests, and UI tests?

**Answer:**

**Unit Tests:**
- Test individual components in isolation
- Fast execution
- No external dependencies
- Mock external services

```swift
func testCalculatorAdd() {
    let calculator = Calculator()
    let result = calculator.add(2, 3)
    XCTAssertEqual(result, 5)
}
```

**Integration Tests:**
- Test multiple components together
- Test interactions
- May use real dependencies
- Slower than unit tests

```swift
func testUserRepositoryFetch() {
    let repository = UserRepository(network: NetworkService())
    let expectation = XCTestExpectation()
    
    repository.fetchUsers { users in
        XCTAssertFalse(users.isEmpty)
        expectation.fulfill()
    }
    
    wait(for: [expectation], timeout: 5.0)
}
```

**UI Tests:**
- Test user interface and flows
- Simulate user interactions
- Slowest
- Black box testing

```swift
func testLoginFlow() {
    let app = XCUIApplication()
    app.textFields["email"].tap()
    app.textFields["email"].typeText("test@example.com")
    app.buttons["Login"].tap()
    XCTAssertTrue(app.staticTexts["Welcome"].exists)
}
```

**Test Pyramid:**
```
       /\
      /UI\      ← Few (slow, fragile)
     /____\
    /Integ.\   ← Some (medium speed)
   /________\
  /   Unit   \ ← Many (fast, stable)
 /____________\
```

### Q2: How do you test asynchronous code?

**Answer:**

**Method 1: XCTestExpectation**

```swift
func testAsyncFetch() {
    let expectation = XCTestExpectation(description: "Fetch users")
    
    userService.fetchUsers { result in
        switch result {
        case .success(let users):
            XCTAssertFalse(users.isEmpty)
        case .failure:
            XCTFail("Should not fail")
        }
        expectation.fulfill()
    }
    
    wait(for: [expectation], timeout: 5.0)
}
```

**Method 2: Async/Await (iOS 13+)**

```swift
func testAsyncAwait() async throws {
    let users = try await userService.fetchUsers()
    XCTAssertFalse(users.isEmpty)
}
```

**Method 3: Multiple Expectations**

```swift
func testMultipleAsync() {
    let exp1 = XCTestExpectation(description: "Task 1")
    let exp2 = XCTestExpectation(description: "Task 2")
    
    service.task1 {
        XCTAssertTrue(condition)
        exp1.fulfill()
    }
    
    service.task2 {
        XCTAssertTrue(condition)
        exp2.fulfill()
    }
    
    wait(for: [exp1, exp2], timeout: 10.0)
}
```

### Q3: What are mocks, stubs, and fakes? When to use each?

**Answer:**

**Mock:**
- Records how it was called
- Verifies interactions
- Use to verify behavior

```swift
class MockNetworkService: NetworkService {
    var fetchCalled = false
    var fetchCallCount = 0
    var lastURL: URL?
    
    func fetch(url: URL, completion: (Data?) -> Void) {
        fetchCalled = true
        fetchCallCount += 1
        lastURL = url
        completion(nil)
    }
}

// Test
func testRepositoryCallsNetwork() {
    let mock = MockNetworkService()
    let repository = UserRepository(network: mock)
    
    repository.fetchUsers { _ in }
    
    XCTAssertTrue(mock.fetchCalled)
    XCTAssertEqual(mock.fetchCallCount, 1)
}
```

**Stub:**
- Returns predetermined data
- Doesn't verify interactions
- Use to provide test data

```swift
class StubNetworkService: NetworkService {
    var stubbedData: Data?
    
    func fetch(url: URL, completion: (Data?) -> Void) {
        completion(stubbedData)  // Return stubbed data
    }
}

// Test
func testParsingUsers() {
    let stub = StubNetworkService()
    stub.stubbedData = loadMockJSON()
    
    let repository = UserRepository(network: stub)
    repository.fetchUsers { users in
        XCTAssertEqual(users.count, 2)
    }
}
```

**Fake:**
- Working implementation with shortcuts
- Simpler than production
- Use for complex dependencies

```swift
class FakeDatabase: Database {
    private var storage: [Int: User] = [:]
    
    func save(_ user: User) {
        storage[user.id] = user  // In-memory storage
    }
    
    func fetch(id: Int) -> User? {
        return storage[id]
    }
}

// Test with real database behavior (but in memory)
```

**Summary:**
- **Mock**: Verify interactions (was method called?)
- **Stub**: Provide data (return predetermined values)
- **Fake**: Working but simplified implementation

### Q4: What is Test-Driven Development (TDD)?

**Answer:**

TDD is writing tests before implementation.

**Process:**

**1. Red:** Write failing test
```swift
func testEmailValidation() {
    let validator = EmailValidator()
    XCTAssertTrue(validator.isValid("test@example.com"))
}
// Fails - EmailValidator doesn't exist
```

**2. Green:** Write minimal code to pass
```swift
class EmailValidator {
    func isValid(_ email: String) -> Bool {
        return email.contains("@")  // Minimal implementation
    }
}
// Test passes
```

**3. Refactor:** Improve code while keeping tests green
```swift
class EmailValidator {
    func isValid(_ email: String) -> Bool {
        let regex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}"
        return NSPredicate(format: "SELF MATCHES %@", regex).evaluate(with: email)
    }
}
// Still passes, but better implementation
```

**Benefits:**
- Better design (write testable code)
- Documentation (tests show usage)
- Confidence (refactor safely)
- Fewer bugs

**Drawbacks:**
- Takes more time initially
- Requires discipline
- Can over-test trivial code

### Q5: How do you test view controllers?

**Answer:**

**1. Test Business Logic (Extract to ViewModel):**

```swift
class UserViewModel {
    func validateEmail(_ email: String) -> Bool {
        return email.contains("@")
    }
}

// Easy to test
func testEmailValidation() {
    let viewModel = UserViewModel()
    XCTAssertTrue(viewModel.validateEmail("test@example.com"))
    XCTAssertFalse(viewModel.validateEmail("invalid"))
}
```

**2. Test View Loading:**

```swift
func testViewLoads() {
    let vc = UserViewController()
    _ = vc.view  // Force view to load
    
    XCTAssertNotNil(vc.view)
    XCTAssertNotNil(vc.tableView)
}
```

**3. Test Actions:**

```swift
func testLoginButtonAction() {
    let vc = LoginViewController()
    _ = vc.view
    
    vc.emailTextField.text = "test@example.com"
    vc.passwordTextField.text = "password"
    vc.loginButton.sendActions(for: .touchUpInside)
    
    // Verify result
    XCTAssertTrue(vc.isLoading)
}
```

**4. Test Delegation:**

```swift
class MockDelegate: LoginViewControllerDelegate {
    var didLoginCalled = false
    
    func didLogin(with user: User) {
        didLoginCalled = true
    }
}

func testDelegateCall() {
    let mockDelegate = MockDelegate()
    let vc = LoginViewController()
    vc.delegate = mockDelegate
    
    vc.performLogin()
    
    XCTAssertTrue(mockDelegate.didLoginCalled)
}
```

**Best Practice:**
- Keep business logic in ViewModel (easy to test)
- Keep View Controllers thin (hard to test)
- Use protocols for dependencies (inject mocks)

### Q6: What's the difference between XCTAssert methods?

**Answer:**

**Equality:**
```swift
XCTAssertEqual(actual, expected)
XCTAssertNotEqual(actual, notExpected)

// With accuracy for floating point
XCTAssertEqual(3.14, piValue, accuracy: 0.01)
```

**Boolean:**
```swift
XCTAssertTrue(condition)
XCTAssertFalse(condition)
```

**Nil Checking:**
```swift
XCTAssertNil(optionalValue)
XCTAssertNotNil(optionalValue)
```

**Error Handling:**
```swift
XCTAssertThrowsError(try riskyFunction())
XCTAssertNoThrow(try safeFunction())
```

**Failure:**
```swift
XCTFail("This shouldn't happen")
```

**Custom Messages:**
```swift
XCTAssertEqual(result, 5, "Calculator add failed")
```

### Q7: How do you achieve high test coverage?

**Answer:**

**1. Test Pyramid Approach:**
- 70% Unit tests
- 20% Integration tests  
- 10% UI tests

**2. Code Coverage Tool:**
- Edit Scheme → Test → Options
- Enable "Gather coverage for: All targets"
- View coverage in Report Navigator

**3. What to Test:**

**High Priority:**
- Business logic
- Data transformations
- Error handling
- Edge cases

**Medium Priority:**
- UI state changes
- Navigation
- Data persistence

**Low Priority:**
- Simple getters/setters
- UI layout code

**4. Test Naming Convention:**
```swift
func test_methodName_condition_expectedResult() {
    // test_validateEmail_invalidFormat_returnsFalse
}
```

**5. AAA Pattern:**
```swift
func testUserCreation() {
    // Arrange
    let name = "John"
    let email = "john@example.com"
    
    // Act
    let user = User(name: name, email: email)
    
    // Assert
    XCTAssertEqual(user.name, name)
    XCTAssertEqual(user.email, email)
}
```

**6. Test Data Builders:**
```swift
class UserBuilder {
    private var id = 1
    private var name = "Test User"
    
    func withId(_ id: Int) -> UserBuilder {
        self.id = id
        return self
    }
    
    func build() -> User {
        return User(id: id, name: name)
    }
}

// Usage in tests
let user = UserBuilder().withId(5).build()
```

**Target:** Aim for 70-80% code coverage, but focus on critical paths.

---

[← Previous: Memory Management](memory-management.md) | [Next: Advanced Topics →](advanced-topics.md)

[Back to Main](../README.md)
