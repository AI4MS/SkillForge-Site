# Skill构建流程操作指南

> 版本：v0.1（初版） | 维护人：高于翔 | 面向对象：组内有计算经验的学生/研究者
>
> 本指南是活文档，随团队实践持续更新。踩坑记录和典型案例欢迎随时补充。

---

## 〇、写在前面：Skill是什么

在Agent 框架下，**Skill 是一段被 Agent 加载、可按需调用的领域知识封装**。它本质上是一份结构化的操作手册——告诉 Agent 在什么场景下、按什么步骤、用什么参数、跑什么命令、怎么验证结果。

一个 Skill 的物理形态就是一个目录：

```
<skill-name>/
├── SKILL.md          # 核心文件：触发条件 + 操作步骤 + pitfalls + 验证清单
├── scripts/          # 可执行脚本（可选）
├── references/       # 详细参考文档（可选，用于拆分大Skill）
├── templates/        # 输入文件模板（可选）
└── config.yaml       # 配置文件（可选，脚本需要时）
```

其中 **SKILL.md 是唯一必需的文件**，其余按需添加。

**一句话理解：Skill = 你脑子里的计算经验，被工程化封装成 Agent 能执行的标准流程。**

大模型写得出代码，但写不出你的经验——参数为什么这么选、哪里容易出错、怎么判断结果可不可信。这些才是 Skill 的核心价值，也是我们团队的护城河。

### ==路由型 Skill vs 任务型 Skill （待完善）==

考虑到后续计算skill的扩展需求，我认为我们团队的 Skill 可以采用"路由+子Skill"模式：

- **路由型 Skill**（如 `vasp`）：写共有的参数逻辑，并负责判断用户意图，分发到对应子Skill。（目前只做VASP，如果后续加入其他DFT计算软件，比如QE，ABACUS等，可以再添加上级路由，DFT）
- **任务型 Skill**（如 `vasp/relax`）：负责具体任务的输入生成、参数选择、流程执行。

如果 Skill 是某个计算引擎的多个任务类型，建议参考这种模式——一个路由 + 多个任务子Skill，而不是把所有逻辑塞进一个文件。

---

## 一、构建流程总览

一个人从零构建一个 Skill，走四步：

```
┌─────────────┐    ┌─────────────┐    ┌──────────────┐    ┌─────────────┐
│ 第一步       │    │ 第二步       │    │ 第三步        │    │ 第四步       │
│ 亲跑计算     │ →  │ 流程抽象     │ →  │ Agent生成初稿  │ →  │ 迭代优化     │
│             │    │             │    │              │    │             │
│ 自己先跑通   │    │ 复盘拆解     │    │ MatCreator    │    │ 领域经验打磨 │
│ 理解全过程   │    │ 三类步骤     │    │ 自动生成初稿    │    │ 评估+修复    │
└─────────────┘    └─────────────┘    └──────────────┘    └─────────────┘
```

**核心思路：先理解，再生成，后打磨。**

- 理解是前提——不理解计算本身，写不出好 Skill。
- Agent 负责降低工程门槛——不需要从零手写，在初稿基础上优化。
- 人负责质量把关——参数逻辑、错误处理、边界条件，这些靠领域经验。

---

## 二、第一步：亲跑计算

### 为什么要亲跑

这是基础中的基础。你不理解计算本身，就不可能写出好的 Skill：
- 不知道哪些参数是关键的 → 写不出准确的参数说明
- 不知道哪些步骤容易出错 → 写不出有用的 pitfalls
- 不知道怎么判断结果好坏 → 写不出可执行的验证步骤

**Agent 能帮你封装流程，但替代不了你对计算的理解。**

### 亲跑到什么程度算够

不是随便跑一次就行，要做到：

1. **完整跑通**——从输入文件准备、参数设置、提交计算、到结果分析，每一步都自己做过。
2. **至少跑一个典型体系**——选一个（或者请博后帮你选一个）经典的、结果可预期的体系作为基准。
3. **记录关键决策点**——每个参数为什么这么选？换一个值会怎样？
4. **踩过的坑都记下来**——报错信息、原因、解决办法。

### 产出物清单

亲跑阶段结束时，你手里应该有：

- [ ] 完整的输入文件（INCAR/POSCAR/POTCAR/KPOINTS，或对应的 LAMMPS 输入等）
- [ ] 完整的输出文件（OUTCAR/OSZICAR 等）
- [ ] 参数选择记录：每个关键参数的值 + 选择理由
- [ ] 踩坑笔记：遇到了什么问题、怎么解决的
- [ ] 结果分析记录：怎么判断计算是否成功、结果是否合理

> 💡 这些产出物是后面所有步骤的基础。流程抽象靠它，初稿生成喂给它，迭代优化对照它。

---

## 三、第二步：流程抽象

跑通之后，复盘整个流程，把计算拆成三类步骤：

### 三类步骤的划分

| 类型 | 特征 | 在 Skill 中的体现 |
|------|------|------------------|
| **标准化步骤** | 固定流程，不需要判断，照做即可 | 写成明确的步骤 + 命令 |
| **需要经验的步骤** | 参数选择、收敛策略、体系判断 | 写成参数决策逻辑 + 选择理由 |
| **容易出错的步骤** | 常见陷阱、边界条件、报错高发区 | 写成 pitfalls + 错误处理 |

### 产出物清单

流程抽象阶段结束时，你手里应该有：

- [ ] 步骤清单：按执行顺序列出所有步骤，标注每步属于哪一类
- [ ] 参数决策树：每个需要判断的参数，列出选择逻辑（什么情况选什么值、为什么）
- [ ] 陷阱列表：每个容易出错的点，列出错误现象 + 原因 + 解决办法

> 💡 这份拆解就是你未来 SKILL.md 正文的骨架。第三类（陷阱）直接变成 pitfalls 章节。

---

## 四、第三步：Agent生成初稿

### 用 MatCreator 生成

MatCreator 是我们内部的 Skill 快速迭代平台，已经具备辅助生成 Skill 初稿的能力。使用你亲跑时的计算输入，让它跑完，确保计算结果和亲跑一致，计算跑完后让它自动生成 SKILL.md 初稿和脚本骨架。

**这一步大大降低门槛**——你不需要从零手写 Markdown 和脚本，而是在初稿基础上优化。

### 初稿的预期质量

初稿能给你一个结构完整的起点：
- frontmatter 基本字段齐全
- 步骤按流程组织好了
- 命令和参数有基础值
- 可能有一些脚本骨架

**但初稿不完美，这是正常的。** 它缺少的是你的领域经验——参数选择的"为什么"、容易出错的地方、边界条件的处理。这些正是第四步要补的。

### 如何审查初稿

拿到初稿后，对照你在第二步做的流程抽象，逐项检查：

1. **步骤是否完整？** ——有没有漏掉的步骤？顺序对不对？
2. **参数是否合理？** ——初稿给的默认值对你的体系适用吗？
3. **决策逻辑有没有？** ——需要判断的参数，初稿有没有写选择逻辑？还是只给了一个硬编码值？
4. **pitfalls 有没有？** ——你踩过的坑，初稿里提到没有？
5. **验证步骤可执行吗？** ——跑完后怎么确认结果对不对？

把缺的补上、把不对的改掉——这就进入了第四步。

---

## 五、第四步：迭代优化

这是最花时间、也最有价值的一步。基于初稿，结合你的领域经验打磨。

### 五个打磨方向

#### 1. 参数选择逻辑

初稿往往只给一个默认值。好的 Skill 要写清楚选择逻辑：

```

K 点网格密度
核心原则：沿每个晶向的 K 点数 ≈ 与该方向晶胞长度成反比，保证倒空间采样各向同性。
  例：若 a1=5Å, a2=5Å, a3=15Å，则取 6×6×2 比取 4×4×4 更合理。

两种设置方式：
  - KPOINTS 文件（推荐，生产计算用）：显式写 N1×N2×N3 网格，Gamma 居中
  - KSPACING 标签（快速初算用）：不写 KPOINTS 文件时在 INCAR 里设。
    N_i = max(1, ceiling( |b_i|·2π / KSPACING ))
    其中 b_i 是倒格子矢量（晶体学约定 b_i·a_j = δ_ij）。
    ⚠ 注意公式里的 2π 系数：实际布里渊区采样间隔 ≈ KSPACING / 2π。
      （例：KSPACING=0.5 → 实际晶体学倒空间间隔 ≈ 0.08 1/Å）
    典型参考值：0.5（默认，稀疏，适合初算）/ 0.2~0.3（中等收敛）/ ≤0.1（密，精确计算）。
```

  

要点：不要只写"KPOINTS 用 4×4×4，ISMEAR=0"——要写清楚网格密度原则（反比于晶胞长度）、什么体系用什么 ISMEAR、什么计算目的不能用什么方法，以及为什么。这些耦合关系才是领域知识的核心。

#### 2. 错误处理

计算会失败。好的 Skill 要预判常见错误并给出解决办法：

- 不收敛 → 怎么调（增加 NELM、放宽 EDIFF、换 ALGO）
- 结果异常 → 怎么判断（能量是否为负、力是否合理、性质是否符合预期）

#### 3. 边界条件

默认流程覆盖的是典型情况。好的 Skill 要覆盖特殊情况，例如：

- 体系很大（>100原子）→ K-points 要减少，可能要用 Gamma-only
- 体系是孤立小分子 → 不需要 K-points，用 Gamma 点

#### 4. pitfalls（常见陷阱）

这是 Skill 最有价值的部分之一。把你踩过的坑写进去，别人就不用再踩一次。格式建议：

```
1. **POTCAR 元素顺序和 POSCAR 不一致**
   - 现象：计算启动即报错，或给出完全错误的能量
   - 原因：VASP 按 POTCAR 中元素顺序匹配 POSCAR
   - 解决：用 grep -c "TITEL" POTCAR 检查元素数，确保顺序一致
```

每条 pitfall 写清楚三件事：现象、原因、解决办法。

#### 5. 验证步骤

跑完后怎么确认结果可信？要写出可执行的验证清单：

```
- [ ] OUTCAR 中 "reached required accuracy" 出现（结构优化收敛）
- [ ] 电子步在 NELM 内收敛（看 OUTCAR 最后一圈）
- [ ] 能量为负且量级合理（对比同类体系）
- [ ] 力的残差 < EDIFFG（若设了 EDIFFG）
```

### 用 Skill评估Meta-Skill 辅助优化（开发中）

我们有自动化的 Skill 质量评估工具（skill-eval），从五个维度打分：

| 维度 | 分值 | 评什么 |
|------|------|--------|
| **触发条件精度** | 25分 | 触发关键词是否准确、有没有写清什么时候不该用 |
| **执行质量** | 35分 | 步骤是否完整、参数文档是否全、错误处理是否完善、脚本是否可靠 |
| **经济性** | 20分 | 篇幅是否精炼（越短越密越好，但要完整） |
| **可信度** | 12分 | 元数据是否齐全、引用的脚本/文件是否存在 |
| **可读性** | 8分 | 结构是否清晰、有没有示例和表格 |

用法：

```bash
# 评估单个 Skill
python <your-skill-eval-path>/scripts/skill_eval_cli.py full <your-skill-path>

# 批量评估
python <your-skill-eval-path>/scripts/skill_eval_cli.py static <your-skills-dir> --batch --brief
```

> 把路径替换成你实际的 skill-eval 安装路径和待评估 Skill 路径。

**建议的迭代节奏：**
1. 初稿改完一轮 → 跑一次评估，看五个维度哪项低
2. 针对低分项修复 → 再跑一次评估，看分数是否提升
3. 重复直到总分达到"稳定可靠"级别（见下方质量分级）

---

## 六、SKILL.md 规范

### 文件结构

一个标准的 SKILL.md 长这样：

```markdown
---
name: my-skill-name
description: Use when <触发条件>. <一句话行为描述>.
version: 1.0.0
author: 你的名字
license: MIT
metadata:
    tags: [短标签1, 短标签2]
    related_skills: [其他skill名]
---

# 标题

## Overview
一到两段：这个 Skill 做什么、为什么需要它。

## When to Use
- 触发条件（什么场景用）
- Don't use for: 什么时候不该用（负向边界）
  
## Prerequisites （如有）
1. 依赖的软件/命令 — 验证命令 + 安装方法
2. 前置产物（如上一步计算的输出文件）
> 缺失时停止执行并向用户说明。

## <具体操作章节>
- 快速参考表
- 代码块（确切命令）
- 分场景的步骤说明

## Common Pitfalls
编号列表：常见错误 + 修复方法

## Verification Checklist
- [ ] 跑完后的验证项
```

### frontmatter 要求（硬性）

frontmatter 是文件开头的 YAML 块，有严格校验：

| 字段                        | 要求         | 说明                        |
| ------------------------- | ---------- | ------------------------- |
| `name`                    | 必填         | 小写+连字符，≤64字符              |
| `description`             | 必填，≤1024字符 | 以 "Use when..." 开头，写清触发条件 |
| `version`                 | 建议填        | 版本号                       |
| `author`                  | 建议填        | 作者                        |
| `license`                 | 建议填        | 许可证                       |
| `metadata.tags`           | 建议填        | 短标签列表                     |
| `metadata.related_skills` | 建议填        | 关联的其他 Skill               |

**通用约定：**
1. 文件以 `---` 开头作为 frontmatter 标记（YAML 标准）
2. description 写清触发条件，越具体越好
3. 整个文件不宜过长，Agent 加载 Skill 时，SKILL.md 全文注入 context，需要给系统提示、对话历史、工具输出留空间（建议 8,000-15,000 字符；超长则拆分到 `references/`）

### description 怎么写

description 是 Skill 的"门面"——Agent 靠它判断要不要加载这个 Skill。

```
✅ 好的 description：
"Use when the user wants to adapt a pre-trained DPA3 model to a new downstream 
dataset. Covers single-task and multi-task fine-tuning workflows."

❌ 差的 description：
"Fine-tune DPA3 model."   （太泛，没写触发条件）
"Use when everything."    （没有边界）
```

写法公式：**Use when <触发场景>. <覆盖的范围/能力>.**

### 正文结构建议

| 章节                          | 是否必需      | 内容                     |
| --------------------------- | --------- | ---------------------- |
| `# 标题`                      | 必需        | 一级标题                   |
| `## Overview`               | 必需        | 一两段概述                  |
| `## When to Use`            | 必需        | 触发条件 + 负向边界            |
| `## Prerequisites`          | 依赖外部软件时必需 | 环境检查 + 安装方法 + 前置产物，见下文 |
| `## <操作章节>`                 | 必需        | 具体步骤、命令、参数说明           |
| `## Common Pitfalls`        | 强烈建议      | 常见错误+修复                |
| `## Verification Checklist` | 强烈建议      | 验证清单                   |

#### 前置条件与环境检查

你的 Skill 如果依赖特定软件（VASP、LAMMPS、DeePMD-kit、ASE...）或需要前置计算产物（SCF 结果、checkpoint 文件...），就必须写清楚环境要求，否则 Agent 在缺依赖的环境里盲跑会直接报错。

写清楚"怎么检查、怎么装、缺了怎么办"。例如：

```markdown
## Prerequisites

1. **VASP 可执行文件** — 验证：`command -v vasp_std`

2. **MPI 环境** — 验证：`command -v mpirun`；VASP 并行计算依赖 MPI

3. **前置产物** — DOS/BAND 需要 SCF 计算的 CHGCAR

> 如果以上任何一项缺失，停止执行并向用户说明缺了什么、怎么补。
```

三类内容：

| 内容   | 作用                | 示例                                             |
| ---- | ----------------- | ---------------------------------------------- |
| 验证命令 | Agent 能自动检测环境是否就绪 | `command -v vasp_std`、`command -v mpirun`      |
| 安装方法 | 缺了怎么装（给链接或命令）     | `pip install deepmd-kit`、见官方文档                 |
| 前置产物 | 上一步计算的输出文件        | SCF 结果（DOS/BAND 的前置）、checkpoint 文件（freeze 的前置） |

**缺失时的行为：明确写出"停止并询问"。** 不要让 Agent 猜测或用替代品——缺了 VASP 许可证就停下来告诉用户，不要自作主张换用其他计算软件。
**复杂 Skill 的自动化检查（可选）：** 如果依赖项多（3 个以上），可以写一个 `scripts/check_env.sh` 一键验证：
### 目录组织

```
<skill名>/
├── SKILL.md
├── scripts/          # 可执行脚本
├── references/       # 详细参考文档（当 SKILL.md 太长时拆出来）
├── templates/        # 输入文件模板
└── config.yaml       # 配置文件（脚本需要时）
```

**什么时候拆分 references/：** 当 SKILL.md 超过 15,000 字符时，把详细参数说明、大段示例拆到 `references/` 下，在 SKILL.md 中引用。保持 SKILL.md 本身精炼。

---

## 七、脚本封装规范

不是所有 Skill 都需要脚本。但如果你的 Skill 涉及文件转换、批量处理、数值计算等重复操作，就该封装成脚本。

### 脚本目录

```
<skill-name>/
└── scripts/
    ├── prepare_input.py    # 准备输入
    ├── parse_output.py     # 解析输出
    └── submit_job.py       # 提交计算
```

### 四条规范

#### 1. 配置用 config.yaml，不要硬编码（如有）

```python
# ✅ 正确：从脚本同级的 config.yaml 读配置
from pathlib import Path

CONFIG_PATH = Path(__file__).parent.parent / "config.yaml"

# ❌ 错误：硬编码绝对路径
CONFIG_PATH = Path.home() / ".hermes/skills/xxx/config.yaml"
```

config.yaml 示例：

```yaml
# 用户编辑这个文件
resource:
  nodes: 1
  cores_per_node: 16
  queue: "short"

limits:
  max_structures: 100    # 或 "All" 表示不限制
```

提供 `config.yaml.example` 作为模板，实际 `config.yaml` 由用户填写。

#### 2. 路径用相对路径，文档用占位符

**脚本里：** 用 `Path(__file__).parent` 相对定位，不要写死绝对路径。

**文档里：** 用 `<your-path>` 占位符，不要写死某台机器的路径。

```markdown
# ✅ 正确
运行评估：
python <your-skill-eval-path>/scripts/eval.py <your-skill-path>

# ❌ 错误
运行评估：
python ~/.hermes/skills/devel-skill/skill-eval/scripts/eval.py ~/.hermes/skills/xxx
```

在文档开头加一个路径说明：

```markdown
> **路径占位符说明：**
> - `<your-skill-path>` — 你的 Skill 安装目录
> - `<your-skill-eval-path>` — skill-eval 工具的安装目录
```

#### 3. 错误信息要可操作

```python
# ✅ 正确：告诉用户怎么修
if not CONFIG_PATH.exists():
    raise ValueError(
        f"配置文件不存在: {CONFIG_PATH}\n"
        "请复制 config.yaml.example 为 config.yaml 并填写"
    )

# ❌ 错误：用户不知道该怎么办
if not CONFIG_PATH.exists():
    raise FileNotFoundError(CONFIG_PATH)
```

#### 4. 资源限制要可配置（如有）

脚本处理文件、调 API 时，限制值不要写死，放到 config.yaml：

```yaml
# config.yaml
limits:
  content_limit: 3000        # 读取字符数上限
  batch_size: 50             # 批量处理大小
  max_structures: "All"      # "All" = 不限制
```

---

## 八、测试验证标准

### 验证清单

一个 Skill 交付前，对照这份清单逐项检查：

**结构完整性：**
- [ ] SKILL.md 存在，frontmatter 以 `---` 开头
- [ ] `name`、`description`、`version`、`author`、`license`、`metadata` 齐全
- [ ] name ≤ 64字符，小写+连字符
- [ ] description ≤ 1024字符，以 "Use when..." 开头
- [ ] 总文件不宜过长（建议 8,000-15,000）

**内容质量：**
- [ ] 有 Overview（这个 Skill 做什么）
- [ ] 有 When to Use（触发条件 + 负向边界）
- [ ] 步骤完整、按执行顺序组织
- [ ] 需要判断的参数有选择逻辑（不只是给一个硬编码值）
- [ ] 有 Common Pitfalls（至少覆盖你亲跑时踩过的坑）
- [ ] 有 Verification Checklist（跑完怎么确认结果对）

**工程规范：**
- [ ] 脚本（如有）放在 `scripts/` 下
- [ ] 脚本用相对路径定位配置文件
- [ ] 文档用 `<your-path>` 占位符，不硬编码绝对路径
- [ ] 有 `config.yaml.example`（脚本需要配置时）
- [ ] 错误信息可操作（告诉用户怎么修）

**实际验证：**
- [ ] 用一个典型体系实际跑一遍 Skill 指导的流程，能跑通
- [ ] 结果和亲跑时一致或更优

### 质量分级

| 等级 | 标准 | 状态 |
|------|------|------|
| **基础可用** | 能跑通，但参数和错误处理不完善 | 不算合格交付 |
| **稳定可靠** | 覆盖常见场景，有错误处理和 pitfalls | ✅ 合格交付 |
| **专家级** | 覆盖边界条件，有深度的参数选择逻辑 | 努力目标 |

**每个 Skill 都需要达到"稳定可靠"级别才算合格交付。**

### 用 skill-eval 评估（开发中）

```bash
# 单个评估（静态+语义+运行时，最全面）
python <your-skill-eval-path>/scripts/skill_eval_cli.py full <your-skill-path>

# 只看静态分（快，不需要 API key）
python <your-skill-eval-path>/scripts/skill_eval_cli.py static <your-skill-path>

# 批量评估某个目录下所有 Skill
python <your-skill-eval-path>/scripts/skill_eval_cli.py static <your-skills-dir> --batch --brief

# JSON 输出（方便程序处理）
python <your-skill-eval-path>/scripts/skill_eval_cli.py full <your-skill-path> --json
```

评估报告会给出五个维度的分数和改进建议。重点关注低分项，按建议修复后重新评估。

> 注：LLM 语义评分有一定随机性，分数可能小幅波动。关键决策建议跑两次取平均。

---

## 九、常见踩坑记录

这部分收集团队实际构建 Skill 时踩过的坑，持续更新。

### 通用踩坑

1. **不亲跑就动手写 Skill**
   - 现象：写出来的步骤缺胳膊少腿，参数选择没逻辑
   - 原因：没真正理解计算流程
   - 解决：先跑通一遍再写，这是铁律

2. **description 写得太泛**
   - 现象：Skill 该加载时不加载，不该加载时乱加载
   - 原因：description 没写清触发条件
   - 解决：用 "Use when..." 句式，写清什么场景用、覆盖什么

3. **只给参数值不给选择逻辑**
   - 现象：换一个体系就不适用了
   - 原因：把"我这个体系的参数"当成了"通用参数"
   - 解决：写清楚什么情况选什么值、为什么

4. **文档里硬编码绝对路径**
   - 现象：换台机器就跑不了，别人用不了
   - 解决：文档用 `<your-path>` 占位符，脚本用相对路径

5. **SKILL.md 太长不拆分**
   - 现象：一个文件几万字，Agent 加载慢，读者也累
   - 解决：超过 15,000 字符时，详细内容拆到 `references/` 下

6. **脚本里硬编码资源限制**
   - 现象：换个数据规模就跑不了或太慢
   - 解决：限制值放 config.yaml，支持 "All" 表示不限制

### 计算引擎相关踩坑

（随团队实践持续补充，以下是已知的典型坑）

**VASP：**
- POTCAR 元素顺序必须和 POSCAR 一致
- 结构优化收敛判据：看 OUTCAR 是否出现 "reached required accuracy"

**LAMMPS：**
- （待补充）

**DeePMD：**
- （待补充）

> 💡 你踩过的坑，请私信高于翔或直接补充到本指南，惠及所有人。

---

## 附录

模板：SKILL.md 骨架

```markdown
---
name: <skill-name>
description: Use when <触发条件>. <一句话行为描述>.
version: 0.1.0
author: <你的名字>
license: MIT
metadata:
	tags: [<短标签>]
    related_skills: [<关联skill>]
---

# <标题>

## Overview

<一两段：这个 Skill 做什么、为什么需要它。>

## When to Use

- <触发条件1>
- <触发条件2>

Don't use for:
- <不该用的情况1>

## Prerequisites

1. **<依赖的软件/命令>** — 验证：`<command -v xxx>`；安装：<链接或命令>
2. **<前置产物>** — 需要 <上一步的输出文件>（来自 <前置步骤名>）
 
> 如果以上任何一项缺失，停止执行并向用户说明缺了什么、怎么补。

## <操作步骤>

### Step 1: <步骤名>

<说明 + 命令>

### Step 2: <步骤名>

<说明 + 命令>

## 参数选择

| 参数 | 默认值 | 选择逻辑 |
|------|--------|---------|
| <参数名> | <值> | <什么情况选什么值> |

## Common Pitfalls

1. **<坑的标题>**
   - 现象：<错误表现>
   - 原因：<为什么出错>
   - 解决：<怎么修>

## Verification Checklist

- [ ] <验证项1>
- [ ] <验证项2>
```

---

## 反馈与贡献

- 本指南是活文档，持续更新。
- 踩坑记录、典型案例、流程改进建议，请私信高于翔或在群里反馈。
- 每个人都是 Skill 的贡献者——这是作业，也是能力建设。

**大模型写得出代码，但写不出你的经验。期待和大家一起把这件事做成。**
