# Lab08 
В рамках выполнения данной лабораторной работы мною были выполнены команды из tutorial с некоторыми изменениями:
1) Скопирован репозиторий из lab07.
```bash
$ git clone https://github.com/${GITHUB_USERNAME}/lab07 lab08
Клонирование в «lab08»...
remote: Enumerating objects: 173, done.
remote: Counting objects: 100% (173/173), done.
remote: Compressing objects: 100% (93/93), done.
remote: Total 173 (delta 58), reused 160 (delta 51), pack-reused 0 (from 0)
Получение объектов: 100% (173/173), 45.43 КиБ | 912.00 КиБ/с, готово.
Определение изменений: 100% (58/58), готово.
```
3) Написан Dockerfile:
```dockerfile
$ cat > Dockerfile << 'EOF'
> FROM ubuntu:22.04

RUN apt update && apt install -yy \
    gcc \
    g++ \
    cmake \
    git

COPY . print/
WORKDIR print

RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=_in
stall -DBUILD_TESTS=OFF

RUN cmake --build _build
RUN cmake --build _build --target install

ENV LOG_PATH /home/logs/log.txt
VOLUME /home/logs
WORKDIR _install/bin
ENTRYPOINT ./demo
EOF
```
Версия 18:04 заменена на 22:04 для совместимости в FetchContent
3) Произведена сборка Docker-образа
```bash
$ sudo docker build --no-cache -t logger .
[+] Building 103.8s (13/13) FINISHED                         docker:default
 => [internal] load build definition from Dockerfile                   0.0s
 => => transferring dockerfile: 508B                                   0.0s
 => [internal] load metadata for docker.io/library/ubuntu:22.04        0.5s
 => [internal] load .dockerignore                                      0.0s
 => => transferring context: 2B                                        0.0s
 => CACHED [1/8] FROM docker.io/library/ubuntu:22.04@sha256:962f6cade  0.0s
 => [2/8] RUN apt update && apt install -yy     gcc     g++     cmak  52.7s
 => [internal] load build context                                      0.1s
 => => transferring context: 58.63kB                                   0.1s
 => [3/8] COPY . print/                                                0.3s
 => [4/8] WORKDIR print                                                0.1s
 => [5/8] RUN cmake -H. -B_build -DCMAKE_BUILD_TYPE=Release -DCMAKE_  10.3s
 => [6/8] RUN cmake --build _build                                    35.7s
 => [7/8] RUN cmake --build _build --target install                    1.2s
 => [8/8] WORKDIR _install/bin                                         0.1s
 => exporting to image                                                 2.9s
 => => exporting layers                                                2.9s
 => => writing image sha256:01ed3f233a22991dc54f58cb082f5a111b67ce42f  0.0s
 => => naming to docker.io/library/logger                              0.0s
```
Результат сборки:
```bash
$ sudo docker images
REPOSITORY   TAG       IMAGE ID       CREATED          SIZE
logger      latest    01ed3f233a22   54 minutes ago   538MB
```
4) Запущен контейнер
```bash
$ docker run -it -v "$(pwd)/logs/:/home/logs/" logger
text1
text2
text3
```
Результат:
```bash
$ sudo docker inspect logger
[
    {
        "Id": "sha256:01ed3f233a22991dc54f58cb082f5a111b67ce42f09f2282e584de9c19345557",
        "RepoTags": [
            "logger:latest"
        ],
        "RepoDigests": [],
        "Parent": "",
        "Comment": "buildkit.dockerfile.v0",
        "Created": "2026-05-14T20:16:07.033924102+03:00",
        "DockerVersion": "",
        "Author": "",
        "Config": {
            "Hostname": "",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
                "LOG_PATH=/home/logs/log.txt"
            ],
            "Cmd": null,
            "Image": "",
            "Volumes": {
                "/home/logs": {}
            },
            "WorkingDir": "/print/_install/bin",
            "Entrypoint": [
                "/bin/sh",
                "-c",
                "./demo"
            ],
            "OnBuild": null,
            "Labels": {
                "org.opencontainers.image.version": "22.04"
            }
        },
        "Architecture": "amd64",
        "Os": "linux",
        "Size": 538022761,
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/eecuj0bimhsq8h7u9qzt9jk9s/diff:/var/lib/docker/overlay2/ls0ap6k7j0a7dowj5o273f69l/diff:/var/lib/docker/overlay2/n7htfgmqz51dr6qdwrswkpbx7/diff:/var/lib/docker/overlay2/w3tu6v7hvfo3wwus0nhd3vo7q/diff:/var/lib/docker/overlay2/pn7etdpqoj95yohj1bcignozq/diff:/var/lib/docker/overlay2/0ctx3a4swrjamfdb8mr9eywnq/diff:/var/lib/docker/overlay2/d7197d65ee79d0e64fbc7e1532743df3be0ceea00b624fe5ff7984ebce755f10/diff",
                "MergedDir": "/var/lib/docker/overlay2/wdkbyrcd5wok47fr6ppe9009i/merged",
                "UpperDir": "/var/lib/docker/overlay2/wdkbyrcd5wok47fr6ppe9009i/diff",
                "WorkDir": "/var/lib/docker/overlay2/wdkbyrcd5wok47fr6ppe9009i/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:39fbf5f8fe523b2ea819cd8eb2bf68807d8eaee383549f7ab80a44503ed6860b",
                "sha256:74316cb8488652481be9942950fc0d57662bd096be8056495bb4e992a527e7cb",
                "sha256:cce468222a525730bb8f7b1968b7d273f6fa3a5b46395e0a56cb2e12042ea2f7",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef",
                "sha256:096447787f18357ddd19a86ad8674ae18de19ddd22802a6c4728fa46269dd6da",
                "sha256:a9d53b492232471fbe83375b9f3da1ea3164ad4658fc1f2fea9c1c8f002c9856",
                "sha256:500f8de6cfd9aa39182b0778699589dbd83b12be1c854f1ff340fc2b5f04c3b0",
                "sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef"
            ]
        },
        "Metadata": {
            "LastTagTime": "2026-05-14T20:16:09.965250976+03:00"
        }
    }
]
```
```bash
$ cat logs/log.txt
text1
text2
text3
```
5) В ci.yml добавлена часть кода про docker
```cmake
...
