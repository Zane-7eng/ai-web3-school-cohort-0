# Hermes Agent 安装与配置记录

## 踩坑：装错 Hermes

第一次 `npm install -g hermes` 装的是 Segment 公司 2014 年的老聊天机器人，和课上讲的 AI Agent 完全无关。

正确做法：`pip install hermes-agent`（Hermes Agent v0.14.0 by Nous Research）。

## 配置

| 项目 | 值 |
|------|-----|
| 版本 | v0.14.0 (2026.5.16) |
| 模型 | deepseek-v4-flash |
| 提供商 | DeepSeek (API: api.deepseek.com) |
| 终端后端 | Local |
| 消息平台 | Telegram (@Zane7eng\_bot) |
| 开机自启 | Windows Startup 文件夹 |

## 配置文件位置

- `~/.hermes/config.yaml` - 主配置
- `~/.hermes/.env` - API Key

## 常用命令

```
hermes                  # 开始对话
hermes setup            # 全部设置
hermes model            # 换模型
hermes setup gateway    # 配消息平台
hermes gateway status   # 查看网关状态
hermes doctor           # 诊断问题
```

## Agent 五零件对照

- 目标：通过 prompt 定义
- 工具：5/10 可用（Vision、TTS、Terminal、Task Planning、Skills）
- 状态：sessions/ 目录持久化
- 权限：只读工具安全，写入须确认
- 停的条件：max_iterations=90

## 踩坑

1. npm `hermes` ≠ Nous Research `hermes-agent`，同名不同物
2. 旧 Telegram Bot 脚本和 Hermes Gateway 会抢 polling → 删掉旧 Bot
3. DeepSeek 在提供商列表里直接有（第 17 项），不用自定义
