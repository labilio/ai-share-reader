---
name: ai-share-reader
description: >
  Read and extract AI chat conversations from share links. Use this skill whenever
  the user sends a URL from claude.ai/share/, kimi.com/share/, chat.deepseek.com/share/, or qianwen.com/share/chat/.
  Also use it when the user asks to "read this chat link", "what does this conversation say",
  "extract this dialogue", "show me the chat content", "what's in this share", "pull out the
  messages from this link", or any similar request involving an AI platform share URL. If the
  user pastes a URL that looks like an AI chat share, always try this skill first — even if
  they don't explicitly say "extract" or "read".
---

# AI Share Reader

Extract AI chat conversations from mainstream platform share links and present them as clean, readable dialogue. This skill is designed to work across any agent platform that supports markdown-based skills.

## What this skill does

- Accepts **one** share URL at a time
- Identifies which AI platform the link comes from
- Extracts the full conversation
- Displays it as clean dialogue text directly to the user — no files saved to disk

## Output format

Present the conversation like this:

```
# [Conversation title if available, otherwise "AI Chat"]

**User:** Hello, can you explain quantum computing?

**Assistant:** Sure! Quantum computing uses qubits instead of bits...

**User:** What's a qubit?

**Assistant:** A qubit is a quantum bit that can exist in multiple states...
```

Each turn is clearly labeled with the speaker role. Keep the formatting simple and readable.

---

## Quick Reference

| Platform | Domain | Page type | Required capability |
|----------|--------|-----------|-------------------|
| Claude | `claude.ai/share/` | Server-rendered | HTTP fetch |
| Kimi | `kimi.com/share/` | Server-rendered | HTTP fetch |
| DeepSeek | `chat.deepseek.com/share/` | Client-rendered SPA | Browser automation |
| 千问 | `qianwen.com/share/chat/` | Client-rendered SPA | Browser automation |

## Flow

1. Parse the URL domain to identify the platform
2. Follow the extraction strategy for that platform (see below)
3. Format the result using the output template above and present to the user

### Retry and fallback rules

When a tool fails, follow these boundaries:

- **Same capability, different tool** — If one HTTP fetcher fails, try another HTTP fetcher your platform provides. For example: WebFetch fails → try WebReader, or curl, or a native HTTP client. This is fine.
- **Do NOT silently cross capability boundaries** — If the platform's strategy says "HTTP fetch," don't silently open a browser instead. HTTP and browser automation are different capabilities with different costs (speed, resources, user expectations). If all HTTP tools fail, tell the user what happened and let them decide whether to try a browser-based approach.
- **Retry once before escalating** — Transient network errors are common. Retry once with the same tool before switching to a different one.

---

## Platform Extraction Strategies

### Claude (Server-rendered)

Claude share pages render the conversation on the server, so the content is in the initial page HTML.

1. Fetch the share URL via HTTP (any HTTP client or page fetcher your platform provides)
2. If the fetch fails with a network error, retry once — transient failures are common
3. If it still fails, tell the user: "Unable to reach claude.ai. If you are in China, please check your VPN connection and try again."
4. Extract the conversation from the page content — it is typically in markdown format
5. Preserve the markdown structure when formatting the output

### DeepSeek (Client-rendered SPA)

DeepSeek share pages are single-page apps — the conversation data lives in JavaScript state, not in the initial HTML. A simple HTTP fetch **will not work** (see Anti-patterns below).

**Prerequisite:** Browser automation capability. If your platform does not support any form of browser automation, tell the user: "DeepSeek share pages require browser automation. Your current platform does not support this."

**Step 1:** Open the share URL in a browser (via your platform's browser automation tool).

**Step 2:** Wait for the page to fully render — wait for network idle or a visible conversation element.

**Step 3:** Extract the conversation. Two methods are available:

#### Method A: JS full extraction (recommended, most universal)

Use JavaScript to find the scrollable conversation container and read its `innerText`. This gets all messages at once regardless of conversation length — no scrolling needed.

```javascript
// Find the main conversation container (DeepSeek renders into a scrollable div)
const container = document.querySelector('[class*="chat"]') || document.querySelector('[class*="conversation"]');
const text = container ? container.innerText : document.body.innerText;
// Returns all messages at once — no scrolling required
```

**Pros:** Works on any platform with browser JS execution (Playwright, Puppeteer, CDP, etc.). Gets all messages in one call regardless of conversation length.
**Cons:** Raw output includes AI thinking process, search references, and citation markers. Filter these out: keep lines that look like natural dialogue, skip blocks starting with "已思考" / "搜索到" / "浏览", and skip citation numbers.

#### Method B: Accessibility snapshot (cleaner, Chrome DevTools only)

If your platform supports Chrome DevTools Protocol (CDP), use the accessibility tree snapshot. This produces cleaner, better-structured output than Method A.

**Pros:** Cleaner output, less filtering needed.
**Cons:** Only works on CDP-based platforms. For long conversations (>5-6 rounds), scroll down and take additional snapshots, then merge.

#### ⚠️ Anti-patterns — DO NOT use these approaches

| Approach | Why it fails |
|----------|-------------|
| **HTTP fetch alone** | DeepSeek is a client-rendered SPA. HTTP fetch returns only page metadata and an empty `<body>`. No conversation content. |
| **React fiber walking** | DeepSeek uses React, but AI responses are split across dozens of deeply nested component fragments. Walking the fiber tree can find user questions but cannot reliably reconstruct complete AI responses. Too fragile for production use. |
| **Scraping CSS class names** | DeepSeek uses hashed CSS module class names that change with every deploy. Any selector built on class names will break unpredictably. |

### 千问 / Qianwen (Client-rendered SPA)

Same strategy as DeepSeek — client-rendered SPA, HTTP fetch alone returns only metadata.

1. Open the share URL in a browser
2. Wait for the page to fully render
3. Extract using the same two methods documented in the DeepSeek section above (Method A: JS extraction, Method B: accessibility snapshot)

### Kimi (Server-rendered)

Kimi share pages render the conversation on the server, so HTTP fetch works directly — same strategy as Claude.

1. Fetch the share URL via HTTP (any HTTP client or page fetcher your platform provides)
2. Extract the conversation from the page content
3. Format turns as User/Assistant dialogue

**Note:** Kimi share links are often copied together with the platform's auto-generated introduction text like "点击链接查看和 Kimi 的对话". Extract the URL from surrounding text when the user pastes the full message — don't require a bare URL.

---

## Graceful Degradation

- **No browser automation available**: DeepSeek and 千问 links are affected. Claude and Kimi work with HTTP fetch alone.
- **Unrecognized domain**: If the URL doesn't match any known platform, tell the user which platforms are currently supported: Claude, Kimi, DeepSeek, and 千问 (more coming).

---

## 跨 Agent 环境适配说明

本 skill 设计为 **agent-agnostic** — Claude Code、Codex、Cursor、OpenClaw、或任何支持 markdown-based skill 的 agent 都可以使用。不同 agent 平台提供的工具能力不同，以下是通用处理方式：

### 能力映射

每个 agent 平台用自己可用的工具完成以下能力，不强制绑定具体工具名：

| 本 skill 需要的能力 | Claude Code 对应 | 其他平台自行映射 |
|---------------------|-----------------|-----------------|
| HTTP 抓取页面 | WebFetch | 任何 HTTP client / page fetcher |
| 浏览器自动化（DeepSeek / 千问） | chrome-devtools MCP | 任何 browser automation 工具 |

### 通用原则

- **工具名称不写入指令** — 平台提取策略描述的是"做什么"和"需要什么能力"，不写"用什么工具做"
- **能力不存在时明确告知** — 如果当前平台缺少某平台所需的提取能力（如无法做浏览器自动化），直接告诉用户，并列出受影响的平台
- **Skill 路径引用均采用相对本 skill 根目录的形式** — `scripts/normalize.py`，agent 按自身安装位置解析

## Dependency Policy

- Prefer HTTP fetch whenever the page is server-rendered (no external dependency needed on most platforms)
- Browser automation is only needed for client-rendered pages (DeepSeek and 千问)
- Goal: 4 platforms covered, at most 1 special capability required (browser automation)
