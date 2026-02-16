# Java 8 to Java 11 Migration Workflow Guide

## 📋 Table of Contents
1. [What is This Workflow?](#what-is-this-workflow)
2. [How It Works](#how-it-works)
3. [Step-by-Step Explanation](#step-by-step-explanation)
4. [How to Run the Workflow](#how-to-run-the-workflow)
5. [Understanding the PR Output](#understanding-the-pr-output)
6. [Troubleshooting](#troubleshooting)
7. [What's Inside Each Tool](#whats-inside-each-tool)

---

## What is This Workflow?

This GitHub Actions workflow **automatically migrates spring-petclinic from Java 8 to Java 11**. It:

✅ Updates your code to be Java 11 compatible  
✅ Uses OpenRewrite to automatically refactor code  
✅ Builds and tests your application  
✅ Creates a Pull Request for review  
✅ Handles build failures gracefully  

**Why is this useful?**
- Java 8 is old and no longer supported (ended in December 2030)
- Java 11 is an LTS (Long-Term Support) release with better performance
- Migrating manually is time-consuming and error-prone
- This workflow does it **automatically** in minutes

---

## How It Works

```
┌─────────────────────────────────────────────────┐
│ 1. You trigger workflow (or schedule runs)      │
└──────────────┬──────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────┐
│ 2. Checkout code from GitHub                    │
└──────────────┬──────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────┐
│ 3. Set up Java 11 (target version)              │
└──────────────┬──────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────┐
│ 4. Update pom.xml for Java 11                   │
└──────────────┬──────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────┐
│ 5. Run OpenRewrite (automatic code fixes)       │
└──────────────┬──────────────────────────────────┘
               │
┌──────────────▼──────────────────────────────────┐
│ 6. Build & Test (Maven verify)                  │
└──────────────┬──────────────────────────────────┘
               │
        ┌──────┴──────────────┐
        │                     │
    ✅ SUCCESS           ❌ FAILURE
        │                     │
        │      ┌──────────────┘
        │      │
        │  ┌───▼──────────────────────────┐
        │  │ 7. Auto-fix (if needed)      │
        │  └───┬──────────────────────────┘
        │      │
        └──┬───┤
           │
        ┌──▼──────────────────────────────┐
        │ 8. Commit changes to new branch  │
        └──┬──────────────────────────────┘
           │
        ┌──▼──────────────────────────────┐
        │ 9. Create Pull Request for       │
        │    review and merge             │
        └────────────────────────────────┘
```

---

## Step-by-Step Explanation

### Step 1: Trigger the Workflow
You have 3 ways to trigger this workflow:

**Option A: Manual Trigger (via GitHub UI)**
1. Go to your GitHub repo
2. Click "Actions" tab
3. Find "Java 8 to 11 Migration" workflow
4. Click "Run workflow" → "Run workflow"

**Option B: Schedule (Automatic)**
The workflow runs automatically every **Monday at 2 AM UTC** (you can change this)

**Option C: Git Command Line**
```bash
gh workflow run java-migration.yml
```

---

### Step 2: Checkout Code
```yaml
- uses: actions/checkout@v4
```
**What it does:** Downloads your code from GitHub to the GitHub Actions runner (a machine in the cloud)

**Why:** The workflow needs your source code to read it, modify it, and test it

---

### Step 3: Set Up Java 11
```yaml
- uses: actions/setup-java@v4
  with:
    distribution: 'temurin'
    java-version: '11'
```
**What it does:** Installs Java 11 JDK on the runner machine

**What is "Temurin"?** It's a distribution of OpenJDK maintained by the Eclipse Adoptium project. Think of it as a reliable, well-maintained version of Java.

**Why Java 11 first?** We set up Java 11 so the compiler will use Java 11 rules and features from the start.

---

### Step 4: Update pom.xml for Java 11
```bash
sed -i 's/<java.version>.*<\/java.version>/<java.version>11<\/java.version>/g' pom.xml
```
**What it does:** Replaces the `<java.version>` property in `pom.xml` with `11`

**What is pom.xml?** It's Maven's configuration file that tells Maven:
- What version of Java to compile for
- Which libraries (dependencies) to use
- How to build the project

**Example before:**
```xml
<java.version>8</java.version>
```

**Example after:**
```xml
<java.version>11</java.version>
```

---

### Step 5: Run OpenRewrite (Automatic Code Modernization)
```bash
mvn org.openrewrite:rewrite-maven-plugin:run
```
**What it does:** Runs OpenRewrite recipes that automatically update your code

**What is OpenRewrite?** It's a tool that **automatically refactors code**. Think of it like a smart "Find & Replace" that understands Java code.

**Example transformations OpenRewrite makes:**
- Changes Java 8 lambda syntax to modern syntax
- Removes deprecated methods
- Updates API calls to newer versions
- Cleans up imports

**Recipes used in this workflow:**
| Recipe Name | What it does |
|---|---|
| `Java8toJava11` | Upgrade Java 8 code to Java 11 |
| `UpgradeJava` | General Java modernization |
| `RemoveUnusedImports` | Clean up unused imports |
| `AutoFormat` | Format code consistently |

---

### Step 6: Build & Test (Maven Verify)
```bash
mvn clean verify
```
**What it does:** 
1. **clean**: Removes old build artifacts
2. **verify**: Compiles code + runs tests + runs integration tests

**Why this step is critical:**
- Catches compilation errors early
- Ensures tests still pass after changes
- Validates the app still works

**If it fails:** The workflow moves to "auto-fix" mode

---

### Step 7: Auto-Fix (If Build Failed)
```bash
mvn compile -Xlint:all
```
**What it does:** Recompiles with verbose output to show errors

**Why "auto-fix"?** If OpenRewrite didn't catch everything, this helps identify remaining issues:
- Compiler errors
- Deprecated API warnings
- Compatibility issues

The workflow creates a **draft PR** so you can manually review and fix issues.

---

### Step 8: Commit Changes
```bash
git config user.name "github-actions[bot]"
git add -A
git commit -m "chore: migrate Java 8 to Java 11"
git push
```
**What it does:** 
1. Configures Git to "sign" commits as `github-actions[bot]`
2. Adds all changed files (`-A`)
3. Creates a commit with a message
4. Pushes to a new branch

**Example branch name:** `auto/java-11-upgrade-1707123456`

---

### Step 9: Create Pull Request
```yaml
- uses: peter-evans/create-pull-request@v5
```
**What it does:** Creates a GitHub Pull Request with:
- **Title:** "🔄 chore: Migrate Java 8 to Java 11"
- **Description:** Detailed info about changes
- **Labels:** `automation, jdk-upgrade, java-11`
- **Draft status:** Draft if build failed, ready if succeeded

**What you do next:** Review the PR, run additional tests, and merge if approved

---

## How to Run the Workflow

### Method 1: Manually (Recommended for First Run)

1. **Go to GitHub**
   - Open your repository in GitHub
   - Click the "Actions" tab

2. **Find the Workflow**
   - Look for "Java 8 to 11 Migration" in the list
   - Click it

3. **Run Workflow**
   - Click the "Run workflow" button
   - Select branch (usually `main` or `master`)
   - Click "Run workflow"

4. **Watch Progress**
   - You'll see a yellow dot next to the run (in progress)
   - Click on the run to see live logs
   - Wait for it to complete (usually 2-5 minutes)

5. **Check Results**
   - Go to the "Pull requests" tab
   - Find the PR named "Migrate Java 8 to Java 11"
   - Review changes

---

### Method 2: Automatic (Scheduled)

The workflow runs **automatically every Monday at 2 AM UTC**.

To change the schedule, edit `.github/workflows/java-migration.yml`:
```yaml
schedule:
  - cron: '0 2 * * 1'  # Monday, 2 AM UTC
```

**Cron Format:** `minute hour day-of-month month day-of-week`
- `0` = minute 0
- `2` = hour 2 (2 AM)
- `*` = any day of month
- `*` = any month
- `1` = Monday

**Common schedules:**
- `'0 2 * * 1'` → Every Monday 2 AM
- `'0 2 * * 0'` → Every Sunday 2 AM
- `'0 0 * * *'` → Every day at midnight

---

### Method 3: Command Line
If you have GitHub CLI installed:
```bash
gh workflow run java-migration.yml
```

---

## Understanding the PR Output

After the workflow runs, you'll see a PR with:

### ✅ If Build Succeeded (Ready PR)
```
Title: 🔄 chore: Migrate Java 8 to Java 11

✅ All checks passed
├── Build: SUCCESS
├── Tests: PASSED
└── OpenRewrite: Applied

Files changed:
├── pom.xml (+2 lines, -2 lines)
├── src/main/java/Foo.java (+10 lines)
└── ...
```

**What to do:** Review the "Files changed" tab, approve, and merge!

---

### ⚠️ If Build Failed (Draft PR)
```
Title: 🔄 chore: Migrate Java 8 to Java 11 [DRAFT]

❌ Build failed
├── Build: FAILURE
├── Tests: 3 failed
└── Manual review needed

Draft PR: Requires manual fixes

Check logs for:
✗ Compilation errors
✗ Test failures
✗ Dependency conflicts
```

**What to do:**
1. Click "View logs" to see error details
2. Fix issues manually in the branch
3. Commit fixes: `git commit -m "fix: resolve Java 11 issues"`
4. Push: `git push`
5. Mark PR as "Ready for review"

---

## Troubleshooting

### ❌ Problem: "Build failed - cannot find symbol"

**Cause:** OpenRewrite didn't fully update the code

**Solution:**
1. Check the PR logs for the specific error
2. Manually edit the file
3. Commit and push to the PR branch
4. The PR updates automatically

**Example:**
```java
// ❌ Before (won't compile on Java 11)
@Deprecated
public void oldMethod() { }

// ✅ After (compiles on Java 11)
@Deprecated(forRemoval = true)
public void oldMethod() { }
```

---

### ❌ Problem: "Test failed - NPE in SomeClass"

**Cause:** Code behavior changed after OpenRewrite refactoring

**Solution:**
1. Run tests locally: `mvn test`
2. Debug the failing test
3. Fix the code
4. Commit: `git commit -m "fix: resolve test failure"`
5. Push to PR branch

---

### ❌ Problem: "GitHub Actions failed - no permission"

**Cause:** GitHub token doesn't have write permissions

**Solution:**
1. Go to repository Settings
2. Click "Actions" → "General"
3. Scroll to "Workflow permissions"
4. Select "Read and write permissions"
5. Re-run the workflow

---

### ❌ Problem: "Workflow file not found"

**Cause:** `.github/workflows/java-migration.yml` doesn't exist

**Solution:**
Make sure you created the file at the correct location:
```
your-repo/
  ├── .github/
  │   └── workflows/
  │       └── java-migration.yml  ✅ MUST be here
  └── ...
```

---

## What's Inside Each Tool

### 1. GitHub Actions
**What it is:** GitHub's built-in automation tool

**How it works:**
- Responds to triggers (manual, schedule, push)
- Runs jobs in a cloud environment
- Supports 100+ pre-built actions
- Logs all steps

**Cost:** Free (up to 2,000 minutes/month for private repos)

---

### 2. OpenRewrite
**What it is:** Automated code refactoring engine

**How it works:**
- Defines "recipes" (transformation rules)
- Parses Java source code into AST (Abstract Syntax Tree)
- Applies transformations
- Outputs modernized code

**Popular recipes:**
- `Java8toJava11`: Upgrade Java 8 → 11
- `Java11to17`: Upgrade Java 11 → 17
- `Spring5to6`: Upgrade Spring Framework
- `SpringBoot2to3`: Upgrade Spring Boot 2 → 3

**Website:** https://docs.openrewrite.org/

---

### 3. Maven
**What it is:** Build automation tool for Java

**Key commands:**
```bash
mvn clean        # Remove old build files
mvn compile      # Compile source code
mvn test         # Run unit tests
mvn verify       # Compile + test + integration tests
mvn package      # Create JAR/WAR
```

**Configuration:** `pom.xml` (Project Object Model)

---

### 4. peter-evans/create-pull-request
**What it is:** GitHub Action that creates PRs programmatically

**Why needed:** The workflow can't natively create PRs, so we use this action

**What it does:**
- Creates a branch
- Commits changes
- Opens a PR
- Adds labels and assignees

---

## Next Steps After Migration

### ✅ Merge the PR
1. Review all changes
2. Run tests locally (optional but recommended)
3. Approve the PR
4. Click "Merge pull request"
5. Delete the branch

### 📝 Update Documentation
- Update README with Java 11 requirement
- Update CI/CD pipelines if needed
- Document any manual changes made

### 🚀 Deploy
- Build Docker image with Java 11 base
- Deploy to staging environment
- Run smoke tests
- Deploy to production

### 🔄 Repeat for Java 17, 21, 25
Use the same workflow for future Java versions:
```yaml
# For Java 17
java-version: '17'
branch-prefix: 'auto/jdk-17'
title: 'chore: Migrate Java 11 to Java 17'
```

---

## Questions?

- **GitHub Actions Docs:** https://docs.github.com/en/actions
- **OpenRewrite Docs:** https://docs.openrewrite.org/
- **Maven Docs:** https://maven.apache.org/
- **Java 11 Migration Guide:** https://docs.oracle.com/en/java/javase/11/migrate/

Happy migrating! 🚀

