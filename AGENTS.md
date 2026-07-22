# Rei — 研发辅助 Agent 项目说明

本目录是团队研发辅助 agent 的工作区。目标：让 opencode 在任意同事机器上**冷启动**即可访问团队的三个 HarmonyOS 媒体相关源码仓库，无需手动下载、不依赖本地路径一致。

## 三个源码仓库

| 仓库 | 访问方式 | opencode 里的入口 |
| --- | --- | --- |
| multimedia_av_session（AVSession，开源） | 内网服务器 SSH 访问（公网 gitcode 在内网不可达） | 见 `docs/avsession-access.md`（已通过 `instructions` 自动加载） |
| MediaPlaybackController（私有） | gitcode references 自动克隆 | `@media-playback-controller` |
| scene_board_ext（私有） | gitcode references 自动克隆 | `@scene-board-ext` |

### AVSession（avsession）
音视频会话框架：媒体会话、控制器、投播等。**不在 references 中**——内网只能通过 SSH 访问极速空间服务器上的源码目录。访问步骤（端口、SSH 选项、user@host、服务器目录、代码目录）见 `docs/avsession-access.md`。agent 需要读 avsession 源码时，按该文件里的非交互 SSH 命令模板在服务器上 grep/cat/ls。

### MediaPlaybackController
HMOS 媒体播放控制器 hap。私有 gitcode 仓库，已配为 reference，用 `@media-playback-controller` 引用。跟踪**开发分支**（分支名待填，见 `opencode.jsonc`）。

### scene_board_ext
HarmonyOS 场景板扩展。私有 gitcode 仓库，已配为 reference，用 `@scene-board-ext` 引用。跟踪**开发分支**（分支名待填，见 `opencode.jsonc`）。

## 冷启动前置准备（新同事必读）

1. **私有 gitcode 仓库**：确保本机 `git` 已配好 gitcode 凭证（SSH key，或 HTTPS token + `git config --global credential.helper store`）。配好后，`media-playback-controller`、`scene-board-ext` 两个 reference 会在 opencode 启动时自动克隆到本地缓存，用 `@alias` 即可访问，无需关心实际路径。
2. **AVSession 内网访问**：按 `docs/avsession-access.md` 确认能 SSH 到极速空间服务器并 `cd` 到代码目录。
3. **改完 `opencode.jsonc` / `AGENTS.md` / `docs/avsession-access.md` 后需重启 opencode** 才能生效（配置仅在启动时加载一次）。

## 代码约定与构建/测试

> TODO：以下按 HarmonyOS 通用流程给出占位，团队确认后补充准确命令与 target。

- AVSession（native）：gn + ninja 构建；具体 target 待确认。
- MediaPlaybackController / scene_board_ext：按各仓 README 与构建脚本确认（hap 可能用 hvigor；native 部分用 gn/ninja）。
- 提交前跑的 lint / 测试命令：TODO。

## 工作约定

- 优先用 `@alias` 引用 references 仓库，**不要**手动 `git clone` 到任意目录（路径会不一致，破坏冷启动一致性）。
- avsession 只能通过 `docs/avsession-access.md` 里的 SSH 模板访问，不要尝试 `git clone` 公网 gitcode（内网不可达）。
- 不要在 `opencode.jsonc` 里内嵌任何 token / 密钥 / 密码；凭证一律走本机 git / SSH 配置。
