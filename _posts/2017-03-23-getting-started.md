---
layout: page
title: "Getting started"
category: install
date: 2017-03-23 00:00:00
order: 0
---

## Getting Started

You must have a working MailScanner set-up and have running copies of:

- MySQL (5.7 or greater is recommended for full UTF-8 support, and MariaDB 10.1-10.5 is reported working well too)
- Apache / Nginx / Caddy
- PHP 5.6 or better (with MySQL and GD support), using last stable and supported version is highly suggested

And for MailScanner CustomConfig modules to be able to use a work, you need:

- Perl
- DBI
- DBD::MariaDB
- Encoding::FixLatin
- Digest::SHA

Please read through the installation/upgrading instructions in their entirety before proceeding.

### Upgrading DBD::MariaDB

Linux distributions don't always ship updated DBD::MariaDB version: for example Debian 10 ships version 1.11, which isn't fully compatible with latest MariaDB Server.

MailWatch works well with DBD::MariaDB version 1.22 or greater, which fully supports UTF8MB4 and a great range of MySQL and MariaDB server versions; to verify your installed DBD::MariaDB version use the following command

```shell
$ perl -MDBD::MariaDB -e 'print $DBD::MariaDB::VERSION'
```

You can update installed DBD::MariaDB package using cpan:

```shell
$ cpan -i DBD::MariaDB
```

Verify that your installed DBD::MariaDB version is at least 1.22, use latest version when possible:

```shell
$ perl -MDBD::mysql -e 'print $DBD::mysql::VERSION'
```

### Installing Encoding::FixLatin

Encoding::FixLatin can be installed with this simple command:

```shell
$ cpan -i Encoding::FixLatin
```

### Installing Digest::SHA

Digest::SHA1 can be installed with this command:

```shell
$ cpan -i Digest::SHA
```
