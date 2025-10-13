# Networking in iOS

[← Back to Main](../README.md) | [Previous: Data Persistence](data-persistence.md) | [Next: Multithreading & Concurrency →](multithreading-concurrency.md)

## Table of Contents
- [URLSession Basics](#urlsession-basics)
- [REST APIs Integration](#rest-apis-integration)
- [Codable Protocol](#codable-protocol-in-swift)
- [JSON Parsing & Error Handling](#json-parsing--error-handling)
- [Combine Framework for Networking](#combine-framework-for-networking)
- [Third-Party Libraries](#third-party-libraries)
- [Handling Offline Data & Caching](#handling-offline-data--caching)

---

## URLSession Basics

URLSession is Apple's API for making network requests.

### Simple GET Request

```swift
import Foundation

func fetchData() {
    let url = URL(string: "https://api.example.com/users")!
    
    let task = URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            print("Error: \(error.localizedDescription)")
            return
        }
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            print("Invalid response")
            return
        }
        
        if let data = data {
            print("Received data: \(data)")
        }
    }
    
    task.resume()
}
```

### URLRequest Configuration

```swift
func makeRequest() {
    let url = URL(string: "https://api.example.com/data")!
    var request = URLRequest(url: url)
    
    request.httpMethod = "POST"
    request.setValue("application/json", forHTTPHeaderField: "Content-Type")
    request.setValue("Bearer token123", forHTTPHeaderField: "Authorization")
    request.timeoutInterval = 30
    request.cachePolicy = .reloadIgnoringLocalCacheData
    
    let body = ["name": "John", "age": 25]
    request.httpBody = try? JSONSerialization.data(withJSONObject: body)
    
    let task = URLSession.shared.dataTask(with: request) { data, response, error in
        // Handle response
    }
    
    task.resume()
}
```

### Custom URLSession Configuration

```swift
class NetworkManager {
    static let shared = NetworkManager()
    
    private let session: URLSession
    
    init() {
        let configuration = URLSessionConfiguration.default
        configuration.timeoutIntervalForRequest = 30
        configuration.timeoutIntervalForResource = 60
        configuration.httpMaximumConnectionsPerHost = 5
        configuration.requestCachePolicy = .reloadIgnoringLocalCacheData
        configuration.urlCache = URLCache(
            memoryCapacity: 10 * 1024 * 1024, // 10 MB
            diskCapacity: 50 * 1024 * 1024, // 50 MB
            diskPath: nil
        )
        
        session = URLSession(configuration: configuration)
    }
    
    func request(url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        let task = session.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NetworkError.noData))
                return
            }
            
            completion(.success(data))
        }
        
        task.resume()
    }
}

enum NetworkError: Error {
    case noData
    case invalidResponse
    case decodingError
}
```

---

## REST APIs Integration

### Network Service Layer

```swift
protocol NetworkServiceProtocol {
    func request<T: Decodable>(endpoint: Endpoint, completion: @escaping (Result<T, Error>) -> Void)
}

struct Endpoint {
    let path: String
    let method: HTTPMethod
    let headers: [String: String]?
    let body: Data?
    
    enum HTTPMethod: String {
        case get = "GET"
        case post = "POST"
        case put = "PUT"
        case delete = "DELETE"
        case patch = "PATCH"
    }
}

class NetworkService: NetworkServiceProtocol {
    private let baseURL: String
    private let session: URLSession
    
    init(baseURL: String, session: URLSession = .shared) {
        self.baseURL = baseURL
        self.session = session
    }
    
    func request<T: Decodable>(endpoint: Endpoint, completion: @escaping (Result<T, Error>) -> Void) {
        guard let url = URL(string: baseURL + endpoint.path) else {
            completion(.failure(NetworkError.invalidURL))
            return
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = endpoint.method.rawValue
        request.httpBody = endpoint.body
        
        endpoint.headers?.forEach { key, value in
            request.setValue(value, forHTTPHeaderField: key)
        }
        
        let task = session.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let httpResponse = response as? HTTPURLResponse else {
                completion(.failure(NetworkError.invalidResponse))
                return
            }
            
            guard (200...299).contains(httpResponse.statusCode) else {
                completion(.failure(NetworkError.httpError(statusCode: httpResponse.statusCode)))
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
                completion(.failure(NetworkError.decodingError(error)))
            }
        }
        
        task.resume()
    }
}

enum NetworkError: Error {
    case invalidURL
    case noData
    case invalidResponse
    case httpError(statusCode: Int)
    case decodingError(Error)
}
```

### API Client

```swift
class APIClient {
    private let networkService: NetworkServiceProtocol
    
    init(networkService: NetworkServiceProtocol) {
        self.networkService = networkService
    }
    
    func getUsers(completion: @escaping (Result<[User], Error>) -> Void) {
        let endpoint = Endpoint(
            path: "/users",
            method: .get,
            headers: ["Content-Type": "application/json"],
            body: nil
        )
        
        networkService.request(endpoint: endpoint, completion: completion)
    }
    
    func createUser(_ user: User, completion: @escaping (Result<User, Error>) -> Void) {
        let body = try? JSONEncoder().encode(user)
        
        let endpoint = Endpoint(
            path: "/users",
            method: .post,
            headers: ["Content-Type": "application/json"],
            body: body
        )
        
        networkService.request(endpoint: endpoint, completion: completion)
    }
    
    func updateUser(_ user: User, completion: @escaping (Result<User, Error>) -> Void) {
        let body = try? JSONEncoder().encode(user)
        
        let endpoint = Endpoint(
            path: "/users/\(user.id)",
            method: .put,
            headers: ["Content-Type": "application/json"],
            body: body
        )
        
        networkService.request(endpoint: endpoint, completion: completion)
    }
    
    func deleteUser(id: Int, completion: @escaping (Result<Void, Error>) -> Void) {
        let endpoint = Endpoint(
            path: "/users/\(id)",
            method: .delete,
            headers: nil,
            body: nil
        )
        
        networkService.request(endpoint: endpoint) { (result: Result<EmptyResponse, Error>) in
            switch result {
            case .success:
                completion(.success(()))
            case .failure(let error):
                completion(.failure(error))
            }
        }
    }
}

struct EmptyResponse: Codable {}

// Usage
let networkService = NetworkService(baseURL: "https://api.example.com")
let apiClient = APIClient(networkService: networkService)

apiClient.getUsers { result in
    switch result {
    case .success(let users):
        print("Users: \(users)")
    case .failure(let error):
        print("Error: \(error)")
    }
}
```

---

## Codable Protocol in Swift

### Basic Codable

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
    let age: Int
}

// Encoding
let user = User(id: 1, name: "John", email: "john@example.com", age: 25)
let encoder = JSONEncoder()
encoder.outputFormatting = .prettyPrinted

if let jsonData = try? encoder.encode(user),
   let jsonString = String(data: jsonData, encoding: .utf8) {
    print(jsonString)
}

// Decoding
let jsonString = """
{
    "id": 1,
    "name": "John",
    "email": "john@example.com",
    "age": 25
}
"""

if let jsonData = jsonString.data(using: .utf8) {
    let decoder = JSONDecoder()
    if let user = try? decoder.decode(User.self, from: jsonData) {
        print(user)
    }
}
```

### Custom Coding Keys

```swift
struct Product: Codable {
    let id: Int
    let productName: String
    let productPrice: Double
    let isAvailable: Bool
    
    enum CodingKeys: String, CodingKey {
        case id
        case productName = "product_name"
        case productPrice = "product_price"
        case isAvailable = "is_available"
    }
}

let json = """
{
    "id": 1,
    "product_name": "iPhone",
    "product_price": 999.99,
    "is_available": true
}
"""

if let data = json.data(using: .utf8),
   let product = try? JSONDecoder().decode(Product.self, from: data) {
    print(product.productName) // "iPhone"
}
```

### Custom Encoding/Decoding

```swift
struct Article: Codable {
    let id: Int
    let title: String
    let publishDate: Date
    let tags: [String]
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        id = try container.decode(Int.self, forKey: .id)
        title = try container.decode(String.self, forKey: .title)
        tags = try container.decode([String].self, forKey: .tags)
        
        // Custom date decoding
        let dateString = try container.decode(String.self, forKey: .publishDate)
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
        
        guard let date = formatter.date(from: dateString) else {
            throw DecodingError.dataCorruptedError(
                forKey: .publishDate,
                in: container,
                debugDescription: "Date string does not match format"
            )
        }
        
        publishDate = date
    }
    
    func encode(to encoder: Encoder) throws {
        var container = encoder.container(keyedBy: CodingKeys.self)
        
        try container.encode(id, forKey: .id)
        try container.encode(title, forKey: .title)
        try container.encode(tags, forKey: .tags)
        
        // Custom date encoding
        let formatter = DateFormatter()
        formatter.dateFormat = "yyyy-MM-dd"
        let dateString = formatter.string(from: publishDate)
        try container.encode(dateString, forKey: .publishDate)
    }
    
    enum CodingKeys: String, CodingKey {
        case id, title, publishDate, tags
    }
}
```

### Nested Objects

```swift
struct BlogPost: Codable {
    let id: Int
    let title: String
    let author: Author
    let comments: [Comment]
    
    struct Author: Codable {
        let id: Int
        let name: String
        let email: String
    }
    
    struct Comment: Codable {
        let id: Int
        let text: String
        let user: String
    }
}

let json = """
{
    "id": 1,
    "title": "My First Post",
    "author": {
        "id": 1,
        "name": "John",
        "email": "john@example.com"
    },
    "comments": [
        {
            "id": 1,
            "text": "Great post!",
            "user": "Alice"
        }
    ]
}
"""

if let data = json.data(using: .utf8),
   let post = try? JSONDecoder().decode(BlogPost.self, from: data) {
    print(post.author.name)
}
```

---

## JSON Parsing & Error Handling

### Comprehensive Error Handling

```swift
enum APIError: Error {
    case invalidURL
    case requestFailed(Error)
    case invalidResponse
    case httpError(Int)
    case decodingFailed(Error)
    case encodingFailed(Error)
    
    var localizedDescription: String {
        switch self {
        case .invalidURL:
            return "The URL is invalid"
        case .requestFailed(let error):
            return "Request failed: \(error.localizedDescription)"
        case .invalidResponse:
            return "Invalid server response"
        case .httpError(let statusCode):
            return "HTTP error with status code: \(statusCode)"
        case .decodingFailed(let error):
            return "Failed to decode response: \(error.localizedDescription)"
        case .encodingFailed(let error):
            return "Failed to encode request: \(error.localizedDescription)"
        }
    }
}

class RobustNetworkService {
    func fetch<T: Decodable>(url: URL, completion: @escaping (Result<T, APIError>) -> Void) {
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            // Handle network error
            if let error = error {
                DispatchQueue.main.async {
                    completion(.failure(.requestFailed(error)))
                }
                return
            }
            
            // Validate HTTP response
            guard let httpResponse = response as? HTTPURLResponse else {
                DispatchQueue.main.async {
                    completion(.failure(.invalidResponse))
                }
                return
            }
            
            // Check status code
            guard (200...299).contains(httpResponse.statusCode) else {
                DispatchQueue.main.async {
                    completion(.failure(.httpError(httpResponse.statusCode)))
                }
                return
            }
            
            // Validate data
            guard let data = data else {
                DispatchQueue.main.async {
                    completion(.failure(.invalidResponse))
                }
                return
            }
            
            // Decode JSON
            do {
                let decoder = JSONDecoder()
                decoder.keyDecodingStrategy = .convertFromSnakeCase
                let decoded = try decoder.decode(T.self, from: data)
                
                DispatchQueue.main.async {
                    completion(.success(decoded))
                }
            } catch {
                DispatchQueue.main.async {
                    completion(.failure(.decodingFailed(error)))
                }
            }
        }
        
        task.resume()
    }
}
```

### Async/Await Networking (iOS 15+)

```swift
class ModernNetworkService {
    func fetch<T: Decodable>(url: URL) async throws -> T {
        let (data, response) = try await URLSession.shared.data(from: url)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw APIError.invalidResponse
        }
        
        do {
            let decoder = JSONDecoder()
            return try decoder.decode(T.self, from: data)
        } catch {
            throw APIError.decodingFailed(error)
        }
    }
    
    func post<T: Encodable, R: Decodable>(url: URL, body: T) async throws -> R {
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        
        do {
            request.httpBody = try JSONEncoder().encode(body)
        } catch {
            throw APIError.encodingFailed(error)
        }
        
        let (data, response) = try await URLSession.shared.data(for: request)
        
        guard let httpResponse = response as? HTTPURLResponse,
              (200...299).contains(httpResponse.statusCode) else {
            throw APIError.invalidResponse
        }
        
        do {
            let decoder = JSONDecoder()
            return try decoder.decode(R.self, from: data)
        } catch {
            throw APIError.decodingFailed(error)
        }
    }
}

// Usage with async/await
Task {
    do {
        let service = ModernNetworkService()
        let users: [User] = try await service.fetch(url: URL(string: "https://api.example.com/users")!)
        print("Fetched \(users.count) users")
    } catch {
        print("Error: \(error)")
    }
}
```

---

## Combine Framework for Networking

### Basic Combine Networking

```swift
import Combine

class CombineNetworkService {
    func fetch<T: Decodable>(url: URL) -> AnyPublisher<T, Error> {
        return URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: T.self, decoder: JSONDecoder())
            .receive(on: DispatchQueue.main)
            .eraseToAnyPublisher()
    }
}

// Usage
class ViewModel {
    private var cancellables = Set<AnyCancellable>()
    private let networkService = CombineNetworkService()
    
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func fetchUsers() {
        isLoading = true
        
        networkService.fetch(url: URL(string: "https://api.example.com/users")!)
            .sink { [weak self] completion in
                self?.isLoading = false
                
                switch completion {
                case .finished:
                    break
                case .failure(let error):
                    self?.errorMessage = error.localizedDescription
                }
            } receiveValue: { [weak self] (users: [User]) in
                self?.users = users
            }
            .store(in: &cancellables)
    }
}
```

### Advanced Combine Patterns

```swift
class AdvancedCombineService {
    private var cancellables = Set<AnyCancellable>()
    
    func searchUsers(query: String) -> AnyPublisher<[User], Never> {
        let url = URL(string: "https://api.example.com/search?q=\(query)")!
        
        return URLSession.shared.dataTaskPublisher(for: url)
            .map(\.data)
            .decode(type: [User].self, decoder: JSONDecoder())
            .replaceError(with: [])
            .eraseToAnyPublisher()
    }
    
    func observeSearchField(_ publisher: Published<String>.Publisher) {
        publisher
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .filter { $0.count > 2 }
            .flatMap { query in
                self.searchUsers(query: query)
            }
            .sink { users in
                print("Found \(users.count) users")
            }
            .store(in: &cancellables)
    }
}
```

---

## Third-Party Libraries

### Alamofire

```swift
import Alamofire

class AlamofireService {
    func fetchUsers(completion: @escaping (Result<[User], Error>) -> Void) {
        AF.request("https://api.example.com/users")
            .validate()
            .responseDecodable(of: [User].self) { response in
                switch response.result {
                case .success(let users):
                    completion(.success(users))
                case .failure(let error):
                    completion(.failure(error))
                }
            }
    }
    
    func createUser(_ user: User, completion: @escaping (Result<User, Error>) -> Void) {
        AF.request(
            "https://api.example.com/users",
            method: .post,
            parameters: user,
            encoder: JSONParameterEncoder.default
        )
        .validate()
        .responseDecodable(of: User.self) { response in
            completion(response.result.mapError { $0 as Error })
        }
    }
}
```

---

## Handling Offline Data & Caching

### URLCache

```swift
class CachingNetworkService {
    private let session: URLSession
    
    init() {
        let configuration = URLSessionConfiguration.default
        
        let cache = URLCache(
            memoryCapacity: 20 * 1024 * 1024, // 20 MB
            diskCapacity: 100 * 1024 * 1024, // 100 MB
            diskPath: "myCache"
        )
        
        configuration.urlCache = cache
        configuration.requestCachePolicy = .returnCacheDataElseLoad
        
        session = URLSession(configuration: configuration)
    }
    
    func fetch(url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        let request = URLRequest(url: url, cachePolicy: .returnCacheDataElseLoad)
        
        session.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NetworkError.noData))
                return
            }
            
            completion(.success(data))
        }.resume()
    }
}
```

### Custom Caching Strategy

```swift
class DataCache<T: Codable> {
    private var cache: [String: CacheEntry<T>] = [:]
    private let expirationTime: TimeInterval
    
    init(expirationTime: TimeInterval = 300) { // 5 minutes
        self.expirationTime = expirationTime
    }
    
    func set(_ value: T, forKey key: String) {
        let entry = CacheEntry(value: value, expirationDate: Date().addingTimeInterval(expirationTime))
        cache[key] = entry
    }
    
    func get(forKey key: String) -> T? {
        guard let entry = cache[key] else { return nil }
        
        if entry.isExpired {
            cache.removeValue(forKey: key)
            return nil
        }
        
        return entry.value
    }
    
    func clear() {
        cache.removeAll()
    }
    
    private struct CacheEntry<T> {
        let value: T
        let expirationDate: Date
        
        var isExpired: Bool {
            return Date() > expirationDate
        }
    }
}

// Usage with NetworkService
class CachedAPIClient {
    private let networkService: NetworkService
    private let cache = DataCache<[User]>()
    
    init(networkService: NetworkService) {
        self.networkService = networkService
    }
    
    func getUsers(forceRefresh: Bool = false, completion: @escaping (Result<[User], Error>) -> Void) {
        let cacheKey = "users"
        
        if !forceRefresh, let cachedUsers = cache.get(forKey: cacheKey) {
            completion(.success(cachedUsers))
            return
        }
        
        let endpoint = Endpoint(path: "/users", method: .get, headers: nil, body: nil)
        networkService.request(endpoint: endpoint) { [weak self] (result: Result<[User], Error>) in
            if case .success(let users) = result {
                self?.cache.set(users, forKey: cacheKey)
            }
            completion(result)
        }
    }
}
```

### Interview Questions & Answers

**Q1: What's the difference between URLSession.shared and custom URLSession?**

**Answer:** 

**URLSession.shared:**
- Uses default configuration
- Cannot be customized
- Good for simple requests
- Shared across app

**Custom URLSession:**
- Full control over configuration
- Custom timeouts, cache policy
- Can set custom delegate
- Better for production apps

**Example:**

```swift
// Standard
URLSession.shared.dataTask(with: url) { data, response, error in
    // Handle response
}.resume()

// Custom
let configuration = URLSessionConfiguration.default
configuration.timeoutIntervalForRequest = 30
configuration.httpMaximumConnectionsPerHost = 5
configuration.requestCachePolicy = .reloadIgnoringLocalCacheData

let session = URLSession(configuration: configuration)
```

**Q2: How do you handle offline scenarios?**

**Answer:**

**1. Check Network Reachability:**
```swift
import Network

class NetworkMonitor {
    static let shared = NetworkMonitor()
    let monitor = NWPathMonitor()
    private(set) var isConnected = false
    
    func startMonitoring() {
        monitor.pathUpdateHandler = { path in
            self.isConnected = path.status == .satisfied
            
            if self.isConnected {
                print("Network available")
            } else {
                print("Network unavailable")
            }
        }
        
        monitor.start(queue: DispatchQueue.global())
    }
}
```

**2. URLCache for Caching:**
```swift
let configuration = URLSessionConfiguration.default
let cache = URLCache(
    memoryCapacity: 20 * 1024 * 1024,  // 20 MB
    diskCapacity: 100 * 1024 * 1024     // 100 MB
)
configuration.urlCache = cache
configuration.requestCachePolicy = .returnCacheDataElseLoad
```

**3. Custom Offline Strategy:**
```swift
class OfflineManager {
    func fetchData(url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        if NetworkMonitor.shared.isConnected {
            // Fetch from network
            networkRequest(url, completion: completion)
        } else {
            // Return cached data
            if let cached = loadFromCache(url: url) {
                completion(.success(cached))
            } else {
                completion(.failure(OfflineError.noCache))
            }
        }
    }
    
    func loadFromCache(url: URL) -> Data? { return nil }
    func networkRequest(_ url: URL, completion: @escaping (Result<Data, Error>) -> Void) { }
}

enum OfflineError: Error {
    case noCache
}
```

**4. Queue Failed Requests:**
```swift
class RequestQueue {
    private var pendingRequests: [(URL, Data)] = []
    
    func queueRequest(url: URL, data: Data) {
        pendingRequests.append((url, data))
    }
    
    func retryPendingRequests() {
        guard NetworkMonitor.shared.isConnected else { return }
        
        for (url, data) in pendingRequests {
            // Retry request
        }
        pendingRequests.removeAll()
    }
}
```

**Q3: Explain the benefits of Combine for networking**

**Answer:**

**Benefits:**
- Declarative syntax
- Composable operators
- Automatic memory management
- Built-in error handling
- SwiftUI integration
- Reactive updates

**Example:**

```swift
// Traditional (callback hell)
networkService.fetchUser { result in
    switch result {
    case .success(let user):
        self.networkService.fetchPosts(userId: user.id) { result in
            switch result {
            case .success(let posts):
                // Nested callbacks
            case .failure(let error):
                // Handle error
            }
        }
    case .failure(let error):
        // Handle error
    }
}

// Combine (clean and composable)
networkService.fetchUser()
    .flatMap { user in
        self.networkService.fetchPosts(userId: user.id)
    }
    .receive(on: DispatchQueue.main)
    .sink { completion in
        if case .failure(let error) = completion {
            print("Error: \(error)")
        }
    } receiveValue: { posts in
        self.posts = posts
    }
    .store(in: &cancellables)
```

**Q4: How do you implement retry logic for failed network requests?**

**Answer:**

**Simple Retry:**
```swift
func fetchWithRetry(url: URL, maxRetries: Int = 3, completion: @escaping (Result<Data, Error>) -> Void) {
    fetchData(url: url, attempt: 0, maxRetries: maxRetries, completion: completion)
}

private func fetchData(url: URL, attempt: Int, maxRetries: Int, completion: @escaping (Result<Data, Error>) -> Void) {
    URLSession.shared.dataTask(with: url) { data, response, error in
        if let error = error {
            if attempt < maxRetries {
                // Retry with exponential backoff
                let delay = pow(2.0, Double(attempt))
                DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
                    self.fetchData(url: url, attempt: attempt + 1, maxRetries: maxRetries, completion: completion)
                }
            } else {
                completion(.failure(error))
            }
            return
        }
        
        if let data = data {
            completion(.success(data))
        }
    }.resume()
}
```

**With Combine:**
```swift
func fetchWithRetry(url: URL) -> AnyPublisher<Data, Error> {
    return URLSession.shared.dataTaskPublisher(for: url)
        .retry(3)
        .map(\.data)
        .eraseToAnyPublisher()
}
```

**Q5: How do you implement request throttling and debouncing?**

**Answer:**

**Throttling** - Limit frequency of requests:
```swift
class ThrottledSearch {
    private var lastSearchTime: Date?
    private let throttleInterval: TimeInterval = 1.0
    
    func search(query: String) {
        let now = Date()
        
        if let lastTime = lastSearchTime,
           now.timeIntervalSince(lastTime) < throttleInterval {
            return  // Skip this request
        }
        
        lastSearchTime = now
        performSearch(query: query)
    }
    
    func performSearch(query: String) {
        // Execute search
    }
}
```

**Debouncing** - Wait until user stops typing:
```swift
class DebouncedSearch {
    private var searchWorkItem: DispatchWorkItem?
    
    func search(query: String) {
        // Cancel previous search
        searchWorkItem?.cancel()
        
        let workItem = DispatchWorkItem {
            self.performSearch(query: query)
        }
        
        searchWorkItem = workItem
        
        // Execute after delay
        DispatchQueue.main.asyncAfter(deadline: .now() + 0.5, execute: workItem)
    }
    
    func performSearch(query: String) {
        print("Searching: \(query)")
    }
}
```

**With Combine:**
```swift
class SearchViewModel: ObservableObject {
    @Published var searchText = ""
    @Published var results: [String] = []
    private var cancellables = Set<AnyCancellable>()
    
    init() {
        $searchText
            .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
            .removeDuplicates()
            .filter { $0.count > 2 }
            .sink { [weak self] query in
                self?.performSearch(query: query)
            }
            .store(in: &cancellables)
    }
    
    func performSearch(query: String) {
        // Search API
    }
}
```

**Q6: What's the difference between Codable, Encodable, and Decodable?**

**Answer:**

**Codable:**
- Typealias for `Encodable & Decodable`
- Can both encode to and decode from external representations
- Use when you need both directions

```swift
typealias Codable = Encodable & Decodable

struct User: Codable {  // Can encode and decode
    let id: Int
    let name: String
}
```

**Encodable:**
- Only convert from Swift to external format (JSON, Plist)
- Use when only sending data

```swift
struct CreateUserRequest: Encodable {
    let name: String
    let email: String
}

let request = CreateUserRequest(name: "John", email: "john@example.com")
let data = try JSONEncoder().encode(request)
```

**Decodable:**
- Only convert from external format to Swift
- Use when only receiving data

```swift
struct ServerResponse: Decodable {
    let message: String
    let code: Int
}

let response = try JSONDecoder().decode(ServerResponse.self, from: data)
```

**Practical Example:**
```swift
// Request only needs encoding
struct LoginRequest: Encodable {
    let username: String
    let password: String
}

// Response only needs decoding
struct LoginResponse: Decodable {
    let token: String
    let expiresAt: Date
}

// User model needs both
struct User: Codable {
    let id: Int
    let name: String
}
```

**Q7: How do you handle authentication tokens in network requests?**

**Answer:**

**Token Storage:**
```swift
class AuthManager {
    static let shared = AuthManager()
    private let keychain = KeychainManager.shared
    
    func saveToken(_ token: String) {
        keychain.save(token, key: "authToken")
    }
    
    func getToken() -> String? {
        return keychain.load(key: "authToken")
    }
    
    func clearToken() {
        keychain.delete(key: "authToken")
    }
}
```

**Automatic Token Injection:**
```swift
class AuthenticatedNetworkService {
    private let session: URLSession
    
    init() {
        let configuration = URLSessionConfiguration.default
        session = URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
    }
    
    func request(url: URL, completion: @escaping (Result<Data, Error>) -> Void) {
        var request = URLRequest(url: url)
        
        // Add auth token
        if let token = AuthManager.shared.getToken() {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        session.dataTask(with: request) { data, response, error in
            // Handle 401 Unauthorized
            if let httpResponse = response as? HTTPURLResponse,
               httpResponse.statusCode == 401 {
                // Token expired, refresh or logout
                self.handleUnauthorized()
                return
            }
            
            // Handle response
        }.resume()
    }
    
    func handleUnauthorized() {
        AuthManager.shared.clearToken()
        // Navigate to login
    }
}
```

**Token Refresh:**
```swift
class TokenManager {
    func refreshToken(completion: @escaping (Result<String, Error>) -> Void) {
        guard let refreshToken = loadRefreshToken() else {
            completion(.failure(TokenError.noRefreshToken))
            return
        }
        
        // Call refresh endpoint
        let url = URL(string: "https://api.example.com/refresh")!
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("Bearer \(refreshToken)", forHTTPHeaderField: "Authorization")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            // Parse new token
            // Save new token
            // completion(.success(newToken))
        }.resume()
    }
    
    func loadRefreshToken() -> String? { return nil }
}
```

---

[← Previous: Data Persistence](data-persistence.md) | [Next: Multithreading & Concurrency →](multithreading-concurrency.md)

[Back to Main](../README.md)

