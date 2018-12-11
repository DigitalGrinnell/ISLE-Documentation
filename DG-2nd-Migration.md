---
Note:  The canonical copy of this text can be found at [https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-2nd-Migration.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-2nd-Migration.md).
---
# DG 2nd Migration

This migration attempt commenced on 11-Dec-2018 by MAM on CentOS 7 host `DGDocker1.Grinnell.edu` (132.161.132.103) following the abandoned `upgration` attempt described in [https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md](https://github.com/DigitalGrinnell/ISLE-Documentation/blob/master/DG-Migration-History.md)

## DGDocker1 Cleanup
From a terminal open to `mcfatem@DGDocker1`, all previous Docker bits were cleaned using...

```
cd /opt/ISLE
docker-compose down
docker stop $(docker ps -q)
docker rm -v $(docker ps -qa)
docker image rm $(docker image ls -q)
docker system prune
```
## Fedora and FGS Status
The DG Fedora repository was previously backed up from the `RepositoryX` server and the backup is stored in an NFS share, `storage2:/nfsshare_dgdocker  1.3T  1.1T  210G  84% /data`, mounted on the `DGDocker1` host at `/data`.  My `docker-compose.yml` includes the following mappings designed to make use of this data...

```
solr:
  volumes:
    - /data/solr:/usr/local/solr

fedora:
  volumes:
    - /data/datastreamStore:/usr/local/fedora/data/datastreamStore   
    - /data/objectStore:/usr/local/fedora/data/objectStore           
    - /data/resourceIndex:/usr/local/fedora/data/resourceIndex       
```
