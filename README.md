# AutoMySQLBackup

- [AutomysqlBackup](#automysqlbackup)
  - [Options Documantation](#Options-Documantation)
  - [Advanced Options](#Advanced-Options)
  - [Backup Rotation](#Backup-Rotation)
  - [Please Note](#Please-Note)
  - [Restoring](#Restoring)

## Options Documantation

### Authentication

Set `USERNAME` and `PASSWORD` of a user that has at least `SELECT` permission to `ALL` databases.

### Target Server to Backup

Set the `DBHOST` option to the server you wish to backup, leave the default to backup "this server".

> **Note:** To backup multiple servers make copies of this file and set the options for that server.

### Databases to Backup

Put in the list of DBNAMES(Databases)to be backed up. If you would like to backup ALL DBs on the server set

    DBNAMES="all".

> **Note:** If set to `"all"` then any new DBs will automatically be backed up without needing to modify this backup script when a new DB is created.

### Spaces in DB Names

If the DB you want to backup has a space in the name replace the space with a % e.g. "data base" will become "data%base"

> **NOTE:** Spaces in DB names may not work correctly when `SEPDIR=no`.

### Location

You can change the backup storage location from `/backups` to anything you like by using the `BACKUPDIR` setting.

### Email Log

The `MAILCONTENT` and `MAILADDR` options and pretty self explanitory, use these to have the backup log mailed to you at any email address or multiple email addresses in a space seperated list.

> **Note:** If you set mail content to "log" you will require access to the "mail" program on your server. If you set this to "files" you will have to have mutt installed on your server. If you set it to "stdout" it will log to the screen if run from the console or to the cron job owner if run through cron. If you set it to "quiet". logs will only be mailed if there are errors reported.

### Email Size

`MAXATTSIZE` sets the largest allowed email attachments total (all backup files) you want the script to send. This is the size before it is encoded to be sent as an email so if your mail server will allow a maximum mail size of 5MB I would suggest setting `MAXATTSIZE` to be 25% smaller than that so a setting of `4000` would probably be fine.

### Cron, Executable, and Symlink

Finally copy `automysqlbackup.sh` to anywhere on your server and make sure to set executable permission. You can also copy the script to /etc/cron.daily to have it execute automatically every night or simply place a symlink in /etc/cron.daily to the file if you wish to keep it somwhere else.

> **NOTE:** On Debian copy the file with no extention for it to be run by cron e.g just name the file "automysqlbackup"

Thats it..

## Advanced Options

### Monthly

The list of `MDBNAMES` is the DB's to be backed up only monthly. You should always include `"mysql"` in this list to backup your user/password information along with any other DBs that you only feel need to be backed up monthly. If using a hosted server then you should probably remove "mysql" as your provider will be backing this up.

> **Note:** If `DBNAMES="all"` then `MDBNAMES` has no effect as all DBs will be backed up anyway.

### Exclude Databases

If you set `DBNAMES="all"` you can configure the option `DBEXCLUDE`. Other wise this option will not be used. This option can be used if you want to backup all db's but you want exclude some of them. (eg. a db is to big).

### Create Database

Set `CREATE_DATABASE` to `"yes"` (the default) if you want your SQL-Dump to create a database with the same name as the original database when restoring. Saying `"no"` here will allow your to specify the database name you want to restore your dump into, making a copy of the database by using the dump created with automysqlbackup.

> **Note:** Not used if `SEPDIR=no`

### Backup Databases to One File

The `SEPDIR` option allows you to choose to have all DBs backed up to a single file (*fast restore of entire server in case of crash*) or to seperate directories for each DB (*each DB can be restored seperately in case of single DB corruption or loss*).

### Specify Day

To set the day of the week that you would like the weekly backup to happen set the `DOWEEKLY` setting, this can be a value from 1 to 7 where 1 is Monday, The default is 6 which means that weekly backups are done on a Saturday.

### Compression

`COMP` is used to choose the copmression used, options are `gzip` or `bzip2`. bzip2 will produce slightly smaller files but is more processor intensive so may take longer to complete.

### Remote Compression

`COMMCOMP` is used to enable or diable mysql client to server compression, so it is useful to save bandwidth when backing up a remote MySQL server over the network.

### Latest Backup

`LATEST` is to store an additional copy of the latest backup to a standard location so it can be downloaded by thrid party scripts.

### Blob

If the DB's being backed up make use of large `BLOB` fields then you may need to increase the `MAX_ALLOWED_PACKET` setting, for example 16MB.

### Socket

When connecting to localhost as the DB server (`DBHOST=localhost`) sometimes the system can have issues locating the socket file. This can now be set using the `SOCKET` parameter. An example may be 

    SOCKET=/private/tmp/mysql.sock

### Pre-Backup and Post-Backup Scripts

Use `PREBACKUP` and `POSTBACKUP` to specify Per and Post backup commands or scripts to perform tasks either before or after the backup process.

## Backup Rotation

* **Daily** Backups are rotated weekly..
* **Weekly** Backups are run by default on Saturday Morning when cron.daily scripts are run. Can be changed with `DOWEEKLY` setting. Weekly Backups are rotated on a 5 week cycle..
* **Monthly** Backups are run on the 1st of the month. Monthly Backups are NOT rotated automatically...

**Note:** It may be a good idea to copy Monthly backups offline or to another server..

## Please Note

*I take no resposibility for any data loss or corruption when using this script.*

> **This script will not help in the event of a hard drive crash.** If a  copy of the backup has **not** be stored offline or on another PC..

You should copy your backups **offline regularly** for best protection.

### Happy backing up!

## Restoring

Firstly you will need to uncompress the backup file.

    gunzip file.gz

or

    bunzip2 file.bz2

Next you will need to use the mysql client to restore the DB from the sql file.

    mysql --user=username --pass=password --host=dbserver database < /path/file.sql

or

    mysql --user=username --pass=password --host=dbserver -e "source /path/file.sql" database

> **NOTE:** Make sure you use "<" and not ">" in the above command because you are piping the file.sql to mysql and not the other way around.

Lets hope you never have to use this.. :)