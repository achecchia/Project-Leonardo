# Klipper-Backup Docs Protection

This page documents the issue where the Raspberry Pi's Klipper-Backup process deleted GitHub documentation files, and the fix that prevents it from happening again.

The goal is simple:

```text
The Raspberry Pi may back up printer config files.
The Raspberry Pi must not manage, delete, stage, or overwrite the docs folder.
```

---

## What Happened

The repository documentation files were created directly in GitHub:

```text
docs/upgrades.md
docs/configuration.md
docs/future-ideas.md
docs/project-history.md
docs/punch-list.md
```

Later, the Mainsail `update_git` button was used. That button runs the local Klipper-Backup script on the Raspberry Pi.

The Pi's local backup working folder did not have the GitHub-created `docs/` folder at that time. The backup script treated the Pi's local folder as the source of truth and pushed a backup commit.

Result:

```text
GitHub docs were deleted because they were missing from the Pi's local backup folder.
```

---

## Root Cause

The dangerous part of the original Klipper-Backup script was this section:

```bash
git rm -r --cached . >/dev/null 2>&1
git add .
```

That logic means:

```text
1. Untrack everything
2. Re-add whatever currently exists on the Pi
3. Anything missing locally is treated as deleted
```

Because `docs/` did not exist locally on the Pi, Git staged those files as deleted and pushed that deletion to GitHub.

There were also cleanup commands that removed everything from the local backup folder except a few Git-related files:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' -exec rm -rf {} \;
```

That cleanup behavior made it even easier for documentation files to be missing locally.

---

## Verified Symptoms

The issue was confirmed by checking the backup script:

```bash
grep -R "git add" ~/klipper-backup -n
```

Problem result:

```text
/home/pi/klipper-backup/script.sh:274:git add .
```

The surrounding script section showed:

```bash
# Untrack all files so that any new excluded files are correctly ignored and deleted from remote
git rm -r --cached . >/dev/null 2>&1
git add .
```

The local backup repo also showed docs were not the problem after the initial fix, but config files were left as deleted after cleanup:

```bash
cd ~/config_backup
git status --short
```

This originally showed deleted `printer_data/config/...` entries after the backup script ran, while `docs/` survived after the protection changes.

---

## Fix Applied

### 1. Backed up the Klipper-Backup script

Before editing the script, a local backup was made:

```bash
cp ~/klipper-backup/script.sh ~/klipper-backup/script.sh.backup-before-docs-protection
```

---

### 2. Changed Git staging behavior

The dangerous block was replaced.

Old behavior:

```bash
# Untrack all files so that any new excluded files are correctly ignored and deleted from remote
git rm -r --cached . >/dev/null 2>&1
git add .
```

New behavior:

```bash
# Stage only Klipper config files.
# Do NOT let Klipper-Backup stage repo documentation changes/deletions.
# Documentation is managed from GitHub/ChatGPT, not from the Raspberry Pi.
git add -A printer_data/config/

# Extra safety: never include docs or README changes in automatic Pi backups.
git reset -- docs/ README.md >/dev/null 2>&1 || true
git checkout -- docs/ README.md >/dev/null 2>&1 || true
```

Why this works:

```text
Only printer_data/config/ is staged.
docs/ and README.md changes are explicitly unstaged/restored before committing.
```

This prevents the Pi from committing documentation deletions.

---

### 3. Protected the docs folder from local cleanup

The script had cleanup commands that removed files from the local backup folder after syncing.

Old cleanup command:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' -exec rm -rf {} \;
```

New cleanup command:

```bash
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' ! -name 'docs' -exec rm -rf {} \;
```

The important addition is:

```bash
! -name 'docs'
```

This keeps the local `docs/` folder from being deleted by the backup script.

Both cleanup lines in `~/klipper-backup/script.sh` were patched.

Verified with:

```bash
grep -n "find \"\$backup_path\"" ~/klipper-backup/script.sh
```

Expected result:

```text
Both cleanup lines include: ! -name 'docs'
```

---

### 4. Added post-cleanup restore

After the final cleanup command, this was added so the local backup repo is left clean after the script runs:

```bash
# Leave the backup repo clean after the post-backup cleanup.
cd "$backup_path"
git restore . >/dev/null 2>&1 || true
```

Verified section near the bottom of the script:

```bash
# Remove files except .git folder after backup so that any file deletions can be logged on next backup
find "$backup_path" -maxdepth 1 -mindepth 1 ! -name '.git' ! -name 'README.md' ! -name '.gitmodules' ! -name 'docs' -exec rm -rf {} \;

# Leave the backup repo clean after the post-backup cleanup.
cd "$backup_path"
git restore . >/dev/null 2>&1 || true
```

---

## Pulling the Restored Docs to the Pi

After the GitHub docs were recreated, the Pi's local backup repo was updated:

```bash
cd ~/config_backup
git pull
```

Then the docs folder was verified:

```bash
ls -la docs
```

Expected files:

```text
configuration.md
future-ideas.md
project-history.md
punch-list.md
upgrades.md
```

---

## Final Verification Test

After patching, the backup script was tested manually:

```bash
cd ~/config_backup
git restore .
~/klipper-backup/script.sh
git status --short
ls -la docs
```

Successful result:

- `git status --short` printed nothing.
- `docs/` still existed.
- All documentation files were still present.
- The backup pushed normally.

Expected docs listing:

```text
configuration.md
future-ideas.md
project-history.md
punch-list.md
upgrades.md
```

This means the Mainsail `update_git` button should be safe to use again.

---

## If This Ever Happens Again

### Step 1 - Do not press the Mainsail backup button again yet

Stop and inspect from SSH first.

---

### Step 2 - Check the backup script staging behavior

```bash
grep -R "git add" ~/klipper-backup -n
```

Safe behavior should include:

```bash
git add -A printer_data/config/
```

Dangerous behavior:

```bash
git add .
```

If `git add .` is present, patch the script before running backups.

---

### Step 3 - Check that docs are protected from cleanup

```bash
grep -n "find \"\$backup_path\"" ~/klipper-backup/script.sh
```

Safe cleanup lines must include:

```bash
! -name 'docs'
```

---

### Step 4 - Check local Git status before pushing

```bash
cd ~/config_backup
git status --short
```

Danger signs:

```text
D docs/upgrades.md
D docs/configuration.md
D docs/future-ideas.md
D docs/project-history.md
D docs/punch-list.md
```

If docs are staged or shown as deleted, run:

```bash
git restore .
git status --short
```

Do not run the backup script until docs deletions are gone.

---

### Step 5 - Re-pull docs if needed

If `docs/` is missing locally but exists on GitHub:

```bash
cd ~/config_backup
git pull
ls -la docs
```

---

### Step 6 - Run a manual backup test

```bash
cd ~/config_backup
git restore .
~/klipper-backup/script.sh
git status --short
ls -la docs
```

Only use the Mainsail `update_git` button after this test succeeds.

---

## Current Project Rule

```text
The Raspberry Pi owns printer_data/config/ backups.
GitHub/ChatGPT owns docs/ and README.md documentation edits.
```

The Pi must never stage documentation changes or documentation deletions.

Any future Klipper-Backup update or reinstall should be checked against this page before using `update_git` again.
