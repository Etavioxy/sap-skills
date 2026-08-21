# experiments 格式规范

SAP experiments 阶段的三类产物：`experiments/index.md`（实验清单）、`cases.*`（case 核对）、`runner.*`（校验结果展示）。

## experiments/ 路径结构

根目录下创建 `experiments/` 文件夹。子文件夹格式：`DD-snake-case`（DD 为数字序号）。

每个实验子文件夹内含：case 代码、caseenv 代码、runner 代码、结果记录（具体目录名/语言随项目）：

```
experiments/DD-snake-case/     ← 一个实验
├── <case 代码>                ← 声明「什么成立」
├── <caseenv 代码>             ← 可选，case 运行环境（需要多环境时才用）
├── <runner 代码>              ← 驱动验证执行
└── <结果记录>                 ← 实验结论
```

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
  action:   <执行被测能力的动作>     // 可选，调用 runner 提供的能力
  scene:    <场景构造>              // 可选，构造验证场景
  data:     { ... }                // runner 注入的输入数据，结构随 action 而定
  envs:     <caseenv 列表>         // 可选，此 case 适用的 caseenv
  expect:   <期望>                 // 可选，验证应成立的结果
  reason:   <说明>                 // 可选，case 的补充说明
  ...                             // 可自行补充
}
```

## runner.* — 验证执行（代码文件）

`runner.*` 是**代码文件**，驱动 caseenv×case 的验证执行（experiments > validation 阶段产物）。作用：

1. **提供 case 的 action 所需能力**——case 的 action 调用 runner 提供的能力执行被测功能。
2. **准备基础设施函数**——起/收环境、读执行日志、场景编排等。
3. **执行验证**——逐个 case：起环境 → 构造场景 → 执行 action → 核对期望 → 收环境 → 输出报告；涉及多 caseenv 时按 case 的 `envs` 组合验证。
