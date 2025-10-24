# iOS Basics

[← Back to Main](../README.md) | [Previous: Introduction](introduction.md) | [Next: Swift Programming →](swift-programming.md)

## Table of Contents
- [History of iOS & Ecosystem](#history-of-ios--ecosystem)
- [Difference Between iOS and Other Mobile Platforms](#difference-between-ios-and-other-mobile-platforms)
- [iOS App Lifecycle](#ios-app-lifecycle)
- [UIViewController Lifecycle](#uiviewcontroller-lifecycle)
- [Common iOS Components](#common-ios-components--their-uses)

---

## History of iOS & Ecosystem

### Evolution of iOS

**2007 - iPhone OS 1.0**
- First iPhone released
- No App Store, no third-party apps
- Built-in apps only (Phone, Mail, Safari, iPod)

**2008 - iPhone OS 2.0**
- App Store launched with 500 apps
- iOS SDK released for developers
- Push notifications introduced

**2010 - iOS 4.0**
- Name changed from iPhone OS to iOS
- Multitasking support
- Folders and FaceTime introduced

**2013 - iOS 7.0**
- Complete UI redesign (flat design)
- Control Center introduced
- AirDrop support

**2014 - iOS 8.0**
- Swift programming language introduced
- App Extensions support
- HealthKit and HomeKit frameworks

**2019 - iOS 13.0**
- Dark Mode
- SwiftUI introduced
- Sign in with Apple

**2021 - iOS 15.0**
- Focus modes
- Live Text
- SharePlay

**2023 - iOS 17.0**
- StandBy mode
- Interactive widgets
- NameDrop

### iOS Ecosystem Components

**Development Tools:**
- Xcode: IDE for iOS development
- Instruments: Performance analysis
- TestFlight: Beta testing
- App Store Connect: App management

**Frameworks:**
- UIKit: Traditional UI framework
- SwiftUI: Modern declarative UI
- Foundation: Core utilities
- Core Data: Data persistence
- Combine: Reactive programming

**Services:**
- iCloud: Cloud storage
- CloudKit: Backend services
- Apple Push Notification Service (APNs)
- In-App Purchase
- Game Center

---

## Difference Between iOS and Other Mobile Platforms

### iOS vs Android

| Aspect | iOS | Android |
|--------|-----|---------|
| **Language** | Swift, Objective-C | Kotlin, Java |
| **IDE** | Xcode (macOS only) | Android Studio (Cross-platform) |
| **UI Framework** | UIKit, SwiftUI | XML, Jetpack Compose |
| **App Distribution** | App Store only (strict review) | Google Play, third-party stores |
| **Device Fragmentation** | Limited (iPhone, iPad) | High (thousands of devices) |
| **Memory Management** | ARC (Automatic Reference Counting) | Garbage Collection |
| **Development Cost** | Higher (Mac required) | Lower (any OS) |
| **Revenue** | Higher per user | Larger user base |
| **Updates** | High adoption rate | Fragmented updates |
| **Security** | Sandboxed, strict | More open, customizable |

### iOS vs React Native / Flutter

| Aspect | Native iOS | React Native | Flutter |
|--------|-----------|--------------|---------|
| **Performance** | Best (native) | Good | Very Good |
| **UI Consistency** | Platform-specific | JavaScript bridge | Custom rendering |
| **Learning Curve** | Steep (Swift + iOS) | Medium (JavaScript) | Medium (Dart) |
| **Code Reuse** | iOS only | iOS + Android | iOS + Android + Web |
| **Community** | Strong | Very Strong | Growing |
| **Hot Reload** | SwiftUI only | Yes | Yes |
| **App Size** | Smaller | Larger | Moderate |

### Why Choose Native iOS Development?

**Advantages:**
- Best performance and user experience
- Full access to latest iOS features immediately
- Better integration with Apple ecosystem
- Superior UI/UX with platform conventions
- Easier debugging and profiling
- Better security and privacy controls

**Disadvantages:**
- Mac and Xcode required
- Separate codebase for Android
- Longer development time for multi-platform
- Higher initial cost

---

## iOS App Lifecycle

The iOS app lifecycle defines the states an app transitions through from launch to termination.

### App States

```
Not Running → Inactive → Active → Background → Suspended → (Terminated)
```

**1. Not Running**
- App is not launched or was terminated by the system

**2. Inactive**
- App is running but not receiving events
- Transitional state between active and background

**3. Active**
- App is in foreground and receiving events
- Normal running state

**4. Background**
- App is executing code but not visible
- Limited execution time (typically 30 seconds)

**5. Suspended**
- App is in memory but not executing code
- Can be purged by system if memory is needed

### App Delegate Methods

```swift
import UIKit

@main
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    var window: UIWindow?
    
    // 1. App is about to launch
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        print("App launched")
        // Initialize app-wide resources
        setupAppearance()
        setupDatabase()
        return true
    }
    
    // 2. App will become active
    func applicationDidBecomeActive(_ application: UIApplication) {
        print("App became active")
        // Restart tasks, refresh UI
        startTimer()
        refreshData()
    }
    
    // 3. App will resign active (phone call, notification)
    func applicationWillResignActive(_ application: UIApplication) {
        print("App will resign active")
        // Pause ongoing tasks
        pauseTimer()
        saveState()
    }
    
    // 4. App entered background
    func applicationDidEnterBackground(_ application: UIApplication) {
        print("App entered background")
        // Save data, release resources
        saveUserData()
        releaseResources()
        
        // Request additional background time if needed
        var backgroundTask: UIBackgroundTaskIdentifier = .invalid
        backgroundTask = application.beginBackgroundTask {
            application.endBackgroundTask(backgroundTask)
            backgroundTask = .invalid
        }
        
        // Perform long-running task
        DispatchQueue.global().async {
            // Do work
            self.syncDataToServer()
            application.endBackgroundTask(backgroundTask)
            backgroundTask = .invalid
        }
    }
    
    // 5. App will enter foreground
    func applicationWillEnterForeground(_ application: UIApplication) {
        print("App will enter foreground")
        // Undo background changes
        refreshUI()
    }
    
    // 6. App will terminate
    func applicationWillTerminate(_ application: UIApplication) {
        print("App will terminate")
        // Save final state
        saveAllData()
        cleanup()
    }
    
    // Supporting methods
    private func setupAppearance() { }
    private func setupDatabase() { }
    private func startTimer() { }
    private func refreshData() { }
    private func pauseTimer() { }
    private func saveState() { }
    private func saveUserData() { }
    private func releaseResources() { }
    private func syncDataToServer() { }
    private func refreshUI() { }
    private func saveAllData() { }
    private func cleanup() { }
}
```

### Scene-Based Lifecycle (iOS 13+)

For apps with multiple windows, use SceneDelegate:

```swift
import UIKit

class SceneDelegate: UIResponder, UIWindowSceneDelegate {
    
    var window: UIWindow?
    
    // Scene is connecting
    func scene(
        _ scene: UIScene,
        willConnectTo session: UISceneSession,
        options connectionOptions: UIScene.ConnectionOptions
    ) {
        guard let windowScene = (scene as? UIWindowScene) else { return }
        
        window = UIWindow(windowScene: windowScene)
        window?.rootViewController = UINavigationController(
            rootViewController: HomeViewController()
        )
        window?.makeKeyAndVisible()
    }
    
    // Scene became active
    func sceneDidBecomeActive(_ scene: UIScene) {
        print("Scene became active")
    }
    
    // Scene will resign active
    func sceneWillResignActive(_ scene: UIScene) {
        print("Scene will resign active")
    }
    
    // Scene entered foreground
    func sceneWillEnterForeground(_ scene: UIScene) {
        print("Scene entered foreground")
    }
    
    // Scene entered background
    func sceneDidEnterBackground(_ scene: UIScene) {
        print("Scene entered background")
    }
    
    // Scene disconnected
    func sceneDidDisconnect(_ scene: UIScene) {
        print("Scene disconnected")
    }
}
```

### Practical Example: Managing User Session

```swift
class AppDelegate: UIResponder, UIApplicationDelegate {
    
    var sessionManager = SessionManager.shared
    
    func application(
        _ application: UIApplication,
        didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
    ) -> Bool {
        sessionManager.initialize()
        return true
    }
    
    func applicationDidEnterBackground(_ application: UIApplication) {
        // Save last active timestamp
        sessionManager.saveLastActiveTime()
    }
    
    func applicationWillEnterForeground(_ application: UIApplication) {
        // Check if session expired
        if sessionManager.isSessionExpired() {
            sessionManager.logout()
            showLoginScreen()
        }
    }
    
    func applicationWillTerminate(_ application: UIApplication) {
        sessionManager.cleanup()
    }
    
    private func showLoginScreen() {
        // Navigate to login
    }
}

class SessionManager {
    static let shared = SessionManager()
    private let sessionTimeout: TimeInterval = 15 * 60 // 15 minutes
    
    func initialize() {
        // Load saved session
    }
    
    func saveLastActiveTime() {
        UserDefaults.standard.set(Date(), forKey: "lastActiveTime")
    }
    
    func isSessionExpired() -> Bool {
        guard let lastActive = UserDefaults.standard.object(forKey: "lastActiveTime") as? Date else {
            return true
        }
        return Date().timeIntervalSince(lastActive) > sessionTimeout
    }
    
    func logout() {
        // Clear session data
        UserDefaults.standard.removeObject(forKey: "authToken")
    }
    
    func cleanup() {
        // Cleanup resources
    }
}
```

---

## UIViewController Lifecycle

ViewControllers manage the views that make up your app's UI. Understanding the lifecycle is crucial for proper resource management.

### Lifecycle Methods Order

```
1. init() / init(coder:)
2. loadView()
3. viewDidLoad()
4. viewWillAppear()
5. viewWillLayoutSubviews()
6. viewDidLayoutSubviews()
7. viewDidAppear()
   ↓ (View is visible)
8. viewWillDisappear()
9. viewDidDisappear()
10. deinit
```

### Detailed Lifecycle Methods

```swift
import UIKit

class ExampleViewController: UIViewController {
    
    // MARK: - Properties
    
    private let titleLabel = UILabel()
    private let dataService = DataService()
    
    // MARK: - Initialization
    
    init() {
        super.init(nibName: nil, bundle: nil)
        print("1. init() - ViewController initialized")
    }
    
    required init?(coder: NSCoder) {
        super.init(coder: coder)
        print("1. init(coder:) - ViewController initialized from storyboard")
    }
    
    deinit {
        print("10. deinit - ViewController deallocated")
        // Cleanup observers, timers
        NotificationCenter.default.removeObserver(self)
    }
    
    // MARK: - Lifecycle Methods
    
    override func loadView() {
        super.loadView()
        print("2. loadView() - Create view hierarchy")
        // Only override if creating views programmatically without storyboard
    }
    
    override func viewDidLoad() {
        super.viewDidLoad()
        print("3. viewDidLoad() - View loaded into memory")
        
        // Called once when view is first loaded
        // Perfect for:
        // - Initial setup
        // - Adding subviews
        // - Setting up constraints
        // - Registering for notifications
        
        setupUI()
        setupConstraints()
        registerNotifications()
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        print("4. viewWillAppear() - View is about to appear")
        
        // Called every time view is about to appear
        // Perfect for:
        // - Refreshing data
        // - Starting animations
        // - Updating UI
        
        navigationController?.setNavigationBarHidden(false, animated: animated)
        refreshData()
    }
    
    override func viewWillLayoutSubviews() {
        super.viewWillLayoutSubviews()
        print("5. viewWillLayoutSubviews() - About to layout subviews")
        
        // Called when bounds change (rotation, multitasking)
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        print("6. viewDidLayoutSubviews() - Subviews laid out")
        
        // Perfect for:
        // - Frame-based layout adjustments
        // - Updating layer properties
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        print("7. viewDidAppear() - View appeared on screen")
        
        // Called when view is fully visible
        // Perfect for:
        // - Starting heavy tasks
        // - Analytics tracking
        // - Starting media playback
        
        startTimer()
        trackScreenView()
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        print("8. viewWillDisappear() - View about to disappear")
        
        // Perfect for:
        // - Saving state
        // - Stopping animations
        // - Pausing media
        
        saveUserInput()
        stopTimer()
    }
    
    override func viewDidDisappear(_ animated: Bool) {
        super.viewDidDisappear(animated)
        print("9. viewDidDisappear() - View disappeared")
        
        // Perfect for:
        // - Cleanup
        // - Stopping background tasks
    }
    
    // MARK: - Memory Warning
    
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        print("Memory warning received")
        
        // Perfect for:
        // - Clearing caches
        // - Releasing recreatable resources
        
        imageCache.removeAll()
    }
    
    // MARK: - Setup Methods
    
    private func setupUI() {
        view.backgroundColor = .white
        
        titleLabel.text = "Example"
        titleLabel.font = .systemFont(ofSize: 24, weight: .bold)
        view.addSubview(titleLabel)
    }
    
    private func setupConstraints() {
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        NSLayoutConstraint.activate([
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            titleLabel.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
    
    private func registerNotifications() {
        NotificationCenter.default.addObserver(
            self,
            selector: #selector(handleNotification),
            name: .dataUpdated,
            object: nil
        )
    }
    
    @objc private func handleNotification() {
        refreshData()
    }
    
    private func refreshData() {
        dataService.fetchData { [weak self] result in
            DispatchQueue.main.async {
                self?.updateUI(with: result)
            }
        }
    }
    
    private func updateUI(with data: Any) {
        // Update UI
    }
    
    private func startTimer() { }
    private func stopTimer() { }
    private func saveUserInput() { }
    private func trackScreenView() { }
    
    private var imageCache: [String: UIImage] = [:]
}

// Supporting types
class DataService {
    func fetchData(completion: @escaping (Any) -> Void) {
        // Fetch data
    }
}

extension Notification.Name {
    static let dataUpdated = Notification.Name("dataUpdated")
}
```

### Practical Example: Managing Video Player

```swift
import UIKit
import AVFoundation

class VideoPlayerViewController: UIViewController {
    
    private var player: AVPlayer?
    private var playerLayer: AVPlayerLayer?
    private var timeObserver: Any?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupPlayer()
        // Don't start playing yet - user might not be on this screen
    }
    
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        // Prepare player for playback
        player?.pause()
    }
    
    override func viewDidLayoutSubviews() {
        super.viewDidLayoutSubviews()
        // Update player layer frame when bounds change
        playerLayer?.frame = view.bounds
    }
    
    override func viewDidAppear(_ animated: Bool) {
        super.viewDidAppear(animated)
        // Now it's safe to start playing
        player?.play()
        addTimeObserver()
    }
    
    override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        // Pause when user navigates away
        player?.pause()
        removeTimeObserver()
    }
    
    deinit {
        // Critical: cleanup player resources
        player?.pause()
        player = nil
        playerLayer?.removeFromSuperlayer()
    }
    
    private func setupPlayer() {
        let url = URL(string: "https://example.com/video.mp4")!
        player = AVPlayer(url: url)
        
        playerLayer = AVPlayerLayer(player: player)
        playerLayer?.videoGravity = .resizeAspect
        view.layer.addSublayer(playerLayer!)
    }
    
    private func addTimeObserver() {
        let interval = CMTime(seconds: 1, preferredTimescale: 600)
        timeObserver = player?.addPeriodicTimeObserver(
            forInterval: interval,
            queue: .main
        ) { [weak self] time in
            self?.updateProgressBar(time: time)
        }
    }
    
    private func removeTimeObserver() {
        if let observer = timeObserver {
            player?.removeTimeObserver(observer)
            timeObserver = nil
        }
    }
    
    private func updateProgressBar(time: CMTime) {
        // Update progress UI
    }
}
```

---

## Common iOS Components & Their Uses

### 1. UILabel

Displays read-only text.

```swift
let titleLabel = UILabel()
titleLabel.text = "Welcome to iOS"
titleLabel.font = .systemFont(ofSize: 24, weight: .bold)
titleLabel.textColor = .black
titleLabel.textAlignment = .center
titleLabel.numberOfLines = 0 // Unlimited lines
titleLabel.lineBreakMode = .byWordWrapping

// Attributed text for rich formatting
let attributedText = NSMutableAttributedString(string: "Hello World")
attributedText.addAttribute(
    .foregroundColor,
    value: UIColor.red,
    range: NSRange(location: 0, length: 5)
)
titleLabel.attributedText = attributedText
```

### 2. UIButton

Interactive button control.

```swift
let button = UIButton(type: .system)
button.setTitle("Tap Me", for: .normal)
button.setTitleColor(.white, for: .normal)
button.backgroundColor = .systemBlue
button.layer.cornerRadius = 8
button.addTarget(self, action: #selector(buttonTapped), for: .touchUpInside)

@objc func buttonTapped() {
    print("Button tapped")
}

// Modern iOS 15+ configuration
var config = UIButton.Configuration.filled()
config.title = "Submit"
config.image = UIImage(systemName: "checkmark")
config.imagePadding = 8
config.cornerStyle = .medium
let modernButton = UIButton(configuration: config)
```

### 3. UITextField

Single-line text input.

```swift
let textField = UITextField()
textField.placeholder = "Enter your name"
textField.borderStyle = .roundedRect
textField.keyboardType = .emailAddress
textField.autocapitalizationType = .none
textField.returnKeyType = .done
textField.delegate = self

// UITextFieldDelegate
extension ViewController: UITextFieldDelegate {
    func textFieldShouldReturn(_ textField: UITextField) -> Bool {
        textField.resignFirstResponder()
        return true
    }
    
    func textField(
        _ textField: UITextField,
        shouldChangeCharactersIn range: NSRange,
        replacementString string: String
    ) -> Bool {
        // Limit to 10 characters
        let currentText = textField.text ?? ""
        guard let stringRange = Range(range, in: currentText) else { return false }
        let updatedText = currentText.replacingCharacters(in: stringRange, with: string)
        return updatedText.count <= 10
    }
}
```

### 4. UITextView

Multi-line text input.

```swift
let textView = UITextView()
textView.font = .systemFont(ofSize: 16)
textView.text = "Enter description here..."
textView.textColor = .placeholderText
textView.layer.borderColor = UIColor.lightGray.cgColor
textView.layer.borderWidth = 1
textView.layer.cornerRadius = 8
textView.delegate = self

extension ViewController: UITextViewDelegate {
    func textViewDidBeginEditing(_ textView: UITextView) {
        if textView.textColor == .placeholderText {
            textView.text = ""
            textView.textColor = .label
        }
    }
    
    func textViewDidEndEditing(_ textView: UITextView) {
        if textView.text.isEmpty {
            textView.text = "Enter description here..."
            textView.textColor = .placeholderText
        }
    }
}
```

### 5. UIImageView

Displays images.

```swift
let imageView = UIImageView()
imageView.image = UIImage(named: "logo")
imageView.contentMode = .scaleAspectFit
imageView.clipsToBounds = true
imageView.layer.cornerRadius = 12

// Load from URL (with URLSession)
func loadImage(from url: URL) {
    URLSession.shared.dataTask(with: url) { [weak self] data, response, error in
        guard let data = data, let image = UIImage(data: data) else { return }
        DispatchQueue.main.async {
            self?.imageView.image = image
        }
    }.resume()
}

// SF Symbols (iOS 13+)
let symbolImage = UIImageView(image: UIImage(systemName: "heart.fill"))
symbolImage.tintColor = .red
```

### 6. UITableView

Scrollable list of rows.

```swift
class TableViewController: UIViewController {
    
    private let tableView = UITableView()
    private var items = ["Apple", "Banana", "Cherry", "Date"]
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        tableView.frame = view.bounds
        tableView.dataSource = self
        tableView.delegate = self
        tableView.register(UITableViewCell.self, forCellReuseIdentifier: "cell")
        view.addSubview(tableView)
    }
}

extension TableViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return items.count
    }
    
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "cell", for: indexPath)
        cell.textLabel?.text = items[indexPath.row]
        cell.accessoryType = .disclosureIndicator
        return cell
    }
}

extension TableViewController: UITableViewDelegate {
    func tableView(_ tableView: UITableView, didSelectRowAt indexPath: IndexPath) {
        tableView.deselectRow(at: indexPath, animated: true)
        print("Selected: \(items[indexPath.row])")
    }
    
    func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCell.EditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            items.remove(at: indexPath.row)
            tableView.deleteRows(at: [indexPath], with: .fade)
        }
    }
}
```

### 7. UICollectionView

Grid or custom layouts.

```swift
class CollectionViewController: UIViewController {
    
    private var collectionView: UICollectionView!
    private let items = Array(1...50)
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let layout = UICollectionViewFlowLayout()
        layout.itemSize = CGSize(width: 100, height: 100)
        layout.minimumInteritemSpacing = 10
        layout.minimumLineSpacing = 10
        layout.sectionInset = UIEdgeInsets(top: 10, left: 10, bottom: 10, right: 10)
        
        collectionView = UICollectionView(frame: view.bounds, collectionViewLayout: layout)
        collectionView.dataSource = self
        collectionView.delegate = self
        collectionView.register(UICollectionViewCell.self, forCellWithReuseIdentifier: "cell")
        collectionView.backgroundColor = .white
        view.addSubview(collectionView)
    }
}

extension CollectionViewController: UICollectionViewDataSource {
    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return items.count
    }
    
    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "cell", for: indexPath)
        cell.backgroundColor = .systemBlue
        cell.layer.cornerRadius = 8
        return cell
    }
}

extension CollectionViewController: UICollectionViewDelegate {
    func collectionView(_ collectionView: UICollectionView, didSelectItemAt indexPath: IndexPath) {
        print("Selected item \(items[indexPath.item])")
    }
}
```

### 8. UIScrollView

Scrollable content container.

```swift
let scrollView = UIScrollView()
scrollView.frame = view.bounds
scrollView.contentSize = CGSize(width: view.bounds.width, height: 2000)
scrollView.delegate = self
scrollView.showsVerticalScrollIndicator = true
scrollView.bounces = true

extension ViewController: UIScrollViewDelegate {
    func scrollViewDidScroll(_ scrollView: UIScrollView) {
        let offset = scrollView.contentOffset.y
        // Update parallax header, etc.
    }
    
    func scrollViewDidEndDecelerating(_ scrollView: UIScrollView) {
        // Pagination logic
        let pageWidth = scrollView.frame.width
        let currentPage = Int(scrollView.contentOffset.x / pageWidth)
        print("Current page: \(currentPage)")
    }
}
```

### 9. UIStackView

Automatic layout for arranged subviews.

```swift
let stackView = UIStackView()
stackView.axis = .vertical
stackView.alignment = .fill
stackView.distribution = .fillEqually
stackView.spacing = 16

let label1 = UILabel()
label1.text = "Label 1"

let label2 = UILabel()
label2.text = "Label 2"

stackView.addArrangedSubview(label1)
stackView.addArrangedSubview(label2)

// Dynamic add/remove
let newLabel = UILabel()
newLabel.text = "New Label"
stackView.insertArrangedSubview(newLabel, at: 1)

// Remove
stackView.removeArrangedSubview(label1)
label1.removeFromSuperview()
```

### 10. UINavigationController

Hierarchical navigation.

```swift
// Setup in AppDelegate or SceneDelegate
let homeVC = HomeViewController()
let navController = UINavigationController(rootViewController: homeVC)
window?.rootViewController = navController

// In HomeViewController
class HomeViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        title = "Home"
        navigationItem.rightBarButtonItem = UIBarButtonItem(
            barButtonSystemItem: .add,
            target: self,
            action: #selector(addTapped)
        )
    }
    
    @objc func addTapped() {
        let detailVC = DetailViewController()
        navigationController?.pushViewController(detailVC, animated: true)
    }
}

// Customization
navigationController?.navigationBar.prefersLargeTitles = true
navigationController?.navigationBar.tintColor = .systemBlue
navigationController?.navigationBar.backgroundColor = .white
```

### 11. UITabBarController

Tab-based navigation.

```swift
let tabBarController = UITabBarController()

let homeVC = HomeViewController()
homeVC.tabBarItem = UITabBarItem(
    title: "Home",
    image: UIImage(systemName: "house"),
    selectedImage: UIImage(systemName: "house.fill")
)

let profileVC = ProfileViewController()
profileVC.tabBarItem = UITabBarItem(
    title: "Profile",
    image: UIImage(systemName: "person"),
    selectedImage: UIImage(systemName: "person.fill")
)

tabBarController.viewControllers = [
    UINavigationController(rootViewController: homeVC),
    UINavigationController(rootViewController: profileVC)
]

// Customization
tabBarController.tabBar.tintColor = .systemBlue
tabBarController.tabBar.unselectedItemTintColor = .gray
tabBarController.tabBar.backgroundColor = .white
```

### 12. UIAlertController

Alerts and action sheets.

```swift
// Alert
func showAlert() {
    let alert = UIAlertController(
        title: "Delete Item",
        message: "Are you sure you want to delete this item?",
        preferredStyle: .alert
    )
    
    alert.addAction(UIAlertAction(title: "Cancel", style: .cancel))
    alert.addAction(UIAlertAction(title: "Delete", style: .destructive) { _ in
        self.deleteItem()
    })
    
    present(alert, animated: true)
}

// Action Sheet
func showActionSheet() {
    let actionSheet = UIAlertController(
        title: "Choose Option",
        message: nil,
        preferredStyle: .actionSheet
    )
    
    actionSheet.addAction(UIAlertAction(title: "Camera", style: .default) { _ in
        self.openCamera()
    })
    actionSheet.addAction(UIAlertAction(title: "Photo Library", style: .default) { _ in
        self.openPhotoLibrary()
    })
    actionSheet.addAction(UIAlertAction(title: "Cancel", style: .cancel))
    
    // For iPad
    if let popover = actionSheet.popoverPresentationController {
        popover.sourceView = self.view
        popover.sourceRect = CGRect(x: view.bounds.midX, y: view.bounds.midY, width: 0, height: 0)
        popover.permittedArrowDirections = []
    }
    
    present(actionSheet, animated: true)
}

// Text Input Alert
func showTextInputAlert() {
    let alert = UIAlertController(
        title: "Enter Name",
        message: nil,
        preferredStyle: .alert
    )
    
    alert.addTextField { textField in
        textField.placeholder = "Name"
    }
    
    alert.addAction(UIAlertAction(title: "Cancel", style: .cancel))
    alert.addAction(UIAlertAction(title: "Save", style: .default) { _ in
        if let text = alert.textFields?.first?.text {
            print("Entered: \(text)")
        }
    })
    
    present(alert, animated: true)
}

func deleteItem() { }
func openCamera() { }
func openPhotoLibrary() { }
```

### Interview Questions

**Q1: What's the difference between viewDidLoad and viewWillAppear?**

**Answer:** `viewDidLoad` is called once when the view is first loaded into memory. It's perfect for one-time setup. `viewWillAppear` is called every time the view is about to appear on screen, ideal for refreshing data or updating UI.

**Q2: When would you use UICollectionView instead of UITableView?**

**Answer:** Use UITableView for simple vertical lists. Use UICollectionView for:
- Grid layouts
- Custom layouts (horizontal, waterfall, circular)
- Complex reordering
- Multiple section layouts

**Q3: What happens if you don't call super in lifecycle methods?**

**Answer:** Not calling super can lead to undefined behavior. The superclass may have important setup that won't execute, potentially causing crashes or incorrect behavior.

**Q4: How do you prevent memory leaks in UIViewController?**

**Answer:**
- Use `[weak self]` in closures
- Remove observers in deinit
- Invalidate timers
- Cancel network requests
- Remove delegates when not needed

**Q5: Explain the iOS App Lifecycle states and when each is called**

**Answer:**

1. **Not Running**: App hasn't been launched or was terminated
2. **Inactive**: App is transitioning between states (brief, like during phone call)
3. **Active**: App is in foreground and receiving events - normal running state
4. **Background**: App is executing code but not visible (limited time ~30 seconds)
5. **Suspended**: App is in memory but not executing code, can be purged by system

**Example Use Cases:**
- `applicationDidEnterBackground`: Save user data, release resources
- `applicationWillEnterForeground`: Refresh UI, restart paused tasks
- `applicationDidBecomeActive`: Start animations, resume game
- `applicationWillResignActive`: Pause ongoing tasks, save state

**Q6: What's the difference between frame and bounds?**

**Answer:**

**Frame:**
- Position and size relative to superview's coordinate system
- Use when positioning a view within its parent
- Example: `view.frame = CGRect(x: 50, y: 100, width: 200, height: 150)`

**Bounds:**
- Position and size in its own coordinate system
- Origin is typically (0,0)
- Use for drawing or positioning subviews
- Changing bounds affects how content is displayed
- Example: `view.bounds = CGRect(x: 0, y: 0, width: 200, height: 150)`

**Practical Example:**
```swift
let view = UIView(frame: CGRect(x: 50, y: 100, width: 200, height: 150))
print(view.frame.origin)  // (50, 100)
print(view.bounds.origin) // (0, 0)
```

**Q7: How do you pass data between view controllers?**

**Answer:**

**1. Property Assignment (Forward):**
```swift
let detailVC = DetailViewController()
detailVC.user = selectedUser
navigationController?.pushViewController(detailVC, animated: true)
```

**2. Delegation (Backward):**
```swift
protocol DetailDelegate: AnyObject {
    func didUpdateUser(_ user: User)
}

class DetailViewController: UIViewController {
    weak var delegate: DetailDelegate?
}
```

**3. Closures (Backward):**
```swift
class DetailViewController: UIViewController {
    var onComplete: ((User) -> Void)?
}
```

**4. Notification Center (Many-to-Many):**
```swift
NotificationCenter.default.post(name: .userUpdated, object: user)
```

**5. Segues (Storyboards):**
```swift
override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
    if let detailVC = segue.destination as? DetailViewController {
        detailVC.user = selectedUser
    }
}
```

**Best Practice:** Use delegation for backward communication, property assignment for forward, avoid NotificationCenter unless truly broadcasting to multiple observers.

---

[← Previous: Introduction](introduction.md) | [Next: Swift Programming →](swift-programming.md)

[Back to Main](../README.md)

