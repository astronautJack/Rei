---
description: 代码 → 多级 wiki（WF1）。用 CRG 生成/刷新目标仓的 markdown wiki。
agent: rei
---
对仓库生成/刷新 markdown wiki。

参数：$ARGUMENTS
用法：/wiki-map <repo-path> [wiki-path]
- repo-path：目标代码仓库绝对路径（必填）
- wiki-path：wiki 根目录（默认 <repo>/docs/wiki）

流程：cartographer 建图 → wiki-writer 跑 `code-review-graph wiki --repo <repo>` 并落到 <wiki-path> → 覆盖前 diff 既有 wiki 交人审。
