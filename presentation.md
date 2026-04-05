# Introduction to Node.js

**Server-Side JavaScript & the Event-Driven Runtime**

V8 | event loop | npm | async I/O | streams | APIs

---

## Table of Contents

1. [Agenda](#slide-02--agenda)
2. [What Is Node.js?](#slide-03--what-is-nodejs)
3. [Runtime Architecture](#slide-04--runtime-architecture)
4. [The Event Loop](#slide-05--the-event-loop)
5. [Blocking vs Non-Blocking I/O](#slide-06--blocking-vs-non-blocking-io)
6. [Modules & the Module System](#slide-07--modules--the-module-system)
7. [npm & Package Management](#slide-08--npm--package-management)
8. [Async Patterns: Callbacks, Promises, async/await](#slide-09--async-patterns-callbacks--promises--asyncawait)
9. [Streams & Buffers](#slide-10--streams--buffers)
10. [Building an HTTP Server](#slide-11--building-an-http-server)
11. [Express.js & Middleware](#slide-12--expressjs--middleware)
12. [File System & Path](#slide-13--file-system--path)
13. [Environment Variables & Configuration](#slide-14--environment-variables--configuration)
14. [Error Handling Patterns](#slide-15--error-handling-patterns)
15. [Testing with Jest & Supertest](#slide-16--testing-with-jest--supertest)
16. [Security Best Practices](#slide-17--security-best-practices)
17. [Performance & Scaling](#slide-18--performance--scaling)
18. [The Node.js Ecosystem](#slide-19--the-nodejs-ecosystem)
19. [Summary & Next Steps](#slide-20--summary--next-steps)

---

## Slide 02 — Agenda

### Foundations

- What is Node.js & why it exists
- V8, libuv, and the runtime architecture
- The event loop in detail
- Blocking vs non-blocking I/O

### Core Concepts

- Modules & the require/import system
- npm & package management
- Callbacks, Promises, async/await
- Streams & Buffers

### Building Things

- HTTP servers from scratch
- Express.js & middleware
- File system & path operations
- Environment variables & config

### Production

- Error handling patterns
- Testing with Jest & Supertest
- Security best practices
- Performance & scaling

---

## Slide 03 — What Is Node.js?

Node.js is an **open-source, cross-platform JavaScript runtime** built on Chrome's V8 engine. Created by **Ryan Dahl** in 2009, it lets developers run JavaScript outside the browser.

The key insight: most server workloads are **I/O-bound**, not CPU-bound. Traditional thread-per-request models waste resources waiting for disk, network, and database responses. Node uses a single-threaded **event-driven** model instead.

### Node.js is NOT

- A programming language (it runs JavaScript)
- A web framework (Express, Fastify, etc. are frameworks)
- Single-threaded for everything (worker threads exist)

### Key Properties

- **Event-driven** — non-blocking I/O
- **V8 engine** — JIT-compiled, fast
- **npm** — largest package ecosystem
- **Cross-platform** — Linux, macOS, Windows
- **Full-stack JS** — same language front & back

### Who Uses Node?

Netflix, LinkedIn, PayPal, Uber, NASA, Walmart, Trello, and millions of startups. npm serves 2+ billion package downloads per week.

---

## Slide 04 — Runtime Architecture

Node.js is built on two core components: Google's **V8** JavaScript engine and **libuv**, a cross-platform async I/O library written in C.

```
┌──────────────────────────────────────────────────┐
│              Your JavaScript Code                │
├──────────────────────────────────────────────────┤
│   Node.js APIs & Bindings (http, fs, net, crypto, stream ...)   │
├────────────────────────┬─────────────────────────┤
│       V8 Engine        │         libuv            │
│  Parses & compiles JS  │  Event loop, thread pool,│
│  → machine code        │  async I/O               │
├────────────────────────┴─────────────────────────┤
│     Operating System (epoll / kqueue / IOCP)     │
└──────────────────────────────────────────────────┘
```

### V8

Google's JS engine. JIT compiles JavaScript to optimised native machine code. Manages heap, garbage collection, and inline caches.

### libuv

Provides the event loop, async TCP/UDP sockets, file system ops, child processes, and a thread pool (default 4 threads) for blocking work.

### Node Bindings

C++ glue layer that connects JavaScript to libuv and OS-level APIs. Exposed to user code as built-in modules like `fs`, `http`, `net`.

---

## Slide 05 — The Event Loop

The event loop is the heart of Node.js. It continuously checks for pending work and dispatches callbacks. Each iteration is called a **tick**.

```
┌─────────┐   ┌────────────┐   ┌──────────────┐   ┌──────┐   ┌───────┐   ┌───────┐
│ timers  │ → │ pending I/O│ → │ idle/prepare │ → │ poll │ → │ check │ → │ close │
└─────────┘   └────────────┘   └──────────────┘   └──────┘   └───────┘   └───────┘
     ↑                                                                        │
     └────────────────────────── (loop back) ─────────────────────────────────┘
```

### timers

Executes callbacks from `setTimeout` and `setInterval` whose threshold has elapsed.

### poll

Retrieves new I/O events. Executes I/O-related callbacks (almost all except close, timers, and `setImmediate`).

### check

Executes `setImmediate()` callbacks. Runs immediately after the poll phase completes.

### Microtasks: process.nextTick() & Promises

Microtasks run *between* every phase. `process.nextTick()` fires before Promise callbacks. Starving the event loop with recursive `nextTick` is a common anti-pattern.

---

## Slide 06 — Blocking vs Non-Blocking I/O

### Blocking (synchronous)

The thread waits until the operation completes. No other work can happen.

```javascript
const fs = require('fs');

// Thread blocks here until file is fully read
const data = fs.readFileSync('/var/log/app.log');
console.log(data.length);
// Nothing else runs until readFileSync returns
```

### Non-blocking (asynchronous)

The call returns immediately. A callback fires when the operation finishes.

```javascript
const fs = require('fs');

// Returns immediately — callback fires later
fs.readFile('/var/log/app.log', (err, data) => {
  if (err) throw err;
  console.log(data.length);
});
// Event loop is free to handle other requests
```

### Why This Matters

A single blocking call on the main thread **freezes the entire server**. With 10,000 concurrent connections, one `readFileSync` blocks all of them. Non-blocking I/O lets Node handle thousands of concurrent connections with a single thread, because the event loop delegates waiting to the OS and libuv thread pool.

### Rule of Thumb

Never use `*Sync` methods in server code. They're acceptable in CLI scripts and startup config loading.

### Thread Pool

libuv uses 4 threads (configurable via `UV_THREADPOOL_SIZE`) for inherently blocking ops: DNS lookups, file system, zlib, crypto.

### Network I/O

TCP/UDP is handled by OS-level async primitives (epoll, kqueue, IOCP) — no thread pool needed.

---

## Slide 07 — Modules & the Module System

Node has two module systems: the original **CommonJS** (`require`) and the newer **ES Modules** (`import`). Both coexist in modern Node.

### CommonJS (CJS)

```javascript
// math.js
function add(a, b) { return a + b; }
module.exports = { add };

// app.js
const { add } = require('./math');
console.log(add(2, 3)); // 5
```

- Synchronous loading
- Modules cached after first `require()`
- `.js` extension — default in Node
- Dynamic: can `require()` inside conditionals

### ES Modules (ESM)

```javascript
// math.mjs
export function add(a, b) { return a + b; }

// app.mjs
import { add } from './math.mjs';
console.log(add(2, 3)); // 5
```

- Asynchronous loading
- Static analysis & tree-shaking friendly
- `.mjs` or `"type": "module"` in package.json
- Top-level `await` supported

### Module Resolution Order

`require('foo')` → 1. Core module? 2. `./node_modules/foo`? 3. Parent `node_modules`? ... walks up to root. Each resolved module is cached by its absolute path.

---

## Slide 08 — npm & Package Management

**npm** (Node Package Manager) ships with every Node installation. It manages dependencies, scripts, and publishing. The registry hosts **2.1 million+** packages.

```bash
# Initialise a new project
npm init -y

# Install dependencies
npm install express        # production dep
npm install -D jest        # dev dependency

# Key files
package.json               # manifest & scripts
package-lock.json          # exact dependency tree
node_modules/              # installed packages
```

### Scripts

```json
{
  "scripts": {
    "start": "node server.js",
    "dev": "node --watch server.js",
    "test": "jest --coverage",
    "lint": "eslint ."
  }
}
```

### Semantic Versioning

**MAJOR**.**MINOR**.**PATCH**

- `^1.2.3` — allow minor + patch updates
- `~1.2.3` — allow patch updates only
- `1.2.3` — exact version

### Lock Files

`package-lock.json` pins the exact version of every transitive dependency. **Always commit it.** Ensures reproducible installs across machines.

### Alternatives

- **yarn** — parallel installs, workspaces
- **pnpm** — content-addressable store, fast, strict
- **corepack** — ships with Node, manages package managers

---

## Slide 09 — Async Patterns: Callbacks → Promises → async/await

### Callbacks (legacy)

Error-first convention. Nesting leads to "callback hell".

```javascript
fs.readFile('a.txt', (err, a) => {
  if (err) return cb(err);
  fs.readFile('b.txt', (err, b) => {
    if (err) return cb(err);
    fs.writeFile('c.txt', a + b, cb);
  });
});
```

### Promises

Chainable `.then()` / `.catch()`. Flat structure.

```javascript
const fsp = require('fs').promises;

fsp.readFile('a.txt')
  .then(a => fsp.readFile('b.txt')
    .then(b => fsp.writeFile('c.txt', a + b)))
  .catch(err => console.error(err));
```

### async / await

Syntactic sugar over Promises. Reads like sync code.

```javascript
const fsp = require('fs').promises;

async function merge() {
  const a = await fsp.readFile('a.txt');
  const b = await fsp.readFile('b.txt');
  await fsp.writeFile('c.txt', a + b);
}
merge().catch(console.error);
```

### Parallel Execution

```javascript
// Run independent tasks concurrently
const [users, orders] = await Promise.all([
  db.query('SELECT * FROM users'),
  db.query('SELECT * FROM orders')
]);
```

### Error Handling

```javascript
try {
  const data = await fetchData();
} catch (err) {
  // Handle or rethrow
  logger.error('Fetch failed', err);
  throw err;
}
```

---

## Slide 10 — Streams & Buffers

Streams process data in **chunks** rather than loading everything into memory. Essential for handling large files, network data, and real-time processing.

| Stream Type   | Example                | Use                  |
|---------------|------------------------|----------------------|
| **Readable**  | `fs.createReadStream`  | Read from source     |
| **Writable**  | `fs.createWriteStream` | Write to destination |
| **Duplex**    | `net.Socket`           | Both read & write    |
| **Transform** | `zlib.createGzip`      | Modify data in transit |

### Buffers

Fixed-size chunks of raw binary data. Allocated outside the V8 heap. Used for file I/O, network protocols, and cryptography.

```javascript
const buf = Buffer.from('Hello, Node!');
console.log(buf.toString('hex'));
// 48656c6c6f2c204e6f646521
```

### Streaming Example

```javascript
const fs = require('fs');
const zlib = require('zlib');

// Compress a large file using streams
// Memory usage stays low regardless of file size
fs.createReadStream('input.log')       // Readable
  .pipe(zlib.createGzip())             // Transform
  .pipe(fs.createWriteStream('input.log.gz')) // Writable
  .on('finish', () => console.log('Done!'));
```

### pipeline() — the modern way

```javascript
const { pipeline } = require('stream/promises');

await pipeline(
  fs.createReadStream('input.log'),
  zlib.createGzip(),
  fs.createWriteStream('input.log.gz')
);
// Automatically handles errors & cleanup
```

---

## Slide 11 — Building an HTTP Server

Node's built-in `http` module can create servers with zero external dependencies. This is what frameworks like Express build on top of.

### Bare Node.js

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  // req = IncomingMessage (Readable stream)
  // res = ServerResponse  (Writable stream)

  if (req.method === 'GET' && req.url === '/api/health') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ status: 'ok' }));
  } else {
    res.writeHead(404);
    res.end('Not Found');
  }
});

server.listen(3000, () => {
  console.log('Listening on http://localhost:3000');
});
```

### What the http Module Gives You

- TCP socket management
- HTTP/1.1 parsing (via llhttp, written in C)
- Keep-alive connection pooling
- Request/response streaming
- HTTPS via `https.createServer()` with TLS certs

### What It Doesn't Give You

- Routing (no path matching / params)
- Body parsing (must read the stream yourself)
- Middleware pipeline
- Static file serving
- Cookie / session handling

This is why frameworks exist.

---

## Slide 12 — Express.js & Middleware

```javascript
const express = require('express');
const app = express();

// Built-in middleware
app.use(express.json());        // parse JSON bodies
app.use(express.static('public'));

// Custom middleware — runs on every request
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();  // pass control to next handler
});

// Route handlers
app.get('/api/users', async (req, res) => {
  const users = await db.findAll();
  res.json(users);
});

app.post('/api/users', async (req, res) => {
  const user = await db.create(req.body);
  res.status(201).json(user);
});

// Error-handling middleware (4 params)
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Server error' });
});

app.listen(3000);
```

### The Middleware Pattern

Middleware are functions with signature `(req, res, next)`. They execute in order and form a pipeline. Each can modify the request/response, end the cycle, or call `next()`.

```
[ logger ] → next() → [ auth ] → next() → [ handler ]
```

### Popular Middleware

- **cors** — Cross-Origin Resource Sharing
- **helmet** — security headers
- **morgan** — HTTP request logging
- **express-rate-limit** — rate limiting
- **passport** — authentication strategies

### Alternatives to Express

**Fastify** (schema-based, 2-3x faster), **Koa** (async-native, minimal), **Hono** (ultrafast, edge-ready), **NestJS** (opinionated, TypeScript-first).

---

## Slide 13 — File System & Path

### fs/promises API (modern)

```javascript
const fs = require('fs/promises');
const path = require('path');

async function processConfig() {
  // Read
  const raw = await fs.readFile('config.json', 'utf8');
  const config = JSON.parse(raw);

  // Write
  config.updatedAt = new Date().toISOString();
  await fs.writeFile(
    'config.json',
    JSON.stringify(config, null, 2)
  );

  // Directory operations
  await fs.mkdir('logs', { recursive: true });
  const files = await fs.readdir('logs');

  // File info
  const stats = await fs.stat('config.json');
  console.log(`Size: ${stats.size} bytes`);
}
```

### path module — cross-platform paths

```javascript
const path = require('path');

path.join('src', 'utils', 'db.js');
// Linux:   'src/utils/db.js'
// Windows: 'src\\utils\\db.js'

path.resolve('..', 'config.json');
// '/home/user/project/config.json'

path.basename('/app/server.js');   // 'server.js'
path.extname('photo.png');         // '.png'
path.dirname('/app/src/index.js'); // '/app/src'

// Common pattern: resolve relative to this file
const configPath = path.join(__dirname, 'config.json');
```

### Watch for Changes

```javascript
const watcher = fs.watch('./src', { recursive: true });
for await (const event of watcher) {
  console.log(event.eventType, event.filename);
}
```

---

## Slide 14 — Environment Variables & Configuration

Environment variables separate **config from code**. The 12-Factor App methodology recommends storing all config in env vars.

```bash
# .env (never commit this file!)
PORT=3000
DATABASE_URL=postgres://user:pass@localhost/mydb
JWT_SECRET=super-secret-key
NODE_ENV=production
```

```javascript
// Access in Node.js
const port = process.env.PORT || 3000;
const dbUrl = process.env.DATABASE_URL;

// Since Node 20.6 — built-in .env support
// node --env-file=.env server.js
```

### Security Rules

- Never commit `.env` files — add to `.gitignore`
- Use different secrets per environment
- Rotate secrets regularly
- Use a vault in production (AWS Secrets Manager, HashiCorp Vault)

### NODE_ENV

The most important env var. Conventional values:

- **development** — verbose logging, stack traces
- **production** — optimised, minimal logging
- **test** — test databases, mocked services

Express and many libraries change behaviour based on this value. In production, Express caches view templates and generates less verbose errors.

### process Object Highlights

- `process.env` — environment variables
- `process.argv` — command-line arguments
- `process.cwd()` — current working directory
- `process.pid` — process ID
- `process.exit(code)` — terminate the process
- `process.memoryUsage()` — heap statistics

---

## Slide 15 — Error Handling Patterns

### Operational vs Programmer Errors

- **Operational** — expected failures: network timeout, invalid user input, file not found. Handle gracefully.
- **Programmer** — bugs: undefined is not a function, wrong type. Fix the code.

### Express async error wrapper

```javascript
const asyncHandler = (fn) => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

app.get('/users/:id', asyncHandler(async (req, res) => {
  const user = await db.findById(req.params.id);
  if (!user) {
    const err = new Error('User not found');
    err.statusCode = 404;
    throw err;
  }
  res.json(user);
}));
```

### Global error handler middleware

```javascript
app.use((err, req, res, next) => {
  const status = err.statusCode || 500;
  const message = status === 500
    ? 'Internal Server Error'
    : err.message;

  if (status === 500) logger.error(err);

  res.status(status).json({ error: message });
});
```

### Process-Level Safety Nets

```javascript
// Catch unhandled promise rejections
process.on('unhandledRejection', (reason) => {
  logger.fatal('Unhandled rejection', reason);
  process.exit(1); // let process manager restart
});

// Catch uncaught exceptions
process.on('uncaughtException', (err) => {
  logger.fatal('Uncaught exception', err);
  process.exit(1);
});
```

### Graceful Shutdown

On `SIGTERM`, stop accepting new connections, finish in-flight requests, close database pools, then exit. Process managers (PM2, Kubernetes) send `SIGTERM` before force-killing.

---

## Slide 16 — Testing with Jest & Supertest

### Unit Test

```javascript
// utils.js
function slugify(text) {
  return text.toLowerCase()
    .replace(/[^a-z0-9]+/g, '-')
    .replace(/^-|-$/g, '');
}
module.exports = { slugify };

// utils.test.js
const { slugify } = require('./utils');

describe('slugify', () => {
  test('converts spaces to hyphens', () => {
    expect(slugify('Hello World')).toBe('hello-world');
  });

  test('strips special characters', () => {
    expect(slugify('Node.js & npm!'))
      .toBe('node-js-npm');
  });

  test('handles empty string', () => {
    expect(slugify('')).toBe('');
  });
});
```

### Integration Test (API)

```javascript
const request = require('supertest');
const app = require('./app');

describe('GET /api/users', () => {
  test('returns 200 and user list', async () => {
    const res = await request(app)
      .get('/api/users')
      .expect('Content-Type', /json/)
      .expect(200);

    expect(res.body).toBeInstanceOf(Array);
    expect(res.body[0]).toHaveProperty('name');
  });

  test('returns 404 for unknown user', async () => {
    await request(app)
      .get('/api/users/99999')
      .expect(404);
  });
});
```

### Test Commands

```bash
npx jest                 # run all tests
npx jest --watch         # re-run on changes
npx jest --coverage      # coverage report
```

---

## Slide 17 — Security Best Practices

| Threat | Mitigation |
|--------|------------|
| **Injection** (SQL, NoSQL, command) | Parameterised queries, input validation, never concatenate user input into commands |
| **XSS** | Escape output, use `helmet` for CSP headers, avoid `innerHTML` |
| **Prototype Pollution** | Freeze prototypes, validate JSON keys, use `Object.create(null)` |
| **ReDoS** | Avoid complex regex on user input, set timeouts, use `re2` |
| **Dependency attacks** | Audit with `npm audit`, pin versions, use lockfiles |

### Security Middleware Example

```javascript
const express = require('express');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Security headers (CSP, HSTS, X-Frame, etc.)
app.use(helmet());

// Rate limiting
app.use(rateLimit({
  windowMs: 15 * 60 * 1000, // 15 min
  max: 100,                  // 100 req/window
}));

// Never expose stack traces in production
app.set('env', 'production');

// Parameterised query — prevents SQL injection
const user = await db.query(
  'SELECT * FROM users WHERE id = $1',
  [req.params.id]  // NOT string concatenation
);
```

### npm audit

Run `npm audit` regularly. Use `npm audit fix` to auto-patch. Integrate into CI.

### Keep Updated

Node.js LTS gets security patches for 30 months. Even-numbered releases (18, 20, 22) are LTS. Odd-numbered are current/experimental.

### Principle of Least Privilege

Run Node processes as non-root. Restrict file system access. Use minimal IAM roles in cloud environments.

---

## Slide 18 — Performance & Scaling

### Single Thread ≠ Single Core

Node runs JS on one thread, but the **cluster** module spawns worker processes to use all CPU cores.

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isPrimary) {
  const cpus = os.cpus().length;
  console.log(`Forking ${cpus} workers`);

  for (let i = 0; i < cpus; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} died`);
    cluster.fork(); // auto-restart
  });
} else {
  require('./server'); // each worker runs server
}
```

### Worker Threads

For CPU-intensive tasks (image processing, crypto, parsing), offload to worker threads to avoid blocking the event loop.

```javascript
const { Worker } = require('worker_threads');

function runHeavyTask(data) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./heavy-task.js', {
      workerData: data
    });
    worker.on('message', resolve);
    worker.on('error', reject);
  });
}

const result = await runHeavyTask({ iterations: 1e8 });
```

### Key Metrics to Monitor

- Event loop lag (should be < 100ms)
- Heap used vs heap total
- Active handles and requests
- GC pause duration

---

## Slide 19 — The Node.js Ecosystem

### Web Frameworks

- **Express** — minimal, ubiquitous
- **Fastify** — schema validation, speed
- **NestJS** — Angular-inspired, TypeScript
- **Hono** — edge-first, ultralight
- **Adonis** — full-stack, Laravel-like

### Databases

- **Prisma** — type-safe ORM
- **Knex** — SQL query builder
- **Mongoose** — MongoDB ODM
- **pg / mysql2** — raw drivers
- **ioredis** — Redis client

### Real-Time

- **Socket.IO** — WebSocket abstraction
- **ws** — lightweight WebSocket lib
- **SSE** — server-sent events (built-in)
- **GraphQL Subscriptions**
- **MQTT.js** — IoT messaging

### DevOps & Tooling

- **PM2** — process manager, clustering
- **nodemon** — auto-restart in dev
- **ESLint** — linting
- **Prettier** — formatting
- **tsx / ts-node** — TypeScript execution

### Testing

- **Jest** — all-in-one test runner
- **Vitest** — Vite-native, fast
- **Mocha + Chai** — classic combo
- **Supertest** — HTTP assertions
- **node:test** — built-in test runner

### Full-Stack Frameworks

- **Next.js** — React SSR/SSG
- **Nuxt** — Vue SSR/SSG
- **Remix** — web standards, React
- **SvelteKit** — Svelte full-stack
- **Astro** — content-focused, multi-framework

---

## Slide 20 — Summary & Next Steps

### Key Takeaways

- Node.js = V8 + libuv — JavaScript on the server
- Event-driven, non-blocking I/O for high concurrency
- The event loop is single-threaded; don't block it
- Use `async/await` for clean async code
- Streams for memory-efficient data processing
- npm is the world's largest package ecosystem
- Express is the de facto web framework (but explore Fastify)
- Security, error handling, and graceful shutdown matter

### Recommended Reading

- **nodejs.org/docs** — official API reference
- **Node.js Design Patterns** — Casciaro & Mammino
- **Distributed Systems with Node.js** — Thomas Hunter II
- **javascript.info** — modern JS tutorial

### Try These

- Build a REST API with Express + a real database
- Write a CLI tool using Node's built-in modules
- Create a real-time chat with Socket.IO
- Deploy to a cloud provider with Docker
- Contribute to an open-source Node.js project

---

*Thank you! — Built with Reveal.js. Single self-contained HTML file.*
