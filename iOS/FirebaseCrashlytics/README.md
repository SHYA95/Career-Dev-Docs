# 📱 Firebase Crashlytics in iOS  
**Comprehensive Guide to Error Reporting with Focus on Non-Fatal Crashes**

---

## 📌 Table of Contents

1. [Introduction](#introduction)
2. [Fatal vs Non-Fatal Crashes](#fatal-vs-non-fatal-crashes)
3. [Installation & Setup](#installation--setup)
4. [Basic Usage](#basic-usage)
5. [Custom Logs & Keys](#custom-logs--keys)
6. [User Identification](#user-identification)
7. [Centralized Logger Utility](#centralized-logger-utility)
8. [Advanced Scenarios](#advanced-scenarios)
9. [Testing & Debugging](#testing--debugging)
10. [Monitoring & Analytics](#monitoring--analytics)
11. [Real-World Use Cases](#real-world-use-cases)
12. [Best Practices](#best-practices)
13. [Resources](#resources)

---

## 🧠 Introduction

Firebase Crashlytics is a real-time crash reporter that helps you track, prioritize, and fix stability issues in your iOS app. It supports:
- **Automatic crash reporting**
- **Manual (non-fatal) error logging**
- **Custom context logging**
- **User-centric tracking**

It is essential for production-quality apps to improve reliability and UX.

---

## 💥 Fatal vs Non-Fatal Crashes

| Crash Type   | Behavior                                  | Use Case                          |
|--------------|-------------------------------------------|-----------------------------------|
| **Fatal**     | App crashes and terminates                | Uncaught exceptions               |
| **Non-Fatal** | App continues running, logs the issue     | API failure, JSON parsing, etc.   |

Use **non-fatal logs** to capture all major runtime issues without breaking UX.

---

## 🛠 Installation & Setup

### 🔧 CocoaPods

```ruby
pod 'Firebase/Crashlytics'
````

### 📦 Swift Package Manager

Use the URL:
`https://github.com/firebase/firebase-ios-sdk`

### ✅ AppDelegate Setup

```swift
import Firebase

func application(_ application: UIApplication,
                 didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    FirebaseApp.configure()
    return true
}
```

### 🧩 Enable Crashlytics

* Confirm dSYM uploads
* Enable "Crashlytics" in the Firebase Console
* Set `Debug Information Format` to `DWARF with dSYM File` in Xcode

---

## 🧰 Basic Usage

### ✅ Log a Non-Fatal Error

```swift
Crashlytics.crashlytics().record(error: error)
```

### ✅ Log an NSError

```swift
let error = NSError(domain: "com.yourapp.network", code: 500, userInfo: [NSLocalizedDescriptionKey: "Server error"])
Crashlytics.crashlytics().record(error: error)
```

---

## 🗝 Custom Logs & Keys

### 📝 Add Custom Logs

```swift
Crashlytics.crashlytics().log("User tapped Pay button")
```

### 🏷 Add Key-Value Pairs

```swift
Crashlytics.crashlytics().setCustomValue("checkout", forKey: "screen")
Crashlytics.crashlytics().setCustomValue(user.role, forKey: "user_role")
```

### 📌 Example with Context

```swift
Crashlytics.crashlytics().log("Payment flow failed")
Crashlytics.crashlytics().setCustomValue("CreditCard", forKey: "payment_method")
Crashlytics.crashlytics().record(error: paymentError)
```

---

## 👤 User Identification

Set user ID to trace issues:

```swift
Crashlytics.crashlytics().setUserID("user_123456")
```

Clear user ID on logout:

```swift
Crashlytics.crashlytics().setUserID("")
```

> 🔒 Make sure this complies with GDPR/local regulations.

---

## 🛡 Centralized Logger Utility

Create a reusable logger to avoid scattered logic.

```swift
enum CrashlyticsLogger {
    static func log(error: Error, customKeys: [String: Any]? = nil) {
        customKeys?.forEach { key, value in
            Crashlytics.crashlytics().setCustomValue(value, forKey: key)
        }
        Crashlytics.crashlytics().record(error: error)
    }

    static func log(_ message: String) {
        Crashlytics.crashlytics().log(message)
    }
}
```

---

## 🚀 Advanced Scenarios

* Track feature flags & A/B tests in keys
* Log degraded states (e.g. offline mode fallback)
* Use enums or error types to tag error categories
* Observe patterns in non-fatal spikes pre-release

---

## 🧪 Testing & Debugging

### 💣 Trigger a fatal crash

```swift
fatalError("Test crash")
```

### 🧪 Trigger a non-fatal

```swift
let error = NSError(domain: "test", code: 1, userInfo: nil)
Crashlytics.crashlytics().record(error: error)
```

### 🔎 Check Logs

Use `Device > Logs` in Firebase Console
Wait \~5–10 minutes after sending from a real/simulated device.

---

## 📈 Monitoring & Analytics

### 🔄 Export to BigQuery

Enable BigQuery export from Firebase Console for:

* Custom queries
* Dashboards
* Aggregated views

### 📊 Slack/Jira Integrations

Use:

* Firebase Extensions
* Cloud Functions
* Third-party tools

---

## 🌍 Real-World Use Cases

| Scenario                     | Action                     |
| ---------------------------- | -------------------------- |
| API call failed but retried  | Log as non-fatal           |
| JSON decode fallback used    | Add custom key & log       |
| Payment declined by gateway  | Track user + flow context  |
| SDK returns nil on success   | Log cause with SDK version |
| Invalid user config detected | Add logs for flag state    |

---

## ✅ Best Practices

* 🧼 Avoid logging high-frequency minor errors
* 🧠 Always include custom keys for context
* 🧪 Test reports regularly on dev/stage builds
* 🧵 Group logs by screen/feature
* 🔔 Set alerts for regression monitoring

---

## 📚 Resources

* [Firebase Crashlytics Documentation](https://firebase.google.com/docs/crashlytics)
* [BigQuery Export Guide](https://firebase.google.com/docs/crashlytics/bigquery-export)
* [Apple dSYM Troubleshooting](https://firebase.google.com/docs/crashlytics/get-started?platform=ios)

