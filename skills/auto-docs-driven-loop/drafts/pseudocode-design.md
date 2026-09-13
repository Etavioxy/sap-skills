# pseudocode-design

> 只记设计，不记状态。Notes 域：不被 `SKILL.md` 链接，不提供给调用 agent。「原因」列记本项被写入或修改/取消的缘由，20 字以内。

## 数据

```
阶段 Phase   = { id, status, commits[], seams[], actualCommits[] }   # commits 只是示例，actual 记实切
计划 Plan    = { phases[], resources[], decisions[] }    # decisions 含被否方向
意见 Finding = { axis: standards|spec, kind: hard|judgement, detail }
轮次 Round   = { fixedPoint, findings[], consistency }
范围 Mandate = { goalCap, phases[] }                     # 授权的一次执行范围：轮数上限 + 阶段清单
文档 DocFiles = { 概念, spec, 草稿, docs, experiments, issues, frictions, gaps }   # 各类文档的落点
```

## 伪代码

```
运行循环(Mandate, Plan):
    for phase in Mandate.phases:
        if Plan[phase].status == done: continue
        测试循环(phase, Plan[phase])
        收敛循环(Plan[phase])
        Plan[phase].status = done; 落盘(Plan); 停手等用户
    端到端验证(Plan)

测试循环(phase, PlanPhase):
    自定seam(PlanPhase.seams)                             # 粒度取最小公开面：够观察该行为即可
    先写测试(PlanPhase.seams); 实现至通过(PlanPhase.seams)
    执行原子提交(PlanPhase); 落盘(PlanPhase)

收敛循环(PlanPhase):
    repeat:
        report = invoke code-review(fixedPoint = HEAD)
        if 代码与文档描述不一致: 改代码 or 改文档              # 二者必改其一
        for finding in report.findings:
            if finding.kind == hard: 修复(finding); 执行原子提交(PlanPhase)
            else: 改文档                                   # judgement，不提交
        记轮次(Round); 落盘(PlanPhase)
    until report.findings 中 无 hard

执行原子提交(PlanPhase):
    for 任务 in PlanPhase.commits:                         # plan 列出的提交任务，逐个执行
        执行(任务)
        while 可通过编译(任务) or 代码量变大(任务):
            提交(任务); PlanPhase.actualCommits += 任务

端到端验证(Plan):
    for case in 关键路径用例:
        独立执行(case); 记证据                             # 不复用上轮结论
        if 失败: return 收敛循环(Plan)
```

## preparePlan——plan 各个字段的建成

```
preparePlan(概念) -> Plan:
    Plan.phases     = 按架构依赖排序(概念)
    Plan.phases[].commits   = 生成阶段提交(概念)
    Plan.resources  = 登记分支与路径(概念)
    Plan.decisions  = 记决策与被否方向(概念)
    返回 Plan

生成阶段提交(概念) -> commits:
    基础组 = 取既定基础(概念); 不延后(基础组)
    候选 = 按真实模块切分(Plan.phases)
    for p in 候选:
        if 与目标相等(p, 切MVP阶段) or 未满足全部docs要求(p):
            丢弃(p)
    返回 基础组 ∪ 候选

未满足全部docs要求(docs) -> true|false:
    for d in docs:                                          # docs 已按 docFiles 字段分类
        if d 属必填 and !满足要求(d): return true
    return false
```

## 词法约定

| 类别 | 语言 | 例 |
|---|---|---|
| 类型名、函数名 | 中文或英文 | 阶段、运行循环、收敛循环、preparePlan 等 |
| 字段名、取值、关键字 | 英文 | `status`、`hard`、`if`、`return`、`true`/`false`、`and`/`or` 等 |
| 外部 skill 的接口原词 | 原样 | `invoke code-review`、`fixedPoint`、`hard`/`judgement` 等 |
| 动词短语 | 中文 | 落盘、停手等用户、改代码、改文档、记证据 等 |
| 数据名 | 中文 + 英文词 | 文档 DocFiles、阶段 Phase、范围 Mandate 等 |

## 外来决定

| 项 | 出自 | 原因 |
|---|---|---|
| ~~向用户确认seam~~ | 本规格（已取消） | auto 流程，无需用户参与 |
| 先写测试、实现至通过 | [[tdd]] | 红绿是测试循环的执行纪律 |
| `hard` / `judgement` | [[code-review]] | 外部原词，自造则报告对不上 |
| `axis: standards / spec` | [[code-review]] | 两轴分开，避免互相遮蔽 |
| `invoke code-review(fixedPoint)` | [[code-review]] | 固定点由 review 决定，只传不定 |
| 实施内容的写法 | [[start-a-project-outline]] 与 `docs-format.md` | 只定对账关系，防同一事实两处家 |
| ~~禁止自动调用~~ | skill frontmatter `disable-model-invocation: true` | 由 harness 拦，不在流程里演 |
| 资源章节 | [[resource-declare]] | 分支与路径格式已有定义 |
