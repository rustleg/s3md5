s3md5 Backup System Read-me
===========================

Overview
========
This is a set of Bash (and Awk) scripts run under Linux which back up user files to Amazon's s3 service. It uses the s3tools package or Amazon's official open source package AWS CLI to do the uploads, downloads and listings. Most backup systems sync your folder structure to the backup service. Instead this system will back up all files into a single s3 folder (Amazon call this a Prefix) by changing the file name to a 32 character string equal to its MD5 hash and ignoring the source folder. It maintains an index to relate the original path/file/timestamp to the backed up file. No files are ever deleted at s3 unless specifically requested, so all historic copies can be retrieved (thwarts Cryptolocker). The index is a text file list with tab separated fields which will load into a spreadsheet or text editor.

Updates can be done in the foreground or background as they require no user interaction. On demand or via scheduled runs using cron.

Restoring files from backup require some effort in locating the file(s) in the index and setting a field value to tell the system you want the file downloaded. The file(s) will be downloaded into a separate area and the original folder structure reconstructed. It should be simple for most users who are capable of basic tasks in Linux, to manipulate the relevant index entries using a text editor or spreadsheet. It is my experience that file restoration is a rare event so a little effort without needing sophisticated GUI tools etc is justified.

For those who worry about 2 different files having the same MD5 hash, a little googling should put your mind at rest.

Who could use this
==================
You need to be running Linux. Some rudimentary knowledge is expected, such as how to run scripts, to specify regular expressions for file/folder exclusion, to manipulate a text file or spreadsheet. No knowledge of Bash or Awk is required unless you want to modify the scripts.

Why?
====
I developed this for two main reasons:
1. I want to be able to rename files/folders and reorganise my file structure from time to time without requiring the files to be re-uploaded. I did use SpiderOak at one time which has the same feature, but I had issues with it which their developers did not address. (Of course they are more focussed on Windows.)
2. I don't want to depend on the whims of commercial developers who are always under pressure to add new "features". I prefer to be in control. It does depend currently on using s3tools or the AWS CLI but only uses a few of their basic commands. 

I have a high regard for s3 in terms of reliability and you pay only for what you store. Some commercial storage services subcontract to s3 - Dropbox a prime example.

Other features:
1. Control is via a simple text file index, only needing a text editor or spreadsheet to manipulate on the rare occasions this is necessary (restoring files or deleting old versions)
2. You don't need versioning turned on at s3. Finding older versions using the index is simple.
3. Only one copy of any file is stored, avoiding redundant multiple backups.
4. If you want to restore a file which happens to also be somewhere else in your file system (but you forget where) it will automatically locate and perform a local copy instead of downloading from s3.
5. If you copy a file or "touch" it to change the timestamp it isn't re-uploaded.
6. If you delete a backed up file it is not deleted from the remote backup.
7. You can specify which folders to back up using a text file of paths. If the paths overlap it won't cause a problem or duplicate anything.
8. File/folder exclusions can be excluded by specifying a list of egrep-style regular expressions. There is a script to test your expressions which produces a list of the files to be excluded so you can make sure your expression works as you intend.
9. Backups are validated by the script and any which didn't work are redone automatically, although you can set a retry limit. A fresh run of the script will keep retrying until all succeed.
10. There is a simple mechanism to abort the backup gracefully by finishing the current file. The next run will then resume where it left off. If you want to interrupt the current upload, s3tools will allow you to press Ctrl-C to abort, but AWS CLI is more stubborn.
11. Each new index is also backed up in s3. Old indexes may be discarded as the latest one includes all historic information.
12. There is an audit script which checks the files at s3 against the index and reschedules any missing backups and reports backup files which have no index entry. This gives you confidence to delete old indexes.
13. Various log files are produced.
14. There are a number of additional utility scripts.

Limitations
===========
1. This does not "watch" your file system for changes so the latest changes might be lost if your file is deleted before any backup is done. It only backs up on demand, or via a schedule if you set up a regular backup job using cron.
2. It takes time to manipulate the index for each pass, although you don't have to sit there and watch it.
3. It doesn't use Amazon's Glacier. Personally I would worry about the cost of restoring from Glacier.
4. It depends on s3tools or the AWS CLI package. s3tools is a little more complete than AWS CLI.
5. Restoring a whole folder with many subfolders requires many entries in the index to be modified. However they are easily sorted by path so they will all be together and the modification should be relatively simple particularly in a spreadsheet using copy and paste.
6. You could accidentally mess up the index while manipulating it. This isn't as serious a problem as it sounds since you have the previous indexes to go back to and the audit script to validate an index.
7. Your s3 credentials should be stored in a safe vault somewhere in case of catastrophe.

Local Setup
===========
Create the following folder heirarchy where you want to locate the scripts, indexes and supporting files.
mys3md5BackupSystem (or whatever)
	bu-<bucket1>
		control
		index
		logs
		recovery
		runfiles
	bu-<bucket2> ... (same subfolders)
	docs
	scripts
		sub

docs are whatever documentation you want to put there
scripts and sub must contain the set of scripts supplied
control must contain:
	BackupRootsRequired.txt - one line per folder to back up - e.g. /home/user or /mnt/nas/data. You could start with a small part of your file structure to get the backup going and then include more folders later. The folders you specify may overlap but this won't create duplicate backups. If you later remove a folder it won't remove the entries from the index file or the backups.
	BackupExcludesRequired.txt - one line per pattern to exclude. This can be a path, part of a path or a complete path/file, etc. It will be used to match the path/files specified in the BackupRoots file using egrep type regular expressions. After specifying the patterns you should run the s3md5-testexclusions script to produce a list of the files it will exclude (BackupExcluded.txt in this folder) so you can be sure the expressions are working as you intend.

Edit the script "s3md5-userparameters" which is the first script to get invoked by the backup. This is to specify your directories, and the name of your default s3 bucket. Also a few optional parameters as explained there. Place or copy this script in to your home folder (I keep it in the scripts/sub folder and put a soft link in home). I also use the home folder to contain my AES plain text key file; some users may want to alter the scripts to secure this properly within their system.

s3 Setup
========
Use the s3 console in a browser to set up the appropriate buckets and subfolders. Don't turn on Amazon s3 versioning.

Bucket names have to be all lowercase (Amazon rule) - my system assumes you have a unique prefix to bucket names as all s3 buckets for all customers reside in the same namespace, so your bucket names must not clash with someone else's. Example buckets are as follows - one for a copy of the scripts, docs,etc and 3 separate backup buckets:
	myuniquename-backupsoftware
	myuniquename-data
	myuniquename-music
	myuniquename-test

Bucket myuniquename-backupsoftware. (This is optional but recommended) - all scripts, etc, including s3tools or other packages, plus documentation. You must upload this stuff yourself manually. Easily done with the s3 console. If your PC goes up in smoke, you have the software and files to recover everything if you have access to your s3 credentials somewhere.

The other buckets are for the backups, you must have at least one, in this example there are  3. Within each bucket create 5 empty folders:
	deletes - temporary area for files being deleted - these are transferred to trash folder after all deletes are confirmed
  files - the actual backup files, no subdirectories
  index - every revised index will be backed up here, so history will be available in case latest ones are corrupt
    each index will be timestamped. Also (pseudo) subfolders are used e.g. myuniquename-datanas/index/2015/07/04/s3md5ix-datanas-2015-07-04-214629.tsv so you can delete a group of them easily in the s3 console by deleting the higher level folder
  trash - deleted files, the original path is appended to the md5 name but with slashes replaced with dashes
  uploads - temporary area for files being uploaded - transferred to files folder after all uploads confirmed

Usage
=====
The main backup script is s3md5bu. Run this in a terminal, set it to run using cron, or use a launcher from an icon on the desktop (my personal preference). Once you have set things up as above there is nothing more to do except to run this when you want to refresh your backup.

There are 2 optional parameters (bucket and throttle) as in this example:
$ s3md5bu --bucket mybucket2 --throttle 80
-b or --bucket : mybucket2 is a non-default bucket, so you can use several different buckets (each must have its own set of local folders such as bu-mybucket2, see above, and the appropriate subfolders in s3). The default bucket is specified in s3md5-userparameters.
-t or --throttle: 80 is an upload speed limit in KB/sec. My system typically will upload at around 130 KB/sec but this will mostly hog all bandwidth, so I tend to use about 2/3 of the max speed.

STOP file
=========
There are 2 ways to abort a backup. If you are using s3tools as your client, you can press Ctrl-C to stop the script. This will probably leave a partially uploaded file in s3 which will not be visible with the s3 console (but can be removed using s3tools). A better way is to allow the current upload to complete by creating a file named STOP (may be empty) in the temporary directory (there is a small script s3md5-stop which will create this file for you). As soon as the current upload is completed this will stop the main backup script (and then delete the stop file to allow restarting). To resume just restart s3md5bu. This will resume where it left off without rescanning the file system or checking the index.

Index
=====
The index is a tab separated text file, one line per backed up file (including files deleted) - see backup design for specification. Tabs are used rather than commas because file names may contain commas.

All indexes are named with a timestamp (example s3md5ix-mydatabucket-2015-05-24-110231.tsv) and stored in the local index folder. Only the latest index is needed so when they start to mount up you may want to delete old versions. When a new index is created it is automatically uploaded to s3 in your bucket's index folder.

Eventually there will be a lot to delete so you might want to delete many at once. There are 2 ways:
1. Issue a delete command using s3tools (but it seems not AWS CLI) as in the following example which will delete all those dated in March 2015. Obviously you can specify more or less granularity as required.
$ s3cmd rm --recursive s3://mybucket/index/2015/03
It would perhaps be comforting to test this first with a list command like so:
$ s3cmd ls --recursive s3://mybucket/index/2015/03
2. Use the s3 console

Restoring files
===============
Files are restored by altering the "request" status in the current index to 2 using a text editor or spreadsheet. Make sure when saving the index manually that the tabs between fields are preserved by the program used.

When the backup is run the downloads will be processed before the uploads. The files will be downloaded, decrypted, renamed with the original name and parent folders recreated according to the original path. This will not overwrite the original path/file but will be found within (example) ...mys3md5BackupSystem/bu-music/recovery. After the run you can move or copy the file(s) or containing parent folder to where you need it and then delete whatever remains in the recovery folder.

Deleting backups
================
Backups are deleted by altering the "request" status in the current index to 0 using a text editor or spreadsheet. Make sure when saving the index manually that the tabs between fields are preserved by the program used.

When the backup is run the deletes will be processed before the uploads. Backup files will not actually be deleted but will be moved to the trash folder within s3. The name will be changed to for example: .../trash/2015-08-11-f26f9a8fe9cf9607927300bcbbd57ed0-mnt-data-clubData-Leagues-Triples.xlsm
You can see that the slashes in the path are replaced by dashes. This enables you to look at the trash folder and know where the file has come from giving you a clue to whether to permanently delete it or possibly rescue it, as from time to time you will probably want to clear the trash.

If a file has a delete request but the same file is still somewhere else in the file system, the backup will not be deleted. You would have to find and mark for deletion all instances of a file before the system deletes the s3 backup.

Index Audit
===========
The script s3md5-auditindex checks the index against the files stored in s3, changes the status where it is wrong and reports discrepancies.

Other scripts
=============
There are a few other utility scripts:
	s3md5-totalvolume
		prints volume of storage used at s3 (according to the indexes)
	s3md5-abortmultipart
		shows incomplete multipart uploads and offers to remove all parts
	s3md5-completemultipart
		completes upload of incomplete multipart uploads
	s3md5-storescripts
		updates stored scripts and docs on s3 (run from scripts folder)
	s3md5-testexclusions
		egrep regular expressions are used in EXCLUDES file
		use this program to check regular expressions are producing the desired results
		run, then examine s3md5BackupExluded.txt		

Logs
====
See the logs folder for various date stamped logs including errors and warnings.


