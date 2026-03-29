---
title: "Building a VMware Security Compliance Checker with Claude Plugins"
date: 2026-03-29 12:00:00 +0200
categories: [VMware, AI]
tags: [claude, mcp, vmware, compliance, security, python]
---

*How we used Claude's MCP connector system to automate CIS and NIST audits across a private cloud infrastructure*

---

## Claude Plugins: A New Way to Work

Claude has recently introduced a powerful integration system called **Connectors** — sometimes referred to as plugins — built on Anthropic's open-source **Model Context Protocol (MCP)**. Instead of copy-pasting data into a chat window, connectors let Claude reach directly into the tools you already use: Google Drive, Slack, Jira, and now, your own internal systems.

The real shift here is architectural. A connector transforms Claude from a conversational assistant into an active participant in your workflow. It can read, analyze, and act — not just respond. And because MCP is an open standard, any team can build a custom connector for any internal system, which is exactly what we did.

---

## The Problem: Compliance Audits Are Painful

If you manage a VMware-based private cloud, you already know the pain. Every quarter (or more often, if you're in a regulated industry), someone has to manually check that your virtual machines are configured correctly — that encryption is enabled, that old snapshots have been cleaned up, that copy-paste between the VM and the host is locked down.

This process typically involves:

- Logging into vCenter and clicking through dozens of VM settings screens
- Cross-referencing each setting against a printed or PDF benchmark document
- Filling in a spreadsheet by hand
- Writing a report that's already outdated by the time it's finished

It's slow, error-prone, and deeply unsatisfying work for any competent engineer. Worse, the gap between audits means misconfigurations can sit undetected for weeks or months.

We wanted to fix this.

---

## The Benchmarks We Use

Our checker validates VM configurations against two industry-standard frameworks.

### CIS VMware ESXi Benchmark v1.3

The **Center for Internet Security (CIS)** publishes hardening benchmarks for virtually every major platform. The VMware ESXi benchmark covers everything from firmware settings to inter-VM isolation controls. Each check is clearly defined, scored by severity, and comes with explicit remediation steps. It is the de facto standard for VMware hardening in enterprise environments.

### NIST SP 800-53 Rev 5

The **National Institute of Standards and Technology** 800-53 framework provides a catalog of security and privacy controls for federal information systems — but it has become the baseline for compliance across finance, healthcare, and critical infrastructure globally. Each of our CIS checks maps directly to a NIST control family, such as AC (Access Control), CM (Configuration Management), SC (System and Communications Protection), and SI (System and Information Integrity). This dual mapping means a single scan produces evidence usable for both technical hardening reviews and formal compliance audits.

---

## What We Built, Step by Step

### Step 1: Defining the Rules Engine

The foundation of the system is a declarative JSON rules file — `cis_nist_rules.json` — that defines every check we want to run. Each rule specifies the CIS check ID, a human-readable title, severity level, the corresponding NIST control, what configuration key to inspect, what the expected value should be, and a remediation instruction.

Keeping the rules in JSON rather than hardcoding them in Python means a security engineer can update or extend the benchmark coverage without touching application logic. Adding a new check is as simple as adding a new object to the array.

```json
{
  "id": "CIS-3.1",
  "title": "Disable Copy/Paste between VM and host",
  "severity": "HIGH",
  "nist_mapping": "AC-3",
  "check_type": "vm_extra_config",
  "check_key": "isolation.tools.copy.disable",
  "expected_value": "TRUE",
  "remediation": "Set isolation.tools.copy.disable = TRUE in VM Advanced Configuration"
}
```

### Step 2: Building the MCP Server

The MCP server (`vmware_compliance_mcp.py`) is what makes this a Claude plugin rather than just a standalone script. It exposes a set of callable tools that Claude can invoke during a conversation:

- `list_vms` — returns the VM inventory from vCenter
- `scan_vm <name>` — runs all CIS checks against a single VM and returns a structured result
- `scan_all` — scans every VM and produces an aggregated report
- `show_failed` — filters results to show only failing checks with their remediation steps

The compliance engine at the heart of the server takes raw VM configuration data and evaluates each rule against it, computing a score (0–100), a risk level (LOW / MEDIUM / HIGH), and a per-check pass/fail/warn status. The output is structured data, not prose — which means Claude can reason over it, summarize it, prioritize remediations, or forward it to other systems.

### Step 3: Connecting to Real vCenter

The mock data layer we built for development gets replaced by `vcenter_api.py`, a client for the vCenter REST API (available from vSphere 7.0 onward). The client handles session authentication via token (not per-request basic auth), then pulls VM inventory and configuration data through the `/api/vcenter/vm` endpoints.

The key design decision here was a factory function, `get_vm_data()`, that accepts a boolean flag. During development and testing it returns mock VMs; in production it connects to a real vCenter host. Switching between modes requires changing a single line.

```python
# Development
vms = get_vm_data(use_real_vcenter=False)

# Production
vms = get_vm_data(
    use_real_vcenter=True,
    host="vcenter.company.local",
    username="admin@vsphere.local",
    password="YourPassword"
)
```

One important note: the vCenter REST API does not expose VM snapshots directly in v7/v8. Snapshot data requires either the older SOAP API via `pyVmomi`, or the vSphere Automation SDK. We documented this gap clearly in the code so it doesn't become a silent blind spot.

### Step 4: The Interactive Dashboard

A compliance score in a terminal is useful. A compliance score in a well-designed dashboard is actionable. We built `compliance_dashboard.html` — a single self-contained HTML file with no external dependencies beyond a Google Fonts import.

The dashboard renders:

- Five KPI cards at the top showing overall score, high-risk VM count, total failures, total passed checks, and warning count
- A score bar section showing each VM's compliance percentage with color-coded severity
- A clickable VM list with circular score rings and risk badges
- A detail panel showing all ten CIS checks for the selected VM, with filters for All / Fail / Pass / Warn
- A simulated "Run Scan" flow with step-by-step status messages

The aesthetic was deliberately chosen to match the environment: dark, dense, technical. Security operations teams spend hours looking at these interfaces and deserve something that reads clearly under pressure.

### Step 5: PDF Export

The final piece is a programmatic PDF report generated with `reportlab`. The report is designed to serve two audiences simultaneously: the engineer who needs to see exactly which configuration key failed and how to fix it, and the compliance officer or auditor who needs a timestamped, printable document.

The PDF includes a cover summary page with the same KPI layout as the dashboard, followed by one dedicated page per VM. Each VM page shows a large score, risk level, and a full check table with status icons, severity tags, NIST control references, and remediation instructions for every failed check.

The report is generated as a dark-themed document — consistent with the dashboard — and includes header and footer metadata on every page: report title, standard references, generation timestamp, and page number.

```python
from generate_pdf import build_pdf
from vcenter_api import get_vm_data

vms = get_vm_data(use_real_vcenter=True, host="...", username="...", password="...")
build_pdf("compliance_report.pdf", vm_data=vms)
```

---

## Results

Running the checker against our three test VMs produced the following:

| VM | Score | Risk | Failures |
|---|---|---|---|
| prod-db-01 | 100% | LOW | 0 |
| prod-web-01 | 60% | HIGH | 4 |
| dev-app-03 | 0% | HIGH | 9 |

The production database server was fully compliant. The web server had four issues — disk encryption missing, paste from host enabled, CD/DVD connected, and a 45-day-old snapshot. The development VM was essentially unconfigured from a security standpoint, which is common for dev environments but important to document and isolate.

The entire scan, from connecting to vCenter to producing a PDF, takes under 30 seconds for a small environment.

---

## What's Next

The current implementation covers the ten highest-impact CIS VMware checks. Natural extensions include:

- **ESXi host-level checks** alongside VM-level checks
- **Scheduled scanning** with delta reports showing what changed between runs
- **Webhook integration** with Jira or ServiceNow to automatically open tickets for new failures
- **Email delivery** of the PDF report on a weekly schedule
- **Trend tracking** to visualize whether the environment's overall compliance score is improving over time

The plugin architecture makes all of these straightforward to add: each one is a new MCP tool that Claude can call, reason about, and chain with the others.

---

## Closing Thoughts

What made this project interesting wasn't any single technical component — it was the way Claude's plugin system changed the interaction model. Instead of running a script and reading a log file, a security engineer can ask Claude in natural language: *"Which of our production VMs have disk encryption disabled?"* and get a direct, actionable answer backed by live data.

That shift — from tool to colleague — is what MCP connectors are really about. The compliance checker is one example. The pattern applies anywhere you have structured internal data and a team that needs to reason over it quickly.

---

*All code from this post is available as a four-file package: `cis_nist_rules.json`, `vmware_compliance_mcp.py`, `vcenter_api.py`, `compliance_dashboard.html`, and `generate_pdf.py`.*
