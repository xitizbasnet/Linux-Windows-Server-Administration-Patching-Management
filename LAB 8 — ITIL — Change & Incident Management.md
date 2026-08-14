# LAB 8 — ITIL — Change & Incident Management

**Change Records | CAB | RCA | ServiceNow Workflow**

> **🎯 Objective:** Patching is a formal IT process governed by ITIL. Every patch deployment must go through Change Management to minimize risk and ensure accountability.

---

## TASK 8.1 — Raising a Change Request (CR) for Patching

### 1. Log In to the ITSM Platform

Log into **ServiceNow / Jira Service Management**.

### 2. Create a Standard Change

Navigate to:

**Change → Create New → Standard Change**

Use a **Standard Change** for routine monthly patching.

### 3. Complete the Change Request Fields

Fill in the Change Request fields:

| Field                   | Details                                             |
| ----------------------- | --------------------------------------------------- |
| **Title**               | Monthly Security Patch — Web Servers — January 2024 |
| **Category**            | Patching / Maintenance                              |
| **Risk**                | Low/Medium                                          |
| **Affected CI**         | List of servers from CMDB                           |
| **Implementation Plan** | Step-by-step patching procedure                     |
| **Rollback Plan**       | VM snapshot restore / `dnf history undo`            |
| **Test Plan**           | Smoke tests, service health checks                  |
| **Maintenance Window**  | Saturday 02:00–06:00 IST                            |

### 4. Submit for CAB Approval

Submit the Change Request for **CAB (Change Advisory Board)** approval — typically during the weekly meeting.

### 5. Execute the Approved Change

After approval, execute the change **during the approved maintenance window only**.

### 6. Update and Close the Change Request

Update the CR status:

**In Progress → Close**

Include the appropriate implementation notes.

---

## TASK 8.2 — Incident Management During Patching

### 1. Raise an Incident if a Patch Causes an Issue

If a patch causes an issue, immediately raise a **P1/P2 Incident** in ServiceNow.

### 2. Execute the Rollback Plan

First, review the available transaction history:

```bash id="kqf6k6"
sudo dnf history list
```

Roll back the affected package transaction:

```bash id="d8p5oc"
sudo dnf history undo <transaction_id> # Roll back packages
```

If the kernel was changed, reboot:

```bash id="c6sv1b"
sudo reboot # If kernel was changed
```

### 3. Restore the VM Snapshot if Required

Restore the VM snapshot if package rollback is insufficient.

### 4. Communicate Status Updates

Communicate status updates **every 30 minutes** to stakeholders via the bridge call.

### 5. Close the Incident After Service Restoration

Once the service is restored, close the Incident with appropriate resolution notes.

### 6. Raise a Problem Record and Conduct RCA

Raise a **Problem** record and conduct a **Root Cause Analysis (RCA)** within 48 hours.

---

## TASK 8.3 — RCA (Root Cause Analysis) Template

| Section               | Content                                                                         |
| --------------------- | ------------------------------------------------------------------------------- |
| **Incident Title**    | Application outage after kernel patch — WebServer01                             |
| **Date & Time**       | 2024-01-15, 03:15 IST — Detected during patching window                         |
| **Impact**            | 100% users unable to access application for 45 minutes                          |
| **Root Cause**        | Kernel 5.14.0-362 introduced incompatibility with app driver                    |
| **Timeline**          | 02:00 - Patch applied | 03:15 - App alert | 03:30 - Rollback | 03:45 - Restored |
| **Immediate Fix**     | Rolled back kernel using `dnf history undo`, restored previous kernel           |
| **Permanent Fix**     | Kernel version pinned; test on staging with app driver before production        |
| **Preventive Action** | Add driver compatibility check to pre-patch validation script                   |
| **Owner**             | Vinod Muleva — Patch Engineer                                                   |

---

## 🏆 BEST PRACTICE TIP

* **Every patch-related change must have a documented rollback plan before CAB approval.**
* **Use Standard Changes for routine monthly patching; Emergency Changes for critical CVEs.**
* **Complete RCA within 48 hours of an incident** — delay erodes trust and breaks compliance.
* **Store all Change Records and RCAs for a minimum of 1 year** for audit purposes.
* **Track patch-related incidents separately** to identify systemic patching quality issues.

> **⚠️ Operational Reminder:** No production patch should proceed outside its approved change window unless the appropriate emergency-change process has been invoked and authorized.
