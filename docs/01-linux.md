# Linux Command Line Workshop

## First try linux command

* Open Cloud Shell and put these commands

```bash
date
cal
who
ls -a
man cal
# try these control: spacebar, q
clear
```

## Control Characters

* ^c to terminate command
  * ^c means press CONTROL and C together.

## Command editing

* You can use the *left and right arrow keys* to move the cursor in the current command.
* You can use *Ctrl + a* to go to the beginning of the line, and *Ctrl + e* to go to the end.
* You can use the *up and down arrows* to select earlier commands.
* You can use *Ctrl + r* to search inside the history of previous commands.
* You can use *double tab* to autocomplete command

## Show content of file

```bash
less /etc/group
# Use arrow keys and page up/down keys
# Try to press ^c
```

## Lab 1

```bash
date
cal
cal 2008
cal 7 2008
cal 9 1970
cal 9 1752

uptime
hostname
uname -a
whoami
bc
3 + 5 + 7
^D
```

## Help

* -h or --help
  * Also get short summary if input invalid argument.
* man <keyword>
  * Displays manual pages for <keyword>

```bash
man
man cal
man man
whatis date
which date
locate date
locate date | more
# spacebar to go on, ^C to stop
```

## Command and pathname

```bash
/usr/bin/cal 2008
cal 2008
echo $PATH
which cal
# Where is path of date command?
```

## ls

```bash
# Lists all the files (including .* files)
ls -a (all)
# Long listing (type, date, size, owner, permissions)
ls -l (long)
# Lists the most recent files first
ls -t (time)
# Lists the biggest files first
ls -S (size)
# Reverses the sort order
ls -r (reverse)
# Long listing, most recent files at the end
ls -ltr (options can be combined)

ls *txt
# The shell first replaces *txt by all the file and directory names ending by txt (including .txt), except those starting with ., and then executes the ls command line.
ls -d .*
# Lists all the files and directories starting with .
# -d tells ls not to display the contents of directories.
cat ?.log
# Displays all the files which names start by 1 character and end by .log
```

## Directory

```bash
pwd
cd /etc/pam.d/
pwd
cd
# What is the command to go to /usr/local directory?

# When you are in /usr/local how to move to /usr/bin with one command?
cd /usr/local
cd ../bin
cd ../local
cd /usr/bin

# Special Directory Name
cd /
cd ~
cd
cd ~ubuntu
cd ../..
cd -
```

## Command File and Directory

### Directory

* ```mkdir dir1 dir2 dir3 ... (make dir)```
  * Creates directories with the given names.
* ```rmdir dir1 dir2 dir3 ... (remove dir)```
  * Removes the given directories
  * Only works when directories and empty.
* ```cp <source_file> <target_file>```
  * Copies the source file to the target.
* ```cp file1 file2 file3 ... dir```
  * Copies the files to the target directory.
* ```cp -i (interactive)```
  * Asks confirmation if the target file already exists
* ```cp -r <source_dir> <target_dir>```
  * Copies the whole directory. (recursive)

### File

* ```mv <old_name> <new_name>```
  * Renames the given file or directory. (move)
* ```mv -i (interactive)```
  * If the new file already exits, asks for user confirm
* ```rm file1 file2 file3 ...	(remove)```
  * Removes the given files.
* ```rm -i (interactive)```
  * Always ask for user confirm.
* ```rm -r dir1 dir2 dir3 (recursive)```
  * Removes the given directories with all their contents.
* ```head [-<n>] <file>```
  * Displays the first <n> lines (or 10 by default)
  * Doesn't have to open the whole file to do this!
* ```tail [-<n>] <file>```
  * Displays the last <n> lines (or 10 by default)
* ```tail -f <file> (follow)```
  * Displays the last 10 lines of the given file and continues to display new lines when they are appended to the file.
  * Very useful to follow the changes in a log file
  * Doesn't have to open the whole file to do this! 
  * No need to load the whole file in RAM!
  * Very useful for huge files.
* ```grep <pattern> <files>```
  * Scans the given files and displays the lines which match the given pattern.
* ```grep error *.log```
  * Displays all the lines containing error in the *.log files
* ```grep -i error *.log```
  * Same, but case insensitive
* ```grep -ri error .```
  * Same, but recursively in all the files in . and its subdirectories
* ```grep -v info *.log```
  * Outputs all the lines in the files except those containing info.
* ```sort <file>```
  * Sorts the lines in the given file in character order and outputs them.
* ```sort -r <file>```
  * Same, but in reverse order.
* ```sort -ru <file>```
  * u: unique. Same, but just outputs identical lines once.

### Play with file and directory command

```bash
mkdir playground
cd playground
mkdir test
ls
mv test camp
ls
rmdir camp
ls

touch file1
cp file1 file2
mv file2 file3
rm file1

cat /etc/protocols
more /etc/protocols
less /etc/protocols
head /etc/protocols
tail /etc/protocols
tail -f /var/log/docker.log
# Open another terminal
sudo service docker restart

grep root /etc/passwd
grep '^r' /etc/passwd
sort /etc/passwd
sort -r /etc/passwd
```

### Excercise File

* Create *wipcamp12* directory inside your *home* directory
* Create *programmer*, *website*, *network* and *ux-ui* directory inside *wipcamp12* directory
* Create *slide01.txt* file inside your *wipcamp12* directory
* Copy *slide01.txt* to *slide02.txt* and *slide03.txt* inside *wipcamp12* directory
* Copy *wipcamp12* directory to *wipcamp13*
* Delete *slide03.txt* inside *wipcamp13*
* Move *slide01.txt* inside *wipcamp12* to *newslide.txt* inside *wipcamp13*
* Delete *wipcamp12* and *wipcamp13*

## Play with permission

```bash
whoami
id
id root
sudo su -
whoami
id
pwd
touch testperm
ls -l testperm
./testperm
chmod 700 testperm
ls -l testperm
./testperm
sudo su postgres

# Open new terminal
./testperm
sudo chgrp user testperm
ls -l testperm
./testperm
sudo chmod g+x testperm
ls -l testperm
./testperm

sudo chmod g+r testperm
ls -l
./testperm
sudo chmod 007 testperm
ls -l
./testperm
```

## IO Redirection

```bash
ls /etc > dir.txt
cat dir.txt
cal 2007 > years
cal 2008 > years
cal 2009 >> years
echo “README: No such file or directory” > README

wc < years
# What the meaning of each number?
sort < dir.txt
# How can we use sort without standard input

ls /etc | grep passwd
ls /etc | more
ls -l /etc | sort | less
dmesg | grep eth
cat auth.log | grep sudo > auth_sudo.log

tail -f /var/log/syslog | tee log
# How to append file?
```

## Linux Command Line Exercise

* Find how to copy all the /etc/ to your home directory and preserve all the permissions
* Find which file in /var/log/ that keep log who using sudo command and how to show it?
* Which files in /proc/ that show how many cores of CPU, how much of memory?

## Linux Process

* ```ps -ux```
  * Lists all the processes belonging to the current user
* ```ps -aux```
  * Lists all the processes running on the system
* ```ps -aux | grep userid | grep bash```
* ```top```

## Quoting

```bash
echo "Hello World"
echo "You are logged as $USER"
echo /var/log/*.log
echo "/var/log/*.log"
echo 'You are logged as $USER'
echo "Using Linux `uname -r`"
cd /lib/modules/`uname -r`
```

## Disk

```bash
du
# returns size on disk of the given file or directory
du -h <file>
du -sh <dir>
df
# Returns disk space information
df -h <dir>
df -h
```

## Tar

```bash
sudo tar cvf etc.tar /etc/
sudo tar cvfz etc.tar.gz /etc/
tar xvf etc.tar
mv etc.tar.gz etc.tgz
tar xvfz etc.tgz
tar tvf etc.tar
tar tvfz etc.tgz
```

## Tar Exercise

* What is difference between
  * sudo tar cvf etc.tar /etc/
  * sudo tar cvf etc.tar /etc
* Does it matter to have .tar or .tar.gz?
* Backup /var/log/apt and /var/log/fsck in bzip2 compress file with one command to your home directory
* Extract above backup file from your home directory to /opt in one command

Next: [Git](02-git.md)
