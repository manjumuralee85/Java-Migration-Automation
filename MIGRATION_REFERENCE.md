# Java Migration Workflow - Quick Reference Card

## 🎯 What Is This?
An **automated GitHub Actions workflow** that migrates your Java 8 code to Java 11 (and beyond) using OpenRewrite for intelligent code refactoring.

---

## 🚀 How to Run (3 Steps)

### Step 1: Go to GitHub
```
Your Repository → Actions Tab → "Java 8 to 11 Migration"
```

### Step 2: Click Run
```
Click "Run workflow" → Select branch (main) → Click "Run workflow"
```

### Step 3: Wait & Review
```
Watch logs → Find PR → Review files → Approve & Merge
```

**Total time:** 3-5 minutes ⏱️

---

## 📊 Workflow Overview

```
┌─ TRIGGER ─────────────────────────────────┐
│ Manual (click button)                      │
│ Scheduled (every Monday 2 AM UTC)          │
│ API (gh workflow run)                      │
└────────────────────────────────────────────┘
              ↓
┌─ SETUP ───────────────────────────────────┐
│ 1. Checkout code from GitHub               │
│ 2. Install Java 11                         │
│ 3. Download dependencies (cached)          │
└────────────────────────────────────────────┘
              ↓
┌─ MODERNIZE ────────────────────────────────┐
│ 1. Update pom.xml (java version)           │
│ 2. Run OpenRewrite recipes                 │
│ 3. Auto-refactor code                      │
└────────────────────────────────────────────┘
              ↓
┌─ BUILD & TEST ─────────────────────────────┐
│ 1. Maven clean verify                      │
│ 2. Compile with Java 11                    │
│ 3. Run all tests                           │
│ 4. Check for errors ✅ or ❌               │
└────────────────────────────────────────────┘
              ↓
┌─ COMMIT & PR ──────────────────────────────┐
│ 1. Create branch (auto/java-11-upgrade)    │
│ 2. Commit changes                          │
│ 3. Push to GitHub                          │
│ 4. Create Pull Request                     │
│ 5. Mark DRAFT if build failed              │
│ 6. Mark READY if build passed              │
└────────────────────────────────────────────┘
              ↓
         You Review PR
              ↓
         Approve & Merge
```

---

## 📝 PR Checklist

What to check before merging:

- [ ] **Title** says "Migrate Java 8 to Java 11"
- [ ] **All checks pass** (green ✅)
- [ ] **Tests pass** (zero failures)
- [ ] **Files look right** (only pom.xml and .java files)
- [ ] **No huge diffs** (should be under 1000 lines total)
- [ ] **Code makes sense** (review a few files)

---

## 🛠️ Files Created

```
.github/workflows/
├─ java-migration.yml                ← Main workflow (Java 8→11)
└─ java-11-to-17-migration.yml       ← Template for Java 11→17

rewrite.xml                           ← Tells OpenRewrite what to do

MIGRATION_QUICKSTART.md               ← Start here (5 min read)
JAVA_MIGRATION_GUIDE.md               ← Detailed guide (15 min)
MIGRATION_ARCHITECTURE.md             ← Advanced topics (30 min)
MIGRATION_SUMMARY.md                  ← Overview
```

---

## 🔄 For Sequential Migrations

### Java 8 → 11 → 17 → 21 → 25

After merging Java 8→11 PR:

```
1. Wait for testing (1-2 weeks)
2. Trigger Java 11→17 workflow
3. Same process, different target
4. Repeat for 21, 25
```

---

## ⚠️ If Build Fails

1. **Check logs** → Actions → click run → scroll down
2. **Find error** → Copy error message
3. **Fix manually** → Edit file that failed
4. **Commit fix** → `git add -A && git commit`
5. **Push fix** → `git push` (to same PR branch)
6. **Re-run tests** → Workflow auto-retries
7. **Merge when ready** → Click "Merge" button

---

## 🎯 Common Issues & Fixes

| Issue | Fix |
|-------|-----|
| "Workflow not found" | Push `.github/workflows/java-migration.yml` to GitHub |
| "Permission denied" | Settings → Actions → Enable "Read and write permissions" |
| "Build failed" | Check logs, fix error, commit, push (PR auto-updates) |
| "No PR created" | Check if there were changes (empty diffs = no PR) |
| "Java not found" | GitHub runners have Java pre-installed, try re-running |

---

## 📊 What Gets Changed

### Files Modified
- **pom.xml** - Java version property updated
- **\*.java** - Code modernized by OpenRewrite

### Example Changes
```java
// Before (Java 8)
for (String name : names) {
    System.out.println(name);
}

// After (Java 11, modernized)
names.forEach(System.out::println);
```

### No Changes To
- Test files (unless code changed)
- XML configs (except pom.xml)
- Resource files
- Build artifacts

---

## 🕐 Timing

| Phase | Time |
|-------|------|
| Setup (Java, Maven) | 30s |
| OpenRewrite | 60s |
| Build | 120s |
| Tests | 60s |
| Commit & PR | 30s |
| **Total** | **4-5 min** |

First run: 5-10 min (downloads Java, deps)  
Subsequent runs: 3-5 min (uses cache)

---

## ✅ Success Indicators

- ✅ Workflow runs without errors
- ✅ PR is created
- ✅ All tests pass (CI shows green)
- ✅ Code compiles with Java 11
- ✅ You can review changes
- ✅ You can merge without issues

---

## 🔐 Security & Permissions

This workflow needs:
- ✅ **Read code** - Standard repo access
- ✅ **Write code** - To commit changes
- ✅ **Create PR** - To open pull requests

Permissions are scoped to your repo only (safe!).

---

## 📚 Learn More

| Want to... | Read |
|-----------|------|
| Run workflow? | MIGRATION_QUICKSTART.md |
| Understand steps? | JAVA_MIGRATION_GUIDE.md |
| Scale to 50 apps? | MIGRATION_ARCHITECTURE.md |
| Learn about tools? | JAVA_MIGRATION_GUIDE.md (Tools section) |
| Troubleshoot? | MIGRATION_QUICKSTART.md (Troubleshooting) |

---

## 🎯 Next 5 Minutes

1. **Push files** → `git push origin main`
2. **Enable Actions** → GitHub Settings
3. **Trigger workflow** → GitHub Actions tab
4. **Watch run** → Click on the running workflow
5. **Review PR** → Find the Pull Request

---

## 📞 Need Help?

1. Check MIGRATION_QUICKSTART.md
2. Check workflow logs (Actions tab)
3. Review error message in PR
4. Check GitHub Actions documentation

---

## 🎉 You're Ready!

Everything is set up. Time to migrate! 🚀

**Start:** Go to Actions tab → Click "Run workflow"

---

**One-liner:** This automated GitHub Actions workflow upgrades your Java code from Java 8 to Java 11 using OpenRewrite, then creates a PR for you to review and merge.

**Time to merge:** Usually 5-10 minutes from trigger to ready-to-merge.

