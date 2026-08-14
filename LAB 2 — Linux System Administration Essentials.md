# LAB 2 — Linux System Administration Essentials

**File Systems | Services | Logs | User Management**

> **🎯 Objective:** Develop the core system administration skills required to manage, monitor, and troubleshoot Linux servers before and after patching activities.

---

## TASK 2.1 — File System Management

### 1. Check Disk Space Usage

```bash
df -hT
```

### 2. Find the Top 10 Largest Directories

```bash
du -ah / --max-depth=2 | sort -rh | head -10
```

### 3. Check Inode Usage

> **⚠️ Important:** This is important for patching because log floods can exhaust available inodes.

```bash
df -i
```

### 4. Mount a New File System

```bash
sudo mount /dev/sdb1 /mnt/data
sudo umount /mnt/data
```

### 5. Make the Mount Persistent

Add the following entry to `/etc/fstab`:

```text
/dev/sdb1 /mnt/data ext4 defaults 0 2
```

### 6. Extend an LVM Volume

> **💡 Note:** Extending LVM volumes is a common task in enterprise Linux environments.

```bash
sudo lvextend -L +10G /dev/vg0/lv_root
sudo resize2fs /dev/vg0/lv_root
```

---

## TASK 2.2 — Service Management with `systemd`

### 1. Check the Status of a Service

```bash
sudo systemctl status sshd
```

### 2. Start / Stop / Restart / Reload a Service

```bash
sudo systemctl start|stop|restart|reload <service>
```

### 3. Enable / Disable a Service on Boot

```bash
sudo systemctl enable <service>
sudo systemctl disable <service>
```

### 4. List All Failed Services

> **🔎 Post-Patch Health Check:** Run this after patching and rebooting to identify services that failed to start.

```bash
systemctl --failed
```

### 5. View Service Journal Logs

```bash
sudo journalctl -u <service> -n 50 --no-pager
```

### 6. Check All Running Services

```bash
systemctl list-units --type=service --state=running
```

---

## TASK 2.3 — Log Management & Analysis

### 1. View the System Log in Real Time

**RHEL:**

```bash
sudo tail -f /var/log/messages # RHEL
```

**Ubuntu:**

```bash
sudo tail -f /var/log/syslog # Ubuntu
```

### 2. Search for Errors in Logs

```bash
sudo grep -i 'error\|failed\|critical' /var/log/messages
```

### 3. View the Authentication Log

> **🔐 Security Audit:** Authentication logs are important for security and audit activities.

**RHEL:**

```bash
sudo cat /var/log/secure # RHEL
```

**Ubuntu:**

```bash
sudo cat /var/log/auth.log # Ubuntu
```

### 4. Use `journalctl` for `systemd` Logs

Filter by time to isolate events during a specific period. This is particularly useful after patching.

```bash
sudo journalctl --since '2024-01-15 10:00' --until '2024-01-15 12:00'
```

### 5. Check the Boot Log

> **🔎 Post-Patch Reboot Validation:** Review the boot log after a patch-related reboot.

```bash
sudo journalctl -b
```

### 6. Rotate Logs Manually

```bash
sudo logrotate -f /etc/logrotate.conf
```

---

## 🏆 BEST PRACTICE TIP

* **Always check disk space (`df -hT`) before patching** — a full `/var` partition will cause patch failures.
* **Run `systemctl --failed` immediately after every reboot** to catch services that did not come back up.
* **Use `journalctl` with time filters** to isolate log entries from the exact patching window.
* **Forward logs to a central SIEM (Splunk / ELK) before patching** so pre/post states are captured.
* **Document baseline service states before patching:**

```bash
systemctl list-units --state=running > pre_patch_services.txt
```

> **📋 Operational Reminder:** Capture pre-patch system state and relevant logs before making changes so that post-patch validation and troubleshooting can be performed against a known baseline.
