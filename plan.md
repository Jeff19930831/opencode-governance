# Plan

## 活跃计划 (Active)

### 2026-06: 初始化 opencode-governance 项目
- [x] 创建项目目录和 git 仓库
- [x] 复制当前配置文件 (opencode.json, oh-my-openagent.json)
- [x] 编写 README 和文档结构
- [x] 编写 agent-model-mapping.md
- [x] 编写 team-mode.md
- [ ] 编写 handoff.md
- [ ] 首次 checkpoint (git commit + push)

## 待办 (Next)

### 短期
- [ ] 将 Category 路由模型统一调整为 glm-5.1 (目前大部分仍为 gpt-5-nano)
- [ ] 添加 omo 安装/升级 runbook
- [ ] 记录 MCP 配置 (agentmemory, ruflo)

### 中期
- [ ] 建立配置变更 review 流程 (修改前必须更新本文档)
- [ ] 添加自动化检查脚本 (验证配置完整性)
- [ ] 记录每个 ADR 的决策背景

### 长期
- [ ] 监控各模型实际使用成本和效果
- [ ] 根据使用数据优化模型分配
- [ ] 建立配置 A/B 测试流程

## 归档 (Archived)

_暂无_

---

## 变更触发条件

当以下情况发生时，必须更新本文档:

1. 新增/删除 agent
2. 修改 agent 模型
3. 新增/修改 provider
4. 修改 team mode 配置
5. 新增/删除 skill/plugin

---

> 📖 **Doc Lifecycle Gate**: 任何配置变更前，先在本文档更新对应计划项，再执行实际修改。
