# 邮件早晚 AI 简报

每天早上 7 点和晚上 9 点自动抓取邮件，用 AI 提炼关键信息和待办事项，并把简报发给相关人员。

## 适用场景

- 老板、销售、客服、运营、项目负责人每天邮件量较大。
- 需要减少漏看重要邮件、遗忘跟进、跨团队信息不同步。
- 邮箱来自 QQ、网易、腾讯企业邮箱、阿里企业邮箱、自建 IMAP/SMTP 等。

## 工作流

1. n8n 的 Schedule Trigger 在每天 `07:00` 和 `21:00` 执行。
2. Execute Command 调用 `scripts/mail_digest.py`。
3. 脚本通过 IMAP 拉取上次执行后新增的邮件。
4. 调用 OpenAI-compatible AI 接口生成中文结构化简报。
5. 通过 SMTP 发送给 `DIGEST_TO` 中配置的收件人。
6. 成功发送后更新本地状态文件，避免重复汇总。

## 文件

- `workflow.json`: n8n 可导入工作流。
- `scripts/mail_digest.py`: 邮件抓取、AI 摘要、发送简报脚本。
- `.env.example`: 环境变量示例。

## 快速使用

### 本地演示模式

这个模式不会连接真实邮箱、不会调用真实 AI、不会发送邮件，只会用样例邮件生成一份 HTML 简报。

```bash
cp .env.demo .env
python scripts/mail_digest.py
```

运行后打开 `preview/latest.html` 即可查看效果。

Windows PowerShell:

```powershell
Copy-Item .env.demo .env
python scripts\mail_digest.py
```

### 真实部署模式

1. 把本目录放到 n8n 容器或服务器中，例如 `/data/templates/mail-digest-imap-ai-smtp`。
2. 复制 `.env.example` 为 `.env`，填入邮箱、AI、收件人配置，并保持 `DEMO_MODE=false`。
3. 在 n8n 导入 `workflow.json`。
4. 修改 Execute Command 节点里的路径：

```bash
cd /data/templates/mail-digest-imap-ai-smtp && python scripts/mail_digest.py
```

5. 确认 n8n 工作流时区是 `Asia/Shanghai`，保存并激活工作流。

## 常见邮箱配置

| 邮箱 | IMAP | SMTP | 说明 |
| --- | --- | --- | --- |
| QQ 邮箱 | `imap.qq.com:993` | `smtp.qq.com:465` | 通常需要开启 IMAP/SMTP 并使用授权码 |
| 163 邮箱 | `imap.163.com:993` | `smtp.163.com:465` | 通常需要客户端授权码 |
| 腾讯企业邮箱 | `imap.exmail.qq.com:993` | `smtp.exmail.qq.com:465` | 通常使用企业邮箱授权码 |
| 阿里企业邮箱 | `imap.qiye.aliyun.com:993` | `smtp.qiye.aliyun.com:465` | 以管理员配置为准 |

## 安全建议

- 使用邮箱授权码，不要使用主登录密码。
- 不要开启自动回复，第一版只做摘要和通知。
- `DIGEST_MARK_SEEN=false` 保持默认，不自动标记已读。
- 本地演示时使用 `.env.demo`，不要填写真实密钥。
- 不要把 `.env` 提交到 GitHub。
- 如果邮件中含合同、客户隐私、财务信息，优先使用企业批准的 AI 服务或私有化模型。

## 输出简报结构

- 今日/本时段概览
- 重要邮件
- 待办事项
- 风险与阻塞
- 等待他人回复
- 建议跟进话术
