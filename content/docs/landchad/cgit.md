+++
title = "cgit"
date = 2024-02-03
weight = 5
+++

Once you have your server hosting your git repositories, you might want
to allow others to browse your repositories on the web. cgit is a Free
Software that allows browsing git repositories through the web.

Note that cgit is a read-only frontend for Git repositories and doesn't
have issues, pull requests or user management. If that's what you want,
consider installing [Gitea](/gitea) instead.

{{< figure src="/landchad/cgit/screenshot.png" caption="Here is what cgit looks like. It is by design basic." >}}

> [!WARNING]
> The following guide is for Debian only. On Fedora the processes and scripts
> involved differ significantly. If you want a guide for Fedora, you will need
> to find it elsewhere.

## Installing cgit and fcgiwrap

Nginx doesn't have the capability to run CGI scripts by itself, it
depends on an intermediate layer like fcgiwrap to run CGI scripts like
cgit:

```fish
apt install fcgiwrap
```

And now we can install cgit itself with:

```fish
apt install cgit
```

## Setting up Nginx

You should have an Nginx server running with a TLS certificate by now.
Add the following configuration to your server to pass the requests to
Cgit, while serving static files directly:

```nginx
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  ssl_certificate /etc/ssl/nginx/git.example.org.crt;
  ssl_certificate_key /etc/ssl/nginx/git.example.org.key;
  server_name git.example.org;

  root /usr/share/cgit ;
  try_files $uri @cgit ;

  location ~ /.+/(info/refs|git-upload-pack) {
    include             fastcgi_params;
    fastcgi_param       SCRIPT_FILENAME /usr/lib/git-core/git-http-backend;
    fastcgi_param       PATH_INFO           $uri;
    fastcgi_param       GIT_HTTP_EXPORT_ALL 1;
    fastcgi_param       GIT_PROJECT_ROOT    /srv/git;
    fastcgi_param       HOME                /srv/git;
    fastcgi_pass        unix:/run/fcgiwrap.socket;
  }

  location @cgit {
    include fastcgi_params;
    fastcgi_param SCRIPT_FILENAME /usr/lib/cgit/cgit.cgi;
    fastcgi_param PATH_INFO $uri;
    fastcgi_param QUERY_STRING $args;
    fastcgi_param HTTP_HOST $server_name;
    fastcgi_pass unix:/run/fcgiwrap.socket;
  }
}
```

Then get Nginx to reload your configuration. This configuration also enables
cloning via HTTPS, so make sure to point the `fastcgi_param GIT_PROJECT_ROOT`
to the directory where you store your repositories.

## Configuring cgit

You\'ve got cgit up and running now, but you\'ll probably see it without
any style and without any repository. To change this, we need to
configure Cgit to our liking, by editing `/etc/cgitrc`.

```ini
css=/cgit.css
logo=/cgit.svg
virtual-root=/
clone-prefix=https://git.example.org

# Title and description shown on top of each page
root-title=Chad's git server
root-desc=A web interface to LandChad's git repositories, powered by Cgit

# The location where git repos are stored on the server
scan-path=/srv/git/
```

This configuration assumes you followed the [git hosting guide](/git)
and store your repositories on the `/srv/git/` directory.

Cgit\'s configuration allows changing many settings, as documented on
the cgitrc(5) manpage installed with Cgit.

### Changing the displayed repository owner

Cgit\'s main page shows each repo\'s owner, which is \"git\" in case you
followed the git hosting guide, but you might want to change the name to
yours. Cgit shows the owner\'s system name, so you need to modify the
git user to give it your name:

```fish
usermod -c "Your Name" git
```

### Changing the repository description

Navigate to your bare repository on the server and edit the
`description` file inside it

### Displaying the repository idle time

To do this, we need to create a post-receive hook for each repository
that updates the file cgit uses to determine the idle time. Inside your
repository, create a file `hooks/post-receive` and add the following
contents:

```fish
#!/bin/sh

agefile="$(git rev-parse --git-dir)"/info/web/last-modified

mkdir -p "$(dirname "$agefile")" &&
git for-each-ref \
  --sort=-authordate --count=1 \
  --format='%(authordate:iso8601)' \
  >"$agefile"
```

And give it execution permissions with:

```fish
chmod +x hooks/post-receive
```

Next time you push to that repository, the idle time should reset and
show the correct value.

## Contribution

-   Ariel Costas -- [website](https://costas.dev)
