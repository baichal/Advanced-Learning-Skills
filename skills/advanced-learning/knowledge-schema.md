# 知识条目结构与写入规范

本文件在知识沉淀时按需读取。所有操作静默执行，不向用户展示任何信息。

## 沉淀原则

沉淀的是信息全貌，不只是"胜出"的结论。
一个好的知识条目应该让未来的调用者理解：这个问题有哪些面、各自的道理是什么、在什么条件下各个结论成立。

## 沉淀判断

### 不沉淀（写入当日 memory 笔记）

满足任一：
- 高度个人化
- 一次性任务中间过程
- 当日强时效信息
- 纯上下文维护
- 特定对话的临时结论

### 沉淀为长期知识

全部满足：
- 通用性（不限于特定个人或项目）
- 经过研究（有来源支撑，不是单纯推测）
- 可复用（未来类似问题可调用）
- 属于：知识型 / 方法型 / 评估型

## 去重与合并

沉淀前先用 memory_search 搜索相关条目，再检查 INDEX.md：

- 完全相同主题已存在：
  - 新研究揭示了旧条目未覆盖的维度 → 补充到旧条目
  - 新研究与旧条目存在分歧 → 在旧条目中补充新的视角和条件
  - 新研究信息更新 → 更新旧条目，保留旧信息作为历史参考
- 部分相关 → 新建条目，related_entries 互相引用
- 无相关 → 新建条目

## 文件位置与命名

位置：`memory/knowledge/{分类}/YYYYMMDD_{主题关键词}.md`

分类：
- 技术/ — 技术对比、选型、开发
- 业务/ — 行业、趋势、商业
- 事实/ — 历史、数据、政策
- 方法论/ — 框架、分析方法、决策模型

命名示例：
- 20241215_Python_vs_Java_后端选型.md
- 20241215_React_性能优化.md
- 20241215_AIGC_版权_法律现状.md

## 条目结构

```yaml
---
id: KB_YYYYMMDD_NNN
created: YYYY-MM-DD HH:mm:ss
updated: YYYY-MM-DD HH:mm:ss
update_count: 0
source_question: ""
type: 共识型 | 多视角型 | 分歧型 | 框架型 | 信息源评估型

# ===== 信息全貌 =====
landscape:
  summary: |
    这个问题的信息全貌概述。
    不是给出单一结论，而是描述全貌的轮廓。

  consensus:
    content: |
      共识内容。
    basis: |
      这些共识基于什么。
    source_independence: true | false

  conditional_conclusions:
    - condition: ""
      conclusion: ""
      reasoning: ""
      supporting_sources: []

  perspectives:
    - angle: ""
      position: ""
      reasoning: ""
      applicable_when: ""
      sources: []

  fundamental_divergences:
    - topic: ""
      positions:
        - stance: ""
          reasoning_chain: ""
          sources: []
        - stance: ""
          reasoning_chain: ""
          sources: []
      fork_point: ""
      divergence_root: ""
      practical_impact: |
        采信不同立场在实践中分别意味着什么。

  uncovered_areas:
    - area: ""
      why_uncovered: ""

# ===== 适用性 =====
applicability:
  best_for:
    - ""
  not_suitable_for:
    - ""
  key_differentiators:
    - question: ""
      if_answer_is: ""
      then_consider: ""

# ===== 信息质量 =====
information_quality:
  overall_confidence: 高 | 中高 | 中 | 中低 | 低
  confidence_reasoning: |
    置信度说明。
    基于来源独立性、证据充分性、推理完整性。
  single_source_claims:
    - claim: ""
      source: ""
      reliability_note: ""

# ===== 来源档案 =====
sources:
  - name: ""
    url: ""
    author_background: ""
    date: YYYY-MM
    contribution: ""
    limitations: ""
    credibility: 高 | 中 | 低

# ===== 时效管理 =====
validity:
  type: 长期有效 | 中期有效 | 短期有效
  reasoning: ""
  expires: YYYY-MM | 无需定期验证 | 事件触发验证
  invalidation_triggers:
    - ""

# ===== 索引 =====
tags: []
one_line_summary: ""
related_entries: []

# ===== 使用与反馈 =====
usage:
  call_count: 0
  last_called: null
  call_log: []

feedback:
  positive: 0
  negative: 0
  details: []

update_history: []
---
```

正文用 Markdown 补充详细分析。

## 条目类型说明

| 类型 | 特征 | landscape 重点填写 |
|------|------|-------------------|
| 共识型 | 各方基本一致 | consensus + conditional_conclusions |
| 多视角型 | 多个角度各自成立 | perspectives + key_differentiators |
| 分歧型 | 存在根本性分歧 | fundamental_divergences（特别是 fork_point 和 divergence_root） |
| 框架型 | 方法论或思考框架 | consensus + conditional_conclusions |
| 信息源评估型 | 对某类信息源的可信度评估 | perspectives + sources |

## 索引更新

每次创建或更新条目后，静默更新 `memory/knowledge/INDEX.md`：
- 新建 → 添加行（one_line_summary 描述全貌而非单一结论）
- 更新 → 修改对应行
- 调用 → call_count +1
```

---

## `skills/advanced-learning/maintenance-procedures.md`

```markdown
# 知识维护方法论

由 HEARTBEAT 或 cron 任务触发时读取。所有操作静默执行。

## 时效性检查

| 状态 | 条件 | 处理 |
|------|------|------|
| 已到期 | 当前日期 ≥ expires | 标记"待验证"，下次相关查询时触发慢通道 |
| 即将到期 | 距 expires < 30 天 | 标记"即将过期"，优先级提升 |
| 严重过期且冷门 | 超期 > 90 天且 call_count = 0 | 置信度降为低，添加警告 |
| 过期但高频 | 已到期且 call_count > 50 | 标记"紧急待验证" |

## 去重与合并

1. 识别：tags 重合 > 70% 或 summary 语义高度相似
2. 对比：信息全貌覆盖范围、来源重叠度、时效、置信度
3. 处理：

| 情况 | 处理 |
|------|------|
| 完全重复 | 保留覆盖更全面的版本，合并 usage，删除重复 |
| 部分重复 | 合并为综合条目，整合双方覆盖的维度 |
| 表面相似实质不同 | 保留两个，互相引用，强化 summary 区分度 |
| 版本演进 | 保留最新为主条目，旧版归档 |

## 归档

满足任一则归档到 `memory/knowledge/archive/`：
- 置信度低 + call_count < 5 + 创建 > 90 天
- 短期有效 + 超期 > 180 天 + call_count < 10
- call_count = 0 + 创建 > 180 天（高置信除外）

操作：移动文件 → 从 INDEX.md 删除 → 保留 30 天可恢复

## 用户反馈处理

高优（negative ≥ 3 或高频条目被纠正）：
- 重新执行慢通道
- 新研究揭示旧条目未覆盖的维度 → 补充全貌
- 新研究与旧条目存在分歧 → 在条目中补充新视角
- 旧条目确实有误 → 更新条目

中优（negative 1-2）：
- 标记待验证，下次相关查询时重新研究

## 索引健康检查

1. 一致性：文件 vs INDEX.md 互查，修复孤儿和幽灵条目
2. 标签覆盖：分析未命中查询，补充同义词
3. 热门排序：按 call_count 排序，Top 50 标记核心知识

## 规模控制

总条目 > 500 时：执行归档 + 合并相似条目 + 评估目录是否需要细分
