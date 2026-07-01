## Lamarck &nbsp; &nbsp; &nbsp; 2026-07-01
#### 该文档用于部署 IgGM
---

## 01  克隆官方源码仓库
> **236 机子路径 /data/lmk/IgGM**
```bash
cd /data/lmk
git clone https://github.com/TencentAI4S/IgGM.git
```

## 02  从 environment.yaml 创建 conda 环境
> 官方自带 environment.yaml，dependencies 里面指定了 Python 3.10 + `pytorch 2.0.1/cu117`；用 `-n` 覆盖掉 yaml 里默认的 `IgGM` 命名
```bash
cd /data/lmk/IgGM
conda env create -n lmk_IgGM -f environment.yaml
```

## 03  修复 torch 版本
> yaml 的 pip 部分没指定 torch 版本，且 pip 在 conda 之后跑，会把 conda 装好的 `2.0.1+cu117` 顶成最新 `cu130` 版；236 驱动只到 CUDA 12.8 → `torch.cuda.is_available()` 为 False，所以需要修复一下 torch
```bash
conda activate lmk_IgGM
python -m pip install torch==2.0.1 torchvision==0.15.2 --index-url https://download.pytorch.org/whl/cu117
```

## 04  修复 openmm 的 libstdc++
> IgGM 导入 openmm 时可能会报 `GLIBCXX_3.4.30 not found`：系统旧 libstdc++ 被优先加载，env 自带的新 libstdc++ 没用上，用 conda 把 env 的 lib 目录绑到 LD_LIBRARY_PATH
```bash
conda env config vars set LD_LIBRARY_PATH="$CONDA_PREFIX/lib"
conda deactivate && conda activate lmk_IgGM
```

## 05  验证
> torch 认到 GPU（返回 True）+ design.py 能加载（打印出参数帮助即成功）
```bash
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"
python design.py --help
```

## 06  IgGM 权重与输入输出目录
**236 机子路径**
> 权重缓存：/data/lmk/IgGM/checkpoints  &nbsp;（首次运行自动从 Zenodo 下载）
> 输入目录：/data/lmk/IgGM_inputs  &nbsp;（设计用 fasta + 抗原 pdb）
> 输出目录：/data/lmk/IgGM_outputs  &nbsp;（设计结果 pdb + fasta）

##### [IgGM 官方仓库](https://github.com/TencentAI4S/IgGM)
