# SentinelX — Cross-Platform Security Hardening

<p align="center">
  <strong>Audit. Harden. Verify. Report.</strong><br/>
  A policy-driven security-hardening application for supported Windows and Linux environments.
</p>

<p align="center">
  <img alt="Go" src="https://img.shields.io/badge/Go-Backend-00ADD8?logo=go&logoColor=white" />
  <img alt="Platforms" src="https://img.shields.io/badge/Platforms-Windows%20%7C%20Ubuntu%20%7C%20CentOS-blue" />
  <img alt="Project" src="https://img.shields.io/badge/Infothon-SIH25237-purple" />
</p>

---

## Overview

**SentinelX** is a system security-hardening project that helps administrators assess selected operating-system settings against a defined set of security policies. It provides a local dashboard and application API for running audits, reviewing findings, requesting supported corrective actions, attempting rollback operations, and exporting PDF reports.

Security configuration can be difficult to manage consistently. Administrators may make changes over time and then struggle to understand which controls were checked, which findings require attention, or what actions were attempted. SentinelX brings policy checks and hardening actions into a more structured workflow, with results and report generation to support review.

> **Project status:** This repository contains a working implementation foundation and platform-specific components. Feature availability and behaviour can differ by operating system and policy. Run and validate the software in a test environment before using it on important systems.

## Why SentinelX?

- **Policy-based audits** — compare selected system settings with expected values defined in policy files.
- **Severity-based profiles** — choose Basic, Moderate, or Strict scan profiles.
- **Remediation actions** — request supported corrective actions for individual rules.
- **Rollback-related operations** — attempt to reverse changes using the rollback actions defined by the policies.
- **Audit reports** — generate PDF reports summarizing check results and control status.
- **Local dashboard and API** — interact with the application through its web interface and local API.
- **Platform-specific components** — repository content includes Windows, Ubuntu, and CentOS-related executables or source components.

## How It Works

```text
        Administrator
              |
              v
       SentinelX Dashboard
              |
              v
       Local Application API
              |
              v
        Policy / Rule Set
              |
              v
     Security Audit Engine
              |
      +-------+--------+
      |                |
      v                v
   Findings       Requested Fix
      |                |
      +-------+--------+
              |
              v
     Results / PDF Report
              |
              v
    Local Action History
```

1. **Load policies:** SentinelX reads rules describing security controls, expected values, and supported actions.
2. **Run checks:** the audit engine inspects relevant settings on the machine where the application is running.
3. **Review findings:** results indicate whether each check passed, failed, or timed out.
4. **Request a corrective action:** an administrator can request a fix for a supported rule.
5. **Review and report:** the application can generate a PDF report and stores action-related history in a local SQLite database.

A failed audit check does **not** necessarily mean a system has been compromised. It means that the specific check did not pass.

## Current Capabilities

| Capability | Description |
|---|---|
| Security audit | Runs policy-defined checks and returns per-control results. |
| Profiles | Basic includes Critical and High rules; Moderate includes Critical, High, and Medium rules; Strict includes all rules in the loaded policy. |
| Remediation | Supports rule-defined actions such as commands and configuration edits, depending on the platform and rule. |
| Rollback | Provides endpoints and policy actions intended to reverse selected changes. |
| Reset | Provides an operation that attempts to reverse fixes defined by the loaded policy. |
| PDF reporting | Generates a compliance-style report with a summary and control-level results. |
| Local API | Exposes endpoints for status, scanning, fixing, rollback, exporting, and reset operations. |

The exact set of supported controls depends on the policy file and platform implementation. Please verify a result by running a scan after a change; an action being requested or logged does not alone prove that the system reached the desired state.

## Technology

- **Go** for the application, API, and hardening engine
- **Gin** for HTTP routing
- **JSON** for policy definitions
- **SQLite** for local action-history data
- **HTML, CSS, and JavaScript/Tailwind CSS** for the web interface
- **PDF generation** for audit reports

## Repository Structure

The repository includes platform-specific application folders, security policies, prebuilt executables, a dashboard, and example reports. Key paths include:

```text
.
├── README.md
├── SIH_PS25237_SentinelX.pptx.pdf
├── centos/
│   ├── cmd/app/main.go
│   ├── internal/
│   │   ├── engine/
│   │   ├── policy/
│   │   ├── platform/
│   │   ├── report/
│   │   ├── state/
│   │   └── dag/
│   ├── policies/
│   └── ui/
├── ubuntu/
│   └── SIH25/
│       ├── cmd/app/main.go
│       ├── internal/
│       ├── policies/
│       └── ui/
├── reports/
├── hardening-tool_centos
├── hardening-tool_ubuntu
└── sentinelx-ui_windows.exe
```

Some binaries and platform files may be distributed separately or have platform-specific requirements. Refer to the files and instructions shipped in the repository before running a build or executable.

## Running the Existing Demo

### Option 1: Use the provided binaries

The repository includes a demo instruction file with these commands. Run the binary matching your environment.

**Windows UI**
1. Download or clone the repository.
2. Open a terminal in the repository directory.
3. Run the UI executable with the privileges required by your test setup:
   ```powershell
   .\sentinelx-ui_windows.exe
   ```

**CentOS**
```bash
sudo ./hardening-tool_centos
``

**Ubuntu**
```bash
sudo ./hardening-tool_ubuntu
``

The Linux hardening tools may require administrator privileges to inspect or change system settings. Only run them on a disposable or approved test machine until you have reviewed the applicable policies and actions.

The Go application's source code prints that the local dashboard is available at `http://localhost:8080`. If you run from source, use the correct platform folder and ensure its relative policy, UI, and database paths are available.

### Option 2: Run from Go source

A Go toolchain and dependencies are required. For the CentOS source tree:

```bash
cd centos
go mod download
go run ./cmd/app
```

For Ubuntu, the source is under `ubuntu/SIH25`:

```bash
cd ubuntu/SIH25
go mod download
go run ./cmd/app
```

After the application starts, try opening [http://localhost:8080](http://localhost:8080) in a browser.

**Important:** this repository has platform-specific relative paths and policies. The commands above are starting points based on the source layout; successful execution may require running on the intended operating system, using the expected working directory, and checking that the correct policy files exist. Validate the runtime behaviour in your environment.

## API Overview

The Go application defines the following local API routes:

| Method | Endpoint | Purpose |
|---|---|---|
| `GET` | `/api/status` | Returns application status and detected OS. |
| `GET` | `/api/scan?level=basic` | Runs a policy audit for a selected profile. Supported levels in code: `basic`, `moderate`, `strict`. |
| `POST` | `/api/fix` | Requests a fix for the supplied rule ID. |
| `POST` | `/api/rollback` | Requests rollback for a supplied rule ID. |
| `GET` | `/api/export?level=strict` | Generates and returns a PDF report for the selected profile. |
| `POST` | `/api/reset` | Attempts to reverse fixes in the loaded policy. |

For the `/api/fix` and `/api/rollback` routes, the request body is JSON containing a rule ID, for example:

```json
{
  "id": "RULE_ID_FROM_POLICY"
}
```

Use only rule IDs that exist in the policy loaded by the running instance. These endpoints can modify operating-system configuration; do not test them against production systems without a controlled validation and recovery plan.

## Safe Testing Guidance

SentinelX can make privileged configuration changes. Before testing:

- Use a virtual machine, disposable lab machine, or approved test environment.
- Read the relevant policy file and understand the command or setting each remediation changes.
- Take a VM snapshot or backup before applying fixes.
- Run an audit before and after each change.
- Review generated reports and compare them with the machine's actual state.
- Do not assume rollback perfectly restores the original configuration.
- Avoid running unfamiliar scripts or binaries with administrator privileges on personal or production machines.

## Scope and Limitations

- The repository contains platform-specific components; the same controls may not be available on every OS.
- Audit coverage is limited to the rules implemented in the loaded policy.
- A `FAIL` result indicates that a check did not pass; it is not proof of an intrusion.
- A recorded action or successful API response should be followed by an audit to verify the resulting state.
- Rollback and reset depend on the actions defined in the policies and are not guaranteed to restore every aspect of the prior system state.
- The inspected application runs a local web server. This README does not claim that the current code implements central management of remote machines or a persistent network-connected agent fleet.
- This project is a hardening and configuration-audit tool; it should not be treated as a replacement for endpoint detection, antivirus, patch management, or a full security monitoring platform.

## Project Context

SentinelX is being presented for **Infothon / SIH problem statement PS25237**. The repository includes the original project presentation at [SIH_PS25237_SentinelX.pptx.pdf](./SIH_PS25237_SentinelX.pptx.pdf).

The current codebase provides a foundation for policy-based auditing, corrective actions, rollback-related operations, and report generation. Any additional features should be documented as implemented only after they have been integrated and tested.

## Contributing

Contributions should be made through focused changes that are easy to review. When modifying a policy or remediation:

1. Explain the security objective.
2. Document the systems and versions the rule applies to.
3. Review the check, remediation, and rollback commands together.
4. Test on a disposable system.
5. Capture before-and-after results and update documentation.

## License

A license file was not identified in the repository during preparation of this README. Unless a license is added by the repository owner, reuse and redistribution terms should be treated as unspecified.

---

<p align="center">
  <strong>SentinelX — making system hardening more structured, visible, and reviewable.</strong>
</p>
