# Agent ↔ Model 映射表

> 最后更新: 2026-06-04
> 配置文件: `configs/oh-my-openagent.json`

## 设计原则

| 用途 | 模型选择 | 理由 |
|------|---------|------|
| **干活** (编码/执行/实现) | `zai-coding-plan/glm-5.1` | 代码能力强，性价比高 |
| **思考** (规划/架构/审查) | `ppchat1/gpt-5.5` | 逻辑推理强，适合复杂决策 |
| **搜索** (代码/文档检索) | `kimi-for-coding/k2.6` | 长上下文，检索精准 |
| **备用** (降级/fallback) | `openai/gpt-5.4-mini-fast` | 轻量、快速、稳定 |

---

## Agent 详细映射

### 可参与 Team Mode 的 Agent (Eligible)

| Agent | 模型 | Variant | Fallback | 角色 |
|-------|------|---------|----------|------|
| **Sisyphus** | `zai-coding-plan/glm-5.1` | - | `ppchat1/gpt-5.5` (high), `zai-coding-plan/glm-5` | 主 orchestror，统筹调度 |
| **Atlas** | `zai-coding-plan/glm-5.1` | medium | - | 执行代理 |
| **Sisyphus-Junior** | `zai-coding-plan/glm-5.1` | medium | - | 轻量执行代理 |
| **Hephaestus** | `zai-coding-plan/glm-5.1` | medium | - | 深度自主执行 |

### 思考/规划类 Agent (Team Mode 外使用)

| Agent | 模型 | Variant | Fallback | 角色 |
|-------|------|---------|----------|------|
| **Oracle** | `ppchat1/gpt-5.5` | high | `google/gemini-3.1-pro-preview` (high) | 架构咨询、复杂调试 |
| **Prometheus** | `ppchat1/gpt-5.5` | xhigh | `google/gemini-3.1-pro-preview` | 战略规划、任务拆解 |
| **Metis** | `ppchat1/gpt-5.5` | high | `kimi-for-coding/k2p5` | 计划咨询、需求分析 |
| **Momus** | `ppchat1/gpt-5.5` | xhigh | `google/gemini-3.1-pro-preview` (high) | 计划审查、质量把关 |

### 搜索/检索类 Agent

| Agent | 模型 | Variant | Fallback | 角色 |
|-------|------|---------|----------|------|
| **Librarian** | `kimi-for-coding/k2.6` | - | `openai/gpt-5.4-mini-fast` | 文档/代码搜索 |
| **Explore** | `kimi-for-coding/k2.6` | - | `openai/gpt-5.4-mini-fast` | 快速代码搜索 |

### 特殊用途 Agent

| Agent | 模型 | Variant | Fallback | 角色 |
|-------|------|---------|----------|------|
| **Multimodal-Looker** | `zai-coding-plan/glm-4.6v` | medium | `zai-coding-plan/glm-4.6v`, `openai/gpt-5-nano` | 图片/PDF 分析 |

---

## Category 路由映射

通过 `task(category="...")` 调用时，Sisyphus-Junior 会自动选择对应模型：

| Category | 默认模型 | Variant | Fallback | 用途 |
|----------|---------|---------|----------|------|
| `quick` | `zai-coding-plan/glm-4.7` | - | `google/gemini-3-flash-preview` | 单文件修改、打字错误 |
| `deep` | `zai-coding-plan/glm-5.1` | medium | `google/gemini-3.1-pro-preview` (high) | 自主研究 + 执行 |
| `ultrabrain` | `zai-coding-plan/glm-5.1` | xhigh | `google/gemini-3.1-pro-preview` (high) | 复杂逻辑、架构决策 |
| `unspecified-low` | `zai-coding-plan/glm-5.1` | medium | `google/gemini-3-flash-preview` | 中等复杂度任务 |
| `unspecified-high` | `zai-coding-plan/glm-5.1` | medium | `google/gemini-3-flash-preview` | 复杂任务 |
| `visual-engineering` | `zai-coding-plan/glm-5` | high | `zai-coding-plan/glm-5`, `kimi-for-coding/k2p5` | 前端、UI/UX、设计 |
| `writing` | `zai-coding-plan/glm-4.7` | - | - | 文档、写作 |
| `artistry` | `zai-coding-plan/glm-4.7` | high | `openai/gpt-5.5` | 创意任务 |

> ⚠️ **注意**: Category 模型目前大部分仍为默认配置（gpt-5-nano），建议逐步调整为 glm-5.1 以保持一致性。

---

## Provider 配置

### ppchat1 (思考模型)

```json
{
  "npm": "@ai-sdk/openai",
  "options": {
    "apiKey": "sk-...",
    "baseURL": "https://code.ddsst.online/v1",
    "setCacheKey": true
  }
}
```

### zai-coding-plan (干活模型)

```json
{
  "npm": "@ai-sdk/openai-compatible",
  "options": {
    "apiKey": "a3c2c19d9b0e44b0b9d26711cc674d71.fNVMPGXx01grOzFz",
    "baseURL": "https://open.bigmodel.cn/api/coding/paas/v4"
  }
}
```

### opencode-go / kimi-for-coding (搜索模型)

通过 OpenCode 内置 provider 路由，无需额外配置。

---

## 变更日志

| 日期 | 变更 | 理由 |
|------|------|------|
| 2026-06-03 | Sisyphus 从 `glm-5` 改为 `glm-5.1` | 升级主模型 |
| 2026-06-03 | 思考类 agent 统一用 `ppchat1/gpt-5.5` | 逻辑推理更强 |
| 2026-06-03 | Explore/Librarian 改为 `kimi-for-coding/k2.6` | 长上下文搜索 |
| 2026-06-04 | Prometheus variant 改为 `xhigh` | 战略规划需要更高质量 |
| 2026-06-04 | `deep`/`ultrabrain`/`unspecified-high`/`unspecified-low` 改为 `glm-5.1` | 大型项目编码需要最强模型 |
| 2026-06-04 | 其余 category 从 `gpt-5-nano` 改为 `glm-4.7` | 提升质量，移除 opencode 默认模型 |

