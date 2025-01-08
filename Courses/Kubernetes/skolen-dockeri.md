# Kubernetes školení

* [docker](#kontejnery-dockerpodman)
* [kubernetes](skoleni-kubernetes.md)

## Kontejnery (docker/podman)

- ale vlastně procesy

```bash
$ docker ps

CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
503ef5987c65   nginx     "/docker-entrypoint.…"   2 minutes ago   Up 2 minutes   80/tcp    nginx

$ ps aux | grep nginx
root       17605  0.0  0.0  10644  5984 ?        Ss   20:20   0:00 nginx: master process nginx -g daemon off;
101        17670  0.0  0.0  11040  2616 ?        S    20:20   0:00 nginx: worker process
```

## Spuštění kontejneru na pozadí

```
docker run -d httpd
```
```
docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS         PORTS     NAMES
93debe1c8d8b   httpd     "httpd-foreground"       10 seconds ago   Up 7 seconds   80/tcp    thirsty_chebyshev
```

## Spuštění kontejneru s proměnnými prostředí

```
docker run -d --name mysql -e MYSQL_ROOT_PASSWORD=pass -e MYSQL_DATABASE=db mysql
```

```
$ docker ps --format 'table {{.Image}}\t{{.Names}}'
IMAGE     NAMES
mysql     mysql
```


## Spuštění kontejneru interaktivně

```
docker run -ti --rm  fedora:33 /bin/bash
```

```
[root@66e3efe3863a /]#  date ; ls /root
Sat Mar 20 17:14:28 UTC 2021
anaconda-ks.cfg  anaconda-post-nochroot.log  anaconda-post.log  original-ks.cfg

[root@66e3efe3863a /]# exit

$ docker ps | grep fedora -c
0
```

## Přidání svazku do kontejneru

**Pozor: Executing podman: Permission denied**

***pokud používáme SELinux musime použit `auto-relabelling` mechanizmus:  přidáním z/Z znaku na konec mountpointu***

```
mkdir data
$ docker run -ti --rm -v $(pwd)/data:/data:z  ubuntu bash
```



```
mkdir data
$ docker run -ti --rm -v $(pwd)/data:/data  ubuntu bash
```

```
root@a3d53a02f2cc:/# echo "Best from $(hostname), $(date)" > /data/regards
root@a3d53a02f2cc:/# exit

$ cat data/regards
Best from a3d53a02f2cc, Sat Mar 20 17:27:19 UTC 2021
```

## Dockerfile & build

```sh
ls
Dockerfile

cat Dockerfile
FROM alpine
MAINTAINER Ondrej Adam Benes <obenes0@centrum.cz>
CMD ["echo", "Hey", "there!"]

docker build -t hey-there:test .
Sending build context to Docker daemon  2.048kB
Step 1/3 : FROM alpine
 ---> 28f6e2705743
Step 2/3 : MAINTAINER Ondrej Adam Benes <obenes0@centrum.cz>
 ---> Running in 85d407e8d1a0
Removing intermediate container 85d407e8d1a0
 ---> c3adc32fef29
Step 3/3 : CMD ["echo", "Hey", "there!"]
 ---> Running in 1406bc51694f
Removing intermediate container 1406bc51694f
 ---> 3f5ebed9c956
Successfully built 3f5ebed9c956
Successfully tagged hey-there:test

docker run --rm hey-there:test
Hey there!
```
