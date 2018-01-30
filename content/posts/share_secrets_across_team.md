---
title: "Sharing secrets across a team"
date: 2018-01-19T17:14:24+01:00
draft: false
categories: ['security', 'collaboration']
author: Yanis Guenane
---

In my team at $DAYJOB, like most teams, we need to share secrets. Hence we started to look at the various options available. There is a plethora of tools out there that address this specific issue, but after hearing everyone in the team expectations, we came out with a minimal set of requirements the tool should fullfill:

  * **No shared password**: Secrets must be available for every team members using their own password, no shared password required.
  * **Offline use**: Secrets must be available offline.
  * **Easy way to change all the password at once**: Easy way of changing all the secrets at once when a team member leaves the team.

<!--more-->

After looking around what were the available options, we agreed on running [pass(1)](http://www.passwordstore.org/), The Standard Unix Password Manager

> Password management should be simple and follow Unix philosophy. With `pass`, each password lives inside of a `gpg` encrypted file whose filename is the title of the website or resource that requires the password.

`pass(1)` is a small program that only purpose is to manage password stores (and the secrets that lives in it). It is installed locally and uses GPG encryption.

## Team environment

### Generate GPG keys

With `pass(1)` each secret lives inside of a `gpg` encrypted file. This means that user that will use `pass(1)` needs a `gpg` key. This means that every member of your team should generate a key if she doesn't own one yet.

```shell
$ jdoe@localhost: gpg2 --gen-key
....
[answer all the questions]
...
$ jdoe@localhost: gpg2 --list-keys
/home/jdoe/.gnupg/pubring.gpg
------------------------
pub   2048R/3B30B0C1 2018-01-28
uid                  User1 (User1 for pass) <user1@example.org>
sub   2048R/F5C4EBBC 2018-01-28
```

### Create a the password-store git repository

In order to be shared across the team, the `pass(1)`'s password store needs to live in a git repository.

Create a new git repository on a server every one from your team can pull from. At first, the content of the directory should look like:

```shell
jdoe@localhost: ls -la
total 0
drwxrwxr-x.  4 jdoe    jdoe    100 Jan 29 18:42 .
drwxrwxrwt. 16 root    root    340 Jan 29 18:41 ..
drwxrwxr-x.  7 jdoe    jdoe    200 Jan 29 18:42 .git
-rw-rw-r--.  1 jdoe    jdoe      0 Jan 29 18:41 .gpg-id
drwxrwxr-x.  2 jdoe    jdoe     40 Jan 29 18:42 .public-keys
```

  * `.gpg-id`: File that Contains the team members emails with which they created their `gpg`  key. One per-line. Value can be obtained by running `gpg2 --list-secret-keys`.

```shell
jdoe@localhost: cat .gpg-id
user1@example.org
user2@example.org
user3@example.org
```

  * `.public-keys/`: Folder that contains the team members gpg public keys. Value can be obtained by running `gpg2 --armor --export user1@example.org`. This isn't a requirement, its simply eases the whole process as it will be explained later.

```shell
jdoe@localhost: ls -l .public-keys/
total 36
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user1@example.org
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user2@example.org
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user3@example.org
```

Run the initial commit and push the repository over to the git server.

### ~/.password-store/store

Time to install `pass(1)`. Use your favorite package manager and simply install the `pass` package. Once pass installed, import the password store that was created on the previous step.

```shell
jdoe@localhost: mkdir -p ~/.password-store
jdoe@localhost: git clone https://example.com/store.git ~/.password-store/store
```

**Note**: This operation needs to happen on every team members workstation.

### sign the keys

GPG principle is based around trust. This means that in order for the secrets to be viewable by the team members, each member needs to trust each other (in the GPG way).

import

```shell
jdoe@localhost: 
```

sign

```shell
```

trust

```shell
```

## Secrets management

### Managing password

Ok, you've done all the hard work. From now on let's enjoy simplicity. In the presentation of this article, we've mentioned `pass(1)` was really easy to use. Here the proof:

  * Adding new password: `pass insert store/mailserver/root`
  * Removing a password: `pass remove store/mailserver/root`
  * Editing a password:

The great thing with `pass(1)` is that each operation generates a git commit. Making it :wq



### Onboard a new user in the team

### Decomissioned a user from the team
