# SonarQube Advanced Security Guide

An existing SonarQube base setup already provides core code quality and security capabilities such as static analysis, taint analysis within first-party code, secrets detection, and infrastructure-as-code scanning.[1] Advanced Security extends that baseline into software supply chain security by adding software composition analysis (SCA), malicious package detection, license policy enforcement, SBOM visibility, and deeper taint analysis across third-party libraries.[1][2]

The practical shift is that analysis no longer stops at the repository boundary. Teams can now evaluate dependency risk, transitive packages, and library interaction paths in the same platform where they already review code quality and security issues.[1][3]

## What Advanced Security Adds

| Capability | Base SonarQube | Advanced Security | Why it matters |
|---|---|---|---|
| First-party static analysis | Yes | Yes | Keeps core code quality and security checks in place.[3] |
| Taint analysis in own code | Yes | Yes | Finds insecure data flow paths in application code.[3] |
| Software Composition Analysis (SCA) | No | Yes | Detects dependency vulnerabilities, malicious packages, and license issues.[1][2] |
| Dependency risk workflow | No | Yes | Adds a dedicated Dependency Risks view with triage and assignment flows.[1] |
| License policy enforcement | No | Yes | Blocks prohibited or risky licenses according to policy.[1][2] |
| SBOM export and dependency inventory | No | Yes | Improves software supply chain visibility and audit readiness.[4][1] |
| Advanced SAST across dependencies | No | Yes | Traces data flow through third-party libraries to uncover hidden vulnerabilities.[3] |
| Dependency-aware quality gates | No | Yes | Fails builds when dependency risks exceed policy thresholds.[1] |

## Advanced SAST

Advanced SAST is the capability that extends Sonar’s taint analysis beyond first-party code and into third-party open-source libraries so the analysis can follow data across code boundaries.[3] This closes the classic blind spot of traditional static analysis, where the tool can inspect the application code but treats external libraries as opaque black boxes.[3]

This matters because many real attack paths are not visible until the full interaction between application code and dependencies is considered. Advanced SAST is designed to uncover deeply hidden vulnerabilities that arise specifically from how the application uses external libraries rather than from the application code alone.[3]

### What teams can do with it

- Detect data flows from untrusted input into sensitive sinks even when the path crosses library boundaries.[3]
- Improve prioritization by surfacing exploitable security findings earlier in pull requests and CI/CD workflows.[3]
- Reduce blind spots that a base-only SAST deployment would leave uncovered.[3]

### Why use it

- It reveals issues that standard repository-only analysis can miss.[3]
- It supports shift-left security by surfacing findings before deployment.[3]
- It keeps the workflow integrated into SonarQube rather than sending developers to a separate security product.[3]

## Software Composition Analysis

Software Composition Analysis identifies and evaluates the open-source components used by a project.[2] In SonarQube Advanced Security, the SCA workflow is centered on the **Dependency Risks** area, where risks can be reviewed across projects, applications, and portfolios.[1]

Each dependency risk is classified as a **Vulnerability**, **Malicious package**, or **Prohibited license**.[1] The interface supports filtering by severity, dependency type, dependency scope, package manager, assignee, and status, which makes it possible to run a repeatable triage process at scale.[1]

### What teams can do with SCA

- Detect known vulnerabilities in direct and transitive dependencies.[1][2]
- Flag malicious packages in the dependency tree.[1]
- Enforce license compliance using license profiles and policies.[1]
- Assign dependency risks to owners, track review state, and document accepted or safe decisions with justification.[1]

### Why use it

- Modern applications rely heavily on third-party packages, so dependency risk is part of the application risk surface.[2]
- SCA gives visibility into legal and compliance exposure, not just security exposure.[2][1]
- It creates an auditable workflow for security and development teams to collaborate on supply chain risk.[1]

## Dependency Risk Model

Advanced Security introduces a structured dependency risk lifecycle. New findings start as **Open**, then can be moved to **Confirmed**, **Accepted**, or **Safe**, with justification required for safe decisions.[1]

Severity is also policy-aware. For vulnerability risks, Sonar combines CVSS severity, CISA KEV status, and EPSS exploitability indicators to prioritize remediation, while malicious packages are always treated as **Blocker** severity.[1]

### Practical meaning of each risk type

| Risk type | Meaning | Typical action |
|---|---|---|
| Vulnerability | A third-party dependency is affected by a publicly reported vulnerability.[1] | Upgrade to a complete or partial fix version after validating impact.[1] |
| Malicious package | The dependency is known to be malicious.[1] | Remove immediately and treat affected machines or builds as potentially compromised.[1] |
| Prohibited license | The dependency license violates the configured license profile or policy.[1] | Replace the package or revise policy if justified.[1] |

## SBOM and Dependency Inventory

Advanced Security improves software supply chain visibility by inventorying dependencies, including transitive dependencies, and supporting standardized export formats for downstream use.[4][5] SonarQube Cloud examples documented by the community show exports in CycloneDX and SPDX 2.3 formats in either JSON or XML, which aligns with common enterprise SBOM workflows.[5]

An SBOM, or Software Bill of Materials, is useful because it gives teams a traceable list of components present in a build or application release.[4][5] That supports audits, incident response, vulnerability impact analysis, and regulatory or customer reporting.[4]

### What teams can do with SBOM-related capabilities

- Maintain a current inventory of dependencies, including transitive packages.[5]
- Export dependency information in standardized formats for audit or sharing workflows.[5]
- Use the inventory to determine whether new CVEs or policy changes affect released software.[4][1]

### Why use it

- It shortens the time needed to answer “where are we using this vulnerable package?”.[4][5]
- It supports security reviews and customer evidence requests.[4]
- It strengthens software supply chain governance without introducing a separate inventory tool.[4][5]

## License Profiles and Policies

One of the most practical additions in Advanced Security is license policy management. Instance administrators can configure license profiles and policies to define which licenses are allowed or prohibited for project dependencies.[1]

This turns open-source license review into an enforceable control instead of a manual spreadsheet exercise. When a dependency violates the selected license policy, it appears as a prohibited license risk in the dependency workflow.[1]

### Why this matters

- It reduces legal and compliance exposure from incompatible open-source licenses.[2][1]
- It provides consistent rules across teams instead of ad hoc judgment calls.[1]
- It enables earlier feedback during development rather than discovering license conflicts late in a release cycle.[2][1]

## Quality Gates for Dependency Risks

After enabling Advanced Security, dependency risks should be incorporated into quality gates; otherwise, the new findings are visible but not automatically enforced.[1] SonarQube supports quality gate conditions for **Prohibited license**, **Malicious package**, and **Vulnerability** risk types, for both new and overall code, and allows thresholds based on count or severity.[1]

This is the control point that turns Advanced Security into an operational safeguard. In practice, it is what allows a pipeline or branch policy to act as a go/no-go gate rather than a passive dashboard.[1]

### Recommended approach

- Create a custom quality gate dedicated to Advanced Security adoption.[1]
- Fail builds on any malicious package finding.[1]
- Set conservative thresholds for new dependency vulnerabilities and prohibited licenses, then tune based on observed signal quality.[1]

## Azure DevOps Pipeline Setup

A common Azure DevOps setup uses the SonarQube extension tasks to prepare analysis, run the build, execute analysis, and publish the quality gate result.[1][6][7] At the platform level, SonarQube also supports Azure DevOps integration through the DevOps Platform Integrations settings, where an Azure DevOps configuration can be created in SonarQube administration.[1]

The basic implementation path is to create a SonarQube service connection in Azure DevOps, install the SonarQube extension, add the analysis tasks to the YAML pipeline, and then publish the quality gate result so the pipeline can reflect policy status.[6][7][8]

### Typical YAML skeleton for Azure DevOps

```yaml
trigger:
  branches:
    include:
      - main
      - develop
      - feature/*

pool:
  vmImage: ubuntu-latest

variables:
  SONARQUBE_ENDPOINT: 'SonarQubeServiceConnection'
  SONAR_PROJECT_KEY: 'your-project-key'

steps:
- task: SonarQubePrepare@5
  displayName: Prepare SonarQube analysis
  inputs:
    SonarQube: '$(SONARQUBE_ENDPOINT)'
    scannerMode: 'CLI'
    configMode: 'manual'
    cliProjectKey: '$(SONAR_PROJECT_KEY)'
    cliProjectName: 'your-project-name'
    cliSources: '.'

- script: |
    echo "Build and test steps go here"
  displayName: Build and test

- task: SonarQubeAnalyze@5
  displayName: Run SonarQube analysis

- task: SonarQubePublish@5
  displayName: Publish quality gate result
  inputs:
    pollingTimeoutSec: '300'
```

This pattern is enough to integrate the project into Azure DevOps and publish the quality gate status to the pipeline summary.[6][7][8] To make Advanced Security effective, the important next step is not just running analysis, but ensuring that dependency risk conditions are part of the quality gate used by the project.[1]

### Recommended Azure DevOps rollout sequence

1. Create the SonarQube service connection in Azure DevOps with the server URL and token.[6][8]
2. Install or verify the SonarQube Azure DevOps extension.[7][8]
3. Add `Prepare`, `Analyze`, and `Publish Quality Gate` tasks to the pipeline.[6][7]
4. Enable Advanced Security features in SonarQube and confirm dependency analysis is active.[2][1]
5. Create a custom quality gate with dependency risk thresholds.[1]
6. Apply that gate to the relevant projects before enforcing branch or release policies.[1]

## How to Work with Each New Capability

### Advanced SAST workflow

1. Run analysis in PRs and CI/CD so findings surface early.[3]
2. Review data flow context and remediation guidance in SonarQube or connected review flows.[3]
3. Fix the insecure flow, then rerun the pipeline to verify the quality gate passes.[3][1]

### SCA workflow

1. Open the **Dependency Risks** tab for the project, application, or portfolio.[1]
2. Filter by risk type, severity, direct versus transitive dependency, and production versus development scope.[1]
3. Review the detailed risk page, including risk explanation and available fixes.[1]
4. Assign, confirm, accept, or mark safe with justification as appropriate.[1]

### SBOM workflow

1. Review the dependency inventory and confirm direct and transitive packages are visible.[5]
2. Export standardized dependency data when audit or downstream sharing is needed.[5]
3. Use the exported inventory during incident response or release evidence preparation.[4][5]

### License policy workflow

1. Define a license profile and policy at instance level.[1]
2. Apply it to projects in scope.[1]
3. Review prohibited license findings inside the dependency risk process and replace offending packages where needed.[1]

## Operational Recommendations

A strong rollout starts with visibility, then moves to controlled enforcement. Teams usually benefit from enabling analysis first, reviewing the first wave of dependency findings, defining ownership and triage rules, and only then turning on strict quality gate enforcement.[1]

The highest-value early controls are usually malicious package blocking, policy-based license checks, and quality gates on new dependency risks. That combination gives a fast security and governance return without requiring a large process redesign.[1][2]

## Summary

For teams that already run base SonarQube, Advanced Security is not a replacement for existing analysis but a meaningful expansion of scope.[3][1] It adds software supply chain visibility, dependency governance, and cross-library data-flow analysis so teams can secure not only the code they write, but also the code they assemble into production systems.[2][4][1]
