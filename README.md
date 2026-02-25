# docker-media-server
It contains different services for media server on linux in docker

# Portainer
It is a lightweight service for manage Docker, Containers
https://hub.docker.com/r/portainer/portainer-ce
url = https://192.168.18.200:9443
setup password at first run

# Filebrowser
It provides a file managing interface in web browser
https://hub.docker.com/r/filebrowser/filebrowser
url = http://192.168.18.200:8095
----------------------------------------------------------------------
get password from this container's log
adnan@adnan-media:~/docker-media-server$ docker container ls

CONTAINER ID  IMAGE                     COMMAND             CREATED         STATUS
b2b9e7758195  filebrowser/filebrowser   "tini -- /init.sh"  6 minutes ago   Up 6 minutes (healthy)

PORTS                                     NAMES
0.0.0.0:8095->80/tcp, [::]:8095->80/tcp  filebrowser

adnan@adnan-media:~/docker-media-server$ docker container logs b2b9e7758195
2026/02/02 16:10:02 Using config file: /config/settings.json
2026/02/02 16:10:02 WARNING: filebrowser.db can't be found. Initialing in /database/
2026/02/02 16:10:02 Using database: /database/filebrowser.db
2026/02/02 16:10:02 Performing quick setup
2026/02/02 16:10:02 User 'admin' initialized with randomly generated password: tzWScy33ggSKiM1K
2026/02/02 16:10:02 Listening on [::]:80
adnan@adnan-media:~/docker-media-server$
----------------------------------------------------------------------

use this password to login
http://192.168.18.200:8095
change the password in Settings -> User Management -> edit user
create new user say test => test123

# Mount media directory
adnan@adnan-media:~/docker$ sudo chmod 777 -R /mnt
adnan@adnan-media:/mnt$ mkdir data
adnan@adnan-media:/mnt$ sudo chmod 777 -R /mnt/data
find out the media hard disk attached 
adnan@adnan-media:/mnt/data/media$ lsblk
NAME                      MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0 931.5G  0 disk
└─sda1                      8:1    0 931.5G  0 part
nvme0n1                   259:0    0 119.2G  0 disk
├─nvme0n1p1               259:1    0     1G  0 part /boot/efi
├─nvme0n1p2               259:2    0     2G  0 part /boot
└─nvme0n1p3               259:3    0 116.2G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0 116.2G  0 lvm  /
our required disk is sda1, now mount it to /mnt/data
adnan@adnan-media:/mnt/data/media$ sudo mount /dev/sda1 /mnt/data

To make this mount permanent
adnan@adnan-media:/mnt/data$ sudo nano /etc/fstab
Add following line in the file
/dev/sda1	/mnt/data	ext4	defaults,nofail	0	0

verify it with following command
adnan@adnan-media:~$ sudo findmnt --verify
none
   [W] non-bind mount source /swap.img is a directory or regular file
   [W] your fstab has been modified, but systemd still uses the old version;
       use 'systemctl daemon-reload' to reload
0 parse errors, 0 errors, 2 warnings

# Transmission
A Fast, Easy and Free Bittorrent Client For macOS, Windows and Linux
url = http://192.168.18.200:9091

run this command so transmission can easily access files without issue
adnan@adnan-media:~/docker$ sudo chmod 777 -R /home/adnan/docker-media-server
adnan@adnan-media:~/docker$ sudo chmod 777 -R /temp-downloads

Install transmission app in android to easy access and give this url and your select username and password
when added a torrent in transmission add destination folder as "/downloads/complete/movies" for movies and "/downloads/complete/series" for series, this will help jellyfin the figure out where to show which video

# Jellyfin
Jellyfin is the volunteer-built media solution that puts you in control of your media. Stream to any device from your own server, with no strings attached. Your media, your server, your way.
url = http://192.168.18.200:8096/web/#/home
Add new user password

Before Add libraries install plugins
Add library Movies for path /mnt/data/media/movies
Add library Series for path /mnt/data/media/series

Plugins
- Open subtitles
- TheTVDB
- jellyfin-plugin-auto-collections
-- https://github.com/KeksBombe/jellyfin-plugin-auto-collections
-- In Jellyfin, go to Dashboard -> Plugins -> Catalog
-- Add repository: @KeksBombe (Auto Collections)
-- Repository URL: https://raw.githubusercontent.com/KeksBombe/jellyfin-plugin-auto-collections/refs/heads/main/manifest.json
-- Click "Save"
-- Search for "Auto Collections" and install
-- Restart Jellyfin

# ariang
AriaNg is a modern, web-based frontend designed to make the aria2 command-line download utility easier to use
url = http://192.168.18.200:443
popup will ask for username password
use user (AG_USERNAME) and password (AG_PASSWORD) from .env file
click on AriaNg Settings -> RPC (192.168.18.200:443)
Add value AG_SECRET from .env in field "Aria2 RPC Secret Token"
Add new link to download the file

# qbittorrent
The qBittorrent project aims to provide an open-source software alternative to µTorrent.
url = http://192.168.18.200:8080
open qbittorrent in browser from url above
----------------------------------------------------------------------
get password from this container's log
adnan@adnan-media:~/docker-media-server$ docker container ls

CONTAINER ID   IMAGE
22b396d698bd   lscr.io/linuxserver/qbittorrent:latest

adnan@adnan-media:~/docker-media-server$ docker container logs 22b396d698bd
[custom-init] No custom files found, skipping...
WebUI will be started shortly after internal preparations. Please wait...

******** Information ********
To control qBittorrent, access the WebUI at: http://localhost:8080
The WebUI administrator username is: admin
The WebUI administrator password was not set. A temporary password is provided for this session: bjzfv7UjJ
You should set your own password in program preferences.
Connection to localhost (::1) 8080 port [tcp/http-alt] succeeded!
[ls.io-init] done.
----------------------------------------------------------------------
use above password to login into qBittorrent
when interface loaded
click on setting icon to open options
goto webui tab
in Authentication section, change the password to test123
switch on the option "Bypass authentication for clients on localhost"
goto downloads tab
switch on the "Keep incomplete torrents in:" with value = "/downloads/incomplete"
click save
logout qbittorrent and login again with new password

add permissions otherwise you will get error when running torrent in qbittorrent
adnan@adnan-media:/$ sudo chmod 777 -R /temp-downloads
#adnan@adnan-media:/$ sudo chmod 777 -R /temp-downloads/qbittorrent


# flaresolverr
url = http://192.168.18.200:8191
open url and check response
{"msg": "FlareSolverr is ready!", "version": "3.4.6", "userAgent": "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/142.0.0.0 Safari/537.36"}

# prowlarr
Prowlarr is a indexer manager/proxy built on the popular arr .net/reactjs base stack to integrate with your various PVR apps. Prowlarr supports both Torrent Trackers and Usenet Indexers. It integrates seamlessly with Sonarr, Radarr, Lidarr, and Readarr offering complete management of your indexers with no per app Indexer setup required (we do it all).
url = http://192.168.18.200:9696
open url in browser
Authentication Required form will open
Authentication Method = Forms (Login Page)
Authentication Required = enabled
Username = admin
Password = test123
fill the above values and press save

go to Settings -> Indexer Proxies
click on plus icon to add "Add Indexer Proxy"
click on FlareSolverr
Name = FlareSolverr
Tags = flare
Host = http://flaresolverr:8191/
click on test, and see if test button shows a green tick
save indexerr

go to Indexers
click on Add Indexer
select a public indexer, e.g. xxxx
popup will open
Name = xxxx
Enable = true
Redirect = 
Sync Profile = Standard
Base url = https://xxxx.xx/
Seed Ratio = 1
Filter by Uploader = 
Download link = 
Download link (fallback) = 
Disable sorting = 
Sort requested from site = 
Order requested from site = 
Tags = flare
click on test, and see if test button shows a green tick
save 

go to Settings -> Apps
click on add application
popup will show, select Radarr
Name = Radarr
Sync Level = Full Sync
Tags = flare
Prowlarr Server = http://prowlarr:9696
Radarr Server = http://radarr:7878
API Key = get api key from radarr application
click on test, and see if test button shows a green tick
save 

go to Indexers
click on Add Indexer
select a public indexer, e.g. xxxx
popup will open
Name = xxxx
Enable = true
Redirect = 
Sync Profile = Standard
Base url = https://xxxx.xx/
Seed Ratio = 1
Filter by Uploader = 
Download link = 
Download link (fallback) = 
Disable sorting = 
Sort requested from site = 
Order requested from site = 
Tags = flare
click on test, and see if test button shows a green tick
save 

go to Settings -> Apps
click on add application
popup will show, select Sonarr
Name = Sonarr
Sync Level = Full Sync
Tags = flare
Prowlarr Server = http://prowlarr:9696
Radarr Server = http://sonarr:8989
API Key = get api key from radarr application
click on test, and see if test button shows a green tick
save 

# radarr
url = http://192.168.18.200:7878/
open url in browser
Authentication Required form will open
Authentication Method = Forms (Login Page)
Authentication Required = enabled
Username = admin
Password = test123
fill the above values and press save

go to settings -> general
look for api key say xxxxxxxxxxxxxxxx
we will use this api key in prowlarr

go to settings -> Media Management
under root folders section click add root folder for movies
in my case root folder will be = /movies/
Rename Movies = true
click on save changes

go to settings -> download clients
click on add button
select qBittorrent
popup will open
Name = qBittorrent
Enable = true
Host = qbittorrent
Port = 8080
Use SSL = check off
Username = admin 
Password = test123
Category = movies
click on test, and see if test button shows a green tick
save 

go to movies
add new
search for a movie say the mummy
popup will open
select root folder will already selected
monitor = movie only
minimum availability = released
quality profile = select a quality of movie you want to download
tags = leave it empty
start search for missing movie check box switch on
and click add movie
the movie will be added in the list
the movie will also be added in the qbittorrent to download
check and confirm qbittorrent

# sonarr
url = http://192.168.18.200:8989/
open url in browser
Authentication Required form will open
Authentication Method = Forms (Login Page)
Authentication Required = enabled
Username = admin
Password = test123
fill the above values and press save

go to settings -> general
look for api key say xxxxxxxxxxxxxxxx
we will use this api key in prowlarr

go to settings -> Media Management
under root folders section click add root folder for movies
in my case root folder will be = /downloads/series
Rename Movies = true
click on save changes

go to settings -> download clients
click on add button
select qBittorrent
popup will open
Name = qBittorrent
Enable = true
Host = qbittorrent
Port = 8080
Use SSL = check off
Username = admin 
Password = test123
Category = series
click on test, and see if test button shows a green tick
save 

go to series
add new
search for a series say star fleet
popup will open
select root folder will already selected
monitor = missing episodes
quality profile = select a quality of movie you want to download
tags = leave it empty
start search for missing episodes check box switch on
and click add series
the series will be added in the list
the series will also be added in the qbittorrent to download
check and confirm qbittorrent