# Kotlin — AGENTS Protocol

## Kotlin

### Minimum Required Linters/Analyzers

- **ktlint** — official Kotlin linter and formatter
- **detekt** — static analysis with complexity and code smell detection
- **Android Lint** — mandatory for Android projects

Install if missing:
```bash
# ktlint
curl -sSLO https://github.com/pinterest/ktlint/releases/latest/download/ktlint
chmod +x ktlint

# detekt (Gradle)
# Add to build.gradle.kts:
plugins {
    id("io.gitlab.arturbosch.detekt") version "1.23.+"
}
```

Run:
```bash
ktlint --editorconfig=.editorconfig "src/**/*.kt"
./gradlew detekt
```

### Null Safety Is the Whole Point

Kotlin's type system distinguishes nullable from non-nullable types. This
is the language's single most important feature. Agents who bypass it with
`!!` are writing Java with extra syntax.

```kotlin
// GOOD: Safe call + elvis with meaningful default/error
fun getUserEmail(userId: String): String {
    val user = userRepository.findById(userId)
        ?: throw UserNotFoundException("User not found: id=$userId")
    return user.email
}

// GOOD: Safe chain for optional display logic
val displayName = user?.profile?.displayName ?: user?.email ?: "Anonymous"

// BAD: !! is a NullPointerException waiting to happen
fun getUserEmail(userId: String): String {
    val user = userRepository.findById(userId)!!  // NPE in production
    return user.email!!  // another NPE
}

// ALSO BAD: Catching NPE instead of handling nullability
try {
    val email = user!!.email!!
} catch (e: NullPointerException) {
    // you just recreated Java's worst pattern in Kotlin
}
```

The `!!` operator exists for interop with Java code that you cannot control.
In pure Kotlin code, every `!!` is a design failure. If you find yourself
reaching for it, the function signature is wrong — make the parameter or
return type nullable and handle it properly.

### Coroutines: Structured Concurrency

```kotlin
// GOOD: Structured concurrency — cancellation propagates correctly
suspend fun fetchUserData(userId: String): UserData = coroutineScope {
    val profile = async { profileService.fetch(userId) }
    val orders = async { orderService.fetch(userId) }
    UserData(
        profile = profile.await(),
        orders = orders.await()
    )
}

// BAD: GlobalScope — fire and forget, no cancellation, resource leak
fun fetchUserData(userId: String) {
    GlobalScope.launch {
        val profile = profileService.fetch(userId)
        // if this coroutine leaks, it runs forever
    }
}

// BAD: Catching CancellationException — breaks structured concurrency
try {
    longRunningWork()
} catch (e: Exception) {  // catches CancellationException too!
    logger.error("Failed", e)
    // coroutine should have been cancelled but now it continues
}

// GOOD: Rethrow CancellationException
try {
    longRunningWork()
} catch (e: CancellationException) {
    throw e  // never swallow this
} catch (e: Exception) {
    logger.error("Failed", e)
}
```

### Data Classes for Data, Sealed Classes for State

```kotlin
// GOOD: Data class for pure data
data class User(
    val id: String,
    val email: String,
    val createdAt: Instant,
)

// GOOD: Sealed class for exhaustive state handling
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String, val cause: Throwable? = null) : Result<Nothing>()
    data object Loading : Result<Nothing>()
}

// Compiler enforces exhaustive when
fun handleResult(result: Result<User>) = when (result) {
    is Result.Success -> showUser(result.data)
    is Result.Error -> showError(result.message)
    is Result.Loading -> showSpinner()
    // no else needed — compiler checks all branches
}

// BAD: Stringly-typed state
fun handleResult(status: String, data: Any?) {
    if (status == "success") { /* cast data and pray */ }
    else if (status == "error") { /* what type is data here? */ }
}
```

### Extension Functions: Don't Abuse Them

```kotlin
// GOOD: Extension that genuinely extends a type's vocabulary
fun String.isValidEmail(): Boolean =
    matches(Regex("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$"))

// BAD: Extension that hides business logic in the wrong place
fun String.processPayment(): PaymentResult {
    // a String should not know how to process payments
}
```

### Scope Functions: Choose Deliberately

```kotlin
// GOOD: apply for object configuration
val client = HttpClient().apply {
    connectTimeout = Duration.ofSeconds(10)
    readTimeout = Duration.ofSeconds(30)
    addInterceptor(loggingInterceptor)
}

// GOOD: let for null-safe transformations
val length = name?.let { it.trim().length } ?: 0

// BAD: Nested scope functions — unreadable
user?.let { u ->
    u.profile?.also { p ->
        p.settings?.run {
            this.apply {
                // what is 'this'? what is 'it'? nobody knows
            }
        }
    }
}
```

### Common Agent Kotlin Mistakes

1. **`!!` everywhere** — defeats Kotlin's null safety. Use `?.`, `?:`, `let`.
2. **GlobalScope.launch** — leaks coroutines. Use structured concurrency.
3. **Catching `Exception` (incl. CancellationException)** — breaks cancellation.
4. **Mutable data classes** — use `val`, not `var`, for data class properties.
5. **`lateinit` for things that can be nullable** — `lateinit` throws if
   accessed before init. Use nullable type + null check instead.
6. **Java-style builders in Kotlin** — use `apply` or named arguments.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
