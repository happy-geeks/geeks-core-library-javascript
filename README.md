# GeeksCoreLibrary.Modules.ScriptInterpreters.JavaScript

## Repository Metadata
- **Type**: Library
- **Language(s)**: C#
- **Framework(s)**: .NET 9.0
- **Database**: None

## Purpose & Context

### What This Repository Does
This repository provides a JavaScript interpreter plugin for the GeeksCoreLibrary's Script Interpreters module. It enables server-side execution of JavaScript code within .NET applications, specifically for the Wiser CMS ecosystem. The library wraps the Jint JavaScript engine and exposes a clean, type-safe API for executing JavaScript code, managing state, and invoking functions from C# applications.

### Business Value
The JavaScript interpreter allows Wiser applications to:
- Execute dynamic business logic stored in the database without redeployment
- Provide customers with scriptable configuration and validation rules
- Enable template-based content generation with JavaScript expressions
- Support complex calculations and transformations in product configurators
- Allow non-developers to customize application behavior through safe script execution

This is particularly valuable for Wiser's multi-tenant architecture where each customer may require different business rules and calculations without modifying the core platform.

### Wiser Ecosystem Role
This library is a **plugin module** for GeeksCoreLibrary and serves as:
- A dependency for Wiser core applications that need JavaScript execution capabilities
- Part of the Script Interpreters module family (alongside potential Python, Lua, or other language interpreters)
- A bridge between database-stored templates/scripts and runtime execution
- A utility library used by product configurators, template engines, and validation systems

## Core Functionality

### Main Components

#### IJavaScriptService (Public Interface)

**State Management:**
- `SetValue(string name, object value)` - Sets a global variable in the JavaScript engine
- `GetValue(string name)` - Retrieves a JavaScript variable as an object
- `GetValue<T>(string name)` - Generic version with automatic type conversion

**Script Execution:**
- `Execute(string script)` - Executes JavaScript statements without returning a value
- `Evaluate(string script)` - Evaluates a JavaScript expression and returns the result
- `Evaluate<T>(string script)` - Generic version with automatic type conversion and error handling

**Function Invocation:**
- `Invoke(string functionName, params object[] arguments)` - Calls a JavaScript function by name
- `Invoke<T>(string functionName, params object[] arguments)` - Generic version with type conversion

#### JavaScriptService (Core Implementation)

Implements HTML-to-PDF conversion using the Jint engine with security constraints:
- **Security**: Strict mode + `StringCompilationAllowed = false` (disables eval)
- **GCL Integration**: Exposes utility functions via global `GCL` object:
  - `GCL.ConvertToSeo(text)` - Converts text to SEO-friendly URL slug
  - `GCL.EncryptWithAes(...)` / `GCL.DecryptWithAes(...)` - AES encryption
  - `GCL.ToSha512(text)` - SHA-512 hashing

## Installation & Setup

### Prerequisites
- **.NET 9.0 SDK** or later
- **GeeksCoreLibrary** v5.3.2508.1 or compatible
- **Jint** v4.2.2 (automatically installed)

### Installation

**Install NuGet package:**
```bash
dotnet add package GeeksCoreLibrary.Modules.ScriptInterpreters.JavaScript
```

**Registration** (automatic via `ITransientService` marker):
```csharp
// In consuming application - no registration needed
// GeeksCoreLibrary auto-registers ITransientService implementations

public class MyController : Controller
{
    private readonly IJavaScriptService _jsService;

    public MyController(IJavaScriptService jsService)
    {
        _jsService = jsService; // Automatically injected
    }
}
```

### Basic Usage

```csharp
using var jsService = serviceProvider.GetService<IJavaScriptService>();

// Execute and evaluate simple expression
var result = jsService.Evaluate<int>("2 + 2"); // Returns: 4

// Set values and use them
jsService.SetValue("name", "Wiser");
jsService.Execute("var greeting = 'Hello, ' + name + '!';");
var greeting = jsService.GetValue<string>("greeting"); // "Hello, Wiser!"

// Invoke functions
jsService.Execute("function multiply(a, b) { return a * b; }");
var product = jsService.Invoke<int>("multiply", 6, 7); // Returns: 42

// Use GCL utilities from JavaScript
jsService.Execute("var seoUrl = GCL.ConvertToSeo('Hello World! 2024');");
var seoUrl = jsService.GetValue<string>("seoUrl"); // "hello-world-2024"
```

## Troubleshooting

### Common Issues

**`eval is not allowed` Error:**
- **Cause**: Script uses `eval()` or `new Function()` (disabled for security)
- **Solution**: Rewrite logic without dynamic code generation

**ES6+ Syntax Errors:**
- **Cause**: Jint only supports ECMAScript 5.1
- **Solution**: Rewrite using ES5 syntax (no arrow functions, `const`, `let`, etc.)

**`InvalidCastException` When Getting Values:**
- **Cause**: Type mismatch between JavaScript and .NET
- **Solution**: Check JavaScript return type matches requested C# type

## Known Issues & Technical Debt

- [ ] **No Unit Tests** - Repository has no test project (high priority)
- [ ] **No Script Execution Timeout** - Scripts can run indefinitely (security concern)
- [ ] **Limited Error Context** - Error logs don't include full script content

---

*This documentation is part of the Wiser Platform knowledge base maintained by Happy Horizon B.V.*