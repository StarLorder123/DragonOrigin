# DragonOrigin 知识库

这是一个基于 [Logseq](https://logseq.com/) 的个人知识库，以 Markdown 笔记的形式记录编程语言、前端、编辑器与 IDE、计算机基础、人工智能与软件工程等方向的学习内容。目前共有 46 篇笔记，统一入口见 [`pages/索引.md`](pages/索引.md)。

## 目录结构

```
DragonOrigin/
├── pages/          # 知识库页面（核心笔记）
├── assets/         # 笔记图片，按页面名称分目录存放
├── journals/       # 每日日志
├── logseq/         # Logseq 配置（config.edn、custom.css）
├── README.md       # 项目说明
└── CHANGELOG.md    # 变更记录
```

## 内容主题

- **编程语言**：JavaScript / TypeScript、Rust、Go、Python、C
- **前端与构建**：React、Vue、Webpack、gulp、Electron
- **编辑器与 IDE**：VSCode 源码解析、Codelldb、LLVM、嵌入式 IDE 可观测性
- **编译与调试**：编译原理、LSP/DAP、GDB RSP、JTAG/SWD
- **计算机基础**：操作系统、计算机网络、Linux、Docker
- **后端与架构**：高并发/高可用、RPC 与 Protobuf、SSE、Go 框架实现
- **数据与人工智能**：数据仓库与知识发现、机器学习、NLP、OpenCV、Word2Vec、Transformer
- **软件工程**：设计原则与设计模式

完整分类和每篇笔记的简介请见 [`pages/索引.md`](pages/索引.md)。

## 使用方式

1. 使用 Logseq 打开本仓库根目录（选择 "Open an existing directory"，图谱类型选 Markdown）
2. 笔记以 Markdown 格式存储，也可以直接用任何文本编辑器阅读
3. 建议从索引页 [`pages/索引.md`](pages/索引.md) 开始浏览

## 维护

- 变更记录见 [CHANGELOG.md](CHANGELOG.md)
- 图片统一存放在 `assets/<页面名>/` 下，笔记中使用 `../assets/<页面名>/...` 的相对路径引用，避免依赖外部图床
- 提交代码遵循本地 `git-commit` skill（`.claude/skills/git-commit`）
