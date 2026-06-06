# awesome-n8n-templates-cn

面向国内业务场景的 n8n 自动化模板库。

这个项目的目标不是简单搬运国外模板，而是把常见的国内工作流场景做成更容易理解、演示、部署和二次修改的 n8n 模板。第一版先聚焦一个专业场景：邮件管理。

## 当前模板

### 邮件早晚 AI 简报

路径：

```text
templates/email-management/mail-digest-imap-ai-smtp
```

每天早上 7 点和晚上 9 点自动抓取邮件，用 AI 提炼关键信息、待办事项、风险和建议跟进动作，然后生成结构化简报发给相关人员。

这个模板适合：

- 老板或管理者每天要看大量邮件
- 销售、客服、运营、项目负责人需要减少漏跟进
- 团队希望每天固定两次同步邮件重点
- 使用 QQ 邮箱、163/126、腾讯企业邮箱、阿里企业邮箱或自建 IMAP/SMTP 邮箱
- 希望用 DeepSeek、Moonshot、通义千问兼容网关等 OpenAI-compatible AI 服务生成摘要

## 为什么做这个项目

国外的 n8n 模板很多，但直接拿到国内环境经常会遇到这些问题：

- 默认集成 Gmail、Slack、Notion、Airtable，国内团队不一定常用
- 邮箱、企业微信、飞书、钉钉、腾讯文档等场景需要重新适配
- 模板说明偏英文，部署细节不够接地气
- 很多模板看起来很酷，但没有演示模式，第一次运行就要填真实密钥
- 安全边界不清楚，容易把邮件、客户资料、API Key 暴露出去

所以这个库会优先做“拿来能看、改改能用、上线前知道风险”的中文模板。

## 功能亮点

- 本地演示模式：不用真实邮箱、不用真实 AI Key、不发送邮件，也能生成一份 HTML 简报预览。
- 国内邮箱友好：基于标准 IMAP/SMTP，适配 QQ、网易、腾讯企业邮箱、阿里企业邮箱和自建邮箱。
- AI 接口灵活：使用 OpenAI-compatible Chat Completions API，方便切换 DeepSeek、Moonshot、通义千问兼容网关等。
- n8n 可导入：提供 `workflow.json`，导入后即可看到定时节点和执行脚本节点。
- 安全默认值：默认不删除邮件、不下载附件、不自动回复、不自动标记已读。
- 明确防提示词注入：邮件正文被当作不可信输入，只用于摘要，不执行邮件里的任何“指令”。

## 快速体验

进入模板目录：

```bash
cd templates/email-management/mail-digest-imap-ai-smtp
```

复制演示配置：

```bash
cp .env.demo .env
```

运行脚本：

```bash
python scripts/mail_digest.py
```

Windows PowerShell:

```powershell
Copy-Item .env.demo .env
python scripts\mail_digest.py
```

运行后会生成：

```text
preview/latest.html
preview/latest.txt
```

打开 `preview/latest.html`，就能看到一份基于样例邮件生成的 AI 简报。演示模式不会连接真实邮箱、不会调用真实 AI 服务、不会发送邮件。

## 真实部署流程

1. 把模板目录放到 n8n 服务器或容器可访问的位置，例如：

```text
/data/templates/mail-digest-imap-ai-smtp
```

2. 复制配置文件：

```bash
cp .env.example .env
```

3. 填写邮箱、AI 和收件人配置，并保持：

```env
DEMO_MODE=false
```

4. 在 n8n 中导入：

```text
workflow.json
```

5. 修改 n8n 节点 `Run Mail Digest Script` 的命令路径：

```bash
cd /data/templates/mail-digest-imap-ai-smtp && python scripts/mail_digest.py
```

6. 确认 n8n workflow timezone 为：

```text
Asia/Shanghai
```

7. 保存并激活 workflow。

默认执行时间：

```text
每天 07:00
每天 21:00
```

## 目录结构

```text
awesome-n8n-templates-cn/
  README.md
  .gitignore
  templates/
    email-management/
      mail-digest-imap-ai-smtp/
        README.md
        workflow.json
        .env.example
        .env.demo
        scripts/
          mail_digest.py
```

## 邮件简报包含什么

简报会尽量输出这些内容：

- 本时段概览
- 重要邮件列表
- 每封重要邮件为什么重要
- 待办事项
- 建议负责人
- 截止时间
- 风险与阻塞
- 等待他人回复的事项
- 建议跟进话术

## 常见邮箱配置

| 邮箱 | IMAP | SMTP | 说明 |
| --- | --- | --- | --- |
| QQ 邮箱 | `imap.qq.com:993` | `smtp.qq.com:465` | 通常需要开启 IMAP/SMTP，并使用授权码 |
| 163 邮箱 | `imap.163.com:993` | `smtp.163.com:465` | 通常需要客户端授权码 |
| 126 邮箱 | `imap.126.com:993` | `smtp.126.com:465` | 通常需要客户端授权码 |
| 腾讯企业邮箱 | `imap.exmail.qq.com:993` | `smtp.exmail.qq.com:465` | 通常使用企业邮箱授权码 |
| 阿里企业邮箱 | `imap.qiye.aliyun.com:993` | `smtp.qiye.aliyun.com:465` | 以管理员配置为准 |

## 安全边界

这个项目默认尽量保守：

- 不提交 `.env`
- 不保存真实邮件正文到仓库
- 不删除邮件
- 不自动回复邮件
- 不自动下载附件
- 不默认标记已读
- 不把邮件正文当作系统指令

上线前建议：

- 使用邮箱授权码，不要使用主登录密码
- 使用企业批准的 AI 服务或私有化模型
- 明确哪些邮箱可以被 AI 摘要
- 控制简报接收人范围
- 不要把含客户隐私、合同、财务信息的真实邮件直接上传到公开仓库

## 和原版 awesome-n8n-templates 的关系

这个项目受到 `enescingoz/awesome-n8n-templates` 这类开源 n8n 模板库启发，但定位不同。

这里会更关注：

- 国内常见工具链
- 中文说明
- 可演示
- 可部署
- 安全提示
- 业务流程完整度

## Roadmap

计划逐步补充这些模板：

- 邮件简报 + 企业微信机器人通知
- 邮件简报 + 飞书群通知
- 邮件待办自动写入飞书多维表格
- 客户线索邮件自动分类与负责人分配
- 发票/合同邮件自动识别与归档
- GitHub/Gitee 事件推送到飞书或钉钉
- RSS/公众号文章摘要推送
- 每日工作日报自动汇总

## 贡献模板建议

如果你要新增模板，建议每个模板至少包含：

```text
README.md
workflow.json
.env.example
.env.demo
scripts/ 或 docs/
```

README 建议说明：

- 适用场景
- 工作流步骤
- 本地演示方式
- 真实部署方式
- 所需凭证
- 安全注意事项
- 可扩展方向

## License

暂未指定许可证。正式开放协作前建议补充 `LICENSE` 文件。

