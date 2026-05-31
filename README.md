# SkillEvolver

> 让 AI Agent 自己进化自己的技能。五步对比学习循环。  
> 支持 **Hermes** · **OpenCode (Crush)** · **Claude Code**

灵感来自清华 [SkillEvolver](https://arxiv.org/abs/2605.10500) 论文。

## 工作原理

```
原始 Skill
  → ① 策略探索（生成 K 个不同改进方案）
  → ② 并行执行（子 Agent 同时测试所有方案）
  → ③ 轨迹对比（成 vs 败 → 提取改进信号）
  → ④ 定向打补丁（只改信号指向的部分）
  → ⑤ 独立审计（过拟合/硬编码检查）
  → 循环 R 轮
  → 进化后 Skill
```

**核心思想**：不靠 AI 自我反思，而是实际跑多个策略、对比成败轨迹、精准定位改进点。

## 快速开始

### Hermes

```bash
# 放置 skill
cp SKILL.md ~/.hermes/skills/meta/skillevolver/SKILL.md

# 使用
进化 code-review skill，任务文件 /path/to/tasks.txt
```

### OpenCode / Crush

```bash
# 放置 skill
mkdir -p ~/.config/opencode/skills/skillevolver
cp SKILL.md ~/.config/opencode/skills/skillevolver/SKILL.md

# 使用
evolve the code-review skill using /path/to/tasks.txt
```

### 验证任务文件

`tasks.txt` 每行一个任务：

```
审查 /path/to/file.py 的代码质量
检查 /etc/nginx/nginx.conf 的语法错误
分析 /var/log/syslog 最近 50 行的异常
```

## 跨平台

| Agent | Skill 路径 | 执行进化 | 产物 |
|-------|-----------|---------|------|
| Hermes | `~/.hermes/skills/` | ✅ 原生 | ✅ |
| OpenCode / Crush | `~/.config/opencode/skills/` | ✅ 原生 | ✅ |
| Claude Code | `~/.claude/skills/` | 需适配 | ✅ |

产出的 `SKILL.md` 文件基于 [Agent Skills](https://agentskills.io) 开放标准，所有平台通用。

## 依赖

- 零外部依赖
- 无需 API key
- 只用到 Agent 自身的文件读写和子 Agent 能力

## 论文

- [SkillEvolver (arXiv:2605.10500)](https://arxiv.org/abs/2605.10500)
- [EmbodiSkill (arXiv:2605.10332)](https://arxiv.org/abs/2605.10332)
- 社区实现: [victorzhong0110/skill-evolution](https://github.com/victorzhong0110/skill-evolution)
