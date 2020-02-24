---

title : "[Docker] Docker Secret Lab Docker (Part 8)"
categories: [ngoprek, server, cloud, docker, container]
---
# بِسْمِ اللَّهِ الرَّحْمَنِ الرَّحِيم
---

Lab ini kita akan mencoba service secret yang ada pada docker

## Get started with secrets #####

#### Eksekusi di node `pod-pod0` ###

Membuat secret sebelum itu kita harus menyalakan docker swarm

```shell
docker swarm init
printf "This is a secret" | docker secret create my_secret_data -
```
```BAS
root@pod0:~/latihan/my_app# docker swarm init
Swarm initialized: current node (4juunpj0nqynqwa4wy5ayxii4) is now a manager.

To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-3myyev16e7p7jufe8wnxjvva2bfpdej8vn2i0qo4trg1oqtpp6-5w45qka34re1y6fu6p4pn3e0a 10.1.40.12:2377

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
root@pod0:~/latihan/my_app# printf "This is a secret" | docker secret create my_secret_data -
naq6d6a8ol179z4rivaqylvl0
```
Membuat service redis dan mengizinkan akses secret my_secret_data & Verfikasi

```shell
docker service  create --name redis --secret my_secret_data redis:alpine
docker service ps redis
```
```shell
root@pod0:~/latihan/my_app# docker service  create --name redis --secret my_secret_data redis:alpine

lkilpy5cks3j34qdjiw85pmsm
overall progress: 1 out of 1 tasks 
1/1: running   [==================================================>] 
verify: Service converged 
root@pod0:~/latihan/my_app# docker service ps redis
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
lvot7d1uge6p        redis.1             redis:alpine        pod0                Running             Running 2 minutes ago                       
```

Membaca content di dalam container
```shell
docker ps --filter name=redis -q
docker container exec $(docker ps --filter name=redis -q) ls -l /run/secrets
docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
```

```shell
root@pod0:~/latihan/my_app# docker ps --filter name=redis -q
43963304e729
root@pod0:~/latihan/my_app# docker container exec $(docker ps --filter name=redis -q) ls -l /run/secrets
total 4
-r--r--r--    1 root     root            16 Oct  7 13:30 my_secret_data
root@pod0:~/latihan/my_app# docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
This is a secret
```

untuk Verifikasi secret tidak ada di image hasil commit container
```shell
docker commit $(docker ps --filter name=redis -q) committed_redis
docker run --rm -it committed_redis cat /run/secrets/my_secret_data
```

```shell
root@pod0:~/latihan/my_app# docker commit $(docker ps --filter name=redis -q) committed_redis
sha256:6cbc73dccdf4b2f1a7a69615bdca8571357f9c889c99730da91c52ebd2d1ccbb
root@pod0:~/latihan/my_app# docker run --rm -it committed_redis cat /run/secrets/my_secret_data
```

Mencoba hapus secret (pastikan gagal karena masih ada service redis dan memiliki akses ke secret)
```shell
docker secret ls
docker secret rm my_secret_data
```

Untuk menghapus izin akses secret pada service redis dengan
```shell
docker service update --secret-rm my_secret_data redis
```

Verifikasi secret tidak ada di dalam container service redis bisa dengan :
```shell
root@pod0:~/latihan/my_app# docker container exec $(docker ps --filter name=redis -q) cat /run/secrets/my_secret_data
cat: can't open '/run/secrets/my_secret_data': No such file or directory
```

Dan juga kita bisa menghapus secret dan service redis :
```shell
docker service rm redis
docker secret rm my_secret_data
```
```shell
root@pod0:~/latihan/my_app# docker service rm redis
redis
root@pod0:~/latihan/my_app# docker secret rm my_secret_data
my_secret_data
```

# Happy,  Enjoy Ngoprek ~
