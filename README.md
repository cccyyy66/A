# dsh-novel-kit

给 **DeepSeek Harness** 用的小说精简工具。

把一本几十万字的 txt 压成「每章一条摘要 + 一句原句」，Agent 聊天时只取命中的那一两条，不用把整本书塞进上下文；而且带**剧透闸**——超过你读到的那一章，它一个字都不说。

## 它解决什么

长篇小说看到 400 章，前面 100 章的人名、伏笔早忘了。两条常见的死路：

- 全本塞进上下文：贵，而且一次只能塞几十万字里的一小块
- 让 AI 自己读：上下文一满就断片，还会编

这套东西走第三条：**先把书压成结构化的骨架，再按需取**。

- 每章一条摘要（约 300 字）：摘要 + 三条关键点 + 最抓人处 + **一句逐字原句**
- 原句是防编造的硬锚点：所有摘要里的引文都能回原文一字对上，对不上就是编的
- 原文永远在本地，只有你要细节时才读一段

## 需要

- DeepSeek Harness 0.2.x
- Node ≥ 18（要带 GBK 解码；官方发行版都带）

## 安装

```bash
git clone <你的仓库地址> dsh-novel-kit
cd dsh-novel-kit
npm install
```

然后在你 DSH profile 的 `cordis.patch.yml` 里加两段 MCP（照 `install/patch.example.yml` 抄，路径换成你自己的），重启 DSH。

装好后你会多出两组工具：

| 组 | 工具 | 给谁用 |
|---|---|---|
| `novel`（只读） | `novel_books` `novel_status` `novel_chapter` `novel_search` `novel_raw` | 所有会话，包括 QQ 之类的仿真会话 |
| `novel-kit`（写入） | `novel_prepare` `novel_check` `novel_merge` `novel_watermark` `novel_rule` | 只给能干活的会话 |

分开的原因：只读那组可以放心给聊天机器人，它碰不到也写不了文件。

## 用法

第一次精简一本书：

1. `novel_prepare({ source: "D:/books/某本书.txt", book: "某本书" })`
   → 自动识别 GBK/UTF-8，切成 `chapters/ch0001.txt` 这样的单章文件
2. 分批让 Agent 写摘要（每批 20~25 章，可并行），存到 `summary/part01.json`
3. `novel_check({ book: "某本书" })` → 必须「没中 0」才算过
4. `novel_merge({ book: "某本书" })` → 生成一份完整大纲 Markdown

之后聊天：

- `novel_books` 看有哪些书
- `novel_chapter({ book, chapter })` 问某一章
- `novel_search({ book, keyword })` 按人名/地名搜
- 看到新进度就 `novel_watermark({ book, readThrough })`

新会话只要调一次 `novel_rule()`，它会把整套流程和硬要求复述一遍。

## 数据放在哪

默认 `~/novels/<书名>/`，可以用环境变量 `NOVEL_HOME` 改。

```
~/novels/某本书/
  progress.json     水位与覆盖范围
  source.json       源文件与切章记录
  chapters/         ch0001.txt …
  summary/          part01.json …
  大纲_合并.md
```

## 规则细节

见 [docs/RULES.md](docs/RULES.md)。摘要怎么写、原句为什么要逐字校验、剧透闸怎么设，都在里面。

## 许可

MIT，能改、能二次分发，保留版权声明就行。
