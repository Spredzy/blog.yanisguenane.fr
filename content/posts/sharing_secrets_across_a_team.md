---
title: "Sharing secrets across a team"
description: "Sharing secrets across a team #security #collaboration"
date: 2018-01-31T13:46:58+01:00
draft: false
categories: ['security', 'collaboration']
author: Yanis Guenane
---

In my team at $DAYJOB, like most teams, we need to share secrets. Hence we started looking at the various options available. There is a plethora of tools out there that address this specific issue, but after hearing everyone in the team and their expectations about the tool, we came up with a minimal set of requirements the tool should fulfill:

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

  * `.gpg-id`: File that contains the team members emails with which they created their `gpg`  key. One per-line.

```shell
jdoe@localhost: cat .gpg-id
user1@example.org
user2@example.org
user3@example.org
```

  * `.public-keys/`: Folder that contains the team members `gpg` public keys. Value can be obtained by running `gpg2 --armor --export user1@example.org`. This isn't a requirement, its simply eases the whole process as it will be explained later.

```shell
jdoe@localhost: ls -l .public-keys/
total 36
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user1@example.org
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user2@example.org
-rw-rw-r--. 1 jdoe jdoe 10199 Jan 29 18:56 user3@example.org
```

Run the initial commit and push the repository over to the git server.

### ~/.password-store/myteam

Time to install `pass(1)`. Use your favorite package manager and simply install the `pass` package. Once pass installed, import the password store that was created on the previous step.

```shell
jdoe@localhost: mkdir -p ~/.password-store
jdoe@localhost: git clone https://example.com/store.git ~/.password-store/myteam
```

With the above example we've created the *myteam* password store.

**Note**: This operation needs to happen on every team members workstation.

### Sign each other keys

GPG principle is based around trust. This means that in order for the secrets to be viewable/editable by the team members, each member needs to sign each other (in the GPG way).

Here is the procedure to sign each other:

  * Import **all** the team members public key (they are available in the `.publick-keys/` directory)

```shell
jdoe@localhost: gpg2 --import userN@example.org
```

  * Sign **all** the team members

```shell
jdoe@localhost: gpg2 --edit-key userN@xample.org
gpg> lsign
gpg> y
gpg> save
```

And... we are all set to enjoy `pass(1)`.

## Secrets management

### What is a secret ?

`pass(1)` defines itself as *the standard unix password manager*. The good thing with pass and the way it was built is that it can manage more than just passwords but any kind really of secrets (ie. not public data).

One, with `pass(1)` can share:

  * passwords
  * private keys
  * API tokens
  * Secret documents
  * And more...

### Managing secrets

Ok, you've done all the hard work. From now on let's enjoy simplicity. In the presentation of this article, we've mentioned `pass(1)` was really easy to use. Here the proof:

  * Adding a new password: `pass insert myteam/mailserver/root`
  * Removing a password: `pass remove myteam/mailserver/root`
  * Editing a password: `pass edit myteam/mailserver/root`
  * Generating a password: `pass generate myteam/mailserver/notifications`

The data organization is really up to you and your team members. This translates to simple file hierarchy on the file system. One can provide more than just the private bits in the secret, it can provide any metadata that might be usefull in the context of the secret.
Refer to the [pass(1)](https://git.zx2c4.com/password-store/about/) man pages for more examples.

The great thing with `pass(1)` is that each operation generates a git commit. Making it easily version controlled. Below an example of what happens when inserting a new secret.

```shell
[user1@localhost ~]$ pass insert myteam/yetanotherone
Enter password for myteam/yetanotherone:
Retype password for myteam/yetanotherone:
gpg: automatically retrieved 'user3@example.org' via Local
gpg: automatically retrieved 'user2@example.org' via Local
gpg: automatically retrieved 'user1@example.org' via Local
[master e992418] Add given password for myteam/yetanotherone to myteam.
 2 files changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 yetanotherone.gpg
```

So after each operation, the team member that did the operation only have to push the changes so all the other users from the team can pull from it and have access to the latest secrets.
The management of stores with regarde to git is exactly the same as any other git project. `pass(1)` will create the commits automatically, but it is up to the team member that altered the store to push those changes to the central server.

### Onboard a new team member

To onboard new team members, this is the procedure to follow:

  1. The new team member needs to generate a `gpg` key.
  2. It needs to retrieve the git repository of the store and add itself to `.gpg-id` and `.public-keys/`, commit and push.
  3. All the other team members needs to import and sign the new team member key.
  4. One of the team member needs to re-encrypt the store so the new team member can decrypt the secrets (ie. `pass init -p myteam $(cat ~/.password-store/myteam/.gpg-id)`) and submit the new commits that were automatically generated.
  5. New team member is onboarded, she can know access secrets and create new ones.


### Decomission a team member

To decomission a team member, this is the procedure to follow:

  1. A team member removes the id of the team member to decomission from `.gpg-id` and its corresponding public key file from `.public-keys/`
  2. Re-encrypt the store so the decomissioned team member can not read the secrets anymore (ie. `pass init -p myteam $(cat ~/.password-store/myteam/.gpg-id)`) and submit the new commits that were automatically generated.
  3. Re-generate all the password in the key store. So the decomissioned team member is left with no knowledge.


## Conclusion

If you and your team members accept the small level of complexity introduced by the `gpg` encryption, `pass(1)` is a little gem. It does one thing, (ie. managing secrets) and does it well. Since one can have multiple stores, the same pattern can be reproduce with other teams one is working with. Thanks to the community and the various clients they provide, one case use `pass(1)` through different interfaces (CLI, Android, iOS, GUI, etc...). SPM
