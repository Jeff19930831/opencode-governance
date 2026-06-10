# Runbook: OpenCode 启动慢排查与修复

> 背景：2026-06-10 排查 "opencode 启动 44 秒以上、偶尔卡死数分钟"，定位到三个独立根因并修复。
> 结果：完整配置解析（含插件加载）从 44s+ → ~3.5s。

## 三个根因

### 1. ruflo MCP 通过 Claude 配置被隐式导入（最大元凶，~31s）

- oh-my-openagent 插件自带 `claude-code-mcp-loader`，会读取 `~/.claude.json` 的全局 `mcpServers` 并**自动注册到 opencode**。
- `~/.claude.json` 里配置过 `ruflo`（claude-flow 马甲，`npx -y ruflo@latest mcp start`），每次启动 npx 重新解析 `@latest` + 冷启动，固定超时 30 秒。
- **陷阱**：从 opencode 各配置文件里删 ruflo 无效——它根本不在 opencode 配置里。

### 2. 插件 `@latest` 写法 + 缓存损坏（3.3s ~ 数分钟）

- `plugin: ["oh-my-openagent@latest"]` 每次启动都要联网解析最新版。
- 缓存目录 `~/.cache/opencode/packages/oh-my-openagent@latest/` 异常为空 → 每次启动重装 136 个包（实测 35s）。
- 后台已有其他 opencode 实例（如 Zed 常驻的 acp 进程）时，安装会**死锁**（实测卡 10 分钟+，零网络活动）。

### 3. agentmemory MCP 未固定版本

- `npx -y @agentmemory/mcp` 每次启动多一次 registry 查询（~1-2s）。

## 修复步骤（2026-06-10 已执行）

1. `~/.claude.json` 删除 `mcpServers.ruflo`（备份：`~/.claude.json.bak-ruflo-fix-*`）。
2. `~/.config/opencode/opencode.json`：`oh-my-openagent@latest` → `oh-my-openagent@4.8.1`。
3. 预填插件缓存：
   ```bash
   mkdir -p ~/.cache/opencode/packages/oh-my-openagent@4.8.1
   cd ~/.cache/opencode/packages/oh-my-openagent@4.8.1
   echo '{"dependencies":{"oh-my-openagent":"4.8.1"}}' > package.json
   npm install --no-audit --no-fund
   ```
4. agentmemory 固定 `@agentmemory/mcp@0.9.24`（同步改 `~/.claude.json`、`~/.config/opencode/opencode.json`、`~/.agents/mcp.json`）。

## 诊断工具

```bash
opencode debug startup        # 打印启动耗时（毫秒）
opencode debug config         # 输出合并后配置（含插件导入的 MCP）；卡住本身就是症状
opencode debug paths          # 显示 data/config/cache 路径
# 日志（UTC 时间）：~/.local/share/opencode/log/
# 找出 ≥500ms 的慢步骤：
awk '{match($0,/\+[0-9]+ms/); if(RSTART){d=substr($0,RSTART+1,RLENGTH-3); if(d+0>=500) print}}' <logfile>
```

## 本机安装拓扑（重要认知）

| 实例 | 路径 | 触发方式 |
|------|------|---------|
| npm 全局 | `%APPDATA%\npm\node_modules\opencode-ai\bin\opencode.exe` | 终端 `opencode` |
| Zed 内置 | `%LOCALAPPDATA%\Zed\external_agents\registry\opencode\v_*\opencode.exe` | Zed agent 面板（acp 常驻） |
| ZCode 内置 | `%LOCALAPPDATA%\Programs\ZCode\resources\opencode\opencode.exe` | ZCode（注入 `~/.zcode/v2/acp-config/opencode/<hash>/opencode.json`，自动再生，勿手改） |

三者**共享**全局配置/缓存/日志，修全局配置对三者都生效；但 Zed/ZCode 常驻进程需重启才加载新配置。

## 预防规则

1. **opencode plugin 和 MCP 一律固定版本号**，禁用 `@latest`。
2. 给 Claude Code 加 MCP 前先想清楚：omo 会把它带进 opencode，慢 MCP 会拖累两边。
3. 升级插件 = 改 pin 版本号 + 重新预填缓存目录（见上），不要回退到 `@latest`。
4. 启动变慢先查 `~/.cache/opencode/packages/` 下对应插件目录是否为空。
