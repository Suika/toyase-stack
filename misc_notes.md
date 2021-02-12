
# 2020-02-11 suika latest version in branch "ayase"


## Create buckets by hand:
$ docker exec -it  seaweed-master /usr/bin/weed shell -master=seaweed-master:9333 -filer=seaweed-filer:8888
> s3.bucket.create -name asagi-thumbs
> s3.bucket.create -name asagi-images



echo "s3.bucket.create -name asagi-thumbs\ns3.bucket.create -name asagi-images\nexit\n" | docker exec -it  seaweed-master /usr/bin/weed shell -master=seaweed-master:9333 -filer=seaweed-filer:8888

$ docker exec -i  seaweed-master /usr/bin/weed shell -master=seaweed-master:9333 -filer=seaweed-filer:8888 <<EOF
s3.bucket.create -name asagi-thumbs
s3.bucket.create -name asagi-images
exit
EOF

## Snippets
### Startup
Init:
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" pull
$ docker-compose -f "docker-compose.yml" up -d
$ docker exec -it  seaweed-master /usr/bin/weed shell -master=seaweed-master:9333 -filer=seaweed-filer:8888
> s3.bucket.create -name asagi-thumbs
> s3.bucket.create -name asagi-images
[^c]
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d

Subsequent:
$ docker-compose -f "docker-compose.yml" up -d
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d

Oneliner to startup:

$ docker-compose -f "docker-compose.yml" up -d 
sleep 300
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d


Apply docker-compose updates:
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" down
$ docker-compose -f "docker-compose.yml" up -d
$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" up -d

### Rebuild from scratch
! DELETES EVERYTHING !

$ docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" down
$ sudo rm  -r seaweedfs/ mysql/
$ bash ./startup.sh

### Diagnosis and debugging
Check if torako is working:
docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml" logs -t torako


View container logs:
$  docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml"  logs -f 4chan-scrape-db
$  docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml"  logs -f torako
$  docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml"  logs -f 4chan-scrape-db
$  docker-compose -f "docker-compose.yml" -f "docker-compose.ayase.yml"  logs -f 4chan-scrape-db


Check listenting ports:
$ sudo netstat -tunlp
$ sudo netstat -tunlp | grep 7865
$ sudo netstat -tunlp |grep docker

Remove containers + network, leave images alone.
$ docker-compose -f "docker-compose.yml" down



Show docker networks:
$ docker network ls

Show docker containers:
$ docker container ls -a

Restart docker if it misbehaves:
$ sudo systemctl restart docker

# == Wiping local docker images and containers ==
DANGEROUS!
see: https://gist.github.com/evanscottgray/8571828

Stop all containers:
$ docker stop $(docker ps -a -q)

Stop all containers by force
$ docker kill $(docker ps -q)

Remove all containers
$ docker rm $(docker ps -a -q)

Remove all docker images
$ docker rmi $(docker images -q)

Purge the rest
$ docker system prune --all --force --volumes

Remove all docker networks:
$ docker network rm $(docker network ls -q)
