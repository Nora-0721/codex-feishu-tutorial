# 我是怎么把 Codex 接进飞书，并且让它真的能写我飞书文档的

这是一份教程，不是概念介绍，也不是空对空聊 Agent。

这份仓库只回答两件事：

1. 怎么把本地 Codex 接到飞书里，让它在飞书里像个能用的智能体。
2. 怎么让本地 Codex 真正拿到权限，直接写我的飞书文档，后面再继续扩到多维表格。

我之前最烦的一点，就是很多教程写得好像已经讲明白了，结果真到自己电脑上，不是权限不对，就是路径不对，要么就是明明连上了，但其实什么都写不了。

所以这份东西我尽量写得直接一点：

- 能做什么，就直说。
- 不能靠什么做到，也直说。
- 哪一步是关键，哪一步最容易翻车，我也会直接标出来。

## 先看效果

飞书里能直接看到本地 Codex 的状态，不是只有一个“看起来像连上了”的壳。

![飞书里的状态页](./assets/01-feishu-status.jpg)

本地 bridge 真正跑起来以后，终端里也会明确显示 profile、bot、agent 和监听状态。

![本地 bridge 运行中的终端](./assets/02-bridge-running.jpg)

## 这套东西到底在干嘛

简单说，分成两层：

### 第一层：飞书只是入口

飞书里的 Agent 不等于“飞书自己会写代码”。

更准确一点说，它只是一个聊天入口。真正干活的还是你本机上的 Codex。  
所以它能不能读你的项目、能不能继承你的本地规则、能不能继续上一次的工作，本质上都取决于你本地这套东西有没有接好。

### 第二层：能对话，不等于能写你的飞书

这个也很关键。

很多人会默认觉得：既然都在飞书里聊天了，那它应该顺手就能改飞书文档吧。

实际上不是。

“能在飞书里对话”和“能用你的身份去写飞书文档 / 多维表格”是两回事。  
前者解决的是入口问题，后者解决的是权限问题。

这也是为什么我把仓库拆成了两篇：

- [01-codex-feishu-agent-deploy.md](./docs/01-codex-feishu-agent-deploy.md)
- [02-local-codex-feishu-write-auth.md](./docs/02-local-codex-feishu-write-auth.md)

## 仓库结构

```text
.
├── README.md
├── docs
│   ├── 01-codex-feishu-agent-deploy.md
│   └── 02-local-codex-feishu-write-auth.md
└── assets
```

## 第一篇讲什么

第一篇主要讲通信层，也就是：

- 本地 Codex 怎么进飞书
- bridge 到底扮演什么角色
- profile 和工作区怎么分
- 为什么共享上下文这件事，核心不是“记住聊天”，而是“记住本地文件”
- 为什么 `/cd` 一定要单独发

一句话总结第一篇：

**飞书是入口，本地 Codex 才是本体。**

入口在这里：

- [01-codex-feishu-agent-deploy.md](./docs/01-codex-feishu-agent-deploy.md)

## 第二篇讲什么

第二篇主要讲权限层，也就是：

- 为什么飞书链接不等于授权
- 为什么最后必须走 `lark-cli` 这条路
- 为什么要用用户身份，而不是只靠 bot
- 文档写入怎么验证才算真的通了
- 多维表格为什么可以沿用同一套授权思路继续做

一句话总结第二篇：

**能聊天不代表能写文档，能写文档也不代表权限就配对了。**

入口在这里：

- [02-local-codex-feishu-write-auth.md](./docs/02-local-codex-feishu-write-auth.md)

## 这四张图分别放在哪里

我已经把图放进来了，不再只是预留位置。

```text
assets/
├── 01-feishu-status.jpg
├── 02-bridge-running.jpg
├── 03-xiaohai-profile.jpg
└── 04-feishu-auth.jpg
```

其中这两张很适合放在“权限”和“应用配置”相关的段落里：

小洛这个页面能很直观说明：飞书侧的智能体只是一个应用入口，开发者、权限这些东西都得单独配。

![飞书里的智能体应用页](./assets/03-xiaohai-profile.jpg)

下面这张则更直接，能一眼看出“开通并授权”到底授权了哪些能力，尤其是文档、多维表格、云空间这类范围。

![飞书开放平台授权页](./assets/04-feishu-auth.jpg)

## 如果你只想先记住 4 句话

1. 飞书里的 Agent 本质上是你本地 Codex 的一个入口。
2. 想让它不“失忆”，重点不是保聊天记录，而是把规则和目标写进本地文件。
3. 想让本地 Codex 直接写飞书，必须把权限链路接完整，不能靠飞书链接碰运气。
4. 文档写通之后，多维表格不是另一套世界，还是这套授权体系往下延伸。

## 参考资料

- [zarazhangrui/lark-coding-agent-bridge](https://github.com/zarazhangrui/lark-coding-agent-bridge)
- [datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents)
- [larksuite/cli](https://github.com/larksuite/cli)
- [飞书开放平台文档](https://open.feishu.cn/document?lang=zh-CN)
