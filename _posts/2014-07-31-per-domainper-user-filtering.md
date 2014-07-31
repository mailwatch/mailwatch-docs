---
layout: page
title: "Per Domain/Per User Filtering"
category: using
date: 2014-07-31 12:37:54
order: 1
---

## Per Domain/Per User Filtering

MailWatch for MailScanner 1.0 has implemented new filtering capabilities to be able to support per-domain filtering or per-user filtering more effectively than previously.

To utilise this new functionality - all you need to do is create MailWatch users named by either their domain or their e-mail address and set their user type accordingly.

For example:

* If you create a user named `smf@example.com` as user type `User` and I log-in as that user, I will only be able to see e-mail address to/from me and to be able to add Blacklist/Whitelist entries for my address (if enabled).
* If I create a user named `example.com` as type `Domain Administrator` and I log-in as that user, I will only be able to see messages to/from my domain or create blacklist/whitelist entries for the entire domain or for a specific user.
* The `Administrator` type can do anything for any user or domain.
* If you need to have 'aliases' for your users - e.g. 'smf@example.com' also has an e-mail alias 'steve.freegard@example.org', then no problem - use the 'Filters' screen to add 'steve.freegard@example.org' and the 'smf@example.com' user will be able to see both.

Easy!
