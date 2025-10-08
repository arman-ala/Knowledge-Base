# whereis command

2025-10-01 09:58
Status: #DONE 
Tags: [[Linux]]

---
The `whereis` command in Linux is a utility for quickly locating the binary, source, and manual page files for a given command by searching through a predefined set of directories .

### 🔍 What is the `whereis` Command?

The `whereis` command is designed specifically to find the essential components of a command or utility. Its key advantage is speed, as it only searches through standard system directories like `/usr/bin`, `/usr/share/man`, and others, instead of scanning the entire filesystem .

The basic syntax is straightforward:
```bash
whereis [options] command_name
```

For example, to find all related files for the `ls` command, you would use:
```bash
whereis ls
```
A typical output might look like: `ls: /bin/ls /usr/share/man/man1/ls.1.gz`, showing the location of the binary and its manual page .

### ⚙️ Command Options and Usage

The behavior of `whereis` can be finely tuned using its options. The table below summarizes the most common ones.

| **Option** | **Description** | **Example Use Case** |
| :--- | :--- | :--- |
| `-b` | Searches only for binary (executable) files . | `whereis -b ls` |
| `-m` | Searches only for manual (documentation) pages . | `whereis -m tar` |
| `-s` | Searches only for source files . | `whereis -s gcc` |
| `-u` | Searches for unusual entries (commands with missing or multiple file types) . | `whereis -u -m *` |
| `-B` | Limits the directories to search for binaries. Requires `-f` . | `whereis -B /bin -f ls` |
| `-M` | Limits the directories to search for manuals. Requires `-f` . | `whereis -M /usr/share/man/man1 -f cal` |
| `-S` | Limits the directories to search for sources. Requires `-f` . | `whereis -S /usr/src -f *` |
| `-l` | Lists the hard-coded paths that `whereis` searches . | `whereis -l` |

Here are practical examples of how to use these options:

- **Finding Specific File Types**
  To locate only the binary of a command:
  ```bash
  whereis -b python
  ```
  To find only its manual page:
  ```bash
  whereis -m python
  ```

- **Searching in Specific Directories**
  You can restrict the search to specific locations. The `-f` option must be used to signal the end of the directory list and the start of filenames .
  ```bash
  whereis -b -B /usr/bin -f ls
  ```

- **Finding Unusual Entries**
  To find commands in the current directory that are missing documentation, you can use:
  ```bash
  whereis -m -u *
  ```
  A command is considered "unusual" if it does not have exactly one entry of each requested type (binary, manual, or source) .

- **Viewing Search Paths**
  To see all the directories that `whereis` scans, use the `-l` (list) option :
  ```bash
  whereis -l
  ```

### ➡️ `whereis` vs. `which` and Other Tools

Understanding how `whereis` complements other search commands is crucial for choosing the right tool.

- **`whereis` vs. `which`**: While `which` only shows the path of the executable that will run in your current shell (honoring your `$PATH`), `whereis` provides a more comprehensive view, including binaries, source files, and manual pages from a hard-coded list of paths .
- **`whereis` vs. `find` and `locate`**: The `find` command is powerful and recursive but can be slow as it scans the live filesystem. The `locate` command uses a database for speed but might not have the most recent file information. `whereis` is the fastest for finding components of system commands but is limited to its predefined paths .

### 💡 Best Practices and Considerations

- **Limitation**: The main limitation of `whereis` is its hard-coded path . If a program is installed in a non-standard location (like `/opt` or a user's home directory), `whereis` will likely not find it. In such cases, `find` or `locate` are better alternatives.
- **Scripting**: For scripting, where you often only need to verify the existence and location of an executable, the `which` command might be more appropriate.
- **Troubleshooting**: If `whereis` returns no results for a command you know is installed, try using the `-l` flag to see its search paths or use a more comprehensive tool like `find`.
