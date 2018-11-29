---
Note:  The canonical copy of this text can be found at [https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md).
---
# DG Migration History

Migration commenced on 27-Nov-2018 by MAM on CentOS 7 host `DGDocker2.Grinnell.edu` (132.161.132.143).  Initial setup followed [install_on_centos.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/docs/01_installation_host_server/install_on_centos.md) with creation and config of user and group `islandora` on the host.  SSH access was configured for Mark's GC MacBook, `ma7053`.

## Moving Back to DGDocker1

*DGDocker1* seems to be configured to allow *Let's Encrypt* cert generation, and internet access --as demonstrated with *OHScribe* and *TextLine*-- so I am moving back to that server to try again.

*Note: The process documented below was originally conducted, as stated above, on host `dgdocker2.grinnell.edu`, but that server was NOT properly configured (the firewall would not allow off-campus access nor successful cert creation).  Consequently, the work was moved to `dgdocker1.grinnell.edu`, that server was updated accordingly and the documentation that follows is all written with respect to `dgdocker1.grinnell.edu`.*

## On DGDocker1

From a terminal open to `mcfatem@DGDocker1`, all previous Docker bits were cleaned using...

```
docker stop $(docker ps -q)
docker rm -v $(docker ps -qa)
docker image rm $(docker image ls -q)
docker system prune
```

Process subsequently moved on to [new_site_installation_guide_single.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/docs/03_installation_new_site/new_site_installation_guide_single.md) as directed at the end of the [install_on_centos.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/docs/01_installation_host_server/install_on_centos.md) document.

## DNS and Certs

My existing DNS entry is `dgdocker1.grinnell.edu` and I have no certs for that address.  So I'm proceeding under the assumption that I'll address my new site as [http://dgdocker1.grinnell.edu](http://dgdocker1.grinnell.edu), at least initially.

I'll plan to obtain certs according to the process documented nicely in [configuring-lets-encrypt](https://islandora-collaboration-group.github.io/ISLE-Documentation/07_appendices/configuring-lets-encrypt/).

## Cloning the ISLE Project

```
[mcfatem@dgdocker1 opt]$ sudo git clone https://github.com/Islandora-Collaboration-Group/ISLE.git
Cloning into 'ISLE'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 15305 (delta 4), reused 0 (delta 0), pack-reused 15293
Receiving objects: 100% (15305/15305), 173.59 MiB | 26.23 MiB/s, done.
Resolving deltas: 100% (4209/4209), done.
```

## Editing Environment Variables

```
[mcfatem@dgdocker1 opt]$ sudo chown -R islandora:islandora ISLE
[mcfatem@dgdocker1 opt]$ cd /opt/ISLE
[mcfatem@dgdocker1 ISLE]$ ls -alh
total 88K
drwxr-xr-x. 7 islandora islandora 4.0K Nov 27 16:48 .
drwxr-xr-x. 6 root      root        67 Nov 27 16:48 ..
drwxr-xr-x. 2 islandora islandora    6 Nov 27 16:48 ansible
drwxr-xr-x. 3 islandora islandora   18 Nov 27 16:48 config
-rw-r--r--. 1 islandora islandora 4.8K Nov 27 16:48 docker-compose.yml
-rw-r--r--. 1 islandora islandora 2.6K Nov 27 16:48 .env
drwxr-xr-x. 8 islandora islandora 4.0K Nov 27 16:48 .git
-rw-r--r--. 1 islandora islandora  870 Nov 27 16:48 .gitignore
-rw-r--r--. 1 islandora islandora 1.1K Nov 27 16:48 .gitmodules
drwxr-xr-x. 9 islandora islandora 4.0K Nov 27 16:48 images
-rw-r--r--. 1 islandora islandora  35K Nov 27 16:48 LICENSE
-rw-r--r--. 1 islandora islandora  14K Nov 27 16:48 README.md
-rw-r--r--. 1 islandora islandora  213 Nov 27 16:48 tomcat.env
drwxr-xr-x. 4 islandora islandora   32 Nov 27 16:48 vagrant
```
Switched to `islandora` user via `ssh islandora@dgdocker1.grinnell.edu`.  The SSH keys worked from `ma7053`.


```
[islandora@dgdocker1 ISLE]$ sudo nano .env

    COMPOSE_PROJECT_NAME=dg-prod
    BASE_DOMAIN=dgdocker1.grinnell.edu
    CONTAINER_SHORT_ID=dg
```

See `/opt/ISLE/.env` file for complete change details.

```
[islandora@dgdocker2 ISLE]$ sudo nano tomcat.env
```

See `/opt/ISLE/tomcat.env` for complete change details.

## Working Through configuring-lets-encrypt

Next step continued in my terminal on *DGDocker1* with [configuring-lets-encrypt](https://islandora-collaboration-group.github.io/ISLE-Documentation/07_appendices/configuring-lets-encrypt/).

```
[islandora@dgdocker1 ISLE]$ cd config/proxy
[islandora@dgdocker1 ISLE]$ touch acme.json
[islandora@dgdocker1 ISLE]$ chmod 600 acme.json
[islandora@dgdocker1 ISLE]$ nano traefik.toml
```
See `/opt/ISLE/config/proxy/traefik.toml` for complete change details.

```
[islandora@dgdocker1 ISLE]# nano docker-compose.yml
```
See `/opt/ISLE/docker-compose.yml` for complete change details including path label modifications for *Portainer* and *Traefik* to...

```
- "traefik.frontend.rule=Host:${BASE_DOMAIN};PathPrefixStrip:/portainer"
#- "traefik.frontend.rule=Host:portainer.${BASE_DOMAIN};"

- "traefik.frontend.rule=Host:${BASE_DOMAIN};PathPrefixStrip:/traefik"
#- traefik.frontend.rule=Host:admin.${BASE_DOMAIN};
```

## Change Owner:Group Before Spin-Up

The `owner:group` settings of `/opt/ISLE`, currently `root:root` does not seem right to me, so I'm doing this before spinning it up...

```
[root@dgdocker1 ISLE]# chown -R islandora:islandora .
[root@dgdocker1 ISLE]# exit
```

## Spin It Up
```
[islandora@dgdocker1 opt]$ cd ISLE
[islandora@dgdocker1 ISLE]$ docker-compose up -d
  ...5 minute pause as recommended...
[islandora@dgdocker1 ISLE]$ docker exec -it isle-apache-dg bash /utility-scripts/isle_drupal_build_tools/isle_islandora_installer.sh
```
My hope is that the *Traefik* dashboard will be available at http://dgdocker1.grinnell.edu/traefik, and *Portainer* at http://dgdocker1.grinnell.edu/portainer.

## Results of Initial 'Up'

It works!  Well, mostly.  My fresh new Islandora site is available, with a valid cert, at https://dgdocker1.grinnell.edu, but neither *Traefik* nor *Portainer* are visible as intended.  *Traefik* is available **on-network only** at http://dgdocker1.grinnell.edu:8080.

## Next Step - Exposing Traefik and Portainer

It so happens that I have existing DNS entries of `traefik1.grinnell.edu` and `portainer1.grinnell.edu` that already point to my host, `dgdocker1`.  So, I'm going to rework my `docker-compose.yml` file now and re-start the stack to see if I can get those addresses working properly.

I added the following `.env` key/value pairs:

  - TRAEFIK_ADDRESS=traefik1.grinnell.edu
  - PORTAINER_ADDRESS=portainer1.grinnell.edu

I also changed two lines in `docker-compose.yml` like so:

  - traefik.frontend.rule=Host:${PORTAINER_ADDRESS}  # was traefik.frontend.rule=Host:portainer.${BASE_DOMAIN}
  - traefik.frontend.rule=Host:${TRAEFIK_ADDRESS}  # was traefik.frontend.rule=Host:admin.${BASE_DOMAIN}

I followed these changes with...

```
[islandora@dgdocker1 opt]$ cd ISLE
[islandora@dgdocker1 ISLE]$ docker-compose down -v
[islandora@dgdocker1 ISLE]$ docker-compose up -d
  ...2 minute pause as recommended...
[islandora@dgdocker1 ISLE]$ docker exec -it isle-apache-dg bash /utility-scripts/isle_drupal_build_tools/isle_islandora_installer.sh
  ...took 10 minutes 50 seconds...
```

And behold, it works!  https://traefik1.grinnell.edu and https://portainer1.grinnell.edu are live with valid certs!  Problem: Neither is protected, yet.

## Next Step - Protecting Traefik and Portainer

Piece of cake... I followed [this post](https://medium.com/@techupbusiness/add-basic-authentication-in-docker-compose-files-with-traefik-34c781234970) to generate and apply hashed authentication for *Traefik*, and [this post](https://github.com/portainer/portainer/issues/1506) for *Portainer*.

The changes made to `docker-compose.yml` were:

  - command: --admin-password $$2y$$05$$JEFVfb8jNSBchvoNcZ.qoufys6AExBEvWMa/E38RGGWAnxkTmVZZa -H unix:///var/run/docker.sock  # was...   command: -H unix:///var/run/docker.sock --no-auth
  - "traefik.frontend.auth.basic=islandora:$$apr1$$aTbvuG3k$$4wlb3Zj0.x2Lshn/FwfWV0"  #...added to traefix|labels

See `docker-compose.yml` for complete details.  *Note*: In both the Traefik and Portainer values it's important to escape all `$` with an additional `$`, hence the `$$` character pairs that occur in 6 places above.

Now, both *Traefik* and *Portainer* dashboards are password protected.  Yay!

## Next - Login to the Site As the Super-User

So, I wanted to add a copy of this document, the text you are reading now, to the new web site as a Drupal page displayed in Markdown, because that what this document is written in.  Obviously it worked because you're reading that page now, I presume.  

First step was to login to the new site as the super-user, in my case that username is `digital`, so that I could subsequently add this text as a `Basic Page`.  I tried using the credentials defined in my `.env` file, but I was not allowed to login.  I'm still not sure what went wrong, but I executed the following steps to quickly work around it...

    - Open Portainer in my brower by visiting https://portainer1.grinnell.edu, and login there.
    - In the Portainer dashboard open a terminal into the `isle-apache-dg` container using its `>_` console link.
    - Inside the console/terminal...
        - cd /var/www/html
        - drush uli digital
        - Copy the generated URL for use in my browser.
    - Back in my browser... paste the generated URL into the address bar and modify it replacing `http://default/` with 'http://dgdocker1.grinnell.edu/'
    - In the 'temporary' page, enter a new password for the super-user, `digital`.
    
Having completed these steps I can now successfully login to my site as the super-user, `digital`.

## Add a Drupal Module to the Site

In the previous section I confirmed two critical things:

    1. I can successfully log in to Portainer and open a console/terminal inside the Apache container.
    2. drush is working nicely inside the Apache container.
    
So, to add the `Markdown` module to my site I repeated the Portainer steps from the previous section, then inside the `isle-apache-dg` container console/terminal...

    - cd /var/www/html
    - drush dl markdown
    - drush en markdown
    - drush cc all

## Add This Document as a Basic Page

Next, I re-visited my site as the super-user, and...

    - Navigated to https://dgdocker1.grinnell.edu/#overlay=node/add,  
    - Selected the `Basic page` link, and in the subsequent form I entered the title and body of this document,
    - Selected `Markdown` from the `Text format` selector,
    - Checked the `Provide a menu link` box, and
    - Clicked `Save`
    
