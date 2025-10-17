# stat command

2025-10-15 14:54
Status: #DONE 
Tags: [[Linux]]

---
# The stat Command: A Comprehensive Guide for System Administrators and DevOps Professionals

## Abstract

The `stat` command in Unix-like systems is a powerful utility for retrieving detailed metadata about files and directories, including permissions, ownership, timestamps, and filesystem attributes. Essential for Linux Professional Institute Certification (LPIC) candidates and DevOps practitioners, `stat` provides precise, customizable output for auditing, scripting, and troubleshooting. Its versatility makes it invaluable for tasks like verifying file integrity, monitoring changes, or automating system checks in CI/CD pipelines. This article offers an in-depth exploration of `stat`, covering its syntax, options, practical applications, and best practices, with a focus on its role in system administration and DevOps workflows.

## Introduction

Understanding file and directory metadata is critical for managing Linux systems, where attributes like ownership, permissions, and timestamps dictate access and behavior. The `stat` command provides a detailed view of these attributes, offering more granularity than `ls -l` and supporting customizable output formats for scripting. Available across Unix-like systems, including Linux distributions like Ubuntu, CentOS, and Debian, `stat` is a lightweight, POSIX-compliant tool that integrates seamlessly into automation workflows and manual diagnostics.

As an LPIC instructor and DevOps practitioner, I rely on `stat` for tasks like verifying file ownership in containerized environments, checking modification times for backup validation, or extracting inode details for filesystem analysis. Its ability to format output with `--format` or `--printf` makes it ideal for parsing in scripts, while its filesystem-specific options enhance its utility in complex environments. This guide covers `stat` comprehensively, including edge cases, output customization, and integration with other tools.

## Syntax and Basic Usage

The `stat` command follows this general syntax:

```bash
stat [OPTION]... FILE...
```

Without options, `stat` displays a default set of metadata for the specified files or directories, including file type, permissions, ownership, timestamps, and size. Multiple files can be targeted, and options control output format, content, and behavior (e.g., following symbolic links). The command does not require root privileges for most operations, making it accessible for regular users, though some filesystem-specific details may be restricted.

The output is highly customizable using format strings, allowing administrators to extract specific fields like UID, GID, or access time. Unlike `ls`, which is designed for human-readable listings, `stat` excels in providing machine-parsable data, making it a staple in DevOps automation.

## Command Options

The `stat` command supports a variety of options to tailor its output and behavior. The table below details all standard options, their long forms, and their functionality, based on GNU coreutils and POSIX standards, with notes on variations across systems like BSD.

| Flag | Long Form | Explanation |
|------|-----------|-------------|
| -L   | --dereference | Follows symbolic links, displaying metadata for the target file or directory instead of the link itself. |
| -f   | --file-system | Displays filesystem metadata (e.g., type, block size) for the filesystem containing the file, rather than file-specific details. |
| -c   | --format=FORMAT | Specifies a custom output format using format sequences (e.g., `%U` for user name). Each file’s output is formatted accordingly. |
|      | --printf=FORMAT | Similar to `--format`, but interprets backslash escapes (e.g., `\n` for newlines) for precise output control. |
| -t   | --terse | Outputs a concise, tab-separated format with essential fields (e.g., size, timestamps, permissions), ideal for scripting. |
| -h   | --help | Displays a help message with available options and exits. |
| --version | N/A | Outputs the version of `stat` (e.g., from GNU coreutils) and exits. |

### Format Sequences
The `--format` and `--printf` options use format sequences to customize output. Common sequences include:
- `%a`: Permissions in octal (e.g., `0644`).
- `%A`: Permissions in symbolic notation (e.g., `rw-r--r--`).
- `%u`: User ID (UID).
- `%U`: User name.
- `%g`: Group ID (GID).
- `%G`: Group name.
- `%s`: Size in bytes.
- `%n`: File name.
- `%y`: Last modification time (human-readable).
- `%Y`: Last modification time (seconds since epoch).
- `%x`: Last access time (human-readable).
- `%X`: Last access time (seconds since epoch).
- `%z`: Last status change time (human-readable).
- `%Z`: Last status change time (seconds since epoch).
- `%F`: File type (e.g., `regular file`, `directory`, `symbolic link`).
- `%d`: Device number (major/minor).
- `%i`: Inode number.
- `%b`: Number of 512-byte blocks allocated.
- For filesystem mode (`-f`):
  - `%a`: Free blocks available.
  - `%c`: Total blocks.
  - `%f`: Free inodes.
  - `%t`: Filesystem type in hexadecimal.

## Detailed Examples

Below are practical examples illustrating `stat` in real-world scenarios, with detailed explanations tailored for LPIC-level understanding and DevOps applications. A sample output with a full description of all fields is included.

1. **Default File Metadata**  
   ```bash:disable-run
   stat document.txt
   ```  
   Sample output:  
   ```
     File: document.txt
     Size: 1234            Blocks: 8          IO Block: 4096   regular file
   Device: 8,1    Inode: 123456      Links: 1
   Access: (0644/-rw-r--r--)  Uid: ( 1000/   user)   Gid: ( 1000/   user)
   Access: 2023-10-10 14:30:45.123456789 +0000
   Modify: 2023-10-09 09:15:22.987654321 +0000
   Change: 2023-10-09 09:15:22.987654321 +0000
   Birth: -
   ```  
   **Field Descriptions**:  
   - **File**: Name of the file (`document.txt`).  
   - **Size**: File size in bytes (`1234`).  
   - **Blocks**: Number of 512-byte blocks allocated (`8`).  
   - **IO Block**: Filesystem block size (`4096` bytes).  
   - **regular file**: File type (others include `directory`, `symbolic link`).  
   - **Device**: Device numbers (major 8, minor 1) for the filesystem.  
   - **Inode**: Unique inode number (`123456`).  
   - **Links**: Number of hard links (`1`).  
   - **Access**: Permissions in octal (`0644`) and symbolic (`-rw-r--r--`) notation.  
   - **Uid**: User ID (`1000`) and name (`user`).  
   - **Gid**: Group ID (`1000`) and name (`user`).  
   - **Access**: Last access time (when file was read).  
   - **Modify**: Last modification time (when content changed).  
   - **Change**: Last status change time (when metadata, like permissions, changed).  
   - **Birth**: Creation time (often unsupported, shown as `-`).  
   Use this for auditing file properties in monitoring scripts.

2. **Custom Output Format**  
   ```bash
   stat --format="%n %s %U:%G %A" document.txt
   ```  
   Output: `document.txt 1234 user:user rw-r--r--`.  
   Extracts file name, size, owner:group, and permissions. Ideal for parsing in scripts:  
   ```bash
   stat --format="%s" document.txt | grep -q '1234' && echo "Size matches"
   ```

3. **Terse Output for Scripting**  
   ```bash
   stat -t document.txt
   ```  
   Output: `document.txt 1234 8 4096 8,1 123456 1 0644 1000 1000 ...`.  
   Provides tab-separated fields (name, size, blocks, etc.) for easy parsing in tools like `awk`:  
   ```bash
   stat -t document.txt | awk '{print $2}'  # Prints size (1234)
   ```

4. **Filesystem Metadata**  
   ```bash
   stat -f /var
   ```  
   Sample output:  
   ```
     File: /var
     ID: 1234567890abcdef Namelen: 255     Type: ext4
   Block size: 4096       Fundamental block size: 4096
   Blocks: Total: 5242880    Free: 4123456    Available: 3987654
   Inodes: Total: 1310720    Free: 1298765
   ```  
   Shows filesystem details like type (`ext4`), block size, and free space. Useful for monitoring disk usage in DevOps pipelines.

5. **Following Symbolic Links**  
   ```bash
   stat -L linkfile
   ```  
   Displays metadata for the target of `linkfile` rather than the link. Compare with:  
   ```bash
   stat linkfile
   ```  
   Which shows the link’s metadata (type: `symbolic link`). Critical for auditing linked resources.

6. **Checking Multiple Files**  
   ```bash
   stat *.txt
   ```  
   Lists metadata for all `.txt` files in the current directory. Combine with `--format` for specific fields:  
   ```bash
   stat --format="%n %y" *.txt
   ```  
   Outputs file names and modification times, useful for backup validation.

7. **Using printf for Formatted Output**  
   ```bash
   stat --printf="%n:\t%s bytes\n" document.txt
   ```  
   Output: `document.txt:    1234 bytes`.  
   Adds custom formatting (tabs, newlines) for reports or logs. Example in a script:  
   ```bash
   stat --printf="%n,%s\n" *.txt > file_sizes.csv
   ```

8. **Help and Version**  
   ```bash
   stat --help
   ```  
   Lists options and format sequences.  
   ```bash
   stat --version
   ```  
   Output: `stat (GNU coreutils) 8.32`. Ensures compatibility in mixed environments.

## Best Practices, Tips, and Tricks

- **Custom Formatting for Scripts**: Use `--format` or `--printf` to extract only needed fields, reducing parsing overhead. Example:  
  ```bash
  stat --format="%s" file | grep -q '^[0-9]\+$' || echo "Invalid size"
  ```

- **Terse Mode for Automation**: Prefer `-t` for machine-parsable output in CI/CD pipelines or monitoring tools like Prometheus:  
  ```bash
  stat -t /app/data | cut -f2  # Extract size
  ```

- **Symbolic Link Handling**: Default to `-L` when auditing link targets, but verify link metadata explicitly with `stat` (no `-L`) to avoid confusion.

- **Filesystem Monitoring**: Use `-f` to check free space or inode exhaustion:  
  ```bash
  stat -f / | grep 'Available' | awk '{if ($2 < 100000) print "Low disk space!"}'
  ```

- **Error Handling**: Redirect stderr (`stat file 2>/dev/null`) to handle nonexistent files in scripts. Example:  
  ```bash
  stat -t file.txt 2>/dev/null || echo "File not found"
  ```

- **Portability Note**: GNU `stat` supports `--printf`, but BSD may not. Test with `stat --version` and use `--format` for cross-platform compatibility.

- **Security Auditing**: Regularly check permissions and ownership:  
  ```bash
  stat --format="%A %U:%G" /etc/*.conf
  ```  
  Identifies misconfigured configuration files.

- **Integration with DevOps**: Use `stat` in Dockerfiles or Ansible to verify file attributes:  
  ```yaml
  - name: Check file size
    command: stat --format=%s /app/data
    register: size
    failed_when: size.stdout | int > 1000000
  ```

- **Timestamp Analysis**: Compare `Access`, `Modify`, and `Change` times to detect unauthorized access or modifications:  
  ```bash
  stat --format="%x %y %z" file
  ```

- **Combine with Other Tools**: Pair with `find` for bulk analysis:  
  ```bash
  find /var -type f -exec stat --format="%n %s" {} \;
  ```  
  Lists file names and sizes for auditing.

The `stat` command’s granularity and flexibility make it indispensable for metadata analysis, enabling precise system administration and robust automation in DevOps workflows.
```