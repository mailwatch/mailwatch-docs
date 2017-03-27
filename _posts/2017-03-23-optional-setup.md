---
layout: page
title: "Optional and MTA dependant setup"
category: install
date: 2017-03-23 00:00:00
order: 2
---
## Optional Setup
MailWatch experience can be enhanced by some optional configuration step that 


### Integrate Blacklist/Whitelist
With MailWatch you can manage whitelist and blacklist from the web interface.

In `MailScanner.conf` you must set:

```cfg
Is Definitely Not Spam = &SQLWhitelist
Is Definitely Spam = &SQLBlacklist
```

### Postfix

#### Adding Postfix relay information to message detail

You can get MailWatch to watch your Postfix MTA logs and store all message relay information which is then displayed on the 'Message Detail' page which helps debugging and makes it easy for a Helpdesk to actually see where a message was delivered to by the MTA and what the response back was (e.g. the remote queue id etc.).

* Copy `tools/Postfix_relay/mailwatch-postfix-relay` cron file into `/etc/cron.hourly/`
* Copy in /usr/local/bin the following files:

  - `tools/Postfix_relay/mailwatch_postfix_relay.php`
  - `tools/Postfix_relay/mailwatch_mailscanner_relay.php`


### Exim


### Sendmail

#### Setup Sendmail Queue Watcher

The `mailwatch_sendmail_queue.php` script process the MTA Exim or Sendmail queue to store messages in MailWatch database and see them in the MailWatch GUI.

Copy the `mailwatch_sendmail_queue.php` file in `/usr/local/bin` and verify if the file is executable: if not, use "chmod +x" to correct this problem.

```shell
 $ cp tools/Sendmail_queue/mailq.php /usr/local/bin
 $ crontab -e

 # Run each minute
 0-59 * * * * 	/usr/local/bin/mailwatch_sendmail_queue.php
```

Note: `mailwatch_sendmail_queue.php` re-creates all entries on each run, so for busy sites you will probably want to change this to run every 5 minutes or greater.

#### Setup the Sendmail Relay Log watcher (optional)

You can get MailWatch to watch your Sendmail MTA logs and store all message relay information which is then displayed on the 'Message Detail' page which helps debugging and makes it easy for a Helpdesk to actually see where a message was delivered to by the MTA and what the response back was (e.g. the remote queue id etc.).

On Debian/Ubuntu:
```shell
# cp tools/Sendmail_relay/mailwatch_sendmail_relay.php /usr/local/bin/.
# cp tools/Sendmail_relay/mailwatch-sendmail-relay /etc/init.d/.
# chmod +x /usr/local/bin/mailwatch_sendmail_relay.php
# chmod +x /etc/init.d/mailwatch-sendmail-relay
# /etc/init.d/mailwatch-sendmail-relay start
# update-rc.d mailwatch-sendmail-relay defaults
```
For others Linux distributions, please change according to.

By default, `mailwatch_sendmail_relay.php` run with 'root' user. Change user to your webserver or Sendmail MTA user (check right one on /var/log/mail.log).


### MailScanner Rule Editor
Make sure MSRE (MailScanner Rule Editor) is enabled in MailWatch's `conf.php`:

```php
<?php
// Enable MailScanner Rule Editor
define('MSRE', true);
define('MSRE_RELOAD_INTERVAL', 5);
define('MSRE_RULESET_DIR', '/etc/MailScanner/rules');
```

Change file permissions so that we can update the rules, and change group and rules directory locations as appropriate

```shell
 $ chgrp -R apache /etc/MailScanner/rules
 $ chmod g+rwxs /etc/MailScanner/rules
 $ chmod g+rw /etc/MailScanner/rules/*.rules
```

See also the INSTALL docs in `tools/MailScanner_rule_editor` and `tools/Cron_jobs` directories.

### LDAP directory for user management

You can use a LDAP directory to authenticate users. Setting `USE_LDAP` to `true` in `conf.php` will enable the backend and will connect to the ldap server `LDAP_HOST` on the port `LDAP_PORT` and binds to it by using `LDAP_USER` and `LDAP_PASS` as credentials. That user must have read access to the users login name and attributes that you are using for the filter.

MailWatch will search users that are allowed to use it by using `LDAP_DN` as search base and `LDAP_FILTER` to filter the ldap entries to get the matching users login name. In the `LDAP_FILTER` you can use `%s` as replacement for the users login name that he entered on the login page of MailWatch.

From this search result MailWatch uses the ldap attribute defined by `LDAP_USERNAME_FIELD` as login name to bind to the server for authentication.
This login name can be extended by a prefix (`LDAP_BIND_PREFIX`) and suffix (`LDAP_BIND_SUFFIX`).
The user then is authenticated with his password and this aggregated login name. Additionally you can define the ldap attribute containing the users mail address with `LDAP_EMAIL_FIELD` which will be used to only show the user mails that are related to his mail address as recipient or sender.

Settings for Active Directory:

```php
define('LDAP_FILTER', 'mail=%s');
define('LDAP_EMAIL_FIELD', 'mail');
define('LDAP_USERNAME_FIELD', 'userprincipalname');
```

<!--%TODO check openldap settings-->
Example for OpenLDAP:

```php
define('LDAP_FILTER', 'mail=%s'); 
define('LDAP_EMAIL_FIELD', 'mail');
define('LDAP_USERNAME_FIELD', 'cn');
define('LDAP_BIND_PREFIX', 'cn=');
define('LDAP_BIND_SUFFIX', ',dc=example,dc=com');
```

### Web interface skinning

From 1.2.1 it is possible to add `skin.css` file to `/opt/mailwatch/mailscanner` directory with your custom css rules.

This file will not be overwritten by git upgrade, but remember to back it up if upgrading with zip method.
