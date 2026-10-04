### 1. Different teams need different Quality Gate rules

**Situation:** A payments team wants very strict rules (90% coverage). A small internal tools team says that is too much for them.

**Why it happens:** One gate for all projects does not fit every risk level.

**Steps to fix:**

1. Go to **Quality Gates → Create** and copy the default "Sonar way" gate. Built-in gates can't be edited, so you copy them.
2. Change the conditions. Example for the strict gate: coverage on new code ≥ 90%, security rating = A.
3. Open each project → **Project Settings → Quality Gate** and choose which gate it uses.

**Simple picture:** Like different exam pass marks for different courses.

**Tip:** Keep conditions on **new code**, so old code doesn't block the team.

---

### 2. SonarQube reports too many false positives and developers ignore it

**Situation:** A rule keeps flagging safe code. Developers lose trust and stop reading the reports.

**Why it happens:** Some rules don't fit your project or framework.

**Steps to fix:**

1. **Single issue:** mark it **False Positive** with a comment.
2. **Whole rule is noisy:** copy the Quality Profile (built-in ones are read-only), then **deactivate** that rule or lower its severity.
3. Assign the new profile to the project.
4. As a last resort for one line of code, use `// NOSONAR`. Use it rarely, because it hides issues.

**Prevention:** Review the rule list with the team once. A smaller, trusted rule set is better than a huge ignored one.

---

### 3. Analyzing a monorepo or multi-module project

**Situation:** One repository has many modules (e.g., `user-service`, `order-service`, `common`). You want clear results for each.

**Why it happens:** By default, the scanner sees the whole repo as one project and mixes everything together.

**Steps to fix (two choices):**

- **Option A: one project with modules.** Set `sonar.modules=user,order,common` and per-module settings. Simple, but all results are in one place.
- **Option B: separate projects per service** (preferred for microservices). Run a scan in each folder with its own project key, such as `company:user-service`. Then group them in a **Portfolio** (Enterprise) or use **monorepo support** in Developer Edition and higher.
- For Maven or Gradle, the build tool detects modules automatically.

**Prevention:** Decide the structure early. Changing project keys later breaks history.

---

### 4. The scan takes too long (e.g., 45 minutes)

**Situation:** The Sonar step makes your pipeline very slow.

**Why it happens:** Too many files are scanned, or the machine and server are underpowered.

**Steps to fix:**

1. **Exclude** generated code, `node_modules`, build folders, and large third-party files (`sonar.exclusions`).
2. Use **pull request analysis**, which scans only the changed files.
3. Give the **scanner** more memory (`SONAR_SCANNER_OPTS=-Xmx2048m`).
4. Check the **server**: Compute Engine memory, an SSD disk, and an external database, not H2.
5. Look at the scan log. It shows the time taken by each step, so you can find the slow one.

**Prevention:** Scan only what you own, and monitor background task times.

---

### 5. SonarQube won't start (Elasticsearch error)

**Situation:** The server starts, then stops. The log shows an Elasticsearch error such as `max virtual memory areas vm.max_map_count [65530] is too low`.

**Why it happens:** SonarQube uses Elasticsearch, which needs more memory-mapping space than Linux gives by default.

**Steps to fix:**

1. Increase the limit:

        `sysctl -w vm.max_map_count=524288`

   To keep it after reboot, add it to `/etc/sysctl.conf`.
2. Also raise the file limit (`fs.file-max`, and `nofile` of at least 131072 for the user running SonarQube).
3. Don't run SonarQube as **root**. Elasticsearch refuses to start as root.
4. Check `es.log` and `sonar.log` in the `logs` folder for the exact message.

**Docker tip:** The setting must be done on the **host machine**, not inside the container.

---

### 6. Security Hotspots are piling up. What do you do with them?

**Situation:** The dashboard shows 40 "Security Hotspots to review" and the team doesn't know what to do.

**Why it happens:** A hotspot is not a confirmed bug. It is code that **may** be unsafe depending on how it is used, such as using a random number generator or a regex on user input.

**Steps to fix:**

1. Open each hotspot. It explains the risk and gives a "how to fix" example.
2. A developer decides:
   - **Safe**: the use is fine (add a comment why).
   - **Fixed**: code was changed.
   - **Acknowledged**: it is risky, and a fix is planned.
3. Reviewed hotspots no longer count against the review rate.
4. The default Quality Gate expects **100% of new hotspots reviewed**.

**Difference to remember:** *Vulnerability* = definitely a problem. *Hotspot* = a human must decide.

---

### 7. Branch and pull request analysis is missing in Community Edition

**Situation:** You try to scan a feature branch or a pull request, but the options are missing, or all branches overwrite the main results.

**Why it happens:** Community Edition analyzes only **one branch** per project (usually main).

**Steps to fix (choose one):**

1. **Upgrade** to Developer Edition or higher, which gives branch and PR analysis.
2. **Workaround in Community:** scan only the main branch after merging. For feature branches, use **SonarLint** in the IDE, or create a separate project per branch (messy, not recommended).
3. Alternatively, use **SonarQube Cloud**, which supports PR analysis.

**Interview tip:** Mention this edition limitation. Interviewers like it.

---

### 8. Integrating SonarQube with GitHub Actions

**Situation:** You want every pull request in GitHub scanned automatically.

**Steps to fix:**

1. In GitHub → **Settings → Secrets**, add `SONAR_TOKEN` and `SONAR_HOST_URL`.
2. In your workflow file, check out the code with `fetch-depth: 0`. Sonar needs full Git history for accurate blame and new code detection.
3. Add the **SonarQube scan action** step, using the secrets.
4. Add the **quality gate check action** after it, so the job fails if the gate fails.
5. Mark the check as **required** in branch protection rules.

**Common mistake:** A shallow checkout (default depth 1) causes "missing blame information" warnings and wrong new-code results.

---

### 9. Users must log in with their company account (SSO)

**Situation:** Managers don't want separate SonarQube passwords for each person.

**Why it matters:** Separate accounts are hard to manage, and people who leave the company keep access.

**Steps to fix:**

1. In **Administration → Configuration → Authentication**, choose the method: **LDAP/Active Directory**, **SAML**, **GitHub**, **GitLab**, or **Azure AD**.
2. Fill in the details (server URL, client ID/secret, etc.).
3. Turn on **group sync**, so company groups map to SonarQube groups, and permissions follow automatically.
4. Keep one **local admin account** as a backup, in case SSO breaks.

**Best practice:** Put SonarQube behind HTTPS (a reverse proxy like Nginx) before enabling SSO.

---

### 10. Management wants one report of quality across 30 projects

**Situation:** A manager asks, "How healthy is our code overall? Which projects are the worst?"

**Why it happens:** Opening 30 dashboards one by one is not practical.

**Steps to fix:**

1. Create a **Portfolio** (Enterprise Edition) and add the projects, by tags or manually.
2. The portfolio shows combined **Reliability, Security, and Maintainability ratings**, coverage, and technical debt.
3. Use **Applications** to group the projects that form one product.
4. For custom reports, use the **Web API** (e.g., `api/measures/component`) in a script to export metrics to Excel or a dashboard.
5. Use **Tags** to organize projects (`payments`, `legacy`) and filter the project list.

**Without Enterprise:** Use the Web API plus a spreadsheet or Grafana.
