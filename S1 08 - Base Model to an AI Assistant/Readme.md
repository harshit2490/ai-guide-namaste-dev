# Chapter 2 — JS on the Server &nbsp; <sup>[⬆ Back to Table of Contents](../README.md#part-1)</sup>

> **Season 1** | Part I — Node.js Fundamentals & Modules

---

### What is "JS on the Server"?

**"JS on the Server"** means running JavaScript outside the browser — on a machine that serves data, files, and APIs to clients. Before Node.js, JavaScript was confined to the browser. Node.js changed that by pairing the **V8 engine** with **libuv**, giving JavaScript access to the file system, network, and OS — enabling it to power backend servers.

> 💡 The same language you use for DOM manipulation can now read files, query databases, and serve HTTP responses.

### How It Works

#### Browser JS vs Server JS (Node.js)

The JavaScript **language** is the same in both environments, but the **runtime APIs** available are completely different:

| Feature           | Browser JS                          | Node.js (Server JS)                        |
| ----------------- | ----------------------------------- | ------------------------------------------ |
| **Engine**        | V8 (Chrome), SpiderMonkey (Firefox) | V8                                         |
| **Global Object** | `window`                            | `global` / `globalThis`                    |
| **DOM APIs**      | ✅ `document`, `window`, `alert()`  | ❌ No DOM at all                           |
| **File System**   | ❌ Sandboxed, no access             | ✅ `fs` module (read, write, delete files) |
| **HTTP Server**   | ❌ Cannot create servers            | ✅ `http.createServer()`                   |
| **Networking**    | `fetch`, `XMLHttpRequest`           | `http`, `https`, `net`, `dgram` modules    |
| **Module System** | ES Modules (`import`/`export`)      | CommonJS (`require`) + ES Modules          |
| **Console**       | Browser DevTools console            | Terminal / Command-line output             |
| **Timers**        | `setTimeout`, `setInterval`         | Same + `setImmediate`, `process.nextTick`  |
| **OS Access**     | ❌ Completely sandboxed             | ✅ `os`, `child_process`, `path` modules   |
| **Use Case**      | UI rendering, user interactions     | APIs, servers, CLI tools, scripting        |

#### The Global Object

Every JavaScript runtime provides a **global object** — the top-level scope container. It differs per environment:

```javascript
// In Browser:
console.log(window); // The global object
console.log(this === window); // true (at global scope)

// In Node.js:
console.log(global); // The global object
console.log(this === global); // false (in modules, `this` === module.exports)
```

| Runtime         | Global Object         | `this` at Top Level                    |
| --------------- | --------------------- | -------------------------------------- |
| **Browser**     | `window`              | `this === window`                      |
| **Node.js**     | `global`              | `this === module.exports` (in modules) |
| **Web Workers** | `self`                | `this === self`                        |
| **Universal**   | `globalThis` (ES2020) | Works everywhere                       |

```javascript
// ✅ Universal way to access the global object (ES2020+)
console.log(globalThis); // Works in Browser, Node.js, Workers — everywhere
```

> 💡 **`globalThis`** was introduced in ES2020 specifically to solve the fragmentation of `window` vs `global` vs `self`. Always prefer `globalThis` for cross-platform code.

#### What Node.js Adds to JavaScript

Node.js extends the V8 engine with C++ bindings to provide APIs that JavaScript alone cannot offer:

| API Category      | Module / Object    | What It Does                                         |
| ----------------- | ------------------ | ---------------------------------------------------- |
| **File System**   | `fs`               | Read, write, delete, watch files and directories     |
| **HTTP**          | `http`, `https`    | Create web servers and make HTTP requests            |
| **Path**          | `path`             | Manipulate file and directory paths cross-platform   |
| **OS**            | `os`               | Get system info — CPU, memory, platform, hostname    |
| **Process**       | `process` (global) | Current process info, env vars, CLI args, exit codes |
| **Events**        | `events`           | Create and handle custom events (EventEmitter)       |
| **Child Process** | `child_process`    | Spawn new processes, execute shell commands          |
| **Streams**       | `stream`           | Handle data in chunks (readable, writable, duplex)   |

#### Servers in Node.js

A **server** is a system that listens for incoming **client requests** and sends back **responses** — typically over the HTTP protocol.

Node.js servers are efficient because of the **event-driven, non-blocking I/O** model:

```
Traditional Server (Apache/PHP):
────────────────────────────────
Request 1 → Thread 1 → [BLOCKING: wait for DB] → Response 1
Request 2 → Thread 2 → [BLOCKING: wait for File] → Response 2
Request 3 → ❌ No threads available → WAIT

Node.js Server:
────────────────────────────────
Request 1 ─┐
Request 2 ─┼──→ Single Thread (Event Loop) → Delegates I/O → Handles callbacks
Request 3 ─┘                                   when operations complete
```

| Aspect                | Traditional (Thread-per-request) | Node.js (Event Loop)          |
| --------------------- | -------------------------------- | ----------------------------- |
| **Concurrency model** | One thread per request           | Single thread + Event Loop    |
| **I/O handling**      | Blocking — thread waits          | Non-blocking — delegates I/O  |
| **Memory usage**      | High (each thread ~2MB)          | Low (one thread + callbacks)  |
| **Best for**          | CPU-intensive work               | I/O-intensive, real-time apps |

#### The V8 Engine Pipeline

The **V8 engine** converts JavaScript from human-readable code to machine code in multiple stages:

```
  JavaScript Source Code
          │
          ▼
  ┌───────────────┐
  │    Parser      │   Reads code, checks syntax, tokenizes
  └───────┬───────┘
          │
          ▼
  ┌───────────────┐
  │      AST       │   Abstract Syntax Tree — structured representation
  └───────┬───────┘
          │
          ▼
  ┌───────────────┐
  │   Ignition     │   V8's Interpreter — generates Bytecode
  │  (Interpreter) │   Executes code quickly on first run
  └───────┬───────┘
          │
      Hot code detected? (frequently executed)
          │
          ▼
  ┌───────────────┐
  │   TurboFan     │   V8's Optimizing Compiler
  │  (Compiler)    │   Compiles hot bytecode → optimized Machine Code
  └───────┬───────┘
          │
          ▼
    Machine Code (CPU executes directly)
```

| Stage          | Component | What Happens                                                 |
| -------------- | --------- | ------------------------------------------------------------ |
| **Parsing**    | Parser    | Reads JS, checks syntax, builds tokens                       |
| **AST**        | Parser    | Creates a tree structure of the code                         |
| **Bytecode**   | Ignition  | Interprets AST → bytecode for quick first execution          |
| **Optimize**   | TurboFan  | Identifies "hot" functions and compiles them to machine code |
| **Deoptimize** | TurboFan  | If assumptions break, falls back to bytecode (Ignition)      |

> ⚠️ V8's full architecture is covered in depth in **Chapter 8 — Deep dive into V8 JS Engine**. This is an introductory overview.

### Code Example

#### Running JavaScript via Terminal

```bash
# Check if Node.js is installed
node --version
# v20.11.0

# Run a JavaScript file
node app.js

# Run inline JavaScript
node -e "console.log('Hello from Node.js!')"
```

#### Your First Node.js Script

```javascript
// app.js — Run with: node app.js

// ✅ These work in Node.js (server-side APIs)
const os = require("os");
const path = require("path");

console.log("=== JS on the Server ===");
console.log("Platform:", os.platform()); // win32, linux, darwin
console.log("CPU Cores:", os.cpus().length); // e.g., 8
console.log("Free Memory:", (os.freemem() / 1e9).toFixed(2), "GB");
console.log("Home Dir:", os.homedir());
console.log("File Extension:", path.extname("app.js")); // .js

// ❌ These would FAIL in Node.js (browser-only APIs)
// document.getElementById("app");   → ReferenceError: document is not defined
// window.alert("Hello");            → ReferenceError: window is not defined
// localStorage.setItem("key", "v"); → ReferenceError: localStorage is not defined
```

**Output:**

```
=== JS on the Server ===
Platform: win32
CPU Cores: 8
Free Memory: 5.23 GB
Home Dir: C:\Users\harshit
File Extension: .js
```

#### Browser-Only vs Node-Only APIs

```javascript
// ===== Browser-Only APIs (will FAIL in Node.js) =====
// document.querySelector(".box");     ❌ No DOM
// window.innerWidth;                  ❌ No window
// alert("Hello");                     ❌ No UI

// ===== Node-Only APIs (will FAIL in Browser) =====
const fs = require("fs"); // ✅ File system access
const http = require("http"); // ✅ Create servers
const crypto = require("crypto"); // ✅ Cryptography

// ===== Available in BOTH =====
console.log("Works everywhere"); // ✅
setTimeout(() => {}, 1000); // ✅
JSON.stringify({ a: 1 }); // ✅
Promise.resolve("ok"); // ✅
```

#### The `global` Object in Node.js

```javascript
// In Node.js, `global` is the global object (like `window` in browser)
console.log(global === globalThis); // true

// Properties on global are available everywhere without require
console.log(typeof setTimeout); // "function"  (on global)
console.log(typeof console); // "object"    (on global)
console.log(typeof process); // "object"    (on global)

// BUT unlike browser, `this` at module level is NOT `global`
console.log(this === global); // false
console.log(this === module.exports); // true (in CommonJS modules)
```

#### A Minimal HTTP Server

```javascript
const http = require("http");

const server = http.createServer((req, res) => {
  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Hello from Node.js Server!");
});

server.listen(3000, () => {
  console.log("Server running at http://localhost:3000");
});
```

```bash
node server.js
# Server running at http://localhost:3000
# Open browser → http://localhost:3000 → "Hello from Node.js Server!"
```

> 💡 This is a preview — **Chapter 11** covers `http.createServer()` in full detail.

### Common Mistakes

| Mistake                                                    | Why It's Wrong                                                                                                              |
| ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| "Browser JS and Node.js are different languages"           | ❌ Same language (ECMAScript). The difference is in the **runtime APIs** — DOM APIs in browser, `fs`/`http` in Node.js      |
| "Node.js has `window` and `document`"                      | ❌ Those are **browser-only** globals. Node.js uses `global` / `globalThis` and has no DOM                                  |
| "`this` at the top level in Node.js equals `global`"       | ❌ In Node.js modules (CommonJS), `this` at the top level equals `module.exports`, **not** `global`                         |
| "V8 interprets JavaScript like Python"                     | ❌ V8 uses **JIT compilation** — it interprets first (Ignition) then compiles hot code to machine code (TurboFan)           |
| "Node.js can only be used for web servers"                 | ❌ Node.js powers CLI tools, desktop apps (Electron), real-time apps, IoT, build tools (Webpack, Vite), scripting, and more |
| "You need to install `console` or `setTimeout` in Node.js" | ❌ These are **globals** — available on the `global` object without any `require()` call                                    |

<div style="font-size: 22px; color: red">
<details>
  <summary><strong>Interview Questions (Click to View)</strong></summary>
  <div style="font-size: 0.9rem; color: black; background:#fff; border:2px solid red; border-radius: 10px;">

- **Q: What does "JS on the Server" mean?**
  - A: It means running JavaScript outside the browser — on a server machine — using Node.js. Node.js provides APIs for file system access, networking, HTTP servers, and OS interaction that JavaScript alone (in a browser) cannot do.

- **Q: What is the difference between Browser JS and Node.js?**
  - A: Both run the same JavaScript language, but the runtime APIs differ. Browsers provide DOM APIs (`document`, `window`, `alert`), while Node.js provides server-side APIs (`fs`, `http`, `os`, `path`). The global object is `window` in browsers and `global`/`globalThis` in Node.js.

- **Q: What is the global object in Node.js?**
  - A: The `global` object. It holds all globally available functions like `setTimeout`, `console`, and `process`. Unlike browsers, `this` at the module level in Node.js equals `module.exports`, not `global`. Use `globalThis` for cross-platform consistency.

- **Q: How does Node.js execute JavaScript code?**
  - A: Node.js uses the **V8 engine** which: (1) Parses JS code into an AST, (2) Ignition interpreter generates bytecode for fast first execution, (3) TurboFan compiler optimizes frequently-run ("hot") code into machine code via JIT compilation.

- **Q: What is JIT compilation?**
  - A: JIT (Just-In-Time) compilation is a technique where code is compiled to machine code **at runtime** rather than ahead of time. V8's Ignition interpreter first runs code as bytecode, then TurboFan compiles hot paths into optimized machine code for maximum speed.

- **Q: What is the V8 engine?**
  - A: V8 is Google's open-source JavaScript engine written in C++. It powers both Chrome and Node.js. It compiles JavaScript directly to machine code using JIT compilation (Ignition interpreter + TurboFan optimizing compiler).

- **Q: How does Node.js handle multiple requests with a single thread?**
  - A: Through its **event-driven, non-blocking I/O** model. Instead of creating a thread per request (like Apache), Node.js uses a single-threaded Event Loop that delegates I/O operations to libuv. When operations complete, callbacks are executed. This allows handling thousands of concurrent connections with minimal overhead.

- **Q: What is `globalThis` and why was it introduced?**
  - A: `globalThis` (ES2020) is a standardized way to access the global object across all JavaScript environments — `window` in browsers, `global` in Node.js, `self` in Web Workers. It was introduced to eliminate the fragmentation of different global object names.

- **Q: Can you use `require()` in the browser?**
  - A: No. `require()` is a CommonJS feature specific to Node.js. Browsers use ES Modules (`import`/`export`). However, bundlers like Webpack and Vite can transform `require()` calls for browser use during build time.

- **Q: What is the difference between `window` and `global`?**
  - A: `window` is the browser's global object — it provides DOM APIs, browser history, location, etc. `global` is Node.js's global object — it provides server-side APIs. Both serve as the top-level scope container, but their available APIs are completely different.

    </div>
  </details>
  </div>

### Key Takeaways

- **Same language, different runtime**: Browser JS and Node.js both execute JavaScript (ECMAScript) but provide completely different APIs
- The browser's global object is `window`; Node.js uses `global`. Use **`globalThis`** for universal code
- In Node.js modules, `this` at the top level equals `module.exports` — **not** `global`
- Node.js extends V8 with **libuv** bindings for file system, networking, OS access, and async I/O
- V8 compiles JS via a pipeline: **Source → Parser → AST → Ignition (Bytecode) → TurboFan (Machine Code)**
- Node.js handles high concurrency via **single-threaded Event Loop + non-blocking I/O** — not thread-per-request
- Run JavaScript files from the terminal with `node filename.js` — no browser needed
- Browser-only APIs (`document`, `window`, `alert`) do **not** exist in Node.js — and vice versa (`fs`, `http`, `os`)

---

<div align="center">

|                                         ← Previous                                         | [📑 Table of Contents](../README.md#part-1) |                                       Next →                                       |
| :----------------------------------------------------------------------------------------: | :-----------------------------------------: | :--------------------------------------------------------------------------------: |
| [Chapter 1: Introduction to NodeJs](../S1%2001%20-%20Introduction%20to%20NodeJs/Readme.md) |                                             | [Chapter 3: Writing First Code](../S1%2003%20-%20Writing%20First%20Code/Readme.md) |

</div>
