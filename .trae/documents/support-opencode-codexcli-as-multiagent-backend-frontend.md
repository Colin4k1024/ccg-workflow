# 计划：支持 OpenCode CodexCLI 作为前后端多Agent

## 目标概述

在 CCG 工作流中集成 opencode-codex-orch 作为可选的前端/后端多Agent解决方案，让用户可以在 Codex CLI 原生架构和 opencode-codex-orch 多Agent编排之间选择。

## 架构分析

### 当前 CCG 架构
```
Claude Code (编排)
       │
   ┌───┴───┐
   ↓       ↓
Codex   Gemini
(后端)   (前端)
   │       │
   └───┬───┘
       ↓
  Unified Patch
```

### opencode-codex-orch 架构
```
用户请求
    ↓
[Intent Gate] — 意图分类
    ↓
[Sisyphus] — 主协调器
    ├─→ [Prometheus] — 战略规划
    ├─→ [Atlas] — 任务编排执行
    ├─→ [Oracle] — 架构咨询
    ├─→ [Librarian] — 文档/代码搜索
    └─→ [Category-based agents] — 按任务类型路由
```

### 集成后架构
```
Claude Code (编排)
       │
   ┌───┴───┐
   ↓       ↓
┌───────────────┐     ┌─────────────────┐
│ Codex 原生模式 │ OR  │ opencode-codex-orch │
│ (codeagent-   │     │ (Sisyphus 编排)  │
│  wrapper)     │     │  - ultrawork    │
└───────────────┘     │  - prometheus    │
                      └─────────────────┘
```

## 实现步骤

### 阶段 1: 类型和配置扩展

1. **修改 `src/types/index.ts`**
   - 添加 `ModelType`: `'opencode'` 作为新选项
   - 更新 `ModelRouting` 接口支持 opencode

2. **修改 `src/utils/config.ts`**
   - 更新 `createDefaultRouting()` 添加 opencode 选项
   - 更新类型定义支持新模型

### 阶段 2: codeagent-wrapper 扩展

3. **修改 `codeagent-wrapper/main.go`**
   - 添加 `--backend opencode` 支持
   - 实现 opencode CLI 调用封装
   - 处理 opencode 的输出格式（与 Codex 不同）

4. **修改 `codeagent-wrapper/config.go`**
   - 添加 opencode 相关配置项
   - 定义 opencode 命令行参数

### 阶段 3: 安装器集成

5. **修改 `src/commands/init.ts`**
   - 添加 opencode CLI 安装检查
   - 添加 opencode API 配置选项
   - 在初始化向导中增加 opencode 选项

6. **修改 `src/utils/installer.ts`**
   - 添加 opencode 安装逻辑
   - 添加 opencode 配置同步

7. **修改 `src/utils/installer-data.ts`**
   - 更新 `WORKFLOW_PRESETS` 支持 opencode
   - 添加 opencode 相关的 workflow 模板

### 阶段 4: Expert Prompts

8. **创建 `templates/prompts/opencode/` 目录和提示词**
   - `architect.md` — Sisyphus 架构师角色
   - `analyzer.md` — 分析角色
   - `reviewer.md` — 审查角色
   - `debugger.md` — 调试角色
   - `optimizer.md` — 优化角色
   - `tester.md` — 测试角色

### 阶段 5: 命令模板

9. **创建/修改命令模板**
   - 创建 `templates/commands/opencode-workflow.md` — opencode 多Agent工作流
   - 创建 `templates/commands/opencode-exec.md` — opencode 全权执行
   - 修改 `templates/commands/workflow.md` 支持 opencode 后端

### 阶段 6: Skill 集成

10. **更新 `templates/skills/orchestration/multi-agent/SKILL.md`**
    - 添加 opencode-codex-orch 作为多Agent选项
    - 文档化 Sisyphus/Prometheus/Atlas 角色映射

### 阶段 7: MCP 集成 (可选)

11. **修改 `src/utils/mcp.ts`**
    - 添加 opencode MCP 工具同步
    - 支持 opencode 配置同步到 `~/.opencode/config.toml`

### 阶段 8: 文档

12. **更新文档**
    - `docs/guide/configuration.md` — 添加 opencode 配置说明
    - `docs/guide/mcp.md` — 添加 opencode MCP 说明

## 文件修改清单

| 文件 | 操作 | 说明 |
|------|------|------|
| `src/types/index.ts` | 修改 | 添加 ModelType = 'opencode' |
| `src/utils/config.ts` | 修改 | 更新默认路由配置 |
| `codeagent-wrapper/main.go` | 修改 | 添加 opencode 后端支持 |
| `codeagent-wrapper/config.go` | 修改 | opencode 配置项 |
| `src/commands/init.ts` | 修改 | opencode 安装选项 |
| `src/utils/installer.ts` | 修改 | opencode 安装逻辑 |
| `src/utils/installer-data.ts` | 修改 | workflow presets |
| `src/utils/mcp.ts` | 修改 | opencode MCP 同步 |
| `templates/prompts/opencode/*.md` | 新建 | 6个专家提示词 |
| `templates/commands/opencode-*.md` | 新建 | opencode 命令模板 |
| `templates/skills/orchestration/multi-agent/SKILL.md` | 修改 | 添加 opencode 文档 |
| `docs/guide/configuration.md` | 修改 | opencode 配置说明 |
| `package.json` | 修改 | 添加 opencode 依赖说明 |
| `CLAUDE.md` | 修改 | 更新变更记录 |

## 关键设计决策

### 1. 双模式支持
- **模式A**: Codex 原生 (`--backend codex`) — 当前架构
- **模式B**: opencode-codex-orch (`--backend opencode`) — 新架构

### 2. 意图路由
opencode-codex-orch 自带的 Intent Gate 可用于：
- 自动识别任务类型（research/implementation/investigation/fix）
- 按类别路由到合适的Agent

### 3. 配置一致性
```toml
# ~/.claude/.ccg/config.toml
[routing]
frontend = { models = ["gemini", "opencode"], primary = "gemini" }
backend = { models = ["codex", "opencode"], primary = "codex" }
```

### 4. 安装检测
```bash
# 检测 opencode 是否安装
which opencode || echo "not installed"
```

## 测试计划

1. **单元测试**: 类型定义、配置生成
2. **集成测试**: codeagent-wrapper opencode 后端调用
3. **手动测试**: 完整工作流验证

## 风险评估

| 风险 | 影响 | 缓解 |
|------|------|------|
| opencode 输出格式不稳定 | 中 | 版本检测 + 适配层 |
| 多后端配置复杂度 | 低 | 清晰的配置向导 |
| API 兼容性问题 | 中 | 分阶段集成，MVP 先只支持 ultrawork 模式 |
