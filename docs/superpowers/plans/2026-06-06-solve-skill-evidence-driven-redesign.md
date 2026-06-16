# solve-skill 证据驱动重构 实现计划

> **面向 AI 代理的工作者：** 必需子技能：使用 superpowers:subagent-driven-development（推荐）或 superpowers:executing-plans 逐任务实现此计划。步骤使用复选框（`- [ ]`）语法来跟踪进度。

**目标：** 将 `skills/solve-skill/SKILL.md` 从“读上下文 + 判断状态”的建议式 skill 重写为“状态路由 + 证据驱动执行控制”的硬规则 skill，并新增标准任务执行合同模板。

**架构：** 保留 `solve-skill` 的外层状态判断与多步骤流程图契约，但把主方法论切换为 `Confirm → Locate → Plan → Implement → Review → Validate → Deliver`。新增 `docs/skill/evidence-driven-task-template.md` 作为复杂任务模板，并在 `SKILL.md` 中定义 `docs/skill/tasks/` 与 `docs/skill/flows/` 的职责边界与分层使用规则。

**技术栈：** Markdown、PowerShell、ripgrep。

---

## 文件结构

- 修改：`skills/solve-skill/SKILL.md`
  - 重写 skill 目标、状态路由、项目查证、证据驱动阶段机、流程图契约、提问机制、实现边界、Review / Validate / Deliver 规则。
- 创建：`docs/skill/evidence-driven-task-template.md`
  - 承载标准 10 节任务执行合同模板，作为复杂任务和高风险任务的标准落盘格式。
- 创建：`docs/skill/tasks/.gitkeep`
  - 让 `docs/skill/tasks/` 目录被版本控制跟踪，作为复杂任务运行时实例文档目录。

---

### 任务 1：重写 skill 顶层定位与阶段机

**文件：**
- 修改：`skills/solve-skill/SKILL.md`

- [ ] **步骤 1：编写失败的内容验证**

运行：

```powershell
rg -n "Evidence-Driven|Confirm → Locate → Plan → Implement → Review → Validate → Deliver|没有合同，不允许查证" skills\solve-skill\SKILL.md
```

预期：无匹配，命令退出码为 `1`。

- [ ] **步骤 2：重写 skill 目标与项目查证章节**

将文件头部到“核心原则”前的定位改为证据驱动版本，至少包含以下内容：

```markdown
# Evidence-Driven Execution Controller Skill

## 目标

该 skill 的目标不是让 Agent 依赖模型记忆完成任务，而是要求 Agent 基于项目证据链完成确认、设计、实现、复核、验证和交付。

AI 面对用户请求时，不能直接假设用户已经表达清楚，也不能直接进入执行。AI 必须先判断任务状态，再根据状态决定是否切入证据驱动流水线。

---

# 项目查证（必须）

## 前置步骤

AI 在处理任何任务前，必须先查证项目证据，而不是只读取摘要文档：

1. 读取 `docs/skill/project-context.md` 作为背景证据来源之一
2. 查找 `docs/skill/` 下其他相关文档
3. 查找项目内已有 Controller / Service / DTO / Type / Entity / Enum / 类似实现
4. 查找项目内已有依赖用法与版本约束

## 核心规则

- `project-context.md` 只是证据来源之一，不是实现依据的替代品。
- 没有代码级证据，不允许进入实现。
- 查证结论必须在后续 Evidence Map 中显式记录。
```

- [ ] **步骤 3：重写核心原则为状态路由 + 证据驱动**

将原“核心原则”改为以下骨架，并补齐文字：

```markdown
# 核心原则

## 1. 先判断任务状态，再选择执行管道

外层状态判断只负责路由，不负责完整实现方法论。

## 2. 方案设计、执行实现、验证测试、总结沉淀必须切入证据驱动流水线

统一流水线：

```text
Confirm → Locate → Plan → Implement → Review → Validate → Deliver
```

## 3. 没有证据，不允许实现

所有新增结构必须先查证项目内是否已有等价能力。

## 4. 多步骤任务设计阶段必须先有流程图契约

流程图确认前，不允许进入 Implement。
```

- [ ] **步骤 4：新增总铁律**

在原则章节末尾加入：

```markdown
## 5. 总铁律

```text
没有合同，不允许查证。
没有证据，不允许计划。
没有计划，不允许实现。
没有复核，不允许验证。
没有验证，不允许交付。
```
```

- [ ] **步骤 5：运行内容验证**

运行：

```powershell
rg -n "Evidence-Driven|Confirm → Locate → Plan → Implement → Review → Validate → Deliver|没有合同，不允许查证|没有证据，不允许计划|项目查证（必须）" skills\solve-skill\SKILL.md
```

预期：输出至少 5 条匹配。

- [ ] **步骤 6：Commit**

```bash
git add skills/solve-skill/SKILL.md
git commit -m "feat: rewrite solve skill around evidence driven stages"
```

---

### 任务 2：重写确认规则与流程图契约章节

**文件：**
- 修改：`skills/solve-skill/SKILL.md`

- [ ] **步骤 1：编写失败的内容验证**

运行：

```powershell
rg -n "Requirement Contract|阻塞性未确认信息|设计阶段必需流程图|流程图不是补充材料|流程图确认前，不允许进入 Implement" skills\solve-skill\SKILL.md
```

预期：无匹配，命令退出码为 `1`。

- [ ] **步骤 2：重写交互规则为确认规则**

将现有“交互规则（硬性）”改为“确认规则（硬性）”，至少包含：

```markdown
# 确认规则（硬性）

## Confirm：Requirement Contract

必须确认：
- 目标
- 包含范围 / 不包含范围
- 参与角色
- 验收标准
- 未确认信息
- 默认假设

以下情况不允许进入 Locate：
- 目标无法一句话说清
- 验收标准不可判断
- 存在阻塞性未确认信息
- 默认假设会影响整体方案正确性
```

- [ ] **步骤 3：保留并重写提问机制为阶段门槛**

在同一章节中保留跨运行时提问兼容逻辑，但要改成门槛式表述：

```markdown
## 提问与确认机制

当 AI 需要向用户确认阻塞性问题、流程图契约或图外动作时，必须优先使用当前环境可用的结构化提问工具。

- Claude 环境如提供 `AskUserQuestion`，优先使用 `AskUserQuestion`
- Codex 环境如提供 `request_user_input` 或等价工具，优先使用该工具
- 没有结构化提问工具时，允许输出固定文本选项兜底，但必须立即停止等待用户回复

如果确认没有完成，不允许进入下一阶段。
```

- [ ] **步骤 4：重写流程图章节为设计阶段硬门槛**

将“业务流程图环节”改名为“设计阶段必需流程图”，并保留现有模板、状态双写、节点对照协议、异常分支、文件回写规则，同时在章节开头加入：

```markdown
# 设计阶段必需流程图

对于多步骤任务，流程图不是补充材料，而是设计阶段的必需契约。

进入实现前必须完成并确认：
- Mermaid 业务流程图
- 节点清单
- 初始变更日志

以下情况不允许进入 Implement：
- 多步骤任务未画流程图
- 流程图未获得确认
- 流程图未覆盖完整业务闭环
- 节点依赖关系不清楚
- 技术子步骤填不出来
```

- [ ] **步骤 5：运行内容验证**

运行：

```powershell
rg -n "Requirement Contract|阻塞性未确认信息|设计阶段必需流程图|流程图不是补充材料|流程图未获得确认|节点对照块协议" skills\solve-skill\SKILL.md
```

预期：输出至少 6 条匹配。

- [ ] **步骤 6：Commit**

```bash
git add skills/solve-skill/SKILL.md
git commit -m "feat: turn confirmation and flowchart into hard gates"
```

---

### 任务 3：新增 Locate / Plan / Implement / Review / Validate / Deliver 规则

**文件：**
- 修改：`skills/solve-skill/SKILL.md`

- [ ] **步骤 1：编写失败的内容验证**

运行：

```powershell
rg -n "Locate：Evidence Map|Plan：Implementation Plan|Implement：受控编码|Review：Diff Review|Validate：Verify Result|Deliver：Delivery Report" skills\solve-skill\SKILL.md
```

预期：无匹配，命令退出码为 `1`。

- [ ] **步骤 2：新增六阶段规则章节**

在流程图章节之后新增以下六个主段落，每段都包含目标、必需产物、核心规则、停止条件：

```markdown
## Locate：Evidence Map
- 目标：查证项目内已有能力、边界与复用点
- 必需产物：已有入口、已有 Service / Repository / QueryService、已有 DTO / Type / Entity / Enum、类似实现、API / 依赖版本确认、证据结论
- 核心规则：没有证据，不允许实现；所有新增结构必须先查证项目内是否已有等价能力
- 停止条件：没有 Evidence Map；关键依赖未查证；新增对象与现有能力重叠但未说明原因；证据不足以支撑策略

## Plan：Implementation Plan
- 目标：在编码前固定实现边界、风险与验证方式
- 必需产物：实现策略、计划修改文件、计划新增代码、计划复用代码、protected paths、forbidden actions、风险分析、Validation Plan
- 核心规则：只允许修改计划内文件；计划外文件如需修改必须先更新计划
- 停止条件：没有 Validation Plan；没有 protected paths；计划范围超过 Requirement Contract

## Implement：受控编码
- 目标：在既定计划和流程图契约下进行受控实现
- 核心规则：只按计划修改；每完成一个节点都必须输出对照块并回写流程图文件；发现计划外文件需要修改时先更新计划
- 停止条件：需要修改数据库结构；需要修改权限模型；需要修改跨服务接口契约；现有 Service 无法满足核心能力；同一问题连续修复 3 次仍失败

## Review：Diff Review
- 目标：检查实际改动是否符合合同、证据和计划
- 必需产物：实际修改文件、实际复用内容、实际新增内容、Diff 检查结果
- 必查项：是否满足需求契约、是否复用已有能力、是否存在重复 DTO / Wrapper、是否违反模块边界、是否存在无关改动
- 停止条件：实际修改文件超出计划；流程图节点完成情况与实际 diff 对不上

## Validate：Verify Result
- 目标：用命令级验证结果证明代码有效
- 必需产物：验证项、验证命令、通过 / 未通过、失败说明、失败修复记录
- 核心规则：不能只写“已验证”；必须给出命令和结果；验证失败后只能针对失败点修复
- 停止条件：没跑验证命令；验证结果未记录；类型检查 / lint / 测试未通过

## Deliver：Delivery Report
- 目标：用明确、诚实、可审计的方式完成最终交付
- 必需产物：本次完成内容、本次未包含内容、关键实现说明、复用说明、验证说明、剩余风险、后续建议
- 停止条件：没有验证说明；没有剩余风险；把默认假设写成已确认事实；把未完成工作包装成已完成
```

- [ ] **步骤 3：重写通用处理流程为路由 + 阶段机**

将原“通用处理流程”替换为：

```text
1. 查证项目背景和代码级证据
2. 判断任务状态
3. 想法阶段 / 需求确认阶段 → 继续澄清
4. 方案设计阶段 / 执行实现阶段 / 验证测试阶段 / 总结沉淀阶段 → 切入证据驱动流水线
5. Confirm：形成 Requirement Contract
6. Locate：形成 Evidence Map
7. Plan：形成 Implementation Plan
8. Implement：按计划受控编码
9. Review：复核 diff 是否符合合同
10. Validate：运行自动验证并记录结果
11. Deliver：输出交付说明、剩余风险和后续建议
```

- [ ] **步骤 4：运行内容验证**

运行：

```powershell
rg -n "Locate：Evidence Map|Plan：Implementation Plan|Implement：受控编码|Review：Diff Review|Validate：Verify Result|Deliver：Delivery Report|切入证据驱动流水线" skills\solve-skill\SKILL.md
```

预期：输出至少 7 条匹配。

- [ ] **步骤 5：Commit**

```bash
git add skills/solve-skill/SKILL.md
git commit -m "feat: add evidence driven execution gates"
```

---

### 任务 4：新增模板文档与任务目录约束

**文件：**
- 创建：`docs/skill/evidence-driven-task-template.md`
- 创建：`docs/skill/tasks/.gitkeep`
- 修改：`skills/solve-skill/SKILL.md`

- [ ] **步骤 1：编写失败的目录与文件验证**

运行：

```powershell
Test-Path docs\skill\evidence-driven-task-template.md
Test-Path docs\skill\tasks\.gitkeep
```

预期：两条命令都输出 `False`。

- [ ] **步骤 2：创建任务模板文档**

创建 `docs/skill/evidence-driven-task-template.md`，文件头部和前两个主段至少写成：

````markdown
# 证据驱动任务执行模板

> 适用于复杂任务、多步骤任务和高风险任务。

## 1. Requirement Contract

### 1.1 目标

### 1.2 需求范围

### 1.3 参与角色

### 1.4 验收标准

### 1.5 未确认信息

### 1.6 默认假设

## 2. Evidence Map

### 2.1 已有入口

### 2.2 已有 Service / Repository / QueryService

### 2.3 已有 DTO / Type / Entity / Enum

### 2.4 类似实现参考

### 2.5 API / 依赖版本确认

### 2.6 证据结论
````

然后继续补齐完整 10 节模板，包括：

- Requirement Contract
- Evidence Map
- Implementation Plan
- Risk Review
- Validation Plan
- Diff Review
- Verify Result
- Delivery Report
- Agent 执行约束

- [ ] **步骤 3：创建任务目录占位文件**

创建空文件：

```text
docs/skill/tasks/.gitkeep
```

- [ ] **步骤 4：在 `SKILL.md` 中新增文档结构设计与复杂度分层**

在 `SKILL.md` 中新增：

```markdown
## 文档结构设计

- `skills/solve-skill/SKILL.md`：定义执行规则
- `docs/skill/evidence-driven-task-template.md`：定义标准任务执行合同模板
- `docs/skill/tasks/YYYY-MM-DD-<task-topic>.md`：复杂任务的运行时实例文档
- `docs/skill/flows/YYYY-MM-DD-<topic>-flow.md`：多步骤任务流程图契约

## 复杂度分层

- 简单任务：可以缩写任务文档，但仍需最小 Requirement Contract、Evidence Map、Validate 记录
- 多步骤或跨模块任务：必须完整使用任务模板，并且必须有流程图文件
- 高风险任务：完整任务模板 + 完整流程图 + 全量验证记录
```

- [ ] **步骤 5：运行内容验证**

运行：

```powershell
Test-Path docs\skill\evidence-driven-task-template.md
Test-Path docs\skill\tasks\.gitkeep
rg -n "文档结构设计|evidence-driven-task-template|docs/skill/tasks/YYYY-MM-DD-<task-topic>.md|复杂度分层" skills\solve-skill\SKILL.md
```

预期：
- 前两条命令输出 `True`
- `rg` 输出至少 4 条匹配

- [ ] **步骤 6：Commit**

```bash
git add docs/skill/evidence-driven-task-template.md docs/skill/tasks/.gitkeep skills/solve-skill/SKILL.md
git commit -m "feat: add evidence driven task template and task docs rules"
```

---

### 任务 5：删除旧冲突表述并完成整体收尾

**文件：**
- 修改：`skills/solve-skill/SKILL.md`

- [ ] **步骤 1：编写失败的冲突表述验证**

运行：

```powershell
rg -n "项目上下文读取（必须）|先做再问|该 skill 用于指导 AI 按照正确的问题解决方法完成任务|读取 `docs/skill/project-context.md` — 获取项目技术栈、架构、编码规范" skills\solve-skill\SKILL.md
```

预期：输出旧表述匹配。

- [ ] **步骤 2：删除或改写旧冲突内容**

完成以下修改：

- 将“项目上下文读取（必须）”整体替换为“项目查证（必须）”
- 删除“先做再问”主段，或改写为不与证据驱动主线冲突的窄范围辅助规则
- 改写开头目标表述，不再使用“按照正确的问题解决方法”这种泛化措辞
- 删除把 `project-context.md` 写成主要实现依据的文字

至少替换为：

```markdown
- `project-context.md` 是背景证据来源之一，不是实现依据的替代品。
- 简单任务可以快推进，但仍必须满足最小合同、最小证据和最小验证。
```

- [ ] **步骤 3：运行整体验证**

运行：

```powershell
rg -n "项目查证（必须）|Evidence-Driven|Confirm → Locate → Plan → Implement → Review → Validate → Deliver|Requirement Contract|Evidence Map|Implementation Plan|Diff Review|Verify Result|Delivery Report|文档结构设计|复杂度分层" skills\solve-skill\SKILL.md
rg -n "项目上下文读取（必须）|先做再问" skills\solve-skill\SKILL.md
```

预期：
- 第一条命令输出多条匹配，覆盖新结构
- 第二条命令无匹配，退出码为 `1`

- [ ] **步骤 4：Commit**

```bash
git add skills/solve-skill/SKILL.md
git commit -m "refactor: finalize evidence driven solve skill"
```

---

### 任务 6：最终验证与规格覆盖检查

**文件：**
- 验证：`skills/solve-skill/SKILL.md`
- 验证：`docs/skill/evidence-driven-task-template.md`
- 验证：`docs/skill/tasks/.gitkeep`
- 参考：`docs/superpowers/specs/2026-06-06-solve-skill-evidence-driven-redesign-design.md`

- [ ] **步骤 1：运行关键需求覆盖验证**

运行：

```powershell
rg -n "状态路由|Confirm：Requirement Contract|Locate：Evidence Map|Plan：Implementation Plan|Implement：受控编码|Review：Diff Review|Validate：Verify Result|Deliver：Delivery Report|流程图不是补充材料|没有证据，不允许实现|没有验证，不允许交付" skills\solve-skill\SKILL.md
```

预期：至少输出 10 条匹配。

- [ ] **步骤 2：运行模板与目录验证**

运行：

```powershell
Test-Path docs\skill\evidence-driven-task-template.md
Test-Path docs\skill\tasks\.gitkeep
Get-Item docs\skill\tasks\.gitkeep | Select-Object Length
```

预期：
- 前两条输出 `True`
- `.gitkeep` 长度为 `0`

- [ ] **步骤 3：运行辅助一致性验证**

运行：

```powershell
rg -n "Requirement Contract|Evidence Map|Implementation Plan|Risk Review|Validation Plan|Diff Review|Verify Result|Delivery Report|Agent 执行约束" docs\skill\evidence-driven-task-template.md
```

预期：输出 9 条或更多匹配，覆盖模板主段。

- [ ] **步骤 4：检查 Git 环境可用性**

运行：

```powershell
Test-Path .git
```

预期：根据当前环境大概率输出 `False`。如果是 `False`，执行者必须在结果中明确说明无法 commit、无法用 `git diff/status` 作为验证手段。

- [ ] **步骤 5：规格覆盖自检**

逐项确认：

```text
背景与目标：任务 1、任务 5 覆盖
重构原则：任务 1、任务 5 覆盖
总体结构：任务 1、任务 3 覆盖
Confirm / 流程图 / Locate / Plan / Implement / Review / Validate / Deliver：任务 2、任务 3 覆盖
文档结构设计：任务 4 覆盖
复杂度分层：任务 4 覆盖
迁移策略：任务 5 覆盖
成功标准：任务 6 覆盖
```

- [ ] **步骤 6：Commit**

如步骤 1-5 发现措辞缺口，先修正文档并重新运行对应验证。全部通过后运行：

```bash
git add skills/solve-skill/SKILL.md docs/skill/evidence-driven-task-template.md docs/skill/tasks/.gitkeep
git commit -m "test: verify evidence driven solve skill redesign"
```
