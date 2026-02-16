# Quick Start: Running the Java Migration Workflow

## 🚀 Get Started in 5 Minutes

### Step 1: Ensure Files Are in Place ✅

Check that these files exist in your repository:

```bash
# Run this command in your terminal
ls -la .github/workflows/java-migration.yml
ls -la pom.xml
ls -la rewrite.xml
```

**Expected output:**
```
-rw-r--r--  1 user  staff  8.2K Feb 16 12:00 .github/workflows/java-migration.yml ✅
-rw-r--r--  1 user  staff  12K Feb 16 12:00 pom.xml ✅
-rw-r--r--  1 user  staff  1.2K Feb 16 12:00 rewrite.xml ✅
```

---

### Step 2: Push Files to GitHub 📤

If you haven't pushed yet:

```bash
# Navigate to your repo directory
cd /Users/A-9740/Services/spring-petclinic

# Add all new files
git add .github/workflows/java-migration.yml
git add pom.xml
git add rewrite.xml
git add JAVA_MIGRATION_GUIDE.md

# Commit
git commit -m "chore: add Java migration workflow with OpenRewrite"

# Push to GitHub
git push origin main  # or your default branch
```

---

### Step 3: Enable GitHub Actions (if needed) 🔧

1. Go to your GitHub repository
2. Click **Settings** tab
3. Click **Actions** → **General**
4. Under "Workflow permissions", select:
   - ☑️ **Read and write permissions**
5. Click **Save**

---

### Step 4: Trigger the Workflow 🎯

#### Option A: Via GitHub UI (Easiest)

1. Go to your repository on GitHub
2. Click **Actions** tab
3. Find **"Java 8 to 11 Migration"** in the left sidebar
4. Click on it
5. Click blue **"Run workflow"** button
6. Select your branch (usually `main` or `master`)
7. Click **"Run workflow"**

**You should see:**
```
Workflow run triggered ✅
🟡 In progress...
```

---

#### Option B: Via GitHub CLI

```bash
# Install GitHub CLI if you haven't
# macOS: brew install gh
# or visit: https://cli.github.com

# Authenticate (first time only)
gh auth login

# Trigger the workflow
gh workflow run java-migration.yml

# View run status
gh run list
```

---

#### Option C: Via Git (Schedule It)

The workflow runs automatically every **Monday at 2 AM UTC**.

To test the schedule:
```bash
# Edit the file to run in 2 minutes from now
# Change cron time, commit, and the workflow will run at that time
```

---

### Step 5: Monitor the Run 📊

1. Click **Actions** tab
2. Click the running workflow (yellow 🟡 dot)
3. Watch the logs in real-time:

```
Workflow: Java 8 to 11 Migration
Status: 🟡 In progress

Jobs:
├─ migrate
│  ├─ 📥 Checkout Code ......... ✅ (5s)
│  ├─ ☕ Setup Java 11 ......... ⏳ (30s)
│  ├─ 🔧 Update pom.xml ....... ⏳ (2s)
│  ├─ 🤖 Run OpenRewrite ....... ⏳ (60s)
│  └─ 🏗️  Build & Test ......... ⏳ (120s)
```

---

### Step 6: Review the Pull Request 📋

When the workflow completes:

1. Go to **Pull requests** tab
2. Find **"Migrate Java 8 to Java 11"**
3. Review the PR:
   - **Files changed**: See what was modified
   - **Commits**: See the commit message
   - **Checks**: All should be ✅ green

**Example PR:**
```
Title: 🔄 chore: Migrate Java 8 to Java 11

✅ All checks passed

Files changed:
├─ pom.xml
│  └─ Updated java.version to 11
├─ src/main/java/.../*.java
│  └─ Code modernized by OpenRewrite
└─ ...

Add your review:
[ ] Comment
[✅] Approve
[ ] Request changes
```

---

### Step 7: Merge the PR ✅

1. **Review changes** (read the PR description)
2. **Run tests locally** (optional but recommended):
   ```bash
   git pull origin auto/java-11-upgrade-*
   mvn test
   ```
3. Click **"Approve"** button
4. Click **"Merge pull request"**
5. Click **"Confirm merge"**
6. Click **"Delete branch"** (optional but recommended)

---

## 🔍 What to Look For in the PR

### ✅ Good Signs (Approve & Merge!)
- All checks are green ✅
- Tests pass
- Only `pom.xml` and Java source files changed
- Changes look reasonable (modernized code)

### ⚠️ Warning Signs (Request Changes)
- Test failures ❌
- Compilation errors
- Unusual file changes (e.g., `.gitignore`)
- Too many changes (diff > 1000 lines)

### 🆘 If Something Goes Wrong
1. Check the **Workflow logs** (Actions tab → click run → view logs)
2. Look for red ❌ errors in the "Build & Test" step
3. Create a comment on the PR:
   ```
   @github-actions This failed because:
   - [reason]
   - [next steps]
   ```

---

## 🛠️ Troubleshooting Checklist

| Issue | Solution |
|-------|----------|
| "Workflow not found" | Ensure `.github/workflows/java-migration.yml` is committed to GitHub |
| "Permission denied" | Go to Settings → Actions → Enable "Read and write permissions" |
| "Build failed" | Check workflow logs, fix issues in PR branch, commit fixes |
| "No PR created" | Check if branch exists and has changes (no error = no changes) |
| "Java 11 not found" | Workflow uses GitHub's runner which has Java pre-installed |

---

## 📚 Files Created

Here's what was set up for you:

```
spring-petclinic/
├─ .github/
│  └─ workflows/
│     └─ java-migration.yml          ← GitHub Actions workflow
├─ pom.xml                           ← Updated with OpenRewrite plugin
├─ rewrite.xml                       ← OpenRewrite configuration
└─ JAVA_MIGRATION_GUIDE.md           ← Detailed guide (this file)
```

---

## 📖 Learn More

- **GitHub Actions:** https://docs.github.com/en/actions
- **OpenRewrite:** https://docs.openrewrite.org/
- **Maven:** https://maven.apache.org/
- **Java 11 Features:** https://openjdk.org/projects/jdk/11/

---

## 🎉 Success!

You now have a **fully automated Java migration workflow** that will:

1. ✅ Update your code to Java 11
2. ✅ Run OpenRewrite recipes
3. ✅ Build and test everything
4. ✅ Create a PR for review
5. ✅ Handle failures gracefully

Next: **Trigger the workflow and watch it work!** 🚀

---

## 💡 Pro Tips

- **Run on schedule:** Workflow runs every Monday (customizable)
- **Batch migrations:** Use for Java 8→11, 11→17, 17→21, 21→25
- **Monitor PRs:** Set up email notifications for new PRs
- **Custom recipes:** Add your own OpenRewrite recipes to `rewrite.xml`

---

Happy migrating! 🎉

