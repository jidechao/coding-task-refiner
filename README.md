# coding-task-refiner

> AI 编程任务提示词优化器 —— 把一句"帮我改一下"，翻译成 AI 编程代理能正确执行的结构化指令。

## 项目定位

这是一个遵循 Agent Skills 标准格式（`SKILL.md`）的 Skill，可安装到任何支持该标准的 AI 编程代理（Coding Agent）中——Claude Code、Codex、Cursor、Windsurf、Qoder 等均可使用。它用于在把任务交给 AI 编程代理之前，先对提示词做一次工程级的梳理：**不直接执行编码**，而是产出可以直接复制粘贴给 AI 编程代理使用的**精炼提示词**。

**核心原则**：小任务靠说清楚，复杂任务靠控过程，关键任务一定要验结果。

## 解决的问题

用户交给 AI 编程代理的任务描述通常存在四类通病：

| 通病 | 后果 |
|------|------|
| 缺少文件/行号定位 | AI 猜错位置，改到无关代码 |
| 缺少规范/文档依据 | AI 编造"合理解释"，包装得像真的一样 |
| 缺少范围边界 | 改一处变成改十几处，越界修改 |
| 缺少验收标准 | AI 说"完成了"但实际没通过测试 |

本 Skill 通过 **四步法交互式梳理**，逐项补齐这些信息。

## 四步法工作流

```
Phase 1: 说清楚   →  给事实锚点，划范围边界
Phase 2: 问明白   →  要依据，不要猜测
Phase 3: 控过程   →  先摸影响面，再决定改什么
Phase 4: 验结果   →  AI 说完成不算完成
```

每个 Phase 都有一个 **✅ 确认检查点**，与用户对齐后再进入下一步，避免 AI 替用户拍板。

详见 [`references/methodology.md`](references/methodology.md)。

## 输出提示词骨架

四个 Phase 梳理完毕后，按以下七段结构组装最终提示词：

```
1. 上下文（Context）        —— 要阅读的文件、规范、测试
2. 范围边界（Scope）        —— 只处理什么，不修改什么
3. 范例参照（Reference）    —— 参考哪个文件的实现方式（可选）
4. 分析要求（Analysis）     —— 找不到依据就提问，不要猜测
5. 影响分析（Impact）       —— 需要同步修改的关联项
6. 执行闸门（Gate）         —— 先出方案，确认后再执行
7. 验收标准（Verification） —— 自查 → 工具验证 → 事实核对 → Diff 检查
```

完整骨架与段落选择规则见 [`assets/prompt-structure.md`](assets/prompt-structure.md)。

## 五类任务模板

针对不同类型的任务，提供开箱即用的模板：

- **排查类** —— 找问题、查隐患、审代码
- **理解类** —— 搞清楚现有代码/设计的来龙去脉
- **生成类** —— 从零新建模块/文件/接口
- **变更类** —— 修改已有代码，风险最高，闸门最多
- **收口类** —— 验证、归档、发布前的最后核对

模板与填充示例见 [`references/templates.md`](references/templates.md)。

## 目录结构

```
coding-task-refiner/
├── SKILL.md                      # Skill 主入口，定义元数据、工作流、约束
├── README.md                     # 本文件
├── agents/
│   └── openai.yaml               # 可选界面配置（显示名、简介、默认提示词），供支持该约定的代理读取
├── assets/
│   └── prompt-structure.md       # 优化后提示词的标准输出骨架
└── references/
    ├── methodology.md            # 四步法详解，含每个 Phase 的检查项与常见陷阱
    ├── templates.md              # 五类任务的现成模板
    └── examples.md               # 完整真实案例（电费计算），从模糊到精确的全过程
```

## 安装

本 Skill 遵循 Agent Skills 标准结构（`SKILL.md` + `references/` + `assets/`），无构建、无依赖，把整个目录放进目标代理的 Skills 目录即可被自动发现。

### 方式一：安装到 Skills 目录（推荐）

| 编程代理 | 项目级 | 用户级（全局） |
|---------|--------|--------------|
| Claude Code | `.claude/skills/coding-task-refiner/` | `~/.claude/skills/coding-task-refiner/` |
| Codex | `.codex/skills/coding-task-refiner/` | `~/.codex/skills/coding-task-refiner/` |
| Cursor | `.cursor/skills/coding-task-refiner/` | — |
| Windsurf | `.windsurf/skills/coding-task-refiner/` | `~/.codeium/windsurf/skills/coding-task-refiner/` |
| Qoder | 项目 Skills 目录 | — |

以 Claude Code 为例：

```bash
# 用户级（所有项目可用）
git clone https://github.com/jidechao/coding-task-refiner.git ~/.claude/skills/coding-task-refiner

# 项目级（仅当前项目）
git clone https://github.com/jidechao/coding-task-refiner.git .claude/skills/coding-task-refiner
```

其他代理同理：把本目录复制或 symlink 到上表对应路径。`agents/openai.yaml` 是可选的界面配置，供支持该约定的代理读取，不影响其余代理使用，不需要也可删除。

### 方式二：作为规则 / 系统提示词导入

如果使用的代理暂不支持 Skills 机制，把 [`SKILL.md`](SKILL.md) 全文粘贴进该代理的自定义规则（Rules / Guidelines / System Prompt），效果等同。

## 使用方式

### 触发词

在与 AI 对话时，出现以下意图会自动激活本 Skill：

- "优化提示词" / "prompt 优化"
- "帮我整理任务再给 AI"
- "这个需求怎么交代"
- "任务梳理" / "帮我写清楚这个任务"
- "AI 老是理解错"
- "怎么给 AI 编程 agent 布置任务"
- "coding prompt" / "refine task"

### 典型交互流程

1. **用户**：粘贴原始的模糊任务描述
2. **Skill**：识别任务类型（排查/理解/生成/变更/收口），给出一句话诊断
3. **Skill**：按 Phase 1 → 4 逐项梳理，每步结束前与用户确认
4. **Skill**：组装最终提示词，用户可直接复制给 AI 编程代理使用

完整交互案例见 [`references/examples.md`](references/examples.md)。

## 适用与不适用

**✅ 适用**
- 需要在把任务交给 AI 编程代理之前，做一次结构化梳理
- 已有 prompt 感觉"AI 老是理解错"，想找出信息缺口
- 复杂变更任务，希望加入人工闸门和影响分析

**❌ 不适用**
- 直接执行编码任务（那是 Coding Agent 的职责）
- 通用 Prompt Engineering 教程
- 非编码类提示词优化（如文案、翻译、绘图）

## 约束

- 不替用户拍板：无法确认的信息标记为「待确认」，不自行填充
- 不生成教程：输出是给 AI 编程代理的指令，不是给用户的教学材料
- 不添加需求：只梳理和结构化用户提供的信息
- 保持简洁：能一句话说清的不写一段
- 高风险操作（删文件、改公共接口、动数据库、Git 提交、发布生产）必须显式标注闸门

## 验收标准

生成的最终提示词必须满足：

- [ ] 包含事实锚点（文件/行号/规范名称中的至少一项）
- [ ] 包含明确的范围边界
- [ ] 涉及修改的任务包含人工闸门
- [ ] 涉及修改的任务包含至少一条验收标准
- [ ] 没有"帮我看看""可能有问题"等模糊表述
- [ ] 可以直接复制粘贴给 AI 编程代理使用，无需额外解释

## License

未指定。
