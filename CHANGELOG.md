# Changelog

All notable changes to the DragonOrigin knowledge base are documented in this file. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) and is organized by month, newest first.

## [2026-11] - 2026-11-29

### Changed

- **仓库结构重排** — 从 Logseq 图谱改为普通 Markdown 知识库：扁平 `pages/` 换成「领域 → 小类」两级目录，48 篇笔记按 7 个领域、21 个小类重新归档
- 笔记间的 `[[双链]]` 全部改写为相对 Markdown 链接，普通编辑器与 GitHub 均可直接跳转
- 图片统一保留在 `assets/<笔记名>/`，笔记内引用由 `../assets/...` 调整为 `../../assets/...`（适配两级目录）
- [`索引.md`](索引.md) 重新生成，并补入此前未收录的 `Dart编程语言`、`LLVM编译器分析`
- `README.md` 按新结构重写；`.gitignore` 移除 Logseq 备份规则

### Removed

- `logseq/`（`config.edn`、`custom.css`、`.recycle`）与空的 `journals/`

## [2026-11] - 2026-11-28

### Added

- **SCCP (Sparse Conditional Constant Propagation)** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — `llvm/lib/Transforms/Scalar/SCCP.cpp`: the sparse dataflow lattice (`Undefined`/`Constant`/`Overdefined`), the fixed-point `runSCCP` driver, the `SCCPSolver` operations, unreachable-block handling, and its `STATISTIC` counters
- **DSE (Dead Store Elimination)** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — `llvm/lib/Transforms/Scalar/DeadStoreElimination.cpp`: MemorySSA-based elimination algorithm, the `OverwriteResult` enum, source `STATISTIC` counters, and dependencies (MemorySSA, AliasAnalysis, PostDominators, MustExecute)
- **LoopVectorize** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — `llvm/lib/Transforms/Vectorize/LoopVectorize.cpp`: the four-part architecture (main pass, Legality, Planner/VPlan, CostModel), the `LoopVectorizePass` flow (legality → cost → plan → execute), vectorization-factor selection via target cost model, `STATISTIC` counters, and reference papers

## [2026-11] - 2026-11-27

### Added

- **GVN (Global Value Numbering) pass** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — `llvm/lib/Transforms/Scalar/GVN.cpp`: redundant-instruction elimination with simple DCE and partial redundancy elimination (PRE), the value-numbering table (`GVNValueTable` with its `Expression` keys and hashing) and leader map (`GVNLeaderMap`), source `STATISTIC` counters, the `GVNPassImpl` core algorithm, and its dependencies (MemoryDependenceAnalysis, AliasAnalysis, DominatorTree, TargetLibraryInfo)

## [2026-11] - 2026-11-26

### Added

- **Core optimization Pass deep-dive** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — InstCombine in `llvm/lib/Transforms/InstCombine/`: its design (worklist-driven instruction combining with before/after examples), canonicalization guarantees (constant operands to RHS, bit-op ordering, `add X, X` → `X << 1`, multiply-by-power-of-two → shift), the `InstCombiner` class with its visitor methods and worklist insertion helper, source `STATISTIC` counters, and a per-file breakdown (`InstCombineAddSub`, `AndOrXor`, `Calls`, `Casts`, `Compares`, `LoadStoreAlloca`, `MulDivRem`, `PHI`, `Select`, `Shifts`)

## [2026-11] - 2026-11-25

### Added

- **Pass manager architecture** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — the new Pass manager (NPM) with its key files (`PassBuilder.h`, `PassBuilder.cpp`, `PassBuilderPipelines.cpp`, `OptimizationLevel.h`), the `PassBuilder` class, `PipelineTuningOptions` flags (loop interleaving/vectorization/unrolling/interchange/fusion, SLP vectorization, function merging, inliner threshold), and the Module → CGSCC → Function → Loop pass execution hierarchy

## [2026-11] - 2026-11-24

### Added

- **LLVM optimization algorithm overview** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — the Pass-based optimization system, the `lib/Transforms` directory layout (Scalar, Vectorize, IPO, InstCombine, Utils) with representative passes, and the `-O0` to `-O3`/`-Os`/`-Oz` optimization levels

## [2026-11] - 2026-11-23

### Added

- **LLVM IR language reference** in `04-编译与工具链/LLVM/LLVM编译器分析.md` — design goals, identifiers (`@` global / `%` local), the type system (integer, floating-point, aggregate, pointer and function types), constants (`undef`, `poison`, `null`), instruction examples (arithmetic/bitwise, memory, control flow with `phi`/`switch`, calls including `tail`/`musttail`), function definitions with attribute groups, a complete module example, and IR well-formedness rules (dominance, type consistency, SSA form, block terminators)

## [2026-11] - 2026-11-22

### Added

- LLVM IR representations in `04-编译与工具链/LLVM/LLVM编译器分析.md` — the text (`.ll`), bitcode (`.bc`) and in-memory forms with a sample IR function
- An end-to-end compilation flow diagram from Clang frontend through analysis/transform passes to backend instruction selection, register allocation, scheduling, code emission and debug-info generation
- A JIT (ORC) framework overview covering IR/object transforms and the compile/materialize stages

## [2026-11] - 2026-11-21

### Added

- **LLVM compiler analysis** (`04-编译与工具链/LLVM/LLVM编译器分析.md`) — the three-stage frontend/optimizer/backend architecture, Clang's lexical, syntax and semantic analysis with LLVM IR generation, the Pass-based middle-end, and backend instruction selection, register allocation, scheduling, code emission and debug-info generation

## [2026-11] - 2026-11-20

### Added

- **Dart programming language** (`01-编程语言/Dart/Dart编程语言.md`) — VSCode environment setup, Hello World and variables with type inference and sound null safety, variable types, functions (parameters, closures, anonymous functions, lexical scope), operators, control flow, exceptions, classes (constructors, getters/setters, abstract classes, implicit interfaces, inheritance, operator overloading, enums, mixins, class variables and methods), generics, library visibility (`import`/`export`/`part`), asynchronous support (`Future`, `async`/`await`, `Stream`), generators (`sync*`/`async*`/`yield*`), callable classes, typedefs, metadata, and comments/documentation

## [2026-11] - 2026-11-18

### Changed

- Localize all 75 remote note images into per-page `assets/<page>/` directories and rewrite the references to relative `../assets/` paths: Go fundamentals (2), advanced Go (2), Go framework implementation (7), RPC protocol (1), and compiler principles (63, including 42 LaTeX formula SVGs); no remote image references remain in `pages/`
- Update `索引.md` — add a Go section and index the previously missing Go notes, `编译原理`, `RPC协议` and `SSE`, renumbering the topic sections to 15
- Rewrite `README.md` — reflect the current 46-note scope, repository layout, content themes and local-asset convention, and link the index

## [2026-11] - 2026-11-09

### Added

- **Compiler principles** (`04-编译与工具链/编译原理/编译原理.md`) — the compiler's five phases, grammars and languages (alphabet, symbol strings, production rules, formal language definition, parse trees, ambiguity, grammar simplification, Chomsky hierarchy), lexical analysis and finite automata (regular expressions, DFA/NFA and their minimization, LL(1) and recursive-descent parsing, operator-precedence parsing, LR(0)/SLR(1)/LR(1)/LALR(1)), attribute grammars (synthesized/inherited attributes, dependency graphs, S-attributed and L-attributed definitions), semantic analysis and intermediate languages (abstract syntax trees, three-address code, quadruples, triples, reverse Polish notation), and runtime environments (static, stack and heap storage allocation)
- **Building Go frameworks** (`01-编程语言/Go/Go进阶实现框架.md`) — hand-built framework implementations: the Gee Web framework (net/http Handler, Context, router, route groups, middleware, templates, panic recovery), GeeCache distributed cache (LRU, single-machine concurrency, HTTP server, consistent hashing, distributed nodes, cache-breakdown protection, protobuf), an ORM framework (database/sql, struct-to-table mapping, insert/query, chainable CRUD, hooks, transactions, migrations), and an RPC framework (message encoding, high-performance client, service registration, timeouts, HTTP support, load balancing, service discovery and registry)
- **Advanced Go** (`01-编程语言/Go/Go进阶知识.md`) — building a Go project (`go mod tidy`/`download`/`vendor`), GORM models and CRUD, Gin (RESTful routes, templates, HTML/JSON rendering, parameters, uploads, redirects, middleware), Go Redis, Go MongoDB and BSON, and unit testing with `go test`, GoConvey, monkey patching, gomock, MySQL/Redis and HTTP tests, and Testify
- **Go practice cases** (`01-编程语言/Go/Go实践案例.md`) — three progressive todo-app practices: a layered dao/entity/controller split with Gin routing, adding GORM MySQL connectivity with AutoMigrate and CRUD handlers, and a bare Gin route skeleton for the Vue todo exercise
- **Go fundamentals** (`01-编程语言/Go/Go基础知识.md`) — Windows installation and VSCode toolchain setup (GOROOT/PATH, GO111MODULE, GOPROXY, Go extension), first module with `go mod init`/`go work init`, variables and scope, constants and enums, basic data types, structs, control flow and operators, functions, methods and interfaces, and error handling with `error`, `panic` and `recover`; plus OOP through anonymous composition and anonymous fields, adding methods to types, value vs reference semantics, and method sets
- **Server-Sent Events** (`07-后端与工程/后端架构与高并发/SSE.md`) — HTTP streaming-based server push, SSE vs WebSocket trade-offs (unidirectional vs full-duplex, protocol weight, built-in reconnection, text vs binary), and Node.js client (`eventsource`) and server examples
- **RPC protocol and Protobuf** (`07-后端与工程/后端架构与高并发/RPC协议.md`) — remote procedure call concepts and synchronous call semantics, the RPC workflow (client/server stubs, marshalling and unmarshalling), Protobuf syntax and schema evolution rules (field numbering, singular/repeated, reserved, import, nested types, Any, oneof, map, package, backward compatibility), and Node.js implementations with JSON-RPC (jayson) and gRPC
- **High concurrency, availability, and performance** (`07-后端与工程/后端架构与高并发/高并发&高可用&高性能.md`) — Java BIO/NIO/AIO models, RPC protocols, and a real-time project folder sync case study (chokidar, Netty, Spring Boot, PostgreSQL/Redis), plus local images
- **Network security** (`05-计算机基础与系统/网络安全/网络安全.md`) — information gathering (active/passive, Google hacking), network commands and tools (arping, netdiscover, ping, traceroute, Nmap), Wireshark capture and display filters, Nmap scanning and service/OS detection, and Burp Suite modules (Proxy, Target, Repeater, Intruder, Decoder, Comparer)
- **C language classic examples** (`01-编程语言/C语言/C语言经典用例.md`) — 18 beginner programs including the multiplication table, matrix rotation, classic math problems, primes and perfect numbers, triangle printing, sorting and searching, struct pointers, and string exercises
- **Data warehouse and knowledge discovery** (`06-人工智能与数据/数据库与数据仓库/数据仓库与知识发现.md`) — data-mining workflow and tasks, OLTP vs OLAP, data warehouse characteristics and three-tier architecture, star/snowflake schemas and data cubes, OLAP operations, preprocessing and normalization, association rules (Apriori, FP-Tree), clustering (hierarchical, K-Means, DBSCAN), and classification (ID3, naive Bayes)
- **Computer networks** (`05-计算机基础与系统/计算机网络/计算机网络.md`) — OSI seven-layer and TCP/IP four-layer models, TCP handshake/teardown and connection states, long vs short connections, HTTP request flow, HTTPS/SSL/TLS, CDN, common application-layer protocols, TCP vs UDP, IP/ICMP/ARP/RARP, Ethernet/PPP, ADSL, and IPv4 vs IPv6
- **Operating system implementation** (`05-计算机基础与系统/操作系统/操作系统.md`) — OS concepts, development environment (gcc/NASM, VirtualBox, CentOS 7, bochs), MBR and boot flow, real/protected mode, segmentation and privilege levels, interrupts, memory/process/thread management, synchronization, syscall implementation, heap management, disk driver, and file system
- **jieba and SnowNLP** (`06-人工智能与数据/人工智能与机器学习/jieba与snownlp.md`) — SnowNLP installation and usage (segmentation, tagging, pinyin, sentiment), custom sentiment model training, jieba segmentation modes and dictionaries, naive-Bayes sentiment internals, keyword extraction (TF-IDF, TextRank), POS tagging, and parallel segmentation
- **SnowNLP practice and BERT** (`06-人工智能与数据/人工智能与机器学习/snownlp实践与BERT模型.md`) — sentiment training on a labeled takeaway-review dataset, the Bayes formulation behind SnowNLP, pretrained-model freezing vs fine-tuning, Seq2Seq review, and the attention mechanism
- **OpenCV** (`01-编程语言/Python与数据分析/OpenCV学习.md`) — image read/write, BGR/HSV channels and masking, mosaic effects, contour recognition, histograms, Fourier transform, video capture, and morphology
- **Transformer and attention** (`06-人工智能与数据/人工智能与机器学习/Transformer模型与Attention机制.md`) — encoder-decoder stack, embedding pipeline, self-attention Q/K/V, scaled dot-product attention, and multi-head attention
- **Word2Vec** (`06-人工智能与数据/人工智能与机器学习/词向量Word2Vec.md`) — word embeddings, CBOW, Skip-gram, softmax/cross-entropy training, and the model's strengths and limitations

### Changed

- Download all 282 remote images referenced by 21 notes into per-page `assets/<page>/` directories and rewrite the references to relative `../assets/` paths

## [2026-10] - 2026-10-30

### Added

- **Python web scraping** (`01-编程语言/Python与数据分析/Python爬虫.md`) — crawler basics and BeautifulSoup, data storage (CSV, MySQL, email), text analysis, document parsing, crawling strategies, anti-scraping countermeasures, Selenium + OCR, and practice cases
- **Python fundamentals** (`01-编程语言/Python与数据分析/Python基础.md`) — design philosophy, data types, functions, advanced containers, scope, OOP, modules and packages, file I/O, and exception handling
- **InversifyJS** (`01-编程语言/JavaScript与TypeScript/InversifyJS.md`) — IoC/DI concepts, TypeScript decorator setup with `reflect-metadata`, interface binding, `Symbol` identifiers, and container resolution
- **Pages index** (`索引.md`) — topic-based index of all notes under `pages/`
- **Linux fundamentals** (`05-计算机基础与系统/Linux与运维/Linux基础.md`) — Q&A-style coverage of installation, file and directory management, Shell programming, users and permissions, disk/filesystem management, RPM/YUM, networking, SSH, NFS, and LVM
- **AI development user guide** (`06-人工智能与数据/人工智能与机器学习/人工智能实践应用.md`) — AGI five-level standards, Ollama and LangChain, RAG, fine-tuning (LoRA/QLoRA/PEFT/FFT), AI Agents and function calling, and Ant Design X chat UI
- **Embedded IDE observability, chapters 6-7** (`03-编辑器与IDE/调试器/嵌入式IDE可观测性.md`) — debug-information reconstruction (ELF/DWARF, stack unwinding, LLDB SB API, Python CLI extensions, async UI sync) and AI-assisted debugging (context assembly, crash forensics, multi-round diagnosis, intelligent breakpoints, natural-language debugging)
- **Rust comprehensive examples** (`01-编程语言/Rust/RustExample.md`) — formatted output, primitive/custom types, bindings, type system and conversion, control flow, functions, and methods
- **JavaScript prototype and prototype chain** (`01-编程语言/JavaScript与TypeScript/原型和原型链.md`) — `__proto__`, `prototype`, `constructor`, inheritance, and property lookup
- **GDB Remote Serial Protocol** (`03-编辑器与IDE/调试器/嵌入式IDE可观测性.md`) — packet layer, control/stop/breakpoint/observation packets, capability negotiation, and protocol gaps, plus local assets
- **JavaScript V8 engine** (`01-编程语言/JavaScript与TypeScript/JavaScript V8引擎.md`) — engine landscape, parser/Ignition/TurboFan/Orinoco internals, JIT, closures and scope, hidden classes and inline caching, event loop, and garbage collection
- **Vue.js fundamentals** (`02-前端开发/前端框架/VUE基础.md`) — directives, lifecycle hooks, computed properties and watchers, Vue+Axios, and form handling
- **React fundamentals** (`02-前端开发/前端框架/React基础.md`) — virtual DOM and Diff algorithm, JSX, class components and lifecycle, refs, Create React App, and an Axios practice app
- **Docker basics** (`05-计算机基础与系统/Linux与运维/Docker基本知识.md`) — installation, container and image management, Dockerfile, networking, Compose, Machine, and Swarm
- **Advanced Rust** (`01-编程语言/Rust/Rust进阶.md`) — single- and multi-threaded web servers, `ThreadPool` with `Arc<Mutex<Receiver>>`, graceful shutdown, and a file search tool with tests
- **Rust fundamentals** (`01-编程语言/Rust/Rust基础.md`) — environment setup, variable bindings and destructuring, mutability, and ownership
- **LLVM architecture and compilation** (`04-编译与工具链/LLVM/LLVM详解.md`) — modular frontend/optimizer/backend design, Clang and IR, the C++ compilation pipeline, toolchain utilities, Pass development, and ObjC obfuscation
- **Codelldb analysis** (`03-编辑器与IDE/调试器/Codelldb解析.md`) — build steps, DAP adapter architecture, startup and reverse-connector flow, Tokio/thread model, VS Code integration, and channel/backpressure techniques
- **VSCode debug configuration** (`03-编辑器与IDE/VSCode/VSCode调试配置.md`) — `launch.json` fields and examples for Node.js, Python, Java, C#, web, and client-server debugging
- **Software design patterns** (`07-后端与工程/软件工程/软件设计模式.md`) — creational, structural, and behavioral patterns
- **Software design principles** (`07-后端与工程/软件工程/软件设计原则.md`) — Open/Closed, Dependency Inversion, Liskov Substitution, Interface Segregation, Demeter, and Composite Reuse, with Java examples
- **Web Shell design and implementation** (`03-编辑器与IDE/桌面应用/Web Shell的设计与实现.md`) — render/server two-process architecture, xterm.js shell, and `ssh2` server-side shell
- **VSCode editor extensions and Open VSX** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — Hover and CodeLens providers, decoration types, and Open VSX deployment/publishing
- **OpenOCD reset and Flash algorithms** (`03-编辑器与IDE/调试器/嵌入式IDE可观测性.md`) — reset signals and configs, Cortex-M reset modes, the reset event chain, and in-RAM Flash programming
- **VSCode extension development practice** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `yo code`, F5 debugging, and a React WebView with `postMessage`
- **VSCode shared process and render-process loading** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `setupSharedProcess`, `MessagePortClient`, `createBrowserWindow`, and the page-loading bootstrap chain
- **Embedded IDE observability, JTAG/SWD** (`03-编辑器与IDE/调试器/嵌入式IDE可观测性.md`) — physical/link layer, TAP state machine, scan chain, SWD frames, and JTAG↔SWD switching, plus local assets
- **Embedded IDE observability, hardware abstraction and debug interaction** (`03-编辑器与IDE/调试器/嵌入式IDE可观测性.md`) — ARM ADI and RISC-V DTM/DMI/DM access, CMSIS-DAP, and OpenOCD target abstraction
- **VSCode source structure and build** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `src/vs` layering, contrib conventions, and the Windows/Linux/macOS build and packaging flow
- **VSCode Server/CLI and process architecture** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — headless Server/Server-CLI/CLI forms, startup, and local vs SSH remote process models
- **VSCode architecture analysis** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — product forms, client startup, window opening, and render-process loading
- **VSCode Remote SSH and IPC patterns** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — one-to-one/one-to-many communication and the SSH tunnel connection flow
- **VSCode IPCClient/IPCServer/Connection** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — channel routing, connection management, and process communication construction

### Changed

- Localize external CDN images to local assets across 8 pages (Electron基础, MarkDown编辑器, Nodejs事件循环机制, TypeScript基础知识, VSCode解析, Webpack基础, 软件设计原则, 软件设计模式)
- Localize sections 4 and 5.1 images of `03-编辑器与IDE/调试器/嵌入式IDE可观测性.md` to local assets
- Reformat the hardware breakpoint section in `03-编辑器与IDE/调试器/嵌入式IDE可观测性.md` for consistent headings and indentation

### Removed

- Remove an unreferenced duplicate JTAG diagram

## [2026-09] - 2026-09-30

### Added

- **VSCode IPC construction** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `MessagePortMain` bridge, Caller-Service request-response, and `ChannelClient`/`ChannelServer`
- **Virtual `vscode` module** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `Module._load()` hijacking and the external extension vs internal contrib mechanisms
- **VSCode extension mechanism** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — plugin loading, extension host startup, and activation
- **Markdown editor analysis** (`03-编辑器与IDE/桌面应用/MarkDown编辑器.md`) — Markdown syntax and the `marked` parser
- **LSP and DAP** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — language servers/clients and the debug adapter protocol
- **VSCode IoC/DI and module loading** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `createDecorator`, `InstantiationService`, `SyncDescriptor`, and the AMD loader
- **VSCode IoC/DI note** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — service identifiers, dependency injection, and the service architecture
- **Barrier** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — synchronizing async operations with `wait()`/`open()`
- **VSCode IPC communication** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — protocol/channel/connection concepts and IPC examples
- **Emitter and observer pattern** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `Emitter.fire`, `get event()`, and `EmitterOptions`
- **Event and Emitter** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `Event<T>`, disposable listeners, and the event utility library
- **Disposable** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — the dispose pattern and resource management
- **JavaScript Proxy** (`01-编程语言/JavaScript与TypeScript/JavaScript Proxy解析.md`) — proxy syntax, handler traps, and its use cases
- **Closure** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — closures, their uses and caveats, and usage in the shared process
- **IIFE** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — local scope, state isolation, and namespace injection
- **Promise** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — states, `then`/`catch`/`finally`, `all` and `race`
- **Node.js event loop** (`01-编程语言/JavaScript与TypeScript/Nodejs事件循环机制.md`) — six phases, macro/micro task queues, and an async walkthrough
- **Webpack basics** (`02-前端开发/构建工具/Webpack基础.md`) — config, plugins, dev server, loaders, and asset modules
- **Electron basics** (`03-编辑器与IDE/桌面应用/Electron基础.md`) — process model, context isolation, IPC, sandboxing, `MessagePort`, and performance
- **TypeScript basics** (`01-编程语言/JavaScript与TypeScript/TypeScript基础知识.md`) — first script, basic types, declarations, and operators
- **Linux socket layer** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `sock` structure hierarchy, connection setup, buffers, and thundering herd
- **SSH port forwarding** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `-D`/`-L`/`-R` forwarding and the corresponding SSH APIs
- **HTTP protocol upgrade** (`03-编辑器与IDE/VSCode/VSCode解析.md`) — `Upgrade` header, WebSocket handshake, and Java/Netty/Node.js examples
- **Repository README** (`README.md`)
- **Initial content** — VSCode analysis skeleton, `02-前端开发/构建工具/gulp构建工具.md` (Vinyl/tasks/globs, VSCode win32 packaging), a daily journal entry, and the local `git-commit` skill

### Removed

- Remove the empty `pages/contents.md` (recycled by Logseq)
