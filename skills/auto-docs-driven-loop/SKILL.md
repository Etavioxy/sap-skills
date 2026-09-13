---
name: auto-docs-driven-loop
description: 以 plan 为唯一进度真源的实现规格——TDD 原子提交、review 循环到无严重意见、多次 e2e。触发词：按 plan 执行、review 循环、原子提交、无严重意见。
disable-model-invocation: true
---

# auto-docs-driven-loop

实现推进规格。`auto` 指流程获授权后的阶段循环自主推进，**不含** agent 自行启动：仅由用户显式命名或给出无歧义指令触发，不得自行调用、不得由「仓里有 plan」推断启动、不得递归重入；缺授权则冻结 goal 并落 plan 后报告。授权覆盖本次 goal 循环内的续跑，不覆盖下次启动；goal 上限与「大阶段」划分落 plan。

plan 是唯一进度真源，落 `<被实施仓根>/plan.md`：资源章节 + 各阶段原子提交表（「改了什么 + 为什么」）。随决策即时更新（含被否方向），每阶段完成写状态回 plan 再进下一阶段。本规格不指定 plan 的写法与提交时机。

## 规格与对账

实施内容归 [[start-a-project-outline]]（SAP）与 `docs-format.md` 管辖；本规格只定对账关系：每轮 review 核对代码与文档描述，不一致则改其中一方，结果进 review 记录。

## 循环

阶段内：invoke [[tdd]] 先测试后实现（seam 先与用户确认）→ 够过即停 → 原子提交代码。阶段边界：连续 invoke [[code-review]] 直到无严重意见，判据「无硬违反」；硬违反修代码并提交，判度裁量若属误解则改文档不提交。收尾：多次 e2e，各轮独立证据。可提交类别仅代码，文档、review 记录、决策与歧义 diff 均不进提交，仅用户同意时提交。
