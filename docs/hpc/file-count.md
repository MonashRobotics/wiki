# Managing File Count (Inode) Limits

Most HPC systems enforce a **file count (inode) quota** that is shared across the project. Hitting it stops jobs from creating new files — even if storage space is available.

| System | Quota | Scope |
|---|---|---|
| NCI Gadi `/scratch/pg06` | 202,000 files | Project-wide |
| Pawsey `/software/pawsey1339` | Project limit | Project-wide |
| M3 `/fs04/mh42` | No published file limit | — |

Check your current usage:

```bash
# NCI Gadi
lquota

# Pawsey Setonix
lfs quota -g pawsey1339 /software

# M3
user_info
```

---

## Why It Happens

A single conda/pip environment typically contains **30,000–80,000 files**. When multiple team members each install their own environment, the project quota fills up fast. Large datasets stored as individual files (images, CSVs) make it worse.

---

## Strategies

### 1. Use containers instead of conda environments

A Singularity/Apptainer `.sif` image is **1 file** regardless of how many packages are inside.

```bash
# NCI Gadi and Pawsey
module load singularity

# M3 and Virga
module load apptainer   # same tool, rebranded

singularity pull pytorch.sif docker://pytorch/pytorch:2.1.0-cuda12.1-cudnn8-runtime
singularity exec --nv pytorch.sif python3 train.py
```

**Containers are read-only** — workarounds for extra packages:

*Option A — bind mount a writable directory (simpler, no persistent image change):*
```bash
mkdir -p $SCRATCH/pip_extra
singularity exec \
  --bind $SCRATCH/pip_extra:/pip_extra \
  pytorch.sif pip install --target=/pip_extra my_package
export PYTHONPATH=/pip_extra:$PYTHONPATH
```

*Option B — overlay image (changes persist, still only 2 files total):*
```bash
singularity overlay create --size 2048 overlay.img
singularity exec --overlay overlay.img pytorch.sif pip install my_package
```

**Editable code (`pip install -e .`):** bind-mount your repo instead — no install needed:
```bash
singularity exec --nv \
  --bind /path/to/myrepo:/myrepo \
  pytorch.sif python /myrepo/train.py
```

---

### 2. Share environments with teammates

If several people need the same packages, one shared `.sif` costs 1 inode instead of N. Put it somewhere accessible to the project group and point everyone at the same file.

---

### 3. Datasets with many small files — zip and extract to RAM or local storage

Store datasets as a single archive. Extract to a RAM disk or job-local NVMe at job start — zero impact on the project inode quota.

```bash
# Extract to RAM disk (fast; limited by node RAM)
tar -xzf /scratch/mydata/dataset.tar.gz -C /dev/shm/

# Extract to job-local NVMe (NCI Gadi — request with #PBS -l jobfs=50gb)
tar -xzf /scratch/mydata/dataset.tar.gz -C $PBS_JOBFS/

# Extract to job-local temp (Pawsey Setonix — request with --gres=tmp:200G)
tar -xzf $MYSCRATCH/dataset.tar.gz -C $TMPDIR/
```

---

### 4. PyTorch DataLoader settings

Use `num_workers > 0` with `pin_memory=True`. Workers preload batches in parallel into pinned memory, reducing reliance on OS-level caching of many small files:

```python
DataLoader(dataset, num_workers=4, pin_memory=True, persistent_workers=True)
```

---

### 5. Clean up unused environments regularly

```bash
# Conda
conda env list
conda env remove -n <env_name>

# pip cache
pip cache purge

# uv cache
rm -rf ~/.cache/uv          # or $SCRATCH/.uv_cache if redirected
```
