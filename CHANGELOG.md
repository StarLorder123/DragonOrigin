# Changelog

## [Unreleased]

### Documentation

- Add Barrier (同步屏障) note to `pages/VSCode解析.md` covering the Barrier type for synchronizing async operations, the `wait()` / `open()` methods, avoiding race conditions, and its implementation
- Add VSCode IPC communication mechanism note (`pages/VSCode解析.md`) covering communication concepts (Protocol, Channel, Connection, IPCClient/IPCServer), the `QueueProtocol` / `TestIPCClient` / `TestIPCServer` / `TestChannel` examples, and one-to-many IPC setup
- Add Emitter event emitter note to `pages/VSCode解析.md` covering `Emitter.fire` / `get event()`, `EmitterOptions` callbacks, an `EditorService` event usage example, the observer (发布-订阅) pattern, and a summary of event-based named/auto-disposed registration
- Add Event and Emitter note to `pages/VSCode解析.md` covering the `Event<T>` interface, disposable-based listener removal, and the Event utility library (`once`, `debounce`, `buffer`, chainable events, DOM/Promise sources)
- Add Disposable / IDisposable note to `pages/VSCode解析.md` covering the Dispose pattern, resource management, and the `dispose` function implementation
- Add JavaScript Proxy note (`pages/JavaScript Proxy解析.md`) covering proxy syntax, handler traps, and data binding / event listening / caching use cases
- Link JavaScript Proxy note from new "3.9 Proxy代理" section in `pages/VSCode解析.md`
- Add closure (闭包) note to `pages/VSCode解析.md` covering what closures are, their uses and caveats, and closure usage in VSCode's shared process setup
- Add IIFE (Immediately Invoked Function Expression) note to `pages/VSCode解析.md` covering local scope, closure state isolation, and namespace injection patterns
- Add Promise note to `pages/VSCode解析.md` covering Promise states, basic usage, then/catch/finally, Promise.all and Promise.race
- Add Node.js event loop note (`pages/Nodejs事件循环机制.md`) covering the six event loop phases, macro/micro task queues, phase details, and an async example walkthrough
- Link Node.js event loop note from "编程语言基础" section in `pages/VSCode解析.md`
- Add Webpack basics note (`pages/Webpack基础.md`) covering install, entry/output config, mode, html-webpack-plugin, webpack-dev-server, css/less loaders, and asset modules
- Link Webpack basics note from "编程语言基础" section in `pages/VSCode解析.md`
- Add TypeScript basics note (`pages/TypeScript基础知识.md`) covering first script, basic types, variable declaration, operators
- Link TypeScript basics note from new "编程语言基础" section in `pages/VSCode解析.md`
- Add repository README (`README.md`)
- Add HTTP protocol upgrade notes (Upgrade header, WebSocket handshake, Java/Netty/Node.js examples) to `pages/VSCode解析.md`
- Add SSH port forwarding notes (-D/-L/-R forwarding, forwardOut/forwardIn/openssh_forwardOutStreamLocal methods) to `pages/VSCode解析.md`
- Add Linux socket layer notes (sock structure hierarchy, socket layer, connection setup, send/recv buffer, thundering herd) to `pages/VSCode解析.md`
- Add Electron basics note (`pages/Electron基础.md`) covering process model, context isolation, IPC patterns, sandboxing, MessagePort, window lifecycle, and performance tips
- Link Electron basics note from "编程语言基础" section in `pages/VSCode解析.md`
- Add VSCode architecture overview note (`pages/VSCode解析.md`)
- Add gulp build tool note covering Vinyl/tasks/globs and VSCode's win32 packaging tasks (`pages/gulp构建工具.md`)
- Add daily journal entry for 2026-09-06 (`journals/2026_09_06.md`)

### Changed

- Remove empty `pages/contents.md` (recycled by Logseq)

### Chore

- Add local `git-commit` skill (`.claude/skills/git-commit`)
