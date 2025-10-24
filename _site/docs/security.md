# Application Security in iOS Apps

[← Back to Main](../README.md) | [Previous: Advanced Topics](advanced-topics.md) | [Next: App Distribution & IAP →](app-distribution-iap.md)

## Table of Contents
- [Secure Coding Practices](#secure-coding-practices-in-swift)
- [SSL Pinning & Certificate Validation](#ssl-pinning--certificate-validation)
- [Data Protection](#data-protection)
- [Security Best Practices](#security-best-practices)

---

## Secure Coding Practices in Swift

### Input Validation

Always validate and sanitize user input to prevent security vulnerabilities.

```swift
// Email validation
func validateEmail(_ email: String) -> Bool {
    let emailRegex = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
    let emailPredicate = NSPredicate(format: "SELF MATCHES %@", emailRegex)
    return emailPredicate.evaluate(with: email)
}

// Sanitize user input
func sanitizeInput(_ input: String) -> String {
    return input.trimmingCharacters(in: .whitespacesAndNewlines)
        .replacingOccurrences(of: "<", with: "&lt;")
        .replacingOccurrences(of: ">", with: "&gt;")
}

// Example usage
let userEmail = "user@example.com"
if validateEmail(userEmail) {
    print("Valid email")
} else {
    print("Invalid email")
}
```

### Secure Data Storage

Never store sensitive data in UserDefaults. Use Keychain for passwords and tokens.

```swift
import Security

func saveToKeychain(key: String, data: Data) -> Bool {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecValueData as String: data,
        kSecAttrAccessible as String: kSecAttrAccessibleWhenUnlockedThisDeviceOnly
    ]
    
    // Delete old value if exists
    SecItemDelete(query as CFDictionary)
    
    // Add new value
    let status = SecItemAdd(query as CFDictionary, nil)
    return status == errSecSuccess
}

func loadFromKeychain(key: String) -> Data? {
    let query: [String: Any] = [
        kSecClass as String: kSecClassGenericPassword,
        kSecAttrAccount as String: key,
        kSecReturnData as String: true,
        kSecMatchLimit as String: kSecMatchLimitOne
    ]
    
    var result: AnyObject?
    let status = SecItemCopyMatching(query as CFDictionary, &result)
    
    return status == errSecSuccess ? result as? Data : nil
}

// Example: Store authentication token
let token = "secret_auth_token"
if let tokenData = token.data(using: .utf8) {
    saveToKeychain(key: "authToken", data: tokenData)
}

// Load token
if let tokenData = loadFromKeychain(key: "authToken"),
   let token = String(data: tokenData, encoding: .utf8) {
    print("Token: \(token)")
}
```

---

## SSL Pinning & Certificate Validation

SSL Pinning prevents man-in-the-middle attacks by validating server certificates.

### Certificate Pinning

```swift
import Foundation

class CertificatePinner: NSObject, URLSessionDelegate {
    
    func urlSession(_ session: URLSession,
                    didReceive challenge: URLAuthenticationChallenge,
                    completionHandler: @escaping (URLSession.AuthChallengeDisposition, URLCredential?) -> Void) {
        
        // Get server trust
        guard let serverTrust = challenge.protectionSpace.serverTrust,
              let certificate = SecTrustGetCertificateAtIndex(serverTrust, 0) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Get server certificate data
        let serverCertificateData = SecCertificateCopyData(certificate) as Data
        
        // Load pinned certificate from app bundle
        guard let pinnedCertPath = Bundle.main.path(forResource: "certificate", ofType: "cer"),
              let pinnedCertData = try? Data(contentsOf: URL(fileURLWithPath: pinnedCertPath)) else {
            completionHandler(.cancelAuthenticationChallenge, nil)
            return
        }
        
        // Compare certificates
        if serverCertificateData == pinnedCertData {
            let credential = URLCredential(trust: serverTrust)
            completionHandler(.useCredential, credential)
        } else {
            completionHandler(.cancelAuthenticationChallenge, nil)
        }
    }
}

// Usage
let session = URLSession(configuration: .default, delegate: CertificatePinner(), delegateQueue: nil)
```

### Public Key Pinning

More flexible than certificate pinning as it allows certificate rotation.

```swift
func validatePublicKey(_ serverTrust: SecTrust) -> Bool {
    let policy = SecPolicyCreateBasicX509()
    SecTrustSetPolicies(serverTrust, policy)
    
    var result: SecTrustResultType = .invalid
    SecTrustEvaluate(serverTrust, &result)
    
    guard result == .unspecified || result == .proceed else {
        return false
    }
    
    guard let serverCertificate = SecTrustGetCertificateAtIndex(serverTrust, 0),
          let serverPublicKey = SecCertificateCopyKey(serverCertificate) else {
        return false
    }
    
    // Compare with pinned public key
    guard let pinnedPublicKey = loadPinnedPublicKey() else {
        return false
    }
    
    return serverPublicKey == pinnedPublicKey
}

func loadPinnedPublicKey() -> SecKey? {
    // Load pinned public key from bundle
    // Implementation depends on your certificate format
    return nil
}
```

---

## Data Protection

Protect files and data at rest.

### File Protection

```swift
import Foundation

// Save file with encryption
func saveSecureFile(data: Data, to url: URL) throws {
    try data.write(to: url, options: .completeFileProtection)
}

// Set protection on existing file
func setFileProtection(for url: URL) throws {
    try FileManager.default.setAttributes(
        [.protectionKey: FileProtectionType.complete],
        ofItemAtPath: url.path
    )
}

// Example
let sensitiveData = "Sensitive information".data(using: .utf8)!
let documentPath = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
let fileURL = documentPath.appendingPathComponent("secure_data.txt")

try? saveSecureFile(data: sensitiveData, to: fileURL)
```

### Secure Network Communication

```swift
class SecureNetworkManager {
    
    func configureSecureSession() -> URLSession {
        let configuration = URLSessionConfiguration.default
        
        // Disable caching for sensitive data
        configuration.urlCache = nil
        configuration.requestCachePolicy = .reloadIgnoringLocalCacheData
        
        // Enforce minimum TLS version
        configuration.tlsMinimumSupportedProtocolVersion = .TLSv12
        
        return URLSession(configuration: configuration, delegate: self, delegateQueue: nil)
    }
}
```

---

## Security Best Practices

### 1. Use Biometric Authentication

```swift
import LocalAuthentication

func authenticateUser(completion: @escaping (Bool) -> Void) {
    let context = LAContext()
    var error: NSError?
    
    // Check if biometric authentication is available
    if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
        let reason = "Authenticate to access your account"
        
        context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: reason) { success, error in
            DispatchQueue.main.async {
                completion(success)
            }
        }
    } else {
        completion(false)
    }
}

// Usage
authenticateUser { success in
    if success {
        print("Authentication successful")
    } else {
        print("Authentication failed")
    }
}
```

### 2. Prevent Screenshots of Sensitive Data

```swift
import UIKit

class SecureViewController: UIViewController {
    
    override func viewDidLoad() {
        super.viewDidLoad()
        preventScreenshots()
    }
    
    func preventScreenshots() {
        // Detect screenshot
        NotificationCenter.default.addObserver(
            forName: UIApplication.userDidTakeScreenshotNotification,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleUnauthorizedScreenshot()
        }
    }
    
    func handleUnauthorizedScreenshot() {
        // Log security event
        print("Unauthorized screenshot detected")
        
        // Take action: logout user, show warning, etc.
        showSecurityAlert()
    }
    
    func showSecurityAlert() {
        let alert = UIAlertController(
            title: "Security Warning",
            message: "Screenshots are not allowed for security reasons",
            preferredStyle: .alert
        )
        alert.addAction(UIAlertAction(title: "OK", style: .default))
        present(alert, animated: true)
    }
}
```

### 3. Secure Logging

Never log sensitive information in production.

```swift
// Safe logging function
func log(_ message: String) {
    #if DEBUG
    print(message)
    #endif
}

// Sanitize sensitive data before logging
func logSanitized(_ message: String) {
    // Remove passwords
    let sanitized = message.replacingOccurrences(
        of: "password=\\w+",
        with: "password=***",
        options: .regularExpression
    )
    
    // Remove tokens
    let final = sanitized.replacingOccurrences(
        of: "token=\\w+",
        with: "token=***",
        options: .regularExpression
    )
    
    log(final)
}

// Example
logSanitized("Login: email=user@example.com password=secret123 token=abc123")
// Output in debug: "Login: email=user@example.com password=*** token=***"
```

### 4. Avoid Hardcoded Secrets

```swift
// BAD: Hardcoded API key
let apiKey = "sk_live_1234567890abcdef"

// GOOD: Load from secure storage
struct APIConfig {
    static var apiKey: String {
        return loadFromKeychain(key: "apiKey") ?? ""
    }
    
    private static func loadFromKeychain(key: String) -> String? {
        guard let data = KeychainHelper.load(key: key),
              let value = String(data: data, encoding: .utf8) else {
            return nil
        }
        return value
    }
}

// Keychain helper
class KeychainHelper {
    static func load(key: String) -> Data? {
        let query: [String: Any] = [
            kSecClass as String: kSecClassGenericPassword,
            kSecAttrAccount as String: key,
            kSecReturnData as String: true
        ]
        
        var result: AnyObject?
        let status = SecItemCopyMatching(query as CFDictionary, &result)
        
        return status == errSecSuccess ? result as? Data : nil
    }
}
```

### 5. Use App Transport Security (ATS)

Configure ATS in Info.plist to enforce secure connections:

```xml
<key>NSAppTransportSecurity</key>
<dict>
    <key>NSAllowsArbitraryLoads</key>
    <false/>
    <key>NSExceptionDomains</key>
    <dict>
        <key>example.com</key>
        <dict>
            <key>NSIncludesSubdomains</key>
            <true/>
            <key>NSRequiresCertificateTransparency</key>
            <true/>
        </dict>
    </dict>
</dict>
```

### 6. Secure API Communication

```swift
class SecureAPIClient {
    private let session: URLSession
    
    init() {
        let config = URLSessionConfiguration.default
        config.tlsMinimumSupportedProtocolVersion = .TLSv12
        config.timeoutIntervalForRequest = 30
        
        session = URLSession(configuration: config)
    }
    
    func request(url: URL, headers: [String: String], completion: @escaping (Result<Data, Error>) -> Void) {
        var request = URLRequest(url: url)
        
        // Add security headers
        headers.forEach { request.setValue($1, forHTTPHeaderField: $0) }
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("Bearer \(APIConfig.apiKey)", forHTTPHeaderField: "Authorization")
        
        session.dataTask(with: request) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NSError(domain: "No data", code: -1)))
                return
            }
            
            completion(.success(data))
        }.resume()
    }
}
```

### 7. Code Obfuscation

```swift
// Use private access control
class SecurityManager {
    private static let instance = SecurityManager()
    
    private init() { }
    
    static func shared() -> SecurityManager {
        return instance
    }
    
    private func sensitiveOperation() {
        // Hidden implementation
    }
}

// Avoid string literals for sensitive values
enum SecurityKeys {
    private static let key1 = "k"
    private static let key2 = "e"
    private static let key3 = "y"
    
    static var apiKey: String {
        return key1 + key2 + key3
    }
}
```

---

## Common Security Interview Questions

**Q: What's the difference between SSL Pinning and Certificate Pinning?**

**Answer:** Certificate pinning validates the entire server certificate against a pinned certificate. Public key pinning validates only the public key portion, which allows certificate rotation without updating the app. Both prevent man-in-the-middle attacks.

**Q: How do you secure sensitive data in iOS?**

**Answer:** 
- Store passwords and tokens in Keychain, not UserDefaults
- Enable Data Protection for files using `.completeFileProtection`
- Use HTTPS with TLS 1.2+ for network communication
- Implement SSL/certificate pinning
- Never hardcode API keys or secrets
- Use biometric authentication (Face ID, Touch ID)
- Encrypt sensitive data before storage

**Q: What is App Transport Security (ATS)?**

**Answer:** ATS is a security feature that enforces secure connections using HTTPS with TLS 1.2 or higher. It prevents accidental insecure network connections and can be configured in Info.plist. You should only disable it for specific domains when absolutely necessary.

**Q: How do you prevent reverse engineering of your iOS app?**

**Answer:**
- Use code obfuscation
- Implement jailbreak detection
- Use private and fileprivate access control
- Don't store sensitive logic in plain Swift code
- Use certificate pinning
- Implement runtime checks
- Use App Store encryption

**Q: What security measures should be taken for API keys?**

**Answer:**
- Never hardcode API keys in source code
- Store keys in Keychain
- Use environment-specific keys (dev, staging, production)
- Rotate keys regularly
- Use backend proxy for sensitive operations
- Implement rate limiting
- Monitor for unauthorized usage

**Q6: How do you detect and prevent jailbreak?**

**Answer:**

**Detection Methods:**

```swift
func isJailbroken() -> Bool {
    // Check for common jailbreak files
    let jailbreakPaths = [
        "/Applications/Cydia.app",
        "/Library/MobileSubstrate/MobileSubstrate.dylib",
        "/bin/bash",
        "/usr/sbin/sshd",
        "/etc/apt",
        "/private/var/lib/apt/"
    ]
    
    for path in jailbreakPaths {
        if FileManager.default.fileExists(atPath: path) {
            return true
        }
    }
    
    // Check if can write outside sandbox
    let testPath = "/private/jailbreak.txt"
    do {
        try "test".write(toFile: testPath, atomically: true, encoding: .utf8)
        try FileManager.default.removeItem(atPath: testPath)
        return true
    } catch {
        return false
    }
}

// Prevent app usage on jailbroken devices
if isJailbroken() {
    showJailbreakAlert()
    exit(0)
}
```

**Prevention:**
- Don't just detect, take action (disable features, logout)
- Obfuscate detection code
- Check at multiple points in app
- Server-side validation

**Considerations:**
- Some users intentionally jailbreak
- Consider your app's security requirements
- May impact legitimate users

**Q7: What is App Attest and how does it work?**

**Answer:**

**App Attest** (iOS 14+) validates that requests come from your genuine app, not modified or cloned versions.

**Implementation:**

```swift
import DeviceCheck

class AppAttestService {
    func generateKey(completion: @escaping (Result<String, Error>) -> Void) {
        DCAppAttestService.shared.generateKey { keyId, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let keyId = keyId else {
                completion(.failure(AppAttestError.noKeyId))
                return
            }
            
            // Save keyId for later use
            UserDefaults.standard.set(keyId, forKey: "appAttestKeyId")
            completion(.success(keyId))
        }
    }
    
    func attestKey(keyId: String, challenge: Data, completion: @escaping (Result<Data, Error>) -> Void) {
        DCAppAttestService.shared.attestKey(keyId, clientDataHash: challenge) { attestation, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let attestation = attestation else {
                completion(.failure(AppAttestError.noAttestation))
                return
            }
            
            // Send attestation to your server for validation
            completion(.success(attestation))
        }
    }
}

enum AppAttestError: Error {
    case noKeyId
    case noAttestation
}
```

**Flow:**
1. App generates key
2. App requests attestation with server challenge
3. Server validates attestation with Apple
4. Server trusts subsequent requests from this device

**Benefits:**
- Verify app authenticity
- Prevent API abuse
- Stop modified apps
- Protect against replay attacks

**Use Cases:**
- Financial apps
- Health apps
- Apps with valuable API access
- Preventing fraud

---

[← Previous: Advanced Topics](advanced-topics.md) | [Next: App Distribution & IAP →](app-distribution-iap.md)

[Back to Main](../README.md)
