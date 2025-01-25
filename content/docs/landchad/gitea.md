+++
title = "Gitea"
date = 2024-02-03
weight = 6
+++

Gitea allows you to self-host your git repositories similar to [bare
repositories](../git), but comes with additional features that you might know
from GitHub, such as issues, pull requests or multiple users. Its advantage
over GitLab -- another free software GitHub clone -- is that it is much more
lightweight and dramatically easier to setup.

![](/landchad/gitea/screenshot.png)

Although Gitea is lighter than GitLab, if you have a VPS with only 512MB of
RAM, you will probably have to upgrade. Gitea is more memory-intensive than
having just a bare git repository. If you just want a minimalist browseable git
server without issue tracking and pull requests, install [cgit](../cgit)
instead.

## Installing Gitea

Unfortunately, despite how popular Gitea is, as far as free software goes, it
is not available as a DEB package for convenient use in Debian, or RPM in
Fedora. However, installing it from a compiled binary is very easy. [Follow the
official guide](https://docs.gitea.com/installation/install-from-binary) that
details all the steps of setting it up, then continue this guide.

It is of note that the official Gitea documentation is excellent, and
accessible even to the most novice of Linux administrators. However, this guide
will make sure to get you up to speed with setting it up.

## Setting up a Nginx reverse proxy

You should know how to generate SSL certificates and use Nginx by now.
Add this to your Nginx config to proxy requests made to your git
subdomain to Gitea running on port 3000:

```nginx
server {
  listen 443 ssl;
  listen [::]:443 ssl;
  ssl_certificate /etc/ssl/nginx/git.example.org.crt;
  ssl_certificate_key /etc/ssl/nginx/git.example.org.key;
  server_name git.example.org;
  location / {
    proxy_pass http://localhost:3000/; # The / is important!
    proxy_redirect off;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
  }
}
```


And reload Nginx:

```fish
systemctl reload nginx
```

## Setting up Gitea

If everything worked fine you should now see a setup screen when you go
to your configured domain in the browser. The options should be pretty
self-explanatory, it is only important to select SQLite3 and to replace
the base url and SSH server domain with your own.

- **Database Type**
  : SQLite3
- **SSH Server Domain**
  : `git.example.org`
- **Gitea Base URL**
  : `git.example.org`

These and other settings can be changed in a configuration file later so
don\'t worry about making wrong decisions right now.

After clicking the install button you should now be able to log into
your Gitea instance with the account you just created! Explore the
settings for more things to do, such as setting up your SSH keys.

If Gitea does not load fully and has random errors, it is possible that
you need to increase your available memory on your VPS. This can usually
be done on your VPS-provider\'s website without too much trouble.

## A few extras

### Automatically create a new repo on push

This is an incredibly useful feature for me. Open up
`/etc/gitea/app.ini` and add `DEFAULT_PUSH_CREATE_PRIVATE = true` to the
`repository` section like so:

```systemd
[repository]
ROOT = /var/lib/gitea/data/gitea-repositories
DEFAULT_PUSH_CREATE_PRIVATE = true
```

If you now add a remote to a repository like this

```fish
git remote add origin 'ssh://gitea@git.example.org/username/coolproject.git'
```

and push, Gitea will automatically create a private `coolproject`
repository in your account!

### Change tab-width

By default Gitea displays tabs 8 spaces wide. However, some people might prefer
4 spaces, or 2 spaces. We can change this.

```fish
mkdir -p /var/lib/gitea/custom/templates/custom/
```

And write this into
`/var/lib/gitea/custom/templates/custom/header.tmpl`:

```css
<style>
.tab-size-8 {
  tab-size: 4 !important;
  -moz-tab-size: 4 !important;
}
</style>
```

## Contribution

-   [phire](https://phire.cc)
