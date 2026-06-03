# opencode-governance

记录 OpenCode 和 oh-my-openagent (omo) 的配置、架构决策和运维手册。

## 项目目的

- **配置即代码**: 将 opencode.json 和 oh-my-openagent.json 的版本变更纳入版本控制
- **架构决策记录 (ADR)**: 记录每个模型选择、provider 配置的技术理由
- **团队模式手册**: Team Mode 的 agent 模型映射、团队定义、使用规范
- **故障恢复**: 配置漂移、secret 丢失时的恢复步骤

## 文档结构

```
opencode-governance/
├── README.md                 # 本文件
├── configs/                  # 配置文件快照
│   ├── opencode.json         # OpenCode 核心配置
│   └── oh-my-openagent.json  # omo 插件配置
├── docs/
│   ├── agent-model-mapping.md    # Agent ↔ Model 映射表
│   ├── team-mode.md              # Team Mode 手册
│   └── runbooks/                 # 运维手册
│       └── recover-config.md
├── plan.md                   # 活跃/待办计划
├── handoff.md                # 接手索引
└── archive/                  # 历史决策

```

## 快速参考

| 组件 | 配置文件路径 |
|------|-------------|
| OpenCode 核心 | `~/.config/opencode/opencode.json` |
| omo 插件 | `~/.config/opencode/oh-my-openagent.json` |
| Team 定义 | `~/.omo/teams/{name}/config.json` |

## 当前配置摘要

- **主模型 (干活)**: `zai-coding-plan/glm-5.1`
- **思考模型 (规划/架构)**: `ppchat1/gpt-5.5`
- **搜索模型 (代码/文档检索)**: `kimi-for-coding/k2.6`
- **备用模型**: `openai/gpt-5.4-mini-fast`

---

> ⚠️ **Doc Lifecycle Gate**: 修改配置前请先在本文档更新对应章节，再修改实际配置文件。
