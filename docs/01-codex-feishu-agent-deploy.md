# 01. 把本地 Codex 接进飞书 Agent

这一篇只解决一件事：让飞书里的 Agent 真正驱动你电脑上的 Codex，而不是在飞书里重新养一个“失忆版机器人”。

如果你和我当时的目标一样:

- 想在飞书里直接和 Codex 对话
- 想让它读取本地项目代码再回写结果
- 想保留本地 `AGENTS.md`、Codex 配置和工作区习惯

那么推荐先走 `lark-channel-bridge` 这条路。

## 这条路能做到什么

根据 `lark-channel-bridge` 官方 README，它本质上是把飞书 / Lark 会话桥接到本地 Claude Code 或 Codex CLI，支持会话连续、`/cd` 切换目录、`/ws` 保存工作区、文件上传下载等能力。也就是说，入口在飞书，真正干活的 agent 在你的电脑上。  
参考：

- [zarazhangrui/lark-coding-agent-bridge](https://github.com/zarazhangrui/lark-coding-agent-bridge)

这很重要，因为它决定了:

1. 飞书不是“代码真身”，本地 Codex 才是。
2. 你要保留上下文，关键不是让飞书记住聊天，而是让它始终回到同一个本地项目和同一套本地说明文件。
3. 真正长期有效的记忆，应该沉淀在 `AGENTS.md`、项目文档、工作区配置里。

## 先讲结论

我这次最终跑通的方案是:

1. 在本机安装 `lark-channel-bridge` 和 `lark-cli`
2. 为飞书里的不同智能体建立独立 profile
3. 让 bridge profile 继承本机 Codex Home
4. 把共享规则写进 bridge 默认工作区和真实项目里的 `AGENTS.md`
5. 在飞书里先单独发送 `/cd <项目路径>`，再发送真正任务

其中第 5 点非常关键，这是我这次最容易踩坑、但也最容易复用的经验。

## 推荐目录规划

如果你不想把 bridge、npm cache、state 都塞进 C 盘，可以像我一样单独放到 D 盘或 E 盘。一个比较清晰的目录例子：

```text
D:\Nora-D\lark-coding-agent-bridge
├── npm-prefix
├── npm-cache
├── state
├── logs
└── state-workspaces
```

这样做的好处是:

- bridge 相关状态和 Node 安装分离明确
- 以后迁移、备份、清理更容易
- 不会把默认 npm 缓存和工具状态混进系统目录

## 安装思路

### 1. 环境要求

官方 README 给出的最低条件包括：

- Node.js `>= 20.12.0`
- 本地已安装并可运行的 Codex CLI
- 一个可绑定的 Feishu / Lark PersonalAgent 应用

参考：

- [lark-channel-bridge README](https://github.com/zarazhangrui/lark-coding-agent-bridge)

### 2. 安装 bridge

官方示例是：

```bash
npm i -g lark-channel-bridge
```

或者首次前台调试时使用：

```bash
lark-channel-bridge run
```

但如果你和我一样，希望把安装、缓存、状态都放在非 C 盘，建议先显式设置 npm 位置，再安装。

一个思路示例：

```powershell
$env:npm_config_prefix = 'D:\Nora-D\lark-coding-agent-bridge\npm-prefix'
$env:npm_config_cache  = 'D:\Nora-D\lark-coding-agent-bridge\npm-cache'
```

如果安装时遇到 `127.0.0.1:7897` 之类的代理报错，优先怀疑是旧代理环境变量残留，而不是 npm 本身坏了。我的处理方法是只在该条命令里临时清空 `HTTP_PROXY` / `HTTPS_PROXY` / `ALL_PROXY`，不要直接乱改全局环境。

### 3. 安装 `lark-cli`

`lark-cli` 是后面“本地 Codex 直接写飞书文档 / 多维表格”的关键工具，最好一并装好。官方 README 提供了：

```bash
npx @larksuite/cli@latest install
```

参考：

- [larksuite/cli README.zh.md](https://github.com/larksuite/cli/blob/main/README.zh.md)

## Profile 设计建议

如果你只打算连一个飞书 Agent，用默认 profile 也可以。

但如果你像我一样想分角色使用，建议一开始就把 profile 分开。例如：

- `codex`：偏日常协作、轻任务
- `alhaitham`：偏写文档、读代码、做开发规划

原因很简单：

- 每个 profile 都能有自己的 app 凭证、工作区、日志和会话
- 以后某个 profile 配坏了，不会把另一个一起拖死
- 你可以给不同角色不同的默认工作区说明

官方 README 也明确写到：每个 profile 会维护自己的 app credentials、sessions、working directories 和 logs。  
参考：

- [lark-channel-bridge README](https://github.com/zarazhangrui/lark-coding-agent-bridge)

## 共享上下文怎么做

这里是很多人一开始最容易误解的地方。

### 误区

误区是：希望飞书里的 Agent 直接继承“当前这个 Codex Desktop 聊天线程的全部记忆”。

这通常不稳，也不是我这次最终采用的做法。

### 更稳的做法

让飞书入口和 Desktop 入口都回到同一套本地上下文：

1. bridge profile 保持继承本机 Codex Home
2. 不要让它忽略本地 rules
3. 把长期要求写进项目里的 `AGENTS.md`
4. 需要复用的开发目标、文档同步规则、验收口径，也都写成文件

例如可以在项目根目录维护：

```text
AGENTS.md
docs/dev-plan.md
docs/feishu-sync.md
```

这样无论你是从 Desktop 进入，还是从飞书里 `/cd` 进入，本地 Codex 看到的都还是同一份事实来源。

## `/cd` 的正确使用方式

这是我这次最真实、最值得写进教程的坑。

### 错误写法

不要把 `/cd` 和真正任务写在同一条消息里，例如：

```text
/cd D:\your-project 请读取项目代码并帮我写飞书文档
```

bridge 很可能会把后面的中文也当成路径的一部分，最后报“目录不存在”。

### 正确写法

先发一条纯路径消息：

```text
/cd D:\your-project
```

等它确认目录切换成功后，再发第二条消息：

```text
请读取这个项目的代码结构，并帮我更新飞书开发文档。
```

如果你只记住这一条，这篇文档就已经值回票价了。

## 建议保留的截图

这篇建议你后续补 3 类截图，放到 `assets/` 里：

1. bridge / terminal 启动成功截图
2. 飞书里 Agent 在线、能够响应的截图
3. `/cd` 切目录成功后的对话截图

推荐命名：

```text
assets/
├── 01-bridge-terminal-ready.png
├── 02-feishu-agent-online.png
└── 03-feishu-cd-success.png
```

## 最后的小结

这一部分的核心不是“把飞书变成开发环境”，而是“让飞书成为你本地 Codex 的远程入口”。

只要你记住下面这三条，整个工作流就会稳定很多：

1. 真正的上下文要落在本地文件，不要只留在聊天里。
2. profile、工作区、规则最好从一开始就分清楚。
3. `/cd` 一定单独发，切完目录再下任务。
