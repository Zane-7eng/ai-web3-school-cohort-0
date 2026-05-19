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
| 消息平台 | Telegram (个人 Bot) |
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

## 排查：TG Bot 持续报"AI 服务出错"

**现象**：Hermes 配好后，TG Bot 间歇性回复"抱歉，AI 服务出错了，请稍后再试。"Gateway 日志持续报 Telegram polling conflict。

**排错过程**：

| 步骤 | 发现 | 解决 |
|------|------|------|
| 1. 查日志 | `Conflict: terminated by other getUpdates request` | 有多个进程在抢同一个 Bot 的 polling |
| 2. 查进程 | PID 47688 是旧 `bot.py` 脚本（自己写的那个）| `taskkill` 杀掉 |
| 3. 再查 | 冲突依旧 | 还有东西 |
| 4. 查启动文件夹 | `OpenClaw Gateway.cmd` 和 `Hermes_Gateway.cmd` 共存 | OpenClaw 是旧 Agent（v2026.4.11），在用同一个 Bot Token |
| 5. 确认 | `/openclaw/openclaw.json` 里 `botToken` 和 Hermes 用的完全相同 | 两个 Agent 抢同一个 Bot |
| 6. 修复 | 删掉 OpenClaw 启动项，关掉 OpenClaw 的 TG 进程 | Hermes 独占 Bot，恢复正常 |

**教训**：
- 一个 TG Bot Token 只能被一个 Gateway 实例使用
- 安装新 Agent 前，先检查旧 Agent 是否占用了同样的平台配置
- OpenClaw → Hermes 迁移时，平台配置不会自动解除绑定
- 排查这类问题时先 `wmic process get CommandLine` 看所有进程在跑什么
