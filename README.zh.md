# sap-skills

[English](README.md) | 中文

[![skills.sh](https://skills.sh/b/Etavioxy/sap-skills)](https://skills.sh/Etavioxy/sap-skills)

Start A Project Skills：我的项目阶段规范，结合 fail-fast DDD 多重检查。

ideas → concepts → spec → experiments → structure & src → plan & acceptance

为什么做、愿景是什么——见 [VISION.zh.md](VISION.zh.md)。

## 安装

### 一键安装

```bash
# 安装
npx skills add Etavioxy/sap-skills

# 卸载
npx skills remove sap-skills
```

### 手动安装

先 clone 本仓库，用脚本安装管理所有 skills，以 Claude 为例：

```bash
# 安装
./install.sh ~/.claude/skills/

# 卸载
./install.sh ~/.claude/skills/ --uninstall
```

## 外部依赖

`report-from-websearch` 在需要 web 调研时引用 [mattpocock/skills](https://github.com/mattpocock/skills) 中的 research skill，不随本仓库分发。

## Skills

- [start-a-project-outline](skills/start-a-project-outline/SKILL.md) — SAP 流程框架：判断项目当前阶段与下一步
- [structure-design](skills/structure-design/SKILL.md) — 多产物 / 多语言仓库的目录布局与 workspace 协调
- [plan-analysis-matrix](skills/plan-analysis-matrix/SKILL.md) — 多方案梯度对比、核心痛点聚焦、推荐路径
- [report-from-websearch](skills/report-from-websearch/SKILL.md) — 把调研素材沉淀为证据分级的 markdown 报告
- [goals-gate-approver](skills/goals-gate-approver/SKILL.md) — 管理持续变化的用户约束并执行门禁

## License

[MIT](LICENSE)
