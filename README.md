# sap-skills

English | [中文](README.zh.md)

[![skills.sh](https://img.shields.io/badge/skills.sh-sap--skills-8A2BE2)](https://www.skills.sh/skills/etavioxy/sap-skills)

Start A Project Skills — my project-stage conventions with fail-fast DDD multi-checks.

ideas → concepts → spec → experiments → structure & src → plan & acceptance

See [VISION.md](VISION.md) for the vision and the why behind this repo.

## Install

### One-line install

```bash
# Install
npx skills add Etavioxy/sap-skills

# Uninstall
npx skills remove sap-skills
```

### Manual install

Clone this repo, then use the script to install and manage all skills (Claude example):

```bash
# Install
./install.sh ~/.claude/skills/

# Uninstall
./install.sh ~/.claude/skills/ --uninstall
```

## External dependencies

`report-from-websearch` references the research skill from [mattpocock/skills](https://github.com/mattpocock/skills) when web research is needed. It is not distributed with this repo.

## Skills

- [start-a-project-outline](skills/start-a-project-outline/SKILL.md) — SAP framework: know where a project is and what to do next
- [structure-design](skills/structure-design/SKILL.md) — directory layout and workspace coordination for multi-product / multi-language repos
- [plan-analysis-matrix](skills/plan-analysis-matrix/SKILL.md) — compare options, focus on core pain points, pick a path
- [report-from-websearch](skills/report-from-websearch/SKILL.md) — turn research material into an evidence-graded markdown report
- [goals-gate-approver](skills/goals-gate-approver/SKILL.md) — manage evolving user constraints with a gate process
- [elegant-code-analysis](skills/elegant-code-analysis/SKILL.md) — evaluate refactor options before changing code: blast radius, consistency, trade-offs

## License

[MIT](LICENSE)
