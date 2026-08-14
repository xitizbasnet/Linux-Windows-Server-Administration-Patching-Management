# LAB 1 — Linux Package Management Fundamentals

**`yum` / `dnf` (RHEL/CentOS) | `apt` (Ubuntu) | `zypper` (SUSE)**

> **🎯 Objective:** Master the three major Linux package managers used in enterprise environments. Each distro family uses its own tool; knowing all three is mandatory for a patching engineer.

---

## TASK 1.1 — `yum` / `dnf` Package Management (RHEL 7/8/9, CentOS)

### 1. Check the OS Version First

Always confirm your distribution before running commands.

```bash
cat /etc/os-release
```

### 2. List All Available Updates

```bash
sudo yum check-update # RHEL 7 / CentOS 7
sudo dnf check-update # RHEL 8/9
```

### 3. View Security-Only Updates

> **🔐 Important:** Security-only update visibility is critical for the patching role.

```bash
sudo yum updateinfo list security
sudo dnf updateinfo list security
```

### 4. Install All Available Security Patches

```bash
sudo yum update --security -y
sudo dnf update --security -y
```

### 5. Install a Specific Package

```bash
sudo dnf install <package-name> -y
```

### 6. Remove a Package

```bash
sudo dnf remove <package-name> -y
```

### 7. View Installed Package History

Use package transaction history for audit trails.

```bash
sudo dnf history
sudo dnf history info <transaction_id>
```

### 8. Roll Back a Bad Update Using a Transaction ID

```bash
sudo dnf history undo <transaction_id>
```

### 9. Clean the Package Cache

```bash
sudo dnf clean all
```

### 10. Reboot After Kernel Updates and Confirm the New Kernel Version

```bash
sudo reboot
uname -r
```

---

## TASK 1.2 — `apt` Package Management (Ubuntu / Debian)

### 1. Update the Package Index

> **💡 Best Practice:** Always perform this step before installing or upgrading packages.

```bash
sudo apt update
```

### 2. List Upgradeable Packages

```bash
apt list --upgradeable
```

### 3. Apply Security Updates Only (Ubuntu)

```bash
sudo unattended-upgrade --dry-run # Preview
sudo unattended-upgrade # Apply
```

### 4. Upgrade All Packages

```bash
sudo apt upgrade -y
```

### 5. Full Distribution Upgrade (`dist-upgrade`)

```bash
sudo apt dist-upgrade -y
```

### 6. Remove Unused Packages

```bash
sudo apt autoremove -y
```

### 7. View the `apt` Log for Audit Purposes

```bash
cat /var/log/apt/history.log
```

### 8. Check If a Reboot Is Required After Patching

```bash
cat /var/run/reboot-required
```

---

## TASK 1.3 — `zypper` Package Management (SUSE / openSUSE)

### 1. Refresh Repositories

```bash
sudo zypper refresh
```

### 2. List Available Patches

```bash
sudo zypper list-patches
```

### 3. Apply Security Patches Only

```bash
sudo zypper patch --category security
```

### 4. Apply All Patches

```bash
sudo zypper patch -y
```

### 5. Search for a Package

```bash
sudo zypper search <package>
```

### 6. Install a Package

```bash
sudo zypper install <package> -y
```

### 7. Check for Lock Files Before Patching

```bash
sudo zypper locks
```

---

## 🏆 BEST PRACTICE TIP

* **Always run `check-update` / `apt update` before applying any patches** — never assume repositories are current.
* **Keep patch history logs** using `dnf history` / `/var/log/apt/history.log` for compliance audits.
* **Test patches on non-production** (`DEV → QA → PROD`) before production rollout.
* **Lock critical package versions** using `dnf versionlock` or `apt-mark hold` to prevent accidental upgrades.
* **After every kernel update, reboot and validate services are healthy** using `systemctl --failed`.
* **Use `needs-restarting` (RHEL)** or check `/var/run/reboot-required` (Ubuntu) to detect mandatory reboots.

> **⚠️ Operational Reminder:** Perform package-management activities according to your organization's approved patching, change-management, backup, and rollback procedures.
