# OverTheWire Bandit - Personal Writeup & Learning Journal

> **Note:** All flags have been redacted per OverTheWire guidelines. This writeup focuses on concepts, techniques, and methodology.

---

## Level 0 → Level 1
**Learning Objectives:** Basic SSH connection, understanding port specification

### Challenge
Connect to the Bandit server and retrieve the password for level 1.

### Concepts
- **SSH (Secure Shell):** Protocol for secure remote login
- **Port specification:** Default SSH port is 22, custom ports use `-p` flag
- **Basic file reading:** `cat` command

### Solution
```bash
# Connect to server
ssh -p 2220 bandit0@bandit.labs.overthewire.org

# Once logged in
cat readme
```

### Commands Learned
| Command | Purpose | Syntax |
|---------|---------|--------|
| `ssh` | Secure shell connection | `ssh -p [port] [user]@[host]` |
| `cat` | Concatenate and display files | `cat [filename]` |
| `ls` | List directory contents | `ls [options]` |

### Flag
```
[REDACTED]
```

---

## Level 1 → Level 2
**Learning Objectives:** Special character handling in filenames, understanding stdin/stdout

### Challenge
Read a file literally named `-`

### The Problem
```bash
cat -        # ❌ This hangs - cat interprets '-' as stdin
^C           # Must Ctrl+C to exit
```

### Concepts
**The three faces of the dash (`-`) in Linux:**

1. **stdin/stdout indicator**
```bash
   echo "hello" | cat -    # Read from stdin
```

2. **Option prefix**
```bash
   ls -la                   # Flags/options
```

3. **Literal filename** (our scenario)
```bash
   touch -                  # Creates file named '-'
```

### Solutions

**Method 1: Path prefix (Recommended)**
```bash
cat ./-
```
✅ The `./` forces bash to interpret it as a path, not an argument

**Method 2: Redirection**
```bash
cat < -
```
✅ Input redirection bypasses argument parsing

**Method 3: Full path**
```bash
cat /home/bandit1/-
```
✅ Absolute path leaves no ambiguity

### Why It Matters
This is a common real-world issue. Files can have names like:
- `-` (stdin indicator)
- `--help` (looks like flags)
- Files starting with `-` created by buggy scripts

### Flag
```
[REDACTED]
```

---

## Level 2 → Level 3
**Learning Objectives:** Whitespace handling, shell argument parsing

### Challenge
Read a file named `spaces in this filename`

### The Problem
```bash
cat spaces in this filename
# ❌ Bash interprets this as 5 separate arguments:
# cat [spaces] [in] [this] [filename]
# Result: cat: spaces: No such file or directory
```

### Concepts
- **Word splitting:** Bash uses spaces to separate arguments by default
- **Quoting:** Prevents word splitting
- **Escaping:** Treats special characters literally

### Solutions

**Method 1: Double quotes**
```bash
cat "spaces in this filename"
```
✅ Preserves spaces, allows variable expansion

**Method 2: Single quotes**
```bash
cat 'spaces in this filename'
```
✅ Preserves everything literally (no variable expansion)

**Method 3: Backslash escaping**
```bash
cat spaces\ in\ this\ filename
```
✅ Escapes each space individually

**Pro tip: Tab completion**
```bash
cat spa<TAB>
# Automatically escapes: cat spaces\ in\ this\ filename
```

### When to Use Each Method

| Method | Use When | Example |
|--------|----------|---------|
| Double quotes `"..."` | Filename has spaces, need variable expansion | `cat "$HOME/my file.txt"` |
| Single quotes `'...'` | Preserve everything exactly | `cat 'file with $special chars'` |
| Backslash `\` | Quick one-off escape | `cat my\ file.txt` |

### Flag
```
[REDACTED]
```

---

## Level 3 → Level 4
**Learning Objectives:** Hidden files in Linux, directory navigation

### Challenge
Find and read a hidden file in the `inhere` directory

### Concepts
**Hidden files in Linux:**
- Start with `.` (dot)
- Not shown by default `ls`
- Used for config files (`.bashrc`, `.ssh/`, etc.)
- Still accessible - just visually hidden

### Solution
```bash
cd inhere
ls -a                    # Show ALL files including hidden
cat ...Hiding-From-You   # Note: THREE dots before name
```

### Common `ls` Options

| Flag | Purpose | Shows |
|------|---------|-------|
| `ls` | Basic listing | Regular files only |
| `ls -a` | All files | Including `.` and `..` |
| `ls -A` | Almost all | Hidden files, excludes `.` and `..` |
| `ls -la` | Long + all | Detailed info + hidden files |
| `ls -lh` | Human readable | File sizes in KB/MB/GB |

### The Trick
File is named `...Hiding-From-You` (note **3 dots**)
- `.` = current directory
- `..` = parent directory
- `...` = valid filename character sequence

### Flag
```
[REDACTED]
```

---

## Level 4 → Level 5
**Learning Objectives:** File type identification, command piping, bulk operations

### Challenge
Find the only human-readable file among multiple files in `inhere/`

### Initial Recon
```bash
cd inhere
ls
# -file00  -file01  -file02  -file03  -file04
# -file05  -file06  -file07  -file08  -file09
```

### Concepts
- **File types:** Linux doesn't rely on extensions - actual content matters
- **`file` command:** Identifies file types by examining contents
- **Piping:** Chain commands together
- **`find` + `exec`:** Powerful file search and operation

### Solutions

**Method 1: Using `find` (Robust)**
```bash
find . -type f -exec file {} \; 2>/dev/null | grep ASCII
```

**Breaking it down:**
```bash
find .              # Start in current directory
  -type f           # Only files (not directories)
  -exec file {} \;  # Execute 'file' command on each result
                    # {} = placeholder for filename
                    # \; = marks end of -exec command
  2>/dev/null       # Redirect errors to /dev/null (hide them)
  | grep ASCII      # Pipe to grep, filter for ASCII text
```

**Method 2: Wildcard + file**
```bash
file ./-file* | grep ASCII
```
✅ Simpler for this specific case

**Method 3: Loop (Educational)**
```bash
for f in ./-file*; do
    file "$f" | grep -q ASCII && cat "$f"
done
```

### Understanding Redirection

| Syntax | Meaning |
|--------|---------|
| `2>` | Redirect stderr (error messages) |
| `1>` | Redirect stdout (normal output) |
| `&>` | Redirect both stdout and stderr |
| `/dev/null` | The "black hole" - discards all input |

### Why This Matters
Real-world scenarios:
- Examining unknown files before opening
- Finding text files in mixed directories
- Forensics and data recovery
- Malware analysis (identifying file types)

### Flag
```
[REDACTED]
```

---

## Quick Reference Card

### Special Filename Characters
| Character | Issue | Solution |
|-----------|-------|----------|
| `-` | Interpreted as stdin/flag | `cat ./-` or `cat < -` |
| Spaces | Word splitting | `"filename"` or `filename\ with\ spaces` |
| `*`, `?` | Glob expansion | Single quotes: `'file*'` |
| `$` | Variable expansion | Single quotes or escape: `\$` |

### Essential Commands So Far
```bash
# Navigation & listing
ls -la              # Detailed list with hidden files
cd directory        # Change directory
pwd                 # Print working directory

# File operations
cat filename        # Display file contents
file filename       # Identify file type
find . -type f      # Find all files recursively

# Text processing
grep pattern        # Search for pattern
| (pipe)            # Chain commands

# Special
2>/dev/null         # Suppress errors
```

### Skills Acquired
- ✅ SSH connection with custom ports
- ✅ Handling special characters in filenames
- ✅ Working with hidden files
- ✅ File type identification
- ✅ Command chaining with pipes
- ✅ Using find with -exec
- ✅ Error redirection

---

## Learning Log

### Mistakes Made & Lessons Learned
1. **Level 1:** Initially tried `cat -` without understanding stdin behavior
   - **Lesson:** Always consider how special characters are interpreted

2. **Level 2:** Forgot to quote filename with spaces
   - **Lesson:** Tab completion is your friend!

3. **Level 4:** Tried checking files individually before learning about `find`
   - **Lesson:** Automation beats repetition

### Resources
- [OverTheWire Bandit](https://overthewire.org/wargames/bandit/)
- [Bash Special Characters](https://www.gnu.org/software/bash/manual/)
- [Linux File Command Man Page](https://man7.org/linux/man-pages/man1/file.1.html)

### Next Steps
- Continue to Level 5 → 6
- Practice: Create test files with special names
- Deeper dive: Learn about file permissions (upcoming levels)

---

**Last Updated:** [Date]  
**Current Level:** 5
