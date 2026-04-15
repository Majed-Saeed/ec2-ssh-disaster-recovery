AWS Infrastructure Recovery: Manual EBS Volume Surgery & SSH Restoration
📌 Executive Summary

A critical misconfiguration in the sshd_config file resulted in a complete lockout from a production-level EC2 instance.
This project documents a Disaster Recovery (DR) operation performed to restore access without data loss or instance replacement, using the EBS Volume Rescue Method.

🛠 The Scenario (The Challenge)
Incident: SSH port changed to a non-standard port (2222) without updating Security Groups
Barrier: AWS Systems Manager (SSM) was not configured, eliminating fallback access
Impact: Total loss of remote access (SSH + SSM failure)
Objective: Restore access while preserving system state and data integrity
🏗 Logical Workflow (The Architecture)
Isolation: Stop the affected instance to prevent data corruption
Extraction: Detach the root EBS volume
Recovery Environment: Attach volume to a temporary rescue instance
Configuration Repair: Mount filesystem and fix SSH config
Restoration: Reattach volume and restore normal operation

## 🚨 Incident
- SSH port changed to 2222
- Security Group not updated
- SSM not working
- Result: full lockout

---

## 🔍 Root Cause Evidence

### 1. SSM Connection Failure
![SSM Failed](screenshots/ssm-connection-failed.png)

---

## 🛠 Recovery Process

### 2. Detach Root Volume from Broken Instance
![Volume Detached](screenshots/ebs-volume-detached-from-instance.png.jpeg)

---

### 3. Attach Volume to Recovery Instance (Select Instance)
![Select Instance](screenshots/select-instance-for-volume-attachment.png.jpeg)

---

### 4. Choose Device Name
![Choose Device](screenshots/choose-device-name-for-ebs-volume.png.jpeg)

---

### 5. Volume Attached Successfully
![Volume Attached](screenshots/ebs-volume-attached-to-rescue-instance.png.jpeg)

---

### 6. Fix SSH Configuration (Mounted Volume)
![Fix SSH](screenshots/fix-sshd-config-on-mounted-volume.png)

---

### 7. Execute Recovery Commands
![Commands](screenshots/ebs-recovery-commands-terminal.png.png)

---

## ⚙️ Commands Used

```bash
lsblk
sudo mkdir /mnt/recovery
sudo mount /dev/nvme1n1p1 /mnt/recovery
sudo nano /mnt/recovery/etc/ssh/sshd_config
sudo umount /mnt/recovery
