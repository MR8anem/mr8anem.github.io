---
title: Windows PrivEsc
authors: 0xMr8anem
image:
    path: assets/images/WinPrivEsc/init-Enum/1_9PBLSDGO9EduEOqdWxoCag.jpg
date: 2024-07-21 20:00:00 +0800
categories: HTB
description: "Windows Privilege Escalation"
tags: [HTB , Windows , PrivEsc]
toc: true
---


# Initial Enumeration

**Interface(s), IP Address(es), DNS Information**

```bash
ipconfig /all
```

**Get ARP Table information**

```bash
arp -a
```

Get **Routing Table**

```powershell
route print
```

**Check Windows Defender Status**

```powershell
Get-MpComputerStatus
```

**List AppLocker Rules**

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

**Test AppLocker Policy**

```powershell
 Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone
```

Using the [tasklist](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist) command to look at running processes

```powershell
tasklist /svc

```

**Display All Environment Variables in cmd**

- When running a program, Windows looks for that program in the CWD (Current Working Directory) first, then from the PATH going left to right.
- it is not uncommon to find administrators (or applications) modify the `PATH`

```bash
set
```

**View Detailed Configuration Information**

- Google the KBs installed under [HotFixes](https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix) to get an idea of when the box has been patched.
- The `System Boot Time` and `OS Version` can also be checked to get an idea of the patch level.
- If the box has not been restarted in over six months, chances are it is also not being patched.

```bash
systeminfo
```

- If `systeminfo` doesn't display hotfixes, they may be queriable with [WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) using the WMI-Command binary with [QFE (Quick Fix Engineering)](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-quickfixengineering) to display patches.

```bash
wmic qfe

```

- We can do this with PowerShell as well using the [Get-Hotfix](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-hotfix?view=powershell-7.1) cmdlet.

```bash
Get-HotFix | ft -AutoSize
```

**Installed Programs**

- WMI can also be used to display installed software.
- This information can often guide us towards hard-to-find exploits. Is `FileZilla`/`Putty`/etc installed?
- Run  [LaZagne](https://github.com/AlessandroZ/LaZagne) to check if stored credentials for those applications are installed. Also, some programs may be installed and running as a service that is vulnerable.

```bash
wmic product get name
```

- We can, of course, do this with PowerShell as well using the [Get-WmiObject](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject?view=powershell-5.1) cmdlet.

```bash
Get-WmiObject -Class Win32_Product |  select Name, Version
```

**Display Listening Processes**

```bash
netstat -ano
```

**User & Group Information**

- Users are often the weakest link in an organization, especially when systems are configured and patched well

Check logged-in Users 

- It is always important to determine what users are logged into a system.
- Are they idle or active? Can we determine what they are working on?
- During an evasive engagement, we would need to tread lightly on a host with other user(s) actively working on it to avoid detection.

```bash
C:\username> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#2           1  Active          .  3/25/2021 9:27 AM
```

**Current User Privileges**

- knowing what privileges our user has can greatly help in escalating privileges

```bash
whoami /priv
```

**Current User Group Information**

```bash
whoami /groups
```

**Get All Users on the system**

```bash
net user
```

**Get All Groups**

```powershell
net localgroup
```

**Details About a Group**

- It is worth checking out the details for any non-standard groups.
- we may find a password or other interesting information stored in the group's description.

```powershell
net localgroup administrators
```

**Get Password Policy & Other Account Information**

```bash
net accounts
```

What service is listening on port 8080 (service name not the executable)?

![Untitled](Initial%20Enumeration%208de532159c4a4e178223b99b3c35a7df/Untitled.png)

- First we need to check the Process ID (PID) of process running on port 8080

```powershell
netstat -ano | findstr "8080"
```

- then we need to check the task list for the PID

```powershell
tasklist | findstr "<PID>"
```

**Get Password Policy & Other Account Information**

```bash
net accounts
```
