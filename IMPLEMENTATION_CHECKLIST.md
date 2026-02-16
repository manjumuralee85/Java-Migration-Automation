# Java Migration Implementation Checklist

## ✅ Pre-Flight Checklist

Before running the workflow, ensure:

- [ ] Repository is pushed to GitHub
- [ ] GitHub Actions are enabled
- [ ] You have write access to the repo
- [ ] `.github/workflows/java-migration.yml` exists
- [ ] `rewrite.xml` exists
- [ ] `pom.xml` has OpenRewrite plugin
- [ ] All docs are in place

---

## 📋 Files to Verify

Run these commands to verify setup:

```bash
# Check workflow file exists
ls -la .github/workflows/java-migration.yml

# Check OpenRewrite config exists
ls -la rewrite.xml

# Check pom.xml has OpenRewrite plugin
grep -A 5 "rewrite-maven-plugin" pom.xml

# Check documentation files
ls -la MIGRATION*.md
ls -la JAVA_MIGRATION_GUIDE.md
```

**Expected output:**
```
✅ .github/workflows/java-migration.yml (8.2K)
✅ rewrite.xml (1.2K)
✅ pom.xml (contains rewrite-maven-plugin)
✅ All documentation files present
```

---

## 🚀 Implementation Steps

### Step 1: Push Files to GitHub ✅
```bash
cd /Users/A-9740/Services/spring-petclinic

# Stage all new files
git add .github/workflows/
git add rewrite.xml
git add pom.xml
git add *.md

# Commit
git commit -m "chore: add Java migration workflow with OpenRewrite"

# Push
git push origin main  # or your default branch

# Verify on GitHub
# Go to repo → you should see new files
```

**Verify:** Check GitHub repo, confirm files are visible

---

### Step 2: Enable GitHub Actions ✅

**Via GitHub UI:**
1. Go to your repo on GitHub
2. Click **Settings** tab
3. Left sidebar: Click **Actions** → **General**
4. Scroll to "Workflow permissions"
5. Select ☑️ **Read and write permissions**
6. Click **Save**

**Verify:** Green checkmark ✅ shows permissions enabled

---

### Step 3: Test Permissions ✅

```bash
# List workflows to verify they're readable
gh workflow list

# Expected output:
# ID    Name                          State       ...
# 12345 Java 8 to 11 Migration        active      ...
```

**Verify:** Workflow shows in the list

---

### Step 4: Run Workflow Manually ✅

**Option A: Via GitHub UI (Easiest)**
1. Go to repo → **Actions** tab
2. Find **"Java 8 to 11 Migration"** in left sidebar
3. Click it
4. Click blue **"Run workflow"** button
5. Select branch: **main** (or your default)
6. Click **"Run workflow"**

**Option B: Via GitHub CLI**
```bash
gh workflow run java-migration.yml
```

**Option C: Via REST API**
```bash
curl -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  -H "Authorization: token YOUR_GITHUB_TOKEN" \
  https://api.github.com/repos/OWNER/REPO/actions/workflows/java-migration.yml/dispatches \
  -d '{"ref":"main"}'
```

**Verify:** Workflow shows as "In progress" (yellow 🟡 dot)

---

### Step 5: Monitor Workflow ✅

1. Go to **Actions** tab
2. Click the running workflow
3. Watch logs in real-time:
   - Yellow 🟡 = In progress
   - Green ✅ = Success
   - Red ❌ = Failed

**Expected log output:**
```
📥 Checkout Code ........................ ✅ 5s
☕ Setup Java 11 ........................ ✅ 30s
🔧 Update pom.xml ....................... ✅ 2s
🤖 Run OpenRewrite ....................... ✅ 60s
🏗️  Maven Build & Test .................. ✅ 120s
⚙️  Configure Git ....................... ✅ 5s
📤 Create Branch and Commit ............. ✅ 10s
🚀 Create Pull Request .................. ✅ 5s

✅ Workflow completed successfully!
```

**Verify:** Workflow completes without errors

---

### Step 6: Review Pull Request ✅

1. Go to **Pull requests** tab
2. Find **"Migrate Java 8 to Java 11"**
3. Click on it to open

**Check these things:**
- [ ] PR title: "🔄 chore: Migrate Java 8 to Java 11"
- [ ] Status: Shows ✅ green checkmarks
- [ ] "Files changed" tab: Review modifications
- [ ] Commits: Shows your automation commit
- [ ] Build status: All checks passing

**Example PR view:**
```
Title: 🔄 chore: Migrate Java 8 to Java 11
Status: ✅ Ready to merge

Conversation | Commits | Checks | Files changed (3)
└─ pom.xml (+2, -2)
└─ src/main/java/App.java (+15, -10)
└─ src/main/java/Service.java (+8, -5)

✅ All checks passed
[Approve] [Request changes] [Comment]
```

**Verify:** PR shows green ✅ and has changes

---

### Step 7: Review Code Changes ✅

Click **"Files changed"** tab and review:

```diff
File: pom.xml
- <java.version>8</java.version>
+ <java.version>11</java.version>

File: src/main/java/App.java
- for(String name : list) {
-     System.out.println(name);
- }
+ list.forEach(System.out::println);

File: src/main/java/Service.java
- Optional.ofNullable(value).orElse(null)
+ value  // Simplified by OpenRewrite
```

**Checklist:**
- [ ] Changes look reasonable
- [ ] Only expected files modified
- [ ] No suspicious files changed
- [ ] Code modernizations make sense

---

### Step 8: Run Tests Locally (Optional) ✅

To verify PR works locally:

```bash
# Fetch the PR branch
git fetch origin auto/java-11-upgrade-*

# Check out the branch
git checkout auto/java-11-upgrade-*

# Run tests
mvn test

# Expected: All tests pass
# [INFO] Tests run: 42, Failures: 0, Errors: 0, Skipped: 0
```

**Verify:** Tests pass locally

---

### Step 9: Approve Pull Request ✅

1. On PR page, click **"Approve"** button
2. Click **"Approve and comment"** (optional comment)
3. Confirmation message appears

**Verify:** Shows "✅ Approved" next to your name

---

### Step 10: Merge Pull Request ✅

1. Click **"Merge pull request"** button
2. Select merge strategy:
   - "Squash and merge" (recommended - cleaner history)
   - "Create a merge commit" (keeps all commits)
   - "Rebase and merge" (linear history)
3. Click **"Confirm merge"**
4. Click **"Delete branch"** (optional, recommended)

**Verify:** PR shows "✅ Merged" with timestamp

---

### Step 11: Verify Main Branch ✅

```bash
# Pull latest main branch
git pull origin main

# Verify Java version updated
grep -A 1 "<java.version>" pom.xml

# Expected output:
# <java.version>11</java.version>

# Verify code compiled
mvn clean compile

# Expected: BUILD SUCCESS
```

**Verify:** pom.xml shows Java 11, build succeeds

---

## 📊 Success Criteria

### ✅ Workflow Success
- Workflow runs without crashing
- PR is created
- All CI checks pass (green ✅)

### ✅ Code Quality
- Tests pass (0 failures)
- Code compiles (0 errors)
- Code style is consistent
- Minimal warnings

### ✅ Changes Appropriate
- Only pom.xml and .java files changed
- Changes are modernization (loops, generics, etc)
- No behavioral changes in business logic
- Dependencies updated appropriately

---

## 🛠️ Troubleshooting Checklist

### Problem: "Workflow file not found"
- [ ] Is `.github/workflows/java-migration.yml` committed?
- [ ] Is it pushed to GitHub?
- [ ] Can you see it in the repo on GitHub?
- **Fix:** Commit and push the file, wait 30s, refresh

### Problem: "Permission denied"
- [ ] Did you enable "Read and write permissions"?
- [ ] Is the GitHub token valid?
- [ ] Are you the repo owner/admin?
- **Fix:** Go to Settings → Actions → Enable permissions

### Problem: "Build failed"
- [ ] Check the logs (Actions → click run → scroll down)
- [ ] What's the exact error message?
- [ ] Is it a compilation error or test failure?
- **Fix:** Check MIGRATION_QUICKSTART.md troubleshooting section

### Problem: "No PR created"
- [ ] Did the workflow complete?
- [ ] Were there any changes made?
- [ ] Check the workflow logs
- **Fix:** If no changes, that's OK - nothing to migrate

### Problem: "Java 11 not found"
- [ ] GitHub runners have Java pre-installed
- [ ] Try re-running the workflow
- [ ] Check that Java version is specified: `java-version: '11'`
- **Fix:** GitHub handles Java setup, should work automatically

---

## 📈 Next Steps (After First Success)

- [ ] Repeat for Java 11 → 17 migration
- [ ] Test Java 17 build in staging
- [ ] Plan Java 21 migration
- [ ] Document any custom fixes made
- [ ] Share learnings with team
- [ ] Scale to 50+ applications

---

## 🎯 Expected Timeline

| Phase | Time | Status |
|-------|------|--------|
| Setup & documentation creation | 30 min | ✅ Done |
| Push to GitHub | 5 min | ⏳ Do now |
| Enable Actions | 5 min | ⏳ Do now |
| Run workflow | 3-5 min | ⏳ Do now |
| Review PR | 10 min | ⏳ Do next |
| Approve & merge | 5 min | ⏳ Do next |
| **Total** | **~1 hour** | |

---

## ✅ Final Checklist

Before you declare "success":

- [ ] Workflow pushed to GitHub
- [ ] Actions permissions enabled
- [ ] Workflow triggered successfully
- [ ] PR created
- [ ] PR reviewed
- [ ] PR approved
- [ ] PR merged
- [ ] Main branch has Java 11 changes
- [ ] Local build succeeds
- [ ] Tests pass locally
- [ ] Documentation reviewed
- [ ] Team informed

---

## 🎉 You're Done!

When all checkboxes are complete, you have successfully:

✅ Automated Java 8 → 11 migration  
✅ Set up OpenRewrite for code modernization  
✅ Created reusable workflow templates  
✅ Built a scalable system for 50+ apps  

**Next:** Plan Java 11 → 17 → 21 → 25 migrations!

---

**Estimated total time to complete all steps: 30-45 minutes** ⏱️

**Questions?** See MIGRATION_QUICKSTART.md or JAVA_MIGRATION_GUIDE.md

Happy migrating! 🚀

