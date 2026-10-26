# How This Repo Is Built

Every lesson is developed the way real engineering teams work: on its own branch, with small commits, merged through a pull request (PR).

## The workflow

```
main ──●──────────────●──────────────●──
        \            / \            /
         ●──●──●───PR   ●──●──●───PR
       lesson-02        lesson-03
```

### 1. Start from an up-to-date main
```bash
git switch main
git pull
git switch -c lesson-03-services
```

### 2. Work and commit in small steps
```bash
git add 03-services/service.yaml
git commit -m "Add ClusterIP Service for nginx"
```

### 3. Push the branch
```bash
git push -u origin lesson-03-services
```

### 4. Open a pull request on GitHub
Click **Compare & pull request**, describe what changed and what was learned, review the **Files changed** tab, then merge and delete the branch.

### 5. Sync and clean up
```bash
git switch main
git pull
git branch -d lesson-03-services
```

## Naming conventions

**Branches:** lowercase with dashes
- `lesson-03-services`
- `docs-beginner-guide`
- `fix-readme-typo`

**Commit messages:** start with a verb, describe what changed, keep under ~60 characters
- ✅ `Add Service manifest exposing nginx on port 80`
- ✅ `Fix indentation in deployment.yaml`
- ❌ `update`, `changes`, `stuff`

## Want to follow along with your own copy?
1. Click **Fork** at the top of this repo on GitHub
2. Clone your fork, replacing `your-username` with your GitHub username:
   ```bash
   git clone https://github.com/your-username/k8s-from-zero.git
   ```
3. Work through the lessons and commit your own notes
