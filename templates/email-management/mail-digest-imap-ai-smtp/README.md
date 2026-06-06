# 邮件早晚 AI 简报

每天早上 7 点和晚上 9 点自动抓取邮件，用 AI 提炼关键信息和待办事项，并把简报发给相关人员。

这个模板既可以作为本地演示项目，也可以接入真实邮箱和 n8n 进行自动化部署。

## 适用场景

- 管理者每天邮件很多，需要快速知道重点。
- 销售、客服、运营、项目负责人需要减少漏看邮件和遗忘跟进。
- 团队希望每天早晚固定同步邮件重点。
- 邮箱来自 QQ、网易、腾讯企业邮箱、阿里企业邮箱或自建 IMAP/SMTP。
- 想用 DeepSeek、Moonshot、通义千问兼容网关等 AI 服务生成中文简报。

## 工作流

1. n8n 的 Schedule Trigger 在每天 `07:00` 和 `21:00` 执行。
2. Execute Command 调用 `scripts/mail_digest.py`。
3. 脚本通过 IMAP 拉取上次执行后新增的邮件。
4. 调用 OpenAI-compatible AI 接口生成中文结构化简报。
5. 通过 SMTP 发送给 `DIGEST_TO` 中配置的收件人。
6. 成功发送后更新本地状态文件，避免重复汇总。

## 本地演示模式

演示模式不会连接真实邮箱、不会调用真实 AI、不会发送邮件，只会用样例邮件生成一份 HTML 简报。

```bash
cp .env.demo .env
python scripts/mail_digest.py
```

Windows PowerShell:

```powershell
Copy-Item .env.demo .env
python scripts\mail_digest.py
```

运行后打开：

```text
preview/latest.html
```

即可查看效果。

## 真实部署模式

1. 把本目录放到 n8n 容器或服务器中，例如：

```text
/data/templates/mail-digest-imap-ai-smtp
```

2. 复制 `.env.example` 为 `.env`，填入邮箱、AI、收件人配置，并保持：

```env
DEMO_MODE=false
```

3. 在 n8n 导入 `workflow.json`。

4. 修改 Execute Command 节点里的路径：

```bash
cd /data/templates/mail-digest-imap-ai-smtp && python scripts/mail_digest.py
```

5. 确认 n8n 工作流时区是：

```text
Asia/Shanghai
```

6. 保存并激活工作流。

## 配置说明

核心配置在 `.env` 中：

```env
DEMO_MODE=false

IMAP_HOST=imap.example.com
IMAP_PORT=993
IMAP_SSL=true
IMAP_USER=your_mail@example.com
IMAP_PASSWORD=your_mail_auth_code_or_password

SMTP_HOST=smtp.example.com
SMTP_PORT=465
SMTP_SSL=true
SMTP_USER=your_mail@example.com
SMTP_PASSWORD=your_mail_auth_code_or_password
MAIL_FROM=your_mail@example.com
DIGEST_TO=person1@example.com,person2@example.com

LLM_BASE_URL=https://api.deepseek.com/v1
LLM_API_KEY=sk-xxxxxxxx
LLM_MODEL=deepseek-chat

COMPANY_NAME=你的公司
TIMEZONE=Asia/Shanghai
DIGEST_LOOKBACK_HOURS=14
DIGEST_MAX_EMAILS=50
```

## 常见邮箱配置

| 邮箱 | IMAP | SMTP | 说明 |
| --- | --- | --- | --- |
| QQ 邮箱 | `imap.qq.com:993` | `smtp.qq.com:465` | 通常需要开启 IMAP/SMTP，并使用授权码 |
| 163 邮箱 | `imap.163.com:993` | `smtp.163.com:465` | 通常需要客户端授权码 |
| 126 邮箱 | `imap.126.com:993` | `smtp.126.com:465` | 通常需要客户端授权码 |
| 腾讯企业邮箱 | `imap.exmail.qq.com:993` | `smtp.exmail.qq.com:465` | 通常使用企业邮箱授权码 |
| 阿里企业邮箱 | `imap.qiye.aliyun.com:993` | `smtp.qiye.aliyun.com:465` | 以管理员配置为准 |

## 输出简报结构

- 本时段概览
- 重要邮件
- 待办事项
- 风险与阻塞
- 等待他人回复
- 建议跟进话术

## 安全建议

- 使用邮箱授权码，不要使用主登录密码。
- 不要开启自动回复，第一版只做摘要和通知。
- `DIGEST_MARK_SEEN=false` 保持默认，不自动标记已读。
- 本地演示时使用 `.env.demo`，不要填写真实密钥。
- 不要把 `.env` 提交到 GitHub。
- 如果邮件中含合同、客户隐私、财务信息，优先使用企业批准的 AI 服务或私有化模型。

## 扩展方向

- 增加企业微信机器人通知。
- 增加飞书群通知。
- 把待办事项写入飞书多维表格。
- 对邮件做客户、项目、优先级分类。
- 对重要邮件生成回复草稿，但不自动发送。

