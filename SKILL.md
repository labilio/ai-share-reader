---
name: ai-share-reader
description: >
  Read and extract AI chat conversations from share links. Use this skill whenever
  the user sends a URL from claude.ai/share/, kimi.com/share/, chat.deepseek.com/share/, qianwen.com/share/chat/, chat.z.ai/s/, chatglm.cn/share/, doubao.com/thread/, agent.minimaxi.com/share/, agent.minimax.io/share/, chatgpt.com/share/, gemini.google.com/share/, perplexity.ai/search/, manus.im/share/, anygen.io/task/, poe.com/s/, grok.com/share/, yb.tencent.com/s/, yiyan.baidu.com/share/, or chat.mistral.ai/chat/.
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
| **Group A: Server-Rendered** |
| Claude | `claude.ai/share/` | Server-rendered | HTTP fetch |
| Kimi | `kimi.com/share/` | Server-rendered | HTTP fetch |
| Perplexity | `perplexity.ai/search/` | Server-rendered | HTTP fetch |
| Poe | `poe.com/s/` | Server-rendered | HTTP fetch |
| Grok | `grok.com/share/` | Server-rendered | HTTP fetch |
| 元宝 | `yb.tencent.com/s/` | Server-rendered | HTTP fetch |
| **Group C: Login Overlay** |
| ChatGPT | `chatgpt.com/share/` | Server-rendered behind login overlay | Browser automation |
| Gemini | `gemini.google.com/share/` | Server-rendered behind login overlay | Browser automation |
| **Group B: Client-Rendered SPA** |
| DeepSeek | `chat.deepseek.com/share/` | Client-rendered SPA | Browser automation |
| 千问 | `qianwen.com/share/chat/` | Client-rendered SPA | Browser automation |
| 智谱 Z.ai | `chat.z.ai/s/` | Client-rendered SPA | Browser automation |
| 智谱清言 | `chatglm.cn/share/` | Client-rendered SPA | Browser automation |
| 豆包 | `doubao.com/thread/` | Client-rendered SPA | Browser automation |
| Minimax | `agent.minimaxi.com/share/` 或 `agent.minimax.io/share/` | Client-rendered SPA | Browser automation |
| 文心一言 | `yiyan.baidu.com/share/` | Client-rendered SPA | Browser automation |
| Mistral | `chat.mistral.ai/chat/` | Client-rendered SPA | Browser automation |
| Manus | `manus.im/share/` | Client-rendered SPA | Browser automation |
| AnyGen | `anygen.io/task/` | Client-rendered SPA | Browser automation |

## Flow

1. Parse the URL domain to identify the platform
2. Follow the extraction strategy for that platform (see below)
3. Format the result using the output template above and present to the user

**Extracting URLs from surrounding text** — Many platforms (Kimi, 智谱清言, AnyGen, 元宝, 文心一言) auto-generate intro text when users copy share links, e.g. "点击查看元宝的回答 https://..." or "元宝团队成立于哪一年？点击查看元宝的回答 https://...". When the user pastes text containing a share URL, extract just the URL — don't require a bare URL.

### Retry and fallback rules

When a tool fails, follow these boundaries:

- **Same capability, different tool** — If one HTTP fetcher fails, try another HTTP fetcher your platform provides. For example: WebFetch fails → try WebReader, or curl, or a native HTTP client. This is fine.
- **Do NOT silently cross capability boundaries** — If the platform's strategy says "HTTP fetch," don't silently open a browser instead. HTTP and browser automation are different capabilities with different costs (speed, resources, user expectations). Additionally, some server-rendered platforms (Perplexity) use anti-bot protection (Cloudflare Turnstile) that will block browser access. If all HTTP tools fail, tell the user what happened and let them decide whether to try a browser-based approach.
- **Retry once before escalating** — Transient network errors are common. Retry once with the same tool before switching to a different one.
- **Server-rendered = HTTP fetch only** — Claude, Kimi, Perplexity, Poe, Grok, and 元宝 are server-rendered. Never use browser automation for these six — HTTP fetch is the correct tool, and browser access may trigger anti-bot challenges.

---

## Platform Extraction Strategies

### Group A: Server-Rendered (HTTP Fetch)

These platforms render the conversation on the server. **Use HTTP fetch only — never open in a browser.** Some have anti-bot protection (Cloudflare Turnstile) that will block browser access.

**通用步骤：**
1. Fetch the share URL via HTTP
2. Extract the conversation from the page content
3. Format turns as User/Assistant dialogue

---

**Claude** — `claude.ai/share/`

- Content is typically in markdown format — preserve the markdown structure.
- If fetch fails with a network error, retry once. If it still fails, tell the user: "Unable to reach claude.ai. If you are in China, please check your VPN connection and try again."

**Kimi** — `kimi.com/share/`

- No special handling needed beyond the general steps.

**Perplexity** — `perplexity.ai/search/`

- Cloudflare Turnstile protection — browser access will be blocked. HTTP fetch is the only option.
- Filter out UI elements from output: "sources", "Follow-ups", "Viewing a shared thread".

**Poe** — `poe.com/s/`

- Only `poe.com/s/{id}` links work. `poe.com/invite/{id}` links require login — not extractable.
- **Multi-bot chats:** Poe supports group chats where multiple AI bots respond to the same prompt. Format bot responses with the bot name: `**BotName:** response text`. The `/compare @BotName` syntax may appear in repeated user messages — preserve these to show which bot is addressed.

**Grok** — `grok.com/share/`

- Cloudflare Turnstile protection — browser access will be blocked. HTTP fetch is the only option.
- Page titles follow the format `{title} | Shared Grok Conversation`. Extract just the conversation title.

**元宝 (Tencent Yuanbao)** — `yb.tencent.com/s/`

- No special handling needed beyond the general steps.

---

### Group B: Client-Rendered SPA (Browser Automation)

These platforms render the conversation client-side — HTTP fetch alone returns only metadata. **Browser automation is required.**

**通用步骤：**
1. Open the share URL in a browser
2. Wait for the page to fully render
3. Extract using one of the two methods below

**Extraction methods (defined once here, referenced by all platforms below):**

- **Method A: JS extraction** — `document.querySelector('main')?.innerText || document.body.innerText`. Gets all messages at once regardless of conversation length. Works on any browser automation platform. Downside: raw output may include thinking process, search references, and UI elements — filter these out.
- **Method B: Accessibility snapshot** — Chrome DevTools Protocol only. Cleaner, better-structured output. Downside: for long conversations (>5-6 rounds), need to scroll and take additional snapshots, then merge.

**Filter keywords** (appear in raw JS output on some platforms — skip these blocks):

| 平台 | 思考标签 | 搜索标签 |
|------|---------|---------|
| DeepSeek | "已思考" / "搜索到" / "浏览" | — |
| Minimax (中国版) | "已思考 X.Xs" | "已完成 Web搜索" |
| Minimax (国际版) | "Thinking Process X.XXs" | "Completed Web Search" |
| 豆包 | — | "参考 N 篇资料" |

**⚠️ Anti-patterns — DO NOT use these approaches:**

| Approach | Why it fails |
|----------|-------------|
| **HTTP fetch alone** | These are client-rendered SPAs. HTTP fetch returns only page metadata. No conversation content. |
| **React fiber walking** | AI responses are split across dozens of deeply nested component fragments. Cannot reliably reconstruct complete responses. |
| **Scraping CSS class names** | Most SPAs use hashed CSS module class names that change with every deploy. |

---

**DeepSeek** — `chat.deepseek.com/share/`

- The reference SPA platform. Both Method A and Method B work well.

**千问 (Qwen)** — `qianwen.com/share/chat/`

- Same as DeepSeek. Both methods work.

**智谱 (Z.ai / 智谱清言)** — `chat.z.ai/s/` + `chatglm.cn/share/`

- Two domains, identical strategy. Both methods work.

**豆包 (Doubao)** — `doubao.com/thread/`

- Both methods work. Accessibility snapshot produces clean output.

**Minimax** — `agent.minimaxi.com/share/` (中国版) + `agent.minimax.io/share/` (国际版)

- Two domains, identical strategy. Both methods work.
- URL format: `/share/{id}?chat_type=2`
- Filter thinking and search labels from output (see filter table above).

**文心一言 (Baidu Yiyan)** — `yiyan.baidu.com/share/`

- Content visible without login (shows "未登录" but displays full conversation). Both methods work.

**Mistral (Le Chat)** — `chat.mistral.ai/chat/`

- Content visible without login. May show a Terms of Service dialog on first visit — dismiss it before extracting. Both methods work.

**Manus** — `manus.im/share/`

- **Task replay pages** — the page replays the agent's multi-step execution (web searches, file operations, intermediate thinking), not a simple Q&A. The final output is the key deliverable; intermediate steps can be summarized parenthetically.
- Format: show the user's initial prompt, then the assistant's final deliverable.

**AnyGen** — `anygen.io/task/`

- **Task execution pages** — similar to Manus but simpler: shows the prompt, execution time ("已完成 5s"), and the AI's result. No detailed execution chain.
- URL format: `/task/{task-slug}?share_id={share_id}`
- Accessibility snapshot produces clean output; alternatively use `document.querySelector('main').innerText`.

---

### Group C: Login Overlay (Browser Automation)

These platforms render the conversation in the DOM without requiring authentication, but HTTP fetch fails (may redirect or return only metadata). The login prompt is just an overlay on top of already-rendered content. **Browser automation required.**

**通用步骤：**
1. Open the share URL in a browser
2. Wait for the page to load — conversation content renders quickly even before auth
3. Extract from the DOM

---

**ChatGPT** — `chatgpt.com/share/`

- Accessibility snapshot gives the cleanest output — the conversation is in a `<main>` element, sidebar and login UI are separate.
- Method A (JS `document.body.innerText`) also works.

**Gemini** — `gemini.google.com/share/`

- **Must use Method A (JS extraction):** the accessibility snapshot only shows the login overlay, not the conversation. Use `document.body.innerText`.
- Filter out UI elements like "登录" and navigation text.
- User turns are labeled "你说" in the source — format as `**User:**`.

---

## Graceful Degradation

- **No browser automation available**: DeepSeek, 千问, 智谱, 豆包, Minimax, ChatGPT, Gemini, 文心一言, Mistral, Manus, and AnyGen links are affected. Claude, Kimi, Perplexity, Poe, Grok, and 元宝 work with HTTP fetch alone.
- **Platform has no share feature**: Some platforms (e.g., 硅基流动) do not offer share links at all. Tell the user this platform cannot be supported.
- **Unrecognized domain**: If the URL doesn't match any known platform, tell the user which platforms are currently supported: Claude, Kimi, DeepSeek, 千问, 智谱, 豆包, Minimax, ChatGPT, Gemini, Perplexity, Poe, Grok, 元宝, 文心一言, Mistral, Manus, and AnyGen (more coming).

---

## 跨 Agent 环境适配说明

本 skill 设计为 **agent-agnostic** — Claude Code、Codex、Cursor、OpenClaw、或任何支持 markdown-based skill 的 agent 都可以使用。不同 agent 平台提供的工具能力不同，以下是通用处理方式：

### 能力映射

每个 agent 平台用自己可用的工具完成以下能力，不强制绑定具体工具名：

| 本 skill 需要的能力 | Claude Code 对应 | 其他平台自行映射 |
|---------------------|-----------------|-----------------|
| HTTP 抓取页面 | WebFetch | 任何 HTTP client / page fetcher |
| 浏览器自动化（DeepSeek / 千问 / 智谱 / 豆包 / Minimax / ChatGPT / Gemini / 文心一言 / Mistral / Manus / AnyGen） | chrome-devtools MCP | 任何 browser automation 工具 |

### 通用原则

- **工具名称不写入指令** — 平台提取策略描述的是"做什么"和"需要什么能力"，不写"用什么工具做"
- **能力不存在时明确告知** — 如果当前平台缺少某平台所需的提取能力（如无法做浏览器自动化），直接告诉用户，并列出受影响的平台
- **Skill 路径引用均采用相对本 skill 根目录的形式** — agent 按自身安装位置解析

## Examples

Per-platform input/output samples are in `examples/` — one file per platform showing the expected URL format and output style.

## Dependency Policy

- Prefer HTTP fetch whenever the page is server-rendered (no external dependency needed on most platforms)
- Browser automation is needed for client-rendered SPA pages and login-overlay pages (Groups B and C above)
- Goal: 18 platforms covered (2 domains each for 智谱 and Minimax)
