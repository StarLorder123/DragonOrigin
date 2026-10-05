# Changelog

## [Unreleased]

### Documentation

- Add VSCode architecture and startup notes (`pages/VSCode解析.md`) covering the product forms (client/server/cli), client startup and window-opening flows, headless Server/Server-CLI/CLI forms, desktop vs SSH Remote process architectures, and the source structure and build/package process
- Add VSCode IPC and Remote SSH notes (`pages/VSCode解析.md`) covering `ChannelClient`/`ChannelServer`, `IPCClient`/`IPCServer`/`Connection`, the `MessagePortMain` bridge, Caller-Service request-response, one-to-one/one-to-many IPC, and the Remote SSH connection flow
- Add VSCode extension mechanism notes (`pages/VSCode解析.md`) covering plugin loading, extension host process startup and activation, and the virtual `vscode` module (hijacking `Module._load()`)
- Add VSCode IoC/DI and module-loading notes (`pages/VSCode解析.md`) covering service identifiers/`createDecorator`, `InstantiationService`/`SyncDescriptor`, and the AMD vs CMD loader (`vscode-loader`)
- Add LSP and DAP protocol notes (`pages/VSCode解析.md`) covering the Language Server Protocol and Debug Adapter Protocol
- Add JavaScript/TypeScript fundamentals and tooling notes (Proxy, closure, IIFE, Promise, Event/Emitter, Disposable, Barrier, the Node.js event loop, TypeScript, Webpack, Electron) to `pages/VSCode解析.md` and related `pages/*.md`
- Add network/system notes (`pages/VSCode解析.md`) covering HTTP protocol upgrade, SSH port forwarding, and the Linux socket layer
- Add embedded IDE observability notes (`pages/嵌入式IDE可观测性.md`) covering the JTAG physical/link layer (pin definitions, TAP state machine, electrical characteristics, timing, connection, daisy chaining, JTAG/SWD pin comparison), the TAP 16-state machine, scan-chain topology, the SWD two-wire half-duplex frame structure (Request/ACK/Turnaround/read-write transactions), and the JTAG↔SWD mode-switch sequence (SWJ switch sequence, `0xE79E`)
- Add gulp build tool note (`pages/gulp构建工具.md`) covering Vinyl/tasks/globs and VSCode win32 packaging
- Add repository README (`README.md`)
- Add daily journal entry for 2026-09-06 (`journals/2026_09_06.md`)

### Changed

- Remove empty `pages/contents.md` (recycled by Logseq)

### Chore

- Add local `git-commit` skill (`.claude/skills/git-commit`)
