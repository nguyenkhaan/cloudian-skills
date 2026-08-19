# C# — AGENTS Protocol

## C#

### Minimum Required Linters/Analyzers

- **Roslyn Analyzers** — built-in, enable all with `<AnalysisLevel>latest-All</AnalysisLevel>`
- **StyleCop.Analyzers** — style enforcement
- **.editorconfig** — formatting rules (committed to repo, not optional)
- **dotnet format** — auto-formatter

```xml
<!-- In .csproj — enable strict analysis -->
<PropertyGroup>
    <AnalysisLevel>latest-All</AnalysisLevel>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
</PropertyGroup>
```

Run:
```bash
dotnet format --verify-no-changes
dotnet build -warnaserror
```

### Nullable Reference Types: Always Enabled

C# 8+ has nullable reference types. `<Nullable>enable</Nullable>` is
mandatory. This is non-negotiable — it catches null reference exceptions
at compile time instead of production.

```csharp
// GOOD: Nullable annotations — compiler catches null bugs
public string GetDisplayName(User user)
{
    ArgumentNullException.ThrowIfNull(user);
    return user.Nickname ?? user.Email ?? throw new InvalidOperationException(
        $"User {user.Id} has neither nickname nor email"
    );
}

// GOOD: Nullable parameter when null is valid
public void SendNotification(User user, string? customMessage = null)
{
    var message = customMessage ?? GetDefaultMessage(user);
    // ...
}

// BAD: Disabling nullable with pragma — hiding bugs
#nullable disable
public string GetDisplayName(User user)
{
    return user.Nickname ?? user.Email;  // no warning if user is null
}
```

### async/await: Don't Block, Don't Fire-and-Forget

```csharp
// GOOD: Async all the way down
public async Task<User> GetUserAsync(string id, CancellationToken ct)
{
    var response = await _httpClient.GetAsync($"/users/{id}", ct);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadFromJsonAsync<User>(ct)
        ?? throw new InvalidOperationException($"Null user response for id={id}");
}

// BAD: .Result / .Wait() — deadlocks in ASP.NET, WPF, etc.
public User GetUser(string id)
{
    var response = _httpClient.GetAsync($"/users/{id}").Result;  // DEADLOCK
    return response.Content.ReadFromJsonAsync<User>().Result;
}

// BAD: async void — exceptions vanish, no way to await
public async void SaveUser(User user)  // async void = fire and forget
{
    await _repository.SaveAsync(user);
    // if this throws, the exception is unobserved and swallowed
}

// GOOD: async Task — caller can await and catch exceptions
public async Task SaveUserAsync(User user)
{
    await _repository.SaveAsync(user);
}
```

### Always Pass CancellationToken

Every async method that does I/O must accept and forward `CancellationToken`.
Without it, cancelled HTTP requests keep running on the server, wasting
resources.

```csharp
// GOOD: Token flows through the entire call chain
public async Task<Report> GenerateReportAsync(
    string userId,
    CancellationToken cancellationToken = default)
{
    var data = await _dataService.FetchAsync(userId, cancellationToken);
    var result = await _processor.ProcessAsync(data, cancellationToken);
    return result;
}

// BAD: Token ignored — request cancelled but work continues
public async Task<Report> GenerateReportAsync(string userId)
{
    var data = await _dataService.FetchAsync(userId);  // no cancellation
    var result = await _processor.ProcessAsync(data);
    return result;
}
```

### IDisposable: If It Implements It, Dispose It

```csharp
// GOOD: using statement ensures disposal
await using var connection = new SqlConnection(connectionString);
await connection.OpenAsync(ct);
await using var command = connection.CreateCommand();
command.CommandText = "SELECT * FROM users WHERE id = @id";
command.Parameters.AddWithValue("@id", userId);

// BAD: Connection leak — eventually pool exhaustion, then crash
var connection = new SqlConnection(connectionString);
connection.Open();
var command = connection.CreateCommand();
// exception here means connection is never closed
```

### Dependency Injection: Constructor Injection Only

```csharp
// GOOD: Dependencies are explicit and immutable
public class UserService
{
    private readonly IUserRepository _repository;
    private readonly ILogger<UserService> _logger;

    public UserService(IUserRepository repository, ILogger<UserService> logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }
}

// BAD: Service locator — hidden dependencies, untestable
public class UserService
{
    public void DoWork()
    {
        var repo = ServiceLocator.Get<IUserRepository>();  // hidden dependency
        var logger = ServiceLocator.Get<ILogger>();
    }
}
```

### LINQ: Readable, Not Clever

```csharp
// GOOD: Readable LINQ
var activeUsers = users
    .Where(u => u.IsActive && u.CreatedAt > cutoffDate)
    .OrderByDescending(u => u.LastLoginAt)
    .Select(u => new UserSummary(u.Id, u.Email, u.LastLoginAt))
    .ToList();

// BAD: Nested LINQ that nobody can read
var result = users.GroupBy(u => u.Department)
    .Select(g => new {
        Dept = g.Key,
        Users = g.Where(u => u.IsActive)
            .SelectMany(u => u.Roles.Where(r => r.Permissions
                .Any(p => p.Level > 3)))
            .GroupBy(r => r.Name)
            .ToDictionary(rg => rg.Key, rg => rg.Count())
    }).Where(d => d.Users.Any()).ToList();
// If you can't explain this in one sentence, split it into methods.
```

### Common Agent C# Mistakes

1. **`.Result` / `.Wait()`** — deadlocks. Use `await` all the way.
2. **`async void`** — unobserved exceptions. Only valid for event handlers.
3. **Missing `CancellationToken`** — cancelled requests waste server resources.
4. **Not disposing `IDisposable`** — connection pool exhaustion, file handle leaks.
5. **`<Nullable>disable</Nullable>`** — hides null reference bugs.
6. **`catch (Exception) { }`** — the empty catch block, C# edition. See main
   AGENTS.md Section 7.10.
7. **Service Locator pattern** — hidden dependencies, impossible to test.

---

> **This file is part of the AGENTS protocol.** For general rules, workflow, and the full table of contents, see [AGENTS.md](./AGENTS.md).
