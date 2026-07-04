---
name: context-checkpoint
description: Context 管理技能。当 context 用量升高（建议 30% 就动手）或任务告一段落时，把当前任务状态、决策和待办导出到 checkpoint 文件，然后 /clear 重新开始。防止 auto-compaction 随机压缩造成关键信息丢失。触发词：/checkpoint、保存进度、context 快满了。
---

# Context Checkpoint Skill

## 使命

在 context window 被填满之前，主动将关键信息导出到文件，然后引导用户 /clear 重新开始，确保零信息丢失。

**核心原则**：手动 checkpoint > auto-compaction（你控制什么留下，而不是让系统随机压缩）。

为什么这也是省 token：上下文是滚雪球的，每一轮都拖着全部历史跑；auto-compaction 要到窗口 ~83.5% 才触发，且由机器决定留什么。主动在低水位收尾，把状态浓缩成 <1k token 的文件，新会话冷启动几乎免费。

---

## 触发时机

1. **主动触发**：用户说 "/checkpoint"、"保存进度"、"context 快满了"
2. **Claude 主动判断**：对话已经很长（>30 轮）、context 用量过 30%、或即将开始新的大任务时，主动建议 checkpoint

---

## 执行步骤

### Step 1: 提炼当前状态

在脑中整理以下信息（不需要输出给用户）：

- **当前任务目标**：在做什么，为什么做
- **已完成的工作**：改了哪些文件，做了什么决策
- **未完成的工作**：接下来要做什么，待解决的问题
- **关键决策和约束**：任何影响后续工作的决定
- **相关文件路径**：需要继续操作的文件

### Step 2: 写入 Checkpoint 文件

用 Write 工具创建文件（目录不存在先创建）：

```
路径: ~/.claude/checkpoints/YYYY-MM-DD-HH-简短描述.md
```

文件格式：

```markdown
# Checkpoint: {任务简述}
> 创建时间: YYYY-MM-DD HH:MM
> 分支: {当前 git 分支}

## 任务目标
{1-2 句话描述目标}

## 已完成
- {完成项 1}
- {完成项 2}

## 待完成
- [ ] {待办 1}
- [ ] {待办 2}

## 关键决策
- {决策 1}: {原因}

## 相关文件
- `path/to/file1` - {说明}
- `path/to/file2` - {说明}

## 恢复指令
请读取此文件，然后继续执行"待完成"中的任务。
上下文：{任何需要恢复工作所需的额外信息}
```

### Step 3: 引导用户

输出（极简版，两步走）：

```
✅ checkpoint 已存 → YYYY-MM-DD-HH-xxx.md

下一步：
  /clear        ← 清空 context
  /resume-cp    ← 自动读最新 checkpoint 恢复工作
```

如果用户想恢复的不是最新的，告诉他们 `/resume-cp 文件名.md` 可以指定。

---

## 注意事项

- Checkpoint 文件要**自包含**：即使没有当前对话上下文，读取文件后也能完全恢复工作
- 文件名要有意义：日期+时间+简短描述，便于查找
- 不要在 checkpoint 中放大段代码，只放路径和关键改动说明
- 配套恢复命令见同仓库的 `commands/resume-cp.md`
