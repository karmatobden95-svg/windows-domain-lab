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
