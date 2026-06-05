# 配置恢复手册

## 场景 1: 配置文件丢失

### 从本仓库恢复

```bash
# 恢复 opencode 核心配置
cp configs/opencode.json ~/.config/opencode/opencode.json

# 恢复 omo 配置
cp configs/oh-my-openagent.json ~/.config/opencode/oh-my-openagent.json

# 重启 opencode
opencode
```

### 验证恢复

```bash
# 检查 provider 是否可用
opencode provider list

# 检查 agent 配置
opencode agent list

# 测试 team mode
opencode team list
```

---

## 场景 2: Secret/Key 丢失

### ⚠️ 双密钥库陷阱（改 key 必读）

opencode 把每个 provider 的 key **同时**存在两处，`auth.json` 会按 provider id **覆盖** config 里的 key。只改一处会出现「改了配置还是用不了」：

| 位置 | 字段 |
|------|------|
| `~/.config/opencode/opencode.json` | `provider.<名>.options.apiKey` |
| `~/.local/share/opencode/auth.json` | `<名>.key` |

**改 key 时两处必须同步更新**。验证一致性：

```bash
node -e 'const a=require(process.env.HOME+"/.local/share/opencode/auth.json"),c=require(process.env.HOME+"/.config/opencode/opencode.json");for(const p of Object.keys(c.provider||{})){const ak=a[p]&&a[p].key,ck=c.provider[p].options&&c.provider[p].options.apiKey;console.log(p, ak===ck?"MATCH":"MISMATCH ❌", ak?ak.slice(-6):"(no auth)")}'
```

> 官方更省事的做法：key 只放 `auth.json`（`opencode auth login`），config 里 provider 不写 `apiKey`。但本治理标准把 key 写在 config，所以保持两边同步。
> 另：删 provider 时记得一并删 `auth.json` 里的同名孤儿项。

### ppchat1 / ddsst API Key

来源: https://code.ddsst.online/
恢复步骤:
1. 登录管理后台
2. 重新生成 API Key
3. 更新 `~/.config/opencode/opencode.json` 的 `provider.<名>.options.apiKey`
4. **同步更新 `~/.local/share/opencode/auth.json` 的 `<名>.key`**（见上方双密钥库陷阱）
5. 同步更新本仓库 `configs/opencode.json`

### zai-coding-plan API Key

来源: https://z.ai/subscribe
恢复步骤:
1. 登录 z.ai
2. 在订阅管理页面查看/重置 Key
3. 更新 `~/.zcode/v2/acp-config/opencode/*/opencode.json`
4. 同步更新本仓库相关配置

---

## 场景 3: 模型不可用

### 现象

Agent 返回 API error，或任务一直失败。

### 排查步骤

1. **检查 provider 状态**
   ```bash
   curl -s https://code.ddsst.online/v1/models -H "Authorization: Bearer $API_KEY"
   ```

2. **查看 fallback 是否生效**
   检查 `configs/oh-my-openagent.json` 中对应 agent 的 `fallback_models`

3. **手动降级**
   临时将 agent 模型改为 `openai/gpt-5.4-mini-fast` 或其他可用模型

4. **更新配置后重启**
   ```bash
   opencode
   ```

---

## 场景 4: Team Mode 无法启动

### 现象

`team_create` 返回 "team_mode not enabled"

### 解决

1. 确认配置文件中存在:
   ```json
   {
     "team_mode": {
       "enabled": true,
       "max_parallel_members": 4,
       "max_members": 8
     }
   }
   ```

2. 确认配置在正确位置:
   - 用户级: `~/.config/opencode/oh-my-openagent.jsonc`
   - 项目级: `.opencode/oh-my-openagent.jsonc`

3. 重启 opencode (必须)

4. 验证:
   ```bash
   opencode team list
   ```

---

## 场景 5: 完整环境重建

### 新机器初始化

```bash
# 1. 安装 opencode
npm install -g opencode

# 2. 安装 omo 插件
bunx oh-my-openagent install

# 3. 克隆本仓库
git clone <repo-url> ~/Projects/opencode-governance

# 4. 恢复配置
cp ~/Projects/opencode-governance/configs/*.json ~/.config/opencode/

# 5. 配置 secret (手动输入)
# - ppchat1 API key
# - zai-coding-plan API key

# 6. 验证
opencode doctor
```

---

## 备份策略

| 频率 | 操作 |
|------|------|
| 每次修改后 | `git commit -m "config: <变更摘要>"` |
| 每周 | `git push` 到远程仓库 |
| 每月 | 导出完整配置快照到 `archive/YYYY-MM/` |

---

## 紧急联系

| 服务 | 状态页 | 支持渠道 |
|------|--------|---------|
| ppchat | - | 管理后台工单 |
| z.ai | - | https://z.ai/support |
| OpenCode | - | Discord/GitHub Issues |

