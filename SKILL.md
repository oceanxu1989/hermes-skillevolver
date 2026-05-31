---
name: skillevolver
description: >-
  Auto-evolve Agent Skills through contrastive trajectory comparison.
  5-phase loop: Strategy→Execute→Compare→Patch→Audit.
  Works on Hermes and OpenCode. Inspired by SkillEvolver (arXiv:2605.10500).
license: MIT
compatibility: "hermes opencode crush"
metadata:
  audience: agent-developers
  category: meta-skill
  inspired_by: "SkillEvolver (arXiv:2605.10500)"
---

# SkillEvolver — 技能自动进化器

> 让 AI Agent 用**对比学习**自动改进任何 skill。
> 支持 **Hermes** / **OpenCode (Crush)** 双平台。

## 核心原理

不靠 AI 自我反思（"你觉得哪里不好？"），而是**实际执行多个改进策略，对比成功和失败的轨迹，从中提取精确的改进信号**。

```
原始 Skill → ① 策略探索(K个方案) → ② 并行执行(子Agent测试)
→ ③ 轨迹对比(成vs败) → ④ 定向打补丁(只改信号指向的)
→ ⑤ 独立审计(过拟合/硬编码检查) → 循环R轮 → 进化后 Skill
```

## 输入

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| 目标 skill 名 | ✅ | — | 如 `code-review`、`deploy-check` |
| 验证任务文件 | ✅ | — | 每行一个任务描述 |
| `--rounds` | ❌ | 2 | 进化轮数 |
| `--strategies` | ❌ | 3 | 每轮并行策略数 |
| `--audit` | ❌ | on | 是否启用审计 |

### 验证任务文件格式

```text
# 每行一个任务描述
审查 /path/to/file.py 的代码质量
检查 /etc/nginx/nginx.conf 的语法错误  
分析 /var/log/syslog 最近 50 行的异常
```

## 执行协议

### Phase 0 — 平台检测

执行前先判断当前运行的 Agent 平台，使用对应的工具：

| 操作 | 🔵 Hermes | 🟢 OpenCode / Crush |
|------|-----------|---------------------|
| 读取 skill | `skill_view(name)` | 直接 `read_file` 读取 `SKILL.md` |
| 修改 skill | `skill_manage(action='patch', ...)` | 直接文件编辑（write/patch） |
| 并行子 Agent | `delegate_task` | 原生 multi-agent（描述任务让其 spawn） |
| 审计子 Agent | `delegate_task` | 原生 multi-agent |

### Phase 1 — 策略探索

1. 读取目标 skill 完整内容
2. 读取验证任务列表  
3. 分析 skill 弱点：步骤不清、pits 缺失、触发器不准、验证不可执行
4. 生成 K 个**不同方向**的改进策略（精确性 / 覆盖率 / 鲁棒性 / 效率，各方向不重叠）

### Phase 2 — 并行执行

对每条策略，**并行**创建子 Agent 测试：

- 子 Agent 得到原始 skill + 该策略的修改描述 + 验证任务列表
- 子 Agent 逐任务执行，报告 ✅/❌ 及失败原因
- **所有策略同时跑**，不串行

### Phase 3 — 轨迹对比

收集所有子 Agent 报告，提取三种 delta 信号：

| 信号类型 | 判定 | 处理 |
|----------|------|------|
| **Skill 缺陷** | 所有策略都失败 | skill 本身结构有问题 |
| **策略收益** | 某策略独自成功 | 该策略的修改有效 |
| **执行失误** | 偶发单次失败 | 标记，不强制修改 |

输出对比矩阵：

```
任务 \ 策略    | 策略A | 策略B | 策略C
任务1          |  ✅   |  ✅   |  ❌
任务2          |  ❌   |  ✅   |  ❌  
任务3          |  ✅   |  ✅   |  ✅
```

### Phase 4 — 定向打补丁

**只改信号指向的部分**：

- 策略收益 → 合并成功策略的修改
- Skill 缺陷 → 新增步骤 / pitfall / 说明
- 未受影响的段落**原样保留**

Hermes 用 `skill_manage(action='patch')`，OpenCode 用文件编辑。

### Phase 5 — 独立审计

用一个**独立子 Agent**（非参与进化的 Agent）审计：

- [ ] 是否过拟合（只对验证任务有效）
- [ ] 是否有硬编码的具体值/路径
- [ ] 新增内容与原有风格是否一致
- [ ] 是否有冗余（与原有步骤重复）
- [ ] 是否有安全风险（权限建议过宽等）

不通过 → 回退上一轮版本。

### 循环

重复 Phase 1-5 共 R 轮，每轮基于上一轮结果。

## 跨平台兼容

本 skill 产出的 `SKILL.md` 文件遵循 [Agent Skills](https://agentskills.io) 开放标准，**所有平台通用**：

| Agent | Skill 路径 | 执行进化 | 消耗产物 |
|-------|-----------|---------|---------|
| Hermes | `~/.hermes/skills/` | ✅ 原生 | ✅ |
| OpenCode / Crush | `~/.config/opencode/skills/` | ✅ 原生 | ✅ |
| Claude Code | `~/.claude/skills/` | 需适配 | ✅ |

## 使用示例

```
# Hermes
进化 code-review skill，任务文件 /root/tasks.txt，3 轮，4 策略

# OpenCode (same SKILL.md, just placed in different location)
evolve the code-review skill using tasks from tasks.txt, 3 rounds
```

## Pitfalls

- 任务 ≥ K×2，确保每个策略有足够样本暴露优劣
- 策略要有区分度——全改同一段没意义
- **对比 > 反思**——Phase 3 是核心价值所在
- **补丁 > 重写**——精准编辑，保留 skill 历史
- 审计不能跳——没有审计的进化越改越偏
- 子 Agent 语言：context 里指定输出语言

## 参考

- SkillEvolver 论文: [arXiv:2605.10500](https://arxiv.org/abs/2605.10500)
- EmbodiSkill 论文: [arXiv:2605.10332](https://arxiv.org/abs/2605.10332)
- Agent Skills 标准: [agentskills.io](https://agentskills.io)
- 社区开源实现: [github.com/victorzhong0110/skill-evolution](https://github.com/victorzhong0110/skill-evolution)
