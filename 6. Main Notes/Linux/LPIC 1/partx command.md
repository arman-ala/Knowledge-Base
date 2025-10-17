# partx command

2025-10-12 01:21
Status: #DONE 
Tags: [[Linux]]

---
The `partx` command in Linux is used to tell the kernel about the presence and numbering of on-disk partitions. It's primarily used to inform the operating system of partition table changes without requiring a reboot.

### 📖 Understanding partx and Its Common Options

Unlike `fdisk` or `parted`, `partx` doesn't create or delete partitions on the disk itself. Instead, it works by instructing the kernel to add or remove partitions from its internal view, which is crucial after modifying a partition table with other tools.

Here are its most common command-line flags:

| **Flag** | **Long Option** | **Description** |
| :--- | :--- | :--- |
| `-a` | `--add` | Add all partitions or a specified range to the kernel's knowledge. |
| `-d` | `--delete` | Delete all partitions or a specified range from the kernel's knowledge. |
| `-s` | `--show` | List the partitions of a specified device. This is the modern way to list partitions. |
| `-l` | `--list` | List the partitions. **Note:** This output format is deprecated in favor of `--show`. |
| `-n M:N` | `--nr M:N` | Specify a range of partitions (e.g., `3:5`). Negative numbers count from the end (e.g., `-1:-1` for the last partition). |
| `-o` | `--output` | Define which output columns to display with `--show` (e.g., `NR, START, SIZE`). |
| `-v` | `--verbose` | Provide verbose output, giving more details about the operation. |
| `-t` | `--type` | Specify the partition table type (e.g., `dos`, `gpt`). Use `--list-types` to see all supported types. |

### 🛠️ How to Use partx: Common Examples

Here are some practical examples of how to use `partx` for various system administration tasks. Remember to use `sudo` for these commands as they require root privileges.

- **Listing Partitions**
    To display the partition table of a disk (e.g., `/dev/sda`), use the `--show` option.
    ```bash
    sudo partx --show /dev/sda
    ```
    You can specify which columns to output for a more focused view. This command shows only the partition number, start sector, and size for partition number 10.
    ```bash
    sudo partx -o NR,START,SIZE --nr 10 /dev/sda
    ```

- **Adding Partitions to Kernel Awareness**
    After creating a new partition with a tool like `fdisk`, use this command to make the kernel aware of all new partitions on the disk. The `-v` (verbose) flag provides helpful feedback.
    ```bash
    sudo partx -v -a /dev/sdb
    ```
    To add a specific range of partitions (e.g., partitions 3 through 5), use the `--nr` option.
    ```bash
    sudo partx -a --nr 3:5 /dev/sdb
    ```

- **Removing Partitions from Kernel Awareness**
    To remove all partition entries for a disk from the kernel, perhaps before re-adding them to clear stale data.
    ```bash
    sudo partx -d /dev/sdb
    ```
    To delete only the last partition on a disk. This uses negative numbers with `--nr` to specify the range.
    ```bash
    sudo partx -d --nr -1:-1 /dev/sdb
    ```

### 💡 A Practical Workflow and Important Considerations

A common workflow after modifying partitions with `fdisk` is to use `partx` to ensure the kernel recognizes the changes without a reboot:
```bash
# 1. After using fdisk and writing changes, the kernel might not see the new layout.
# 2. First, delete the old kernel entries for the disk.
sudo partx -d /dev/sda

# 3. Then, add the new partition table entries.
sudo partx -a /dev/sda

# 4. Verify the changes are visible.
sudo partx --show /dev/sda
```

**Key points to remember:**
- **`partx` does not alter the on-disk partition table**; it only manages the kernel's in-memory view.
- If you encounter `BLKPG: Device or resource busy` errors, try specifying both the partition and the disk (e.g., `partx -a /dev/sdb1 /dev/sdb`) or ensure no processes are using the partitions.
- `partx` is part of the `util-linux` package, which is standard on most Linux distributions.

## Comprehensive Examples for `partx` Commands and Flags

### Understanding `partx` and Its Role

`partx` is a utility that informs the kernel about partition table changes. Unlike partitioning tools that modify the on-disk partition table (like `fdisk`, `parted`, or `sfdisk`), `partx` updates the kernel's in-memory view of partitions. This is crucial after modifying partition tables to ensure the kernel recognizes the new or changed partitions without requiring a system reboot.

#### Why `partx` is Necessary:
- **Kernel Partition Cache**: The kernel maintains its own cache of partition information that doesn't automatically update when the on-disk partition table changes
- **Avoiding Reboots**: Without `partx`, you would need to reboot the system for the kernel to recognize partition changes
- **Stale Partition Entries**: When partitions are deleted or modified, the kernel might still hold references to old partitions, causing conflicts

### Basic Command Flag Examples

#### `-a, --add` (Add partitions to kernel awareness)
```bash
# Add all partitions from /dev/sdb to kernel awareness
sudo partx -a /dev/sdb

# Add specific partition (partition 3) to kernel awareness
sudo partx -a --nr 3 /dev/sdb

# Add range of partitions (3 through 5) to kernel awareness
sudo partx -a --nr 3:5 /dev/sdb

# Add partitions with verbose output
sudo partx -v -a /dev/sdb
# Sample verbose output:
# partx: /dev/sdb: partition 1 added
# partx: /dev/sdb: partition 2 added
# partx: /dev/sdb: partition 3 added
```

#### `-d, --delete` (Remove partitions from kernel awareness)
```bash
# Remove all partitions for /dev/sdb from kernel awareness
sudo partx -d /dev/sdb

# Remove specific partition (partition 2) from kernel awareness
sudo partx -d --nr 2 /dev/sdb

# Remove range of partitions (2 through 4) from kernel awareness
sudo partx -d --nr 2:4 /dev/sdb

# Remove the last partition using negative indexing
sudo partx -d --nr -1:-1 /dev/sdb

# Remove with verbose output
sudo partx -v -d /dev/sdb
# Sample verbose output:
# partx: /dev/sdb: partition 1 deleted
# partx: /dev/sdb: partition 2 deleted
```

#### `-s, --show` (List partitions)
```bash
# List all partitions for /dev/sda
sudo partx --show /dev/sda

# List partitions with custom output columns
sudo partx --show -o NR,START,END,SIZE /dev/sda

# List specific partition (partition 5)
sudo partx --show --nr 5 /dev/sda

# List range of partitions (3 through 7)
sudo partx --show --nr 3:7 /dev/sda

# Sample output:
# NR  START      END        SECTORS    SIZE NAME
#  1  2048       206847     204800     100M EFI System
#  2  206848     411647     204800     100M Microsoft basic data
#  3  411648     2306047    1894400    925M Linux filesystem
```

#### `-l, --list` (List partitions - deprecated)
```bash
# List partitions using deprecated format
sudo partx --list /dev/sda

# Note: This is deprecated in favor of --show
# Output format is less structured and may be removed in future versions
```

#### `-n M:N, --nr M:N` (Specify partition range)
```bash
# Specify a single partition (partition 4)
sudo partx --nr 4 --show /dev/sda

# Specify range of partitions (2 through 5)
sudo partx --nr 2:5 --show /dev/sda

# Specify from partition 3 to the end
sudo partx --nr 3:-1 --show /dev/sda

# Specify the last two partitions
sudo partx --nr -2:-1 --show /dev/sda

# Negative indexing examples:
# -1 means last partition
# -2 means second to last partition
# etc.
```

#### `-o, --output` (Define output columns)
```bash
# Show only partition number and size
sudo partx --show -o NR,SIZE /dev/sda

# Show partition number, start sector, and type
sudo partx --show -o NR,START,TYPE /dev/sda

# Show all available columns
sudo partx --show -o ALL /dev/sda

# Available columns include:
# NR, START, END, SECTORS, SIZE, NAME, UUID, TYPE, FLAGS
```

#### `-v, --verbose` (Verbose output)
```bash
# Add partitions with verbose output
sudo partx -v -a /dev/sda
# Sample output:
# partx: /dev/sda: partition 1 added
# partx: /dev/sda: partition 2 added
# partx: /dev/sda: partition 3 added

# Delete partitions with verbose output
sudo partx -v -d /dev/sda
# Sample output:
# partx: /dev/sda: partition 1 deleted
# partx: /dev/sda: partition 2 deleted
# partx: /dev/sda: partition 3 deleted
```

#### `-t, --type` (Specify partition table type)
```bash
# List supported partition table types
sudo partx --list-types

# Add partitions specifying GPT type
sudo partx -t gpt -a /dev/sda

# Add partitions specifying MBR (DOS) type
sudo partx -t dos -a /dev/sda

# Show partitions specifying type
sudo partx -t gpt --show /dev/sda
```

### Practical Workflow Examples

#### Complete Partition Update Workflow
```bash
# Scenario: You've modified partition table with fdisk and need to update kernel

# 1. First, remove all old partition entries from kernel
sudo partx -v -d /dev/sdb

# 2. Then add all new partition entries to kernel
sudo partx -v -a /dev/sdb

# 3. Verify the changes
sudo partx --show /dev/sda

# Alternative: If you only modified specific partitions
# Remove only the affected partitions
sudo partx -d --nr 3:5 /dev/sdb

# Add back the modified partitions
sudo partx -a --nr 3:5 /dev/sdb
```

#### Handling Busy Partitions
```bash
# If you get "BLKPG: Device or resource busy" error
# Try specifying both partition and device
sudo partx -a /dev/sdb3 /dev/sdb

# If that doesn't work, ensure no processes are using the partition
# Check what's using the partition
sudo lsof /dev/sdb3

# Stop processes using the partition, then try again
sudo partx -d /dev/sdb3 /dev/sdb
sudo partx -a /dev/sdb3 /dev/sdb
```

#### Scripting Partition Updates
```bash
#!/bin/bash
# Script to update kernel partition table after modifications

DEVICE="/dev/sdb"

# Function to update kernel partition table
update_partitions() {
    echo "Updating kernel partition table for $DEVICE"
    
    # Remove all old entries
    sudo partx -v -d $DEVICE
    
    # Add all new entries
    sudo partx -v -a $DEVICE
    
    # Verify
    echo "Current partition table:"
    sudo partx --show $DEVICE
}

# Main execution
update_partitions
```

### Advanced Usage Examples

#### Working with Specific Partition Types
```bash
# Show only Linux partitions (type 83 for MBR, Linux for GPT)
sudo partx --show -o NR,TYPE,SIZE /dev/sda | grep -E "(83|Linux)"

# Show only swap partitions (type 82 for MBR, Linux swap for GPT)
sudo partx --show -o NR,TYPE,SIZE /dev/sda | grep -E "(82|Linux swap)"

# Show only EFI partitions (type U for GPT)
sudo partx --show -o NR,TYPE,NAME /dev/sda | grep "EFI"
```

#### Partition Size Analysis
```bash
# Show partition sizes in human-readable format
sudo partx --show -o NR,SIZE /dev/sda | \
    awk 'NR>1 {printf "Partition %s: %s\n", $1, $2}'

# Find partitions larger than 1GB
sudo partx --show -o NR,SIZE /dev/sda | \
    awk 'NR>1 && $2 ~ /G/ {print $1, $2}'

# Calculate total disk space used by partitions
sudo partx --show -o SIZE /dev/sda | \
    awk 'NR>1 {sum += $1} END {print "Total:", sum}'
```

#### Error Handling and Recovery
```bash
# Check if partition operations succeeded
if sudo partx -a /dev/sdb 2>/dev/null; then
    echo "Partitions added successfully"
else
    echo "Failed to add partitions"
    # Try alternative approach
    sudo partx -d /dev/sdb && sudo partx -a /dev/sdb
fi

# Verify partition consistency
sudo partx --verify /dev/sdb
```

### Troubleshooting Examples

#### Handling Partition Conflicts
```bash
# If partitions appear duplicated in kernel view
# First, remove all entries
sudo partx -d /dev/sda

# Then add them back
sudo partx -a /dev/sda

# Check for conflicts
ls /dev/sda* | grep -v "/dev/sda$"
```

#### Working with Loop Devices
```bash
# Create a loop device from a disk image
sudo losetup -f /path/to/disk.img

# Add partitions from loop device to kernel
sudo partx -a /dev/loop0

# Show loop device partitions
sudo partx --show /dev/loop0

# Remove loop device partitions when done
sudo partx -d /dev/loop0
sudo losetup -d /dev/loop0
```

#### Handling GPT Protective MBR
```bash
# For GPT disks with protective MBR, specify type explicitly
sudo partx -t gpt -a /dev/sda

# Show GPT partition details
sudo partx -t gpt --show -o NR,NAME,UUID /dev/sda
```

### Integration with Other Tools

#### Using `partx` with `fdisk`
```bash
# Create partition with fdisk
echo -e 'n\n\n\n+1G\nw' | sudo fdisk /dev/sdb

# Update kernel view with partx
sudo partx -a /dev/sdb

# Verify
sudo partx --show /dev/sdb
```

#### Using `partx` with `parted`
```bash
# Create partition with parted
sudo parted /dev/sdb mkpart primary ext4 1MiB 1001MiB

# Update kernel view
sudo partx -a /dev/sdb

# Verify
sudo partx --show /dev/sdb
```

### Best Practices

#### When to Use `partx`
1. **After Partition Creation**: Always run `partx -a` after creating new partitions
2. **After Partition Deletion**: Run `partx -d` to remove old partition entries
3. **After Partition Modification**: Remove and re-add modified partitions
4. **Before Formatting**: Ensure kernel recognizes partitions before creating filesystems

#### Safety Precautions
1. **Backup Data**: Always backup important data before modifying partitions
2. **Verify Device**: Double-check device names to avoid modifying the wrong disk
3. **Use Verbose Mode**: Use `-v` to monitor operations and catch errors
4. **Check Dependencies**: Ensure no processes are using partitions before removing them

#### Performance Considerations
1. **Batch Operations**: Use range operations (`--nr M:N`) for multiple partitions
2. **Avoid Unnecessary Updates**: Only run `partx` when partition table has changed
3. **Use Specific Partitions**: Target only affected partitions rather than entire disk

### Summary

`partx` is an essential utility for managing the kernel's view of partitions. While it doesn't modify the on-disk partition table, it plays a crucial role in ensuring the operating system recognizes partition changes without requiring reboots. By mastering `partx` commands and understanding when to use them, you can effectively manage disk partitions in both interactive and automated environments.

Remember that `partx` is part of the `util-linux` package, which is standard on most Linux distributions. The examples provided here cover common use cases and should help you handle most partition management scenarios effectively.