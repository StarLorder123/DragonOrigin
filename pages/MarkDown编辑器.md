- 通过解析marked开源项目，来对Markdown语法进行一个全面的解析。
- marked项目提供一个快速、可扩展且符合 CommonMark/GFM 规范的 Markdown 解析器，并附带 CLI 与浏览器/Node 两用打包。项目用 TypeScript 编写核心逻辑，在发布时编译为 JavaScript，同时配套了丰富的测试与文档。
- marked 的核心完全用纯 JavaScript/TypeScript 编写，不依赖 Node-only API，因此既能在服务器端运行，也能直接在浏览器中使用。
- 项目地址：[https://github.com/markedjs/marked](https://github.com/markedjs/marked)
- # 1.  介绍
- ## 1.1.  Markdown
- Markdown 是一种轻量级标记语言，它允许人们使用易读易写的纯文本格式编写文档。
- Markdown 语言在 2004 由约翰·格鲁伯（英语：John Gruber）创建。
- Markdown 的设计理念是"易读易写"，让人们能够使用简单的纯文本格式来编写结构化文档。
- Markdown 编写的文档可以导出 HTML 、Word、图像、PDF、Epub 等多种格式的文档。
- Markdown 编写的文档后缀为 .md, .markdown。
- Markdown 的核心特点包括：
	- 简洁性：使用直观的符号来表示格式，比如用 # 表示标题，用 * 表示列表项。这些符号在视觉上就能传达其含义，即使不进行渲染也具有良好的可读性。
	- 可读性：即使是纯文本形式的 Markdown 文档，也能清晰地展现文档的结构和层次。读者无需专门的软件就能理解内容的组织方式。
	- 便携性：Markdown 文件是纯文本格式，可以在任何文本编辑器中打开和编辑，不依赖特定的软件或操作系统。
	- 转换性：可以轻松转换为 HTML、PDF、Word 文档等多种格式，满足不同的发布需求。
- ## 1.2.  GFM
  
  GFM是GitHub Flavored Markdown的简称。是github在标准的markdown语法基础上做了扩展。用markdown来写文章，博客，文档，可以让你更加专注于文字。与其他常见的富文本编辑器相比，它至少具有以下优点：
- 纯文本，兼容性强，可以用所有文本编辑器打开
- 支持多种格式输出，例如，pdf,html等
- markdown的语法有极好的可读性
- 也可以使用 html标记
  
  **参考文档**：[https://ds-ebooks.github.io/GFM/typesetting/split-line.html](https://ds-ebooks.github.io/GFM/typesetting/split-line.html)
- # 2.  解析
- ## 2.1.  概述
- 解析链路由 6 个核心类构成（同名文件在 src/ 下）：
	- Lexer（块级扫描）
	- Tokenizer（行内/块级正则规则，负责切 token，依赖 rules.ts）
	- Parser（递归下降，把 token 序列转成节点）
	- Renderer（把节点渲染成 HTML；支持用户覆盖）
	- Hooks（preprocess / postprocess 可注入自定义逻辑）
	- Instance（封装一套可独立配置的解析器，外部 API 均代理给单例 Marked）
- 配合 defaults.ts 提供默认配置，Tokens.ts 定义所有 token 类型。
- 流程示意：
  
  ![](https://cdn.nlark.com/yuque/0/2025/png/2713067/1753582010786-76f5b30f-0d6e-4faa-a809-ad614443c28e.png)
- ## 2.2.  核心文件说明
  
  运行时流程图
  
  ![](https://cdn.nlark.com/yuque/0/2025/png/2713067/1753886914575-514f6451-c20b-4bfb-bd8a-1cb2f8a46081.png)
  
  核心文件角色说明
- marked.ts
	- 对外暴露统一入口函数 marked()，内部持有全局 Marked 实例。
	- 同步 / 异步解析、配置修改、插件注册等高级 API 都在这里做薄封装。
- Instance.ts（class Marked）
	- 负责“拼装”一次完整的 Markdown 处理流水线：
	- 提前把选项合并到 this.defaults。
	- 创建/缓存 Lexer、Parser、Renderer、Tokenizer 实例并互相关联。
	- 提供 parse / parseInline / walkTokens / use / setOptions 等方法，是扩展系统的核心。
- defaults.ts + MarkedOptions.ts
	- defaults.ts 给出所有配置项的默认值；MarkedOptions.ts 则定义 TypeScript 类型。
	- 当调用 setOptions 或 use 时，都会先把传入对象与默认值深度合并。
- rules.ts
	- 集中维护 Block 与 Inline 两大类正则。
	- 根据 gfm / pedantic / breaks 等布尔开关自动派生多套规则，供 Tokenizer 动态切换。
- Tokenizer.ts
	- 把正则规则编译成真正的解析函数；按 Block → Inline 两级结构输出 Token。
	- 支持插件在 “block” 或 “inline” 两个 level 注入自定义 tokenizer。
- Lexer.ts
	- 组织 Tokenizer 完成“整段”与“行内”双阶段的分词。
	- 解析完成后返回 TokensList，并缓存未处理的行内片段，最终保证顺序正确。
- Tokens.ts
	- 纯类型/接口文件，对所有 Token 结构（约 25 种）做了细粒度声明。
	- 供 Lexer、Parser、Renderer 和用户插件共享同一套类型。
- Parser.ts
	- 递归遍历 Token 数组，针对不同 token.type 选择对应的 Renderer 方法输出 HTML。
	- 若用户通过插件注册了 extensions.renderers[type]，优先执行扩展逻辑，可返回 false 让默认逻辑兜底。
- Renderer.ts / TextRenderer.ts
	- Renderer：把单个 Token 渲染成 HTML 字符串；内部大量使用 helpers.escape 等工具。
	- TextRenderer：输出纯文本，用于 headingIds、搜索索引等场景。
- Hooks.ts
	- 提供 beforeRender / afterRender / lexer / parser 等生命周期钩子数组。
	- Instance 在对应阶段调用，插件可异步返回 Promise 以开启全链路异步模式。
- helpers.ts
	- 一些与 Markdown 语法无关的小工具，如 escape / unescape / mangle / cleanUrl 等。