---
sidebar_position: 2
---

# Setup docker

- [Docker](https://www.docker.com/) : A container is a standard software unit that bundles code and all its dependencies so that the application runs quickly and reliably from one computing environment to another.

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```
```bash
sudo apt update
sudo apt install \
    ca-certificates \
    curl \
    gnupg
```
```bash
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
```
```bash
echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## alias helpers

```bash
# dans .bashrc ou .zshrc
alias dip="docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}'"
alias dkill="docker kill $(docker ps -q)"
alias drm="docker rm $(docker ps -qa)"
alias dr="USER_ID=$(id -u) GROUP_ID=$(id -g) docker-compose run --rm"
```

```bash
# exemples
# dr [service] command

dr php bin/console c:c
dr node yarn install
```

[Read More](https://docs.docker.com/engine/install/ubuntu/)