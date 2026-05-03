# M3 Admin Guide (Project Leaders)

This page is for **mh42 project leaders** — anyone with supervisor or admin responsibilities over the shared storage. Students do not need to read this.

---

## The Core Rule: Supervisors Create Project Folders

**Never let students create the project folder at the mh42 root.** Students only create their `stack_{name}/` folder inside a project folder that already exists.

Why this matters: `setfacl` (the tool that gives leaders permanent access to student data) requires you to **own** the directory. Group write permission is not enough. If a student creates the project folder, leaders cannot set default ACLs on it without helpdesk intervention.

---

## Setting Up a New Project Folder

Run this immediately after creating each new project folder:

```bash
# 1. Create the project folder (supervisor does this, not the student)
mkdir /fs04/mh42/{project_folder}

# 2. Set default ACLs for all active leaders
#    danak must always be included — she is the permanent PI
setfacl -d -m u:danak:rwx /fs04/mh42/{project_folder}/
setfacl -d -m u:lmen0023:rwx /fs04/mh42/{project_folder}/
setfacl -d -m u:kkatuwan:rwx /fs04/mh42/{project_folder}/

# 3. Verify the ACL was set
getfacl /fs04/mh42/{project_folder}/
```

**What default ACL does:** The `-d` flag makes the ACL a *template*. Every file and subdirectory a student creates inside automatically inherits an entry giving the listed leaders full access — no student cooperation required, even after they graduate.

!!! note "Always include `danak`"
    Dana Kulic (`danak`) is the permanent PI and will outlast any postdoc or student leader. Always include her in the default ACL. Other leaders (postdocs, co-supervisors) may leave — `danak` will not.

!!! tip "Confirmed working on M3's Lustre"
    `setfacl` with default ACLs is confirmed working on `/fs04/mh42` (Lustre filesystem, tested 2026-05-02). Inheritance propagates correctly to both subdirectories and files. The effective permission on files is `rw-` (execute bit masked automatically — this is correct).

---

## Checking for Non-Standard Root Items

Run this periodically (e.g. start of each semester) to catch students who dropped files in the wrong place:

```bash
for item in /fs04/mh42/*; do
    name=$(basename "$item")
    if ! echo "$name" | grep -qE \
        '^FYP[0-9]+S[12]_[0-9]+$|^PhD_[0-9]+_|^Postdoc_[0-9]+_|^Staff_|^scripts$|^0_mh42_README_START_HERE\.md$|^PROJECTS\.md$'; then
        echo "NON-STANDARD: $name  (owner: $(stat -c '%U' "$item"), modified: $(stat -c '%y' "$item" | cut -d' ' -f1))"
    fi
done
```

Non-standard items cannot be moved by leaders (Lustre restricts renaming items owned by other users, even with group write on the parent). To reorganize them, email [help@massive.org.au](mailto:help@massive.org.au) with the exact `mv` commands — this is a zero-space-cost rename, processed quickly.

---

## Fixing Folders You Don't Own

If a student created the project folder themselves (so you can't set ACLs), options are:

1. **Email helpdesk** — ask them to `chown` the project folder to you, then set the ACLs yourself
2. **Ask the student** — if they still have access, ask them to run `setfacl -d -m u:danak:rwx` on their project folder
3. **Ask the student to chmod** — `chmod -R g+rwX stack_{name}/` as offboarding requirement opens group read+write to all mh42 members, including leaders

---

## Storage Policy Enforcement

The mh42 project quota is **500 GB shared and cannot be increased**. As project leader you have the authority to:

- Move or archive stale data without prior notice (per the storage policy in the README)
- Contact students whose data is non-standard, stale, or oversized
- Request scratch2 quota increases at [tinyurl.com/massive-m3-quota-request](https://tinyurl.com/massive-m3-quota-request) (3 TB default, can increase; allow up to 2 weeks)

**Priority cleanup targets** (largest space consumers):
- Apptainer `.sif` container images — share one per team instead of per-student copies
- Conda environments — students often duplicate base installs
- `kit_cache/`, `kit_data/` folders inside Isaac Sim installs — regeneratable caches, safe to delete

**Archiving to S Drive:** when project quota is tight, archive completed runs to `S:\ENG-ECSE\Robotics-M3` via the [S Drive guide](../storage/s-drive.md).

---

## Helpdesk Contact

For operations that require root (moving other users' files, chown, quota changes):

- **Email:** [help@massive.org.au](mailto:help@massive.org.au)
- **Quota increase form:** [tinyurl.com/massive-m3-quota-request](https://tinyurl.com/massive-m3-quota-request)
- Response time: typically 48 business hours
