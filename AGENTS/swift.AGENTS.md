# Swift — AGENTS Protocol

## Swift

### Minimum Required Linters/Analyzers

- **SwiftLint** — style and convention enforcement
- **swift-format** — official Apple formatter
- **Xcode Analyzer** — static analysis (Build Settings → Analyze During Build = Yes)

Install if missing:
```bash
brew install swiftlint swift-format
# or via SPM:
# .package(url: "https://github.com/realm/SwiftLint.git", from: "0.55.0")
```

Run:
```bash
swiftlint lint --strict
swift-format lint -r Sources/
```

### Optionals: Unwrap Safely or Don't Unwrap

Force unwrapping (`!`) in Swift is the same anti-pattern as `!!` in Kotlin
and `unwrap()` in Rust. It crashes at runtime. Handle the absence.

```swift
// GOOD: Guard-let for early exit
func loadUser(id: String) throws -> User {
    guard let data = cache.get(id) else {
        throw AppError.userNotFound(id: id)
    }
    guard let user = User(data: data) else {
        throw AppError.corruptedData(id: id, detail: "Failed to decode User")
    }
    return user
}

// GOOD: if-let for conditional logic
if let email = user.email {
    sendWelcome(to: email)
}

// GOOD: Nil coalescing for defaults
let displayName = user.nickname ?? user.email ?? "Anonymous"

// BAD: Force unwrap — crash in production
func loadUser(id: String) -> User {
    let data = cache.get(id)!  // crash if nil
    return User(data: data)!   // crash if decode fails
}

// ALSO BAD: try! — "I'm sure this never fails"
let config = try! JSONDecoder().decode(Config.self, from: data)
// narrator: it failed
```

### Error Handling: No Untyped Throws

```swift
// GOOD: Typed errors with context
enum NetworkError: Error, LocalizedError {
    case timeout(url: URL, after: TimeInterval)
    case httpError(url: URL, statusCode: Int, body: String)
    case decodingFailed(type: String, underlying: Error)

    var errorDescription: String? {
        switch self {
        case .timeout(let url, let duration):
            return "Request to \(url) timed out after \(duration)s"
        case .httpError(let url, let code, let body):
            return "HTTP \(code) from \(url): \(body.prefix(200))"
        case .decodingFailed(let type, let error):
            return "Failed to decode \(type): \(error.localizedDescription)"
        }
    }
}

// BAD: Stringly-typed errors
throw NSError(domain: "NetworkError", code: -1, userInfo: [
    NSLocalizedDescriptionKey: "something went wrong"
])
```

### ARC and Memory Management

Swift uses Automatic Reference Counting. It is not garbage collection.
Strong reference cycles will leak memory silently.

```swift
// GOOD: weak self in closures that outlive the owner
class ViewController: UIViewController {
    func loadData() {
        dataService.fetch { [weak self] result in
            guard let self else { return }
            self.updateUI(with: result)
        }
    }
}

// BAD: Strong capture — ViewController can never be deallocated
class ViewController: UIViewController {
    func loadData() {
        dataService.fetch { result in
            self.updateUI(with: result)  // strong ref cycle
        }
    }
}
```

### Concurrency: Structured with async/await

```swift
// GOOD: Structured concurrency with TaskGroup
func fetchDashboard(userId: String) async throws -> Dashboard {
    async let profile = profileService.fetch(userId)
    async let orders = orderService.fetchRecent(userId)
    async let notifications = notificationService.unread(userId)

    return Dashboard(
        profile: try await profile,
        orders: try await orders,
        notifications: try await notifications
    )
}

// BAD: Unstructured Task — no cancellation propagation
func fetchDashboard(userId: String) {
    Task.detached {
        // if the caller is cancelled, this keeps running
        let profile = try await self.profileService.fetch(userId)
    }
}
```

### Value Types vs Reference Types

```swift
// GOOD: Struct for data — value semantics, no shared mutable state
struct Point {
    var x: Double
    var y: Double
}

// GOOD: Class only when you need identity or inheritance
class DatabaseConnection {
    private let handle: OpaquePointer
    deinit { sqlite3_close(handle) }
}

// BAD: Class for pure data — shared mutable state nightmare
class Point {
    var x: Double = 0
    var y: Double = 0
    // passed by reference — mutations in one place affect all holders
}
```

### Common Agent Swift Mistakes

1. **Force unwrap (`!`) and `try!`** — crashes at runtime. Use guard/if-let.
2. **Strong reference cycles** — closures capturing `self` strongly leak
   the entire view controller tree.
3. **`Task.detached` / unstructured concurrency** — no cancellation, no scope.
4. **Implicitly unwrapped optionals (`String!`)** — only acceptable for
   `@IBOutlet`. Everywhere else, use `String?` and handle nil.
5. **Massive view controllers** — extract logic into view models, services,
   coordinators.
6. **Ignoring `@Sendable` warnings** — data races in concurrent code.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
