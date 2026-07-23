---
description: BUG 定位（WF3）。结合 wiki + CRG 代码图定位根因。
agent: dev
---
结合 wiki 与 CRG 代码图定位 BUG 根因。

参数：$ARGUMENTS
用法：/bug <报告|失败代码路径> [--repo <path>] [--wiki <path>]
- 报告：bug 描述，或失败/新增代码的路径
- --repo：目标仓绝对路径（默认 cwd）
- --wiki：wiki 根路径

流程：bug-tracer 读 wiki 拿预期行为 → CRG 图沿数据/控制流反向回溯 → 根因假设+证据链+影响面+修复建议 → 🛑报告交人审 →（可选）coder 施修 → tester → stage。
