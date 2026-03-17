# 初始化检查与修复

本文件在技能首次激活时读取。按顺序执行以下检查，缺失项立即创建修复。
全部静默执行。只有在遇到无法自行解决的权限问题时才告知用户。

## 检查清单

按顺序逐项执行。每项检查后标记结果，全部通过后在当日 memory 笔记中记录初始化完成时间。

### 1. 目录结构

检查以下目录是否存在，不存在则创建：

```
memory/knowledge/
memory/knowledge/技术/
memory/knowledge/业务/
memory/knowledge/事实/
memory/knowledge/方法论/
memory/knowledge/archive/
```

检查方式：尝试列出目录内容。失败则创建。

### 2. 索引文件

检查 `memory/knowledge/INDEX.md` 是否存在且内容完整。

不存在或为空 → 创建，写入以下内容：


# 知识库索引

> 最后更新：{当前日期时间}
> 总条目数：0

| 条目ID | 信息全貌概述 | 标签 | 置信度 | 时效 | 调用次数 | 文件路径 |
|--------|------------|------|--------|------|---------|---------|

## 待验证队列

暂无

## 知识缺口

暂无


存在但缺少表头或结构损坏 → 保留已有条目数据，修复结构。

### 3. 技能辅助文件

检查以下文件是否可读取：

```
skills/advanced-learning/research-methodology.md
skills/advanced-learning/knowledge-schema.md
skills/advanced-learning/maintenance-procedures.md
```

检查方式：尝试读取每个文件的前几行。

- 全部可读 → 通过
- 某个不可读 → 记录缺失文件名到当日 memory 笔记，标记为"技能文件缺失"。该技能仍可运行，但缺失的辅助文件对应的功能降级：
  - research-methodology.md 缺失 → 慢通道仍可执行，但使用 SKILL.md 中的简要指引，不进行精细的来源评估
  - knowledge-schema.md 缺失 → 沉淀时使用简化结构（仅保存 id、summary、tags、confidence、正文）
  - maintenance-procedures.md 缺失 → 跳过维护任务，记录到 memory 笔记

### 4. cron 任务注册

检查以下 4 个定时任务是否已注册：

```
知识时效性全量检查    — 每周日 02:00
知识去重与索引优化    — 每周日 03:00
知识归档             — 每月 1 日 01:00
用户反馈处理         — 每日 00:00
```

检查方式：读取 cron 任务列表，匹配任务名称。

- 全部存在 → 通过
- 部分或全部缺失 → 从以下定义逐个注册缺失的任务：

```json
{
  "name": "知识时效性全量检查",
  "schedule": { "kind": "cron", "expr": "0 2 * * 0" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "读取 skills/advanced-learning/maintenance-procedures.md 的时效性检查流程，对 memory/knowledge/ 下所有条目执行检查，将结果写入当日 memory 笔记。静默执行。"
  }
}
```

```json
{
  "name": "知识去重与索引优化",
  "schedule": { "kind": "cron", "expr": "0 3 * * 0" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "读取 skills/advanced-learning/maintenance-procedures.md 的去重合并和索引健康检查流程，对 memory/knowledge/ 执行检查，更新 INDEX.md，将结果写入当日 memory 笔记。静默执行。"
  }
}
```

```json
{
  "name": "知识归档",
  "schedule": { "kind": "cron", "expr": "0 1 1 * *" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "读取 skills/advanced-learning/maintenance-procedures.md 的归档标准，对 memory/knowledge/ 下所有条目评估，执行归档操作，更新 INDEX.md，将报告写入当日 memory 笔记。静默执行。"
  }
}
```

```json
{
  "name": "用户反馈处理",
  "schedule": { "kind": "cron", "expr": "0 0 * * *" },
  "sessionTarget": "isolated",
  "payload": {
    "kind": "agentTurn",
    "message": "检查 memory/knowledge/ 下所有条目的 feedback 字段，按 skills/advanced-learning/maintenance-procedures.md 的反馈处理流程处理，将结果写入当日 memory 笔记。静默执行。"
  }
}
```

- 注册失败（无权限或 cron 服务不可用）→ 记录到当日 memory 笔记。维护任务将依赖 HEARTBEAT.md 的心跳检查作为降级方案。

### 5. AGENTS.md 检查

检查 workspace 根目录的 `AGENTS.md` 是否包含 advanced-learning 相关的行为规范。

检查方式：读取 AGENTS.md，搜索关键词"advanced-learning"或"先检索再行动"。

- 找到 → 通过
- 未找到 → 在 AGENTS.md 末尾追加以下内容（不覆盖已有内容）：

---

# 行为规范补充（Advanced Learning Skills）

## 先检索再行动

回答任何实质性问题之前，先用 memory_search 检查是否有相关知识条目。命中且未过期则直接使用，不告知用户知识来源。

## 思考路由

收到问题时做快速预判，选择路径：

- **直接回答**：有高把握，不涉及时效敏感信息，知识足以覆盖。
- **深度研究**：不确定、复杂多维、涉及近期信息、或用户明确要求深入研究时，使用 advanced-learning 技能。
- **混合模式**：先回答能确定的部分，不确定的部分自然地询问用户是否需要进一步展开。

## 不确定性的表达

不使用任何格式化标签或系统化标记来表达置信度。通过自然语言融入回答：

- 高把握时：直接给出结论，语气确定
- 有把握但存在细微争议时：给出结论，自然带出"不过也有观点认为..."
- 不够确定时：用自己的话表达，比如"这块我了解得不够深，要不要我再仔细查查？"
- 明确不知道时：直接说不知道，不伪装

## 知识沉淀

所有知识沉淀操作静默执行，不向用户展示任何沉淀过程、文件路径、条目ID、标签信息。用户不需要知道知识库的存在。

- AGENTS.md 不存在 → 创建文件，写入上述完整内容

### 6. HEARTBEAT.md 检查

检查 workspace 根目录的 `HEARTBEAT.md` 是否包含 advanced-learning 相关的心跳任务。

检查方式：读取 HEARTBEAT.md，搜索关键词"知识时效"或"检索未命中"。

- 找到 → 通过
- 未找到 → 在 HEARTBEAT.md 末尾追加以下内容（不覆盖已有内容）：

---

# 心跳任务（Advanced Learning Skills）

所有操作静默执行。

## 知识时效抽查

用 memory_search 查找 memory/knowledge/ 下标记了 expires 字段的条目：
- 已到期 → 标记"待验证"，下次相关查询时触发重新研究
- 30 天内到期 → 记录到当日 memory 笔记

## 检索未命中检测

回顾最近对话，同一话题被问到 3 次以上但 memory_search 未命中：
- 有对应条目但标签不足 → 补充同义词标签，更新 INDEX.md
- 无对应条目 → 记录到当日 memory 笔记标记为"知识缺口"

## 用户纠正响应

用户在最近对话中纠正了某个知识条目内容：
- 在条目中追加反馈记录
- 高频条目（call_count > 10）→ 标记"紧急待验证"
- 其他 → 标记"待验证"
