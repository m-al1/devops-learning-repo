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
| 8 → 9 | sort and uniq | Done |
| 9 → 10 | strings and grep | Done |
| 10 → 11 | base64 | Done |
| 11 → 12 | ROT13 with tr | Done |
| 12 → 13 | Hexdump and layered compression | Done |
| 13 → 14 | SSH key login | Done |
| 14 → 15 | Talking to a port with nc | Done |
| 15 → 16 | TLS with openssl s_client | Done |
| 16 → 17 | Port scanning with nmap | Done |
| 17 → 18 | diff | Done |
| 18 → 19 | Running a command over ssh | Done |
| 19 → 20 | setuid binary | Done |

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
**Goal:** find the one line in a file that appears only once.

**Steps:**
1. `sort data.txt | uniq -u`

**Learned:** `uniq` only compares neighbouring lines, so the file has to be sorted first. `-u` prints only lines that aren't repeated.

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

## Level 14 → 15
**Goal:** send the current password to a service listening on a local port.

**Steps:**
1. `nc localhost <port>` (the port is on the level page).
2. Paste the current password and press Enter.
3. The service replies `Correct!` and the next password.

**Learned:** every network service listens on a numbered port, and `nc` (netcat) lets me talk to one directly with no SSH involved. The block on `localhost` only applies to SSH logins, so this works from inside the server.

## Level 15 → 16
**Goal:** do the same thing, but over an encrypted (TLS) connection.

**Steps:**
1. `openssl s_client -connect localhost:<port> -quiet`
2. Paste the current password and press Enter.
3. The service replies `Correct!` and the next password.

**Learned:** `nc` can't do the TLS handshake, but `openssl s_client` can. `-quiet` hides the certificate noise.
**Went wrong:** without `-quiet` I got a wall of certificate text and "read R BLOCK", which looked like an error but just meant it was waiting for me to type.

## Level 16 → 17
**Goal:** find which port in a range speaks TLS and returns credentials.

**Steps:**
1. `nmap -sV -p <start>-<end> localhost` to list open ports and identify the services. Ports shown as `ssl/...` speak TLS.
2. Try each TLS port with `openssl s_client -connect localhost:<port> -quiet` and send the current password.
3. Decoy ports echo the password back. The right one replies `Correct!` and a private key.
4. Save the key to a file on my Mac, `chmod 400` it, and log in with `ssh -i`.

**Learned:** `nmap` scans a range of ports and `-sV` identifies what runs on each. The reward this time was a key file, not a password.
**Went wrong:**
- Without `-quiet`, `openssl s_client` read my password as a command, because it began with `k`. That triggered `KEYUPDATE` and a wrong password.
- My first key file had no `BEGIN`/`END` lines and a stray `\`, so SSH said `invalid format`.
- I copied one of my own commands to the clipboard after the key, so the key file contained the command instead of the key. `cat` on the file showed it straight away.
- Fix: copy the whole block from `BEGIN` to `END`, paste it into the file, and check it with `cat` and `head -1` / `tail -1` before using it.

## Level 17 → 18
**Goal:** find the one line that differs between two files.

**Steps:**
1. `ls` to see the two files.
2. `diff passwords.old passwords.new`
3. The password is the `>` line, from the second file.

**Learned:** `<` marks lines from the first file and `>` from the second. Swapping the order flips the sides.

## Level 18 → 19
**Goal:** get a file when `.bashrc` logs me out as soon as I connect.

**Steps:**
1. Logging in normally printed `Byebye !` and disconnected.
2. Gave `ssh` the command to run instead: `ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme`

**Learned:** `ssh` can run a single command on the server without starting a shell, so the edited `.bashrc` never runs.

## Level 19 → 20
**Goal:** read a password file that only the next user can read, using a setuid program.

**Steps:**
1. `ls -l` and find the file with an `s` where the owner's `x` would be.
2. Run it with no arguments to read its usage.
3. Test with `./<program> id` and check it reports the next user.
4. `./<program> cat /etc/bandit_pass/<next-user>`

**Learned:** a setuid program runs with its owner's permissions, not mine. It's the same problem as Level 13 (the file belongs to someone else), solved a different way. Linux needs `./` to run a program in the current directory.
