# Team Mode 手册

> 最后更新: 2026-06-03
> 依赖: `oh-my-openagent` (omo) v4.7.5+

## 启用 Team Mode

在 `~/.config/opencode/oh-my-openagent.jsonc` 中添加:

```jsonc
{
  "team_mode": {
    "enabled": true,
    "max_parallel_members": 4,
    "max_members": 8,
    "tmux_visualization": false
  }
}
```

重启 opencode 后，`team_*` 工具族可用。

---

## Agent  eligibility

只有以下 agent 可以作为 team member:

| Agent | 能否作为 member | 备注 |
|-------|----------------|------|
| **Sisyphus** | ✅ Lead | 主 orchestror，通常作为 lead |
| **Atlas** | ✅ | 执行代理 |
| **Sisyphus-Junior** | ✅ | 轻量执行 |
| **Hephaestus** | ✅ (需权限) | 需 `teammate: "allow"` |
| **Oracle** | ❌ Hard-reject | 无法写入 mailbox，用 `delegate-task` |
| **Librarian** | ❌ Hard-reject | 同上 |
| **Explore** | ❌ Hard-reject | 同上 |
| **Metis** | ❌ Hard-reject | 同上 |
| **Momus** | ❌ Hard-reject | 同上 |
| **Prometheus** | ❌ Hard-reject | 同上 |

---

## 团队定义

### 用户级团队

路径: `~/.omo/teams/{name}/config.json`

示例:

```json
{
  "name": "security-audit",
  "description": "Security research team",
  "lead": { "kind": "subagent_type", "subagent_type": "sisyphus" },
  "members": [
    {
      "kind": "category",
      "name": "hunter-1",
      "category": "deep",
      "prompt": "Focus on authentication vulnerabilities"
    },
    {
      "kind": "category",
      "name": "hunter-2",
      "category": "deep",
      "prompt": "Focus on injection vulnerabilities"
    }
  ]
}
```

### Member kinds

| Kind | 说明 | 必需参数 |
|------|------|---------|
| `subagent_type` | 直接 agent | `subagent_type`: sisyphus/atlas/sisyphus-junior/hephaestus |
| `category` | 通过 sisyphus-junior 路由 | `category`: quick/deep/ultrabrain/visual-engineering/... |

---

## 工具列表

| 工具 | 用途 | 谁可以调用 |
|------|------|-----------|
| `team_create` | 创建团队 | Lead |
| `team_delete` | 解散团队 | Lead (需所有 member 已退出) |
| `team_shutdown_request` | 请求 member 结束 | Lead |
| `team_approve_shutdown` | 批准结束 | Member/Lead |
| `team_reject_shutdown` | 拒绝结束 | Member/Lead |
| `team_send_message` | 点对点消息 | 全员 (Lead 可广播) |
| `team_task_create` | 创建任务 | Lead |
| `team_task_list` | 列出任务 | 全员 |
| `team_task_update` | 更新任务状态 | Member (claim/complete/fail) |
| `team_task_get` | 获取任务详情 | 全员 |
| `team_status` | 查看团队状态 | 全员 |
| `team_list` | 列出所有团队 | 全员 |

---

## 使用模式

### 模式 1: 探索型团队

```
Lead (Sisyphus)
├── scout-1 (category: deep) → 探索模块 A
├── scout-2 (category: deep) → 探索模块 B
└── scout-3 (category: quick) → 检查测试覆盖
```

### 模式 2: 审查型团队

```
Lead (Sisyphus)
├── reviewer-1 (category: unspecified-high) → 安全性审查
├── reviewer-2 (category: unspecified-high) → 性能审查
└── reviewer-3 (category: writing) → 文档审查
```

### 模式 3: hyperplan (内置)

5 个 hostile critic 从不同角度审查计划:
- 技术可行性
- 安全风险
- 维护成本
- 性能影响
- 用户体验

调用: `/hyperplan`

### 模式 4: security-research (内置)

3 个漏洞 hunter + 2 个 PoC 工程师并行审计:

调用: `/security-research`

---

## 边界与限制

| 限制项 | 默认值 | 说明 |
|--------|--------|------|
| 最大成员数 | 8 | 含 lead |
| 最大并行成员 | 4 | 同时运行的 member |
| 单消息体大小 | 32 KB | 超过会被截断 |
| 收件箱未读上限 | 256 KB | 超过会丢弃旧消息 |
| 单次运行最大消息数 | 10,000 | 防止无限循环 |
| 单次运行最大时间 | 120 分钟 | wall clock |
| 单个 member 最大轮数 | 500 | 防止单个 agent 垄断 |

---

## 最佳实践

1. **Lead 只协调不干活**: Lead 负责创建任务、收集团队输出、做最终决策
2. **明确任务边界**: 每个 member 的 `prompt` 要明确 scope，避免重叠
3. **及时清理**: 任务完成后用 `team_shutdown_request` + `team_approve_shutdown` 优雅退出
4. **不用 team 做简单任务**: 单 agent 能搞定的不要开 team，减少 overhead
5. **监控消息量**: 如果 member 开始重复发送相同内容，可能是陷入了循环，及时干预

---

## 故障排查

| 现象 | 可能原因 | 解决 |
|------|---------|------|
| `team_create` 失败 | `team_mode.enabled` 未设置 | 检查配置文件并重启 |
| member 无响应 | 模型 API 超时 | 检查 provider 状态，或调整 fallback |
| 消息丢失 | 超过 32KB 限制 | 分多条发送，或精简 prompt |
| 无法删除团队 | 还有 active member | 先 `team_shutdown_request` 所有 member |

