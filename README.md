# AWS Infrastructure Recovery: Restoring SSH Access via EBS Volume Rescue

# Recovering a Locked-Out EC2 Instance with the EBS Rescue Method

![AWS](https://img.shields.io/badge/AWS-EC2-FF9900?logo=amazonaws&logoColor=white)
![EBS](https://img.shields.io/badge/Storage-EBS-232F3E?logo=amazonaws&logoColor=white)
![Linux](https://img.shields.io/badge/OS-Linux-FCC624?logo=linux&logoColor=black)
![SSH](https://img.shields.io/badge/Access-SSH-444444?logo=openssh&logoColor=white)

> One wrong line in `sshd_config` locked me out of a production server.
> This is how I got back in — without losing data or rebuilding the instance.

-----

## At a glance

|            |                                                                |
|------------|----------------------------------------------------------------|
|**Problem** |Complete loss of remote access to a production EC2 instance     |
|**Cause**   |SSH moved to port 2222, but the Security Group was never updated|
|**Fallback**|None — SSM agent wasn’t installed                               |
|**Fix**     |Repaired the config on the disk itself, from a separate instance|
|**Outcome** |Access restored, zero data loss, original instance kept intact  |

-----

## The incident

I switched the SSH port from 22 to 2222 and saved the config — but I never
opened 2222 in the Security Group. The moment the SSH service restarted, the
old session dropped and the new port was sitting behind a closed firewall.

There was no second door, either. The instance had no SSM agent, so the
usual “connect through Systems Manager and fix it live” route wasn’t an
option. The server was healthy and running the whole time. I just couldn’t
reach it.

## The idea behind the fix

If you can’t reach the operating system over the network, stop trying to.
Take the disk out, plug it into a machine you *can* reach, fix the file
there, and put the disk back. The EBS volume rescue method is exactly that.

The plan was five moves:

1. Stop the broken instance so its root volume can be detached.
1. Detach that volume.
1. Attach it to a temporary rescue instance as a second disk.
1. Mount it, fix `sshd_config`, unmount.
1. Move the volume back home and start the instance.

-----

## Walkthrough

**Confirming there was no way in.** SSM was the last fallback, and it failed
too. That ruled out any in-place fix and made the volume rescue the only path
forward.

![SSM connection failed](screenshots/ssm-connection-failed.png)

**Detaching the root volume.** With the instance stopped, I detached its root
EBS volume to move it to the rescue instance.

![Volume detached](screenshots/ebs-volume-detached-from-instance.png.jpeg)

**Picking the rescue instance.** I attached the volume to a running instance
in the same Availability Zone.

![Select instance](screenshots/select-instance-for-volume-attachment.png.jpeg)

**Assigning a device name.** The volume needs a device name that doesn’t
collide with the rescue instance’s own root disk.

![Choose device name](screenshots/choose-device-name-for-ebs-volume.png.jpeg)

**Volume attached.** It now shows up as a second disk, ready to mount.

![Volume attached](screenshots/ebs-volume-attached-to-rescue-instance.png.jpeg)

**Fixing the config.** I mounted the filesystem, opened `sshd_config` on the
mounted volume, and set the port back to 22.

![Fix sshd_config](screenshots/fix-sshd-config-on-mounted-volume.png)

**The commands, running.**

![Recovery commands](screenshots/ebs-recovery-commands-terminal.png.png)

-----

## The commands

```bash
# Find the attached volume
lsblk

# Mount its root partition
sudo mkdir /mnt/recovery
sudo mount /dev/nvme1n1p1 /mnt/recovery

# Edit the SSH config on the mounted disk — set Port back to 22
sudo nano /mnt/recovery/etc/ssh/sshd_config

# Unmount once saved
sudo umount /mnt/recovery
```

From there I detached the volume from the rescue instance, reattached it to
the original instance as its root device (`/dev/xvda`), and powered it on.
SSH came back on port 22 with everything exactly where I left it.

-----

## What this involved

- **EC2 & EBS lifecycle** — stopping instances, detaching and reattaching
  root volumes across machines
- **Linux internals** — identifying block devices, mounting a foreign
  filesystem, editing system config offline
- **SSH / `sshd` troubleshooting** — diagnosing the lockout and tracing it to
  the port/Security-Group mismatch
- **Disaster recovery under constraints** — recovering with no live access
  and no SSM fallback, while protecting the data

-----

## What I’d do differently

- Open the new port in the Security Group **before** changing it in
  `sshd_config`, and test it in a fresh session before closing the old one.
- Install the SSM agent up front. With it, this entire operation would have
  been a one-minute fix instead of a disk transplant.
- Treat the rescue method as a known, rehearsed procedure — not something to
  figure out for the first time during an outage.