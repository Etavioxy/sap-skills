---
name: start-a-project-outline
description: SAP 是一个从想法到落地的推进流程框架。如果采用 SAP 框架，在这条线上做架构推进时，用此 Skill 判断当前在哪一阶段、下一步做什么。触发词：SAP文档、brainstorming、ideas & spec & experiments & checks。
---

# Start-A-Project Outline

## 阶段与定位

| 阶段 | 产物标志 | 相关文件 |
|---|---|---|
| ideas | 一句话需求，尚无成形概念 | ideas.md |
| concepts | 核心概念已提出并对齐 | concepts.md |
| spec1 | 只有基础的设计，只包含技术栈和初步 insight，初稿已产出 | spec.md |
| spec1 > tech-stack | 技术选型已定，分工未定 | spec.md |
| spec1 > concrete-insight | 数据从产生到消费的完整链路已用真实数据/图具体化 | docs/concrete-insight.md |
| experiments | 已列 fail-fast 实验清单，正在验证 | experiments/index.md |
| experiments > brainstorming | 实验清单已定，正在对比解法方案 | — |
| experiments > proposition | 实验写成一条命题（端到端场景），写并跑通，尚未用验证产物核对 | README.md |
| experiments > checks | 建立核对清单，用验证产物逐项核对命题中的关键路径 | 验证产物 |
| experiments > validation | 已用验证产物核对结果，生成可供展示给用户的内容，待出结论 | 验证产物 |
| spec2 | 完整规格文档（随 src 阶段持续修订，不是一次定稿） | spec.md |
| spec2 > structure | spec2 的模块结构/目录布局已定，随 src 微调 | spec.md |
| spec2 > docs | 实现层文档（docs/*.md），随 src 持续修订 | docs/*.md |
| structure & src | 目录/模块结构已定，正在从实验代码迁移、跑通、强化、补文档 | spec.md |
| structure & src > first runthrough | 实验代码迁移完，薄连接层已搭好，只组装不写逻辑，跑通一次 | — |
| structure & src > elegant code | 对概念的代码设计模式优雅化：增加约束，提高抽象程度，减少耦合，可读可维护 | — |
| structure & src > reinforce | 测试框架和沙盒化正在补 | — |
| structure & src > reinforce > test framework | 端到端、突变、回归等高级测试已搭好 | — |
| structure & src > reinforce > sandboxing | 沙盒隔离已做 | — |
| plan & acceptance | 基础实现已完成，正在定验收标准和测试计划，作为下一版本的起点 | — |

看已有产物，找匹配的最细分那一行（子阶段优先于父阶段），它所属的顶层阶段就是你当前所在的阶段。

## 文档结构和职责

- **concept**（concepts.md）: ① 术语与概念定义，全系统唯一术语源 ② 概念间关系（覆盖矩阵） ③ 供各文档引用标号，不重复定义
- **spec**（spec.md）: ① 技术决策 / 选型 ② 模块结构 + 目录布局 ③ 验收基线，随 src 修订
- **docs/\*.md**（实现层文档）: ① 概念明细，实现原则 ② 推导依据，怎么体现了实现的优雅和一致性 ③ 实现落点（不含详细业务代码），随 src 修订

## 文档格式规范

各文档的具体格式见对应 format 文件（与 SKILL.md 同级，写/改文档前先通读）：

- `concepts.md` → [`concepts-format.md`](./concepts-format.md)
- `spec.md` → [`spec-format.md`](./spec-format.md)
- `docs/concrete-insight.md` → [`concrete-insight-format.md`](./concrete-insight-format.md)
- `docs/*.md` → [`docs-format.md`](./docs-format.md)
- `experiments/`（README.md + 验证产物 + index.md）→ [`experiments-format.md`](./experiments-format.md)

## 对抗收敛

相邻阶段会互相修正，允许早期产物被后期结果推翻：

- concepts ↔ concrete-insight：真实数据/脚本形态具象概念时，暴露概念的缺漏或边界不清，反过来修订概念；修订后再回填 concrete-insight。
- spec ↔ experiments：实验可以推翻或演化规格决定（技术选型等），规格同时约束实验边界，避免无结论发散。
- structure ↔ src：结构约束源码组织，源码实现中的真实约束会反推结构微调。
- structure/reinforce → experiments：实现卡住或达不到最优，回实验代码重新迭代验证。

## 多agent协作

experiments 和 reinforce 阶段通常是多 agent 并行推进的，各自在独立 worktree 里工作。你可能不是唯一一个在这个阶段工作的 agent。

## 原子提交

事务一致：错误与修复合并进同一次提交；迭代过程保留，不同的迭代步骤各自成一条 commit，不吞并成一个大提交。

### structure & src 的循环

常见节奏：迁移一次提交、跑通一次提交、调文档多次原子提交；之后每轮重构也是"原子重构→跑通→提交，调文档→提交"。
