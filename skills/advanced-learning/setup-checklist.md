# 初始化检查与修复

本文件在技能首次激活时读取。按顺序执行以下全部步骤，缺失项立即创建修复。
全部静默执行。只有在遇到无法自行解决的权限问题时才用自然语言告知用户需要协助。

## 前置判断

读取 `memory/knowledge/.installed` 文件。
- 文件存在且包含 `installed: true` → 初始化已完成，停止阅读本文件，返回 SKILL.md 继续正常流程。
- 文件不存在 → 继续执行以下全部步骤。

---

## 步骤 1：创建目录结构

检查以下目录是否存在，不存在则创建：

```
memory/knowledge/
memory/knowledge/技术/
memory/knowledge/业务/
memory/knowledge/事实/
memory/knowledge/方法论/
memory/knowledge/archive/
```

检查方式：尝试列出目录内容。列出失败则创建该目录。

---

## 步骤 2：创建索引文件

检查 `memory/knowledge/INDEX.md` 是否存在且内容完整。

- 不存在或为空 → 创建文件，写入以下内容（完整复制，`{当前日期时间}` 替换为实际时间）：

~~~markdown
# 知识库索引

> 最后更新：{当前日期时间}
> 总条目数：0

| 条目ID | 信息全貌概述 | 标签 | 置信度 | 时效 | 调用次数 | 文件路径 |
|--------|------------|------|--------|------|---------|---------|

## 待验证队列

暂无

## 知识缺口

暂无
~~~

- 存在但缺少表头或结构损坏 → 保留已有条目数据行，修复表头和分区结构。

---

## 步骤 3：检查技能辅助文件

检查以下文件是否可读取：

```
skills/advanced-learning/research-methodology.md
skills/advanced-learning/knowledge-schema.md
skills/advanced-learning/maintenance-procedures.md
```

检查方式：尝试读取每个文件的前几行。

- 全部可读 → 通过。
- 某个不可读 → 记录缺失文件名到当日 memory 笔记，标记为"技能文件缺失"。缺失文件对应的功能降级：
  - research-methodology.md 缺失 → 慢通道使用 SKILL.md 中的简要指引，不进行精细来源评估
  - knowledge-schema.md 缺失 → 沉淀时使用简化结构（仅 id、summary、tags、confidence、正文）
  - maintenance-procedures.md 缺失 → 跳过维护任务

不因辅助文件缺失中止安装流程，继续执行后续步骤。

---

## 步骤 4：注册定时任务

检查以下 4 个定时任务是否已注册。检查方式：读取 cron 任务列表，匹配任务名称。

全部存在 → 通过。
部分或全部缺失 → 逐个注册缺失的任务，定义如下：

**任务 1：**
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

**任务 2：**
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

**任务 3：**
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

**任务 4：**
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

注册失败（无权限或 cron 服务不可用）→ 记录到当日 memory 笔记。维护任务将依赖 HEARTBEAT.md 的心跳检查作为降级方案。不因注册失败中止安装流程。

---

## 步骤 5：配置 AGENTS.md

读取 workspace 根目录的 AGENTS.md，搜索关键词 "Advanced Learning Skills"。

- 找到 → 通过，跳到步骤 6。
- 未找到但文件存在 → 在文件末尾追加下方内容。
- 文件不存在 → 创建文件，写入下方内容。

要写入的内容（完整复制）：

~~~markdown

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
~~~

---

## 步骤 6：配置 HEARTBEAT.md

读取 workspace 根目录的 HEARTBEAT.md，搜索关键词 "Advanced Learning Skills"。

- 找到 → 通过，跳到步骤 7。
- 未找到但文件存在 → 在文件末尾追加下方内容。
- 文件不存在 → 创建文件，写入下方内容。

要写入的内容（完整复制）：

~~~markdown

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
~~~

---

## 步骤 7：写入安装标记

创建 `memory/knowledge/.installed`，写入：

```yaml
installed: true
time: {当前日期时间}
version: 1.0.0
```

---

## 步骤 8：验证安装

逐项验证：

1. 列出 `memory/knowledge/` 目录 → 确认 技术/、业务/、事实/、方法论/、archive/ 存在
2. 读取 `memory/knowledge/INDEX.md` 前 3 行 → 确认包含"知识库索引"
3. 读取 `AGENTS.md` → 确认包含 "Advanced Learning Skills"
4. 读取 `HEARTBEAT.md` → 确认包含 "Advanced Learning Skills"
5. 列出 cron 任务 → 确认 4 个任务名称存在
6. 读取 `memory/knowledge/.installed` → 确认包含 `installed: true`

每项标记 ✓ 或 ✗。

**全部 ✓：**
在当日 memory 笔记写入：
```
[Advanced Learning Skills] 初始化完成 {当前日期时间}
全部 6 项检查通过。
```
返回 SKILL.md 继续处理用户原始问题。

**存在 ✗：**
对失败项重试一次。仍失败则：

1. 在当日 memory 笔记记录具体失败项
2. 按以下降级表继续运行，不完全中止：

| 失败项 | 降级行为 |
|--------|---------|
| 目录创建失败 | 沉淀功能关闭，仅写入当日 memory 笔记。研究和回答功能正常。 |
| INDEX.md 创建失败 | 同上 |
| 辅助文件缺失 | 对应功能使用 SKILL.md 简要指引替代 |
| cron 注册失败 | 无定时维护，依赖 HEARTBEAT.md 心跳 |
| AGENTS.md 写入失败 | 不自动路由，需用户手动触发技能 |
| HEARTBEAT.md 写入失败 | 无心跳维护，依赖 cron |
| cron 和 HEARTBEAT 都失败 | 无自动维护，需用户手动触发 |

3. 用自然语言告知用户有部分配置未完成，说明哪些功能可能受限。不暴露内部文件路径和结构细节。

---

## 卸载流程

当用户明确要求卸载、移除或禁用 Advanced Learning Skills 时执行。
按顺序执行以下全部步骤。每步执行前确认，每步执行后验证。

### 卸载步骤 1：移除定时任务

删除以下 4 个 cron 任务（按名称匹配）：

```
知识时效性全量检查
知识去重与索引优化
知识归档
用户反馈处理
```

验证：列出 cron 任务列表，确认以上 4 个名称不存在。

### 卸载步骤 2：清理 HEARTBEAT.md

读取 workspace 根目录的 HEARTBEAT.md。
删除从 `# 心跳任务（Advanced Learning Skills）` 开始到文件末尾或下一个同级 `---` 分隔线之前的全部内容，包括该标题上方紧邻的 `---` 分隔线。

- 删除后文件内仍有其他内容 → 保留文件。
- 删除后文件为空或只剩空白 → 删除整个文件。
- 文件不存在 → 跳过。

### 卸载步骤 3：清理 AGENTS.md

读取 workspace 根目录的 AGENTS.md。
删除从 `# 行为规范补充（Advanced Learning Skills）` 开始到文件末尾或下一个同级 `---` 分隔线之前的全部内容，包括该标题上方紧邻的 `---` 分隔线。

- 删除后文件内仍有其他内容 → 保留文件。
- 删除后文件为空或只剩空白 → 删除整个文件。
- 文件不存在 → 跳过。

### 卸载步骤 4：处理知识库数据

询问用户：

> 知识库中积累的知识条目要一起删除吗？还是保留下来？

**用户选择删除：**
删除 `memory/knowledge/` 目录及其下全部内容，包括：

```
memory/knowledge/.installed
memory/knowledge/INDEX.md
memory/knowledge/技术/          ← 及其下所有 .md 文件
memory/knowledge/业务/          ← 及其下所有 .md 文件
memory/knowledge/事实/          ← 及其下所有 .md 文件
memory/knowledge/方法论/        ← 及其下所有 .md 文件
memory/knowledge/archive/       ← 及其下所有 .md 文件
```

**用户选择保留：**
仅删除安装标记文件 `memory/knowledge/.installed`。
知识条目、INDEX.md、目录结构全部保留。
在当日 memory 笔记中记录：知识库数据已保留在 memory/knowledge/，但管理技能已卸载，数据不再自动维护。

**用户未明确回复：**
默认保留数据，按"用户选择保留"处理。

### 卸载步骤 5：删除技能文件

删除 `skills/advanced-learning/` 目录及其下全部文件：

```
skills/advanced-learning/SKILL.md
skills/advanced-learning/setup-checklist.md
skills/advanced-learning/research-methodology.md
skills/advanced-learning/knowledge-schema.md
skills/advanced-learning/maintenance-procedures.md
```

### 卸载步骤 6：验证卸载

逐项验证：

1. 列出 cron 任务 → 确认 4 个任务不存在
2. 读取 AGENTS.md → 确认不包含 "Advanced Learning Skills"
3. 读取 HEARTBEAT.md → 确认不包含 "Advanced Learning Skills"
4. 列出 skills/ 目录 → 确认 advanced-learning/ 不存在
5. 如果用户选择删除数据 → 确认 memory/knowledge/ 不存在
6. 如果用户选择保留数据 → 确认 memory/knowledge/.installed 不存在

**全部通过：**
在当日 memory 笔记写入：

```
[Advanced Learning Skills] 已卸载 {当前日期时间}
知识库数据：{已删除/已保留}
```

用自然语言告知用户卸载完成。

**存在失败项：**
告知用户哪些项未能自动清理，用自然语言描述需要手动处理的内容，不暴露具体文件路径。
