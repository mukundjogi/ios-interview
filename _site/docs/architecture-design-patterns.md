# Architecture & Design Patterns

[← Back to Main](../README.md) | [Previous: Multithreading & Concurrency](multithreading-concurrency.md) | [Next: Dependency Management →](dependency-management.md)

## Table of Contents
- [MVC, MVVM, MVP, VIPER](#architecture-patterns)
- [Coordinators](#coordinators-in-ios)
- [Dependency Injection](#dependency-injection-in-swift)
- [Common Design Patterns](#common-design-patterns)
- [SOLID Principles](#solid-principles-in-ios-development)
- [Clean Architecture](#clean-architecture-in-ios)

---

## Architecture Patterns

### MVC (Model-View-Controller)

Apple's default pattern.

```swift
// Model
struct User {
    let id: Int
    let name: String
    let email: String
}

// View
class UserView: UIView {
    let nameLabel = UILabel()
    let emailLabel = UILabel()
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupUI()
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
    private func setupUI() {
        addSubview(nameLabel)
        addSubview(emailLabel)
        // Setup constraints
    }
    
    func configure(with user: User) {
        nameLabel.text = user.name
        emailLabel.text = user.email
    }
}

// Controller
class UserViewController: UIViewController {
    private let userView = UserView()
    private var user: User?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        fetchUser()
    }
    
    private func fetchUser() {
        // Fetch data from network/database
        user = User(id: 1, name: "John", email: "john@example.com")
        updateView()
    }
    
    private func updateView() {
        guard let user = user else { return }
        userView.configure(with: user)
    }
}
```

### MVVM (Model-View-ViewModel)

Separates business logic from view controller.

```swift
// Model
struct Product {
    let id: Int
    let name: String
    let price: Double
}

// ViewModel
class ProductViewModel {
    private let product: Product
    
    var name: String {
        return product.name
    }
    
    var formattedPrice: String {
        return "$\(String(format: "%.2f", product.price))"
    }
    
    init(product: Product) {
        self.product = product
    }
}

// ViewController (View)
class ProductViewController: UIViewController {
    private let nameLabel = UILabel()
    private let priceLabel = UILabel()
    
    var viewModel: ProductViewModel! {
        didSet {
            updateUI()
        }
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
    }
    
    private func setupUI() {
        view.addSubview(nameLabel)
        view.addSubview(priceLabel)
    }
    
    private func updateUI() {
        nameLabel.text = viewModel.name
        priceLabel.text = viewModel.formattedPrice
    }
}

// Usage
let product = Product(id: 1, name: "iPhone", price: 999.99)
let viewModel = ProductViewModel(product: product)
let viewController = ProductViewController()
viewController.viewModel = viewModel
```

### MVVM with Combine

```swift
import Combine

class UserListViewModel: ObservableObject {
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    private let userService: UserService
    private var cancellables = Set<AnyCancellable>()
    
    init(userService: UserService = UserService()) {
        self.userService = userService
    }
    
    func fetchUsers() {
        isLoading = true
        
        userService.getUsers()
            .sink { [weak self] completion in
                self?.isLoading = false
                
                if case .failure(let error) = completion {
                    self?.errorMessage = error.localizedDescription
                }
            } receiveValue: { [weak self] users in
                self?.users = users
            }
            .store(in: &cancellables)
    }
}

class UserService {
    func getUsers() -> AnyPublisher<[User], Error> {
        // Network request
        return Just([User(id: 1, name: "John", email: "john@example.com")])
            .setFailureType(to: Error.self)
            .eraseToAnyPublisher()
    }
}
```

### MVP (Model-View-Presenter)

```swift
// Model
struct Task {
    let id: Int
    var title: String
    var isCompleted: Bool
}

// View Protocol
protocol TaskViewProtocol: AnyObject {
    func displayTask(title: String, isCompleted: Bool)
    func showError(message: String)
}

// Presenter
class TaskPresenter {
    weak var view: TaskViewProtocol?
    private var task: Task
    
    init(task: Task) {
        self.task = task
    }
    
    func viewDidLoad() {
        view?.displayTask(title: task.title, isCompleted: task.isCompleted)
    }
    
    func toggleCompletion() {
        task.isCompleted.toggle()
        view?.displayTask(title: task.title, isCompleted: task.isCompleted)
    }
}

// View (ViewController)
class TaskViewController: UIViewController, TaskViewProtocol {
    private let titleLabel = UILabel()
    private let checkboxButton = UIButton()
    
    var presenter: TaskPresenter!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        presenter.viewDidLoad()
    }
    
    private func setupUI() {
        view.addSubview(titleLabel)
        view.addSubview(checkboxButton)
        
        checkboxButton.addTarget(self, action: #selector(toggleTapped), for: .touchUpInside)
    }
    
    @objc private func toggleTapped() {
        presenter.toggleCompletion()
    }
    
    func displayTask(title: String, isCompleted: Bool) {
        titleLabel.text = title
        checkboxButton.isSelected = isCompleted
    }
    
    func showError(message: String) {
        // Show error
    }
}
```

### VIPER (View-Interactor-Presenter-Entity-Router)

```swift
// Entity
struct Article {
    let id: Int
    let title: String
    let content: String
}

// View Protocol
protocol ArticleViewProtocol: AnyObject {
    func showArticle(title: String, content: String)
    func showLoading()
    func hideLoading()
    func showError(message: String)
}

// Presenter Protocol
protocol ArticlePresenterProtocol: AnyObject {
    var view: ArticleViewProtocol? { get set }
    var interactor: ArticleInteractorProtocol? { get set }
    var router: ArticleRouterProtocol? { get set }
    
    func viewDidLoad()
    func didSelectShareButton()
}

// Interactor Protocol
protocol ArticleInteractorProtocol: AnyObject {
    var presenter: ArticlePresenterProtocol? { get set }
    func fetchArticle(id: Int)
}

// Router Protocol
protocol ArticleRouterProtocol: AnyObject {
    static func createModule(articleId: Int) -> UIViewController
    func navigateToShareScreen(article: Article)
}

// Presenter Implementation
class ArticlePresenter: ArticlePresenterProtocol {
    weak var view: ArticleViewProtocol?
    var interactor: ArticleInteractorProtocol?
    var router: ArticleRouterProtocol?
    
    private let articleId: Int
    private var article: Article?
    
    init(articleId: Int) {
        self.articleId = articleId
    }
    
    func viewDidLoad() {
        view?.showLoading()
        interactor?.fetchArticle(id: articleId)
    }
    
    func didSelectShareButton() {
        guard let article = article else { return }
        router?.navigateToShareScreen(article: article)
    }
}

// Interactor Implementation
class ArticleInteractor: ArticleInteractorProtocol {
    weak var presenter: ArticlePresenterProtocol?
    
    func fetchArticle(id: Int) {
        // Fetch from network
        let article = Article(id: id, title: "Title", content: "Content")
        // Notify presenter
    }
}

// Router Implementation
class ArticleRouter: ArticleRouterProtocol {
    weak var viewController: UIViewController?
    
    static func createModule(articleId: Int) -> UIViewController {
        let view = ArticleViewController()
        let presenter = ArticlePresenter(articleId: articleId)
        let interactor = ArticleInteractor()
        let router = ArticleRouter()
        
        view.presenter = presenter
        presenter.view = view
        presenter.interactor = interactor
        presenter.router = router
        interactor.presenter = presenter
        router.viewController = view
        
        return view
    }
    
    func navigateToShareScreen(article: Article) {
        // Navigate to share screen
    }
}

// View Implementation
class ArticleViewController: UIViewController, ArticleViewProtocol {
    var presenter: ArticlePresenterProtocol?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        presenter?.viewDidLoad()
    }
    
    func showArticle(title: String, content: String) {
        // Display article
    }
    
    func showLoading() {
        // Show loading indicator
    }
    
    func hideLoading() {
        // Hide loading indicator
    }
    
    func showError(message: String) {
        // Show error
    }
}
```

---

## Coordinators in iOS

Coordinators handle navigation flow.

```swift
protocol Coordinator {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get set }
    
    func start()
}

class AppCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        showHome()
    }
    
    func showHome() {
        let homeVC = HomeViewController()
        homeVC.coordinator = self
        navigationController.pushViewController(homeVC, animated: false)
    }
    
    func showUserProfile(userId: Int) {
        let profileVC = ProfileViewController(userId: userId)
        profileVC.coordinator = self
        navigationController.pushViewController(profileVC, animated: true)
    }
    
    func showSettings() {
        let settingsCoordinator = SettingsCoordinator(navigationController: navigationController)
        childCoordinators.append(settingsCoordinator)
        settingsCoordinator.start()
    }
}

class HomeViewController: UIViewController {
    weak var coordinator: AppCoordinator?
    
    @objc func profileButtonTapped() {
        coordinator?.showUserProfile(userId: 1)
    }
    
    @objc func settingsButtonTapped() {
        coordinator?.showSettings()
    }
}

class ProfileViewController: UIViewController {
    weak var coordinator: AppCoordinator?
    let userId: Int
    
    init(userId: Int) {
        self.userId = userId
        super.init(nibName: nil, bundle: nil)
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

class SettingsCoordinator: Coordinator {
    var childCoordinators = [Coordinator]()
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        let settingsVC = SettingsViewController()
        settingsVC.coordinator = self
        navigationController.pushViewController(settingsVC, animated: true)
    }
}

class SettingsViewController: UIViewController {
    weak var coordinator: SettingsCoordinator?
}
```

---

## Dependency Injection in Swift

### Constructor Injection

```swift
protocol NetworkServiceProtocol {
    func fetchData(completion: @escaping (Result<Data, Error>) -> Void)
}

class NetworkService: NetworkServiceProtocol {
    func fetchData(completion: @escaping (Result<Data, Error>) -> Void) {
        // Fetch data
    }
}

class UserRepository {
    private let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }
    
    func getUsers(completion: @escaping (Result<[User], Error>) -> Void) {
        networkService.fetchData { result in
            // Process data
        }
    }
}

// Usage
let networkService = NetworkService()
let userRepository = UserRepository(networkService: networkService)
```

### Property Injection

```swift
class ViewController: UIViewController {
    var viewModel: ViewModel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        viewModel.loadData()
    }
}

class ViewModel {
    func loadData() {
        // Load data
    }
}

// Usage
let viewController = ViewController()
viewController.viewModel = ViewModel()
```

### Service Locator Pattern

```swift
class ServiceLocator {
    static let shared = ServiceLocator()
    
    private var services: [String: Any] = [:]
    
    func register<T>(_ service: T, for type: T.Type) {
        let key = String(describing: type)
        services[key] = service
    }
    
    func resolve<T>() -> T? {
        let key = String(describing: T.self)
        return services[key] as? T
    }
}

// Register services
ServiceLocator.shared.register(NetworkService(), for: NetworkServiceProtocol.self)

// Resolve
if let networkService: NetworkServiceProtocol = ServiceLocator.shared.resolve() {
    // Use service
}
```

---

## Common Design Patterns

### Singleton

```swift
class Configuration {
    static let shared = Configuration()
    
    var apiKey: String = ""
    var baseURL: String = ""
    
    private init() { }
}

// Usage
Configuration.shared.apiKey = "abc123"
```

### Observer (Delegation)

```swift
protocol DataDelegate: AnyObject {
    func didReceiveData(_ data: Data)
}

class DataManager {
    weak var delegate: DataDelegate?
    
    func fetchData() {
        let data = Data()
        delegate?.didReceiveData(data)
    }
}

class ViewController: UIViewController, DataDelegate {
    let dataManager = DataManager()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        dataManager.delegate = self
    }
    
    func didReceiveData(_ data: Data) {
        print("Received data")
    }
}
```

### Factory

```swift
protocol Vehicle {
    func drive()
}

class Car: Vehicle {
    func drive() { print("Driving car") }
}

class Bike: Vehicle {
    func drive() { print("Riding bike") }
}

enum VehicleType {
    case car, bike
}

class VehicleFactory {
    static func createVehicle(type: VehicleType) -> Vehicle {
        switch type {
        case .car:
            return Car()
        case .bike:
            return Bike()
        }
    }
}

// Usage
let vehicle = VehicleFactory.createVehicle(type: .car)
vehicle.drive()
```

### Adapter

```swift
protocol PaymentProcessor {
    func processPayment(amount: Double)
}

class StripePayment {
    func charge(amount: Double) {
        print("Charging \(amount) via Stripe")
    }
}

class StripeAdapter: PaymentProcessor {
    private let stripe: StripePayment
    
    init(stripe: StripePayment) {
        self.stripe = stripe
    }
    
    func processPayment(amount: Double) {
        stripe.charge(amount: amount)
    }
}
```

### Builder

```swift
class User2 {
    let firstName: String
    let lastName: String
    let email: String?
    let age: Int?
    let address: String?
    
    private init(builder: Builder) {
        self.firstName = builder.firstName
        self.lastName = builder.lastName
        self.email = builder.email
        self.age = builder.age
        self.address = builder.address
    }
    
    class Builder {
        let firstName: String
        let lastName: String
        var email: String?
        var age: Int?
        var address: String?
        
        init(firstName: String, lastName: String) {
            self.firstName = firstName
            self.lastName = lastName
        }
        
        func with(email: String) -> Builder {
            self.email = email
            return self
        }
        
        func with(age: Int) -> Builder {
            self.age = age
            return self
        }
        
        func with(address: String) -> Builder {
            self.address = address
            return self
        }
        
        func build() -> User2 {
            return User2(builder: self)
        }
    }
}

// Usage
let user = User2.Builder(firstName: "John", lastName: "Doe")
    .with(email: "john@example.com")
    .with(age: 25)
    .build()
```

---

## SOLID Principles in iOS Development

### Single Responsibility Principle

```swift
// BAD: Multiple responsibilities
class UserManager {
    func fetchUser() { }
    func saveUser() { }
    func validateUser() { }
    func sendEmail() { }
}

// GOOD: Single responsibility
class UserRepository {
    func fetchUser() { }
    func saveUser() { }
}

class UserValidator {
    func validateUser() { }
}

class EmailService {
    func sendEmail() { }
}
```

### Open/Closed Principle

```swift
// Open for extension, closed for modification

protocol Shape {
    func area() -> Double
}

class Circle: Shape {
    let radius: Double
    
    init(radius: Double) {
        self.radius = radius
    }
    
    func area() -> Double {
        return .pi * radius * radius
    }
}

class Rectangle2: Shape {
    let width: Double
    let height: Double
    
    init(width: Double, height: Double) {
        self.width = width
        self.height = height
    }
    
    func area() -> Double {
        return width * height
    }
}

class AreaCalculator {
    func totalArea(shapes: [Shape]) -> Double {
        return shapes.reduce(0) { $0 + $1.area() }
    }
}
```

### Liskov Substitution Principle

```swift
// Subtypes must be substitutable for their base types

class Bird2 {
    func eat() { }
}

class FlyingBird: Bird2 {
    func fly() { }
}

class Sparrow: FlyingBird {
    override func eat() { }
    override func fly() { }
}

class Penguin2: Bird2 {
    override func eat() { }
    // Doesn't inherit fly() - correct!
}
```

### Interface Segregation Principle

```swift
// Clients shouldn't depend on interfaces they don't use

// BAD: Fat interface
protocol Worker {
    func work()
    func eat()
    func sleep()
}

// GOOD: Segregated interfaces
protocol Workable {
    func work()
}

protocol Eatable {
    func eat()
}

protocol Sleepable {
    func sleep()
}

class Human: Workable, Eatable, Sleepable {
    func work() { }
    func eat() { }
    func sleep() { }
}

class Robot: Workable {
    func work() { }
}
```

### Dependency Inversion Principle

```swift
// Depend on abstractions, not concretions

// BAD: High-level module depends on low-level module
class MySQLDatabase {
    func save() { }
}

class UserService {
    let database = MySQLDatabase()
}

// GOOD: Both depend on abstraction
protocol Database {
    func save()
}

class MySQLDatabase2: Database {
    func save() { }
}

class PostgreSQLDatabase: Database {
    func save() { }
}

class UserService2 {
    let database: Database
    
    init(database: Database) {
        self.database = database
    }
}
```

---

## Clean Architecture in iOS

```swift
// Domain Layer - Entities
struct Post2 {
    let id: Int
    let title: String
    let content: String
}

// Domain Layer - Use Case Protocol
protocol FetchPostsUseCase {
    func execute(completion: @escaping (Result<[Post2], Error>) -> Void)
}

// Data Layer - Repository Protocol
protocol PostRepository {
    func getPosts(completion: @escaping (Result<[Post2], Error>) -> Void)
}

// Data Layer - Repository Implementation
class PostRepositoryImpl: PostRepository {
    private let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }
    
    func getPosts(completion: @escaping (Result<[Post2], Error>) -> Void) {
        networkService.fetchData { result in
            // Map data to domain model
            completion(.success([]))
        }
    }
}

// Domain Layer - Use Case Implementation
class FetchPostsUseCaseImpl: FetchPostsUseCase {
    private let repository: PostRepository
    
    init(repository: PostRepository) {
        self.repository = repository
    }
    
    func execute(completion: @escaping (Result<[Post2], Error>) -> Void) {
        repository.getPosts(completion: completion)
    }
}

// Presentation Layer - ViewModel
class PostsViewModel2 {
    private let fetchPostsUseCase: FetchPostsUseCase
    var posts: [Post2] = []
    
    init(fetchPostsUseCase: FetchPostsUseCase) {
        self.fetchPostsUseCase = fetchPostsUseCase
    }
    
    func loadPosts() {
        fetchPostsUseCase.execute { [weak self] result in
            switch result {
            case .success(let posts):
                self?.posts = posts
            case .failure(let error):
                print("Error: \(error)")
            }
        }
    }
}
```

---

[← Previous: Multithreading & Concurrency](multithreading-concurrency.md) | [Next: Dependency Management →](dependency-management.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: Explain the difference between MVC, MVVM, and VIPER

**Answer:**

**MVC (Model-View-Controller):**
- Apple's default pattern
- Controller mediates between Model and View
- Problem: Massive View Controllers
- Best for: Simple apps

```swift
Model ← → Controller ← → View
```

**MVVM (Model-View-ViewModel):**
- ViewModel handles business logic
- View binds to ViewModel
- Testable (ViewModel doesn't depend on View)
- Best for: Medium to large apps

```swift
Model ← → ViewModel ← → View
```

**VIPER (View-Interactor-Presenter-Entity-Router):**
- Highly modular and testable
- Each component has single responsibility
- More boilerplate code
- Best for: Large, complex apps with teams

```swift
View ← → Presenter ← → Interactor ← → Entity
            ↓
        Router
```

**When to Choose:**
- **MVC**: Quick prototypes, simple apps
- **MVVM**: Production apps, need testability
- **VIPER**: Large teams, complex business logic

### Q2: How do you implement dependency injection in iOS?

**Answer:**

**1. Constructor Injection (Preferred):**

```swift
protocol NetworkService {
    func fetch(url: URL) -> Data
}

class UserRepository {
    private let networkService: NetworkService
    
    init(networkService: NetworkService) {
        self.networkService = networkService
    }
}

// Usage
let network = NetworkServiceImpl()
let repository = UserRepository(networkService: network)
```

**Benefits:**
- Dependencies are explicit
- Immutable after initialization
- Easy to test

**2. Property Injection:**

```swift
class ViewController: UIViewController {
    var viewModel: ViewModel!  // Injected after creation
}

let vc = ViewController()
vc.viewModel = ViewModel()
```

**3. Method Injection:**

```swift
func processData(using parser: DataParser) {
    // Parser injected when needed
}
```

**Why Use DI:**
- Testability (inject mocks)
- Flexibility (swap implementations)
- Loose coupling
- Easier maintenance

**Example Test:**
```swift
class MockNetworkService: NetworkService {
    func fetch(url: URL) -> Data {
        return mockData  // Controlled data for testing
    }
}

let mockNetwork = MockNetworkService()
let repository = UserRepository(networkService: mockNetwork)
// Test repository with predictable data
```

### Q3: What are the SOLID principles? Give iOS examples

**Answer:**

**S - Single Responsibility:**
Each class should have one job.

```swift
// BAD
class UserManager {
    func fetchUser() { }
    func validateEmail() { }
    func sendEmail() { }
}

// GOOD
class UserRepository { func fetchUser() { } }
class EmailValidator { func validate() { } }
class EmailService { func send() { } }
```

**O - Open/Closed:**
Open for extension, closed for modification.

```swift
protocol PaymentMethod {
    func pay(amount: Double)
}

class CreditCard: PaymentMethod {
    func pay(amount: Double) { }
}

class PayPal: PaymentMethod {
    func pay(amount: Double) { }
}

// Can add new payment methods without modifying existing code
```

**L - Liskov Substitution:**
Subtypes must be substitutable for base types.

```swift
protocol Bird {
    func eat()
}

protocol FlyingBird: Bird {
    func fly()
}

class Sparrow: FlyingBird {
    func eat() { }
    func fly() { }
}

class Penguin: Bird {
    func eat() { }
    // Doesn't conform to FlyingBird - correct!
}
```

**I - Interface Segregation:**
Don't force clients to depend on unused methods.

```swift
// BAD
protocol Worker {
    func work()
    func eat()
}

// GOOD
protocol Workable { func work() }
protocol Eatable { func eat() }

class Human: Workable, Eatable { }
class Robot: Workable { }
```

**D - Dependency Inversion:**
Depend on abstractions, not concretions.

```swift
// BAD
class UserService {
    let database = SQLiteDatabase()  // Concrete class
}

// GOOD
protocol Database { }

class UserService {
    let database: Database  // Protocol
    init(database: Database) {
        self.database = database
    }
}
```

### Q4: Explain the Coordinator pattern

**Answer:**

**Purpose:** Separates navigation logic from view controllers.

**Benefits:**
- Reusable view controllers
- Testable navigation
- Clear navigation flow
- Better deep linking

**Implementation:**

```swift
protocol Coordinator {
    var childCoordinators: [Coordinator] { get set }
    var navigationController: UINavigationController { get set }
    func start()
}

class AppCoordinator: Coordinator {
    var childCoordinators: [Coordinator] = []
    var navigationController: UINavigationController
    
    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }
    
    func start() {
        showLogin()
    }
    
    func showLogin() {
        let loginVC = LoginViewController()
        loginVC.coordinator = self
        navigationController.pushViewController(loginVC, animated: false)
    }
    
    func userDidLogin() {
        showHome()
    }
    
    func showHome() {
        let homeVC = HomeViewController()
        homeVC.coordinator = self
        navigationController.pushViewController(homeVC, animated: true)
    }
}

class LoginViewController: UIViewController {
    weak var coordinator: AppCoordinator?
    
    func loginButtonTapped() {
        // Perform login
        coordinator?.userDidLogin()
    }
}
```

**When to Use:**
- Complex navigation flows
- Deep linking requirements
- Large apps with many screens
- Need to support multiple flows (onboarding, main app, settings)

### Q5: What's the difference between Singleton and Dependency Injection?

**Answer:**

**Singleton:**

```swift
class NetworkManager {
    static let shared = NetworkManager()
    private init() { }
}

// Usage
NetworkManager.shared.request()
```

**Pros:**
- Global access
- Single instance guaranteed
- Easy to use

**Cons:**
- Hard to test
- Hidden dependencies
- Tight coupling
- Can't swap implementation

**Dependency Injection:**

```swift
protocol NetworkService { }

class NetworkManager: NetworkService { }

class UserRepository {
    let network: NetworkService
    init(network: NetworkService) {
        self.network = network
    }
}

// Usage
let network = NetworkManager()
let repository = UserRepository(network: network)
```

**Pros:**
- Testable
- Flexible
- Explicit dependencies
- Can swap implementations

**Cons:**
- More setup code
- Need DI container for large apps

**When to Use Each:**
- **Singleton**: Configuration, logging (truly global state)
- **DI**: Business logic, services, repositories

**Hybrid Approach:**
```swift
class NetworkManager {
    static let shared = NetworkManager()
    
    // But allow injection for testing
    func makeUserRepository(network: NetworkService = NetworkManager.shared) -> UserRepository {
        return UserRepository(network: network)
    }
}
```

### Q6: Explain Clean Architecture for iOS

**Answer:**

Clean Architecture separates concerns into layers with clear dependencies.

**Layers (Inner to Outer):**

```
Entities (Domain Models)
    ↑
Use Cases (Business Logic)
    ↑
Interface Adapters (Presenters, Controllers)
    ↑
Frameworks & Drivers (UI, Network, Database)
```

**Dependency Rule:** Outer layers depend on inner layers, never reverse.

**Implementation:**

```swift
// 1. Domain Layer - Entities
struct User {
    let id: Int
    let name: String
    let email: String
}

// 2. Domain Layer - Use Case Protocol
protocol FetchUserUseCase {
    func execute(id: Int) async throws -> User
}

// 3. Data Layer - Repository Protocol
protocol UserRepository {
    func getUser(id: Int) async throws -> User
}

// 4. Data Layer - Repository Implementation
class UserRepositoryImpl: UserRepository {
    private let networkService: NetworkService
    private let database: Database
    
    init(networkService: NetworkService, database: Database) {
        self.networkService = networkService
        self.database = database
    }
    
    func getUser(id: Int) async throws -> User {
        // Try cache first
        if let cached = try? await database.getUser(id: id) {
            return cached
        }
        
        // Fetch from network
        let user = try await networkService.fetchUser(id: id)
        try? await database.saveUser(user)
        return user
    }
}

// 5. Domain Layer - Use Case Implementation
class FetchUserUseCaseImpl: FetchUserUseCase {
    private let repository: UserRepository
    
    init(repository: UserRepository) {
        self.repository = repository
    }
    
    func execute(id: Int) async throws -> User {
        return try await repository.getUser(id: id)
    }
}

// 6. Presentation Layer - ViewModel
class UserViewModel {
    private let fetchUserUseCase: FetchUserUseCase
    @Published var user: User?
    
    init(fetchUserUseCase: FetchUserUseCase) {
        self.fetchUserUseCase = fetchUserUseCase
    }
    
    func loadUser(id: Int) async {
        user = try? await fetchUserUseCase.execute(id: id)
    }
}
```

**Benefits:**
- Testable (each layer independently)
- Flexible (swap implementations)
- Maintainable (clear separation)
- Scalable (easy to add features)

**When to Use:**
- Large, complex apps
- Long-term projects
- Need high testability
- Multiple developers/teams

### Q7: What design patterns are commonly used in iOS?

**Answer:**

**1. Delegation:**
- UITableViewDelegate, UITextFieldDelegate
- One-to-one communication

**2. Observer:**
- NotificationCenter
- KVO (Key-Value Observing)
- Combine publishers

**3. Singleton:**
- UserDefaults.standard
- URLSession.shared
- FileManager.default

**4. Factory:**
- Creating objects without specifying exact class

```swift
class ViewControllerFactory {
    static func makeHomeVC() -> UIViewController {
        return HomeViewController()
    }
}
```

**5. Builder:**
- Constructing complex objects

```swift
URLRequest.Builder()
    .url(url)
    .method(.POST)
    .headers(["Auth": "token"])
    .build()
```

**6. Adapter:**
- Converting one interface to another

```swift
class LegacyAPIAdapter: ModernAPI {
    private let legacyAPI: LegacyAPI
    
    func fetchData() {
        legacyAPI.getData()  // Adapt old API to new interface
    }
}
```

**7. Coordinator:**
- Managing navigation flow
- Decouples view controllers

**8. Repository:**
- Abstracts data source

```swift
protocol UserRepository {
    func getUsers() -> [User]
}
```

---

[← Previous: Multithreading & Concurrency](multithreading-concurrency.md) | [Next: Dependency Management →](dependency-management.md)

[Back to Main](../README.md)
