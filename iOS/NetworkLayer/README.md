# 📡 iOS Networking Layer

A robust, secure, and modular networking layer designed for iOS applications using **Swift** and **Alamofire**. This layer abstracts all network-related logic, simplifies API interaction, manages authentication, and supports automatic retries with token refresh.

---

## 📚 Table of Contents

* [Overview](#overview)
* [Requirements](#requirements)
* [Architecture](#architecture)
* [Components](#components)

  * [APIRequestBuilder](#1-apirequestbuilder)
  * [TokenInterceptor](#2-tokeninterceptor)
  * [APIRequestProvider](#3-apirequestprovider)
  * [TokenManager](#4-tokenmanager)
* [Concepts Explained](#concepts-explained)

  * [Request Builder](#request-builder)
  * [Request Adapter](#request-adapter)
  * [Request Retrier](#request-retrier)
  * [Request Interceptor](#request-interceptor)
  * [HTTP Status Codes](#http-status-codes)
  * [Differences Between Adapter, Retrier, and Interceptor](#differences-between-adapter-retrier-and-interceptor)
  * [SSL Pinning](#ssl-pinning)
  * [Authentication & Token Refresh](#authentication--token-refresh)
* [Usage Example](#usage-example)
* [Testing](#testing)
* [Resources](#resources)

---

## 🧩 Overview

This layer ensures:

✅ Modular components
✅ Secure token management & SSL pinning
✅ Automatic token refreshing
✅ Clean and scalable architecture
✅ Retry failed requests based on logic

---

## 🛠 Requirements

* iOS 13+
* Swift 5.5+
* Alamofire 5.6+
* Xcode 13+

---

## 🧱 Architecture

The system follows a clear separation of concerns:

* **Builder:** Defines and encodes requests
* **Interceptor (Adapter + Retrier):** Adds tokens and retries on failure
* **Session:** Handles SSL pinning and integrates interceptors
* **Manager:** Manages access and refresh tokens

---

## 🔧 Components

### 1. `APIRequestBuilder.swift`

Defines all API endpoints and conforms to `URLRequestConvertible`.

```swift
enum APIRequestBuilder: URLRequestConvertible {
    case login(email: String, password: String)
    case getUserProfile

    var baseURL: URL {
        return URL(string: "https://api.example.com")!
    }

    var method: HTTPMethod {
        switch self {
        case .login: return .post
        case .getUserProfile: return .get
        }
    }

    var path: String {
        switch self {
        case .login: return "/auth/login"
        case .getUserProfile: return "/user/profile"
        }
    }

    var headers: HTTPHeaders {
        var headers: HTTPHeaders = ["Content-Type": "application/json"]
        if let token = TokenManager.shared.accessToken {
            headers.add(name: "Authorization", value: "Bearer \(token)")
        }
        return headers
    }

    var parameters: Parameters? {
        switch self {
        case .login(let email, let password):
            return ["email": email, "password": password]
        default:
            return nil
        }
    }

    func asURLRequest() throws -> URLRequest {
        let url = baseURL.appendingPathComponent(path)
        var request = try URLRequest(url: url, method: method, headers: headers)
        if let params = parameters {
            request = try JSONEncoding.default.encode(request, with: params)
        }
        return request
    }
}
```

---

### 2. `TokenInterceptor.swift`

Combines **Adapter** and **Retrier** logic.

```swift
class TokenInterceptor: RequestInterceptor {
    private let retryLimit = 1
    private var retryCount = 0

    func adapt(_ urlRequest: URLRequest,
               for session: Session,
               completion: @escaping (Result<URLRequest, Error>) -> Void) {
        var adaptedRequest = urlRequest
        if let token = TokenManager.shared.accessToken {
            adaptedRequest.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        completion(.success(adaptedRequest))
    }

    func retry(_ request: Request,
               for session: Session,
               dueTo error: Error,
               completion: @escaping (RetryResult) -> Void) {
        guard retryCount < retryLimit else {
            completion(.doNotRetry)
            return
        }

        if let response = request.task?.response as? HTTPURLResponse, response.statusCode == 401 {
            TokenManager.shared.refreshToken { success in
                if success {
                    self.retryCount += 1
                    completion(.retry)
                } else {
                    completion(.doNotRetry)
                }
            }
        } else {
            completion(.doNotRetry)
        }
    }
}
```

---

### 3. `APIRequestProvider.swift`

Creates a customized `Session` object with SSL pinning and the interceptor.

```swift
class APIRequestProvider {
    static let shared = APIRequestProvider()

    private let evaluators: [String: ServerTrustEvaluating] = [
        "api.example.com": PinnedCertificatesTrustEvaluator()
    ]

    lazy var session: Session = {
        let configuration = URLSessionConfiguration.default
        let manager = ServerTrustManager(evaluators: evaluators)
        let interceptor = TokenInterceptor()
        return Session(configuration: configuration,
                       interceptor: interceptor,
                       serverTrustManager: manager)
    }()
}
```

---

### 4. `TokenManager.swift`

Handles access and refresh token storage and logic.

```swift
class TokenManager {
    static let shared = TokenManager()
    var accessToken: String?
    var refreshToken: String?

    func refreshToken(completion: @escaping (Bool) -> Void) {
        // Simulate token refresh logic
        DispatchQueue.global().asyncAfter(deadline: .now() + 1.0) {
            self.accessToken = "new_access_token"
            completion(true)
        }
    }
}
```

---

## 🧠 Concepts Explained

### Request Builder

Builds all requests before sending them. Defines method, headers, params, etc.

### Request Adapter

Modifies the request **before sending**, e.g., adds an `Authorization` header.

### Request Retrier

Determines **if and when to retry** a failed request (e.g., if `401 Unauthorized`).

### Request Interceptor

Combines Adapter + Retrier into one object for simplicity.

```swift
let interceptor = Interceptor(adapter: YourAdapter(), retrier: YourRetrier())
```

---

## 🌐 HTTP Status Codes

| Code | Meaning               | Usage Example                       |
| ---- | --------------------- | ----------------------------------- |
| 200  | OK                    | Request succeeded                   |
| 201  | Created               | Resource created                    |
| 204  | No Content            | No data to return                   |
| 400  | Bad Request           | Invalid input data                  |
| 401  | Unauthorized          | Missing or expired token            |
| 403  | Forbidden             | Access denied                       |
| 404  | Not Found             | Endpoint not found                  |
| 422  | Unprocessable Entity  | Validation error (e.g. wrong input) |
| 500  | Internal Server Error | Something went wrong on the server  |

---

## 🔄 Differences Between Adapter, Retrier, and Interceptor

| Concept     | When It Runs   | Responsibility                 | Example           |
| ----------- | -------------- | ------------------------------ | ----------------- |
| Adapter     | Before request | Adds/modifies headers          | Add token         |
| Retrier     | After failure  | Handles retries                | Refresh token     |
| Interceptor | Wraps both     | Adapter + Retrier in one class | Centralized logic |

---

## 🔐 SSL Pinning

Used for ensuring secure communication with backend.

```swift
let evaluators: [String: ServerTrustEvaluating] = [
    "api.example.com": PinnedCertificatesTrustEvaluator()
]

let manager = ServerTrustManager(evaluators: evaluators)
let session = Session(serverTrustManager: manager)
```

---

## 🔑 Authentication & Token Refresh

1. Save access/refresh tokens in Keychain or secure store.
2. Use `Adapter` to inject token.
3. Use `Retrier` to detect `401`, call refresh endpoint, and retry.

---

## 📤 Usage Example

```swift
APIRequestProvider.shared.session
    .request(APIRequestBuilder.login(email: "shrouk@example.com", password: "password123"))
    .validate()
    .responseDecodable(of: LoginResponse.self) { response in
        switch response.result {
        case .success(let user):
            print("✅ Logged in:", user)
        case .failure(let error):
            print("❌ Error:", error)
        }
    }
```

---

## 🧪 Testing

Use libraries like `OHHTTPStubs` or mock your `Session` to simulate network behavior. Write unit tests for:

* Token refresh
* Unauthorized handling
* Request encoding

---

## 📘 Resources

* [Alamofire Documentation](https://alamofire.github.io/Alamofire/)
* [HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
* [Apple Networking Docs](https://developer.apple.com/documentation/foundation/urlsession)

