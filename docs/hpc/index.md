# HPC Overview

The lab has allocations on three HPC systems. Choose based on your workload:

| System | GPU | Best for |
|---|---|---|
| [M3 (Monash)](m3.md) | NVIDIA A40, A100, H100 | Isaac Sim, CUDA workloads, general training |
| [Pawsey Setonix](pawsey.md) | AMD MI250X (ROCm) | PyTorch, MuJoCo, non-CUDA workloads |
| [NCI Gadi](nci.md) | NVIDIA H200, A100 | CUDA, Isaac Lab, JupyterLab via ARE |

!!! note
    Isaac Sim requires an A40 GPU specifically (RT Cores). Only M3 has A40s.
