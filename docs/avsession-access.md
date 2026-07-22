# AVSession 内网访问说明

`multimedia_av_session`（AVSession）是开源库，但公司内网无法直连公网 gitcode。源码托管在内网"极速空间"服务器上，需通过 SSH 访问。本文件被 `opencode.jsonc` 的 `instructions` 引用，会自动加载进 agent 上下文。

> ⚠️ 下方 `<...>` 为占位符，填入实际值后删除占位标记。若能拿到旧同事 AGENTS.md 原文，直接覆盖本文件即可。

## SSH 连接信息

- 端口：`<PORT>`（旧原文 `ssh -p ***` 中的端口）
- SSH 选项：`<OPTS>`（旧原文 `-o ****` 中的选项，如 `StrictHostKeyChecking=no`、`ProxyJump=...`、`IdentityFile=...` 等）
- 用户@主机：`<USER>@<HOST>`
- 服务器目录：`<SERVER_DIR>`（登录后所在/约定的服务器目录）
- AVSession 代码目录：`<CODE_DIR>`（服务器上 avsession 源码的绝对路径，`cd` 后可达）

原始交互式命令（旧同事写法，供参考，agent 不直接用）：

```
ssh -p <PORT> -o <OPTS> -t <USER>@<HOST> "cd <SERVER_DIR>; bash"
```

## agent 使用的非交互模板

opencode 的 bash 工具非交互执行，**不要用 `-t` / 交互 bash**。用如下模板在服务器上对 avsession 源码执行命令：

```
ssh -p <PORT> -o <OPTS> <USER>@<HOST> "cd <CODE_DIR> && <命令>"
```

常用示例（填好占位符后）：

- 列目录：`ssh -p <PORT> -o <OPTS> <USER>@<HOST> "cd <CODE_DIR> && ls -la"`
- 搜代码：`ssh -p <PORT> -o <OPTS> <USER>@<HOST> "cd <CODE_DIR> && grep -rn '<关键字>' ."`
- 读文件：`ssh -p <PORT> -o <OPTS> <USER>@<HOST> "cd <CODE_DIR> && cat <相对路径>"`

## 备注

- 若该服务器上的 `<CODE_DIR>` 其实是个 git 仓库（可 `git clone ssh://<USER>@<HOST>:<PORT>/<CODE_DIR>`），可改为在 `opencode.jsonc` 的 `references` 里加一条 SSH Git URL，实现冷启动自动克隆，体验更好。确认后告知，我来切换。
- 不要把 SSH 私钥 / 密码写进本文件；凭证走本机 SSH agent / `~/.ssh/config`。
