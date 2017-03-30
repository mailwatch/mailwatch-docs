---
layout: page
title: "Getting started"
category: doc
date: 2017-02-26 18:35:26
order: 0
---

## Getting Started

You must have a working MailScanner set-up and have running copies of:

- MySQL 5.5+ or MariaDB 10.x
- Apache or nginx
- PHP 5.4+ (with MySQL and GD support)

And for MailScanner CustomConfig modules to be able to use a database, you need:

- Perl
- DBI
- DBD-MySQL

You also need Perl Encoding::FixLatin to deal with email subjects that contain characters in more than one encoding.

Please read through the installation/upgrading instructions in their entirety before proceeding.