# OverTheWire Bandit: Levels 0-20

Progress log of the Bandit wargame, written in my own words. Passwords are left out on purpose.

| Level | Topic | Status |
|-------|-------|--------|
| 0 → 1 | SSH login | Done |
| 1 → 2 | File named `-` | Done |
| 2 → 3 | Spaces in filename | Done |
| 3 → 4 | Hidden files | Done |
| 4 → 5 | Identifying file types | Done |
| 5 → 6 | find by size and permissions | Done |
| 6 → 7 | find by owner, group and size | Done |
| 7 → 8 | grep | Done |
| 8 → 9 | | |
| 9 → 10 | strings and grep | Done |
| 10 → 11 | base64 | Done |
| 11 → 12 | ROT13 with tr | Done |
| 12 → 13 | Hexdump and layered compression | Done |
| 13 → 14 | SSH key login | Done |
| 14 → 15 | | |
| 15 → 16 | | |
| 16 → 17 | | |
| 17 → 18 | | |
| 18 → 19 | | |
| 19 → 20 | | |

---

## Level 0 → 1
**Goal:** log in over SSH on a non-standard port.
**Commands:** `ssh bandit0@bandit.labs.overthewire.org -p 2220`
**Learned:** `-p` sets the port. Nothing shows as you type a password.

## Level 1 → 2
**Goal:** read a file called `-`.
**Commands:** `cat ./-`
**Learned:** a bare `-` means "read from stdin". The `./` makes it a path.
**Went wrong:** I typed `cat ./ -` with a space, so `cat` waited for keyboard input. Ctrl+C gets out.

## Level 2 → 3
**Goal:** read a file with spaces in its name.
**Commands:** `cat "./--spaces in this filename--"`
**Learned:** quotes keep the name as one argument, and `./` stops the leading dashes being read as options.

## Level 3 → 4
**Goal:** find a hidden file.
**Commands:** `cd inhere`, `ls -la`
**Learned:** `ls` hides names starting with a dot. `-a` shows them.

## Level 4 → 5
**Goal:** find the one human-readable file among many.
**Commands:** `file ./*`
**Learned:** `file` looks inside a file and reports its type, such as `ASCII text`.

## Level 5 → 6
**Goal:** find a file by size and permissions.
**Commands:** `find . -type f -size 1033c ! -executable`
**Learned:** the `c` suffix means bytes, and `!` negates a test.
**Went wrong:** I checked folders by hand and missed a hidden file. `find` didn't.

## Level 6 → 7
**Goal:** find a file by owner, group and size anywhere on the server.
**Commands:** `find / -type f -user <user> -group <group> -size 33c 2>/dev/null`
**Learned:** `2>/dev/null` hides "Permission denied" noise. Searching from `.` only covers the current directory.

## Level 7 → 8
**Goal:** find one line in a huge file.
**Commands:** `grep "<word>" data.txt`
**Learned:** `grep` prints only the lines that match.

## Level 8 → 9
**Goal:**
**Commands:**
**Learned:**

## Level 9 → 10
**Goal:** pull readable text out of a mostly binary file.
**Commands:** `strings data.txt | grep "=="`
**Learned:** `strings` extracts printable text, and `|` pipes one command's output into the next.

## Level 10 → 11
**Goal:** decode base64.
**Commands:** `base64 -d data.txt`
**Learned:** base64 is an encoding, not encryption. Anyone can reverse it.

## Level 11 → 12
**Goal:** undo ROT13.
**Commands:** `cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'`
**Learned:** `tr` swaps characters list-to-list. ROT13 undoes itself when applied twice.

## Level 12 → 13
**Goal:** unpack a hexdump that was compressed many times.
**Commands:** `mktemp -d`, `xxd -r`, `file`, `mv`, `gzip -d`, `bzip2 -d`, `tar -xf`
**Learned:** gzip and bzip2 compress one file, and tar bundles many into an archive. Run `file` after every step to see what you have.
**Went wrong:** `gzip -d` and `bzip2 -d` need the right file ending, so rename with `mv` first.

## Level 13 → 14
**Goal:** log in with a private key instead of a password.
**Commands:** `scp -P 2220 user@host:sshkey.private .`, `chmod 400 sshkey.private`, `ssh -i sshkey.private user@host -p 2220`
**Learned:** `-i` points SSH at a key file. `scp` uses a capital `-P` for the port, and the server blocks logging in to `localhost`.
**Went wrong:** I first tried `ssh ... localhost` from inside the server and got blocked. The key had to be copied out and used from my Mac.

