# LAB 4 — Linux Patching — Full Patch Cycle

**Plan | Schedule | Validate | Deploy | Verify | Document**

> **🎯 Objective:** This lab simulates a real-world monthly patching cycle from planning to documentation, aligned with ITIL Change Management processes.

---

## TASK 4.1 — Pre-Patch Planning & Scheduling

### 1. Identify Servers in Scope

Get the inventory list from the CMDB or a CSV file:

```bash
cat /etc/ansible/hosts # Or your inventory file
```

### 2. Check the Current Kernel Version on All Servers Before Patching

```bash
uname -r
```

### 3. Check Which CVEs/Security Advisories Are Pending

```bash
sudo dnf updateinfo list cves --security
sudo dnf updateinfo list --security
```

### 4. Create a Patching Window in the ITSM Tool

Create the maintenance window in **ServiceNow/Jira** with:

* Affected servers
* Maintenance window date/time
* Rollback plan
* Contact details

### 5. Notify Application Owners and Get Approval

Notify the application owners and obtain approval for the maintenance window.

### 6. Take a Snapshot/Backup Before Patching

For VM environments:

```bash
aws ec2 create-snapshot --volume-id vol-xxxx --description 'pre-patch-$(date +%Y%m%d)'
```

---

## TASK 4.2 — Patch in Non-Production (Validation)

### 1. Deploy Patches to the DEV Environment First

Run the following on the DEV server:

```bash
sudo dnf update --security -y # On DEV server
```

### 2. Run Application Smoke Tests After Patching DEV

Validate that the application continues to work as expected.

### 3. Check for Broken Dependencies

```bash
sudo dnf check
sudo rpm -Va 2>>&1 | grep -v 'c '
```

### 4. Promote Patches to QA/UAT

If DEV is healthy, promote the patches to **QA/UAT** and repeat the validation.

### 5. Document Validation Results

Document:

* Which patches were applied
* Any issues found

### 6. Get Sign-Off Before Production

Get sign-off from the **QA/App team** before promoting patches to production.

---

## TASK 4.3 — Production Patching & Post-Patch Validation

### 1. Connect to the Production Server

During the approved maintenance window, connect to the production server.

### 2. Run the Pre-Patch Health Check Script

Use the script from **Lab 3**:

```bash
sudo ./pre_patch_check.sh
```

### 3. Apply Security Patches

```bash
sudo dnf update --security -y 2>&1 | tee /var/log/prod_patch_$(date +%Y%m%d).log
```

### 4. Reboot If the Kernel Was Updated

If the kernel was updated, schedule a reboot during the maintenance window:

```bash
sudo needs-restarting -r
sudo reboot
```

### 5. Verify the New Kernel and Services After Reboot

```bash
uname -r
systemctl --failed
systemctl status <critical-app-service>
```

### 6. Verify Critical Application Endpoints

Verify that critical application endpoints are responding by running the appropriate smoke tests.

### 7. Close the Change Request

Close the **Change Request** in ServiceNow with:

* Patch status
* Completion time

### 8. Retain the Patch Log for Compliance Reporting

```bash
cat /var/log/prod_patch_$(date +%Y%m%d).log
```

---

## 🏆 BEST PRACTICE TIP

* **Never patch production without a validated rollback plan** (snapshot, LVM snapshot, or backup).
* **Always patch in the order:** `DEV → QA/UAT → STAGING → PRODUCTION`.
* **Coordinate maintenance windows with application teams at least 1 week in advance.**
* **Keep a “freeze period” (no patching) around major releases or year-end business activities.**
* **Document every patch applied:** CVE ID, package name, version before/after, reboot status.
* **After patching, monitor system metrics** (CPU, memory, disk I/O) for at least 30 minutes.

> **⚠️ Operational Reminder:** Production patching should be performed only within an approved maintenance window and in accordance with your organization's change, backup, rollback, and application-validation procedures.
