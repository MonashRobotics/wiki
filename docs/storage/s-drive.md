# S Drive (Shared Lab Storage)

The lab S Drive is persistent shared storage hosted by Monash eSolutions. Unlike M3 project storage (500 GB shared quota), the S Drive is backed up and does not get purged.

| | |
|---|---|
| **UNC path** | `\\ad.monash.edu\shared\ENG-ECSE\Robotics-M3` |
| **Windows path** | `S:\ENG-ECSE\Robotics-M3` |
| **Purpose** | Archive completed training runs, share assets across the lab |
| **Access** | Request via Lingheng — Monash account required |

!!! warning
    S Drive uses SMB (network file share) — it is **not suitable for training I/O**. Use it only for one-time archiving and restoring. Active training must run on M3 project storage or scratch.

---

## Getting Access

Email Lingheng (`lingheng.meng1@monash.edu`) with your Monash authcate ID or Monash email address. He will add you as a Group Member. Allow up to 3 hours for permissions to propagate.

---

## Folder Structure

The S Drive follows the **same two-level convention as M3**: project folder → `stack_{name}/` subfolder.

```
S:\ENG-ECSE\Robotics-M3\
  FYP2026S1_3487\
    stack_alice\          ← Alice creates this herself
      checkpoints\
      logs\
    stack_bob\
  PhD_2023_dave\
    stack_dave\
  shared_assets\          ← shared lab assets (Isaac Sim .sif, models)
```

**Project folder naming** (same as M3):

| Role | Format | Example |
|---|---|---|
| FYP student | `FYP{YYYY}S{1\|2}_{ProjectID}` | `FYP2026S1_3487` |
| PhD student | `PhD_{YYYY}_{firstname}` | `PhD_2023_dave` |
| Postdoc | `Postdoc_{YYYY}_{firstname}` | `Postdoc_2026_somayeh` |

!!! note
    **Supervisors create the project folder** — the same rule as M3. Students create only their `stack_{name}/` inside it. This ensures consistent structure and makes it possible to manage the drive as people come and go.

Each person is responsible for creating their own `stack_{name}/` folder inside the project folder. Do not write into someone else's stack.

!!! note
    There is no per-folder access control — every Group Member can read and write anywhere in the share. This is trust-based, the same as M3 project storage.

---

## Storage Policy

The S Drive is shared lab storage. To keep it manageable:

1. **Follow the folder convention.** All data must live under `{project_folder}/stack_{name}/`. Non-standard folders and loose files at the root will be reorganized by the admin without notice.
2. **Don't leave orphaned data.** When you finish a project, update the `STATUS` file in your project folder (same values as M3: `ACTIVE`, `DONE-{YYYY-MM}`, `ARCHIVE`, `OK-TO-DELETE`).
3. **Shared assets go in `shared_assets/`.** Large files shared across the lab (Isaac Sim `.sif`, base models) belong in `shared_assets/` — not duplicated in each person's stack.

---

## Mounting on M3

Run on the **login node only** — not inside a SLURM job.

**Step 1 — Mount**

```bash
gio mount 'smb://MONASH;{your_monash_id}@ad.monash.edu/Shared'
```

Enter your **Monash password** when prompted (not your M3/MASSIVE password).

**Step 2 — Create a symlink for convenience**

```bash
ln -s /run/user/$(id -u)/gvfs/smb-share:domain=MONASH,server=ad.monash.edu,share=shared,user={your_monash_id}/ENG-ECSE/Robotics-M3 ~/s_drive
ls ~/s_drive   # should show S Drive contents
```

**Step 3 — Unmount when done**

```bash
gio mount -u 'smb://MONASH;{your_monash_id}@ad.monash.edu/Shared'
```

!!! warning
    The mount is session-scoped. Compute nodes cannot see it. Do NOT use S Drive paths inside SLURM job scripts.

---

## Archiving Data from M3 to S Drive

Use this to free M3 project quota by moving completed runs to S Drive.

```bash
SHARE=~/s_drive
PROJECT=FYP2026S1_3487       # your project folder name
MY_NAME=alice                 # your stack name

# Archive a training run
rsync -av --progress \
  /fs04/mh42/${PROJECT}/stack_${MY_NAME}/logs/<run-folder>/ \
  $SHARE/${PROJECT}/stack_${MY_NAME}/logs/<run-folder>/

# Verify the copy, then free M3 space
rm -rf /fs04/mh42/${PROJECT}/stack_${MY_NAME}/logs/<run-folder>
```

**Restoring from S Drive back to M3**

```bash
rsync -av \
  $SHARE/${PROJECT}/stack_${MY_NAME}/logs/<run-folder>/ \
  /fs04/mh42/${PROJECT}/stack_${MY_NAME}/logs/<run-folder>/
```

---

## Recommended Workflow

```
S Drive  ←→  (rsync via login node, one-time transfers)  ←→  M3 scratch  ←→  M3 project
(archive)                                                      (active jobs)   (lean, active only)
```

1. Mount S Drive on login node
2. Copy needed data: S Drive → scratch (large, fast Lustre)
3. Submit SLURM job — reads/writes scratch or project only
4. After job, archive results: scratch/project → S Drive
5. Clean up scratch and project to free space

For large shared assets (e.g. Isaac Sim `.sif`, ~20 GB): keep **one copy** in `s_drive/shared_assets/` and copy to scratch before jobs — do not keep duplicate copies in the project folder.

---

## Mounting on Windows (personal laptop)

Requires Monash VPN connected first.

1. Open File Explorer → This PC → **⋯** → **Map network drive**
2. Drive: `S:`, Folder: `\\ad.monash.edu\shared\ENG-ECSE\Robotics-M3`
3. Tick **Reconnect at sign-in** and **Connect using different credentials**
4. Credentials: `MONASH\{your_monash_id}` + Monash password

---

## Speed Reference

| | M3 Lustre (project/scratch) | S Drive (SMB) |
|--|---|---|
| Sequential read/write | 200–500+ MB/s | ~50–150 MB/s |
| Many small files | Reasonable | Very slow (~1–5 MB/s) |
| 100 GB transfer | ~3–8 min | ~12–33 min |
| Suitable for training I/O | Yes | **No** |

---

## Priority Targets for Freeing M3 Space

1. Old training checkpoints and logs no longer actively used
2. Personal apptainer sandboxes (`apptainer/sandbox/`, ~20 GB each) — rebuild from `s_drive/shared_assets/` if needed
3. Duplicate conda environments — coordinate with teammates to share one copy
