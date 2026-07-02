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

> **00 补侧链(通用 flag)-- |全原子输出|--relax|需 PyRosetta|**

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

> **01 部分 CDR 设计 -- |CDR-H3|完整复合物→自动表位|默认参数|**

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

> **02 全 CDR 设计 -- |全 6 个 CDR|完整复合物→自动表位|默认参数|**

fasta 中 6 个 CDR 全部挖成 `X`,一次设计全部 CDR 并预测结构，命令同上
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_All.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs
```

> **03 从头设计 -- |全 CDR|仅抗原 + 手动表位|--epitope|**

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

##### [IgGM 官方仓库](https://github.com/TencentAI4S/IgGM)
