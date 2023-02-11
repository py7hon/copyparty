copyparty is availabe in these repos:
* https://hub.docker.com/r/copyparty
* https://github.com/9001?tab=packages&repo_name=copyparty


# getting started

run this command to grab the latest copyparty image and start it:
```bash
docker run --rm -it -u 1000 -p 3923:3923 -v /mnt/nas:/w -v $PWD/cfgdir:/cfg copyparty/ac
```

* `/w` is the path inside the container that gets shared by default, so mount one or more folders to share below there
* `/cfg` is an optional folder with zero or more config files (*.conf) to load
* `copyparty/ac` is the recommended [image edition](#editions)
* you can download the image from github instead by replacing `copyparty/ac` with `ghcr.io/9001/copyparty-ac`

i'm unfamiliar with docker-compose and alternatives so let me know if this section could be better 🙏


## configuration

the container has the same default config as the sfx and the pypi module, meaning it will listen on port 3923 and share the "current folder" (`/w` inside the container) as read-write for anyone

the recommended way to configure copyparty inside a container is to mount a folder which has one or more [config files](https://github.com/9001/copyparty/blob/hovudstraum/docs/example.conf) inside; `-v /your/config/folder:/cfg`

* but you can also provide arguments to the docker command if you prefer that
* config files must be named `something.conf` to get picked up


## editions

with image size after installation and when gzipped

* `min` (57 MiB, 20 gz) is just copyparty itself
* `im` (69 MiB, 24 gz) can create thumbnails using pillow (pics only)
* `ac` (163 MiB, 56 gz) is `im` plus ffmpeg for video/audio thumbnails + audio transcoding
* `iv` (211 MiB, 73 gz) is `ac` plus vips for faster heif / avic / jxl thumbnails 
* `dj` (309 MiB, 104 gz) is `iv` plus beatroot/keyfinder to detect musical keys and bpm

`ac` is recommended since the additional features available in `iv` and `dj` are rarely useful


## detecting bpm and musical key

the `dj` edition comes with `keyfinder` and `beatroot` which can be used to detect music bpm and musical keys

enable them globally in a config file:
```yaml
[global]
e2dsa, e2ts  # enable filesystem indexing and multimedia indexing
mtp: .bpm=f,t30,/mtag/audio-bpm.py  # should take ~10sec
mtp: key=f,t190,/mtag/audio-key.py  # should take ~50sec
```

or enable them for just one volume,
```yaml
[/music]  # share name / URL
  music   # filesystem path inside the docker volume `/w`
  flags:
    e2dsa, e2ts
    mtp: .bpm=f,t30,/mtag/audio-bpm.py
    mtp: key=f,t190,/mtag/audio-key.py
```

or using commandline arguments,
```
-e2dsa -e2ts -mtp .bpm=f,t30,/mtag/audio-bpm.py -mtp key=f,t190,/mtag/audio-key.py
```


# build the images yourself

put `copyparty-sfx.py` into `../dist/` (or [build that from scratch](../docs/devnotes.md#just-the-sfx) too) then run `make`