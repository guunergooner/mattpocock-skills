# Agentic 软件研发交付流程

本文是软件交付 Squad 的总览和共享契约。可直接部署的流程与角色定义已拆分到 `squad/` 和 `agents/`，不再把 Squad 编排、Agent Instructions 和 Skill 方法混在一个文档中。

## 模块索引

### Squad

- [Software Delivery Squad](../squad/software-delivery.md)：Technical Project Manager Leader Instructions、Mermaid 流程图、父子 Issue、门禁、失败 handoff、循环熔断和人工介入。

### Agents

- [Technical Project Manager](../agents/technical-project-manager.md)：Squad Leader，负责路由和门禁，不生产专业产物。
- [Product Manager](../agents/product-manager.md)：交付可验收 Product Spec。
- [Tech Lead](../agents/tech-lead.md)：交付 ADR、技术方案和纵向 Tickets。
- [Software Engineer](../agents/software-engineer.md)：交付代码、测试、PR 和实现证据。
- [QA Engineer](../agents/qa-engineer.md)：独立交付 QA 结论和复验证据。

## 四层职责

| 层 | 唯一职责 | 存放位置 |
| --- | --- | --- |
| Squad | 流程状态机、条件路由、门禁、循环、人工升级 | `squad/` |
| Agent | 单一工程角色的职责、输入、输出和停止条件 | `agents/` |
| Skill | 可复用工作方法；直接使用现有 Matt Pocock skills | `skills/` |
| Artifact | 跨 Issue、跨会话的事实和证据 | `issues/<date>/<slug>/` |

Melos Squad 只把任务路由给 `leader_id`，不会自动 fan-out。Leader 必须使用父子 Issue 显式派发成员；Squad Instructions 只注入 Leader，Agent Instructions 只注入对应 Agent。

## 共享路径变量

所有派发必须先确定以下变量：

```yaml
delivery_date: YYYY-MM-DD
issue_slug: lowercase-kebab-case
root_issue_key: WORKSPACE-NUMBER
child_issue_key: WORKSPACE-NUMBER
artifact_root: issues/<delivery_date>/<issue_slug>
branch_type: feat | fix | docs | style | refactor | test | chore
```

例如：

```yaml
delivery_date: 2026-08-31
issue_slug: add-sandbox-provider-multiple-regions
root_issue_key: MUL-100
child_issue_key: MUL-103
artifact_root: issues/2026-08-31/add-sandbox-provider-multiple-regions
branch_type: feat
```

## 固定交付目录

```text
issues/<YYYY-MM-DD>/<issue-slug>/
├── README.md
├── issue/
│   ├── 00-delivery-state.md
│   ├── 01-product-spec.md
│   ├── 02-technical-plan.md
│   ├── 03-implementation/
│   │   └── <child-issue-key-lowercase>.md
│   └── 04-qa-report.md
└── adr/
    └── <NNNN>-<decision-slug>.md
```

路径规则：

- `issue-slug` 只使用小写字母、数字和连字符，根 Issue 开始交付后保持稳定。
- `child-issue-key-lowercase` 例如 `mul-103`，保证并行实现 Ticket 不互相覆盖报告。
- `README.md` 是唯一索引入口；必须链接根/子 Issue、当前 Stage、分支、PR、ADR 和全部交付物。
- Issue 评论只返回仓库相对路径、commit、PR 和结论；长正文必须写入上述文件。
- Artifact 必须记录生成它的 child Issue、输入版本和当前 commit，禁止用“最新版本”作为不可复现引用。

## 分支格式

```text
<branch-type>/<YYYY-MM-DD>/<issue-slug>
```

并行实现 Ticket 使用：

```text
<branch-type>/<YYYY-MM-DD>/<issue-slug>-<child-issue-key-lowercase>
```

例如：`feat/2026-08-31/add-sandbox-provider-multiple-regions`。分支、PR 标题或 PR 正文至少一处包含可路由 Issue Key。需要 PR 合并后关闭 Issue 时，在 PR 标题或正文使用 `Closes <ISSUE-KEY>`；仅在分支名出现 Issue Key 只建立链接，不表示关闭意图。

## 通用派发契约

```yaml
work_item: <child-issue-key>
root_issue: <root-issue-key>
root_issue_id: <root-issue-uuid>
previous_issue: <previous-issue-key>
role: <standard-role>
stage: <positive-integer>
attempt: <1..3>
input_refs:
  - path: <repo-relative-path>
    version: <commit-sha>
output_path: <repo-relative-path>
acceptance_criteria: [AC-001]
verification: [<reproducible-command-or-check>]
change_boundary: [<allowed-path-or-module>]
blocked_by: [<issue-key>]
```

任何必需字段缺失，Agent 必须返回 `blocked`，说明缺什么、为什么需要、由谁提供，不得自行补造关键事实。

## Skill 原则

- 不新增包装型主 Skill；流程只写在 Squad 文件。
- Agent Instructions 只写职责与交付契约，方法直接绑定现有 Matt Pocock skills。
- 创建 Agent 后需单独绑定 Skill，优先 additive `skills add`，避免 replace-all `skills set` 误删能力。
- Artifact sync 若需要，只负责写固定仓库路径并返回相对路径，不重新解释业务内容。
