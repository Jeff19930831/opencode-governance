# Handoff

## Persistent Context

- **项目目的**: 记录 OpenCode + oh-my-openagent (omo) 的配置快照、架构决策和运维手册；configs/ 是 live 配置（`~/.config/opencode/`）的镜像，checkpoint 时同步。
- **位置**: `~/Projects/opencode-governance`，origin = github.com/Jeff19930831/opencode-governance（私有，允许明文 secret，2026-06-03 策略）。

### 常驻问题雷达

| 问题 | 风险 | Runbook |
|------|------|---------|
| 插件/MCP 用 `@latest` 回潮 | 启动重新联网解析甚至重装，曾致 44s+/死锁 | [startup-performance](docs/runbooks/startup-performance.md) |
| omo 的 claude-mcp-loader 隐式导入 `~/.claude.json` 的 MCP | 给 Claude 加慢 MCP 会拖累 opencode | [startup-performance](docs/runbooks/startup-performance.md) |
| 三个 opencode 实例（npm 全局 / Zed / ZCode）共享全局配置 | 常驻进程不重启不加载新配置 | [startup-performance](docs/runbooks/startup-performance.md) |
| 第三方代理 gpt-5.5 配置坑 | server_error / 卡住 | [gpt55-omo-setup](docs/runbooks/gpt55-omo-setup.md) |

## Current Takeover

- **当前目标**: 启动性能优化已完成并验证（44s+ → ~3.5s），项目进入配置看护状态。
- **下一步**: 重启 Zed / ZCode 后验证常驻实例加载新配置（ruflo 不再出现、启动 <5s）；补 omo 升级 runbook。
- **阻塞**: 无。
- **验证命令**:
  ```bash
  opencode debug startup                          # 应 <1500ms
  opencode debug config | grep -c ruflo           # 应为 0
  ls ~/.cache/opencode/packages/                  # 应有 oh-my-openagent@4.8.1/（非空）
  ```

## 当前环境状态

| 组件 | 状态 | 位置 |
|------|------|------|
| OpenCode 1.16.2 (npm 全局) | ✅ | `~/.config/opencode/opencode.json` |
| omo 插件 | ✅ pin `oh-my-openagent@4.8.1` | 缓存 `~/.cache/opencode/packages/oh-my-openagent@4.8.1/` |
| MCP agentmemory | ✅ pin `@0.9.24` → `http://43.133.86.33:3111` | opencode.json / `~/.claude.json` / `~/.agents/mcp.json` |
| MCP ruflo | ❌ 已移除（2026-06-10，曾致 30s 启动超时） | 备份 `~/.claude.json.bak-ruflo-fix-*` |
| MCP codegraph | ✅ | `%APPDATA%\opencode\opencode.jsonc` |

## 模型分配速查

| 用途 | 模型 | Agent |
|------|------|-------|
| 干活 | `zai-coding-plan/glm-5.1` | Sisyphus, Hephaestus, Atlas, Sisyphus-Junior |
| 思考 | `ppchat1/gpt-5.5` | Oracle, Prometheus, Metis, Momus |
| 搜索 | `kimi-for-coding/k2.6` | Librarian, Explore |

详见 [docs/agent-model-mapping.md](docs/agent-model-mapping.md)。

## Index

- [plan.md](./plan.md) — Active / Next
- [progress.md](./progress.md) — 已完成归档（含 2026-06-10 启动优化证据）
- [docs/runbooks/startup-performance.md](docs/runbooks/startup-performance.md) — 启动慢排查/修复/预防
- [docs/runbooks/gpt55-omo-setup.md](docs/runbooks/gpt55-omo-setup.md) — ppchat gpt-5.5 配置
- [docs/runbooks/recover-config.md](docs/runbooks/recover-config.md) — 配置恢复
- [docs/agent-model-mapping.md](docs/agent-model-mapping.md) / [docs/team-mode.md](docs/team-mode.md)

---

> 🔄 **Last Updated**: 2026-06-10
> 📌 **持久化检查**: 修改本文件后请立即 `git commit`
