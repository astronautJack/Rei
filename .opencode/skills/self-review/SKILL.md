---
name: self-review
description: 代码自审 checklist。Use when review/自审 当前改动；对照命名/错误处理/边界/测试覆盖/性能。
---

# self-review — 自审 checklist

对当前改动逐项检查（借鉴 PR-Agent `/review` 颗粒度）：
- [ ] 命名清晰、符合 `AGENTS.md` 约定
- [ ] 错误处理完整（异常/返回码/资源释放）
- [ ] 边界条件（空/越界/并发）
- [ ] 测试覆盖（新增用例、回归）
- [ ] 性能（无显式退化、无 N+1）
- [ ] 安全（无注入、无敏感信息泄露）

> TODO：团队补充具体 review 规则。
