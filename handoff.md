# Handoff

## 项目概览

**项目名称**: opencode-governance
**目的**: 记录 OpenCode + oh-my-openagent (omo) 的配置、架构决策和运维手册
**创建时间**: 2026-06-03
**位置**: `~/Projects/opencode-governance`

## 快速接手

### 1. 了解当前配置

```bash
# 查看核心配置
cat configs/opencode.json
cat configs/oh-my-openagent.json

# 查看 agent-model 映射
cat docs/agent-model-mapping.md

# 查看 team mode 手册
cat docs/team-mode.md
```

### 2. 当前环境状态

| 组件 | 状态 | 配置位置 |
|------|------|---------|
| OpenCode 核心 | ✅ 运行中 | `~/.config/opencode/opencode.json` |
| omo 插件 | ✅ 已安装 | `~/.config/opencode/oh-my-openagent.json` |
| Team Mode | ✅ 已启用 | `oh-my-openagent.json` 中的 `team_mode` |
| MCP (agentmemory) | ✅ 已连接 | `http://43.133.86.33:3111` |
| MCP (ruflo) | ✅ 已连接 | 通过 `npx ruflo@latest mcp start` |

### 3. 模型分配速查

| 用途 | 模型 | Agent |
|------|------|-------|
| 干活 | `zai-coding-plan/glm-5.1` | Sisyphus, Hephaestus, Atlas, Sisyphus-Junior |
| 思考 | `ppchat1/gpt-5.5` | Oracle, Prometheus, Metis, Momus |
| 搜索 | `kimi-for-coding/k2.6` | Librarian, Explore |
| 备用 | `openai/gpt-5.4-mini-fast` | Fallback |

### 4. 常用命令

```bash
# 验证配置
opencode doctor

# 查看 team 列表
opencode team list

# 刷新 onboarding
opencode refresh-onboarding

# 运行 checkpoint
opencode checkpoint
```

## 重要决策

### 模型选择
- **主模型用 glm-5.1**: 代码能力强，性价比高
- **思考模型用 gpt-5.5**: 逻辑推理强，适合复杂决策
- **搜索模型用 kimi-2.6**: 长上下文，检索精准

### Team Mode 策略
- Lead 只用 Sisyphus (glm-5.1)
- Member 用 category 路由，根据任务类型自动选择模型
- 搜索类任务（Oracle/Librarian/Explore）不在 team 内，通过 `delegate-task` 调用

## 已知问题

_暂无_

## 下一步行动

见 [plan.md](./plan.md)

## 联系

_个人项目，无外部依赖_

---

> 🔄 **Last Updated**: 2026-06-03
> 📌 **持久化检查**: 修改本文件后请立即 `git commit`
