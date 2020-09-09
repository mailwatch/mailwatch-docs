---
layout: page
title: "Upgrading"
category: doc
date: 2017-03-02
order: 2
---

## Upgrade instructions

When a new version of MailWatch is release upgrading to it is highly suggested: security and bug fixes and new feature will be present in new releases!

You can find new release on the frontpage of [MailWatch website](http://mailwatch.org) or at [this page](https://github.com/mailwatch/1.2.0/releases) on GitHub 

Follow this procedure to make sure that the upgrade process will go as smooth as possible.

- Find a good downtime window. This process may take a while when run on big installations.

- Stop MailScanner and the MTA (Exim, Postfix or Sendmail).

- Move /var/www/html/mailscanner to /var/www/html/mailscanner-old and move the
  mailscanner directory from the tarball to /var/www/html/.

- Copy conf.php.example to conf.php and edit the file to suit your configuration.

- Replace MailWatch.pm, SQLBlackWhiteList.pm and SQLSpamSettings.pm with new version 
  from MailScanner_perl_scripts and set database connection settings in 00-MailWatch-conf.pm
  if not already done.

- Install the files in tools/Cron_jobs following INSTALL doc.

- Install the file in tools/Sendmail-Exim_queue following INSTALL doc.

- Install optional files in tools to suit your MTA (Postfix_relay or Sendmail_relay) 
  following INSTALL doc.
  
- Install the file mailwatch in tool/sudo following INSTALL doc.

- Run upgrade.php to update database schema. Look for errors and correct them if needed.

- Start MailScanner and your MTA. Check the log in /var/log/syslog.

Enjoy your upgraded MailWatch!

ATTENTION:

Several files have changed their name and location in MailWatch 1.2.0, 
particularly the PHP files that are launched in cron and the Init script files.
Please read the documentation carefully to remove old files during the upgrade.