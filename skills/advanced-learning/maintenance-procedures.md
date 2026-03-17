# 知识维护方法论

由 HEARTBEAT 或 cron 任务触发时读取。所有操作静默执行。

## 时效性检查

扫描 memory/knowledge/ 下所有条目的 validity.expires 字段，按以下规则处理：

| 状态 | 条件 | 处理 |
|------|------|------|
| 已到期 | 当前日期 ≥ expires | 在条目 YAML 头部添加 `status: 待验证`，下次相关查询时触发慢通道重新研究 |
| 即将到期 | 距 expires < 30 天 | 在条目 YAML 头部添加 `status: 即将过期`，优先级提升 |
| 严重过期且冷门 | 超期 > 90 天且 call_count = 0 | 将 overall_confidence 降为低，添加 `warning: 严重过期未验证` |
| 过期但高频 | 已到期且 call_count > 50 | 在条目 YAML 头部添加 `status: 紧急待验证` |

处理完成后将结果汇总写入当日 memory 笔记：
- 已到期条目数及列表
- 即将到期条目数及列表
- 严重过期条目数
- 紧急待验证条目数

## 去重与合并

### 识别疑似重复

扫描所有条目，识别满足以下任一条件的组合：
- tags 重合度 > 70%
- one_line_summary 语义高度相似

### 深度对比

对每组疑似重复执行对比：
- landscape.summary 的覆盖范围是否高度重叠
- 来源是否重叠（sources 中 url 重合度）
- 时效哪个更新（validity.expires 或 updated）
- 置信度哪个更高（overall_confidence）

### 处理规则

| 情况 | 处理 |
|------|------|
| 完全重复（全貌覆盖高度重叠且来源重叠 > 80%） | 保留覆盖更全面的版本，将另一条目的 usage.call_count 合并累加，删除重复条目，更新 INDEX.md |
| 部分重复（有重叠但各有独特内容） | 合并为综合条目，整合双方的 landscape 各维度，删除旧版本，创建合并后版本，更新 INDEX.md |
| 表面相似实质不同（关键词相似但 landscape 核心不同） | 保留两个条目，在各自的 related_entries 字段互相引用，强化 one_line_summary 区分度 |
| 版本演进（同主题不同时期，内容有实质更新） | 保留最新版本为主条目，旧版本移至 memory/knowledge/archive/，更新 INDEX.md |

## 归档

### 归档条件

满足任一则将条目移至 `memory/knowledge/archive/{原分类}/`：

- overall_confidence = 低 且 call_count < 5 且 创建时间 > 90 天前
- validity.type = 短期有效 且 超期 > 180 天 且 call_count < 10
- call_count = 0 且 创建时间 > 180 天前（overall_confidence = 高 的条目豁免）

### 归档操作

1. 移动文件到 `memory/knowledge/archive/{原分类}/`
2. 在文件 YAML 头部添加：
   ~~~yaml
   archived: true
   archive_time: {当前日期时间}
   archive_reason: 低价值 | 过时 | 冷门
   ~~~
3. 从 `memory/knowledge/INDEX.md` 删除对应行
4. 保留 30 天可恢复期，30 天后标记为只读

将归档操作结果写入当日 memory 笔记：
- 归档条目数及列表
- 各归档原因分布

## 用户反馈处理

### 优先级分类

| 优先级 | 条件 |
|--------|------|
| 高 | feedback.negative ≥ 3，或 call_count > 10 的条目收到负面反馈 |
| 中 | feedback.negative 为 1-2 |
| 低 | 仅有正面反馈（统计用途） |

### 高优先级处理

1. 提取用户反馈的具体问题（从 feedback.details 中读取）
2. 对该条目重新执行慢通道研究
3. 对比新旧研究结果：
   - 新研究揭示了旧条目未覆盖的维度 → 补充到 landscape，更新 updated 和 update_count
   - 新研究与旧条目存在分歧 → 在 landscape.fundamental_divergences 补充新视角
   - 旧条目确实有误 → 更新核心内容，降低或重新评估 overall_confidence，在 update_history 记录变更原因
4. 将 feedback.details 中对应条目的 status 改为"已处理"

### 中优先级处理

1. 在条目 YAML 头部添加 `status: 待验证`
2. 下次用户查询该条目相关主题时，优先重新验证
3. 验证完成后按高优先级处理流程更新条目

## 索引健康检查

### 一致性检查

1. 列出 memory/knowledge/ 下所有 .md 文件（不含 INDEX.md、archive/ 目录、.installed 文件）
2. 对比 INDEX.md 中的文件路径列
3. 处理差异：
   - 有文件无索引行（孤儿文件）→ 读取文件的 YAML 头部，提取 id、one_line_summary、tags、overall_confidence、validity.expires、usage.call_count，补充到 INDEX.md
   - 有索引行无文件（幽灵索引）→ 从 INDEX.md 删除该行

### 标签覆盖优化

读取当日及最近 7 天的 memory 笔记，找出标记为"知识缺口"的记录：
- 检查是否有条目内容匹配但 tags 未能命中
- 若有 → 补充同义词、别名或常见提问方式到该条目的 tags 字段，同步更新 INDEX.md 对应行的标签列

### 热门排序

按 usage.call_count 对所有条目降序排列：
- Top 50 在 INDEX.md 中标记为核心知识，供快通道优先检索

## 规模控制

当 INDEX.md 中总条目数超过 500 时：

1. 优先执行归档（按上述归档条件）
2. 归档后仍超过 500 → 扫描疑似重复并执行合并
3. 合并后仍超过 500 → 评估各分类目录的条目数分布：
   - 单一分类超过 150 条 → 在当日 memory 笔记中记录建议细分子目录
   - 等待用户确认后执行目录拆分（目录结构变更不自主执行）
