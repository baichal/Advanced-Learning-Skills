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
   ```yaml
   archived: true
   archive_time: {当前日期时间}
   archive_reason: {低价值/过时/冷门}
