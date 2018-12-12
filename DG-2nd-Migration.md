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
No commands to show here.  This data was backed up and copied earlier to the NFS share shown here mounted on as `/data` on `dgdocker1`:
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
In order for the imported Drupal database to work properly here, we need to bring the new DB credentials into `/opt/ISLE/persistent/html/sites/default/settings.php` with these changes...

### Old 'settings.php'...

```
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

Using *Portainer* at https://portainer1.grinnell.edu, and a terminal into the `isle-mysql-dg` container and the `/import` folder...
`root@f3d2a5159757:/import# gunzip -c dg7.sql.gz | mysql -u isle_dg_user -p isle_dg`
