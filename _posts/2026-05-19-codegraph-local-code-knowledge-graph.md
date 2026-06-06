---
layout: post
title: "CodeGraph：为 Claude Code 打造的本地代码知识图谱"
date: 2026-05-19
category: ai-tools
tags:
  - AI 工具
  - Claude Code
  - CodeGraph
  - MCP
  - 代码知识图谱
  - tree-sitter
excerpt: "CodeGraph 用 tree-sitter 预索引代码知识图谱，并通过 MCP 为 Claude Code 提供本地符号搜索、上下文构建和影响分析能力。"
source_path: "/Users/zengxianming/Documents/Obsidian Vault/wiki/AI工具/CodeGraph本地代码知识图谱.md"
type: concept
---

> 由 [colbymchenry](https://github.com/colbymchenry) 开发的开源工具，通过 tree-sitter 预索引代码知识图谱，替代 Claude Code 运行时的文件扫描，大幅减少工具调用和 token 消耗。
>
> 仓库：[github.com/colbymchenry/codegraph](https://github.com/colbymchenry/codegraph) \| MIT 许可证

---

## 一、核心问题

当 Claude Code 面对探索性任务时，会启动 Explore 智能体通过 `grep`、`find`、`Read` 等工具反复扫描文件来理解代码结构。这个过程消耗大量 token 和时间。

**CodeGraph 的解法：** 在项目初始化时提前用 tree-sitter 解析整个代码库，构建符号关系图存入本地 SQLite。Claude Code 通过 MCP 协议查询图谱，**一次工具调用即可获得完整上下文**。

---

## 二、工作原理

### 三层架构

```
┌─────────────────────────────────────────────────────────────────┐
│                        Claude Code                               │
│  ┌─────────────────┐      ┌─────────────────┐                   │
│  │  Explore Agent  │ ──── │  Explore Agent  │                   │
│  └────────┬────────┘      └────────┬────────┘                   │
└───────────┼────────────────────────┼─────────────────────────────┘
            │                        │
            ▼                        ▼
┌───────────────────────────────────────────────────────────────────┐
│                     CodeGraph MCP Server                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐               │
│  │   Search    │  │   Callers   │  │   Context   │               │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘               │
│         └────────────────┼────────────────┘                       │
│                          ▼                                        │
│              ┌───────────────────────┐                            │
│              │   SQLite Graph DB     │                            │
│              └───────────────────────┘                            │
└───────────────────────────────────────────────────────────────────┘
```

| 层级 | 说明 |
|------|------|
| **提取层（Extraction）** | tree-sitter 解析 AST，提取节点（函数、类、方法）和边（调用、导入、继承） |
| **存储层（Storage）** | 存入本地 SQLite（`.codegraph/codegraph.db`），支持 FTS5 全文搜索 |
| **服务层（Resolution）** | 解析引用关系，框架识别补充路由→处理器关联 |

### 增量同步

MCP 服务器使用操作系统原生文件事件（FSEvents/inotify/ReadDirectoryChangesW）监听项目变化，2 秒安静窗口后触发增量同步，无需任何配置。

### 核心特性

| 特性 | 说明 |
|------|------|
| **智能上下文构建** | 一次工具调用返回入口点、相关符号和代码片段 |
| **全文搜索** | FTS5 引擎，跨代码库即时搜索 |
| **影响分析** | 追踪调用者/被调用者，修改前评估影响范围 |
| **自动同步** | 原生文件事件监听，保存即更新 |
| **19+ 语言** | 全量支持 TypeScript、Python、Go、Rust、Java 等 |
| **框架感知路由** | 识别 13 种 Web 框架的路由→处理器关联 |
| **100% 本地** | 数据不离机，无 API key，无外部服务 |

---

## 三、基准测试

### 概览

在 6 个真实代码库上的对比测试（Claude Opus 4.6 + Claude Code v2.1.91）：

> **平均：92% 更少工具调用 · 71% 更快**

| 代码库 | 语言 | CodeGraph | 不用 CodeGraph | 提升幅度 |
|--------|------|-----------|---------------|----------|
| **VS Code** | TypeScript | 3 调用，17s | 52 调用，1m37s | **94% 减少，82% 加速** |
| **Excalidraw** | TypeScript | 3 调用，29s | 47 调用，1m45s | **94% 减少，72% 加速** |
| **Claude Code** | Python+Rust | 3 调用，39s | 40 调用，1m8s | **93% 减少，43% 加速** |
| **Claude Code** | Java | 1 调用，19s | 26 调用，1m22s | **96% 减少，77% 加速** |
| **Alamofire** | Swift | 3 调用，22s | 32 调用，1m39s | **91% 减少，78% 加速** |
| **Swift Compiler** | Swift/C++ | 6 调用，35s | 37 调用，2m8s | **84% 减少，73% 加速** |

### 详细对比

**使用 CodeGraph——Explore agent 依赖 `codegraph_explore`：**

| 代码库 | 索引文件数 | 节点数 | 工具调用 | Tokens | 耗时 | 文件读取 |
|--------|-----------|--------|---------|--------|------|---------|
| VS Code | 4,002 | 59,377 | 3 | 56.6k | 17s | 0 |
| Excalidraw | 626 | 9,859 | 3 | 57.1k | 29s | 0 |
| Claude Code (Python+Rust) | 115 | 3,080 | 3 | 67.1k | 39s | 0 |
| Claude Code (Java) | — | — | 1 | 40.8k | 19s | 0 |
| Alamofire | 102 | 2,624 | 3 | 57.3k | 22s | 0 |
| Swift Compiler | 25,874 | 272,898 | 6 | 77.4k | 35s | 0 |

**不使用 CodeGraph——Explore agent 依赖 grep/find/ls/Read：**

| 代码库 | 工具调用 | Tokens | 耗时 | 文件读取 |
|--------|---------|--------|------|---------|
| VS Code | 52 | 89.4k | 1m37s | ~15 |
| Excalidraw | 47 | 77.9k | 1m45s | ~20 |
| Claude Code (Python+Rust) | 40 | 69.3k | 1m8s | ~15 |
| Claude Code (Java) | 26 | 73.3k | 1m22s | ~15 |
| Alamofire | 32 | 52.4k | 1m39s | ~10 |
| Swift Compiler | 37 | 99.1k | 2m8s | ~20 |

### 关键发现

- **零文件读取**：使用 CodeGraph 时 agent 从未回落到文件读取，完全信任 `codegraph_explore` 的结果
- **跨语言查询有效**：Python+Rust 混合代码库中能追踪跨语言边界的关系
- **大型代码库可用**：Swift Compiler（25,874 文件，272,898 节点）4 分钟内完成索引，复杂问题用 6 次 explore 调用 + 零文件读取在 35 秒内答完
- **调用链追溯**：Alamofire 基准测试中，从 `Session.request()` 到 `URLSession.dataTask()` 的 **9 步调用链** 在一个 explore 调用中完整捕获

---

## 四、安装与使用

### 1. 运行安装器

```bash
npx @colbymchenry/codegraph
```

交互式安装器自动完成：
- 检测已安装的 agent（Claude Code、Cursor、Codex CLI、opencode）
- 安装 `codegraph` 到 PATH
- 配置各 agent 的 MCP server
- 写入指令文件（`CLAUDE.md` / `.cursor/rules/codegraph.mdc` 等）
- 设置 Claude Code 自动允许权限

**非交互安装（脚本/CI）：**

```bash
codegraph install --yes                              # 自动检测 agent
codegraph install --target=cursor,claude --yes       # 指定 agent
codegraph install --target=auto --location=local     # 项目级安装
codegraph install --print-config codex               # 仅打印配置
```

| Flag | 值 | 默认 |
|------|-----|------|
| `--target` | `auto`, `all`, `none`, 或 csv | 交互提示 |
| `--location` | `global`, `local` | 交互提示 |
| `--yes` | 布尔值 | 每步确认 |
| `--no-permissions` | 跳过 Claude 自动允许 | 启用 |
| `--print-config <id>` | 打印配置片段 | — |

### 2. 重启 Agent

重启 Claude Code / Cursor / Codex CLI 以使 MCP server 生效。

### 3. 初始化项目

```bash
cd your-project
codegraph init -i
```

构建项目知识图谱索引。之后当 `.codegraph/` 目录存在时，agent 自动使用 CodeGraph 工具。

### 手动配置

```bash
npm install -g @colbymchenry/codegraph
```

添加到 `~/.claude.json`：

```json
{
  "mcpServers": {
    "codegraph": {
      "type": "stdio",
      "command": "codegraph",
      "args": ["serve", "--mcp"]
    }
  }
}
```

可选添加到 `~/.claude/settings.json`（自动允许）：

```json
{
  "permissions": {
    "allow": [
      "mcp__codegraph__codegraph_search",
      "mcp__codegraph__codegraph_context",
      "mcp__codegraph__codegraph_callers",
      "mcp__codegraph__codegraph_callees",
      "mcp__codegraph__codegraph_impact",
      "mcp__codegraph__codegraph_node",
      "mcp__codegraph__codegraph_status",
      "mcp__codegraph__codegraph_files"
    ]
  }
}
```

---

## 五、重要设计理念：子代理隔离

> **永远不要在主会话中直接调用 `codegraph_explore` 或 `codegraph_context`**——它们返回大量源码，会迅速填满主会话上下文。探索类问题应该 spawn 一个 Explore 子智能体来处理。

主会话只应使用轻量级工具：`codegraph_search`、`codegraph_callers`、`codegraph_callees`、`codegraph_impact`、`codegraph_node`。

这与 `Claude体系/微型claude/s04-subagent` 的设计理念一脉相承。

安装器会自动在 `~/.claude/CLAUDE.md` 中添加以下全局指令：

> **If `.codegraph/` exists in the project**
>
> NEVER call `codegraph_explore` or `codegraph_context` directly in the main session. These tools return large amounts of source code that fills up main session context. Instead, ALWAYS spawn an Explore agent for any exploration question.
>
> When spawning Explore agents, include this instruction:
>
> > This project has CodeGraph initialized (.codegraph/ exists). Use `codegraph_explore` as your PRIMARY tool — it returns full source code sections from all relevant files in one call.
> >
> > **Rules:**
> > 1. Follow the explore call budget in the `codegraph_explore` tool description.
> > 2. Do NOT re-read files that codegraph_explore already returned source code for.
> > 3. Only fall back to grep/glob/read for files listed under "Additional relevant files".
>
> **If `.codegraph/` does NOT exist**
>
> Ask the user: "I notice this project doesn't have CodeGraph initialized. Would you like me to run `codegraph init -i` to build a code knowledge graph?"

---

## 六、框架感知路由

CodeGraph 检测 Web 框架路由文件，将 URL 模式与处理器关联：

| 框架 | 识别模式 |
|------|----------|
| **Django** | `path()`, `re_path()`, `url()`, `include()` in `urls.py` |
| **Flask** | `@app.route('/path')`, blueprint routes |
| **FastAPI** | `@app.get(...)`, `@router.post(...)` |
| **Express** | `app.get(...)`, `router.post(...)` with middleware chains |
| **Laravel** | `Route::get()`, `Route::resource()`, `Controller@action` |
| **Rails** | `get '/x', to: 'users#index'` |
| **Spring** | `@GetMapping`, `@PostMapping`, `@RequestMapping` |
| **Gin / chi / gorilla / mux** | `r.GET(...)`, `router.HandleFunc(...)` |
| **Axum / actix / Rocket** | `.route("/x", get(handler))` |
| **ASP.NET** | `[HttpGet("/x")]` attributes |
| **Vapor** | `app.get("x", use: handler)` |
| **React Router / SvelteKit** | Route component nodes |

---

## 七、CLI 参考

```bash
codegraph                         # 交互式安装
codegraph install                 # 显式安装
codegraph init [path]             # 初始化项目（--index 同时索引）
codegraph uninit [path]           # 移除 CodeGraph
codegraph index [path]            # 全量索引（--force 重建）
codegraph sync [path]             # 增量更新
codegraph status [path]           # 查看统计
codegraph query <search>          # 搜索符号（--kind, --limit, --json）
codegraph files [path]            # 文件结构（--format, --filter, --max-depth）
codegraph context <task>          # 构建 AI 上下文
codegraph affected [files...]     # 查找受影响的测试文件
codegraph serve --mcp             # 启动 MCP server
```

### codegraph affected

通过导入依赖传递追踪，找出受变更影响的测试文件：

```bash
codegraph affected src/utils.ts src/api.ts           # 指定文件
git diff --name-only | codegraph affected --stdin     # 从 git diff 管道传入
codegraph affected src/auth.ts --filter "e2e/*"       # 自定义测试文件模式
```

| 选项 | 说明 | 默认 |
|------|------|------|
| `--stdin` | 从 stdin 读取文件列表 | `false` |
| `-d, --depth <n>` | 最大依赖遍历深度 | `5` |
| `-f, --filter <glob>` | 自定义测试文件识别模式 | 自动检测 |
| `-j, --json` | JSON 输出 | `false` |
| `-q, --quiet` | 仅输出文件路径 | `false` |

---

## 八、支持的语言

| 语言 | 扩展名 | 状态 |
|------|--------|------|
| TypeScript | `.ts`, `.tsx` | 完整支持 |
| JavaScript | `.js`, `.jsx`, `.mjs` | 完整支持 |
| Python | `.py` | 完整支持 |
| Go | `.go` | 完整支持 |
| Rust | `.rs` | 完整支持 |
| Java | `.java` | 完整支持 |
| C# | `.cs` | 完整支持 |
| PHP | `.php` | 完整支持 |
| Ruby | `.rb` | 完整支持 |
| C | `.c`, `.h` | 完整支持 |
| C++ | `.cpp`, `.hpp`, `.cc` | 完整支持 |
| Swift | `.swift` | 完整支持 |
| Kotlin | `.kt`, `.kts` | 完整支持 |
| Scala | `.scala`, `.sc` | 完整支持 |
| Dart | `.dart` | 完整支持 |
| Svelte | `.svelte` | 完整支持（Svelte 5 runes, SvelteKit routes） |
| Vue | `.vue` | 完整支持（Nuxt 路由） |
| Liquid | `.liquid` | 完整支持 |
| Pascal / Delphi | `.pas`, `.dpr`, `.dpk`, `.lpr` | 完整支持 |

---

## 九、MCP 工具接口

| 工具 | 用途 |
|------|------|
| `codegraph_search` | 按名称搜索符号 |
| `codegraph_context` | 为任务构建相关代码上下文 |
| `codegraph_callers` | 查找某函数的调用方 |
| `codegraph_callees` | 查找某函数调用的其他函数 |
| `codegraph_impact` | 分析更改某符号的影响范围 |
| `codegraph_node` | 获取单个符号详情（含源码） |
| `codegraph_files` | 获取索引的文件结构（比文件系统扫描更快） |
| `codegraph_status` | 检查索引健康状态和统计 |

---

## 十、配置

`.codegraph/config.json`：

```json
{
  "version": 1,
  "languages": ["typescript", "javascript"],
  "exclude": ["node_modules/**", "dist/**", "build/**", "*.min.js"],
  "frameworks": [],
  "maxFileSize": 1048576,
  "extractDocstrings": true,
  "trackCallSites": true
}
```

| 选项 | 说明 | 默认 |
|------|------|------|
| `languages` | 要索引的语言（留空自动检测） | `[]` |
| `exclude` | 忽略的 glob 模式 | `["node_modules/**", ...]` |
| `frameworks` | 框架提示 | `[]` |
| `maxFileSize` | 跳过大于此值的文件（字节） | `1048576` (1MB) |
| `extractDocstrings` | 提取文档字符串 | `true` |
| `trackCallSites` | 记录调用位置 | `true` |

---

## 十一、Library 用法

```typescript
import CodeGraph from '@colbymchenry/codegraph';

const cg = await CodeGraph.init('/path/to/project');

await cg.indexAll({
  onProgress: (p) => console.log(`${p.phase}: ${p.current}/${p.total}`)
});

const results = cg.searchNodes('UserService');
const callers = cg.getCallers(results[0].node.id);
const context = await cg.buildContext('fix login bug', { maxNodes: 20 });

cg.watch();   // 自动同步文件变更
cg.close();
```

---

## 十二、故障排查

| 问题 | 解决方案 |
|------|----------|
| **"CodeGraph not initialized"** | 运行 `codegraph init` 初始化项目 |
| **索引缓慢** | 检查 `node_modules` 等目录是否被排除 |
| **索引慢 / "database is locked" / WASM 回退** | 运行 `codegraph status` 查看 `Backend:` 行；若为 `wasm`，安装 C 编译工具后执行 `npm rebuild better-sqlite3` |
| **MCP 未连接** | 确认项目已初始化，验证 `codegraph serve --mcp` 能否正常运行 |
| **符号缺失** | MCP server 在保存后几秒自动同步，也可手动运行 `codegraph sync` |

**WASM 回退修复：**

```bash
# macOS
xcode-select --install

# Linux (Debian/Ubuntu)
sudo apt install build-essential python3 make

# 重新编译
npm rebuild better-sqlite3
# 或强制安装
npm install better-sqlite3 --save
```

修复后 `codegraph status` 应显示 `Backend: native`。

---

## 十三、适用场景

| ✅ 推荐 | ❌ 不推荐 |
|---------|----------|
| 中大型代码库理解任务 | 未初始化 CodeGraph 的项目 |
| 跨文件/跨模块影响分析 | 需要实时修改立即生效的场景（2s 同步延迟） |
| 调用链追溯 | |
| 路由→处理器关联查询 | |
| CI 中自动发现受影响的测试 | |
