# Hermes SkillEvolver

> 让你的 Hermes Agent 自己进化自己的技能。五步对比学习循环，无需外部依赖。

灵感来自清华大学 [SkillEvolver](https://arxiv.org/abs/2605.10500) 和 [EmbodiSkill](https://arxiv.org/abs/2605.10332) 论文。

## 工作原理

```
原始 Skill → 策略探索(生成K个方案) → 并行执行(子Agent测试)
→ 轨迹对比(成败vs成败) → 定向打补丁(只改信号指向的)
→ 独立审计(过拟合/硬编码检查) → 循环R轮 → 进化后 Skill
```

## 快速开始

1. 将 `SKILL.md` 放到 Hermes 的 skills 目录（如 `~/.hermes/skills/meta/hermes-skillevolver/`）

2. 准备一个验证任务文件 `tasks.txt`，每行一个任务：
```
审查 /path/to/file.py 的代码质量
检查 /etc/nginx/nginx.conf 的语法错误
分析 /var/log/syslog 最近 100 行的异常
```

3. 在 Hermes 中调用：
```
进化 code-review skill，任务文件 /root/tasks.txt，2 轮，3 策略
```

## 依赖

- Hermes Agent（自带所有工具）
- 无需任何外部 pip 包、API key

## 核心设计

- **对比优于反思**：对比成功/失败轨迹提取改进信号，而非让 AI 自我反思
- **补丁优于重写**：用 `skill_manage patch` 精准修改，保留 skill 历史
- **独立审计**：独立子 Agent 检查过拟合/硬编码/安全风险
- **并行执行**：K 个策略用 `delegate_task` 并行测试，不串行等待

## 论文

- SkillEvolver: [arXiv:2605.10500](https://arxiv.org/abs/2605.10500)
- EmbodiSkill: [arXiv:2605.10332](https://arxiv.org/abs/2605.10332)
