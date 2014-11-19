---
layout: page
title: "Contributing"
category: dev
date: 2014-07-30 12:49:27
order: 0
---

## Contributing

**MailWatch** is open source software; contributions from the community are encouraged. Please take a moment to read these guidelines before submitting changes.

### Getting Help with using MailWatch
For discussion, questions, and help with implementation, we use a mailing list named [mailwatch-users](http://lists.sourceforge.net/lists/listinfo/mailwatch-users) hosted on SourceForge

### Requesting New Features

If you've got a feature request for MailWatch, file it with the [GitHub issue tracker]((https://github.com/mailwatch/1.2.0/issues)).
We're aiming to create an useful utility, and would love to hear your ideas.

### Submitting Bug Reports

If you find a problem with MailWatch, whether you are stuck or want to address it yourself, first log it using the [GitHub issue tracker](https://github.com/mailwatch/1.2.0/issues).
That way, we can take a look and let you know how best to proceed, and if work is already underway on this issue.

Note: The issue tracker is for bugs and new features, not help requests. For procedural questions - like help - see the above "Getting Help" section.

### Contributing to MailWatch

#### Getting Started
* Make sure you have a [GitHub account](https://github.com/signup/free).
* [Submit an issue](https://github.com/mailwatch/1.2.0/issues) for your issue, assuming one does not already exist.
   * Clearly describe the issue and your enviroment including steps to reproduce when it is a bug.
   * Make sure you fill in the earliest version that you know has the issue.
* Fork the repository on GitHub or do a hard reset (first [add upstream to your fork](https://help.github.com/articles/configuring-a-remote-for-a-fork/) and then issue a `git fetch upstream && git reset --hard upstream/master && git push` command) to update an existing fork.

#### Branching and pull requests
As a guideline, please follow this process:

 1. [Fork the repository](https://help.github.com/articles/fork-a-repo).
 2. Create a topic branch for the change:
    * Please ensure the branch is clearly labelled as a feature or fix.
    * Avoid working directly on the **master** branch.
 3. Make the relevant changes.
 4. [Squash](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Changing-Multiple-Commit-Messages) commits if necessary.
 5. It should go without saying, but do not increment the version number in your commits.
 6. [Submit a pull request](https://help.github.com/articles/using-pull-requests) to the **master** branch.

Please note this is a general guideline only.

#### Additional Resources
* [General GitHub documentation](http://help.github.com/)
* [GitHub pull request documentation](http://help.github.com/send-pull-requests/)
* [Pro Git book](http://git-scm.com/book)