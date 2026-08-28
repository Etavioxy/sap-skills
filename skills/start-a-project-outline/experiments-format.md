# experiments 格式规范

SAP experiments 阶段的三类产物：`experiments/index.md`（实验清单）、`cases.*`（case 核对）、`runner.*`（校验结果展示）。

## experiments/ 路径结构

根目录下创建 `experiments/` 文件夹。子文件夹格式：`DD-snake-case`（DD 为数字序号）。

每个实验是一个独立的 package（不依赖主仓库），随语言有自己的构建配置；代码文件可能在 `src/` 里，也可能直接在实验根目录。内含 case 代码、runner 代码、README.md：

```
experiments/DD-snake-case/     ← 一个实验
├── <case 代码>                ← 包含 case 和 caseenv
├── <runner 代码>              ← 驱动验证执行
└── README.md                  ← 实验设计和结论
```

多个实验同处一个 git 仓库（monorepo）。

## experiments/index.md — 实验清单

列出距离实现 MVP 最值得 failfast 测试的实验。等用户确认后记录为 index.md。

- **格式**：三列表格（编号 / 实验名称 / 描述），描述只写实验目的/问题，一个实验一行。
- **表格下方**：每个实验一个 `## DD-name` 小节，记录该实验的：状态（通过/失败 + 验收方式）、验收、结论、skip 项、局限、**遗漏、疑虑点**（未开始的实验不写）。

```
| 编号 | 实验名称 | 描述 |
|------|---------|------|
| 01 | <名称> | <目的/问题> |

## 01-<name>

- **状态**：<通过/失败>（<验收方式>）
- **验收**：<人工/自动>
- **结论**：<验证结果>
- **skip 项**：<未跑项及原因>
- **局限**：<已知局限>
- **疑虑点**：<疑虑>
```

## cases.* — case 验收框架（代码文件）

- 根据用户的需要（story），编写 case。
- `cases.*` ——代码文件，是可运行的验收，注意不是 `.md` 文档。
- `cases.*` 是一个文件——所有 case 及其 action（script）写在这一个文件里，不拆成多个脚本文件。
- 全部测试都要放在 `cases.*` 展示出来——直接用 cases 运行，不另建 `tests/` 目录或独立测试文件。

### caseenv（可选）

- caseenv 是 case 的运行环境，与 cases 同处（代码文件）。
- **可选**——只有需要**多个不同规模和特点**的环境时才声明。
- case 用 `envs` 声明它适用的 caseenv，可在一个或多个 caseenv 上验证。

### case 文件内部结构

每个 case 声明「一个场景下应该成立什么」。结构体 + 注释：

```
case <name> {
  type:     <case 类型>            // 不同类字段不同
  story:    <story 背景>           // 必选，用自然语言全部概括
  action:   <执行被测能力的动作序列> // 可选，通常为数组——调用 runner 提供的能力，按序执行
  scene:    <场景构造>              // 可选，构造验证场景
  data:     { ... }                // 一个序列化对象，runner 注入的输入数据，结构随 action 而定
  envs:     <caseenv 列表>         // 可选，此 case 适用的 caseenv
  expect:   <期望>                 // 可选，验证应成立的结果
  reason:   <说明>                 // 可选，case 的补充说明
  ...                             // 可自行补充
}
```

## runner.* — 验证执行（被 cases import 的函数库）

`runner.*` 是函数库（experiments > validation 阶段产物）。cases 会使用 runner 的能力。作用：

1. **提供 case 的 action 所需能力**——runner 暴露能力函数，case 的 `action` 数组引用它们执行被测功能。
2. **准备基础设施函数**——起/收环境（沙盒构造）、读执行日志、场景编排等。
3. **执行验证**——runner 提供 `run_case`（构造场景 → 执行 action → 核对 expect）；涉及多 caseenv 时按 case 的 `envs` 组合验证。
