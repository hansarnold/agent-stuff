# 我的 Pi 基础配置

基于 [mitsuhiko/agent-stuff](https://github.com/mitsuhiko/agent-stuff)，保留上游源码与 Apache-2.0 许可证。个人配置维护在默认分支 `personal`，`main` 保留作上游基线。已在 Linux、Pi 0.87.1 上检查资源加载。

## 默认启用

白名单在根目录 `package.json` 的 `pi` 字段中，修改后在 Pi 内执行 `/reload`。

| 功能 | 使用方式 |
| --- | --- |
| 完成通知 | 本地 Linux 桌面优先 `notify-send`；失败或通过 SSH 使用时回退到上游 OSC 777 终端通知 |
| TODO | `/todos`；模型也可使用 `todo` 工具；任务保存在项目 `.pi/todos` |
| 交互答题 | `/answer` 或 `Ctrl+.`，把上一条助手回复中的问题提取为交互表单 |
| 代码审查 | `/review`、`/end-review`；PR 模式需要已登录的 `gh` |
| 用量统计 | `/session-breakdown`，查看最近 7/30/90 天的会话、token 和费用记录 |
| 讨论方案 | `/discuss`，在实现前逐轮厘清需求 |
| 提交规范 | `commit` skill，可通过 `/skill:commit` 调用 |

`/answer` 会额外调用模型提取问题。保留上游的模型选择逻辑：优先可用的快速 Codex 模型，然后 Haiku，最后当前模型。它处理已经提出的问题，并非主动 `ask_user` 工具。

通知在 `agent_settled` 触发，即自动重试、压缩及排队续跑结束后。非交互调用不发送通知。本地 Linux 需要 `notify-send` 与可用的桌面通知服务；终端回退需要 OSC 777 支持。`notify-send` 不成功时回退并不保证桌面一定能收到通知。

其余扩展、skills 和主题保留源码但不加载，包括自动信任作者仓库、Bash/edit 替换、多代理、长期自动运行和 macOS 专用功能。以后添加功能时，把对应路径加入白名单即可。

## 安装和移除

已安装 Pi 后，克隆本仓库的个人分支并加载本地目录：

```bash
git clone --branch personal https://github.com/hansarnold/agent-stuff.git
cd agent-stuff
pi install "$PWD"
```

当前五个扩展仅导入 Pi 提供的模块和 Node 内置模块，不需要额外执行 `npm install`。上游保留的 `diff` 依赖用于其他扩展；将来启用它们时再安装相应依赖。`private: true` 防止把个人 fork 意外发布为上游 npm 包。

已有会话执行 `/reload`，或重启 Pi。移除时，在仓库目录执行：

```bash
pi remove "$PWD"
```

Pi 的模型、登录凭据及会话继续放在用户自己的 `~/.pi/agent`，不纳入这个仓库。

## 同步上游

本机已设置 `origin` 为自己的 fork，`upstream` 为原仓库。新克隆可以先运行：

```bash
git remote add upstream https://github.com/mitsuhiko/agent-stuff.git
```

在工作区干净且已提交个人修改后，手动合并上游更新：

```bash
git fetch upstream
git switch personal
git merge upstream/main
```

合并时保留 `package.json` 的白名单、`private: true` 和通知适配，验证加载后再推送个人分支并执行 `/reload`。本地路径安装直接读取此 checkout；Pi 的包更新命令不会替你合并上游。

本配置不启用上游自动发布流程，也不创建版本标签。继续保留上游的包名、版本及仓库来源信息，以便追溯；个人入口是本 fork 的 `personal` 分支。
