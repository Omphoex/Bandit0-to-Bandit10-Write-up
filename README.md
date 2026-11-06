# OverTheWire Bandit Write-up (Levels 0 to 10)

My walkthrough of the first 10 levels of [OverTheWire Bandit](https://overthewire.org/wargames/bandit/), a beginner wargame for learning Linux command-line and security basics.

> Passwords are intentionally not included. This write-up covers the method only.

## How to connect (all levels)

```bash
ssh bandit<LEVEL>@bandit.labs.overthewire.org -p 2220
```

- `-p 2220` sets the non-default SSH port the game uses.
- Enter the password found in the previous level when prompted.
- Run `exit` to leave a level before connecting to the next.

---

## Level 0

**Goal:** Log in to the game server.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

**Learned:** Basic SSH login with a custom port.

---

## Level 0 > Level 1

**Goal:** Find the password in a file called `readme`.

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
ls
cat readme
```

**Learned:** `ls` lists files, `cat` prints a file's contents.

---

## Level 1 > Level 2

**Goal:** The password is in a file named `-`.

```bash
cat ./-
```
**Why `./`:** A bare `-` is treated by many commands as "standard input", not a filename. Prefixing `./` makes it an explicit path.

**Learned:** Dashed filenames and relative paths.

---

## Level 2 > Level 3

**Goal:** The password is in a file named `--spaces in this filename--`.

```bash
cat "./--spaces in this filename--"
```

**Why:** Quotes keep the spaces as part of one filename, and `./` stops the leading `--` from being read as an option.

**Learned:** Quoting and escaping special filenames.

---
## Level 3 > Level 4

**Goal:** The password is in a hidden file inside the `inhere` directory.

```bash
ls
cd inhere
ls -a
cat .<hidden-file-name>
```
**Learned:** Files starting with `.` are hidden. `ls -a` shows them.

---
## Level 4 > Level 5

**Goal:** The password is in the only human-readable file in `inhere`.

```bash
cd inhere
file ./*
cat ./<file marked "ASCII text">
```

**Learned:** `file` identifies file types, so you don't have to open every file.

---

## Level 5 > Level 6

**Goal:** Find a file in `inhere` that is human-readable, 1033 bytes, and not executable.

```bash
cd inhere
find . -type f -size 1033c ! -executable
cat <path-from-find>
```

**Learned:** `find` filters by type, size (`c` = bytes) and permissions.

---

## Level 6 > Level 7

**Goal:** Find a file anywhere on the server owned by user `bandit7`, group `bandit6`, 33 bytes.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
cat <path-from-find>
```

### Problem I hit and how I solved it

My first attempt used `2>dev/null`, and it did not hide the errors. Without redirecting errors, `find` printed a huge number of "Permission denied" lines with only one real result, and I picked out the password file by hand.

**The bug:** I typed `2>dev/null` instead of `2>/dev/null`. The missing leading `/` makes it a relative path, so the shell looks for a `dev` folder in the current directory and fails.

**The fix:** Use the full path `/dev/null`.

- `2>` redirects **stderr** (error messages, file descriptor 2).
- `/dev/null` is a special file that discards everything written to it.

So `2>/dev/null` hides the "Permission denied" noise and leaves only the real match.

**Learned:** Searching the whole filesystem, stderr redirection, and why exact paths matter.

---

## Level 7 > Level 8

**Goal:** The password is next to the word `millionth` in `data.txt`.

```bash
grep millionth data.txt
```

**Learned:** `grep` searches text for a pattern.

---

## Level 8 > Level 9

**Goal:** The password is the only line in `data.txt` that appears once.

```bash
sort data.txt | uniq -u
```

**Why `sort` first:** `uniq` only compares adjacent lines, so the file must be sorted.

**Learned:** Pipes (`|`), `sort`, `uniq -u`.

---

## Level 9 > Level 10

**Goal:** The password is one of the few human-readable strings in `data.txt`, preceded by several `=` characters.

```bash
strings data.txt | grep "=="
```

**Learned:** `strings` pulls readable text out of binary files, and `grep` narrows the result.

---

## Commands used

`ssh`, `ls`, `ls -a`, `cd`, `cat`, `file`, `find`, `grep`, `sort`, `uniq`, `strings`, pipes (`|`), redirection (`2>/dev/null`)

## Takeaways

- Special filenames need `./` and quotes.
- `find` plus `2>/dev/null` makes system-wide searches usable.
- Pipes let you chain small tools into powerful one-liners.
