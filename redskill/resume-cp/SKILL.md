---
name: 上下文恢复resume
description: 配合 context-checkpoint 使用的恢复技能。用户 /clear 之后说"恢复checkpoint""继续上次的工作""resume-cp"时，读取 ~/.claude/checkpoints/ 下最新的 checkpoint 文件，对齐任务状态后等用户确认再继续。
---

# Resume from latest checkpoint

用户已通过 `/checkpoint` 把上一段对话浓缩成了 markdown 文件，现在需要从那份存档恢复工作。

## 执行步骤

1. **定位 checkpoint 文件**
   - 如果用户提供了参数 `$ARGUMENTS`：读 `~/.claude/checkpoints/$ARGUMENTS`（注意自动补上目录前缀）
   - 如果没参数：跑 `ls -t ~/.claude/checkpoints/*.md | head -1` 找最新的那个

2. **读取并吸收内容**
   - 用 Read 工具读整个文件
   - 对齐这些信息：任务目标、已完成、待完成、关键决策、相关文件

3. **向用户报告恢复结果**（格式）

   ```
   ✅ 已加载 checkpoint: {文件名}

   📋 任务：{1 句话目标}
   🔹 已完成 {N} 项
   🔹 待办 {M} 项 —— 下一步建议从【{待办首项}】开始

   确认继续？或告诉我要改方向。
   ```

4. **等用户确认再动手**
   - 不要自己上来就改代码
   - 如果待办里涉及具体文件修改，先等用户说"继续"或具体指示

## 重要

- checkpoint 目录固定在 `~/.claude/checkpoints/`
- 如果目录为空，告诉用户"没找到 checkpoint，是否之前没保存？"
- 读完的 checkpoint 不要自动删除
