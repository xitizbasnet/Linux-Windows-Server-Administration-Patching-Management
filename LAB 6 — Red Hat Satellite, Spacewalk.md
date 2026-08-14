# LAB 6 — Red Hat Satellite / Spacewalk

**Content Views | Lifecycle Environments | Errata Management**

> **🎯 Objective:** Red Hat Satellite is the enterprise patch management platform for RHEL environments. It allows centralized management of content, patching, and compliance across thousands of servers.

---

## TASK 6.1 — Satellite Server Setup & Registration

### 1. Register a RHEL Server to Satellite

```bash
sudo subscription-manager register --org='MyOrg' --activationkey='patch-key'
```

### 2. Verify Registration

```bash
sudo subscription-manager status
sudo subscription-manager list --consumed
```

### 3. Install the Katello Agent for Satellite Management

```bash
sudo dnf install katello-agent -y
sudo systemctl enable goferd --now
```

### 4. Verify the Host in the Satellite UI

Verify that the host appears in the Satellite UI under:

**Hosts → All Hosts**

### 5. Assign the Host to the Correct Host Group and Lifecycle Environment

Assign the host to the appropriate:

* **Host Group**
* **Lifecycle Environment:** `DEV` / `QA` / `PROD`

---

## TASK 6.2 — Content Views & Patching via Satellite

### 1. Create a Content View

In the Satellite Web UI, navigate to:

**Content → Content Views → Create New View**

### 2. Add Repositories to the Content View

Add the required repositories:

* BaseOS
* AppStream
* Security

### 3. Publish a New Version of the Content View

Publish a new version of the content view after synchronization.

### 4. Promote the Content View to the DEV Lifecycle Environment

Promote the newly published content view to the **DEV** lifecycle environment first.

### 5. Apply Errata to a Specific Host from the CLI

```bash
hammer host errata apply --errata-ids RHSA-2024:001 --host web01.company.com
```

### 6. Apply All Applicable Security Errata to a Host

```bash
hammer host errata apply --types security --host web01.company.com
```

### 7. List Available Errata for a Host

```bash
hammer host errata list --host web01.company.com --errata-restrict-applicable
true
```

### 8. Promote the Content View to PROD

After validation in the lower environments, promote the content view to **PROD**.

---

## 🏆 BEST PRACTICE TIP

* **Always sync Satellite repositories with Red Hat CDN before a patching cycle begins.**
* **Use lifecycle environments (`DEV → QA → PROD`)** to control which patches reach which servers.
* **Content Views act as a “snapshot” of packages** — always publish a new version before promoting.
* **Use Activation Keys** to automatically assign the correct content and lifecycle environment at registration time.
* **Schedule errata application during maintenance windows** using Satellite's remote execution feature.
* **Use the `hammer` CLI for scripting bulk operations across many hosts** instead of the UI.

> **⚠️ Operational Reminder:** Validate content, errata, lifecycle promotion, and host assignments in non-production environments before promoting patches to production. Ensure production changes follow the approved maintenance window and change-management process.
