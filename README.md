# DragonOrigin 知识库

这是一个基于 [Logseq](https://logseq.com/) 的个人知识库，主要用于记录 **VSCode 源码解析** 的学习笔记，涵盖计算机基础知识、构建工具、网络协议等相关内容。

## 目录结构

```
DragonOriginDataBase/
├── pages/          # 知识库页面（核心笔记）
│   ├── VSCode解析.md      # VSCode 架构与源码解析笔记
│   └── gulp构建工具.md    # gulp 构建工具笔记（Vinyl / 任务 / globs）
├── journals/       # 每日日志
├── whiteboards/    # Logseq 白板
├── logseq/         # Logseq 配置（config.edn、custom.css）
└── CHANGELOG.md    # 变更记录
```

## 内容主题

- **VSCode 解析**：VSCode 的定位与技术栈（Electron、TypeScript、Monaco、LSP、DAP）、计算机基础（HTTP 协议升级、WebSocket 等）
- **gulp 构建工具**：Vinyl 文件对象、任务（Tasks）、glob 模式，以及 VSCode win32 打包任务

## 使用方式

1. 使用 Logseq 打开本仓库根目录（选择 "Open an existing directory"，图谱类型选 Markdown）
2. 笔记以 Markdown 格式存储，也可以直接用任何文本编辑器阅读

## 维护

- 变更记录见 [CHANGELOG.md](CHANGELOG.md)
- 提交代码遵循本地 `git-commit` skill（`.claude/skills/git-commit`）
