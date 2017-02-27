---
layout: page
title: "Installation"
category: doc
date: 2017-02-27
order: 1
---

## Installation instructions

MailWatch for MailScanner is developed on Debian 7 & Ubuntu 12.04, so these docs will reflect this and I will make note on anything that will be required to run on other distro's or operating systems.

If you're upgrading from a previous version, read `UPGRADING`.

### Before you start

You must have a working MailScanner set-up and have running copies of MySQL, Apache, PHP (with MySQL and GD support) and for MailScanner to be able to use a database you need Perl DBI and DBD::mysql; you also need Perl Encoding::FixLatin to deal with email subjects that contain characters in more than one encoding.

Some PHP extensions and executable software are required to make MailWatch fully works:

* MySQL extension (required to connect to database)
* GD extension (required to generate graphs on reports)
* MBstring extension (required to display non-ascii characters)
* exec function not disabled in php.ini
* Curl extension or fsockopen function enabled (needed to download GeoIP files)
* Zlib extension or gunzip executable (needed to extract GeoIP files)
* Ldap extension (needed if you are authenticating users on LDAP server or Active Directory)

### Support

Please use the mailing-list [mailwatch-users](http://lists.sourceforge.net/lists/listinfo/mailwatch-users) on Sourceforge.  Note that you will get faster support if you use the mailing-list.

### Notes for PHP configuration

PHP should have the following set in php.ini (possibly others too....)

```ini
safe_mode = Off
register_globals = Off
magic_quotes_gpc = Off
magic_quotes_runtime = Off
session.auto_start = 0
date.timezone = '<your time zone>'
```

## Installation

All commands below should be run as the 'root'.

### Create the database

```shell
 $ mysql < create.sql
```

NOTE: you will need to modify the above as necessary for your system if you have a root password for your MySQL database (recommended!).

### Create a MySQL user and password & Set-up MailScanner for SQL logging

```shell
 $ mysql
```
```sql
mysql> GRANT ALL ON mailscanner.* TO mailwatch@localhost IDENTIFIED BY '<password>';
mysql> GRANT FILE ON *.* TO mailwatch@localhost IDENTIFIED BY '<password>';
mysql> FLUSH PRIVILEGES;
```

Edit MailWatch-DB.pm and change the `$db_user` and `$db_pass` values to fit your need and move `MailWatch-DB.pm` to:

  * MailScanner v4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  
  * MailScanner v4.86.1 or V5: `/usr/share/MailScanner/perl/custom`

Copy MailWatch.pm to:

  * MailScanner v4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  
  * MailScanner v4.86.1 or V5: `/usr/share/MailScanner/perl/custom`

### Create a MailWatch web user

You need at minimum to create the administrator user 'admin' needed to access the MailWatch GUI and to handle default Spam Setting used by SQLSpamSettings.pm script.

```shell
 $ mysql mailscanner -u mailwatch -p
```
```sql
Enter password: ******
mysql> INSERT INTO users SET username = 'admin', password = MD5('<password>'), fullname = '<name>', type = 'A'
```

### Install and Configure MailWatch

* Move the `mailscanner` directory to the web server's root.

```shell
 $ mv mailscanner /var/www/html/
```

* Check the permissions of `/var/www/html/mailscanner/images` and `/var/www/html/mailscanner/images/cache` - they should be ug+rwx and owned by root and in the same group as the web server user (www-data on Debian/Ubuntu or apache on RedHat). The web server user also needs write and read access to /var/www/html/mailscanner/temp.

```shell
 $ chown root:apache images
 $ chmod ug+rwx images
 $ chown root:apache images/cache
 $ chmod ug+rwx images/cache
 $ chown root:apache temp
 $ chmod g+rw temp
```

* Create `conf.php` by copying `conf.php.example` and edit the values to suit, you will need to set `DB_USER` and `DB_PASS` to the MySQL user and password that you created earlier.

    Note that MailWatch 1.0 and later can use the quarantine more effectively when used with MailScanner version 4.43 or later as MailScanner developers added some code for MailWatch to keep track of messages quarantined by using a flag in the maillog table.

    This means that MailWatch 1.0 is *much* faster when you have a large quarantine directory.  The new quarantine report requires the use of the new functionality - so you must upgrade if you want to run this.
    The new quarantine flag is used by default and you must disable the clean.quarantine script supplied by MailScanner and use the new quarantine_maint.php script in the tools directory instead.

    To clean the quarantine - set `QUARANTINE_DAYS_TO_KEEP` in conf.php and run './quarantine_maint --clean'.  This should then be run daily from cron.
    If you are still using MailScanner 4.42 or older, updating your installation is highly recommanded; if you can't update you need to set the `QUARANTINE_USE_FLAG` to false in conf.php and use the clean.quarantine script supplied by MailScanner.

```shell
 $ cp conf.php.example conf.php
```

### Set-up MailScanner

Stop MailScanner

```shell
 $ service mailscanner stop
```

Next create the file `/etc/MailScanner/00_mailwatch.conf` - Copy the following options in it:

Setup for Exim:

```cfg
# Example config for exim  /etc/MailScanner/conf.d/00_mailwatch.conf
Run As User = Debian-exim
Run As Group = Debian-exim
MTA = exim
Incoming Work User = Debian-exim
Incoming Work Group = mtagroup
Incoming Work Permissions = 0660
Detailed Spam Reports = yes
Quarantine Whole Message = yes
Quarantine Whole Messages As Queue Files = no
Include Scores In SpamAssassin Report = yes
Quarantine User = Debian-exim
Quarantine Group = mtagroup
Quarantine Permissions = 0644
Always Looked Up Last = &MailWatchLogging
Incoming Queue Dir = /var/spool/exim4/input
Outgoing Queue Dir = /var/spool/exim4_outgoing/input
SpamAssassin Local State Dir = /var/lib/spamassassin

Sendmail = /usr/sbin/exim4
Sendmail2 = /usr/sbin/exim4 -DOUTGOING
```

Next modify the file `/etc/default/exim4` - Modify the following options according to this:

```cfg
# /etc/default/exim4
EX4DEF_VERSION=''

# 'combined' -	 one daemon running queue and listening on SMTP port
# 'no'       -	 no daemon running the queue
# 'separate' -	 two separate daemons
# 'ppp'      -   only run queue with /etc/ppp/ip-up.d/exim4.
# 'nodaemon' - no daemon is started at all.
# 'queueonly' - only a queue running daemon is started, no SMTP listener.
# setting this to 'no' will also disable queueruns from /etc/ppp/ip-up.d/exim4
QUEUERUNNER='separate'
# how often should we run the queue
QUEUEINTERVAL='15m'
# options common to quez-runner and listening daemon
COMMONOPTIONS=''
# more options for the daemon/process running the queue (applies to the one
# started in /etc/ppp/ip-up.d/exim4, too.
QUEUERUNNEROPTIONS='-DOUTGOING'
# special flags given to exim directly after the -q. See exim(8)
QFLAGS=''
# Options for the SMTP listener daemon. By default, it is listening on
# port 25 only. To listen on more ports, it is recommended to use
# -oX 25:587:10025 -oP /var/run/exim4/exim.pid
SMTPLISTENEROPTIONS='-odq'
```

Next create the file `/etc/exim4/conf.d/main/01_mailscanner_config` and copy the following options in it:

```cfg
# Config for MailScanner Gateway
# conf.d/main/01_mailscanner_config

.ifdef OUTGOING

# outgoing queue where MailScanner places scanned messages for exim to
# deliver
    SPOOLDIR = /var/spool/exim4_outgoing
    log_file_path = /var/log/exim4_outgoing/%slog
.else

# incoming queue where exim places message for MailScanner to scan
    queue_only = true
    queue_only_override = false
    SPOOLDIR = /var/spool/exim4
.endif
```

Setup for Postfix:

```cfg
# Example config for postfix /etc/MailScanner/conf.d/00_mailwatch.conf
Run As User = postfix
Run As User = mtagroup
MTA = postfix
Incoming Work User = postfix
Incoming Work Group = mtagroup
Incoming Work Permissions = 0660
Detailed Spam Report = yes
Quarantine Whole Message = yes
Quarantine Whole Messages As Queue Files = no
Include Scores In SpamAssassin Report = yes
Quarantine User = postfix
Quarantine Group = mtagroup
Quarantine Permissions = 0664
Always Looked Up Last = &MailWatchLogging
Incoming Queue Dir = /var/spool/postfix/hold
Outgoing Queue Dir = /var/spool/postfix/incoming
SpamAssassin User State Dir = /var/spool/MailScanner/spamassassin
```

Next create the file `/etc/postfix/header_checks` and copy the following options in it:

```cfg
/^Received:/ HOLD
```

Spam Actions and High Scoring Spam Actions should also have 'store' as one of the keywords if you want to quarantine items for learning/viewing in MailWatch.

### Integrate Blacklist/Whitelist (optional)

With MailWatch you can manage whitelist and blacklist from the web interface.

Copy `SQLBlackWhiteList.pm` to:

  * MailScanner v4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  
  * MailScanner v4.86.1 or v5: `/usr/share/MailScanner/perl/custom`

and in `MailScanner.conf` set:

```cfg
Is Definitely Not Spam = &SQLWhitelist
Is Definitely Spam = &SQLBlacklist
```

Move the Bayesian Databases and set-up permissions (skip this if you don't use bayes).

Edit `/etc/MailScanner/spamassassin.conf`(MailScanner v5) or `/etc/MailScanner/spam.assassin.prefs.conf`(MailScanner v4) and set:

```cfg
bayes_path /etc/MailScanner/bayes/bayes
bayes_file_mode 0660
```

Create the 'new' bayes directory, make the directory owned by a group that contains the web server user and the user your mailscanner is started with (e.g. mtagroup) and make the directory setgid:

```shell
 $ mkdir /etc/MailScanner/bayes
 $ chown root:mtagroup /etc/MailScanner/bayes
 $ chmod g+rws /etc/MailScanner/bayes
```

Copy the existing bayes databases and set the permissions:

```shell
 $ cp /root/.spamassassin/bayes_* /etc/MailScanner/bayes
 $ chown root:mtagroup /etc/MailScanner/bayes/bayes_*
 $ chmod g+rw /etc/MailScanner/bayes/bayes_*
```

Test SpamAssassin to make sure that it is using the new databases correctly:

MailScanner v5:
```shell
 $ spamassassin -D -p /etc/MailScanner/spamassassin.conf --lint
```
Or for MailScanner v4:
```shell
 $ spamassassin -D -p /etc/MailScanner/spam.assassin.prefs.conf --lint
```

and you should see something like:

```
debug: using "/etc/MailScanner/spamassassin.conf" for user prefs file
debug: bayes: 28821 tie-ing to DB file R/O /etc/MailScanner/bayes/bayes_toks
debug: bayes: 28821 tie-ing to DB file R/O /etc/MailScanner/bayes/bayes_seen
debug: bayes: found bayes db version 2
debug: Score set 3 chosen.
```

Start MailScanner up again.

```shell
 $ service mailscanner start && tail -f /var/log/maillog
```

You should see something like:

```
Jun 13 12:18:23 hoshi MailScanner[26388]: MailScanner E-Mail Virus Scanner version 5.03-3 starting...
Jun 13 12:18:24 hoshi MailScanner[26388]: Config: calling custom init function MailWatchLogging
Jun 13 12:18:24 hoshi MailScanner[26388]: Initialising database connection
Jun 13 12:18:24 hoshi MailScanner[26388]: Finished initialising database connection
```

Congratulations - you now have MailScanner logging to MySQL.

If you want to see the output of `MailScanner --lint` in Tools/MailScanner Lint (Test) edit conf.php and set MS_EXECUTABLE_PATH, the follow instruction in tools/sudo/INSTALL

### Database cleanup of maillog records

Copy `tools/Cron_jobs/mailwatch` to `/etc/cron.daily/`

You can edit `/etc/cron.daily/mailwatch` t fit your need.

Copy `tools/Cron_jobs/mailwatch_db_clean.php` to `/usr/local/bin/`

Verify if the file `mailwatch_db_clean.php` is executable. If not, use "chmod +x" to correct this problem.

You need to edit conf.php and change the RECORD_DAYS_TO_KEEP definition.

### Quarantine Maintenance

For MailScanner v4: Delete the clean.quarantine cron file.

For MailScanner v5: Change the variable for Quarantine Retention to "q_days=0" in /etc/MailScanner/defaults.

Copy `tools/Cron_jobs/mailwatch_quarantine_maint.php` to `/usr/local/bin/`

Verify if the file `mailwatch_db_clean.php` is executable. If not, use "chmod +x" to correct this problem.

You need to edit in conf.php the QUARANTINE_DAYS_TO_KEEP variable.

### Quarantine Reporting

Copy `tools/Cron_jobs/mailwatch_quarantine_report.php` to `/usr/local/bin/`

Verify if the file `mailwatch_db_clean.php` is executable. If not, use "chmod +x" to correct this problem.

### Sudo file for MailWatch

Edit `mailwatch` file to match web server user (www-data, apache or other) and MTA Queue
  and change executable path if needed.
  
Copy edited `mailwatch` file in /etc/sudoers.d/

Set permission to 0440 (`chmod 440 /etc/sudoers.d/mailwatch`)

This instruction should work in Debian 6 and 7, Ubuntu 12.04 and newer, RHEL 5 and 6 and CentOS 5 and 6.

### Test the MailWatch interface

Point your browser to http://your-server-hostname/mailscanner/ - you should be prompted for a username and password - enter the details of the MailWatch web user that you created earlier, and you should see a list of the last 50 messages processed by MailScanner.

- Update the SpamAssassin Rules table
  MailWatch keeps a list of all the SpamAssassin rules and descriptions which are displayed on the 'Message Detail' page - to show the descriptions, you need to run the updater every time you add new rules or upgrade SpamAssassin.
  Click on the 'Other' menu and select 'Update SpamAssassin Rule Descriptions' and click 'Run Now'.

- Update the GeoIP database
  Click on the 'Other' menu and select 'Update GeoIP database' and click 'Run Now'.

### Optional steps for Exim or Sendmail

#### Setup Sendmail Queue Watcher

You can get MailWatch to watch and display your sendmail queue directories.

Copy the `tools/Sendmail-Exim_queue/mailwatch_sendmail_queue.php` file in `/usr/local/bin/`

Verify if the file `mailwatch_sendmail_queue.php` is executable. If not, use "chmod +x" to correct this problem.

Setup the cronjob:

```shell
 $ crontab -e
```
```
# Run sendmail_queue.php each minute
0-59 * * * * 	/usr/local/bin/mailwatch_sendmail_queue.php
```

Note: mailwatch_sendmail_queue.php re-creates all entries on each run, so for busy sites you will probably want to change this to run every 5 minutes or greater.

#### Setup the Sendmail Relay Log watcher (optional)

You can get MailWatch to watch your Sendmail MTA logs and store all message relay information which is then displayed on the 'Message Detail' page which helps debugging and makes it easy for a Helpdesk to actually see where a message was delivered to by the MTA and what the response back was (e.g. the remote queue id etc.).

On Debian/Ubuntu:

```shell
 $ cp tools/Sendmail_relay/mailwatch_sendmail_relay.php /usr/local/bin/.
 $ cp tools/Sendmail_relay/mailwatch-sendmail-relay /etc/init.d/.
 $ chmod +x /usr/local/bin/mailwatch_sendmail_relay.php
 $ chmod +x /etc/init.d/mailwatch-sendmail-relay
 $ /etc/init.d/mailwatch-sendmail-relay start
 $ update-rc.d mailwatch-sendmail-relay defaults
```

### Optional steps for Postfix

#### Adding Postfix relay information

Copy the 'mailwatch-postfix-relay' cron file into /etc/cron.hourly/

Copy the `tools/Sendmail-Postfix_relay/mailwatch_postfix_relay.php` file in `/usr/local/bin/`

Copy the `tools/Postfix_relay/mailwatch_mailscanner_relay.php` file in `/usr/local/bin/`

Verify if the two files are executable. If not, use "chmod +x" to correct this problem.

You will find more detail in tools/Postfix_relay/INSTALL.

### Optional steps for MailScanner Rule Editor

Make sure MailWatch's `conf.php` has the following lines at the end (amend as appropriate)

```php
<?php
// Enable MailScanner Rule Editor
define('MSRE', true);
define('MSRE_RELOAD_INTERVAL', 5);
define('MSRE_RULESET_DIR', "/etc/MailScanner/rules");
```

Change file permissions so that we can update the rules and change group and rules directory locations as appropriate

```shell
 $ chgrp -R apache /etc/MailScanner/rules
 $ chmod g+rwxs /etc/MailScanner/rules
 $ chmod g+rw /etc/MailScanner/rules/*.rules
```

See also the INSTALL docs in `tools/MailScanner_rule_editor/INSTALL` and in `tools/Cron_jobs/INSTALL` directories.

### FINISHED!! (Phew!)

Please open an [issue on GitHub](https://github.com/mailwatch/1.2.0/issues) or report to the mailing-list [mailwatch-users](http://lists.sourceforge.net/lists/listinfo/mailwatch-users) if you find any errors or omissions.

Thanks!
