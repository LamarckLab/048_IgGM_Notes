## Lamarck &nbsp; &nbsp; &nbsp; 2026-07-01
#### 该文档用于记录 server 上跑 IgGM 的各种命令
---

*236 机子的环境*
```bash
conda activate lmk_IgGM
```

*输入输出路径*
```bash
权重缓存:   /data/lmk/IgGM/checkpoints   # 首次运行自动从 Zenodo 下载
输入目录:   /data/lmk/IgGM_inputs        # 设计用 fasta + 抗原 pdb
输出目录:   /data/lmk/IgGM_outputs       # 设计结果 pdb + fasta
```

*GPU 选择*
```bash
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2
```

---

> **00 补侧链 (通用 flag) -- |全原子输出|--relax|**

任意设计命令末尾加 `--relax`即可补全侧链 + 能量最小化,输出全原子结构;可叠加到下面 01/02/03 任意功能，纯 CPU、明显更慢
```bash
cd /data/lmk/IgGM
rm -f /data/lmk/IgGM_outputs/8hpu_M_N_A_CDR_H3_0.pdb /data/lmk/IgGM_outputs/8hpu_M_N_A_CDR_H3_0.fasta
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_H3.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs \
  --relax # 加这一行
```

> **01 部分 CDR 设计 -- |CDR-H3|完整复合物→自动表位|**

fasta 文件中 CDR-H3 用 `X` 挖空，搭配完整复合物 pdb，设计 CDR-H3 序列并预测复合物结构
> 完整复合物模式下，IgGM 只从 pdb 取 **抗原结构** + **表位**（抗原侧接触残基）；不取抗体结构，也不看 CDR-H3 真实序列
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_H3.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs
```

> **02 全 CDR 设计 -- |全 CDR|完整复合物→自动表位|**

fasta 中 6 个 CDR 全部挖成 `X`,一次设计全部 CDR 并预测结构，命令同上
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_All.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs
```

> **03 从头设计 -- |全 CDR|仅抗原 + 手动表位|**

无复合物,只给抗原-only pdb + `--epitope` 手动指定表位来设计 CDR
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_All.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_A_antigen_only.pdb \
  --epitope 7 8 9 10 11 12 13 14 108 109 110 111 112 113 114 115 116 118 167 157 158 160 161 162 163 164 \
  --output /data/lmk/IgGM_outputs
```

> **04 逆向设计 -- |结构→序列|**

`--run_task inverse_design` 换用另一套权重 `antibody_inverse_design_trunk`，只设计序列、不预测结构，输出**只有 fasta、没有 pdb**（首次运行会自动下载该权重）
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_H3.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs \
  --run_task inverse_design
```

> **05 亲和力成熟 -- |单点扫描|**

在已有抗体上做逐点突变扫描：`--fasta_origin` 给原始序列作起点，`--fasta` 的 `X` 标出待突变位点；每次只挖一个位点、其余保持原始并重设计，重复 `--num_samples` 次。**只出 fasta**（ `num_samples × 位点数` 个单点变体，无 pdb）
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_H3.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --fasta_origin /data/lmk/IgGM_inputs/8hpu_M_N_A.fasta \
  --run_task affinity_maturation \
  --num_samples 2 \
  --output /data/lmk/IgGM_outputs
```

> **06 人源化 -- |fr_design 模型|**

FR 区人源化精修：具体命令与结果待实际用到时再测试补充

##### [IgGM 官方仓库](https://github.com/TencentAI4S/IgGM)
