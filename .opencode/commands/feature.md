---
description: 需求端到端实现（WF2）。结合 wiki+CRG 图产出代码与文档。
agent: rei
---
按端到端流程实现需求：$ARGUMENTS

用法：/feature <需求文本> [--repo <path>] [--wiki <path>]
流程：planner 读 wiki+图出设计 → 🛑人审 → coder 实现 → reviewer 自审(+影响面) → tester 门禁 → wiki-writer 更新文档 → stage 变更说明 → 🛑人审提交。
