# Home_RaspberryPi

A tutorial on how to setup different services on a Raspberry Pi. The services covered in this tutorial are [Pihole](https://github.com/pi-hole/pi-hole), [Nextcloud](https://github.com/nextcloud/docker), [Gitea](https://gitea.io/en-us/), [Portainer](https://github.com/portainer/portainer) and [Nginx Proxy Manager](https://github.com/jc21/nginx-proxy-manager). Most services can be installed on a singular device but port conflicts will occur when multiple services need access to port `80`, which is the case here with Pihole and Nginx Proxy Manager (install these on seperate devices). This setup was executed with a Raspberry Pi 4 Model B 4GB.

<p align="center">
  <a aria-label="Docker_shield" href="https://github.com/docker" target="_blank">
    <img alt="Run on Docker" src="https://img.shields.io/badge/Docker-2CA5E0?style=for-the-badge&logo=docker&logoColor=white" target="_blank" />
  </a>
  <a aria-label="Raspberry_Pi_shield" href="https://www.raspberrypi.com/" target="_blank">
    <img alt="Build on Raspberry Pi" src="https://img.shields.io/badge/Raspberry%20Pi-A22846?style=for-the-badge&logo=Raspberry%20Pi&logoColor=white" target="_blank"     />
  </a>
</p>

## Prerequisites

- Buy a domain name.
- Set up a static IP address for your device (`ifconfig` could help find network information).
- Set up port forwarding for port `80` and `443`.

## Install OS

1. Download [Ubuntu Server (64-bit)](https://ubuntu.com/download/raspberry-pi) on the Micro SD card with [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

2. Add an `ssh` file to the installed OS (alternatively use Ctrl-Shift-X to open advanced settings and enable `ssh`).

3. Start up Raspberry Pi and ssh into it or connect directly (default username:`ubuntu` and default password:`ubuntu`).

4. When you first login it will ask you to change the password, do so then connect again.

5. Have the system packages up to date.

```
sudo apt update && sudo apt upgrade
```

5. (Optional) Change the timezone configuration.

```
sudo dpkg-reconfigure tzdata
```

## Install Docker

1. Install Docker through install script.

```
curl -sSL https://get.docker.com | sh
```

2. (Optional) Check if Docker is installed.

```
docker -v
```

3. Give the current user permissions.

```
sudo usermod -aG docker ${USER}
```

4. (Optional) Check if the user is added.

```
groups ${USER}
```

5. Enable the Docker system service to start containers on boot.

```
sudo systemctl enable docker
```

6. Install Docker compose.

```
sudo apt install docker-compose
```

7. Reboot the Raspberry Pi to let the changes take effect.

```
sudo reboot
```

## Setup portainer

[Portainer](https://www.portainer.io/) is a useful tool for setting up and managing docker containers.

```
cd ~
mkdir portainer
cd portainer
```

Create a `docker-compose.yml` file in the `portainer` directory with following content.

```
version: "3"

services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce
    ports:
      - "8000:8000"
      - "9000:9000"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "portainer_data:/data"
    restart: always
```

Create the container.

```
docker-compose up -d
```

Navigate to `http://<pi-ip-address>:9000`. Setup the admin username and password. In the next page, select `Docker` as the container environment to manage. You will now be brought to the portainer homepage.

## Setup Nginx Proxy Manager

We are using [Nginx Proxy Manager](https://nginxproxymanager.com/) to make it easy to setup a reverse proxy and create SSL certificates.

```
docker network create proxy
cd ~
mkdir reverse-proxy
cd reverse-proxy
```

Create a `docker-compose.yml` file in the `reverse-proxy` directory with following content.

```
version: "3"

networks:
  proxy:
    external: true

services:
  reverse-proxy:
    image: "jc21/nginx-proxy-manager:latest"
    restart: always
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    environment:
      DB_SQLITE_FILE: "/data/database.sqlite"
      DISABLE_IPV6: "true"
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
    networks:
      - proxy
```

Create the container.

```
docker-compose up -d
```

Navigate to `http://<pi-ip-address>:81` and login (email: `admin@example.com` and password: `changeme`).  
You will immediately be prompted to change these.

## Add storage drives

To get extra storage with your nextcloud installation you can add HDD's and/or SSD's to the pi. These storage drives can also be used in a [raid setup](https://en.wikipedia.org/wiki/Standard_RAID_levels) to build in redundancy and/or improve speed. Here I will explain how to add a HDD drive and setup raid 1. Other setups are quite similar but not discussed here any further.

### Partition drives

1. Insert the drive into the raspberry pi through an usb adapter.

<div align="center">
  <img height="300" width="auto" src="https://ae01.alicdn.com/kf/Hce474db8189f405c9ed0138f880d36e1s/Ugreen-Sata-Usb-Adapter-Usb-3-0-2-0-Naar-Sata-3-Kabel-Converter-Cabo-Voor.jpg">
  <p>The adapter I used.</p>
</div>

2. (Optional) check if drives are correctly plugged in.

```
cd /dev
ll sd*
```

3. Select the storage disk you want to create partitions on and open the `fdisk` partition creator (the name of my drive here is `sda`).

```
sudo fdisk /dev/sda
```

4. Run through the partition creator (see picture below for an overview of all action commands).
    1. Create a partition with `n`.  
    2. Use `+` followed by a desired partition size with `G` = Gigabyte or `M`= Megabyte at the end to determine partition size, like `+12G`.  
    3. Write the created partition to the drive with `w`.  

<div align="center">
  <img height="300" width="auto" src="https://www.howtogeek.com/wp-content/uploads/2012/02/Screenshot-at-2012-02-25-04_06_44.png?trim=1,1&bg-color=000&pad=1,1">
</div>

5. Repeat for other drives.
6. Format the partition.

```
sudo mkfs -t ext4 /dev/sda1
```

7. (Optional) Some extra helpfull commands for drive information.

```
sudo fdisk -l
lsblk
```

### Create Raid partition

1. Create raid 1 volume.

```
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sda1 /dev/sdb1
```

2. Format the partition.

```
sudo mkfs.ext4 -v -m .1 -b 4096 -E stride=32,stipe-width=64 /dev/md0
```

3. Mount drive.

```
sudo mount /dev/md0 /mnt/raid1
```

4. Add the following text to the `fstab` file in the `/etc` folder so that the drive is mounted on bootup.

```
/dev/md0  /mnt/raid1  ext4  defaults  0 0
```

## Setup Nextcloud

```
cd ~
mkdir nextcloud
cd nextcloud
mkdir conf.d
```

Create a `docker-compose.yml` file in the `nextcloud` directory with following content (change the `PASSWORD` values).

```
version: '3'

networks:
  frontend:
    external:
      name: proxy
  backend:

services:

  nextcloud-app:
    image: nextcloud
    restart: always
    volumes:
      - /mnt/raid1/nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD=PASSWORD
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=nextcloud-db
      - PHP_UPLOAD_LIMIT=10G
      - PHP_MEMORY_LIMIT=512M
    networks:
      - frontend
      - backend

  nextcloud-db:
    image: mariadb
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW --innodb-file-per-table=1 --skip-innodb-read-only-compressed
    volumes:
      - /mnt/raid1/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=PASSWORD
      - MYSQL_PASSWORD=PASSWORD
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
    networks:
      - backend
```

Create the container.

```
docker-compose up -d
```

## Set up the proxy and SSL certificates

Navigate back to the Nginx Proxy Manager Web UI and set up a SSL certificate for your domain using web UI on the `SSL Certificates` tab. This will only be possible once the domain has **propagated**.

From the `Hosts` tab, set up a new proxy host for your domain and point it to the hostname `nextcloud-app` on port `80` (external port is 82, but the internal one which is used is still 80). Note as the docker compose was set up using the external network the containers can resolve themselves using their names. On the `SSL` tab, select the certificate created earlier and then select the `Force SSL`, `HTTP/2 Support` and `HSTS Enabled` options.

## Configure Nextcloud

You should now be able to navigate to your domain and the login screen should pop up! You should also now see a secure connection using a lets encrypt certificate. Create the admin user and untick the Install recommended apps, for performance reasons using the pi, install only what you need using the nextcloud web gui later. After a short while, and possibly a couple of page refreshes, you should be able to log in and upload/download files from the nextcloud web gui.

### Extra configuration steps

Log into the console for the nextcloud container, if you installed portainer this can be done through the portainer web ui by navigating to the container and clicking the console icon. Otherwise use `docker exec -it nextcloud_nextcloud-app_1 /bin/bash`.

- use `apt update && apt upgrade` to get the newest packages.
- use `apt install nano` to install nano.

To remove the php-imagick warning you can install libmagickcore-dev:

- use `apt install libmagickcore-dev`

To remove svg warning you can install libmagickcore-6.q16-6-extra:

- use `apt-get install libmagickcore-6.q16-6-extra`

To set the default phone region and to allow access from nextcloud sync apps, use nano to add the following details to the `config.php` file found in `/var/www/html/config`.

```
'default_phone_region' => 'COUNTRY-CODE',
'overwrite.cli.url' => 'https://<your-domain>',
'overwritehost' => '<your-domain>',
'overwriteprotocol' => 'https',
```

For caldav and carddav, add the following to the Custom Nginx Configuration in the `advanced` tab of the `nextcloud proxy` set up in the `Nginx Proxy Manager`.

```
location /.well-known/carddav {
  return 301 https://<your-domain>/remote.php/dav;
}

location /.well-known/caldav {
  return 301 https://<your-domain>/remote.php/dav;
}
```

## Setup Pi-Hole

Pi-Hole is a network-wide ad blocker.

```
cd ~
mkdir pihole
cd pihole
```

Create a `docker-compose.yml` file in the `pihole` directory with following content (change the `PASSWORD` values).

```
version: "3"

services:
  # More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    # ports already used
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "80:80/tcp"
    environment:
      TZ: Europe/Amsterdam
      WEBPASSWORD: PASSWORD
      WEBTHEME: default-darker
    # Volumes store your data between container upgrades
    volumes:
      - "./etc-pihole/:/etc/pihole/"
      - "./etc-dnsmasq.d/:/etc/dnsmasq.d/"
    # Recommended but not required (DHCP needs NET_ADMIN)
    # https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
```

Create the container.

```
docker-compose up -d
```

You should now be able to access the web gui on port `80` of the network and login with the `PASSWORD`. Add `<pi-ip-address>` to the DNS Server record in your router settings to enable network-wide adblocking.

### Potential bug

Port `53` may be occupied by the `systemd-resolve` service. Changing the pihole ports does not work. The easiest way to fix this bug is by changing the `systemd-resolve` service, this can be done by editing `/etc/systemd/resolved.conf` to the following (here 1.1.1.1 is the Cloudflare DNS).

```
[Resolve]
DNS=1.1.1.1
#FallbackDNS=
#Domains=
#LLMNR=no
#MulticastDNS=no
#DNSSEC=no
#DNSOverTLS=no
#Cache=no
DNSStubListener=no
#ReadEtcHosts=yes
```

Saving this file and rebooting the system should fix the bug.

## Update container

To update the software when a newer version is available you first need to update the docker image if this is possible. For this you need to remove the container. Saved data of the container will remain, however adjustments that where made inside the container need to be executed again (like the Nextcloud extra configuration steps). To check for the current images that are installed run:

```
docker images list
```

Pull the latest version of the docker image you want to update:

```
docker pull <image-name>
```

Stop the active container by running the following command:

```
docker stop <container-name>
```

Remove the container by running the following command:

```
docker rm <container-name>
```

Now reinitiate the container with the newer image by rerunning the docker-compose command (make sure you are located in the folder with the correct `docker-compose.yml`):

```
docker-compose up -d
```
