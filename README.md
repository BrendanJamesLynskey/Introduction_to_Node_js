# ⬢ Introduction to Node.js

An interactive Reveal.js presentation covering Node.js — from the V8 engine and event loop through to Express middleware, streams, async patterns, security, and scaling to production.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_Node_js/)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | Node.js overview |
| 02 | Agenda | Topics at a glance |
| 03 | What Is Node.js? | Origin, key properties, who uses it |
| 04 | Runtime Architecture | V8, libuv, Node bindings, OS layer |
| 05 | The Event Loop | Phases, ticks, microtasks, process.nextTick |
| 06 | Blocking vs Non-Blocking I/O | Sync vs async, thread pool, why it matters |
| 07 | Modules | CommonJS vs ES Modules, resolution order |
| 08 | npm & Package Management | Init, install, semver, lock files, alternatives |
| 09 | Async Patterns | Callbacks, Promises, async/await, parallel execution |
| 10 | Streams & Buffers | Readable, Writable, Duplex, Transform, pipeline |
| 11 | Building an HTTP Server | Bare Node http module, request/response lifecycle |
| 12 | Express.js & Middleware | Routing, middleware pattern, popular packages |
| 13 | File System & Path | fs/promises API, path module, watching for changes |
| 14 | Environment Variables | process.env, .env files, NODE_ENV, security rules |
| 15 | Error Handling | Operational vs programmer errors, graceful shutdown |
| 16 | Testing | Jest, Supertest, unit and integration tests |
| 17 | Security | Injection, XSS, prototype pollution, npm audit, helmet |
| 18 | Performance & Scaling | Cluster module, worker threads, key metrics |
| 19 | The Ecosystem | Frameworks, databases, real-time, testing, full-stack |
| 20 | Summary & Next Steps | Key takeaways and recommended reading |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

Node.js Foundation, *Node.js Documentation* — nodejs.org · Casciaro & Mammino, *Node.js Design Patterns*, 3rd ed., Packt, 2020 · Thomas Hunter II, *Distributed Systems with Node.js*, O'Reilly, 2020 · Express.js, *API Reference* — expressjs.com · libuv, *Design Overview* — docs.libuv.org

## License

Educational use. Code examples provided as-is.
