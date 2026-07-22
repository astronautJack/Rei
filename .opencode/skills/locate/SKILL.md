---
name: locate
description: 需求→代码定位。Use when 按需求/bug 定位相关代码与业务逻辑；经 DeepWiki 问答 + Serena 符号级补充。
---

# locate — 需求→代码定位

接到需求/bug 时：
1. 先查 DeepWiki（design.md §3-B）做问答式定位，得到相关模块/文件/函数。
2. 用 Serena（find symbol / find references）做符号级精确定位补充。
3. 输出：相关代码位置清单 + 业务上下文摘要。

> DeepWiki 接入方式待定向隔壁组（见 design.md §15）；未就绪前用 Serena + references grep 临时替代。
