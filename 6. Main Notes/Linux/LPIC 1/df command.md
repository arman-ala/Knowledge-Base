# df command

2025-10-13 04:05
Status: #DONE 
Tags: [[Linux]]

---
# Mastering Disk Space Analysis: An In-Depth Guide to the `df` Command

## Introduction
In Linux system administration, monitoring filesystem utilization is a fundamental task. The `df` (disk free) command provides critical insights into disk space allocation across mounted filesystems. This comprehensive guide explores `df`'s functionality, options, and practical applications for both LPIC certification candidates and DevOps practitioners.

#### Command Syntax and Structure
```bash
df [OPTIONS] [FILE...]
```
When executed without arguments, `df` displays space usage for all mounted filesystems. Specifying files or directories shows usage for the filesystems containing those paths.

## Essential Command Options

| Option | Description | Values |
|--------|-------------|--------|
| `-a`/`--all` | Include pseudo, duplicate, or inaccessible filesystems | N/A |
| `-B`/`--block-size` | Scale sizes by specified block size | SIZE (e.g., K, M, G) |
| `-h`/`--human-readable` | Print sizes in human-readable format | N/A |
| `-H`/`--si` | Use powers of 1000 instead of 1024 | N/A |
| `-i`/`--inodes` | Display inode information instead of block usage | N/A |
| `-l`/`--local` | Restrict output to local filesystems | N/A |
| `-t`/`--type` | Limit output to specified filesystem types | TYPE (e.g., ext4, xfs) |
| `-x`/`--exclude-type` | Exclude specified filesystem types | TYPE |
| `--output` | Customize output fields | FIELD_LIST |
| `-T`/`--print-type` | Show filesystem type | N/A |

## Practical Examples and Explanations

**1. Basic Disk Usage Analysis**
```bash
df
```
**Sample Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
tmpfs            16384        0     16384   0% /dev/shm
```
- **Filesystem**: Block device or mount point
- **1K-blocks**: Total capacity in 1KB units
- **Used**: Consumed space
- **Available**: Free space
- **Use%**: Utilization percentage
- **Mounted on**: Mount point directory

**2. Human-Readable Format**
```bash
df -h
```
**Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
```
*Best Practice: Always use `-h` for routine checks to avoid calculation errors*

**3. Inode Utilization Analysis**
```bash
df -i /home
```
**Output:**
```
Filesystem      Inodes  IUsed  IFree IUse% Mounted on
/dev/sda1      1310720 124312 1186408   10% /home
```
*Tip: Monitor inode usage when dealing with numerous small files*

**4. Filesystem-Type Filtering**
```bash
df -t ext4 -t xfs
```
**Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
/dev/sdb1      103080128 2354560 100725568   3% /data
```
*Application: Useful in environments with mixed filesystem types*

**5. Custom Output Formatting**
```bash
df --output=source,size,pcent,target
```
**Output:**
```
Filesystem     1K-blocks Use% Mounted on
/dev/sda1       20510332  19% /
```
*DevOps Tip: Integrate with monitoring systems using customized fields*

**6. Combined Options Example**
```bash
df -hT --type=ext4
```
**Output:**
```
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda1      ext4   20G  3.5G   15G  19% /
```

### Advanced Usage Scenarios

**1. Script Integration for Monitoring**
```bash
#!/bin/bash
THRESHOLD=90
df -h | awk 'NR>1 {gsub(/%/,""); if ($5 > '$THRESHOLD') print "Alert: " $1 " at " $5 "%"}'
```
This script triggers alerts when usage exceeds 90%, demonstrating proactive capacity planning.

**2. Docker Container Analysis**
```bash
docker system df
```
*DevOps Insight: Containerized environments require specialized storage analysis tools*

**3. Network Filesystem Monitoring**
```bash
df -t nfs4 -h
```
*Best Practice: Monitor network storage separately due to different performance characteristics*

## Practical Examples and Explanations

### Basic Usage Examples

**1. Default Disk Usage Display**
```bash
df
```
**Sample Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
tmpfs            16384        0     16384   0% /dev/shm
/dev/sdb1      103080128 2354560 100725568   3% /data
```
**Explanation:** This is the most basic form of the `df` command, displaying disk usage information for all mounted filesystems. The output shows:
- **Filesystem**: The device name or remote filesystem identifier
- **1K-blocks**: Total storage capacity in 1KB blocks
- **Used**: Amount of used space in 1KB blocks
- **Available**: Amount of free space in 1KB blocks
- **Use%**: Percentage of used space
- **Mounted on**: The directory where the filesystem is mounted

This command is ideal for quick system checks but requires manual calculation to convert block sizes to human-readable format.

**2. Human-Readable Format**
```bash
df -h
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
tmpfs            16M     0   16M   0% /dev/shm
/dev/sdb1        99G  2.2G   96G   3% /data
```
**Explanation:** The `-h` flag transforms the output into a human-readable format using units like K (kilobytes), M (megabytes), and G (gigabytes). This is the most commonly used option for manual inspection as it eliminates the need for mental conversion of block sizes. Notice how the column headers change from "1K-blocks" to "Size" and the values are now in more intuitive units. This format is particularly useful during system administration tasks when quick assessment of disk usage is needed.

**3. SI Units (Powers of 1000)**
```bash
df -H
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        22G  3.8G   17G  19% /
tmpfs            17M     0   17M   0% /dev/shm
/dev/sdb1       106G  2.4G  104G   3% /data
```
**Explanation:** The `-H` flag is similar to `-h` but uses powers of 1000 (SI standard) instead of 1024 (binary standard). This results in slightly larger values because 1GB in SI terms is 1,000,000,000 bytes, while in binary terms it's 1,073,741,824 bytes. This option is useful when comparing disk usage with storage devices that use SI units on their packaging or specifications, which can help avoid confusion when planning capacity.

### Filesystem Analysis Examples

**4. Inode Utilization Analysis**
```bash
df -i
```
**Sample Output:**
```
Filesystem      Inodes  IUsed  IFree IUse% Mounted on
/dev/sda1      1310720 124312 1186408   10% /
tmpfs            4096      1   4095    1% /dev/shm
/dev/sdb1      6553600  23456 6530144    1% /data
```
**Explanation:** The `-i` flag displays inode information instead of block usage. Inodes are data structures that store information about files (metadata like permissions, ownership, etc.). Each file consumes one inode, regardless of its size. This output is critical when:
- Dealing with systems containing many small files
- Troubleshooting "no space left on device" errors despite available disk space
- Analyzing filesystems that might run out of inodes before running out of disk space
The columns show total inodes, used inodes, free inodes, and the percentage of used inodes.

**5. Filesystem Type Display**
```bash
df -T
```
**Sample Output:**
```
Filesystem     Type     1K-blocks    Used Available Use% Mounted on
/dev/sda1      ext4      20510332 3678864  15773316  19% /
tmpfs          tmpfs        16384       0     16384   0% /dev/shm
/dev/sdb1      xfs      103080128 2354560 100725568   3% /data
```
**Explanation:** The `-T` flag adds a "Type" column showing the filesystem type for each mounted filesystem. This is valuable for:
- Identifying which filesystems are local vs. network-based
- Applying filesystem-specific maintenance or optimization strategies
- Troubleshooting issues that might be related to specific filesystem types
In this example, we can see the system has ext4, tmpfs, and xfs filesystems, each with different characteristics and optimal use cases.

**6. Filtering by Filesystem Type**
```bash
df -t ext4
```
**Sample Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
```
**Explanation:** The `-t` flag filters the output to show only filesystems of the specified type. This is particularly useful in environments with multiple filesystem types when you want to:
- Focus on a specific filesystem type for maintenance
- Isolate issues to particular filesystem types
- Generate reports for specific filesystem categories
You can specify multiple types by repeating the flag (e.g., `df -t ext4 -t xfs`).

**7. Excluding Filesystem Types**
```bash
df -x tmpfs -x devtmpfs
```
**Sample Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
/dev/sdb1      103080128 2354560 100725568   3% /data
```
**Explanation:** The `-x` flag excludes filesystems of the specified type from the output. This is helpful for:
- Filtering out pseudo-filesystems that aren't real storage
- Focusing on persistent storage filesystems
- Creating cleaner reports by excluding system-specific filesystems
In this example, we've excluded tmpfs and devtmpfs, which are virtual filesystems in memory, to focus only on physical storage.

### Custom Output Examples

**8. Custom Output Fields**
```bash
df --output=source,size,used,avail,pcent,target
```
**Sample Output:**
```
Filesystem     Size  Used Avail Use% Mounted on
/dev/sda1       20G  3.5G   15G  19% /
/dev/sdb1       99G  2.2G   96G   3% /data
```
**Explanation:** The `--output` flag allows you to customize which fields are displayed and in what order. Available fields include:
- `source`: Filesystem device name
- `size`: Total size
- `used`: Used space
- `avail`: Available space
- `pcent`: Usage percentage
- `target`: Mount point
- `itotal`: Total inodes
- `iused`: Used inodes
- `iavail`: Available inodes
- `ipcent`: Inode usage percentage
- `fstype`: Filesystem type
This customization is valuable for:
- Creating focused reports
- Extracting specific data for processing in scripts
- Integrating with monitoring systems that require specific data formats

**9. Combining Multiple Options**
```bash
df -hT --type=ext4 --output=source,fstype,size,used,avail,pcent
```
**Sample Output:**
```
Filesystem     Type  Size  Used Avail Use%
/dev/sda1      ext4   20G  3.5G   15G  19%
```
**Explanation:** This example demonstrates how multiple options can be combined for highly customized output. Here we've:
- Used `-h` for human-readable format
- Added `-T` to show filesystem type
- Filtered with `--type=ext4` to show only ext4 filesystems
- Specified exact output fields with `--output`
This combination is powerful for creating targeted reports or extracting specific information for system analysis and monitoring.

### Advanced Usage Examples

**10. Checking Specific Directory's Filesystem**
```bash
df -h /var/log
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
```
**Explanation:** When you specify a file or directory path as an argument, `df` shows information only for the filesystem that contains that path. This is useful when:
- You need to check available space for a specific application directory
- You're troubleshooting space issues in a particular location
- You want to know which filesystem a directory resides on
Note that this shows the entire filesystem's usage, not just the usage of the specified directory.

**11. Including All Filesystems**
```bash
df -a
```
**Sample Output:**
```
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sda1       20510332 3678864  15773316  19% /
proc                  0       0        0    - /proc
sysfs                 0       0        0    - /sys
devtmpfs           16384       0     16384   0% /dev
tmpfs            16384        0     16384   0% /dev/shm
devpts               0       0        0    - /dev/pts
securityfs           0       0        0    - /sys/kernel/security
tmpfs            16384       0     16384   0% /run
tmpfs            16384       0     16384   0% /sys/fs/cgroup
cgroup               0       0        0    - /sys/fs/cgroup/systemd
pstore               0       0        0    - /sys/fs/pstore
cgroup               0       0        0    - /sys/fs/cgroup/cpu,cpuacct
cgroup               0       0        0    - /sys/fs/cgroup/memory
cgroup               0       0        0    - /sys/fs/cgroup/freezer
cgroup               0       0        0    - /sys/fs/cgroup/pids
cgroup               0       0        0    - /sys/fs/cgroup/net_cls,net_prio
systemd-1            -       -        -    - /proc/sys/fs/binfmt_misc
debugfs              0       0        0    - /sys/kernel/debug
mqueue               0       0        0    - /dev/mqueue
hugetlbfs            0       0        0    - /dev/hugepages
fusectl             0       0        0    - /sys/fs/fuse/connections
/dev/sdb1      103080128 2354560 100725568   3% /data
```
**Explanation:** The `-a` flag includes all filesystems, including pseudo-filesystems like proc, sysfs, and cgroup. These virtual filesystems don't represent actual storage but provide interfaces to kernel data structures. This option is primarily used for:
- Complete system auditing
- Troubleshooting when you need to see all mount points
- Understanding the full filesystem hierarchy
Note that most pseudo-filesystems show 0 blocks, as they don't consume actual disk space.

**12. Local Filesystems Only**
```bash
df -l -h
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
/dev/sdb1        99G  2.2G   96G   3% /data
```
**Explanation:** The `-l` flag restricts output to local filesystems only, excluding network filesystems like NFS, SMB/CIFS, etc. This is useful when:
- You want to focus on physical storage directly attached to the system
- Network filesystems might be slow to respond
- You're troubleshooting local storage issues
- You're generating reports about local capacity
Combined with `-h` for human-readable format, this provides a clean view of local storage resources.

### Scripting and Automation Examples

**13. Checking for High Usage**
```bash
df -h | awk 'NR>1 {gsub(/%/,""); if ($5 > 80) print "Warning: " $1 " is " $5 "% full on " $6}'
```
**Sample Output:**
```
Warning: /dev/sda1 is 85% full on /
```
**Explanation:** This one-liner combines `df` with `awk` to check for filesystems with usage exceeding 80%. It:
- Uses `df -h` to get human-readable output
- Pipes to `awk` for processing
- Skips the header row with `NR>1`
- Removes the % sign with `gsub`
- Checks if usage (5th column) exceeds 80
- Prints a warning message if the threshold is exceeded
This type of command is commonly used in:
- Monitoring scripts
- Cron jobs for automated alerts
- Health check systems
- Capacity planning tools

**14. Exporting to CSV Format**
```bash
df -h --output=source,size,used,avail,pcent,target | awk 'NR>1 {print $1","$2","$3","$4","$5","$6}' > disk_usage.csv
```
**Sample Output (in disk_usage.csv file):**
```
/dev/sda1,20G,3.5G,15G,19%,/
/dev/sdb1,99G,2.2G,96G,3%,/data
```
**Explanation:** This command exports disk usage information to CSV format for further analysis or reporting. It:
- Uses `df` with custom output fields
- Pipes to `awk` to format as CSV
- Skips the header row
- Combines fields with commas
- Redirects output to a file
This technique is valuable for:
- Generating reports for management
- Importing data into spreadsheet applications
- Feeding data to monitoring systems
- Creating historical records for capacity planning

**15. Finding the Largest Filesystem**
```bash
df -h --output=size,target | sort -hr | head -1
```
**Sample Output:**
```
99G /data
```
**Explanation:** This command identifies the largest filesystem by size. It:
- Uses `df` with custom output showing only size and mount point
- Pipes to `sort -hr` for human-readable reverse sort (largest first)
- Uses `head -1` to get only the top result
This type of command is useful for:
- Quickly identifying where the most storage space is available
- Scripting that needs to determine the best location for large files
- Automated system management tasks
- Capacity analysis and planning

### Real-World Scenarios

**16. Checking Root Filesystem Space**
```bash
df -h /
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
```
**Explanation:** Checking the root filesystem specifically is important because:
- System updates often require space in / or subdirectories
- Many applications install to /usr or /opt
- Log files in /var/log can grow and fill the root filesystem
- Running out of root filesystem space can cause system instability
This command provides a focused view of the most critical filesystem on most Linux systems.

**17. Checking User Home Directories**
```bash
df -h /home
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        99G  2.2G   96G   3% /home
```
**Explanation:** Checking the /home filesystem is important because:
- User files typically consume the most storage
- Running out of home directory space affects user productivity
- It helps identify if user quotas need adjustment
- It can indicate if users need storage management education
In this example, we can see that the /home filesystem has plenty of available space, which is ideal for user environments.

**18. Checking Temporary Directory Space**
```bash
df -h /tmp
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        20G  3.5G   15G  19% /
```
**Explanation:** Checking /tmp space is crucial because:
- Many applications use /tmp for temporary files during processing
- Package managers often use /tmp during software installation
- Large file operations may require temporary space
- Insufficient /tmp space can cause application failures
In this example, /tmp is part of the root filesystem, which has adequate space.

**19. Checking Application Data Directories**
```bash
df -h /var/lib/mysql
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdb1        99G  2.2G   96G   3% /var/lib/mysql
```
**Explanation:** Checking application-specific directories like /var/lib/mysql is important because:
- Database storage requirements can grow rapidly
- Application performance can degrade as storage fills up
- Running out of database storage can cause data corruption
- Planning for database growth is essential for system stability
This command helps database administrators monitor their storage usage effectively.

**20. Checking Network Storage**
```bash
df -h -t nfs4 -t cifs
```
**Sample Output:**
```
Filesystem      Size  Used Avail Use% Mounted on
server1:/data   500G  350G  150G  70% /mnt/data
server2:/backup 1.0T  600G  400G  60% /mnt/backup
```
**Explanation:** Checking network filesystems is important because:
- Network storage might be shared among multiple systems
- Network storage performance can degrade as it fills up
- Network storage quotas might be different from local storage
- Network storage availability depends on network connectivity
This example shows how to filter for NFS and CIFS filesystems specifically, which is useful in environments with mixed local and network storage.

### Troubleshooting Common Issues

**1. Mount Point Resolution**
```bash
df /var/log/
```
Shows space for the filesystem containing `/var/log`, not the directory itself.

**2. Handling Stale Mount Points**
```bash
df -a | grep -v proc
```
Filters pseudo-filesystems while including potentially stale network mounts.

**3. Capacity Planning**
```bash
df --output=source,size,used,avail,pcent | sort -k5 -hr
```
Sorts filesystems by usage percentage for prioritized attention.

### Performance Considerations
- Use `-l` for local filesystems only to improve response time
- Combine with `timeout` commands for network filesystems
- Cache results in monitoring systems to reduce system load

## Conclusion
The `df` command remains an indispensable tool for system administrators and DevOps engineers. Mastering its options and understanding output interpretation enables effective capacity planning, troubleshooting, and performance optimization. Regular monitoring combined with automated alerting forms the foundation of robust storage management strategies in both traditional and cloud-native environments.

*Professional Tip: Integrate `df` output with centralized monitoring solutions like Prometheus for historical trend analysis and predictive capacity planning.*