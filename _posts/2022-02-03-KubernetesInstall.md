---
title: "Minikube Setup on EC2"
---

The plan right now is to migrate my FoundryVTT and Project Zomboid servers to Kubernetes and run a Jenkins pod to learn more about CI/CD pipelines. It would also really save me time to automate httpd configs and ssl cert management in the near future. Anyway, I only want a single node so I'm using Minikube and in this post I'll be installing it.

## Install Docker
```bash
# install
sudo amazon-linux-extras install docker
# start the service
sudo service docker start
# set the service to start at boot
sudo systemctl enable docker
```

## Install Minikube (RPM)
Install
```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-latest.x86_64.rpm
```
```bash
sudo rpm -Uvh minikube-latest.x86_64.rpm
```

## Alias Kubectl (Optional)
Using Minikube's version of `kubectl` mean's typing out `minikube kubectl --` every time. Make an alias instead.
### bash
```bash
echo 'alias k="minikube kubectl --"' >> ~/.bashrc && source ~/.bashrc
```
### zsh
```bash
echo 'alias k="minikube kubectl --"' >> ~/.zshrc && source ~/.zshrc
```

## References
[1](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html) https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

[2](https://minikube.sigs.k8s.io/docs/start/) https://minikube.sigs.k8s.io/docs/start/
