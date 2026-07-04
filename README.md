# claude-context-checkpoint

Claude Code 的上下文洁癖工作流：**checkpoint 存档 → /clear → resume-cp 一键恢复**。

手动 checkpoint > auto-compaction。系统要到窗口 ~83.5% 才自动压缩，而且是机器决定留什么；这套两件套让你在任意水位主动收尾，把任务状态浓缩成 <1k token 的文件，新会话冷启动几乎免费，重要信息零丢失。

## 为什么省 token

- 上下文是滚雪球的，每一轮都拖着全部历史重新计费
- auto-compact 触发晚、不可控，重要决策可能被压没
- checkpoint 文件自包含，/clear 之后历史随时敢扔

## 安装

把两个文件拷到 Claude Code 的用户目录：

```bash
mkdir -p ~/.claude/skills/context-checkpoint ~/.claude/commands ~/.claude/checkpoints
cp skills/context-checkpoint/SKILL.md ~/.claude/skills/context-checkpoint/
cp commands/resume-cp.md ~/.claude/commands/
```

## 用法

```
（干活干到一个段落，或 /context 看到用量过 30%）
/checkpoint      ← Claude 把任务状态/决策/待办写进 ~/.claude/checkpoints/
/clear           ← 放心清空
/resume-cp       ← 读最新 checkpoint，对齐状态后接着干
/resume-cp 2026-07-04-15-xxx.md   ← 也可以指定某一份
```

## 文件

| 文件 | 装到哪 | 干什么 |
|---|---|---|
| `skills/context-checkpoint/SKILL.md` | `~/.claude/skills/context-checkpoint/` | 存档：提炼状态→写 checkpoint 文件→引导 /clear |
| `commands/resume-cp.md` | `~/.claude/commands/` | 恢复：定位最新存档→读取→报告→等确认 |

`redskill/` 目录是小红书 RedSkill 平台的单文件打包版，内容一致。

## License

MIT
