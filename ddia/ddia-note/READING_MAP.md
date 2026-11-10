# 教材索引与学习顺序

核对日期：2026-11-02。以下描述本地下载快照；路径相对于本文件。用户已于同日确认下述学习顺序。

## 来源、目录与版本

- 来源：[Vonng/ddia](https://github.com/Vonng/ddia)。本地 `../ddia-main/` 未发现 `.git` 元数据，不能由此确定下载对应的提交号。
- [README](../ddia-main/README.md)、[站点配置](../ddia-main/hugo.yaml) 和章节标题共同确认第二版正文的 14 章结构。README 同时含第一版时期的历史说明，不能将其中所有日期和描述都套用到第二版。
- `content/zh/`：第二版简体中文正文，主教材；`content/en/`：对应英文正文，用于核对术语和表述；`content/tw/`：第二版繁体中文正文。三处均有 `ch1.md`～`ch14.md`。
- `content/v1/`、`content/v1_tw/`：第一版简体、繁体中文，各 12 章；不作为本次默认教材，不能与第二版直接按同章号混读。
- `static/fig/`：正文插图。`assets/`、`layouts/`、`data/`、`i18n/`、`bin/` 等主要用于网站展示与维护，学习时不需要构建站点。

## 完整程度与例外

- 中英文正文 14 章均有实质内容、章末总结和参考文献。排除示例标题、提示框标题及参考文献标题的格式差异后，各章小节锚点及顺序一致。
- 英文以标题表示的示例和提示框，在中文中也能找到对应 ID，只是采用 Hugo 短代码或 HTML 锚点。标题数量差异不等于缺译。
- 正文中检查到的 `/fig/`、`/map/` 本地图片引用涉及 117 个不同资源，均存在。此检查不包含所有外部链接，也未逐句审校译文。
- **不能认定为出版终稿的逐字副本**：英文第 1、11、12、13、14 章仍保留 Early Release 提示。
- **序言不是完整的第二版序言**：[中文序言](../ddia-main/content/zh/preface.md) 明示保留第一版译文，仅“本书纲要”按第二版目录维护；[英文序言](../ddia-main/content/en/preface.md) 明示仍为第一版。
- **英文部分导言有版本例外**：`content/en/part-i.md`、`part-ii.md`、`part-iii.md` 均标注来自第一版。中文导言存在适配第二版的修改，不能将这组文件假定为完全对等的第二版中英文。
- `preface.md`、`part-i.md`、`part-ii.md`、`part-iii.md`、`glossary.md`、`indexes.md`、`colophon.md` 中英文文件都存在；存在文件不代表已证实其与出版终稿一致。术语表和索引用于按需查阅。

## 正文章节对应

同一行的中文、英文文件互相对应。进入具体小节时，用相同的 `{#...}` 锚点定位，不能依赖行号相同。

| 章 | 中文 | 英文 |
| --- | --- | --- |
| 1 | [数据系统架构中的权衡](../ddia-main/content/zh/ch1.md) | [Trade-offs in Data Systems Architecture](../ddia-main/content/en/ch1.md) |
| 2 | [定义非功能性需求](../ddia-main/content/zh/ch2.md) | [Defining Nonfunctional Requirements](../ddia-main/content/en/ch2.md) |
| 3 | [数据模型与查询语言](../ddia-main/content/zh/ch3.md) | [Data Models and Query Languages](../ddia-main/content/en/ch3.md) |
| 4 | [存储与检索](../ddia-main/content/zh/ch4.md) | [Storage and Retrieval](../ddia-main/content/en/ch4.md) |
| 5 | [编码与演化](../ddia-main/content/zh/ch5.md) | [Encoding and Evolution](../ddia-main/content/en/ch5.md) |
| 6 | [复制](../ddia-main/content/zh/ch6.md) | [Replication](../ddia-main/content/en/ch6.md) |
| 7 | [分片](../ddia-main/content/zh/ch7.md) | [Sharding](../ddia-main/content/en/ch7.md) |
| 8 | [事务](../ddia-main/content/zh/ch8.md) | [Transactions](../ddia-main/content/en/ch8.md) |
| 9 | [分布式系统的麻烦](../ddia-main/content/zh/ch9.md) | [The Trouble with Distributed Systems](../ddia-main/content/en/ch9.md) |
| 10 | [一致性与共识](../ddia-main/content/zh/ch10.md) | [Consistency and Consensus](../ddia-main/content/en/ch10.md) |
| 11 | [批处理](../ddia-main/content/zh/ch11.md) | [Batch Processing](../ddia-main/content/en/ch11.md) |
| 12 | [流处理](../ddia-main/content/zh/ch12.md) | [Stream Processing](../ddia-main/content/en/ch12.md) |
| 13 | [流式系统的哲学](../ddia-main/content/zh/ch13.md) | [A Philosophy of Streaming Systems](../ddia-main/content/en/ch13.md) |
| 14 | [做正确的事情](../ddia-main/content/zh/ch14.md) | [Doing the Right Thing](../ddia-main/content/en/ch14.md) |

## 建议学习顺序

以第二版原书顺序 1 → 14 为主线，逐小节学习；熟悉的部分通过检验后可缩短，薄弱处停下来推导和验证，不预设完成日期。

1. **第 1～5 章：建立数据系统判断基础。** 从事务型与分析型负载、需求、数据模型、存储，到编码与演化。适合补足从 Java 服务开发走向数据工程所需的概念。
2. **第 6～10 章：建立故障与一致性推理能力。** 学习复制、分片、事务、分布式故障、一致性与共识，为后续评审重复处理、恢复和跨系统正确性打基础。
3. **第 11～13 章：重点学习数据处理与系统集成。** 保留“批处理 → 流处理 → 流式系统的哲学”的顺序；先理解数据流、混洗和衍生结果，再深入时间、状态、容错及端到端正确性。此处按实际相关性联系 Kafka、Flink、Doris 和 BMS。
4. **第 14 章：扩展设计责任的边界。** 讨论数据使用、隐私和系统影响，不把设计质量只归结为吞吐、延迟和一致性。

正文前言和各部分导言按需提供背景，不把保留的第一版内容当作第二版必修单元。工程中的临时问题可以插入，解决后返回原位置。

首个单元：第 1 章“分析型与事务型系统”中的“事务处理与分析的特征”（`sec_introduction_oltp`，中文当前第 71 行、英文第 152 行；行号可能随教材更新变化）。从原书问题出发，讲完一个知识单元后只提出一道理解题并等待回答。
