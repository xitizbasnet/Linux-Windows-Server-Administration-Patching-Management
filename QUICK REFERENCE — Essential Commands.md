# QUICK REFERENCE — Essential Commands

## Essential Linux and Ansible Commands

| Command / Action           | Linux (RHEL)                                | Linux (Ubuntu)                              | Notes              |
| -------------------------- | ------------------------------------------- | ------------------------------------------- | ------------------ |
| **Update package index**   | `dnf check-update`                          | `apt update`                                | Always run first   |
| **Apply security patches** | `dnf update --security -y`                  | `unattended-upgrade`                        | Core patching cmd  |
| **Check pending patches**  | `dnf updateinfo list --security`            | `apt list --upgradeable`                    | Pre-patch report   |
| **Reboot required check**  | `needs-restarting -r`                       | `cat /var/run/reboot-required`              | Post-patch must-do |
| **View patch history**     | `dnf history`                               | `/var/log/apt/history.log`                  | Audit trail        |
| **Rollback a patch**       | `dnf history undo <ID>`                     | `apt-get install <pkg>=<ver>`               | Emergency rollback |
| **Failed services check**  | `systemctl --failed`                        | `systemctl --failed`                        | Post-reboot check  |
| **Disk space check**       | `df -hT`                                    | `df -hT`                                    | Pre-patch always   |
| **View kernel version**    | `uname -r`                                  | `uname -r`                                  | Before and after   |
| **Ansible dry run**        | `ansible-playbook patch.yml --check --diff` | `ansible-playbook patch.yml --check --diff` | Always before prod |

---

## 🏆 BEST PRACTICE TIP

* **GOLDEN RULE:** Always follow `DEV → QA → PROD` for every patch — no exceptions.
* **Every patch action must have a Change Request** — never patch production ad-hoc.
* **Document everything:** before, during, and after every patching cycle.
* **Validate services are healthy within 30 minutes** of every patching window.
* **Build relationships with application teams** — they are your partners, not blockers.
* **Stay current on CVEs** via NVD (`nvd.nist.gov`), Red Hat Security Advisories, and Ubuntu USNs.

> **🔑 Quick Reference Principle:** Treat patching as a controlled lifecycle: **Assess → Plan → Test → Approve → Deploy → Validate → Document**.
