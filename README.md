# Context Compactor

> A Claude Code skill that keeps long sessions coherent by rolling old conversation into a single, constantly-updated summary — so the context window stops overflowing and the assistant stops giving degraded or off-topic answers.
>
> 一个 Claude Code skill：把变长的对话滚动压缩进**一份**不断更新的摘要里，让上下文窗口不再溢出，AI 也不再开始遗忘、重复、答非所问。

Inspired by SillyTavern's `/hide` message-management workflow, adapted for Claude Code's continuous-history context model.

灵感来自 SillyTavern 的 `/hide` 楼层管理思路，针对 Claude Code 的连续对话上下文模型做了改造。

---

## English

### The problem

Long sessions degrade. As a conversation grows, the early turns push the model toward its context limit. The symptoms are familiar: the assistant forgets decisions made earlier, repeats suggestions, contradicts itself, or drifts off the actual question.

### The idea

Instead of stacking summary after summary (which just grows forever), keep **exactly one** summary and rewrite it in place each time you compact. Old turns get dropped, the summary stays a roughly constant size, and context usage **converges to a stable band** instead of snowballing.

```
dropped old turns  +  one constant-size summary  +  last N verbatim turns
```

It uses only ordinary abilities — reading, writing, and summarizing — so it changes nothing about Claude Code itself. It's a behavior convention, not a patch.

> **Honest disclaimer:** this is a buffer, not infinity. It extends a session substantially, but it does not make it eternal. For a genuinely fresh start, opening a new session is still the right move.

### Install

Drop the `context-compactor/` folder into your project (or your Claude Code skills directory). Claude reads `SKILL.md` and follows it when a session runs long.

### Use

It triggers on its own when a session gets long or quality slips. You can also ask explicitly:

- "compact the context"
- "summarize what we've done so far"
- "trim the old stuff, you're starting to forget"

See [`SKILL.md`](./SKILL.md) for the full workflow, the keep/drop rules, and the summary format.

---

## 中文

### 解决什么问题

长对话会"变笨"。对话越长，早期的内容越把模型推向上下文上限，症状很典型：AI 开始忘记之前定好的事、重复给同样的建议、自相矛盾，或者答非所问。

### 思路

不要一份摘要叠一份摘要（那只是把雪球越滚越大），而是**始终只保留一份**摘要，每次压缩时就地重写它。旧对话被丢弃，摘要长度基本恒定，上下文占用**收敛在一个稳定区间**，而不是无限累加。

```
丢弃的旧对话  +  一份恒定大小的摘要  +  最近 N 楼原文
```

它只用到最普通的能力——读、写、总结——所以不改动 Claude Code 本身分毫。它是一套**行为约定**，不是补丁。

> **诚实说明：** 这是缓冲，不是无限。它能让一条对话续命很久，但不能让它永生。真要彻底重开，开新 session 才是正解。

### 安装

把 `context-compactor/` 文件夹放进你的项目（或你的 Claude Code skills 目录）。对话变长时，Claude 会读取 `SKILL.md` 并照着做。

### 使用

对话变长或质量下滑时它会自动触发。你也可以直接喊：

- "压缩一下上下文"
- "总结一下我们到目前为止做了啥"
- "把旧的删掉，你开始忘事了"

完整工作流、保留/丢弃规则、摘要格式都在 [`SKILL.md`](./SKILL.md) 里。

---

## Acknowledgements / 致谢

The summary-driven, table-based context management approach was inspired by practices in the SillyTavern community and its `/hide` message-management workflow. This project is an independent reimplementation of that general idea, adapted for Claude Code.

本项目以摘要 + 表格管理上下文的思路，受到 SillyTavern 社区相关实践及其 `/hide` 楼层管理工作流的启发。这是对该通用思路的独立重新实现，并针对 Claude Code 做了改造。

---

## License

[MIT](./LICENSE) © 2026 Yukidoh ([@makisekurisu0827](https://github.com/makisekurisu0827))
