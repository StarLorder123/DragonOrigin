# DragonOrigin 知识库

个人技术知识库，以 Markdown 笔记记录编程语言、前端、编辑器与 IDE、编译与调试、计算机基础、人工智能、后端架构等方向的学习内容。共 48 篇笔记，按「领域 → 小类」两级目录组织，完整分类见 [索引](索引.md)。

## 目录结构

```
DragonOrigin/
├── 01-编程语言/            # JavaScript与TypeScript、Rust、Go、Python与数据分析、Dart、C语言
├── 02-前端开发/            # 前端框架、构建工具
├── 03-编辑器与IDE/         # VSCode、桌面应用、调试器
├── 04-编译与工具链/         # 编译原理、LLVM
├── 05-计算机基础与系统/      # 操作系统、计算机网络、Linux与运维、网络安全
├── 06-人工智能与数据/       # 人工智能与机器学习、数据库与数据仓库
├── 07-后端与工程/          # 后端架构与高并发、软件工程
├── assets/                # 图片，按笔记名分目录
├── 索引.md                 # 全部笔记的分类索引
├── README.md              # 项目说明
└── CHANGELOG.md           # 变更记录
```

## 内容主题

- **编程语言**：JavaScript / TypeScript、Rust、Go、Python、Dart、C
- **前端与构建**：React、Vue、Webpack、gulp
- **编辑器与 IDE**：VSCode 源码解析、Electron、Markdown 编辑器、Web Shell、调试器（Codelldb、嵌入式 IDE 可观测性）
- **编译与工具链**：编译原理、LLVM 架构与核心优化 Pass
- **计算机基础与系统**：操作系统、计算机网络、Linux、Docker、网络安全
- **人工智能与数据**：NLP 与大模型实践、Word2Vec、Transformer、数据仓库与知识发现
- **后端与工程**：高并发/高可用/高性能、RPC 与 Protobuf、SSE、软件设计原则与模式

## 使用方式

1. 从 [索引](索引.md) 开始浏览，或直接进入感兴趣的分类目录
2. 笔记之间使用相对 Markdown 链接互相引用，GitHub、VS Code、Obsidian 等均可直接跳转
3. 图片存放在 `assets/<笔记名>/` 下，笔记内以 `../../assets/...` 引用，不依赖外部图床

## 维护

- 变更记录见 [CHANGELOG.md](CHANGELOG.md)
- 新增笔记：放入对应的「领域/小类」目录，并在 [`索引.md`](索引.md) 中登记
- 新增图片：放入 `assets/<笔记名>/`，引用写成 `../../assets/<笔记名>/<文件名>`
- 提交代码遵循本地 `git-commit` skill（`.claude/skills/git-commit`）
