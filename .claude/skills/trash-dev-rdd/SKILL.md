---
name: trash-dev-rdd
description: 重新进入 RDD 需求分析阶段。将已完成状态重置为未完成，允许修改需求文档。
---

# /trash-dev-rdd - 重新进入 RDD

当用户输入 `/trash-dev-rdd` 时，按以下流程执行。

## 执行流程

### 1. 读取状态

读取 `docs/task.json`。

### 2. 重置 RDD 状态

将 `rdd_discovery` 阶段重置为未完成：

1. 设置 `需求分析层.rdd_discovery.status = "in_progress"`
2. 设置 `需求分析层.rdd_discovery.completed_at = null`
3. 回退所有依赖 RDD 的后续阶段：
   - 遍历所有阶段，如果 `depends_on` 链上经过 `rdd_discovery`：
     - 设置 `completed_at = null`
     - 设置 `status = "not_started"`
4. 保存更新后的 `docs/task.json`

### 3. 加载上下文

- 如果 `docs/rdd/talk.md` 存在 → 读取到 context
- 如果 `docs/rdd/用户需求.md` 存在 → 读取到 context，告知用户: "已加载已有需求文档，你可以从头开始重新收集需求，或者在已有基础上修改。"

### 4. 进入 RDD 流程

按照 `/trash-dev` 中的 RDD 流程（4.1-4.6 步骤）执行，重新进行需求分析。
