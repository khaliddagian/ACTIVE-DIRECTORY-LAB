# Active Directory Home Lab

This project documents a Windows Server Active Directory home lab I built to better understand how domains, DNS, permissions, and Group Policy actually behave in a real environment. The main goal was not just to “set things up,” but to break things on purpose, troubleshoot them, and understand why Windows behaves the way it does.

This lab simulates common helpdesk and junior system administrator scenarios such as users losing access to shared drives, permissions not behaving as expected, and Group Policy changes not taking effect immediately.

---

## Lab Environment

- **Host OS:** Windows 11 Pro  
- **Hypervisor:** VMware Workstation Pro  
- **Server:** Windows Server 2022 (Domain Controller)  
- **Client:** Windows 11 Pro (Domain-joined workstation)  
- **Network:** VMware NAT (VMnet8)

The Domain Controller hosts both **Active Directory Domain Services and DNS**.  
The client machine points its DNS directly to the Domain Controller to ensure proper domain name resolution and Group Policy processing.

---

## What I Built

- An Active Directory domain (`domain.local`)
- A Domain Controller (`LAB-DC01`) with AD DS and DNS
- A Windows 11 client (`LAB-CL01`) joined to the domain
- Organizational Units to separate users and computers
- Security groups to control access instead of assigning permissions directly to users
- A file share secured using **Share permissions + NTFS permissions**
- A mapped network drive using **Group Policy Preferences**

---

## Access Control Design (Important Part)

Access to the shared folder is controlled using a security group:

- **SG-File-Access**

Users gain or lose access by being added to or removed from this group.  
This group is used in **two places**:
1. NTFS permissions (actual access to the files)
2. Group Policy drive mapping (whether the drive appears)

This mirrors how access is handled in real enterprise environments.

---

## Group Policy Drive Mapping (What I Learned)

A network drive (Z:) is mapped using Group Policy Preferences.

During testing, I ran into a very common real-world issue:

- Removing a user from the security group **removed access**
- But the mapped drive **still appeared**

This helped me understand an important concept:

> NTFS permissions control access,  
> but Group Policy controls whether the drive exists.

A mapped drive does not automatically disappear unless Windows is explicitly told to remove it.

To fix this correctly, I implemented:
- A **Create** drive mapping (when the user is in the group)
- A **Delete** drive mapping (when the user is NOT in the group)
- Item-level targeting based on group membership
- Domain-level GPO linking so cleanup always runs

After a logoff/reboot, the drive is removed cleanly.

---

## Troubleshooting & Validation

Throughout the lab, I validated behavior using:
- `gpupdate /force`
- `gpresult /r`
- DNS resolution tests
- Logging in and out as different users
- Confirming access allowed vs access denied scenarios

This helped reinforce how Group Policy timing, scope, and targeting actually work.

---

## Why This Lab Matters

It helped me build confidence working with:
- Active Directory
- DNS
- Group Policy
- NTFS permissions
- Real-world troubleshooting instead of just following guides
