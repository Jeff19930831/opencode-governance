# Runbook: 让 omo (Sisyphus) 跑通自定义 ppchat gpt-5.5

> 适用：OpenCode + oh-my-openagent 插件 + 第三方 OpenAI 兼容代理（ppchat / ddsst）的 gpt-5.5。
> 背景：2026-06-06 排查 "装了 omo 插件后 gpt-5.5 用不了（server_error / 卡住）"，定位到四个独立坑。

## 可用配置（已验证）

见 [`configs/opencode.json`](../../configs/opencode.json)。核心四要素缺一不可：

```jsonc
{
  "provider": {
    "ppchat-codex": {
      "npm": "@ai-sdk/openai",                         // ① 必须用 @ai-sdk/openai (Responses API)
      "options": {
        "baseURL": "https://code.ppchat.vip/v1",
        "apiKey": "sk-..."
      },
      "models": {
        "gpt-5.5": {
          "name": "gpt-5.5",                           // ② model 必须有 name 字段
          "options": { "reasoningEffort": "high" }     // ④ reasoning 在这里设，不要靠 omo variant
        }
      }
    }
  },
  "plugin": ["oh-my-openagent@latest"],
  "mcp": {
    "ruflo": { "enabled": false }                      // ③ 必须禁用 ruflo
  }
}
```

`oh-my-openagent.json` 里 agent/category 的 model 引用要用 `ppchat-codex/gpt-5.5`（与 provider key 一致），不要残留旧 provider 名。

---

## 根因 1（最关键）：ruflo 工具撑爆请求 → 上游 server_error

omo 插件**自带捆绑** ruflo MCP（在 omo 包的 `.opencode/opencode.json` 里，你自己的 config 不写也会加载）。实测工具数分布：

| MCP | 工具数 |
|---|---|
| **ruflo** | **293** |
| agentmemory | 53 |
| 其余 (codegraph/lsp/ast_grep/context7/grep_app/websearch) | ~37 |
| **合计** | **383 → 请求体 ~377KB** |

ppchat 的 gpt-5.5 上游**扛不住 383 工具的巨型请求**，生成中途返回 HTTP 500 `server_error`（带 "you can retry" + OpenAI request id）。

- **不装 omo 时能用**：默认 `build` agent 只有几个工具，请求很小。
- **装了 omo 就坏**：Sisyphus + 383 工具。

**抓包二分实测阈值**：工具 ≤300 能用，383 必崩。禁用 ruflo → **90 工具 / 177KB → 200 OK**。

> 代价：失去 ruflo 的 293 个 swarm/agent 工具。这个低价代理就是扛不住大请求；要 ruflo 得换能扛的端点/模型。

诊断手段：把 opencode 的 `baseURL` 临时指向一个本地日志代理（转发到真实端点并打印请求体），就能看到 omo 实际发出的 `tools` 数量和 `reasoning` 参数——日志文件里看不到请求体。

## 根因 2：npm 包必须用 `@ai-sdk/openai`（Responses API）

| npm 包 | 端点 | reasoning+tools 表现 |
|---|---|---|
| `@ai-sdk/openai-compatible` | `/v1/chat/completions` | reasoning 同步死等 **45s** → 连接被 reset |
| **`@ai-sdk/openai`** | `/v1/responses` | reasoning 服务端处理 **~9s** → OK |

这个代理的 reasoning 只有在 Responses API 路径下才高效。**不要**按 OpenCode 官方文档无脑选 openai-compatible。

## 根因 3：model 要有 `name` 字段

`"gpt-5.5": {}` 空对象会导致模型元数据解析不全。必须 `"gpt-5.5": { "name": "gpt-5.5" }`。

## 根因 4：reasoning effort 设在 opencode 模型 options，不是 omo variant

自定义 provider 的 gpt-5.5 不在 OpenCode/models.dev 模型缓存里 → omo 走 "heuristic-backed" 模式，**agent 的 `variant` 字段被忽略**，reasoning 默认 medium。

正确做法（opencode 直接透传）：

```json
"models": { "gpt-5.5": { "name": "gpt-5.5", "options": { "reasoningEffort": "high" } } }
```

可选值：`none / minimal / low / medium / high / xhigh`。high 更慢更耗 token。

---

## 相关上游 issue（佐证）

- omo #3616 "Still Can't Use Sisyphus by gpt-5.5" —— 自定义 provider 落到 heuristic 模式
- omo #1788 "reasoningSummary parameter sent to non-Claude models" —— omo 给 GPT 误发 Claude 专用参数（PR #1794：reasoning:false 时过滤）
- opencode #16154 "gpt-5 defaults + openai-compatible sends incompatible reasoning params"

## 验证

```bash
opencode run "say hi"     # 应正常返回，无 server_error
```
TUI 里启动后发一句话；日志 `~/.local/share/opencode/log/` 里搜 `server_error` 应为 0。
