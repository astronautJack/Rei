---
description: 生成 DeepWiki 风格架构 wiki（读 CRG 结构页+下钻源码，写知识库级可读文档+mermaid）。
agent: dev
---
为目标仓生成 DeepWiki 风格架构 wiki。

参数：$ARGUMENTS
用法：/wiki-doc <repo-path> [out-dir] [community]
- repo-path：目标仓绝对路径（必填）
- out-dir：wiki 输出目录（默认 <repo>/docs/wiki）
- community：只做某一个社区（slug，见结构页 index.md），省略=全部

流程（wiki-writer 执行）：code-review-graph build+wiki 出结构页 → 读每社区结构页+下钻源码 → 写架构文档页（职责/组成/原理/流程(mermaid)/模块关系/注意点/锚点 + frontmatter）→ README 索引。
