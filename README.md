# Rei

> 基于 opencode 的代码↔Wiki / 需求端到端 / BUG 定位 工具集。
> 对**任意路径**的本地代码仓库 + 多级 markdown wiki 提供三条能力。

本仓是工具集本身（agents / commands / 配置）。目标仓作为参数传入，不动目标仓的 opencode 配置。

---

## 三条能力

| 能力 | 命令 | 一句话 |
|---|---|---|
| 代码 → Wiki | `/wiki-map`、`/wiki-doc` | 给任意本地仓生成结构页 或 面向知识库的散文架构文档 |
| 需求端到端 | `/feature` | 读 wiki+代码库 → 设计 → 实现 → 审 → 测试 → 更新 wiki |
| BUG 定位 | `/bug` | 读 wiki 拿预期 + 沿 CRG 代码图回溯根因 → 报告（修复 opt-in） |

核心思想：**先读 wiki 拿地图，再下钻代码。** wiki 是导航层（what/why），代码是真相层（how）。下钻由 CRG 代码图引导，只读相关文件。

---

## 前置条件

| 依赖 | 说明 |
|---|---|
| **opencode** ≥ 1.18 | 工具集跑在 opencode 上 |
| **Git for Windows** | **生产为 Windows**：提供 Git Bash；opencode 的 bash 工具在 Windows 上走 Git Bash，下面所有 `~`/`$HOME`/`export PATH`/`~/.bashrc` 写法直接可用（`~` = `C:\Users\<你>`） |
| **uv** + **code-review-graph** (CRG) | `uv tool install code-review-graph` → `~/.local/bin/code-review-graph`（v2.3.7+）。代码图 + 影响面 + 结构 wiki |
| **opencode-dcp** (DCP) | `opencode plugin @tarquinen/opencode-dcp@latest --global` → 全局插件，上下文兜底压缩 |
| **glm-5.2 端点** | 公司内网 LLM 端点（`model` / `small_model` 由全局 opencode 配置提供，本仓不写死） |

> opencode 的 bash PATH 默认不含 `~/.local/bin`。要么 `export PATH="$HOME/.local/bin:$PATH"`（可写进 `~/.bashrc`），要么 agent 用全路径 `$HOME/.local/bin/code-review-graph`（`AGENTS.md` 已注明）。
>
> **Windows 生产**：opencode 的 shell 指向 Git Bash（Git for Windows 自带）后，上面的 Linux 写法原样可用——`$HOME`/`~` 解析为 `C:\Users\<你>`，`~/.bashrc` 即 `C:\Users\<你>\.bashrc`，Git Bash 会自动给可执行文件补 `.exe`。当前开发在 Linux，命令一致。

---

## 安装

> Linux/macOS 直接照跑。**Windows 生产**：先装 [Git for Windows](https://git-scm.com/download/win)（带 Git Bash），让 opencode 的 shell 走 Git Bash，下面命令在 Git Bash 里原样可用。

```bash
# 1. 拿到本仓
git clone <本仓地址> Rei && cd Rei

# 2. 装 uv（Git Bash 跑这个；PowerShell 用 irm https://astral.sh/install.ps1 | iex）
curl -LsSf https://astral.sh/install.sh | sh

# 3. 装 CRG
uv tool install code-review-graph

# 4. 装 DCP 全局插件
opencode plugin @tarquinen/opencode-dcp@latest --global

# 5. 让本会话能直接找到 code-review-graph（否则 agent 用全路径，见前置说明）
export PATH="$HOME/.local/bin:$PATH"   # 永久：写进 ~/.bashrc

# 6. 在本仓启动 opencode（加载 .opencode/ + AGENTS.md + opencode.json）
opencode
```

改完 `.opencode/`、`opencode.json`、或 `~/.config/opencode/dcp.jsonc`（Windows：`C:\Users\<你>\.config\opencode\dcp.jsonc`）后**重启 opencode** 才生效（配置仅启动时加载一次）。

---

## 命令一览

所有命令在 opencode 里输入。`<repo>` 用绝对路径接任意目标仓。

| 命令 | 作用 | 参数 |
|---|---|---|
| `/wiki-map <repo> [wiki]` | WF1 **结构页**（CRG 原始社区页：Overview/Members/Flows/Dependencies） | `<repo>` 必填；`[wiki]` 默认 `<repo>/docs/wiki` |
| `/wiki-doc <repo> [out] [community]` | WF1 **散文架构档**（DeepWiki 风格、知识库级，wiki-writer 读源码写） | `[out]` 默认 `<repo>/docs/wiki`；给 `[community]` 只做单社区 |
| `/feature <需求> [--repo] [--wiki]` | WF2 需求端到端 | `--repo` 默认 cwd；`--wiki` 指定 wiki 路径 |
| `/bug <报告\|路径> [--repo] [--wiki]` | WF3 BUG 定位 | 传 bug 报告文本或失败代码路径 |
| `/review` | 当前改动自审（checklist + CRG 影响面） | — |
| `/gates` | 跑 build/lint/typecheck/test（按目标仓自动发现） | — |
| `/stage` | 汇总本地改动 + 变更说明 | — |

人在 loop：`/feature` 有设计 + 提交两个硬检查点；`/bug` 报告供审、修复 opt-in；`/wiki-map` 覆盖前 diff 供审。

---

## 快速上手

### 1. 给一个仓生成 wiki

```bash
# 结构页（快，纯结构）
/wiki-map /path/to/your-repo

# 散文架构档（慢，可读，进知识库）—— 先单社区试
/wiki-doc /path/to/your-repo /path/to/your-repo/docs/wiki src-core

# 满意后全量（去掉末尾 community 参数）
/wiki-doc /path/to/your-repo /path/to/your-repo/docs/wiki
```

### 2. 端到端实现一个需求

```
/feature 给 AVSession 加一个 X 功能，需要兼容历史接口 --repo /path/to/your-repo --wiki /path/to/your-repo/docs/wiki
```
`rei` 会先出设计（🛑检查点1）→ 实现 → 自审+影响面 → 测试 → 更新 wiki → `stage`（🛑检查点2）。

### 3. 定位一个 BUG

```
/bug 投播失败，日志报 AVSessionRadar 未注册 --repo /path/to/your-repo --wiki /path/to/your-repo/docs/wiki
```
`bug-tracer` 读 wiki 拿预期行为 + 架构 → 沿 CRG 图从症状回溯根因 → 出报告（根因/证据/修复建议），修复 opt-in。

---

## 约定与注意

- **不自动 commit/push**：工具集对目标仓只产本地改动 + 变更说明，提交留给人。
- **CRG 副作用**：会在目标仓建 `.code-review-graph/`（图库 + 生成的 wiki）。建议加进该仓 `.gitignore`。
- **wiki-writer 只在输出目录下写**，不碰仓库源码。
- **CRG `wiki` 是纯结构、不调 LLM**；要拿能进知识库的可读架构文档用 `/wiki-doc`（wiki-writer 读源码自己写）。结构页有截断（Members 前 50、Flows 前 10），无参数可解。
- **重复操作某仓**：在该仓丢一个 mini `opencode.json`（设 `references.wiki` + `instructions` 指向该仓 `AGENTS.md`），工具集自动套用约定。
- **DCP 上下文阈值**：默认 100K/50K 固定数字，按 ~128K 窗口调的。glm-5.2 窗口大，已在 `~/.config/opencode/dcp.jsonc` 改成百分比（`maxContextLimit:"75%"`、`minContextLimit:"45%"`），跟着真实窗口走。小窗口模型需再降。

---

## 目录结构

```
Rei/
├── opencode.json          # 工具集配置（默认 agent=rei、权限、compaction）
├── AGENTS.md              # agent-facing 约定（成品表、用法、约束）
├── README.md              # 本文件（用户使用说明）
├── 方案设计.md            # 给人看的设计稿（成品接线/拓扑/三流/后续）
├── .opencode/
│   ├── agents/            # rei / cartographer / wiki-writer / bug-tracer
│   │                      # / planner / coder / reviewer / tester（8 个）
│   └── commands/          # /wiki-map / /wiki-doc / /feature / /bug
│                          # / /review / /gates / /stage（7 个）
├── avsession_wiki/        # demo：avsession 结构页（gitignore）
└── avsession_wiki_prose/  # demo：avsession 散文架构档样本（gitignore）
```

详细设计见 `方案设计.md`；agent 约定见 `AGENTS.md`。
