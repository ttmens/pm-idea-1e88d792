# Context — idea-1e88d792 (测试项目)

## Glossary

| Term | Definition |
|------|------------|
| PM Pipeline | Hermes Agent 驱动的产品从想法到 MVP 的端到端自动化流水线，包含 align → research → spec → MVP → ship 等阶段 |
| Greenfield | 从零开始的新项目场景，无存量代码或文档，所有产物由 pipeline 首次生成 |
| Align Stage | 对齐阶段，产出 CONTEXT.md（术语表 + 系统上下文）和 decisions.md（架构决策记录），通过 gates 验证后进入下一阶段 |
| Gates | 阶段性质量检查清单，以 gates.json 格式存储，包含问题陈述、用户画像、成功指标、非目标等必填字段的通过/失败状态 |
| Stage-complete | 阶段完成脚本，负责运行 gates 验证、更新 PROGRESS.md、git commit + push、触发下游任务 dispatch |
| Human Checkpoint | 人工检查点，在 align/spec/ship 阶段后阻塞 pipeline，等待用户确认产物后再继续 |
| Kanban Worker | 被 Kanban dispatcher 分配到特定任务的 Hermes Agent worker，以 headless 模式自主完成工作 |
| ADR (Architecture Decision Record) | 架构决策记录，记录不可逆选择及其上下文、决策内容和后果 |
| Harness Rules | harness-rules.yaml 中定义的项目运行时规则，包含技术栈、构建/测试命令、人工检查点列表等 |
| Inner Loop | MVP 编码阶段的内循环迭代机制，在 max_iterations 限制内自动构建-测试-修复 |
| 飞书 Grill | 通过飞书机器人与用户进行 1-2 轮交互问答，补充 brief 中缺失的关键信息 |
| Brownfield | 存量项目场景，已有代码/文档基础，pipeline 侧重审计和增量迭代 |

## Stakeholders

| 角色 | 说明 |
|------|------|
| PM Pipeline 运维团队 | 验证 pipeline 端到端流程正确性 |
| 个人开发者 | [ASSUMED] 通过测试项目了解 pipeline 能力 |

## Constraints

- 测试用途，不投入真实研发资源
- 技术栈限定 Python
- 语言 zh-CN
- Inner loop 最多 3 次迭代
- 三个强制人工检查点：align / spec / ship
- [ASSUMED] 无需飞书额外交互，grill preflight 已完成
- [ASSUMED] GitHub Pages 部署通道已可用

## Non-goals

- 不产出面向最终用户的商业产品
- 不进行真实市场调研
- 不搭建生产级基础设施
- 不验证复杂场景下的 pipeline 健壮性

## System Context

```
[用户 /goal] → Gateway → feishu-grill-preflight → 00-brief.md
                                                    ↓
                                    decompose-pm-pipeline → Kanban Board
                                                    ↓
                        pm-aligner (align) → CONTEXT.md + decisions.md
                                                    ↓
                              Human Checkpoint (align) → unblock
                                                    ↓
                        pm-analyst (research) → 02-research.md
                                                    ↓
                        ... → MVP → Ship → Retro
```

## 风险与假设

### 假设清单
1. 测试项目使用最小可行配置，不涉及复杂集成
2. 各阶段 worker 可以正常 dispatch 到对应 profile
3. GitHub Pages 部署通道已可用
4. 无需飞书额外交互
5. 测试完成后项目可归档

### 风险
- **低风险**: 测试项目范围极小，即使某个阶段失败也不影响 pipeline 核心功能验证
- **中风险**: 如果 GitHub Pages 推送失败，ship 阶段无法完成，但不影响 align/research/spec 验证
- **验证策略**: 每个阶段通过 gates.json 确认 4/4 检查通过后再推进

## 反合理化表格

| 可能的合理化 | 实际风险 | 应对措施 |
|-------------|---------|---------|
| "反正只是测试，不用太认真" | Gates 不通过会导致 pipeline 阻塞 | 严格按 schema 填写所有必填字段 |
| "跳过人工检查点节省时间" | 违背 pipeline 设计原则 | align/spec/ship 三次检查点不可跳过 |
| "用真实项目配置会更好看" | 偏离测试目标，浪费资源 | 保持最小可行配置 |

## 失败模式

| 失败模式 | 触发条件 | 影响 | 缓解措施 |
|---------|---------|------|---------|
| Gates 验证失败 | CONTEXT/decisions 行数不足 | stage-complete 退出码 1 | 确保文件 ≥50 行 |
| 04-mvp 目录缺失 | test_cmd/build_cmd 执行失败 | validate_gates 报错 | 创建空目录或 stub 文件 |
| debates/ 目录缺失 | G1/G2 辩论产物检查失败 | validate_gates 报错 | 创建 debates/ 目录 |
| Git push 失败 | 远程仓库权限问题 | 产物无法同步 | 本地保留，手动排查 |
