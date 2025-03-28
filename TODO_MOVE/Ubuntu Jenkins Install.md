---
title: Ubuntu Jenkins Install
tags: [devops, jenkins]

---

# Ubuntu Jenkins Install

```shell=
sudo apt-get update
sudo apt-get upgrade

sudo apt install openjdk-11-jdk

java -version

curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo tee   /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]   https://pkg.jenkins.io/debian-stable binary/ | sudo tee   /etc/apt/sources.list.d/jenkins.list > /dev/null


sudo apt-get update
sudo apt-get install jenkins



```

## 安裝 docker

* [官方教學](https://docs.docker.com/engine/install/ubuntu/)

> 解決 Docker: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```shell=
sudo usermod -a -G docker jenkins
```
重啟jenkins


## Jenkins 常用指令

```shell=
# daemon-reload:
sudo systemctl daemon-reload

# You can start the Jenkins service with the command:
sudo systemctl start jenkins

# You can check the status of the Jenkins service using the command:
sudo systemctl status jenkins
```

## remove jenkins
```bash=
sudo apt-get remove --purge jenkins
```

## Jenkins Run docker Permission denied
```bash=
# Running

sudo usermod -aG docker jenkins

# and then
sudo service jenkins restart
```

## add docker permision
```shell=
sudo usermod -a -G docker jenkins
```

## change port
First open the /etc/default/jenkins file.
Then under JENKINS_ARGS section, you can change the port like this HTTP_PORT=9999.

Then you should restart Jenkins with sudo service jenkins restart.

Then to check the status use this command sudo systemctl status jenkins

## 分布式 Node 設定

### slave docker cmd
```shell
docker run -d -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker --name jenkins_agent --init jenkins/inbound-agent -url http://139.162.65.19:8080/ -workDir=/home/jenkins/agent c429f2d0d8a946da3035bfd40e4dc3f1ef0ffcc36e85082ad31afd5f16c05651  jenkinsFlowAgent

docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker --name jenkins_agent --init jenkins/inbound-agent -url http://139.162.65.19:8080/ -workDir=/home/jenkins/agent c429f2d0d8a946da3035bfd40e4dc3f1ef0ffcc36e85082ad31afd5f16c05651  jenkinsFlowAgent

docker run -d --restart always -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker --name jenkins_agent --init jenkins/inbound-agent -url http://139.162.65.19:8080/ -workDir=/home/jenkins/agent c429f2d0d8a946da3035bfd40e4dc3f1ef0ffcc36e85082ad31afd5f16c05651  jenkinsFlowAgent


```

###### tags: `devops` `jenkins`