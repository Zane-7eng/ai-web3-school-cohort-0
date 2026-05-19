# Handbook 反馈：Hermes Agent Windows 安装体验

## 问题 1：两个 Hermes 容易搞混

npm 上有一个老项目也叫 `hermes`（Segment 公司 2014 年的聊天机器人），和课程讲的 Hermes Agent 完全不是同一个东西。

新手在没看清的情况下 `npm install -g hermes` 就会装错。

**建议**：Handbook 首页安装命令旁边加一个醒目提示——"注意：请用 pip install hermes-agent，不是 npm install hermes"

---

## 问题 2：OpenClaw → Hermes 迁移时 Telegram 冲突

如果之前装过 OpenClaw 并配了 Telegram Bot，迁移到 Hermes 后两个 Gateway 会抢同一个 Bot Token，导致持续 polling conflict，Bot 间歇性"死机"。

普通用户看到 "Conflict: terminated by other getUpdates request" 完全不知道是什么问题。

**建议**：
- Hermes setup 检测到已有 OpenClaw 配置时，自动提示"检测到旧 Agent 正在使用同一 Bot，是否禁用旧配置？"
- 或者在 migration 文档里加一条：迁移前务必先删掉 OpenClaw 启动项

---

## 问题 3：Windows 原生安装路径不够明确

官方文档推荐 WSL2，但 Win11 原生 PowerShell 安装也是 beta 支持的。安装脚本 `install.ps1` 和 `pip install hermes-agent` 两种方式，新手不知道选哪个。

**建议**：给一个决策表——什么情况用 install.ps1，什么情况用 pip。
