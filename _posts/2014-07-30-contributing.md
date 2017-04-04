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

If you've got a feature request for MailWatch, file it with the [GitHub issue tracker]((https://github.com/mailwatch/MailWatch/issues)).
We're aiming to create an useful utility, and would love to hear your ideas.

### Submitting Bug Reports

If you find a problem with MailWatch, whether you are stuck or want to address it yourself, first log it using the [GitHub issue tracker](https://github.com/mailwatch/MailWatch/issues).
That way, we can take a look and let you know how best to proceed, and if work is already underway on this issue.

Note: The issue tracker is for bugs and new features, not help requests. For procedural questions - like help - see the above "Getting Help" section.

### Contributing to MailWatch

#### Getting Started
* Make sure you have a [GitHub account](https://github.com/signup/free).
* [Submit an issue](https://github.com/mailwatch/MailWatch/issues) for your issue, assuming one does not already exist.
   * Clearly describe the issue and your enviroment including steps to reproduce when it is a bug.
   * Make sure you fill in the earliest version that you know has the issue.
* If you want to contribute new code or fix existing one fork the repository on GitHub or sync your existing branch like described below.

#### Branching and pull requests
As a guideline, please follow this process:

 1. [Fork the repository](https://help.github.com/articles/fork-a-repo).
 2. Create a topic branch for the change:
    * Please ensure the branch is clearly labelled as a feature or fix.
    * Avoid working directly on the **master** branch, create a new branch from **develop** and work on it.
 3. Make the relevant changes.
 4. [Squash](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Changing-Multiple-Commit-Messages) commits to keep git log clear.
 5. It should go without saying, but do not increment the version number in your commits.
 6. [Submit a pull request](https://help.github.com/articles/using-pull-requests) to the **develop** branch.

Please make sure that your fork is in sync with with upstream:
on your repository follow this procedure:

 1. `git checkout develop`
 2. `git remote add upstream https://github.com/mailwatch/MailWatch.git`
 3. `git fetch upstream`
 4. `git rebase upstream/develop`
 5. fix code inconsistency if any (there should be none, if you work always on branch derived from develop)

Before creating a new feature branch repeat the last 2 steps:

 1. `git fetch upstream`
 2. `git rebase upstream/develop`
 3. create a new branch from **develop** branch: `git checkout -b myNewFeature develop`

Please note this is a general guideline only.

#### Additional Resources
* [General GitHub documentation](http://help.github.com/)
* [GitHub pull request documentation](http://help.github.com/send-pull-requests/)
* [Pro Git book](http://git-scm.com/book)