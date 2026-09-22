# structured-report

一个遵循 [Agent Skills 开放标准](https://agentskills.io/specification) 的 skill，用于按结构化表达原则撰写或重写汇报类文档。可在 Claude Code、Cursor、Codex、Kiro、Gemini CLI、GitHub Copilot 等支持该标准的 agent 中使用。

## 目录结构

```
.
└── skills/
    └── structured-report/
        ├── SKILL.md                  # 入口：输出边界 + 七步工作流 + 触发约束
        ├── LICENSE
        ├── references/               # 按需加载的详细文档
        │   ├── principles.md         # 19 条原则
        │   ├── frameworks.md         # 金字塔 / SCQA / STAR / 矩阵 / 5 Why
        │   ├── writing-rules.md      # 11 条精简改写规则 + 反例对照
        │   └── checklist.md          # 自检清单：硬门禁 + 优化项
        └── assets/                   # 文档模板
            ├── template-progress.md
            ├── template-decision.md
            └── template-review.md
```

## 安装

Agent 不会扫描本仓库的 `skills/` 目录，它只读各自约定的发现目录。把 `skills/structured-report/` 整个目录复制到下表任一位置即可。

| Agent | 项目级 | 全局 |
|---|---|---|
| 通用标准位置 | `.agents/skills/` | `~/.agents/skills/` |
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Kiro | `.kiro/skills/` | `~/.kiro/skills/` |
| GitHub Copilot | `.github/skills/` | `~/.copilot/skills/` |
| Codex / Cursor / Gemini CLI / OpenCode | `.agents/skills/` | `~/.agents/skills/` |

`.agents/skills/` 是多数工具共同支持的通用位置，只装这一个通常就够。

**Windows**

```powershell
# 装到某个项目
Copy-Item -Recurse skills\structured-report D:\git\my-repo\.agents\skills\

# 装到全局，所有项目可用
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
Copy-Item -Recurse skills\structured-report "$HOME\.agents\skills\"
```

**macOS / Linux**

```bash
# 装到某个项目
mkdir -p ~/code/my-repo/.agents/skills
cp -R skills/structured-report ~/code/my-repo/.agents/skills/

# 装到全局
mkdir -p ~/.agents/skills
cp -R skills/structured-report ~/.agents/skills/
```

复制后重启 agent 或重新加载窗口，skill 即可被发现。

> 想让改动即时生效、不必反复复制，可以改用目录链接代替复制：
> Windows `New-Item -ItemType Junction -Path <目标> -Target <源>`，
> Unix `ln -s <源绝对路径> <目标>`。

## 使用

```
/structured-report 帮我写 Q3 增长团队的季度汇报
/structured-report 改写这份周报（粘贴草稿）
```

不支持斜杠命令的 agent 直接说明即可：「用结构化汇报 skill 写一下这个复盘」。

## 触发控制

本 skill 刻意设计为**仅显式调用**。Agent Skills 标准下 `description` 是 agent 匹配 skill 的唯一依据，因此采用正向触发词 + 负向排除场景的写法：

```yaml
description: 结构化撰写与重写汇报文档（……）。仅在用户显式调用时使用：输入 /structured-report，
  或明确说出「用结构化汇报 skill」……用户只是普通地让你写文字、写文档、写 README、
  写注释、写邮件时不要使用本 skill。
```

SKILL.md 正文开头重复一次该约束，作为二次保险。

各平台还有更强的隔离手段，按需选用：Claude Code 可用 subagent 限定可见 skill；Kiro 可用 custom agent 的 `skill://` 白名单，或改成 `inclusion: manual` 的 steering 文件。

## 原则概览

| 分组 | 原则 |
|---|---|
| 想清楚 | 受众导向、隐性思维显性化、So What / Why 双向追问、事实与观点分离 |
| 排结构 | 结论先行、以上统下、MECE、逻辑递进、显性思维结构化、黄金三点法、平行结构、结构思维形象化 |
| 写文字 | 短语 + 句子合成列表项、标题即结论、量化配基准、删减 30% |
| 对人 | 行动项 SMART、风险前置 + 闭环、归属与出处 |

完整说明见 `skills/structured-report/references/principles.md`。

## 输出边界

`SKILL.md` 里五条硬约束优先于工作流，不设字数上限，约束的是内容：

| 组 | 约束 | 要点 |
|---|---|---|
| 写什么 | 范围锁定 | 只写用户界定范围内的内容；范围内的坏消息仍须报 |
| 写什么 | 归属与出处 | 他人产出、转述数据、未验证结论必须标注 |
| 怎么写 | 结构配额 | 条目数、句长、嵌套深度的上限，数值见 `SKILL.md` |
| 怎么写 | 信息密度 | 每个列表项携带一个新事实：数据 / 根因 / 状态变化 / 口径 |
| 怎么写 | 内容形态 | 列表项 = 粗体短语 + 句子，不用短语当标题 |

## 规范符合性

- `SKILL.md` 含合法 YAML front-matter，`name` 与目录名一致，仅含小写字母与连字符
- `description` 说明了做什么和何时用，含触发关键词，未超 1024 字符
- 可选字段 `license`、`compatibility`、`metadata` 均符合标准约束
- 遵循渐进式披露：入口文件承载硬约束与流程，长文档放 `references/`，模板放 `assets/`
- 文件引用为相对路径且只有一层深
- 硬约束单一来源：配额数值、必备章节、门禁条目只在 `SKILL.md` 声明，`references/` 只做展开，避免两处漂移
- 全文不含具体业务场景：规则与示范一律用占位符和抽象类型，不绑定行业、产品或真实指标名，便于迁移到任意场景

可用官方 `skills-ref` 校验库验证 front-matter 与命名规范。

## 许可

MIT，见 `skills/structured-report/LICENSE`。
