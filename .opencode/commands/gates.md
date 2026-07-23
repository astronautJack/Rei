---
description: 跑质量门禁 build/lint/typecheck/test。
agent: rei
---
跑质量门禁：$ARGUMENTS

流程：tester 按仓自动发现命令，依次 lint → typecheck → unit test → build；失败交 coder 修复，循环至全绿。
