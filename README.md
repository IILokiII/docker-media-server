# Intro
Plex, Radarr, Sonarr, Bazarr, Prowlarr, Transmission and Tautulli (optional) running on Docker with encrypted Google Drive volume via rclone

This was inspired by [animosity22's homescripts](https://github.com/animosity22/homescripts) but I wanted it to be all inside docker without having to mess about with systemd and figuring out ways to make the containers stop if rclone had problems, with this setup that's no longer an issue because docker handless all that stuff

I guess this is just the lazy version of homescripts ¯\\_(ツ)_/¯

## Requirements

  - Running Linux (Windows not tested)
  - [Docker engine & compose plugin installed](https://docs.docker.com/engine/install/)
  - [Rclone installed](https://rclone.org/)

# Guide

## Install rclone Docker plugin

Follow [documentation](https://rclone.org/docker/) (recommended) or follow these steps:

  0. Install fuse

  ```bash
  sudo apt-get install fuse # apt - Ubuntu / Debian
  sudo dnf install fuse # dnf - Fedora
  ```
  1. Create plugin directories
  
  ```bash
  # both of these directories can be changed later
  sudo mkdir -p /var/lib/docker-plugins/rclone/config # config directory
  sudo mkdir -p /var/lib/docker-plugins/rclone/cache # cache directory
  ```
  
  2. Install plugin
  
  ```bash
  sudo docker plugin install rclone/docker-volume-rclone:amd64 args="-v" --alias rclone --grant-all-permissions # install
  sudo docker plugin list # verify it's installed and enabled
  ```
  
  3. Changing the cache or config directories (Optional)

  ```bash
  sudo docker plugin disable rclone # plugin needs to be disabled to edit
  sudo docker plugin set rclone config=/path/to/config cache=/path/to/cache # change whatever settings you want
  sudo docker plugin enable rclone # re-enable plugin
  # look at the rclone documentation for more settings that can be changed
  # the only one I like to change is the cache dir and put it on another disk, that's where the files will be downloaded to when plex is playing them
  ```
  
  4. Rclone config
  
  Follow the [documentation](https://rclone.org/drive/) to create an rclone drive remote and optionally encrypt it with a [crypt remote](https://rclone.org/crypt/)
  
  Your config file is usually located in ~/.config/rclone/rclone.conf and It should look similar to this:
  ```conf
  [YOUR-REMOTE]
type = drive
client_id = YOUR-CLIENT-ID
scope = drive
client_secret = YOUR-CLIENT-SECRET # optional
token = {"access_token":"YOUR-ACCESS-TOKEN"}
team_drive = YOUR-TEAM-DRIVE-ID # optional, if you're using shared drives
root_folder_id = 
service_account_file = ${RCLONE_CONFIG_DIR}/credentials.json # optional, if you're using a service account I keep mine next to the config file

[YOUR-REMOTE-CRYPT] # optional, only if you're encrypting
type = crypt
remote = YOUR-REMOTE:
password = YOUR-PASSWORD
password2 = YOUR-SALT
  ```
  Now just copy your config, and optionally your service account credentials, over to your docker rclone config folder
  
  If you didn't change it on the last step then it's /var/lib/docker-plugins/rclone/config
  
  And you're done setting up the docker rclone plugin
  
## Docker compose

Download the docker-compose.yml file from the repo or copy the following and then edit it to your liking

You'll need to edit paths, the transmission login info and the rclone remote

Optionally the ports, timezones and rclone volume flags

<details>
  
  <summary>docker-compose</summary>

```yaml
services:
  # more info https://hub.docker.com/r/linuxserver/plex
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=1000
      - PGID=1000
      - VERSION=docker
    volumes:
      - /path/to/data:/config
      - rclone:/gdrive # mapping the rclone volume defined at the bottom to the gdrive directory on the container
    restart: unless-stopped


  # more info https://hub.docker.com/r/linuxserver/radarr
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
      - /path/to/downloads:/downloads # needs the same download directory as transmission
      - rclone:/gdrive # mapping the rclone volume defined at the bottom to the gdrive directory on the container
    ports:
      - 7878:7878
    restart: unless-stopped


  # more info https://hub.docker.com/r/linuxserver/sonarr
  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
      - /path/to/downloads:/downloads # needs the same download directory as transmission
      - rclone:/gdrive # mapping the rclone volume defined at the bottom to the gdrive directory on the container
    ports:
      - 8989:8989
    restart: unless-stopped


  # more info https://hub.docker.com/r/linuxserver/bazarr
  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
      - /path/to/downloads:/downloads # needs the same download directory as transmission
      - rclone:/gdrive # mapping the rclone volume defined at the bottom to the gdrive directory on the container
    ports:
      - 6767:6767
    restart: unless-stopped

  
  # more info https://hub.docker.com/r/linuxserver/prowlarr
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:develop
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
    ports:
      - 9696:9696
    restart: unless-stopped


  # VERY OPTIONAL
  # more info https://hub.docker.com/r/linuxserver/tautulli
  tautulli:
    image: lscr.io/linuxserver/tautulli:latest
    container_name: tautulli
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/data:/config
    ports:
      - 8181:8181
    restart: unless-stopped


  # more info https://hub.docker.com/r/linuxserver/transmission
  transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - TRANSMISSION_WEB_HOME=/flood-for-transmission/
      - USER=ChangeMyUsername
      - PASS=ChangeMyPassword
      - PEERPORT=51413 # bittorrent port, can be whatever
    volumes:
      - /path/to/data:/config
      - /path/to/downloads:/downloads
    ports:
      - 9091:9091 # this is the webui port, internally has to be 9091, external can be whatever
      - 51413:51413 # bittorrent port, can be whatever externally or internally, needs to be the same as PEERPORT in environment
      - 51413:51413/udp
    restart: unless-stopped


volumes:
  rclone: # if you change this, you'll also need to change the volumes above from rclone:/gdrive to whatever:/gdrive
    driver: rclone
    driver_opts:
      remote: 'YOUR-REMOTE-CRYPT:' # use your crypt remote name or just the regular one if not using encryption
      allow_other: 'true'
      umask: 002 
      dir_cache_time: 5000h 
      poll_interval: 10s
      vfs_cache_mode: full
      vfs_cache_max_size: 200G # max size of cache
      vfs_cache_max_age: 10h # max age of cache
      vfs_cache_poll_interval: 5m
      vfs_write_back: 1h # how long to wait before uploading the files to the remote
    # look at the documentation for an explanation of these flags and for more flags to add if you need them
    # animosity22 also explains them well: https://github.com/animosity22/homescripts/blob/master/systemd/rclone-movies.service
```
  
</details>

After you have the docker-compose.yml configured simply run `docker compose up` to start it and it should automatically create rclone volume and all the containers and run them

# Configuring the services

### Plex

If you already had a plex library you can follow their [guide](https://support.plex.tv/articles/201370363-move-an-install-to-another-system/) to migrate it to docker

Otherwise it's just regular plex, configure it normally

### Radarr & Sonarr

Everything can be configured normally

When it comes to adding the torrent client just add it normally but changing the host from `localhost` to `transmission`, or however you named the container

### Bazarr

Once again, regular configuration but when connecting it to radarr and sonarr changing the host from `localhost` to the container name, `radarr` and `sonarr` for example

### Prowlarr

Same stuff, configure normally but changing `localhost` to container names of other apps, `radarr`, `sonarr` and `transmission`

### Transmission and Tautulli

Just regular configuration
