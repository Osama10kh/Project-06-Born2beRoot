> *This repository contains the Born2beRoot project, developed as part of the 42 curriculum by **o.khader**.*

---

# 🚀 Born2beRoot

## 📌 Overview

**Born2beRoot** is a system administration project centered around setting up a minimal, hardened **Debian** server inside a VirtualBox virtual machine. Operating strictly without a graphical interface, the goal is to implement robust security practices including an encrypted LVM disk structure, hardened SSH access, active firewall rules, strict password aging and complexity, detailed `sudo` accounting, and automated background monitoring.

---

## ⚙️ Architecture & Configurations

| Parameter | Configuration Detail |
| :--- | :--- |
| **Operating System** | **Debian 13 (Trixie)** stable branch, headless (no GUI). |
| **Storage & LVM** | Encrypted LVM on `/dev/sda5` with logical volumes for `/` (root), `swap`, and `/home`. Unencrypted `/boot` partition for bootloader execution. |
| **SSH Security** | OpenSSH on custom port **`4242`**. Direct `root` login disabled. |
| **Firewall** | **UFW** enabled at boot. Inbound traffic blocked except for port **`4242`**. |
| **Identity & Access** | Primary user: **`o.khader`** (member of `sudo` and `user42`). Hostname: **`o.khader42`**. |
| **Services Running** | `ssh`, `ufw`, `cron`, `apparmor`, and `sudo`. |

### 🔐 Password Policy Settings
- **Expiration Controls** (`/etc/login.defs` & `chage`): Max age of **30 days**, min **2 days** between changes, warning period of **7 days**.
- **Quality Requirements** (`libpam-pwquality`):
  - Minimum length: **10 characters**
  - Must include: **1 uppercase**, **1 lowercase**, **1 digit**
  - Max repeated characters: **3**
  - Username prohibited inside password
  - Difference requirement: At least **7 characters** changed from previous password

### 🛡️ Sudo Configuration (`/etc/sudoers.d/sudo_config`)
- Maximum **3 password attempts** before termination.
- Custom authentication failure warning message.
- Full input/output session logging stored in `/var/log/sudo/`.
- Mandatory TTY execution (`Defaults requiretty`).
- Custom restricted `secure_path` defined.

### 📊 Automated Monitoring
A custom script (`monitoring.sh`) runs via `cron` at system boot and every **10 minutes** thereafter, using the `wall` command to broadcast live server metrics across all active open terminals.

---

## 🔍 Technical Comparisons

<details>
<summary><b>1. Debian vs. Rocky Linux</b></summary>

* **Debian:** Feature-rich package repository, huge community base, straightforward setup for beginners, uses `apt`/`dpkg`, default AppArmor security.
* **Rocky Linux:** Red Hat Enterprise Linux (RHEL) downstream alternative, uses `dnf`/`rpm`, defaults to SELinux & firewalld, complex setup targeted at corporate environments.
* **Selection:** **Debian** was selected as the optimal platform for learning foundational Linux administration.
</details>

<details>
<summary><b>2. AppArmor vs. SELinux</b></summary>

* **AppArmor:** Uses **path-based** profile restrictions. Simple, human-readable rules focused on application containment.
* **SELinux:** Uses **label-based** security contexts applied to every system object and process. Provides granular, high-level control at the cost of operational complexity.
</details>

<details>
<summary><b>3. UFW vs. firewalld</b></summary>

* **UFW (Uncomplicated Firewall):** A streamlined interface for `iptables`/`nftables` common across Debian and Ubuntu setups.
* **firewalld:** Dynamic firewall engine using network zones and service definitions, allowing live configuration updates without restarting services (standard on RHEL distros).
</details>

<details>
<summary><b>4. VirtualBox vs. UTM</b></summary>

* **VirtualBox:** Cross-platform hypervisor for x86 systems (Windows, Linux, Intel macOS) utilizing native `.vdi` disk files.
* **UTM:** Front-end GUI for QEMU built for Apple Silicon (ARM) Macs, supporting emulation and native virtualization with `.qcow2` images.
</details>

---

## 🛠️ Step-by-Step Installation Guide

1. **Create Virtual Machine**  
   Set up a new VM in VirtualBox (*Linux / Debian 64-bit*, `1024 MB` RAM, `30 GB` dynamic VDI disk), skipping automated installation scripts.
2. **Configure Port Forwarding**  
   Add a NAT rule: map **Host Port `4242`** to **Guest Port `4242`**.
3. **Install Base OS**  
   Install Debian using guided **Encrypted LVM** with a dedicated `/home` volume. *Deselect all desktop environments.*
4. **Apply System Hardening**  
   Set up user permissions, network configurations, UFW firewall rules, SSH daemon settings, PAM password rules, and sudo policies.
5. **Deploy System Monitor**  
   Place `monitoring.sh` in `/usr/local/bin/` and configure root's `crontab`:
   ```bash
   @reboot /usr/local/bin/monitoring.sh
   */10 * * * * /usr/local/bin/monitoring.sh
   ```
6. **Verify Remote Access**  
   Test the hardened SSH connection from your host terminal:
   ```bash
   ssh o.khader@127.0.0.1 -p 4242
   ```
7. **Calculate Disk Checksum**  
   Power down the VM and generate the SHA-1 hash of the main storage file to save into `signature.txt`:
   ```cmd
   certUtil -hashfile born2beroot.vdi sha1
   ```

---

## 📚 References & Resources

- 🌐 [Debian Official Documentation](https://www.debian.org/doc/)
- 📖 System Manuals: `man sudoers`, `man sshd_config`, `man pwquality.conf`, `man login.defs`, `man chage`, `man crontab`
- 🛡️ Security Wiki Pages: [Debian AppArmor Wiki](https://wiki.debian.org/AppArmor) \| [Debian UFW Guide](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29)
- 🖥️ [VirtualBox User Manual](https://www.virtualbox.org/manual/)

---

### 🤖 AI Usage Disclosure

*AI (Claude) was utilized as an instructional guide during setup to clarify configuration concepts (such as LVM encryption mechanics, SSH hardening, PAM rules, and sudo directives) and assist in troubleshooting setup issues (network port mapping, disk partitioning). It also assisted in structuring `monitoring.sh` and organizing this document. All configurations were personally written, tested, and verified on the target machine by the author to ensure full understanding prior to evaluation.*