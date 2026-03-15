# Working with Amazon EBS

## Overview

I completed a comprehensive hands-on lab on Amazon Elastic Block Store (EBS), AWS's primary block storage service for EC2 instances. Through this lab, I gained practical experience creating EBS volumes, attaching them to running instances, and creating file systems to mount persistent storage. I demonstrated the ability to manage storage lifecycle operations including volume creation, attachment, mounting, and data persistence. By the end of this lab, I showcased proficiency with EBS operations essential for managing stateful applications and ensuring data accessibility in cloud infrastructure.

## Topics Covered

After completing this lab, I was able to:

- Create new EBS volumes with custom size and type configurations
- Attach EBS volumes to running EC2 instances
- Configure device names for volume attachment
- Connect to EC2 instances using EC2 Instance Connect
- Create file systems on EBS volumes using mkfs
- Mount EBS volumes to directory mount points
- Configure persistent mounting using fstab
- Verify storage capacity and utilization with df
- Write and read files on mounted EBS volumes
- Tag EBS resources for organization and identification
- Manage EBS volume state through attachment and detachment
- Verify data persistence and availability across storage operations

## Introducing the Technologies

### Amazon Elastic Block Store (EBS)

Amazon EBS provides persistent block-level storage volumes for use with EC2 instances. Each volume is automatically replicated within its Availability Zone to protect against hardware failures.

| EBS Characteristic | Benefit |
|-------------------|---------|
| **Persistent** | Data persists even when instance is stopped (unlike instance store) |
| **High Performance** | Low-latency, high-throughput storage suitable for databases |
| **Automatically Replicated** | Within Availability Zone for durability |
| **Snapshots** | Point-in-time backups stored in S3 for disaster recovery |
| **Flexible** | Can be attached/detached from instances, resized, and encrypted |

### EBS Volume Types

| Type | Performance | Use Case | Cost |
|------|-------------|----------|------|
| **General Purpose (gp2)** | Balanced | Most workloads, databases, web apps | Moderate |
| **General Purpose (gp3)** | Improved gp2 | General-purpose with custom IOPS | Moderate |
| **Provisioned IOPS (io1/io2)** | High | Databases, data warehouses | High |
| **Throughput Optimized (st1)** | Sequential I/O | Big data, data warehouses | Moderate |
| **Cold (sc1)** | Low cost | Infrequent access, archives | Low |

### Linux File System Commands

| Command | Purpose | Example |
|---------|---------|---------|
| **mkfs** | Create file system | `mkfs -t ext3 /dev/sdb` |
| **mount** | Mount volume to directory | `mount /dev/sdb /mnt/data-store` |
| **df** | Display disk space utilization | `df -h` |
| **ls** | List files in directory | `ls /mnt/data-store/` |
| **fstab** | Configure persistent mounts | `/dev/sdb /mnt/data-store ext3 defaults,noatime 1 2` |

### EBS Snapshots

EBS snapshots are point-in-time copies of EBS volumes stored in Amazon S3. They enable:

| Snapshot Feature | Benefit |
|-----------------|---------|
| **Durability** | Stored in S3 with 11 nines durability |
| **Disaster Recovery** | Restore entire volumes from snapshots |
| **Data Cloning** | Create multiple volumes from single snapshot |
| **Cross-Region Copy** | Copy snapshots across regions for multi-region architecture |
| **Incremental** | Only changed blocks stored, reducing storage costs |

## Tasks

### Task 1: Create and Tag an EBS Volume

#### Step 1.1: Identify Instance Availability Zone

I accessed the AWS Management Console and navigated to **EC2** → **Instances**.

I located the pre-created **Lab** instance and noted its **Availability Zone** (displayed as us-west-2a).

📝 **Note**: EBS volumes must be created in the same Availability Zone as the EC2 instance they will attach to. Cross-zone attachment is not possible.

#### Step 1.2: Create New EBS Volume

I navigated to **EC2** → **Elastic Block Store** → **Volumes**.

I observed the existing 8 GiB root volume attached to the Lab instance.

I clicked **Create volume** and configured:

| Setting | Value |
|---------|-------|
| Volume type | General Purpose SSD (gp2) |
| Size | 1 GiB |
| Availability Zone | us-west-2a (same as Lab instance) |

**Volume type rationale:** General Purpose SSD provides a balance of performance and cost, suitable for most workloads including databases and web applications.

#### Step 1.3: Add Descriptive Tag

In the **Tags - optional** section, I added:

| Key | Value |
|-----|-------|
| Name | My Volume |

💭 **Consider**: Tags serve multiple purposes: resource identification in the console, cost allocation via AWS Cost Explorer, automated resource management, and backup policies.

I clicked **Create volume** to provision the volume.

✅ **Task complete**: EBS volume created with status "Creating" (transitions to "Available" within seconds).

### Task 2: Attach Volume to EC2 Instance

#### Step 2.1: Select Volume and Attach

I selected **My Volume** from the Volumes list.

From the **Actions** dropdown, I selected **Attach volume**.

#### Step 2.2: Configure Attachment Parameters

In the attachment dialog, I configured:

| Setting | Value |
|---------|-------|
| Instance | Lab (from dropdown) |
| Device name | /dev/sdb |

**Device name significance:** The device name (`/dev/sdb`) identifies the volume within the instance. Additional volumes are named `/dev/sdc`, `/dev/sdd`, etc.

I clicked **Attach volume** to complete the attachment.

#### Step 2.3: Verify Attachment

The volume state changed to **In-use**, confirming successful attachment to the Lab instance.

✅ **Task complete**: EBS volume successfully attached to EC2 instance.

📝 **Note**: Attaching a volume does not automatically make it usable. Additional steps (file system creation and mounting) are required before data can be written to the volume.

### Task 3: Connect to EC2 Instance

#### Step 3.1: Access EC2 Instance Connect

I navigated to **EC2** → **Instances** and selected the **Lab** instance.

I clicked **Connect** to access connection options.

I selected the **EC2 Instance Connect** tab and clicked **Connect**.

A new browser tab opened with an interactive terminal connected to the Lab instance.

💭 **Consider**: EC2 Instance Connect provides browser-based SSH access without managing key pairs or security groups. It requires an IAM role with appropriate permissions (pre-configured on this instance).

#### Step 3.2: Verify Terminal Connection

The terminal displayed a prompt (`ec2-user@...`), confirming successful connection to the Lab instance.

✅ **Task complete**: Successfully connected to Lab instance via EC2 Instance Connect.

### Task 4: Create File System and Mount Volume

#### Step 4.1: Verify Current Storage

In the EC2 Instance Connect terminal, I checked current storage:

```bash
df -h
```

**Output displayed:**
```
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        464M     0  464M   0% /dev
tmpfs           473M     0  473M   0% /dev/shm
tmpfs           473M  464K  472M   1% /run
tmpfs           473M     0  473M   0% /sys/fs/cgroup
/dev/nvme0n1p1  8.0G  1.7G  6.4G  21% /
tmpfs            95M     0   95M   0% /run/user/0
tmpfs            95M     0   95M   0% /run/user/1000
```

The 1 GiB attached volume did not yet appear, as it had no file system.

#### Step 4.2: Create ext3 File System

I created a file system on the attached volume:

```bash
sudo mkfs -t ext3 /dev/sdb
```

**Command explanation:**
- `mkfs`: Make file system
- `-t ext3`: Type of file system (ext3 = extended file system version 3)
- `/dev/sdb`: Device path from the attachment step

The command completed with initialization messages.

⚠️ **Caution**: Creating a file system destroys existing data on the volume. Ensure you select the correct device path to avoid accidental data loss.

#### Step 4.3: Create Mount Point Directory

I created a directory where the volume would be mounted:

```bash
sudo mkdir /mnt/data-store
```

**Mount point purpose:** This directory serves as the access point for the volume's contents within the Linux file system.

#### Step 4.4: Mount the Volume

I mounted the volume to the directory:

```bash
sudo mount /dev/sdb /mnt/data-store
```

#### Step 4.5: Configure Persistent Mounting

I added an entry to `/etc/fstab` to ensure the volume automatically mounts after instance reboot:

```bash
echo "/dev/sdb   /mnt/data-store ext3 defaults,noatime 1 2" | sudo tee -a /etc/fstab
```

**fstab entry explanation:**
- `/dev/sdb`: Device to mount
- `/mnt/data-store`: Mount point directory
- `ext3`: File system type
- `defaults,noatime`: Mount options (defaults = standard options, noatime = don't update access times)
- `1`: Dump priority (0 = don't backup)
- `2`: File system check order (2 = after root)

📝 **Note**: Without this fstab entry, the volume would remain attached but unmounted after instance restart, making data inaccessible until manually mounted again.

#### Step 4.6: Verify fstab Configuration

I viewed the updated fstab file:

```bash
cat /etc/fstab
```

The output showed the new mount entry at the end of the file.

#### Step 4.7: Verify Mount and Storage

I checked storage again:

```bash
df -h
```

**Updated output included:**
```
/dev/nvme1n1    975M   60K  924M   1% /mnt/data-store
```

The 1 GiB volume now appeared as available storage at `/mnt/data-store`.

✅ **Task complete**: EBS volume formatted, mounted, and configured for persistent mounting.

#### Step 4.8: Write Test Data to Volume

I created a test file on the mounted volume:

```bash
sudo sh -c "echo some text has been written > /mnt/data-store/file.txt"
```

I verified the file contents:

```bash
cat /mnt/data-store/file.txt
```

**Output:**
```
some text has been written
```

This confirmed that the volume was fully operational and capable of storing data.

✅ **Task complete**: Successfully wrote and read data from the mounted EBS volume.



## Storage Architecture Summary

The infrastructure I created:

```
EC2 Instance (Lab)
├── Root Volume (/dev/nvme0n1p1)
│   └── 8 GiB (OS and system files)
│
└── Data Volume (/dev/sdb)
    ├── 1 GiB EBS volume (gp2)
    ├── ext3 file system
    ├── Mounted at /mnt/data-store
    └── Contains: file.txt ("some text has been written")
```

## Conclusion

I successfully demonstrated comprehensive EBS storage operations:

- ✅ Created a new 1 GiB General Purpose (gp2) EBS volume
- ✅ Tagged the volume for organization and identification
- ✅ Identified the correct Availability Zone for volume creation
- ✅ Attached the volume to the Lab EC2 instance
- ✅ Configured the correct device name (/dev/sdb) for attachment
- ✅ Connected to the EC2 instance using EC2 Instance Connect
- ✅ Created an ext3 file system on the unformatted volume
- ✅ Created a mount point directory (/mnt/data-store)
- ✅ Mounted the volume to the mount point
- ✅ Configured persistent mounting via /etc/fstab
- ✅ Verified storage capacity using df command
- ✅ Wrote test data to the mounted volume
- ✅ Verified data accessibility and integrity

This lab provided essential knowledge for building resilient, stateful applications on AWS. EBS volumes enable persistent storage for databases, application servers, and file services, ensuring data accessibility and reliability in cloud infrastructure.

## Additional Resources

- [Amazon EBS Documentation](https://docs.aws.amazon.com/ebs/)
- [EBS Volume Types](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html)
- [EBS Snapshots User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EBSSnapshots.html)
- [Attaching EBS Volumes](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-attaching-volume.html)
- [Linux File System Commands](https://linux.die.net/man/8/mkfs)
- [fstab Configuration Reference](https://linux.die.net/man/5/fstab)
- [EC2 Instance Connect](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html)
- [EBS Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-best-practices.html)
- [EBS Encryption](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-encryption.html)
- [EBS Performance Optimization](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-performance.html)
