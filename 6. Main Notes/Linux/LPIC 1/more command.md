# more command

2025-10-01 03:30
Status: #DONE 
Tags: [[Linux]]

---
### Comprehensive Guide to the `more` Command in Linux

As an LPIC instructor and DevOps-focused Medium blog writer, I often highlight the `more` command as a fundamental tool for navigating and inspecting text files in Linux. While it may seem overshadowed by its more feature-rich cousin `less`, the `more` command remains a lightweight, widely available utility for viewing file contents page by page. It’s particularly useful in minimal environments, such as embedded systems or containers, and for quick file inspections in scripts or interactive shells. In this article, I’ll dive deep into the `more` command, covering its syntax, options, interactive commands, and practical applications. I’ll provide detailed examples, best practices, and tips to maximize its utility in DevOps workflows, concluding with a summary table of its options for quick reference.

#### Basic Syntax and Core Functionality

The `more` command displays text files or standard input one screen at a time, allowing users to scroll through content interactively. Its general syntax is:

```bash
more [OPTION]... [FILE]...
```

- **Purpose**: Reads files or input and paginates output, pausing after each screenful (based on terminal size).
- **Core Behavior**: Displays content sequentially, with basic navigation (spacebar to advance, `q` to quit).
- **Input**: Can take one or more files, or read from standard input (e.g., via a pipe).
- **Output**: Sends to standard output (stdout), typically the terminal.

For example, to view a file named `logfile.txt`:

```bash
more logfile.txt
```

If `logfile.txt` contains hundreds of lines, `more` shows the first screenful (e.g., 24 lines in a standard terminal) and pauses with a prompt like `:`, waiting for user input. Pressing the spacebar advances to the next screen, while `q` exits.

To read from standard input, pipe output to `more`:

```bash
ls -l /etc | more
```

This paginates the directory listing, making it easier to read long outputs.

**Tip**: Use `more` for quick inspections when `less` isn’t available, such as in minimal Docker images or recovery environments.

#### Detailed Explanation of Options

The `more` command (GNU coreutils version) supports several options to modify its behavior. Options are typically single-dash (`-`), though some implementations vary (e.g., BSD). Below, I detail each option with examples and use cases. Note that options are case-sensitive and must precede file names.

1. **-d**:
   - Displays user-friendly prompts, including instructions like `[Press space to continue, 'q' to quit.]` and warnings for invalid keys.
   - Example: View a file with helpful prompts:

```bash
     more -d logfile.txt
```

     Output shows a clear prompt at the bottom, guiding novice users.

   - **Best Practice**: Enable `-d` in interactive sessions or training environments to improve usability.

2. **-f**:
   - Counts logical lines (based on newline characters) rather than physical lines wrapped by terminal width. Useful for files with long lines, like CSV files.
   - Example: For a file `data.csv` with lines exceeding terminal width:

```bash
     more -f data.csv
```

     Without `-f`, `more` counts wrapped lines, skewing line-based navigation. With `-f`, it respects newlines.

   - **Tip**: Use `-f` when analyzing structured data (e.g., logs or CSVs) to maintain line integrity.

3. **-l**:
   - Ignores form feed characters (`^L`, ASCII 12), treating them as regular characters instead of page breaks. Useful for files with embedded form feeds.
   - Example: If `report.txt` contains `^L` to separate sections:

```bash
     more -l report.txt
```

     `more` displays `^L` as text rather than forcing a page break.

   - **Trick**: Combine with `cat -v` to visualize form feeds before piping: `cat -v report.txt | more -l`.

4. **-p**:
   - Clears the screen before displaying each page, providing a cleaner view.
   - Example:

```bash
     more -p logfile.txt
```

     Each page starts fresh, reducing visual clutter.

   - **Best Practice**: Use `-p` in terminals with heavy scrollback to focus on current content.

5. **-c**:
   - Similar to `-p`, but paints each screen from the top without scrolling, reducing flicker on slow terminals.
   - Example:

```bash
     more -c logfile.txt
```

     Content is redrawn cleanly for each page.

   - **Tip**: Prefer `-c` in environments with older terminals or slow connections (e.g., SSH over high-latency networks).

6. **-s**:
   - Squeezes multiple blank lines into a single blank line, similar to `cat -s`. Reduces clutter in sparse files.
   - Example: For a file `sparse.txt` with:

```
     Line 1
     
     
     
     Line 2
```

```bash
     more -s sparse.txt
```
Outputs:
```
     Line 1
     
     Line 2
```

   - **Best Practice**: Use `-s` for logs or documentation with excessive blank lines to save screen space.

7. **-u**:
   - Disables underlining and bold formatting, treating escape sequences as plain text. Useful for files with ANSI codes or markup.
   - Example: If `styled.txt` contains underlined text (e.g., via ANSI codes):

```bash
     more -u styled.txt
```

     Shows raw codes instead of rendering effects.

   - **Trick**: Pipe through `cat -v` to inspect escape sequences: `cat -v styled.txt | more -u`.

8. **-+NUM**:
   - Starts displaying from line number `NUM`.
   - Example: Start at line 50 of `logfile.txt`:

```bash
     more +50 logfile.txt
```

     The first screen begins at line 50.

   - **Tip**: Use with logs to jump to known error points: `more +$(grep -n "ERROR" logfile.txt | cut -d: -f1 | head -1) logfile.txt`.

9. **-+/"PATTERN"**:
   - Starts at the first line matching the pattern (case-sensitive).
   - Example: Jump to the first occurrence of “ERROR”:

```bash
     more +/"ERROR" logfile.txt
```

     Displays from the matching line onward.

   - **Trick**: Combine with `-d` for a user-friendly experience when searching: `more -d +/"ERROR" logfile.txt`.

10. **--help**:
    - Displays a help message summarizing options and usage.
    - Example:

      ```bash
      more --help
      ```

      Outputs a concise guide.

11. **--version**:
    - Shows the version of `more` (e.g., from GNU coreutils).
    - Example:

      ```bash
      more --version
      ```

      Useful for confirming compatibility in mixed environments.

#### Interactive Commands in `more`

Once `more` displays a file, it enters an interactive mode with a prompt (e.g., `:` or `--More--`). Users can navigate using key commands:

- **Spacebar**: Advances one screenful.
- **Enter**: Advances one line.
- **b**: Moves back one screenful (not supported in all implementations).
- **q**: Quits `more`.
- **h**: Displays help with all interactive commands.
- **/PATTERN**: Searches forward for `PATTERN`. Press `n` to find the next match.
- **=**: Shows the current line number.
- **v**: Opens the file in the default editor (defined by `$VISUAL` or `$EDITOR`, typically `vi`).
- **!COMMAND**: Runs a shell command (e.g., `!ls` lists the directory).
- **:n**: Moves to the next file (if multiple files were specified).
- **:p**: Moves to the previous file.
- **:f**: Displays the current file name and line number.

**Example**: While viewing `logfile.txt`, type `/ERROR` at the prompt to jump to “ERROR”. Press `n` to find the next match, then `q` to exit.

**Best Practice**: Learn the `h` command for quick reference during interactive use, especially in training or debugging sessions.

#### Advanced Usage in DevOps and Scripting

In DevOps, `more` is useful for inspecting logs, configuration files, or pipeline outputs. Here are practical scenarios:

1. **Piping Logs**:
   - Paginate large log outputs:

     ```bash
     journalctl -u nginx.service | more
     ```

     Allows scrolling through system logs without overwhelming the terminal.

2. **Multiple Files**:
   - View multiple logs sequentially:

     ```bash
     more /var/log/app1.log /var/log/app2.log
     ```

     Use `:n` to switch between files.

3. **Script Integration**:
   - Display script output interactively:

     ```bash
     ./generate_report.sh | more -s
     ```

     The `-s` option cleans up blank lines for readability.

4. **Debugging Pipelines**:
   - Inspect intermediate pipeline output:

     ```bash
     cat /var/log/syslog | grep "error" | more -d
     ```

     The `-d` option makes navigation intuitive.

**Trick**: Set `$PAGER` to `more` in minimal environments:

```bash
export PAGER=more
```

This ensures tools like `man` use `more` when `less` is unavailable.

#### Potential Pitfalls and Security Considerations

- **Large Files**: `more` loads content sequentially but can be slow for very large files. Consider `less` for better performance.
- **Binary Files**: `more` may garble terminals with binary data. Use `more -u` or `cat -v | more` to mitigate.
- **Interactive Shells**: Avoid `more` in non-interactive scripts, as it expects user input. Use `head` or `tail` instead.
- **Security**: Be cautious with untrusted files, as escape sequences could execute terminal commands. Use `-u` to disable rendering.

**Tip**: For binary files, pipe through `strings` first:

```bash
strings binaryfile | more
```

#### Best Practices and Tips

- **Use `-d` for Novices**: Improves usability with clear prompts.
- **Combine with Pipes**: Leverage `more` in pipelines for readable output.
- **Prefer `less` for Advanced Needs**: If available, `less` offers more features (e.g., backward navigation).
- **Set Environment Variables**: Configure `$VISUAL` or `$EDITOR` for seamless editing with `v`.
- **Error Handling in Scripts**: Redirect stderr to avoid `more` hanging:

  ```bash
  more nonexistent.txt 2>/dev/null || echo "File not found"
  ```

**Trick**: Use `more` with `find` to browse multiple files:

```bash
find /etc -name "*.conf" | xargs more
```

This paginates all `.conf` files sequentially.

#### Summary Table of `more` Options

| Option       | Description                                                | Example Use Case                       |
| ------------ | ---------------------------------------------------------- | -------------------------------------- |
| -d           | Shows user-friendly prompts with navigation instructions   | Training or interactive sessions       |
| -f           | Counts logical lines, ignoring terminal wrapping           | Viewing CSV or long-line files         |
| -l           | Ignores form feed (`^L`) characters                        | Files with embedded page breaks        |
| -p           | Clears screen before each page                             | Cleaner display in cluttered terminals |
| -c           | Paints screen from top, avoiding scroll flicker            | Slow or remote terminals               |
| -s           | Squeezes multiple blank lines into one                     | Sparse logs or documentation           |
| -u           | Disables underlining/bold, treats escape sequences as text | Files with ANSI codes                  |
| -+NUM        | Starts at line number NUM                                  | Jumping to specific log lines          |
| -+/"PATTERN" | Starts at first line matching PATTERN                      | Searching for errors in logs           |
| --help       | Displays help message                                      | Quick reference for options            |
| --version    | Shows version information                                  | Checking compatibility in environments |
