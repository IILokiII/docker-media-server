## Intro
Plex, Radarr, Sonarr, Bazarr, Prowlarr, Transmission and Tautulli (optional) running on Docker with encrypted Google Drive volume via rclone

## Requirements

  - Running Linux (Windows not tested)
  - [Docker engine & compose plugin installed](https://docs.docker.com/engine/install/)

## Guide

[Install rclone Docker plugin](https://rclone.org/docker/)

Follow official guide or follow these steps:

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
  
  
