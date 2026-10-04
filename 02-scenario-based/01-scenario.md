### 1. The Quality Gate failed, but the developer says the code is fine

**Situation:** A developer pushes code. The build turns red because the Quality Gate failed. The developer says, "My code works, SonarQube is wrong."

**Why it happens:** The Quality Gate is not about whether code *works*. It checks quality rules, such as "no new bugs" or "coverage on new code must be 80% or more". Code can work perfectly and still fail these rules.

**Steps to fix:**

1. Open the project dashboard and click the failed Quality Gate. It shows exactly which condition failed. Example: *Coverage on new code: 65% (required 80%)*.
2. Decide if it is a real problem or a false alarm.
   - **Real problem** (low coverage, a new bug): the developer fixes it, for example by adding tests.
   - **False alarm** (SonarQube flagged safe code): mark the issue as **False Positive**, or **Accepted** if the team agrees to live with it. Always add a comment explaining why.
3. Re-run the analysis. The gate should now pass.

**What not to do:** Don't lower the Quality Gate rules just to make the build green. That defeats its purpose.

**Prevention:** Install SonarLint in the IDE so developers see issues before they push.

---

### 2. Coverage shows 0% even though you have unit tests

**Situation:** The team has many unit tests, but SonarQube shows 0% coverage.

**Why it happens:** SonarQube **does not run your tests**. It only *reads a coverage report* made by another tool (like JaCoCo for Java). If it can't find that report, it assumes nothing was tested and shows 0%.

**Steps to fix (check in this order):**

1. **Do tests run before the scan?** In your pipeline, the order must be: build → run tests → generate report → run Sonar scan. If the scan runs first, there is no report yet.
2. **Is the report generated?** Check that the file exists (for example `target/site/jacoco/jacoco.xml`).
3. **Is the path correct?** Tell the scanner where the file is:

        `sonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml`

4. **Check the scan logs.** Look for a line saying the coverage report was imported, or a warning that it was not found.

**Prevention:** Keep the coverage path in `sonar-project.properties` so it is the same every time.

---

### 3. Stop a Jenkins pipeline from deploying code that fails the Quality Gate

**Situation:** The scan runs, the gate fails, but Jenkins still deploys to production.

**Why it happens:** By default, Jenkins only *starts* the scan and moves on. It does not wait for the result. The result is calculated later by the SonarQube server.

**Steps to fix:**

1. **Run the scan inside `withSonarQubeEnv`.** This connects Jenkins to your SonarQube server.
2. **Create a webhook in SonarQube** (Administration → Configuration → Webhooks) pointing to your Jenkins URL, such as `http://jenkins/sonarqube-webhook/`. When the analysis finishes, SonarQube calls Jenkins and tells it the result.
3. **Add the `waitForQualityGate` step** after the scan. Jenkins pauses until it receives the result. If the gate fails, the step fails the build, and the deploy stage never runs.

**Simple picture:** Jenkins sends the code to the examiner (SonarQube) and waits at the door. The webhook is the examiner calling back with "Pass" or "Fail".

**Common mistake:** If the webhook is missing, `waitForQualityGate` waits forever and times out.

---

### 4. A legacy project has 5,000 issues and the team is overwhelmed

**Situation:** You connect an old project to SonarQube. It shows thousands of issues. The team says, "We can't fix all this."

**Why it happens:** Old code was never checked, so problems built up over the years.

**Steps to fix: use "Clean as You Code":**

1. **Don't fix the old issues first.** That is slow and risky.
2. **Set the New Code Period.** For example, "since the last release" or "last 30 days".
3. **Apply the Quality Gate to new code only.** Every new or changed line must be clean.
4. When developers touch an old file for other work, they fix the issues nearby.
5. Over time, old code is replaced or cleaned and the numbers drop.

**Simple picture:** Imagine a messy house. Instead of cleaning everything in one weekend, you promise that every new thing you bring in stays tidy. Slowly, the house gets clean.

---

### 5. SonarQube found a hardcoded password in the code

**Situation:** The scan reports a password or API key written directly in the source code.

**Why it matters:** Anyone with access to the code (or the Git history) can see it. This is one of the most common causes of real security breaches.

**Steps to fix:**

1. **Remove it from the code.** Read it from an environment variable or a secrets manager (like HashiCorp Vault or AWS Secrets Manager) instead.
2. **Rotate the secret.** Change the password or generate a new key. This is the step people forget. Even after deleting it from the code, the old value is still in **Git history** and must be treated as leaked.
3. **Re-run the scan** to confirm the issue is gone.

**Prevention:** Use SonarLint in the IDE and secrets detection in the pipeline to catch it before it is committed.

---

### 6. Exclude generated code or test files from analysis

**Situation:** Reports are full of issues from auto-generated code or third-party libraries that nobody on your team wrote.

**Why it matters:** Those files add noise and make the quality numbers wrong.

**Steps to fix:** Add exclusions in `sonar-project.properties`. There are three types, and each does something different:

| Property | What it does | Example |
| --- | --- | --- |
| `sonar.exclusions` | Skips the files completely (no issues, no metrics) | `**/generated/**` |
| `sonar.coverage.exclusions` | Files are analyzed but not counted in coverage | `**/dto/**` |
| `sonar.cpd.exclusions` | Files are not checked for duplication | `**/models/**` |

**Tip:** `**` means "any folder at any depth", so `**/generated/**` matches every folder named *generated*.

---

### 7. Developers want to see issues before merging, not after

**Situation:** Issues are found only after code reaches the main branch, which is too late.

**Steps to fix: use three layers:**

1. **Pull Request analysis.** SonarQube scans each pull request separately before merging. (Needs **Developer Edition** or higher.)
2. **PR Decoration.** Connect SonarQube to GitHub, GitLab, Bitbucket, or Azure DevOps. It then posts the Quality Gate result and comments on the exact lines with problems, right inside the pull request.
3. **Branch protection rules.** In GitHub or GitLab, make the SonarQube check *required*, so a failed gate blocks the merge button.
4. **SonarLint in the IDE.** Developers see issues while typing, even before committing.

**Result:** Problems are caught at the earliest and cheapest point.

---

### 8. The scan runs, but the SonarQube dashboard does not update

**Situation:** The pipeline says the scan succeeded, but the dashboard still shows old data.

**Why it happens:** The scanner only *uploads* the results. The server must then *process* them in the background (the **Compute Engine**). If that processing is stuck or fails, the dashboard doesn't change.

**Steps to troubleshoot:**

1. Go to **Administration → Projects → Background Tasks**. Check the status of your task:
   - **Pending**: waiting in the queue (the server may be busy or stuck).
   - **Failed**: click it to see the error message.
2. Check the **Compute Engine log** (`ce.log`) on the server for the detailed error.
3. Check the basics:
   - **Disk space** (a full disk is a very common cause)
   - **Memory** (Java heap too small)
   - **Database connection** (is the DB up and reachable?)
4. Fix the cause, then re-run the scan.

---

### 9. The CI/CD scan fails with an authentication error

**Situation:** The pipeline shows "Not authorized" or "401 Unauthorized" during the scan.

**Why it happens:** The scanner could not prove who it is, or it is not allowed to do the job.

**Check these three things:**

1. **Token problem**: it is wrong, expired, or revoked. Generate a new one under *My Account → Security*.
2. **Permission problem**: the token's user needs the **Execute Analysis** permission on the project (and *Create Projects* if it is a new project).
3. **URL problem**: `sonar.host.url` is wrong, or points to HTTP when the server uses HTTPS.

**Best practice:** Store the token as a **secret** in your CI tool (Jenkins credentials, GitHub Secrets). Never write it in code or in `sonar-project.properties`.

---

### 10. Upgrade SonarQube to a newer version safely

**Situation:** You must move to a new SonarQube version without losing data or breaking the setup.

**Why care:** An upgrade changes the database structure. Once migrated, you **cannot easily go back** without a backup.

**Steps:**

1. **Prepare**: read the release notes, and check that your plugins and database version are supported.
2. **Back up**: the database (most important), plus the `extensions` and `conf` folders.
3. **Test first**: do the upgrade on a staging copy of the server.
4. **Upgrade**: stop SonarQube, install the new version, copy your settings and plugins, and start it.
5. **Migrate**: open `http://your-server/setup` and run the database migration.
6. **Verify**: log in, run a test scan, and confirm the Quality Gate and dashboards work.

**Rollback plan:** If something fails, restore the database backup and start the old version.
