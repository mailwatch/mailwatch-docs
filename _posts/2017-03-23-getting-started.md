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
- PHP 5.3.17 or better (with MySQL and GD support), using last stable and supported version is highly suggested

And for MailScanner CustomConfig modules to be able to use a work, you need:

- Perl
- DBI
- DBD-MySQL (full UTF-8 support exists from version 4.032, installing 4.042 is highly suggested)
- Encoding::FixLatin

Please read through the installation/upgrading instructions in their entirety before proceeding.
