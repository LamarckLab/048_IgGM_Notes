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

> **01 CDR-H3 设计 -- |CDR-H3|完整复合物→自动表位|默认参数|**

抗体 fasta 中 CDR-H3 用 `X` 挖空,配完整复合物 pdb(自动算表位),设计 CDR-H3 序列并预测复合物结构
```bash
cd /data/lmk/IgGM
CUDA_DEVICE_ORDER=PCI_BUS_ID CUDA_VISIBLE_DEVICES=2 \
python design.py \
  --fasta /data/lmk/IgGM_inputs/8hpu_M_N_A_CDR_H3.fasta \
  --antigen /data/lmk/IgGM_inputs/8hpu_M_N_A.pdb \
  --output /data/lmk/IgGM_outputs
```

##### [IgGM 官方仓库](https://github.com/TencentAI4S/IgGM)
