# Dependency Management

[← Back to Main](../README.md) | [Previous: Architecture & Design Patterns](architecture-design-patterns.md) | [Next: Memory Management →](memory-management.md)

## Table of Contents
- [Swift Package Manager](#swift-package-manager)
- [CocoaPods](#cocoapods)
- [Carthage](#carthage)
- [Comparison & Best Practices](#comparison--best-practices)

---

## Swift Package Manager

Apple's official dependency manager.

### Package.swift Example

```swift
// swift-tools-version:5.5
import PackageDescription

let package = Package(
    name: "MyLibrary",
    platforms: [
        .iOS(.v13),
        .macOS(.v10_15)
    ],
    products: [
        .library(
            name: "MyLibrary",
            targets: ["MyLibrary"]),
    ],
    dependencies: [
        .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.6.0"),
    ],
    targets: [
        .target(
            name: "MyLibrary",
            dependencies: ["Alamofire"]),
        .testTarget(
            name: "MyLibraryTests",
            dependencies: ["MyLibrary"]),
    ]
)
```

### Adding SPM Dependency

In Xcode:
1. File → Add Packages
2. Enter repository URL
3. Select version rules
4. Add to target

### Creating SPM Package

```bash
mkdir MyPackage
cd MyPackage
swift package init --type library
```

---

## CocoaPods

Ruby-based dependency manager.

### Podfile

```ruby
# Podfile
platform :ios, '13.0'
use_frameworks!

target 'MyApp' do
  pod 'Alamofire', '~> 5.6'
  pod 'SDWebImage', '~> 5.0'
  pod 'SnapKit', '~> 5.0'
  
  target 'MyAppTests' do
    inherit! :search_paths
    pod 'Quick'
    pod 'Nimble'
  end
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '13.0'
    end
  end
end
```

### Commands

```bash
# Install CocoaPods
sudo gem install cocoapods

# Install dependencies
pod install

# Update dependencies
pod update

# Search for pod
pod search Alamofire
```

---

## Carthage

Decentralized dependency manager.

### Cartfile

```
github "Alamofire/Alamofire" ~> 5.6
github "onevcat/Kingfisher" ~> 7.0
```

### Commands

```bash
# Install Carthage
brew install carthage

# Build dependencies
carthage update --platform iOS

# Build specific framework
carthage update Alamofire --platform iOS
```

---

## Comparison & Best Practices

| Feature | SPM | CocoaPods | Carthage |
|---------|-----|-----------|----------|
| **Official** | Yes | No | No |
| **Integration** | Xcode native | Workspace | Manual |
| **Build Time** | Fast | Slower | Fast |
| **Complexity** | Low | Medium | Medium |
| **Adoption** | Growing | High | Declining |

### Best Practices

- Use SPM for new projects
- Document all dependencies
- Pin versions for stability
- Regular security audits
- Minimize dependencies

---

[← Previous: Architecture & Design Patterns](architecture-design-patterns.md) | [Next: Memory Management →](memory-management.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between Swift Package Manager, CocoaPods, and Carthage?

**Answer:**

**Swift Package Manager (SPM):**
- ✅ Apple's official tool
- ✅ Native Xcode integration
- ✅ Fast, no extra files
- ✅ Growing ecosystem
- ❌ Relatively newer

**CocoaPods:**
- ✅ Largest ecosystem
- ✅ Mature and stable
- ✅ Easy to use
- ❌ Generates workspace
- ❌ Slower build times
- ❌ Ruby dependency

**Carthage:**
- ✅ Decentralized
- ✅ No project modification
- ✅ Builds frameworks
- ❌ Manual integration
- ❌ Declining adoption
- ❌ No dependency resolution

**Recommendation:** Use SPM for new projects, CocoaPods if need specific libraries unavailable in SPM.

### Q2: How do you add a Swift Package to an iOS project?

**Answer:**

**Method 1: Xcode UI**
1. File → Add Packages
2. Enter repository URL (e.g., `https://github.com/Alamofire/Alamofire.git`)
3. Select version rules:
   - Exact version: `5.6.0`
   - Up to next major: `5.6.0 < 6.0.0`
   - Branch: `main`
4. Click Add Package
5. Select target

**Method 2: Package.swift Dependencies**

```swift
dependencies: [
    .package(url: "https://github.com/Alamofire/Alamofire.git", from: "5.6.0"),
]
```

**Version Rules:**
```swift
.package(url: "...", from: "1.0.0")              // 1.0.0 <= version < 2.0.0
.package(url: "...", .upToNextMajor(from: "1.0.0"))  // Same as above
.package(url: "...", .upToNextMinor(from: "1.0.0"))  // 1.0.0 <= version < 1.1.0
.package(url: "...", exact: "1.0.0")              // Exactly 1.0.0
.package(url: "...", branch: "develop")           // Specific branch
.package(url: "...", revision: "abc123")          // Specific commit
```

### Q3: How do you handle dependency conflicts?

**Answer:**

**Conflict Types:**

**1. Version Conflict:**
```
App depends on:
  - PackageA 1.0 (requires PackageC 1.0)
  - PackageB 2.0 (requires PackageC 2.0)
```

**Solutions:**
- Update dependencies to compatible versions
- Use version ranges flexibly
- Contact package maintainers
- Fork and modify if necessary

**2. Duplicate Symbols:**
Two packages define same symbols

**Solutions:**
```swift
// Use module naming
import PackageA
import PackageB

let valueA = PackageA.SharedClass()
let valueB = PackageB.SharedClass()
```

**Best Practices:**
- Keep dependencies updated
- Use semantic versioning
- Minimize dependencies
- Audit dependencies regularly

### Q4: What's the difference between static and dynamic frameworks?

**Answer:**

**Static Framework:**
- Compiled into app binary
- No separate .framework at runtime
- Faster launch time
- Larger app size
- Can't share code between app and extensions

**Dynamic Framework:**
- Loaded at runtime
- Separate .framework file
- Slower launch time (minimal)
- Smaller app size
- Can share between app and extensions

**Example:**

```swift
// Static linking
-static-library flag

// Dynamic linking
- Embedded Binaries

```

**When to Use:**
- **Static**: Third-party libraries, single target
- **Dynamic**: Your own frameworks, shared code, extensions

### Q5: How do you create a private Swift Package?

**Answer:**

**Option 1: Private GitHub Repository**

```swift
dependencies: [
    .package(url: "https://github.com/company/PrivatePackage.git", from: "1.0.0")
]
```

**Xcode:** Configure GitHub authentication in Xcode preferences.

**Option 2: Local Package**

```swift
dependencies: [
    .package(path: "../LocalPackage")
]
```

**Option 3: Internal Package Registry**

Configure in Package.swift:
```swift
dependencies: [
    .package(url: "https://packages.company.com/MyPackage.git", from: "1.0.0")
]
```

**Q6: What are XCFrameworks and why use them?**

**Answer:**

**XCFramework:** Bundle that contains variants for multiple platforms/architectures.

**Structure:**
```
MyFramework.xcframework/
  - ios-arm64/
  - ios-arm64-simulator/
  - ios-x86_64-simulator/
```

**Benefits:**
- Single distribution for all architectures
- Supports iOS devices and simulators
- Supports Mac Catalyst
- No need for lipo or stripping

**Creating XCFramework:**

```bash
# Build for device
xcodebuild archive \
  -scheme MyFramework \
  -destination "generic/platform=iOS" \
  -archivePath "build/ios" \
  SKIP_INSTALL=NO

# Build for simulator
xcodebuild archive \
  -scheme MyFramework \
  -destination "generic/platform=iOS Simulator" \
  -archivePath "build/ios-simulator" \
  SKIP_INSTALL=NO

# Create XCFramework
xcodebuild -create-xcframework \
  -framework build/ios.xcarchive/Products/Library/Frameworks/MyFramework.framework \
  -framework build/ios-simulator.xcarchive/Products/Library/Frameworks/MyFramework.framework \
  -output MyFramework.xcframework
```

**Use Case:**
- Distributing binary frameworks
- Supporting multiple platforms
- Closed-source libraries

### Q7: How do you manage dependencies in a large team?

**Answer:**

**1. Lock Files:**
- **SPM:** Package.resolved
- **CocoaPods:** Podfile.lock
- **Commit to version control** for reproducible builds

**2. Dependency Approval Process:**
- Security audit
- License check
- Performance impact
- Maintenance status
- Team vote

**3. Internal Package Registry:**
```swift
// Custom registry for approved packages
dependencies: [
    .package(url: "https://internal.company.com/ApprovedPackages.git", from: "1.0.0")
]
```

**4. Dependency Documentation:**
```markdown
# Dependencies.md
- Alamofire 5.6.0 - Networking
  - Reason: Robust HTTP client
  - Maintainer: @john
  - Alternatives considered: URLSession, Moya
```

**5. Regular Audits:**
- Check for updates monthly
- Security vulnerability scans
- Remove unused dependencies
- Update major versions yearly

**6. Modularization:**
- Break app into modules
- Each module has own dependencies
- Easier to manage and test

---

[← Previous: Architecture & Design Patterns](architecture-design-patterns.md) | [Next: Memory Management →](memory-management.md)

[Back to Main](../README.md)
