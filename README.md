# trash-developer-skill

DD Pipeline - 多 Agent 协作的需求驱动开发管线系统。

## 命令

| 命令 | 说明 |
|------|------|
| `/trash-dev` | 主命令：初始化项目，进入需求分析阶段 |
| `/trash-dev-rdd` | 重新进入需求分析阶段 |

## 架构

基于 DD Pipeline 2.0.0 架构，MVP 实现 RDD（需求驱动开发）阶段。

完整 Pipeline 6 层：
- 需求分析层：RDD → SDD → BDD
- 业务建模层：DDD 统一语言 → DDD 战略设计
- 架构设计层：ADD → SADD → 架构决策
- 详细设计层：RDD 精化 → DDD 战术设计
- 实现层：DbC → TDD → EDD
- 集成层：CDD-Contract → EDD 驱动 → SDD-Specification
