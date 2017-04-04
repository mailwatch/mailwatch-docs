---
layout: page
title: "Installation"
category: install
date: 2017-03-23 00:00:00
order: 1
---

## Installation instructions

MailWatch for MailScanner is developed on Debian 7 & Ubuntu 12.04, so these docs will reflect this and I will make note on anything that will be required to run on other distro's or operating systems.

If you're upgrading from a previous version, read [upgrading instructions]({% post_url 2017-03-23-upgrading %}).

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

PHP should have at least the following set in php.ini (possibly others too):

```ini
safe_mode = Off
register_globals = Off
magic_quotes_gpc = Off
magic_quotes_runtime = Off
session.auto_start = 0
```

## Installation

All commands below should be run as 'root' user.

### Installing MailWatch
There are 2 methods to install MailWatch: zip install and git cloning.

In the following instructions `/opt/mailwatch` is used as base directory for MailWatch Install, if you prefer you can use another directory that you like.

#### Zip install

```shell
mkdir /opt/mailwatch
cd /opt/mailwatch
wget https://github.com/mailwatch/MailWatch/archive/v1.2.2.zip
unzip v1.2.2.zip -d /opt/mailwatch
rm v1.2.2.zip
```

#### Git cloning
```shell
mkdir /opt/mailwatch
cd /opt/mailwatch
git clone --depth=1 https://github.com/mailwatch/MailWatch.git .
```

### Create the database

```shell
 $ mysql < create.sql
```

NOTE: you will need to modify the above as necessary for your system if you have a root password for your MySQL database (which is highly recommended!).

### Create a MySQL user and password & Set-up MailScanner for SQL logging

```shell
 $ mysql
```
```sql
mysql> GRANT ALL ON mailscanner.* TO mailwatch@localhost IDENTIFIED BY '<password>';
mysql> GRANT FILE ON *.* TO mailwatch@localhost IDENTIFIED BY '<password>';
mysql> FLUSH PRIVILEGES;
mysql> quit
```

Copy `MailWatchConf.pm` in MailScanner CustomFunctions directory and edit it to match your database settings.
MailScanner CustomFunctions path depends on MailScanner version:

  * MailScanner V4: `/usr/lib/MailScanner/MailScanner/CustomFunctions` (this could be `/opt/MailScanner/lib/MailScanner/MailScanner/CustomFunctions` on non-RPM systems).
  * MailScanner V4.86.1 or V5: `/usr/share/MailScanner/perl/custom`

Symlink the others .pm files in the same MailScanner CustomFunctions directory:

```shell
 $ ln -s /opt/mailwatch/MailScanner_perl_scripts/MailWatch.pm /usr/share/MailScanner/perl/custom
 $ ln -s /opt/mailwatch/MailScanner_perl_scripts/SQLBlackWhiteList.pm /usr/share/MailScanner/perl/custom
 $ ln -s /opt/mailwatch/MailScanner_perl_scripts/SQLSpamSettings.pm /usr/share/MailScanner/perl/custom
```
Symlinking instead of copying assure a painless upgrade in the future.

### Create a MailWatch web user

```shell
 $ mysql mailscanner -u mailwatch -p
```
```sql
Enter password: ******
mysql> INSERT INTO users SET username = '<username>', password = MD5('<password>'), fullname = '<name>', type = 'A';
```

### Install & Configure MailWatch

#### Checking permissions

Check the permissions of `/opt/mailwatch/mailscanner/images` and `/opt/mailwatch/mailscanner/images/cache` - they should be ug+rwx and owned by root and in the same group as the web server user (www-data on Debian/Ubuntu or apache on RedHat). The web server user also needs write and read access to /opt/mailwatch/mailscanner/temp.

```shell
 $ chown root:apache images
 $ chmod ug+rwx images
 $ chown root:apache images/cache
 $ chmod ug+rwx images/cache
 $ chown root:apache temp
 $ chmod g+rw temp
```

#### Edit conf.php
Create `conf.php` by copying `conf.php.example` and edit the values to suit, you will need to set `DB_USER` and `DB_PASS` to the MySQL user and password that you created earlier.

```shell
 $ cp conf.php.example conf.php
```

Note that MailWatch 1.0 and later can use the quarantine more effectively when used with MailScanner version 4.43 or later as MailScanner developers added some code for MailWatch to keep track of messages quarantined by using a flag in the maillog table.

This means that MailWatch 1.0 is *much* faster when you have a large quarantine directory.  The new quarantine report requires the use of the new functionality - so you must upgrade if you want to run this.
The new quarantine flag is used by default and you must disable the clean.quarantine script supplied by MailScanner and use the new quarantine_maint.php script in the tools directory instead.

To clean the quarantine - set `QUARANTINE_DAYS_TO_KEEP` in conf.php and run './quarantine_maint --clean'.  This should then be run daily from cron.
If you are still using MailScanner 4.42 or older, updating your installation is highly recommanded; if you can't update you need to set the `QUARANTINE_USE_FLAG` to false in conf.php and use the clean.quarantine script supplied by MailScanner.


#### Configure webserver

It's recommended to setup a virtualhost to manage MailWatch; this setup depends on your webserver of choice.
Below you'll find basic config example for apache and nginx, adapt them to your prefered setup (ssl, additional header, and so on).

#### nginx
```apacheconfig
server {
    listen 80 default_server;
    server_name mailwatch.example.org;

    access_log /var/log/nginx/mailwatch/access.log main_ext;
    error_log /var/log/nginx/mailwatch/error.log warn;

    root /opt/mailwatch/mailscanner;

    location / {
        index  index.php;
    }
    
    location ~* \.php$ {
        fastcgi_index   index.php;
        fastcgi_pass    unix:/var/run/php5-fpm.sock;
        include         fastcgi.conf;
        fastcgi_param   SCRIPT_FILENAME    $document_root$fastcgi_script_name;
        fastcgi_param   SCRIPT_NAME        $fastcgi_script_name;
    }
}
```

#### Apache
```apacheconfig
<virtualhost *:80>
    ServerName mailwatch.example.org
    DocumentRoot "/opt/mailwatch/mailscanner"
    ErrorLog "/var/log/apache/mailwatch/error.log"
    CustomLog "/var/log/apache/mailwatch/access.log" combined

    <Directory />
        # Apache 2.4
        Require all granted
        
        # Apache 2.2
        #Order allow,deny
        #Allow from all
    </Directory>

    AddType application/x-httpd-php .php
    DirectoryIndex index.php
</virtualhost>
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

### Move the Bayesian Databases and set-up permissions (skip this if you don't use bayes).

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

add `mailwatch_db_clean.php` to `/etc/cron.daily/`

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

Point your browser to http://your-mailwatch-virtualhost/ - you should be prompted for a username and password - enter the details of the MailWatch web user that you created earlier, and you should see a list of the last 50 messages processed by MailScanner.

- Update the SpamAssassin Rules table
  MailWatch keeps a list of all the SpamAssassin rules and descriptions which are displayed on the 'Message Detail' page - to show the descriptions, you need to run the updater every time you add new rules or upgrade SpamAssassin.
  Click on the 'Other' menu and select 'Update SpamAssassin Rule Descriptions' and click 'Run Now'.

- Update the GeoIP database
  Click on the 'Other' menu and select 'Update GeoIP database' and click 'Run Now'.

### FINISHED!! (Phew!)

Please open an [issue on GitHub](https://github.com/mailwatch/MailWatch/issues) or report to the mailing-list [mailwatch-users](http://lists.sourceforge.net/lists/listinfo/mailwatch-users) if you find any errors or omissions.

Thanks!
