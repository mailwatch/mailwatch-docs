---
layout: page
title: "Upgrading"
category: install
date: 2017-03-23 00:00:00
order: 3
---

## Upgrade instructions

When a new version of MailWatch is release upgrading to it is highly suggested: security and bug fixes, and new feature will be present in new releases!

You can find new release on the frontpage of [MailWatch website](http://mailwatch.org) or at [releases page](https://github.com/mailwatch/1.2.0/releases) on GitHub 

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
* copy `MailWatch.pm`, `SQLBlackWhiteList.pm`, `SQLSpamSettings.pm` and `MailWatchConf.pm` (the last one only if explicitly stated in release note) to Mailscanner custom function directory and edit `MailWatchConf.pm` to match the sql settings
* start MailScanner
* restart your mail flow (e.g.: start MTA or unblock by firewall)
* Check the mail server logs (e.g.: /var/log/mail.log and /var/log/syslog) 
* enjoy your upgraded MailWatch!
