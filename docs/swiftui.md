# SwiftUI

[← Back to Main](../README.md) | [Previous: UIKit Development](uikit-development.md) | [Next: Data Persistence →](data-persistence.md)

## Table of Contents
- [Introduction to SwiftUI](#introduction-to-swiftui)
- [Declarative UI vs Imperative UI](#declarative-ui-vs-imperative-ui)
- [State Management](#state-management)
- [Navigation in SwiftUI](#navigation-in-swiftui)
- [Lists & Grids](#lists--grids)
- [Animations](#animations-in-swiftui)
- [Forms & User Input](#forms--user-input)
- [Interoperability with UIKit](#interoperability-with-uikit)
- [Best Practices & Performance](#best-practices--performance-optimization)

---

## Introduction to SwiftUI

SwiftUI is Apple's modern declarative framework for building user interfaces across all Apple platforms.

### Key Features

- **Declarative**: Describe what UI should look like, not how to build it
- **Cross-platform**: Works on iOS, iPadOS, macOS, watchOS, tvOS
- **Live Preview**: See changes instantly in Xcode
- **Automatic Updates**: UI automatically reflects state changes
- **Native Performance**: Compiles to native code
- **Accessibility**: Built-in accessibility support

### Basic SwiftUI View

```swift
import SwiftUI

struct ContentView: View {
    var body: some View {
        Text("Hello, SwiftUI!")
            .font(.largeTitle)
            .foregroundColor(.blue)
            .padding()
    }
}

// Preview
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        ContentView()
    }
}
```

### Common Views

```swift
struct BasicViewsExample: View {
    var body: some View {
        VStack(spacing: 20) {
            // Text
            Text("Welcome to SwiftUI")
                .font(.title)
                .fontWeight(.bold)
            
            // Image
            Image(systemName: "star.fill")
                .foregroundColor(.yellow)
                .font(.system(size: 50))
            
            // Button
            Button("Tap Me") {
                print("Button tapped")
            }
            .buttonStyle(.borderedProminent)
            
            // TextField
            TextField("Enter text", text: .constant(""))
                .textFieldStyle(.roundedBorder)
                .padding(.horizontal)
            
            // Toggle
            Toggle("Enable notifications", isOn: .constant(true))
                .padding(.horizontal)
            
            // Slider
            Slider(value: .constant(0.5))
                .padding(.horizontal)
            
            // ProgressView
            ProgressView(value: 0.6)
                .padding(.horizontal)
        }
        .padding()
    }
}
```

### View Modifiers

```swift
struct ModifiersExample: View {
    var body: some View {
        Text("Styled Text")
            .font(.title)
            .fontWeight(.semibold)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .cornerRadius(10)
            .shadow(radius: 5)
            .padding(.horizontal)
    }
}
```

---

## Declarative UI vs Imperative UI

### Imperative (UIKit)

```swift
// UIKit - Imperative
class ImperativeViewController: UIViewController {
    private let label = UILabel()
    private let button = UIButton()
    private var counter = 0
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Setup label
        label.text = "Count: 0"
        label.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(label)
        
        // Setup button
        button.setTitle("Increment", for: .normal)
        button.setTitleColor(.blue, for: .normal)
        button.addTarget(self, action: #selector(incrementTapped), for: .touchUpInside)
        button.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(button)
        
        // Setup constraints
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor),
            
            button.topAnchor.constraint(equalTo: label.bottomAnchor, constant: 20),
            button.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }
    
    @objc func incrementTapped() {
        counter += 1
        label.text = "Count: \(counter)" // Manual update
    }
}
```

### Declarative (SwiftUI)

```swift
// SwiftUI - Declarative
struct DeclarativeView: View {
    @State private var counter = 0
    
    var body: some View {
        VStack(spacing: 20) {
            Text("Count: \(counter)")
            
            Button("Increment") {
                counter += 1 // Automatic UI update
            }
        }
    }
}
```

### Comparison

| Aspect | Imperative (UIKit) | Declarative (SwiftUI) |
|--------|-------------------|---------------------|
| **Approach** | How to build UI | What UI should be |
| **Updates** | Manual | Automatic |
| **Code** | More verbose | Concise |
| **State** | Managed manually | Managed by framework |
| **Preview** | Simulator/Device | Live preview |
| **Learning Curve** | Steeper | Gentler |

---

## State Management

### @State

Local mutable state owned by the view.

```swift
struct StateExample: View {
    @State private var name = ""
    @State private var age = 18
    @State private var isSubscribed = false
    
    var body: some View {
        Form {
            Section("Personal Info") {
                TextField("Name", text: $name)
                Stepper("Age: \(age)", value: $age, in: 0...120)
                Toggle("Subscribe to newsletter", isOn: $isSubscribed)
            }
            
            Section("Summary") {
                Text("Name: \(name)")
                Text("Age: \(age)")
                Text("Subscribed: \(isSubscribed ? "Yes" : "No")")
            }
        }
    }
}
```

### @Binding

Reference to state owned by parent view.

```swift
struct BindingExample: View {
    @State private var isOn = false
    
    var body: some View {
        VStack {
            Text("Switch is \(isOn ? "ON" : "OFF")")
            
            ToggleView(isOn: $isOn)
        }
    }
}

struct ToggleView: View {
    @Binding var isOn: Bool
    
    var body: some View {
        Toggle("Toggle", isOn: $isOn)
            .padding()
    }
}
```

### @ObservableObject & @Published

For complex state management.

```swift
class UserViewModel: ObservableObject {
    @Published var name = ""
    @Published var email = ""
    @Published var users: [User] = []
    @Published var isLoading = false
    @Published var errorMessage: String?
    
    func fetchUsers() {
        isLoading = true
        
        // Simulate API call
        DispatchQueue.main.asyncAfter(deadline: .now() + 2) {
            self.users = [
                User(id: 1, name: "John Doe", email: "john@example.com"),
                User(id: 2, name: "Jane Smith", email: "jane@example.com")
            ]
            self.isLoading = false
        }
    }
    
    func addUser() {
        let newUser = User(
            id: users.count + 1,
            name: name,
            email: email
        )
        users.append(newUser)
        name = ""
        email = ""
    }
    
    func deleteUser(at offsets: IndexSet) {
        users.remove(atOffsets: offsets)
    }
}

struct User: Identifiable {
    let id: Int
    let name: String
    let email: String
}

struct ObservableObjectExample: View {
    @StateObject private var viewModel = UserViewModel()
    
    var body: some View {
        NavigationView {
            VStack {
                if viewModel.isLoading {
                    ProgressView("Loading...")
                } else {
                    List {
                        ForEach(viewModel.users) { user in
                            VStack(alignment: .leading) {
                                Text(user.name)
                                    .font(.headline)
                                Text(user.email)
                                    .font(.subheadline)
                                    .foregroundColor(.gray)
                            }
                        }
                        .onDelete(perform: viewModel.deleteUser)
                    }
                }
            }
            .navigationTitle("Users")
            .toolbar {
                Button("Refresh") {
                    viewModel.fetchUsers()
                }
            }
            .onAppear {
                viewModel.fetchUsers()
            }
        }
    }
}
```

### @StateObject vs @ObservedObject

```swift
// @StateObject - View owns the object
struct StateObjectExample: View {
    @StateObject private var viewModel = MyViewModel()
    
    var body: some View {
        Text(viewModel.text)
    }
}

// @ObservedObject - Object passed from parent
struct ObservedObjectExample: View {
    @ObservedObject var viewModel: MyViewModel
    
    var body: some View {
        Text(viewModel.text)
    }
}

class MyViewModel: ObservableObject {
    @Published var text = "Hello"
}
```

### @EnvironmentObject

Share object across many views.

```swift
class AppState: ObservableObject {
    @Published var isLoggedIn = false
    @Published var username = ""
    @Published var theme: Theme = .light
}

enum Theme {
    case light, dark
}

@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)
        }
    }
}

struct ContentView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        if appState.isLoggedIn {
            HomeView()
        } else {
            LoginView()
        }
    }
}

struct HomeView: View {
    @EnvironmentObject var appState: AppState
    
    var body: some View {
        VStack {
            Text("Welcome, \(appState.username)!")
            
            Button("Logout") {
                appState.isLoggedIn = false
            }
        }
    }
}

struct LoginView: View {
    @EnvironmentObject var appState: AppState
    @State private var username = ""
    
    var body: some View {
        VStack {
            TextField("Username", text: $username)
                .textFieldStyle(.roundedBorder)
                .padding()
            
            Button("Login") {
                appState.username = username
                appState.isLoggedIn = true
            }
        }
    }
}
```

### @Environment

Access system-provided values.

```swift
struct EnvironmentExample: View {
    @Environment(\.colorScheme) var colorScheme
    @Environment(\.horizontalSizeClass) var sizeClass
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        VStack {
            Text("Color Scheme: \(colorScheme == .dark ? "Dark" : "Light")")
            Text("Size Class: \(sizeClass == .compact ? "Compact" : "Regular")")
            
            Button("Dismiss") {
                dismiss()
            }
        }
    }
}
```

---

## Navigation in SwiftUI

### NavigationStack (iOS 16+)

```swift
struct NavigationStackExample: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            List {
                NavigationLink("Go to Detail") {
                    DetailView(text: "First Detail")
                }
                
                NavigationLink("Go to Settings") {
                    SettingsView()
                }
                
                Button("Deep Link") {
                    path.append("Detail 1")
                    path.append("Detail 2")
                }
            }
            .navigationTitle("Home")
            .navigationDestination(for: String.self) { value in
                DetailView(text: value)
            }
        }
    }
}

struct DetailView: View {
    let text: String
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        VStack {
            Text(text)
                .font(.largeTitle)
            
            Button("Go Back") {
                dismiss()
            }
            
            NavigationLink("Go Deeper") {
                DetailView(text: "Nested Detail")
            }
        }
        .navigationTitle("Detail")
        .navigationBarTitleDisplayMode(.inline)
    }
}

struct SettingsView: View {
    var body: some View {
        Form {
            Section("General") {
                Toggle("Notifications", isOn: .constant(true))
                Toggle("Dark Mode", isOn: .constant(false))
            }
        }
        .navigationTitle("Settings")
    }
}
```

### Sheet & FullScreenCover

```swift
struct SheetExample: View {
    @State private var showingSheet = false
    @State private var showingFullScreen = false
    
    var body: some View {
        VStack(spacing: 20) {
            Button("Show Sheet") {
                showingSheet = true
            }
            
            Button("Show Full Screen") {
                showingFullScreen = true
            }
        }
        .sheet(isPresented: $showingSheet) {
            SheetContentView()
        }
        .fullScreenCover(isPresented: $showingFullScreen) {
            FullScreenContentView()
        }
    }
}

struct SheetContentView: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        NavigationView {
            VStack {
                Text("Sheet Content")
                    .font(.title)
            }
            .navigationTitle("Sheet")
            .navigationBarTitleDisplayMode(.inline)
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("Done") {
                        dismiss()
                    }
                }
            }
        }
    }
}

struct FullScreenContentView: View {
    @Environment(\.dismiss) var dismiss
    
    var body: some View {
        ZStack {
            Color.blue.ignoresSafeArea()
            
            VStack {
                Text("Full Screen Content")
                    .foregroundColor(.white)
                    .font(.title)
                
                Button("Dismiss") {
                    dismiss()
                }
                .foregroundColor(.white)
            }
        }
    }
}
```

### Alert & Confirmation Dialog

```swift
struct AlertExample: View {
    @State private var showingAlert = false
    @State private var showingDialog = false
    
    var body: some View {
        VStack(spacing: 20) {
            Button("Show Alert") {
                showingAlert = true
            }
            
            Button("Show Dialog") {
                showingDialog = true
            }
        }
        .alert("Important Message", isPresented: $showingAlert) {
            Button("OK", role: .cancel) { }
            Button("Delete", role: .destructive) {
                print("Deleted")
            }
        } message: {
            Text("This is a detailed message")
        }
        .confirmationDialog("Select an option", isPresented: $showingDialog) {
            Button("Option 1") {
                print("Option 1")
            }
            Button("Option 2") {
                print("Option 2")
            }
            Button("Cancel", role: .cancel) { }
        }
    }
}
```

---

## Lists & Grids

### List

```swift
struct ListExample: View {
    @State private var items = ["Apple", "Banana", "Cherry", "Date"]
    
    var body: some View {
        NavigationView {
            List {
                Section("Fruits") {
                    ForEach(items, id: \.self) { item in
                        HStack {
                            Image(systemName: "leaf.fill")
                                .foregroundColor(.green)
                            Text(item)
                        }
                    }
                    .onDelete(perform: delete)
                    .onMove(perform: move)
                }
                
                Section("Actions") {
                    Button("Add Item") {
                        items.append("New Item")
                    }
                }
            }
            .navigationTitle("List Example")
            .toolbar {
                EditButton()
            }
        }
    }
    
    func delete(at offsets: IndexSet) {
        items.remove(atOffsets: offsets)
    }
    
    func move(from source: IndexSet, to destination: Int) {
        items.move(fromOffsets: source, toOffset: destination)
    }
}
```

### Custom List Rows

```swift
struct CustomListExample: View {
    let users = [
        User(id: 1, name: "John Doe", email: "john@example.com"),
        User(id: 2, name: "Jane Smith", email: "jane@example.com"),
        User(id: 3, name: "Bob Johnson", email: "bob@example.com")
    ]
    
    var body: some View {
        List(users) { user in
            UserRow(user: user)
        }
    }
}

struct UserRow: View {
    let user: User
    
    var body: some View {
        HStack(spacing: 12) {
            Image(systemName: "person.circle.fill")
                .font(.system(size: 40))
                .foregroundColor(.blue)
            
            VStack(alignment: .leading, spacing: 4) {
                Text(user.name)
                    .font(.headline)
                
                Text(user.email)
                    .font(.subheadline)
                    .foregroundColor(.gray)
            }
            
            Spacer()
            
            Image(systemName: "chevron.right")
                .foregroundColor(.gray)
        }
        .padding(.vertical, 8)
    }
}
```

### LazyVGrid & LazyHGrid

```swift
struct GridExample: View {
    let columns = [
        GridItem(.adaptive(minimum: 100), spacing: 16)
    ]
    
    let items = Array(1...50)
    
    var body: some View {
        ScrollView {
            LazyVGrid(columns: columns, spacing: 16) {
                ForEach(items, id: \.self) { item in
                    RoundedRectangle(cornerRadius: 10)
                        .fill(Color.blue)
                        .frame(height: 100)
                        .overlay(
                            Text("\(item)")
                                .foregroundColor(.white)
                                .font(.title)
                        )
                }
            }
            .padding()
        }
    }
}

struct HorizontalGridExample: View {
    let rows = [
        GridItem(.fixed(100)),
        GridItem(.fixed(100))
    ]
    
    var body: some View {
        ScrollView(.horizontal) {
            LazyHGrid(rows: rows, spacing: 16) {
                ForEach(1...20, id: \.self) { item in
                    RoundedRectangle(cornerRadius: 10)
                        .fill(Color.green)
                        .frame(width: 100)
                }
            }
            .padding()
        }
    }
}
```

---

## Animations in SwiftUI

### Basic Animations

```swift
struct BasicAnimationExample: View {
    @State private var isExpanded = false
    
    var body: some View {
        VStack {
            RoundedRectangle(cornerRadius: isExpanded ? 50 : 10)
                .fill(isExpanded ? Color.blue : Color.red)
                .frame(width: isExpanded ? 200 : 100, height: 100)
                .animation(.spring(), value: isExpanded)
            
            Button("Animate") {
                isExpanded.toggle()
            }
        }
    }
}
```

### Animation Types

```swift
struct AnimationTypesExample: View {
    @State private var offset: CGFloat = 0
    
    var body: some View {
        VStack(spacing: 20) {
            Circle()
                .fill(Color.blue)
                .frame(width: 50, height: 50)
                .offset(x: offset)
                .animation(.linear(duration: 1), value: offset)
            
            Circle()
                .fill(Color.red)
                .frame(width: 50, height: 50)
                .offset(x: offset)
                .animation(.easeIn(duration: 1), value: offset)
            
            Circle()
                .fill(Color.green)
                .frame(width: 50, height: 50)
                .offset(x: offset)
                .animation(.spring(response: 0.5, dampingFraction: 0.6), value: offset)
            
            Button("Animate") {
                offset = offset == 0 ? 100 : 0
            }
        }
    }
}
```

### Transitions

```swift
struct TransitionExample: View {
    @State private var isShowing = false
    
    var body: some View {
        VStack {
            Button("Toggle") {
                withAnimation(.spring()) {
                    isShowing.toggle()
                }
            }
            
            if isShowing {
                Text("Hello!")
                    .font(.largeTitle)
                    .transition(.scale.combined(with: .opacity))
            }
        }
    }
}
```

### Custom Animations

```swift
struct PulsingAnimation: View {
    @State private var isPulsing = false
    
    var body: some View {
        Circle()
            .fill(Color.red)
            .frame(width: 100, height: 100)
            .scaleEffect(isPulsing ? 1.3 : 1.0)
            .opacity(isPulsing ? 0.6 : 1.0)
            .animation(
                .easeInOut(duration: 1)
                .repeatForever(autoreverses: true),
                value: isPulsing
            )
            .onAppear {
                isPulsing = true
            }
    }
}

struct RotatingAnimation: View {
    @State private var rotation: Double = 0
    
    var body: some View {
        Image(systemName: "arrow.right")
            .font(.system(size: 50))
            .rotationEffect(.degrees(rotation))
            .onAppear {
                withAnimation(.linear(duration: 2).repeatForever(autoreverses: false)) {
                    rotation = 360
                }
            }
    }
}
```

---

## Forms & User Input

### Form

```swift
struct FormExample: View {
    @State private var name = ""
    @State private var email = ""
    @State private var birthdate = Date()
    @State private var selectedCountry = "USA"
    @State private var notifications = true
    @State private var newsletter = false
    
    let countries = ["USA", "UK", "Canada", "Australia"]
    
    var body: some View {
        NavigationView {
            Form {
                Section("Personal Information") {
                    TextField("Name", text: $name)
                    TextField("Email", text: $email)
                        .keyboardType(.emailAddress)
                        .autocapitalization(.none)
                    
                    DatePicker(
                        "Birthdate",
                        selection: $birthdate,
                        displayedComponents: .date
                    )
                }
                
                Section("Location") {
                    Picker("Country", selection: $selectedCountry) {
                        ForEach(countries, id: \.self) { country in
                            Text(country)
                        }
                    }
                }
                
                Section("Preferences") {
                    Toggle("Enable Notifications", isOn: $notifications)
                    Toggle("Subscribe to Newsletter", isOn: $newsletter)
                }
                
                Section {
                    Button("Submit") {
                        submitForm()
                    }
                }
            }
            .navigationTitle("Registration")
        }
    }
    
    func submitForm() {
        print("Name: \(name)")
        print("Email: \(email)")
        print("Birthdate: \(birthdate)")
        print("Country: \(selectedCountry)")
    }
}
```

### Validation

```swift
struct ValidatedFormExample: View {
    @State private var email = ""
    @State private var password = ""
    @State private var confirmPassword = ""
    @State private var showingAlert = false
    @State private var alertMessage = ""
    
    var isValidEmail: Bool {
        email.contains("@") && email.contains(".")
    }
    
    var isValidPassword: Bool {
        password.count >= 8
    }
    
    var passwordsMatch: Bool {
        password == confirmPassword
    }
    
    var isFormValid: Bool {
        isValidEmail && isValidPassword && passwordsMatch
    }
    
    var body: some View {
        NavigationView {
            Form {
                Section("Email") {
                    TextField("Email", text: $email)
                        .autocapitalization(.none)
                        .keyboardType(.emailAddress)
                    
                    if !email.isEmpty && !isValidEmail {
                        Text("Invalid email format")
                            .foregroundColor(.red)
                            .font(.caption)
                    }
                }
                
                Section("Password") {
                    SecureField("Password", text: $password)
                    
                    if !password.isEmpty && !isValidPassword {
                        Text("Password must be at least 8 characters")
                            .foregroundColor(.red)
                            .font(.caption)
                    }
                    
                    SecureField("Confirm Password", text: $confirmPassword)
                    
                    if !confirmPassword.isEmpty && !passwordsMatch {
                        Text("Passwords don't match")
                            .foregroundColor(.red)
                            .font(.caption)
                    }
                }
                
                Section {
                    Button("Register") {
                        register()
                    }
                    .disabled(!isFormValid)
                }
            }
            .navigationTitle("Sign Up")
            .alert("Registration", isPresented: $showingAlert) {
                Button("OK", role: .cancel) { }
            } message: {
                Text(alertMessage)
            }
        }
    }
    
    func register() {
        alertMessage = "Registration successful!"
        showingAlert = true
    }
}
```

---

## Interoperability with UIKit

### Using UIViewController in SwiftUI

```swift
import SwiftUI
import UIKit

struct UIViewControllerExample: UIViewControllerRepresentable {
    let viewController: UIViewController
    
    func makeUIViewController(context: Context) -> UIViewController {
        return viewController
    }
    
    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
        // Update if needed
    }
}

// Usage
struct ContentViewWithUIKit: View {
    var body: some View {
        UIViewControllerExample(viewController: MyUIKitViewController())
    }
}

class MyUIKitViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBlue
        
        let label = UILabel()
        label.text = "UIKit View Controller"
        label.textColor = .white
        label.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(label)
        
        NSLayoutConstraint.activate([
            label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            label.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

### Using UIView in SwiftUI

```swift
struct UIViewExample: UIViewRepresentable {
    let color: UIColor
    
    func makeUIView(context: Context) -> UIView {
        let view = UIView()
        view.backgroundColor = color
        return view
    }
    
    func updateUIView(_ uiView: UIView, context: Context) {
        uiView.backgroundColor = color
    }
}

// Map View Example
import MapKit

struct MapView: UIViewRepresentable {
    @Binding var region: MKCoordinateRegion
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        return mapView
    }
    
    func updateUIView(_ uiView: MKMapView, context: Context) {
        uiView.setRegion(region, animated: true)
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, MKMapViewDelegate {
        var parent: MapView
        
        init(_ parent: MapView) {
            self.parent = parent
        }
    }
}
```

### Using SwiftUI in UIKit

```swift
import SwiftUI
import UIKit

class HostingController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        let swiftUIView = MySwiftUIView()
        let hostingController = UIHostingController(rootView: swiftUIView)
        
        addChild(hostingController)
        view.addSubview(hostingController.view)
        hostingController.view.frame = view.bounds
        hostingController.didMove(toParent: self)
    }
}

struct MySwiftUIView: View {
    var body: some View {
        VStack {
            Text("SwiftUI View")
                .font(.largeTitle)
            
            Button("Tap Me") {
                print("Button tapped")
            }
        }
    }
}
```

---

## Best Practices & Performance Optimization

### Avoid Common Pitfalls

```swift
// BAD: Creating new view in body
struct BadExample: View {
    var body: some View {
        VStack {
            // Creates new instance every time
            Text("Hello")
                .foregroundColor(randomColor())
        }
    }
    
    func randomColor() -> Color {
        [Color.red, .blue, .green].randomElement()!
    }
}

// GOOD: Use state
struct GoodExample: View {
    @State private var color = Color.red
    
    var body: some View {
        VStack {
            Text("Hello")
                .foregroundColor(color)
        }
        .onAppear {
            color = [Color.red, .blue, .green].randomElement()!
        }
    }
}
```

### Performance Tips

```swift
// Use @ViewBuilder for complex views
@ViewBuilder
func complexView() -> some View {
    if condition {
        ViewA()
    } else {
        ViewB()
    }
}

// Prefer LazyVStack/LazyHStack for long lists
struct OptimizedListView: View {
    let items = Array(1...1000)
    
    var body: some View {
        ScrollView {
            LazyVStack {
                ForEach(items, id: \.self) { item in
                    Text("Item \(item)")
                }
            }
        }
    }
}

// Use equatable for complex views
struct ExpensiveView: View, Equatable {
    let data: String
    
    var body: some View {
        Text(data)
            .padding()
    }
    
    static func == (lhs: ExpensiveView, rhs: ExpensiveView) -> Bool {
        lhs.data == rhs.data
    }
}
```

### Best Practices

```swift
// 1. Break down complex views
struct ComplexView: View {
    var body: some View {
        VStack {
            HeaderView()
            ContentView()
            FooterView()
        }
    }
}

// 2. Use private for internal state
struct BestPracticeView: View {
    @State private var count = 0 // private
    
    var body: some View {
        Text("\(count)")
    }
}

// 3. Prefer immutability
struct DataModel {
    let id: Int
    let name: String
    // Instead of var
}

// 4. Use proper state management
class AppViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var error: Error?
    @Published var data: [Item] = []
}
```

### Interview Questions

**Q1: What's the difference between @State and @StateObject?**

**Answer:** @State is for simple value types owned by the view. @StateObject is for reference types (ObservableObject) that should persist across view updates. Use @StateObject when the view creates and owns the object.

**Q2: When should you use @ObservedObject vs @EnvironmentObject?**

**Answer:** Use @ObservedObject when passing an object from parent to child. Use @EnvironmentObject when you need to share an object across many views without passing it explicitly through each level.

**Q3: How do you optimize List performance in SwiftUI?**

**Answer:**
- Use LazyVStack instead of VStack for long lists
- Implement Identifiable protocol
- Use .id() modifier to help SwiftUI track changes
- Avoid heavy computations in body
- Use @ViewBuilder for conditional views

---

[← Previous: UIKit Development](uikit-development.md) | [Next: Data Persistence →](data-persistence.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between @State, @Binding, @StateObject, and @ObservedObject?

**Answer:**

**@State** - Source of truth for view-local state:
```swift
struct CounterView: View {
    @State private var count = 0  // View owns this
    
    var body: some View {
        Button("Count: \(count)") {
            count += 1
        }
    }
}
```

**@Binding** - Reference to @State owned by parent:
```swift
struct ToggleView: View {
    @Binding var isOn: Bool  // Parent owns this
    
    var body: some View {
        Toggle("Toggle", isOn: $isOn)
    }
}
```

**@StateObject** - Source of truth for ObservableObject (view creates it):
```swift
struct ProfileView: View {
    @StateObject private var viewModel = ProfileViewModel()  // View owns
}
```

**@ObservedObject** - Reference to ObservableObject (parent owns it):
```swift
struct DetailView: View {
    @ObservedObject var viewModel: DetailViewModel  // Parent owns
}
```

**Summary:**
- Use `@State` for simple value types
- Use `@Binding` for two-way connection to parent's @State
- Use `@StateObject` when view creates the object
- Use `@ObservedObject` when object passed from parent

### Q2: How does data flow work in SwiftUI?

**Answer:**

**Data flows down, events flow up.**

**1. Parent to Child (Down):**
```swift
struct ParentView: View {
    @State private var name = "John"
    
    var body: some View {
        ChildView(name: name)  // Pass down
    }
}

struct ChildView: View {
    let name: String  // Receives from parent
    var body: some View {
        Text(name)
    }
}
```

**2. Child to Parent (Up via Binding):**
```swift
struct ParentView: View {
    @State private var name = ""
    
    var body: some View {
        ChildView(name: $name)  // Pass binding
    }
}

struct ChildView: View {
    @Binding var name: String
    var body: some View {
        TextField("Name", text: $name)  // Updates parent
    }
}
```

**3. Across App (Environment):**
```swift
@main
struct MyApp: App {
    @StateObject private var appState = AppState()
    
    var body: some Scene {
        WindowGroup {
            ContentView()
                .environmentObject(appState)  // Share globally
        }
    }
}
```

**Best Practices:**
- Keep state as close to where it's used as possible
- Use @StateObject for data that view creates
- Use @ObservedObject for injected dependencies
- Use @EnvironmentObject sparingly (true global state)

### Q3: What are ViewModifiers and how do you create custom ones?

**Answer:**

ViewModifiers transform views without changing the original view.

**Built-in Modifiers:**
```swift
Text("Hello")
    .font(.title)
    .foregroundColor(.blue)
    .padding()
```

**Custom ViewModifier:**
```swift
struct PrimaryButtonStyle: ViewModifier {
    func body(content: Content) -> some View {
        content
            .font(.headline)
            .foregroundColor(.white)
            .padding()
            .background(Color.blue)
            .cornerRadius(10)
    }
}

extension View {
    func primaryButtonStyle() -> some View {
        modifier(PrimaryButtonStyle())
    }
}

// Usage
Button("Submit") {
    submit()
}
.primaryButtonStyle()
```

**Conditional Modifiers:**
```swift
extension View {
    @ViewBuilder
    func conditional<Content: View>(
        _ condition: Bool,
        transform: (Self) -> Content
    ) -> some View {
        if condition {
            transform(self)
        } else {
            self
        }
    }
}

// Usage
Text("Hello")
    .conditional(isPremium) { view in
        view.foregroundColor(.gold)
    }
```

**Benefits:**
- Code reuse
- Consistent styling
- Easy to test
- Composable

### Q4: How do you handle navigation in SwiftUI?

**Answer:**

**1. NavigationStack (iOS 16+):**
```swift
struct HomeView: View {
    @State private var path = NavigationPath()
    
    var body: some View {
        NavigationStack(path: $path) {
            List {
                NavigationLink("Detail", value: "detail")
            }
            .navigationDestination(for: String.self) { value in
                DetailView(text: value)
            }
        }
    }
}
```

**2. NavigationLink (Simple):**
```swift
NavigationView {
    List {
        NavigationLink("Go to Detail") {
            DetailView()
        }
    }
}
```

**3. Programmatic Navigation:**
```swift
@State private var isActive = false

NavigationLink(isActive: $isActive) {
    DetailView()
} label: {
    EmptyView()
}

Button("Navigate") {
    isActive = true
}
```

**4. Sheet (Modal):**
```swift
@State private var showingSheet = false

Button("Show Sheet") {
    showingSheet = true
}
.sheet(isPresented: $showingSheet) {
    DetailView()
}
```

**5. FullScreenCover:**
```swift
.fullScreenCover(isPresented: $showingFullScreen) {
    FullScreenView()
}
```

### Q5: How do you optimize SwiftUI performance?

**Answer:**

**1. Use LazyVStack/LazyHStack:**
```swift
// BAD - Creates all views upfront
ScrollView {
    VStack {
        ForEach(1...1000, id: \.self) { item in
            ItemView(item: item)
        }
    }
}

// GOOD - Creates views on demand
ScrollView {
    LazyVStack {
        ForEach(1...1000, id: \.self) { item in
            ItemView(item: item)
        }
    }
}
```

**2. Minimize State:**
```swift
// BAD - Too much state
@State private var firstName = ""
@State private var lastName = ""
@State private var fullName = ""

// GOOD - Computed property
var fullName: String {
    "\(firstName) \(lastName)"
}
```

**3. Use Equatable:**
```swift
struct UserView: View, Equatable {
    let user: User
    
    var body: some View {
        Text(user.name)
    }
    
    static func == (lhs: UserView, rhs: UserView) -> Bool {
        lhs.user.id == rhs.user.id
    }
}
```

**4. Extract Subviews:**
```swift
// BAD - Everything in one view
var body: some View {
    VStack {
        // 100 lines of code
    }
}

// GOOD - Extracted subviews
var body: some View {
    VStack {
        HeaderView()
        ContentView()
        FooterView()
    }
}
```

**5. Avoid Creating Views in Body:**
```swift
// BAD
var body: some View {
    VStack {
        createButton()  // Creates new view on every render
    }
}

// GOOD
var submitButton: some View {
    Button("Submit") { }
}

var body: some View {
    VStack {
        submitButton
    }
}
```

### Q6: How do you integrate UIKit components in SwiftUI?

**Answer:**

Use `UIViewRepresentable` for UIKit views:

```swift
struct MapView: UIViewRepresentable {
    @Binding var region: MKCoordinateRegion
    
    func makeUIView(context: Context) -> MKMapView {
        let mapView = MKMapView()
        mapView.delegate = context.coordinator
        return mapView
    }
    
    func updateUIView(_ uiView: MKMapView, context: Context) {
        uiView.setRegion(region, animated: true)
    }
    
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }
    
    class Coordinator: NSObject, MKMapViewDelegate {
        var parent: MapView
        
        init(_ parent: MapView) {
            self.parent = parent
        }
        
        func mapView(_ mapView: MKMapView, didUpdate userLocation: MKUserLocation) {
            // Handle updates
        }
    }
}
```

**UIViewController:**
```swift
struct VideoPlayerView: UIViewControllerRepresentable {
    let url: URL
    
    func makeUIViewController(context: Context) -> AVPlayerViewController {
        let controller = AVPlayerViewController()
        controller.player = AVPlayer(url: url)
        return controller
    }
    
    func updateUIViewController(_ uiViewController: AVPlayerViewController, context: Context) {
        // Update if needed
    }
}
```

### Q7: What's the difference between @ViewBuilder and @resultBuilder?

**Answer:**

**@ViewBuilder** is a result builder specifically for SwiftUI views.

**Without @ViewBuilder:**
```swift
func makeView() -> some View {
    Text("Hello")  // Can only return one view
}
```

**With @ViewBuilder:**
```swift
@ViewBuilder
func makeView() -> some View {
    Text("Hello")
    Text("World")  // Multiple views allowed
    if condition {
        Text("Conditional")
    }
}
```

**In Custom Views:**
```swift
struct Card<Content: View>: View {
    let content: Content
    
    init(@ViewBuilder content: () -> Content) {
        self.content = content()
    }
    
    var body: some View {
        VStack {
            content
        }
        .padding()
        .background(Color.white)
        .cornerRadius(10)
    }
}

// Usage
Card {
    Text("Title")
    Text("Subtitle")
    Button("Action") { }
}
```

**@resultBuilder** is the general mechanism:
- @ViewBuilder is built on @resultBuilder
- Can create custom result builders
- Enables DSL-like syntax

---

[← Previous: UIKit Development](uikit-development.md) | [Next: Data Persistence →](data-persistence.md)

[Back to Main](../README.md)
