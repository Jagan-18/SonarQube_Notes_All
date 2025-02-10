

sudo apt update 

sudo apt install docker.io -y

sudo usermod -aG docker ubuntu

 newgrp docker

docker pull hello-world

---
```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

### Breakdown:
- `docker run`: Runs the container.
- `-d`: Detached mode (runs in the background).
- `--name sonar`: Names the container "sonar".
- `-p 9000:9000`: Maps the container’s port 9000 to your machine’s port 9000 (default port for SonarQube).
- `sonarqube:lts-community`: Uses the Long-Term Support (LTS) version of SonarQube with the community edition.

---

docker ps



### To start the container that has exited  flow the blow steps:
docker ps -a (or) docker ps -a | grep sonar

docker start sonar
docker ps

-----
# Set upping the Community branch plugin with SonarQube
- Open the google browser and seach Community branch plugin with SonarQube and select the github line

https://github.com/mc1arke/sonarqube-community-branch-plugin

### Before install plugin we need check container – sonar properties.
docker ps

docker exec -it <container ID> /bin/bash
ls
cd extensions
ls
cd plugins

---

---
#### If we need to set up through a Docker container, we need to remove the old container from the server machine.
---