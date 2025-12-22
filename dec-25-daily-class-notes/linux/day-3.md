# Day 3

### File Viewing Commands <a href="#file-viewing-commands" id="file-viewing-commands"></a>

### 41. `more filename`

**Purpose**: Display file content page by page (forward only navigation).\
**Usage**:

* Space bar = scroll down one page
* Enter = scroll down one line
* q = quit
* b = scroll back (limited)\
  **Output**: File displayed in chunks\
  **Example**: `more /var/log/syslog` - view system log page by page\
  **Limitation**: Cannot scroll backward efficiently; use `less` for better control

### 42. `less filename`

**Purpose**: Display file content with better navigation (backward and forward).\
**Usage**:

* Space bar / Page Down = scroll down one page
* Page Up / b = scroll up one page
* Arrow keys = line-by-line navigation
* /pattern = search forward for pattern
* ?pattern = search backward for pattern
* n = next search match
* N = previous search match
* g = go to beginning of file
* G = go to end of file
* q = quit\
  **Output**: File displayed with flexible navigation\
  **Example**: `less large_logfile.log` - navigate through large files efficiently\
  **Advantage**: More powerful than `more`; allows backward scrolling and search

***

### System Information Commands <a href="#system-information-commands" id="system-information-commands"></a>

### 62. `df -h .`

**Purpose**: Show disk usage of current directory's filesystem in human-readable format.\
**Breakdown**:

* `df` = Disk free/usage
* `-h` = Human-readable (K, M, G, T)
* `.` = Current directory\
  **Output Columns**: Filesystem, Size, Used, Available, Use%, Mounted on\
  **Example Output**:

```
textFilesystem      Size  Used Avail Use%
/dev/sda1       100G   60G   40G  60%
```

**Use Case**: Check available space before downloading/installing software

### 63. `free -g` or `free -m`

**Purpose**: Display memory usage in gigabytes (-g) or megabytes (-m).\
**Breakdown**:

* `free` = Show free and used memory
* `-g` = Display in gigabytes
* `-m` = Display in megabytes\
  **Output Columns**: Total, Used, Free, Shared, Buffer/Cache, Available\
  **Example with -g**:

```
text              total   used   free
Mem:             16      8      8
Swap:             4      0      4
```

**Example with -m**:

```
text              total    used    free
Mem:           16384    8192    8192
Swap:            4096       0    4096
```

**Use Case**: Monitor available RAM before running memory-intensive applications

***

### File Sorting and Deduplication <a href="#file-sorting-and-deduplication" id="file-sorting-and-deduplication"></a>

### 64. `uniq file`

**Purpose**: Remove adjacent duplicate lines from a file.\
**Important**: File must be sorted first or adjacent duplicates won't be detected.\
**Output**: Lines with adjacent duplicates removed\
**Example**:

```
textInput:
apple
apple
banana
banana
cherry

Output (uniq file):
apple
banana
cherry
```

**Limitation**: Only removes consecutive duplicates\
**Common Pattern**: `sort file | uniq` to remove all duplicates

### 65. `sort file`

**Purpose**: Sort lines in a file alphabetically (default).\
**Output**: File contents sorted line by line\
**Example**:

```
textInput:
zebra
apple
banana

Output:
apple
banana
zebra
```

**Flags**:

* `-n` = Numeric sort
* `-r` = Reverse sort
* `-k 2` = Sort by column 2
* `-u` = Sort and remove duplicates

### 66. `sort file|uniq`

**Purpose**: Sort file contents and remove all duplicate lines (including non-adjacent).\
**Breakdown**: `sort file` → sorts entire file → `uniq` → removes adjacent duplicates (which are all duplicates after sort)\
**Output**: Sorted unique lines only\
**Example**:

```
textInput:
banana
apple
banana
cherry
apple

Output:
apple
banana
cherry
```

**Use Case**: Get list of unique values from a file (e.g., unique IPs from logs)

### 67. `sort -r file|uniq`

**Purpose**: Sort file in reverse order and remove duplicate lines.\
**Breakdown**:

* `sort -r file` = Sort in reverse (Z to A)
* `uniq` = Remove adjacent duplicates\
  **Output**: Sorted unique lines in reverse order\
  **Example**:

```
textInput:
banana
apple
banana
cherry
apple

Output:
cherry
banana
apple
```

**Use Case**: List unique items in descending alphabetical order

***

### File Permissions (chmod) <a href="#file-permissions-chmod" id="file-permissions-chmod"></a>

### Understanding Permission Notation

**Permission Symbols**:

* r (read) = 4
* w (write) = 2
* x (execute) = 1

**Three Permission Sets** (owner, group, others):

* 7 = rwx (4+2+1)
* 6 = rw- (4+2)
* 5 = r-x (4+1)
* 4 = r-- (4)
* 3 = -wx (2+1)
* 2 = -w- (2)
* 1 = --x (1)
* 0 = --- (0)

**Examples of Permission Calculations**:

```
textrwx rwx rwx = 7 7 7 (full permissions for all)
rw- r-- r-- = 6 4 4 (owner reads/writes, group/others read only)
rwx r-x r-- = 7 5 4 (owner full, group execute, others read)
rw- --- --- = 6 0 0 (owner only can read/write)
```

### 68. `chmod 777 filename`

**Purpose**: Set read, write, execute permissions for owner, group, and others on a file.\
**Breakdown**: 7 7 7 = rwx rwx rwx\
**Effect**: Everyone can read, write, and execute the file\
**Security Risk**: Avoid in production; too permissive\
**Example**: `chmod 777 script.sh` - makes script fully accessible\
**Use Case**: Temporary development permissions only

### 69. `chmod 777 directory`

**Purpose**: Set read, write, execute permissions for owner, group, and others on a directory.\
**Breakdown**: 7 7 7 = rwx rwx rwx\
**Effect**: Everyone can read, write, and enter the directory\
**Warning**: Dangerous for system directories\
**Example**: `chmod 777 temp_folder/`\
**Use Case**: Temporary shared directories only

### 70. `chmod 644 directory1 directory2 directory3`

**Purpose**: Set read/write for owner, read-only for group/others on multiple directories.\
**Breakdown**: 6 4 4 = rw- r-- r--\
**Effect**: Owner can read/write, group and others can only read\
**Multiple targets**: Applies to all three directories\
**Example**: `chmod 644 config1 config2 config3`\
**Use Case**: Configuration files accessible for reading but not modification

### 71. `chmod 755 file1 file2 file3`

**Purpose**: Set read/write/execute for owner, read/execute for group/others on multiple files.\
**Breakdown**: 7 5 5 = rwx r-x r-x\
**Effect**: Owner full access, group/others can read and execute\
**Multiple targets**: Applies to all three files\
**Example**: `chmod 755 script1.sh script2.sh script3.sh`\
**Use Case**: Standard permissions for executable scripts and public programs

### 72. `chmod -R 777 folder1`

**Purpose**: Recursively set read, write, execute permissions for folder and ALL contents.\
**Breakdown**:

* `-R` = Recursive (applies to directory and all subdirectories/files)
* `777` = rwx rwx rwx\
  **Effect**: All files and subdirectories get full permissions\
  **Warning**: Very dangerous; can expose system security\
  **Example**: `chmod -R 777 /var/www/html/` (not recommended)\
  **Use Case**: Development environments only, never in production

### 73. `chmod a+w file`

**Purpose**: Add write permission for all (owner, group, others).\
**Breakdown**:

* `a` = all (owner, group, others)
* `+` = add permission
* `w` = write\
  **Effect**: Everyone gets write permission added (even if they had none before)\
  **Example**: `chmod a+w document.txt` - makes file writable by everyone\
  **Before**: rw- r-- r-- (644)\
  **After**: rw- rw- rw- (666)

### 74. `chmod a+rwx folder`

**Purpose**: Add read, write, execute permissions for all users on a folder.\
**Breakdown**:

* `a` = all
* `+` = add permission
* `rwx` = read, write, execute\
  **Effect**: Everyone gets all permissions on the folder\
  **Example**: `chmod a+rwx shared_folder/`\
  **Use Case**: Shared development folders (not production)

### 75. `chmod u+rw file`

**Purpose**: Add read and write permissions for owner (user) only.\
**Breakdown**:

* `u` = user/owner
* `+` = add permission
* `rw` = read and write\
  **Effect**: Owner gets read and write (execute not added)\
  **Example**: `chmod u+rw important_file.txt`\
  **Use Case**: Restore write permission on protected files

### 76. `chmod g+w director`

**Purpose**: Add write permission for group on a directory.\
**Breakdown**:

* `g` = group
* `+` = add permission
* `w` = write\
  **Effect**: Group members can now modify directory contents\
  **Example**: `chmod g+w project_folder/`\
  **Use Case**: Allow team members to edit shared folders

### 77. `chmod o+r file`

**Purpose**: Add read permission for others on a file.\
**Breakdown**:

* `o` = others (everyone except owner and group)
* `+` = add permission
* `r` = read\
  **Effect**: Others can now read the file\
  **Example**: `chmod o+r public_document.txt`\
  **Use Case**: Make file readable to public/all users

***

### Umask (Default Permission Mask) <a href="#umask-default-permission-mask" id="umask-default-permission-mask"></a>

Umask determines default permissions for new files/directories. It SUBTRACTS from 777 (directories) or 666 (files).

### 78. `umask 641`

**Purpose**: Set default permission mask to 641.\
**Effect**: New files/dirs subtract 641 from defaults\
**Breakdown** (subtracting from 777):

* 777 - 641 = 136
* Owner: 7 - 6 = 1 (--x)
* Group: 7 - 4 = 3 (-wx)
* Others: 7 - 1 = 6 (rw-)\
  **Result for new directories**: --x -wx rw-\
  **Example**: `umask 641`

### 79. `umask 777`

**Purpose**: Set most restrictive umask (removes all permissions by default).\
**Effect**: 777 - 777 = 000\
**Result**: New files/directories have NO permissions (--- --- ---)\
**Example**: `umask 777`\
**Use Case**: Highly secure environment requiring explicit permission grants

### 80. `umask 000`

**Purpose**: Set most permissive umask (no permission restrictions).\
**Effect**: 777 - 000 = 777\
**Result**: New files/directories have FULL permissions (rwx rwx rwx)\
**Example**: `umask 000`\
**Use Case**: Development/testing environments only\
**Security**: Not recommended for production

***

### File Deletion <a href="#file-deletion" id="file-deletion"></a>

### 81. `rm filename`

**Purpose**: Delete a single file permanently.\
**Warning**: Deletion is immediate and irreversible\
**Example**: `rm old_file.txt`\
**Safe option**: Use `-i` flag for interactive confirmation: `rm -i filename`

### 82. `rm -rf directory_name`

**Purpose**: Recursively delete directory and ALL contents (files and subdirectories).\
**Breakdown**:

* `-r` = Recursive (delete directory and contents)
* `-f` = Force (don't prompt for confirmation)\
  **Warning**: EXTREMELY DANGEROUS; no recovery possible\
  **Example**: `rm -rf temp_folder/`\
  **Safer alternative**: `rm -ri directory_name` (interactive, asks for confirmation)\
  **Common mistake**: `sudo rm -rf /` (system-destroying command)

### 83. `rm f1 f2 f3`

**Purpose**: Delete multiple files at once.\
**Output**: All three files deleted\
**Example**: `rm file1.txt file2.txt file3.txt`\
**Safe option**: `rm -i f1 f2 f3` (prompts before each deletion)

### 84. `rm -rf folder1 folder2 folder3`

**Purpose**: Recursively delete multiple directories and all contents.\
**Breakdown**:

* `-r` = Recursive
* `-f` = Force\
  **Effect**: All three folders and everything inside deleted\
  **Example**: `rm -rf backup1/ backup2/ backup3/`\
  **Warning**: Very destructive; use with extreme caution

### 85. `rm director/dir1/file`

**Purpose**: Delete a file located in a subdirectory.\
**Output**: File deleted from nested path\
**Example**: `rm /home/user/Documents/old_file.txt`\
**Safe option**: `rm -i director/dir1/file`

### 86. `rm -rf /dir1/dir2/dir3`

**Purpose**: Recursively delete directory tree starting from dir1.\
**Breakdown**:

* `-r` = Recursive
* `-f` = Force (no prompts)
* `/dir1/dir2/dir3` = Full or relative path\
  **Effect**: Entire directory structure deleted\
  **Example**: `rm -rf /home/user/old_project/`\
  **Critical Warning**: Using `/` (root) at start is extremely dangerous

***

### File Moving (Rename) <a href="#file-moving-rename" id="file-moving-rename"></a>

### 87. `mv old_name New_name`

**Purpose**: Rename a file or directory.\
**Output**: File/directory renamed in same location\
**Example**: `mv report.txt report_final.txt`\
**Note**: Same as "moving" within same directory

### 88. `mv file dir1/file`

**Purpose**: Move file to dir1 (can also rename in process).\
**Output**: File relocated to dir1\
**Example**: `mv document.txt /home/user/Documents/document.txt`\
**Syntax**: `mv [source] [destination]`

### 89. `mv dir2/f1 .`

**Purpose**: Move file f1 from dir2 to current directory (.).\
**Output**: f1 relocated to current directory\
**Example**: `mv backup/data.zip .` (moves data.zip to current folder)\
**Syntax**: `.` represents current directory

***

### File Copying <a href="#file-copying" id="file-copying"></a>

### 90. `cp file file2`

**Purpose**: Copy file to create file2 (duplicate).\
**Output**: file and file2 both exist with same content\
**Example**: `cp original.txt backup.txt`\
**Note**: Original file unchanged

### 91. `cp file dir1/dir2/file`

**Purpose**: Copy file to nested directory structure (may need to create dirs first).\
**Output**: Copy placed in dir1/dir2/\
**Example**: `cp config.conf /etc/app/config.conf`\
**Note**: Target path must exist

### 92. `cp -i file file2`

**Purpose**: Copy with interactive prompt if file2 already exists.\
**Breakdown**: `-i` = Interactive (prompts before overwriting)\
**Output**: Prompts user before overwriting\
**Example**: `cp -i important.txt important.txt.backup`\
**Safe**: Prevents accidental overwrites

### 93. `cp -i test dir1/test`

**Purpose**: Copy file 'test' to dir1 with interactive confirmation if target exists.\
**Breakdown**: `-i` = Interactive prompt\
**Output**: Copies to dir1, prompts if file exists\
**Example**: `cp -i config config/config.old`

### 94. `cp -r dir2 dir3`

**Purpose**: Recursively copy entire directory dir2 as dir3.\
**Breakdown**: `-r` = Recursive (copy directory and all contents)\
**Output**: New directory dir3 created with all contents from dir2\
**Example**: `cp -r /var/www/html /var/www/html_backup`\
**Use Case**: Backing up entire directory structures

### 95. `cp -r dir1 dir2/dir3/dir4`

**Purpose**: Copy directory dir1 into dir2/dir3/dir4/.\
**Output**: dir1 becomes a subdirectory inside dir4\
**Example**: `cp -r project project_archive/2024/Q1/`\
**Note**: dir2/dir3/dir4 must exist

### 96. `cp -i dir1/file dir2/dir3/dir4/`

**Purpose**: Copy file from dir1 into nested dir with interactive prompt.\
**Breakdown**:

* `-i` = Interactive (prompt if exists)
* `dir2/dir3/dir4/` = Destination directory (note the trailing `/`)\
  **Output**: File copied with confirmation if target exists\
  **Example**: `cp -i /etc/config /backup/configs/old/`

***

### File Listing with Wildcards <a href="#file-listing-with-wildcards" id="file-listing-with-wildcards"></a>

### 97. `ls Devops*`

**Purpose**: List all files/directories starting with "Devops".\
**Output**: Files matching pattern\
**Example Output**:

```
textDevopsClass
DevopsGuide.txt
DevopsProject
```

**Pattern**: `*` = wildcard matching any characters after "Devops"

### 98. `ls test*`

**Purpose**: List all files/directories starting with "test".\
**Output**: Files matching pattern\
**Example Output**:

```
texttest.txt
test_file.log
test_script.sh
testing.md
```

**Pattern**: `*` matches zero or more characters

### 99. `ls *`

**Purpose**: List all files and directories in current directory.\
**Output**: All items in current folder\
**Example Output**:

```
textfile1.txt
file2.txt
folder1
folder2
README.md
```

**Note**: Same as `ls` without arguments

***

### Grep with Regex Anchors <a href="#grep-with-regex-anchors" id="grep-with-regex-anchors"></a>

Anchors match specific positions in a line:

* `^` = Start of line
* `$` = End of line

### 100. `grep -i "^Devops" file_name`

**Purpose**: Find lines STARTING with "Devops" (case-insensitive).\
**Breakdown**:

* `-i` = Case-insensitive (matches "Devops", "DEVOPS", "devops")
* `^Devops` = Must start with "Devops"\
  **Output**: Lines beginning with "Devops"\
  **Example Output**:

```
textDevops is important
DEVOPS class 13
DevopsTeam meeting
```

**Won't match**: "Learn Devops here" (doesn't start with Devops)

### 101. `grep -i "^t" file_name`

**Purpose**: Find lines STARTING with "t" (case-insensitive).\
**Breakdown**:

* `-i` = Case-insensitive (T or t)
* `^t` = Must start with "t"\
  **Output**: Lines beginning with "t" or "T"\
  **Example Output**:

```
texttest case
The beginning
training module
```

**Won't match**: "at the start" (doesn't start with t)

### 102. `grep -i "dev$" file_name`

**Purpose**: Find lines ENDING with "dev" (case-insensitive).\
**Breakdown**:

* `-i` = Case-insensitive (matches "dev", "DEV", "Dev")
* `dev$` = Must end with "dev"\
  **Output**: Lines ending with "dev"\
  **Example Output**:

```
textI am a dev
Full stack dev
DevOps dev
```

***

### Summary Table <a href="#summary-table" id="summary-table"></a>

| Command       | Purpose                       | Key Option              |
| ------------- | ----------------------------- | ----------------------- |
| more          | Page through file (forward)   | Space to scroll         |
| less          | Navigate file (bidirectional) | /pattern to search      |
| df -h         | Disk space usage              | . for current           |
| free          | Memory usage                  | -g or -m                |
| sort          | Sort lines                    | -r for reverse          |
| uniq          | Remove duplicates             | Works after sort        |
| chmod         | Change permissions            | -R for recursive        |
| umask         | Default permissions           | Subtracts from 777      |
| rm            | Delete files                  | -rf for recursive       |
| mv            | Move/rename                   | Works in same operation |
| cp            | Copy files                    | -r for directories      |
| grep ^pattern | Lines starting with           | $ for end anchors       |
