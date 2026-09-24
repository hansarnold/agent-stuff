# 我的 Pi 基础配置

基于 [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff)，保留上游源码与 Apache-2.0 许可证。个人配置维护在默认分支 `personal`，`main` 保留作上游基线。已在 Linux、Pi 0.87.1 上检查资源加载。

## 默认启用

个人包白名单在根目录 `package.json` 的 `pi` 字段中。修改、提交并推送后，用 `pi update` 更新安装，再在 Pi 内执行 `/reload`。

| 功能 | 使用方式 |
| --- | --- |
| 完成通知 | 本地 Linux 桌面优先 `notify-send`；失败或通过 SSH 使用时回退到上游 OSC 777 终端通知 |
| TODO | `/todos`；模型也可使用 `todo` 工具；任务保存在项目 `.pi/todos` |
| 交互答题 | `/answer` 或 `Ctrl+.`，把上一条助手回复中的问题提取为交互表单 |
| 代码审查 | `/review`、`/end-review`；PR 模式需要已登录的 `gh` |
| 用量统计 | `/session-breakdown`，查看最近 7/30/90 天的会话、token 和费用记录 |
| 讨论方案 | `/discuss`，在实现前逐轮厘清需求 |
| 提交规范 | `commit` skill，可通过 `/skill:commit` 调用 |
| GitHub 操作 | `github` skill，可通过 `/skill:github` 调用；使用已登录的 `gh` 操作 Issue、PR 和 CI |
| 网页搜索、正文读取 | 单独安装固定版本 `pi-web-access@0.31.0`；模型按需调用 `web_enable` 后使用 `web_search`、`fetch_content` 和 `get_search_content`，`/search` 浏览本会话搜索记录 |

`/answer` 会额外调用模型提取问题。保留上游的模型选择逻辑：优先可用的快速 Codex 模型，然后 Haiku，最后当前模型。它处理已经提出的问题，并非主动 `ask_user` 工具。

通知在 `agent_settled` 触发，即自动重试、压缩及排队续跑结束后。非交互调用不发送通知。本地 Linux 需要 `notify-send` 与可用的桌面通知服务；终端回退需要 OSC 777 支持。`notify-send` 不成功时回退并不保证桌面一定能收到通知。

其余扩展、skills 和主题保留源码但不加载，包括自动信任作者仓库、Bash/edit 替换、多代理、长期自动运行和 macOS 专用功能。以后添加功能时，把对应路径加入白名单即可。

## 安装和移除

已安装 Pi 后，直接从 GitHub 安装个人分支，再安装网页插件：

```bash
pi install git:github.com/hansarnold/agent-stuff@personal
pi install npm:pi-web-access@0.31.0
```

这两个声明写入 `~/.pi/agent/settings.json` 的 `packages`，示例见 [config/settings.example.json](config/settings.example.json)。保留你自己的模型和其他设置，不要用示例覆盖整个设置文件。Pi 将 GitHub 包下载到它管理的 Git 缓存，将 npm 包安装到它管理的 npm 目录；运行不依赖开发 checkout，也无需另外执行 `npm install`。

首次配置网页插件时，把 [config/web-search.json](config/web-search.json) 保存到 `~/.pi/agent/web-search.json`；如果该文件已经存在，合并所需字段。这份配置默认使用 Exa，未配置 Exa key 时走免 key 的 MCP 搜索；读取网页使用直接 HTTP 和本地正文提取。保留完整内容检索工具，默认不生成额外摘要；关闭独立来源核验、自动克隆 GitHub、图片、PDF 和视频处理。GitHub 任务交给 `github` skill + `gh`。

网页配置是独立用户设置，Pi 安装包时不会自动复制它。本机已经应用该配置。以后修改仓库中的模板，需要同步到用户设置后重新加载。只安装个人 GitHub 包不会自动安装单独的网页插件。

`private: true` 防止把个人 fork 意外发布为上游 npm 包。上游的 `diff` 依赖仍保留，以便后续启用其他扩展。

已有会话执行 `/reload`，或重启 Pi。移除时执行：

```bash
pi remove git:github.com/hansarnold/agent-stuff@personal
pi remove npm:pi-web-access@0.31.0
```

Pi 的模型、登录凭据及会话继续放在用户自己的 `~/.pi/agent`，不纳入这个仓库。

## 同步上游

日常获取自己 fork 的更新：

```bash
pi update git:github.com/hansarnold/agent-stuff@personal
```

这里的 `personal` 是分支引用，更新会获取该分支最新提交。网页插件固定在 `0.31.0`，升级时要明确安装新版本。

开发 checkout 是 `/home/jack/proj/agent-stuff`，仅用于修改、提交和合并上游；不要直接修改 Pi 管理的缓存。它已设置 `origin` 为自己的 fork，`upstream` 为原仓库。新开发 checkout 可以这样创建：

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
