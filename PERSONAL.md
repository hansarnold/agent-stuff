# 我的 Pi 基础配置（macOS / Linux）

基于 [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff)，保留上游源码与 Apache-2.0 许可证。个人配置维护在默认分支 `personal`，`main` 保留作上游基线。macOS 和 Linux 使用相同的插件清单、配置模板与管理命令，由通知扩展自动选择系统实现。

## 跨平台约定

| 项目 | macOS | Linux |
| --- | --- | --- |
| 用户配置 | `~/.pi/agent/` | `~/.pi/agent/` |
| 插件来源 | GitHub 个人分支 + 固定版本 npm 包 | 同左 |
| 基础工具 | Pi、Git；GitHub 功能另需 `gh` | 同左 |
| 本地桌面通知 | 系统自带 `/usr/bin/osascript` | `notify-send` + 桌面通知服务 |
| SSH / 通知命令失败 | OSC 777 终端通知 | 同左 |

只同步仓库中的清单和配置模板；在各机器上用 Pi 安装包并登录账号。配置不依赖 `/home/...` 或 `/Users/...` 等绝对路径。若设置了 `PI_CODING_AGENT_DIR`，下文的 `~/.pi/agent` 应替换为该目录。

已在 Linux、Pi 0.87.1 上检查资源加载；macOS 通知分支通过模拟验证，实际系统通知仍需在 Mac 上确认。

## 默认启用

个人包白名单在根目录 `package.json` 的 `pi` 字段中。修改、提交并推送后，用 `pi update git:github.com/hansarnold/agent-stuff@personal` 更新安装，再在 Pi 内执行 `/reload`。

| 功能 | 使用方式 |
| --- | --- |
| 完成通知 | 本地 macOS 使用 `osascript`，Linux 桌面使用 `notify-send`；命令失败或通过 SSH 使用时回退到 OSC 777 终端通知 |
| TODO | `/todos`；模型也可使用 `todo` 工具；任务保存在项目 `.pi/todos` |
| 交互答题 | `/answer` 或 `Ctrl+.`，把上一条助手回复中的问题提取为交互表单 |
| 代码审查 | `/review`、`/end-review`；PR 模式需要已登录的 `gh` |
| 用量统计 | `/session-breakdown`，查看最近 7/30/90 天的会话、token 和费用记录 |
| 讨论方案 | `/discuss`，在实现前逐轮厘清需求 |
| 提交规范 | `commit` skill，可通过 `/skill:commit` 调用 |
| GitHub 操作 | `github` skill，可通过 `/skill:github` 调用；使用已登录的 `gh` 操作 Issue、PR 和 CI |
| 网页搜索、正文读取 | 单独安装固定版本 `pi-web-access@0.31.0`；模型按需调用 `web_enable` 后使用 `web_search`、`fetch_content` 和 `get_search_content`，`/search` 浏览本会话搜索记录 |
| 订阅额度 | 单独安装固定版本 `@narumitw/pi-usage@0.61.1`；`/usage` 查看当前登录账号的订阅额度与重置时间，状态栏显示剩余百分比和倒计时 |

`/answer` 会额外调用模型提取问题。保留上游的模型选择逻辑：优先可用的快速 Codex 模型，然后 Haiku，最后当前模型。它处理已经提出的问题，并非主动 `ask_user` 工具。

通知在 `agent_settled` 触发，即自动重试、压缩及排队续跑结束后。非交互调用不发送通知。macOS 使用系统自带的 [AppleScript 通知](https://developer.apple.com/library/archive/documentation/LanguagesUtilities/Conceptual/MacAutomationScriptingGuide/DisplayNotifications.html)，无需安装额外通知包；通知显示受系统通知设置和专注模式影响。Linux 桌面需要 `notify-send`（例如 Debian / Ubuntu 的 `libnotify-bin`）与可用的桌面通知服务。SSH 会通知当前终端，不会尝试在远端桌面弹窗。

终端回退需要 OSC 777 支持，如 Ghostty、iTerm2、WezTerm；macOS 的 Terminal.app 不支持此回退，但本地系统通知不依赖终端类型。通知命令成功退出也不代表系统一定显示横幅；回退同样不保证桌面一定收到通知。

其余扩展、skills 和主题保留源码但不加载，包括自动信任作者仓库、Bash/edit 替换、多代理、长期自动运行和 macOS 专用功能。以后添加功能时，把对应路径加入白名单即可。

## 安装和移除

### 新机器准备

macOS 和 Linux 均可按 [Pi 官方安装说明](https://pi.dev/docs/latest/quickstart) 安装。使用 npm 安装时，需要 Node.js 22.19 或更新版本及 npm：

```bash
npm install -g --ignore-scripts @earendil-works/pi-coding-agent
pi --version
```

确认 Git 可用；需要 GitHub 操作和 PR 审查时，安装 [GitHub CLI](https://cli.github.com/) 并执行 `gh auth login`。启动 `pi` 后，用 `/login` 登录模型服务，用 `/model` 选择模型。两台机器各自登录。

### 安装相同插件

两端执行相同命令，直接从 GitHub 安装个人分支，再安装网页和额度插件：

```bash
pi install git:github.com/hansarnold/agent-stuff@personal
pi install npm:pi-web-access@0.31.0
pi install npm:@narumitw/pi-usage@0.61.1
```

这三个声明写入 `~/.pi/agent/settings.json` 的 `packages`，示例见 [config/settings.example.json](config/settings.example.json)。保留你自己的模型和其他设置，不要用示例覆盖整个设置文件。Pi 将 GitHub 包下载到它管理的 Git 缓存，将 npm 包安装到它管理的 npm 目录；运行不依赖开发 checkout，也无需另外执行 `npm install`。

首次配置网页插件时，把 [config/web-search.json](config/web-search.json) 保存到 `~/.pi/agent/web-search.json`；如果该文件已经存在，合并所需字段。这份配置默认使用 Exa，未配置 Exa key 时走免 key 的 MCP 搜索；读取网页使用直接 HTTP 和本地正文提取。保留完整内容检索工具，默认不生成额外摘要；关闭独立来源核验、自动克隆 GitHub、图片、PDF 和视频处理。GitHub 任务交给 `github` skill + `gh`。

首次配置额度插件时，把 [config/pi-usage.json](config/pi-usage.json) 保存到 `~/.pi/agent/pi-usage.json`；已有文件则合并这三个字段。默认保持 `codexFastMode: false`，显示剩余额度与重置倒计时。`/usage` 使用当前 Pi 账号查看订阅限制；`/session-breakdown` 仍用于查看本地历史 token 和费用记录，两者含义不同。额度插件还提供 `/fast` 切换加速模式，本配置保持关闭。

网页和额度配置都是独立用户设置，Pi 安装包时不会自动复制它们。在新 Mac 或 Linux 上也需要按上文保存这两个 JSON 文件；以后修改仓库中的模板，需要同步到各机器的用户设置后重新加载。只安装个人 GitHub 包不会自动安装单独的网页或额度插件。

用 `pi list` 查看已配置的包，`pi config` 选择启用的资源。在 Pi 内检查 `/todos`、`/review`、`/usage` 是否可用，即可确认主要插件已加载。插件管理使用 [Pi 自带命令](https://pi.dev/docs/latest/packages)。

`private: true` 防止把个人 fork 意外发布为上游 npm 包。上游的 `diff` 依赖仍保留，以便后续启用其他扩展。

已有会话执行 `/reload`，或重启 Pi。移除时执行：

```bash
pi remove git:github.com/hansarnold/agent-stuff@personal
pi remove npm:pi-web-access@0.31.0
pi remove npm:@narumitw/pi-usage@0.61.1
```

Pi 的模型、登录凭据及会话继续放在用户自己的 `~/.pi/agent`，不纳入这个仓库。

## 同步上游

日常获取自己 fork 的更新：

```bash
pi update git:github.com/hansarnold/agent-stuff@personal
```

这里的 `personal` 是分支引用，更新会获取该分支最新提交。网页插件固定在 `0.31.0`，额度插件固定在 `0.61.1`，升级时要明确安装新版本。

开发 checkout 可以放在任意目录，仅用于修改、提交和合并上游；不要直接修改 Pi 管理的缓存。`origin` 指向自己的 fork，`upstream` 指向原仓库。新开发 checkout 可以这样创建：

```bash
git clone --branch personal https://github.com/hansarnold/agent-stuff.git
cd agent-stuff
git remote add upstream https://github.com/mitsuhiko/agent-stuff.git
```

在工作区干净且已提交个人修改后，手动合并上游更新：

```bash
git fetch upstream
git switch personal
git merge upstream/main
```

合并时保留 `package.json` 的白名单、`private: true`、通知适配与 `config` 配置模板。验证后推送个人分支，再用上述 `pi update` 更新安装并执行 `/reload`。Pi 的包更新命令会获取你 fork 的提交，不会替你合并原作者的上游改动。

本配置不启用上游自动发布流程，也不创建版本标签。继续保留上游的包名、版本及仓库来源信息，以便追溯；个人入口是本 fork 的 `personal` 分支。
