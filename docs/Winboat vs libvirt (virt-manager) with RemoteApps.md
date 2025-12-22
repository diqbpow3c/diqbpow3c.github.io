---
layout: default
title: Winboat vs RemoteApps + virt-manager
nav_order: 3
---

# Winboat vs RemoteApps + virt-manager
{: .no_toc }


{: .fs-6 .fw-300 }

# Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---


---

# Running Windows Apps on Linux: **Winboat vs RemoteApps + virt-manager**

Running Windows applications on a Linux desktop has always been a balancing act between **integration, performance, and control**. In recent years, two very different approaches have emerged:

* **Winboat** – a convenience-first, docker based solution
* **Windows RemoteApps and virt-manager** – a VM-based, isolation-first solution

This article compares the two approaches , then walks through **how to properly set up Windows RemoteApps using virt-manager**.

**Summary**: Use Winboat if you want to easily run Windows apps, and don't need high reliability. Expect some bugs and management issues, since it uses docker compose and kvm, making management of the VMs somewhat unintuitive. Use virt-manager(libvirt) and RemoteApps if you want reliability, more control over the VM, and complete customization capabilities.


## 1. Conceptual Difference

### Winboat

* Runs like this: Winboat->docker->kvm->Windows VM->Windows Apps
* Focuses on **ease of use**

### RemoteApps + virt-manager (libvirt)

* Runs a **real Windows VM**
* Windows apps are rendered **remotely** on Linux
* Focuses on **correctness, compatibility, and isolation**



## 2. How to choose

**Winboat**

**Pros**: 
- Automation: It handles the Windows installation, networking, and RDP configuration for you.

- Simplicity: You don't need to know CLI commands; it’s a "point and click" interface.

- Integration: It provides a neat UI to manage apps and even supports UWP (Microsoft Store) apps.

**Cons**: 
- Overhead: Running Electron + Docker + KVM can be resource-heavy.

- Beta Status: As of late 2025, it's still in beta; you might encounter bugs or performance issues.

**Manual RemoteApps + virt-manager (The "Power User" Way)**

**Pros**: 
- Efficiency: No extra layers like Docker or Electron; you're closer to the "metal."

- Control: Total control over VM resources, snapshots, and hardware passthrough (like GPUs).

- Stability: Uses battle-tested Linux virtualization tools.

**Cons**: 
- Setup Time: Requires significant manual configuration on both the Linux host and Windows guest.

- Maintenance: You have to manually create the "shortcuts" to launch apps from your Linux host.

---

## 3. How to set up RemoteApps + virt-manager

- Create a Windows virtual machine. You can follow the guide from <https://sysguides.com/install-a-windows-11-virtual-machine-on-kvm> . Note that you need Pro versions of Windows (or Enterprise versions) to use the remote desktop features. 
- Install [remoteapptool](https://github.com/kimmknight/remoteapptool) in the Windows VM.
- After installation of remoteapptool, add the Windows apps you want to use.
- On Linux, use xfreerdp3 to connect to the apps. wlfreerdp3 and sdl-freerdp3 do not currently work with RemoteApps. You can use the following command as an example:

`xfreerdp3 /u:YourUser /p:YourPass --app "||Microsoft Word" /v:192.168.1.100 /sound /scale-desktop:140 /scale:140`


