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

In this section I confirmed two critical things:

    1. I can successfully login to *Portainer* and open a console/terminal inside the Apache container.
    2. drush is working nicely inside the Apache container.

## Adding a Drupal Module to the Site

So, to add the `Markdown` module to my site I repeated the *Portainer* steps from the previous section, then inside the `isle-apache-dg` container console/terminal...

    - cd /var/www/html
    - drush dl markdown
    - drush en markdown
    - drush cc all

## Adding This Document as a Basic Page

Next, I re-visited my site as the super-user, and...

    - Navigated to https://dgdocker1.grinnell.edu/#overlay=admin/config/content/formats/add in order to add a new text format I've named `Markdown`,
    - Check the box reqiured to enable the `Markdown` filter and clicked `Save configuration`,
    - Navigated to https://dgdocker1.grinnell.edu/#overlay=node/add,  
    - Selected the `Basic page` link, and in the subsequent form I entered the title and body of this document,
    - Selected `Markdown` from the `Text format` selector,
    - Checked the `Provide a menu link` box, and
    - Clicked `Save`

## Applying the Custom 'Digital Grinnell' Theme

So, the current theme used by Digital Grinnell lives in https://github.com/DigitalGrinnell/digital_grinnell_theme. It's NOT a Drupal.org theme, so I don't believe it can be downloaded and enabled using `drush`.  However, I have had some non-ISLE success in the past with installation of bits like this theme using `composer`, and that process is now nicely documented in [Installing a Module with Composer](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/docs/07_appendices/installing-module-with-composer.md).

It should be noted that in order for the documented process to work the module to be installed, or theme in my case, must have a proper `composer.json` file.  To create such a file for your own theme I'd advise looking at the copy in [https://github.com/DigitalGrinnell/digital_grinnell_theme](https://github.com/DigitalGrinnell/digital_grinnell_theme) for guidance.

Following the instructions--modified for a theme instead of a module--in the aforementioned document...

```
 cd /var/www/html/sites/all/themes
 git clone https://github.com/DigitalGrinnell/digital_grinnell_theme.git
 cd digital_grinnell_theme/
 composer install
```
I navigated my browser to [https://dgdocker1.grinnell.edu/#overlay=admin/appearance](https://dgdocker1.grinnell.edu/#overlay=admin/appearance), and selected the `Digital Grinnell` theme to be enabled AND set as the default.  It worked!

## Loading the Custom DG7 Module

Nearly all of the functional customization in Digital Grinnell is held in the custom [DG7 Module](https://github.com/DigitalGrinnell/dg7.git).  Like the custom theme above, this is a custom module and NOT a Drupal.org module so I've added a `composer.json` file to the project repository and will attempt to download and install it similar to what I did with the theme above:

```
 cd /var/www/html/sites/all/modules
 git clone https://github.com/DigitalGrinnell/dg7.git
 cd dg7/
 composer install
 cd /var/www/html/sites/default
 drush en dg7
 drush cc all
```
After the command sequence above, I visited my site and navigated to `#overlay=admin/modules` where I was able to confirm that the `DG7` module is loaded and enabled.  I also visited `#overlay=admin/structure/views` to see if the Drupal views defined in `DG7` were active, but they were NOT.  So I visited `#overlay=admin/reports/dblog` and found log messages like the following:

    dg7_views_default_views has been called but not in MAINTENANCE MODE, or dg7_collection already exists, so the view will NOT be updated.

So I put the site into maintenance mode from `#overlay=admin/config/development/maintenance` then revisited `#overlay=admin/modules` where I disabled the `DG7` module, then re-enabled it.

Unfortunately, NONE of this worked, presumably because the site is not yet linked to my Fedora repository so there are NO collections to display.

## Reindexing Fedora

I opened https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/docs/04_installation_migration/migration_reindex_process.md and attempted `Shutdown Fedora Method 1`, but I get a 404 error at `http://dgdocker1.grinnell.edu:8080/manager/html`.  So I tried visiting just `http://dgdocker1.grinnell.edu:8080` and I get the *Traefik* dashboard.  Moving on to `Shutdown Fedora Method 2`.

I opened *Portainer* at `https://portainer1.grinnell.edu` and opened a shell/console/terminal in the `isle-fedora-dg` container.  The *Fedora* container is working, but it's pointed to a new/empty repository, and I can't open an admin web interface to it (nor to *Solr*), presumably because ports 8081 and 8082 are blocked.

I made an urgent ticket request to open ports `8081, 8082, and 8083` to campus, and to create DNS entries `fedora1` and `solr1` so that I can write *Traefik* rules to provide *Fredora* and *Solr* dashboard access without port numbers.

## Re-Configuring to Try again
So, at this point the plan is to...

- Spin down my stack... `docker-compose down`,
- Modify `docker-compose.yml` volumes to point to the backup copy of my old *Fedora* data,
- Spin the stack back up... `docker-compose up -d`,
- Assess where we stand with the new site and old data.

## So Far, So Good

Just did `docker-compose up -d` as indicated above and the site, apparently with all my earlier changes, is back at https://dgdocker1.grinnell.edu.  

Better still, when I visit the [Islandora Repository](https://dgdocker1.grinnell.edu/islandora/object/islandora%3Aroot) link on the home page... I can see some of my old objects!

*Three cheers for the ICG and the awesome folks behind ISLE!*

## Ports are Open!
Lo-and-behold miracles never cease... the port open ticket I submitted just 2 hours ago has been completed, and I do indeed now have access to ports 8081, 8082 and 8083 from my connection on campus!  The advantages of having these ports open on the host should probably be documented in one of the host requirements documents.

## Reindex Fedora
Next step follows the [migration_reindex_process.md](https://github.com/Islandora-Collaboration-Group/ISLE-Documentation/blob/master/docs/04_installation_migration/migration_reindex_process.md) to reindex my *Fedora* content.

I started by examining my `tomcat.env` file to determine what I had set for the `TOMCAT_MANAGER_USER` and `TOMCAT_MANAGER_PASS` variables.  Having identified those I was able to open the *Tomcat Manager* page at `https://dgdocker1.grinnell.edu:8081/manager/html` and stop *Fedora* using `Shutdown Fedora Method 1`.

I was subsequently able to follow the directions in `Reindex Fedora RI (1 of 3)` and got my repository re-indexed in just under 7 minutes time.

I moved on to `Reindex SQL database (2 of 3)` but could not log in to *MySQL* as root, getting...

```
root@a96078fddeff:/# mysql -h mysql -u root -pNotMyRealPassword
mysql: [Warning] Using a password on the command line interface can be insecure.
ERROR 1045 (28000): Access denied for user 'root'@'172.28.0.6' (using password: YES)
```

To work around this I used the *Portainer* dashboard to open a shell into the *MySQL* container where I was able to open a `mysql>` prompt using...

```
root@18570400f5d4:/# mysql -u root -p
```
I specified the password when prompted after the login command. continuing on...

```
use fedora3;
show tables;
+---------------------+
| Tables_in_fedora3   |
+---------------------+
| dcDates             |
| doFields            |
| doRegistry          |
| fcrepoRebuildStatus |
| modelDeploymentMap  |
| pidGen              |
+---------------------+
6 rows in set (0.00 sec)

truncate table dcDates;
truncate table doFields;
truncate table doRegistry;
truncate table fcrepoRebuildStatus;
truncate table modelDeploymentMap;
truncate table pidGen;
```
So I returned to my *Fedora* container shell in the *Portainer* dashboard, and...

```
cd /usr/local/fedora/server/bin
time /bin/sh fedora-rebuild.sh -r org.fcrepo.server.utilities.rebuild.SQLRebuilder > /usr/local/tomcat/logs/sql_ri.log 2>&1
```
**Note**: I advocate that folks always put `time` in front of any long-running scripts, like the one above, so that we get a sense of how long they take to run under different circumstances.  `13.25 minutes in my case`

Moving on to `Reindex Solr (3 of 3)`...

```
cd /usr/local/tomcat/webapps/fedoragsearch/client
time /bin/sh runRESTClient.sh localhost:8080 updateIndex fromFoxmlFiles
```
`Just under 53 minutes for me.`

## Restart Fedora

Having completed the re-indexing I returned to http://dgdocker1.grinnell.edu:8081/manager/html and used the link there there `Start` *Fedora*.  That worked.

## Adding More Custom Code
So, a visit to my home page shows one notice and one warning...

```
Notice: Undefined offset: 1 in Digital_Grinnell_preprocess_page() (line 131 of /var/www/html/sites/all/themes/digital_grinnell_theme/drupal7_theme_methods.php).

User warning: The following module is missing from the file system: icu. For information about how to fix this, see the documentation page. in _drupal_trigger_error_with_delayed_logging() (line 1143 of /var/www/html/includes/bootstrap.inc).
```
The warning I can easily take care of.  The `dg7` module that I loaded earlier has one rather obscure reference to another module of mine, `icu`, the *Islandora Common Utilities*.  As it turns out, that one function in `dg7` isn't going to be needed in an *ISLE* environment, so I'm going to stub off the function in `dg7` and just make it return a message indicating that the feature has been deprecated, just in case it is ever called again.

I made the necessary changes in a new copy of `dg7` from https://github.com/GrinnellCollege-Private/dg7.git, moved the existing module out of the way in the *Apache* server at `/var/www/html/sites/all/modules/dg7`, and cloned the new module into its place.  I did `composer update` followed by `drush cc all` and now when I visit https://dgdocker1.grinnell.edu/islandora/object/islandora:root the missing `icu` warning is gone.  

The offset notice remains so I forked https://DigitalGrinnell/digital_grinnell_theme to https://GrinnellCollege-Private/digital_grinnell_theme and committed a change to eliminate the notice.  Then, just as with `dg7` I replaced the code in *Apache*'s `/var/www/html/themes/digital_grinnell_theme` directory and repeated steps from above.  Now, when I visit https://dgdocker1.grinnell.edu/islandora/object/islandora:root the notice is gone too!

## More Modules to Be Enabled
We've come a long way, but... from a shell/terminal running at `digital7.grinnell.edu`, and another running inside my *Apache* container, I've been doing this:

```
cd /var/www/<path to site>/site/default
drush pm-list --fields=Name,Version,Status --format=csv --no-core > modules.list.csv
```
This `drush` command output, piped to a .csv file, gives me two lists of ALL the non-core modules and themes available in each environment, along with their version and status ('Enabled', 'Not Installed', etc.).  I pulled these two lists together in an Excel spreadsheet and did a little 'VLOOKUP' magic to get an idea of what's in DG that's not yet enabled in ISLE.  *I found 30 "necessary" modules in DG that are not yet in ISLE, and another 30 that I can perhaps live without, at least for now.*

I could install and enable all of these using the `drush` and/or `composer` methods employed above, but I'm going to investigate https://github.com/Islandora-Collaboration-Group/ISLE-Drupal-Build-Tools first to see if that might be a better solution.  So, I've forked that project to a *public* repo in https://github.com/DigitalGrinnell/ISLE-Drupal-Build-Tools and will take up the fight there.  Note that I've also created a *private* copy, not a fork, of the canonical *ISLE* repo at https://github.com/GrinnellCollege-Private/ISLE.

## Going For Broke
I spent the last 24 hours editing in the `Digital.Grinnell` branch of my https://github.com/DigitalGrinnell/ISLE-Drupal-Build-Tools repository and cleaning up some customizations of Islandora 7.x modules.  For details see my comments in the `isle_islandora_istaller.sh` file, and the two `.make` files in the `isle-drush_make/` directory on the `Digital.Grinnell` branch of https://github.com/DigitalGrinnell/ISLE-Drupal-Build-Tools.

I have subsequently visited my vSphere console and created a snapshot of the `DGDocker1` host... so that I can easily restore to the current state if needed. Next, I modified a portion of the `.env` file at `dgdocker1:/opt/ISLE/` to read as follows...

```
## ISLE Drupal Build tools:

# Pull build tools from git repo?  bool (true|false), default: true
PULL_ISLE_BUILD_TOOLS=true

# Repo and branch to pull?
ISLE_BUILD_TOOLS_REPO=https://github.com/DigitalGrinnell/isle_drupal_build_tools.git
ISLE_BUILD_TOOLS_BRANCH=Digital.Grinnell
```
Now I'm going to repeat a process I ran earlier...
- Spin down my stack... `docker-compose down`,
- Spin the stack back up... `docker-compose up -d`,
- `time docker exec -it isle-apache-dg bash /utility-scripts/isle_drupal_build_tools/isle_islandora_installer.sh`,
- ...watch for problems while the script runs...
- Assess where we stand with the new modules, site, and new data.

**Massive fail!***
Lots of problems with the process above.  Resetting and restarting on Monday, December 10...

## Going for Broke, Properly This Time
So, I did this...
- Delete my fork of ISLE-Drupal-Build-Tools, it is a mess.
- Make a new fork of ISLE-Drupal-Build-Tools and a new Digital.Grinnell branch within.
- In the Digital.Grinnell branch I need to copy `templates/isle_islandora_installer.sh.tpl` and make a new `grinnell_installer.sh` in the root of the repository.
- Make additions to the two `.make` files, leaving them “in-place”, but note that end-of-line comments are NOT allowed!
- Make minor additions to `migrations_site_vsets.sh`, also "in-place"

Ultimately, I had to comment out the enabling of my `digital_grinnell_theme` (not sure why?) and some modules/extensions, and I got a relatively clean run with a few expected warnings, and one error that I'm going to ignore for now.  The complete output is in [this gist](https://gist.github.com/McFateM/f5e72ad7c4bf2e5719165db5cb6bb4ed).

Improvement noted... this time I was able to login to the new site as the superuser (`user 1`) without having to reset my password via `drush`.  Having done that I visited the site [Appearance](https://dgdocker1.grinnell.edu/#overlay=admin/appearance) page and set my `Digital Grinnell` theme as enabled, and as the default.  That worked nicely; still not sure why I was unable to do the same in my installer script though?

The following errors/warnings were present at https://dgdocker1.grinnell.edu/islandora/object/islandora%3Aroot once the theme was set:
```
Error searching Solr index "400" Status:
Warning: Invalid argument supplied for foreach() in islandora_solr_views_query->execute() (line 179 of /var/www/html/sites/all/modules/islandora/islandora_solr_views/islandora_solr_views_query.inc).
PDOException: SQLSTATE[42S02]: Base table or view not found: 1146 Table 'isle_dg.xml_form_builder_form_associations' doesn't exist: SELECT fa.* FROM {xml_form_builder_form_associations} fa WHERE ( (content_model = :db_condition_placeholder_0) ); Array ( [:db_condition_placeholder_0] => islandora:collectionCModel ) in xml_form_builder_get_database_associations() (line 90 of /var/www/html/sites/all/modules/islandora/islandora_xml_forms/builder/includes/associations.inc).
```

I'm going to post this document as a `markdown` formatted article, and promote it to the front page of the site again.  To do that I needed to visit [the modules](https://dgdocker1.grinnell.edu/#overlay=admin/modules) page to enable `Markdown filter` module (*Odd*, I thought I had done that in the installer script?).  Then I visited [the configuration page](https://dgdocker1.grinnell.edu/#overlay=admin/config) and added `markdown` as a text format option.  Having done that I clicked `Add content` and add this text, actually `markdown`, to the site.

Now, it's time to check some of my other settings and see if I can remedy the above issues ASAP.

## Checking Module Config and Site Settings

In order to understand and hopefully mitigate the errors/warnings shown above, I'm visiting
