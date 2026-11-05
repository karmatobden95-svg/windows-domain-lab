# windows-domain-lab

A home lab where I build a small company network using Windows Server 2025 and Active Directory. I'll be breaking it on purpose, then fix it, and write up each fix it.

## Setup

| Machine | System | Job |
|---|---|---|
| DC01 | Windows Server 2025 | Domain controller (Active Directory, DNS, DHCP) |
| FS01 | Windows Server 2025 | File server |
| CL01 | Windows 11 Enterprise | Staff computer |
| CL02 | Windows 11 Enterprise | Second staff computer |

All four machines run in VirtualBox on a private network (kt-lab.local, 192.168.10.0/24). Nothing inside the lab can reach my home network.

Host PC: Windows 11 Home, 32 GB RAM.

## Log

### Step 1: Checking the host PC

What I checked:
- `systeminfo` showed Windows 11 Home and 32 GB RAM. Under Hyper-V Requirements it said "A hypervisor has been detected".
- `msinfo32` showed Virtualization-based security running one service: Hypervisor enforced Code Integrity.
- Windows Features: Virtual Machine Platform, Windows Hypervisor Platform and Windows Subsystem for Linux were all turned off.

Note to self: This means that Windows 11 Home does not include Hyper-V, so something else was using the CPU's virtualization feature. The cause is Memory integrity (Windows Security > Core Isolation). While it is on, VirtualBox has to run in a slower mode.

Decision: 
Leave Memory integrity on for now and test how the first VM runs. If it is too slow or will not start, turn Memory integrity off.

### Step 2: Getting Windows Server

- Downloaded the Windows Server 2025 evaluation (180-day trial) from the Microsoft Evaluation Center.
- File: 26100.32230.260111-0550.lt_release_svc_refresh_SERVER_EVAL_x64FRE_en-us.iso (about 7 GB).

### Step 3: Installing Windows Server
- First install froze at 47% for over 20 minutes in VirtualBox's slow mode (green turtle).
- Powered off the VM, deleted the leftover partitions, and installed again. Second attempt finished.
- Note to self: try the simple fix (a clean retry) before changing security settings.

### Step 4: Basic setup
- Installed VirtualBox Guest Additions so the screen resizes and the mouse works smoothly.
- Renamed the server to DC01 and set the time zone to Brisbane.
- Took a snapshot (01-fresh-install) as a restore point.

### Step 5: Activation
- Checked licensing with `slmgr /dlv`: Licensed, 180 days, 1 rearm left.

### Step 6: Lab network
- Moved DC01 from NAT to a VirtualBox Internal Network (LAB-NET). Had to switch VirtualBox settings from Basic to Expert mode to see the option.
- Set a static IP: 192.168.10.10/24, no gateway, DNS 192.168.10.10 then 127.0.0.1.
- First check failed: `ipconfig /all` still showed a NAT address (10.0.2.15) because the static IP hadn't saved. Redid it and confirmed.
- `ping 8.8.8.8` returns "General failure": no route out, so the lab is isolated.

### Step 7: Promoted DC01 to a domain controller
- New forest kt-lab.local (NetBIOS KT-LAB), DNS installed with it.
- Verified with `whoami` (kt-lab\administrator) and `Get-ADDomain`.
- `dcdiag`: core tests passed. DFSREvent failed due to warnings logged during promotion; investigating.

- ### Step 8: Checked the dcdiag failures in Event Viewer
- System log: error 16642 at 3:24:27 PM, then 16648 ("account-identifier pool completed successfully") at 3:24:28 PM. The error fixed itself one second later.
- DFS Replication log: event 4602 at 3:24:29 PM, so SYSVOL was set up successfully.
- Conclusion: domain healthy. Both dcdiag failures came from startup events and the lab having no internet, not a current fault.
- Snapshot: 03-dc-promoted.
