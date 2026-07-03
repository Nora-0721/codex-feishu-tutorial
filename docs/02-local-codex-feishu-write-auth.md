# 02. 让本地 Codex 直接写飞书文档与多维表格

这一篇讲的是另一个层面的问题：

飞书里的 Agent 能调用本地 Codex，不等于当前这个本地 Codex Desktop 对话已经天然拥有“代你写飞书”的权限。

如果你想做到的是：

- 在当前本地 Codex 对话里直接读写你的飞书文档
- 根据本地项目代码生成内容后，马上回写到飞书
- 后续进一步扩展到多维表格（Base / Bitable）写入

那你需要补上的是 `lark-cli` 这条“权限与执行通道”。

## 先讲清楚权限边界

### 不能靠链接即授权

只把一个飞书 wiki / doc 链接发给 Codex，并不会自动让它继承你的飞书登录态。

也就是说：

- 文档链接不是你的账号权限
- 网页登录态不能直接当成安全、稳定的 agent 接口
- 想让本地 Codex 真正读写飞书，还是要走官方应用授权 / CLI / API

### 我这次实际选择的是用户身份

在我这次的跑通方案里，最终选择的是 `user-default` 这类用户身份路径，而不是只用 bot 身份。

因为目标不是“让 bot 改它自己有权限的文档”，而是：

- 让当前 Desktop 对话可以按我的飞书用户身份访问 wiki / docs
- 后续继续写我自己的开发文档

这条路权限更强，所以也更值得把验证步骤写清楚。

## 为什么是 `lark-cli`

`lark-cli` 是飞书官方 CLI。它的官方 README 明确覆盖了多个业务域，包括：

- Docs
- Drive
- Wiki
- Sheets
- Base

其中 Base 对应的就是现在官方文案里的多维表格能力。  
参考：

- [larksuite/cli README](https://github.com/larksuite/cli)
- [飞书多维表格 API 概览](https://open.feishu.cn/document/server-docs/docs/bitable-v1/bitable-overview?lang=zh-CN)

所以从“能力规划”上看，文档和多维表格并不是两条完全割裂的路，而是可以复用同一套应用授权体系和同一套本地工具链。

## 我这次实际跑通了什么

已验证成功的部分：

1. 本地 Codex Desktop 通过 `lark-cli` 以用户身份完成授权
2. 能读取指定飞书 wiki 文档
3. 能执行真实写入
4. 能验证修订版本递增
5. 能删除测试块，避免污染正式文档

也就是说，**“本地 Codex 直写飞书文档”这部分是实测成功的**。

多维表格方面：

- 官方 CLI 和官方 API 都支持 Base / 多维表格
- 这说明同样的本地权限通道可以继续扩展到表结构、字段、记录写入
- 但我这次真实对话里重点验证的是文档写入，不是完整的 Base 自动化流程

所以更准确的说法应该是：

**文档写入已实测；多维表格写入在官方能力上可行，推荐沿用同一套 CLI / API 授权路径继续扩展。**

## 关键授权流程

### 1. 让 Desktop 复用 bridge 已有配置

我这次不是重新建一套飞书配置，而是让本地 Desktop 复用已经跑通的 bridge 配置。

一个很关键的经验是：

- `lark-channel-bridge` 的状态目录可以放在 D 盘
- 但 `lark-cli` 在 Windows 上往往还是会默认去找 `C:\Users\<用户名>\.lark-cli`

因此如果你已经用 bridge 跑通过飞书 Agent，通常要解决的是“如何让 Desktop 侧 CLI 看到同一份配置”，而不是从零再绑一次。

我这次最终可用的方法，是把默认路径通过联接 / junction 暴露给 CLI 使用。

## 2. 登录授权

我这次实测成功的授权命令思路是：

```bash
lark-cli auth login --domain docs --domain drive --domain wiki --no-wait --json
```

它会返回一组需要用户配合完成的授权信息，例如：

- `verification_url`
- `device_code`

接下来流程是：

1. 把授权链接或二维码发给用户
2. 用户在飞书里确认授权
3. 本地 agent 再用 `device_code` 完成登录闭环

这一步很适合写进公开教程，因为它非常符合“本地 agent + 用户手动授权”的真实使用场景。

### 3. 验证身份是否真的生效

完成授权后，不要急着直接写文档，先验身份。

建议至少做两步：

```bash
lark-cli whoami
lark-cli auth status --json --verify
```

如果这一关没有过，就不要继续往下做“写文档”测试。

## 文档写入验证流程

我推荐把验证拆成四步：

### 第 1 步：先读

```bash
lark-cli docs +fetch --doc "<你的 wiki/doc 链接>" --as user
```

确认：

- 链接能解析
- 文档标题能读出来
- 当前身份确实有访问权

### 第 2 步：先 dry-run

在正式写入前，先做一次预演，避免改错文档。

### 第 3 步：追加一小段测试内容

例如追加一个很短的测试段落，确认：

- 调用参数正确
- 更新 API 没有报权限问题
- 文档内容真的发生变化

### 第 4 步：验证后清理测试块

如果只是做链路验证，最好把测试内容删掉，让正式文档保持干净。

这一步很能体现教程质量，因为它说明你不只是“调通一次”，而是考虑了真实文档的整洁性。

## 我这次踩过的 4 个坑

### 1. 自定义 bridge home 不一定会被 CLI 直接识别

一开始我尝试直接把 CLI 指向 bridge 的自定义状态目录，但并没有稳定成功。

最后真正靠谱的做法，是让默认路径能看到 bridge 状态，而不是死磕“自定义 home 一定要直接被识别”。

### 2. 旧代理会导致授权和安装假失败

如果你遇到：

```text
127.0.0.1:7897
ECONNREFUSED
proxyconnect tcp
```

优先查代理环境变量。

经验是：只在当前命令临时清空代理变量，不要直接粗暴修改系统全局设置。

### 3. Windows 下 `&` 可能把授权链接拆坏

我这次生成授权二维码时，URL 里带 `&`，结果 `.cmd` 包装层把它拆成了多段命令。

更稳的做法是：

- 直接调用真实的 `lark-cli.exe`
- 或者避免让 `.cmd` / `cmd.exe` 参与 URL 解析

### 4. 不要把“能读文档”误判成“肯定能写”

最稳的链路一定是：

1. `whoami`
2. `auth status`
3. `docs +fetch`
4. `docs +update`
5. 再读回去验证

不要在第 3 步就宣布大功告成。

## 多维表格怎么接上

这一部分我建议你在公开仓库里诚实一点写：

### 已经可以复用的东西

- 同一套飞书应用凭证
- 同一套 `lark-cli` 登录态
- 同一台本地 Codex
- 同一套“用户授权后，本地 agent 执行”的安全模型

### 后续要补的东西

- 具体目标 Base 的 `app_token`
- table / field / record 的结构设计
- 对应的 Base API 或 CLI 命令封装

官方 README 已经明确把 Base 归为一等能力域，官方开放平台也提供了创建数据表、列字段、查询记录等 API。  
参考：

- [larksuite/cli README](https://github.com/larksuite/cli)
- [新增一个数据表](https://open.feishu.cn/document/server-docs/docs/bitable-v1/app-table/create?lang=zh-CN)
- [查询多维表格记录](https://open.feishu.cn/document/docs/bitable-v1/app-table-record/search?lang=zh-CN)

因此如果你后续要扩展成“本地 Codex 帮我维护飞书多维表格”，最自然的路线就是：

1. 先复用这篇里的 `lark-cli` 授权
2. 再针对 Base 写一层命令模板或脚本
3. 最后把它收敛成项目里的固定工作流

## 建议保留的截图

这一篇建议你补 4 类截图：

1. 飞书授权页 / 二维码截图
2. `whoami` 或 `auth status` 成功截图
3. 文档写入成功后的修订或内容对比截图
4. 如果你之后补了 Base 流程，再放一张多维表格写入效果截图

推荐命名：

```text
assets/
├── 04-feishu-auth.png
├── 05-lark-cli-whoami.png
├── 06-doc-write-success.png
└── 07-bitable-write-success.png
```

## 最后的小结

这一部分最核心的价值，不是“背了几条命令”，而是明确了一个稳定边界：

- 飞书里的 Agent 负责提供入口
- 本地 Codex 负责读取代码、生成内容
- `lark-cli` / 官方 API 负责把权限和写入动作做实

这样你的技术分享就不只是“我好像连上了”，而是能讲清楚：

1. 为什么单靠链接不行
2. 为什么要单独做用户授权
3. 为什么本地项目和飞书文档可以真正串起来
4. 为什么这套路径还能继续扩展到多维表格
