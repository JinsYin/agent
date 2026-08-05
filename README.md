# Agent Skills

自定义 Agent 配置与技能（Skills）集合。

## 安装与使用

### 方式一：作为 skills 库

```bash
npx skills@latest add jinsyin/skills
```

### 方式二：作为 Claude Code plugin

在 Claude Code 中添加本仓库为 marketplace，再按需安装：

```
/plugin marketplace add jinsyin/skills
/plugin install gsx@jinsyin-skills
/plugin install sdd@jinsyin-skills
```

本地调试：

```bash
claude --plugin-dir plugins/gsx --plugin-dir plugins/sdd
```

## 包含 Plugin

| Plugin | 内容 | 说明 |
| --- | --- | --- |
| [`gsx`](plugins/gsx/) | 20 个 `gsx-*` skill | GSD 工作流薄前门，覆盖计划、执行、评审、UAT 全流程 |
| [`sdd`](plugins/sdd/) | 4 套 `*-best-practices` + `setup-rules` + `design-to-code` | 规范驱动开发：立规范 → 按规范产出代码 |

两个 plugin 通过符号链接复用 `skills/` 下的原始目录，**内容单一来源**：编辑 `skills/<name>/SKILL.md` 即可，plugin 侧自动生效。未被 plugin 收纳的 skill（如 `to-md`）仍可通过方式一单独使用。

## 包含技能

- `frontend-ui-best-practices` - 前端 UI 开发最佳实践
- `devops-best-practices` - DevOps 运维最佳实践
- `doc-writing-best-practices` - 文档编写最佳实践
- `spring-boot-best-practices` - Spring Boot 后端开发最佳实践
- `setup-rules` - Agent 规则配置指南
- `to-md` - 内容转换 Markdown 工具
- `design-to-code` - 将高保真设计/原型（HTML + React JSX）还原为 Vite + React + TypeScript + Tailwind + shadcn/ui 生产级代码
- `gsx-*`（20 个）- GSD 工作流的薄前门 skill，包裹 `/gsd:*` 命令并附加项目专属校验（Context7 文档核对、讨论前置等），覆盖计划、执行、评审、UAT 全流程

## 项目目录

- `skills/` - 自定义技能资源库（唯一事实来源）
- `plugins/` - Claude Code plugin 打包（`gsx`、`sdd`），内含指向 `skills/` 的符号链接
- `.claude-plugin/marketplace.json` - marketplace 清单

