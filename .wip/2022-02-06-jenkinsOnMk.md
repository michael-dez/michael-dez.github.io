et up Jenkins on Minikube
## Create Namespace
```bash
kubectl create namespace jenkins
# verify with
kubectl get namespaces
`````````````

## Install Helm
```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Configure Persistent Volume
Size is set to 20, I set to 15. I also made a directory to store the configs and anything related, ~/k8s/
```bash
curl -o ~/k8s/jenkins-volume.yaml https://raw.githubusercontent.com/jenkins-infra/jenkins.io/master/content/doc/tutorials/kubernetes/installing-jenkins-on-kubernetes/jenkins-volume.yaml
# make configuration changes before proceeding
kubectl apply -f jenkins-volume.yaml
# ssh to minikube
minikube ssh
# I believe docs are missing the step to create the folder, added below
sudo mkdir /data/jenkins-volume
# correct folder permissions
sudo chown -R 1000:1000 /data/jenkins-volume
```



## Reference
[https://www.jenkins.io/doc/book/installing/kubernetes/](https://www.jenkins.io/doc/book/installing/kubernetes/)
[https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

