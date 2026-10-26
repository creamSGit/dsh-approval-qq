# dsh-approval-qq
DeepSeek Harness 插件：把审批弹窗转发到 QQ，通过 OneBot v11 在手机上回复「同意/拒绝」。


这个插件是什么、怎么用


功能说明

审批上手机	DSH 触发 ask 审批（模型要提权执行命令）时，私聊推送到你的 QQ

人话理由	把 escalate sandbox to danger-full-access: ... 翻译成「要临时把沙箱提到完全访问，DSH 规定提权必须本人确认」

命令原文	通过 tools/pre-execute 抓工具原始参数，把要执行的命令原样贴出来，不截断

危险警示	命中删除/格式化/写系统目录/启进程/强推等关键词，加一行 ⚠️ 危险操作：…

双端并行	电脑弹窗和 QQ 消息同时存在，谁先决定谁生效，另一端自动撤回/收起

免编号回复	直接回「同意」/「拒绝」；多条同时挂着时默认处理最新一条，也可「同意 ab12cd」指定

只认管理员	只有配置的 adminQq 能授权，其他人回复忽略（可选回一句「无权限」）

失败兜底	QQ 没连上 / 发送失败 / 任何异常 → 立即退回电脑端弹窗，不挂住请求

全程日志	~/.dsh/approval-qq/approval.log，记 id、命令、来源（qq/desktop）、结果


使用步骤

装（三选一）：

dsh plugin add dsh-approval-qq                      # npm（若已发布）

dsh plugin add github:<用户名>/dsh-approval-qq      # GitHub

dsh plugin add file:C:\path\dsh-approval-qq         # 本地目录

重启 DSH；

填配置：设置 → 插件 → dsh-approval-qq

adminQq：必填，你的 QQ 号

onebotHttpUrl / onebotHttpToken：NapCat 的 HTTP API（发消息用）

onebotWsUrl / onebotWsToken：NapCat 的正向 WS（收回复用）


验证：让模型跑一条需要提权的命令（或手动加 sandbox_permissions），QQ 应收到审批消息；

回复「同意」→ 命令执行；回复「拒绝」→ 工具被拒，模型会解释。


注意事项

必须有 OneBot v11 服务（NapCat / Lagrange / LLOneBot 都行），且开着 HTTP + 正向 WS；

WS 没连上时插件会自动跳过 QQ，只走电脑端——所以「没收到消息」先查 WS 是否连上；

adminQq 不填插件不生效（只打警告），这是故意的安全默认值；

这是 DSH 插件，需要 Node ≥ 22 + 一个 OneBot v11 服务（NapCat 等），并且不能直接装 zip——zip 里是分发包，要解压后装那个 .tgz。

不要和别的审批转发插件同时启用，否则一条审批两份消息、两个 answerer 抢决定；

日志里有命令原文，别把 approval.log 提交到公开仓库；

OneBot 的 HTTP/WS 端口不要暴露到公网，token 用强随机串。
