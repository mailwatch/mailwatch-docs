---
layout: page
title: "Getting started"
category: install
date: 2017-03-23 00:00:00
order: 0
---

## Getting Started

You must have a working MailScanner set-up and have running copies of:

- MySQL (5.6 is recommended for full UTF-8 support, and MariaDB 10.x is reported working well too)
- Apache or nginx
- PHP 5.3.17 or better (with MySQL and GD support), using last stable and supported version is highly suggested. For GeoIP2 support PHP 5.4 or higher is required

And for MailScanner CustomConfig modules to be able to use a work, you need:

- Perl
- DBI
- DBD::mysql (full UTF-8 support exists from version 4.032, but installing 4.042 is highly suggested)
- Encoding::FixLatin
- Digest::SHA1

Please read through the installation/upgrading instructions in their entirety before proceeding.

### Upgrading DBD::mysql

Linux distributions don't always ship updated DBD::mysql version: for example Debian 8 ships version 4.028, which can't manage UTF-8 very well.

MailWatch works well with DBD::mysql version 4.042 or greater, which supports utf8mb4 charset used by MySQL; to verify your installed DBD::mysql version use the following command

```shell
$ perl -MDBD::mysql -e 'print $DBD::mysql::VERSION'
```

You can update installed DBD::mysql package using cpan, which require also development files for mysql client:

```shell
$ apt-get install libmysqlclient-dev
$ cpan -i DBD::mysql
```

Verify that your installed DBD::mysql version is at least 4.032, but 4.042 is highly suggested

```shell
$ perl -MDBD::mysql -e 'print $DBD::mysql::VERSION'
```

### Installing Encoding::FixLatin

Encoding::FixLatin can be installed with this simple command:

```shell
$ cpan -i Encoding::FixLatin
```

### Installing Digest::SHA1

Digest::SHA1 can be installed with this command:

```shell
$ cpan -i Digest::SHA1
```
