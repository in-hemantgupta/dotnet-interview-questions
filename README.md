# 🎯 .NET & C# Interview Questions — Complete Guide

> A curated collection of **200+ real interview questions** with detailed answers, code examples, and explanations. From fresher to senior level. Used by thousands of developers to crack .NET interviews.

![GitHub stars](https://img.shields.io/github/stars/in-hemantgupta/dotnet-interview-questions?style=social)
![GitHub forks](https://img.shields.io/github/forks/in-hemantgupta/dotnet-interview-questions?style=social)
![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)

---

## 📌 Table of Contents

- [C# Fundamentals](#-c-fundamentals)
- [OOP Concepts](#-oop-concepts)
- [SOLID Principles](#-solid-principles)
- [Design Patterns](#-design-patterns-gof)
- [ASP.NET Core](#-aspnet-core)
- [Entity Framework](#-entity-framework)
- [Async & Threading](#-async--threading)
- [Memory & Performance](#-memory--performance)
- [SQL & Database](#-sql--database)
- [System Design](#-system-design)

---

## 🔷 C# Fundamentals

### 1. What is the difference between `value type` and `reference type`?

| Feature | Value Type | Reference Type |
|---|---|---|
| Stored in | Stack | Heap |
| Examples | int, float, struct, bool | class, string, array |
| Copy behavior | Copies the value | Copies the reference |
| Default value | 0 / false | null |

```csharp
// Value type — copy is independent
int a = 10;
int b = a;
b = 20;
Console.WriteLine(a); // 10 — unchanged

// Reference type — both point to same object
var list1 = new List<int> { 1, 2, 3 };
var list2 = list1;
list2.Add(4);
Console.WriteLine(list1.Count); // 4 — changed!
```

---

### 2. What is the difference between `==` and `.Equals()`?

- `==` checks **reference equality** for objects (unless overloaded)
- `.Equals()` checks **value equality** (can be overridden)

```csharp
string a = new string("hello");
string b = new string("hello");

Console.WriteLine(a == b);        // True (string overloads ==)
Console.WriteLine(a.Equals(b));   // True
Console.WriteLine(object.ReferenceEquals(a, b)); // False
```

---

### 3. What is `boxing` and `unboxing`?

```csharp
// Boxing — value type → object (heap allocation happens)
int num = 42;
object boxed = num;

// Unboxing — object → value type (explicit cast required)
int unboxed = (int)boxed;
```
> ⚠️ Boxing/unboxing is expensive — avoid in performance-critical loops.

---

### 4. What is the difference between `string` and `StringBuilder`?

| | string | StringBuilder |
|---|---|---|
| Mutability | Immutable | Mutable |
| Performance | Slow for concatenation | Fast for concatenation |
| Memory | New object on each change | Same object modified |

```csharp
// Bad — creates 1000 new string objects
string result = "";
for (int i = 0; i < 1000; i++)
    result += i;

// Good — modifies same object
var sb = new StringBuilder();
for (int i = 0; i < 1000; i++)
    sb.Append(i);
string result = sb.ToString();
```

---

### 5. What are `nullable` types?

```csharp
int? age = null;  // Nullable int

if (age.HasValue)
    Console.WriteLine(age.Value);

// Null coalescing operator
int display = age ?? 0; // 0 if null

// Null conditional operator
string name = null;
int? len = name?.Length; // null, no exception
```

---

### 6. What is the difference between `const` and `readonly`?

| | const | readonly |
|---|---|---|
| Set at | Compile time | Runtime (constructor) |
| Can be static | Always static | Can be instance |
| Type | Primitives only | Any type |

```csharp
public class Config
{
    public const double PI = 3.14159;           // compile-time
    public readonly DateTime StartTime;          // runtime

    public Config()
    {
        StartTime = DateTime.Now;               // set in constructor
    }
}
```

---

### 7. What is `delegate` in C#?

A delegate is a **type-safe function pointer**.

```csharp
// Declare delegate
delegate int MathOperation(int a, int b);

// Use delegate
MathOperation add = (a, b) => a + b;
MathOperation multiply = (a, b) => a * b;

Console.WriteLine(add(3, 4));       // 7
Console.WriteLine(multiply(3, 4));  // 12
```

---

### 8. What is the difference between `Action`, `Func`, and `Predicate`?

```csharp
// Action — no return value
Action<string> print = name => Console.WriteLine(name);
print("Hemant");

// Func — has return value (last type param is return type)
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(3, 4)); // 7

// Predicate — returns bool
Predicate<int> isEven = n => n % 2 == 0;
Console.WriteLine(isEven(4)); // True
```

---

### 9. What is LINQ?

LINQ (Language Integrated Query) allows querying collections in a SQL-like syntax.

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Method syntax
var evenNumbers = numbers
    .Where(n => n % 2 == 0)
    .OrderByDescending(n => n)
    .ToList();

// Query syntax
var result = from n in numbers
             where n % 2 == 0
             orderby n descending
             select n;
```

---

### 10. What is `IEnumerable` vs `IQueryable`?

| | IEnumerable | IQueryable |
|---|---|---|
| Execution | In-memory (client-side) | Database (server-side) |
| Use case | Collections, lists | Entity Framework, ORMs |
| Performance | Loads all data first | Translates to SQL query |

```csharp
// IEnumerable — loads ALL users then filters in memory
IEnumerable<User> users = dbContext.Users.ToList().Where(u => u.Age > 18);

// IQueryable — sends WHERE to SQL, only loads matching rows
IQueryable<User> users = dbContext.Users.Where(u => u.Age > 18);
```

---

## 🔷 OOP Concepts

### 11. What are the 4 pillars of OOP?

**1. Encapsulation** — hiding internal state
```csharp
public class BankAccount
{
    private decimal _balance;
    public decimal Balance => _balance; // read-only outside

    public void Deposit(decimal amount)
    {
        if (amount > 0) _balance += amount;
    }
}
```

**2. Inheritance** — reusing parent class behavior
```csharp
public class Animal { public void Eat() => Console.WriteLine("Eating"); }
public class Dog : Animal { public void Bark() => Console.WriteLine("Woof!"); }
```

**3. Polymorphism** — same method, different behavior
```csharp
public class Shape { public virtual double Area() => 0; }
public class Circle : Shape { public override double Area() => Math.PI * r * r; }
public class Square : Shape { public override double Area() => side * side; }
```

**4. Abstraction** — hiding complexity
```csharp
public abstract class Vehicle
{
    public abstract void Start(); // must implement
    public void Stop() => Console.WriteLine("Stopped"); // shared behavior
}
```

---

### 12. What is the difference between `abstract class` and `interface`?

| | Abstract Class | Interface |
|---|---|---|
| Multiple inheritance | ❌ No | ✅ Yes |
| Constructor | ✅ Yes | ❌ No |
| Fields | ✅ Yes | ❌ No |
| Access modifiers | ✅ Yes | Public only |
| Default implementation | ✅ Yes | ✅ Yes (C# 8+) |

> **Rule of thumb:** Use **interface** for "can do" (IFlyable, ISerializable). Use **abstract class** for "is a" (Animal, Vehicle).

---

### 13. What is `virtual` vs `override` vs `new`?

```csharp
public class Base
{
    public virtual void Show() => Console.WriteLine("Base");
}

public class Child : Base
{
    public override void Show() => Console.WriteLine("Child"); // polymorphic
}

public class AnotherChild : Base
{
    public new void Show() => Console.WriteLine("AnotherChild"); // hides, not overrides
}

Base obj = new Child();
obj.Show(); // "Child" — override is polymorphic

Base obj2 = new AnotherChild();
obj2.Show(); // "Base" — new keyword is NOT polymorphic
```

---

## 🔷 SOLID Principles

### 14. Single Responsibility Principle (SRP)

> A class should have **only one reason to change**.

```csharp
// ❌ Bad — handles both user logic AND email
public class UserService
{
    public void CreateUser(User user) { /* save to DB */ }
    public void SendWelcomeEmail(User user) { /* send email */ }
}

// ✅ Good — separated concerns
public class UserService { public void CreateUser(User user) { } }
public class EmailService { public void SendWelcomeEmail(User user) { } }
```

---

### 15. Open/Closed Principle (OCP)

> Open for **extension**, closed for **modification**.

```csharp
// ❌ Bad — must modify class to add new discount type
public class DiscountService
{
    public double GetDiscount(string type)
    {
        if (type == "Student") return 0.2;
        if (type == "Senior") return 0.3;
        return 0;
    }
}

// ✅ Good — add new discount without changing existing code
public interface IDiscount { double GetDiscount(); }
public class StudentDiscount : IDiscount { public double GetDiscount() => 0.2; }
public class SeniorDiscount : IDiscount { public double GetDiscount() => 0.3; }
```

---

### 16. Liskov Substitution Principle (LSP)

> Subtypes must be **substitutable** for their base types.

```csharp
// ❌ Bad — Square breaks Rectangle behavior
public class Rectangle { public virtual int Width { get; set; } public virtual int Height { get; set; } }
public class Square : Rectangle
{
    public override int Width { set { base.Width = base.Height = value; } }
    // This breaks: rectangle.Width = 4; rectangle.Height = 5; area should be 20
}

// ✅ Good — use a common abstraction instead
public abstract class Shape { public abstract int Area(); }
```

---

### 17. Interface Segregation Principle (ISP)

> Don't force classes to implement interfaces they **don't use**.

```csharp
// ❌ Bad — Printer forced to implement Fax
public interface IMachine { void Print(); void Scan(); void Fax(); }

// ✅ Good — split into smaller interfaces
public interface IPrinter { void Print(); }
public interface IScanner { void Scan(); }
public interface IFax { void Fax(); }

public class SimplePrinter : IPrinter { public void Print() { } }
```

---

### 18. Dependency Inversion Principle (DIP)

> Depend on **abstractions**, not concretions.

```csharp
// ❌ Bad — tightly coupled to SqlDatabase
public class OrderService
{
    private SqlDatabase _db = new SqlDatabase();
    public void Save(Order order) => _db.Save(order);
}

// ✅ Good — depends on abstraction
public interface IDatabase { void Save(Order order); }
public class OrderService
{
    private readonly IDatabase _db;
    public OrderService(IDatabase db) { _db = db; } // inject any database
    public void Save(Order order) => _db.Save(order);
}
```

---

## 🔷 Design Patterns (GOF)

### 19. Singleton Pattern

> Ensures only **one instance** of a class exists.

```csharp
public class Singleton
{
    private static readonly Lazy<Singleton> _instance =
        new Lazy<Singleton>(() => new Singleton());

    private Singleton() { }

    public static Singleton Instance => _instance.Value;
}

// Usage
var s1 = Singleton.Instance;
var s2 = Singleton.Instance;
Console.WriteLine(s1 == s2); // True — same instance
```

---

### 20. Factory Pattern

> Creates objects without exposing creation logic.

```csharp
public interface IAnimal { void Speak(); }
public class Dog : IAnimal { public void Speak() => Console.WriteLine("Woof!"); }
public class Cat : IAnimal { public void Speak() => Console.WriteLine("Meow!"); }

public class AnimalFactory
{
    public static IAnimal Create(string type) => type switch
    {
        "Dog" => new Dog(),
        "Cat" => new Cat(),
        _ => throw new ArgumentException("Unknown animal")
    };
}

// Usage
var animal = AnimalFactory.Create("Dog");
animal.Speak(); // Woof!
```

---

### 21. Repository Pattern

> Abstracts data access logic — very common in .NET interviews!

```csharp
public interface IUserRepository
{
    User GetById(int id);
    IEnumerable<User> GetAll();
    void Add(User user);
    void Delete(int id);
}

public class UserRepository : IUserRepository
{
    private readonly AppDbContext _context;
    public UserRepository(AppDbContext context) { _context = context; }

    public User GetById(int id) => _context.Users.Find(id);
    public IEnumerable<User> GetAll() => _context.Users.ToList();
    public void Add(User user) { _context.Users.Add(user); _context.SaveChanges(); }
    public void Delete(int id)
    {
        var user = GetById(id);
        if (user != null) { _context.Users.Remove(user); _context.SaveChanges(); }
    }
}
```

---

## 🔷 ASP.NET Core

### 22. What is the difference between `AddScoped`, `AddSingleton`, `AddTransient`?

| Lifetime | Created | Use case |
|---|---|---|
| `AddTransient` | Every time requested | Lightweight, stateless services |
| `AddScoped` | Once per HTTP request | DB context, per-request services |
| `AddSingleton` | Once per app lifetime | Config, caching, logging |

```csharp
services.AddTransient<IEmailService, EmailService>();
services.AddScoped<IUserRepository, UserRepository>();
services.AddSingleton<IConfiguration, Configuration>();
```

---

### 23. What is Middleware in ASP.NET Core?

```csharp
// Custom middleware
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next) { _next = next; }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request: {context.Request.Path}");
        await _next(context); // call next middleware
        Console.WriteLine($"Response: {context.Response.StatusCode}");
    }
}

// Register in Program.cs
app.UseMiddleware<LoggingMiddleware>();
```

---

### 24. What is the difference between `IActionResult` and `ActionResult<T>`?

```csharp
// IActionResult — flexible, any response type
[HttpGet("{id}")]
public IActionResult GetUser(int id)
{
    var user = _repo.GetById(id);
    if (user == null) return NotFound();
    return Ok(user);
}

// ActionResult<T> — type-safe, better for Swagger docs
[HttpGet("{id}")]
public ActionResult<User> GetUser(int id)
{
    var user = _repo.GetById(id);
    if (user == null) return NotFound();
    return user; // implicit Ok()
}
```

---

## 🔷 Entity Framework

### 25. What is the difference between `eager loading`, `lazy loading`, and `explicit loading`?

```csharp
// Eager loading — loads related data immediately with Include()
var orders = dbContext.Orders
    .Include(o => o.Customer)
    .Include(o => o.Items)
    .ToList();

// Lazy loading — loads on access (requires virtual + proxy)
public class Order
{
    public virtual Customer Customer { get; set; } // loaded when accessed
}

// Explicit loading — manually load when needed
var order = dbContext.Orders.Find(1);
dbContext.Entry(order).Reference(o => o.Customer).Load();
```

---

### 26. What is N+1 problem in Entity Framework?

```csharp
// ❌ N+1 problem — 1 query for orders + N queries for each customer
var orders = dbContext.Orders.ToList();
foreach (var order in orders)
    Console.WriteLine(order.Customer.Name); // separate query each time!

// ✅ Fix with eager loading — just 1 query with JOIN
var orders = dbContext.Orders.Include(o => o.Customer).ToList();
```

---

## 🔷 Async & Threading

### 27. What is `async`/`await`?

```csharp
// ❌ Synchronous — blocks the thread
public string GetData()
{
    var result = httpClient.GetStringAsync(url).Result; // BLOCKS!
    return result;
}

// ✅ Async — frees thread while waiting
public async Task<string> GetDataAsync()
{
    var result = await httpClient.GetStringAsync(url); // doesn't block
    return result;
}
```

---

### 28. What is the difference between `Task.Run` and `async/await`?

```csharp
// Task.Run — for CPU-bound work (heavy computation)
var result = await Task.Run(() => HeavyCalculation());

// async/await — for I/O-bound work (network, database, file)
var data = await httpClient.GetStringAsync(url);
```

---

### 29. What is `deadlock` and how to avoid it?

```csharp
// ❌ Deadlock — .Result blocks thread, await can't resume
public void BadMethod()
{
    var result = GetDataAsync().Result; // DEADLOCK!
}

// ✅ Fix 1 — go async all the way
public async Task GoodMethod()
{
    var result = await GetDataAsync();
}

// ✅ Fix 2 — use ConfigureAwait(false) in library code
public async Task<string> GetDataAsync()
{
    return await httpClient.GetStringAsync(url).ConfigureAwait(false);
}
```

---

## 🔷 Memory & Performance

### 30. What is `IDisposable` and `using`?

```csharp
public class FileProcessor : IDisposable
{
    private StreamReader _reader;

    public FileProcessor(string path)
    {
        _reader = new StreamReader(path);
    }

    public void Dispose()
    {
        _reader?.Dispose();
    }
}

// usage — auto-disposes even if exception occurs
using (var processor = new FileProcessor("file.txt"))
{
    // use processor
}

// C# 8+ — even cleaner
using var processor = new FileProcessor("file.txt");
```

---

## 🔷 SQL & Database

### 31. What is the difference between `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`?

```sql
-- INNER JOIN — only matching rows from both tables
SELECT o.Id, c.Name
FROM Orders o
INNER JOIN Customers c ON o.CustomerId = c.Id;

-- LEFT JOIN — all rows from left + matching from right (null if no match)
SELECT o.Id, c.Name
FROM Orders o
LEFT JOIN Customers c ON o.CustomerId = c.Id;

-- RIGHT JOIN — all rows from right + matching from left
SELECT o.Id, c.Name
FROM Orders o
RIGHT JOIN Customers c ON o.CustomerId = c.Id;
```

---

### 32. What is an index and when to use it?

```sql
-- Create index on frequently queried column
CREATE INDEX IX_Users_Email ON Users(Email);

-- ✅ Use index when:
-- 1. Column is frequently used in WHERE clause
-- 2. Column is used in JOIN conditions
-- 3. Column is used in ORDER BY

-- ❌ Avoid index when:
-- 1. Table has very few rows
-- 2. Column has low cardinality (e.g., boolean)
-- 3. Table has frequent INSERT/UPDATE (index slows writes)
```

---

## 🔷 System Design

### 33. How would you design a URL shortener?

```
Components:
1. API Layer — POST /shorten → returns short URL
2. Database — store mapping: shortCode → originalUrl
3. Redirect Service — GET /{code} → 301 redirect

Short Code Generation:
- Base62 encoding (a-z, A-Z, 0-9)
- 6 characters = 62^6 = 56 billion unique URLs

Database Schema:
CREATE TABLE urls (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE,
    original_url TEXT,
    created_at DATETIME,
    click_count INT DEFAULT 0
);

Scaling:
- Cache hot URLs in Redis
- Use CDN for redirect service
- Horizontal scaling with load balancer
```

---

### 34. What is REST vs gRPC?

| | REST | gRPC |
|---|---|---|
| Protocol | HTTP/1.1 | HTTP/2 |
| Format | JSON | Protocol Buffers (binary) |
| Speed | Slower | Much faster |
| Browser support | ✅ Yes | ❌ Limited |
| Use case | Public APIs | Microservices |

---

## 🤝 Contributing

Found a bug or want to add more questions? PRs are welcome!

1. Fork the repo
2. Create a branch: `git checkout -b add-new-questions`
3. Add your questions with code examples
4. Submit a PR

---

## ⭐ Support

If this helped you crack an interview, please **star this repo** — it helps others find it too!

---

## 📄 License

MIT License — free to use, share, and modify.

---

*Made with ❤️ for the .NET community*