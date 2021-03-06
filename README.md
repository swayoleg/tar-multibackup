tar-multibackup
===============


FORK FROM frdmn/tar-multibackup
https://github.com/frdmn/tar-multibackup 

Bash script to backup multiple folders and to clean up old backups based on a retention time. Features configurable post/pre-commands, tar excludes as well as backup retentions.

### Installation

    cd /usr/local/src
    git clone https://github.com/swayoleg/tar-multibackup.git
    ln -sf /usr/local/src/tar-multibackup/multibackup /usr/local/bin/multibackup
    cp /usr/local/src/tar-multibackup/multibackup.conf ~/.multibackup.conf

### Configuration and usage

* `timestamp` = Format of the timestamp, used in the backup target filename
* `backup_destination` = Directory which is used to store the archives/backups
* `folders_to_backup` = Array of folders to backup
* `backup_retention` = Retention time how long we should keep the backups
* `pre_commands` = Array of commands that are executed before the backup starts (stop specific service)
* `post_commands` = Array of commands that are executed after the backup finished (start specific service)
* `use_destination_as_root_folder` = 0 or 1. Default 0; Use 1 if you don' want to create sub-folders for each backup source - just appear postfix in name, read example.


### Environment configurations

* `DEBUG` = if set to "true", `set -x` will be set
* `CONFIG` = if you want to use a different configuration file than the 

Example: 

    CONFIG=/tmp/testbackup.conf DEBUG=true multibackup

#### Example configuration 

In the example below you can find a `multibackup` configuration file to backup a productional [LiveConfig](http://www.liveconfig.com/) instance.

`vi ~/.multibackup.conf`

    # Timestamp format, used in the backup target filename
    timestamp=$(date +%Y%m%d)

    # Destination where you want to store your backups
    backup_destination="/var/backups"

    # Folders to backup
    folders_to_backup=(
      "/etc"
      "/var/mail"
      "/var/www"
      "/var/lib/mysql"
      "/var/spool/cron"
      "/var/lib/liveconfig"
    )

    # Files and folders that are excluded in the tar command
    tar_excludes=(
      "nginx-php-fcgi.sock"
    )

    # How long to you want to keep your backups (in days)
    backup_retention="+7"

    # Commands that are executed before the backup started
    # (We have to make sure the liveconfig process is not running)
    # (otherwise the databases changes while we try to save it)
    pre_commands=(
      "service liveconfig stop"
    )

    # Commands that are executed after the backup is completed
    # (To restart the liveconfig process again, once the backup is completed)
    post_commands=(
      "service liveconfig start"
    )

#### Example for use_destination_as_root_folder option
If you set this option as 0 and you have sources like:

`"/var/www"`  
`"/var/lib/mysql"`
  
You get
 ` /var/backups/var-www/$timestamp.tar.gz`  
 ` /var/backups/var-lib-mysql/$timestamp.tar.gz`

And if you set this option to 1 you get:
 ` /var/backups/$timestamp-var-www.tar.gz`  
 ` /var/backups/$timestamp-var-lib-mysql.tar.gz`


#### Cronjob setup

To make sure the backup is executed automatically and recurring, we're going to add a simple cronjob:

`vi /etc/cron.d/backup-liveconfig`

    #
    # cronjob to backup LiveConfig, daily at 5:00 am
    #

    0 5 * * *        root       /usr/local/bin/multibackup &>/dev/null

### Version
1.2.0

### Lincense
[MIT](LICENSE)
