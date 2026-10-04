# SonarQube — Important Concepts

### 1. Static Code Analysis

SonarQube reads your code *without running it* and looks for problems. Think of it like a spell-checker, but for code. It catches mistakes early, before they reach users.

### 2. Bugs

A bug is code that is likely to fail or give wrong results when the program runs. Example: using a variable that might be empty (null), which can crash the app. Bugs should be fixed first because they affect how the app works.

### 3. Vulnerabilities

These are security holes that a hacker could use to attack your app. Examples: SQL injection (hacker sends harmful database commands) or passwords written directly in the code. Fixing them protects your data and users.

### 4. Code Smells

Code that works fine today but is messy, confusing, or hard to change later. Examples: very long methods, copy-pasted code, unclear names. It's not broken, but it makes future work slower and more painful.

### 5. Security Hotspots

Code that *might* be risky depending on how it's used (for example, handling passwords or encryption). SonarQube can't be sure it's unsafe, so it asks a developer to review it and mark it as "safe" or "needs fixing".

### 6. Quality Gate

A set of rules your code must pass, like a checkpoint at an airport. Example: "No new bugs, and test coverage must be at least 80%." If the code fails, the build or merge can be stopped. This keeps bad code from going live.

### 7. Quality Profile

A list of rules SonarQube uses to check a specific language (Java, Python, etc.). You can turn rules on or off to match your team's standards. Different languages have different profiles.

### 8. Technical Debt

The estimated time needed to fix all the code smells and maintenance issues, like "2 days 4 hours". Just like financial debt, if you ignore it, it grows and makes the project harder to work on.

### 9. Code Coverage and Duplications

- **Coverage**: how much of your code is tested by automated tests. Higher coverage means fewer hidden bugs.
- **Duplications**: how much code is copy-pasted. Too much duplication means a bug must be fixed in many places.

### 10. New Code (Clean as You Code)

Instead of fixing years of old problems, SonarQube focuses on the code you recently added or changed. If all new code is clean, the overall quality improves step by step without overwhelming the team.

### 11. SonarScanner

The tool that actually reads your code and sends the results to the SonarQube server. The server only shows reports, and the scanner does the checking. Different build tools have their own scanners (Maven, Gradle, .NET, CLI).

### 12. Rules

Each rule is one specific check, such as "don't leave a catch block empty" or "don't hardcode a password." SonarQube has thousands of rules, and each one is tagged as a bug, vulnerability, code smell, or hotspot.

### 13. Issues and Severity

An issue is one problem SonarQube found in your code. Each issue has a severity (Blocker, High, Medium, Low, Info) so you know what to fix first. Blockers are the most urgent.

### 14. Ratings (A to E)

SonarQube gives your project a grade for three areas:

- **Reliability** (bugs)
- **Security** (vulnerabilities)
- **Maintainability** (code smells)

A is the best and E is the worst, just like school grades. It makes quality easy to understand at a glance.

### 15. Cognitive Complexity

A score showing how hard your code is for a human to understand. Too many nested loops and if-else statements raise the score. Lower is better because simple code is easier to read and has fewer bugs.

### 16. Issue Lifecycle (Status)

Every issue moves through states: Open, Confirmed, Resolved, Closed. Developers can also mark an issue as **False Positive** (SonarQube was wrong) or **Accepted** (we know about it and will live with it). This keeps the issue list clean and meaningful.

### 17. Branch and Pull Request Analysis

SonarQube can analyze each branch separately and check every pull request before it is merged. Developers see problems in the PR itself, so bad code is stopped before it reaches the main branch.

### 18. Authentication Tokens

Instead of using a username and password, the scanner uses a secret token to log in to the SonarQube server. It is safer, especially in CI/CD pipelines, and can be revoked anytime.

### 19. SonarLint

A free plugin for IDEs like IntelliJ, VS Code, and Eclipse. It shows SonarQube-style problems *while you type*, like a live spell-checker. Problems are fixed before the code is even committed.

### 20. Webhooks

SonarQube can automatically notify other tools when an analysis finishes. For example, Jenkins can wait for the webhook to learn whether the Quality Gate passed or failed, then decide whether to continue the pipeline.

### 21. sonar-project.properties

A small config file in your project that tells the scanner what to analyze. It holds the project key, project name, source folder, and the server URL. Without it, you would have to type all these settings in the command every time.

### 22. Project Key

A unique ID for each project on the SonarQube server, such as `my-company:payment-app`. The scanner uses it to know which project the results belong to. If two projects share the same key, their results get mixed up.

### 23. Coverage Reports (JaCoCo, Cobertura, etc.)

SonarQube does not run your tests and cannot measure coverage by itself. Your build runs the tests with a tool like JaCoCo, which creates a coverage report file. You then give that file's path to the scanner, which uploads it. If the path is wrong, coverage shows 0%.

### 24. SonarQube Architecture (3 main parts)

- **Web Server**: shows the dashboard and handles your clicks.
- **Compute Engine**: processes the scanner's results in the background.
- **Search Server (Elasticsearch)**: makes searching and filtering issues fast.

The scanner only *sends* data. The server does the heavy processing.

### 25. Database

SonarQube stores all its data in a database, such as issues, history, users, and settings. PostgreSQL is the most common choice. The built-in H2 database is only for testing, never for real use. If the database is lost, all your history is lost, so back it up.

### 26. Plugins and Marketplace

Plugins add features such as support for new languages, integrations, or extra rules. You can install them from the built-in Marketplace or by copying the plugin file into the server folder. After installing, the server needs a restart.

### 27. Users, Groups and Permissions

Control who can do what. Example: developers can view and comment on issues, while admins can change Quality Gates. Permissions are best given to groups instead of individual users. SonarQube also supports LDAP, SAML, and GitHub login, so people can use their company account.

### 28. Portfolios and Applications

A way to group many projects into one view. A manager can see the overall quality of 20 microservices on a single page instead of opening each project. An Application groups projects that together make one product.

### 29. Editions (Community, Developer, Enterprise, Data Center)

SonarQube comes in different versions:

- **Community**: free, with the basics (but no branch or pull request analysis).
- **Developer**: adds branch and PR analysis and more languages.
- **Enterprise**: adds portfolios and reporting.
- **Data Center**: high availability for very large setups.

### 30. Security Reports (OWASP Top 10, CWE)

SonarQube maps its security findings to well-known standards such as the OWASP Top 10 and CWE. This helps with audits and compliance. For example, you can show your team "we have 0 issues in the Injection category."

### 31. Clean as You Code (CaYC)

SonarQube's main philosophy. Don't try to fix all old problems at once. Just make sure every new or changed piece of code is clean. Over time, the old messy code gets replaced or fixed naturally, and the project improves without a huge cleanup effort.

### 32. New Code Period

The definition of what counts as "new code". You can set it as the last 30 days, since a specific version, since a reference branch, or since the previous analysis. The Quality Gate checks only this code, so choose a period that matches your release cycle.

### 33. Quality Gate Conditions

Each Quality Gate is made of individual conditions. Examples: "Coverage on new code ≥ 80%", "Duplications on new code ≤ 3%", "Security rating on new code = A". If any one condition fails, the whole gate fails. The default gate is called "Sonar way".

### 34. Reliability, Security and Maintainability Remediation Effort

Each issue has an estimated time to fix it, such as 5 min or 2 hours. SonarQube adds these up to give a total effort for each area. This helps teams plan: "We need about 3 days to clear all Reliability issues."

### 35. Technical Debt Ratio

The cost of fixing all code smells compared to the cost of writing the code from scratch, shown as a percentage. For example, 2% debt ratio means the code is in good shape. The Maintainability rating (A to E) is based on this ratio.

### 36. Taint Analysis

An advanced security check that follows data from where it enters your app (like a user form) to where it is used (like a database query). If the data is never cleaned along the way, SonarQube flags it as a vulnerability, such as SQL injection. It works across many files and methods, so it finds problems simple checks would miss. (Available in paid editions.)

### 37. Secrets Detection

SonarQube scans for passwords, API keys, tokens, and private keys accidentally written in code. Leaked secrets are a very common cause of real-world breaches, so fixing these quickly is important. If one is found, remove it and rotate (change) the secret, since it may already be exposed.

### 38. Infrastructure as Code (IaC) Analysis

SonarQube also checks configuration files such as Dockerfiles, Terraform, Kubernetes YAML, and CloudFormation. It finds problems like open security groups, containers running as root, or unencrypted storage, so cloud mistakes are caught before deployment.

### 39. Pull Request Decoration

SonarQube posts its results directly on your pull request in GitHub, GitLab, Bitbucket, or Azure DevOps. It shows the Quality Gate status and comments on the exact lines with issues. Developers don't need to open SonarQube to see what to fix.

### 40. Web API

SonarQube has a REST API to automate tasks without using the web page. You can create projects, fetch metrics, check Quality Gate status, or manage users with scripts. Authentication is done with a token. Example: a script can call the API to check whether the gate passed before deploying.

### 41. Maintainability Rating

A grade from A to E showing how easy your code is to maintain. It is based on the Technical Debt Ratio. A means very low debt, E means the code needs heavy cleanup. On new code, the default Quality Gate expects an A.

### 42. Severity vs Impact (Clean Code Taxonomy)

Newer SonarQube versions describe each issue by its *quality impact*: Security, Reliability, or Maintainability, with a level of High, Medium, or Low. This replaces the old Blocker/Critical/Major labels in many views. It tells you what part of your software the issue hurts and how badly.

### 43. Clean Code Attributes

Each rule is linked to a quality of good code, such as Consistent, Intentional, Adaptable, or Responsible. Example: an unused variable breaks "Intentional" because the code's purpose is unclear. It explains *why* an issue matters, not just that it exists.

### 44. Quality Gate Status in CI/CD

Your pipeline can wait for the Quality Gate result and fail the build if it did not pass. In Jenkins, the `waitForQualityGate` step does this. In GitHub Actions, the quality gate action does the same. This stops bad code from being deployed automatically.

### 45. Analysis Parameters (`sonar.sources`, `sonar.exclusions`)

Settings that control what the scanner looks at:

- `sonar.sources`: folders to analyze.
- `sonar.tests`: folders containing tests.
- `sonar.exclusions`: files to skip, like generated code.
- `sonar.coverage.exclusions`: files not counted for coverage.

Excluding generated or third-party code keeps your results accurate.

### 46. Scanner for Maven and Gradle

For Java projects, you don't need a separate install. Run `mvn sonar:sonar` for Maven or `./gradlew sonar` for Gradle, and the build tool runs the analysis. It reads settings from `pom.xml` or `build.gradle`, so setup is simple.

### 47. Compute Engine Tasks (Background Tasks)

After the scanner uploads results, the server places them in a queue and processes them one by one. You can view each task's status (Pending, In Progress, Success, Failed) under Administration. If your dashboard is not updating, check here for errors.

### 48. Housekeeping

SonarQube automatically deletes old data to save database space. For example, it keeps daily snapshots for a while, then weekly, then monthly. It also removes old branches that are no longer used. You can change these settings if you need longer history.

### 49. Backup and Upgrade

Backing up means backing up the **database**, plus the `extensions` and `conf` folders. Before upgrading, always take a backup, check plugin compatibility, and read the release notes. Upgrades run a database migration step, so you cannot simply roll back without a backup.

### 50. Performance and Sizing

SonarQube needs enough memory and disk speed, mainly for Elasticsearch and the database. Use fast SSD storage, give the Compute Engine and search processes enough Java heap, and use an external database like PostgreSQL. Large projects with slow servers lead to long analysis times.
