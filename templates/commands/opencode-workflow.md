---
description: 'OpenCode 多Agent工作流 - 使用 opencode-codex-orch 进行全自动化任务执行'
---

<!-- CCG:OPENCOD:WORKFLOW:START -->

**核心哲学**

opencode-codex-orch 将单个 AI Agent 转变为协调的开发团队。与 CCG 原生的多模型协作（Claude 编排 + Codex/Gemini 执行）不同，opencode 工作流提供：
- **完全自动化**：输入任务 → Sisyphus 自动分解 → 分配 → 执行 → 验证
- **多模型编排**：自动路由不同任务到合适的模型（GPT 深度推理 / Gemini 前端 / Haiku 快速任务）
- **并行 Agent**：同时启动 5+ 个后台 Agent（研究/实现/验证并行）
- **意图分类**：自动识别任务类型（research/implementation/investigation/fix）

**适用场景**

- 需要多 Agent 并行协作的复杂任务
- 需要深度 GPT 推理能力的任务
- 需要完全自动化执行的场景
- 想要体验 opencode-codex-orch 的高级编排能力

**vs CCG 原生工作流**

| 特性 | `/ccg:workflow` | `/ccg:opencode-workflow` |
|------|-----------------|-------------------------|
| 编排层 | Claude Code | Sisyphus (opencode) |
| 执行层 | Codex + Gemini | 多模型自动路由 |
| 自动化程度 | 半自动（用户确认） | 全自动（ultrawork） |
| Agent 并行 | 无 | 5+ 并行 |
| 适合场景 | 精细控制 | 快速交付 |

---

**命令语法**

```
/ccg:opencode-workflow <任务描述>
```

**参数**

- `$ARGUMENTS` 或 `$CLAUDE_REPL`: 任务描述（必需）

**Guardrails**

- 工作目录：自动使用当前工作目录
- opencode 必须已安装并配置
- 任务通过 Sisyphus 自动分解，无需手动规划
- ultrawork 模式：完全自动化，无需用户交互（除非遇到阻塞）
- prometheus 模式：按 Tab 键可进入面试式规划模式

**Steps**

---

### Step 1: 环境准备

1. **获取工作目录**
   ```
   Bash: pwd
   ```
   保存为 WORKDIR。

2. **检查 opencode 安装**
   ```
   Bash: which opencode
   ```
   如果未安装，提示用户安装：
   ```
   # 安装 opencode (需要 OpenAI API key 或兼容 provider)
   npm install -g opencode-codex-orch
   # 或参考 https://github.com/allOwO/opencode-codex-orch
   ```

---

### Step 2: 任务输入

1. **解析任务描述**
   - 如果 `$ARGUMENTS` 为空，使用 `AskUserQuestion` 请求任务描述
   - 任务描述应包含：目标、约束、验收标准

2. **确定执行模式**
   ```
   // 默认使用 ultrawork 模式（完全自动）
   MODE: "ultrawork"
   
   // 或使用 prometheus 模式（需要用户交互规划）
   MODE: "prometheus"  // 按 Tab 键激活
   ```

---

### Step 3: 调用 opencode-codex-orch

1. **Ultrawork 模式（推荐）**

   ```
   Bash({
     command: "~/.claude/bin/codeagent-wrapper --progress --backend opencode \"{{WORKDIR}}\" <<'EOF'\n<TASK>\n{{$ARGUMENTS}}\n</TASK>\nEOF",
     timeout: 7200000,  // 2小时超时
     description: "opencode ultrawork: {{$ARGUMENTS}}"
   })
   ```

   Sisyphus 会自动：
   - 使用 Intent Gate 分类任务
   - 创建任务分解
   - 分配给专业 Agent（Prometheus/Atlas/Oracle/Librarian/Explore）
   - 并行执行
   - 验证完成

2. **Prometheus 模式（可选）**

   ```
   Bash({
     command: "cd \"{{WORKDIR}}\" && opencode prometheus \"{{$ARGUMENTS}}\"",
     timeout: 3600000,  // 1小时超时
     description: "opencode prometheus: {{$ARGUMENTS}}"
   })
   ```

   Prometheus 会：
   - 面试式提问以澄清需求
   - 建立详细计划
   - 然后交给 Atlas 执行

---

### Step 4: 监控执行

1. **进度监控**
   - 使用 `--progress` 参数查看实时进度
   - 监控 Sisyphus 的任务分解和分配
   - 监控各 Agent 的执行状态

2. **日志查看**
   ```
   # 查看详细日志
   cat ~/.codex/logs/*.log
   
   # 查看最近活动
   opencode status
   ```

3. **阻塞处理**
   - 如果 Sisyphus 遇到无法解决的阻塞
   - 使用 `AskUserQuestion` 请求用户介入
   - 提供选择或额外信息

---

### Step 5: 结果收集

1. **获取执行结果**
   - opencode 会输出完整的执行报告
   - 包含：完成的变更、测试结果、审查结果

2. **结果验证**
   ```
   Bash({
     command: "cd \"{{WORKDIR}}\" && git status",
     description: "检查变更状态"
   })
   
   Bash({
     command: "cd \"{{WORKDIR}}\" && git diff --stat",
     description: "查看变更统计"
   })
   ```

3. **运行测试（如有）**
   ```
   Bash({
     command: "cd \"{{WORKDIR}}\" && npm test 2>&1 | head -50",
     description: "运行测试套件"
   })
   ```

---

### Step 6: 报告汇总

向用户展示执行摘要：

```
🕸️ OpenCode Multi-Agent 执行完成

【执行引擎】
  - 编排器: Sisyphus (opencode-codex-orch)
  - 模式: ultrawork
  - 工作目录: {{WORKDIR}}

【任务分解】
  - 总任务数: N
  - 并行 Agent: N 个
  - 完成率: X%

【变更摘要】
  - 修改文件: N 个
  - 新增文件: N 个
  - 删除文件: N 个

【质量检查】
  - 测试: 通过/失败
  - Lint: 通过/失败
  - Typecheck: 通过/失败

【下一步】
  - 查看详情: git diff
  - 运行测试: npm test
  - 提交代码: /ccg:commit
```

---

**Exit Criteria**

- [ ] opencode 执行完成
- [ ] 变更文件已生成
- [ ] 测试通过（如有测试套件）
- [ ] 执行报告已展示给用户
- [ ] 下一步建议已提供

**相关命令**

- `/ccg:opencode-exec` — opencode 全权执行（更简洁的界面）
- `/ccg:workflow` — CCG 原生多模型协作
- `/ccg:team` — CCG Agent Teams 工作流
- `/ccg:team-exec` — CCG Agent Teams 并行实施

<!-- CCG:OPENCOD:WORKFLOW:END -->
