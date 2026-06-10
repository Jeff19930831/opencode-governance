# Plan

## 活跃计划 (Active)

_暂无进行中事项_

## 待办 (Next)

### 短期
- [ ] 添加 omo 安装/升级 runbook（升级 = 改 pin 版本 + 重填缓存，见 startup-performance.md 预防规则）
- [ ] 记录 MCP 配置 ADR（agentmemory 已在 runbook 覆盖；补 codegraph / websearch 等插件自带 MCP 的说明）
- [ ] Zed / ZCode 常驻 opencode 实例重启后，验证新配置生效（ruflo 不再出现、启动 <5s）

### 中期
- [ ] 建立配置变更 review 流程 (修改前必须更新本文档)
- [ ] 添加自动化检查脚本 (验证配置完整性 + 检测 `@latest` 写法回潮 + 插件缓存目录非空)
- [ ] 记录每个 ADR 的决策背景

### 长期
- [ ] 监控各模型实际使用成本和效果
- [ ] 根据使用数据优化模型分配
- [ ] 建立配置 A/B 测试流程

## 归档 (Archived Plans)

- 2026-06 初始化项目 → 已全部完成，见 [progress.md](./progress.md)（仓库/配置快照/文档/handoff/首次 checkpoint）
- 2026-06-04 Category 路由模型统一调整为 glm 系 → 完成，见 [progress.md](./progress.md)

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
