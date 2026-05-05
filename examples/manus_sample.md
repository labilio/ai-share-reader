# Manus share example

## Input

```
https://manus.im/share/xxxxxxxxxxxxxxxx
```

## Output

```
# Hermes Agent是什么及其与其他AI的区别

**User:** 好像最新推出了一款新产品叫Hermes Agent我完全没听过那是什么，但是我玩过openclaw，claudecode等产品，你能用举例和类比的方式介绍一下Hermes 是什么吗？他厉害在哪里？他跟其他AI产品有什么相同和不同的地方？

**Assistant:** 没问题，我会为您调研 Hermes Agent 的相关信息。

（Agent 执行了多步操作：搜索 Hermes Agent 最新资讯 → 访问官网了解学习循环和技能创建机制 → 整理信息撰写对比文档）

## 调研结果

**1. 基本定义**
- 开发者: Nous Research
- 定位: 开源、自进化的 AI 智能体（Self-improving AI Agent）
- 核心理念: "The agent that grows with you"（伴随你成长的智能体）
- 许可证: MIT 开源协议

**2. 核心特性**
- **学习循环 (Learning Loop)**: 能够从经验中创建技能，并在使用过程中不断改进。
- **技能系统 (Skills System)**: 遵循 agentskills.io 标准。技能是按需生成的知识文档或脚本，存储在本地，比传统的"工具（Tools）"更易创建且可移植。
- **持久化记忆 (Persistent Memory)**: 多层级记忆系统，能够跨会话记住项目细节、用户偏好和解决问题的方案。
- **多平台接入**: 支持 Telegram, Discord, Slack, WhatsApp, Signal, Email, CLI 等。
- **自动化调度**: 支持自然语言设置 Cron 任务。
- **沙箱环境**: 支持 Docker, SSH, Singularity, Modal 等多种后端。

**3. 与其他产品对比**
- **Claude Code**: 像是一个高级电动工具（如激光切割机），精准高效，但不会自己学习新技能。
- **OpenClaw**: 像是一个万能工具箱，配件丰富，但需要自己动手配置。
- **Hermes Agent**: 像是一个数字学徒/实习生。他住在你的服务器里，你通过微信或 Slack 吩咐他做事。他第一次可能不会，但他会自己查资料、写脚本（创建技能），下次再问他，他就已经学会了。
```
