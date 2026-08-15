---
name: structure-design
description: 为多产物 / 多语言 monorepo 决定目录布局、官方脚手架命令、workspace 协调方式。触发词：脚手架、monorepo、workspace、目录结构、项目布局。
---

# Structure Design

为多产物 / 多语言 monorepo 决定目录布局、官方脚手架命令、workspace 协调方式。

## 触发场景

- 用户要"创建项目脚手架 / monorepo 结构 / workspace 布局"
- 涉及多语言（Rust + JS、Go + JS 等）或多产物（多 demo / 多 binary / 多 app）
- 已有项目要重组目录或拆分仓库结构

## 核心原则

1. **用官方脚手架命令，不手写子包配置**——`cargo new` / `pnpm create vite` / `pnpm create tauri-app` 等已经处理好默认依赖、标准约定、生态契合
2. **目录命名沿用生态既有约定**：
   - `apps/` = applications；`packages/` = shared packages（[Turborepo 文档](https://turborepo.com/docs/crafting-your-repository/structuring-a-repository)）
   - `crates/` 是 Cargo workspace 习惯目录（[Cargo Book — Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)）
3. **命名**：Rust crate 用连字符（`timeline-core`），下划线只在源码 `mod`；JS 包按需加 scope（`@<repo>/<name>`）
4. **`--vcs none`**：在 monorepo 内 `cargo new` 必须加 `--vcs none`，否则会建嵌套 `.git`

## 必须手写的最小协调文件

`cargo new` 和 `pnpm init` 都不会自动生成 workspace 总文件，这两份必须手写：

### Tauri 例外

`pnpm create tauri-app` 会强行带前端模板（vanilla / react / svelte 必选其一）。如果 Tauri 应用的前端来自 monorepo 内其它 app，可以选 vanilla 后清掉它的前端模板，或直接 `cargo new` + 手写 `tauri.conf.json` 跳过脚手架。

## 常见易漏点（参考，非强制）

- 在 monorepo 内跑 `cargo new` 时考虑加 `--vcs none`，避免嵌套 `.git`
- `.gitignore` 一般会需要 `target/` `node_modules/` `dist/`

## 反模式（避免）

- 顶层 `src/` 平铺源码——丢掉 monorepo 多产物语义
- 顶层 `backend/` `frontend/` 按角色分目录——按角色而非产物切目录，导致跨语言依赖、共享库归属、构建工具配置都被切断；同一种 workspace 工具（cargo / pnpm）也无法直接挂载
- `apps/` 下放共享库——破坏 apps = 终端产物的语义；应放 `packages/`
- `crates/` 下放 JS 内容——破坏语言并列原则
- 用 `Cargo.toml` 的 feature flag 在同一 crate 内编译两种产物（如 HTTP server 与 Tauri 二选一）——容易误开串味，应拆成两个 bin crate 物理隔离
- 用根目录 `package.json` 直接装多个 demo 的依赖，不开 pnpm workspace——demo 之间依赖无法独立、删一个 demo 牵连他人

## 结构审计 subagent

为已有项目派 subagent 输出结构 / 代码量 / 模块覆盖报告时，下面可以作为prompt参考：

- **明确 worktree 的 base ref 与 HEAD**：worktree 默认从 master（或目标分支）显式开 worktree，在 prompt 里写出预期 HEAD hash。

- **统计用命令行工具产出，不 Read 源文件手数**：行数 / 文件数用 `find ... | wc -l` 或 `tokei` / `cloc`；依赖用 `cargo tree` / `pnpm list`；commit 历史用 `git log`。直接 Read 源文件应避免。

- **统计按模块切分**：先列出 spec / design / structure 文档中声明的模块清单，代码量、文件数、位置都按模块归类。

### 报告图表参考

HTML 报告通常需要这些图表种类，按数据特征选用即可：

- **横向柱状图**：单一维度的项目排序（按行数、文件数、commit 数）
- **环形图 / 饼图**：占比构成（按目录、按文件类型、按模块）
- **纵向柱状图**：分类计数（commit 类型、错误类型、模块完成度）
- **折线图**：时间序列（逐日 commit、逐周 churn、活跃度）
- **节点关系图**：依赖图（模块互引、文件 import、agent 协作链）
- **ASCII / 树形 pre 块**：物理隔离结构、目录树、协议时序

CDN 用 Chart.js 或 ECharts 都可，单 HTML 内嵌、无构建步骤。

## 参考

- [Cargo Book — Workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html)
- [pnpm — Workspaces](https://pnpm.io/workspaces)
- [Turborepo — Structuring a repository](https://turborepo.com/docs/crafting-your-repository/structuring-a-repository)
- [Tauri v2 — Distribution](https://tauri.app/distribute/)
- [Vite — Scaffolding Your First Vite Project](https://vite.dev/guide/#scaffolding-your-first-vite-project)
- [Bun — Workspaces](https://bun.sh/docs/install/workspaces)
