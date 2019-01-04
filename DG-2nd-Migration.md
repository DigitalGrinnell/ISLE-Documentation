---
Note:  The canonical copy of this text can be found at [https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-2nd-Migration.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-2nd-Migration.md).
---
# Digital.Grinnell 2nd Migration

This migration attempt commenced on 11-Dec-2018 by MAM on CentOS 7 host `DGDocker1.Grinnell.edu` (132.161.132.103) following the abandoned `upgration` attempt described in [https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md)

## DGDocker1 Cleanup

From a terminal open to `islandora@DGDocker1`, all previous Docker bits were cleaned and a new `ISLE` repo created using...

```
cd /opt/ISLE
docker-compose down
docker stop $(docker ps -q)
docker rm -v $(docker ps -qa)
docker image rm $(docker image ls -q)
docker system prune

cd /opt
sudo mv -f ISLE /opt/upgration-ISLE
sudo git clone https://github.com/Islandora-Collaboration-Group/ISLE.git
sudo chown -R islandora:islandora ISLE/
```
The last four commands in that sequence moved the old `/opt/ISLE` directory out of the way and cloned the `ISLE` project to make a new `/opt/ISLE` directory.

## Working Through 'migration_export_checklist.md'

I have a fork of https://github.com/Islandora-Collaboration-Group/ISLE-Documentation at https://github.com/DigitalGrinnell/ISLE-Documentation.  The work that follows was subsequently guided by https://github.com/DigitalGrinnell/ISLE-Documentation/docs/04_installation_migration/migration_export_checklist.md and the `dg-migration` branch of that repository is where I captured the command sequence used to harvest existing components.  

### For the Apache Container...
As user `islandora` on host `dgdocker1`...
```
cd /opt/ISLE
mkdir persistent
rsync -aruvi vagrant@digital.grinnell.edu:/var/www/drupal7/. persistent/ --progress
```

This `rsync` brought the entire production Drupal webroot from my old production server to the new one; some subsequent clean-up was in order, so I did this...
```
cd /opt/ISLE/persistent/html
rm -f *.jpg *.tiff *.csv *.html *.bak *.sh *.css *.sql drupal7.41
rm -fr transcribe images
rm -fr sites/prairieJournal
rm -f sites/all/modules/custom/*.gz
du -a . | sort -n -r | head -n 25                  # list 25 largest folders/files
rm -f sites/default/files/backup_migrate/manual/*
rm -f sites/default/temp.sql
rm -f sites/default/files/ihcReport_progress.log
rm -f sites/default/files/files.tar.gz
rm -f sites/default/digital.grinnell.edu.sql
rm -f sites/default/files/coversheets.backup
rm -f sites/all/themes/themes.tar.gz
rm -fr sites/all/modules/contrib/.out-of-the-way
rm -fr sites/all/modules/custom/.out-of-the-way
rm -fr sites/default/modules/.out-of-the-way
rm -fr sites/default/modules/contrib/.out-of-the-way
rm -fr sites/default/themes/.out-of-the-way
rm -fr sites/default/themes/.out-of-the-way/Digital_Grinnell_Moved_13-May-2016/.out-of-the-way
rm -fr sites/default/modules/contrib/islandora_xml_forms/builder/self_transforms/out-of-the-way
rm -fr sites/default/modules/contrib/islandora_xml_forms/builder/transforms/out-of-the-way
rm -f sites/default/files/drupal6.sql
rm -f sites/default/files/ihcCollect_progress.log
find . -type f -size +99M                           # check for any files greater than 99M in size
```

### For the Fedora Container...
No commands to show here.  This data was backed up and copied earlier to the NFS share shown here mounted as `/data` on `dgdocker1`:
```
Filesystem                   Size  Used Avail Use% Mounted on
storage2:/nfsshare_dgdocker  1.3T  1.1T  209G  84% /data
```

### For GSearch...
No commands to show here; nothing needed.

### For the MySQL Container...
As user `vagrant` on host `digital7`...
```
cd /var/www/drupal7/sites/default
drush cc all
mysqldump -u digital -p digitalGrinnell | gzip > dg7.sql.gz
```
As user `islandora` on host `dgdocker1`...
```
mkdir -p /opt/ISLE/imports
rsync -aruvi vagrant@digital7.grinnell.edu:/var/www/drupal7/sites/default/dg7.sql.gz /opt/ISLE/imports/ --progress
```
### For Tomcat...
No commands to show here; nothing needed.

### For the Solr Container...
On host `dgdocker1` as user `mcfatem`...

```
mkdir -p /data/solr
rsync -aruvi vagrant@repositoryx.grinnell.edu:/usr/local/fedora/solr/. /data/solr/ --progress
```

## Fedora and FGS Status
The DG Fedora repository was previously backed up from the `RepositoryX` server and the backup is stored in an NFS share, `storage2:/nfsshare_dgdocker  1.3T  1.1T  210G  84% /data`, mounted on the `DGDocker1` host at `/data`.  My `docker-compose.yml` includes the following mappings designed to make use of this data...

```
solr:
  volumes:
    - /data/solr:/usr/local/solr

mysql:
  volumes:
    - ./imports:/import    # comment this out AFTER successful initial spin up and Drupal DB creation

fedora:
  volumes:
    - /data/datastreamStore:/usr/local/fedora/data/datastreamStore   
    - /data/objectStore:/usr/local/fedora/data/objectStore           
    - /data/resourceIndex:/usr/local/fedora/data/resourceIndex       
```

## Modifying 'settings.php'
In order for the imported Drupal database to work properly here, we need to bring the new DB credentials, and the new/temporary address, into `/opt/ISLE/persistent/html/sites/default/settings.php` with these changes...

### Old 'settings.php'...

```
$base_url = 'https://digital.grinnell.edu';  // NO trailing slash!

$cookie_domain = 'digital.grinnell.edu';

$databases = array (  
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'digitalGrinnell',
      'username' => 'digital',
      'password' => 'xxxxxxxxx',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

### New 'settings.php'...

```
$base_url = 'https://dgdocker1.grinnell.edu';  // NO trailing slash!  @TODO...This is a temporary address!

$cookie_domain = 'dgdocker1.grinnell.edu';     // @TODO...This is temporary!

$databases = array (  
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'isle_dg',
      'username' => 'isle_dg_user',
      'password' => 'xxxxxxxxxxxxxxxxxxxxxxxx',
      'host' => 'isle-mysql-dg',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

## Fresh Spin-Up

From a terminal open to `islandora@DGDocker1`, all previous Docker bits were cleaned and a new instance created like so...

```
cd /opt/ISLE
docker-compose down
docker stop $(docker ps -q)
docker rm -v $(docker ps -qa)
docker image rm $(docker image ls -q)
docker system prune

docker-compose up | tee up.out
```

## After Initial Spin-Up -- DB Import

Using *Portainer* at https://portainer1.grinnell.edu, and a terminal into the `isle-mysql-dg` container...
`root@f3d2a5159757:/# gunzip -c import/dg7.sql.gz | mysql -u isle_dg_user -p isle_dg`

## Login as User 1 - System Admin

I successfully visited https://dgdocker1.grinnell.edu/, with my `docker-compose up` command still running, and successfully logged in as `User 1`, alias *System Admin*.  Next, I visited https://dgdocker1.grinnell.edu/admin/reports/status to get an sense of any "known problems" and found three warnings...

```
User warning: The following module is missing from the file system: <em class="placeholder">ldap_sso</em>. For information about how to fix this, see <a href="https://www.drupal.org/node/2487215">the documentation page</a>. in trigger_error() (line 1143 of /var/www/html/includes/bootstrap.inc).
Warning: ldap_start_tls(): Unable to start TLS: Can't contact LDAP server in ldap_start_tls() (line 297 of /var/www/html/sites/all/modules/contrib/ldap/ldap_servers/LdapServer.class.php).
Warning: file_put_contents(private:///.htaccess): failed to open stream: &quot;DrupalPrivateStreamWrapper::stream_open&quot; call failed in file_put_contents() (line 496 of /var/www/html/includes/file.inc).
```

...and a handful of other issues shown in these screen shots...

![Status Report Part 1 of 2](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/dg-migration/Screen%20Shot%202018-12-11%20at%208.59.09%20PM.png
       "Status Report - Part 1")
![Status Report Part 3 of 2](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/dg-migration/Screen%20Shot%202018-12-11%20at%209.00.03%20PM.png
              "Status Report - Part 2")

## Spin Down and Back Up Again

I still have an 'open' `docker-compose up | tee up.out` process running, so now I'm going to shut that down, then bring it back up, but detached from my terminal with `docker-compose up -d`.  Then I'll return to the site and begin to address the warnings and issues reported above.

From a terminal open to `islandora@DGDocker1`, all previous Docker bits were cleaned and a new instance created like so...

```
cd /opt/ISLE
docker-compose down
docker-compose up -d
```
## Accessing the Site

The stack came back up just fine so I developed this handy table of addresses to help with browsing the site and its admin interfaces:

| Address                                                 | Description                 |
| ---                                                     | ---                         |
| https://dgdocker1.grinnell.edu                          | The site                    |
| https://traefik1.grinnell.edu                           | *Traefik* admin interface   |
| https://portainer1.grinnell.edu                         | *Portainer* admin interface |
| https://dgdocker1.grinnell.edu:8081                     | *Tomcat* status             |
| https://dgdocker1.grinnell.edu:8081/fedora/admin/       | *Fedora* Web Administrator  |
| https://dgdocker1.grinnell.edu:8081/fedoragsearch/rest/ | *FGSearch* REST interface    |
| https://dgdocker1.grinnell.edu:8082/solr/#/             | *Solr* admin interface      |

## Addressing Known Issues
I'm currently working through a list of known problems/issues.  I'll address each one individually with its own subsection here...

### One Critical Solr Schema Problem

Checking logs and opening some of the admin interfaces pointed out one critical problem with *Solr*.  *Digital Grinnell* currently runs *Solr* version 4.2.1 but the `isle-solr-dg` container runs version 4.10.4, and there's at least one schema difference between the two... the newer version no longer allows use of a `maxChars` attribute in `field` specifications.  

So, I opened `/home/islandora/solr/collection1/conf/schema.xml` on the *DGDocker1* host and removed `maxChars=300` from four lines like this one:
`<field name="dc.relation_s" type="string" maxChars="300" indexed="true" stored="true" multiValued="true"/>`.  That produced four modified lines like this: `<field name="dc.relation_s" type="string" indexed="true" stored="true" multiValued="true"/>`.  I subsequently killed the `isle-solr-dg` container and did a new `docker-compose up -d`, and the problem was fixed.

### Site Performance is Very Poor

https://dgdocker1.grinnell.edu is working, but it's performing very poorly with very long page-load times.  I'm seeing a number of syslog warnings about `memcache` module not working, and I suspect that could be a big part of the problem.  I'm going to turn `memcache` off and see what happens.

And I got this...
```
root@3c8268dfcc7d:/var/www/html/sites/default# drush dis memcache
memcache is already disabled.                                                                  [ok]
There were no extensions that could be disabled.                                               [ok]
WD memcache: You must enable the PHP memcache (recommended) or memcached extension to use      [error]
memcache.inc.
WD memcache: Failed to connect to memcache server: 127.0.0.1:11211                             [error]
```

Going to have a look at my `settings.php` file now... Found lots of `memcache` gunk at the bottom of the file and have commented it out.

Now when I spin back up I'm getting errors about `memcache_admin`, so...

```
root@f62e2c1cd861:/var/www/html/sites/default# drush dis memcache_admin
root@f62e2c1cd861:/var/www/html/sites/default# drush cc all
```

Eureka!  The site is back and it's MUCH FASTER than before!  Still some obvious issues to be worked out.

### Private Filesystem
So, my production instance of DG declares a "private" file system, with a path outside of the webroot, as advised.  **I don't think the migration documents address this issue, but I could be wrong?**  In any case, I'm going to create a path on my host and map it into the `apache` service container in my `docker-compose.yml`, and see what happens.

As user `islandora` on the `DGDocker1` host...

```
mkdir ~/private
chmod 777 ~/private
cd /opt/ISLE
docker-compose rm -f -s -v apache     # kills ONLY the isle-apache-dg container
```  
And from a terminal on `digital7`...
```
rsync -aruvi /var/private/. root@dgdocker1.grinnell.edu:/home/islandora/private/ --progress
```
Adding this line to the `apache | volumes` portion of `docker-compose.yml`...
```
      - /home/islandora/private:/var/private
```
Now back to `/opt/ISLE` and `docker-compose up -d`.  Previous warnings about not finding `private:///.htaccess` are gone so I think we're good now!

### LDAP Problems
Seeing lots of LDAP issues in the new site.  For example...
```
User warning: The following module is missing from the file system: <em class="placeholder">ldap_sso</em>. For information about how to fix this, see <a href="https://www.drupal.org/node/2487215">the documentation page</a>. in trigger_error() (line 1143 of /var/www/html/includes/bootstrap.inc).

Warning: ldap_start_tls(): Unable to start TLS: Can't contact LDAP server in ldap_start_tls() (line 297 of /var/www/html/sites/all/modules/contrib/ldap/ldap_servers/LdapServer.class.php).

LDAP Server status: Grinnell_LDAP (ldap.grinnell.edu)	Connection: Failed, check logs for details.
```

In response to these warnings, I did the following in a bash terminal inside the `isle-apache-dg` container...

```
root@8b20dd1b4177:/var/www/html/sites/default# drush dl ldap_sso
Project ldap_sso (7.x-2.3) downloaded to /var/www/html/sites/default/modules/contrib/ldap_sso. [success]
root@8b20dd1b4177:/var/www/html/sites/default# drush cc all
```
That took care of the first warning about `ldap_sso` not found in the filesystem.  Now checking the logs for details about the `ldap_start_tls()` warning...

Comparing LDAP settings from https://digital.grinnell.edu with those at https://dgdocker1.grinnell.edu I found that in the new site the `ldap_authentication` module was disabled; that was not the case with https://digital.grinnell.edu.  So, I enabled the module and ensured it is turned ON, then made sure all the settings match between instances of the LDAP module.

Still no good, but I learned that LDAP via TLS uses port 636, so I'm going to try adding this spec to my `docker-compose.yml`:
```
ports:
  - "636:636"      # added for LDAP	over TLS
  - "389:389"      # added for LDAP	over SSL
```

These changes did NOT help.  So I tried opening them to *Traefik*, and again, no help.  8^(  Going to need help from ITS, or someone, on this issue.

**Important!**  Julia did some research into this issue just before winter break and determined that the LDAP problem is in PHP 5.6.  We need to run ISLE in PHP 7!

## Exporting the DB

With the exception of LDAP, things are working pretty well, so now I need to export my database so that I can use it elsewhere.  Before doing this I need to remove the `ihc`, `idu` and `icu` modules...they are a mess and keep getting in the way!  So I disabled each of these and will now `Uninstall` the following:

  - Fedora Health Check (FHC)
  - Fedora Import Control (FIC)
  - Fedora Object Control (FOC)
  - ICG Tuque Fedora Access
  - Islandora Drush Utility (IDU)
  - Islandora Health Check (IHC)
  - Islandora Sync Field Collection
  - Islandora Sync Relation
  - Islandora XQuery
  - Memcache
  - Memcache Admin
  - Taxonomy Access Control Lite Create
  - Taxonomy Permissions
