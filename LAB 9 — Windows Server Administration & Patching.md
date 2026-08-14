# LAB 9 — Windows Server Administration & Patching

**WSUS | PowerShell | Windows Update | Group Policy**

> **🎯 Objective:** Windows Server patching is managed via WSUS, Windows Update, and PowerShell. This lab covers manual and automated patching procedures for Windows environments.

---

## TASK 9.1 — Windows Update via PowerShell

### 1. Open PowerShell as Administrator

Open **PowerShell as Administrator** on the Windows Server.

### 2. Install the `PSWindowsUpdate` Module

> **ℹ️ Note:** This is a one-time setup.

```powershell
Install-Module PSWindowsUpdate -Force
Import-Module PSWindowsUpdate
```

### 3. Check Available Updates

```powershell
Get-WindowsUpdate
```

### 4. List Security Updates Only

```powershell
Get-WindowsUpdate -Category 'Security Updates'
```

### 5. Install All Updates Automatically

```powershell
Install-WindowsUpdate -AcceptAll -AutoReboot
```

### 6. Install Security Updates Only

```powershell
Install-WindowsUpdate -Category 'Security Updates' -AcceptAll
```

### 7. Check Windows Update History

```powershell
Get-WUHistory | Select-Object -First 20 | Format-Table Date, Title, Result
```

### 8. Check if a Reboot Is Pending

```powershell
Get-WURebootStatus
```

---

## TASK 9.2 — WSUS (Windows Server Update Services) Setup

### 1. Install the WSUS Role on the WSUS Server

```powershell
Install-WindowsFeature -Name UpdateServices -IncludeManagementTools
```

### 2. Run the WSUS Post-Installation Wizard

```powershell
& 'C:\Program Files\Update Services\Tools\WsusUtil.exe' postinstall
SQL_INSTANCE_NAME=. CONTENT_DIR=C:\WSUS
```

### 3. Open the WSUS Console

Navigate to:

**Server Manager → Tools → Windows Server Update Services**

### 4. Synchronize WSUS with Microsoft Update

Synchronize WSUS with **Microsoft Update**.

> **⏳ Note:** The initial synchronization may take hours.

### 5. Configure Automatic Approvals

Configure automatic approvals for the following update classifications:

* **Critical**
* **Security**

### 6. Create Computer Groups

Create the following computer groups and assign servers accordingly:

* **DEV**
* **QA**
* **PROD**

### 7. Configure Target Servers to Use WSUS via Group Policy

Navigate to:

**Computer Configuration → Administrative Templates → Windows Components → Windows Update → Specify intranet Microsoft update service location**

Set your WSUS URL.

### 8. Force Group Policy Update on Client Servers

```powershell
gpupdate /force
wuauclt /detectnow
```

---

## TASK 9.3 — Windows Patching Health Checks

### 1. Check Windows Service Status After Patching

```powershell
Get-Service | Where-Object {$_.Status -eq 'Stopped'}
```

### 2. Check Windows Event Log for Errors Post-Patch

```powershell
Get-EventLog -LogName System -EntryType Error -Newest 20
```

### 3. Check for Pending Reboots After Patching

```powershell
Test-Path 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired'
```

### 4. View Installed Hotfixes/KBs

```powershell
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 10
```

### 5. Check Disk Space on All Drives Before/After Patching

```powershell
Get-PSDrive -PSProvider FileSystem | Select-Object Name, Used, Free
```

### 6. Remotely Check Patch Status of Multiple Servers

```powershell
Invoke-Command -ComputerName web01,web02 -ScriptBlock {Get-HotFix |
Select-Object -Last 5}
```

---

## 🏆 BEST PRACTICE TIP

* **Always test updates on a WSUS “Pilot” group** (2–3 machines) before approving updates for the full fleet.
* **Decline superseded updates in WSUS regularly** to keep the console manageable.
* **Use PowerShell Remoting (WinRM)** to patch and check multiple servers simultaneously.
* **Schedule Windows reboots** using `Restart-Computer -ComputerName web01 -Force` during maintenance windows.
* **Export the Windows Event Log to a central SIEM before patching** for post-patch comparison.
* **Disable Windows Update automatic restarts during business hours via Group Policy.**

> **⚠️ Operational Reminder:** Validate Windows updates in the WSUS Pilot group before broad deployment, and ensure production updates and reboots occur within an approved maintenance window.
