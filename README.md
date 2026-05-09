# AI Share Reader

> _AI 分享链接扔进来，干净的对话吐出去。_
> _你手机上的 AI 对话记录，可以随时变成 Agent 的上下文。_

[![License](https://img.shields.io/badge/License-Apache_2.0-green)](LICENSE)
[![Agent-Agnostic](https://img.shields.io/badge/Agent-Agnostic-blueviolet)]()
[![Claude Code Skill](https://img.shields.io/badge/Claude_Code-Skill-orange)]()

**一个给 Claude Code、Codex、Openclaw、Hermes 等 AI Agent 用的 Skill。好的 Skill 不挑 Agent。**

 读取主流 AI 平台的分享链接，自动识别 18 个平台入口、选择提取策略、输出干净对话文本。

## 安装方法

### 方式一：下载 ZIP（推荐）

从 [Releases](https://github.com/labilio/ai-share-reader/releases) 页面下载最新版本的 `ai-share-reader-vX.X.X.zip`，解压到你 Agent 的 skills 目录即可。

以 Claude Code 为例：

```bash
# 解压到 Claude Code 的 skills 目录
unzip ai-share-reader-v*.zip -d ~/.claude/skills/
```

### 方式二：Git Clone

```bash
git clone https://github.com/labilio/ai-share-reader.git ~/.claude/skills/ai-share-reader
```

## 为什么你需要这个 Skill

**场景一：跨设备，自己给自己传  ——  一个没人解决过的痛点。**

**你其实并不是随时都在电脑前用 Claude Code 或 OpenClaw。**

- 通勤路上、排队时，掏出手机用 DeepSeek App 聊了一路，产出了不错的思路。
- 回家打开电脑，想让 Agent 把这对话当上下文接着用——怎么传？
  - 复制粘贴？几十轮对话根本搬不动。
  - 截图？ 实测 Agent 里面用的一些大模型读图能力并不高。
  - 截图 OCR？丢一半内容。

- 最自然的方式：手机 App 里点分享复制链接 → 发给自己 → 电脑上 Agent 直接读取。
  
  但问题来了——这些链接 Agent 自己读不了，因为每个平台的分享页都是给人看、不是给 Agent 看的。

**这个需求一直存在，但从来没有人做过。**  AI 平台的分享链接本质上是"人类阅读器"，Agent 无法从中提取结构化对话。

**AI Share Reader 是第一个系统解决这个问题的 Skill。**

**场景二：别人分享给你。**

- **必须打开浏览器**才能看——DeepSeek、千问、豆包等平台的分享页是纯前端渲染，链接预览、HTTP 抓取都拿不到内容
- **要求登录**才能看——ChatGPT、Gemini 弹出登录框挡住页面，实际上对话内容已经在背后渲染好了
- **链接里夹带广告语**——复制链接时平台会自动加上"点击查看 ××× 的回答 https://..."，还不能直接粘贴到浏览器，很麻烦
- **每个平台界面不一样**——分享页充斥着侧边栏、推荐、登录按钮，只想快速看对话内容却很费劲

**更深一层的问题是：每个 AI 平台的分享页设计逻辑完全不同。**

 大致分为三大类，细分还有更多变种：

| 类型 | 代表平台 | 特点 |
|------|---------|------|
| A 类 · 服务端渲染 | Claude、Kimi、Perplexity | 对话在 HTML 里，HTTP 请求直接拿到 |
| B 类 · 客户端 SPA | DeepSeek、千问、豆包、Manus | HTTP 能拿到 HTML，但是空壳——对话由 JS 动态生成，必须浏览器渲染 |
| C 类 · 登录遮罩 | ChatGPT、Gemini | HTTP 直接被拦截（未登录），但浏览器打开后对话在遮罩背后已渲染，无需真正登录 |

同一类里还有差别：Perplexity 有 Cloudflare 反爬、Gemini 的快照只能看到遮罩层、Manus 展示的是任务回放而非 Q&A……这意味着**每接入一个平台，都需要单独分析、单独适配**。

**而这恰恰是** **Skill 最擅长的场景**——不管是自己跨设备传给自己，还是别人分享给你，都是同一个操作：粘贴链接。剩下的，Skill 自动识别平台、选择策略、提取干净对话。

## 能做什么

| 操作 | 说明 |
|------|------|
| 粘贴即读 | 粘贴分享链接，自动提取完整对话 |
| 自动识别 | 18 个平台入口，自动匹配提取策略 |
| 容错输入 | 链接夹带广告语也能正确解析 |
| 免登录 | ChatGPT、Gemini 无需登录即可读取 |
| 统一格式 | 所有平台输出统一的 `**User:**` / `**Assistant:**` 对话格式 |

## 触发方式

以下任意一种输入都会自动激活 Skill：

- 直接粘贴 AI 分享链接
- "帮我读一下这个对话"
- "这个链接里说了什么？"
- "提取这个分享的内容"
- "看看这个 AI 聊天记录"

即使只贴链接、不说任何话，Skill 也会自动识别并开始提取。

## 使用示例

直接粘贴**任意 AI 对话分享链接**即可，支持夹带广告语的容错输入：

```
https://chat.deepseek.com/share/xxxxxxxxxxxxxxxxxxxxxxx
```

```
点击查看元宝的回答 https://yb.tencent.com/s/xxxxxxxxxxxxxx
```

```
https://claude.ai/share/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

Skill 自动识别平台并提取：

```
对话标题

User: 用户的问题
Assistant: AI 的回答

User: 追问
Assistant: 后续回答

User: 追问
Assistant: 后续回答
```

## 支持平台

共 18 个平台入口，覆盖 16 个 AI 产品。每个平台均使用真实分享链接逐一测试验证。

### A 类 · 服务端渲染（HTTP 提取）

| # | 支持平台 | 域名 |
|---|------|------|
| 1 | Claude | `claude.ai/share/` |
| 2 | Kimi | `kimi.com/share/` |
| 3 | Perplexity | `perplexity.ai/search/` |
| 4 | Poe | `poe.com/s/` |
| 5 | Grok | `grok.com/share/` |
| 6 | 元宝 | `yb.tencent.com/s/` |

### B 类 · 客户端 SPA（浏览器提取）

| # | 支持平台 | 域名 |
|---|------|------|
| 7 | DeepSeek | `chat.deepseek.com/share/` |
| 8 | 千问 | `qianwen.com/share/chat/` |
| 9 | 智谱 | `chat.z.ai/s/` · `chatglm.cn/share/` |
| 10 | 豆包 | `doubao.com/thread/` |
| 11 | Minimax | `agent.minimaxi.com` · `agent.minimax.io` |
| 12 | 文心一言 | `yiyan.baidu.com/share/` |
| 13 | Mistral | `chat.mistral.ai/chat/` |
| 14 | Manus | `manus.im/share/` |
| 15 | AnyGen | `anygen.io/task/` |

### C 类 · 登录遮罩（浏览器提取）

| # | 支持平台 | 域名 |
|---|------|------|
| 16 | ChatGPT | `chatgpt.com/share/` |
| 17 | Gemini | `gemini.google.com/share/` |



B 类和 C 类虽然都用浏览器，但原因不同：**B 类是 HTTP 拿到了空壳**（内容不存在于 HTML 中，需要 JS 执行才能生成）；**C 类是 HTTP 根本拿不到**（被服务器拦截），但浏览器打开时遮罩背后内容已经在了，无需真正登录。

---

### 补充说明

**别担心兼容性问题，有专门适配！**

| 支持平台       | 适配对象              |
| -------------- | --------------------- |
| Manus          | 任务回放部分          |
| AnyGen         | 任务执行部分          |
| Poe            | 多个 AI ChatBot 群聊  |
| Mistral        | ToS 弹窗              |
| 智谱 & Minimax | 中国版 & 国际版双域名 |
| ……             | ……                    |

**实时更新，确保有效**！

- 各平台测试链接已储存，实时测试 Skill 方法是否依旧有效
- 如厂商前端 UI 更新，Skill 会同步更新



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
```

## Limitations

- **不支持需要登录才能看到内容的平台**（如 Poe 的 `invite` 链接）。分享链接本身必须公开可访问。
- **输出为纯文本**，不保留原始平台的表情、代码高亮、LaTeX 渲染等富文本效果。
- **部分平台（Perplexity、Grok）** 有 Cloudflare 人机验证，只能用 HTTP 方式提取，不能用浏览器打开。
- **不支持连续多页对话**。如果分享链接只展示部分对话（如 Poe 的长对话分页），只能提取当前可见部分。

## 设计过程

这个 Skill 不是拍脑袋写的。动工之前，先花时间把行业最优秀的 Skill 是怎么做的搞清楚了。

### 1. 研究 Anthropic 官方设计规范

首先系统学习了 [Anthropic 官方 Skills 仓库](https://github.com/anthropics/skills) 的整套方法论，提炼出核心设计原则：

- **Progressive Disclosure（三级加载）**：元数据 → SKILL.md 正文 → 按需加载资源。Claude 只在触发时才读正文，不要把所有东西塞进上下文
- **祈使句 + 解释为什么**：写"Extract the conversation"而非"You should extract..."；用原因替代大写的 MUST/ALWAYS
- **description 是触发核心**：Skill 能不能被正确触发，关键在 frontmatter 的 description，要写得偏"pushy"（Claude 倾向于欠触发）
- **保持 SKILL.md < 500 行**：超过就拆出 references/，Claude 只读需要的那份

### 2. 审计全部 17 个官方 Skill

光知道原则不够——需要看真实案例。于是把 Anthropic 全部 17 个 Skill 逐一拆解，按复杂度分成六个等级（Level 0 ~ Level 5），分析每种文件什么时候该出现、什么时候不该出现：

| 发现 | 结论 |
|------|------|
| scripts/ 里全是 Python 脚本，只做 Agent 做不好的事：二进制读写（PDF/DOCX/XLSX）、打包构建、确定性计算 | ai-share-reader 的核心逻辑是"判断平台 → 选策略 → 提取"，全是 Agent 擅长的决策和文本处理，不需要 Python |
| examples/ 不限于 `.py`——internal-comms 就有 `.md` 示例 | 我们的 `examples/` 目录有先例支撑 |
| 没有一个 Skill 是为了"看起来复杂"而加文件 | 保持诚实：Level 0 就是最合适的结构 |

### 3. 落地到 ai-share-reader

基于以上研究，做了几个刻意选择：

- **SKILL.md 不到 500 行**，不需要拆分 references/——保持简单
- **examples/ 保留**——每个平台一个 `.md` 样例，既是文档也是测试预期
- **不用 MUST/ALWAYS 堆砌规则**——每个平台策略写清楚"为什么"这么做
- **不加 Python 脚本**——Anthropic 的 scripts/ 只在处理二进制文件（PDF、DOCX）、打包构建、需要确定性计算时才出现。我们这个 Skill 做的事——识别域名、选择策略、文本提取——全是 Agent 原生擅长的，不需要 Python 代劳

### 4. 多 Agent 适配

跨平台适配方案参考了 [huashu-design](https://github.com/alchaincyf/huashu-design) 的设计——工具名不写入指令，能力映射表替代具体工具名，让不同 agent 平台用自己的工具完成相同能力。

---

站在巨人的肩膀上，向 Anthropic Skills 团队和 huashu-design 作者致敬。优秀的开源工作让后来者不必从零开始。

## 许可

Apache 2.0
