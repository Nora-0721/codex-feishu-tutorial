# Codex 接入飞书与直写飞书文档实践

一个偏经验总结的教程仓库，记录我如何把本地 Codex 接进飞书 Agent，以及如何让当前本地 Codex 对话真正拥有“读写我的飞书文档”的能力。

这不是一篇只讲概念的文章，而是基于一次真实折腾过程整理出来的可复用工作流。里面会重点写清楚：

- 飞书 Agent 为什么不等于“云端会写代码的另一个机器人”
- 为什么只发文档链接并不能让 Codex 直接拥有你的权限
- 如何让飞书侧入口和本地 Codex 共享项目上下文
- 如何让本地 Codex Desktop 通过官方工具真正写入飞书文档
- 这条路后续怎么继续扩展到多维表格（Base / Bitable）

这份内容主要来自三段连续实操：

1. 先判断“飞书 Agent 接本地 Codex”和“Desktop 直接写飞书”分别该怎么做
2. 再把 bridge、profile、工作区和共享上下文真正搭起来
3. 最后验证本地 Codex 以用户身份读写飞书文档，并把踩坑过程沉淀成文档

## 适合谁看

如果你正好有下面这些需求，这个仓库会比较对口：

- 想在飞书里直接和本地 Codex 对话
- 想让 Agent 读取你电脑上的项目代码，再回写飞书文档
- 不想每次在飞书里重新写一长串提示词
- 想让当前本地 Codex 对话真正具备“帮我写飞书”的能力
- 后续还想把文档同步扩展到飞书多维表格

## 你会看到什么

这个仓库按两部分组织：

```text
.
├── README.md
├── docs
│   ├── 01-codex-feishu-agent-deploy.md
│   └── 02-local-codex-feishu-write-auth.md
└── assets
```

### 第一部分：把本地 Codex 接进飞书

重点回答：

- bridge 到底解决什么问题
- 为什么“共享本地上下文”比“继承某个聊天线程”更可靠
- profile、工作区、`AGENTS.md` 应该怎么组织
- `/cd` 为什么必须单独发，不能和任务写在同一条消息里

入口：

- [docs/01-codex-feishu-agent-deploy.md](./docs/01-codex-feishu-agent-deploy.md)

### 第二部分：让本地 Codex 直接写飞书文档 / 多维表格

重点回答：

- 为什么文档链接不等于授权
- 为什么我最后选择了用户身份授权
- `lark-cli` 在这条链路里扮演什么角色
- 本地 Codex 直写飞书文档是怎么验证成功的
- 这套授权路径如何继续复用到多维表格

入口：

- [docs/02-local-codex-feishu-write-auth.md](./docs/02-local-codex-feishu-write-auth.md)

## 这次实践得出的核心结论

如果只想记住最重要的几句话，可以先看这里：

1. 飞书里的 Agent 最好理解为“你本地 Codex 的聊天入口”，而不是另一个会自动继承全部上下文的新 agent。
2. 长期要复用的项目上下文，不要只放在聊天里，要落到 `AGENTS.md` 和项目文档里。
3. 如果要让当前本地 Codex 直接写飞书，必须把“写飞书”这件事变成一个本地可调用的正式工具链，而不是幻想链接即授权。
4. 我这次实测跑通的是“本地 Codex 以用户身份读写飞书文档”；多维表格能力则可以继续沿着同一套官方 CLI / API 路径扩展。

## 建议你先看哪一篇

- 如果你现在还没把 Codex 接进飞书：先看 [01](./docs/01-codex-feishu-agent-deploy.md)
- 如果你已经能在飞书里对话，但还不能让 Desktop 直接写飞书：先看 [02](./docs/02-local-codex-feishu-write-auth.md)
- 如果你两边都想做：按 `01 -> 02` 顺序看最顺

## 截图预留

我已经把截图位预留在文档结构里了，后续建议把这些图放进 `assets/`：

```text
assets/
├── 01-bridge-terminal-ready.png
├── 02-feishu-agent-online.png
├── 03-feishu-cd-success.png
├── 04-feishu-auth.png
├── 05-lark-cli-whoami.png
├── 06-doc-write-success.png
└── 07-bitable-write-success.png
```

你已经有这些素材会非常加分：

- 终端连接成功截图
- 飞书授权截图
- 对话过程截图

后面只要按文件名放进去，再把文档里对应位置补成图片引用即可。

## 参考资料

本仓库写作时主要参考了这些公开资料：

- [zarazhangrui/lark-coding-agent-bridge](https://github.com/zarazhangrui/lark-coding-agent-bridge)
- [datawhalechina/hello-agents](https://github.com/datawhalechina/hello-agents)
- [larksuite/cli](https://github.com/larksuite/cli)
- [飞书开放平台文档](https://open.feishu.cn/document?lang=zh-CN)

## 一点说明

这份仓库更偏“实战经验总结”，不是飞书开放平台全量能力手册。

所以我会刻意把内容写得更像：

- 哪条路真的跑通了
- 哪些地方容易踩坑
- 哪些结论是我这次亲自验证过的
- 哪些能力虽然官方支持，但还需要你后续按自己项目继续补

如果你也在折腾“本地 agent + 飞书协作”的组合，希望这份记录能帮你少走一点弯路。
