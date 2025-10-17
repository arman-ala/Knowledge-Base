# sfdisk command

2025-10-11 23:32
Status: #DONE 
Tags: [[Linux]]

---
`sfdisk` is a powerful, script-oriented command-line tool for manipulating disk partition tables on Linux. It supports modern standards like **GPT (GUID Partition Table)** and the legacy **MBR (DOS)** format, making it ideal for automation and system administrators who prefer non-interactive partitioning.

### 📚 Basic Command Flags

Unlike interactive tools, `sfdisk` is designed to read commands from scripts or standard input. These common flags control its overall behavior:

| Flag | Description |
| :--- | :--- |
| `-l, --list` | **Lists** the partition tables for specified devices or all known devices. |
| `-d, --dump` | **Dumps** a device's partition table in a format usable as **input to `sfdisk`**, ideal for backups. |
| `-X, --label` | Specifies the disk **label type** (e.g., `dos` for MBR, `gpt` for GPT) to create. |
| `-n, --no-act` | Performs a **dry run**; does everything except writing to the device. |
| `-f, --force` | **Disables** all consistency checking. |
| `-b, --backup` | **Backs up** the current partition table sectors before making changes. |
| `-a, --append` | **Appends** new partitions to an existing table instead of creating a new one. |
| `-V, --verify` | **Verifies** the integrity of the partition table. |
| `-w, --wipe` | **Wipes** existing signatures from the device to avoid collisions. |

### 🛠️ Operation Commands

These commands perform specific actions on a partition or disk and are often used with the `-N` option to target a specific partition.

| Command | Description |
| :--- | :--- |
| `--delete` | **Deletes** all or specified partitions on a device. |
| `--part-type` | Changes a partition's **type ID** (e.g., to `83` for Linux, `82` for swap). |
| `-A, --activate` | Sets the **bootable flag** for a partition (MBR only). |
| `--part-label` | Sets the **name (label)** for a GPT partition. |
| `--part-uuid` | Sets the **UUID** for a GPT partition. |
| `--disk-id` | Changes the disk's **identifier** (MBR) or **GUID** (GPT). |
| `-r, --reorder` | **Renumbers** partitions to match their order on the disk. |

### 💡 Key Usage Examples

#### Listing and Backing Up Partitions

To **list all partitions** on a specific disk, use the `-l` flag:
```bash
sudo sfdisk -l /dev/sdb
```
To **back up** the partition layout of `/dev/sdb` to a file for later restoration:
```bash
sudo sfdisk -d /dev/sdb > sdb_backup.txt
```
You can later **restore** this layout using:
```bash
sudo sfdisk /dev/sdb < sdb_backup.txt
```

#### Creating Partitions with Input Format

`sfdisk` uses a flexible input format where you define partitions with lines containing `start, size, type, bootable`. Omitted fields use sensible defaults.

To create a new GPT disk with a 1 GiB boot partition (type EFI) and a 5 GiB Linux root partition:
```bash
echo -e 'label: gpt\nsize=1G, type=U\nsize=5G, type=L' | sudo sfdisk /dev/sdb
```
- **`label: gpt`**: Specifies the partition table type.
- **`size=1G, type=U`**: Creates a 1 GiB partition. `U` is a shortcut for the EFI System Partition type in GPT.
- **`size=5G, type=L`**: Creates a 5 GiB partition. `L` is a shortcut for the Linux filesystem type.

For MBR, create a 500 MiB FAT32 partition (`type=c`) and a Linux partition using the remaining space:
```bash
echo -e ',500M,c\n;' | sudo sfdisk /dev/sdb
```
- **`,`**: Omits the start sector, so `sfdisk` chooses the optimal aligned location.
- **`500M,c`**: Creates a 500 MiB FAT32 partition (`c` is the MBR type ID shortcut).
- **`;`**: Creates a partition using all remaining free space with the default Linux type.

#### Modifying and Managing Existing Partitions

To **delete the second partition** on `/dev/sdb`:
```bash
sudo sfdisk --delete /dev/sdb 2
```
To **change the type** of partition 1 on `/dev/sdb` to Linux swap (`82` for MBR):
```bash
sudo sfdisk --part-type /dev/sdb 1 82
```
To **mark partition 1 as bootable** on an MBR disk:
```bash
sudo sfdisk -A /dev/sdb 1
```

## Comprehensive Examples for `sfdisk` Commands and Flags

### Basic Command Flag Examples

### `-l, --list` (List partition tables)
```bash
# List partition table for a specific device
sudo sfdisk -l /dev/sda

# List partition tables for all devices
sudo sfdisk -l

# List with detailed information including sector sizes
sudo sfdisk -l -o Device,Start,End,Sectors,Size,Type /dev/sda
```

### `-d, --dump` (Dump partition table in script format)
```bash
# Dump partition table to a file for backup
sudo sfdisk -d /dev/sda > sda_backup.txt

# View the dumped format
cat sda_backup.txt
# Example output:
# label: dos
# label-id: 0x12345678
# device: /dev/sda
# unit: sectors
# /dev/sda1 : start=        2048, size=     204800, type=83, bootable
# /dev/sda2 : start=      206848, size=    2097152, type=82
```

### `-X, --label` (Specify disk label type)
```bash
# Create a new GPT partition table
sudo sfdisk --label gpt /dev/sdb

# Create a new MBR (DOS) partition table
sudo sfdisk --label dos /dev/sdb

# Create a Sun partition table
sudo sfdisk --label sun /dev/sdb
```

### `-n, --no-act` (Perform dry run)
```bash
# Test creating partitions without actually writing to disk
echo -e 'label: gpt\nsize=1G, type=U\nsize=5G, type=L' | sudo sfdisk --no-act /dev/sdb

# Test restoring a partition table without making changes
sudo sfdisk --no-act /dev/sdb < sda_backup.txt
```

### `-f, --force` (Disable consistency checking)
```bash
# Force partition table creation even if there are errors
sudo sfdisk --force /dev/sdb <<EOF
label: gpt
size=1G, type=U
EOF

# Force deletion of partitions without confirmation
sudo sfdisk --force --delete /dev/sdb 1
```

### `-b, --backup` (Backup current partition table)
```bash
# Create partitions with automatic backup
echo -e 'label: gpt\nsize=1G, type=U' | sudo sfdisk --backup /dev/sdb

# The backup will be saved as /dev/sdb.bak
sudo ls -la /dev/sdb.bak
```

### `-a, --append` (Append new partitions)
```bash
# Add a new partition to existing table without recreating
echo -e 'size=2G, type=L' | sudo sfdisk --append /dev/sdb

# Verify the new partition was added
sudo sfdisk -l /dev/sdb
```

### `-V, --verify` (Verify partition table integrity)
```bash
# Verify partition table on a device
sudo sfdisk --verify /dev/sda

# Verify after making changes
echo -e 'label: gpt\nsize=1G, type=U' | sudo sfdisk /dev/sdb
sudo sfdisk --verify /dev/sdb
```

### `-w, --wipe` (Wipe existing signatures)
```bash
# Wipe all signatures before creating new partition table
sudo sfdisk --wipe /dev/sdb <<EOF
label: gpt
size=1G, type=U
EOF

# Wipe only filesystem signatures
sudo sfdisk --wipe-signatures /dev/sdb
```

### Operation Command Examples

### `--delete` (Delete partitions)
```bash
# Delete a specific partition
sudo sfdisk --delete /dev/sdb 1

# Delete all partitions on a device
sudo sfdisk --delete /dev/sdb

# Delete multiple partitions
sudo sfdisk --delete /dev/sdb 1 2 3
```

### `--part-type` (Change partition type)
```bash
# Change partition 1 to Linux swap (MBR type 82)
sudo sfdisk --part-type /dev/sdb 1 82

# Change partition 2 to EFI System (GPT type U)
sudo sfdisk --part-type /dev/sdb 2 U

# List available partition types for GPT
sudo sfdisk --part-type /dev/sdb 1 L
```

### `-A, --activate` (Set bootable flag - MBR only)
```bash
# Set partition 1 as bootable
sudo sfdisk -A /dev/sdb 1

# Remove bootable flag from partition 1
sudo sfdisk -A /dev/sdb 1 off

# Check bootable status
sudo sfdisk -d /dev/sdb | grep bootable
```

### `--part-label` (Set partition name - GPT only)
```bash
# Set partition 1 name to "EFI System"
sudo sfdisk --part-label /dev/sdb 1 "EFI System"

# Set partition 2 name to "Root Filesystem"
sudo sfdisk --part-label /dev/sdb 2 "Root Filesystem"

# View partition names
sudo sfdisk -l -o Device,Name /dev/sdb
```

### `--part-uuid` (Set partition UUID - GPT only)
```bash
# Set custom UUID for partition 1
sudo sfdisk --part-uuid /dev/sdb 1 "C12A7328-F81F-11D2-BA4B-00A0C93EC93B"

# Generate a random UUID for partition 2
sudo sfdisk --part-uuid /dev/sdb 2 random

# Verify UUID assignment
sudo sfdisk -l -o Device,UUID /dev/sdb
```

### `--disk-id` (Change disk identifier)
```bash
# Change MBR disk identifier
sudo sfdisk --disk-id /dev/sdb 0x12345678

# Change GPT disk GUID
sudo sfdisk --disk-id /dev/sdb "12345678-1234-5678-ABCD-123456789ABC"

# Verify disk identifier
sudo sfdisk -d /dev/sdb | grep -E "label-id|identifier"
```

### `-r, --reorder` (Renumber partitions)
```bash
# Create partitions with non-sequential numbers
sudo sfdisk /dev/sdb <<EOF
label: gpt
size=1G, type=U
size=1G, type=L
EOF

# Delete partition 1 to create non-sequential numbering
sudo sfdisk --delete /dev/sdb 1

# Reorder partitions to match disk order
sudo sfdisk --reorder /dev/sdb

# Verify renumbering
sudo sfdisk -l /dev/sdb
```

### Advanced Usage Examples

#### Creating Complex Partition Schemes
```bash
# Create a complete GPT scheme with multiple partitions
sudo sfdisk /dev/sdb <<EOF
label: gpt
size=512M, type=U, name="EFI System"
size=2G, type=L, name="Root"
size=1G, type=S, name="Swap"
size=10G, type=L, name="Home"
EOF

# Create MBR scheme with extended partition
sudo sfdisk /dev/sdb <<EOF
label: dos
size=100M, type=c, bootable
size=1G, type=5
size=500M, type=82
;
EOF
```

#### Backup and Restore Workflow
```bash
# Backup partition table
sudo sfdisk -d /dev/sda > sda_layout.txt

# Make changes to partition table
sudo sfdisk /dev/sda <<EOF
label: gpt
size=1G, type=U
EOF

# Restore original layout if needed
sudo sfdisk /dev/sda < sda_layout.txt
```

#### Scripting Partition Management
```bash
#!/bin/bash
# Example script to set up a disk with sfdisk

DEVICE="/dev/sdb"

# Create backup
sudo sfdisk -d $DEVICE > ${DEVICE}.backup

# Create new partition table
sudo sfdisk $DEVICE <<EOF
label: gpt
size=512M, type=U
size=5G, type=L
size=2G, type=S
EOF

# Format partitions
sudo mkfs.vfat -F32 ${DEVICE}1
sudo mkfs.ext4 ${DEVICE}2
sudo mkswap ${DEVICE}3

echo "Partition setup complete"
```

#### Partition Information Extraction
```bash
# Extract specific partition information
sudo sfdisk -d /dev/sda | awk '/^\/dev\// {print $1, $4, $6}'

# Get total disk size in sectors
sudo sfdisk -l /dev/sda | grep "Disk /dev/sda" | awk '{print $5}'

# Check if partition is bootable
sudo sfdisk -d /dev/sda | grep bootable
```

### Safety and Verification Examples

#### Dry Run Before Execution
```bash
# Always test with --no-act before making changes
PARTITION_SCHEME="label: gpt\nsize=1G, type=U\nsize=5G, type=L"

echo -e $PARTITION_SCHEME | sudo sfdisk --no-act /dev/sdb

# If output looks correct, proceed without --no-act
echo -e $PARTITION_SCHEME | sudo sfdisk /dev/sdb
```

#### Verification After Changes
```bash
# Verify partition table was created correctly
sudo sfdisk --verify /dev/sdb

# Check filesystem creation
sudo lsblk /dev/sdb

# Verify partition types
sudo sfdisk -l -o Device,Type /dev/sdb
```

#### Error Handling
```bash
# Check for partition errors
if ! sudo sfdisk --verify /dev/sdb; then
    echo "Partition table has errors"
    exit 1
fi

# Check if device exists
if [ ! -b "/dev/sdb" ]; then
    echo "Device /dev/sdb does not exist"
    exit 1
fi
```
