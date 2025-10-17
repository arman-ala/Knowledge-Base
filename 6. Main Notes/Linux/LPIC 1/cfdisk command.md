# cfdisk command

2025-10-11 23:25
Status: #DONE 
Tags: [[Linux]]

---
`cfdisk` is a user-friendly, text-based disk partition table manipulation program for Linux. It provides a curses-based menu-driven interface that is more intuitive than traditional `fdisk`, making it ideal for common partitioning tasks.

Here is a quick overview of `cfdisk`'s command-line options and its interactive commands.

### ⌨️ Common Command-Lary Line Options

Most options are not needed for daily use, but the following can be helpful:

| Option | Description |
| :--- | :--- |
| `-h`, `--help` | Display help text and exit. |
| `-z`, `--zero` | Start without reading the existing partition table. Ideal for creating a new table or working from a script. |
| `-L`, `--color[=when]` | Colorize output (`auto`, `never`, or `always`). |
| `-v`, `--version` | Display version information. |

### 🖥️ Interactive Commands

Once inside the `cfdisk` interface, you use single-key commands for all operations:

| Command | Action |
| :--- | :--- |
| `n` | **C**reate a **n**ew partition from free space. |
| `d` | **D**elete the current partition. |
| `t` | Change the partition **t**ype (e.g., to Linux swap or EFI). |
| `b` | Toggle **b**ootable flag on the current partition (for MBR tables). |
| `r` | **R**esize the current partition. **Warning:** Shrinking may cause data loss. |
| `s` | **S**ort partitions by their start sector. |
| `W` (uppercase) | **W**rite the partition table to disk. Requires confirmation. |
| `q` | **Q**uit without saving any changes. |

### 🛠️ A Practical Workflow: Partitioning a Disk

Here is a typical workflow for partitioning a new disk, `/dev/sdb`, using `cfdisk`.

1.  **Launch cfdisk**
    Open your terminal and start `cfdisk` with the target device. You will need administrative privileges.
    ```bash
    sudo cfdisk /dev/sdb
    ```

2.  **Choose a Partition Table Type**
    If the disk is new and has no partition table, `cfdisk` will prompt you to create one.
    - Select **`gpt`** for modern UEFI systems and disks larger than 2TB.
    - Select **`dos`** for older BIOS-based systems.

3.  **Create a Root Partition**
    - Use the **arrow keys** to highlight `Free space` and press `n`.
    - Enter the desired size, for example, `20G` for a 20 GiB partition, and press Enter. The default size uses all remaining free space.
    - The partition will be created with a default type, typically "Linux filesystem."

4.  **Create a Swap Partition**
    - With free space still highlighted, press `n` again.
    - Enter the size, for example, `4G` for a 4 GiB swap partition.
    - With the new partition highlighted, press `t` to change its type. A list of types will appear. Find and select **`Linux swap`** (its code is `82` in MBR).

5.  **Write Changes and Exit**
    - Review your partition layout. If correct, press `W` (capital W) to write the table to the disk.
    - You must type `yes` to confirm this destructive operation.
    - Press `q` to quit the program.

## Comprehensive Examples for `cfdisk` Commands and Options

### Command-Line Option Examples

### `-h`, `--help` (Display help text)
```bash
# Display help information
cfdisk -h
# or
cfdisk --help

# Sample output:
# Usage: cfdisk [options] <device>
#
# Options:
#  -L, --color[=<when>]     colorize output (auto, never, always)
#  -h, --help               display this help
#  -l, --list               display partitions and exit
#  -r, --read-only          force read-only mode
#  -s, --size <SIZE>        specify sector size
#  -v, --version            output version information and exit
#  -z, --zero               start with zeroed partition table
```

### `-z`, `--zero` (Start without reading existing partition table)
```bash
# Start cfdisk on /dev/sdb without reading existing partition table
# Useful for creating a completely new partition table
sudo cfdisk -z /dev/sdb

# This will show a blank disk with no partitions, even if the disk has partitions
# You can then create a new partition table from scratch
```

### `-L`, `--color[=when]` (Colorize output)
```bash
# Enable color output (auto-detects terminal capability)
sudo cfdisk -L /dev/sda

# Force color output on
sudo cfdisk --color=always /dev/sda

# Disable color output
sudo cfdisk --color=never /dev/sda

# Auto color (default, enables if terminal supports it)
sudo cfdisk --color=auto /dev/sda
```

### `-v`, `--version` (Display version information)
```bash
# Display cfdisk version
cfdisk -v
# or
cfdisk --version

# Sample output:
# cfdisk from util-linux 2.37.2
```

### Interactive Command Examples

### `n` (Create new partition)
```bash
# Launch cfdisk on /dev/sdb
sudo cfdisk /dev/sdb

# Inside cfdisk interface:
# 1. Use arrow keys to highlight "Free space"
# 2. Press 'n' to create new partition
# 3. You'll be prompted for partition size:
#    Partition size: [5G]  # Enter size like 5G, 500M, etc.
# 4. Press Enter to create partition

# Example sequence in terminal:
#   Disk: /dev/sdb
#   Size: 10 GiB, 10737418240 bytes, 20971520 sectors
#   Label: gpt, identifier: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
#
#   Device     Start     End Sectors  Size Type
#   Free space 2048 20969471 20967424  10G
#
#   [ Bootable ] [ Delete ] [  Help  ] [  Quit  ] [  Write  ]
#   [  Type   ] [ Units  ] [ Resize ] [  Sort  ]
#
#   → Free space
#
# After pressing 'n':
#   Partition size: 5G
#
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048 10487807 10485760    5G Linux filesystem
```

### `d` (Delete partition)
```bash
# Inside cfdisk interface:
# 1. Use arrow keys to highlight the partition to delete
# 2. Press 'd' to delete the partition
# 3. Confirm deletion if prompted

# Example sequence:
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048 10487807 10485760    5G Linux filesystem
#
#   → /dev/sdb1
#
# After pressing 'd':
#   Device     Start     End Sectors  Size Type
#   Free space 2048 20969471 20967424  10G
```

### `t` (Change partition type)
```bash
# Inside cfdisk interface:
# 1. Highlight the partition to change
# 2. Press 't' to change type
# 3. Select new type from the list

# Example sequence:
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048 10487807 10485760    5G Linux filesystem
#
#   → /dev/sdb1
#
# After pressing 't':
#   Partition type: Linux filesystem (default)
#   Linux swap  82
#   Linux filesystem  83
#   Linux LVM  8e
#   ... (long list)
#
# Select "Linux swap" (type 82):
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048 10487807 10485760    5G Linux swap
```

### `b` (Toggle bootable flag - MBR only)
```bash
# Inside cfdisk interface with MBR partition table:
# 1. Highlight the partition to make bootable
# 2. Press 'b' to toggle bootable flag
# 3. An asterisk (*) will appear in the "Boot" column

# Example sequence:
#   Device Boot Start     End Sectors  Size Type Id
#   /dev/sdb1      2048 10487807 10485760    5G Linux 83
#
#   → /dev/sdb1
#
# After pressing 'b':
#   Device Boot Start     End Sectors  Size Type Id
#   /dev/sdb1  *   2048 10487807 10485760    5G Linux 83
```

### `r` (Resize partition)
```bash
# ⚠️ WARNING: Resizing partitions can cause data loss!
# Inside cfdisk interface:
# 1. Highlight the partition to resize
# 2. Press 'r' to resize
# 3. Enter new size

# Example sequence:
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048 10487807 10485760    5G Linux filesystem
#
#   → /dev/sdb1
#
# After pressing 'r':
#   Resize partition from 5G to: 3G
#
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048  6293503  6291456    3G Linux filesystem
#
# Note: The filesystem itself must be resized separately with resize2fs or equivalent
```

### `s` (Sort partitions by start sector)
```bash
# Inside cfdisk interface:
# 1. Press 's' to sort partitions by their start sector
# 2. Partitions will be reordered in the display

# Example sequence:
# Before sorting:
#   Device     Start     End Sectors  Size Type
#   /dev/sdb3  10487808 14681087  4193280    2G Linux swap
#   /dev/sdb1  2048     10487807 10485760    5G Linux filesystem
#   /dev/sdb2  14681088 20969471  6288384    3G Linux LVM
#
# After pressing 's':
#   Device     Start     End Sectors  Size Type
#   /dev/sdb1  2048     10487807 10485760    5G Linux filesystem
#   /dev/sdb3  10487808 14681087  4193280    2G Linux swap
#   /dev/sdb2  14681088 20969471  6288384    3G Linux LVM
```

### `W` (Write partition table to disk)
```bash
# Inside cfdisk interface:
# 1. Press 'W' (uppercase) to write changes to disk
# 2. Confirm by typing "yes"
# 3. Changes become permanent

# Example sequence:
#   [ Bootable ] [ Delete ] [  Help  ] [  Quit  ] [  Write  ]
#   [  Type   ] [ Units  ] [ Resize ] [  Sort  ]
#
#   → Write
#
# After pressing 'W':
#   Are you sure you want to write the partition table to disk? (yes/no): yes
#   The partition table has been altered.
#
#   Syncing disks.
```

### `q` (Quit without saving)
```bash
# Inside cfdisk interface:
# 1. Press 'q' to quit without saving changes
# 2. All changes made in the session will be discarded

# Example sequence:
#   [ Bootable ] [ Delete ] [  Help  ] [  Quit  ] [  Write  ]
#   [  Type   ] [ Units  ] [ Resize ] [  Sort  ]
#
#   → Quit
#
# After pressing 'q':
#   # Returns to shell prompt, no changes written to disk
```

### Practical Workflow Example

#### Creating a Complete Partition Scheme with `cfdisk`

```bash
# 1. Launch cfdisk on a new disk /dev/sdb
sudo cfdisk /dev/sdb

# 2. If the disk is new, select partition table type:
#    Select: gpt (for modern systems)

# 3. Create EFI partition:
#    Highlight "Free space" → Press 'n'
#    Partition size: 512M
#    With partition highlighted → Press 't' → Select "EFI System"

# 4. Create root partition:
#    Highlight remaining "Free space" → Press 'n'
#    Partition size: 20G (or leave default for all space)

# 5. Create swap partition:
#    Highlight remaining "Free space" → Press 'n'
#    Partition size: 4G
#    With partition highlighted → Press 't' → Select "Linux swap"

# 6. Verify partition layout:
#    Device     Start     End Sectors  Size Type
#    /dev/sdb1  2048  1050623  1048576  512M EFI System
#    /dev/sdb2  1050624 41945087 40894464   20G Linux filesystem
#    /dev/sdb3  41945088 50331647  8386560    4G Linux swap

# 7. Write changes:
#    Press 'W' → Confirm with "yes"

# 8. Format partitions after exiting cfdisk:
sudo mkfs.vfat -F32 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
sudo mkswap /dev/sdb3
```

### Safety and Verification Examples

#### Verifying Partition Creation
```bash
# After creating partitions, verify with multiple tools:
lsblk /dev/sdb
sudo fdisk -l /dev/sdb
sudo parted /dev/sdb print

# Check filesystem creation:
sudo blkid /dev/sdb1 /dev/sdb2 /dev/sdb3
```

#### Safe Quit Example
```bash
# If you make a mistake and want to discard changes:
# Inside cfdisk interface:
# Press 'q' to quit without saving
# All changes made in the session will be lost
```

#### Backup Before Major Changes
```bash
# Backup partition table before modifying:
sudo sfdisk -d /dev/sda > partition_table_backup.txt

# Restore if needed:
sudo sfdisk /dev/sda < partition_table_backup.txt
```

### ⚠️ Important Considerations and Best Practices

- **Safety First**: `cfdisk` operates directly on disk data. **Always back up critical data** before modifying partition tables. Changes are only made permanent after you confirm the `Write` command; use `Quit` to exit safely without saving changes.
- **Partitioning vs. Formatting**: `cfdisk` only sets up the partition table. To use a new partition, you must create a filesystem on it (e.g., `sudo mkfs.ext4 /dev/sdb1`) or initialize swap space (`sudo mkswap /dev/sdb2`).
- **Resizing Partitions**: While `cfdisk` can resize partitions, **shrinking a partition carries a high risk of data loss** if it contains a filesystem. It is safer to back up data, delete the partition, and create a new one with the desired size.
