---
name: trash-dev
description: DD Pipeline 主命令 - 初始化项目、检查状态、进入需求分析阶段。MVP 实现 RDD 阶段。
---

# /trash-dev - DD Pipeline 主命令

你是 DD Pipeline 的执行者。当用户输入 `/trash-dev` 时，按以下流程执行。

## 执行流程

### 1. 初始化检查

检查 `docs/` 目录是否存在：

- **不存在** → 执行初始化：
  1. 创建 `docs/rdd/` 目录
  2. 创建 `docs/agents/` 目录
  3. 创建 `docs/task.json`（写入完整初始状态，包含 6 层 Pipeline 所有阶段，均为 not_started）
  4. 创建 `docs/rdd/talk.md`（初始内容：`# 需求分析对话记录\n\n> 此文件记录需求分析的完整对话历史`）
  5. 告知用户: "项目已初始化，开始需求分析"

- **存在** → 读取 `docs/task.json`，进入状态检查

task.json 的初始内容（创建时写入）：
```json
{
  "project": "trash-developer-skill",
  "created_at": "[当前日期 YYYY-MM-DD]",
  "stages": {
    "需求分析层": {
      "rdd_discovery": { "status": "not_started", "depends_on": null, "completed_at": null, "document": "docs/rdd/用户需求.md", "talk_log": "docs/rdd/talk.md" },
      "sdd_story": { "status": "not_started", "depends_on": "rdd_discovery", "completed_at": null, "document": null },
      "bdd_scene": { "status": "not_started", "depends_on": "sdd_story", "completed_at": null, "document": null }
    },
    "业务建模层": {
      "ddd_language": { "status": "not_started", "depends_on": "bdd_scene", "completed_at": null, "document": null },
      "ddd_strategic": { "status": "not_started", "depends_on": "ddd_language", "completed_at": null, "document": null }
    },
    "架构设计层": {
      "add_quality": { "status": "not_started", "depends_on": "ddd_strategic", "completed_at": null, "document": null },
      "sadd_security": { "status": "not_started", "depends_on": "add_quality", "completed_at": null, "document": null },
      "arch_decision": { "status": "not_started", "depends_on": "sadd_security", "completed_at": null, "document": null }
    },
    "详细设计层": {
      "rdd_refine": { "status": "not_started", "depends_on": "arch_decision", "completed_at": null, "document": null },
      "ddd_tactical": { "status": "not_started", "depends_on": "rdd_refine", "completed_at": null, "document": null }
    },
    "实现层": {
      "dbc_contract": { "status": "not_started", "depends_on": "ddd_tactical", "completed_at": null, "document": null },
      "tdd_test": { "status": "not_started", "depends_on": "dbc_contract", "completed_at": null, "document": null },
      "edd_event": { "status": "not_started", "depends_on": "tdd_test", "completed_at": null, "document": null }
    },
    "集成层": {
      "cdd_contract": { "status": "not_started", "depends_on": "edd_event", "completed_at": null, "document": null },
      "edd_driven": { "status": "not_started", "depends_on": "cdd_contract", "completed_at": null, "document": null },
      "sdd_spec": { "status": "not_started", "depends_on": "edd_driven", "completed_at": null, "document": null }
    }
  }
}
```

### 2. 状态检查

读取 `docs/task.json`，检查 `需求分析层.rdd_discovery` 的状态：

- **completed_at != null（已完成）** →
  展示 `docs/rdd/用户需求.md` 的内容，询问用户：
  "已有需求文档。你想：\n1. 修改需求文档\n2. 保持现状进入下一阶段（暂未实现）\n3. 重新开始（输入 /trash-dev-rdd）"

- **completed_at == null 但 用户需求.md 存在（进行中）** →
  读取 `docs/rdd/用户需求.md` 和 `docs/rdd/talk.md` 到 context，告知用户：
  "需求分析进行中。继续上次的话题，还是重新开始？"

- **不存在（全新项目）** → 进入 RDD 流程

### 3. 依赖检查

进入当前阶段前，检查依赖链：
- 遍历 task.json 所有阶段
- 如果某阶段的 `depends_on` 不为 null，且依赖的阶段 `completed_at == null`：
  - 提示: "上一阶段 [{依赖阶段}] 未完成，自动回退"
  - 进入未完成的前置阶段

### 4. RDD 流程

进入需求分析流程，按以下步骤执行：

#### 4.1 Agent 选择

1. 扫描 `docs/agents/` 目录，读取所有 `.md` 文件
2. 提取每个文件的 frontmatter（name, capabilities, required_skills）
3. 向用户展示 Agent 候选列表：

```
可选辅助 Agent（选择 2 个）：

| # | Agent | 能力 | 需加载 Skill |
|---|-------|------|-------------|
| 1 | xxx   | xxx  | xxx         |
| 2 | xxx   | xxx  | xxx         |
...

系统默认推荐: 用户研究员 + 商业策略师
请选择 2 个（直接回复编号），或回复"默认"使用推荐组合。
```

#### 4.2 创建 Agent

用户确认后：
1. 你担任 **主 Agent（需求分析师）**，直接主持对话
2. 用 Agent tool 创建 2 个子代理作为辅助 Agent：

```
Agent(
  description: "辅助分析 - [Agent名称]",
  prompt: "读取 docs/agents/[文件名].md 的完整内容作为你的角色设定。你是一个辅助分析师，职责是：\n1. 监听主 Agent 与用户的对话\n2. 从你的专业角度发现遗漏和盲区\n3. 提出补充问题\n\n当前对话上下文：[附上最近的对话内容]\n\n请分析是否有遗漏，如果有，提出 1-2 个补充问题。",
  subagent_type: "general-purpose"
)
```

#### 4.3 需求收集对话

主持需求收集对话，依次运用以下视角：
1. **产品视角**（product-manager）- 产品愿景、核心问题、功能边界
2. **用户视角**（ux-researcher）- 用户画像、使用场景、痛点
3. **商业视角**（product-strategist）- 商业模式、市场定位、竞品
4. **技术视角**（systems-architect）- 技术可行性、复杂度

每轮对话后：
1. 追加记录到 `docs/rdd/talk.md`（见下方格式要求）
2. 调用 2 个辅助 Agent 进行后台分析
3. 如果辅助 Agent 发现遗漏，补充提问

#### 4.4 结束判定

每轮对话后评估需求完整度。当覆盖率 ≥80% 时，告知用户：

"需求已经基本完整，是否生成需求文档？（也可以继续补充）"

用户确认后，进入步骤 4.5。

#### 4.5 生成需求文档

启动一个独立的子代理来生成需求文档：

```
Agent(
  description: "故事化需求文档生成",
  prompt: "你是一个擅长用故事讲述产品的专家。请基于以下对话记录，提取所有功能点和用户需求，编写成故事形式的需求文档。\n\n要求：\n1. 语言简单，小白也能看懂\n2. 用故事形式讲述（背景与痛点 → 他需要什么 → 理想的样子）\n3. 读者能通过故事了解自己的实际需求\n4. 不要技术术语，用场景化的语言\n\n对话记录：\n[附上 docs/rdd/talk.md 的完整内容]\n\n输出格式：\n# 需求文档\n\n## 故事的起点（背景与痛点）\n...\n\n## 他需要什么（功能需求）\n...\n\n## 理想的样子（成功愿景）\n...",
  subagent_type: "general-purpose"
)
```

将生成结果保存到 `docs/rdd/用户需求.md`。

#### 4.6 用户确认

向用户展示生成的需求文档，询问：

"这是根据我们的对话生成的需求文档。是否满意？\n1. 满意 - 保存并完成\n2. 需要修改 - 告诉我改什么"

- **满意** →
  1. 更新 `docs/task.json`：设置 `rdd_discovery.status = "completed"`，`rdd_discovery.completed_at = "[当前时间 ISO 8601]"`
  2. 提交 git commit
  3. 告知: "需求文档已保存，RDD 阶段完成"

- **不满意** → 返回步骤 4.3 继续对话修改

## talk.md 追加格式

每轮对话追加以下内容到 `docs/rdd/talk.md`：

```markdown
## [YYYY-MM-DD HH:MM:SS] 用户
> [用户输入内容]

## [YYYY-MM-DD HH:MM:SS] 主 Agent (需求分析师)
> [Agent 回复内容]
>
> [调用的 Skill 视角]

## [YYYY-MM-DD HH:MM:SS] 辅助 Agent [名称] - 后台
> [辅助 Agent 分析和建议]
```

## 强制加载的 Skills

需求分析过程中，按需要调用以下 Skills：
- `product-manager` / `agile-product-owner` - 产品需求
- `ux-researcher-designer` - 用户体验
- `systems-architect` - 技术可行性
- `competitive-teardown` - 竞品分析
- `product-strategist` - 市场定位
