# Rei — 代码↔Wiki / 端到端 / BUG 定位 工具集

基于 opencode，对**任意路径**的本地代码仓库 + 多级 markdown wiki 提供三条能力：

1. 代码 → Wiki（`/wiki-map` 结构页 / `/wiki-doc` 架构文档）
2. 需求端到端实现（`/feature`）
3. BUG 定位（`/bug`）

## 成品接线（不自造轮子）

| 角色 | 成品 | 形态 |
|---|---|---|
| 代码图 + 影响面 + wiki 生成 | **code-review-graph**（CRG） | CLI `code-review-graph`（`~/.local/bin/`），`--repo <target>` 任意路径 |
| 上下文兜底压缩 | **opencode-dcp**（DCP） | 全局插件，自动；agent 无需调用 |

CRG 一身二任：当 **code_graph_review**（图 / blast-radius 影响面）+ 当 **DeepWiki**（`wiki` 子命令出 markdown wiki）。

## 核心思想

**先读 wiki 拿地图，再下钻代码。** wiki 是导航层（what/why），代码是真相层（how）。下钻用 CRG 的代码图引导只读相关文件；上下文快炸时 DCP 自动兜底压缩。

## 用法

- 命令以绝对路径接任意目标仓：`/wiki-map /path/to/repo /path/to/wiki`（结构页）；`/wiki-doc /path/to/repo /path/to/wiki`（DeepWiki 风格架构文档，wiki-writer 读源码写，可进知识库）。
- CRG 常用子命令（都带 `--repo <R>`）：
  - `build` 建图 / `update` 增量 / `status` 统计
  - `detect-changes --brief` 影响面（reviewer/bug-tracer）
  - `wiki` 出 markdown wiki（WF1）
  - `visualize --format json` 导出全图（cartographer/planner）
- CRG 不在 PATH 时用全路径 `$HOME/.local/bin/code-review-graph`（**Windows 生产走 Git Bash**：`$HOME` 解析为 `C:\Users\<你>`，Git Bash 自动补 `.exe`，写法不变）。
- 重复操作某仓：在该仓丢一个 mini `opencode.json`，设 `references.wiki` + `instructions` 指向该仓约定。
- 改完 `.opencode/` 或 `opencode.json` 后**重启 opencode** 才生效（配置仅启动时加载一次）。

## 约定

- **不自动 commit/push**：agent 只产本地改动 + 变更说明，提交留给人。
- **人在 loop**：`/feature` 有设计 + 提交两个硬检查点；`/bug` 报告供审、修复 opt-in；`/wiki-map` 覆盖前 diff 供审。
- **wiki-writer 只在 wiki 根下写**，不碰仓库源码。
- **CRG 副作用**：会在目标仓建 `.code-review-graph/`（图库 + 生成的 wiki）；建议加入该仓 `.gitignore`。
- **经验沉淀进 wiki**，不灌本文件 / skills。

## 模型配置

`model` / `small_model` 由全局 opencode 配置提供（指向你的端点）。本仓 `opencode.json` 不写死模型。
