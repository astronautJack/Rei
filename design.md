# 开发端到端 Agent 设计文档

> 以 opencode 为原型，为团队设计「需求→设计→实现→自审→测试→提交」的端到端开发 agent。
> 构建方式：**配置优先**（agents / skills / commands / references / permissions），能力缺口再补 MCP / 插件。
> 自治度：**人在 loop**——设计方案与提交两个关键节点等人确认。
> 提交策略：**本地优先**——agent 只产出本地改动 + 变更说明，由人手动提交；为后续接公司 MCP 自动提交留接口。

## 1. 目标与范围

**目标**：团队任意同事冷启动 opencode，输入一个需求 / bug，agent 借助「知识层」从需求快速定位到代码，自动走完「设计→实现→自审→测试」，产出本地改动 + 变更说明后停下等人提交。

**端到端覆盖**：
1. 需求理解与拆解（查知识层定位相关代码 / 业务逻辑）
2. 设计方案（改动点 / 风险 / 测试计划）→ 🛑 人审
3. 实现（写代码，遵循 `AGENTS.md` 约定）
4. 自审（对照约定 + review checklist）
5. 测试（build / lint / typecheck / test 全绿）
6. 产出本地改动 + 变更说明 → 🛑 人审 → 人工 `git add/commit`

**不在范围（v1）**：自动 `commit/push/merge`、CI 配置变更、跨仓重构。仓库交互的精确 CLI 命令暂不固化，先全部落本地。

## 2. 设计原则

- **配置优先**：能用 opencode 原生 agents/skills/commands/references/permissions 解决就不写插件。
- **成品优先、照搬或借鉴**：知识层与各环节先找成品方案（**DeepWiki** / **Serena** / **PR-Agent** / aider repo-map），能照搬就照搬、能借鉴就借鉴，不自造轮子（见 §3、§6）。
- **内网/本地、隐私安全**：所有工具本地或内网自托管，LLM 走公司内网端点，禁公网云 / 托管 SaaS / 遥测，代码不外发（见「隐私与内网部署约束」）。
- **人在 loop**：设计、提交两处硬检查点；其余自动。
- **本地优先、提交留接口**：agent 不碰 git 提交，只产本地改动 + 变更说明；后续公司 MCP 可无缝接上提交环节。
- **冷启动一致**：所有约定进 `AGENTS.md` / `instructions` / `references`，不依赖本地路径。
- **权限最小化**：每个 subagent 只给其职责所需权限（reviewer / tester 默认 `deny edit`）。
- **沉淀到 wiki（A），不灌 AGENTS.md/skills**：团队经验沉淀进 wiki（§3-A），`AGENTS.md` / skills 只放稳定约定，不堆积经验；A 未就绪前用 Serena memory / 临时 wiki 暂存。

## 隐私与内网部署约束（硬约束）

- **LLM 端点**：opencode 与所有工具的 LLM 调用一律指向公司内网模型端点（OpenAI 兼容 baseURL），禁用 OpenAI / Anthropic 等公网云 API。在 `opencode.jsonc` 的 `provider` 配公司端点。
- **本地/内网自托管**：开源工具必须本地或内网运行——
  - Serena 走本地 LSP 后端（`uv tool install` 本地进程，禁 JetBrains 付费插件以免外联）；
  - PR-Agent 自托管（CLI/Docker，禁 qodo 托管云），且指向公司内网模型端点而非 OpenAI；
  - **DeepWiki 采用隔壁组已验证的内网部署**（照搬，不接公网 deepwiki.com）；**运行位置优先本地（负载不大时），否则公司服务器；LLM 走公司内网 API 端点**。
- **MCP 类型**：只接 `type: local` 的 MCP（本地进程）；`type: remote` 仅限公司内网 URL，禁公网远程 MCP。
- **无遥测**：接入前核查并关闭工具遥测/外联（Serena、PR-Agent 均需查配置项）。
- **代码不外发**：三仓库源码、公司 wiki 不得离开内网；references 克隆、Serena 索引、DeepWiki 索引都在内网/本地完成。

## 3. 知识层（核心）

让 agent 从需求快速定位到代码，靠三层知识。**B 用 DeepWiki 照搬落地，C 用 Serena 落地，A 待定（亦为经验沉淀目标）。**

| 层 | 是什么 | 落地形态 | 状态 |
| --- | --- | --- | --- |
| A. 公司 wiki 知识库 | 业务需求、代码逻辑等文档，同步到本地 markdown；**亦为团队经验沉淀目标** | 作为 `reference`（上下文，可写回沉淀） | **待定，先不做** |
| B. 需求→代码导航 | 需求 / 模块 / 代码 的问答式定位 | **DeepWiki**（隔壁组内网部署，照搬） | 采用 |
| C. 代码逻辑梳理 | 调用 / 引用 / 依赖，符号级导航与重构 | **Serena MCP**（成品） | 采用 |

**B 的落地（成品照搬）**：**DeepWiki**——隔壁组已在内网部署验证，agent 经它做"需求→代码"问答式定位，免自建 L1-L4 索引。**目标运行形态**：LLM 走公司内网 API 端点；优先本地（负载不大时），否则公司服务器。接入方式（MCP / API / 本地导出）与部署细节待向隔壁组确认（见 §15）。
**C 的落地**：**Serena MCP**——语义检索 / 引用查找 / 重构 / 调试 + memory，覆盖精确符号级梳理，与 DeepWiki（wiki 问答）分工互补、重叠最小。HarmonyOS native（C++）在 Serena 支持列表内；ArkTS 需验证。
**自建 markdown wiki**：降为可选补充——当 DeepWiki 对某仓覆盖不足时，再手写 L1-L3，格式借鉴 Obsidian/Logseq 双链。

**A 的同步与回写（待定，先不做）**：公司 wiki → 本地 markdown 的读取候选——(a) 公司 wiki MCP 按需取；(b) wiki 克隆成 git repo → `references.repository`；(c) 同步 script 拉到本地 → `references.path`。**写回**：团队经验沉淀进 A，需配套可写回 wiki 的机制（同 MCP / git push / 编辑 API），随 A 一起定。

## 4. Agent 拓扑

primary `dev` 编排全程，按阶段调用 subagent；每个 subagent 独立权限。planner 经 DeepWiki 做需求→代码定位；reviewer 调 Serena 做符号梳理。

| Agent | mode | 职责 | 关键权限 |
| --- | --- | --- | --- |
| `dev`（primary） | primary | 编排端到端、人在 loop 检查点、产出变更说明 | edit: allow, bash: allow |
| `planner` | subagent | 查 DeepWiki 定位代码、拆需求、出设计 | edit: deny, bash: 只读 |
| `coder` | subagent | 按设计实现 | edit: allow, bash: allow |
| `reviewer` | subagent | 自审，对照 `AGENTS.md` / checklist / Serena | edit: deny, bash: 只读 |
| `tester` | subagent | 跑 build/lint/typecheck/test | edit: deny, bash: allow |

落地形态：`.opencode/agent/dev.md`（primary），`.opencode/agents/{planner,coder,reviewer,tester}.md`（subagent）。每个文件 frontmatter 带 `mode` / `model` / `permission` / `description`。

## 5. 端到端工作流

```
用户输入需求/bug
  │
  ▼
[dev] 调 planner（查 DeepWiki 定位相关代码与业务逻辑）
  │
[planner] 拆需求 → 出设计（改动点/风险/测试计划）
  │
  ▼ 🛑 检查点1：人审设计（dev 暂停，等人确认或改）
  │
[dev] 调 coder（按设计实现，遵循 AGENTS.md）
  │
[dev] 调 reviewer（自审 + Serena 代码梳理，出意见）
  │   └─ reviewer 发现问题 → 回 coder 修复（循环至通过）
  │
[dev] 调 tester（跑 gates：build/lint/typecheck/test）
  │   └─ 失败 → 回 coder 修复（循环至全绿）
  │
[dev] 产出本地改动 + 变更说明（diff 摘要/影响/测试结果/自审结论）
  │
  ▼ 🛑 检查点2：人审 → 人工 git add/commit/push
  │（后续可接公司 MCP 自动提交，接口已留）
```

人在 loop 实现：`dev` 在两个检查点用 `question` 工具向用户确认；未确认不进入下一阶段。

## 6. 成品工具选型（各环节优先用现成 skill / MCP）

| 环节 | 成品候选 | 部署/隐私 |
| --- | --- | --- |
| 知识/定位 | **DeepWiki**（采用，隔壁组内网）+ Serena（L4 符号层） | 内网部署；LLM 走内网端点 |
| 代码逻辑梳理 | **Serena MCP** | 本地 LSP，无外联 |
| 自审/PR | **PR-Agent**（`The-PR-Agent/pr-agent`，CLI/自托管，支持 Gitea≈gitcode；`/review` `/describe` `/improve`） | 自托管 + 内网模型(非 OpenAI)；禁 qodo 托管云 |
| 设计/实现/测试 | （先靠 opencode 原生 + `AGENTS.md`） | LLM 走内网端点 |
| 提交（暂缓） | 公司提交 MCP（后续接） | 公司内网 |

> 选型原则：先确认公司现有 MCP 清单（见 §15），与公司 MCP 重叠则优先公司版；接入前按「隐私与内网部署约束」逐项核查。

## 7. Skills

`.opencode/skills/<name>/SKILL.md`，按 `description` 关键词触发。

- `locate` — 触发词 定位/在哪；经 DeepWiki 做需求→代码查询（Serena 补符号级）。
- `self-review` — 触发词 review / 自审；团队 review checklist（借鉴 PR-Agent `/review` 颗粒度：命名 / 错误处理 / 边界 / 测试覆盖 / 性能）。
- `harmony-build` — 触发词 build / test / 编译；三仓库构建与测试命令。**命令待团队确认填入**。
- `change-summary` — 触发词 变更/summary；借鉴 PR-Agent `/describe` 生成本地变更说明（不提交）；格式规范写进 `coder` agent 文件，待后定。

## 8. Commands

`.opencode/command/<name>.md`，显式调用。

| 命令 | 作用 |
| --- | --- |
| `/feature <需求>` | 启动端到端特性开发 |
| `/bugfix <描述>` | 复现→定位→修复→测试→变更说明 |
| `/review` | 对当前改动跑 reviewer + self-review |
| `/gates` | 跑 build/lint/typecheck/test 门禁 |
| `/stage` | 产出本地改动 + 变更说明（不提交，等人审） |

每个 command 用 `$ARGUMENTS` 接参，`agent: dev`。

## 9. 上下文与记忆

- **references**：三代码仓库（`@media-playback-controller` `@scene-board-ext` + avsession SSH）；公司 wiki 本地缓存（§3-A，待加）；可选 `team-conventions`。
- **instructions**：`AGENTS.md`（稳定约定）+ `docs/avsession-access.md`。
- **知识层**：DeepWiki（B 需求→代码）+ Serena（C 符号梳理）。
- **记忆**：Serena 自带 memory 系统 + opencode session memory 做会话内记忆；跨 session 经验沉淀进 wiki（§3-A），不进 `AGENTS.md` / skills；A 未就绪前用 Serena memory / 临时 wiki 暂存。

## 10. 工具与权限边界

顶层 `permission`（`opencode.jsonc`）：
- `edit`: ask（`dev` / `coder` 子 agent 覆盖为 allow）
- `bash`: `{ "git *": "allow", "rm -rf *": "deny", "*": "ask" }`
- `external_directory`: references / wiki 目录 allow

注：v1 允许 `git *` 供人/agent 查看 status/diff，但**提交动作由人手动执行**（agent 不自动 commit/push）。MCP 只接 `type: local`（Serena）或公司内网 remote；DeepWiki 接入方式待定（见 §15）。subagent 权限见 §4。

## 11. 质量门禁

提交前必跑（tester subagent 执行，全绿才放行）：
1. lint
2. typecheck
3. unit test
4. build

命令待团队按仓库确认填入 `harmony-build` skill。失败回退 coder，循环至全绿。

## 12. 可观测与反馈

- opencode session 日志 + `share` 供团队回放。
- Serena memory 做会话内记忆；失败模式（构建错误、review 反复、定位偏差）沉淀进 wiki（§3-A），下次自动避坑。
- TODO：是否接公司可观测平台（MCP）。

## 13. 落地路线图

- **Phase 0（已完成）**：references 三代码仓库 + `AGENTS.md` + `docs/avsession-access.md`。
- **Phase 1**：接 **DeepWiki**（B，照搬隔壁组内网部署）+ **Serena MCP**（C，本地 LSP）。
- **Phase 2**：定义 `dev` primary + planner / coder / reviewer / tester subagent（含权限）。
- **Phase 3**：commands（`/feature` `/bugfix` `/review` `/gates` `/stage`）+ skills（`locate` / `self-review` / `harmony-build` / `change-summary`）。
- **Phase 4**：接两个 human-in-loop 检查点；试跑 1-2 个真实需求。
- **Phase 5**：评估能力缺口 → PR-Agent 接入（自审/变更说明）、公司 MCP 接入（含 wiki 同步 A、自动提交）、A 同步+回写机制定稿。

## 14. 参考实现

- **DeepWiki**（Cognition/Devin）：**采用**——隔壁组已在内网部署验证，照搬，作 §3-B 需求→代码定位成品。不接公网 deepwiki.com；**目标运行**：LLM 走公司内网 API 端点，优先本地（负载不大时）/ 公司服务器；部署细节待向隔壁组确认。
- **Serena**（`oraios/serena`，MIT，26.7k★）：MCP 语义代码工具包，原生集成 opencode，C/C++ 等 40+ 语言，带 memory。作 §3-C 成品落地；用本地 LSP 后端，禁 JetBrains 付费插件。
- **PR-Agent**（`The-PR-Agent/pr-agent`，MIT，12.2k★）：开源 PR 审查，CLI/自托管，支持 Gitea（gitcode 基于 Gitea），`/review` `/describe` `/improve`。作 §6 自审/变更说明候选；自托管 + 内网模型端点，禁 qodo 托管云。
- **aider repo-map**：tree-sitter + 图排序的精简代码地图，作 §3 代码侧 fallback 思路（仅借鉴算法，不引入 aider 工具）。
- **pi-rpiv**：用户体验过、评价很高，作为自审 / 提交流程参考。**待确认是否即 PR-Agent 或其它**，及关键可借鉴点（review 颗粒度？变更说明格式？交互方式？）。

## 15. 待确认（协作填）

> 涉及具体业务细节的项（命名规范、变更说明格式、build/test 命令等）统一留到后面处理，现在先空着占位，不提前臆造。

- [ ] **DeepWiki**：向隔壁组确认部署细节 + agent 接入方式（MCP / API / 本地 md 导出）；定本地 vs 公司服务器运行（按负载评估，轻负载优先本地）；LLM 走公司内网 API 端点
- [ ] opencode `provider` 指向公司内网模型端点（禁公网云）
- [ ] Serena 关闭遥测/外联，用 LSP 后端；ArkTS 覆盖验证（C++ 已支持）
- [ ] PR-Agent 自托管 + 配内网模型 baseURL（非 OpenAI），核查无遥测；gitcode（Gitea 协议）可用性
- [ ] MCP 只接 `type: local` 或公司内网 remote
- [ ] 公司现有 MCP 清单（是否已有 DeepWiki 类 / Serena 类 / 提交类，避免重复造）
- [ ] 公司 wiki 同步 + 经验回写机制（§3-A，B/C 跑通后再定）
- [ ] pi-rpiv 是不是 PR-Agent？关键可借鉴点（§14）
- [ ] 三仓库的 build / lint / typecheck / test 精确命令（填进 `harmony-build` skill）
- [ ] 团队变更说明 / 分支命名规范——写进写代码 subagent（`coder` / `reviewer`）的 agent 文件作稳定约束；具体内容待后定，先空着
- [ ] 是否需要 `team-conventions` reference
