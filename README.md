# docker-media-server
It contains different services for media server on linux in docker

# Portainer
It is a lightweight service for manage Docker, Containers
https://hub.docker.com/r/portainer/portainer-ce
url = http://192.168.18.200:9000

# Filebrowser
It provides a file managing interface in web browser
https://hub.docker.com/r/filebrowser/filebrowser
url = http://192.168.18.200:8095

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


use this password to login
http://192.168.18.200:8095
change the password in Settings -> User Management -> edit user
create new user say test => test123

Note:-
adnan@adnan-media:~/docker$ sudo chmod 777 -R /mnt/data
So files can be moved easily

# Mount media directory
adnan@adnan-media:~/docker$ sudo chmod 777 -R /mnt
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
adnan@adnan-media:~/docker$ sudo chmod 777 -R /temp-downloads

Install transmission app in android to easy access and give this url and your select username and password
when added a torrent in transmission add destination folder as "/downloads/complete/movies" for movies and "/downloads/complete/series" for series, this will help jellyfin the figure out where to show which video

# Jellyfin
Jellyfin is the volunteer-built media solution that puts you in control of your media. Stream to any device from your own server, with no strings attached. Your media, your server, your way.
url = http://192.168.18.200:8096/web/#/home
Add new user password
Add library Movies for path /mnt/data/media/movies
Add library Series for path /mnt/data/media/series

