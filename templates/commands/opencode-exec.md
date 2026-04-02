---
description: 'OpenCode 全权执行 - 使用 opencode-codex-orch 的 ultrawork 模式进行完全自动化执行'
---

<!-- CCG:OPENCOD:EXEC:START -->

**核心哲学**

`/ccg:opencode-exec` 是 opencode-codex-orch 的最简入口。只需提供任务描述，Sisyphus 自动完成所有工作：分析 → 规划 → 分配 → 执行 → 验证。

**与 `/ccg:opencode-workflow` 的区别**

| 特性 | `/ccg:opencode-workflow` | `/ccg:opencode-exec` |
|------|-------------------------|---------------------|
| 交互程度 | 显示执行细节 | 最小化输出 |
| 用途 | 演示/学习 | 生产快速执行 |
| 错误处理 | 详细日志 | 简洁报告 |

**适用场景**

- 快速执行简单任务
- 自动化脚本集成
- 追求效率的生产使用
- 需要 Claude token 极低消耗

---

**命令语法**

```
/ccg:opencode-exec <任务描述>
```

**参数**

- `$ARGUMENTS`: 任务描述（必需）

**执行流程**

```
用户输入 → Sisyphus (ultrawork) → 自动分解 → 分配 → 执行 → 报告
```

**Steps**

---

### Step 1: 快速环境检查

```
Bash: pwd
```

保存 WORKDIR。

---

### Step 2: Ultrawork 执行

```
Bash({
  command: "~/.claude/bin/codeagent-wrapper --backend opencode \"{{WORKDIR}}\" <<'EOF'\n<TASK>\n{{$ARGUMENTS}}\n</TASK>\nEOF",
  timeout: 7200000,
  description: "opencode ultrawork"
})
```

Sisyphus 会自动：
1. **Intent Gate** — 分类任务类型
2. **Task Decomposition** — 分解为可执行子任务
3. **Agent Spawning** — 分配给专业 Agent
4. **Parallel Execution** — 并行执行
5. **Verification** — 验证完成
6. **Reporting** — 生成报告

---

### Step 3: 简洁结果

等待执行完成，展示：

```
✅ OpenCode Ultrawork 完成

【变更】
- 文件: N 个
- 类型: 添加/修改/删除

【验证】
- 状态: 通过/失败

【下一步】
git diff  # 查看变更
```

---

**Exit Criteria**

- [ ] 执行完成
- [ ] 简洁报告已展示

**相关命令**

- `/ccg:opencode-workflow` — 详细执行流程
- `/ccg:workflow` — CCG 原生工作流
- `/ccg:codex-exec` — Codex 全权执行

<!-- CCG:OPENCOD:EXEC:END -->
