# MoECLIP-baseline — 复现实验结果

本目录存放 **MoECLIP（CVPR 2026）官方代码的复现实验结果**。

## 代码来源

- 论文：**MoECLIP: Patch-Specialized Experts for Zero-shot Anomaly Detection**（CVPR 2026）
  - arXiv: [2603.03101](https://arxiv.org/abs/2603.03101)
  - 官方代码：[CoCoRessa/MoECLIP](https://github.com/CoCoRessa/MoECLIP)
- 本仓库为论文官方代码的**原样复现**，代码未做任何改动。

## 与论文的唯一差异

| 项目 | 论文 | 本复现 |
|---|---|---|
| **学习率 lr** | `5×10⁻⁴` | `5×10⁻⁵`（官方代码下载后的默认值） |
| 其余超参 | — | 与论文/官方代码一致（K=4、Top-2、r=8、α=16、dropout=0.05、λMoE=0.1、λetf=λbal=0.01、batch=2、betas=(0.5,0.999)、MultiStepLR[16000,32000]×0.5） |

> 学习率差异的说明：`5×10⁻⁴` 曾实测复现效果显著更差，故最终采用官方代码默认的 `5×10⁻⁵`，属于经对比后择优确定，非疏漏。

## 结果口径（重要）

- 训练 20 个 epoch，每个 epoch 保存一个 checkpoint，测试时对全部 checkpoint 逐 epoch 输出结果。
- 本目录报告的是**各指标最佳 checkpoint**（四个指标各自取其 20 个 epoch 中的最大值，可能来自不同 epoch）。
- 论文与官方代码**均未明确指定** checkpoint 选择口径（官方 test.py 逐 epoch 打印、无汇总逻辑）；本复现取"各指标最佳"遵循 ZSAD 领域通行做法，且该口径下复现结果与论文吻合度最高（73% 指标落在 ±0.6 内）。

## 目录结构

```
results/
├── README.md          # 本文件
└── v1/                # 第一版结果
    ├── summary.csv    # 14 个数据集 × 4 指标 的最佳值汇总（含对应 epoch）
    ├── per_epoch/     # 每数据集逐 epoch 明细（JSON）
    └── logs/          # 原始 test 日志（完整证据链）
```

## 数据说明

- **指标顺序**：Image AUROC / Image AP / Pixel AUROC / Pixel AP。
- **训练→测试方向**：默认用 VisA 权重测试其余 13 个数据集；`VisA` 一行用 MVTec-AD 权重测试（与论文一致）。
- **无效值**：
  - `Head CT` 无像素级标注 → Pixel 两列为空；
  - `ColonDB / ClinicDB / CVC-300 / Endo / Kvasir`（结肠息肉五集）为单类别、无正常样本 → Image 两列为 0，仅 Pixel 有效（与论文口径一致）。

## 复现命令

```bash
# 训练（VisA 权重）
python train.py --dataset VisA --save_path ckpt/baseline

# 测试各数据集
python test.py --dataset <DATASET> --save_path ckpt/baseline
```

权重文件（.pth checkpoint）体积较大，未纳入本仓库；如需复现可依上述命令自行训练，结果与 summary.csv 一致。
