# MatBenchとは
- 多様な固体材料の性質を予測する最先端の機械学習アルゴリズムのベンチマークを自動化するためのリーダーボード
- MatbenchはMaterials Projectによってホストおよび管理されている
- 材料科学のためのImageNetであり、ベンチマークおよび公平な比較のために編集された13の教師あり学習用のタスクのセット  
- 事前にクリーニングされており、すぐに使用できる状態
- 提出するためには論文（要査読）やJupyterNotebookにて裏付けが必要


# データについて

| Task name            | Task type/input       | Target column (unit)        | Samples  | Score |
|----------------------|-----------------------|-----------------------------|----------|---------------------------------------------------|
| matbench_steels      | regression/composition | yield strength (MPa)        | 312      | 229.3743|
| matbench_jdft2d      | regression/structure  | exfoliation_en (meV/atom)   | 636      | 67.2020|
| matbench_phonons     | regression/structure  | last phdos peak (cm^-1)     | 1,265    | 323.7870|
| matbench_expt_gap    | regression/composition | gap expt (eV)               | 4,604    | 1.1432|
| matbench_dielectric  | regression/structure  | n (unitless)                | 4,764    | 0.8085|
| matbench_expt_is_metal | classification/composition | is_metal                | 4,921    | 0.4981|
| matbench_glass       | classification/composition | gfa                     | 5,680    | 0.7104|
| matbench_log_gvrh    | regression/structure  | log10(G_VRH) (log10(GPa))   | 10,987   | 0.2931|
| matbench_log_kvrh    | regression/structure  | log10(K_VRH) (log10(GPa))   | 10,987   | 0.2897|
| matbench_perovskites | regression/structure  | e_form (eV/unit cell)       | 18,928   | 0.5660|
| matbench_mp_gap      | regression/structure  | gap pbe (eV)                | 106,113  | 1.3271|
| matbench_mp_is_metal | classification/structure | is_metal                 | 106,113  | 0.4349|
| matbench_mp_e_form   | regression/structure  | e_form (eV/atom)            |132,752	| 1.0059|

composition: 組成式。例）SiO2  
structure: 材料の構造。その結晶構造や分子構造など、原子や分子の配置や相互作用パターン  
Score: MAD (regression) or Fraction True (classification)  

# わからないこと
- なぜGaAsなどの代表的な物質についての情報がないのか
- なぜあまりベンチマークとして利用されていないのか
- MP番号との紐付け方
- 組成式の場合には構造異性体の処理はどのようにしているのか

# これからの進路
- わからないことの調査
- 使用例の作成

# 参考
MatBench：https://matbench.materialsproject.org/
Structureについて：https://qiita.com/ojiya/items/37b594150115531481f0

