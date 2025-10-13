# Data Persistence & Storage

[← Back to Main](../README.md) | [Previous: SwiftUI](swiftui.md) | [Next: Networking →](networking.md)

## Table of Contents
- [UserDefaults](#userdefaults)
- [Keychain](#keychain)
- [Core Data](#core-data)
- [SQLite in iOS](#sqlite-in-ios)
- [File System](#file-system-in-ios)
- [Realm Database](#realm-database)

---

## UserDefaults

UserDefaults is a simple key-value storage for small amounts of data like user preferences.

### Basic Usage

```swift
// Save data
UserDefaults.standard.set("John Doe", forKey: "username")
UserDefaults.standard.set(25, forKey: "age")
UserDefaults.standard.set(true, forKey: "isLoggedIn")
UserDefaults.standard.set(Date(), forKey: "lastLogin")

// Retrieve data
let username = UserDefaults.standard.string(forKey: "username") // Optional<String>
let age = UserDefaults.standard.integer(forKey: "age") // Int (0 if not found)
let isLoggedIn = UserDefaults.standard.bool(forKey: "isLoggedIn") // Bool (false if not found)
let lastLogin = UserDefaults.standard.object(forKey: "lastLogin") as? Date

// Remove data
UserDefaults.standard.removeObject(forKey: "username")

// Check if key exists
if UserDefaults.standard.object(forKey: "username") != nil {
    print("Username exists")
}
```

### Storing Complex Types

```swift
struct User: Codable {
    let id: Int
    let name: String
    let email: String
}

class UserDefaultsManager {
    static let shared = UserDefaultsManager()
    private let defaults = UserDefaults.standard
    
    func saveUser(_ user: User) {
        if let encoded = try? JSONEncoder().encode(user) {
            defaults.set(encoded, forKey: "currentUser")
        }
    }
    
    func loadUser() -> User? {
        guard let data = defaults.data(forKey: "currentUser"),
              let user = try? JSONDecoder().decode(User.self, from: data) else {
            return nil
        }
        return user
    }
    
    func deleteUser() {
        defaults.removeObject(forKey: "currentUser")
    }
}

// Usage
let user = User(id: 1, name: "John", email: "john@example.com")
UserDefaultsManager.shared.saveUser(user)

if let savedUser = UserDefaultsManager.shared.loadUser() {
    print("User: \(savedUser.name)")
}
```

### Property Wrapper

```swift
@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T
    
    var wrappedValue: T {
        get {
            UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}

class Settings {
    @UserDefault(key: "theme", defaultValue: "light")
    static var theme: String
    
    @UserDefault(key: "fontSize", defaultValue: 14)
    static var fontSize: Int
    
    @UserDefault(key: "notificationsEnabled", defaultValue: true)
    static var notificationsEnabled: Bool
}

// Usage
Settings.theme = "dark"
print(Settings.theme) // "dark"
```

### Best Practices

```swift
class UserDefaultsKeys {
    static let isFirstLaunch = "isFirstLaunch"
    static let userTheme = "userTheme"
    static let lastSyncDate = "lastSyncDate"
}

// Use structured keys
extension UserDefaults {
    var isFirstLaunch: Bool {
        get { bool(forKey: UserDefaultsKeys.isFirstLaunch) }
        set { set(newValue, forKey: UserDefaultsKeys.isFirstLaunch) }
    }
    
    var userTheme: String {
        get { string(forKey: UserDefaultsKeys.userTheme) ?? "light" }
        set { set(newValue, forKey: UserDefaultsKeys.userTheme) }
    }
}

// Usage
if UserDefaults.standard.isFirstLaunch {
    // Show onboarding
    UserDefaults.standard.isFirstLaunch = false
}
```

---

## Keychain

Keychain is a secure storage for sensitive data like passwords, tokens, and certificates.

### Basic Keychain Wrapper

```swift
import Security
import Foundation

class KeychainManager {
    static let shared = KeychainManager()
    
    func save(_ data: Data, service: String, account: String) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecValueData as String: data
        ]
        
        SecItemDelete(query as CFDictionary) // Delete old item
        
        let status = SecItemAdd(query as CFDictionary, nil)
        return status == errSecSuccess
    }
    
    func load(service: String, account: String) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        return status == errSecSuccess ? result as? Data : nil
    }
    
    func delete(service: String, account: String) -> Bool {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: account
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        return status == errSecSuccess
    }
}

// Extension for String storage
extension KeychainManager {
    func saveString(_ string: String, service: String, account: String) -> Bool {
        guard let data = string.data(using: .utf8) else { return false }
        return save(data, service: service, account: account)
    }
    
    func loadString(service: String, account: String) -> String? {
        guard let data = load(service: service, account: account) else { return nil }
        return String(data: data, encoding: .utf8)
    }
}

// Usage
let keychain = KeychainManager.shared

// Save token
keychain.saveString("my_secret_token", service: "MyApp", account: "authToken")

// Load token
if let token = keychain.loadString(service: "MyApp", account: "authToken") {
    print("Token: \(token)")
}

// Delete token
keychain.delete(service: "MyApp", account: "authToken")
```

### Generic Keychain Service

```swift
protocol KeychainServiceProtocol {
    func save<T: Codable>(_ item: T, key: String) throws
    func load<T: Codable>(key: String) throws -> T
    func delete(key: String) throws
}

enum KeychainError: Error {
    case encodingFailed
    case decodingFailed
    case saveFailed(OSStatus)
    case loadFailed(OSStatus)
    case deleteFailed(OSStatus)
    case itemNotFound
}

class KeychainService: KeychainServiceProtocol {
    private let service: String
    
    init(service: String = Bundle.main.bundleIdentifier ?? "com.app.keychain") {
        self.service = service
    }
    
    func save<T: Codable>(_ item: T, key: String) throws {
        let encoder = JSONEncoder()
        guard let data = try? encoder.encode(item) else {
            throw KeychainError.encodingFailed
        }
        
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecValueData as String: data
        ]
        
        SecItemDelete(query as CFDictionary)
        
        let status = SecItemAdd(query as CFDictionary, nil)
        guard status == errSecSuccess else {
            throw KeychainError.saveFailed(status)
        }
    }
    
    func load<T: Codable>(key: String) throws -> T {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true,
            kSecMatchLimit as String: kSecMatchLimitOne
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        guard status == errSecSuccess else {
            if status == errSecItemNotFound {
                throw KeychainError.itemNotFound
            }
            throw KeychainError.loadFailed(status)
        }
        
        guard let data = result as? Data else {
            throw KeychainError.loadFailed(status)
        }
        
        let decoder = JSONDecoder()
        guard let item = try? decoder.decode(T.self, from: data) else {
            throw KeychainError.decodingFailed
        }
        
        return item
    }
    
    func delete(key: String) throws {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrService as String: service,
            kSecAttrAccount as String: key
        ]
        
        let status = SecItemDelete(query as CFDictionary)
        guard status == errSecSuccess || status == errSecItemNotFound else {
            throw KeychainError.deleteFailed(status)
        }
    }
}

// Usage with Codable
struct Credentials: Codable {
    let username: String
    let password: String
}

let keychain = KeychainService()

do {
    let creds = Credentials(username: "john", password: "secret123")
    try keychain.save(creds, key: "userCredentials")
    
    let loaded: Credentials = try keychain.load(key: "userCredentials")
    print("Username: \(loaded.username)")
    
    try keychain.delete(key: "userCredentials")
} catch {
    print("Keychain error: \(error)")
}
```

---

## Core Data

Core Data is Apple's object graph and persistence framework.

### Setting Up Core Data

```swift
import CoreData

class CoreDataManager {
    static let shared = CoreDataManager()
    
    lazy var persistentContainer: NSPersistentContainer = {
        let container = NSPersistentContainer(name: "DataModel")
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Unable to load persistent stores: \(error)")
            }
        }
        return container
    }()
    
    var context: NSManagedObjectContext {
        return persistentContainer.viewContext
    }
    
    func saveContext() {
        let context = persistentContainer.viewContext
        if context.hasChanges {
            do {
                try context.save()
            } catch {
                let nserror = error as NSError
                fatalError("Unresolved error \(nserror), \(nserror.userInfo)")
            }
        }
    }
}
```

### Creating Entities

```swift
// Define entity in .xcdatamodeld file
// Entity: Person
// Attributes: id (UUID), name (String), age (Int16), email (String)

extension Person {
    @nonobjc public class func fetchRequest() -> NSFetchRequest<Person> {
        return NSFetchRequest<Person>(entityName: "Person")
    }
    
    @NSManaged public var id: UUID?
    @NSManaged public var name: String?
    @NSManaged public var age: Int16
    @NSManaged public var email: String?
}
```

### CRUD Operations

```swift
class PersonRepository {
    let context = CoreDataManager.shared.context
    
    // Create
    func createPerson(name: String, age: Int, email: String) {
        let person = Person(context: context)
        person.id = UUID()
        person.name = name
        person.age = Int16(age)
        person.email = email
        
        CoreDataManager.shared.saveContext()
    }
    
    // Read
    func fetchAllPersons() -> [Person] {
        let fetchRequest: NSFetchRequest<Person> = Person.fetchRequest()
        
        do {
            return try context.fetch(fetchRequest)
        } catch {
            print("Failed to fetch persons: \(error)")
            return []
        }
    }
    
    func fetchPersons(with predicate: NSPredicate) -> [Person] {
        let fetchRequest: NSFetchRequest<Person> = Person.fetchRequest()
        fetchRequest.predicate = predicate
        
        do {
            return try context.fetch(fetchRequest)
        } catch {
            print("Failed to fetch persons: \(error)")
            return []
        }
    }
    
    // Update
    func updatePerson(_ person: Person, name: String?, age: Int?) {
        if let name = name {
            person.name = name
        }
        if let age = age {
            person.age = Int16(age)
        }
        
        CoreDataManager.shared.saveContext()
    }
    
    // Delete
    func deletePerson(_ person: Person) {
        context.delete(person)
        CoreDataManager.shared.saveContext()
    }
    
    func deleteAllPersons() {
        let fetchRequest: NSFetchRequest<NSFetchRequestResult> = Person.fetchRequest()
        let deleteRequest = NSBatchDeleteRequest(fetchRequest: fetchRequest)
        
        do {
            try context.execute(deleteRequest)
            CoreDataManager.shared.saveContext()
        } catch {
            print("Failed to delete all persons: \(error)")
        }
    }
}

// Usage
let repository = PersonRepository()

// Create
repository.createPerson(name: "John Doe", age: 25, email: "john@example.com")

// Read
let allPersons = repository.fetchAllPersons()
for person in allPersons {
    print("\(person.name ?? "Unknown") - \(person.age)")
}

// Fetch with predicate
let predicate = NSPredicate(format: "age > %d", 20)
let adults = repository.fetchPersons(with: predicate)

// Update
if let person = allPersons.first {
    repository.updatePerson(person, name: "Jane Doe", age: 30)
}

// Delete
if let person = allPersons.first {
    repository.deletePerson(person)
}
```

### Relationships

```swift
// Define in .xcdatamodeld:
// Entity: Department
// Attributes: id, name
// Relationship: employees (To-Many to Employee)

// Entity: Employee
// Attributes: id, name
// Relationship: department (To-One to Department)

class DepartmentRepository {
    let context = CoreDataManager.shared.context
    
    func createDepartment(name: String, employees: [String]) {
        let department = Department(context: context)
        department.id = UUID()
        department.name = name
        
        for employeeName in employees {
            let employee = Employee(context: context)
            employee.id = UUID()
            employee.name = employeeName
            employee.department = department
        }
        
        CoreDataManager.shared.saveContext()
    }
    
    func fetchDepartmentWithEmployees(name: String) -> Department? {
        let fetchRequest: NSFetchRequest<Department> = Department.fetchRequest()
        fetchRequest.predicate = NSPredicate(format: "name == %@", name)
        fetchRequest.relationshipKeyPathsForPrefetching = ["employees"]
        
        do {
            return try context.fetch(fetchRequest).first
        } catch {
            print("Failed to fetch department: \(error)")
            return nil
        }
    }
}
```

---

## SQLite in iOS

Direct SQLite usage without Core Data.

### SQLite Wrapper

```swift
import SQLite3

class SQLiteDatabase {
    private var db: OpaquePointer?
    
    init(path: String) {
        if sqlite3_open(path, &db) != SQLITE_OK {
            print("Error opening database")
        }
    }
    
    deinit {
        sqlite3_close(db)
    }
    
    func createTable() {
        let createTableQuery = """
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT NOT NULL,
            age INTEGER
        );
        """
        
        executeQuery(createTableQuery)
    }
    
    func insert(name: String, email: String, age: Int) {
        let insertQuery = "INSERT INTO users (name, email, age) VALUES (?, ?, ?);"
        
        var statement: OpaquePointer?
        
        if sqlite3_prepare_v2(db, insertQuery, -1, &statement, nil) == SQLITE_OK {
            sqlite3_bind_text(statement, 1, (name as NSString).utf8String, -1, nil)
            sqlite3_bind_text(statement, 2, (email as NSString).utf8String, -1, nil)
            sqlite3_bind_int(statement, 3, Int32(age))
            
            if sqlite3_step(statement) == SQLITE_DONE {
                print("Successfully inserted row")
            } else {
                print("Could not insert row")
            }
        }
        
        sqlite3_finalize(statement)
    }
    
    func fetchAll() -> [(id: Int, name: String, email: String, age: Int)] {
        let query = "SELECT * FROM users;"
        var results: [(Int, String, String, Int)] = []
        var statement: OpaquePointer?
        
        if sqlite3_prepare_v2(db, query, -1, &statement, nil) == SQLITE_OK {
            while sqlite3_step(statement) == SQLITE_ROW {
                let id = Int(sqlite3_column_int(statement, 0))
                let name = String(cString: sqlite3_column_text(statement, 1))
                let email = String(cString: sqlite3_column_text(statement, 2))
                let age = Int(sqlite3_column_int(statement, 3))
                
                results.append((id, name, email, age))
            }
        }
        
        sqlite3_finalize(statement)
        return results
    }
    
    private func executeQuery(_ query: String) {
        var statement: OpaquePointer?
        
        if sqlite3_prepare_v2(db, query, -1, &statement, nil) == SQLITE_OK {
            if sqlite3_step(statement) == SQLITE_DONE {
                print("Query executed successfully")
            } else {
                print("Query execution failed")
            }
        }
        
        sqlite3_finalize(statement)
    }
}

// Usage
let dbPath = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)[0] + "/mydb.sqlite"
let db = SQLiteDatabase(path: dbPath)

db.createTable()
db.insert(name: "John", email: "john@example.com", age: 25)

let users = db.fetchAll()
for user in users {
    print("\(user.name) - \(user.email)")
}
```

---

## File System in iOS

### File Manager

```swift
class FileManagerHelper {
    static let shared = FileManagerHelper()
    private let fileManager = FileManager.default
    
    var documentsDirectory: URL {
        return fileManager.urls(for: .documentDirectory, in: .userDomainMask)[0]
    }
    
    var cachesDirectory: URL {
        return fileManager.urls(for: .cachesDirectory, in: .userDomainMask)[0]
    }
    
    var tempDirectory: URL {
        return fileManager.temporaryDirectory
    }
    
    // Save data
    func save(_ data: Data, to fileName: String) throws {
        let fileURL = documentsDirectory.appendingPathComponent(fileName)
        try data.write(to: fileURL)
    }
    
    // Load data
    func load(from fileName: String) throws -> Data {
        let fileURL = documentsDirectory.appendingPathComponent(fileName)
        return try Data(contentsOf: fileURL)
    }
    
    // Delete file
    func delete(fileName: String) throws {
        let fileURL = documentsDirectory.appendingPathComponent(fileName)
        try fileManager.removeItem(at: fileURL)
    }
    
    // Check if file exists
    func fileExists(fileName: String) -> Bool {
        let fileURL = documentsDirectory.appendingPathComponent(fileName)
        return fileManager.fileExists(atPath: fileURL.path)
    }
    
    // List files
    func listFiles(in directory: URL) -> [String] {
        do {
            return try fileManager.contentsOfDirectory(atPath: directory.path)
        } catch {
            print("Error listing files: \(error)")
            return []
        }
    }
}

// Usage
let fileHelper = FileManagerHelper.shared

// Save
let text = "Hello, World!"
if let data = text.data(using: .utf8) {
    try? fileHelper.save(data, to: "greeting.txt")
}

// Load
if let data = try? fileHelper.load(from: "greeting.txt"),
   let content = String(data: data, encoding: .utf8) {
    print(content)
}

// Delete
try? fileHelper.delete(fileName: "greeting.txt")
```

### Codable with Files

```swift
class CodableFileStorage {
    static let shared = CodableFileStorage()
    private let encoder = JSONEncoder()
    private let decoder = JSONDecoder()
    
    func save<T: Encodable>(_ object: T, to fileName: String) throws {
        let data = try encoder.encode(object)
        try FileManagerHelper.shared.save(data, to: fileName)
    }
    
    func load<T: Decodable>(from fileName: String) throws -> T {
        let data = try FileManagerHelper.shared.load(from: fileName)
        return try decoder.decode(T.self, from: data)
    }
}

// Usage
struct Settings: Codable {
    var theme: String
    var fontSize: Int
    var notifications: Bool
}

let settings = Settings(theme: "dark", fontSize: 16, notifications: true)

// Save
try? CodableFileStorage.shared.save(settings, to: "settings.json")

// Load
if let loadedSettings: Settings = try? CodableFileStorage.shared.load(from: "settings.json") {
    print("Theme: \(loadedSettings.theme)")
}
```

---

## Realm Database

Realm is a modern alternative to Core Data and SQLite.

### Setup

```swift
import RealmSwift

// Model
class Person: Object {
    @Persisted(primaryKey: true) var id: ObjectId
    @Persisted var name: String
    @Persisted var age: Int
    @Persisted var email: String
    
    convenience init(name: String, age: Int, email: String) {
        self.init()
        self.name = name
        self.age = age
        self.email = email
    }
}
```

### CRUD Operations

```swift
class RealmManager {
    static let shared = RealmManager()
    private var realm: Realm {
        return try! Realm()
    }
    
    // Create
    func create(_ person: Person) {
        try? realm.write {
            realm.add(person)
        }
    }
    
    // Read
    func fetchAll() -> Results<Person> {
        return realm.objects(Person.self)
    }
    
    func fetch(id: ObjectId) -> Person? {
        return realm.object(ofType: Person.self, forPrimaryKey: id)
    }
    
    // Update
    func update(_ person: Person, name: String? = nil, age: Int? = nil) {
        try? realm.write {
            if let name = name {
                person.name = name
            }
            if let age = age {
                person.age = age
            }
        }
    }
    
    // Delete
    func delete(_ person: Person) {
        try? realm.write {
            realm.delete(person)
        }
    }
    
    func deleteAll() {
        try? realm.write {
            realm.deleteAll()
        }
    }
}

// Usage
let realmManager = RealmManager.shared

// Create
let person = Person(name: "John", age: 25, email: "john@example.com")
realmManager.create(person)

// Read
let allPersons = realmManager.fetchAll()
for person in allPersons {
    print("\(person.name) - \(person.age)")
}

// Update
if let first = allPersons.first {
    realmManager.update(first, name: "Jane")
}

// Delete
if let first = allPersons.first {
    realmManager.delete(first)
}
```

### Interview Questions & Answers

**Q1: When should you use UserDefaults vs Keychain?**

**Answer:** 

**UserDefaults** - Non-sensitive data:
- User preferences (theme, language)
- App settings
- Simple flags (isFirstLaunch)
- Small amounts of data
- Not encrypted by default

**Keychain** - Sensitive data:
- Passwords
- Authentication tokens
- API keys
- Certificates
- Encrypted by default
- Survives app reinstall

**Example:**

```swift
// UserDefaults - Settings
UserDefaults.standard.set("dark", forKey: "theme")

// Keychain - Token
KeychainManager.shared.save("auth_token_xyz", key: "authToken")
```

**Q2: Core Data vs Realm - which to choose?**

**Answer:**

**Core Data:**
- ✅ Apple's official framework
- ✅ Deep iCloud integration
- ✅ Better for complex data models
- ✅ Mature and stable
- ❌ Steeper learning curve
- ❌ More boilerplate code
- ❌ Harder to debug

**Realm:**
- ✅ Simpler API
- ✅ Faster for most operations
- ✅ Cross-platform (iOS, Android)
- ✅ Live objects (auto-updates)
- ✅ Better documentation
- ❌ Third-party dependency
- ❌ Larger binary size

**Choose Core Data if:**
- Already invested in Apple ecosystem
- Need CloudKit sync
- Complex relationships and queries
- Long-term Apple support critical

**Choose Realm if:**
- New project
- Cross-platform requirements
- Want simpler API
- Speed is priority

**Q3: How do you migrate Core Data models?**

**Answer:**

**Lightweight Migration** (Automatic):

```swift
let container = NSPersistentContainer(name: "DataModel")

let description = container.persistentStoreDescriptions.first
description?.shouldMigrateStoreAutomatically = true
description?.shouldInferMappingModelAutomatically = true

container.loadPersistentStores { description, error in
    if let error = error {
        fatalError("Migration failed: \(error)")
    }
}
```

**When Lightweight Works:**
- Add new entity
- Add new attribute
- Delete attribute
- Make optional attribute required (with default)
- Rename entity or attribute

**Manual Migration:**

For complex changes, create mapping model in Xcode:
1. Editor → Add Model Version
2. Create NSEntityMigrationPolicy subclass
3. Implement custom migration logic

**Best Practices:**
- Test migration on copies of production data
- Support multiple versions back
- Provide fallback options
- Log migration events

**Q4: How do you handle large amounts of data in Core Data?**

**Answer:**

**1. Batch Fetching:**
```swift
let fetchRequest: NSFetchRequest<Person> = Person.fetchRequest()
fetchRequest.fetchBatchSize = 20

let results = try context.fetch(fetchRequest)
```

**2. Fault Objects:**
```swift
// Core Data loads objects as "faults" (lightweight placeholders)
// Data loaded only when accessed
let person = results.first  // Fault
let name = person.name  // Now fully loaded
```

**3. NSFetchedResultsController:**
```swift
let fetchRequest: NSFetchRequest<Person> = Person.fetchRequest()
fetchRequest.sortDescriptors = [NSSortDescriptor(key: "name", ascending: true)]
fetchRequest.fetchBatchSize = 20

let controller = NSFetchedResultsController(
    fetchRequest: fetchRequest,
    managedObjectContext: context,
    sectionNameKeyPath: nil,
    cacheName: "PersonCache"
)

try controller.performFetch()
```

**4. Predicates for Filtering:**
```swift
let predicate = NSPredicate(format: "age > %d AND city == %@", 18, "New York")
fetchRequest.predicate = predicate
```

**5. Background Context:**
```swift
let backgroundContext = container.newBackgroundContext()

backgroundContext.perform {
    // Perform heavy operations on background thread
    let fetchRequest: NSFetchRequest<Person> = Person.fetchRequest()
    let persons = try? backgroundContext.fetch(fetchRequest)
    
    // Process data
}
```

**Q5: What's the difference between UserDefaults.standard and a custom UserDefaults suite?**

**Answer:**

**UserDefaults.standard:**
- Default shared instance
- Accessible across app
- Stored in app's preference domain

**Custom Suite:**
- Separate storage domain
- Useful for App Groups (sharing between app and extensions)
- Better organization

**Example:**

```swift
// Standard
UserDefaults.standard.set("value", forKey: "key")

// Custom Suite (App Groups)
let shared = UserDefaults(suiteName: "group.com.company.app")
shared?.set("value", forKey: "key")

// Widget can access this data
```

**Use Cases for Custom Suite:**
- Share data between app and Today Extension
- Share between app and Watch app
- Share between app and Widget
- Separate test data from production

**Q6: How do you handle Core Data concurrency?**

**Answer:**

Core Data contexts are not thread-safe. Always use proper concurrency patterns:

**1. Main Queue Context:**
```swift
let mainContext = persistentContainer.viewContext
// Use only on main thread
```

**2. Private Queue Context:**
```swift
let backgroundContext = persistentContainer.newBackgroundContext()

backgroundContext.perform {
    // Safe to use here
    let person = Person(context: backgroundContext)
    person.name = "John"
    try? backgroundContext.save()
}
```

**3. Passing Data Between Contexts:**
```swift
// BAD - Don't pass managed objects between contexts
let person = fetchFromMainContext()
backgroundContext.perform {
    person.name = "New"  // CRASH!
}

// GOOD - Pass object IDs
let objectID = person.objectID
backgroundContext.perform {
    if let bgPerson = try? backgroundContext.existingObject(with: objectID) as? Person {
        bgPerson.name = "New"
        try? backgroundContext.save()
    }
}
```

**4. Merge Policies:**
```swift
context.mergePolicy = NSMergeByPropertyObjectTrumpMergePolicy
```

**Best Practices:**
- Never share contexts across threads
- Use perform/performAndWait
- Pass ObjectIDs, not objects
- Save on background thread for heavy operations

**Q7: How do you implement data caching strategy?**

**Answer:**

**Multi-Level Caching:**

```swift
class DataManager {
    // Level 1: Memory cache (fastest)
    private var memoryCache = NSCache<NSString, AnyObject>()
    
    // Level 2: Disk cache
    private let diskCache = FileManager.default
    
    // Level 3: Database (Core Data/Realm)
    private let database = CoreDataManager.shared
    
    func getData(key: String, completion: @escaping (Data?) -> Void) {
        // 1. Check memory
        if let cached = memoryCache.object(forKey: key as NSString) as? Data {
            completion(cached)
            return
        }
        
        // 2. Check disk
        if let diskData = loadFromDisk(key: key) {
            memoryCache.setObject(diskData as AnyObject, forKey: key as NSString)
            completion(diskData)
            return
        }
        
        // 3. Check database
        if let dbData = loadFromDatabase(key: key) {
            saveToDisk(data: dbData, key: key)
            memoryCache.setObject(dbData as AnyObject, forKey: key as NSString)
            completion(dbData)
            return
        }
        
        // 4. Fetch from network
        fetchFromNetwork(key: key) { data in
            guard let data = data else {
                completion(nil)
                return
            }
            
            self.saveToDatabase(data: data, key: key)
            self.saveToDisk(data: data, key: key)
            self.memoryCache.setObject(data as AnyObject, forKey: key as NSString)
            completion(data)
        }
    }
    
    func loadFromDisk(key: String) -> Data? { return nil }
    func saveToDisk(data: Data, key: String) { }
    func loadFromDatabase(key: String) -> Data? { return nil }
    func saveToDatabase(data: Data, key: String) { }
    func fetchFromNetwork(key: String, completion: @escaping (Data?) -> Void) { }
}
```

**Cache Expiration:**

```swift
struct CacheEntry<T> {
    let value: T
    let expirationDate: Date
    
    var isExpired: Bool {
        return Date() > expirationDate
    }
}

class TimedCache<T> {
    private var cache: [String: CacheEntry<T>] = [:]
    private let expirationTime: TimeInterval
    
    init(expirationTime: TimeInterval = 300) {
        self.expirationTime = expirationTime
    }
    
    func set(_ value: T, forKey key: String) {
        let entry = CacheEntry(
            value: value,
            expirationDate: Date().addingTimeInterval(expirationTime)
        )
        cache[key] = entry
    }
    
    func get(forKey key: String) -> T? {
        guard let entry = cache[key], !entry.isExpired else {
            cache.removeValue(forKey: key)
            return nil
        }
        return entry.value
    }
}
```

---

[← Previous: SwiftUI](swiftui.md) | [Next: Networking →](networking.md)

[Back to Main](../README.md)

