# TCR_DA pMHC 可选预设

仅在 `D:\000-ic\TCR_DA_local`，或用户明确提到 TCR_DA、standard C/D、pTrp6、pMHC groove 时应用。它是项目预设，不是通用模板；使用前仍须检查当前输入是否符合以下身份合同。

## 冻结身份合同

- Protein：6069 原子；A/B/C：4309/1623/137。
- MHC-A 对齐集：Protein atom `id 1-4309 and name CA`，应为 274 个 C-alpha。
- peptide：atom `id 5933-6069`，共 137 原子、69 个重原子。
- pTrp6 是 peptide 第 6 位 Trp，不表示磷酸化。其侧链 10 个重原子 ID：`5999+6002+6003+6005+6007+6008+6010+6012+6014+6016`。
- GROMACS 导出的代表 PDB 可能没有 chain ID；这类文件使用已验证 ID，不使用 `chain C`。
- standard C 是 free-like pMHC；standard D 是 TCR-bound-source 去除 TCR 后的 bound-like pMHC。默认不重新显示 TCR。

任一原子数、映射或来源不符时停止套用本预设，回到通用流程重新核验；不要为了通过预设而修改结构。

## 轨迹抽帧的 PBC 顺序

TCR_DA 轨迹使用匹配的 Protein=6069 TPR。先单独执行 `trjconv -pbc whole`；再单独执行 `trjconv -pbc cluster -center -ur compact`，使用已核验的 Protein 作为 cluster/output group、MHC-A C-alpha 作为 center group。必须从日志确认三次实际选择均正确，不能遗漏 output group。完成 PBC 整理后再做 274 个 MHC-A C-alpha 拟合，不把拟合与 PBC 处理混成一步。

导出的 Protein 应保持 6069 原子，并通过坐标连续性或目视检查确认 peptide、groove 和蛋白链没有被周期盒切开。直接 raw/nojump 导出或只依赖 `-rmpbc` 不足以完成这个门。

## 对齐与对象语义

1. 先加载临时 full-Protein 对象，以 274 个 MHC-A C-alpha、`cycles=0` 对齐到 standard C。
2. groove 以 standard C peptide 周围 8 A 内的 MHC-A 残基闭包定义；具体 PyMOL 选择应按 residue 扩展，并验证非空。
3. `GROOVE_standard_C` 只包含上述 groove 原子，不再用“groove”名称承载完整 MHC-A。
4. `MHC_context_standard_C` 包含 standard C 的 MHC-A 减去 groove 原子。它与 `GROOVE_standard_C` 必须互斥，二者并集才构成需要显示的 MHC-A 背景。
5. 每个 peptide 比较对象只含 `id 5933-6069`。不得保留隐藏着 MHC 的 full-Protein 对象冒充 peptide。
6. 创建最终对象后删除 PyMOL 运行时临时对象；不得删除磁盘文件。

小型 C/D/轨迹帧叠加且没有 measurement 时，右侧栏保持扁平：一个 MHC context、一个 groove，以及各个真实 peptide-only 对象。不要添加 `STRUCTURES` 总组。

如果某个状态带距离测量，则只为该状态建立一个语义基础 group，将对应 peptide/局部结构对象与 measurement 放在一起；共享的 MHC context 和 groove 不重复塞进每个状态组。

## 冻结视觉基线

- 白背景。
- `MHC_context_standard_C` 使用 `gray70` 半透明 cartoon，透明度约 `0.72`。
- `GROOVE_standard_C` 使用 `gray80` 局部 surface，透明度约 `0.58`；需要连续骨架语境时可同时显示克制的 cartoon。
- peptide 只显示 `not elem H` sticks；普通 stick radius 约 `0.10`，pTrp6 侧链约 `0.24`。
- standard C 蓝色 `[0.10,0.28,0.55]`；standard D 橙色 `[0.84,0.37,0.00]`。轨迹帧可用青绿、紫色或 cividis 序列，但必须明显区分。
- 仅一套 MHC 背景；隐藏其他 MHC、chain B、水、离子和 TCR。

若添加 distance label：使用深色 label，并把 `label_outline_color` 设置成同一深色以消除可见描边，`label_shadow_mode=0`；measurement 与所属结构进入同一个基础 group。

## 项目参考

- 视觉基线：`artifacts/TCR_DA_key_baseline_v3_authoritative/02_pymol_c04_keyframes/`
- 已有对象语义示例：`artifacts/pmhc_model_qc/c04_c11_structural_overlay_v2/OPEN_C04_C11_STRUCTURES.pml`

引用前确认路径仍存在。参考只提供视觉与项目身份基线，不替代当前结构、帧时间和映射验收。
