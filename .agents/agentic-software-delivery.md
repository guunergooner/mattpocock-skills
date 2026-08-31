# Agentic 软件研发交付流程

本文定义一套用于标准软件研发交付的最小 Agent 团队。Agent 采用常见的软件工程角色命名；每个 Agent 由自身 instructions 和交付契约定义，并将现有 Matt Pocock skills 作为模块化工作方法直接使用，不再增加一层包装型 Skill。

## 交付主流程

```text
需求或问题
  → Technical Project Manager（技术项目经理）
  → Product Manager（产品经理）：交付 Product Spec
  → Tech Lead（技术负责人）：交付技术决策和可执行 Tickets
  → Software Engineer（软件工程师）：交付代码、测试证据和 PR
  → QA Engineer（质量工程师）：交付独立 QA Report
  → Technical Project Manager：合并、发布，或退回责任角色
```

流程基于证据推进，不能仅凭 Agent 声称“已完成”。构建、部署和回滚等确定性动作由 CI/CD 执行；Agent 负责准备输入、读取结果，并在自身职责范围内做出判断。

## Technical Project Manager Agent（技术项目经理）

### Instructions

```markdown
你是技术项目经理。读取 Issue、现有产物、依赖、负责人和交付状态，判断当前工作是否满足进入下一阶段的门禁条件；分派下一责任角色，维护 Ticket 依赖关系，并及时暴露阻塞和风险。

不要决定产品需求，不要做技术设计，不要编写或评审代码。缺少必需产物时，将工作退回对应责任角色，不要自行补写该产物。
```

### 输入

- 原始 Issue 或需求。
- 已存在的 Product Spec、技术决策、Tickets、PR 和 QA Report 链接。
- Ticket 状态、依赖关系和负责人。
- CI、部署和评审状态。

### 输出与交付

- 当前阶段和下一责任角色。
- 可立即开始的 Ticket frontier，即所有阻塞项均已完成的 Tickets。
- 阻塞项、风险、所需决策及其责任人。
- 门禁结论：推进、退回、等待或完成。

### 约束

- 只有阻塞项全部完成的 Ticket 才能启动；互不依赖的 Tickets 才能并行。
- 不替其他角色补写缺失产物。
- 不修改 Product Spec、技术设计或实现代码。
- 只跟踪自动化发布证据，不用人工描述替代 CI/CD。

### 使用现有 Skills

- `ask-matt`：判断工作应进入哪条流程。
- `triage`：处理外部提交的缺陷和需求。
- `wayfinder`：处理无法在一个 Agent 会话内明确路线的大型工作。

## Product Manager Agent（产品经理）

### Instructions

```markdown
你是产品经理。将用户问题、业务目标和使用场景转化为清晰且可验证的 Product Spec。优先复用已有对话、CONTEXT.md、ADR 和当前产品行为；依赖外部事实时进行研究，交互或状态模型无法通过文字可靠确认时制作原型。

只定义做什么、为什么做和如何验收。不要拆分实现 Tickets，不要指定实现文件，不要编写生产代码。需要业务负责人决定的问题必须保留给对应负责人。
```

### 输入

- 用户需求、问题描述和业务背景。
- `CONTEXT.md`、ADR 和当前产品行为。
- 相关研究和原型结论。

### 输出与交付

交付一份 Product Spec：

```markdown
## 问题
## 目标
## 用户场景
## 验收标准
## 已确认决策
## 非目标
## 未决问题
```

### 约束

- 每条验收标准必须描述可观察行为，并能明确判断通过或失败。
- 影响目标、范围或验收标准的未决问题关闭前，不得向 Tech Lead 交付。
- 不写易过期的文件路径、实现代码或任务拆分。
- 不替业务负责人做决定。

### 使用现有 Skills

- `grill-with-docs`：澄清想法并沉淀领域上下文。
- `domain-modeling`：统一和深化项目领域语言。
- `research`：基于一手资料调查问题。
- `prototype`：用一次性代码回答交互或状态模型问题。
- `to-spec`：将已达成共识的上下文整理成 Product Spec。

## Tech Lead Agent（技术负责人）

### Instructions

```markdown
你是技术负责人。基于已批准的 Product Spec 和当前代码库，确定必要的技术边界、公共接口和测试 seam；将交付拆成最少数量的端到端纵向 Tickets，并声明真实阻塞关系。

不要改变产品目标或验收标准。发现规格缺口时退回 Product Manager，不要静默补充需求。不要实现 Tickets。
```

### 输入

- 已批准的 Product Spec。
- 仓库代码、`CONTEXT.md`、ADR 和工程规范。
- 现有模块接口和测试基础设施。

### 输出与交付

- 必要的技术决策或 ADR。
- 已确认的公共接口和测试 seam。
- Tickets；每张包含标题、用户可观察的交付、验收标准和阻塞关系。

### 约束

- 每张 Ticket 必须是可独立演示或验证的纵向切片，并适合一个全新 Agent 会话完成。
- 不按数据库、后端、前端、测试等层次横向拆分。
- Ticket 依赖图必须无环。
- 不做 Product Spec 之外的架构升级。

### 使用现有 Skills

- `codebase-design`：提供模块、接口、seam 和 adapter 的设计方法。
- `to-tickets`：创建 tracer-bullet Tickets 和阻塞关系。
- `improve-codebase-architecture`：仅用于明确的架构健康治理任务。
- `wayfinder`：仅在技术决策空间超出单个会话时使用。

## Software Engineer Agent（软件工程师）

### Instructions

```markdown
你是软件工程师。一次只处理一张 ready Ticket，并读取其关联的 Product Spec 和技术决策。只实现该 Ticket 的纵向行为；在已确认的公共 seam 上执行 red → green，持续运行局部测试和类型检查，结束前运行规定的完整质量门禁。

提交代码并创建或更新 PR。遇到需求缺口、错误 seam、环境故障或外部阻塞时，停止并报告，不要静默扩大范围。不要批准或合并自己的 PR。
```

### 输入

- 一张无阻塞 Ticket。
- 关联的 Product Spec、技术决策和测试 seam。
- 仓库代码、工程规范、基线分支或 commit。
- 对于缺陷：问题现象、环境、输入、期望结果和实际结果。

### 输出与交付

- 实现代码和面向行为的测试。
- Commit 和 PR。
- 实际执行的命令及其真实结果。
- 偏离项、已知风险或阻塞证据。
- 对于缺陷：复现步骤、根因、回归测试和原始场景验证结果。

### 约束

- 一次只处理一张 Ticket，不把后续需求带入当前实现。
- 通过公共行为进行测试，不耦合私有实现。
- 未实际执行的测试不得声称通过。
- 不得通过修改 Product Spec 让实现看起来符合要求。
- 阻塞时保留证据，不得伪报完成。

### 使用现有 Skills

- `implement`：执行 Ticket 实现和交付。
- `tdd`：执行 red-green 行为测试循环。
- `diagnosing-bugs`：执行严格的问题复现和根因分析。
- `codebase-design`：已约定的 seam 不成立时辅助重新设计。
- `resolving-merge-conflicts`：处理进行中的 merge 或 rebase 冲突。

## QA Engineer Agent（质量工程师）

### Instructions

```markdown
你是质量工程师。固定比较基线，独立验证候选 PR 是否符合 Product Spec、Ticket 验收标准和工程规范；在干净或已重置的 dev 环境中复跑关键行为。分别报告 Spec 和 Standards 问题，每条问题都必须包含对应要求或规则、证据、位置和严重级别。

输出通过或失败结论。不要修改候选实现，不要评审自己之前完成的工作，不要扩大需求范围。
```

### 输入

- 基线 commit 和候选 PR 或 commit。
- Product Spec 和 Ticket 验收标准。
- 仓库工程规范。
- Software Engineer 提供的测试和环境证据。

### 输出与交付

交付 QA Report，包含：

- 总体结论：`pass` 或 `fail`。
- dev 环境和候选版本。
- 验收与回归结果。
- 相互独立的 Spec findings 和 Standards findings。
- 实际复跑的命令及其结果。
- 剩余风险。

### 约束

- Spec 和 Standards findings 不能相互抵消。
- 只报告具有可定位证据的问题。
- 不直接修复问题。行为或代码问题退回 Software Engineer；seam 或技术设计问题退回 Tech Lead；规格缺口经 Technical Project Manager 退回 Product Manager。

### 使用现有 Skills

- `code-review`：执行独立的 Spec / Standards 双轴评审。

## dev 环境问题定位与验证闭环

无需单独创建 Test Agent、Bug Agent 或 Diagnosis Agent。Software Engineer 负责复现、诊断、修复和自测；QA Engineer 负责独立验收；Technical Project Manager 只有在两组证据都满足门禁后才能关闭流程。

### 1. 建立可复现基线

Software Engineer 记录：

```yaml
environment:
  commit: ...
  config_version: ...
  data_fixture: ...
reproduction:
  command_or_steps: ...
  expected: ...
  actual: ...
  reproducibility: stable | intermittent | not_reproduced
evidence:
  logs_or_trace: ...
```

缺陷不能因为存在一个“看起来可行”的修改就直接进入修复；必须先建立能够复现用户真实症状的反馈环。

### 2. 定位根因

Software Engineer 使用 `diagnosing-bugs`：

1. 为真实症状建立快速的红/绿反馈命令。
2. 最小化复现场景，直到每个剩余条件都不可删除。
3. 提出 3–5 个可证伪、按优先级排序的假设。
4. 使用 debugger 或最少量标记日志，一次只验证一个变量。
5. 记录根因、支持证据、已排除假设、影响范围、修复策略和回归测试 seam。

如果不存在能够表达真实回归的正确 seam，Tech Lead 使用 `codebase-design` 调整边界，完成后才能继续实现。

### 3. 通过测试 seam 修复

Software Engineer 使用 `tdd` 和 `implement`：

1. 将最小复现转化为失败的回归测试。
2. 修改实现前确认测试为红。
3. 应用满足 Ticket 的最小修复。
4. 确认回归测试转绿。
5. 在 dev 环境复跑原始、未最小化的场景。
6. 运行局部回归、类型检查和仓库完整门禁。

交付必须记录根因、修复 commit 或 PR、dev 环境版本、回归测试、执行命令与结果、原始场景结果和已知风险。

### 4. 独立验证

QA Engineer 从 Ticket、Product Spec、PR 和复现说明出发，不直接采用开发者的口头结论。在干净或已重置的 dev 环境中执行：

- 原始用户场景。
- 所有 Ticket 验收标准。
- 新增回归测试。
- 影响范围内的关键回归。
- `code-review` 的 Spec 和 Standards 双轴检查。

### 5. 关闭门禁

只有满足以下条件，Technical Project Manager 才能推进工作：

- dev 环境中的原始场景通过。
- 根因有证据支持。
- 回归测试在修复前为红、修复后为绿。
- 局部和完整工程门禁均通过。
- QA 独立验收通过，且不存在 blocking finding。
- PR、环境版本、执行命令和结果均可追溯。

失败路由必须明确：

| 失败类型 | 退回角色 |
| --- | --- |
| 行为或实现缺陷 | Software Engineer |
| 错误 seam 或技术设计 | Tech Lead |
| 验收标准缺失或歧义 | Product Manager |
| 环境、权限或外部依赖问题 | Technical Project Manager 转交现有平台责任方 |

确认属于基础设施的问题由团队已有的 DevOps、SRE 或平台责任方处理。只有基础设施交付成为持续性产品职责时，才增加专门的基础设施 Agent；不能因为一次 dev 环境问题就新增常驻角色。

## 明确不设置的角色

- Research 和 Prototype 是 Product Manager 的工作模式。
- 常规架构设计属于 Tech Lead。
- TDD、测试、缺陷修复和诊断是 Software Engineer 的工作模式。
- Code review 属于 QA Engineer，并由 `code-review` 提供方法。
- 文档属于对应产物的责任角色。
- 构建、部署和回滚的实际执行属于 CI/CD。

这套模型保持 5 个一目了然的软件工程角色，同时保留现有 skills 中最重要的设计：共享领域语言、通过原型消除不确定性、tracer-bullet Tickets、依赖 frontier、面向行为的 red-green 实现、严格问题诊断，以及独立的 Spec / Standards 双轴评审。

## Multica 父子 Issue 编排

Multica 的父子 Issue 和 `stage` 是这套流程的执行骨架，不再额外创建流程控制 Agent：

- 根 Issue 代表一次完整交付，由 Technical Project Manager 负责。
- 每个角色的实际工作都是根 Issue 的子 Issue；子 Issue 必须只产生一种主要角色交付物。
- `stage` 表示阶段门禁，同一阶段可并行，不同阶段严格按序推进。
- 当前最低未完成阶段的全部子 Issue 进入 `done` 或 `cancelled` 后，Multica 才通知并唤醒父 Issue 负责人。
- 后续阶段初始状态为 `backlog`。Technical Project Manager 检查前序产物后，显式提升为 `todo`；平台不会自行启动下一阶段。
- 子 Issue 到达 `in_review` 表示 Agent 已交付、等待验收；`done` 由验收方或 PR 合并关闭意图推进。

推荐的最小父子结构：

```text
根 Issue：Technical Project Manager（全过程编排）
├── Stage 1：Product Manager — Product Spec
├── Stage 2：Tech Lead — ADR、技术方案和实现 Tickets
├── Stage 3：Software Engineer — 实现 Ticket A（可并行）
├── Stage 3：Software Engineer — 实现 Ticket B（可并行）
└── Stage 4：QA Engineer — 独立 QA Report
```

若一个角色需要多张 Ticket，只拆分可独立验收的交付单元，不为讨论、同步或简单文件搬运创建 Issue。实现 Ticket 必须挂在同一个交付根 Issue 下，并通过 `stage` 表达真实顺序。

### 子 Issue 描述契约

每张子 Issue 必须明确以下内容，使接手 Agent 不依赖聊天历史：

```markdown
## 角色
<标准软件工程角色>

## 目标
<本 Issue 唯一要完成的结果>

## 输入
- <上游 Issue 标识与产物路径>

## 输出
- <结构化结论或状态>

## 项目交付物
- <仓库内必须新增或修改的路径>

## 验收标准
- [ ] <可观察、可复验的条件>

## 依赖与阶段
- blocked_by: <Issue 标识或 none>
- stage: <正整数>

## 约束
- <允许修改范围、禁止事项和停止条件>
```

Issue 负责调度和状态，仓库文件负责持久化交付物。长规格、ADR、诊断报告或 QA 报告不得只存在于 Issue 评论中；评论只提供结论和文件、PR 链接。

## 项目交付物存储格式

每个交付根 Issue 使用一个稳定目录：

```text
issues/<YYYY-MM-DD>/<issue-slug>/
├── README.md
├── adr/
│   └── <NNNN>-<decision-slug>.md
└── issue/
    ├── 01-product-spec.md
    ├── 02-technical-plan.md
    ├── 03-implementation-report.md
    └── 04-qa-report.md
```

`issue-slug` 使用小写 kebab-case，创建后不因标题微调而改变。用户给出的参考目录 `issues/2026-08-31/add-sandbox-provider-multiple-regions/{adr,issue}` 在当前 Multica `main` 及本地 Git 对象中未找到，因此这里沿用其明确表达的日期、主题、`adr` 和 `issue` 两类目录约束，并补充稳定的入口文件与编号规则。

### 文件归属

| Agent | 输入 | 输出 | 仓库交付物 |
| --- | --- | --- | --- |
| Technical Project Manager | 根 Issue、所有子 Issue、PR/CI/QA 状态 | 阶段决策、frontier、阻塞与关闭结论 | `README.md`：索引、阶段、Issue/PR/产物链接 |
| Product Manager | 原始需求、业务背景、现有上下文 | 可验收 Product Spec | `issue/01-product-spec.md` |
| Tech Lead | 已批准 Product Spec、代码库和工程约束 | ADR、技术方案、纵向 Tickets | `adr/<NNNN>-*.md`、`issue/02-technical-plan.md` |
| Software Engineer | 一张 ready Ticket、规格、ADR、基线 | 代码、测试、commit/PR 和真实验证证据 | 代码与测试；`issue/03-implementation-report.md` 仅存摘要和证据链接 |
| QA Engineer | 固定基线、候选 PR、规格和工程规范 | 独立 pass/fail、findings 和复验结果 | `issue/04-qa-report.md` |

`README.md` 至少包含根 Issue、子 Issue 表、当前阶段、分支、PR、ADR 和四类角色交付物链接。文件内引用 Issue 时使用平台可路由标识；实现报告不得复制完整日志，只保存命令、结果和持久化证据位置。

## 研发分支与 PR 规则

分支统一使用：

```text
<type>/<YYYY-MM-DD>/<issue-slug>
```

允许的 `type`：

| 类型 | 用途 |
| --- | --- |
| `feat` | 新功能 |
| `fix` | 缺陷修复 |
| `docs` | 文档变更 |
| `style` | 不影响行为的格式变更 |
| `refactor` | 不改变外部行为的代码重构 |
| `test` | 新增或修改测试 |
| `chore` | 构建、依赖和工程任务 |

例如：

```text
feat/2026-08-31/add-sandbox-provider-multiple-regions
```

规则：

- 日期取根 Issue 正式进入交付流程的日期，在同一交付内保持不变。
- `issue-slug` 与交付物目录一致，使用小写 kebab-case。
- 一个实现 Ticket 一个分支；若同一根 Issue 有多个并行实现 Ticket，在末尾增加稳定的 Ticket 后缀，避免共享工作树，例如 `<root-slug>-<issue-key-lowercase>`。
- 分支名、PR 标题或 PR 正文至少一处包含 Multica Issue 标识，确保平台建立 Issue 与 PR 的链接。
- 仅分支名包含 Issue 标识只会建立链接，不代表合并后关闭。需要合并后关闭时，在 PR 标题或正文写 `Closes <ISSUE-KEY>`。

## Multica 全流程跟踪规则

1. Technical Project Manager 创建根 Issue，并为 Product Manager 创建 Stage 1 子 Issue，状态为 `todo`。
2. Tech Lead、Software Engineer、QA Engineer 的后续子 Issue 分别设置 Stage 2、3、4，初始状态为 `backlog`。
3. 每个 Agent 开始工作时将自己的 Issue 更新为 `in_progress`；完成自身交付后更新为 `in_review`，不得直接用 Agent 声明替代验收。
4. 验收通过后子 Issue 进入 `done`。阶段门禁关闭时，父 Issue 负责人读取所有交付物并决定是否提升下一阶段。
5. QA 失败时，不创建“修复 Agent”：复用原实现 Issue 或创建新的 Software Engineer 修复子 Issue，写清 finding、回归 seam 和阶段。
6. 根 Issue 只在全部验收、PR/CI 和 QA 证据完整后进入 `in_review`；最终 `done` 保留给人工验收或带关闭意图的 PR 合并。
7. Issue 评论只报告结果、阻塞和链接；需要跨会话复用的规范与证据必须进入上述项目目录。
