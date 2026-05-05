# 腾讯元宝 share example

## Input

```
https://yb.tencent.com/s/xxxxxxxxxxxxxx
```

**Note:** 元宝 share links are often copied together with auto-generated text like "元宝团队成立于哪一年？"点击查看元宝的回答. Extract the URL from surrounding text.

## Output

```
# 元宝团队成立于哪一年？

**User:** 元宝团队成立于哪一年？

**Assistant:** 腾讯"元宝"产品团队于 2023 年在内部立项并组建（具体为2023年12月），其对应的AI助手App则于2024年5月30日正式上线。

如果是问背后的大模型研发力量，腾讯"混元"团队（早期叫"混元助手"）则是在 2023年2月开始集结组建的。

**User:** 腾讯元宝和DeepSeek有何关系？

**Assistant:** 腾讯元宝与 DeepSeek 主要是"应用与模型"的接入合作关系，同时也存在潜在的资本关联：

- **模型接入**：腾讯元宝作为 AI 助手应用，在自研"混元"模型之外，也接入了 DeepSeek 系列模型（如 V3、R1 等）。用户可以在元宝内自由切换，借助腾讯云的算力与生态来使用 DeepSeek 的能力。
- **资本关联**：有接近交易的消息称，腾讯（与阿里）正洽谈参与 DeepSeek 约 18 亿美元的融资（估值约 200 亿美元），不过该交易尚未完全敲定。

两者并非同一家公司，DeepSeek（深度求索）是由幻方量化创始人梁文锋创立的独立 AI 公司。
```
