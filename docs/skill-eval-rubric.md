# Skill Evaluation Rubric

<!--
格式说明：
  - "## Dimension:" 行定义维度，weight 为该维度在百分制中的权重（0-1）
  - "### Sub-item:" 行定义子项，weight 为该子项在维度内的权重（0-1）
  - 维度内所有子项权重之和应为 1.0
  - 所有维度权重之和应为 1.0
  - content_hint:   一行，自然语言，告诉 Phase 1 LLM 去哪里找相关内容
  - content_scope:  可选，默认 skill_md；预留值：scripts / references
                    当前版本中非 skill_md 的子项会被跳过并在结果中标注
  - rubric:         多行，每行缩进 2 空格，直到下一个标题前结束
                    内容原样传给 Phase 2 LLM，只描述 ABCD 等级标准
  - 不要在 "## Dimension:" 或 "### Sub-item:" 行后加注释，会干扰解析

评分体系（来自「Skill 评估统一评分细则 v2.0」）：
  - ABCD 四等制：A=100%, B=60%, C=30%, D=0%
  - LLM 只判等级（A/B/C/D），权重计算在代码中完成
  - 子项得分 = 100 × 维度权重 × 子项权重 × 等级系数
  - 总分 = Σ(所有子项得分)，满分 100
  - 缺失即 D 等，不得分
  - 维度间正交（MECE），两两语义不重叠

附加规则：
  - 若 Skill 无任何破坏性/安全相关操作，安全性维度所有子项自动评 A 等
  - 若 Skill 不含 references/ 或 scripts/（纯文本 Skill），渐进披露子项自动评 A 等
  - 若任一维度得分率 < 30%，标注为短板维度

维度概览：
  1. 可发现性  15%  — Agent 能否在正确时机找到并加载此 Skill？
  2. 清晰度    15%  — 已有的指令是否无歧义、结构化？
  3. 完备性    20%  — 必要信息是否全部覆盖？
  4. 一致性    10%  — 已有信息之间是否自洽无矛盾？
  5. 安全性    15%  — 风险和破坏性操作是否受控？
  6. 经济性    5%   — Token 消耗是否合理、无冗余？
  7. 可移植性  10%  — 换个环境还能用吗？
  8. 可维护性  10%  — 将来容易修改和扩展吗？
-->

## Dimension: 可发现性  weight: 0.15

### Sub-item: 命名质量  weight: 0.30
content_hint: SKILL.md frontmatter 的 name 字段
content_scope: skill_md
rubric: |
  仅评估 name 字段本身的命名质量。
  A：动作导向+具体对象，一见即知功能（如 github-pr-workflow）
  B：可接受但不够具体（如 github）
  C：有命名但含义不明，需猜测才能理解功能
  D：无 name 字段，或无意义命名（helper/utils/misc）
  注意：只看 name 字段本身，不结合 description 判断。

### Sub-item: 描述准确性  weight: 0.40
content_hint: SKILL.md frontmatter 的 description 字段，以及文档中的 Overview 或 When to Use 章节
content_scope: skill_md
rubric: |
  评估 description 字段及文档中 Overview/When to Use 章节的内容质量。
  description 应同时回答"做什么"和"何时用"。
  A：精确说明做什么+何时用，含关键触发词，第三方一看即懂
  B：说明做什么+何时用，但关键词不够精确或表述笼统
  C：粗略说明做什么，但未说明何时用，缺乏关键词
  D：无 description 或极度模糊，无法判断做什么
  注意：只看 description 及 Overview/When to Use 章节的质量，不评 name 或负向边界。

### Sub-item: 负向边界声明  weight: 0.30
content_hint: SKILL.md 中 When to Use、Negative Boundaries 或类似章节，描述不适用范围
content_scope: skill_md
rubric: |
  仅评估 Skill 中"不适用范围"的显式声明。
  A：明确列举不处理场景，且与相似 Skill 有可辨识的区分标准
  B：明确列举不处理的场景类型
  C：有边界说明但模糊，无法据此排除具体场景
  D：无任何"不处理什么"的说明
  注意：只看负向边界是否显式声明，不评 name 或 description 质量。

## Dimension: 清晰度  weight: 0.15

### Sub-item: 指令语义精确性  weight: 0.50
content_hint: SKILL.md 中每条操作指令、条件语句的用词和语法
content_scope: skill_md
rubric: |
  评估每条指令/条件语句的用词和语法精度。
  A：所有指令（含条件分支）有明确主语+动词+宾语，条件均可做 Y/N 判断
  B：大部分指令可执行，个别用词模糊或条件需推测
  C：部分指令可执行，但多处用词模糊或条件需推测
  D：大量模糊动词（consider/ensure/appropriate），缺主语或宾语，条件分支无法做确定性判断
  注意：评估的是"写出来的指令是否语义清晰"，不评估"该写的指令是否都写了"（后者属完备性）。

### Sub-item: 结构化格式  weight: 0.50
content_hint: SKILL.md 的标题层级、编号步骤、代码块、分隔符等视觉组织方式
content_scope: skill_md
rubric: |
  评估文档的视觉层次和组织方式。
  A：充分利用标题层级、编号步骤、代码块、分隔符，视觉层次清晰
  B：有合理的标题层级和编号步骤，代码块使用正确
  C：有基本标题或编号，但层次不清或缺少代码块标记
  D：纯文本段落堆砌，无标题/编号/分隔
  注意：评估的是排版和视觉组织，不评估单条指令的语义精确性（后者属 2.1）。

## Dimension: 完备性  weight: 0.20

### Sub-item: 主流程覆盖  weight: 0.25
content_hint: SKILL.md 的操作章节，包含核心任务的步骤说明
content_scope: skill_md
rubric: |
  评估正常执行路径（happy path）的步骤完整性。
  A：核心流程完整、顺序准确、无遗漏
  B：基本完整，有轻微遗漏但不阻断主流程
  C：有基本步骤但存在明显遗漏，部分环节需猜测
  D：关键步骤缺失，按现有内容无法完成核心任务
  注意：只评正常路径的完整性，异常路径属 3.3。

### Sub-item: 外部依赖声明  weight: 0.20
content_hint: SKILL.md 的 Prerequisites、Requirements 或 Setup 章节
content_scope: skill_md
rubric: |
  评估运行前需要的工具/权限/环境依赖的声明完整性。
  A：所有工具/权限/环境依赖完整列出且说明获取方式
  B：列出了主要依赖，获取方式基本说明
  C：列出部分依赖，但获取方式缺失或不完整
  D：未列出任何依赖
  注意：评的是"运行前需要准备什么"（外部工具/权限/环境），不评"运行时数据长什么样"（后者属 3.4）。

### Sub-item: 异常处理指导  weight: 0.20
content_hint: SKILL.md 中关于失败场景、边界情况、错误处理的文字策略说明
content_scope: skill_md
rubric: |
  评估对失败/边界场景的文字处理策略（非示例）。
  A：系统覆盖输入缺失、格式错误、依赖不可用等典型异常，每种有明确处理分支
  B：覆盖了部分常见错误，有初步处理指导
  C：提及少数常见错误，但处理方式笼统
  D：完全未提及任何失败/边界情况的处理方式
  注意：评的是"文字层面有没有异常处理策略"，不评"有没有具体示例来演示"（后者属 3.5）。

### Sub-item: 输入输出契约  weight: 0.15
content_hint: SKILL.md 中输入输出的数据结构定义、类型说明、约束条件
content_scope: skill_md
rubric: |
  评估输入输出的数据结构定义。
  A：完整的输入/输出规格（类型、默认值、约束、校验规则）
  B：有主要字段定义，但缺少部分约束或默认值
  C：有大致描述但缺字段约束或类型
  D：未定义输入输出的任何规格
  注意：评的是"运行时数据长什么样"（字段和类型），不评"运行前需要准备什么"（后者属 3.2）。

### Sub-item: 示例完备度  weight: 0.20
content_hint: SKILL.md 中的示例、用例、演示部分
content_scope: skill_md
rubric: |
  评估具体示例的数量和场景覆盖。
  A：正例+反例+边界用例均覆盖，参数完整，与描述一致
  B：有正向示例，但反例或边界用例不全
  C：仅有一个正向示例，缺反例或边界用例
  D：无任何示例
  注意：只评估示例本身的质量，不评估异常处理策略文字说明（后者属 3.3）。

## Dimension: 一致性  weight: 0.10

### Sub-item: 指令无矛盾  weight: 0.60
content_hint: SKILL.md 中任意两条指令之间的逻辑关系，检查是否存在语义冲突
content_scope: skill_md
rubric: |
  检测任意两条指令之间是否存在语义冲突。
  A：全部指令逻辑自洽；若有潜在冲突场景，已给出明确优先级排序
  B：整体自洽，个别边缘指令略有矛盾
  C：存在边缘指令矛盾，无消解机制
  D：同一场景存在多处直接冲突的指令，且无消解机制
  注意：这是负面检查（找矛盾），与 4.2 的正面检查（看术语统一）不同。

### Sub-item: 术语一致性  weight: 0.40
content_hint: SKILL.md 全文用词，检查同一概念是否使用同一术语
content_scope: skill_md
rubric: |
  检测全文用词是否对同一概念使用同一术语。
  A：全文术语严格统一，有术语表或别名说明
  B：术语基本一致，偶尔混用不影响理解
  C：偶尔混用，部分造成理解困难
  D：同一概念多处使用不同名称，造成混淆
  注意：这是正面检查（看词汇统一），与 4.1 的负面检查（找指令矛盾）不同。

## Dimension: 安全性  weight: 0.15

### Sub-item: 意图拒绝边界  weight: 0.25
content_hint: SKILL.md 中关于拒绝恶意/有害请求的安全声明和规则
content_scope: skill_md
rubric: |
  评估对恶意/有害请求的拒绝规则（INBOUND 方向）。
  适用范围：Skill 因功能需要而声明 API Key / 网络访问需求，不扣分。
  若 Skill 无任何破坏性/安全相关操作，安全性维度全部子项自动评 A 等。
  A：明确列举需拒绝的意图场景，给出拒绝模板
  B：有明确的拒绝规则，但场景列举不够全面
  C：有简单安全声明但边界模糊
  D：Skill 可触达有害操作但无任何拒绝规则
  注意：评的是"什么请求该拒绝"，不评"合法操作如何安全执行"（后者属 5.2）。

### Sub-item: 破坏性操作约束  weight: 0.25
content_hint: SKILL.md 中涉及删除、推送、发送等不可逆操作时的确认/保护/回滚说明
content_scope: skill_md
rubric: |
  评估对合法但不可逆操作的保护（EXECUTION 方向）。
  若 Skill 无任何破坏性/安全相关操作，安全性维度全部子项自动评 A 等。
  A：不可逆操作有明确确认步骤或回滚方案
  B：有确认步骤，但部分操作缺少保护
  C：有提醒但无强制保护机制
  D：涉及不可逆操作无任何确认/保护
  注意：评的是"合法操作如何安全执行"，不评"什么请求该拒绝"（后者属 5.1）。

### Sub-item: 凭据安全管理  weight: 0.25
content_hint: SKILL.md 中关于 API Key、Token、密码等凭据的传入和存储方式说明
content_scope: skill_md
rubric: |
  评估凭据的传入和存储安全（CREDENTIALS IN 方向）。
  若 Skill 无任何破坏性/安全相关操作，安全性维度全部子项自动评 A 等。
  A：凭据通过环境变量/配置注入，明确不记录到日志
  B：凭据通过环境变量注入，但日志处理不明确
  C：凭据作为参数但未说明安全存储方式
  D：在文档/脚本中硬编码凭据或敏感信息
  注意：评的是"凭证如何进入系统"，不评"敏感数据如何离开系统"（后者属 5.4）。

### Sub-item: 输出脱敏规则  weight: 0.25
content_hint: SKILL.md 中关于输出内容的敏感信息过滤、脱敏/匿名化规则
content_scope: skill_md
rubric: |
  评估输出中的敏感信息过滤（DATA OUT 方向）。
  若 Skill 无任何破坏性/安全相关操作，安全性维度全部子项自动评 A 等。
  A：明确要求脱敏/匿名化，定义过滤规则
  B：有基本脱敏要求，但规则不够具体
  C：有提醒但无具体过滤策略
  D：输出可能含敏感内容但无任何过滤规则
  注意：评的是"敏感数据如何离开系统"，不评"凭证如何进入系统"（后者属 5.3）。

## Dimension: 经济性  weight: 0.05

### Sub-item: 绝对长度  weight: 0.35
content_hint: SKILL.md 主文件的总行数和内容量
content_scope: skill_md
rubric: |
  评估主文件行数和 token 数（纯量度判断）。
  A：≤500 行且 ≤5000 token
  B：略超标但有组织，可接受
  C：明显超标（300-500 行）但有一定组织
  D：>500 行或 token 严重超标
  注意：只评绝对量（多大），不评内容是否值得（后者属 6.2）。

### Sub-item: 信息密度  weight: 0.35
content_hint: SKILL.md 各段内容的必要性和信息量
content_scope: skill_md
rubric: |
  评估每段内容的必要性（质量判断）。
  A：每段都有不可替代的信息量——"删掉这段会出错吗？"→ 会
  B：有少量解释性冗余，整体密度可接受
  C：有较多解释性冗余，信息密度偏低
  D：大量填充/通用知识，Agent 不看也不影响执行
  注意：评的是内容密度（值不值），不是绝对长度（多大）。短文件可能冗余，长文件可能每句都必要。

### Sub-item: 渐进披露  weight: 0.30
content_hint: SKILL.md 主文件与 references/ 目录的内容分布策略，以及引用关系说明
content_scope: skill_md
rubric: |
  评估主文件 vs references/ 的内容分布策略。
  若 Skill 不含 references/ 或 scripts/（纯文本 Skill），此子项自动评 A 等。
  A：详细内容正确使用 references/，"何时读哪个文件"有明确说明
  B：有合理的拆分，引用关系基本清楚
  C：有部分拆分但引用关系不清
  D：所有内容堆在主文件，未做拆分
  注意：评的是内容放哪里（组织结构），不是内容值不值（后者属 6.2）。

## Dimension: 可移植性  weight: 0.10

### Sub-item: 值外部化  weight: 0.50
content_hint: SKILL.md 中的路径、URL、阈值、模板等可变值是否提取为配置或变量
content_scope: skill_md
rubric: |
  评估所有可变值（路径/URL/阈值/模板）是否提取为配置或变量。
  A：全部可变值通过配置区块/环境变量/占位符注入，集中管理
  B：多数可变值已外部化，个别遗漏
  C：部分值提取为配置或占位符，关键值仍部分硬编码
  D：关键值硬编码在正文（含绝对路径、固定 URL、魔法数字）
  注意：评的是值的抽象程度（技术层面），不评跨平台运行能力（后者属 7.2）。

### Sub-item: 跨平台兼容  weight: 0.50
content_hint: SKILL.md 中关于操作系统、工具链的假设和兼容性说明
content_scope: skill_md
rubric: |
  评估是否能在多个平台上运行，或明确标注支持范围。
  A：跨平台可用，或明确标注"支持 X，不支持 Y"及原因
  B：有平台假设但提供了部分兼容说明
  C：有平台假设但无兼容说明
  D：假设特定 OS/工具链且无替代方案或说明
  注意：评的是跨平台运行能力（架构层面），不评值的抽象程度（后者属 7.1）。

## Dimension: 可维护性  weight: 0.10

### Sub-item: 模块化结构  weight: 0.40
content_hint: SKILL.md 的功能拆分方式，各章节/子模块是否独立可修改
content_scope: skill_md
rubric: |
  评估功能是否拆分为独立可修改的单元。
  A：独立功能拆分为子模块/子提示，单一职责，修改局部化
  B：有合理的分段，部分逻辑可独立修改
  C：有分段但关键逻辑耦合，改一处需动多处
  D：所有内容堆在一起，无分段
  注意：评的是结构拆分（骨架），不评文字注释（后者属 8.2）也不评版本追踪（后者属 8.3）。

### Sub-item: 注释与文档  weight: 0.35
content_hint: SKILL.md 中的注释、设计意图说明、决策理由
content_scope: skill_md
rubric: |
  评估是否解释了设计意图和决策理由。
  A：解释"为什么这么做"+设计权衡，有完整的使用/修改指南
  B：有部分"为什么这么做"的解释，但不完整
  C：有注释但只说"做了什么"，缺设计意图
  D：无注释或注释与内容无关
  注意：评的是文字注释（血肉），不评结构拆分（后者属 8.1）也不评版本追踪（后者属 8.3）。

### Sub-item: 版本与变更追踪  weight: 0.25
content_hint: SKILL.md frontmatter 或文档中的版本号、CHANGELOG、更新说明
content_scope: skill_md
rubric: |
  评估是否有版本号和变更记录。
  A：语义化版本+CHANGELOG 或更新说明
  B：有版本号和简单的变更记录
  C：有版本号但无变更记录
  D：无版本信息
  注意：评的是版本追踪（时间线），不评结构拆分（后者属 8.1）也不评文字注释（后者属 8.2）。
