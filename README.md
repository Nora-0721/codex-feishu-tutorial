# Codex 接入飞书并直写飞书文档实践

一共有两个教程，第一种是创建飞书Agent，第二种是让你本地的Agent有权利阅读和修改飞书文档：

- `docs/01-codex-feishu-agent-deploy.md`
  讲的是怎么把本地 Codex 接进飞书，让它能在飞书里对话、切项目目录、继续处理本地代码。
- `docs/02-local-codex-feishu-write-auth.md`
  讲的是怎么把“会聊天”进一步做到“能直接写我的飞书文档”。

## 目录

```text
.
├── README.md
├── docs
│   ├── 01-codex-feishu-agent-deploy.md
│   └── 02-local-codex-feishu-write-auth.md
└── assets
```

## 文档入口

### 1. 飞书 Agent 接入

- [01-codex-feishu-agent-deploy.md](./docs/01-codex-feishu-agent-deploy.md)

### 2. 本地 Codex 直写飞书

- [02-local-codex-feishu-write-auth.md](./docs/02-local-codex-feishu-write-auth.md)

## 配图

仓库里已经放了这 4 张图：

```text
assets/
├── 01-feishu-status.jpg
├── 02-bridge-running.jpg
├── 03-xiaohai-profile.jpg
└── 04-feishu-auth.jpg
```

其中：

- `01-feishu-status.jpg`：飞书里查看当前 bridge / session / cwd 状态
- `02-bridge-running.jpg`：本地 bridge 启动后的终端状态
- `03-xiaohai-profile.jpg`：飞书智能体页面
- `04-feishu-auth.jpg`：飞书开放平台里给 Agent 开权限时的授权页

## 参考

- [zarazhangrui/lark-coding-agent-bridge](https://github.com/zarazhangrui/lark-coding-agent-bridge)
- [larksuite/cli](https://github.com/larksuite/cli)
- [Claude Code/Codex 接入飞书教程：Lark Coding Agent Bridge 实践](https://www.feishu.cn/content/article/7647408304896953549)
