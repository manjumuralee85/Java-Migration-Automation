# Java Migration Automation Architecture & Implementation Guide

## 📋 Overview

This document explains the **complete Java migration automation system** for spring-petclinic. It covers architecture, components, workflow execution, and how to scale to 50+ applications.

---

## 🏗️ Architecture

### System Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    GitHub Repository                            │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │              Source Code                                 │  │
│  │  ├─ pom.xml (Maven config)                               │  │
│  │  ├─ src/main/java/*.java (Application code)              │  │
│  │  └─ src/test/java/*.java (Tests)                         │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │         Workflow Configuration                           │  │
│  │  ├─ .github/workflows/java-migration.yml                │  │
│  │  ├─ .github/workflows/java-11-to-17-migration.yml       │  │
│  │  └─ rewrite.xml (OpenRewrite recipes)                    │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
         ▲                                          │
         │                                          │
         │ (Trigger)                                │ (Push changes)
         │                                          ▼
    ┌────────────────┐                    ┌─────────────────┐
    │  User Triggers │                    │ GitHub Actions  │
    │  Workflow via  │                    │ Runner (Cloud)  │
    │  GitHub UI or  │                    │                 │
    │  Schedule/API  │                    │ 1. Checkout     │
    └────────────────┘                    │ 2. Setup Java   │
                                          │ 3. OpenRewrite  │
                                          │ 4. Build/Test   │
                                          │ 5. Create PR    │
                                          └─────────────────┘
                                                   │
                                                   ▼
                                          ┌─────────────────┐
                                          │ Pull Request    │
                                          │                 │
                                          │ [DRAFT/READY]   │
                                          │                 │
                                          │ Files changed:  │
                                          │ - pom.xml       │
                                          │ - *.java        │
                                          │                 │
                                          │ [Approve/Merge] │
                                          └─────────────────┘
                                                   │
                                                   ▼
                                          ┌─────────────────┐
                                          │ Main Branch     │
                                          │ Updated!        │
                                          └─────────────────┘
```

---

## 🔄 Detailed Workflow Execution

### Phase 1: Initialization (0-30 seconds)

```
Step 1: Trigger
├─ User clicks "Run workflow" in GitHub UI
└─ Workflow starts on GitHub's cloud runner

Step 2: Checkout Code
├─ GitHub runner downloads your repository
├─ All branches are available
└─ Ready for modifications
```

**Key Variables:**
- `GITHUB_TOKEN`: Allows workflow to commit/push
- `RUNNER_OS`: Ubuntu (Linux container)
- `JAVA_HOME`: Path to Java installation

---

### Phase 2: Environment Setup (30-90 seconds)

```
Step 3: Setup Java 11 (or 17, 21, etc.)
├─ Download Temurin JDK
├─ Install to runner
├─ Set JAVA_HOME environment variable
└─ Verify: java -version

Step 4: Setup Maven Cache
├─ Download Maven dependencies from cache
├─ Speeds up subsequent builds (2-5 minutes faster)
└─ Cache stored in GitHub's cloud
```

**Why Caching Matters:**
- First build: 3-5 minutes (downloads all deps)
- Subsequent builds: 30-60 seconds (uses cache)

---

### Phase 3: Code Modernization (90-150 seconds)

```
Step 5: Update pom.xml
├─ Find: <java.version>8</java.version>
├─ Replace: <java.version>11</java.version>
└─ Effect: Tells Maven to compile for Java 11

Step 6: Run OpenRewrite Recipes
├─ Load rewrite.xml configuration
├─ Parse Java source code into AST
├─ Apply transformation recipes:
│  ├─ Java8toJava11 (API updates)
│  ├─ RemoveUnusedImports (cleanup)
│  └─ AutoFormat (code style)
├─ Write transformed code back to files
└─ Example transformations:
   ├─ for(String s : list) → list.forEach()
   ├─ Optional.empty() → Optional.empty()
   └─ Deprecation removals
```

**Key Concept: AST (Abstract Syntax Tree)**

OpenRewrite understands Java code structure:
```
Code:          for(String s : list) { ... }
                        ▼
AST:           ForStatement
               ├─ variable: String s
               ├─ collection: list
               └─ body: BlockStatement
                        ▼
Transformed:   list.forEach(s -> { ... })
```

---

### Phase 4: Build & Test (150-350 seconds)

```
Step 7: Maven Clean Verify
├─ clean:
│  └─ Delete target/ directory (old artifacts)
│
├─ compile:
│  ├─ Compile src/main/java with Java 11 compiler
│  └─ Compilation errors ❌ or Success ✅
│
├─ test:
│  ├─ Compile src/test/java
│  ├─ Run JUnit tests
│  └─ Report failures
│
├─ integration-test:
│  └─ Run integration tests (if any)
│
└─ verify:
   └─ Run final checks

Output Example:
  [INFO] BUILD SUCCESS
  [INFO] Tests run: 42, Failures: 0, Errors: 0, Skipped: 0
```

**If Build Fails:**
- Logs show exact error line
- Workflow enters "auto-fix" mode
- Creates DRAFT PR for manual review

---

### Phase 5: Git Operations (350-400 seconds)

```
Step 8: Configure Git
├─ git config user.name "github-actions[bot]"
├─ git config user.email "github-actions[bot]@users.noreply.github.com"
└─ Purpose: Sign commits with bot identity

Step 9: Create Branch
├─ Branch name: auto/java-11-upgrade-{timestamp}
├─ Example: auto/java-11-upgrade-1708049832
├─ Purpose: Isolate changes from main branch
└─ Timeline:
   1. Branch created: auto/java-11-upgrade-*
   2. Files modified: pom.xml, *.java
   3. Files staged: git add -A
   4. Commit: git commit -m "chore: migrate Java 8 to 11"
   5. Push: git push origin auto/java-11-upgrade-*

Step 10: Create Pull Request
├─ API call: POST /repos/{owner}/{repo}/pulls
├─ Parameters:
│  ├─ title: "🔄 chore: Migrate Java 8 to Java 11"
│  ├─ body: Detailed PR description
│  ├─ head: auto/java-11-upgrade-*
│  ├─ base: main
│  └─ draft: true/false (based on build status)
└─ Result: PR created and ready for review
```

---

## 🔧 Tool Explanations

### 1. GitHub Actions

**What:** GitHub's CI/CD platform

**When it runs:**
- Manual trigger: Click "Run workflow"
- Schedule: Cron expression (e.g., Monday 2 AM)
- Event: On push/pull request (if configured)

**Pricing:**
- Free: 2,000 minutes/month (private repos)
- Paid: $0.24 per additional minute

**Syntax:**
```yaml
name: Job Name
on:
  workflow_dispatch:  # Manual trigger
  schedule:
    - cron: '0 2 * * 1'  # Monday 2 AM UTC
  
jobs:
  job-name:
    runs-on: ubuntu-latest
    steps:
      - name: Step 1
        run: echo "Hello"
```

---

### 2. OpenRewrite

**What:** Automated code refactoring engine

**How it works:**
1. **Parse:** Convert Java source → AST
2. **Match:** Find code patterns
3. **Transform:** Apply recipes
4. **Write:** Output modernized code

**Popular Recipes:**

| Recipe | Purpose | Example |
|--------|---------|---------|
| `Java8toJava11` | Upgrade Java 8 → 11 | Removes old APIs |
| `Java11to17` | Upgrade Java 11 → 17 | Adds records, sealed classes |
| `UpgradeJava` | General modernization | Uses var, lambda, etc. |
| `Spring5to6` | Spring 5 → 6 upgrade | Updates imports, annotations |
| `SpringBoot2to3` | Spring Boot 2 → 3 upgrade | jakarta.* migration |

**Usage in Workflow:**
```bash
mvn org.openrewrite:rewrite-maven-plugin:run \
  -Drecipes='org.openrewrite.java.migrate.Java8toJava11' \
  -DskipTests
```

**Configuration File (rewrite.xml):**
```xml
<rewrite>
  <recipes>
    <recipe>org.openrewrite.java.migrate.Java8toJava11</recipe>
    <recipe>org.openrewrite.java.cleanup.RemoveUnusedImports</recipe>
  </recipes>
</rewrite>
```

---

### 3. Maven

**What:** Build automation tool for Java

**Key Commands:**
```bash
mvn clean              # Remove old builds
mvn compile            # Compile source
mvn test               # Run tests
mvn verify             # Full build + tests
mvn package            # Create JAR
mvn install            # Install to local repo
```

**Configuration (pom.xml):**
```xml
<properties>
  <java.version>11</java.version>
  <maven.compiler.release>11</maven.compiler.release>
</properties>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <configuration>
        <release>11</release>
      </configuration>
    </plugin>
  </plugins>
</build>
```

---

### 4. Peter Evans Create PR Action

**What:** GitHub Action to create PRs programmatically

**Why needed:** Workflows can't natively create PRs

**Usage:**
```yaml
- uses: peter-evans/create-pull-request@v5
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
    branch: auto/java-11-upgrade
    title: "Migrate Java 8 to 11"
    body: "Automated migration using OpenRewrite"
    labels: "automation,jdk-upgrade"
    draft: false
```

**Output:** New PR created with changes

---

## 📊 Scaling to 50+ Applications

### Strategy: Centralized Workflow Template

Instead of copying workflows to each repo, use a **reusable workflow**:

```yaml
# Central repo: org/java-migration-templates
.github/workflows/
├─ java-migration-template.yml  (shared, reusable)
└─ README.md

# Each app repo uses:
.github/workflows/
└─ call-template.yml (minimal, calls central)
```

**Central Workflow (reuse):**
```yaml
# .github/workflows/java-migration-template.yml
name: java-migration-template
on:
  workflow_call:
    inputs:
      java-version:
        type: string
        required: true
      branch-prefix:
        type: string
        default: auto/jdk

jobs:
  migrate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Java ${{ inputs.java-version }}
        uses: actions/setup-java@v4
        with:
          java-version: ${{ inputs.java-version }}
      
      # ... rest of migration steps ...
```

**Per-App Workflow (minimal):**
```yaml
# Each app's .github/workflows/migrate-java.yml
name: migrate-java
on:
  workflow_dispatch:

jobs:
  call-template:
    uses: org/java-migration-templates/.github/workflows/java-migration-template.yml@main
    with:
      java-version: '17'
      branch-prefix: 'auto/jdk-17'
```

**Benefits:**
- Update workflow once, affects 50+ apps
- Reduce maintenance overhead
- Ensure consistency across all migrations
- Easy to add new migration paths (21, 25)

---

## 📈 Execution Plan for 50+ Apps

### Phase 1: Pilot (Week 1)
- Select 3-5 representative apps
- Test workflow end-to-end
- Identify issues and fix

### Phase 2: Batch 1 (Week 2)
- Deploy to 10 apps
- Monitor for failures
- Fix common issues

### Phase 3: Batch 2 (Week 3)
- Deploy to 20 more apps
- Parallel execution (all at once)
- Monitor PR merge rates

### Phase 4: Batch 3 (Week 4)
- Deploy to remaining apps
- Automate PR merges if confident
- Document lessons learned

### Phase 5: Monitor (Ongoing)
- Watch for compatibility issues
- Track metrics: merge time, test pass rate
- Prepare next Java version (21, 25)

---

## 📊 Monitoring & Metrics

### Key Metrics

```
Per App:
├─ Workflow duration: 3-10 minutes
├─ PR creation time: < 1 minute
├─ Build success rate: 95%+ (target)
├─ Test pass rate: 100% (must fix failures)
└─ Manual review time: 15-30 minutes

Batch Level:
├─ Apps processed: X/50+
├─ Success rate: 95%+
├─ Avg time per app: 5-20 minutes (total)
├─ Bottlenecks: Identify and fix
└─ Cost: $0.24 per minute × workflow duration
```

### Dashboard (GitHub Projects)

```
Migration Status: Java 8 → 11 → 17 → 21 → 25

╔════════════════════════════════════════╗
║ Java 8 → 11 (Current)                 ║
╠════════════════════════════════════════╣
║ ✅ Complete:  15 apps                  ║
║ 🔄 In Progress: 5 apps                 ║
║ ⏳ Queued: 30 apps                     ║
║ ❌ Failed: 2 apps (needs manual fix)  ║
╚════════════════════════════════════════╝
```

---

## 🚨 Failure Handling

### Common Failures & Solutions

| Failure | Cause | Solution |
|---------|-------|----------|
| **Compilation Error** | Old API removed | Manual code update |
| **Test Failure** | Behavior changed | Debug test, fix code |
| **Permission Denied** | No token | Check GitHub Actions permissions |
| **Timeout** | Build too slow | Optimize pom.xml, cache issues |
| **Dependency Conflict** | Version incompatibility | Update pom.xml manually |

### Draft PR Workflow

If build fails:
1. Workflow creates **DRAFT PR** (marked for review)
2. Logs show exact error
3. Developer reviews error logs
4. Developer commits fixes to PR branch
5. Re-run workflow or manual fix
6. Mark PR as "Ready for review"
7. Merge when complete

---

## 🔐 Security Considerations

### Token Permissions

```yaml
permissions:
  contents: write       # Can commit and push
  pull-requests: write  # Can create PRs
```

### Limiting Scope

- Token tied to specific repo
- Workflow can't access other repos
- Use `secrets.GITHUB_TOKEN` (scoped)
- Rotate tokens periodically

### Branch Protection

```yaml
# GitHub repo settings
├─ Require PR reviews: 1 person
├─ Require CI checks: ✅ All pass
├─ Dismiss stale reviews: ✅
├─ Require code review from owners: ✅
└─ Allow auto-merge: ✅ (for trusted changes)
```

---

## 📚 Implementation Checklist

- [ ] Create `.github/workflows/java-migration.yml`
- [ ] Create `rewrite.xml` with recipes
- [ ] Add OpenRewrite plugin to `pom.xml`
- [ ] Create `JAVA_MIGRATION_GUIDE.md`
- [ ] Test workflow on single app
- [ ] Enable GitHub Actions (repo settings)
- [ ] Set up branch protection rules
- [ ] Create monitoring dashboard (GitHub Projects)
- [ ] Document custom recipes (if any)
- [ ] Plan pilot phase (3-5 apps)
- [ ] Plan batch rollout schedule
- [ ] Train team on workflow usage

---

## 🎯 Success Criteria

✅ **Successful Migration:**
- Build passes without errors
- All tests pass
- PR created and merged
- No production incidents

✅ **Scalability:**
- 50+ apps migrated in 4 weeks
- Avg 5-20 minutes per app
- 95%+ success rate on first run
- < 15 minutes manual review per app

---

## 📖 Additional Resources

- [GitHub Actions Docs](https://docs.github.com/en/actions)
- [OpenRewrite Recipes](https://docs.openrewrite.org/recipes)
- [Maven Documentation](https://maven.apache.org/)
- [Java 11 Migration](https://docs.oracle.com/en/java/javase/11/migrate/)
- [Java 17 Migration](https://docs.oracle.com/en/java/javase/17/migrate/)

---

## 🎉 Summary

You now have a **complete, production-ready Java migration automation system** that:

1. ✅ Automates Java version upgrades
2. ✅ Uses OpenRewrite for code modernization
3. ✅ Builds, tests, and validates changes
4. ✅ Creates PRs for human review
5. ✅ Handles failures gracefully
6. ✅ Scales to 50+ applications
7. ✅ Provides monitoring and metrics

**Next Step:** Trigger the workflow on spring-petclinic and watch it work! 🚀

