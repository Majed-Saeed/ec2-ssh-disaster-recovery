# AWS EC2 SSH Disaster Recovery using EBS Volume Rescue

## 📌 Overview
Real-world incident where SSH access was lost due to misconfiguration. Recovery was done using EBS volume rescue method without data loss.

---

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
