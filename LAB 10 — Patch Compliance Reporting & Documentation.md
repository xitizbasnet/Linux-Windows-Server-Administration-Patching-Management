# LAB 10 — Patch Compliance Reporting & Documentation

**1L0AB**

**Reports | Dashboards | Compliance Records | Runbooks**

> **🎯 Objective:** Patching without documentation is incomplete. This lab covers how to produce compliance reports, maintain patching records, and build the documentation trail required for audits.

---

## TASK 10.1 — Generating Patch Compliance Reports (Linux)

### 1. Generate a List of Installed Packages with Versions (RHEL)

```bash
rpm -qa --queryformat '%{NAME}-%{VERSION}-%{RELEASE}.%{ARCH}\n' | sort > /tmp/installed_packages.txt
```

### 2. Generate a List of Recently Applied Patches (Last 30 Days)

```bash
dnf history | head -30
dnf history info <id> | grep 'Install\|Updated'
```

### 3. Check Pending Security Updates (Compliance Gap Report)

```bash
dnf updateinfo summary --security
dnf updateinfo list --security | wc -l # Count of pending patches
```

### 4. Generate a Compliance Summary Script and Save to File

```bash
echo "Hostname: $(hostname)" > /tmp/patch_compliance.txt
echo "Date: $(date)" >> /tmp/patch_compliance.txt
echo "Kernel: $(uname -r)" >> /tmp/patch_compliance.txt
dnf updateinfo summary --security >> /tmp/patch_compliance.txt
cat /tmp/patch_compliance.txt
```

### 5. For Ubuntu — Check Compliance Status

```bash
apt list --upgradeable 2>/dev/null | wc -l
ubuntu-security-status
```

---

## TASK 10.2 — Patch Record Template (for Every Patching Cycle)

| Field                      | Value                                           |
| -------------------------- | ----------------------------------------------- |
| **Server Name**            | web01.company.com                               |
| **IP Address**             | 10.10.1.50                                      |
| **OS**                     | RHEL 8.8                                        |
| **Patch Date**             | 2024-01-15                                      |
| **Maintenance Window**     | 02:00 – 04:00 IST                               |
| **Patches Applied**        | 48 (Security: 12, Bugfix: 36)                   |
| **Critical CVEs Resolved** | CVE-2023-1234, CVE-2023-5678                    |
| **Kernel Before**          | 5.14.0-362.el8.x86_64                           |
| **Kernel After**           | 5.14.0-427.el8.x86_64                           |
| **Reboot Required**        | Yes — rebooted at 03:45 IST                     |
| **Services Validated**     | httpd, mysql, application_service — All Running |
| **Change Request ID**      | CHG0012345                                      |
| **Patched By**             | Vinod Muleva                                    |
| **Status**                 | COMPLETED — No Issues                           |

---

## TASK 10.3 — Compliance Dashboard KPIs

| KPI                             | Metric | Target    | Description                                  |
| ------------------------------- | ------ | --------- | -------------------------------------------- |
| **Patch Compliance Rate**       | —      | >= 95%    | % of servers fully patched within SLA        |
| **Critical CVE MTTR**           | —      | <= 48 hrs | Mean Time to Remediate Critical CVEs         |
| **High CVE MTTR**               | —      | <= 7 days | Mean Time to Remediate High CVEs             |
| **Patch Success Rate**          | —      | >= 99%    | % of patches applied without rollback        |
| **Patch-Induced Incidents**     | —      | < 2/month | Incidents caused by patching activities      |
| **Change Approval Rate**        | —      | 100%      | All patches must have approved Change Record |
| **Reboot Compliance**           | —      | >= 98%    | Servers rebooted within 24h of kernel patch  |
| **Vulnerability Scan Coverage** | —      | 100%      | All servers scanned monthly                  |

---

## 🏆 BEST PRACTICE TIP

* **Automate monthly compliance reports** using scripts that pull from WSUS / Satellite / Ansible facts.
* **Store all patch records in a shared drive or ITSM tool** — never on a local engineer's machine.
* **Review and present patch KPIs to management monthly** to demonstrate value and compliance posture.
* **Use dashboards in Grafana, Power BI, or ServiceNow** to visualize compliance trends over time.
* **Retain all patch documentation for a minimum of 1 year** (SOC 2 / ISO 27001 requirement).
* **Build runbooks for each patching procedure** so any team member can execute in an emergency.

> **📋 Documentation Principle:** Every patching cycle should leave a complete audit trail covering the systems affected, patches applied, vulnerabilities addressed, validation results, change records, reboot status, incidents, and final compliance status.
