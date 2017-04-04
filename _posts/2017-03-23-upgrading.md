---
layout: page
title: "Upgrading"
category: install
date: 2017-03-23 00:00:00
order: 3
---

## Upgrade instructions

When a new version of MailWatch is release upgrading to it is highly suggested: security and bug fixes, and new feature will be present in new releases!

You can find new release on the frontpage of [MailWatch website](http://mailwatch.org) or at [releases page](https://github.com/mailwatch/MailWatch/releases) on GitHub 

Follow this procedure to make sure that the upgrade process will go as smooth as possible.

### Preliminary steps
* find a good downtime window, this process may take a while when run on big installations
* create a backup of your MailWatch database, safety first!
* stop your mail flow (e.g.: stop yor MTA or block by firewall)
* stop MailScanner
* make a copy of `conf.php` file

There are 2 install methods: zip install or git cloning: upgrade procedure differs a bit.

### Zip install
* download the new release zip file
* uncompress it in a temporary directory
* remove old installed directory and replace it with the new one
* copy `conf.php` in `mailscanner` directory
* replace all cron files with the ones from new release

### Git cloning
* cd into MailWatch root directory
* `git pull`

### Continue upgrade procedure

* run `php upgrade.php`
* eventually adjust `conf.php` with new configuration entries that `upgrade.php` warned you about
* symlink, if not already done, `MailWatch.pm`, `SQLBlackWhiteList.pm`, and `SQLSpamSettings.pm` from `MailScanner_perl_scripts` directory to MailScanner custom function directory and copy and edit `MailWatchConf.pm` to match the sql settings
* start MailScanner
* restart your mail flow (e.g.: start MTA or unblock by firewall)
* check the mail server logs (e.g.: /var/log/mail.log and /var/log/syslog) 

enjoy your upgraded MailWatch!

### Warning
Several files have changed their name and location in MailWatch 1.2.0, particularly the PHP files that are launched in cron and the Init script files.

Please read the documentation carefully to remove old files during the upgrade.

## Upgrading from 1.2.0 to 1.2.1
`00MailWatchConf.pm` was renamed to `MailWatchConf.pm` because it was failing on some perl versions: rename your file accordly and restart MailScanner

```shell
 $ mv /usr/share/MailScanner/perl/custom/00MailWatchConf.pm /usr/share/MailScanner/perl/custom/MailWatchConf.pm
 $ service restart mailscanner
```
