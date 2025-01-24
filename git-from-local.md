# Git Setup Guide: Local → GitHub

## **Step 1: Initialize Git Locally**
```bash
# Navigate to your project folder
cd /path/to/project

# Initialize Git
git init

# Stage all files
git add .

# Commit
git commit -m "Initial commit"
```

---

## **Step 2: Create a GitHub Repository**
1. Go to [GitHub](https://github.com/new).
2. **Name**: Match your project (e.g., `my-repo`).
3. **Do NOT** add README/.gitignore/license.
4. Click **Create Repository**.

---

## **Step 3: Link Local Repo to GitHub**
```bash
# Add the remote (replace with your URL)
git remote add origin https://github.com/your-username/my-repo.git

# Verify
git remote -v
```

---

## **Step 4: Push to GitHub**
```bash
# Rename branch to 'main'
git branch -M main

# Push
git push -u origin main
```

---

## **Troubleshooting**
### Force-push (overwrite remote):
```bash
git push -u origin main --force
```

### Fix branch name mismatch:
```bash
git branch -M main
```

---

## **Best Practices**
- Use `.gitignore` to exclude files like `__azurite_db_*` or `.env`.
- Avoid `--force` unless absolutely necessary.