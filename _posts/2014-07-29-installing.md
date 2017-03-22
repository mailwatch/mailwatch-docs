---
layout: page
title: "Installation"
category: doc
date: 2014-07-29 18:35:27
order: 1
---

## Installation instructions

MailWatch for MailScanner is developed on Debian 7 & Ubuntu 12.04, so these docs will reflect this and I will make note on anything that will be required to run on other distro's or operating systems.

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

Edit MailWatch.pm and change the `$db_user` and `$db_pass` values (around line 42) accordingly and move `MailWatch.pm` to:

  * MailScanner V4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  * MailScanner V4.86.1 or V5: `/usr/share/MailScanner/perl/custom`

### Create a MailWatch web user

```shell
 $ mysql mailscanner -u mailwatch -p
```
```sql
Enter password: ******
mysql> INSERT INTO users SET username = '<username>', password = MD5('<password>'), fullname = '<name>', type = 'A'
```

### Install & Configure MailWatch

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
 $ service MailScanner stop
```

Next edit `/etc/MailScanner/MailScanner.conf` - you need to make sure that the following options are set:

```cfg
Always Looked Up Last = &MailWatchLogging
Detailed Spam Report = yes
Quarantine Whole Message = yes
Quarantine Whole Messages As Queue Files = no
Include Scores In SpamAssassin Report = yes
Quarantine User = root
Quarantine Group = apache (this should be the same group as your web server)
Quarantine Permissions = 0660
```

If you are using Exim or Postfix you also have to adjust the following settings (replace `Debian-exim` with `postfix` when using postfix :

```cfg
Run As User = Debian-exim
Run As Group = Debian-exim
Incoming Work User = Debian-exim
Incoming Work Group = mtagroup
Incoming Work Permissions = 0660
Quarantine User = Debian-exim
Quarantine Group = mtagroup
Quarantine Permissions = 0644
```

Spam Actions and High Scoring Spam Actions should also have 'store' as one of the keywords if you want to quarantine items for learning/viewing in MailWatch.

### Integrate Blacklist/Whitelist (optional)
With MailWatch you can manage whitelist and blacklist from the web interface.

Edit `SQLBlackWhiteList.pm` file and change the connection string in the `CreateList` subroutine (lines 103-106) to match `MailWatch.pm`.

Copy `SQLBlackWhiteList.pm` to:

  * MailScanner V4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  * MailScanner V4.86.1 or V5: `/usr/share/MailScanner/perl/custom`

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

Create the 'new' bayes directory, make the directory owned by the main group of the web server (eg. apache or www-data; a supplemental group won't work) and the user your mailscanner is started with (eg. postfix/Debian-exim) and make the directory setgid:

```shell
 $ mkdir /etc/MailScanner/bayes
 $ chown postfix:apache /etc/MailScanner/bayes
 $ chmod g+rws /etc/MailScanner/bayes
```

Copy the existing bayes databases and set the permissions:

```shell
 $ cp /root/.spamassassin/bayes_* /etc/MailScanner/bayes
 $ chown postfix:apache /etc/MailScanner/bayes/bayes_*
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
 $ service MailScanner start && tail -f /var/log/maillog
```

You should see something like:

```
Jun 13 12:18:23 hoshi MailScanner[26388]: MailScanner E-Mail Virus Scanner version 4.20-3 starting...
Jun 13 12:18:24 hoshi MailScanner[26388]: Config: calling custom init function MailWatchLogging
Jun 13 12:18:24 hoshi MailScanner[26388]: Initialising database connection
Jun 13 12:18:24 hoshi MailScanner[26388]: Finished initialising database connection
```

Congratulations - you now have MailScanner logging to MySQL.

If you want to see the output of `MailScanner --lint` in Tools/MailScanner Lint (Test) edit conf.php and set MS_EXECUTABLE_PATH, the follow instruction in tools/sudo/INSTALL

### Database cleanup of maillog records

add `db_clean.php` to `/etc/cron.daily/`

You will then to edit `conf.php` the RECORD_DAYS_TO_KEEP definition.

You will need to edit the `db_clean.php` to reflect the location of the `functions.php` file

### Quarantine Maintenance

Remove the clean.quarantine cronjob configured with MailScanner.

Edit and copy `quarantine_maint.sh` to `/etc/cron.daily/`

You will then to edit `conf.php` the QUARANTINE_DAYS_TO_KEEP definition.

You will need to edit the `quarantine_maint.php` to reflect the location of the `functions.php` file

### Quarantine Reporting

Add `quarantine_report.php` to `/etc/cron.daily`

You will need to edit the `quarantine_report.php` to reflect the location of the `functions.php` file

### Test the MailWatch interface

Point your browser to http://your-server-hostname/mailscanner/ - you should be prompted for a username and password - enter the details of the MailWatch web user that you created earlier, and you should see a list of the last 50 messages processed by MailScanner.

- Update the SpamAssassin Rules table
  MailWatch keeps a list of all the SpamAssassin rules and descriptions which are displayed on the 'Message Detail' page - to show the descriptions, you need to run the updater every time you add new rules or upgrade SpamAssassin.
  Click on the 'Other' menu and select 'Update SpamAssassin Rule Descriptions' and click 'Run Now'.

- Update the GeoIP database
  Click on the 'Other' menu and select 'Update GeoIP database' and click 'Run Now'.

### Optional steps for Sendmail

#### Setup Sendmail Queue Watcher

You can get MailWatch to watch and display your sendmail queue directories.

Edit `tools/Sendmail_queue/mailq.php` to change the require line to point to the location of `functions.php`; copy `mailq.php` file to `/usr/local/bin` and setup the cronjob:

```shell
 $ cp tools/Sendmail_queue/mailq.php /usr/local/bin
 $ crontab -e

 0-59 * * * * 	/usr/local/bin/mailq.php
```

Note: `mailq.php` re-creates all entries on each run, so for busy sites you will probably want to change this to run every 5 minutes or greater.

#### Setup the Sendmail Relay Log watcher (optional)

You can get MailWatch to watch your sendmail logs and store all message relay information which is then displayed on the 'Message Detail' page which helps debugging and makes it easy for a Helpdesk to actually see where a message was delivered to by the MTA and what the response back was (e.g. the remote queue id etc.).

```shell
 $ cp tools/Sendmail_relay/sendmail_relay.php /usr/local/bin
 $ cp tools/Sendmail_relay/sendmail_relay.init /etc/rc.d/init.d/
 $ chmod 777 /etc/rc.d/init.d/sendmail_relay.init
 $ /etc/rc.d/init.d/sendmail_relay.init start
 $ ln -s /etc/rc.d/init.d/sendmail_relay.init /etc/rc.2/S30sendmail_relay.init
```

### Optional steps for Postfix

#### Adding Postfix relay information

* Add the table to the database

* Edit `mailwatch_relay.sh` and modify it to point to your php location and correct MailWatch installation path
* Add `mailwatch_relay.sh` as an hourly cron job
* Fix permissions of the postfix queue so that mailwatch can monitor them:

``` shell
 $ chown postfix.apache /var/spool/postfix/incoming/
 $ chown postfix.apache /var/spool/postfix/hold
 $ chmod g+r /var/spool/postfix/hold
 $ chmod g+r /var/spool/postfix/incoming/
```

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

See also the INSTALL docs in `tools/MailScanner_rule_editor` and `tools/Cron_jobs` directories.

### Optional steps to use LDAP directory for user management

You can use a LDAP directory to authenticate users. `define('USE_LDAP', true);` will enable the backend and will connect to the ldap server `LDAP_HOST` on the port `LDAP_PORT` and binds to it by using `LDAP_USER` and `LDAP_PASS` as credentials. That user must have read access to the users login name and attributes that you are using for the filter.

MailWatch will search users that are allowed to use it by using `LDAP_DN` as search base and `LDAP_FILTER` to filter the ldap entries to get the matching users login name. In the `LDAP_FILTER` you can use `%s` as replacement for the users login name that he entered on the login page of MailWatch.

From this search result MailWatch uses the ldap attribute defined by `LDAP_USERNAME_FIELD` as login name to bind to the server for authentication. This login name can be extended by a prefix (`LDAP_BIND_PREFIX`) and suffix (`LDAP_BIND_SUFFIX`). The user then is authenticated with his password and this aggregated login name. Additionally you can define the ldap attribute containing the users mail address with `LDAP_EMAIL_FIELD` which will be used to only show the user mails that are related to his mail address as recipient or sender.

Settings for Active Directory:
```
define('LDAP_FILTER', 'mail=%s');
define('LDAP_EMAIL_FIELD', 'mail');
define('LDAP_USERNAME_FIELD', 'userprincipalname');
```
<!--%TODO check openldap settings-->
Example for OpenLDAP:
```
define('LDAP_FILTER', 'mail=%s'); 
define('LDAP_EMAIL_FIELD', 'mail');
define('LDAP_USERNAME_FIELD', 'cn');
define('LDAP_BIND_PREFIX', 'cn=');
define('LDAP_BIND_SUFFIX', ',dc=example,dc=com');
```

### FINISHED!! (Phew!)

Please open an [issue on GitHub](https://github.com/mailwatch/1.2.0/issues) or report to the mailing-list [mailwatch-users](http://lists.sourceforge.net/lists/listinfo/mailwatch-users) if you find any errors or omissions.

Thanks!
