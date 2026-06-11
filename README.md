# SonarQube Advanced Security â€” Feature Guide

This document covers the new capabilities introduced by SonarQube Advanced Security on top of an existing Enterprise base deployment. Each section explains what the feature does, why it matters, how to use it from the SonarQube dashboard, and what prerequisites apply. Image placeholders mark where screenshots should be inserted.

---

## Prerequisites

Advanced Security is available exclusively as a paid add-on on top of SonarQube Server **Enterprise Edition or Data Center Edition**, starting at version **2025.3**. Any version in the 9.9.x or 10.x branch does not include this functionality regardless of license.

| Requirement | Detail |
|---|---|
| Edition | Enterprise or Data Center |
| Minimum version | 2025.3 |
| Additional license | Yes â€” Advanced Security is a separate add-on |
| Outbound internet from server | Yes â€” HTTPS to `api.sonarcloud.io` (required for SCA) |
| Outbound internet from CI/CD agents | No change needed |
| Advanced SAST internet requirement | None â€” runs locally in the scanner |

---

## What Changes After Enabling Advanced Security

| Capability | Base SonarQube | Advanced Security |
|---|---|---|
| SAST on first-party code | âœ… | âœ… |
| Secrets detection | âœ… | âœ… |
| IaC scanning | âœ… | âœ… |
| Basic taint analysis (within own code) | âœ… | âœ… |
| SCA â€” open source dependency vulnerabilities | âŒ | âœ… |
| Malicious package detection | âŒ | âœ… |
| License management + SBOM | âŒ | âœ… |
| Advanced SAST across third-party libraries | âŒ | âœ… |
| Dependency-aware Quality Gates | âŒ | âœ… |
| Compliance reports including SCA data | âŒ | âœ… (2026.2+) |

---

## Feature 1 â€” Software Composition Analysis (SCA)

### What it does

SCA scans all open-source dependencies in the project â€” both direct and transitive â€” and cross-references them against continuously updated vulnerability databases (CVE/NVD). Results appear in a dedicated **Dependency Risks** panel that is separate from the existing Issues view.

### Why it matters

A large portion of any modern application is composed of third-party packages. Each dependency is a potential attack surface. SCA automates the surveillance of that risk, surfaces prioritized findings directly in developer workflows, and provides actionable remediation guidance for each finding.

### How it works technically

When the scanner runs, it collects the project's dependency manifest files (`pom.xml`, `package.json`, `requirements.txt`, `go.mod`, `*.csproj`, etc.) and lock files. These are packaged and sent via HTTPS to Sonar's cloud service (`api.sonarcloud.io`), which processes the dependency analysis and returns results to the SonarQube server. No source code is transmitted.

### Supported ecosystems

Java (Maven, Gradle), Kotlin, Scala, JavaScript/TypeScript (npm, yarn), Python (pip), C# (NuGet), Go, Rust, Ruby, PHP.

### How to use it from the dashboard

#### Viewing dependency risks

1. Open a **Project** in SonarQube.
2. Click the **Dependency Risks** tab in the top navigation.
3. Use the left sidebar filters to narrow results by:
   - **Risk type**: Vulnerability, Malicious package, Prohibited license
   - **Risk severity**: Blocker, High, Medium, Low, Info
   - **Dependency type**: Direct or Transitive
   - **Dependency scope**: Production or Development
   - **Package manager**, **Status**, **Assignee**

> ðŸ“¸ **[INSERT SCREENSHOT: Dependency Risks tab â€” project overview with filter sidebar visible]**

#### Reading a risk detail card

Each risk card in the list shows:
- Descriptive title of the risk
- Risk type and severity badge
- Current status (Open, Confirmed, Accepted, Safe)
- Assignee
- Affected dependency name and version
- Time since first detected

> ðŸ“¸ **[INSERT SCREENSHOT: Individual risk card in the Dependency Risks list]**

#### Opening the detailed view

Click the title of any risk to open its full detail page. The detail view contains:

- **What's the risk?** â€” CVSS severity score, KEV (known exploited) status, EPSS exploitability probability
- **How can I fix it?** â€” Available fix versions classified as Complete fix, Partial fix, or Affected version
- **Maintainer insights** â€” guidance from Sonar-partnered maintainers on real-world impact and workarounds

> ðŸ“¸ **[INSERT SCREENSHOT: Dependency risk detail page â€” "What's the risk?" tab open]**

> ðŸ“¸ **[INSERT SCREENSHOT: Dependency risk detail page â€” "How can I fix it?" tab open showing fix versions]**

#### Managing risk status and assignment

1. From the risk list or detail page, click **Change Status**.
2. Select the new status from the dropdown: **Confirmed**, **Accepted**, or **Safe**.
3. For **Safe**, a mandatory justification must be provided in the text box.
4. To assign the risk to a team member, click the **Unassigned** dropdown and type a name.

> ðŸ“¸ **[INSERT SCREENSHOT: Change Status modal with dropdown and justification field]**

### Risk severity model

| Severity | Criteria |
|---|---|
| Blocker | Vulnerability is on the CISA KEV list (actively exploited in the wild) |
| High | EPSS > 5% AND CVSS > 7.0 |
| Medium | EPSS > 0.5% (or no EPSS) AND CVSS > 4.0 |
| Low | Any remaining vulnerability not fitting higher categories |
| Info | Declared false positive by maintainer, or withdrawn by NIST/OSV |

### Prerequisites

- SonarQube Server Enterprise 2025.3+
- Advanced Security add-on license
- Outbound HTTPS from the SonarQube server to `api.sonarcloud.io`

---

## Feature 2 â€” Malicious Package Detection

### What it does

Malicious package detection identifies dependencies that are known to contain active malware, not just known vulnerabilities. While SCA finds packages with security flaws, this feature flags packages that are themselves the attack vector.

### Why it matters

Supply chain attacks via malicious packages have grown significantly. A compromised package can exfiltrate credentials, execute arbitrary code, or compromise the build environment itself. These risks cannot be caught by CVE-based scanning alone because the package may not have any CVE record â€” it is simply malicious.

### How it works from the dashboard

Malicious package findings always carry **Blocker** severity. They appear in the Dependency Risks panel under the **Malicious package** risk type filter.

When a malicious package is detected, the **How can I fix it?** tab in the detail view provides specific response steps, including notifying the information security team, since any machine that installed the package should be considered compromised.

> ðŸ“¸ **[INSERT SCREENSHOT: Dependency Risks list filtered by "Malicious package" risk type showing a Blocker severity finding]**

> ðŸ“¸ **[INSERT SCREENSHOT: Malicious package detail page â€” "How can I fix it?" tab with remediation steps]**

### Prerequisites

Included within SCA â€” same requirements apply. No additional configuration needed.

---

## Feature 3 â€” License Management and Policy Enforcement

### What it does

License management scans the licenses of all project dependencies and validates them against configurable organizational policies. It automatically flags dependencies whose licenses are prohibited or incompatible before code reaches production.

### Why it matters

Open-source license risk is as real as security risk. Using a dependency with an incompatible license (such as a copyleft license like GPL in a proprietary product) can create legal exposure. Doing this check manually across dozens or hundreds of dependencies is not viable at development speed.

### How to configure license policies

License profiles and policies are configured by an instance administrator:

1. Go to **Administration > Configuration > Advanced Security > License Profiles**.
2. Select a predefined profile (e.g., *Permissive open-source licenses*) or create a custom one.
3. Define which licenses are **allowed** and which are **prohibited**.
4. Apply the profile to the relevant projects or the entire instance.

> ðŸ“¸ **[INSERT SCREENSHOT: Administration > License Profiles page showing predefined profiles list]**

> ðŸ“¸ **[INSERT SCREENSHOT: License profile configuration â€” allowed/prohibited license list]**

### Viewing license violations from the dashboard

1. Open the **Dependency Risks** tab on any project.
2. Filter by **Risk type: Prohibited license**.
3. Each violation shows the dependency name, the detected license, and a link to resources about that license type.
4. Resolution typically requires replacing the package with a compatible alternative.

> ðŸ“¸ **[INSERT SCREENSHOT: Dependency Risks list filtered by "Prohibited license" showing affected packages]**

### Prerequisites

- SonarQube Server Enterprise 2025.3+
- Instance administrator access to configure license profiles
- Same SCA connectivity requirements apply

---

## Feature 4 â€” SBOM (Software Bill of Materials)

### What it does

The SBOM feature generates and maintains a complete, traceable inventory of all software components in the project, including transitive dependencies. It is automatically populated as part of every SCA scan â€” no additional configuration is required.

### Why it matters

An SBOM is increasingly required by enterprise customers, regulators, and security frameworks. It answers the critical incident-response question: *"Where are we using this vulnerable library?"* â€” across all projects, instantly. It also supports audit processes and supply chain transparency requirements.

### How to access and export the SBOM

1. Open the project and navigate to **Inventory > Dependencies**.
2. The full dependency tree is listed, including direct and transitive packages with version and license information.
3. To export, click the **Export** button and select the desired format:
   - **CycloneDX** (JSON or XML)
   - **SPDX 2.3** (JSON or XML)

> ðŸ“¸ **[INSERT SCREENSHOT: Inventory > Dependencies tab showing the full dependency tree]**

> ðŸ“¸ **[INSERT SCREENSHOT: Export modal with format options â€” CycloneDX and SPDX]**

### Downloading a full dependency risk report

A risk-focused report (not just the SBOM) can also be downloaded for a project, application, or portfolio:

1. From the **Project overview** page, click **Download dependency risk report**.
2. The report is available in **JSON** and **CSV** format.
3. A **VEX (Vulnerability Exploitability eXchange)** report is also available, which includes raw vulnerability detail and justifications for risks marked as Safe.

> ðŸ“¸ **[INSERT SCREENSHOT: Project overview page showing the "Download dependency risk report" option]**

### Prerequisites

- Included with SCA â€” no additional setup beyond enabling Advanced Security
- SonarQube Server Enterprise 2025.3+

---

## Feature 5 â€” Advanced SAST

### What it does

Advanced SAST extends Sonar's existing taint analysis beyond first-party code and into third-party open-source libraries. It traces data flows across code boundaries â€” from user input through application code and into library functions â€” to uncover complex vulnerabilities that traditional SAST tools cannot detect because they treat external libraries as black boxes.

### Why it matters

Many real attack paths only become visible when the full interaction between application code and its dependencies is analyzed. For example, a SQL injection vulnerability may originate in application code, pass through a framework method, and reach a database sink inside a library. Without cross-library analysis, this path is invisible.

### How it differs from base SAST

| | Base SAST | Advanced SAST |
|---|---|---|
| Scope | First-party code only | First-party + third-party libraries |
| Taint analysis boundary | Stops at library call | Crosses into and out of libraries |
| Configuration required | No | No |
| Performance impact | None | None |
| Languages supported | 35+ | Java, C# (expanding per version) |
| Internet required | No | No |

### How findings appear in the dashboard

Advanced SAST findings appear in the standard **Issues** view alongside regular security vulnerabilities. There is no separate panel â€” they are fully integrated into the existing workflow.

To identify Advanced SAST findings:
1. Open the **Issues** tab on a project.
2. Filter by **Type: Vulnerability** and **Security category**.
3. Look for issues whose data flow trace spans into library code â€” this is visible in the issue detail's code flow visualization.

> ðŸ“¸ **[INSERT SCREENSHOT: Issues list filtered by Vulnerability type, showing a finding from Advanced SAST]**

> ðŸ“¸ **[INSERT SCREENSHOT: Issue detail page with code flow visualization showing the data path crossing into a third-party library]**

### Prerequisites

- SonarQube Server Enterprise 2025.3+
- Advanced Security add-on license
- No connectivity requirements â€” analysis runs locally in the scanner
- Current language support: **Java** and **C#** (Python top-1000 libraries added in 2026.2)

---

## Feature 6 â€” Dependency-Aware Quality Gates

### What it does

Quality Gates can now include conditions based on dependency risks â€” vulnerability count, malicious package detection, and prohibited license violations â€” in addition to the existing code quality and SAST conditions. This turns dependency findings into enforceable pipeline controls.

### Why it matters

Without Quality Gate integration, dependency findings are visible in the dashboard but do not block deployments. Adding dependency conditions creates an automatic go/no-go gate: a build with a malicious package or a critical vulnerability can be blocked before it reaches any environment.

### How to configure a dependency-aware Quality Gate

> âš ï¸ After enabling Advanced Security, a new custom Quality Gate must be created manually. The default gate does not include dependency risk conditions.

1. Go to **Administration > Quality Gates**.
2. Click **Create** to define a new gate, or copy an existing one.
3. Click **Add Condition** and select from the dependency risk metrics:
   - **Dependency vulnerabilities** â€” set threshold by count and/or severity
   - **Malicious packages** â€” recommended: fail on any finding (threshold = 0)
   - **Prohibited licenses** â€” recommended: fail on any finding (threshold = 0)
4. Choose whether the condition applies to **New Code** or **Overall Code**.
5. Assign the Quality Gate to the relevant projects.

> ðŸ“¸ **[INSERT SCREENSHOT: Quality Gate administration page â€” Add Condition dropdown showing dependency risk options]**

> ðŸ“¸ **[INSERT SCREENSHOT: Quality Gate conditions list showing configured dependency risk thresholds]**

### Recommended initial configuration

| Condition | Threshold | Applies to |
|---|---|---|
| Malicious packages | 0 (any finding fails) | Overall code |
| Prohibited licenses | 0 (any finding fails) | Overall code |
| New dependency vulnerabilities â€” Blocker | 0 | New code |
| New dependency vulnerabilities â€” High | 0â€“5 (tune based on team capacity) | New code |

---

## Feature 7 â€” Continual Dependency Re-Analysis

### What it does

Once a branch has been analyzed with SCA at least once, SonarQube Advanced Security re-analyzes its dependencies automatically on a scheduled basis â€” without requiring a new commit or pipeline run. New CVEs published after the last scan are surfaced as new findings automatically.

### Why it matters

Vulnerability databases are updated continuously. A dependency that was safe at the time of the last commit can become vulnerable the next day when a new CVE is published. Without continual re-analysis, teams would only discover this risk the next time a developer commits code.

### How to configure re-analysis schedules

1. Go to **Administration > Configuration > Advanced Security > Configure Branch Rescanning**.
2. Set the **frequency**: daily, weekly, or disabled.
3. Select which branches are re-scanned: main branch only, all permanent branches, or all branches.

> ðŸ“¸ **[INSERT SCREENSHOT: Administration > Advanced Security > Configure Branch Rescanning settings page]**

---

## Connectivity and Infrastructure Summary

| Component | Needs internet? | Why |
|---|---|---|
| SonarQube Server | Yes â€” HTTPS to `api.sonarcloud.io` | SCA dependency processing happens in Sonar's cloud |
| CI/CD Agents | No change | Scanner sends manifests to the server, not to Sonar cloud directly |
| Advanced SAST | No | Analysis runs locally in the scanner process |
| Air-gapped environments | SCA not supported | No offline mode available for dependency analysis |

---

## Version Compatibility Reference

| Version | Advanced Security available | Notes |
|---|---|---|
| 9.9.x, 10.x | âŒ No | Not supported regardless of license |
| 2025.1, 2025.2 | âŒ No | Predates GA release |
| **2025.3** | âœ… First GA version | SCA, Advanced SAST, SBOM, License management |
| **2025.4 (LTA)** | âœ… Yes | Supported until January 2027 |
| **2026.1 (LTA current)** | âœ… Yes | Recommended â€” supported until August 2027 |
| **2026.2** | âœ… Yes | Adds Python top-1000 Advanced SAST, SCA in compliance reports |