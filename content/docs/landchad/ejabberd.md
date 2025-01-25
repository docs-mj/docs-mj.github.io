+++
title = "ejabberd"
date = 2024-02-03
weight = 3
+++

ejabberd is a free XMPP server. It's a program that runs on your server and
enables communication via instant messaging, chat, and presence information. In
simpler terms, it's the server part of the equation that makes XMPP possible.

XMPP (*Extensible Messaging and Presence Protocol*) is a communication protocol
used for instant messaging, presence, and other related applications. It's the
underlying technology that allows users of different clients/programs to
communicate with each other even if they are using different software. This is
known as federation.

{{% details title="What is federation?" closed="true" %}}

Federation allows separate deployments of a communication service to
communicate with each other - for instance a mail server run by Google
federates with a mail server run by Microsoft when you send email from
`@gmail.com` to `@hotmail.com`.

Interoperable clients may simply be running on the same deployment - whereas in
federation the deployments themselves are exchanging data in a compatible
manner.

XMPP provides open federation - meaning that anyone on the internet can join
into the XMPP ecosystem by deploying their own server. Credentials from your
own server can be used as valid credentials on a different, totally unrelated
server.

{{% /details %}}

## Installation

ejabberd is easiest installed through their official DEB and RPM packages
hosted on GitHub. DEB is what you use on Debian, and RPM is what you use on
Fedora. Head over to the GitHub page
[here](https://github.com/processone/ejabberd/releases) to install the latest
package. For instance, if you run Debian on an x86_64 machine, you download the
`ejabberd_xx.yy-z_amd64.deb` package. Thereafter, you run the following
commands in the directory where you downloaded your package:

{{< tabs items="Debian,Fedora" >}}
  {{< tab >}}
  ```fish
  apt install ./ejabberd_xx.yy-z_amd64.deb
  ```
  {{< /tab >}}

  {{< tab >}}

  Download the RPM package instead of DEB. Otherwise the steps are the same.

  ```fish
  sudo dnf install ./ejabberd_xx.yy-z_amd64.rpm
  ```
  {{< /tab >}}
{{< /tabs >}}

More information on the [official documentation
page](https://docs.ejabberd.im/admin/installation/#linux-deb-and-rpm-installers).
There are also Docker images and Linux distribution packages, but those are
either outdated or complicated compared to simply getting the official ejabberd
packages from GitHub.

## Configuration

The ejabberd server is configured in `/etc/ejabberd/ejabberd.yml` on Debian, or
`/opt/ejabberd/conf/ejabberd.yml` on Fedora. Changes are only applied by
restarting the ejabberd service in systemd:

```fish
systemctl restart ejabberd
```

Everything is done through this configuration file. Unlike many other programs
which typically have two or more configuration files, ejabberd is controlled
through one.

### Hostnames

The XMPP hostname is specified in the hosts section of `ejabberd.yml`:

```yaml
hosts:
  - example.org
```

### Certificates

ejabberd doesn't come equipped with a script that can automatically copy over
the relevant certificates to a directory where the ejabberd user can read them.

One way of organizing certificates for ejabberd is to have them stored in
`/etc/ejabberd/certs`, with each domain having a separate directory for both
the fullchain cert and private key.

Using certbot, this process can be easily automated with these commands:

```fish
DOMAIN=example.org

certbot --nginx -d $DOMAIN certonly
mkdir -p /etc/ejabberd/certs/$DOMAIN
cp /etc/letsencrypt/live/$DOMAIN/fullchain.pem /etc/ejabberd/certs/$DOMAIN
cp /etc/letsencrypt/live/$DOMAIN/privkey.pem /etc/ejabberd/certs/$DOMAIN
```

{{< callout type="info" >}}
  You might want to write this script to a file and setup a cronjob to run it
  periodically. This should help prevent your certificates from expiring.
{{< /callout >}}

Make sure all the certificates are readable by the `ejabberd` user:

```fish
chown -R ejabberd:ejabberd /etc/ejabberd/certs
```

To enable the use of all these certificates in ejabberd, the following configuration is necessary:

```yaml
certfiles:
  - "/etc/ejabberd/certs/*/*.pem"
```

### Admin user

The admin user can be specified in `ejabberd.yml` under the `acl` section:

```yaml
acl:
  admin:
    user: admin
```

This would make `admin@example.org` the user with administrator privileges.
This is how login credentials look in XMPP, which might look strange if never
used IRC before, and are used to how modern software operates.

### File uploads

To ensure full compliance with XMPP standards, add the following configuration to mod_http_upload:

```yaml
mod_http_upload:
  put_url: https://@HOST@:5443/upload
  docroot: /var/www/upload
  custom_headers:
    "Access-Control-Allow-Origin": "https://@HOST@"
    "Access-Control-Allow-Methods": "GET,HEAD,PUT,OPTIONS"
    "Access-Control-Allow-Headers": "Content-Type"
```

Make sure to create and give the ejabberd user ownership of `/var/www/upload`
or any other directory you choose to use for file uploads:

```fish
chown -R ejabberd:ejabberd /var/www/upload
```

### Message archives

The ejabberd server supports keeping archives of messages through its mod_mam
module. This can be enabled by uncommenting the following lines:

```yaml
mod_mam:
  assume_mam_usage: true
  default: always
```

It's disabled by default, in line with the philosophy that privacy should be
opted out of, not into.

### Database

We can find the following comment in the `mod_mam` section of the configuration file:

```yaml
mod_mam:
  ## Mnesia is limited to 2GB, better to use an SQL backend
  ## For small servers SQLite is a good fit and is very easy
  ## to configure. Uncomment this when you have SQL configured:
  ## db_type: sql
```

As these comments imply, an SQL backend is recommended if you wish to use your
ejabberd server for anything more than just testing. Ejabberd supports MySQL,
SQLite and PostgreSQL.

Setting up SQLite for ejabberd is very simple. Add the following lines
somewhere in your configuration. These are top-level configuration (you don't
put them under anything else):

```yaml
default_db: sql
sql_type: sqlite
sql_database: /var/ejabberd/ejabberd.db
```

The `sql_database` entry can of course be any directory and file name you like.
Make sure the `ejabberd` user has permissions for that directory.

Lastly, your server will need the SQLite runtime libraries.

{{< tabs items="Debian,Fedora" >}}
  {{< tab >}}
  ```fish
  apt install sqlite3
  ```
  {{< /tab >}}

  {{< tab >}}

  ```fish
  sudo dnf install sqlite
  ```
  {{< /tab >}}
{{< /tabs >}}

## Using ejabberd

To begin using ejabberd, firstly start the ejabberd daemon:

```fish
systemctl start ejabberd
```

Then, using `ejabberdctl` as the `ejabberd` user, register the admin user which
is set in the configuration file:

```fish
su -c "ejabberdctl register admin example.org password" ejabberd
```

This will create the user `admin@example.org`.

### Web admin interface

By default, ejabberd has a web interface accessible from
`http://example.org:5280/admin`. When accessing this interface, you will be
prompted for the admin credentials:

![ejabberd login prompt](/landchad/ejabberd/ejabberd-login.webp)

After signing in with the admin credentials, you will be able to manage your
ejabberd server from this web interface:

![ejabberd web admin screenshot](/landchad/ejabberd/ejabberd-admin.webp)

## Reviewing the clients

After following this guide, you should have an ejabberd server up and running.
However, this is just the server. We of course need a client to go with it.
Because, as said, XMPP is an open protocol, anyone can make any server and
client implementation. Strangely, the mobile clients are the best, while there
is only one graphical desktop program that is serviceable, and it is middling.
Only mobile clients support voice chat.

As of the latest update of this article, the following clients are available:

- **Web**
  : [Converse.js](https://conversejs.org/)
- **Desktop**
  : [Gajim](https://gajim.org/) (GUI)
  : [Profanity](https://profanity-im.github.io/) (CLI)
- **Android**
  : [Cheogram](https://cheogram.com/)
  : [Conversations](https://conversations.im/)
- **iOS and macOS**
  : [Monal IM](https://monal-im.org/)

The XMPP project, sponsored by the company behind ejabberd and others,
maintains [an interactive list](https://xmpp.org/software/) of available
clients for XMPP. However, the above list lists the only serviceable clients.
All others are poor or abandonware, at least for now. 

## Going from here

This guide is less of a comprehensive resource for everything ejabberd-related
and more of a springboard towards getting you started as a landchad; who can
run his own federated chat.

For a deeper look into all the modules and options, have a look at the
following ejabberd documentation and concepts:

{{< tabs items="Modules,Top-level Options,Listen Modules" >}}
  {{< tab >}}
  In ejabberd, everything is loaded as and managed by modules. All the available
  ejabberd modules and their documentation can be read
  [here](https://docs.ejabberd.im/admin/configuration/modules/)
  {{< /tab >}}
  {{< tab >}}
  Some options for ejabberd are not dependent on any module and affect the
  whole program. These can be found
  [here](https://docs.ejabberd.im/admin/configuration/toplevel/)
  {{< /tab >}}
  {{< tab >}}
  Lastly, there are what ejabberd calls [listen
  modules](https://docs.ejabberd.im/admin/configuration/listen/). These modules
  allow you to simply spread the networking part of the program: which module
  uses which port, subdomain, or even IP address.
  {{< /tab >}}
{{< /tabs >}}
