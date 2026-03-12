---
name: x-push-skill
description: Push user-level skills prefixed with "x-" to a remote Git repository. Use this skill whenever the user says "push skills", "推送技能", "sync skills to git", "push x- skills", or wants to back up their custom skills to a remote repo. Also triggers when the user wants to publish or share their x- prefixed skills.
---

# Push x-Skills to Git

Push all `x-` prefixed skills from `~/.agents/skills/` to a remote Git repository, saving them under the repo's `skills/` directory.

## Default Configuration

- **Source**: `~/.agents/skills/x-*/` (all directories starting with `x-`)
- **Default repo**: `git@github.com:JinsYin/agent.git`
- **Target path in repo**: `skills/`

## Workflow

### 1. Parse Arguments

Check if the user specified a custom repository URL. If not, use the default `git@github.com:JinsYin/agent.git`.

Also check if the user wants to push specific skills (e.g., "push x-git-commit") or all x- skills (the default).

### 2. Discover x-Skills

```bash
ls -d ~/.agents/skills/x-*/
```

List all found skills and confirm with the user before pushing. Show each skill name so they know exactly what will be synced.

### 3. Clone or Update the Repo

Use a temporary working directory to avoid conflicts with any existing local checkout.

```bash
WORK_DIR=$(mktemp -d)
git clone --depth 1 <repo-url> "$WORK_DIR/repo"
```

If the clone fails (e.g., auth issue, network error), report the error clearly and stop.

### 4. Sync Skills

For each x-skill, copy its entire directory (SKILL.md and any bundled resources like scripts/, references/, assets/) into the repo's `skills/` directory:

```bash
mkdir -p "$WORK_DIR/repo/skills"

# For each x-skill directory
for skill_dir in ~/.agents/skills/x-*/; do
  skill_name=$(basename "$skill_dir")
  # Use rsync to mirror the directory, deleting files that no longer exist locally
  rsync -a --delete "$skill_dir" "$WORK_DIR/repo/skills/$skill_name/"
done
```

### 5. Commit and Push

```bash
cd "$WORK_DIR/repo"
git add skills/
# Check if there are actual changes
if git diff --cached --quiet; then
  echo "No changes to push — remote is already up to date."
else
  git commit -m "chore: sync x-skills from local"
  git push origin main
fi
```

If the default branch isn't `main`, detect it:

```bash
default_branch=$(git remote show origin | grep 'HEAD branch' | awk '{print $NF}')
git push origin "$default_branch"
```

### 6. Clean Up and Report

```bash
rm -rf "$WORK_DIR"
```

Report to the user:
- Which skills were pushed (list them)
- The repo and branch they were pushed to
- Whether there were actual changes or everything was already in sync

## Edge Cases

- **No x-skills found**: Tell the user there are no x- prefixed skills in `~/.agents/skills/` and stop.
- **Git auth failure**: Suggest the user check their SSH key or credentials. Don't retry automatically.
- **Merge conflicts**: Since we do a fresh shallow clone each time, conflicts shouldn't occur. If push is rejected (remote has newer commits), pull first with `git pull --rebase` then retry the push once.
- **Custom branch**: If the user specifies a branch, use that instead of the default branch.
- **Selective push**: If the user names specific skills (e.g., "push x-git-commit and x-zh-to-en"), only sync those instead of all x- skills.
