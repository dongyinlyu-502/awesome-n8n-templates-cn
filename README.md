# awesome-n8n-templates-cn

面向国内常见业务环境的 n8n 自动化模板集合。

当前先做一个专业模板：

- `templates/email-management/mail-digest-imap-ai-smtp`: 邮件早晚 AI 简报

## 设计原则

- 优先兼容国内邮箱：QQ 邮箱、网易 163/126、阿里企业邮箱、腾讯企业邮箱、自建 IMAP/SMTP。
- AI 默认使用 OpenAI-compatible HTTP API，方便接入 DeepSeek、Moonshot、通义千问兼容网关等服务。
- 模板默认不删除邮件、不下载附件、不自动回复，避免误操作。
- 邮件正文作为不可信输入处理，只做摘要和待办提取。

