# Pawsey Setonix

[Pawsey](https://pawsey.org.au) is Australia's national supercomputing centre. [Setonix](https://pawsey.org.au/systems/setonix/) is their flagship supercomputer with **AMD Instinct MI250X GPUs** (ROCm).

**Project:** `pawsey1339` — expires 2026-06-30
**Support:** help@pawsey.org.au

!!! warning
    Setonix GPUs are **AMD MI250X — ROCm only**. CUDA code (e.g. Isaac Sim) will **not** run. For CUDA workloads use [NCI Gadi](nci.md) or [M3](m3.md) instead.

---

## Getting Access

Contact Lingheng (`lingheng.meng1@monash.edu`) to be invited. You'll receive an email from Pawsey — click the link to create your account and set a password.

---

## Connecting

```bash
ssh {your_pawsey_id}@setonix.pawsey.org.au
```

You land on a login node — use it for editing scripts, submitting jobs, and checking status. Do not run heavy computation on login nodes.

---

## Storage

| Path | Env var | Quota | Purge | Use for |
|------|---------|-------|-------|---------|
| `/home/{username}` | `$HOME` | 1 GiB | No | Shell config, dotfiles only |
| `/software/projects/pawsey1339/{username}` | `$MYSOFTWARE` | 256 GiB shared | No | Conda envs, installs, SLURM scripts |
| `/scratch/pawsey1339/{username}` | `$MYSCRATCH` | 1 PiB shared | **21 days** | Job input/output, working data |
| `/acacia` (project) | — | 512 GB shared | No | Long-term archival, results |
| `/acacia` (personal) | — | 100 GB | No | Personal long-term storage |

```bash
cd $MYSCRATCH       # go to your working directory
echo $MYSOFTWARE    # your software/install dir
```

!!! warning
    `/scratch` is purged **21 days after last modification**. Do not use `touch` to avoid purge — move important outputs to `/acacia` promptly.

!!! tip
    `/software` quota is **shared across the whole project**. Keep conda environments lean and remove unused ones.

### Shared datasets

All project members share a common directory — use it to avoid duplicating large datasets:

```bash
# Persistent shared space (no purge)
/software/projects/pawsey1339/shared/datasets/

# High-performance shared space (21-day purge)
/scratch/pawsey1339/shared/
```

---

## Transferring Files

Use the **data mover node** for large transfers:

```bash
# Upload (run on your local machine)
scp myfile.py {username}@data-mover.pawsey.org.au:$MYSCRATCH/

# Download (run on your local machine)
scp {username}@data-mover.pawsey.org.au:$MYSCRATCH/output.log .
```

---

## Software Modules

```bash
module avail            # list all available software
module avail python     # search for a specific package
module load python/3.11 # load a module
module list             # show currently loaded modules
```

---

## Submitting Jobs (SLURM)

| Partition | Use | Max walltime |
|-----------|-----|-------------|
| `work` | Standard CPU | 24 hrs |
| `gpu` | AMD MI250X GPU (ROCm) | 24 hrs |
| `debug` | Quick tests | 1 hr |
| `long` | Long CPU jobs | 96 hrs |

### CPU job

```bash
#!/bin/bash
#SBATCH --job-name=my_job
#SBATCH --account=pawsey1339
#SBATCH --partition=work
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=%x-%j.out

module load python/3.11
cd $MYSCRATCH
python train.py
```

### GPU job (AMD ROCm)

```bash
#!/bin/bash
#SBATCH --job-name=gpu_job
#SBATCH --account=pawsey1339
#SBATCH --partition=gpu
#SBATCH --nodes=1
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=04:00:00
#SBATCH --output=%x-%j.out

module load rocm
cd $MYSCRATCH
python train.py
```

### Essential SLURM commands

```bash
sbatch job.sh        # submit a job
squeue --me          # check your jobs
scancel <jobid>      # cancel a job
seff <jobid>         # show job efficiency (CPU/memory usage)
```

---

## Checking Usage

```bash
# Storage and SU balance
pawseyAccountBalance -s

# Scratch quota
lfs quota -g $PAWSEY_PROJECT -h /scratch

# Home directory
/usr/bin/quota -s -f /home
```

SU usage is also visible in the [Origin portal](https://portal.pawsey.org.au/origin/) → project → compute resources.

!!! note
    SUs are allocated quarterly and do **not** carry over. Jobs can still run after the budget is exhausted but at lower priority.

---

## Useful Links

- [Getting Started](https://pawsey.atlassian.net/wiki/spaces/US/pages/51925850/Getting+Started+with+Supercomputing)
- [Pawsey Filesystems](https://pawsey.atlassian.net/wiki/spaces/US/pages/51925876/Pawsey+Filesystems+and+their+Use)
- [GPU Jobs on Setonix](https://pawsey.atlassian.net/wiki/spaces/US/pages/51929056)
- [Acacia Object Storage](https://pawsey.atlassian.net/wiki/spaces/US/pages/51924576/Pawsey+Object+Storage+Acacia)
- [System Status](https://status.pawsey.org.au)
