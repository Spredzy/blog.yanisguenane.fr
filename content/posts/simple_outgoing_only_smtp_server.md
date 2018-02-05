---
title: "Simple outgoing-only SMTP server"
description: "Simple outgoing-only SMTP server #smtp #opensmtpd"
date: 2018-02-04T16:21:02+01:00
draft: false
categories: ['SMTP']
author: Yanis Guenane
---


It's 2018, [RFC 2821](https://tools.ietf.org/html/rfc2821) *(Simple Mail Transfer Protocol)* turns 17 this year, yet emails are the most widely used type of notification. When one builds/deploys a service - most of the time - it will requires the ability to send emails.

With spam emails being a significant amount of global emails volume, rules to be considered a legitimate email has tighten up. Autenthenticated users, TLS, SPF, DKIM, DMARC, ...

At $DAYJOB time came to launch a new service and hence a new outgoing-only SMTP server. Rather than going down the habitual [Postfix](http://www.postfix.org/) route deployment, we got curious about what were the alternatives out there, and if there was one that was simplier.

Let me introduce you to [OpenSMTPD](https://www.opensmtpd.org/).

<!--more-->

## Postfix

First thing first, what's wrong with [Postfix](http://www.postfix.org/)? Functionally nothing, nothing that would prevent us from using it.

The main criticism I can do to Postfix is its configuration. Many files, many parameters, not always obvious to remember what does what.
Then two files `main.cf` and `master.cf` to configure the core programs. So all in all, if one is not a Postfix administrator by day, there is a good chance, everytime one needs to setup Postfix a trip to the documentation is required.


## OpenSMTPD

### Installation and Configuration

Install the `opensmtpd` package with the package manager of your choice.  Edit the `smtpd.conf(5)` file and start the service.

Below is a `smtpd.conf(5)` example for a basic TLS-enabled outgoing-only SMTP server. (Yep, only 7 lines of configuration)

```
pki mail.example.org key "/etc/ssl/private/mail.example.org.key"
pki mail.example.org certificate "/etc/ssl/pem/mail.example.org.pem"

table secrets file:/etc/opensmtpd/secrets

listen on eth0 tls pki mail.example.org
listen on eth0 port 587 hostname mail.example.org tls-require pki mail.example.org auth <secrets> mask-source

accept from local for any relay
accept from source <INCOMING-IP-OR-NETWORK> for any relay
```

### User Management

In our use case user management is really simple. And the file based system provided by OpenSMTPD is more than enough.

Edit the file specified in your configuration as the secrets table. And follow this pattern:

```
user@domain.tld $(smtpctl encrypt mypassword)
```

So it will look like:

```
notification@example.org $6$cC77yphfPQr44YxL$hDM0tPw0G3G/iz.QpSFBNnIU46cODjCvv7lGQ/KdqDw./EH0A0WAdi7UhBaAUfaEstC8QrbGt4577b5Xl7VxW/
promotions@example.org $6$GhRrWDXTx9mhvyFk$aNtTdv2FuJb1EwFemwkekJ3NJixvY14v7v4dK8syWg9zTdMudeXQNeYBLf23bWcma/BWo1UlArgvvuZUrXHSY/
```

  * Need to add a new user: add a new line with the proper password.
  * Need to update a user password: regenerate the password with `smtpctl encrypt`.
  * Need to remove a user: remove the corresponding line,

Hard to make it simplier.

## Security

Emails security topic is outside the scope of this post. There has been plenty of blog posts written about it.
Just to give some pointers:

  * Authenticated users: The aforementioned configuration uses authenticated users.
  * TLS: The aforementioned configuration uses TLS.
  * SPF: (outside the scope of opensmtpd) one needs to create the proper TXT records in one's DNS entries. More information available at http://www.openspf.org/Introduction
  * DKIM: one can easily interface OpenSMTPD with [DKIMproxy](http://dkimproxy.sourceforge.net/manual/dkimproxy.out.html). It adds 3 lines to the aforementioned configuration.


## Conclusion

As one can see, configuring a fully working TLS-enabled outgoing-only SMTP server takes 7 lines of configuration. 7 lines that I can read, understand what they do and eventually remember quite easily to reproduce. OpenSMTPD might not answer everyone use-cases, yet for our use-case which is only sending authenticated-user emails (with limited accounts), it does the job and does it well. SPM.
