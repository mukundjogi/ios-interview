# App Distribution & In-App Purchase

[← Back to Main](../README.md) | [Previous: Security](security.md)

## Table of Contents
- [Certificates & Code Signing](#certificates--code-signing)
- [Provisioning Profiles](#provisioning-profiles)
- [App Deployment Process](#app-deployment-process)
- [In-App Purchase](#in-app-purchase)
- [Push Notification Certificates](#push-notification-certificates)

---

## Certificates & Code Signing

Code signing ensures that your app comes from a trusted source and hasn't been modified.

### Types of Certificates

**1. Development Certificate**
- Used for testing on physical devices
- Each developer needs their own
- Valid for 1 year

**2. Distribution Certificate**
- Used for App Store distribution
- Used for Ad-Hoc and Enterprise distribution
- Valid for 1 year
- Limit: 3 per account

### Creating Certificates on developer.apple.com

#### Step 1: Generate Certificate Signing Request (CSR)

On Mac, open **Keychain Access**:

1. Go to **Keychain Access** → **Certificate Assistant** → **Request a Certificate from a Certificate Authority**
2. Enter your email and name
3. Choose "Saved to disk"
4. This creates a `.certSigningRequest` file

#### Step 2: Create Certificate on Apple Developer Portal

1. Go to [developer.apple.com](https://developer.apple.com)
2. Navigate to **Certificates, Identifiers & Profiles**
3. Click **Certificates** → **+** (Create)
4. Select certificate type:
   - **iOS App Development** (for development)
   - **Apple Distribution** (for App Store)
5. Upload the CSR file
6. Download the certificate (`.cer` file)
7. Double-click to install in Keychain

### Creating P12 File

P12 files contain your certificate and private key, used for CI/CD or sharing with team.

**Steps:**

1. Open **Keychain Access**
2. Find your certificate under "My Certificates"
3. Expand the certificate to see the private key
4. Select both certificate and private key
5. Right-click → **Export 2 items**
6. Save as `.p12` file
7. Set a password (required)

**Command Line Method:**

```bash
# Export certificate and private key to p12
security export -k ~/Library/Keychains/login.keychain-db \
  -t identities \
  -f pkcs12 \
  -o certificate.p12 \
  -P "your_password"
```

**Use Cases:**
- CI/CD pipelines (Fastlane, Jenkins)
- Sharing with team members
- Backup purposes

### Creating P8 File (APNs Auth Key)

P8 files are newer and more secure for push notifications. They don't expire!

**Steps:**

1. Go to **developer.apple.com** → **Certificates, Identifiers & Profiles**
2. Click **Keys** → **+** (Create)
3. Enter key name
4. Check **Apple Push Notifications service (APNs)**
5. Click **Continue** → **Register**
6. Download the `.p8` file (can only download once!)
7. Note the **Key ID** and **Team ID**

**Important:**
- Download immediately (can't download again)
- Store securely (it's like a password)
- One key can be used for all apps
- Never expires (unlike certificates)

**Using P8 in Code:**

```swift
import UserNotifications

// P8 configuration
struct APNsConfig {
    let keyID = "ABC123XYZ"  // From developer portal
    let teamID = "DEF456ABC"  // Your team ID
    let bundleID = "com.yourapp.example"
    
    // Path to p8 file
    let p8FilePath = Bundle.main.path(forResource: "AuthKey_ABC123XYZ", ofType: "p8")
}
```

### Code Signing in Xcode

**Automatic Signing:**

1. Select your project in Xcode
2. Go to **Signing & Capabilities**
3. Check **Automatically manage signing**
4. Select your team
5. Xcode handles everything automatically

**Manual Signing:**

1. Uncheck **Automatically manage signing**
2. Select provisioning profile manually
3. Choose signing certificate
4. Used for more control in large teams

```swift
// Build Settings for Manual Signing
CODE_SIGN_IDENTITY = "Apple Distribution: Your Name (TEAMID)"
PROVISIONING_PROFILE_SPECIFIER = "YourApp AppStore"
DEVELOPMENT_TEAM = "TEAMID"
```

### Troubleshooting Common Issues

**Problem: "No signing certificate found"**

Solution:
```bash
# List available certificates
security find-identity -v -p codesigning

# Import certificate
security import certificate.p12 -k ~/Library/Keychains/login.keychain-db -P password
```

**Problem: "Profile doesn't include signing certificate"**

Solution:
- Regenerate provisioning profile
- Ensure certificate is added to profile
- Download and install updated profile

---

## Provisioning Profiles

Provisioning profiles link your app, certificate, and devices together.

### Types of Provisioning Profiles

**1. Development Profile**
- For testing on registered devices
- Contains: App ID, Development certificate, Device UDIDs
- Used during development

**2. Ad-Hoc Profile**
- For distributing to specific devices (up to 100)
- Contains: App ID, Distribution certificate, Device UDIDs
- Used for beta testing outside TestFlight

**3. App Store Profile**
- For App Store distribution
- Contains: App ID, Distribution certificate
- No device limitation

**4. Enterprise Profile**
- For in-house distribution
- Requires Enterprise account ($299/year)
- No device limitation

### Creating Provisioning Profile

**Step 1: Register App ID**

1. Go to **Identifiers** → **+** (Create)
2. Select **App IDs** → **App**
3. Enter Bundle ID (e.g., `com.company.appname`)
4. Select capabilities needed:
   - Push Notifications
   - In-App Purchase
   - iCloud
   - etc.
5. Click **Continue** → **Register**

**Step 2: Register Devices (for Development/Ad-Hoc)**

1. Go to **Devices** → **+**
2. Enter device name and UDID
3. Click **Continue**

**Finding Device UDID:**

```bash
# Connect device and run:
instruments -s devices

# Or in Xcode:
# Window → Devices and Simulators → Select device → Copy UDID
```

**Step 3: Create Provisioning Profile**

1. Go to **Profiles** → **+** (Create)
2. Select type (Development, Ad-Hoc, App Store)
3. Select App ID
4. Select certificates
5. Select devices (if Development/Ad-Hoc)
6. Enter profile name
7. Download `.mobileprovision` file

**Installing Profile:**

```bash
# Manual install
# Double-click .mobileprovision file
# Or copy to:
cp MyApp.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/

# List installed profiles
ls ~/Library/MobileDevice/Provisioning\ Profiles/
```

### Profile Management in Xcode

**Automatic:**
- Xcode downloads and installs automatically
- Managed in Xcode preferences

**Manual:**
1. Project settings → **Build Settings**
2. Search for "Provisioning Profile"
3. Set `PROVISIONING_PROFILE_SPECIFIER`

### Profile Validation

```swift
// Check profile expiration
// Build Settings → Other Code Signing Flags
// Add: --verify

// Verify installed profiles
security cms -D -i ~/Library/MobileDevice/Provisioning\ Profiles/profile.mobileprovision
```

---

## App Deployment Process

Complete guide to deploying your app to the App Store.

### Pre-Deployment Checklist

**1. App Requirements:**
- ✅ Unique Bundle ID
- ✅ App icons (all sizes)
- ✅ Launch screen
- ✅ Privacy policy (if collecting data)
- ✅ Support URL
- ✅ Copyright information

**2. Build Configuration:**
- ✅ Release build configuration
- ✅ Correct provisioning profile
- ✅ Version and build number
- ✅ Supported devices and iOS versions

**3. Testing:**
- ✅ No crashes
- ✅ Works on all supported devices
- ✅ Passes App Store guidelines
- ✅ Privacy permissions properly requested

### Step 1: Configure App in App Store Connect

1. Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com)
2. Click **My Apps** → **+** → **New App**
3. Fill in details:
   - Platform (iOS)
   - App Name
   - Primary Language
   - Bundle ID
   - SKU (unique identifier)
4. Click **Create**

### Step 2: Prepare App Information

**Required Information:**

```
App Information:
├── Name (30 characters max)
├── Subtitle (30 characters max)
├── Category (Primary & Secondary)
├── Content Rights
└── Age Rating

App Privacy:
├── Privacy Policy URL
└── Data Collection Details

Pricing and Availability:
├── Price Tier
├── Availability Date
└── Countries/Regions

Version Information:
├── Screenshots (all required sizes)
├── Description (4000 characters max)
├── Keywords (100 characters max)
├── What's New (4000 characters max)
└── Promotional Text (170 characters max)
```

**Screenshot Sizes Required:**

```
iPhone:
- 6.7" (1290 x 2796) - iPhone 14 Pro Max
- 6.5" (1284 x 2778) - iPhone 14 Plus
- 5.5" (1242 x 2208) - iPhone 8 Plus

iPad:
- 12.9" (2048 x 2732) - iPad Pro
- 11" (1668 x 2388) - iPad Pro 11"
```

### Step 3: Archive and Upload

**Archive in Xcode:**

1. Select **Any iOS Device** or **Generic iOS Device**
2. **Product** → **Archive**
3. Wait for archive to complete
4. Archive Organizer opens automatically

**Upload to App Store Connect:**

1. In Organizer, select your archive
2. Click **Distribute App**
3. Select **App Store Connect**
4. Click **Upload**
5. Select signing options:
   - **Automatically manage signing** (recommended)
   - Or select manual certificate and profile
6. Click **Upload**
7. Wait for processing (10-60 minutes)

**Using Command Line (Fastlane):**

```ruby
# Fastfile
lane :release do
  # Build app
  gym(
    scheme: "YourApp",
    export_method: "app-store"
  )
  
  # Upload to App Store Connect
  deliver(
    skip_metadata: true,
    skip_screenshots: true
  )
end
```

### Step 4: Submit for Review

1. Go to App Store Connect
2. Select your app
3. Click on version
4. Add build (select uploaded build)
5. Fill all required information
6. Answer compliance questions:
   - Export Compliance
   - Content Rights
   - Advertising Identifier
7. Click **Submit for Review**

### Step 5: TestFlight Beta Testing

**Internal Testing:**

1. Add internal testers in App Store Connect
2. Testers receive email invitation
3. Install TestFlight app
4. Test the build
5. Provide feedback

**External Testing:**

1. Add external testers (up to 10,000)
2. Requires beta app review (1-2 days)
3. Share public link or invite by email
4. Collect feedback

**TestFlight Code:**

```swift
// Check if running in TestFlight
#if DEBUG
let isTestFlight = false
#else
let isTestFlight = Bundle.main.appStoreReceiptURL?.lastPathComponent == "sandboxReceipt"
#endif

if isTestFlight {
    print("Running in TestFlight")
}
```

### App Review Process

**Typical Timeline:**
- Submission → In Review: 24-48 hours
- Review: 1-3 days
- Total: 2-5 days (can vary)

**Review Status:**

```
Waiting for Review → In Review → Processing → Ready for Sale
                         ↓
                    Rejected (fix and resubmit)
```

**Common Rejection Reasons:**

1. Crashes and bugs
2. Broken links or features
3. Incomplete information
4. Privacy policy missing
5. Using private APIs
6. Poor user experience
7. Misleading functionality

### Updating Your App

**Version Update:**

1. Increment version number (e.g., 1.0 → 1.1)
2. Create new version in App Store Connect
3. Upload new build
4. Add "What's New" description
5. Submit for review

**Build Update (Same Version):**

1. Increment build number only
2. Upload new build
3. Select new build in App Store Connect
4. No new review needed (for same version in review)

---

## In-App Purchase

Monetize your app with In-App Purchases (IAP).

### Types of In-App Purchases

**1. Consumable**
- Can be purchased multiple times
- Examples: Game coins, extra lives, power-ups
- Not restored on new device

**2. Non-Consumable**
- Purchased once, available forever
- Examples: Remove ads, unlock features, premium content
- Restored on new device

**3. Auto-Renewable Subscription**
- Automatically renews until canceled
- Examples: Monthly subscription, annual membership
- Durations: 1 week, 1 month, 2 months, 3 months, 6 months, 1 year

**4. Non-Renewing Subscription**
- Fixed duration, doesn't auto-renew
- Examples: Season pass, limited-time access
- You manage the subscription logic

### Setting Up IAP in App Store Connect

**Step 1: Configure IAP Capability**

In Xcode:
1. Select target → **Signing & Capabilities**
2. Click **+ Capability**
3. Add **In-App Purchase**

**Step 2: Create IAP Products**

1. Go to App Store Connect
2. Select your app
3. Go to **Features** → **In-App Purchases**
4. Click **+** to create
5. Select type (Consumable, Non-Consumable, etc.)
6. Fill in details:
   - Reference Name
   - Product ID (e.g., `com.app.coins100`)
   - Price Tier
   - Localization (name and description)
7. Click **Save**

**Product IDs Example:**

```
Consumables:
- com.myapp.coins.100
- com.myapp.coins.500
- com.myapp.coins.1000

Non-Consumables:
- com.myapp.premium
- com.myapp.removeads

Subscriptions:
- com.myapp.subscription.monthly
- com.myapp.subscription.yearly
```

### Implementing IAP in Swift

**Step 1: Import StoreKit**

```swift
import StoreKit

class IAPManager: NSObject, SKProductsRequestDelegate, SKPaymentTransactionObserver {
    static let shared = IAPManager()
    
    private override init() {
        super.init()
        SKPaymentQueue.default().add(self)
    }
    
    deinit {
        SKPaymentQueue.default().remove(self)
    }
    
    // Product identifiers
    enum ProductID: String {
        case coins100 = "com.myapp.coins.100"
        case coins500 = "com.myapp.coins.500"
        case premium = "com.myapp.premium"
        case subscriptionMonthly = "com.myapp.subscription.monthly"
    }
}
```

**Step 2: Fetch Products**

```swift
extension IAPManager {
    func fetchProducts(completion: @escaping ([SKProduct]) -> Void) {
        let productIDs: Set<String> = [
            ProductID.coins100.rawValue,
            ProductID.coins500.rawValue,
            ProductID.premium.rawValue
        ]
        
        let request = SKProductsRequest(productIdentifiers: productIDs)
        request.delegate = self
        request.start()
        
        // Store completion for later use
        self.productsCompletion = completion
    }
    
    private var productsCompletion: (([SKProduct]) -> Void)?
    
    // SKProductsRequestDelegate
    func productsRequest(_ request: SKProductsRequest, didReceive response: SKProductsResponse) {
        let products = response.products
        productsCompletion?(products)
        
        // Invalid products
        for invalidID in response.invalidProductIdentifiers {
            print("Invalid product ID: \(invalidID)")
        }
    }
    
    func request(_ request: SKRequest, didFailWithError error: Error) {
        print("Product request failed: \(error.localizedDescription)")
        productsCompletion?([])
    }
}
```

**Step 3: Purchase Product**

```swift
extension IAPManager {
    func purchase(product: SKProduct) {
        guard SKPaymentQueue.canMakePayments() else {
            print("User cannot make payments")
            return
        }
        
        let payment = SKPayment(product: product)
        SKPaymentQueue.default().add(payment)
    }
    
    // SKPaymentTransactionObserver
    func paymentQueue(_ queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {
        for transaction in transactions {
            switch transaction.transactionState {
            case .purchasing:
                print("Purchasing...")
                
            case .purchased:
                print("Purchase successful!")
                completeTransaction(transaction)
                
            case .failed:
                print("Purchase failed: \(transaction.error?.localizedDescription ?? "")")
                SKPaymentQueue.default().finishTransaction(transaction)
                
            case .restored:
                print("Purchase restored")
                completeTransaction(transaction)
                
            case .deferred:
                print("Purchase deferred (awaiting approval)")
                
            @unknown default:
                break
            }
        }
    }
    
    private func completeTransaction(_ transaction: SKPaymentTransaction) {
        // Deliver content to user
        deliverContent(for: transaction.payment.productIdentifier)
        
        // Finish transaction
        SKPaymentQueue.default().finishTransaction(transaction)
    }
    
    private func deliverContent(for productID: String) {
        switch productID {
        case ProductID.coins100.rawValue:
            // Add 100 coins to user account
            UserDefaults.standard.set(UserDefaults.standard.integer(forKey: "coins") + 100, forKey: "coins")
            
        case ProductID.premium.rawValue:
            // Unlock premium features
            UserDefaults.standard.set(true, forKey: "isPremium")
            
        default:
            break
        }
        
        // Post notification
        NotificationCenter.default.post(name: .purchaseCompleted, object: productID)
    }
}

extension Notification.Name {
    static let purchaseCompleted = Notification.Name("purchaseCompleted")
}
```

**Step 4: Restore Purchases**

```swift
extension IAPManager {
    func restorePurchases() {
        SKPaymentQueue.default().restoreCompletedTransactions()
    }
    
    func paymentQueueRestoreCompletedTransactionsFinished(_ queue: SKPaymentQueue) {
        print("Restore completed")
        // Handle restored purchases
    }
    
    func paymentQueue(_ queue: SKPaymentQueue, restoreCompletedTransactionsFailedWithError error: Error) {
        print("Restore failed: \(error.localizedDescription)")
    }
}
```

**Step 5: UI Implementation**

```swift
class StoreViewController: UIViewController {
    private var products: [SKProduct] = []
    
    override func viewDidLoad() {
        super.viewDidLoad()
        loadProducts()
    }
    
    private func loadProducts() {
        IAPManager.shared.fetchProducts { [weak self] products in
            self?.products = products
            self?.displayProducts()
        }
    }
    
    private func displayProducts() {
        for product in products {
            print("Product: \(product.localizedTitle)")
            print("Price: \(formatPrice(product))")
        }
    }
    
    private func formatPrice(_ product: SKProduct) -> String {
        let formatter = NumberFormatter()
        formatter.numberStyle = .currency
        formatter.locale = product.priceLocale
        return formatter.string(from: product.price) ?? ""
    }
    
    @objc private func purchaseButtonTapped(_ sender: UIButton) {
        let product = products[sender.tag]
        IAPManager.shared.purchase(product: product)
    }
    
    @objc private func restoreButtonTapped() {
        IAPManager.shared.restorePurchases()
    }
}
```

### Subscription Implementation

```swift
// Check subscription status
func checkSubscriptionStatus(productID: String) -> Bool {
    // Verify receipt with server
    // Check expiration date
    // Return subscription status
    return UserDefaults.standard.bool(forKey: "hasActiveSubscription")
}

// Auto-renewable subscription handling
func handleSubscriptionPurchase(_ transaction: SKPaymentTransaction) {
    // Validate receipt
    validateReceipt(transaction.transactionReceipt) { isValid, expirationDate in
        if isValid {
            // Grant subscription access
            UserDefaults.standard.set(true, forKey: "hasActiveSubscription")
            UserDefaults.standard.set(expirationDate, forKey: "subscriptionExpiry")
        }
    }
}

func validateReceipt(_ receipt: Data?, completion: @escaping (Bool, Date?) -> Void) {
    // Send receipt to your server for validation
    // Server verifies with Apple's receipt validation service
    // Returns subscription status and expiration date
}
```

### Testing IAP

**Sandbox Testing:**

1. Create sandbox test account:
   - App Store Connect → **Users and Access** → **Sandbox Testers**
   - Click **+** to add tester
   - Use fake email (doesn't need to exist)

2. Sign out of real App Store on device:
   - Settings → App Store → Sign Out

3. Run app and make purchase
   - Sign in with sandbox account when prompted
   - Purchases are free in sandbox

**Testing Scenarios:**

```swift
// Test different scenarios
1. Successful purchase
2. Cancelled purchase
3. No internet connection
4. Restore purchases
5. Multiple purchases
6. Invalid product ID
```

### Receipt Validation

**Local Validation (Basic):**

```swift
func verifyReceipt() -> Bool {
    guard let receiptURL = Bundle.main.appStoreReceiptURL,
          let receiptData = try? Data(contentsOf: receiptURL) else {
        return false
    }
    
    // Receipt exists
    return true
}
```

**Server Validation (Recommended):**

```swift
func validateReceiptWithServer(completion: @escaping (Bool) -> Void) {
    guard let receiptURL = Bundle.main.appStoreReceiptURL,
          let receiptData = try? Data(contentsOf: receiptURL) else {
        completion(false)
        return
    }
    
    let receiptString = receiptData.base64EncodedString()
    
    // Send to your server
    let parameters = ["receipt": receiptString]
    
    // Your server validates with Apple:
    // https://buy.itunes.apple.com/verifyReceipt (Production)
    // https://sandbox.itunes.apple.com/verifyReceipt (Sandbox)
    
    // Server returns validation result
}
```

---

## Push Notification Certificates

### P12 vs P8 for Push Notifications

**P12 (Certificate-based):**
- Traditional method
- Expires after 1 year
- Separate certificates for development and production
- Need to renew annually

**P8 (Token-based) - Recommended:**
- Modern approach
- Never expires
- One key for all apps
- Simpler to manage
- More secure

### Using P8 for Push Notifications

**Server-side Implementation:**

```python
# Python example using p8 file
import jwt
import time

def generate_apns_token(key_id, team_id, p8_file_path):
    with open(p8_file_path, 'r') as f:
        key = f.read()
    
    headers = {
        'alg': 'ES256',
        'kid': key_id
    }
    
    payload = {
        'iss': team_id,
        'iat': time.time()
    }
    
    token = jwt.encode(payload, key, algorithm='ES256', headers=headers)
    return token

# Send push notification
token = generate_apns_token('ABC123', 'TEAMID', 'AuthKey_ABC123.p8')
```

**Using P12:**

```bash
# Convert .cer to .pem
openssl x509 -in aps_development.cer -inform der -out cert.pem

# Convert .p12 to .pem
openssl pkcs12 -in cert.p12 -out key.pem -nodes -clcerts

# Combine for APNs
cat cert.pem key.pem > apns.pem
```

### Interview Questions

**Q: What's the difference between development and distribution certificates?**

**A:** Development certificates are used for testing on physical devices during development. Distribution certificates are used for App Store submission, Ad-Hoc distribution, and Enterprise distribution.

**Q: Why use P8 instead of P12 for push notifications?**

**A:** P8 (token-based) is recommended because:
- Never expires (no yearly renewal)
- One key works for all apps
- More secure
- Simpler management
- Supports both development and production

**Q: What are the different types of provisioning profiles?**

**A:**
- **Development:** Testing on registered devices
- **Ad-Hoc:** Distributing to up to 100 specific devices
- **App Store:** Distribution through App Store
- **Enterprise:** In-house distribution (requires Enterprise account)

**Q: Explain the IAP types and when to use each**

**A:**
- **Consumable:** Items used once (coins, lives) - not restored
- **Non-Consumable:** One-time purchase (premium features) - restored
- **Auto-Renewable Subscription:** Recurring payment (monthly subscription)
- **Non-Renewing Subscription:** Fixed duration (season pass)

---

[← Previous: Security](security.md)

[Back to Main](../README.md)


## Interview Questions & Answers

### Q1: What's the difference between P12 and P8 files?

**Answer:**

**P12 File (Certificate-based):**
- Contains certificate + private key
- Expires after 1 year
- Separate for development and production
- Traditional method
- Need to renew annually

**Creating P12:**
1. Export from Keychain Access
2. Select certificate + private key
3. Export as .p12
4. Set password

**P8 File (Token-based):**
- APNs authentication key only
- Never expires
- One key for all apps
- Works for both development and production
- Download only once from developer portal

**Creating P8:**
1. developer.apple.com → Keys
2. Create key with APNs enabled
3. Download .p8 file
4. Note Key ID and Team ID

**Comparison:**

| Feature | P12 | P8 |
|---------|-----|-----|
| Expiration | 1 year | Never |
| Scope | One app | All apps |
| Complexity | Higher | Lower |
| Security | Good | Better |
| Recommendation | Legacy | Preferred |

**When to Use P8:**
- New projects (recommended)
- Simpler management
- Multiple apps

**When to Use P12:**
- Legacy projects
- Already set up
- Specific requirement

### Q2: Explain the complete App Store submission process

**Answer:**

**Phase 1: Preparation (1-2 days)**

1. **Create App in App Store Connect:**
   - Bundle ID
   - App name
   - SKU
   - Category

2. **Prepare Assets:**
   - App icons (all sizes)
   - Screenshots (all device sizes)
   - App preview videos (optional)

3. **Write Metadata:**
   - Description (4000 chars)
   - Keywords (100 chars)
   - Support URL
   - Marketing URL
   - Privacy policy URL

**Phase 2: Archive & Upload (30 minutes)**

1. **In Xcode:**
   - Select "Any iOS Device"
   - Product → Archive
   - Wait for archive to complete

2. **Organizer:**
   - Select archive
   - Distribute App → App Store Connect
   - Upload
   - Processing: 10-60 minutes

**Phase 3: Submit for Review (1 hour)**

1. **Select Build:**
   - Go to App Store Connect
   - Select your app version
   - Choose uploaded build

2. **Answer Questions:**
   - Export compliance
   - Content rights
   - Advertising identifier usage
   - Age rating

3. **Submit:**
   - Click "Submit for Review"

**Phase 4: Review (1-3 days)**

**Status Flow:**
```
Waiting for Review → In Review → Processing → Ready for Sale
                         ↓
                    Rejected (fix and resubmit)
```

**Phase 5: Release**
- Automatic release after approval
- Or manual release (your choice)
- Phased release (gradual rollout) available

**Common Rejection Reasons:**
- Crashes and bugs
- Incomplete functionality
- Privacy issues
- Guideline violations
- Broken links
- Missing info

**Tip:** Respond quickly to rejection with fixes to maintain queue position.

### Q3: What are the different types of In-App Purchases?

**Answer:**

**1. Consumable:**
- Used once and depleted
- Can purchase multiple times
- Not restored
- Examples: Game coins, lives, power-ups

```swift
// Implementation
case .purchased:
    if productID == "com.app.coins100" {
        UserDefaults.standard.set(
            UserDefaults.standard.integer(forKey: "coins") + 100,
            forKey: "coins"
        )
    }
```

**2. Non-Consumable:**
- Purchased once
- Available forever
- Restored on new devices
- Examples: Premium features, remove ads

```swift
// Implementation
case .purchased:
    if productID == "com.app.premium" {
        UserDefaults.standard.set(true, forKey: "isPremium")
    }

// Must support restore
func restorePurchases() {
    SKPaymentQueue.default().restoreCompletedTransactions()
}
```

**3. Auto-Renewable Subscription:**
- Automatically renews until canceled
- Durations: 1 week, 1 month, 2 months, 3 months, 6 months, 1 year
- Examples: Monthly subscription, annual membership

```swift
// Check subscription status
func hasActiveSubscription() -> Bool {
    guard let expiryDate = UserDefaults.standard.object(forKey: "subscriptionExpiry") as? Date else {
        return false
    }
    return Date() < expiryDate
}
```

**4. Non-Renewing Subscription:**
- Fixed duration
- Doesn't auto-renew
- You manage expiration
- Examples: Season pass, limited-time access

**Comparison:**

| Type | Multiple Purchase | Restore | Auto-Renew |
|------|-------------------|---------|------------|
| Consumable | Yes | No | No |
| Non-Consumable | No | Yes | No |
| Auto-Renewable Sub | No | Yes | Yes |
| Non-Renewing Sub | No | Yes | No |

### Q4: How do you test In-App Purchases?

**Answer:**

**Sandbox Testing:**

**1. Create Sandbox Tester:**
- App Store Connect → Users and Access → Sandbox Testers
- Click + to add tester
- Use fake email (doesn't need to exist)
- Set country and password

**2. Prepare Device:**
- Sign out of production App Store
- Settings → App Store → Sign Out
- Don't sign in to sandbox yet

**3. Test Purchase:**
- Run app from Xcode
- Trigger purchase
- Sign in with sandbox account when prompted
- Complete purchase (free in sandbox)

**4. Testing Scenarios:**

```swift
// Test cases to cover:
1. Successful purchase
2. Canceled purchase  
3. Purchase with no internet
4. Purchase with invalid product ID
5. Restore purchases
6. Multiple quick purchases
7. Purchase during app in background
```

**StoreKit Testing (iOS 14+):**

Create StoreKit configuration file in Xcode:
- File → New → StoreKit Configuration File
- Add test products
- Run without server setup
- Faster iteration

**Subscription Testing:**
- Sandbox subscriptions renew every few minutes
- Auto-renew: 5 minutes = 1 month
- Easy to test renewal flow

**5. Verify Receipt:**
```swift
// Sandbox endpoint
let verifyURL = URL(string: "https://sandbox.itunes.apple.com/verifyReceipt")!

// Production endpoint
let verifyURL = URL(string: "https://buy.itunes.apple.com/verifyReceipt")!
```

**Best Practices:**
- Test on real device (not simulator)
- Test all purchase states
- Test restore functionality
- Test with sandbox account only

### Q5: How do you validate IAP receipts?

**Answer:**

**Why Validate:**
- Prevent fraud
- Verify purchase authenticity
- Check subscription status
- Validate on server (recommended)

**Local Validation (Basic):**

```swift
func hasReceipt() -> Bool {
    guard let receiptURL = Bundle.main.appStoreReceiptURL,
          FileManager.default.fileExists(atPath: receiptURL.path) else {
        return false
    }
    return true
}
```

**Server Validation (Recommended):**

**Client Side:**
```swift
func validateReceipt(completion: @escaping (Bool) -> Void) {
    guard let receiptURL = Bundle.main.appStoreReceiptURL,
          let receiptData = try? Data(contentsOf: receiptURL) else {
        completion(false)
        return
    }
    
    let receiptString = receiptData.base64EncodedString()
    
    // Send to your server
    let parameters = ["receipt": receiptString]
    sendToServer(parameters) { isValid in
        completion(isValid)
    }
}

func sendToServer(_ params: [String: String], completion: @escaping (Bool) -> Void) {
    // Implementation
}
```

**Server Side (Your Backend):**

```python
import requests
import json

def validate_receipt(receipt_data):
    # Try production first
    url = "https://buy.itunes.apple.com/verifyReceipt"
    payload = {
        "receipt-data": receipt_data,
        "password": "your_shared_secret"  # From App Store Connect
    }
    
    response = requests.post(url, json=payload)
    result = response.json()
    
    # If sandbox receipt, retry with sandbox endpoint
    if result.get("status") == 21007:
        url = "https://sandbox.itunes.apple.com/verifyReceipt"
        response = requests.post(url, json=payload)
        result = response.json()
    
    return result.get("status") == 0
```

**Response Fields:**
- status: 0 = valid
- receipt: Contains purchase info
- latest_receipt_info: Subscription details
- pending_renewal_info: Auto-renew status

**Best Practices:**
- Always validate on server
- Store receipt validation results
- Check expiration for subscriptions
- Handle receipt refresh

### Q6: What's the difference between development and distribution provisioning profiles?

**Answer:**

**Development Profile:**
- **Purpose:** Testing on physical devices
- **Certificate:** Development certificate
- **Devices:** Must register device UDIDs (up to 100)
- **Distribution:** Cannot submit to App Store
- **Use Case:** Development and debugging

**Distribution Profile (3 Types):**

**1. App Store Profile:**
- **Purpose:** App Store submission
- **Certificate:** Distribution certificate
- **Devices:** No device limitation
- **Use Case:** Public app distribution

**2. Ad-Hoc Profile:**
- **Purpose:** Beta testing outside TestFlight
- **Certificate:** Distribution certificate
- **Devices:** Must register UDIDs (up to 100)
- **Use Case:** Internal testing, client demos

**3. Enterprise Profile:**
- **Purpose:** Internal company distribution
- **Certificate:** Enterprise certificate ($299/year)
- **Devices:** Unlimited
- **Use Case:** Company-internal apps

**Comparison:**

| Type | Certificate | Devices | App Store | Cost |
|------|-------------|---------|-----------|------|
| Development | Dev | 100 (registered) | No | Included |
| App Store | Dist | Unlimited | Yes | Included |
| Ad-Hoc | Dist | 100 (registered) | No | Included |
| Enterprise | Enterprise | Unlimited | No | $299/year |

**Common Issues:**
- Wrong profile type for build
- Expired certificates
- Device not registered
- Bundle ID mismatch

### Q7: How do you handle subscription renewals and cancellations?

**Answer:**

**Subscription Lifecycle:**

```
New Subscription → Active → Billing Issue/Cancel → Grace Period → Expired
                    ↓
                  Renew
```

**Checking Subscription Status:**

```swift
func checkSubscriptionStatus(completion: @escaping (SubscriptionStatus) -> Void) {
    // Validate receipt with server
    validateReceipt { response in
        guard let latestReceipt = response["latest_receipt_info"] as? [[String: Any]],
              let subscription = latestReceipt.first else {
            completion(.notSubscribed)
            return
        }
        
        guard let expiresDateString = subscription["expires_date"] as? String,
              let expiresDate = ISO8601DateFormatter().date(from: expiresDateString) else {
            completion(.notSubscribed)
            return
        }
        
        if Date() < expiresDate {
            completion(.active(expiryDate: expiresDate))
        } else {
            completion(.expired)
        }
    }
}

enum SubscriptionStatus {
    case active(expiryDate: Date)
    case expired
    case notSubscribed
    case inGracePeriod
}

func validateReceipt(completion: @escaping ([String: Any]) -> Void) {
    // Server validation
}
```

**Handle Renewal:**

```swift
func paymentQueue(_ queue: SKPaymentQueue, updatedTransactions transactions: [SKPaymentTransaction]) {
    for transaction in transactions {
        switch transaction.transactionState {
        case .purchased:
            // New subscription or renewal
            validateAndUnlockContent(transaction)
            
        case .restored:
            // User restored subscription
            restoreSubscription(transaction)
            
        default:
            break
        }
    }
}
```

**Server-Side Notifications (Recommended):**

Configure server URL in App Store Connect to receive:
- DID_RENEW - Subscription renewed
- DID_FAIL_TO_RENEW - Payment failed
- DID_CANCEL - User canceled
- REFUND - Purchase refunded

**Grace Period:**
- 16-day window to fix payment issue
- User retains access
- Handle gracefully

**Best Practices:**
- Check status on app launch
- Validate on server
- Cache subscription status
- Handle edge cases (grace period, billing retry)
- Test all states in sandbox

---

[← Previous: Security](security.md)

[Back to Main](../README.md)
