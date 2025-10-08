# nl command

2025-10-01 03:38
Status: #DONE 
Tags: [[Linux]]

---
### Comprehensive Guide to the `nl` Command in Linux

As an LPIC instructor and DevOps-focused Medium blog writer, I often highlight the `nl` command as a specialized yet powerful tool for numbering lines in text files. Part of the GNU coreutils package, `nl` (number lines) is designed to add line numbers to files or standard input, offering greater flexibility than similar commands like `cat -n` or `cat -b`. Its customization options make it invaluable for formatting output in scripts, documentation, or log analysis, particularly in DevOps workflows where precise line referencing is crucial. In this article, I’ll provide an in-depth exploration of the `nl` command, covering its syntax, options, arguments, and practical applications. I’ll include detailed examples, best practices, and tips to maximize its utility, concluding with a summary table of options for quick reference.

#### Basic Syntax and Core Functionality

The `nl` command reads files or standard input, adds line numbers according to specified rules, and outputs the result to standard output (stdout). Its general syntax is:

```bash
nl [OPTION]... [FILE]...
```

- **Purpose**: Adds line numbers to text lines, with customizable formatting and conditions for numbering.
- **Core Behavior**: Numbers lines based on default or user-defined styles, skipping blank lines or specific patterns if configured.
- **Input**: Accepts one or more files, or standard input if no file is specified or `-` is used.
- **Output**: Writes numbered output to stdout, suitable for display or redirection.

For example, to number lines in a file `example.txt`:

```bash
nl example.txt
```

If `example.txt` contains:

```
First line
Second line

Third line
```

The default output is:

```
     1  First line
     2  Second line
       
     3  Third line
```

By default, `nl` numbers non-blank lines, aligns numbers right-justified in a 6-character field with padding, and uses a tab (`\t`) as the separator between numbers and text.

**Tip**: Redirect output to a new file for formatted documentation:

```bash
nl example.txt > numbered_example.txt
```

#### Detailed Explanation of Options

The `nl` command supports a range of options to customize numbering style, format, and behavior. Options are typically single-dash (`-`) or double-dash (`--`), and some allow fine-grained control over headers, bodies, and footers. Below, I detail each option with examples and use cases, focusing on the GNU implementation.

1. **-b STYLE, --body-numbering=STYLE**:
   - Specifies which lines in the body section to number. Options include:
     - `a`: Number all lines, including blanks.
     - `t` (default): Number non-blank lines only.
     - `n`: Number no lines (disables numbering for the body).
     - `pREGEXP`: Number only lines matching the regular expression `REGEXP`.
   - Example: Number all lines, including blanks, in `example.txt`:

     ```bash
     nl -b a example.txt
     ```

     Output:

     ```
          1  First line
          2  Second line
          3  
          4  Third line
     ```

   - Example: Number only lines containing “line”:

     ```bash
     nl -b p'line' example.txt
     ```

     Output:

     ```
          1  First line
          2  Second line
             
          3  Third line
     ```

   - **Best Practice**: Use `-b pREGEXP` for selective numbering in logs, e.g., `nl -b p'ERROR' logfile.txt` to highlight error lines.

2. **-d CC, --section-delimiter=CC**:
   - Defines the delimiter for section markers (default: `\:`, escaped as `\\:`). Used to identify header, body, and footer sections.
   - Example: For a file `sections.txt` with custom delimiters `@@`:

     ```
     @@header@@
     Title
     @@body@@
     Content
     @@footer@@
     End
     ```

     ```bash
     nl -d @@ sections.txt
     ```

     Numbers only the body section.

   - **Tip**: Use custom delimiters for structured documents like reports or scripts with embedded sections.

3. **-f STYLE, --footer-numbering=STYLE**:
   - Specifies numbering style for footer sections (same styles as `-b`).
   - Example: Number footer lines only:

     ```bash
     nl -f a sections.txt
     ```

   - **Trick**: Rarely used standalone; combine with `-h` and `-b` for full document control.

4. **-h STYLE, --header-numbering=STYLE**:
   - Specifies numbering style for header sections (same styles as `-b`).
   - Example: Number header lines:

     ```bash
     nl -h a sections.txt
     ```

   - **Best Practice**: Use for formatting structured documentation with numbered headers.

5. **-i NUM, --line-increment=NUM**:
   - Sets the increment between line numbers (default: 1).
   - Example: Number every other line:

     ```bash
     nl -i 2 example.txt
     ```

     Output:

     ```
          1  First line
          3  Second line
             
          5  Third line
     ```

   - **Trick**: Use `-i` for sampling large datasets, e.g., `nl -i 10 data.txt` to number every 10th line.

6. **-l NUM, --join-blank-lines=NUM**:
   - Treats `NUM` consecutive blank lines as a single line for numbering purposes.
   - Example: For `sparse.txt`:

     ```
     Line 1
     
     
     Line 2
     ```

     ```bash
     nl -l 2 sparse.txt
     ```

     Output:

     ```
          1  Line 1
             
          2  Line 2
     ```

   - **Best Practice**: Use `-l` to clean up sparse logs for compact numbering.

7. **-n FORMAT, --number-format=FORMAT**:
   - Sets the format of line numbers:
     - `ln`: Left-justified, no leading zeros.
     - `rn` (default): Right-justified, no leading zeros.
     - `rz`: Right-justified, leading zeros.
   - Example: Use left-justified numbers:

     ```bash
     nl -n ln example.txt
     ```

     Output:

     ```
     1    First line
     2    Second line
          
     3    Third line
     ```

   - Example: Use leading zeros:

     ```bash
     nl -n rz -w 4 example.txt
     ```

     Output:

     ```
     0001 First line
     0002 Second line
          
     0003 Third line
     ```

   - **Tip**: Use `-n rz` for fixed-width numbering in reports or logs.

8. **-p, --no-renumber**:
   - Prevents resetting the line number at logical page boundaries (e.g., new sections).
   - Example: Continuous numbering across sections:

     ```bash
     nl -p sections.txt
     ```

   - **Best Practice**: Use `-p` for consistent numbering in multi-section documents.

9. **-s STRING, --number-separator=STRING**:
   - Sets the separator between line numbers and text (default: tab, `\t`).
   - Example: Use a colon separator:

     ```bash
     nl -s ": " example.txt
     ```

     Output:

     ```
          1: First line
          2: Second line
               
          3: Third line
     ```

   - **Trick**: Use `-s` for custom formatting in documentation, e.g., `-s " - "` for a clean dash separator.

10. **-v NUM, --starting-line-number=NUM**:
    - Sets the starting line number (default: 1).
    - Example: Start numbering at 10:

      ```bash
      nl -v 10 example.txt
      ```

      Output:

      ```
         10  First line
         11  Second line
              
         12  Third line
      ```

    - **Tip**: Use `-v` for appending numbered output to existing documents with prior numbering.

11. **-w NUM, --number-width=NUM**:
    - Sets the width of the line number field (default: 6).
    - Example: Use a 3-character field:

      ```bash
      nl -w 3 example.txt
      ```

      Output:

      ```
       1  First line
       2  Second line
            
       3  Third line
      ```

    - **Best Practice**: Adjust `-w` for alignment in narrow or wide outputs.

12. **--help**:
    - Displays a help message summarizing options.
    - Example:

      ```bash
      nl --help
      ```

      Outputs a concise guide.

13. **--version**:
    - Shows the `nl` version (GNU coreutils).
    - Example:

      ```bash
      nl --version
      ```

      Useful for compatibility checks.

#### Advanced Usage in DevOps and Scripting

In DevOps, `nl` is ideal for formatting logs, generating reports, or preparing numbered outputs for debugging or documentation. Examples include:

1. **Log Analysis**:
   - Number error lines in a log:

     ```bash
     grep "ERROR" logfile.txt | nl -b p'ERROR'
     ```

     Only lines with “ERROR” are numbered.

2. **Scripting**:
   - Generate a numbered configuration file:

     ```bash
     nl -s ": " -w 2 config.txt > numbered_config.txt
     ```

     Creates a cleanly formatted output for documentation.

3. **Pipelining**:
   - Number lines in a pipeline:

     ```bash
     dmesg | nl -b a > numbered_dmesg.txt
     ```

     Numbers all kernel messages for reference.

4. **Structured Documents**:
   - Number sections of a report:

     ```bash
     nl -d @@ -b a report.txt
     ```

     Numbers body lines within custom sections.

**Trick**: Combine with `awk` for complex formatting:

```bash
nl -b a data.txt | awk '{printf "%-10s%s\n", $1, $2}'
```

This aligns numbers and text precisely.

#### Potential Pitfalls and Security Considerations

- **Large Files**: `nl` processes files sequentially, so it’s efficient but may lag with massive files. Pipe through `head` or `tail` for subsets.
- **Binary Files**: `nl` is designed for text. For binaries, use `strings | nl` to avoid garbage output.
- **Regex Errors**: Invalid `-b pREGEXP` patterns cause `nl` to skip numbering. Test regex with `grep` first.
- **Security**: Avoid untrusted files, as malformed input could disrupt output formatting. Validate inputs in scripts.

**Tip**: Check file existence before processing:

```bash
[ -f "file.txt" ] && nl file.txt || echo "File not found"
```

#### Best Practices and Tips

- **Use `-b pREGEXP` for Selective Numbering**: Ideal for highlighting specific lines in logs or reports.
- **Customize Separators**: Use `-s` for readable output in documentation.
- **Adjust Width with `-w`**: Match number width to content size for alignment.
- **Continuous Numbering with `-p`**: Ensures consistency across sections.
- **Pipe with Other Tools**: Combine with `grep`, `awk`, or `sed` for advanced processing.

**Trick**: Create a numbered backup of a log:

```bash
nl -b a -s ": " logfile.txt | tee numbered_log.txt
```

This displays and saves the numbered output.

#### Summary Table of `nl` Options

| Option               | Long Form                    | Description                                                                 | Example Use Case                          |
|----------------------|------------------------------|-----------------------------------------------------------------------------|-------------------------------------------|
| -b STYLE            | --body-numbering=STYLE      | Sets body numbering: `a` (all), `t` (non-blank, default), `n` (none), `pREGEXP` | Selective log line numbering             |
| -d CC               | --section-delimiter=CC      | Sets section delimiter (default: `\:`)                                      | Structured document processing           |
| -f STYLE            | --footer-numbering=STYLE    | Sets footer numbering style (same as `-b`)                                  | Numbering report footers                 |
| -h STYLE            | --header-numbering=STYLE    | Sets header numbering style (same as `-b`)                                  | Numbering document headers               |
| -i NUM              | --line-increment=NUM        | Sets line number increment (default: 1)                                    | Sampling large datasets                  |
| -l NUM              | --join-blank-lines=NUM      | Treats NUM blank lines as one for numbering                                 | Cleaning sparse logs                     |
| -n FORMAT           | --number-format=FORMAT      | Sets number format: `ln` (left), `rn` (right, default), `rz` (right, zeros) | Fixed-width numbering                    |
| -p                  | --no-renumber               | Prevents resetting line numbers at section boundaries                      | Continuous numbering across sections     |
| -s STRING           | --number-separator=STRING   | Sets separator between number and text (default: tab)                      | Custom formatting for reports            |
| -v NUM              | --starting-line-number=NUM  | Sets starting line number (default: 1)                                      | Appending to existing numbered documents |
| -w NUM              | --number-width=NUM          | Sets width of number field (default: 6)                                    | Aligning numbers in output               |
| --help              | (none)                      | Displays help message                                                      | Quick option reference                   |
| --version           | (none)                      | Shows version information                                                  | Compatibility checks                     |
