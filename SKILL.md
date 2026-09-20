---
name: icpml-skill
description: Use only when iC explicitly asks to create, revise, package, run, audit, or troubleshoot a PyMOL .pml file, PyMOL commands or session, a PyMOL render, or the PyMOL object panel. Do not invoke merely because a task discusses molecular structures, MD, binding sites, or visualization; use ic-structure-view for view planning.
---

# iC PyMOL/PML

## 定位

这是 PyMOL/PML 的实现与验收层，不是通用结构可视化技能。

- 新建或重做结构视图时，先应用 `ic-structure-view` 确定背景、关注对象、相互作用区域、比较状态和视觉层级，再用本技能落实为 PyMOL 对象与命令。
- 仅排查已有 PML 的语法、路径、对象、选择或渲染错误时，可以单独使用本技能。
- 不因任务提到蛋白、MD、结合位点、结构叠合或“可视化”而自动扩大为 PML 工作。

不得假定链名、残基编号、序列长度、配体名称、原子编号或蛋白必然带 peptide。项目固定值只能放在明确匹配的条件预设中。

## 实现流程

1. 明确要创建、修订、审计还是排错 PML，以及预期交付物和现有视图方案。
2. 检查实际输入结构的对象、链、残基、原子、altloc 和氢原子情况；从文件和上下文推导选择，不凭文件名猜测。
3. 将已确定的视觉角色落实为语义真实、尽量互斥的 PyMOL 对象，完成对齐、表示、颜色、镜头和对象栏整理。
4. 清理临时对象和选择，生成版本化 PML，并按实际可用条件完成静态或真实渲染验收。

## 选择与对象真实性

- 对象名必须准确描述实际原子内容；不能加载完整复合物后仅靠 `hide` 冒充 peptide-only、ligand-only 或 site-only 对象。
- 短 peptide、小分子、辅因子或底物需要完整 sticks 时，通常选择 `not elem H`；大分子只落实视图方案要求的局部原子，不默认整条展开。
- 用户要求某个 groove、domain 或 site 单列时，其对象只含该区域；同层其他展示对象默认排除这些原子，除非视图方案明确要求重叠。
- 可以使用临时 full-structure 对象完成对齐，再通过 `create` 生成最终子集，并 `delete` 运行时临时对象。这里的 `delete` 绝不删除磁盘源文件。
- 不残留无用途的临时对象、named selections、空 group 或 measurement。

## 对象栏与测量

- 小型比较默认保持有序的扁平对象栏，不创建无意义的 `STRUCTURES`、`ALL` 等总组。
- 只有在状态很多、真实层级能降低混乱，或需要把 measurement 绑定到结构时才建立语义 group。
- 每个 distance measurement 必须归属于对应结构，并与结构放入同一简洁基础 group，使用户能一次显示或隐藏结构、虚线和数字。
- measurement 名称要表达所属结构和测量含义，不使用 `dist01` 一类无语义名称。
- 白背景下数字使用深色高对比文本；需要无描边效果时，将 `label_outline_color` 设为与 `label_color` 相同并关闭 label shadow。
- 若只需数值而不需图上虚线，使用 `get_distance` 或离线计算，不创建多余 measurement。

## 轨迹结构进入 PyMOL

从轨迹提取 Protein 或复合物时，必须使用匹配拓扑先处理 PBC，不能直接把 raw XTC/TRR 抽帧交给 PyMOL。通常先使分子完整（whole/molecule repair）；多链复合物可能分散到不同盒像时，再以核验过的完整复合物分组执行 cluster/center，最后才在科学上合适的共同原子集上对齐。

PBC 整理与旋转拟合必须保持为可核查的独立步骤。链字段缺失时，使用当前体系已经核验的 atom ID、residue mapping 或拓扑选择，不硬套 `chain`。

## 交付与验收

- 保护原件；修订写入版本化 sibling，不覆盖已验收文件。
- 默认交付版本化 `.pml`。输入位于同一稳定目录时使用相对路径；只有来源分散或路径不可移植时才建立便携目录，并只复制实际用到的 PDB/GRO 等轻量结构，不复制大型轨迹。
- **Source QC**：核对结构来源、帧时间、匹配拓扑、PBC/对齐谱系和原件未改变。轨迹抽帧还要确认实际 center/cluster/output group、Protein 原子数，并至少执行一次坐标连续性或目视检查。
- **Static QC**：检查 load 目标、相对路径、选择引用、非空对象、对象真实原子组成、必要的互斥关系、group 成员和最终对象名单。
- **Render QC**：只有在真实 PyMOL 中无加载或命令错误并完成目视检查后才可 PASS；没有 PyMOL 时明确写 `STATIC PASS / RENDER NOT_RUN`。
- 只做能捕获错误结果的必要检查，不为简单视图扩展成大型验证框架。

## 项目预设路由

仅当工作区或用户上下文明示 TCR_DA、pMHC、standard C/D 或 pTrp6 时，读取 [TCR_DA pMHC 预设](references/tcr-da-pmhc.md)。预设中的固定编号和颜色不得泄漏到其他项目。

## 禁止模式

| 禁止 | 应改为 |
|---|---|
| 普通结构可视化讨论也调用 PML 实现层 | 仅用 `ic-structure-view` 进行规划 |
| 固定假设“蛋白 + peptide” | 从实际组成和已确认的视图角色生成选择 |
| 在通用流程写死 chain/resi/id | 当前结构核验后再生成选择 |
| `peptide` 对象仍含完整 Protein | 对齐后创建真实 peptide-only 对象 |
| 从 raw 轨迹直接提 Protein 后加载 | 匹配拓扑下先 whole，必要时 cluster/center，核验后再拟合和导出 |
| 少量对象仍套一个大总组 | 默认扁平；只为真实层级或测量绑定分组 |
| 隐藏结构后距离数字仍悬空 | 结构与 measurement 放进同一基础 group |
| 静态脚本通过就称视觉验收 | 明确区分 STATIC 与 RENDER |
