> 中文 | [English](README.en.md)

# dsh-approval-qq

**把 DeepSeek Harness 的审批弹窗搬到 QQ。** 模型要提权执行命令时，你的手机会收到一条私聊消息：
看清命令和理由，回一句「同意」就放行；电脑端的弹窗同时存在，谁先点谁生效。

- 独立插件：**不依赖**任何 QQ 机器人插件，直接对接 OneBot v11（NapCat / Lagrange / LLOneBot…）；
- 零运行时依赖：只用 Node 内置 `fetch` / `WebSocket`（Node ≥ 22）；
- 只认你配置的管理员 QQ，其他人回复一律忽略；
- QQ 不可用时自动退回电脑端弹窗，**绝不挂住请求**。

---

## 它长什么样

```
【审批 #4c5f89】需要你点头才能继续

我要做什么：执行一条 PowerShell 命令
在哪个目录：D:\work\demo
为什么要你同意：要临时把沙箱提到「完全访问（可以修改系统任何位置）」，DSH 规定提权必须本人确认。
理由：需要读取工作区外的配置文件

命令（原样）：
Get-Content "C:\ProgramData\app\config.json"

时间：2026-10-26 19:38:04 周一
——————
回复「同意」或「拒绝」即可（多条时默认处理最新一条；要指定也可加编号）
```

回复规则：

| 情况 | 回复 |
|---|---|
| 只有一条待审批 | 「**同意**」或「**拒绝**」 |
| 同时挂着多条 | 「同意」默认处理**最新一条**，回执会说明还剩哪几条 |
| 想指定某条 | 「同意 ab12cd」（编号 = 标题里的 `#ab12cd`） |

## 安装

### 从 npm（若已发布）

```bash
dsh plugin add dsh-approval-qq
```

### 从 GitHub

```bash
dsh plugin add github:<你的用户名>/dsh-approval-qq
```

### 本地包

```bash
dsh plugin add file:C:\path\to\dsh-approval-qq
```

装完 **重启一次 DSH**，然后到 **设置 → 插件 → dsh-approval-qq** 填配置。

## 配置

| 字段 | 默认 | 说明 |
|---|---|---|
| `adminQq` | `0` | **必填**。审批人 QQ 号。不填则插件不生效（审批只在电脑端）。 |
| `onebotHttpUrl` | `http://127.0.0.1:3000` | OneBot v11 HTTP API 地址，用于发消息 / 撤回。 |
| `onebotHttpToken` | 空 | HTTP API 的 `access_token`。 |
| `onebotWsUrl` | `ws://127.0.0.1:3001` | 正向 WebSocket 地址，用于接收你的回复。 |
| `onebotWsToken` | 空 | WebSocket 的 `access_token`（会拼成 `?access_token=`）。 |
| `idLength` | `6` | 审批编号长度（4~8）。 |
| `showFullCommand` | `true` | `false` 时命令只贴前 800 字。 |
| `notifyUnauthorized` | `true` | 别人回「同意」时是否私聊回一句「无权限」。 |
| `logFile` | 空 | 留空 = `~/.dsh/approval-qq/approval.log`。 |

> NapCat 的 HTTP 与 WS 通常不在同一个端口：HTTP 默认 `3000`，反向/正向 WS 默认 `3001`，
> 以你自己的 NapCat 配置为准。WS 没连上时插件会跳过 QQ、直接走电脑端——不会出现
> 「消息发出去了但没人能回复」的悬挂。

## 它是怎么工作的

```
DSH 触发 approval/request（policy=ask）
   ├─ 本插件：发 QQ 私聊（短 id + 人话理由 + 命令原文）
   └─ next()：交给组合里的下一个 answerer（通常是电脑端弹窗）
   
两者 Promise.race：
   QQ 先决定 → 返回真实 outcome，并撤回 QQ 消息
   电脑先决定 → 返回桌面结果，同时撤回 QQ 消息（手机上立即消失）
```

- **命令原文**从哪来：DSH 的审批请求只带 justification，所以插件挂了
  `tools/pre-execute`，按 `callId` 缓存工具原始参数，审批时取出 `args.command`；
- **理由**：`displayReason` 可能是 `{zh,en}` 对象，插件会统一压成一句人话；
- **危险操作**（删除 / 格式化 / 写系统目录 / 启进程 / 强推）会多一行 `⚠️` 警示；
- **日志**：每次 asked / reply / decided / qq-unavailable 都写一行 JSON，
  含 id、工具、命令、来源（qq / desktop）、结果。

## 安全说明

- 插件**不含**任何账号、令牌或个人路径的默认值；`adminQq` 默认 0，必须自己填；
- 只有 `adminQq` 的私聊回复会被接受，其他人回复不会产生任何授权；
- 审批消息经你配置的 OneBot 服务转发，走本机回环地址，默认不出网（NapCat 连哪台服务器由你决定）；
- 日志默认写在 `~/.dsh/approval-qq/approval.log`，里面会有命令原文，注意别把它提交到仓库。

## 已知边界

- 电脑端弹窗**不由本插件主动关闭**：QQ 先同意时请求先 resolve，桌面 UI 依赖 DSH 自身收尾；
- 不支持多人会签，只认一个管理员；
- 部分工具（如只带路径的文件操作）没有 `command` 字段，这时消息里只展示理由和参数 JSON；
- 如果你同时装了别的审批转发插件（例如 dsh-onebot-qq 里的 approval 子插件），
  会出现两条消息/两个 answerer，请只保留一个。

## 开发

```bash
# 语法自检（无构建步骤）
node --check index.js
```

## License

[MIT](LICENSE)
