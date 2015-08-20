# s3md5
**Linux PC backup to Amazon s3 using bash scripts**

This is a set of Bash (and Awk) scripts run under Linux which back up user files to Amazon's s3 service. It uses the s3tools package or Amazon's official open source package AWS CLI to do the uploads, downloads and listings.

Most backup systems sync your folder structure to the backup service. Instead this system will back up all files into a single s3 folder (Amazon call this a Prefix) by changing the file name to a 32 character string equal to the file's MD5 hash and ignoring the source folder.

It maintains an index to relate the original path/file/timestamp to the MD5-named backup file.

No files are ever deleted at s3 unless specifically requested, so all historic copies can be retrieved (thwarts Cryptolocker). Versioning in s3 is not required as the MD5 changes when the file is changed. Different versions are identified by the path/file/timestamp in the index.

The index is a text file list with tab separated fields which will load into a spreadsheet or text editor. The scripts handle normal maintenance of the index, but for the rare occasions when you need to recover files or want to delete backups you will need to edit the request status in the index entry for the relevant file. Using a spreadsheet can make finding and editing multiple files easy.

