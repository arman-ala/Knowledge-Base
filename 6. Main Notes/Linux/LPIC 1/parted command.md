# parted command

2025-10-12 05:57
Status: #DONE 
Tags: [[Linux]]

---
`parted` is a powerful disk partitioning tool in Linux, ideal for managing large disks and modern partition tables. Let's explore its capabilities and how to use it effectively.

### ⚙️ What is Parted and When to Use It?

Parted (GNU Parted) is a command-line utility for creating, resizing, deleting, and managing disk partitions. Unlike some older tools, it supports both **MBR (msdos)** and **GPT** partition tables.

You'll find `parted` particularly useful when:
- Working with disks larger than **2TB**, as MBR-partitioned disks cannot address space beyond this limit
- Needing to **resize existing partitions** without losing data (with supported filesystems)
- Preferring a tool that works in both **interactive** and **non-interactive** (scriptable) modes

### 📋 Parted Command-Line Options and Interactive Commands

Parted's functionality is accessed through command-line options for initial setup and interactive commands for partition manipulation.

#### Command-Line Options
These options control how `parted` behaves when you start it:

| **Option** | **Description** |
| :--- | :--- |
| `-h`, `--help` | Display help information. |
| `-l`, `--list` | List partition layouts for all block devices. |
| `-s`, `--script` | Run in script mode without user prompts. |
| `-i`, `--interactive` | Prompt user when necessary. |
| `-a`, `--align` | Set alignment type (`none`, `cylinder`, `minimal`, `optimal`). |
| `-v`, `--version` | Display version information. |

#### Interactive Commands
Once inside `parted`, use these commands for disk operations:

| **Command** | **Description** |
| :--- | :--- |
| `mklabel LABEL-TYPE` | Create a new partition table (e.g., `gpt` or `msdos`). |
| `mkpart PART-TYPE [FS-TYPE] START END` | Create a new partition. |
| `print` | Display the partition table. |
| `rm NUMBER` | Delete a partition. |
| `resizepart NUMBER END` | Resize a partition. |
| `set NUMBER FLAG STATE` | Change a partition's flag (e.g., `boot` on/off). |
| `quit` | Exit the program. |

### 🛠️ How to Use Parted: Common Workflows

Here are practical examples of using `parted` for disk management tasks.

#### Listing Partitions and Selecting a Disk
To begin, list all detected disks and their partitions:
```bash
sudo parted -l
```
Then, select the disk you want to work with:
```bash
sudo parted /dev/sdb
```

#### Creating a GPT Partition Table and Partitions
For disks over 2TB or for modern systems, use GPT. This process destroys existing data.
```bash
# Create a new GPT partition table
(parted) mklabel gpt

# Create a partition using percentages or specific sizes
(parted) mkpart primary ext4 1MiB 500GiB
(parted) mkpart primary linux-swap 500GiB 516GiB
```

#### Resizing a Partition
You can expand a partition if there's free space after it. **Always backup data first** and ensure the partition is unmounted.
```bash
# Expand partition 1 to use up to 600GiB
(parted) resizepart 1 600GiB
```
After resizing, you may need to resize the filesystem with tools like `resize2fs` (for ext4).

#### Deleting a Partition
Removing a partition is straightforward but irreversible:
```bash
# First, list partitions to identify the correct number
(parted) print

# Delete partition number 2
(parted) rm 2
```

### ⚠️ Important Considerations and Best Practices

- **⚠️ Data Safety First**: Partition modifications can lead to **data loss**. Always **back up critical data** before making changes
- **📝 Parted vs. fdisk**: While `parted` can create partitions, it doesn't format them with a filesystem. You must use commands like `mkfs.ext4` or `mkswap` afterwards
- **🔧 Non-Interactive Scripting**: The `-s` option allows you to script partitioning, which is useful for automation
- **🚫 Unmount Partitions**: The partition you intend to modify should not be in use or mounted

### 💡 Parted vs. Other Partitioning Tools

This table compares `parted` with other common Linux partitioning tools:

| **Feature** | **parted** | **fdisk** | **gdisk** |
| :--- | :--- | :--- | :--- |
| **GPT Support** | Yes | Limited | Yes |
| **MBR Support** | Yes | Yes | No |
| **>2TB Disk Support** | Yes | No | Yes |
| **Interactive Mode** | Yes | Yes | Yes |
| **Script Mode** | Yes | No | No |
| **Resize Partition** | Yes | No | No |

## Comprehensive Examples for `parted` Commands and Options

### Command-Line Option Examples

#### `-h`, `--help` (Display help information)
```bash
# Display basic help information
parted -h

# Display detailed help information
parted --help

# Sample output includes:
# Usage: parted [OPTION]... [DEVICE [COMMAND [PARAMETERS]...]...]
# Apply or remove partitioning changes to DEVICE.
#
#   -h, --help                     display this help and exit
#   -l, --list                     list partition layout on each device
#   -s, --script                   never prompt for user intervention
#   -i, --interactive             prompt when necessary
#   -a, --align=[none|cylinder|minimal|optimal]  alignment for new partitions
#   -v, --version                 display version information and exit
```

#### `-l`, `--list` (List partition layouts)
```bash
# List partition layouts for all block devices
sudo parted -l

# List partition layout for a specific device
sudo parted -l /dev/sda

# List partition layouts in script mode (no prompts)
sudo parted -s -l

# Sample output for a device:
# Model: ATA Samsung SSD 860 (scsi)
# Disk /dev/sda: 500GB
# Sector size (logical/physical): 512B/512B
# Partition Table: gpt
# Disk Flags: 
#
# Number  Start   End     Size    File system  Name     Flags
#  1      1049kB  538MB   537MB   fat32        EFI      boot, esp
#  2      538MB   274GB   273GB   ext4                  msftdata
#  3      274GB   500GB   226GB   ext4
```

#### `-s`, `--script` (Run in script mode)
```bash
# Create a partition table and partitions in script mode
sudo parted -s /dev/sdb mklabel gpt mkpart primary ext4 1MiB 1001MiB

# Delete a partition in script mode
sudo parted -s /dev/sdb rm 1

# Create multiple partitions in script mode
sudo parted -s /dev/sdb <<EOF
mklabel gpt
mkpart primary ext4 1MiB 1001MiB
mkpart primary linux-swap 1001MiB 2001MiB
set 1 boot on
EOF
```

#### `-i`, `--interactive` (Prompt user when necessary)
```bash
# Run parted in interactive mode (default behavior)
sudo parted -i /dev/sdb

# This is the same as just running:
sudo parted /dev/sdb

# In interactive mode, parted will prompt for confirmation before destructive operations
```

#### `-a`, `--align` (Set alignment type)
```bash
# Create partitions with optimal alignment (default)
sudo parted -a optimal /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Create partitions with minimal alignment
sudo parted -a minimal /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Create partitions with cylinder alignment (legacy)
sudo parted -a cylinder /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Create partitions with no alignment (not recommended)
sudo parted -a none /dev/sdb mkpart primary ext4 1MiB 1001MiB
```

#### `-v`, `--version` (Display version information)
```bash
# Display parted version
parted -v

# Sample output:
# parted (GNU parted) 3.3
# Copyright (C) 2020 Free Software Foundation, Inc.
# License GPLv3+: GNU GPL version 3 or later <https://gnu.org/licenses/gpl.html>
# This is free software: you are free to change and redistribute it.
# There is NO WARRANTY, to the extent permitted by law.
```

### Interactive Command Examples

#### `mklabel LABEL-TYPE` (Create new partition table)
```bash
# Start parted on target device
sudo parted /dev/sdb

# Create a new GPT partition table
(parted) mklabel gpt

# Create a new MBR (msdos) partition table
(parted) mklabel msdos

# Verify the partition table was created
(parted) print
# Sample output:
# Partition Table: gpt
```

#### `mkpart PART-TYPE [FS-TYPE] START END` (Create new partition)
```bash
# Create a primary partition with ext4 filesystem
(parted) mkpart primary ext4 1MiB 1001MiB

# Create a logical partition (only for MBR)
(parted) mkpart logical ext4 1001MiB 2001MiB

# Create a partition without specifying filesystem type
(parted) mkpart primary 1MiB 1001MiB

# Create a swap partition
(parted) mkpart primary linux-swap 1MiB 2001MiB

# Create an EFI system partition
(parted) mkpart primary fat32 1MiB 513MiB
(parted) set 1 boot on
(parted) set 1 esp on
```

#### `print` (Display partition table)
```bash
# Display current partition table
(parted) print

# Display partition table with free space
(parted) print free

# Display partition table in sectors
(parted) unit s
(parted) print

# Display partition table in compact form
(parted) print all

# Sample output:
# Model: ATA QEMU HARDDISK (scsi)
# Disk /dev/sdb: 10.7GB
# Sector size (logical/physical): 512B/512B
# Partition Table: gpt
# Disk Flags: 
#
# Number  Start   End     Size    File system  Name     Flags
#  1      1049kB  538MB   537MB   ext4         primary
#  2      538MB   1075MB  537MB   linux-swap   primary
```

#### `rm NUMBER` (Delete partition)
```bash
# First, list partitions to identify the correct number
(parted) print

# Delete partition number 1
(parted) rm 1

# Delete partition number 2
(parted) rm 2

# Verify deletion
(parted) print
```

#### `resizepart NUMBER END` (Resize partition)
```bash
# First, list partitions to see current size
(parted) print

# Resize partition 1 to 2GB
(parted) resizepart 1 2001MiB

# Resize partition 2 to 3GB
(parted) resizepart 2 3001MiB

# Verify resize
(parted) print
```

#### `set NUMBER FLAG STATE` (Change partition flag)
```bash
# Set partition 1 as bootable (MBR)
(parted) set 1 boot on

# Set partition 1 as EFI System Partition (GPT)
(parted) set 1 esp on

# Set partition 1 as hidden
(parted) set 1 hidden on

# Set partition 2 as RAID
(parted) set 2 raid on

# Remove boot flag from partition 1
(parted) set 1 boot off

# Verify flags
(parted) print
```

#### `quit` (Exit the program)
```bash
# Exit parted without saving changes (if changes haven't been committed)
(parted) quit

# If you've made changes, parted will ask for confirmation:
# Warning: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want to continue?
# Yes/No? yes
```

### Practical Workflow Examples

#### Creating a Complete GPT Partition Scheme
```bash
# Start parted on target device
sudo parted /dev/sdb

# Create GPT partition table
(parted) mklabel gpt

# Create EFI System Partition (500MB)
(parted) mkpart primary fat32 1MiB 501MiB
(parted) set 1 esp on
(parted) set 1 boot on

# Create root partition (20GB)
(parted) mkpart primary ext4 501MiB 21001MiB

# Create swap partition (4GB)
(parted) mkpart primary linux-swap 21001MiB 25001MiB

# Create home partition (remaining space)
(parted) mkpart primary ext4 25001MiB 100%

# Verify the partition table
(parted) print

# Quit parted
(parted) quit
```

#### Creating a Complete MBR Partition Scheme
```bash
# Start parted on target device
sudo parted /dev/sdb

# Create MBR partition table
(parted) mklabel msdos

# Create primary partition (100MB)
(parted) mkpart primary ext4 1MiB 101MiB
(parted) set 1 boot on

# Create extended partition
(parted) mkpart extended 101MiB 100%

# Create logical swap partition (1GB) within extended
(parted) mkpart logical linux-swap 101MiB 1101MiB

# Create logical data partition within extended
(parted) mkpart logical ext4 1101MiB 100%

# Verify the partition table
(parted) print

# Quit parted
(parted) quit
```

#### Resizing a Partition Workflow
```bash
# Start parted on target device
sudo parted /dev/sdb

# First, check current partition layout
(parted) print free

# Identify partition to resize and available free space
# Suppose partition 1 is 10GB and we want to expand it to 15GB

# Resize partition 1 to 15GB
(parted) resizepart 1 15GiB

# Verify the resize
(parted) print

# Note: After resizing the partition, you'll need to resize the filesystem
# For ext4, use: sudo resize2fs /dev/sdb1
```

### Scripting Examples

#### Automated Partition Creation Script
```bash
#!/bin/bash
# Script to create a standard partition scheme

DEVICE="/dev/sdb"

# Create GPT partition table and partitions
sudo parted -s $DEVICE <<EOF
mklabel gpt
mkpart primary fat32 1MiB 501MiB
set 1 esp on
set 1 boot on
mkpart primary ext4 501MiB 21001MiB
mkpart primary linux-swap 21001MiB 25001MiB
mkpart primary ext4 25001MiB 100%
EOF

# Format the partitions
sudo mkfs.vfat -F32 ${DEVICE}1
sudo mkfs.ext4 ${DEVICE}2
sudo mkswap ${DEVICE}3

echo "Partitioning complete"
```

#### Automated Partition Deletion Script
```bash
#!/bin/bash
# Script to delete all partitions on a device

DEVICE="/dev/sdb"

# Get the number of partitions
PART_COUNT=$(sudo parted -s $DEVICE print | grep -c "^[[:space:]]*[0-9]")

# Delete partitions in reverse order
for ((i=$PART_COUNT; i>=1; i--))
do
    sudo parted -s $DEVICE rm $i
    echo "Deleted partition $i"
done

echo "All partitions deleted"
```

### Best Practices and Safety Tips

#### Always Backup Before Making Changes
```bash
# Backup partition table before modifying
sudo sfdisk -d /dev/sdb > sdb_partition_table_backup.txt

# Backup important data
rsync -av /important/data/ /backup/location/
```

#### Verify Device Identity
```bash
# Check disk identity before operations
sudo fdisk -l /dev/sdb

# Verify disk size
sudo parted -s /dev/sdb print | grep "Disk /dev/sdb"
```

#### Unmount Partitions Before Modifying
```bash
# Check if partition is mounted
mount | grep /dev/sdb1

# Unmount if mounted
sudo umount /dev/sdb1

# Stop processes using the partition
sudo fuser -km /dev/sdb1
```

#### Use Appropriate Units
```bash
# Use consistent units (MiB, GiB) to avoid confusion
sudo parted /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Use percentage for remaining space
sudo parted /dev/sdb mkpart primary ext4 5GiB 100%
```

### Troubleshooting Examples

#### Handling "unrecognized disk label" Error
```bash
# If you see "unrecognized disk label", create a partition table first
sudo parted /dev/sdb mklabel gpt
```

#### Handling "partition doesn't exist" Error
```bash
# If you get "partition doesn't exist", check partition numbers
sudo parted /dev/sdb print

# Use correct partition number
sudo parted /dev/sdb resizepart 1 2000MiB
```

#### Handling "can't have overlapping partitions" Error
```bash
# If you get overlapping partitions, check start/end points
sudo parted /dev/sdb print free

# Delete and recreate partitions with correct boundaries
sudo parted /dev/sdb rm 1
sudo parted /dev/sdb mkpart primary ext4 1MiB 1001MiB
```

### Integration with Other Tools

#### Using `parted` with `mkfs`
```bash
# Create partition with parted
sudo parted -s /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Format with mkfs
sudo mkfs.ext4 /dev/sdb1

# Verify filesystem
sudo fsck /dev/sdb1
```

#### Using `parted` with `resize2fs`
```bash
# Resize partition with parted
sudo parted -s /dev/sdb resizepart 1 2000MiB

# Resize filesystem with resize2fs
sudo resize2fs /dev/sdb1

# Verify new size
sudo df -h /dev/sdb1
```

These examples cover the full range of `parted` functionality from basic listing to complex partitioning schemes. Remember to always verify device names and back up important data before modifying partition tables. The `parted` tool is particularly valuable for working with large disks and modern partition tables like GPT.