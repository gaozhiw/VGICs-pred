# Voltage-Gated Channel Prediction Benchmark (DeepGO / NetGO vs UniProt annotations)

对电压门控离子通道（voltage-gated ion channel, VGIC）预测模型的系统评测与鲁棒性验证。

- **预测模型**：DeepGO（[DeepGOWeb API](https://deepgo.cbrc.kaust.edu.sa/), version 1.0.28）与 NetGO3 / NetGO4（5 列 txt 输出）。
- **标签来源**：UniProt 2023（KW "Voltage-gated channel" 注释）与人工整理的电压/非电压通道序列集合；UniProt 2026 版本用于标签漂移分析。
- **统一判定口径**：25 个电压门控 GO 标签（molecular function 通道活性类）的分数最大值作为 `Voltage_Score`，`Predicted=1 ⟺ Voltage_Score ≥ 阈值`。
- **评测指标**：Sensitivity / Specificity / MCC（随判定阈值变化）+ AUROC / AUPRC（排序指标，与阈值无关）；5 个随机数种子（2023/1234/567/42/100）作为 5 次重复报告 mean±SD。

## 数据流

```
FASTA 序列
   ├─ DeepGO API 预测 ──────────────► .json（functions: [[GO, term, score], ...]）
   ├─ NetGO3/4 服务器预测 ──────────► .txt（entry\tGO\tscore\tcat\tterm）
   └─ 匿名化（entry_anon_mapping.csv）
            │
            ▼
Voltage_Score = max(25 个 voltage GO 分数)          ← data/labels/all_voltage_go_tags.csv
Predicted = 1  ⟺  Voltage_Score ≥ 阈值（0.1–0.9 扫描）
            │
            ▼
指标计算：Sens / Spec / MCC / AUROC / AUPRC
   ├─ 2026 easy 812（五源：netgo3/netgo4/redundant/t010/default）
   ├─ 5 种子 test 69 / train 274（DeepGO t010 + NetGO4 + anon）
   ├─ 108 条共有 entry（5 种子交叉）
   └─ trEMBL 5551（UniProt 2023/2026 注释对照 → Panel A/B）
```

## 目录结构

```
GO_VGIC_analysis_github/
├── README.md
├── requirements.txt
├── data/
│   ├── labels/                  # 标签文件 + 25-GO 判定集合
│   │   ├── voltage_channels_sequences_515_2023.csv      # 电压通道（label=1）
│   │   ├── non_voltage_channels_sequences_872_2023.csv  # 非电压通道（label=0）
│   │   ├── labeled_sequences_for_encoding.csv           # 2026 easy 812 标签
│   │   ├── sequences_label_2026_dataset_cdhit_0.7.csv   # 2026 全量序列标签
│   │   └── all_voltage_go_tags.csv                      # ★ 25-GO 判定集合
│   ├── entry_anon_mapping.csv   # 匿名 ID 对照（File/Entry/Anon_ID/Original_Header）
│   └── fasta/                   # 输入序列（2026 easy、5 种子 test/train、trEMBL 5551）
├── notebooks/                   # 6 个可复现 Jupyter notebook（见下）
└── results/                     # 最终结果 CSV
    ├── 2026_easy/               # easy_25go_metrics_*.csv + GO 审计表
    ├── seeds/                   # 5 种子 test/train 25-GO 指标 + mean±SD + 每 seed 明细
    ├── common108/               # 108 条共有 entry 指标
    ├── netgo4_train/            # NetGO4 train 阈值扫描
    └── trembl/                  # Panel A/B (25-GO) + 标签对照表
```

## Notebooks

| # | Notebook | 内容 |
|---|---|---|
| 01 | `01_deepgo_predict.ipynb` | DeepGO REST API 批量预测（batch=10，threshold=0.1，逐批落盘） |
| 02 | `02_parse_and_anonymize.ipynb` | 预测解析（Entry 提取规则）+ 匿名 ID 映射 |
| 03 | `03_voltage_metrics_25go.ipynb` | 25-GO 判定、阈值扫描、指标（含 mean±SD） |
| 04 | `04_trEMBL_panels_25go.ipynb` | trEMBL Panel A/B（2023/2026 注释对照） |
| 05 | `05_go_tag_audit.ipynb` | 电压 GO 标签审计（25-GO 集合的确立） |
| 06 | `06_summary_tables.ipynb` | 汇总关键结果表 |

## 关键结果摘要（25-GO 口径）

### 2026 easy 812 条（pos=154 / neg=658）

| 预测源 | 最佳 MCC (阈值) | AUROC | AUPRC |
|---|---|---|---|
| netgo3 | 0.8118 (0.7) | 0.9659 | 0.9286 |
| netgo4 | 0.8107 (0.6) | 0.9764 | 0.9278 |
| redundant | **0.8350 (0.9)** | **0.9781** | **0.9327** |
| t010 (DeepGO) | 0.8346 (0.1) | 0.9225 | 0.9090 |
| default (DeepGO) | 0.7632 (0.1) | 0.8348 | 0.8105 |

### 5 种子 test 69（mean±SD，n=5）

| 源 | 最佳 MCC (阈值) | AUROC | AUPRC |
|---|---|---|---|
| netgo4 | 0.7781±0.0661 (0.8) | 0.9412±0.0151 | 0.8936±0.0350 |
| t010 | 0.7634±0.0811 (0.2) | 0.9290±0.0228 | 0.8843±0.0471 |
| anon | 0.7645±0.0702 (0.9) | 0.9357±0.0137 | 0.8803±0.0417 |

### trEMBL Panel A（2023 注释重合，Predicted VGICs=2093）

| Reference annotation | Annotated VGICs | Overlap | Positive overlap (%) | Unannotated |
|---|---|---|---|---|
| KW-0851 | 1375 | 1331 | 63.59 | 762 |
| KW-0851 + GO (25 tags) | 1631 | 1565 | **74.77** | 528 |

### 关键结论
- 25-GO 判定显著提升 NetGO 家族（netgo3/4/redundant）指标（easy 上 MCC 峰值由两 GO 时约 0.54 升至 0.81–0.84）；DeepGO 几乎不变（其电压蛋白的 GO:0022832 分数恒为最高）。
- AUROC/AUPRC 为排序指标，与判定阈值无关；MCC/Sens/Spec 随阈值变化（最佳 MCC 阈值用于最终报告）。
- trEMBL：KW-0851 + GO(25) 与预测重合率 74.77%；2023 无注释 3920 条中，2026 新增注释 149 条（2.68%），2023 有注释 1631 条中 2026 取消 224 条（4.04%）。

## 复现步骤

```bash
# 1. 安装依赖
pip install -r requirements.txt

# 2. 预测（可选；本仓库未内置大体积预测 JSON/TXT）
#    运行 notebooks/01_deepgo_predict.ipynb 重新生成 DeepGO 预测；
#    NetGO3/4 需在 NetGO 服务器提交，输出 5 列 txt。

# 3. 指标计算
#    运行 notebooks/03_voltage_metrics_25go.ipynb（读取 data/labels + 预测文件）

# 4. trEMBL Panel（需本地 uniprot_2023_earliest_txt/ 与 _panel_b_2026_recheck.json）
#    运行 notebooks/04_trEMBL_panels_25go.ipynb
```

## 注意事项

- **体积**：原始预测 JSON/TXT（2026 easy 五源 ~54 MB、5 种子 ~90 MB、train ~30 MB）与
  UniProt 2023 最早版本 txt（6938 个）未内置仓库，可从原项目 `GO模型测试/` 复制或按 01/04 notebook 重新生成。
- **Entry 提取**：DeepGO JSON 的 `protein_info` 可能是完整 FASTA 头（`sp|Q7Z020|TRPA1_DROME ...`），
  统一取 `sp|`/`tr|` 后、下一 `|` 前的文本；NetGO txt 第一列同理。
- **匿名化**：`data/entry_anon_mapping.csv` 提供全局唯一 Anon_ID 对照，匿名 FASTA 与匿名预测
  （`result_test*_anon.txt` 等）用于不泄露 accession 的复现。
- **2026 Removed 口径**：trEMBL Panel B 的"2026 Removed"列基于历史标记（两 GO 近似），严格 25-GO 需重查 UniProt 2026。

## License

For research use only.
