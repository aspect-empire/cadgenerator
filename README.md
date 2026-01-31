# ShipBaseGen — MVP0（Langflow 串联 Demo）

> 目标：用 **Langflow + LLM 中枢**把“需求 -> 命令流 -> CATIA建模 -> STEP -> OCCT展示/读取”串起来。  
> 本阶段不追求全自动最优结构，只要“能跑通 + 可迭代 + 模块解耦”。

---

## 1. 本 MVP 的目标（必须做到）
1) **命令流 DSL v0**
- 定义一个最小 CAD 命令流（JSON/YAML 均可），支持：
  - 新建 Part
  - 基准草图（矩形/多段线/圆）
  - Pad/Extrude（拉伸）
  - Hole（打孔：优先实现“直孔”，线程孔可先作为字段保留但不强制实现）
  - 保存为 `.CATPart`（可选）与导出 `.step`

2) **pycatia 执行器（可运行）**
- 提供 `PyCatiaExecutor`：输入命令流 DSL，驱动 CATIA V5 自动建模（最小：草图+拉伸+孔）。
- 需要 `--dry-run`：无 CATIA 环境时也能跑测试（只输出计划与日志）。

3) **pythonocc/OCCT 预览与信息读取（可运行）**
- 提供 `OccStepViewer`：加载 STEP，显示/导出截图（截图可选），并能读取：
  - bounding box
  - 体积/表面积（若实现困难可先仅 bbox）
  - 拓扑计数（faces/edges/vertices）

4) **最小 rule 图谱 + case 图谱**
- rule 图谱：最少 10 条规则（板厚、孔边距、孔径范围、腹板高度范围、肘板最小尺寸等）
- case 图谱：最少 6 个“异形基座案例”（跨双甲板、横向固定、合并基座、开孔避让等），每个案例包含：
  - `problem_text`
  - `BaseSpec`（可简化）
  - `command_flow`（可简化）
  - `tags`

5) **Langflow 串联工作流（能在 UI 里跑通）**
- 用 Langflow 自定义组件把上述模块串起来，至少包含这些节点：
  1. TextInput（用户需求）
  2. CaseRetrieve（从 case 图谱检索相似案例）
  3. RuleLoad（加载 rule 图谱）
  4. PromptAssemble（拼提示词：需求+案例+规则+输出格式约束）
  5. LLM（输出命令流 DSL JSON）
  6. DSLValidate（JSON Schema/pydantic 校验）
  7. RuleCheck（对 BaseSpec/命令流做规则检查）
  8. PyCatiaExecute（执行并导出 STEP）
  9. OccInspect（读取 STEP 信息并输出摘要）

> 注意：本 MVP 允许“人工在 Langflow 里多次运行迭代”，不要求自动循环修复。

---

## 2. 非目标（本 MVP 不做）
- 不做 Diffusion/EBM 训练
- 不做复杂装配（CATProduct）与多零件
- 不做 CATIA <-> OCCT 的实时双向同步（只要求 STEP 级别的离线联动）
- 不追求工程强约束完备，只要能扩展

---

## 3. 推荐运行环境
- Windows 10/11（CATIA V5 + COM 自动化）
- Python 3.11（建议；Langflow 要求 Python 3.10~3.13）
- 可选：Node/npm（如果你用 Codex CLI）

---

## 4. 如何用 Codex 生成代码（推荐工作方式）
### 4.1 用 Codex CLI（可选）
> Windows 上 Codex CLI 支持是实验性的；你也可以直接用 ChatGPT / IDE 插件来跑 Codex。
- 在仓库根目录执行：`codex`
- 让 Codex 按本 README 逐条实现，并在每个里程碑后运行 tests

### 4.2 生成策略
- 先搭骨架（目录 + pydantic schema + 接口）
- 再做 dry-run 可测
- 最后做 pycatia 真机集成（用 feature flag + 环境检测）

---

## 5. 目录结构（Codex 必须创建）
