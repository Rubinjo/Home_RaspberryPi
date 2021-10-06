# Home_RaspberryPi

## Setup

1. Have the system up to date.

```
sudo apt update && sudo apt upgrade
```

2. Install docker through install script.

```
curl -sSL https://get.docker.com | sh
```

3. Install docker compose via pip.

```
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
sudo pip3 install docker-compose
```

4. Enable the docker system service to start containers on boot.

```
sudo systemctl enable docker
```

https://dev.to/elalemanyo/how-to-install-docker-and-docker-compose-on-raspberry-pi-1mo

5. Create a network so that containers can communicate.

```
docker network create nextcloud_network
```

6. Run the [docker-compose](https://github.com/Rubinjo/Home_RaspberryPi/blob/main/docker-compose.yml) file to create the containers.

```
docker-compose up -d
```
