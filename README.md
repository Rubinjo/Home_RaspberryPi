# Home_RaspberryPi

## Setup pi

1. Install Raspberry Pi OS (Lite) on the Micro SD card with [Raspberry Pi Imager](https://www.raspberrypi.com/software/).

2. Add an `ssh` file to the installed OS.

3. Start up Raspberry Pi and ssh into it or connect directly.

4. Change default credentials (username:`pi` and password:`raspberry`).

```
passwd
```

4. Have the system packages up to date.

```
sudo apt update && sudo apt upgrade
```

5. (Optional) Check configuration settings like Localisation Options -> Timezone.

```
sudo raspi-config
```

## Setup containerization

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

6. Reboot the Raspberry Pi to let the changes take effect.

```
sudo reboot
```

7. Install Docker compose via pip.

```
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
sudo pip3 install docker-compose
```

8. Create a network so that containers can communicate.

```
docker network create nextcloud_network
```

9. Install git.

```
sudo apt install git
```

10. Clone this Github repository.

```
git clone https://github.com/Rubinjo/Home_RaspberryPi.git
```

11. Change credentials in the [docker-compose](https://github.com/Rubinjo/Home_RaspberryPi/blob/main/docker-compose.yml) file.

```
cd Home_RaspberryPi
nano docker-compose.yml
```

12. Run the [docker-compose](https://github.com/Rubinjo/Home_RaspberryPi/blob/main/docker-compose.yml) file to create the containers.

```
docker-compose up -d
```

Sources:
https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo
