# AI Share Reader

> _分享链接扔进来，干净的对话吐出去。_

[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)
[![Agent-Agnostic](https://img.shields.io/badge/Agent-Agnostic-blueviolet)]()

读取主流 AI 平台分享链接中的对话内容。支持 18 个平台入口，自动识别平台、选择提取策略、输出干净对话。跨 agent 通用。

```bash
git clone https://github.com/Secret24/ai-share-reader.git ~/.claude/skills/ai-share-reader
```

## 为什么需要这个 Skill

当你收到朋友发来的 AI 对话分享链接时：

- **必须打开浏览器**才能看——DeepSeek、千问、豆包等平台的分享页是纯前端渲染，链接预览、HTTP 抓取都拿不到内容
- **要求登录**才能看——ChatGPT、Gemini 弹出登录框挡住页面，实际上对话内容已经在背后渲染好了
- **链接里夹带广告语**——复制链接时平台会自动加上"点击查看 ××× 的回答 https://..."，直接粘贴很乱
- **每个平台界面不一样**——分享页充斥着侧边栏、推荐、登录按钮，只想快速看对话内容却很费劲

**更深一层的问题是：每个 AI 平台的分享页设计逻辑完全不同。** 大致分为三大类，细分还有更多变种：

| 类型 | 代表平台 | 特点 |
|------|---------|------|
| 服务端渲染 | Claude、Kimi、Perplexity | 对话在 HTML 里，HTTP 请求即可拿到 |
| 客户端 SPA | DeepSeek、千问、豆包、Manus | 对话在 JS 里，必须用浏览器渲染 |
| 登录遮罩 | ChatGPT、Gemini | 对话已渲染但被登录框挡住，需特殊处理 |

同一类里还有差别：Perplexity 有 Cloudflare 反爬、Gemini 的快照只能看到遮罩层、Manus 展示的是任务回放而非 Q&A……这意味着**每接入一个平台，都需要单独分析、单独适配**。

而这恰恰是 **Skill 最擅长的场景**——把每种平台的适配逻辑封装进同一个 Skill，用户只需要粘贴链接，Skill 自动识别平台、选择策略、提取干净对话。用户不需要知道背后是 HTTP 还是浏览器、是 SSR 还是 SPA。

## 能做什么

| 操作 | 说明 |
|------|------|
| 粘贴即读 | 粘贴分享链接，自动提取完整对话 |
| 自动识别 | 18 个平台入口，自动匹配提取策略 |
| 容错输入 | 链接夹带广告语也能正确解析 |
| 免登录 | ChatGPT、Gemini 无需登录即可读取 |
| 统一格式 | 所有平台输出统一的 `**User:**` / `**Assistant:**` 对话格式 |

## 使用示例

```
你: https://chat.deepseek.com/share/xxxxxxxxxxxxxxxxxxxxxxx
你: 点击查看元宝的回答 https://yb.tencent.com/s/xxxxxxxxxxxxxx
你: https://claude.ai/share/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

输出：

```
# 对话标题

**User:** 用户的问题

**Assistant:** AI 的回答

**User:** 追问

**Assistant:** 后续回答
```

## 支持平台

共 18 个平台入口，覆盖 16 个 AI 产品。每个平台均使用真实分享链接逐一测试验证。

### 服务端渲染（HTTP 提取）

| 平台 | 域名 |
|------|------|
| Claude | `claude.ai/share/` |
| Kimi | `kimi.com/share/` |
| Perplexity | `perplexity.ai/search/` |
| Poe | `poe.com/s/` |
| Grok | `grok.com/share/` |
| 元宝 | `yb.tencent.com/s/` |

### 客户端 SPA（浏览器提取）

| 平台 | 域名 |
|------|------|
| DeepSeek | `chat.deepseek.com/share/` |
| 千问 | `qianwen.com/share/chat/` |
| 智谱 | `chat.z.ai/s/` · `chatglm.cn/share/` |
| 豆包 | `doubao.com/thread/` |
| Minimax | `agent.minimaxi.com` · `agent.minimax.io` |
| 文心一言 | `yiyan.baidu.com/share/` |
| Mistral | `chat.mistral.ai/chat/` |
| Manus | `manus.im/share/` |
| AnyGen | `anygen.io/task/` |

### 登录遮罩（浏览器提取）

| 平台 | 域名 |
|------|------|
| ChatGPT | `chatgpt.com/share/` |
| Gemini | `gemini.google.com/share/` |

## 仓库结构

```
ai-share-reader/
├── SKILL.md                 # 主文档（给 agent 读）
├── README.md                # 本文件
├── LICENSE
├── examples/                # 每个平台的输入/输出样例
│   ├── claude_sample.md
│   ├── deepseek_sample.md
│   ├── kimi_sample.md
│   ├── qianwen_sample.md
│   ├── zai_sample.md
│   ├── chatglm_sample.md
│   ├── doubao_sample.md
│   ├── minimax_sample.md
│   ├── chatgpt_sample.md
│   ├── gemini_sample.md
│   ├── perplexity_sample.md
│   ├── poe_sample.md
│   ├── grok_sample.md
│   ├── yuanbao_sample.md
│   ├── yiyan_sample.md
│   ├── mistral_sample.md
│   ├── manus_sample.md
│   └── anygen_sample.md
└── dev/                     # 开发笔记
```

## Limitations

- **不支持需要登录才能看到内容的平台**（如 Poe 的 `invite` 链接）。分享链接本身必须公开可访问。
- **输出为纯文本**，不保留原始平台的表情、代码高亮、LaTeX 渲染等富文本效果。
- **部分平台（Perplexity、Grok）** 有 Cloudflare 人机验证，只能用 HTTP 方式提取，不能用浏览器打开。
- **不支持连续多页对话**。如果分享链接只展示部分对话（如 Poe 的长对话分页），只能提取当前可见部分。

## 设计参考

本 Skill 的设计遵循 **Anthropic 官方 Skill 设计规范**，并使用 **skill-creator** 方法论进行迭代开发。

- Anthropic Skills 官方仓库：[github.com/anthropics/skills](https://github.com/anthropics/skills)
- 多 agent 适配方案参考了 [huashu-design](https://github.com/alchaincyf/huashu-design) 的跨平台设计

## 许可

MIT
