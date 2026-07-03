# 02. 权限配置：让本地 Codex 直接写飞书文档

这一篇只讲一件事：  
怎么让当前这个本地 Codex 会话直接读写飞书文档。

这里说的不是飞书里的 Agent，也不是小洛 / 艾尔海森那条接入链。  
这里说的是 **当前这个 Codex Desktop 会话**，怎么通过 `lark-cli` 拿到用户身份，然后直接操作飞书文档。

按之前那三段对话里的实际验证结果，这一篇当前能确认的范围是：

- `docs`
- `drive`
- `wiki`

也就是飞书文档 / 知识库文档这一条线。  
没有验证多维表格读写，所以这一篇不展开写。

## 一、先装好 `lark-cli`

如果前面已经装过，这一步可以跳过。

```bash
npx @larksuite/cli@latest install
```

## 二、先发起用户身份授权

这一步是在本地终端里做，不是在飞书聊天框里做。

先执行：

```bash
lark-cli auth login --domain docs --domain drive --domain wiki --no-wait --json
```

这条命令会返回两样关键东西：

1. `verification_url`
2. `device_code`

后面会用到：

- `verification_url`：给手机打开授权页
- `device_code`：手机授权完成后，回终端收尾

## 三、把授权链接转成二维码，再用手机扫

拿到 `verification_url` 以后，可以继续生成二维码给手机扫。

我当时的流程就是：

1. 终端先执行 `lark-cli auth login --domain docs --domain drive --domain wiki --no-wait --json`
2. 拿到 `verification_url`
3. 再把这个授权链接转成二维码发出来给手机扫码

如果你在 Windows 下直接走 `.cmd` 包装层，URL 里有 `&` 时很容易被拆坏。  
更稳的做法是直接调用真正的 `lark-cli.exe`，不要让 `.cmd` 去解析带 `&` 的 URL。

## 四、手机上完成授权

手机扫完二维码后，打开的是飞书授权页。

这一页确认通过以后，继续点授权即可。

这里要注意一点：  
这一步是 **给当前本地 `lark-cli` 授权成用户身份**，不是给飞书 Agent 建应用，也不是在做 bridge 绑定。

## 五、手机授权完以后，回终端收尾

手机上点完授权以后，还没结束。

终端里还要再执行一次：

```bash
lark-cli auth login --device-code <device_code>
```

这里的 `<device_code>` 就是第二步那条命令返回的那个值。

执行完这一步，本地这套用户授权才算真正写进 `lark-cli` 配置。

## 六、先检查当前身份是不是对的

授权完成后，不要急着直接写文档，先检查状态。

```bash
lark-cli whoami
lark-cli auth status --json --verify
```

主要看三样：

1. `identity` 是不是 `user`
2. token 是否有效
3. `docs / drive / wiki` 这几个域是不是正常

如果这里不是 `user`，后面就不要继续做文档写入测试。

## 七、先读文档

先拿一个已经存在的飞书文档或 wiki 链接做读取测试：

```bash
lark-cli docs +fetch --doc "<你的飞书文档链接>" --as user
```

这一步确认的是：

1. 当前身份能不能读这个文档
2. 当前目标文档有没有选错

## 八、再做写入测试

不要一上来就全量覆盖正式文档。

更稳的顺序是：

1. 先 dry-run
2. 再追加一小段测试内容
3. 再读回去确认
4. 最后把测试段落删掉

这样不会把正式文档改乱，也方便排查到底是：

- 写入参数不对
- 当前身份不对
- 还是目标文档权限不够

## 九、当前这篇能确认到哪一步

按之前那几段实际对话，已经验证过的是：

1. 本地 Codex Desktop 可以通过 `lark-cli` 以用户身份登录
2. 可以读取指定飞书 wiki / 文档
3. 可以做一次真实写入
4. 可以再把测试内容删掉

所以这一篇现在写的是“本地 Codex 直写飞书文档权限”。
