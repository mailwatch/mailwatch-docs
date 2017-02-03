---
layout: page
title: "Upgrading"
category: doc
date: 2017-02-03 17:47:00
order: 2
---

## Upgrade instructions

When a new version of MailWatch is release upgrading to it is highly suggested: security and bug fixes and new feature will be present in new releases!

You can find new release on the frontpage of [MailWatch website](http://mailwatch.org) or at [this page](https://github.com/mailwatch/1.2.0/releases) on GitHub 

Follow this procedure to make sure that the upgrade process will go as smooth as possible.

* find a good downtime window, this process may take a while when run on big installations
* create a backup of your MailWatch database, safety first!
* make a copy of `conf.php` file
* replace all old files with the ones from new release
* copy `conf.php` in `mailscanner` directory
* run `php upgrade.php`
* eventually adjust `conf.php` with new configuration entries that `upgrade.php` warned you about
* copy and edit `MailWatch.pm`, `SQLBlackWhiteList.pm` and `SQLSpamSettings.pm` to Mailscanner custom function directory
* enjoy your upgraded MailWatch!