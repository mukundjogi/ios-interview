# Advanced iOS Topics

[← Back to Main](../README.md) | [Previous: Testing](testing.md) | [Next: Security →](security.md)

## Table of Contents
- [Combine Framework Basics](#combine-framework-basics)
- [App Extensions](#app-extensions)
- [Push Notifications](#push-notifications--apns)
- [Deep Linking & Universal Links](#deep-linking--universal-links)

---

## Combine Framework Basics

Reactive programming framework from Apple.

### Publishers & Subscribers

```swift
import Combine

// Simple publisher
let publisher = Just("Hello, Combine!")

let subscription = publisher.sink { value in
    print(value)
}

// PassthroughSubject
let subject = PassthroughSubject<String, Never>()

subject.sink { value in
    print("Received: \(value)")
}.store(in: &cancellables)

subject.send("Hello")
subject.send("World")

// CurrentValueSubject
let currentValue = CurrentValueSubject<Int, Never>(0)
print(currentValue.value) // 0

currentValue.send(5)
print(currentValue.value) // 5
```

### Operators

```swift
var cancellables = Set<AnyCancellable>()

// Map
[1, 2, 3].publisher
    .map { $0 * 2 }
    .sink { print($0) }
    .store(in: &cancellables)

// Filter
[1, 2, 3, 4, 5].publisher
    .filter { $0 % 2 == 0 }
    .sink { print($0) }
    .store(in: &cancellables)

// Debounce
searchTextField.textPublisher
    .debounce(for: .milliseconds(500), scheduler: DispatchQueue.main)
    .sink { text in
        print("Search: \(text)")
    }
    .store(in: &cancellables)
```

---

## App Extensions

Extend functionality beyond the main app.

### Widget Extension

```swift
import WidgetKit
import SwiftUI

struct SimpleWidget: Widget {
    let kind: String = "SimpleWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            SimpleWidgetView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("This is a simple widget")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date())
    }
    
    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> Void) {
        let entry = SimpleEntry(date: Date())
        completion(entry)
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<SimpleEntry>) -> Void) {
        let entries = [SimpleEntry(date: Date())]
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
}

struct SimpleWidgetView: View {
    let entry: SimpleEntry
    
    var body: some View {
        Text(entry.date, style: .time)
    }
}
```

### Today Extension

Create custom Today view widgets.

### Share Extension

Enable sharing content from other apps.

---

## Push Notifications & APNs

### Register for Notifications

```swift
import UserNotifications

class AppDelegate: UIResponder, UIApplicationDelegate {
    func application(_ application: UIApplication,
                     didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
        UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in
            if granted {
                DispatchQueue.main.async {
                    application.registerForRemoteNotifications()
                }
            }
        }
        
        return true
    }
    
    func application(_ application: UIApplication,
                     didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
        print("Device Token: \(token)")
    }
}
```

### Handle Notifications

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound, .badge])
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        let userInfo = response.notification.request.content.userInfo
        print("Notification tapped: \(userInfo)")
        completionHandler()
    }
}
```

---

## Deep Linking & Universal Links

### Custom URL Scheme

```swift
// Info.plist
// Add URL Types with identifier and scheme

func application(_ app: UIApplication,
                 open url: URL,
                 options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    
    if url.scheme == "myapp" {
        // Handle myapp://user/123
        handleDeepLink(url)
        return true
    }
    
    return false
}

func handleDeepLink(_ url: URL) {
    let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
    // Parse and navigate
}
```

### Universal Links

```swift
// Associated Domains in capabilities
// Add apple-app-site-association file to website

func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else {
        return false
    }
    
    handleUniversalLink(url)
    return true
}
```

---

[← Previous: Testing](testing.md) | [Next: Security →](security.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: Explain the key concepts of Combine framework

**Answer:**

**Core Concepts:**

**1. Publishers:**
- Emit values over time
- Can complete or fail
- Types: Just, Future, PassthroughSubject, CurrentValueSubject

**2. Subscribers:**
- Receive values from publishers
- Types: sink, assign

**3. Operators:**
- Transform, filter, combine publishers
- Examples: map, filter, flatMap, debounce

**Example:**

```swift
import Combine

var cancellables = Set<AnyCancellable>()

// Publisher
let numbers = [1, 2, 3, 4, 5].publisher

// Operators + Subscriber
numbers
    .filter { $0 % 2 == 0 }
    .map { $0 * 2 }
    .sink { value in
        print(value)  // 4, 8
    }
    .store(in: &cancellables)
```

**Benefits:**
- Declarative syntax
- Composable
- Memory safe
- Integrates with SwiftUI

**Use Cases:**
- Networking
- Form validation
- Real-time updates
- Search with debouncing

### Q2: How do you implement push notifications in iOS?

**Answer:**

**Step 1: Request Permission**

```swift
import UserNotifications

func requestNotificationPermission() {
    UNUserNotificationCenter.current().requestAuthorization(options: [.alert, .badge, .sound]) { granted, error in
        if granted {
            DispatchQueue.main.async {
                UIApplication.shared.registerForRemoteNotifications()
            }
        }
    }
}
```

**Step 2: Register Device Token**

```swift
func application(_ application: UIApplication,
                 didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
    let token = deviceToken.map { String(format: "%02.2hhx", $0) }.joined()
    print("Device Token: \(token)")
    // Send token to your server
}
```

**Step 3: Handle Notifications**

```swift
extension AppDelegate: UNUserNotificationCenterDelegate {
    // Foreground
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                willPresent notification: UNNotification,
                                withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        completionHandler([.banner, .sound, .badge])
    }
    
    // Tapped
    func userNotificationCenter(_ center: UNUserNotificationCenter,
                                didReceive response: UNNotificationResponse,
                                withCompletionHandler completionHandler: @escaping () -> Void) {
        let userInfo = response.notification.request.content.userInfo
        handleNotificationTap(userInfo: userInfo)
        completionHandler()
    }
}
```

**Step 4: Send Notification (Server)**

```swift
// HTTP/2 POST to:
// https://api.push.apple.com/3/device/{device-token}

// Headers:
// authorization: bearer {token}  // JWT with P8 key
// apns-topic: {bundle-id}

// Payload:
{
    "aps": {
        "alert": {
            "title": "New Message",
            "body": "You have a new message"
        },
        "badge": 1,
        "sound": "default"
    },
    "customData": "value"
}
```

### Q3: What's the difference between local and remote notifications?

**Answer:**

**Local Notifications:**
- Scheduled by the app itself
- Don't require internet
- Triggered by time or location

```swift
func scheduleLocalNotification() {
    let content = UNMutableNotificationContent()
    content.title = "Reminder"
    content.body = "Time to check in!"
    content.sound = .default
    
    // Trigger after 60 seconds
    let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 60, repeats: false)
    
    let request = UNNotificationRequest(identifier: "reminder", content: content, trigger: trigger)
    
    UNUserNotificationCenter.current().add(request) { error in
        if let error = error {
            print("Error: \(error)")
        }
    }
}
```

**Remote Notifications:**
- Sent from server via APNs
- Require internet connection
- Server initiates

**Comparison:**

| Feature | Local | Remote |
|---------|-------|--------|
| Origin | App itself | Server |
| Internet | Not required | Required |
| Setup | Simple | Complex (APNs, certificates) |
| Use Case | Reminders, alarms | Messages, updates |

### Q4: How do you implement deep linking?

**Answer:**

**Custom URL Scheme:**

**1. Configure in Info.plist:**
```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>myapp</string>
        </array>
    </dict>
</array>
```

**2. Handle in AppDelegate:**
```swift
func application(_ app: UIApplication,
                 open url: URL,
                 options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    
    // URL: myapp://profile/123
    let components = URLComponents(url: url, resolvingAgainstBaseURL: false)
    
    if url.host == "profile",
       let userId = components?.path.components(separatedBy: "/").last {
        navigateToProfile(userId: userId)
        return true
    }
    
    return false
}
```

**Universal Links:**

**1. Configure Associated Domains:**
- Signing & Capabilities
- Add Associated Domains
- Add `applinks:yourdomin.com`

**2. apple-app-site-association file on server:**
```json
{
    "applinks": {
        "apps": [],
        "details": [{
            "appID": "TEAMID.com.company.app",
            "paths": ["/products/*", "/profile/*"]
        }]
    }
}
```

**3. Handle in AppDelegate:**
```swift
func application(_ application: UIApplication,
                 continue userActivity: NSUserActivity,
                 restorationHandler: @escaping ([UIUserActivityRestoring]?) -> Void) -> Bool {
    
    guard userActivity.activityType == NSUserActivityTypeBrowsingWeb,
          let url = userActivity.webpageURL else {
        return false
    }
    
    // Handle: https://yourapp.com/profile/123
    handleUniversalLink(url)
    return true
}
```



### Q5: What are app extensions and their types?

**Answer:**

**Common Extension Types:**

**1. Today Extension (Widget):**
- Show information in Today view
- Quick access to app features

**2. Share Extension:**
- Share content to your app from other apps

**3. Action Extension:**
- Perform actions on content in other apps

**4. Photo Editing Extension:**
- Edit photos in Photos app

**5. Custom Keyboard:**
- System-wide custom keyboard

**6. Notification Service/Content Extension:**
- Modify notifications before display

**Implementation Example (Widget):**

```swift
import WidgetKit
import SwiftUI

@main
struct MyWidget: Widget {
    let kind = "MyWidget"
    
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            WidgetView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("Shows useful information")
        .supportedFamilies([.systemSmall, .systemMedium, .systemLarge])
    }
}

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date())
    }
    
    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> Void) {
        completion(SimpleEntry(date: Date()))
    }
    
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> Void) {
        let entries = [SimpleEntry(date: Date())]
        let timeline = Timeline(entries: entries, policy: .atEnd)
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
}
```

**Limitations:**
- Limited memory
- No continuous execution
- Separate bundle
- Limited APIs

**Data Sharing:**
Use App Groups to share data between app and extension.

### Q6: How do you use Combine with URLSession?

**Answer:**

**Basic Request:**

```swift
import Combine

func fetchUsers() -> AnyPublisher<[User], Error> {
    let url = URL(string: "https://api.example.com/users")!
    
    return URLSession.shared.dataTaskPublisher(for: url)
        .map(\.data)
        .decode(type: [User].self, decoder: JSONDecoder())
        .receive(on: DispatchQueue.main)
        .eraseToAnyPublisher()
}

// Usage
var cancellables = Set<AnyCancellable>()

fetchUsers()
    .sink { completion in
        switch completion {
        case .finished:
            print("Success")
        case .failure(let error):
            print("Error: \(error)")
        }
    } receiveValue: { users in
        print("Users: \(users)")
    }
    .store(in: &cancellables)
```

**With Error Handling:**

```swift
fetchUsers()
    .retry(3)
    .catch { error -> Just<[User]> in
        print("Error: \(error)")
        return Just([])  // Return empty array on error
    }
    .sink { users in
        self.users = users
    }
    .store(in: &cancellables)
```

**Multiple Requests:**

```swift
let users = fetchUsers()
let posts = fetchPosts()

Publishers.Zip(users, posts)
    .sink { users, posts in
        print("Got both: \(users.count) users, \(posts.count) posts")
    }
    .store(in: &cancellables)
```

### Q7: How do you implement silent push notifications?

**Answer:**

Silent notifications wake app in background without alerting user.

**Payload:**
```json
{
    "aps": {
        "content-available": 1
    },
    "data": {
        "type": "sync",
        "userId": 123
    }
}
```

**Handle in AppDelegate:**

```swift
func application(_ application: UIApplication,
                 didReceiveRemoteNotification userInfo: [AnyHashable : Any],
                 fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
    
    // Process notification
    if let type = userInfo["type"] as? String, type == "sync" {
        syncData { success in
            if success {
                completionHandler(.newData)
            } else {
                completionHandler(.failed)
            }
        }
    } else {
        completionHandler(.noData)
    }
}

func syncData(completion: @escaping (Bool) -> Void) {
    // Sync data from server
}
```

**Requirements:**
- Enable Background Modes → Remote notifications
- Call completion handler within 30 seconds
- Return appropriate UIBackgroundFetchResult

**Use Cases:**
- Sync data in background
- Update content before user opens app
- Download new content
- Refresh cache

---

[← Previous: Testing](testing.md) | [Next: Security →](security.md)

[Back to Main](../README.md)
