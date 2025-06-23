---
creation_date: 2025-06-21
project_name: Active Directory Lab Setup
cert_alignment: SSCP
project_status: Completed
related_notes: "[[pfSense Firewall VM Setup]]"
tags: 
cybersecurity_area: Security Administration
---

# Active Directory Lab Setup: Deploying a Domain Controller for Centralized Management

> This project focused on the successful installation, configuration, and promotion of a Windows Server VM to a Domain Controller, establishing a new Active Directory domain for centralized identity and access management within the `LabNet`.

---

## 🚀 Overview

This lab project's primary objective was to deploy a Windows Server 2022 VM as a Domain Controller (DC), thereby establishing a functional Active Directory (AD) domain within the isolated `LabNet`. This setup forms the core of an enterprise-like identity management system. Through this exercise, I aimed to develop and showcase proficiency in:
* Planning and provisioning a Windows Server VM for a critical infrastructure role.
* Configuring static IP addressing and DNS for a Domain Controller.
* Installing and promoting the Active Directory Domain Services (AD DS) role.
* Performing initial post-promotion verification of AD DS and network functionality.
* Understanding the foundational role of Active Directory in network security and administration.

### Goals & Objectives

* [x] **Primary Goal:** Successfully install Windows Server 2022 on a new VM and promote it to a Domain Controller.
* [x] **Secondary Goal:** Establish a new Active Directory forest and domain (e.g., `echoes.local`).
* [x] **Stretch Goal:** Verify domain controller functionality, including DNS resolution and internet connectivity, post-promotion.

---

## 🏛️ Lab Architecture & Design

This section details the virtual environment setup for the Domain Controller, integrating it with the existing `LabNet` managed by `pfSense-Firewall`.

### Virtual Machine Specifications

| VM Name            | Operating System      | RAM  | vCPUs | Disk Size | IP Address (Static/DHCP) | Purpose                               |
| :----------------- | :-------------------- | :--- | :---- | :-------- | :----------------------- | :------------------------------------ |
| `pfSense-Firewall` | pfSense               | 1GB  | 1     | 16GB      | WAN: DHCP, LAN: 192.168.1.1/24 | Lab Gateway, DHCP, DNS, Firewall      |
| `Win10-Client-01`  | Windows 10 Pro        | 2GB  | 1     | 40GB      | DHCP                     | First Lab Workstation                 |
| `DC01`             | Windows Server 2022 (Desktop Experience)| 3GB  | 2     | 60GB      | Static: 192.168.1.10/24  | Active Directory Domain Controller / DNS Server |

---

## 🛠️ The Journey: Step-by-Step Implementation

This section demonstrates the practical steps undertaken, focusing on technical configuration and key decisions made during the Domain Controller deployment. 

### 1.  Windows Server ISO Acquisition
* **Task:** Obtained the necessary Windows Server 2022 installation media.
* **Details:** Downloaded the official Windows Server 2022 Evaluation ISO from Microsoft's Evaluation Center.
* **Justification:** A valid Windows Server ISO is required for deploying the operating system on the new VM.

### 2. VirtualBox VM Creation for `DC01`
* **Task:** Created the base virtual machine for the Domain Controller (`DC01`).
* **Details:**
    * Utilized VirtualBox's "Expert Mode".
    * Named the VM `DC01`.
    * Specified OS Type: `Microsoft Windows`, Version: `Windows 2022 (64-bit)`.
    * Attached the downloaded Windows Server 2022 ISO.
    * Allocated **3072 MB (3 GB)** of Base Memory (RAM) and **2 Processors**.
    * Provisioned a new virtual hard disk of **60 GB**, configured as a `Dynamically allocated` VDI.
* **Justification:** Increased RAM (3GB) and vCPUs (2) were allocated to accommodate server roles and Active Directory services within host constraints. Dynamic allocation conserves host storage.
* ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750716201667.png)

### 3. Network Adapter Configuration for `DC01`
* **Task:** Configured the network adapter to integrate `DC01` into the `LabNet`.
* **Details:**
    * Accessed the VM's `Network` settings (Adapter 1).
    * Enabled the network adapter.
    * Set "Attached to:" as **`Internal Network`**, and named it **`LabNet`**.
    * Set "Promiscuous Mode" to **`Deny`**.
    * Ensured Adapter 2, 3, and 4 were disabled.
* **Justification:** Connecting to `Internal Network` named `LabNet` ensures `DC01` is part of the isolated lab segment, allowing it to communicate with pfSense and other lab VMs. `Deny` for promiscuous mode is appropriate for a server in this role.
* ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750716354442.png)

### 4. Windows Server 2022 OS Installation
* **Task:** Performed the full installation of Windows Server 2022 within the `DC01` VM.
* **Details:**
    * Started the VM, proceeding through the Windows Server installer.
    * Selected language, time, and keyboard layout.
    * Chose **"Windows Server 2022 Standard (Desktop Experience)"** edition.
    * Selected "Custom: Install Windows only (advanced)" for installation type.
    * Selected the 60GB unallocated virtual disk, partitioned, and proceeded.
    * Allowed installation to complete through multiple reboots.
    * Set a strong password for the built-in `Administrator` account upon first boot. Logged in to the Server Manager dashboard.
* **Justification:** "Desktop Experience" provides the necessary GUI. A clean custom install is standard. Setting a strong administrator password is a basic security hygiene.

### 5. Initial Network Configuration (Static IP & DNS)
* **Task:** Configured `DC01` with a static IP address and initial DNS settings.
* **Details:**
    * Verified initial DHCP assignment from pfSense (`ipconfig /all` showed `192.168.1.x` and `192.168.1.1` gateway). Confirmed internet connectivity (`ping google.com`).
    * Opened Network Connections (`ncpa.cpl`).
    * Set Ethernet adapter's IPv4 properties to:
        * **IP address:** `192.168.1.10`
        * **Subnet mask:** `255.255.255.0`
        * **Default gateway:** `192.168.1.1` (pfSense LAN IP)
        * **Preferred DNS server:** `192.168.1.1` (pfSense LAN IP - temporary, to be updated after AD promotion)
        * **Alternate DNS server:** (Left blank).
    * Verified new static IP with `ipconfig /all`.
* **Justification:** Domain Controllers require a static IP for reliable network services. Initial DNS pointing to pfSense ensures internet access during AD installation.
* ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750716807281.png)
### 6. Active Directory Domain Services Role Installation
* **Task:** Installed the necessary AD DS role binaries on `DC01`.
* **Details:**
    * In Server Manager, navigated to "Manage" -> "Add Roles and Features".
    * Selected "Role-based or feature-based installation" for `DC01`.
    * Checked "Active Directory Domain Services" under Server Roles, adding required features.
    * Proceeded through the wizard, checking "Restart the destination server automatically if required".
    * Clicked "Install".
* **Justification:** This step stages the binaries for Active Directory, a prerequisite before promoting the server to a Domain Controller.

### 7. Promote Server to Domain Controller & Domain Creation
* **Task:** Promoted `DC01` to a Domain Controller, establishing the Active Directory domain.
* **Details:**
    * Launched the "Promote this server to a domain controller" wizard from Server Manager's notification flag.
    * Selected **"Add a new forest"**.
    * Set **Root domain name:** `mylab.local` 
    * Functional levels for Forest and Domain defaulted to **"Windows Server 2016"** (accepted as perfectly fine for lab purposes despite Server 2022 OS).
    * Confirmed DNS server & Global Catalog (GC) were checked.
    * Set a strong **Directory Services Restore Mode (DSRM) password**.
    * Accepted default DNS options, NetBIOS name, and paths.
    * Passed prerequisite checks and clicked "Install".
    * `DC01` automatically restarted after successful promotion.
* **Justification:** Promoting the server centralizes identity management. Creating a new forest establishes the root of the AD hierarchy. Functional levels dictate feature sets for compatibility. DSRM password is crucial for recovery.
* ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750717053678.png)

---

## ✅ Verification & Evidence of Understanding

This section details the rigorous verification steps performed to confirm the Domain Controller's successful deployment, operational status, and correct integration within the `LabNet`.

* **Test 1: Windows Server OS Installation & Basic VM Functionality**
    * **Method:** Confirmed successful boot to Windows Server 2022 Desktop Experience and access to Server Manager dashboard.
    * **Expected Result:** Functional Windows Server GUI accessible.
    * **Actual Result:** Logged in to Windows Server 2022 with Server Manager open.
    * **Significance:** Validates the foundational OS installation on the VM.

* **Test 2: Initial Network Connectivity (Pre-Promotion Static IP & DNS)**
    * **Method:** Executed `ipconfig /all` from Command Prompt to verify DHCP IP. Performed `ping google.com`. Configured static IP `192.168.1.10`, Subnet `255.255.255.0`, Gateway `192.168.1.1`, Preferred DNS `192.168.1.1`. Re-verified with `ipconfig /all`.
    * **Expected Result:** `DC01` obtained a `192.168.1.x` address initially, then consistently held `192.168.1.10`. Successfully pinged external hosts.
    * **Actual Result:** Initial DHCP and subsequent static IP configuration confirmed. Successful pings to `google.com`.
    * **Significance:** Ensures `DC01` has reliable network identity and internet access before critical AD roles.

* **Test 3: Active Directory Domain Services Role Installation & Promotion**
    * **Method:** Monitored the Server Manager "Add Roles and Features" wizard and the "Promote this server to a domain controller" wizard for successful completion messages. Observed automatic reboot.
    * **Expected Result:** Role installation and promotion should conclude with "succeeded" messages, followed by an automatic server restart.
    * **Actual Result:** Role installation reported success, and promotion completed as expected, leading to server reboot.
    * **Significance:** Confirms the core Active Directory components are installed and the server is designated as a Domain Controller.

* **Test 4: Post-Promotion DNS Configuration & Active Directory Tools Access**
    * **Method:** After logging into the promoted DC, checked Ethernet adapter's TCP/IPv4 DNS settings (`ncpa.cpl`). Accessed `Active Directory Users and Computers` via Server Manager -> Tools.
    * **Expected Result:** Preferred DNS should be `127.0.0.1` or `192.168.1.10`. Alternate DNS optionally `192.168.1.1`. AD Users and Computers console should open, displaying the new domain.
    * **Actual Result:** DNS settings automatically updated to `127.0.0.1` (itself). AD Users and Computers console successfully opened, showing `mylab.local` domain.
    * **Significance:** Crucial verification that DNS is correctly configured for Active Directory (pointing to itself) and that AD management tools are accessible, confirming core domain functionality.
    * ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750717343318.png)

* **Test 5: Post-Promotion Internet Connectivity**
    * **Method:** Executed `ping google.com` from `DC01`'s Command Prompt.
    * **Expected Result:** Successful ping replies.
    * **Actual Result:** Pings to `google.com` were successful.
    * **Significance:** Confirms that the Domain Controller can still resolve external DNS names and reach the internet through pfSense, vital for updates and external resource access.
    * ![](./Assets/Project%20-%20Active%20Directory%20Lab-1750717395241.png)



---

## 💡 The Experience: Lessons & Reflections

### What Went Well

* **Success 1:** The Active Directory Domain Services role installation and server promotion proceeded largely as expected after initial network configuration.
    * **Reason:** Prior careful planning of the `LabNet` and the foundational pfSense setup provided a stable network environment for the DC.

### Challenges & How I Overcame Them

* **Challenge 1:** Domain Controller Functional Levels defaulted to Windows Server 2016 despite using Windows Server 2022 ISO.
    * **Diagnosis:** The AD DS Configuration Wizard only presented functional levels up to Windows Server 2016. This is a known common behavior with certain Windows Server 2022 builds or evaluation ISOs.
    * **Solution:** Accepted the Windows Server 2016 functional level as it provides all necessary core Active Directory features for the lab environment.
    * **Lesson Learned:** Not all new OS versions immediately support their own highest functional level from the start, or in all evaluation builds. For lab purposes, lower functional levels are often perfectly adequate and don't hinder learning core concepts.

* **Challenge 2:** Ensuring proper DNS configuration post-promotion.
    * **Diagnosis:** Domain Controllers require precise DNS settings (pointing to themselves primarily) to function correctly. While largely automated, verification is key.
    * **Solution:** Explicitly verified the DNS server settings in `ncpa.cpl` to confirm `127.0.0.1` or the DC's static IP as Preferred, and pfSense as Alternate, as correctly set by the promotion wizard.
    * **Lesson Learned:** DNS is fundamental to Active Directory; always verify post-installation. The promotion wizard typically handles this correctly, but manual confirmation is essential.

### Key Takeaways

* **Technical Skill Enhancement:** Gained comprehensive hands-on experience in deploying, configuring, and promoting a Windows Server to a Domain Controller. Deepened understanding of static IP addressing, DNS roles in AD, and the initial setup of AD DS.
* **Cybersecurity Conceptual Reinforcement:** Solidified understanding of Active Directory as a central identity and access management system. Reinforced the hierarchy of AD (forest, domain, DC) and its reliance on DNS. Grasped the importance of the DSRM password for AD recovery.
* **Analytical Insight ("Aha!" Moment):** Realized the intricate dance between network configuration (static IP, DNS forwarding) and Active Directory functionality, where precise DNS setup is the lifeblood of the domain. Also, confirmed that functional levels are distinct from the OS version and may not always be the highest available.

---

## 🏆 Final Outcome & Future Scope

### Project Summary

This project successfully established a robust Active Directory domain, `mylab.local`, by deploying and promoting `DC01` (Windows Server 2022) to a Domain Controller within the existing `LabNet`. All core services, including DNS and internet connectivity, were verified, providing a foundational enterprise-like environment for identity management and security policy enforcement.

### Future Improvements

* [ ] **Objective 1:** Join `Win10-Client-01` to the `mylab.local` domain.
* [ ] **Objective 2:** Create Organizational Units (OUs) and user accounts within Active Directory.
* [ ] **Objective 3:** Implement and test Group Policies (GPOs) from `DC01` to manage settings on domain-joined clients.
* [ ] **Objective 4:** Explore more advanced AD DS features (e.g., DNS management, DHCP server role on DC if desired, FSMO roles).

---