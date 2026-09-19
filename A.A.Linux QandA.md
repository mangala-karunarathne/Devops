# Linux Interview Questions & Answers — Quick Reference

Short, open-book style notes. English answer + Sinhala explanation for each.

---

## Freshers Level (System Design Basics)

### Q1. What is Linux, and how is it different from UNIX?
**Answer:** Linux is a free, open-source, Unix-like operating system kernel created by Linus Torvalds. UNIX is the original proprietary OS from the 1970s that Linux was inspired by. Linux is open-source and free; UNIX variants (like AIX, Solaris) are commercial and closed-source.
**සිංහල:** Linux කියන්නේ Unix ආකෘතියේම නමුත් නොමිලේ, open-source operating system kernel එකක්. UNIX කියන්නේ මුල් වාණිජ (commercial) OS එක. Linux free & open-source, UNIX බොහෝවිට සල්ලි ගෙවන්න ඕන closed-source.

### Q2. What is a Linux Kernel? Why is it important?
**Answer:** The kernel is the core part of the OS that manages hardware (CPU, memory, devices) and provides an interface for software to interact with hardware. It's important because it controls resource allocation, process scheduling, and system security.
**සිංහල:** Kernel කියන්නේ OS එකේ core part එක - hardware (CPU, memory, devices) manage කරන්නේ meke. Software එකට hardware එකත් එක්ක talk කරන්න පාලමක් විදිහට වැඩ කරනවා. Resources බෙදාහැරීම, process scheduling, security control කරන නිසා වැදගත්.

### Q3. What is a shell in Linux, and how is it different from bash?
**Answer:** A shell is a command-line interpreter that takes user commands and passes them to the kernel. Bash (Bourne Again Shell) is one specific type of shell — the most commonly used one on Linux. Other shells include sh, zsh, ksh.
**සිංහල:** Shell කියන්නේ user දෙන commands ගන්නවා, ඒවා kernel එකට pass කරන interpreter එකක්. Bash කියන්නේ shell වර්ගයක් - Linux වල වැඩිපුරම පාවිච්චි කරන shell එක. sh, zsh වගේ shell වෙනත් වර්ග තියෙනවා.

### Q4. What are the basic components of a Linux OS?
**Answer:** Kernel (core), Shell (command interpreter), File System (data storage/organization), System Utilities (tools for admin tasks), and Hardware layer.
**සිංහල:** Kernel (core part), Shell (commands interpret කරන එක), File System (data organize කරන විදිහ), System Utilities (admin tasks වලට tools), සහ Hardware layer එක.

### Q5. What is the init process in Linux?
**Answer:** `init` is the first process started by the kernel at boot (PID 1). It initializes the system and starts all other processes/services. Modern systems mostly use `systemd` instead of traditional SysV init.
**සිංහල:** `init` කියන්නේ boot වෙනකොට kernel එක ආරම්භ කරන පළවෙනි process එක (PID 1). System එක initialize කරලා අනිත් processes/services ටික start කරනවා. දැන් වැඩිපුරම `systemd` පාවිච්චි කරනවා.

### Q6. How do you find files in Linux?
**Answer:** Use `find /path -name "filename"` to search by name, or `locate filename` (uses a pre-built index, faster). Example: `find / -name "*.txt"`.
**සිංහල:** `find /path -name "filename"` කියලා නමින් search කරන්න පුළුවන්, නැත්නම් `locate filename` (කලින් index කරපු data එකෙන් හොයනවා, වේගවත්).

### Q7. What is the difference between a soft link and a hard link?
**Answer:** A **hard link** points directly to the same inode (data) as the original file — deleting the original doesn't affect it. A **soft link (symlink)** is a pointer/shortcut to the file path — if the original is deleted, the symlink breaks.
**සිංහල:** **Hard link** එකක් original file එකේ inode (data) එකටම direct point කරනවා - original එක delete කළත් hard link වැඩ කරනවා. **Soft link (symlink)** කියන්නේ path එකට pointer එකක් - original file එක delete කළොත් symlink එක break වෙනවා.

---

## Intermediate Level

### Q8. How do you change file permissions using chmod?
**Answer:** `chmod` changes read/write/execute permissions. Symbolic: `chmod u+x file` (add execute for user). Numeric: `chmod 755 file` (owner: rwx, group: r-x, others: r-x).
**සිංහල:** `chmod` කියන්නේ file එකේ read/write/execute permissions වෙනස් කරන command එක. `chmod 755 file` කිව්වොත් owner ට full (rwx), group & others ට read+execute විතරයි.

### Q9. What are the different types of permissions available for files?
**Answer:** Three types: **Read (r=4)** - view content, **Write (w=2)** - modify content, **Execute (x=1)** - run as a program. Each applies to three levels: owner, group, others.
**සිංහල:** තුනක් තියෙනවා: **Read (r=4)** - content බලන්න, **Write (w=2)** - content වෙනස් කරන්න, **Execute (x=1)** - program එකක් විදිහට run කරන්න. මේවා owner, group, others කියලා තුන් කොටසකට apply වෙනවා.

### Q10. How do you create and manage symbolic links?
**Answer:** Create with `ln -s /path/to/target /path/to/link`. View/verify with `ls -l` (shows `->` pointing to target). Remove with `rm link_name` (removes only the link, not the target).
**සිංහල:** `ln -s /path/to/target /path/to/link` කියලා හදනවා. `ls -l` කරලා බලනකොට `->` සලකුණෙන් target එක පෙන්නනවා. `rm link_name` කරලා ඉවත් කරන්න පුළුවන් (link එක විතරයි ඉවත් වෙන්නේ, original file එක නෙවෙයි).

### Q11. How do you check your current path/directory?
**Answer:** Use `pwd` (print working directory) — shows the full absolute path of where you currently are in the file system.
**සිංහල:** `pwd` (print working directory) command එක use කරලා දැනට ඉන්න directory එකේ full path එක බලාගන්න පුළුවන්.

### Q12. How do you combine two commands, and what is the use of a pipe (|)?
**Answer:** A pipe `|` takes the output of one command and feeds it as input to another. Example: `ls -l | grep ".txt"` — lists files, then filters only `.txt` files.
**සිංහල:** Pipe (`|`) කියන්නේ එක command එකක output එක අනිත් command එකේ input එක විදිහට pass කරන එක. උදා: `ls -l | grep ".txt"` — files list කරලා ඒවායින් `.txt` file විතරක් filter කරනවා.

### Q13. How can you check for free disk space?
**Answer:** Use `df -h` (disk free, human-readable) to see space per mounted filesystem/partition. Use `du -sh foldername` to check space used by a specific folder.
**සිංහල:** `df -h` කියලා command එකෙන් partitions වල තියෙන free space human-readable විදිහට බලාගන්න පුළුවන්. නිශ්චිත folder එකක් කොච්චර space use කරනවද බලන්න `du -sh foldername`.

### Q14. Write a command to find files with the .txt extension containing a specific string.
**Answer:** `grep -l "specific_string" *.txt` (search in current dir), or recursively: `grep -rl "specific_string" --include="*.txt" /path`. Alternatively: `find /path -name "*.txt" -exec grep -l "specific_string" {} \;`
**සිංහල:** `grep -l "specific_string" *.txt` කියලා current directory එකේ .txt files වල string එක සොයන්න පුළුවන්. Sub-folders ඇතුළුවත් search කරන්න ඕන නම් `grep -rl "specific_string" --include="*.txt" /path` use කරන්න.

### Q15. What are the different ways to view the content of a file without using cat?
**Answer:** `less filename` (scrollable, best for large files), `more filename` (page by page), `head filename` (first 10 lines), `tail filename` (last 10 lines), `tail -f filename` (live/streaming view), `vim`/`nano` (editors, view+edit).
**සිංහල:** `less filename` (scroll කරලා බලන්න, ලොකු files වලට හොඳම), `more filename` (page by page), `head` (මුල් lines 10), `tail` (අන්තිම lines 10), `tail -f` (live updates බලන්න), `vim`/`nano` (edit කරන්නත් පුළුවන් editors).

---

## Experienced Level

### Q16. How do you check the current IP address of your Linux server?
**Answer:** `ip addr show` or `ip a` (modern command). Older systems: `ifconfig`. Also `hostname -I` gives just the IP quickly.
**සිංහල:** `ip addr show` හෝ `ip a` (modern command). පරණ systems වල `ifconfig` use කරනවා. ඉක්මනට IP එක විතරක් ගන්න `hostname -I` use කරන්න පුළුවන්.

### Q17. What is SSH, and how is it used to access a Linux server remotely?
**Answer:** SSH (Secure Shell) is an encrypted network protocol for securely connecting to and controlling a remote server. Command: `ssh username@server_ip`. It also supports key-based authentication (no password) via SSH key pairs.
**සිංහල:** SSH (Secure Shell) කියන්නේ encrypted විදිහට remote server එකකට connect වෙලා control කරන protocol එකක්. Command: `ssh username@server_ip`. Key-based authentication (password නැතුව SSH keys use කරලා) support කරනවා.

### Q18. What is a package manager in Linux, and why is it useful?
**Answer:** A package manager installs, updates, configures, and removes software along with resolving dependencies automatically. Examples: `apt` (Debian/Ubuntu), `yum`/`dnf` (RedHat/CentOS), `pacman` (Arch). Useful because it saves manual dependency handling.
**සිංහල:** Package manager කියන්නේ software install, update, remove කරන්න සහ dependencies auto විදිහට resolve කරන්න use කරන tool එක. උදාහරණ: `apt` (Ubuntu), `yum`/`dnf` (CentOS), `pacman` (Arch). Manual dependency handling වළක්වන නිසා ප්‍රයෝජනවත්.

### Q19. How do you terminate an ongoing process in Linux?
**Answer:** Find PID with `ps aux | grep process_name`, then kill with `kill PID` (graceful, SIGTERM) or `kill -9 PID` (force kill, SIGKILL). Or use `pkill process_name` to kill by name directly.
**සිංහල:** `ps aux | grep process_name` කියලා PID එක හොයාගන්න, ඊට පස්සේ `kill PID` (සාමාන්‍ය විදිහට stop කරන්න) හෝ `kill -9 PID` (force කරලා stop කරන්න) use කරන්න. නම දාලාම කෙළින්ම `pkill process_name` use කරන්නත් පුළුවන්.

### Q20. How do you check system architecture and CPU/memory stats?
**Answer:** Architecture: `uname -m` (e.g., x86_64). CPU info: `lscpu` or `cat /proc/cpuinfo`. Memory: `free -h` (human-readable). Combined live view: `top` or `htop`.
**සිංහල:** Architecture බලන්න `uname -m` (උදා: x86_64). CPU details `lscpu` හෝ `cat /proc/cpuinfo`. Memory `free -h` (human-readable). Live විදිහට හැම එකම එකපාරටම බලන්න `top` හෝ `htop` use කරන්න.

---

*Quick tip for the interview:* practice running each command once on a terminal before the interview — muscle memory helps more than memorizing the explanation.
