# NCI Gadi

[NCI](https://nci.org.au) is Australia's national research computing facility. [Gadi](https://nci.org.au/our-systems/hpc-systems/gadi) is their flagship supercomputer with **NVIDIA GPUs and full CUDA support**.

**Project:** `pg06` — CUDA-Accelerated Robot Simulation and Deep Learning
**Portal:** [my.nci.org.au](https://my.nci.org.au)
**Support:** help@nci.org.au

---

## Why NCI over Pawsey

| | NCI Gadi | Pawsey Setonix |
|---|---|---|
| GPU | NVIDIA H200, A100, V100 | AMD MI250X |
| Framework | **CUDA** | ROCm only |
| Isaac Sim / Isaac Lab | **Supported** | Not supported |
| PyTorch / MuJoCo | Excellent | Excellent |

**Use NCI** when you need CUDA — Isaac Sim, Isaac Lab, or any NVIDIA-only library.
**Use Pawsey** for large-scale PyTorch or MuJoCo workloads where AMD support is sufficient.

---

## GPU Nodes

| GPU | Memory | Notes |
|---|---|---|
| NVIDIA H200 141GB | 141 GB HBM3 | Newest, highest performance |
| NVIDIA A100 80GB | 80 GB HBM2e | Excellent for deep learning |
| NVIDIA V100 32GB | 32 GB HBM2 | Widely available |

---

## Getting Access

1. Register at [my.nci.org.au](https://my.nci.org.au)
2. Request to join project `pg06` — Lingheng will approve
3. Once approved, log in or access [ARE](https://are.nci.org.au) straight away

---

## Connecting

**SSH**

```bash
ssh {your_nci_id}@gadi.nci.org.au
```

**ARE — browser-based (no SSH needed)**

[ARE (Advanced Research Environment)](https://are.nci.org.au) lets you launch **JupyterLab** or a **GPU Virtual Desktop** directly in your browser. This is the easiest way to get started with interactive work.

---

## Storage

| Path | Quota | Purge | Use for |
|---|---|---|---|
| `/home/{username}` | 10 GiB | No | Config files, scripts |
| `/scratch/pg06/{username}` | 1 TiB (project) | **100 days without access** | Active job data |
| `massdata/pg06` | — | No | Long-term archival |

```bash
# Check quota and SU balance
nci_account

# Check scratch (live)
lquota

# Check home (live)
quota -s
```

!!! warning
    `/scratch` is purged after **100 days without access**. Move important results to `massdata/pg06` before they expire. Check expiry with `nci-file-expiry`.

---

## Submitting Jobs (PBS)

NCI uses **PBS** (not SLURM). Key differences: `#PBS` directives, `qsub` to submit, `qstat` to check.

### GPU job example

```bash
#!/bin/bash
#PBS -N gpu_job
#PBS -P pg06
#PBS -q gpuvolta
#PBS -l ncpus=12
#PBS -l ngpus=1
#PBS -l mem=96GB
#PBS -l walltime=04:00:00
#PBS -l storage=scratch/pg06
#PBS -l jobfs=50GB

cd $PBS_O_WORKDIR
module load python3
python train.py
```

### Essential PBS commands

```bash
qsub job.sh          # submit a job
qstat -u $USER       # check your jobs
qdel <jobid>         # cancel a job
nqstat               # view queue summary
```

---

## Useful Links

- [NCI Help](https://opus.nci.org.au/display/Help/NCI+Help)
- [Gadi Queue Structure & SU rates](https://opus.nci.org.au/display/Help/Gadi+Queue+Structure)
- [ARE User Guide](https://opus.nci.org.au/display/Help/ARE+User+Guide)
- [Gadi Filesystems](https://opus.nci.org.au/display/Help/Gadi+File+Systems)
