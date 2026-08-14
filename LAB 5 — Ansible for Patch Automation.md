# LAB 5 — Ansible for Patch Automation

**Inventory | Playbooks | Roles | Ad-hoc Commands**

> **🎯 Objective:** Ansible is the industry-standard tool for patch automation across large fleets. This lab covers setting up inventory, writing patching playbooks, and running them safely.

---

## TASK 5.1 — Ansible Setup & Inventory

### 1. Install Ansible on the Control Node

**RHEL:**

```bash
sudo dnf install ansible -y # RHEL
```

**Ubuntu:**

```bash
sudo apt install ansible -y # Ubuntu
```

### 2. Verify the Installation

```bash
ansible --version
```

### 3. Create a Host Inventory File

```bash
vi /etc/ansible/hosts
```

### 4. Add Server Groups to the Inventory

```ini
[webservers]
web01.company.com
web02.company.com

[dbservers]
db01.company.com
db02.company.com

[all:vars]
ansible_user=vinod
ansible_become=yes
ansible_become_method=sudo
```

### 5. Test Connectivity to All Hosts

```bash
ansible all -m ping
```

### 6. Run an Ad-Hoc Command to Check Uptime on All Hosts

```bash
ansible all -m command -a 'uptime'
```

---

## TASK 5.2 — Write a Patching Playbook

### 1. Create the Playbook File

```bash
vi /home/vinod/patch_servers.yml
```

### 2. Write the Following Patching Playbook

```yaml
---
- name: Monthly Security Patch Cycle
  hosts: all
  become: yes
  serial: 5 # Patch 5 servers at a time

  pre_tasks:
    - name: Pre-patch disk check
      command: df -h
      register: disk_out

    - debug: var=disk_out.stdout_lines

  tasks:
    - name: Apply security patches (RHEL/CentOS)
      dnf:
        name: '*'
        state: latest
        security: yes
        update_only: yes
      when: ansible_os_family == 'RedHat'

    - name: Apply security patches (Ubuntu)
      apt:
        upgrade: yes
        update_cache: yes
      when: ansible_os_family == 'Debian'

    - name: Check if reboot required
      stat:
        path: /var/run/reboot-required
      register: reboot_req

    - name: Reboot if required
      reboot:
        reboot_timeout: 300
      when: reboot_req.stat.exists

  post_tasks:
    - name: Verify services healthy
      command: systemctl --failed
      register: svc_out

    - debug: var=svc_out.stdout_lines
```

### 3. Validate the Playbook Syntax Before Running

```bash
ansible-playbook patch_servers.yml --syntax-check
```

### 4. Perform a Dry Run

Use **check mode** to preview changes without making modifications:

```bash
ansible-playbook patch_servers.yml --check --diff
```

### 5. Run on a Single Host First

Use a single host to validate the playbook before running it across the fleet:

```bash
ansible-playbook patch_servers.yml --limit web01.company.com
```

### 6. Run on All Hosts with Verbose Output

```bash
ansible-playbook patch_servers.yml -v
```

---

## 🏆 BEST PRACTICE TIP

* **Use `serial: 5` to patch servers in batches** — avoid taking down all servers at once.
* **Always run `--check` (dry run) before executing playbooks in production.**
* **Use Ansible Vault to encrypt sensitive data** (passwords, API keys) in playbooks.
* **Organize large environments with Ansible roles** for clean, reusable patching logic.
* **Log Ansible runs to a file:**

```bash
ansible-playbook patch.yml | tee /var/log/ansible_patch.log
```

* **Use `max_fail_percentage: 20`** to abort if more than 20% of hosts fail during patching.

> **⚠️ Operational Reminder:** Test automation against non-production systems first, use controlled rollout batches, and ensure that production execution is covered by an approved change record and rollback procedure.
