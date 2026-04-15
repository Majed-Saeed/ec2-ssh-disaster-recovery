# AWS EC2 SSH Recovery (EBS Volume Rescue Method)

## 📌 Overview

This project demonstrates a real-world incident where SSH access to an AWS EC2 instance was completely lost due to a configuration mistake. The recovery was performed using an EBS volume rescue method without any data loss.

---

## 🚨 Problem

* SSH stopped working after modifying configuration
* SSH port was changed from 22 to 2222
* Security Group was not updated
* EC2 Instance Connect failed
* SSM Session Manager was unavailable

The instance was running but completely inaccessible.

---

## 🎯 Objective

Restore access to the EC2 instance without losing data or rebuilding the server.

---

## 🧠 Solution

A recovery process was used by accessing the instance disk externally:

1. Stop the EC2 instance
2. Detach the root EBS volume
3. Attach the volume to another working EC2 instance
4. Mount the volume
5. Fix SSH configuration
6. Detach and reattach the volume to the original instance
7. Start the instance

---

## ⚙️ Commands Used

```bash
lsblk
sudo mkdir /mnt/recovery
sudo mount /dev/nvme1n1p1 /mnt/recovery
sudo nano /mnt/recovery/etc/ssh/sshd_config
sudo umount /mnt/recovery
```

---

## 📸 Evidence

### SSM Connection Failure

![SSM Failed](screenshots/ssm-connection-failed.png)

---

### EBS Volume Detached

![Volume Detached](screenshots/ebs-volume-detached-from-instance.png)

---

### Volume Attached to Recovery Instance

![Volume Attached](screenshots/ebs-volume-attached-to-rescue-instance.png)

---

### Fixing SSH Configuration

![Fix SSH](screenshots/fix-sshd-config-on-mounted-volume.png)

---

### Commands Execution

![Commands](screenshots/ebs-recovery-commands-terminal.png)

---

## 🔍 Root Cause

* SSH port changed without updating Security Group
* No backup access method configured (SSM not enabled)
* Direct modification of critical system configuration

---

## ✅ Result

* SSH access restored successfully
* No data loss
* Instance fully operational

---

## 📚 Key Lessons

* Always configure SSM as backup access
* Take snapshot before modifying system configs
* Keep Security Groups aligned with service changes
* Avoid risky changes without rollback plan

---

## 🚀 Skills Demonstrated

* AWS EC2 management
* EBS volume handling
* Linux troubleshooting
* SSH recovery
* Incident resolution
