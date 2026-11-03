---
layout: page
title: "Upgrading"
category: install
date: 2017-03-23 00:00:00
order: 3
---

## Upgrade instructions

When a new version of MailWatch is released, upgrading to it is highly suggested: security and bug fixes, and new features will be present in new releases!

You can find the new release on the frontpage of [MailWatch website](http://mailwatch.org) or at [releases page](https://github.com/mailwatch/MailWatch/releases) on GitHub 

Follow this procedure to make sure that the upgrade process will go as smoothly as possible.

### Preliminary steps
* find a good downtime window, this process may take a while when run on big installations
* create a backup of your MailWatch database, safety first!
* read release notes to check if any files change name or path
* stop your mail flow (e.g.: stop your MTA or block by firewall)
* stop MailScanner
* make a copy of `conf.php` file

There are two install methods: zip install or git cloning: upgrade procedure differs a bit.

### Zip install
* download the new release zip file
* uncompress it in a temporary directory
* remove the old installed directory and replace it with the new one
* copy `conf.php` in `mailscanner` directory
* replace all cron files with the ones from the new release

### Git cloning
* cd into MailWatch root directory
* `git pull`

### Continue upgrade procedure

* run `php upgrade.php`
* eventually adjust `conf.php` with new configuration entries that `upgrade.php` warned you about
* symlink, if not already done, `MailWatch.pm`, `SQLBlackWhiteList.pm`, and `SQLSpamSettings.pm` from `MailScanner_perl_scripts` directory to MailScanner custom function directory and copy and edit `MailWatchConf.pm` to match the SQL settings
* start MailScanner
* restart your mail flow (e.g.: start MTA or unblock by firewall)
* check the mail server logs (e.g.: /var/log/mail.log and /var/log/syslog) 

enjoy your upgraded MailWatch!

### Warning
Several files have changed their name and location in MailWatch 1.2.0, particularly the PHP files that are launched in cron and the Init script files.

Please read the documentation carefully to remove old files during the upgrade.

## Upgrading from 1.2.28 to 1.2.29
* Run `php upgrade.php`: it creates the new `login_failures` table and lists the users whose password was set to `xxxxxxxx` by the own settings form. Those users must get a new password. Until `upgrade.php` has run, logins work as before but failed logins are not limited.
* Add the new `conf.php` entries `LOGIN_MAX_FAILURES` and `LOGIN_LOCKOUT_MINUTES` (see Optional Setup, "Failed login limit"); without them the defaults are 5 failures and 15 minutes.
* `DB_CLEAN_OPTIMIZE` now defaults to `false`: `mailwatch_db_clean.php` runs `OPTIMIZE TABLE` only when you set it to `true`.
* Copy the updated tools over the installed ones: `tools/Cron_jobs/mailwatch` (in `/etc/cron.daily/`), `tools/Cron_jobs/mailwatch.cron.monthly` (in `/etc/cron.monthly/mailwatch`), the `mailwatch_*.php` scripts from `tools/Cron_jobs/` in `/usr/local/bin/`, the Postfix and Sendmail relay scripts and `tools/LDAP/mailwatch_ldap_sync.sh`, then restart the relay services. The cron jobs now send their output to syslog with the tag `mailwatch-<job>`.

## Upgrading from 1.2.23 to 1.2.24
`DBD:mysql` was replace by `DBD::MariaDB` to fully support UTF8 and MariaDB server, you need to install the new perl library:

```shell
 $ cpan -i DBD::MariaDB
```
and copy the updated perl modules from `MailWatch_perl_scripts` directory to MailScanner custom function directory.

## Upgrading from 1.2.0 to 1.2.1
`00MailWatchConf.pm` was renamed to `MailWatchConf.pm` because it was failing on some perl versions: rename your file accordly and restart MailScanner

```shell
 $ mv /usr/share/MailScanner/perl/custom/00MailWatchConf.pm /usr/share/MailScanner/perl/custom/MailWatchConf.pm
 $ service restart mailscanner
```
