---
name: auto-docs-driven-loop
description: 以 plan 为唯一进度真源的实现规格——TDD 原子提交、review 循环到无严重意见、多次 e2e。触发词：按 plan 执行、review 循环、原子提交、无严重意见。
disable-model-invocation: true
---

# auto-docs-driven-loop

实现推进规格。`auto` 指流程获授权后的阶段循环自主推进，**不含** agent 自行启动：仅由用户显式命名或给出无歧义指令触发，不得自行调用、不得由「仓里有 plan」推断启动、不得递归重入；缺授权则冻结 goal 并落 plan 后报告。授权覆盖本次 goal 循环内的续跑，不覆盖下次启动；goal 上限与「大阶段」划分落 plan。阶段边界即 goal 循环的轮次边界：每阶段完成回写 plan 并结束本轮，下一阶段由 goal 续跑。

plan 是唯一进度真源，落 `<被实施仓根>/plans/<主题>.md`。各阶段落 commits、seams、fixedPoint、findings、consistency；plan 级落 resources、授权区、decisions（含被否方向）、e2eRounds。随决策即时更新，每阶段完成写状态回 plan 再进下一阶段。本规格不指定 plan 的写法与提交时机。

设计依据见 `docs/pseudocode-design.md`。

## 主循环

阶段内 invoke [[tdd]] 先测试后实现，够过即停，原子提交代码。阶段边界连续 invoke [[code-review]] 直到每轴都无硬违反：review 轴 = 基础两轴 ∪ plan 登记的域维度，一轴一 agent（独立上下文），各轴结论互不抵消；轴只收只看 code 可判的面，需跑程序的判据归 e2e。硬违反修代码并提交，judgement 若属误解则改文档不提交。每轴无 hard 之后才进 e2e，跑剧本到一遍过。

`阶段 status`：`pending` → `tdd`（进测试循环）→ `review_fix`（收敛循环内有 hard）→ `terminating`（收敛循环已过、无 hard）→ `done`（运行循环回写）。

e2e 轮次：`scenarioRuns` 全部 `done` 即本轮结束；未通过开下一轮，通过则结束。

## trait 与钩子

trait 是类型修饰，声明一次；钩子返回 `Ok | Rejected(reason) | Purpose(action)`；钩子的 `this` 是 trait 挂在的文件实例。

| trait | 钩子 | 判据 |
|---|---|---|
| `append-only` | `onEdit(old, new)` | old 是 new 的前缀 |
| `entry-append-only` | `onEdit(old, new)` | 逐条目同 `append-only` |
| `exclusive-commit` | `onCommit(files)` | files 只含本文件 |
| `commit-now` | `onEdit` | 修完即 `Purpose(commit)` |
| `no-commit` | `onCommit(files)` | 一律 Rejected |
| `end-of-phase` | `onEdit` / `onNextPhase(phase)` | 阶段到 `terminating` / 本文件已提交 |
| `end-of-e2e-round` | `onEdit` / `onNextE2eRound(round)` | 本轮已跑完 / 本 scenario 已过 |

## 端到端验证

e2e = 任何非单元测试且需启动整个程序的过程；单元测试不属 e2e，归测试循环。

e2e 在 review 全面完成（每轴无 hard）之后才开始；e2e 里发现缺陷时，处置见下节「缺陷处置」。

e2e 写**剧本**（`Scenario`），不写脚本：脚本跑命令序列，剧本跑「用户会怎么用」。剧本写完先 review。

剧本执行用独立上下文——e2e 要发现的正是主 agent 自己的观测与调用盲区。

卡点三条：agent 无法观测、无法调用、调用超时。任何超时都不算一遍过，记为卡点。

## 缺陷处置

发现一个缺陷 ⇒ 派一个独立上下文的 agent **后台**深入调查该代码问题（仍是 review，复用 `code-review`，入参 = 缺陷 + 现场说明 + 已知形状）；**调查不阻塞 e2e**，结论回来即处置，本轮结论把在飞调查计入：

- **`hard`**：修复 + 原子提交，该剧本回 pending（下一轮重跑）—— e2e 之后允许改代码；
- **`judgement`**：改文档；
- **覆盖缺口**：立为新的 e2e 剧本（e2e 期间不得新增 review 轴：轴只在 review 阶段由 `建模评审轴` 定）。

禁止只修那一处。

## 时间与超时

**超时必须最小且尽快触发**：禁止 sleep、timeout 从小起步逐级增大、任何超时都记为卡点。

理由：auto 流程没有人在场。卡点如果不能自己暴露，就会静默消耗 goal 轮数。

timeout 推荐值回写剧本。

等待循环必须绑定终止条件并设有限次数。

## Auto 规则

| 情景 | 规则 | 依据 |
|---|---|---|
| AskUserQuestion | 不调用 AskUserQuestion harness tool | 刻意无人值守 |
| 需用户选择方案 | elegant code 判断取最优，产出 report，派 subagent review，落盘 spec / docs（不引用 report） | 无人应答时由 agent 代裁，故须独立上下文复核 |
| 通用 e2e 过程 | 超时必须最小且尽快触发 | auto 流程没有人在场 |

## 规格与对账

实施内容归 [[start-a-project-outline]]（SAP）与 `docs-format.md` 管辖；本规格定对账关系：每轮 review 核对代码与文档描述，不一致则改其中一方，结果进 plan。review 读 spec、docs、concepts；可改 spec 与 docs。主动 docs 修改记决策，并调 `docs-review(docsDiff())`（签名自造，非该 skill 原有）。

## 提交边界

可提交类别仅代码。文档、review 记录、决策与歧义 diff 均不进提交，仅用户同意时提交；docs 全程不提交，只交付 diff。
