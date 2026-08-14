# LAB 3 — Bash Scripting for Patching Automation

**Scripts | Loops | Cron Jobs | Pre/Post Checks**

> **🎯 Objective:** Automation with Bash is a core skill for patching engineers. These tasks teach you to write reusable scripts that perform pre-patch checks, apply updates, and validate results.

---

## TASK 3.1 — Pre-Patch Health Check Script

### 1. Create the Script File

```bash
vi /home/vinod/pre_patch_check.sh
```

### 2. Write the Pre-Patch Check Script

Paste the following content:

```bash
#!/bin/bash
# Pre-Patch Health Check Script
# Author: Vinod Muleva | Date: $(date)
LOGFILE="/var/log/pre_patch_$(date +%Y%m%d).log"
echo "=== PRE-PATCH HEALTH CHECK: $(date) ===" | tee $LOGFILE
echo "-- Hostname --" | tee -a $LOGFILE
hostname -f | tee -a $LOGFILE
echo "-- OS Version --" | tee -a $LOGFILE
cat /etc/os-release | tee -a $LOGFILE
echo "-- Disk Space --" | tee -a $LOGFILE
df -hT | tee -a $LOGFILE
echo "-- Memory --" | tee -a $LOGFILE
free -h | tee -a $LOGFILE
echo "-- Failed Services --" | tee -a $LOGFILE
systemctl --failed | tee -a $LOGFILE
echo "-- Available Updates --" | tee -a $LOGFILE
dnf check-update 2>&1 | tee -a $LOGFILE
echo "=== CHECK COMPLETE ===" | tee -a $LOGFILE
```

### 3. Make the Script Executable and Run It

```bash
chmod +x /home/vinod/pre_patch_check.sh
sudo ./pre_patch_check.sh
```

### 4. Review the Generated Log File

```bash
cat /var/log/pre_patch_$(date +%Y%m%d).log
```

---

## TASK 3.2 — Automated Patching Script

### 1. Create the Patching Automation Script

```bash
vi /home/vinod/auto_patch.sh
```

### 2. Write the Script Content

```bash
#!/bin/bash
# Automated Patching Script — Security Updates Only
LOGFILE="/var/log/patch_$(date +%Y%m%d_%H%M).log"
echo "Patch Start: $(date)" | tee $LOGFILE
# Detect OS family
if [ -f /etc/redhat-release ]; then
dnf update --security -y 2>&1 | tee -a $LOGFILE
elif [ -f /etc/debian_version ]; then
apt-get update -y && apt-get upgrade -y 2>&1 | tee -a $LOGFILE
elif [ -f /etc/SuSE-release ]; then
zypper patch --category security -y 2>&1 | tee -a $LOGFILE
fi
echo "Patch End: $(date)" | tee -a $LOGFILE
# Check if reboot required
needs-restarting -r 2>/dev/null && echo 'REBOOT REQUIRED' | tee -a $LOGFILE
```

### 3. Schedule with `cron` to Run Every First Sunday at 2 AM

```bash
crontab -e
```

Add this line:

```cron
# Add this line:
0 2 1-7 * 0 /home/vinod/auto_patch.sh >> /var/log/cron_patch.log 2>&1
```

### 4. Verify `cron` Is Scheduled

```bash
crontab -l
```

---

## 🏆 BEST PRACTICE TIP

* **Always test scripts with `--dry-run` or in a VM** before running on production servers.
* **Use `set -euo pipefail` at the top of every Bash script** to catch errors early.
* **Log every patching action with timestamps to a dedicated log file** — required for compliance.
* **Validate that your `cron` user has `sudo` privileges** before scheduling automated patching.
* **Send email/Slack alerts at the end of the script:**

```bash
mail -s "Patch Done" admin@company.com < $LOGFILE
```

> **⚠️ Operational Reminder:** Automated patching should follow your organization's approved maintenance windows, change-management procedures, access controls, and rollback plans.
