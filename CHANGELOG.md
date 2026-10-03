# Changelog

## [Unreleased]

### Documentation

- Add VSCode Server/CLI forms and process architecture note (`pages/VSCode解析.md`) covering the headless Server / Server-CLI / CLI forms, the Server startup flow (`src/server-main.js` `main()` → `http.createServer` → `getRemoteExtensionHostAgentServer().handleRequest`), SSH remote usage, and the desktop vs SSH Remote process architectures (Main / Shared / Render / Extension Host / Debugger / Search processes locally, and the remote Code-Server / Extension Host / FileWatcher / Pty processes)
- Add embedded IDE observability note (`pages/嵌入式IDE可观测性.md`) covering the JTAG physical/link layer — standard pin definitions (TCK/TMS/TDI/TDO/TRST), the TAP state machine, electrical characteristics (VTREF reference voltage, CMOS/LVTTL input levels, TDO drive strength), timing relationships, typical connections and daisy chaining, and a JTAG vs SWD pin comparison
- Add VSCode architecture analysis note (`pages/VSCode解析.md`) covering the three product forms (client/server/cli), the client startup flow (`node out/main.js` → `main.js` `app.once('ready')` → `bootstrap-amd` loading `vs/code/electron-main/main` → `CodeMain.main()` → `startup()` → `CodeApplication.startup()` with shared process + renderer process), and the window opening flow (`openFirstWindow` → `windowsMainService.open`/`doOpen` → `openInBrowserWindow` → `doOpenInBrowserWindow` creating a `CodeWindow` then `load()` → `loadURL` of `workbench.html` → `workbench.js` → `bootstrap-window.js` loading `workbench.desktop.main`)
- Add VSCode Remote SSH and one-to-one/one-to-many communication notes (`pages/VSCode解析.md`) covering one-to-one bidirectional IPC achieved by having a single object implement both `ChannelClient` and `ChannelServer`, and the Remote SSH connection flow (SSH download/start of code-server, parsing the server socket file, allocating a local port/socket, and the local port → socket → remote port SSH tunnel chain) plus the `open-remote-ssh` plugin source
- Add VSCode `IPCClient` / `Connection` / `IPCServer` source note (`pages/VSCode解析.md`) covering the `IPCClient` implementation (wrapping `ChannelClient`/`ChannelServer` for one-to-one IPC), the `Connection` (an `IPCClient` bound to a unique ctx identifier) and `IPCServer` source walkthrough (connection management, `getChannel` routing, `getMulticastEvent`), and the process communication construction flow (main/shared/renderer processes, communication bridge for tool processes)
- Add VSCode inter-process communication (IPC) construction note (`pages/VSCode解析.md`) covering the `MessagePortMain` communication bridge (`connect` transferring ownership via `postMessage` with a `MessageChannelMain` port), the Caller-Service request-response mechanism built on `IServerChannel`/`IChannel`, and `ChannelClient`/`ChannelServer` source walkthrough (request id handling, `handlers` map, `activeRequests`/`pendingRequests`, initialize handshake, and channel registration)
- Add VSCode virtual `vscode` module and extension limitation notes (`pages/VSCode解析.md`) covering the hijacked `Module._load()` in `defineAPI` that resolves `require('vscode')` to a per-extension API implementation, the module as a declaration-only `.d.ts` (like a C++ header), and the external `extension` vs internal `contrib` dual extension mechanism
- Add VSCode extension mechanism note (`pages/VSCode解析.md`) covering plugin loading (NativeExtensionService initialization, extension host managers, and host process forking), extension host process startup (host process as a Node.js utility process launched from the `extensionHostProcess` entry point), and plugin activation (`_activateExtension` → `_doActivateExtension` → `_getEntryPoint` → `_callActivate` invoking `extensionModule.activate.apply`)
- Add LSP and DAP protocol notes to `pages/VSCode解析.md` covering the Language Server Protocol (client/server split, `vscode-languageclient`/`vscode-languageserver`, the `lsp-sample` demo's `extension.ts` client and `server.ts` server) and the Debug Adapter Protocol (adapter layer abstracting debugger/runtime communication)
- Add VSCode IoC/DI implementation and AMD loader note (`pages/VSCode解析.md`) covering the `createDecorator` / `storeServiceDependency` internals, the `$di$dependencies` / `$di$target` magic strings, and the AMD vs CMD module systems plus `vscode-loader`'s `ScriptLoader` / `ModuleManager` and global `define`/`require` patching
- Add IoC and dependency injection note (`pages/VSCode解析.md`) covering the inversion-of-control principle, the dependency-injection design pattern, VSCode's service-based architecture, service identifiers/`createDecorator`, the `InstantiationService` container, `SyncDescriptor`, and `createInstance` / `invokeFunction` / `createChild` usage
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
