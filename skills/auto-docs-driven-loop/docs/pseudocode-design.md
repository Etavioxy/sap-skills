# pseudocode-design

> `auto-docs-driven-loop` 的设计依据：数据、伪代码、trait 与外来决定。

## 数据

```
trait append-only, entry-append-only, exclusive-commit, commit-now, end-of-phase, end-of-e2e-round, no-commit
    钩子返回 Ok | Rejected(reason) | Purpose(action)
    钩子的 this 是 trait 挂在的文件实例

阶段 Phase   = { id, status, commits[], seams[], findings[], consistency, fixedPoint }
                # status: pending | tdd | review_fix | terminating | done
                # commits 是计划项，findings/consistency 记本阶段 review 结论

计划 Plan    = { phases[], resources[], authorizations[], decisions[], reviewAxes[], e2eRounds[] }
              # decisions 含被否方向；reviewAxes = 本 plan 登记的域维度（基础两轴恒在，不登记）
评审轴 ReviewAxis = { name, description, websearch }   # description = 该轴判什么；websearch = 该轴是否需联网查证
授权项 Authorization = { action, target, bounds, source, expiry }   # 用户显式给出、动作级的放行
意见 Finding = { axis, kind: hard|judgement, detail }
              # axis = 该意见挂的评审轴名：基础两轴取 standards / spec，域维度取 ReviewAxis.name
授权 Mandate = { goalRoundCap, disallows = auto规则 }
文档 DocFiles = { concepts, spec, docs, issues, map, tickets, reports, agents, scenarios }
源码 Codebase = { workingRepos, canonicalRepos, references }
仓库 GitRepo = { actualCommits, docsDiff }
场景 Scenario = { id, steps, status: pending | passed | blocked }   # status 跨轮保留
轮次 E2eRound = { round, scenarioRuns, blockers, conclusion }
scenarioRun  = { testScenario, status: pending | running | done }
Harness      = { drafts, frictions, scenarios }   # 不进 review
```

`workingRepos = (GitRepo)plan.resources`——`plan.resources` 是 `[[resource-declare]]` 的散文登记，此处解释成仓对象，过程中保持。

`plan.authorizations`（授权区）：`action` / `target` / `bounds` 取具体表述——动词 + 具体对象 + 不覆盖什么（例：`改写某个只读文件` / `当次文件路径` / `只改该文件，不动同目录其余文件`），宽泛表述不成条；`expiry` 随 goal 循环、不跨启动；撤销即删行并在 `decisions` 记原因；越出三字段任一即越出授权，按缺授权处理。本区只放行 harness 与各仓 `AGENTS.md` 判为「需用户确认」类的动作，不豁免「Auto 规则」与「提交边界」。

`fixedPoint = Map(keys = workingRepos, value = workingRepos.HEAD)`——上个 phase（首个 phase 为 plan 开始）的逐仓点。

`当前Phase` = `Plan.phases` 里 `status` 未 `done` 的那一个；状态机保证至多一个。

`当前E2eRound` = `Plan.e2eRounds` 里 `scenarioRuns` 未全部 `done` 的那一个。

`基础两轴` = 名为 `standards`、`spec` 的两条评审轴（`description` 取自 [[code-review]]；恒在，不进 `reviewAxes`；`websearch` 取 false）。

复数取值即元素类型：`Plan.phases` 是 `Phase`、`Plan.e2eRounds` 是 `E2eRound`、`scenarioRuns` 是 `scenarioRun`。类型可由表达式推出时不另声明。

`Harness` 的成员不进 review：`drafts`、`frictions` 是上游记录，`scenarios` 在阶段之后才跑。

## 伪代码

```
运行循环(Mandate, Plan):
    for phase in Mandate.phases:
        if Plan[phase].status == done: continue
        测试循环(phase, Plan[phase])
        收敛循环(Plan[phase])
        if Plan[phase].findings 中 有 hard: 报出，不落 done
        Plan[phase].status = done; 落盘(Plan); 本阶段结束
    端到端验证(Plan)

测试循环(phase, PlanPhase):
    自定seam(PlanPhase.seams)                             # 粒度取最小公开面：够观察该行为即可
    先写测试(PlanPhase.seams); 实现至通过(PlanPhase.seams)
    执行原子提交(PlanPhase); 落盘(PlanPhase)

收敛循环(PlanPhase):
    repeat:
        DIFF = PlanPhase.commits + diff {issues map tickets}        # 这些状态变更是 review 对象
        for axis in 基础两轴 ∪ Plan.reviewAxes:
            report[axis] = invoke code-review(axis, DIFF)       # 独立上下文：一个 axis 一个 agent
        if 代码与文档描述不一致: 改代码 or 改文档              # 二者必改其一
        for axis in report:
            for finding in report[axis].findings:
                if finding.kind == hard: 修复(finding); 执行原子提交(PlanPhase)
                else: 改文档                                   # judgement，不提交
        本阶段决策落盘(spec, docs)                          # 主动 docs 修改记决策
        invoke docs-review(PlanPhase.docsDiff())           # docs 变更审查
        PlanPhase.findings += 各 report[axis].findings
        PlanPhase.consistency = 一致性结论
        落盘(PlanPhase)
    until 每个 report[axis].findings 中 无 hard

端到端验证(Plan):
    启动轮次(round = 1)

启动轮次(round):
    if round > 1:
        verdict = end-of-e2e-round.onNextE2eRound(E2eRound(round - 1))
        if verdict 是 Reject: 报出，不开下一轮                       # 有 scenario 未跑完，不静默续跑
    当前E2eRound = { round: round, scenarioRuns: [], blockers: [], conclusion: null }
    for scenario in DocFiles.scenarios:
        if scenario.status != passed:
            当前E2eRound.scenarioRuns += { testScenario: scenario, status: pending }
    运行轮次()

运行轮次():
    listen(调查缺陷 返回 findings) trap:                      # 全程监听：回调一回来即处置，不等 run 边界
        for finding in findings:
            if finding.kind == hard: 修复(finding); 原子提交; 缺陷.run.status = pending   # 该剧本下一轮重跑
            elif finding.kind == judgement: 改文档                                   # 同收敛循环
            if 覆盖缺口(finding): DocFiles.scenarios += 剧本(finding)                 # e2e 期间只能补剧本
    for run in 当前E2eRound.scenarioRuns:
        run.status = running
        按 run.testScenario.steps 模拟用户行为; 记证据          # 禁 sleep，timeout 由小逐级增大
        if 出现卡点:                                          # 无法观测 / 无法调用 / 超时
            run.testScenario.status = blocked
            当前E2eRound.blockers += 卡点
        run.status = done
        回写(run.testScenario.timeout → run.testScenario.steps)
    if 发现缺陷:
        DIFF = ∪ Plan.phases[].commits                              # e2e 侧看整个 plan 的改动汇总
        invoke code-review(缺陷, DIFF, callback = 调查缺陷)      # 独立上下文后台跑 review：background 不阻塞
    当前E2eRound.conclusion = 全部 scenarioRuns.status == done and blockers 为空 and 无在飞调查

调查缺陷(缺陷, 结论) -> findings:                            # 后台回调：结论由 code-review 带回
    返回 结论.findings
```

## preparePlan——plan 各个字段的建成

```
preparePlan(概念) -> Plan:
    Plan.phases     = 按架构依赖排序(概念)
    Plan.phases[].commits   = 生成阶段提交(phase, 概念)
    Plan.resources  = 登记分支与路径(概念)
    Plan.authorizations = 登记用户放行的动作(概念)          # 动作级 + 对象 + 边界；无则空
    Plan.decisions  = 记决策与被否方向(概念)
    Plan.reviewAxes = 建模评审轴(概念)
    返回 Plan

生成阶段提交(phase, 概念) -> commits:
    基础组 = 取既定基础(phase, 概念); 不延后(基础组)      # 既定基础先成组，不进候选再排序
    候选 = 按真实模块切分(phase)                           # 真实模块 = 架构依赖排出的模块边界
    for p in 候选:
        if 与目标相等(p, 切MVP阶段) or 未满足全部docs要求(phase):
            丢弃(p)
    返回 基础组 ∪ 候选

未满足全部docs要求(phase) -> true|false:
    for d in phase 对应的文档:                              # 依 DocFiles 各项落点
        if d 属必填 and !满足要求(d): return true
    return false

建模评审轴(概念) -> axes:
    axes = {}                                      # 只登记域维度；基础两轴恒在
    for 面 in 能力面(概念) ∪ 测试面(概念) ∪ 特有风险面(概念):      # 面 = { name, description }
        if 只看代码可判(面) and 独立可判(面) and 需独立提供改进意见(面):
            axes += {name: 面.name, description: 面.description, websearch: 需联网查证(面)}
    返回 axes

执行原子提交(PlanPhase):
    for 任务 in PlanPhase.commits:                         # 增量执行：只做尚未落提交的任务
        执行(任务)
        if 可通过编译(任务) or 代码量变大(任务):            # 够过即停，条件不回头重判
            提交(任务); GitRepo.actualCommits += 任务
```

## trait 与钩子

```
append-only:        onEdit(old, new) -> old 是 new 的前缀 ? Ok : Rejected("只追加")
entry-append-only:  onEdit(old, new) -> 逐条目同 append-only
exclusive-commit:   onCommit(files)  -> files != [this] ? Rejected("该文件需独占提交，不可与别的变更混装") : Ok
commit-now:         onEdit         -> Purpose(commit)          # 修完立即独占提交
no-commit:          onCommit(files) -> Rejected("该文件全程不提交，需要用户review")

end-of-phase:
    onEdit            -> 当前Phase.status == terminating ? Ok : Rejected("延后到阶段末")
    onNextPhase(phase) -> commits(repo.HEAD, phase.fixedPoint[repo]).files.contains(this)
                            ? Ok : Rejected("该文件在本阶段未被提交")

end-of-e2e-round:
    onEdit               -> this.status != passed and 当前E2eRound.scenarioRuns[this].status == done
                                ? Ok : Rejected("本轮未跑完")
    onNextE2eRound(round) -> this.status == passed or round.scenarioRuns[this].status == done
                                ? Ok : Rejected("该文件在本轮未通过")
```

调用时机：`onNextPhase(phase)` 在 `运行循环` 取下个 phase 之前、当前 phase 回写 `done` 后立即调用；`onNextE2eRound(round)` 在开新一轮之前、当前轮全部 `scenarioRun` 回写后调用。

## 端到端验证

脚本跑命令序列，剧本跑「用户会怎么用」——e2e 写**剧本**，不写脚本。剧本写完先 review。

卡点三条：agent 无法观测、无法调用、调用超时。任何超时都不算一遍过，记为卡点。

剧本执行在独立上下文：e2e 要发现的正是主 agent 自己的观测与调用盲区。

## 时间与超时

**超时必须最小且尽快触发**：禁止 sleep、timeout 从小起步逐级增大、任何超时都记为卡点。

理由：auto 流程没有人在场。卡点如果不能自己暴露，就会静默消耗时间，拖慢 goal 循环。

timeout 推荐值回写剧本。

## Auto 规则

| 情景 | 规则 | 依据 |
|---|---|---|
| AskUserQuestion | 不调用 AskUserQuestion harness tool | 刻意无人值守 |
| 需用户选择方案 | elegant code 详细判断取最优，并产出 report，并派 subagent review，落盘 spec / docs（不引用 report） | 无人应答时由 agent 代裁，故须独立上下文复核 |
| 通用 e2e 过程 | 超时必须最小且尽快触发 | auto 流程没有人在场 |

## 词法约定

| 类别 | 语言 | 例 |
|---|---|---|
| 类型名、函数名 | 中文或英文 | 阶段、运行循环、收敛循环、preparePlan 等 |
| 字段名、取值、关键字 | 英文 | `status`、`hard`、`if`、`return`、`true`/`false`、`and`/`or` 等 |
| 外部 skill 的接口原词 | 原样 | `invoke code-review`、`fixedPoint`、`hard`/`judgement` 等 |
| 动词短语 | 中文 | 落盘、报出、改代码、改文档、记证据 等 |
| 数据名 | 中文 + 英文词 | 文档 DocFiles、阶段 Phase、授权 Mandate 等 |

注释语言随所在段落；数据段用中文注释。

## 外来决定

「出自」写来源（禁止写「本规格」与「用户原话」）；「原因」写该决定的缘由，20 字以内。

带删除线的行 = 该决定**不进本规格流程**（由别处承担，或不采用）；列在此处只为显式排除。

| 项 | 出自 | 原因 |
|---|---|---|
| ~~向用户确认seam~~ | [[tdd]] | auto 流程，无需用户参与 |
| 先写测试、实现至通过 | [[tdd]] | 红绿是测试循环的执行纪律 |
| `hard` / `judgement` | [[code-review]] | 外部原词，自造则报告对不上 |
| 基础两轴的原词 `standards` / `spec` | [[code-review]] | 两轴分开，避免互相遮蔽 |
| `fixedPoint` 默认 = 上个 phase 的点 | [[code-review]] | 补只传不定的空；取 SHA 不取值会动的符号 |
| spec 源含 `concepts` | [[code-review]] | 判据源扩展，原列表无它 |
| 实施内容的写法 | [[start-a-project-outline]] 与 `docs-format.md` | 只定对账关系，防同一事实两处家 |
| 仓内写法规范不采用 | [[start-a-project-outline]] | SAP 无此类源，故不采 |
| ~~禁止自动调用~~ | skill frontmatter `disable-model-invocation: true` | 由 harness 拦，不在流程里演 |
| 资源章节 | [[resource-declare]] | 分支与路径格式已有定义 |
| `resource-declare(profile)` | [[resource-declare]] | 跨环境靠 profile，用户可增且 ignore |
| ~~gaps 机制~~ | [[friction-lifecycle]] | 暂不引入，先跳过 |
| `entry-append-only` 的追加语义 | [[friction-lifecycle]] | 条目与段落两级只追加 |
| `issues` 的 `exclusive-commit` | [[issue-lifecycle]] | 每次变更独占一次提交 |
| `frictions` 的 `exclusive-commit` | [[friction-lifecycle]] | 立即单独提交 |
| trait `append-only` | [[skill-authoring]] | drafts 仅 append 修改 |
| `AskUserQuestion` 禁令 | [[skill-authoring]] 与 harness | 两侧都禁；扩充为 skill + harness |
| `Scenario` / e2e 阶段 | [[start-a-project-outline]] | 阶段模型未含 e2e，故扩一阶段 |
| `docs-review(docsDiff())` | [[docs-review]] | 调用签名自造，非该 skill 原有 |
| 每轮 review 后的主动 docs 修改 | [[skill-authoring]] | 每次修改后审查结论一致性 |
