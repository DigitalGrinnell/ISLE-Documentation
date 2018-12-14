---
Note:  The canonical copy of this text can be found at https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/ISLE-Dev.md.
---
# ISLE-Dev

The goal of this project is to spin up a pristine Islandora stack using https://github.com/Islandora-Collaboration-Group/ISLE.git, then introduce elements like the [Digital Grinnell theme](https://github.com/DigitalGrinnell/digital_grinnell_theme) and custom modules like [DG7](https://github.com/DigitalGrinnell/dg7).  Once these pieces are in-place and working, I'll save my work into a private repository for distribution to colleagues, and subsequent module and theme development and testing.

## Working Initially on MA8660 (My iMac)

From a terminal open to `mcfatem@ma8660`, all previous Docker bits were cleaned and a new `ISLE-Dev` repo created using...

```
cd ~
docker stop $(docker ps -q)
docker rm -v $(docker ps -qa)
docker image rm $(docker image ls -q)
docker system prune

git clone https://github.com/Islandora-Collaboration-Group/ISLE.git ISLE-Dev
cd ISLE-Dev
docker-compose up -d
docker exec -it isle-apache-ld bash /utility-scripts/isle_drupal_build_tools/isle_islandora_installer.sh
```

The following lines were subsequently added to the `/etc/hosts` file on `ma8660` and all other similar lines were commented out:
```
### For ISLE
127.0.0.1    localhost isle.localdomain admin.isle.localdomain portainer.isle.localdomain
```

## Accessing the Site and Associated Tools

The following table of addresses may be useful when working with `ISLE-Dev`:

Address | Description | Login Credentials
--- | --- | ---
https://isle.localdomain/  | The stack | isle / isle
https://github.com/Islandora-Collaboration-Group/ISLE | The canonical ISLE repository | None
https://github.com/Islandora-Collaboration-Group/ISLE-Documentation  | The canonical ISLE-Documentation repository | None
https://portainer.isle.localdomain/#/home | The stack's Portainer admin dashboard | None
https://admin.isle.localdomain/dashboard/ | The stack's Traefik admin dashboard | None

## Adding Our Custom Theme

Repository | Description
--- | ---
https://github.com/DigitalGrinnell/digital_grinnell_theme | The Digital Grinnell Theme

Returning to my `ma8660` terminal, I opened a `bash` shell inside the running `isle-apache-ld` container where I added and enabled my `digital_grinnell_theme`...

Command | Result
--- | ---
`docker exec -it isle-apache-ld bash` | Opened `bash` terminal in `isle-apache-ld`  
`cd /var/www/html/sites/default` | Set working directory
`mkdir themes` | Created `themes` directory
`git clone https://github.com/DigitalGrinnell/digital_grinnell_theme.git themes/digital_grinnell_theme` | Cloned the theme
`drush pm-enable digital_grinnell_theme` | Enabled the theme
`drush vset theme_default digital_grinnell_theme` | Set theme as default  

A visit to the site, https://isle.localdomain, shows that this worked, but there's not much there to see yet, of course.

## Adding Our Custom Module

Repository | Description
--- | ---
https://github.com/DigitalGrinnell/dg7 | The DG7 custom module

Returning to the open `bash` shell inside the running `isle-apache-ld` container...

Command | Result
--- | ---
`cd /var/www/html/sites/default` | Set working directory
`mkdir modules` | Created `modules` directory
`git clone https://github.com/DigitalGrinnell/dg7.git modules/dg7` | Cloned the module
`drush en dg7` | Enabled the module
`drush cc all` | Cleared all caches  

A visit to the site, https://isle.localdomain, shows that this worked since the user login prompt has been modified by the new module.  Otherwise, there's still not much there to see yet.

## Adding Other Critical Modules

I've prepared a document comparing the modules/themes in https://digital.grinnell.edu with those found in a pristine ISLE stack.  The document is https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/ModuleComparison.md but it doesn't display well in GitHub for some reason.
