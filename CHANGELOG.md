# Changelog

All notable changes to the DragonOrigin knowledge base are documented in this file. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and is organized by month, newest first.

## [2026-11] - 2026-11-09

### Added

- **Go fundamentals** (`pages/Go基础知识.md`) — Windows installation and VSCode toolchain setup (GOROOT/PATH, GO111MODULE, GOPROXY, Go extension), first module with `go mod init`/`go work init`, variables and scope, constants and enums, basic data types, structs, control flow and operators, functions, methods and interfaces, and error handling with `error`, `panic` and `recover`; plus OOP through anonymous composition and anonymous fields, adding methods to types, value vs reference semantics, and method sets
- **Server-Sent Events** (`pages/SSE.md`) — HTTP streaming-based server push, SSE vs WebSocket trade-offs (unidirectional vs full-duplex, protocol weight, built-in reconnection, text vs binary), and Node.js client (`eventsource`) and server examples
- **RPC protocol and Protobuf** (`pages/RPC协议.md`) — remote procedure call concepts and synchronous call semantics, the RPC workflow (client/server stubs, marshalling and unmarshalling), Protobuf syntax and schema evolution rules (field numbering, singular/repeated, reserved, import, nested types, Any, oneof, map, package, backward compatibility), and Node.js implementations with JSON-RPC (jayson) and gRPC
- **High concurrency, availability, and performance** (`pages/高并发&高可用&高性能.md`) — Java BIO/NIO/AIO models, RPC protocols, and a real-time project folder sync case study (chokidar, Netty, Spring Boot, PostgreSQL/Redis), plus local images
- **Network security** (`pages/网络安全.md`) — information gathering (active/passive, Google hacking), network commands and tools (arping, netdiscover, ping, traceroute, Nmap), Wireshark capture and display filters, Nmap scanning and service/OS detection, and Burp Suite modules (Proxy, Target, Repeater, Intruder, Decoder, Comparer)
- **C language classic examples** (`pages/C语言经典用例.md`) — 18 beginner programs including the multiplication table, matrix rotation, classic math problems, primes and perfect numbers, triangle printing, sorting and searching, struct pointers, and string exercises
- **Data warehouse and knowledge discovery** (`pages/数据仓库与知识发现.md`) — data-mining workflow and tasks, OLTP vs OLAP, data warehouse characteristics and three-tier architecture, star/snowflake schemas and data cubes, OLAP operations, preprocessing and normalization, association rules (Apriori, FP-Tree), clustering (hierarchical, K-Means, DBSCAN), and classification (ID3, naive Bayes)
- **Computer networks** (`pages/计算机网络.md`) — OSI seven-layer and TCP/IP four-layer models, TCP handshake/teardown and connection states, long vs short connections, HTTP request flow, HTTPS/SSL/TLS, CDN, common application-layer protocols, TCP vs UDP, IP/ICMP/ARP/RARP, Ethernet/PPP, ADSL, and IPv4 vs IPv6
- **Operating system implementation** (`pages/操作系统.md`) — OS concepts, development environment (gcc/NASM, VirtualBox, CentOS 7, bochs), MBR and boot flow, real/protected mode, segmentation and privilege levels, interrupts, memory/process/thread management, synchronization, syscall implementation, heap management, disk driver, and file system
- **jieba and SnowNLP** (`pages/jieba与snownlp.md`) — SnowNLP installation and usage (segmentation, tagging, pinyin, sentiment), custom sentiment model training, jieba segmentation modes and dictionaries, naive-Bayes sentiment internals, keyword extraction (TF-IDF, TextRank), POS tagging, and parallel segmentation
- **SnowNLP practice and BERT** (`pages/snownlp实践与BERT模型.md`) — sentiment training on a labeled takeaway-review dataset, the Bayes formulation behind SnowNLP, pretrained-model freezing vs fine-tuning, Seq2Seq review, and the attention mechanism
- **OpenCV** (`pages/OpenCV学习.md`) — image read/write, BGR/HSV channels and masking, mosaic effects, contour recognition, histograms, Fourier transform, video capture, and morphology
- **Transformer and attention** (`pages/Transformer模型与Attention机制.md`) — encoder-decoder stack, embedding pipeline, self-attention Q/K/V, scaled dot-product attention, and multi-head attention
- **Word2Vec** (`pages/词向量Word2Vec.md`) — word embeddings, CBOW, Skip-gram, softmax/cross-entropy training, and the model's strengths and limitations

### Changed

- Download all 282 remote images referenced by 21 notes into per-page `assets/<page>/` directories and rewrite the references to relative `../assets/` paths

## [2026-10] - 2026-10-30

### Added

- **Python web scraping** (`pages/Python爬虫.md`) — crawler basics and BeautifulSoup, data storage (CSV, MySQL, email), text analysis, document parsing, crawling strategies, anti-scraping countermeasures, Selenium + OCR, and practice cases
- **Python fundamentals** (`pages/Python基础.md`) — design philosophy, data types, functions, advanced containers, scope, OOP, modules and packages, file I/O, and exception handling
- **InversifyJS** (`pages/InversifyJS.md`) — IoC/DI concepts, TypeScript decorator setup with `reflect-metadata`, interface binding, `Symbol` identifiers, and container resolution
- **Pages index** (`pages/索引.md`) — topic-based index of all notes under `pages/`
- **Linux fundamentals** (`pages/Linux基础.md`) — Q&A-style coverage of installation, file and directory management, Shell programming, users and permissions, disk/filesystem management, RPM/YUM, networking, SSH, NFS, and LVM
- **AI development user guide** (`pages/人工智能实践应用.md`) — AGI five-level standards, Ollama and LangChain, RAG, fine-tuning (LoRA/QLoRA/PEFT/FFT), AI Agents and function calling, and Ant Design X chat UI
- **Embedded IDE observability, chapters 6-7** (`pages/嵌入式IDE可观测性.md`) — debug-information reconstruction (ELF/DWARF, stack unwinding, LLDB SB API, Python CLI extensions, async UI sync) and AI-assisted debugging (context assembly, crash forensics, multi-round diagnosis, intelligent breakpoints, natural-language debugging)
- **Rust comprehensive examples** (`pages/RustExample.md`) — formatted output, primitive/custom types, bindings, type system and conversion, control flow, functions, and methods
- **JavaScript prototype and prototype chain** (`pages/原型和原型链.md`) — `__proto__`, `prototype`, `constructor`, inheritance, and property lookup
- **GDB Remote Serial Protocol** (`pages/嵌入式IDE可观测性.md`) — packet layer, control/stop/breakpoint/observation packets, capability negotiation, and protocol gaps, plus local assets
- **JavaScript V8 engine** (`pages/JavaScript V8引擎.md`) — engine landscape, parser/Ignition/TurboFan/Orinoco internals, JIT, closures and scope, hidden classes and inline caching, event loop, and garbage collection
- **Vue.js fundamentals** (`pages/VUE基础.md`) — directives, lifecycle hooks, computed properties and watchers, Vue+Axios, and form handling
- **React fundamentals** (`pages/React基础.md`) — virtual DOM and Diff algorithm, JSX, class components and lifecycle, refs, Create React App, and an Axios practice app
- **Docker basics** (`pages/Docker基本知识.md`) — installation, container and image management, Dockerfile, networking, Compose, Machine, and Swarm
- **Advanced Rust** (`pages/Rust进阶.md`) — single- and multi-threaded web servers, `ThreadPool` with `Arc<Mutex<Receiver>>`, graceful shutdown, and a file search tool with tests
- **Rust fundamentals** (`pages/Rust基础.md`) — environment setup, variable bindings and destructuring, mutability, and ownership
- **LLVM architecture and compilation** (`pages/LLVM详解.md`) — modular frontend/optimizer/backend design, Clang and IR, the C++ compilation pipeline, toolchain utilities, Pass development, and ObjC obfuscation
- **Codelldb analysis** (`pages/Codelldb解析.md`) — build steps, DAP adapter architecture, startup and reverse-connector flow, Tokio/thread model, VS Code integration, and channel/backpressure techniques
- **VSCode debug configuration** (`pages/VSCode调试配置.md`) — `launch.json` fields and examples for Node.js, Python, Java, C#, web, and client-server debugging
- **Software design patterns** (`pages/软件设计模式.md`) — creational, structural, and behavioral patterns
- **Software design principles** (`pages/软件设计原则.md`) — Open/Closed, Dependency Inversion, Liskov Substitution, Interface Segregation, Demeter, and Composite Reuse, with Java examples
- **Web Shell design and implementation** (`pages/Web Shell的设计与实现.md`) — render/server two-process architecture, xterm.js shell, and `ssh2` server-side shell
- **VSCode editor extensions and Open VSX** (`pages/VSCode解析.md`) — Hover and CodeLens providers, decoration types, and Open VSX deployment/publishing
- **OpenOCD reset and Flash algorithms** (`pages/嵌入式IDE可观测性.md`) — reset signals and configs, Cortex-M reset modes, the reset event chain, and in-RAM Flash programming
- **VSCode extension development practice** (`pages/VSCode解析.md`) — `yo code`, F5 debugging, and a React WebView with `postMessage`
- **VSCode shared process and render-process loading** (`pages/VSCode解析.md`) — `setupSharedProcess`, `MessagePortClient`, `createBrowserWindow`, and the page-loading bootstrap chain
- **Embedded IDE observability, JTAG/SWD** (`pages/嵌入式IDE可观测性.md`) — physical/link layer, TAP state machine, scan chain, SWD frames, and JTAG↔SWD switching, plus local assets
- **Embedded IDE observability, hardware abstraction and debug interaction** (`pages/嵌入式IDE可观测性.md`) — ARM ADI and RISC-V DTM/DMI/DM access, CMSIS-DAP, and OpenOCD target abstraction
- **VSCode source structure and build** (`pages/VSCode解析.md`) — `src/vs` layering, contrib conventions, and the Windows/Linux/macOS build and packaging flow
- **VSCode Server/CLI and process architecture** (`pages/VSCode解析.md`) — headless Server/Server-CLI/CLI forms, startup, and local vs SSH remote process models
- **VSCode architecture analysis** (`pages/VSCode解析.md`) — product forms, client startup, window opening, and render-process loading
- **VSCode Remote SSH and IPC patterns** (`pages/VSCode解析.md`) — one-to-one/one-to-many communication and the SSH tunnel connection flow
- **VSCode IPCClient/IPCServer/Connection** (`pages/VSCode解析.md`) — channel routing, connection management, and process communication construction

### Changed

- Localize external CDN images to local assets across 8 pages (Electron基础, MarkDown编辑器, Nodejs事件循环机制, TypeScript基础知识, VSCode解析, Webpack基础, 软件设计原则, 软件设计模式)
- Localize sections 4 and 5.1 images of `pages/嵌入式IDE可观测性.md` to local assets
- Reformat the hardware breakpoint section in `pages/嵌入式IDE可观测性.md` for consistent headings and indentation

### Removed

- Remove an unreferenced duplicate JTAG diagram

## [2026-09] - 2026-09-30

### Added

- **VSCode IPC construction** (`pages/VSCode解析.md`) — `MessagePortMain` bridge, Caller-Service request-response, and `ChannelClient`/`ChannelServer`
- **Virtual `vscode` module** (`pages/VSCode解析.md`) — `Module._load()` hijacking and the external extension vs internal contrib mechanisms
- **VSCode extension mechanism** (`pages/VSCode解析.md`) — plugin loading, extension host startup, and activation
- **Markdown editor analysis** (`pages/MarkDown编辑器.md`) — Markdown syntax and the `marked` parser
- **LSP and DAP** (`pages/VSCode解析.md`) — language servers/clients and the debug adapter protocol
- **VSCode IoC/DI and module loading** (`pages/VSCode解析.md`) — `createDecorator`, `InstantiationService`, `SyncDescriptor`, and the AMD loader
- **VSCode IoC/DI note** (`pages/VSCode解析.md`) — service identifiers, dependency injection, and the service architecture
- **Barrier** (`pages/VSCode解析.md`) — synchronizing async operations with `wait()`/`open()`
- **VSCode IPC communication** (`pages/VSCode解析.md`) — protocol/channel/connection concepts and IPC examples
- **Emitter and observer pattern** (`pages/VSCode解析.md`) — `Emitter.fire`, `get event()`, and `EmitterOptions`
- **Event and Emitter** (`pages/VSCode解析.md`) — `Event<T>`, disposable listeners, and the event utility library
- **Disposable** (`pages/VSCode解析.md`) — the dispose pattern and resource management
- **JavaScript Proxy** (`pages/JavaScript Proxy解析.md`) — proxy syntax, handler traps, and its use cases
- **Closure** (`pages/VSCode解析.md`) — closures, their uses and caveats, and usage in the shared process
- **IIFE** (`pages/VSCode解析.md`) — local scope, state isolation, and namespace injection
- **Promise** (`pages/VSCode解析.md`) — states, `then`/`catch`/`finally`, `all` and `race`
- **Node.js event loop** (`pages/Nodejs事件循环机制.md`) — six phases, macro/micro task queues, and an async walkthrough
- **Webpack basics** (`pages/Webpack基础.md`) — config, plugins, dev server, loaders, and asset modules
- **Electron basics** (`pages/Electron基础.md`) — process model, context isolation, IPC, sandboxing, `MessagePort`, and performance
- **TypeScript basics** (`pages/TypeScript基础知识.md`) — first script, basic types, declarations, and operators
- **Linux socket layer** (`pages/VSCode解析.md`) — `sock` structure hierarchy, connection setup, buffers, and thundering herd
- **SSH port forwarding** (`pages/VSCode解析.md`) — `-D`/`-L`/`-R` forwarding and the corresponding SSH APIs
- **HTTP protocol upgrade** (`pages/VSCode解析.md`) — `Upgrade` header, WebSocket handshake, and Java/Netty/Node.js examples
- **Repository README** (`README.md`)
- **Initial content** — VSCode analysis skeleton, `pages/gulp构建工具.md` (Vinyl/tasks/globs, VSCode win32 packaging), a daily journal entry, and the local `git-commit` skill

### Removed

- Remove the empty `pages/contents.md` (recycled by Logseq)
