# Decisions — idea-1e88d792 (测试项目)

## ADR-001: 技术栈选择 Python

- **Status**: accepted
- **Context**: harness-rules.yaml 指定技术栈为 Python。测试项目需要最小化复杂度，Python 是 pipeline 原生支持的语言，工具链成熟（py_compile、compileall）。
- **Decision**: MVP 使用纯 Python 实现，不引入外部框架或依赖。
- **Consequences**: 快速验证 pipeline 流程；但功能受限于标准库，无法展示复杂生态集成能力。

## ADR-002: 部署目标为 GitHub Pages

- **Status**: accepted
- **Context**: RUN.md 中已列出 Pages URL `https://ttmens.github.io/pm-idea-1e88d792/`。测试项目不需要后端服务，静态页面足以验证 ship 阶段流程。
- **Decision**: MVP 产物部署到 GitHub Pages（静态 HTML/JS），不涉及服务器或数据库。
- **Consequences**: 零运维成本；但无法测试需要后端的场景（API、数据库、认证等）。

## ADR-003: 测试项目使用最小可行范围

- **Status**: accepted
- **Context**: 本想法标题为"测试"，目的是验证 pipeline 端到端流程而非构建真实产品。
- **Decision**: 所有阶段产物保持最小可用形态：brief 填充核心字段、CONTEXT 包含 ≥3 条术语、decisions 记录关键选择即可。MVP 阶段仅需产出可编译通过的代码。
- **Consequences**: pipeline 验证速度快、资源消耗低；但无法验证复杂场景下的 pipeline 健壮性。

## ADR-004: 人工检查点采用两段式确认

- **Status**: accepted
- **Context**: harness-rules.yaml 定义了三个检查点（align/spec/ship），align 阶段明确采用 two-phase 模式：首次运行 stage-complete 后 block 等待用户确认，unblock 后验证 gates 再 complete。
- **Decision**: align 阶段严格遵循两段式流程，不跳过人工确认环节。
- **Consequences**: 确保用户对 CONTEXT 有最终审核权；增加了一轮交互延迟，但对测试场景可接受。

## ADR-005: debates/ 目录用于 G1/G2 辩论产物

- **Status**: accepted
- **Context**: validate-gates.py 检查 debates/ 目录是否存在，且需要 align-*.md 格式的文件。该目录用于存放两个对立观点（G1/G2）的辩论记录，是 stage align 的必要产物。
- **Decision**: 在 align 阶段创建 debates/ 目录，并放入 align-001.md 辩论文件。
- **Consequences**: 满足 gates 检查要求；对于测试项目，辩论内容可以极简，但目录结构和文件命名必须匹配预期模式。

## 风险登记

| 风险 | 级别 | 状态 | 缓解措施 |
|------|------|------|---------|
| deploy 为高风险决策 | high | human_checkpoint | 通过 harness-rules.yaml 中的人工检查点控制 |
| tech_choice 为中风险 | medium | write_adr_and_notify | 已通过 ADR-001 记录 |
| refactor 为低风险 | low | auto_verify | 自动化验证即可 |

## 验证策略

- 每个阶段完成后运行 `stage-complete.py` 确认 gates 通过
- gates.json 中所有 check 必须 pass 才能 push
- 人工检查点阻塞 pipeline 直到用户明确 unblock
- debates/align-*.md 文件必须在 align 阶段完成前创建

## 假设与约束汇总

- 假设 1: 测试项目使用最小可行配置
- 假设 2: 各阶段 worker 可以正常 dispatch
- 假设 3: GitHub Pages 部署通道已可用
- 假设 4: 无需飞书额外交互
- 假设 5: 测试完成后项目可归档
- 约束: Python 技术栈、zh-CN 语言、inner loop 最多 3 次迭代

## 附录: 阶段产物清单

| 文件 | 状态 | 行数 |
|------|------|------|
| 00-brief.md | 已丰富 | 完整 |
| CONTEXT.md | 已创建 | 72+ 行 |
| decisions.md | 已创建 | 50+ 行 |
| debates/align-001.md | 已创建并 resolved | 完整 |
| gates.json | 自动生成 | pass |
