# Progress

## 2026-06

### 2026-06-10: 启动性能优化 + 首次完整 checkpoint

- 完成: 排查 opencode 启动 44s+/卡死，定位三根因（ruflo MCP 经 claude-mcp-loader 隐式导入超时 30s、插件 `@latest` 每次重装且缓存损坏死锁、agentmemory npx 未固定版本）
- 完成: 修复并验证 — `opencode debug config` 从卡死 10 分钟+ 降到 ~3.5s；合并配置中 ruflo 归零
- 完成: [RUNBOOK] `docs/runbooks/startup-performance.md`（根因、修复、诊断命令、本机三实例拓扑、预防规则）
- 完成: configs/ 快照同步至当前 live 配置（插件 pin `oh-my-openagent@4.8.1`、agentmemory pin `@0.9.24`）
- 完成: handoff.md 编写 + 首次 checkpoint（plan 2026-06 初始化条目全部闭环）
- 证据: `~/.local/share/opencode/log/2026-06-10T000534.log`（44s 慢启动现场）、`2026-06-10T005525.log`（修复后 1.7s 插件加载）；备份 `~/.claude.json.bak-ruflo-fix-*`

### 2026-06-06: gpt-5.5 on ppchat 跑通（历史，见 git log）

- 完成: Responses API + disable ruflo 的 omo gpt-5.5 工作配置（commit c6fbe02）
- 完成: [RUNBOOK] `docs/runbooks/gpt55-omo-setup.md`

### 2026-06-03/04: 项目初始化

- 完成: 创建项目目录和 git 仓库
- 完成: 复制配置文件快照 (opencode.json, oh-my-openagent.json)
- 完成: README + 文档结构、agent-model-mapping.md、team-mode.md
- 完成: Category 路由模型从 gpt-5-nano 统一调整为 glm 系（06-04，agent-model-mapping.md 同步更新）

## [SECRET-REF] 记录

- `configs/opencode.json` 含 ppchat provider apiKey（私有仓明文策略，2026-06-03 用户授权）。恢复入口: `docs/runbooks/recover-config.md`。

> governance lint 2026-06-10: OK, 5 WARN (Jeff1993 contracts 未引用, 均为其他项目既有问题, 与本项目无关).
