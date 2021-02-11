# FoolStack v2 - torako + seaweedfs + ayase
Quick guide to using this stuff.
Non-local access is outside the current scope.

## Install preprequisites
* [Install docker](https://docs.docker.com/engine/install/)
* [Install docker-compose](https://docs.docker.com/compose/install/)

## Quickstart
```
git clone https://github.com/Suika/toyase-stack

docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" pull
docker-compose -f "docker-compose.yml" up -d

docker exec seaweed-master -it /usr/bin/weed -master=seaweed-master:9333 -filer=seaweed-filer:8888
s3.bucket.create -name asagi-thumbs
s3.bucket.create -name asagi-images
exit

docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d
```

## Prep
This script makes them if they do not exist and assigns permissions permissive enough to make sure seaweedfs can use them.  
Technically not needed if you run docker and use seaweedfs v2.24.
```
$ bash mkdirs.sh
```

## Configuration
### Seaweed
Look in docker-compose.yml#L7, see `-volumeSizeLimitMB`? That is the size of volumes that seaweed will create and pre-allocate space for.  
At least 2 volumes will be created. One for internal metadata of seaweed and another for your data. This numbre is only a softlimit.  
So increasing it in the future will allow for the volumes to grow, but think about it beforehand and set a number you like. I recommend 200G.  
Ports will be explained sometime later, or figure it out [here](https://github.com/chrislusf/seaweedfs/wiki) yourself.  
Port 7865 is the only one you should think about for now, this is where the files can be loaded from.

### Torako
Check `config/torako.toml`  
L67-81: Set the boards that you want to archive.  
L98,102: Should media and thumbs be saved.  
L115: If this a new setup, keep it `true`, if it's an old setup migration docs are TBA  
L152: I don't recomend changing it for now, unless you know how seaweed works.  
L251: DB config, no real need to change unless you do productive setup.  
L264,270: Keep it this way if you only use torako. But it's up to you.  
L369-381: S3/Seaweed config. For now keep it that way. The S3 credentials are located in `config.json`

### Ayase
Check `config/ayase.config.toml`  
L27-101: What boards should be show, basically what you do added in `Torako.toml#L67-81`  
L161-162: 7865 is the RO port of seaweed filer, the IP should be of the host where the stack is running or localhost.  
L204-213: DB config. Same as in `Torako.toml` or `docker-compose.yml`

## Running
Pull containers images:
```
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" pull
```

Start/create seaweed + MySQL containers in the background:
```
$ docker-compose -f "docker-compose.yml" up -d
```

Create the buckets where the files will be located (this creates two different collections, basically own volumes different from others):
```
docker exec -it seaweed-master /usr/bin/weed -master=seaweed-master:9333 -filer=seaweed-filer:8888
s3.bucket.create -name asagi-thumbs
s3.bucket.create -name asagi-images
ctrl+d/ctrl+c/exit
```

NOTE:
----
Torako crashing because MySQL is not up in time is fine.  
Torako writing to seaweed before all volumes are up, [not good](https://youtu.be/FNWmn6bKOmk?t=1181).  
Things that will happen are not easy to fix, so don't. Thx.
----

Start/create Torako + Ayase containers in the background:
```
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d
```

Stop/remove containers:
```
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" down
```


### Viewing logs
```
$ docker-compose -f "docker-compose.yml" logs
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" logs ayase
```

### Running commands inside containers manually:
```
$ docker exec -it 4chan-scrape-db mysql -uroot -pbunbuncha
container.mysql> SELECT host, user FROM mysql.user;
```

Installing packages in container for diagnostics:
```
apt-get update && apt-get install -y traceroute curl iputils
```


## Ayase website
In a web browser, open this address to view the ayase instance provided by the container "ayase":

`http://127:0.0.1:29080/`


## DB Export
* Note: The method demonstrated does not strip secrets such as deletion passwords or IP addresses from the generated file.

Backup the entire database:
```
$ docker exec "4chan-scrape-db" /usr/bin/mysqldump -h"localhost" -u"root" -p"bunbuncha" --tz-utc --quick --opt --single-transaction  --skip-lock-tables --all-databases  | gzip > "4chan-scrape-db.all-databases.t`date -u +%s`.sql.gz"
```

Export a single board:
```
$ docker exec "4chan-scrape-db" /usr/bin/mysqldump -h"localhost" -u"root" -p"bunbuncha" --tz-utc --quick --opt --single-transaction  --skip-lock-tables "asagi" "g" "g_images" | gzip > "4chan-scrape-db.g.t`date -u +%s`.sql.gz"
```

Export multiple boards:
```
$ docker exec "4chan-scrape-db" /usr/bin/mysqldump -h"localhost" -u"root" -p"bunbuncha" --tz-utc --quick --opt --single-transaction  --skip-lock-tables "asagi" "g" "g_images"  "co" "co_images" | gzip > "4chan-scrape-db.my_boards_export.t`date -u +%s`.sql.gz"
```