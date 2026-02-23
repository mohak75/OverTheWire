# OverTheWire: Bandit Writeup

A collection of notes and lessons learned working through the Bandit wargame.

---

## Level 0 → Level 1

**Connection:** `ssh -p 2220 bandit0@bandit.labs.overthewire.org`

The password for Level 0 is provided on the OverTheWire website. Once logged in, the password for the next level is stored in a file called `readme` in the home directory.

---

## Level 1 → Level 2

**Command:** `cat ./-`

**Key lesson:** The `-` character has three different interpretations in Linux. When used as a filename, you can't just run `cat -` because `cat` treats `-` as a special argument meaning *stdin*, causing it to hang waiting for input. To read a file literally named `-`, prefix it with a path: `cat ./-`.

---

## Level 2 → Level 3

**Command:** `cat "./spaces in this filename"`

**Key lesson:** Filenames with spaces must be wrapped in quotes or have each space escaped with a backslash (`\ `), otherwise the shell splits the name into multiple arguments. Since this file also starts with `--` (like a flag), the `./` prefix is needed inside the quotes to avoid it being interpreted as a command option.

---

## Level 3 → Level 4

**Command:** `cat "...Hiding-From-You"`

**Key lesson:** The hidden file in the `inhere` directory starts with `...` (three dots), which is unusual. Remember that hidden files in Linux start with a single `.`, so this file is hidden but uses multiple dots as part of its actual name. Use `ls -a` to reveal hidden files.

---

## Level 4 → Level 5

**Command:** `find . -type f -exec file {} \; 2>/dev/null | grep ASCII`

**Key lesson:** This level requires identifying the only human-readable file among several. Breaking down the command:
- `.` — search the current directory
- `-type f` — look for files (not directories)
- `-exec file {} \;` — run the `file` command on each result to determine its type
- `{}` — placeholder for the found filename
- `\;` — terminates the `-exec` expression
- `2>/dev/null` — suppresses permission errors
- `| grep ASCII` — filters output to show only ASCII (human-readable) files

---

## Level 5 → Level 6

**Command:** `find . -type f -size 1033c ! -executable`

**Key lesson:** The `-size` flag accepts units — `c` means bytes. The `!` operator negates a condition, so `! -executable` finds files that are *not* executable. (`-not` is an alternative but is not POSIX-compliant.) A common mistake here is using `-type d` (directory) instead of `-type f` (file) — the target is still a file, just inside a specific directory.

---

## Level 6 → Level 7

**Command:** `find / -type f -group bandit6 -user bandit7 -size 33c -print 2>/dev/null`

**Alternative:** `find / -type f -group bandit6 -size 33c -print 2>/dev/null | grep bandit7`

**Key lesson:** When you need to search the entire system, use `/` instead of `.` as the root path. The `find` command supports both `-user` and `-group` flags for ownership filtering. Piping to `grep` can also filter by user in the output path. Redirecting stderr to `/dev/null` keeps the output clean by hiding all the "Permission denied" errors.

---

## Level 7 → Level 8

**Command:** `grep "millionth" data.txt`

**Alternative:** `strings data.txt | grep "millionth"`

**Key lesson:** `grep` alone is sufficient here since the file is text. The `strings` command is useful for extracting human-readable text from binary or mixed-format files. In this case either approach works, but `grep` directly is the simplest.

---

## Level 8 → Level 9

**Command:** `sort data.txt | uniq -u`

**Alternative:** `sort data.txt | uniq -c` (shows counts)

**Key lesson:** `uniq` only detects *adjacent* duplicate lines, so the file must be sorted first. The `-u` flag reports only lines that appear exactly once. The `-c` flag adds a count prefix to each line, which can help visually identify the unique one.

---

## Level 9 → Level 10

**Command:** `strings data.txt | grep ==`

**Alternative:** `strings data.txt | grep -E '=+'`

**Key lesson:** The password is stored alongside several `=` characters. `strings` extracts printable text from binary data, and `grep ==` (or `-E '=+'` for one or more `=`) filters down to the relevant line.

---

## Level 10 → Level 11

**Command:** `base64 -d data.txt`

**Key lesson:** The `-d` flag decodes a Base64-encoded file back to its original content.

---
