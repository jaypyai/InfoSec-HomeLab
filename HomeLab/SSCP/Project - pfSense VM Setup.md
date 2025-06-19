---
creation_date: 2025-06-18
project_name: pfSense Firewall VM Setup
cert_alignment: SSCP
project_status: Completed
cybersecurity_area: Security Administration
---

# pfSense Firewall VM Setup: Home  Lab Network Gateway

> This project successfully established a virtual firewall using pfSense, configuring its network interfaces to create an isolated lab network and connect it to the internet.

---

## 🚀 Overview

This lab project's primary objective was to deploy and configure the pfSense firewall as the central network gateway for my isolated virtual cybersecurity lab. Through this hands-on exercise, I aimed to develop and showcase proficiency in:
* Virtual Machine creation, configuration, and troubleshooting in VirtualBox on a Linux host.
* Understanding and implementing crucial virtual networking concepts (Bridged, Internal Network, Promiscuous Mode).
* Performing a bare-metal operating system installation (pfSense) and initial console configuration.
* Diagnosing and resolving complex kernel driver and network interface issues.

### Goals & Objectives

* [x] **Primary Goal:** Successfully create and configure the pfSense virtual machine to act as the lab's internet gateway and internal network router.
* [x] **Secondary Goal:** Ensure the pfSense VM can obtain an IP from the home network (WAN) and provide DHCP services to an isolated internal lab network (LAN).
* [x] **Stretch Goal:** Resolve any unforeseen virtualization or network driver issues on the host system to ensure proper VM functionality.

---

## 🏛️ Lab Architecture & Design

This section details the virtual environment setup for this project.

### Virtual Machine Specifications

| VM Name            | Operating System      | RAM  | vCPUs | Disk Size | IP Address (Static/DHCP) | Purpose                               |
| :----------------- | :-------------------- | :--- | :---- | :-------- | :----------------------- | :------------------------------------ |
| `pfSense-Firewall` | pfSense (FreeBSD 64-bit) | 1GB  | 1     | 16GB      | WAN: DHCP (192.168.50.117/24), LAN: 192.168.1.1/24 | Lab Gateway, DHCP, DNS, Firewall      |

---

## 🛠️ The Journey: Step-by-Step Implementation

This section includes the steps taken, focusing on technical configuration and key decisions.

### 1. pfSense ISO Download & Decompression
* **Task:** Obtained the pfSense installation image.
* **Details:** Downloaded `pfSense-CE-RELEASE-amd64.iso.gz` from the official pfSense website. Decompressed the `.gz` file using `gunzip` on the Arch Linux host to obtain the usable `.iso` file.
* **Justification:** The `.gz` format is compressed and cannot be used directly by VirtualBox; decompression is a necessary prerequisite.

### 2. VirtualBox VM Creation (Expert Mode)
* **Task:** Created the base `pfSense-Firewall` virtual machine.
* **Details:**
    * Used VirtualBox's **Expert Mode** for precise control.
    * Named the VM: `pfSense-Firewall`.
    * OS Type: `BSD`, Version: `FreeBSD (64-bit)`.
    * Attached the decompressed pfSense ISO image.
    * Allocated `1024 MB` (1 GB) of Base Memory (RAM) and `1` Processor.
    * Created a new virtual hard disk: `16 GB` (Dynamically Allocated VDI).
* **Justification:** These specifications are minimal yet sufficient for pfSense in a virtualized lab environment, conserving the host's 16GB RAM. Expert Mode allowed for more setup options upfront.

### 3. Initial Network Adapter Configuration
* **Task:** Configured the two essential network adapters for the `pfSense-Firewall` VM.
* **Details:**
    * **Adapter 1 (WAN Interface - Initial attempt):** Attached to `NAT`. Promiscuous Mode: Attempted `Allow All`.
    * **Adapter 2 (LAN Interface):** Attached to `Internal Network`, named `LabNet`. Promiscuous Mode: Attempted `Allow All`.
* **Justification:** `NAT` provides initial internet access for pfSense; `Internal Network` creates the isolated segment for the lab. `Allow All` Promiscuous Mode is crucial for a firewall's routing capability.

### 4. pfSense OS Installation Start
* **Task:** Start the pfSense OS installation process within the VM.
* **Details:**
    * Started the VM, selected "Install" from the boot menu.
    * Accepted license, chose default keymap.
    * Selected `Auto (ZFS)` for file system and partitioning (as recommended by installer).
    * Selected `Stripe - No redundancy` for ZFS virtual device type (due to single disk).
    * Confirmed disk erase (`Yes` to destroy `ada0`).
* **Justification:** Following the installer's recommended options ensures a stable and modern pfSense base. Assigning interfaces early ensures consistency with VirtualBox setup.

---

## ✅ Verification & Evidence of Understanding

This section details how the successful deployment and configuration of the pfSense firewall were confirmed.

* **Test 1: VirtualBox VM & Network Adapter Configuration**
    * **Method:** Visually confirmed VM presence, RAM/CPU/Disk settings, and reviewed Network Adapter 1 (Bridged to Ethernet, `Allow All` Promiscuous Mode) and Adapter 2 (Internal Network `LabNet`, `Allow All` Promiscuous Mode) in VirtualBox settings.
    * **Expected Result:** All settings match the planned configuration.
    * **Actual Result:** All settings were correctly applied after troubleshooting.
    * **Significance:** Verifies the underlying hypervisor setup is correct for the firewall's role.

* **Test 2: pfSense OS Boot and Initial IP Configuration**
    * **Method:** After installation and reboot, observed the pfSense console menu.
    * **Expected Result:** Console displays correct WAN and LAN IP addresses.
    * **Actual Result:**
        * WAN: `192.168.50.117/24` (DHCP from home router).
        * LAN: `192.168.1.1/24` (default pfSense LAN IP).
    * **Significance:** Confirms successful OS installation, correct interface assignment within pfSense, and proper IP address acquisition.

* **Test 3: pfSense Internet Connectivity**
    * **Method:** From the pfSense console menu, selected Option 7 ("Ping host") and pinged `google.com`.
    * **Expected Result:** Successful ping replies to `google.com`.
    * **Actual Result:** Pings were successful.
    * **Significance:** Verifies that pfSense's WAN interface is correctly routing traffic to the internet, confirming the external connectivity needed for the lab.


---

## 💡 The Experience: Lessons & Reflections

### What Went Well

* **Success 1:** The core conceptual design of using pfSense as a central router with internal and external interfaces was successful.
    * **Reason:** Careful pre-planning of network segments (Bridged for WAN, Internal for LAN) paid off, leading to correct routing when the underlying issues were resolved.

### Challenges & How I Overcame Them

* **Challenge 1: `Kernel driver not installed (rc=-1908)` error & Promiscuous Mode Greyed Out.**
    * **Diagnosis:** VirtualBox was unable to load its kernel modules. `lsmod` showed `vboxdrv` was not loaded, and `modprobe vboxdrv` returned "module not found." The "Promiscuous Mode" option was greyed out in VirtualBox settings. Initially suspected wireless card limitations and host VPN.
    * **Solution:**
        1.  Confirmed `Secure Boot` was disabled.
        2.  Identified that the Realtek RTL8822CE **wireless network card** was the most probable cause of the promiscuous mode limitation on Linux.
        3.  Connected the laptop via its **built-in Ethernet port**.
        4.  Changed VirtualBox Adapter 1 (WAN) from `NAT` to `Bridged Adapter` and selected the physical Ethernet interface.
        5.  This enabled Promiscuous Mode to be set to `Allow All` for both Adapter 1 and Adapter 2.
        6.  Forced reinstallation of `virtualbox-host-dkms` using `paru -S virtualbox-host-dkms`.
        7.  Manually loaded modules: `sudo modprobe vboxdrv`, `sudo modprobe vboxnetadp`, `sudo modprobe vboxnetflt`. This made `vboxdrv` show `2 vboxnetadp,vboxnetflt` in `lsmod`.
        8.  Rebooted the system.
    * **Lesson Learned:** Troubleshooting requires systematic elimination. Hardware/driver limitations (especially with wireless cards) can prevent critical virtualization features like promiscuous mode. Ethernet is far more reliable for lab environments. Understanding how `dkms` and `modprobe` interact with kernel modules is vital for Linux-based virtualization.

* **Challenge 2: pfSense Installer Flow Discrepancies.**
    * **Diagnosis:** The pfSense installer prompted for interface assignments (`em0`/`em1`) and network mode adjustments (WAN/LAN) at unexpected points *during* the installation, rather than exclusively post-reboot.
    * **Solution:** Followed the principle that VirtualBox Adapter 1 (WAN) maps to the first detected interface (`em0`) and Adapter 2 (LAN) maps to the second (`em1`), and kept default DHCP for WAN and 192.168.1.1 for LAN.
    * **Lesson Learned:** Installer flows can vary; understanding the underlying network design is more important than memorizing exact GUI sequences. Stick to core principles.

* **Challenge 3: VM Freezing on Reboot after Installation.**
    * **Diagnosis:** After force-ejecting the ISO, the VM console froze at the reboot prompt.
    * **Solution:** Force powered off the VM from VirtualBox Manager, then started it again.
    * **Lesson Learned:** Forcing power off is a safe troubleshooting step for unresponsive VMs in a non-critical lab environment, especially after OS installation is complete.

### Key Takeaways

* **Technical Skill:** Deepened expertise in VirtualBox configuration (network modes, promiscuous mode, adapter types). Mastered troubleshooting `dkms` and kernel module loading on Arch Linux. Gained hands-on experience with pfSense OS installation and initial console configuration.
* **Cybersecurity Concept:** Solidified understanding of the firewall's role as a network gateway, DHCP server, and the importance of network segmentation (isolated `Internal Network`) for a secure lab. Reinforced the concept of network address translation (NAT/Bridged) and its implications.
* **"Aha!" Moment:** The direct correlation between specific wireless card chipsets/drivers and the limitation of promiscuous mode, highlighting a real-world hardware constraint in virtualization environments. The importance of the "used by" count in `lsmod` for module dependencies.

---

## 🏆 Final Outcome & Future Scope

### Project Summary

This project successfully established the foundational virtual network environment for my cybersecurity lab, centered around a fully operational pfSense firewall. All critical components, including correct VM setup, network interface configuration, successful OS installation, and confirmed internet connectivity, have been verified. This provides a robust, isolated, and realistic network for future learning.

### Future Improvements

* [ ] **Idea 1:** Proceed with the full Windows 10/11 operating system installation within the newly created `Win10-Client-01` VM.
* [ ] **Idea 2:** Verify DHCP lease acquisition and internet connectivity from within the installed Windows client.
* [ ] **Idea 3:** Explore basic pfSense web interface configuration (e.g., changing admin password, setting up DNS resolver).

### Related Notes

* [[Project - Windows 10 Client]] (The next project!)

---