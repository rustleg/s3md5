# s3md5bu
**Linux file backup to Amazon s3 via Bash scripts**

This is a set of Bash (and Awk) scripts run under Linux which back up user files to Amazon's s3 service. It uses the s3tools package or Amazon's official open source package AWS CLI to do the uploads, downloads and listings.

**Yet another internet backup system?**

Most backup systems sync your folder structure to the backup service. Instead this system will back up all files into a single s3 folder (Amazon call this a Prefix) by changing the file name to a 32 character string equal to the file's MD5 hash and ignoring the source folder structure.

**Why?**

1. Reorganise/rename your files without re-uploading. Multiple copies only require one backup file.
2. Keep historic versions without overwriting existing backups (thwarts Cryptolocker). s3 versioning not required.
3. Just click a desktop icon (or schedule via cron) to automatically update the backups. But to clean up (delete) old backups or recover files, first use a text editor or spreadsheet to simply manipulate the backup index, then run the main script. Gives simple but complete control over the backups.

