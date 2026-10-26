# dsh-approval-qq 0.1.0 —— 安装说明（先看这个）

把 DeepSeek Harness 的审批弹窗搬到 QQ：模型要提权执行命令时，手机上收到私聊，
看清命令和理由，回一句「同意」就放行；电脑端弹窗同时存在，谁先决定谁生效。

---

## 一、30 秒装好

前提：已装 DeepSeek Harness（DSH ≥ 0.2.1-rc.2），Node ≥ 22，有一个 OneBot v11 服务
（NapCat / Lagrange / LLOneBot 等）。

在本目录执行（路径按实际改）：

```bash
dsh plugin add file:.\dsh-approval-qq-0.1.0.tgz
```

然后**重启一次 DSH**。

> 也可以直接把压缩包里的 `dsh-approval-qq-0.1.0.tgz` 拖到任意位置，
> 用绝对路径：`dsh plugin add file:C:\somewhere\dsh-approval-qq-0.1.0.tgz`
>
> 注意：**不能直接安装 .zip**。pnpm 的 `file:` 只认目录或 .tgz/.tar.gz。

## 二、必须填的配置

DSH → 设置 → 插件 → `dsh-approval-qq`：

| 字段 | 填什么 |
|---|---|
| `adminQq` | **你的 QQ 号**（只有这个号能审批，不填插件不生效） |
| `onebotHttpUrl` | OneBot 的 HTTP API，例如 `http://127.0.0.1:3000` |
| `onebotHttpToken` | HTTP 的 access_token（没有就留空） |
| `onebotWsUrl` | OneBot 的**正向** WebSocket，例如 `ws://127.0.0.1:3001` |
| `onebotWsToken` | WS 的 access_token（没有就留空） |

其余可选：`idLength`（编号长度，默认 6）、`showFullCommand`（默认 true）、
`notifyUnauthorized`（别人回同意时是否回「无权限」）、`logFile`（默认
`~/.dsh/approval-qq/approval.log`）。

## 三、验证

让模型执行一条需要提权的命令（或在工具参数里加 `sandbox_permissions`），
QQ 应该收到：

```
【审批 #4c5f89】需要你点头才能继续

我要做什么：执行一条 PowerShell 命令
在哪个目录：D:\work\demo
为什么要你同意：要临时把沙箱提到「完全访问…」，DSH 规定提权必须本人确认。
理由：需要读取工作区外的配置文件

命令（原样）：
Get-Content "C:\ProgramData\app\config.json"

⚠️ 危险操作：…（命中危险关键词时才有）

时间：2026-10-26 19:38:04 周一
——————
回复「同意」或「拒绝」即可（多条时默认处理最新一条；要指定也可加编号）
```

回复「同意」→ 命令执行；「拒绝」→ 工具被拒，模型会解释。

## 四、收不到消息 / 不生效怎么办

1. **确认 DSH 重启过**（插件在启动时加载）；
2. **确认 `adminQq` 填了**（留 0 时插件只在日志里打警告，不工作）；
3. **确认 OneBot WS 连上了**：插件日志里应有
   `[approval-qq] OneBot WebSocket 已连接`；WS 没连上时插件会**主动跳过 QQ**
   （避免发出去了却收不到回复），此时审批只在电脑端出现；
4. **HTTP 和 WS 不是同一个端口**：NapCat 默认 HTTP 3000、WS 3001，以你的配置为准；
5. 看日志：`~/.dsh/approval-qq/approval.log`，每次 asked / reply / decided 都有一行 JSON，
   带 `source`（qq / desktop）和 `cmdSource`（args / none）。

## 五、注意事项

- **不要和别的审批转发插件同时启用**（例如某些 QQ 机器人插件自带的 approval），
  否则一条审批会收到两份消息、两个 answerer 互相抢；
- 日志里**包含命令原文**，别把 `approval.log` 提交到公开仓库；
- OneBot 的端口**不要暴露到公网**，token 用强随机串；
- 只支持一个管理员（`adminQq`），不支持多人会签；
- 电脑端弹窗的关闭依赖 DSH 自身收尾，插件只保证 QQ 侧撤回。

## 六、卸载

```bash
dsh plugin remove dsh-approval-qq
```

再重启 DSH。日志文件不会自动删除，可手动清理。

---

校验值（SHA256）：见同目录 `SHA256SUMS.txt`。
许可证：MIT（见 `LICENSE`）。
