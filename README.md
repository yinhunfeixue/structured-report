# structured-report

一个遵循 [Agent Skills 开放标准](https://agentskills.io/specification) 的 skill，用于按结构化表达原则撰写或重写汇报类文档。可在 Claude Code、Cursor、Codex、Kiro、Gemini CLI、GitHub Copilot 等支持该标准的 agent 中使用。

## 目录结构

```
.
└── skills/
    └── structured-report/
        ├── SKILL.md                  # 入口：16 条硬约束 + 五步流程 + 触发约束
        ├── LICENSE
        ├── references/               # 查阅用，不需要通读
        │   ├── principles.md         # 表达原则全集，标明哪些进了硬约束
        │   ├── frameworks.md         # 金字塔 / SCQA / STAR / 矩阵 / 5 Why
        │   ├── writing-rules.md      # 12 条改写规则 + 整段诊断演示
        │   └── checklist.md          # 16 条门禁核对清单
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
/structured-report 写一份本季度的项目进展汇报
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

## 设计取舍

汇报出错分两类，代价差一个量级：**事实性错误**（口径算错、归属虚报、坏消息漏报）发出去收不回，读者也无法从文档本身发现；**表达问题**（太长、没重点）改几句就好。

因此本 skill 的硬约束只管第一类，共 16 条，全部可判定：

| 组 | 条数 | 管什么 | 不遵守的后果 |
|---|---|---|---|
| A 口径 | 5 | 数据源、时间边界、主体标识、纳入排除、基准来源 | 整篇作废 |
| B 归属 | 4 | 他人产出、转述数据、未验证结论的标注 | 整篇失信 |
| C 范围 | 3 | 只写范围内的内容，但坏消息不能漏 | 答非所问或隐瞒 |
| D 表达 | 4 | 结论先行、事实观点分离、列表形态、信息密度 | 读者读不到重点 |

表达层面的其余内容（条目数、句长、标题写法、19 条原则、12 条改写规则）放在 `references/` 里当建议，不进门禁。

**v3.0.0 的主要变化**：约束从 92 条降到 16 条。上一版把「写得更漂亮」和「不出错」混在同一级门禁里，实测无人逐条执行；同时最容易造成事实性错误的口径环节反而只有一张表。这版按后果重新分层，口径与归属加厚，形态类砍掉大半。

## 规范符合性

- `SKILL.md` 含合法 YAML front-matter，`name` 与目录名一致，仅含小写字母与连字符
- `description` 说明了做什么和何时用，含触发关键词，未超 1024 字符
- 可选字段 `license`、`compatibility`、`metadata` 均符合标准约束
- 遵循渐进式披露：入口文件承载 16 条硬约束与流程，方法手册放 `references/`，模板放 `assets/`
- 文件引用为相对路径且只有一层深
- 单一来源：硬约束与建议值只在 `SKILL.md`，删除词表只在 `writing-rules.md` 规则 4，其余文件引用不复述
- 不含真实业务数据：示例用虚构但具体的场景（如图书借阅系统），既不泄露使用者的内部信息，也不退化成「某个量从 X 变成 Y」这类教不会东西的占位符

可用官方 `skills-ref` 校验库验证 front-matter 与命名规范。

## 许可

MIT，见 `skills/structured-report/LICENSE`。
